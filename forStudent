import streamlit as st
import gspread
from google.oauth2.service_account import Credentials
import pandas as pd
import json

# Set page layout and title
st.set_page_config(
    page_title="Student Dashboard - Angle Belearn",
    page_icon="🎓",
    layout="wide",
    menu_items={
        "Get Help": None,
        "Report a bug": None,
        "About": None
    }
)

# Function to load credentials from Streamlit secrets
def load_credentials_from_secrets():
    try:
        credentials_info = json.loads(st.secrets["google_credentials_new_project"]["data"])
        return credentials_info
    except KeyError:
        st.error("Google credentials not found in Streamlit secrets.")
        return None

# Function to connect to Google Sheets
def connect_to_google_sheets(spreadsheet_id, worksheet_name):
    credentials_info = load_credentials_from_secrets()
    if not credentials_info:
        return None
    
    scopes = [
        "https://www.googleapis.com/auth/spreadsheets",
        "https://www.googleapis.com/auth/drive",
        "https://www.googleapis.com/auth/drive.file"
    ]
    
    try:
        credentials = Credentials.from_service_account_info(
            credentials_info,
            scopes=scopes
        )
        client = gspread.authorize(credentials)
        sheet = client.open_by_key(spreadsheet_id).worksheet(worksheet_name)
        return sheet
    except gspread.exceptions.SpreadsheetNotFound:
        st.error(f"Spreadsheet with ID '{spreadsheet_id}' not found. Check the spreadsheet ID and permissions.")
    except gspread.exceptions.WorksheetNotFound:
        st.error(f"Worksheet '{worksheet_name}' not found in the spreadsheet. Verify the worksheet name.")
    except Exception as e:
        st.error(f"Unexpected error connecting to Google Sheets: {e}")
    return None

# Function to fetch data from Google Sheets
def fetch_data_from_sheet(spreadsheet_id, worksheet_name):
    sheet = connect_to_google_sheets(spreadsheet_id, worksheet_name)
    if not sheet:
        st.warning(f"Could not establish a connection to the worksheet '{worksheet_name}'.")
        return pd.DataFrame()
    try:
        data = sheet.get_all_values()
        if data:
            headers = pd.Series(data[0]).fillna('').str.strip()
            headers = headers.where(headers != '', other='Unnamed')
            headers = headers + headers.groupby(headers).cumcount().astype(str).replace('0', '')
            df = pd.DataFrame(data[1:], columns=headers)
            df.replace('', pd.NA, inplace=True)
            df.ffill(inplace=True)
            if 'Hr' in df.columns:
                df['Hr'] = pd.to_numeric(df['Hr'], errors='coerce').fillna(0)
            return df
        else:
            st.warning(f"No data found in worksheet '{worksheet_name}'.")
            return pd.DataFrame()
    except gspread.exceptions.APIError as api_error:
        st.error(f"Google Sheets API error fetching data from '{worksheet_name}': {api_error}")
    except Exception as e:
        st.error(f"Error fetching data from '{worksheet_name}': {e}")
    return pd.DataFrame()

# Function to handle student data
def handle_student_data(data):
    st.subheader("Student Dashboard")

    if "MM" in data.columns:
        month = st.selectbox("Select Month", sorted(data["MM"].unique()))
    else:
        st.warning("Month data ('MM' column) not found. Available columns are:")
        st.write(data.columns.tolist())
        return

    student_id = st.text_input("Enter Your Student ID").strip().lower()
    student_name_part = st.text_input("Enter any part of your name (minimum 4 characters)").strip().lower()

    if st.button("Verify Student"):
        filtered_data = data[(data["MM"] == month) & 
                             (data["Student ID"].str.lower().str.strip() == student_id) & 
                             (data["Student"].str.lower().str.contains(student_name_part))]

        if not filtered_data.empty:
            # Display student's name at the top
            student_name = filtered_data["Student"].iloc[0]
            st.subheader(f"👨‍🎓 Welcome, {student_name}!")

            # Check for required columns
            required_columns = ["Date", "Subject", "Hr", "Teachers Name", "Chapter taken", "Type of class"]
            missing_columns = [col for col in required_columns if col not in filtered_data.columns]

            if missing_columns:
                st.error(f"The following required columns are missing: {missing_columns}")
                st.write("Available columns in filtered_data:", filtered_data.columns.tolist())
            else:
                # Select relevant columns for display
                filtered_data = filtered_data[required_columns]
                st.subheader("📚 Your Monthly Class Data")
                st.write(filtered_data)

                # Calculate total hours
                total_hours = filtered_data["Hr"].sum()
                st.write(f"**Total Hours for {month}th month:** {total_hours:.2f}")

                # Subject-wise breakdown
                subject_hours = filtered_data.groupby("Subject")["Hr"].sum().reset_index()
                subject_hours = subject_hours.rename(columns={"Hr": "Total Hours"})
                st.subheader("📊 Subject-wise Hour Breakdown")
                st.write(subject_hours)
        else:
            st.error("Verification failed. Please check your details.")

# Main function
def main():
    st.image("https://anglebelearn.kayool.com/assets/logo/angle_170x50.png", width=170)
    st.title("Angle Belearn: Student Dashboard")

    # Refresh Data button
    if st.sidebar.button("Refresh Data"):
        # Clear cached data if it exists to ensure a fresh fetch
        st.session_state.data = fetch_data_from_sheet(
            "17_Slyn6u0G6oHSzzXIpuuxPhzxx4ayOKYkXfQTLtk-Y", "Student Data"
        )
        st.success("Data refreshed successfully!")

    # Load data if it is not already in session state
    if "data" not in st.session_state:
        st.session_state.data = fetch_data_from_sheet(
            "17_Slyn6u0G6oHSzzXIpuuxPhzxx4ayOKYkXfQTLtk-Y", "Student Data"
        )

    # Display student data
    if not st.session_state.data.empty:
        handle_student_data(st.session_state.data)
    else:
        st.warning("No data available. Please check the data source.")

if __name__ == "__main__":
    main()
