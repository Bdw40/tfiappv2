import streamlit as st
import pandas as pd
import numpy as np
import altair as alt
from datetime import datetime, timedelta
import random

# Set page configuration
st.set_page_config(
    page_title="Digital TFI Leap Card",
    page_icon="🚌",
    layout="wide",
    initial_sidebar_state="expanded"
)

# Custom color theme - Green
green_theme = {
    'primary': '#2E8B57',  # Sea Green
    'secondary': '#3CB371',  # Medium Sea Green
    'background': '#F0FFF0',  # Honeydew
    'text': '#006400',  # Dark Green
    'highlight': '#00FF7F'  # Spring Green
}

# Custom CSS
st.markdown(f"""
<style>
    .main {{
        background-color: {green_theme['background']};
    }}
    .stButton>button {{
        background-color: {green_theme['primary']};
        color: white;
    }}
    .stButton>button:hover {{
        background-color: {green_theme['secondary']};
        color: white;
    }}
    .highlight {{
        background-color: {green_theme['highlight']};
        padding: 10px;
        border-radius: 5px;
    }}
    h1, h2, h3 {{
        color: {green_theme['primary']};
    }}
    .css-1544g2n.e1fqkh3o4 {{
        padding-top: 2rem;
    }}
    .card {{
        background-color: white;
        border-radius: 10px;
        padding: 20px;
        margin: 10px 0px;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    }}
    .tier-card {{
        background-color: white;
        border-radius: 10px;
        padding: 20px;
        margin: 10px 0px;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        transition: transform 0.3s;
    }}
    .tier-card:hover {{
        transform: translateY(-5px);
    }}
    .tier-header {{
        font-weight: bold;
        font-size: 1.2em;
        color: {green_theme['primary']};
        margin-bottom: 10px;
    }}
    .price {{
        font-size: 1.5em;
        font-weight: bold;
        color: {green_theme['primary']};
    }}
</style>
""", unsafe_allow_html=True)

# Sample transport data
@st.cache_data
def load_transport_data():
    # Generate some sample data
    dates = pd.date_range(start='2023-01-01', end='2024-03-31', freq='D')
    
    # Journey types
    journey_types = ['Bus', 'Luas', 'DART', 'Commuter Rail']
    
    data = []
    
    for date in dates:
        for journey in journey_types:
            # Create realistic patterns with weekday variations and seasonal trends
            weekday_factor = 1.0 if date.weekday() < 5 else 0.6
            month_factor = 1.0 + 0.2 * np.sin(date.month * np.pi / 6)  # Seasonal variation
            
            # Base counts by transport type
            base_counts = {
                'Bus': 125000, 
                'Luas': 80000, 
                'DART': 50000, 
                'Commuter Rail': 35000
            }
            
            # Add some randomness
            count = int(base_counts[journey] * weekday_factor * month_factor * random.uniform(0.8, 1.2))
            
            data.append({
                'Date': date,
                'Journey Type': journey,
                'Count': count
            })
    
    return pd.DataFrame(data)

# Generate sample user data
@st.cache_data
def generate_user_data():
    # Create sample user data with demographics
    age_groups = ['18-25', '26-35', '36-45', '46-55', '56-65', '65+']
    user_types = ['Commuter', 'Tourist', 'Occasional User']
    
    data = []
    
    for age in age_groups:
        for user_type in user_types:
            # Different adoption rates based on demographics
            if age in ['18-25', '26-35']:
                digital_adoption = random.uniform(0.75, 0.95)
            elif age in ['36-45', '46-55']:
                digital_adoption = random.uniform(0.60, 0.80)
            else:
                digital_adoption = random.uniform(0.30, 0.60)
            
            # Tourists generally more likely to go digital
            if user_type == 'Tourist':
                digital_adoption += 0.15
                if digital_adoption > 1:
                    digital_adoption = 0.99
                    
            data.append({
                'Age Group': age,
                'User Type': user_type,
                'Digital Adoption Rate': digital_adoption,
                'Physical Card Rate': 1 - digital_adoption,
                'Sample Size': random.randint(100, 500)
            })
    
    return pd.DataFrame(data)

