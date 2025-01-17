import streamlit as st
from datetime import datetime, timedelta
import requests
from bs4 import BeautifulSoup
from typing import List, Dict

def get_tennis_court_data(target_date: datetime) -> List[Dict]:
    """Henter ledige tennistider for indendørsbaner på en given dato."""
    url = "https://herlevtennis.halbooking.dk/newlook/proc_baner.asp"
    date_str = target_date.strftime('%d-%m-%Y')
    current_time = datetime.now().strftime('%d-%m-%Y%H:%M:%S')
    
    base_payload = {
        'banedato': date_str,
        'banedato_mindate': datetime.now().strftime('%d-%m-%Y'),
        'soeg_omraede': '2',
        'visallebaner': '',
        'mf_owlitems': '0,1,2,3,4,5',
        'ligenu': current_time,
        'mf_mbmenu': '1',
        'mf_menu': '1'
    }
    
    session = requests.Session()
    
    # 1. Vælg indendørsbaner
    indoor_payload = {**base_payload, 
                     'mf_funktion': 'omr_soeg',
                     'dagvalgt': ';',
                     'mf_sidstefunk': 'soegdato###'}
    
    session.post(url, data=indoor_payload)
    
    # 2. Vælg dato
    date_payload = {**base_payload,
                   'mf_funktion': 'soegdato',
                   'dagvalgt': date_str,
                   'mf_sidstefunk': 'soegdato###'}
    
    response = session.post(url, data=date_payload)
    
    soup = BeautifulSoup(response.text, 'html.parser')
    courts = soup.find_all('div', class_='text-center bane')
    
    court_data = []
    for court in courts:
        court_name = court.find('span', class_='baneheadtxt').text.strip()
        time_slots = []
        
        for slot in court.find_all('span', class_='banefelt'):
            time_div = slot.find('div', class_='padding3')
            if not time_div:
                continue
                
            time_text = time_div.text.strip().split('\n')[0]
            
            if 'Træning' in time_div.text:
                status = 'træning'
            elif 'bane_redbg' in slot.get('class', []):
                status = 'optaget'
            elif 'bane_rest' in slot.get('class', []):
                status = 'passeret'
            else:
                status = 'ledig'
                
            time_slots.append({'time': time_text, 'status': status})
            
        if time_slots:
            court_data.append({
                'name': court_name,
                'slots': time_slots
            })
            
    return court_data

# Streamlit app
st.set_page_config(page_title="Tennis Booking - Herlev", page_icon="🎾")

st.title("🎾 Tennis Booking - Herlev")
st.subheader("Find ledige indendørs tennisbaner")

# Dato vælger
min_date = datetime.now()
max_date = min_date + timedelta(days=365)  # Et år frem
selected_date = st.date_input(
    "Vælg dato",
    min_value=min_date,
    max_value=max_date,
    value=min_date
)

if st.button("Søg ledige tider"):
    with st.spinner("Henter tider..."):
        court_data = get_tennis_court_data(selected_date)
        
        if not court_data:
            st.error("Ingen baner fundet")
        else:
            # Vis data i en pæn tabel
            for court in court_data:
                st.markdown(f"### 📍 {court['name']}")
                
                # Opret en liste af ledige tider
                ledige_tider = [
                    slot['time'] for slot in court['slots'] 
                    if slot['status'] == 'ledig'
                ]
                
                if ledige_tider:
                    for tid in ledige_tider:
                        st.markdown(f"✅ {tid}")
                else:
                    st.markdown("❌ Ingen ledige tider")
                
                st.markdown("---")

# Footer
st.markdown("---")
st.markdown("Lavet med af CH")
