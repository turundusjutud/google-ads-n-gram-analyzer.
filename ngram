import streamlit as st
import pandas as pd
import plotly.express as px
import numpy as np
from collections import Counter

# --- PAGE CONFIG ---
st.set_page_config(
    page_title="Turundusjutud | Google Ads N-Gram Auditor", 
    page_icon="🔎",
    layout="wide"
)

# --- CUSTOM BRANDING CSS ---
# Colors: Dark Green #052623, Teal #1A776F, Orange #FF7F40, Bg #FAFAFA
st.markdown("""
    <style>
        /* Main Background */
        .stApp {
            background-color: #FAFAFA;
            color: #052623;
        }
        
        /* Typography */
        h1, h2, h3, h4, h5, h6, p, div, label, span, li, button {
            color: #052623;
            font-family: 'Aftika', 'Helvetica Neue', 'Helvetica', 'Arial', sans-serif !important;
        }
        
        /* Headings */
        h1, h2, h3 {
            font-weight: 800;
        }
        
        /* Metric Cards */
        div[data-testid="stMetric"] {
            background-color: #FFFFFF;
            padding: 15px;
            border-radius: 10px;
            box-shadow: 0 4px 6px -1px rgba(0,0,0,0.05);
            border: 1px solid #E5E7EB;
            border-left: 5px solid #1A776F; /* Brand Teal */
        }
        div[data-testid="stMetric"] label {
            color: #1A776F !important;
            font-size: 0.9rem;
        }
        div[data-testid="stMetric"] div[data-testid="stMetricValue"] {
            color: #052623 !important;
            font-weight: 700;
        }

        /* Buttons (Brand Orange) */
        div.stButton > button {
            background-color: #FF7F40;
            color: white !important;
            border-radius: 8px;
            border: none;
            padding: 0.5rem 1.5rem;
            font-weight: bold;
            transition: all 0.2s;
        }
        div.stButton > button:hover {
            background-color: #E66A2E;
            color: white !important;
            box-shadow: 0 4px 12px rgba(230, 106, 46, 0.2);
        }

        /* Tables */
        div[data-testid="stDataFrame"] {
            border: 1px solid #E5E7EB;
            border-radius: 8px;
        }
        
        /* Tabs */
        .stTabs [data-baseweb="tab-list"] {
            gap: 10px;
        }
        .stTabs [data-baseweb="tab"] {
            height: 50px;
            background-color: #FFFFFF;
            border-radius: 8px;
            color: #052623;
            border: 1px solid #E5E7EB;
        }
        .stTabs [aria-selected="true"] {
            background-color: #1A776F !important;
            color: white !important;
        }
        
        /* Info/Success Boxes */
        div.stAlert {
            background-color: #F0FDFA;
            border: 1px solid #1A776F;
            color: #052623;
        }
    </style>
""", unsafe_allow_html=True)

# --- INSTRUCTIONS ---
with st.expander("📝 How to export data from Google Ads (Read this first!)", expanded=False):
    st.markdown("""
    To find hidden wasted spend, we need your **Search Terms Report**.
    
    1. Go to **Google Ads** -> **Insights & Reports** -> **Search Terms**.
    2. Set Date Range to **Last 90 Days** (or longer).
    3. Ensure these columns are visible:
       * `Search term`
       * `Cost`
       * `Conversions`
       * `Conv. value`
       * `Impressions`
       * `Clicks`
    4. **Download** -> **CSV**.
    5. Upload that file on the left.
    """)

st.title("Search Query N-Gram Auditor")
st.markdown("Find the hidden patterns in your search terms that are secretly wasting your budget.")

# --- HELPER FUNCTIONS ---
@st.cache_data
def load_and_clean_data(file):
    try:
        # Skip first two rows (Google Ads header junk)
        df = pd.read_csv(file, skiprows=2)
        
        # Clean Column Names
        df.columns = [c.lower().strip() for c in df.columns]
        
        # Map Google's weird column names to standard ones
        col_map = {
            'search term': 'term',
            'cost': 'cost',
            'conversions': 'conversions',
            'conv. value': 'value',
            'clicks': 'clicks',
            'impressions': 'imps'
        }
        
        # Filter only needed columns
        existing_cols = [c for c in df.columns if c in col_map.keys()]
        df = df[existing_cols].rename(columns=col_map)
        
        # Clean Numeric Data (Remove currency symbols, commas)
        numeric_cols = ['cost', 'conversions', 'value', 'clicks', 'imps']
        for c in numeric_cols:
            if c in df.columns:
                if df[c].dtype == 'object':
                    df[c] = df[c].astype(str).str.replace(r'[^\d.]', '', regex=True)
                    df[c] = pd.to_numeric(df[c], errors='coerce').fillna(0)
        
        # Drop total row (usually the last one or first one)
        df = df[df['term'] != 'Total']
        
        return df
    except Exception as e:
        return None

