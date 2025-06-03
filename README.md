# 📊 Loan Default Risk Analysis – Power BI Report

This Power BI report presents an in-depth analysis of loan default risks based on credit score, income level, age group, employment type, and other key customer demographics. It is built on a large dataset and follows a complete data pipeline from Excel to Power BI Service.

---

## 🔗 Data Pipeline & Structure

- **Raw Source**: Excel file containing:
  - **255,354 rows**
  - **19 columns**
- **ETL Flow**:
  1. Loaded from **Excel** into **SQL Server**
  2. Pulled into **Power BI Dataflow**
  3. Modeled and visualized in **Power BI Desktop**
  4. Published to **Power BI Service**

---

## 🧮 Calculated Columns (DAX)

To enhance analysis, the following custom columns were created using DAX:

### 📌 Age Group
```dax
Age Group = 
IF('Loan_default'[Age]<=19, "Teen",
   IF('Loan_default'[Age]<=30, "Adult",
      IF('Loan_default'[Age]<=59, "Middle Aged", "Senior")))
```

### 📌 Credit Score Bins
```dax
Credit Score Bins = 
IF('Loan_default'[CreditScore]<=400, "Very Low",
   IF('Loan_default'[CreditScore]<=450, "Low",
      IF('Loan_default'[CreditScore]<=650, "Medium", "High")))
```

### 📌 Income Bracket
```dax
Income Bracket = 
SWITCH(
    TRUE(),
    'Loan_default'[Income] < 30000, "Low Income",
    'Loan_default'[Income] >= 30000 && 'Loan_default'[Income] < 60000, "Medium Income",
    'Loan_default'[Income] >= 60000, "High"
)
```

---

## 📄 Power BI Report Pages

### 🧾 Page 1: Overview Dashboard

**Visuals**:
- KPI cards: Total Loans, Defaults, Avg Credit Score, Avg Income
- Bar/Column charts for:
  - Age Group
  - Gender
  - Marital Status
  - Education
  - Employment Type

**DAX Measures**:

#### Average Income by Employment Type
```dax
Average Income by Employee Type = 
CALCULATE(
    AVERAGE('Loan_default'[Income]),
    ALLEXCEPT('Loan_default', 'Loan_default'[EmploymentType])
)
```

#### Average Loan by Age Group
```dax
Average Loan by Age Group = 
AVERAGEX(
    VALUES('Loan_default'[Age Group]),
    AVERAGE('Loan_default'[LoanAmount])
)
```

#### Default Rate by Employment Type
```dax
Default rate by Employee type = 
VAR totalrecords = 
    CALCULATE(COUNTROWS('Loan_default'), ALL('Loan_default'))

VAR defaultcases = 
    CALCULATE(
        COUNTROWS('Loan_default'), 
        'Loan_default'[Default] = TRUE(), 
        ALLEXCEPT('Loan_default', 'Loan_default'[EmploymentType])
    )

RETURN DIVIDE(defaultcases, totalrecords) * 100
```

#### Default Rate by Year
```dax
Default rate by year = 
VAR TotalRows = CALCULATE(
    COUNTROWS('Loan_default'), 
    ALLEXCEPT('Loan_default', 'Loan_default'[Year])
)

VAR DefaultRows = CALCULATE(
    COUNTROWS('Loan_default'), 
    ALLEXCEPT('Loan_default', 'Loan_default'[Year]),
    FILTER('Loan_default', 'Loan_default'[Default] = TRUE())
)

RETURN DIVIDE(DefaultRows, TotalRows) * 100
```

#### Loan Amount by Purpose
```dax
Loan amount by purpose = 
SUMX(
    FILTER('Loan_default', NOT(ISBLANK('Loan_default'[LoanAmount]))),
    'Loan_default'[LoanAmount]
)
```
---

