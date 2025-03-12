import streamlit as st
import pandas as pd

st.title("🧹 Data Cleaning Automation 🚀")

# Step 1: Get dataset via URL or File Upload
source = st.text_input("Enter Dataset URL (or leave blank if uploading a file):")
uploaded_file = st.file_uploader("Or Upload CSV File", type=["csv"])

if source or uploaded_file:
    try:
        if source:
            df = pd.read_csv(source)
        else:
            df = pd.read_csv(uploaded_file)

        st.write("📊 **Original Dataset Preview:**", df.head())

        # Step 2: Ask for cleaning method
        method = st.radio("Choose Cleaning Mode:", ["Automatic", "Manual"])

        # Function to clean data automatically
        def clean_data(df):
            cleaning_steps = []
            original_size = df.shape[0]

            # Handling missing values
            missing_count = df.isna().sum().sum()  # Total
            for col in df.columns:                           
                if df[col].dtype == 'object':  # Categorical
                    df[col].fillna(df[col].mode()[0], inplace=True)
                else:  # Numerical
                    df[col].fillna(df[col].median(), inplace=True)
            cleaning_steps.append(f"Handled {missing_count} missing values")

            # Removing duplicates
            duplicate_count = df.duplicated().sum()
            df.drop_duplicates(inplace=True)
            cleaning_steps.append(f"Removed {duplicate_count} duplicates")

            # Handling outliers (IQR method)
            numeric_cols = df.select_dtypes(include=['number'])
            Q1 = numeric_cols.quantile(0.25)
            Q3 = numeric_cols.quantile(0.75)
            IQR = Q3 - Q1
            df = df[~((numeric_cols < (Q1 - 1.5 * IQR)) | (numeric_cols > (Q3 + 1.5 * IQR))).any(axis=1)]
            outlier_count = - df.shape[0]
            cleaning_steps.append(f"Handled {outlier_count} outliers using IQR method")

            return df, "✅ Cleaning Completed: " + ", ".join(cleaning_steps)

        if method == "Automatic":
            if st.button("Start Automatic Cleaning"):
                cleaned_df, status = clean_data(df)
                st.success(status)
                st.write("✅ **Cleaned Data Preview:**", cleaned_df.head())

                # Download option
                cleaned_df.to_csv("cleaned_data.csv", index=False)
                st.download_button("⬇️ Download Cleaned Data", data=open("cleaned_data.csv", "rb"), file_name="cleaned_data.csv")

        elif method == "Manual":
            st.write("✍️ Write your Pandas code below and execute it:")
            st.write("Your dataset was stored in the variable named df📄.")
            user_code = st.text_area("Enter your Python code here", "")

            if st.button("Run Code"):
                try:
                    exec(user_code, {"df": df, "pd": pd})
                    st.success("✅ Code executed successfully!")
                    st.write(df.head())

                    # Download option
                    df.to_csv("cleaned_data.csv", index=False)
                    st.download_button("⬇️ Download Cleaned Data", data=open("cleaned_data.csv", "rb"), file_name="cleaned_data.csv")

                except Exception as e:
                    st.error(f"⚠️ Error in your code: {e}")

    except Exception as e:
        st.error(f"⚠️ Error loading dataset: {e}")