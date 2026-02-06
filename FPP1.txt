import streamlit as st
import math
import numpy as np

# -----------------------------
# APP TITLE
# -----------------------------
st.title("Theme Park Fast Pass Pricing Optimizer")
st.write("""
Adjust operational inputs to estimate the optimal Fast Pass price and expected revenue.
Behavioral factors are preset and not adjustable.
""")

# -----------------------------
# USER INPUTS (editable)
# -----------------------------
st.sidebar.header("Operational Inputs")

attendance = st.sidebar.number_input(
    "Projected Attendance", min_value=1000, max_value=100000, value=30000, step=500
)
capacity = st.sidebar.number_input(
    "Park Capacity", min_value=5000, max_value=100000, value=42000, step=1000
)
hours = st.sidebar.number_input(
    "Operating Hours", min_value=1, max_value=24, value=12
)
ride_hr = st.sidebar.number_input(
    "Ride Capacity per Hour", min_value=100, max_value=20000, value=5000, step=500
)
allocation = st.sidebar.slider(
    "Fast Pass Allocation %", min_value=1, max_value=50, value=10, step=1
) / 100  # convert to decimal

# -----------------------------
# FIXED MODEL PARAMETERS
# -----------------------------
QUALITY = 0.7
MAX_PEN = 0.12
BASE_SENS = 0.018
BOOST = 2

# -----------------------------
# CALCULATIONS
# -----------------------------
# Fast pass supply
supply = hours * ride_hr * allocation

# Effective market penetration
penetration = MAX_PEN * QUALITY

# Adjusted sensitivity based on crowd
crowd_ratio = attendance / capacity
adj_sens = BASE_SENS / (1 + BOOST * crowd_ratio)

# Price search range (realistic minimum)
prices = range(0, 201)  # starts at $0, but verified math ensures revenue=0

revenues = []
sold_list = []

for p in prices:
    buyers = attendance * penetration * math.exp(-adj_sens * p)
    sold = min(buyers, supply)  # enforce supply cap
    revenue = p * sold
    revenues.append(revenue)
    sold_list.append(sold)

# Optimal price
idx = np.argmax(revenues)
opt_price = prices[idx]
opt_rev = revenues[idx]
opt_sold = sold_list[idx]

# -----------------------------
# OUTPUT RESULTS
# -----------------------------
st.header("Results")

st.metric("Fast Pass Supply", f"{int(supply):,} tickets")
st.metric("Demand Ceiling", f"{int(attendance * penetration):,} potential buyers")
st.metric("Optimal Price", f"${opt_price}")
st.metric("Expected Revenue", f"${int(opt_rev):,}")
st.metric("Tickets Sold at Optimal Price", f"{int(opt_sold):,}")

# -----------------------------
# REVENUE CHART
# -----------------------------
st.subheader("Revenue vs Price")
st.line_chart({"Revenue": revenues})
