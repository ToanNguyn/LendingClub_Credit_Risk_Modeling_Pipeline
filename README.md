# LendingClub Credit Risk Modeling Pipeline

## 1. Introduction
This project develops a complete **credit risk modeling pipeline** using LendingClub loan data. It includes:

- Data preprocessing and feature cleaning  
- Fine Classing → Coarse Classing → Weight of Evidence (WoE) transformation  
- PD model development using Logistic Regression with p-values  
- Scorecard development (FICO-style scaling)  
- Model validation using AUC, KS, Gini, Accuracy  
- Model monitoring using PSI  
- LGD and EAD modeling  
- Expected Loss (EL) estimation  

Datasets:
- **2007–2014**: training + validation  
- **2015**: model monitoring 
---

## 2. Dataset & Feature Groups
The dataset includes the following categories:

### **Personal & Employment Information**
- addr_state, emp_length, emp_title  
- home_ownership  
- zip_code  

### **Income & Repayment Capacity**
- annual_inc  
- dti (Debt-to-Income ratio)  
- installment  
- pymnt_plan  
- verification_status_joint  

### **Loan Information**
- loan_amnt, int_rate, term  
- grade, sub_grade  
- purpose, title  
- initial_list_status  
- loan_status (target variable)

### **Credit History**
- delinq_2yrs  
- earliest_cr_line  
- inq_last_6mths  
- open_acc, total_acc  
- pub_rec  
- mths_since_last_major_derog  
- mths_since_last_record  

### **Revolving Credit**
- revol_bal  
- revol_util  
- total_rev_hi_lim  

### **Collections & Recoveries**
- collections_12_mths_ex_med  
- tot_coll_amt  
- tot_cur_bal  

**Target definition (Bad loans):**  
`Charged Off`, `Default`, `Does not meet the credit policy: Charged Off`,  
`Late (31–120 days)`

---

## 3. Preprocessing Pipeline
Main components:
- Remove redundant or leakage variables  
- Parse datetime variables  
- Dummy encoding  
- Missing value treatment  
- Construct binary target variable (Good/Bad)

---

## 4. Feature Engineering: Fine Classing → Coarse Classing → WoE

### **Fine Classing**
- Split variables into small bins  
- Calculate event rate, temporary WoE, IV  
- Explore underlying risk patterns

### **Coarse Classing**
- Group adjacent bins  
- Improve statistical stability  
- Reduce noise & overfitting

### **WOE Transformation**
$$
\text{WOE} = \ln\left(\frac{\text{Good}\%}{\text{Bad}\%}\right)
$$

Interpretation:
- WOE > 0 → good customers dominate  
- WOE < 0 → bad customers dominate  
- WOE = 0 → no discriminatory power  

Processing rules:
- **Discrete variables:** WoE → coarse classing → dummies  
- **Continuous variables:** fine classing → WoE → coarse → dummies  

---

## 5. PD Model Development
The Probability of Default (PD) represents the likelihood that a borrower will default within a given time horizon

This project builds the PD model using Logistic Regression, with:
- p-values for coefficient significance  
- hypothesis testing for β ≠ 0  

Variable selection rules:
- If all dummies are significant → keep entire variable  
- If none significant → remove entire variable  
- If at least one dummy significant → keep all dummies for that variable  
- p-value < 0.05 → statistically significant  

---

## 6. Model Validation
Evaluated on out-of-sample data:
- **AUC-ROC**  
- **KS Statistic**  
- **Accuracy**  
- **Gini Coefficient**  

---

## 7. Scorecard Development
Converting logistic regression coefficients into a scorecard scale (300–850).

Includes:
- Score scaling  
- Points allocation per attribute  
- Final customer score  
- Mapping threshold → approval rate and rejection rate  

---

## 8. Model Monitoring 
To simulate production monitoring, the 2015 dataset is used.

Steps:
- Compute feature-wise score contribution  
- Bin the score contributions  
- Calculate PSI between (2007–2014) and 2015  

$$
\text{PSI} = \sum (P_i - Q_i) \cdot \ln\left(\frac{P_i}{Q_i}\right)
$$


PSI thresholds:
- < 0.1 → stable  
- 0.1–0.25 → moderate shift  
- ≥ 0.25 → significant drift  

---

## 9. LGD Modeling: Two-Stage Approach
Loss Given Default represents the share of exposure that the lender expects to lose if a borrower defaults

### Stage 1 — Probability that LGD > 0  
Using Logistic Regression to predict whether a defaulted loan results in any loss.

### Stage 2 — Expected LGD given LGD > 0  
Using Linear Regression to estimate the magnitude of loss, but only for cases where a loss actually occurs.

Combined formula:

$$
\text{LGD} = P(\text{LGD} > 0) \times \mathbb{E}[\text{LGD} \mid \text{LGD} > 0]
$$


---

## 10. EAD Modeling
Exposure at Default represents the expected amount of the loan that will be outstanding at the moment the borrower defaults.

For products where borrowers can draw additional funds before default (e.g., credit lines, credit cards), Basel requires estimating a Credit Conversion Factor (CCF). CCF measures the proportion of the unused or undrawn portion that is likely to be drawn down by the borrower prior to default.

$$
\text{EAD} = \text{CCF} \times \text{FundedAmount}
$$

$$
\text{CCF} = \frac{\text{OutstandingDefault}}{\text{FundedAmount}}
$$

Linear Regression is used to predict the expected CCF.

---

## 11. Expected Loss
Final risk estimate:

$$
\text{EL} = \text{PD} \times \text{LGD} \times \text{EAD}
$$

Aggregated EL represents the total required loss provisioning.

---

## 12. src
Contains core functions:
- LogisticRegression_with_p_values  
- WoE & IV functions  
- plot_by_woe  
- create_summary_table  
- Custom LinearRegression with p-values  

---

## 13. Conclusion
This project replicates a full **industry-grade credit risk modeling framework**, covering:
- PD modeling & scorecard  
- LGD & EAD estimation  
- Expected Loss calculation  
- Model monitoring (PSI)  
- End-to-end pipeline from raw data to risk metrics