# App state
class AppState:
    def __init__(self):
        self.user_data = {
            'name': '',
            'email': '',
            'card_type': 'Standard',
            'balance': 0.0,
            'has_active_pass': False,
            'active_pass_type': None,
            'pass_expiry': None,
            'recent_transactions': []
        }
        
        # Set defaults if empty
        if not self.user_data['name']:
            self.user_data['balance'] = 20.0  # Default balance
            
    def add_transaction(self, transaction_type, amount, description):
        transaction = {
            'timestamp': datetime.now(),
            'type': transaction_type,
            'amount': amount,
            'description': description
        }
        self.user_data['recent_transactions'].insert(0, transaction)
        # Keep only the last 10 transactions
        self.user_data['recent_transactions'] = self.user_data['recent_transactions'][:10]
        
    def purchase_pass(self, pass_type, duration_days, cost):
        if self.user_data['balance'] >= cost:
            self.user_data['balance'] -= cost
            self.user_data['has_active_pass'] = True
            self.user_data['active_pass_type'] = pass_type
            self.user_data['pass_expiry'] = datetime.now() + timedelta(days=duration_days)
            
            self.add_transaction('Purchase', -cost, f"Purchased {pass_type} - {duration_days} day pass")
            return True
        else:
            return False
    
    def top_up(self, amount):
        self.user_data['balance'] += amount
        self.add_transaction('Top Up', amount, f"Added €{amount:.2f} to balance")

# Initialize app state
state = AppState()

# Tier configurations
tier_config = {
    'TFI Basic': {
        'description': 'Unlimited Dublin Bus, Luas, and DART',
        'attractions': 0,
        'perks': 'Real-time route planner, transit alerts',
        'prices': {
            1: 10,
            3: 25,
            7: 40
        },
        'color': '#3CB371'  # Medium Sea Green
    },
    'TFI Explorer': {
        'description': 'Unlimited transport across all Dublin zones',
        'attractions': 3,
        'perks': 'All Basic perks + Discounts on tours & food',
        'prices': {
            1: 35,
            3: 70,
            7: 110
        },
        'color': '#228B22'  # Forest Green
    },
    'TFI Plus': {
        'description': 'Unlimited transport + Express Airport Transfer',
        'attractions': 10,
        'perks': 'All Explorer perks + VIP fast-track entry, premium dining discounts',
        'prices': {
            1: 70,
            3: 130,
            7: 180
        },
        'color': '#006400'  # Dark Green
    }
}

