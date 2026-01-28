# UK Retail Banking AML Monitoring Project

## 1. Introduction

This document provides a comprehensive overview of an advanced Anti-Money Laundering (AML) monitoring system designed for a UK retail banking context. Developed entirely within Alteryx Designer, this project demonstrates a robust solution for identifying suspicious financial activities, calculating transaction-level risk scores, and generating detailed reports for compliance and operational review. It showcases sophisticated data processing, feature engineering, rule-based detection, and proxies for advanced machine learning and network analysis.

## 2. Project Overview

The primary goal of this project is to automate and enhance the detection of various money laundering typologies relevant to the UK retail banking sector. The workflow systematically ingests raw transaction and customer data, enriches these datasets, and applies a series of analytical techniques to assess the risk associated with each transaction. The outcome is a structured dataset of flagged transactions and a multi-page PDF report, providing actionable insights into high-risk activities and entities.

## 3. Features Implemented

### 3.1. Data Integration & Preparation
- **Transaction Data Ingestion**: Processes detailed transaction records including unique identifiers, amounts, dates, types, payment channels, counterparty information (name, country), and international flags.
- **Customer Data Ingestion**: Incorporates customer profiles such as unique identifiers, full names, dates of birth, occupations, annual incomes, account opening dates, and critical risk flags (PEP, Sanctions, internal risk rating).
- **Unified Dataset Creation**: A central `Join` operation links transaction records with corresponding customer profiles using `customer_id`, creating a comprehensive dataset for downstream analysis.

### 3.2. AML Scenarios & Risk Indicators
The workflow implements and flags several critical AML scenarios:
- **Large Transaction Compared to Income**: Identifies transactions where the `amount_gbp` exceeds 50% of the customer's declared `annual_income`, indicating potential unexplained wealth or money mule activity.
- **Granular Frequent Structuring (Smurfing)**: Detects patterns indicative of structuring by identifying multiple transactions (each below a reporting threshold, e.g., £10,000) occurring within a rolling 2-day window for the same customer.
- **Transactions to High-Risk Countries**: Flags any international transfer where the `counterparty_country` is designated as high-risk (e.g., Iran, North Korea, Syria, Afghanistan).
- **New Account + High Activity**: Identifies suspicious activity in newly opened accounts by flagging transactions with an `amount_gbp` greater than £5,000 occurring within 30 days of the `account_open_date`.

### 3.3. Advanced Analytics Proxies
To simulate advanced AML capabilities, the project incorporates proxies for sophisticated analytical techniques:
- **Behavioral Profiling & ML Anomaly Detection**:
  - Calculates the `Avg_amount_gbp` for each customer across all their transactions.
  - Flags transactions as `Isolation_Forest_Anomaly_Flag` if their `amount_gbp` is more than 3 times the customer's calculated `Avg_amount_gbp`, serving as a rule-based proxy for statistical outlier detection.
- **Network & Relationship Analysis (High-Risk Counterparties)**:
  - Identifies `counterparty_name` values that are frequently involved in transactions already classified as 'High' risk based on an intermediate risk assessment.
  - Flags subsequent transactions as `High_Risk_Counterparty_Flag` if they involve these identified high-risk counterparties.

### 3.4. Comprehensive Risk Scoring & Categorization
- **Risk Score Calculation**: A cumulative `Risk_Score` is calculated for each transaction by assigning specific points to each triggered AML flag. The points are weighted based on the perceived severity of the risk indicator.
  - Sanctions Flag: +100 points
  - Isolation Forest Anomaly Flag: +50 points
  - PEP Flag: +40 points
  - High-Risk Counterparty Flag: +40 points
  - High-Risk Country Transaction: +30 points
  - Frequent Structuring Flag: +25 points
  - New Account High Activity: +20 points
- **Risk Category Assignment**: The final `Risk_Score` is mapped to a qualitative `Risk_Category`:
  - **Low**: Score 0-39
  - **Medium**: Score 40-69
  - **High**: Score 70+

