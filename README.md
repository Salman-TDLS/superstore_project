"""
ga_airbnb_analysis.py
---------------------
Step-by-step data-cleaning + exploratory analysis pipeline
for GA_Project_01.xlsx (Chicago Airbnb listings).

Inspired by the Superstore-in-Excel process :contentReference[oaicite:0]{index=0}, but implemented in Python 🐍.

Usage:
    python ga_airbnb_analysis.py

Requires:
    pandas, matplotlib, openpyxl
"""

import pandas as pd
import matplotlib.pyplot as plt
from pathlib import Path


def load_data(path: str | Path, sheet: str = "listings_raw") -> pd.DataFrame:
    """Load a sheet from the Excel workbook."""
    return pd.read_excel(path, sheet_name=sheet)


# ----------------------------- 1. Cleaning Data ----------------------------- #
def clean_listings(df: pd.DataFrame) -> pd.DataFrame:
    """
    Basic data-quality fixes:
      • Drop duplicates – keep first occurrence of each listing ID  
      • Handle missing values – nix rows missing neighbourhood or price  
      • Convert data types – strip currency symbols, cast price → float, etc.  
      • Normalise headers – lower-snake-case, no spaces
    """
    df = df.drop_duplicates(subset="id")
    df = df.dropna(subset=["neighbourhood_cleansed", "price"])

    # tidy column names
    df.columns = (
        df.columns.str.strip()
        .str.lower()
        .str.replace(" ", "_")
        .str.replace("(", "", regex=False)
        .str.replace(")", "", regex=False)
    )

    # parse price "$1,234.00" → 1234.00
    df["price"] = (
        df["price"]
        .astype(str)
        .str.replace(r"[^0-9.]", "", regex=True)
        .astype(float)
    )

    # availability_365 → int
    if "availability_365" in df.columns:
        df["availability_365"] = pd.to_numeric(
            df["availability_365"], errors="coerce"
        ).fillna(0)

    return df


# ---------------------------- 2. Data Analysis ------------------------------ #
def summarise_metrics(df: pd.DataFrame) -> dict:
    """
    Generate pivot-style summaries analogous to the Excel dashboard.
    Returns a dict of DataFrames.
    """
    metrics = {}

    # Listing count by neighbourhood
    metrics["listings_by_neighbourhood"] = (
        df.groupby("neighbourhood_cleansed")["id"]
        .nunique()
        .sort_values(ascending=False)
        .to_frame("listing_count")
    )

    # Review distribution by room_type
    if {"room_type", "number_of_reviews"}.issubset(df.columns):
        metrics["reviews_by_room_type"] = (
            df.groupby("room_type")["number_of_reviews"]
            .sum()
            .sort_values(ascending=False)
            .to_frame("total_reviews")
        )

    # Revenue by host (top 10)
    if "estimated_revenue" in df.columns:
        metrics["top_hosts_revenue"] = (
            df.groupby("host_id")["estimated_revenue"]
            .sum()
            .nlargest(10)
            .to_frame("total_estimated_revenue")
        )

    # Average price by property_type
    if "property_type" in df.columns:
        metrics["avg_price_by_property_type"] = (
            df.groupby("property_type")["price"]
            .mean()
            .sort_values(ascending=False)
            .to_frame("avg_price")
        )

    return metrics


def plot_top_neighbourhoods(df: pd.DataFrame, top_n: int = 10) -> None:
    """Bar-chart of listing density per neighbourhood."""
    top = (
        df.groupby("neighbourhood_cleansed")["id"]
        .nunique()
        .sort_values(ascending=False)
        .head(top_n)
    )

    plt.figure(figsize=(10, 5))
    top.plot(kind="bar")
    plt.title(f"Top {top_n} Chicago Neighbourhoods by Active Listings")
    plt.ylabel("Listing Count")
    plt.tight_layout()
    plt.show()


# --------------------------- 3. Insights (console) -------------------------- #
def print_insights(df: pd.DataFrame) -> None:
    """Quick narrative highlights, à la README ‘Insights’ section."""
    hood_counts = df.groupby("neighbourhood_cleansed")["id"].nunique()
    top_hood = hood_counts.idxmax()
    top_hood_cnt = hood_counts.max()

    room_reviews = df.groupby("room_type")["number_of_reviews"].sum()
    fav_room_type = (room_reviews / room_reviews.sum()).idxmax()
    fav_room_share = (room_reviews / room_reviews.sum()).max() * 100

    if "estimated_revenue" in df.columns:
        host_revenue = df.groupby("host_id")["estimated_revenue"].sum()
        top_host = host_revenue.idxmax()
        top_host_rev = host_revenue.max()
    else:
        top_host = top_host_rev = None

    print("\nKey Insights")
    print("-" * 40)
    print(f"• Most active neighbourhood: {top_hood} ({top_hood_cnt:,} listings)")
    print(f"• Guest preference: {fav_room_share:,.1f}% of reviews for {fav_room_type}")
    if top_host:
        print(f"• Revenue outlier: host {top_host} ≈ ${top_host_rev:,.0f}/yr")
    print("-" * 40)


# ----------------------------- Driver script -------------------------------- #
def main() -> None:
    wb_path = Path("GA_Project_01.xlsx")
    if not wb_path.exists():
        raise FileNotFoundError("GA_Project_01.xlsx not found in the current directory.")

    raw = load_data(wb_path, sheet="listings_raw")
    listings = clean_listings(raw)

    # Persist cleaned CSV for Tableau/Excel follow-ups
    listings.to_csv("listings_clean.csv", index=False)

    # Export summary pivots
    metrics = summarise_metrics(listings)
    for name, table in metrics.items():
        table.to_csv(f"{name}.csv")
        print(f"Saved {name}.csv")

    # Quick visual & narrative
    plot_top_neighbourhoods(listings)
    print_insights(listings)


if __name__ == "__main__":
    main()
