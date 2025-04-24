
# 📄 DAX Formulas Used in Power BI Project

This file contains all DAX formulas used in the Credit Card Financial Analysis project. These formulas were implemented in Power BI to derive meaningful insights from transactional and customer data.

---

## 1. Running Total of Credit Card Transactions
```DAX
Running Total = 
    CALCULATE(
        [Total Transaction Amount],
        FILTER(
            ALL('credit card'),
            'credit card'[Week_Start_Date] <= MAX('credit card'[Week_Start_Date])
        )
    )
```

---

## 2. 4-Week Moving Average of Credit Limit
```DAX
Moving_average_4_weeks = 
VAR weeks4 = DATESINPERIOD('Calendar'[Date], MAX('Calendar'[Date]), -28, DAY)
VAR total_amount = CALCULATE([Total Transaction Amount], weeks4)
VAR num_of_weeks = CALCULATE(DISTINCTCOUNT('Calendar'[week_num]), weeks4)
RETURN DIVIDE(total_amount, num_of_weeks, 0)
```

---

## 3. MoM% and WoW% Growth on Transaction Amount
```DAX
mom%growth = 
VAR pre_month = CALCULATE([Total Transaction Amount], DATEADD('Calendar'[Date], -1, MONTH))
RETURN DIVIDE([Total Transaction Amount] - pre_month, pre_month, 0)

wow%growth = 
VAR prev_week = CALCULATE([Total Transaction Amount], DATEADD('Calendar'[Date], -7, DAY))
RETURN DIVIDE([Total Transaction Amount] - prev_week, prev_week, 0)
```

---

## 4. Customer Acquisition Cost (CAC) as a Ratio of Transaction Amount
```DAX
ratio_cac_transaction_amount = 
    DIVIDE(SUM('credit card'[Customer_Acq_Cost]), [Total Transaction Amount], 0)
```

---

## 5. Yearly Average of Avg_Utilization_Ratio
```DAX
avg_utilization_ratio = 
    AVERAGE('credit card'[Avg_Utilization_Ratio])
```

---

## 6. Percentage of Interest_Earned Compared to Total_Revolving_Bal
```DAX
interest_earned_by_revol_balance = 
    DIVIDE(SUM('credit card'[Interest_Earned]), SUM('credit card'[Total_Revolving_Bal]))
```

---

## 7. Top 5 Clients by Total Transaction Amount
```DAX
top_5_clients = 
    TOPN(5, 
        SUMMARIZE(
            'credit card',
            'credit card'[Client_Num],
            "total amount", [Total Transaction Amount]
        ),
        [total amount], DESC
    )
```

---

## 8. Identify Clients with Avg_Utilization_Ratio > 80%
```DAX
check_exceeds_80 = 
    IF([Avg_Utilization_Ratio] > 0.8, TRUE, FALSE)
```

---

## 9. Customer Churn Indicator (No Transactions in Last 6 Months)
```DAX
churn = 
VAR balance = CALCULATE(
    'Table'[Total Transaction Amount],
    DATESINPERIOD('Calendar'[Date], MAX('Calendar'[Date]), -6, MONTH)
)
RETURN IF(ISBLANK(balance), "Churned", "Not_Churned")
```

---

## 10. Delinquency Rate
```DAX
Delinquency Rate = 
VAR greater_zero = CALCULATE(COUNTROWS('credit card'), 'credit card'[Delinquent_Acc] > 0)
VAR total_rows = COUNTROWS('credit card')
RETURN DIVIDE(greater_zero, total_rows, 0)
```

---

## 11. Credit Risk Score
```DAX
normalized_revolving_balance = 
    DIVIDE(
        'credit card'[Total_Revolving_Bal] - MIN('credit card'[Total_Revolving_Bal]),
        MAX('credit card'[Total_Revolving_Bal]) - MIN('credit card'[Total_Revolving_Bal]),
        0
    )

credit_risk_score = 
    [Avg_Utilization_Ratio] * 0.5 + 
    'credit card'[normalized_revolving_balance] * 0.3 + 
    'credit card'[Delinquent_Acc] * 0.2
```

---

## 13. Average Customer Satisfaction Score by Card Category
```DAX
avg_satisfaction_score = 
    SUMMARIZE(
        'credit card',
        'credit card'[Card_Category],
        "avg_satisfaction_score", AVERAGE(customer[Cust_Satisfaction_Score])
    )
```

---

## 14. Loan Approval vs Credit Limit
```DAX
loan_approval_yes = 
    CALCULATE(AVERAGE('credit card'[Credit_Limit]), customer[Personal_loan] = "yes")

loan_approval_no = 
    CALCULATE(AVERAGE('credit card'[Credit_Limit]), customer[Personal_loan] = "no")
```

---

## 15. High Risk Clients Flag
```DAX
flag_clients = 
    IF(
        'credit card'[normalized_revolving_balance] > 0.9 && 
        [Avg_Utilization_Ratio] > 0.8,
        "flagged",
        "not flagged"
    )
```
