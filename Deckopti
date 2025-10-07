import streamlit as st
import pandas as pd
from io import BytesIO
import matplotlib.pyplot as plt

def calculate_required_cuts(deck_length, deck_width, joist_spacing, include_borders):
    # Main deck: Rows across width, each row = deck_length
    num_rows = int(deck_width / joist_spacing) + 1  # Round up for full coverage
    main_cuts = [deck_length] * num_rows  # Each row needs full length
    
    # Borders: 2x long sides, 2x short sides
    border_cuts = []
    if include_borders:
        border_cuts = [deck_length, deck_length, deck_width, deck_width]  # Two of each
    
    return main_cuts + border_cuts

def greedy_first_fit_decreasing(required_lengths, stock_length, kerf):
    required = sorted(required_lengths, reverse=True)  # Longest first
    boards = []
    while required:
        board_remaining = stock_length
        current_cuts = []
        i = 0
        while i < len(required):
            if required[i] + kerf <= board_remaining:
                current_cuts.append(required[i])
                board_remaining -= (required[i] + kerf)
                required.pop(i)
            else:
                i += 1
        if current_cuts:
            boards.append(current_cuts)
    total_waste = sum(stock_length - (sum(cuts) + (len(cuts)-1)*kerf) for cuts in boards)
    waste_pct = (total_waste / (len(boards) * stock_length)) * 100 if boards else 0
    return boards, waste_pct

# Streamlit UI
st.title("Decking Board Optimizer")
st.header("Input Deck Details")
deck_length = st.number_input("Deck Length (m)", value=6.0, min_value=0.1, step=0.1)
deck_width = st.number_input("Deck Width (m)", value=4.0, min_value=0.1, step=0.1)
joist_spacing = st.number_input("Joist Spacing (m)", value=0.4, min_value=0.1, step=0.01)
include_borders = st.checkbox("Include Border Boards", value=True)
stock_length = st.number_input("Stock Board Length (m)", value=4.8, min_value=0.1, step=0.1)
kerf = st.number_input("Kerf (Saw Width, m)", value=0.003, min_value=0.0, step=0.001)

if st.button("Optimize Cuts"):
    # Calculate cuts
    required_cuts = calculate_required_cuts(deck_length, deck_width, joist_spacing, include_borders)
    boards, waste_pct = greedy_first_fit_decreasing(required_cuts, stock_length, kerf)
    
    # Display results
    st.header("Results")
    st.write(f"**Boards Needed**: {len(boards)}")
    st.write(f"**Waste**: {waste_pct:.1f}%")
    
    # Cut List Table
    df = pd.DataFrame({
        "Board #": range(1, len(boards)+1),
        "Cuts (m)": [", ".join(f"{cut:.2f}" for cut in cuts) for cuts in boards]
    })
    st.dataframe(df)
    
    # Text-based Diagram
    st.header("Board Layout")
    for i, cuts in enumerate(boards, 1):
        layout = f"Board {i}: {' + '.join(f'{cut:.2f}m' for cut in cuts)}"
        if len(cuts) > 1:
            layout += f" (Waste: {stock_length - sum(cuts) - (len(cuts)-1)*kerf:.2f}m)"
        st.text(layout)
    
    # Simple Visualization
    st.header("Visual Layout")
    fig, ax = plt.subplots(figsize=(8, len(boards)*0.5))
    for i, cuts in enumerate(boards):
        pos = 0
        for cut in cuts:
            ax.barh(i, cut, left=pos, height=0.4, color="skyblue")
            pos += cut + kerf
        ax.barh(i, stock_length - pos, left=pos, height=0.4, color="lightgray", alpha=0.5)
    ax.set_ylim(-0.5, len(boards)-0.5)
    ax.set_xlim(0, stock_length)
    ax.set_xlabel("Length (m)")
    ax.set_ylabel("Board")
    st.pyplot(fig)
    
    # Export CSV
    csv = df.to_csv(index=False)
    st.download_button("Download Cut List", csv, "deck_cut_list.csv", "text/csv")