def generate_ngrams(df, n=1):
    # Dictionary to store aggregated stats
    ngram_stats = {}
    
    for index, row in df.iterrows():
        term = str(row['term']).lower()
        words = term.split()
        
        # Generate N-grams for this row
        if len(words) >= n:
            row_ngrams = [' '.join(words[i:i+n]) for i in range(len(words)-n+1)]
            
            # Add stats to dictionary
            for grams in row_ngrams:
                if grams not in ngram_stats:
                    ngram_stats[grams] = {'cost': 0, 'conversions': 0, 'value': 0, 'clicks': 0, 'imps': 0, 'count': 0}
                
                stats = ngram_stats[grams]
                stats['cost'] += row.get('cost', 0)
                stats['conversions'] += row.get('conversions', 0)
                stats['value'] += row.get('value', 0)
                stats['clicks'] += row.get('clicks', 0)
                stats['imps'] += row.get('imps', 0)
                stats['count'] += 1
                
    # Convert to DataFrame
    ngram_df = pd.DataFrame.from_dict(ngram_stats, orient='index').reset_index()
    ngram_df.rename(columns={'index': 'ngram'}, inplace=True)
    
    # Calculate KPIs
    ngram_df['cpa'] = np.where(ngram_df['conversions'] > 0, ngram_df['cost'] / ngram_df['conversions'], 0)
    ngram_df['roas'] = np.where(ngram_df['cost'] > 0, ngram_df['value'] / ngram_df['cost'], 0)
    ngram_df['cpc'] = np.where(ngram_df['clicks'] > 0, ngram_df['cost'] / ngram_df['clicks'], 0)
    ngram_df['ctr'] = np.where(ngram_df['imps'] > 0, (ngram_df['clicks'] / ngram_df['imps']) * 100, 0)
    
    return ngram_df

# --- MAIN APP ---
uploaded_file = st.sidebar.file_uploader("Upload Search Terms CSV", type=['csv'])

if uploaded_file is not None:
    df = load_and_clean_data(uploaded_file)
    
    if df is not None and not df.empty:
        st.sidebar.success("✅ Data Processed")
        
        # --- SUMMARY METRICS ---
        total_spend = df['cost'].sum()
        total_conv = df['conversions'].sum()
        total_value = df['value'].sum()
        account_roas = total_value / total_spend if total_spend > 0 else 0
        
        c1, c2, c3, c4 = st.columns(4)
        c1.metric("Total Spend", f"€{total_spend:,.0f}")
        c2.metric("Conversions", f"{total_conv:,.0f}")
        c3.metric("Account ROAS", f"{account_roas:.2f}x")
        c4.metric("Analyzed Terms", f"{len(df):,}")
        
        st.markdown("---")
        
        # --- N-GRAM GENERATION ---
        # Generate all 3 levels at once
        grams_1 = generate_ngrams(df, n=1)
        grams_2 = generate_ngrams(df, n=2)
        grams_3 = generate_ngrams(df, n=3)
        
        # --- ANALYSIS TABS ---
        tab1, tab2 = st.tabs(["💸 Wasted Spend Auditor", "🏆 High Performance Finder"])
        
        # --- TAB 1: WASTED SPEND (ZERO CONVERSIONS) ---
        with tab1:
            st.header("Where are you burning money?")
            st.caption("These words appear in search terms that spent money but produced **0 conversions**.")
            
            # Filter for waste
            waste_df = grams_1[grams_1['conversions'] == 0].sort_values('cost', ascending=False)
            top_waste = waste_df.head(10)
            
            if not top_waste.empty:
                # Calculate Potential Savings
                potential_savings = waste_df['cost'].sum()
                st.warning(f"🚨 **Potential Savings:** You spent **€{potential_savings:,.0f}** on words with 0 conversions.")
                
                # Bar Chart
                fig_waste = px.bar(
                    top_waste, 
                    x='cost', 
                    y='ngram', 
                    orientation='h',
                    title="Top 10 Costly Words with ZERO Results",
                    labels={'cost': 'Wasted Spend (€)', 'ngram': 'Word'},
                    text_auto='.0f'
                )
                fig_waste.update_traces(marker_color='#FF7F40') # Brand Orange
                fig_waste.update_layout(yaxis=dict(autorange="reversed"), paper_bgcolor='rgba(0,0,0,0)', plot_bgcolor='rgba(0,0,0,0)', font=dict(color='#052623'))
                st.plotly_chart(fig_waste, use_container_width=True)
                
                # Actionable Table
                st.subheader("📋 Negative Keyword Candidates")
                st.dataframe(
                    waste_df[['ngram', 'cost', 'clicks', 'cpc']].style.format({'cost': '€{:.2f}', 'cpc': '€{:.2f}'}),
                    use_container_width=True
                )
            else:
                st.success("✅ Incredible! No single word has 0 conversions. Your targeting is very tight.")

        # --- TAB 2: N-GRAM EXPLORER ---
        with tab2:
            st.header("Deep Dive: Performance by Phrase")
            
            # Sub-tabs for 1, 2, 3 grams
            sub_t1, sub_t2, sub_t3 = st.tabs(["1-Word (Single)", "2-Words (Pairs)", "3-Words (Phrases)"])
            
            def show_ngram_table(data, n):
                # Filter out low data
                min_clicks = st.slider(f"Min Clicks ({n}-gram)", 0, 100, 5, key=f"s_{n}")
                filtered = data[data['clicks'] >= min_clicks]
                
                # Styling
                st.dataframe(
                    filtered[['ngram', 'cost', 'conversions', 'roas', 'cpa', 'clicks']].sort_values('cost', ascending=False).style.format({
                        'cost': '€{:.2f}',
                        'roas': '{:.2f}x',
                        'cpa': '€{:.2f}',
                        'conversions': '{:.0f}'
                    }).background_gradient(subset=['roas'], cmap='Greens'),
                    use_container_width=True
                )
            
            with sub_t1:
                st.caption("Single words appearing in your search terms.")
                show_ngram_table(grams_1, 1)
                
            with sub_t2:
                st.caption("Word pairs. Great for finding specific intent (e.g., 'buy nike' vs 'nike reviews').")
                show_ngram_table(grams_2, 2)
                
            with sub_t3:
                st.caption("Longer phrases. Helps identify specific product questions or intent.")
                show_ngram_table(grams_3, 3)

    else:
        st.error("Error reading file. Please make sure it's the standard Google Ads Search Terms CSV.")
else:
    st.info("👈 Upload your Search Terms Report to start the audit.")