### 3.5. Reporting & Output
The workflow generates two primary outputs:
- **Detailed Transaction Output**: An Excel file (`AML Result.xlsx`) containing every processed transaction record, enriched with all calculated flags, the final `Risk_Score`, and `Risk_Category`. This file serves as the primary data source for further detailed analysis or integration with external dashboarding tools.
- **Comprehensive PDF Report**: A multi-page PDF document (`Anti-Money Laundering Report.pdf`) designed for compliance officers, providing a summarized view of the AML assessment. The report includes:
  - **Overall KPIs**: A table summarizing key metrics such as Total Transactions, Total Customers, Total High Risk Customers, Total High-Risk Country Transfers, and Total Anomalies.
  - **Transactions by Risk Level**: A pie chart illustrating the distribution of transactions across Low, Medium, and High-risk categories.
  - **Geographic Risk**: A donut chart showing the count of high-risk transactions by `counterparty_country`.
  - **Monthly Suspicious Trend**: A bar chart depicting the trend of high-risk transactions over time.
  - **Top 10 High Risk Customers**: A table listing the top 10 customers based on their aggregated risk scores, including their `customer_id` and `full_name`.

## 4. Tools and Technologies
- **Alteryx Designer**: The core platform used for developing, executing, and managing the entire data processing, analysis, and reporting workflow.
- **Alteryx Native Tools**: Extensive utilization of tools from various categories:
  - **In/Out**: Input Data, Output Data
  - **Join**: Join
  - **Preparation**: Filter, Formula, Multi-Row Formula, Sort, Sample, Select
  - **Transform**: Summarize
  - **Reporting**: Interactive Chart, Table, Layout, Render, Report Header
- **Microsoft Excel**: Used as the format for input data sources and the detailed transaction output.
- **PDF**: The final output format for the comprehensive AML report.

## 5. Data Overview

The project relies on two primary datasets:

### 5.1. Transaction Data
This dataset contains individual transaction records.

| Field Name | Purpose | Data Type (Alteryx) |
| :------------------- | :------------------------------------------ | :------------------ |
| `transaction_id` | Unique identifier for each transaction | V_String |
| `customer_id` | Links to the customer profile | V_String |
| `transaction_date` | Date of the transaction | DateTime |
| `amount_gbp` | Transaction amount in Great British Pounds | Double |
| `currency` | Currency of the transaction | V_String |
| `transaction_type` | Type of transaction (e.g., Faster Payments, CHAPS, Debit card) | V_String |
| `payment_channel` | Channel used (e.g., Online, Branch, ATM) | V_String |
| `counterparty_country` | Country of the transaction's counterparty | V_String |
| `counterparty_name` | Name of the beneficiary or sender | V_String |
| `account_balance_after` | Customer's account balance post-transaction | Double |
| `is_international` | Boolean flag: `True` if international, `False` if domestic | Bool |
| `is_high_risk_country` | Boolean flag: `True` if `counterparty_country` is high-risk | Bool |
| `transaction_time` | Time of the transaction | Time |

### 5.2. Customer Data
This dataset contains customer demographic and risk profile information.

| Field Name | Purpose | Data Type (Alteryx) |
| :------------------- | :------------------------------------------ | :------------------ |
| `customer_id` | Unique identifier for each customer | V_String |
| `full_name` | Customer's full name | V_String |
| `date_of_birth` | Customer's date of birth | Date |
| `occupation` | Customer's occupation | V_String |
| `annual_income` | Customer's declared annual income | Double |
| `customer_type` | Type of customer (e.g., Personal, Business, Student) | V_String |
| `account_open_date` | Date the customer's account was opened | DateTime |
| `pep_flag` | Boolean flag: `True` if Politically Exposed Person | Bool |
| `sanctions_flag` | Boolean flag: `True` if hit on sanctions screening | Bool |
| `risk_rating` | Internal customer risk rating (e.g., Low, Medium, High) | V_String |
| `address_country` | Customer's country of residency | V_String |

