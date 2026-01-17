# 🔎 Google Ads Search Query N-Gram Auditor

## What is this?
This is a **Python-based audit tool** built with Streamlit. It solves the biggest problem in Google Ads management: **"Death by 10,000 rows."**

Analyzing a Search Terms report line-by-line is impossible for large accounts. This tool uses **N-Gram Analysis** to break every search query down into individual words (1-grams) and phrases (2-grams & 3-grams) to find hidden performance patterns.

## Key Features
* **💸 Wasted Spend Auditor:** Instantly aggregates cost for words that have generated **0 conversions**. (e.g., find out that the word *"free"* appeared in 500 different queries and wasted €1,200).
* **🏆 High-Performance Finder:** Identifies the specific phrases that drive the highest ROAS and lowest CPA.
* **Deep Dive Explorer:** Filter by click volume to ignore statistical noise and focus on significant data.

## How to Use
### Step 1: Export Data from Google Ads
To get the correct format, follow these steps exactly:
1.  Log in to **Google Ads**.
2.  Go to **Insights & Reports** -> **Search Terms**.
3.  Set the Date Range to **Last 90 Days** (or longer for better data).
4.  Ensure these columns are visible in the table:
    * `Search term`
    * `Cost`
    * `Conversions`
    * `Conv. value`
    * `Impressions`
    * `Clicks`
5.  Click **Download** -> **CSV**.

### Step 2: Analyze
1.  Open the tool (Local or Hosted URL).
2.  Drag and drop your CSV file into the sidebar.
3.  Switch between the **"Wasted Spend"** and **"Deep Dive"** tabs to find negative keyword opportunities.

## Tech Stack
* **Python 3.10+**
* **Streamlit** (Frontend)
* **Pandas** (Data Processing)
* **Plotly** (Visualizations)

## License
Private tool built for **Turundusjutud.ee**.