# App sidebar
def sidebar():
    with st.sidebar:
        st.image("https://via.placeholder.com/150x150?text=TFI+Leap", width=150)
        st.markdown(f"### Welcome, {state.user_data['name'] if state.user_data['name'] else 'Guest'}")
        
        st.markdown(f"#### Card Balance: **€{state.user_data['balance']:.2f}**")
        
        # Display active pass if any
        if state.user_data['has_active_pass']:
            days_left = (state.user_data['pass_expiry'] - datetime.now()).days
            hours_left = ((state.user_data['pass_expiry'] - datetime.now()).seconds // 3600)
            
            st.markdown("### Active Pass")
            st.markdown(f"""
            <div class="highlight">
                <strong>{state.user_data['active_pass_type']}</strong><br>
                Expires in: {days_left} days, {hours_left} hours
            </div>
            """, unsafe_allow_html=True)
        
        # Quick top-up
        st.markdown("### Quick Top-up")
        col1, col2 = st.columns(2)
        with col1:
            if st.button("€10", key="topup10"):
                state.top_up(10)
        with col2:
            if st.button("€20", key="topup20"):
                state.top_up(20)
        
        if st.button("€50", key="topup50", use_container_width=True):
            state.top_up(50)
        
        st.markdown("---")
        st.markdown("### App Navigation")
        
        app_options = [
            "Home", 
            "Buy Tourist Pass", 
            "View Transport Usage", 
            "My Account"
        ]
        
        selected_option = st.radio("", app_options)
        
        return selected_option

# Home page
def home_page():
    st.markdown("# Digital TFI Leap Card")
    
    st.markdown("""
    <div class="card">
        Welcome to the new Digital TFI Leap Card system! This app allows you to:
        <ul>
            <li>Manage your digital Leap Card</li>
            <li>Purchase tourist passes</li>
            <li>Track your transport usage</li>
            <li>Access exclusive discounts and offers</li>
        </ul>
        <p>Go paperless and enjoy seamless travel around Dublin!</p>
    </div>
    """, unsafe_allow_html=True)
    
    # Display any active passes prominently
    if state.user_data['has_active_pass']:
        st.markdown("## Your Active Pass")
        days_left = (state.user_data['pass_expiry'] - datetime.now()).days
        hours_left = ((state.user_data['pass_expiry'] - datetime.now()).seconds // 3600)
        
        st.markdown(f"""
        <div class="card" style="background-color: #e6ffe6;">
            <h3>{state.user_data['active_pass_type']}</h3>
            <p>Valid for {days_left} more days and {hours_left} hours</p>
            <p>Scan the QR code below when boarding:</p>
        </div>
        """, unsafe_allow_html=True)
        
        # Display a fake QR code
        st.image("https://via.placeholder.com/300x300?text=Scan+QR+Code", width=300)
    
    # Recent activity
    st.markdown("## Recent Activity")
    if state.user_data['recent_transactions']:
        for transaction in state.user_data['recent_transactions'][:3]:
            st.markdown(f"""
            <div class="card">
                <strong>{transaction['type']}</strong> • {transaction['timestamp'].strftime('%d %b, %H:%M')}
                <br>{transaction['description']}
                <br><span style="color: {'green' if transaction['amount'] > 0 else 'red'};">
                    {'€' + str(transaction['amount']) if transaction['type'] == 'Top Up' else '-€' + str(abs(transaction['amount']))}
                </span>
            </div>
            """, unsafe_allow_html=True)
    else:
        st.info("No recent activity to display")
    
    # Quick actions
    st.markdown("## Quick Actions")
    col1, col2 = st.columns(2)
    
    with col1:
        if st.button("📱 Add to Apple Wallet", use_container_width=True):
            st.success("Card added to Apple Wallet!")
            
    with col2:
        if st.button("📲 Add to Google Pay", use_container_width=True):
            st.success("Card added to Google Pay!")

# Tourist Pass Page
def tourist_pass_page():
    st.markdown("# TFI TravelPass+ System")
    st.markdown("Choose the perfect pass for your Dublin adventure!")
    
    # Duration selector with tabs
    st.markdown("## Select Duration")
    durations = {1: "1 Day", 3: "3 Days", 7: "7 Days"}
    duration_tabs = st.tabs(list(durations.values()))
    
    # Display tier options for each duration
    for i, days in enumerate([1, 3, 7]):
        with duration_tabs[i]:
            st.markdown(f"### {durations[days]} Pass Options")
            
            cols = st.columns(len(tier_config))
            
            for j, (tier_name, tier_data) in enumerate(tier_config.items()):
                with cols[j]:
                    st.markdown(f"""
                    <div class="tier-card" style="border-top: 5px solid {tier_data['color']}">
                        <div class="tier-header">{tier_name}</div>
                        <p>{tier_data['description']}</p>
                        <p><strong>Attractions:</strong> {'None' if tier_data['attractions'] == 0 else f'Access to {tier_data["attractions"]} attractions'}</p>
                        <p><strong>Perks:</strong> {tier_data['perks']}</p>
                        <div class="price">€{tier_data['prices'][days]}</div>
                    </div>
                    """, unsafe_allow_html=True)
                    
                    if st.button(f"Buy {tier_name}", key=f"buy_{tier_name}_{days}"):
                        result = state.purchase_pass(tier_name, days, tier_data['prices'][days])
                        if result:
                            st.success(f"Successfully purchased {tier_name} for {days} days!")
                            st.balloons()
                        else:
                            st.error(f"Insufficient balance! Please top up.")

# Transport usage page with charts
def transport_usage_page():
    st.markdown("# Transport Usage Analytics")
    
    # Load sample data
    transport_data = load_transport_data()
    
    # Overview stats
    st.markdown("## Overview")
    
    # Calculate summary statistics
    total_journeys = transport_data['Count'].sum()
    avg_daily = int(transport_data.groupby('Date')['Count'].sum().mean())
    most_popular = transport_data.groupby('Journey Type')['Count'].sum().idxmax()
    
    # Display stats in columns
    col1, col2, col3 = st.columns(3)
    with col1:
        st.metric("Total Journeys", f"{total_journeys:,}")
    with col2:
        st.metric("Avg. Daily Journeys", f"{avg_daily:,}")
    with col3:
        st.metric("Most Popular Mode", most_popular)
    
    # Transport type distribution
    st.markdown("## Transport Mode Distribution")
    
    # Group data by journey type
    mode_data = transport_data.groupby('Journey Type')['Count'].sum().reset_index()
    
    # Create the chart
    mode_chart = alt.Chart(mode_data).mark_bar().encode(
        x=alt.X('Journey Type:N', sort='-y', title='Transport Mode'),
        y=alt.Y('Count:Q', title='Number of Journeys'),
        color=alt.Color('Journey Type:N', scale=alt.Scale(scheme='greens')),
        tooltip=['Journey Type', alt.Tooltip('Count:Q', format=',')]
    ).properties(
        width='container',
        height=400,
        title='Journeys by Transport Mode'
    )
    
    st.altair_chart(mode_chart, use_container_width=True)
    
    # Monthly trends
    st.markdown("## Monthly Trends")
    
    # Group by month
    transport_data['Month'] = transport_data['Date'].dt.strftime('%Y-%m')
    monthly_data = transport_data.groupby(['Month', 'Journey Type'])['Count'].sum().reset_index()
    
    # Create line chart
    monthly_chart = alt.Chart(monthly_data).mark_line(point=True).encode(
        x='Month:T',
        y=alt.Y('Count:Q', title='Number of Journeys'),
        color=alt.Color('Journey Type:N', scale=alt.Scale(scheme='greens')),
        tooltip=['Month', 'Journey Type', alt.Tooltip('Count:Q', format=',')]
    ).properties(
        width='container',
        height=400,
        title='Monthly Transport Usage Trends'
    )
    
    st.altair_chart(monthly_chart, use_container_width=True)
    
    # Digital adoption insights
    st.markdown("## Digital Adoption Insights")
    
    # Load user data
    user_data = generate_user_data()
    
    # Demographics chart
    demo_chart = alt.Chart(user_data).mark_bar().encode(
        x='Age Group:O',
        y='Digital Adoption Rate:Q',
        color=alt.Color('User Type:N', scale=alt.Scale(scheme='greens')),
        tooltip=['Age Group', 'User Type', alt.Tooltip('Digital Adoption Rate:Q', format='.1%')]
    ).properties(
        width='container',
        height=400,
        title='Digital Adoption Rate by Demographics'
    )
    
    st.altair_chart(demo_chart, use_container_width=True)
    
    # Insights text
    st.markdown("""
    <div class="card">
        <h3>Key Insights</h3>
        <ul>
            <li>Younger users (18-35) show 80%+ digital adoption rates</li>
            <li>Tourists are the most likely group to adopt digital payments</li>
            <li>Bus remains the most popular mode of transport</li>
            <li>We project 60% of all Leap Card users will switch to digital within 3 years</li>
        </ul>
    </div>
    """, unsafe_allow_html=True)

# Account page
def account_page():
    st.markdown("# My Account")
    
    # User info
    st.markdown("## User Information")
    
    col1, col2 = st.columns(2)
    
    with col1:
        new_name = st.text_input("Name", value=state.user_data['name'])
        if new_name != state.user_data['name']:
            state.user_data['name'] = new_name
    
    with col2:
        new_email = st.text_input("Email", value=state.user_data['email'])
        if new_email != state.user_data['email']:
            state.user_data['email'] = new_email
    
    # Transaction history
    st.markdown("## Transaction History")
    
    if state.user_data['recent_transactions']:
        transactions_df = pd.DataFrame(state.user_data['recent_transactions'])
        transactions_df['timestamp'] = transactions_df['timestamp'].dt.strftime('%Y-%m-%d %H:%M')
        
        st.dataframe(
            transactions_df[['timestamp', 'description', 'amount']],
            use_container_width=True,
            hide_index=True
        )
    else:
        st.info("No transactions to display")
    
    # Manual top-up
    st.markdown("## Balance Top-up")
    
    top_up_amount = st.number_input("Amount (€)", min_value=5.0, max_value=500.0, value=20.0, step=5.0)
    
    if st.button("Add Funds", use_container_width=True):
        state.top_up(top_up_amount)
        st.success(f"Successfully added €{top_up_amount:.2f} to your balance!")

# Main app
def main():
    selected_page = sidebar()
    
    if selected_page == "Home":
        home_page()
    elif selected_page == "Buy Tourist Pass":
        tourist_pass_page()
    elif selected_page == "View Transport Usage":
        transport_usage_page()
    elif selected_page == "My Account":
        account_page()

if __name__ == "__main__":
    main()
