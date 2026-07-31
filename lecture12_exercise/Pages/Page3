# pages/03_demand.py — YOUR new page (BBD squiggle level 3: demand story)
import streamlit as st
import plotly.express as px
import sys, os
sys.path.insert(0, os.path.dirname(os.path.dirname(__file__)))
from utils import load_data, sidebar_filters

# ─────────────────────────────────────────────────────────────────────────────
# Load data + shared sidebar
# One call to sidebar_filters gives you the SAME sidebar as pages 1 and 2 —
# and the filter choices carried over from them, for free.
# Then add a question title + caption.
# ─────────────────────────────────────────────────────────────────────────────
df, p95 = load_data()
df = sidebar_filters(df, p95)

st.title("Where is guest demand strongest?")
st.caption("Which listings pull in the most reviews per month, and at what price?")

# ─────────────────────────────────────────────────────────────────────────────
# A persisted widget of your own
# e.g. a radio or selectbox to focus on one room type, key='sel_room'
#   - initialise the key in session_state once
#   - keep it alive: st.session_state.sel_room = st.session_state.sel_room
#   - GUARD: if the saved value was filtered out, fall back to a valid option
#     BEFORE creating the widget
# TEST: pick a value, visit page 1, change a filter, come back — your choice
# must still be selected (or gracefully replaced if filtered out).
# ─────────────────────────────────────────────────────────────────────────────
room_options = sorted(df["room_type"].unique().tolist())

if "sel_room" not in st.session_state:
    st.session_state.sel_room = room_options[0]

# GUARD: fall back to a valid option before the widget is built
if st.session_state.sel_room not in room_options:
    st.session_state.sel_room = room_options[0]

st.selectbox("Focus on a room type", room_options, key="sel_room")

focused = df[df["room_type"] == st.session_state.sel_room]
# ─────────────────────────────────────────────────────────────────────────────
# KPI row (st.columns(3)) about the focused selection
# e.g. listings, median reviews/month vs filtered market, median price
# 5-second test: the metrics alone should answer the page's question
# ─────────────────────────────────────────────────────────────────────────────
market_median = df["reviews_per_month"].median()
focused_median = focused["reviews_per_month"].median()

col1, col2, col3 = st.columns(3)
col1.metric("Listings", f"{len(focused):,}")
col2.metric(
    "Median reviews/month",
    f"{focused_median:.1f}",
    delta=f"{focused_median - market_median:+.1f} vs market",
)
col3.metric("Median price", f"£{focused['price'].median():,.0f}")

st.divider()

# ─────────────────────────────────────────────────────────────────────────────
# One chart — demand story
# Suggestion: px.scatter of price vs reviews_per_month (reviews as a demand
# proxy), highlight column for your focused selection.
# BBD REQUIREMENTS:
#   - Name the colour type in a comment (highlight: blue vs grey)
#   - No red-green, no pies, no packed bubbles
# SWD REQUIREMENTS:
#   - White background, Arial font, insight title, use_container_width=True
# px REQUIREMENT:
#   - Start from px, highlight column + color_discrete_map — no go.Figure()
# ─────────────────────────────────────────────────────────────────────────────
plot_df = df.copy()
plot_df["highlight"] = plot_df["room_type"].where(
    plot_df["room_type"] == st.session_state.sel_room, "Other"
)

direction = "more" if focused_median >= market_median else "fewer"
fig = px.scatter(
    plot_df,
    x="price",
    y="reviews_per_month",
    color="highlight",
    color_discrete_map={st.session_state.sel_room: "#1f77b4", "Other": "#d3d3d3"},
    title=f"{st.session_state.sel_room} listings get {direction} reviews per month than the rest of the market",
)
fig.update_traces(marker=dict(opacity=0.6, size=6))
fig.update_layout(
    plot_bgcolor="white",
    paper_bgcolor="white",
    font=dict(family="Arial"),
    legend_title_text="",
)

st.plotly_chart(fig, use_container_width=True)
