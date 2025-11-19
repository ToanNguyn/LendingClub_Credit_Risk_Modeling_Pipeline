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
- **2015**: model monitoring (“pseudo-live” data)

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
Industry-standard transformation for credit scoring:

### **Fine Classing**
- Split variables into small bins  
- Calculate event rate, temporary WoE, IV  
- Explore underlying risk patterns

### **Coarse Classing**
- Group adjacent bins  
- Improve statistical stability  
- Reduce noise & overfitting

### **WOE Transformation**