#### Image
![{BAE62B14-DC7D-4C75-9A7E-64ADFD8E8AE7}](https://github.com/user-attachments/assets/50a64e0d-dbf2-4733-b2fb-d7e65269d18a)


---

**Insights**:
- Defaults are high in Adults and Middle Aged
- Self-employed borrowers show greater risk
- Single and Divorced categories show elevated default

---

### 📊 Page 2: Credit Score & Income Risk

**Visuals**:
- Distribution by Credit Score Bins
- Analysis by Income Bracket
- Matrix: Credit Score vs Income
- KPI Cards for segment-specific metrics

**DAX Measures**:

#### Average Loan Amount (High Credit)
```dax
Average Loan Amount(High Credit) = 
AVERAGEX(
    FILTER('Loan_default', 'Loan_default'[Credit Score Bins] = "High"),
    'Loan_default'[LoanAmount]
)
```

#### Median by Credit Score Bins
```dax
Median by credit score bins = 
MEDIANX('Loan_default', 'Loan_default'[LoanAmount])
```

#### Total Loan (Credit Bins, Adults Only)
```dax
Total Loan (Credit bins) = 
CALCULATE(
    SUM('Loan_default'[LoanAmount]),
    'Loan_default'[Age Group] = "Adult",
    ALLEXCEPT(
        'Loan_default',
        'Loan_default'[Age],
        'Loan_default'[Age Group],
        'Loan_default'[Credit Score Bins]
    )
)
```

#### Total Loan (Middle Aged Adults)
```dax
Total Loan (Middle Age Adults) = 
SUMX(
    FILTER('Loan_default', 'Loan_default'[Age Group] = "Middle Aged"),
    'Loan_default'[LoanAmount]
)
```

#### Total Loans by Education Type
```dax
Total Loans by Education Type = 
COUNTROWS(
    FILTER('Loan_default', NOT(ISBLANK('Loan_default'[LoanID])))
)
```
---

#### Image
![{AFE5012D-FE80-4890-97E3-BB7CD268A210}](https://github.com/user-attachments/assets/a2e01279-b5fc-4b4e-a78b-a5c3de043e58)

---

**Insights**:
- Medium credit score borrowers dominate defaults
- Low-income applicants are the riskiest
- High-credit borrowers take bigger loans but default less

---

### 📈 Page 3: Financial Risk Metrics

**Visuals**:
- YOY Analysis Cards (Default %, Loan Amount %)
- YTD Loan Trend Line
- Sankey Chart: Credit Score → Marital Status → Default
- Decomposition Tree: Loan Amount by Employment & Income

**DAX Measures**:

#### YOY Default Loans Change
```dax
YOY Default Loans Change = 
VAR Curr = CALCULATE(
    COUNTROWS(
        FILTER('Loan_default', 'Loan_default'[Default] = TRUE())
    ),
    'Loan_default'[Year] = YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY]))
)

VAR Prev = CALCULATE(
    COUNTROWS(
        FILTER('Loan_default', 'Loan_default'[Default] = TRUE())
    ),
    'Loan_default'[Year] = YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY])) - 1
)

RETURN DIVIDE(Curr - Prev, Prev, 0) * 100
```

#### YOY Loan Amount Change
```dax
YOY Loan Amount Change = 
VAR curr = CALCULATE(
    SUM('Loan_default'[LoanAmount]),
    'Loan_default'[Year] = YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY]))
)

VAR prev = CALCULATE(
    SUM('Loan_default'[LoanAmount]),
    'Loan_default'[Year] = YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY])) - 1
)

RETURN DIVIDE(curr - prev, prev, 0) * 100
```

#### YTD Loan Amount
```dax
YTD Loan Amount = 
CALCULATE(
    SUM('Loan_default'[LoanAmount]),
    DATESYTD('Loan_default'[Loan_Date_DD_MM_YYYY].[Date]),
    ALLEXCEPT('Loan_default', 'Loan_default'[Credit Score Bins], 'Loan_default'[MaritalStatus])
)
```
---

#### Image
![{A5622942-0750-4056-86BB-0BA4B5F8BEEC}](https://github.com/user-attachments/assets/19a9714e-c2ec-435b-b5c6-73e1114f0927)

---
**Insights**:
- YOY default rates dropped slightly despite increased loan issuance
- High-income, employed applicants carry bulk of loans
- Married individuals show lower YTD default ratios

---

## 🛠️ Tools & Technologies

- **Excel** – Raw data
- **SQL Server** – Data staging
- **Power BI Dataflow** – Transformations
- **Power BI Desktop** – Modeling & DAX
- **Power BI Service** – Publishing & Sharing