## 6. Workflow Notes and Assumptions
- **Input Data Sources**: The workflow is configured to read data from local Excel files (`Transaction Data.xlsx`, `Customer Data.xlsx`). For deployment in a production environment, these `Input Data` tools would typically be reconfigured to connect to enterprise databases (e.g., SQL Server, Oracle, Snowflake) or data lakes.
- **Sample Data**: The workflow includes embedded sample data within `Text Input` tools for initial testing and demonstration. For full-scale operation, these `Text Input` tools should be replaced with `Input Data` tools pointing to external files or database connections.
- **Configurable Thresholds**: All numerical thresholds used in the AML rules (e.g., £10,000 for structuring, £5,000 for new account activity, `Count_transaction_id > 1` for high-risk counterparties, `3 * Avg_amount_gbp` for anomaly detection) are defined within `Filter` and `Formula` tools. These thresholds are critical and should be regularly reviewed and fine-tuned by compliance experts based on evolving risk appetites, regulatory guidance, and observed data patterns.
- **ML Proxy Limitations**: The `Isolation_Forest_Anomaly_Flag` and `High_Risk_Counterparty_Flag` are implemented using rule-based logic as proxies for more complex machine learning and network analysis techniques. For a truly advanced system, these would be replaced by actual ML models (e.g., using Alteryx's Python/R tools or Intelligence Suite) and potentially integrated with graph databases.
- **Reporting Customization**: The visual layout, specific chart types, axes, table fields, and styling within the `Interactive Chart`, `Table`, `Layout`, and `Render` tools require manual configuration within Alteryx Designer. The workflow prepares the necessary data streams, but the final report presentation is an interactive design task.
- **Performance Considerations**: For processing very large datasets (millions of records), consider optimizing the workflow by leveraging Alteryx's In-Database processing capabilities, utilizing the AMP Engine, and carefully managing memory limits.
- **Time Zone Consistency**: Assumes consistent handling of time zones for `transaction_date` and `transaction_time` across all data sources to ensure accurate time-based analysis.
- **Data Quality**: Assumes a reasonable level of data quality in the input datasets, particularly for `customer_id` consistency, which is crucial for successful data joining.

## 7. How to Use

### 7.1. Prerequisites
1. **Alteryx Designer**: Ensure you have Alteryx Designer installed on your system.
2. **Input Data Files**:
   - Create two Excel files: `Transaction Data.xlsx` and `Customer Data.xlsx`.
   - Populate these files with data matching the schema described in Section 5.
   - Update the `Input Data` tools (Tool ID 28 and 29) in the workflow to point to the correct paths of your Excel files. (Alternatively, you can use the embedded sample data in the `Text Input` tools for initial testing).

### 7.2. Running the Workflow
1. **Open the Workflow**: Open the `.yxmd` file in Alteryx Designer.
2. **Execute**: Click the "Run" button (typically a green arrow icon) in the Alteryx Designer toolbar.
3. **Review Output**:
   - **Detailed Transaction Output**: The `AML Result.xlsx` file will be generated in the directory specified in the `Output Data` tool (Tool ID 22).
   - **Comprehensive PDF Report**: The `Anti-Money Laundering Report.pdf` will be generated in the directory specified in the `Render` tool (Tool ID 76).

### 7.3. Configuring Reports (Manual Steps in Alteryx Designer)
To customize the visual appearance and layout of the generated report:
1. **Locate Reporting Tools**: Identify the `Interactive Chart`, `Table`, `Layout`, and `Render` tools within the workflow.
2. **Configure Each Tool**: Select each reporting tool and use its respective Configuration window to:
   - **Interactive Chart**: Define chart types (e.g., Line, Bar, Pie), X/Y axes, titles, and styling.
   - **Table**: Select which fields to display, set headers, and apply basic formatting.
   - **Layout**: Arrange the order and positioning of charts and tables (e.g., vertical stacking, horizontal arrangement).
   - **Render**: Specify the output file path, format (PDF, HTML, DOCX), and page settings.

## 8. Future Enhancements

This project provides a strong foundation, but several areas can be explored for further advancement:
- **True Machine Learning Integration**: Implement actual anomaly detection models (e.g., Isolation Forest, One-Class SVM) or predictive models (e.g., Logistic Regression) using Alteryx's Python/R tools or the Intelligence Suite.
- **Dynamic and Adaptive Thresholds**: Replace fixed thresholds with dynamically calculated values based on statistical distributions, customer segments, or historical behavior.
- **Advanced Behavioral Profiling**: Incorporate peer group analysis, more complex time-series anomaly detection, and customer lifecycle analysis.
- **External Data Enrichment**: Integrate with real-time external data sources such as sanctions lists, Politically Exposed Persons (PEP) databases, and adverse media screening APIs.
- **Case Management System Integration**: Develop direct API integrations with AML case management systems for automated alert creation and workflow management.
- **Performance Optimization for Big Data**: Further optimize the workflow for extremely large datasets by leveraging advanced In-Database processing techniques and distributed computing.
- **User Interface Development**: Create an Alteryx Analytic App or deploy the workflow to Alteryx Server to provide a user-friendly interface for parameter input and on-demand execution.
