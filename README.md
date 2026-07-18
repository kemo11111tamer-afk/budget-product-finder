# budgeimport streamlit as st
import pandas as pd

# --- PAGE CONFIGURATION ---
st.set_page_config(
    page_title="BudgetFinder | Smart Matcher",
    page_icon="💰",
    layout="centered",
    initial_sidebar_state="expanded"
)

# --- CUSTOM CSS FOR REFINED LOOK ---
st.markdown("""
<style>
    .main { background-color: #fcfcfc; }
    h1 { color: #1E293B; font-weight: 800; }
    .product-card {
        padding: 20px;
        border-radius: 12px;
        background-color: #ffffff;
        box-shadow: 0 4px 6px -1px rgba(0,0,0,0.05), 0 2px 4px -1px rgba(0,0,0,0.03);
        margin-bottom: 20px;
        border-left: 5px solid #3B82F6;
    }
    .premium-card {
        padding: 15px;
        border-radius: 10px;
        background-color: #FFFBEB;
        border: 1px dashed #F59E0B;
        margin-top: 10px;
    }
</style>
""", unsafe_allow_html=True)

# --- IN-MEMORY DATABASE (2026 EGP Estimates) ---
@st.cache_data
def load_product_data():
    products = [
        # E-Scooters
        {"category": "E-Scooters", "name": "Xiaomi Electric Scooter 3 Lite", "price": 17500, "specs": "250W Motor, 20km range, Sleek folding design", "vendor": "B.TECH", "installments": "2,916 EGP/mo x 6 mos (0%)"},
        {"category": "E-Scooters", "name": "Smart Gate Jet 350W", "price": 14000, "specs": "350W Motor, 25km range, Dual rear suspension", "vendor": "Noon Egypt", "installments": "2,333 EGP/mo x 6 mos (0%)"},
        {"category": "E-Scooters", "name": "Xiaomi Electric Scooter 4 Pro", "price": 34000, "specs": "700W Max Power, 55km ultra range, Self-sealing tubeless tires", "vendor": "Amazon Egypt", "installments": "5,666 EGP/mo x 6 mos (0%)"},
        
        # Smartphones
        {"category": "Smartphones", "name": "Redmi Note 14 Pro", "price": 11500, "specs": "8GB RAM, 256GB Storage, 120Hz AMOLED, 200MP Main Camera", "vendor": "Raya Shop", "installments": "1,916 EGP/mo x 6 mos (0%)"},
        {"category": "Smartphones", "name": "Samsung Galaxy A17", "price": 8200, "specs": "6GB RAM, 128GB Storage, High-brightness display, 5000mAh battery", "vendor": "Aman Stores", "installments": "1,366 EGP/mo x 6 mos (0%)"},
        {"category": "Smartphones", "name": "Samsung Galaxy S26 Ultra", "price": 68000, "specs": "12GB RAM, 512GB, Galaxy AI Engine, 200MP Zoom, Titanium Frame", "vendor": "Tradeline", "installments": "11,333 EGP/mo x 6 mos (0%)"},
        
        # AirPods & TWS
        {"category": "AirPods & Audio", "name": "Anker Soundcore P20i", "price": 1200, "specs": "10mm Drivers, Big Bass, 30-Hour total playtime, IPX5 Waterproof", "vendor": "Noon Egypt", "installments": "N/A (Under threshold)"},
        {"category": "AirPods & Audio", "name": "Oraimo FreePods 4", "price": 2100, "specs": "Active Noise Cancellation (ANC), 35.5-hr playtime, Low latency game mode", "vendor": "Amazon Egypt", "installments": "350 EGP/mo x 6 mos (0%)"},
        {"category": "AirPods & Audio", "name": "Apple AirPods Pro (2nd Gen)", "price": 13500, "specs": "H2 Chip, Advanced ANC, Adaptive Audio, USB-C Charging Case", "vendor": "Tradeline", "installments": "2,250 EGP/mo x 6 mos (0%)"},
        
        # TVs
        {"category": "TVs", "name": "LG 43\" UHD 4K Smart TV", "price": 14500, "specs": "HDR10 Pro, WebOS Smart Platform, AI Sound Processor", "vendor": "B.TECH", "installments": "2,416 EGP/mo x 6 mos (0%)"},
        {"category": "TVs", "name": "Samsung 55\" QLED 4K", "price": 26000, "specs": "Quantum Dot Color 100%, Quantum Processor Lite, AirSlim Design", "vendor": "Raya Shop", "installments": "4,333 EGP/mo x 6 mos (0%)"},
        
        # Refrigerators
        {"category": "Refrigerators", "name": "Kiriazi 14 Feet No Frost", "price": 19500, "specs": "No Frost system, Multi-layer cooling air flow, Anti-fungal door gasket", "vendor": "Cairo Sales", "installments": "3,250 EGP/mo x 6 mos (0%)"},
        {"category": "Refrigerators", "name": "LG Digital Inverter 410L", "price": 39000, "specs": "Smart Inverter Compressor (10y warranty), Door Cooling+, Linear Cooling", "vendor": "B.TECH", "installments": "6,500 EGP/mo x 6 mos (0%)"}
    ]
    return pd.DataFrame(products)

