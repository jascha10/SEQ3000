# COBOL SEQ3000 – Sales Report Program

## 📘 Introduction
This COBOL program generates a detailed **Year-To-Date Sales Report** using customer data. It calculates both the **change in sales amount** and the **percentage change** between the current year and the previous year.

In addition to customer-level data, the program also produces:
- Sales Representative totals
- Branch totals
- Grand totals across all data

---

## 📑 Table of Contents
- [👥 Authors](#-authors)
- [📌 What does it do?](#-what-does-it-do)
- [🖥️ Output Example](#️-output-example)
- [🧠 COBOL Concepts Used](#-cobol-concepts-used)

---

## 👥 Authors
  
👨‍💻 **Jacob Schamp**

- **Jacob Schamp GitHub Profile**: [jascha10](https://github.com/jascha10)
  
- **Email**: [jascha10@wsc.edu]

---

## 📌 What does it do?

For each run, the program:

1. Reads customer sales data from an input file  
2. Calculates:
   - Sales difference (This Year - Last Year)
   - Percentage change in sales  
3. Outputs a formatted report including:
   - Branch Number  
   - Sales Representative Number  
   - Customer Number  
   - Customer Name  
   - Sales (This Year)  
   - Sales (Last Year)  
   - Change in Amount  
   - Change in Percentage  
4. Groups and summarizes data at multiple levels:
   - Sales Representative totals  
   - Branch totals  
   - Grand totals  

---

## 🖥️ Output Example
RPT5000:

<img width="691" height="472" alt="image" src="assets/output.png" />

---

## 🧠 COBOL Concepts Used

### 1. Control Break Logic
The program detects changes in **Branch** or **Sales Representative** to trigger totals.

When a change occurs:
- Prints totals
- Resets accumulators

Example:
IF WS-CURRENT-BRANCH NOT = WS-PREVIOUS-BRANCH

---

### 2. Accumulators (Totals)
Values are continuously added and rolled up:
ADD CM-SALES-THIS-YTD TO ST-THIS-YTD

Totals are calculated at:
- Salesrep level → Branch level → Grand total

---

### 3. Percentage Calculation
COMPUTE WS-CHANGE-PERCENT =
(WS-CHANGE-AMOUNT / CM-SALES-LAST-YTD) * 100

Special case to prevent division by zero:

IF CM-SALES-LAST-YTD = 0
MOVE 999.9

---

### 4. Data Formatting (PIC Clauses)
- `Z,ZZZ,ZZ9.99-` → formatted numeric output  
- `X(20)` → text field  
- `9(5)` → numeric field  

---

### 5. File Handling
- Reads from input file (`I_CUSTMAST`)
- Writes formatted report to output file (`O_RPT5000`)

---
### Resources
- [GeeksForGeeks](https://www.geeksforgeeks.org/cobol/file-handling-in-cobol/?utm_source=chatgpt.com)
- Note: GeeksForGeeks is good for help when troubleshooting or getting broken code working.
  Stick to the textbooks when you can.
