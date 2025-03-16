# Fraud Detection System

## Overview

This project implements a fraud detection system that analyzes suspicious email transactions. It flags fraudulent transactions based on various factors such as repeated transactions within a short window, sender's fraud history, and suspicious domains. The system also identifies fraudulent patterns by analyzing email subjects, domains, and transaction histories. The dataset used in this project is sourced from [Kaggle: Phishing Email Dataset](https://www.kaggle.com/datasets/naserabdullahalam/phishing-email-dataset?utm_source=chatgpt.com).

## Features

1. **Extract Numbers from Email Subjects**  
   Extracts numerical data from email subjects to identify potential fraud indicators.
   
2. **Flag Repeated Transactions**  
   Flags transactions from the same sender that occur within a 10-minute window, potentially indicating fraudulent behavior.

3. **Track Fraud History**  
   Flags customers with a history of fraudulent transactions within the past 30-day window.

4. **Identify Suspicious Domains**  
   Extracts and checks email domains for a history of fraudulent activity, helping to detect phishing attempts.

5. **Generate Fraudulent Transaction Report**  
   Generates a detailed JSON report summarizing fraudulent transactions, including total fraudulent transactions, suspicious and repeated transactions, and the most common senders and receivers involved in fraud.

## Dataset

This project utilizes the **Phishing Email Dataset** available on [Kaggle](https://www.kaggle.com/datasets/naserabdullahalam/phishing-email-dataset?utm_source=chatgpt.com), which contains email metadata and labels indicating whether the email is phishing or not.

## Requirements

- Python 3.x
- Pandas
- NumPy
- JSON
- Regular Expressions (re)

## Installation

1. Clone the repository:

    ```bash
    git clone https://github.com/yourusername/phishing-email-detector.git
    ```

## Usage

### Step 1: Data Preprocessing

The first part of the project involves data preprocessing. We extract numerical data from email subject lines and filter the dataset for fraudulent transactions.

```python
df_filtered = df[df["extracted_numbers"].apply(lambda x: len(x) > 0)]
```

### Step 2: Converting Date Strings to Datetime Format

In this step, we define a function to convert date strings into a proper `datetime` format. The function uses the `pd.to_datetime()` method to convert the date string into a datetime object, handling time zones and formatting issues.

```python
def convert_to_datetime(date_string):
    try:
        return pd.to_datetime(date_string, format='%a, %d %b %Y %H:%M:%S %z', errors='raise')
    except ValueError:
        return pd.NaT

df['date'] = df['date'].apply(convert_to_datetime)
```

### Step 3: Flagging Repeated Transactions

In this step, a function is created to identify repeated transactions from the same sender within a 10-minute window. The function sorts the transactions by sender and date, and then compares each transaction to the previous one. If two transactions have the same sender and occur within the specified time window, the second transaction is flagged as repeated.

```python
def flag_repeated_transactions(df, time_window_minutes=10):
    df = df.sort_values(by=['sender', 'date'])
    df['repeated_transaction'] = False
    
    for i in range(1, len(df)):
        if df.iloc[i]['sender'] == df.iloc[i-1]['sender'] and (df.iloc[i]['date'] - df.iloc[i-1]['date']) <= timedelta(minutes=time_window_minutes):
            df.at[i, 'repeated_transaction'] = True
    
    return df
```

### Step 4: Flagging Customers with Fraud History

In this step, a function is created to identify customers with a history of fraudulent transactions within the past 30 days. The function checks each fraudulent transaction and identifies if the sender has more than two fraudulent transactions in the past 30 days. If the sender meets this criteria, their transaction is flagged as having a fraud history.

```python
def flag_customer_fraud_history(df, days_window=30):
    df['fraud_history'] = False
    for idx, row in df.iterrows():
        if row['label'] == 1:  
            fraud_history = suspicious_transactions[(suspicious_transactions['sender'] == row['sender']) &
                                                    (suspicious_transactions['date'] <= row['date']) &
                                                    (suspicious_transactions['date'] >= row['date'] - timedelta(days=days_window))]
            if len(fraud_history) > 2:
                df.at[idx, 'fraud_history'] = True
    
    return df
```

### Step 5: Flagging Suspicious Transactions

In this step, a function is created to flag transactions as suspicious if they meet certain conditions. A transaction is flagged as suspicious if it is repeated or if the sender has a fraud history. The function creates a new column `is_suspicious` that is set to `True` if either of the conditions is met.

```python
def flag_suspicious_transaction(df):
    df['is_suspicious'] = False
    df['is_suspicious'] = df['repeated_transaction'] | df['fraud_history']
    return df
```

### Step 6: Generating Fraudulent Transaction Report

In this step, a detailed fraud report is generated based on the fraudulent transactions present in the dataset. The report includes the following metrics:

- **Total Fraudulent Transactions**: Total number of fraudulent transactions.
- **Fraudulent Senders**: The count of occurrences of each sender involved in fraudulent transactions.
- **Fraudulent Receivers**: The count of occurrences of each receiver involved in fraudulent transactions.
- **Suspicious Transactions**: The total number of fraudulent transactions that are flagged as suspicious.
- **Repeated Transactions**: The total number of fraudulent transactions flagged as repeated.

The report is then saved to a JSON file named `fraudulent_report.json` for further analysis.

```python
fraudulent_transactions = df_cleaned[df_cleaned['label'] == 1]

report = {
    'total_fraudulent_transactions': int(len(fraudulent_transactions)),  
    'fraudulent_senders': fraudulent_transactions['sender'].value_counts().to_dict(),
    'fraudulent_receivers': fraudulent_transactions['receiver'].value_counts().to_dict(),
    'suspicious_transactions': int(fraudulent_transactions['is_suspicious'].sum()),  
    'repeated_transactions': int(fraudulent_transactions['repeated_transaction'].sum()),  
}

with open('fraudulent_report.json', 'w') as f:
    json.dump(report, f, indent=4)
```

### Step 7: Displaying the Fraudulent Transaction Report
To read and display the fraudulent transaction report:

```python
def print_fraud_report(file_path):
    with open(file_path, 'r') as f:
        report = json.load(f)
    
    print("Fraudulent Transactions Report:")
    print(f"Total Fraudulent Transactions: {report['total_fraudulent_transactions']}")
    print(f"Suspicious Transactions: {report['suspicious_transactions']}")
    print(f"Repeated Transactions: {report['repeated_transactions']}")
    print("\nFraudulent Senders:")
    for sender, count in report['fraudulent_senders'].items():
        print(f"{sender}: {count} occurrences")
    
    print("\nFraudulent Receivers:")
    for receiver, count in report['fraudulent_receivers'].items():
        print(f"{receiver}: {count} occurrences")

print_fraud_report('fraudulent_report.json')
```

### Conclusion
This fraud detection system is designed to help identify and flag potentially fraudulent transactions by analyzing email metadata, including the sender, receiver, transaction timestamps, and email subjects. The system can be further extended to incorporate more advanced fraud detection techniques such as machine learning for anomaly detection.