df = load_product_data()

# --- SIDEBAR NAV & CONFIG ---
st.sidebar.title("💎 App Control Panel")
app_mode = st.sidebar.radio("Navigate App Screens", ["🔍 Product Matcher", "👑 Manage Premium Membership"])

if "is_premium" not in st.session_state:
    st.session_state.is_premium = False

if st.session_state.is_premium:
    st.sidebar.success("Account Status: PREMIUM PRO")
else:
    st.sidebar.warning("Account Status: FREE TIER")

# --- SCREEN 1: PRODUCT MATCHER ---
if app_mode == "🔍 Product Matcher":
    st.title("💰 Smart Budget Product Finder")
    st.write("Enter your absolute target budget and select a shopping vertical to view your best optimized matches.")
    
    col1, col2 = st.columns([1, 1])
    with col1:
        user_budget = st.number_input("Your Maximum Budget (EGP):", min_value=1000, max_value=150000, value=20000, step=500)
    with col2:
        selected_category = st.selectbox("Product Category:", df["category"].unique())
        
    st.markdown("---")
    
    filtered_df = df[(df["category"] == selected_category) & (df["price"] <= user_budget)]
    filtered_df = filtered_df.sort_values(by="price", ascending=False)
    
    if not filtered_df.empty:
        st.subheader(f"✨ Top Recommendations for {selected_category} Under {user_budget:,} EGP")
        
        for idx, row in filtered_df.iterrows():
            st.markdown(f"""
            <div class="product-card">
                <h3 style='margin:0; color:#1F2937;'>{row['name']}</h3>
                <p style='color:#059669; font-weight:bold; font-size:1.1rem; margin:5px 0;'>Price: {row['price']:,} EGP</p>
                <p style='color:#4B5563; font-size:0.95rem; line-height:1.4;'><b>Key Specifications:</b> {row['specs']}</p>
            </div>
            """, unsafe_allow_html=True)
            
            if st.session_state.is_premium:
                st.markdown(f"""
                <div class="premium-card">
                    <span style='color:#D97706; font-weight:bold;'>📍 PREMIUM INSIGHT: BEST PLACE TO BUY</span><br>
                    <table style='width:100%; margin-top:8px; border-collapse:collapse;'>
                        <tr>
                            <td style='padding:4px 0; color:#374151;'><b>Recommended Retailer:</b></td>
                            <td style='padding:4px 0; color:#111827;'>{row['vendor']}</td>
                        </tr>
                        <tr>
                            <td style='padding:4px 0; color:#374151;'><b>Installment Plan Options:</b></td>
                            <td style='padding:4px 0; color:#111827;'>{row['installments']}</td>
                        </tr>
                        <tr>
                            <td style='padding:4px 0; color:#374151;'><b>Warranty Validation:</b></td>
                            <td style='padding:4px 0; color:#059669;'>✓ Verified Local Dealer Warranty</td>
                        </tr>
                    </table>
                </div>
                """, unsafe_allow_html=True)
            else:
                st.info("🔒 Unlock Premium Mode from the sidebar panel to uncover verified vendor names, current stock allocations, and monthly installment payment structures.")
            
            st.write("")
            
    else:
        st.error(f"No products found in {selected_category} under {user_budget:,} EGP. Try adjusting your budget ceiling or switching categories.")

# --- SCREEN 2: PREMIUM MANAGEMENT ---
elif app_mode == "👑 Manage Premium Membership":
    st.title("👑 Premium Account Upgrade")
    st.write("Maximize your purchasing value by enabling the elite localization engine.")
    
    if st.session_state.is_premium:
        st.success("🎉 You have unlocked Premium Access! Head back to the Product Matcher to view local vendors and installment matrices.")
        if st.button("Downgrade Account to Free Tier"):
            st.session_state.is_premium = False
            st.rerun()
    else:
        st.markdown("""
        ### What's Included in Premium?
        *   **Dynamic Vendor Scraping:** Instantly map your product to the cheapest available local vendor (Amazon, Noon, B.TECH, etc.).
        *   **Installment Calculator Integrations:** Review 0% interest monthly payment terms across services like Valu, Aman, or local credit cards.
        *   **Stock Tracking Checkers:** Ensure the items aren't completely sold out before visiting physical branch locations.
        """)
        
        if st.button("Unlock All Premium Features (Simulate Upgrade)"):
            st.session_state.is_premium = True
            st.success("Upgrade successful!")
            st.rerun()t-product-finder
