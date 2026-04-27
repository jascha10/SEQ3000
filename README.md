# COBOL SEQ3000 – Employee File Maintenance Program

## 📘 Introduction

This COBOL program performs **sequential file maintenance** on an employee master file. It processes a transaction file against an existing employee master, producing an updated master file and an error transaction file for any invalid records.

The program supports three types of transactions:

- **Add (A)** – Insert a new employee record into the master file
- **Delete (D)** – Remove an existing employee record from the master file
- **Change (C)** – Update one or more fields on an existing employee record

---

## 📑 Table of Contents

- [👥 Authors](#-authors)
- [📌 What does it do?](#-what-does-it-do)
- [📂 Files Used](#-files-used)
- [🧠 COBOL Concepts Used](#-cobol-concepts-used)

---

## 👥 Authors

👨‍💻 **Jacob Schamp**

- **Jacob Schamp GitHub Profile**: [jascha10](https://github.com/jascha10)
- **Email**: [jascha10@wsc.edu](mailto:jascha10@wsc.edu)

---

## 📌 What does it do?

For each run, the program:

1. Opens four files: the old master (`OLDEMP`), the transaction file (`EMPTRAN`), the new master (`NEWEMP`), and an error file (`ERRTRAN3`)
2. Reads and compares employee IDs from the transaction and master files to determine the appropriate action:
   - **Master ID > Transaction ID** → Process the transaction alone (Add or Error)
   - **Master ID < Transaction ID** → Write master record as-is to new master and advance
   - **Master ID = Transaction ID** → Apply Delete or Change, or write an error if the code is invalid
3. Applies the transaction:
   - **Add** – Moves transaction data into the new master record and writes it
   - **Delete** – Skips the master record (effectively removing it)
   - **Change** – Selectively updates only non-blank/non-zero fields on the existing master record
4. Writes invalid or unmatched transactions to the error file (`ERRTRAN3`)
5. Continues until both files are fully processed (both IDs reach `HIGH-VALUE`)

---

## 📂 Files Used

| File | Role | Description |
|------|------|-------------|
| `EMPTRAN` | Input | Transaction records (Add, Delete, Change) |
| `OLDEMP` | Input | Existing employee master file |
| `NEWEMP` | Output | Updated employee master file |
| `ERRTRAN3` | Output | Error transactions that could not be applied |

---

## 🧠 COBOL Concepts Used

### 1. Sequential File Maintenance (Chapter 13)

This program is a classic **sequential file update** pattern. Both the master file and transaction file must be sorted in the same key order (Employee ID) before processing. The program reads one record at a time from each file and compares keys to decide what to do.

```cobol
IF EM-EMPLOYEE-ID > ET-EMPLOYEE-ID
    PERFORM 350-PROCESS-HI-MASTER
ELSE IF EM-EMPLOYEE-ID < ET-EMPLOYEE-ID
    PERFORM 360-PROCESS-LO-MASTER
ELSE
    PERFORM 370-PROCESS-MAST-TRAN-EQUAL.
```

---

### 2. Working with Disk Files (Chapter 12)

The program uses the `FILE-CONTROL` section to assign logical file names to physical files, and `FILE STATUS` fields to detect I/O errors at runtime.

```cobol
SELECT NEWEMP  ASSIGN TO NEWEMP
               FILE STATUS IS NEWEMP-FILE-STATUS.
```

After each write, the file status is checked:

```cobol
IF NOT NEWEMP-SUCCESSFUL
    DISPLAY "WRITE ERROR ON NEWEMP FOR ITEM NUMBER " ET-EMPLOYEE-ID
    SET ALL-RECORDS-PROCESSED TO TRUE.
```

---

### 3. Switch-Driven Logic

The program uses a set of `PIC X` switches with level-88 condition names to control the flow of reads and writes without duplicating logic. This avoids reading a new record until the current one has been fully processed.

```cobol
05  NEED-TRANSACTION-SWITCH    PIC X   VALUE "Y".
    88  NEED-TRANSACTION               VALUE "Y".
05  WRITE-MASTER-SWITCH        PIC X   VALUE "N".
    88  WRITE-MASTER                   VALUE "Y".
```

```cobol
IF NEED-TRANSACTION
    PERFORM 310-READ-INVENTORY-TRANSACTION
    MOVE "N" TO NEED-TRANSACTION-SWITCH.
```

---

### 4. End-of-File Handling with HIGH-VALUE

When either file reaches end-of-file, its key field is set to `HIGH-VALUE` so comparisons still work correctly. Processing ends when **both** IDs equal `HIGH-VALUE` simultaneously.

```cobol
READ EMPTRAN INTO EMPLOYEE-TRANSACTION
    AT END
        MOVE HIGH-VALUE TO ET-EMPLOYEE-ID.
```

```cobol
IF ET-EMPLOYEE-ID = HIGH-VALUES
    SET ALL-RECORDS-PROCESSED TO TRUE.
```

---

### 5. Add, Delete, and Change Transactions

**Add:** Moves all transaction fields into the new master record layout and sets the write flag.

```cobol
MOVE ET-EMPLOYEE-ID    TO NM-EMPLOYEE-ID.
MOVE ET-EMPLOYEE-NAME  TO NM-EMPLOYEE-NAME.
MOVE ET-ANUAL-SALARY   TO NM-ANUAL-SALARY.
SET WRITE-MASTER TO TRUE.
```

**Delete:** Simply advances both the master and transaction pointers — no write occurs.

```cobol
SET NEED-MASTER TO TRUE.
SET NEED-TRANSACTION TO TRUE.
```

**Change:** Only updates fields that are non-blank or non-zero, leaving unchanged fields intact.

```cobol
IF ET-EMPLOYEE-NAME NOT = SPACE
    MOVE ET-EMPLOYEE-NAME TO EM-EMPLOYEE-NAME.
IF ET-ANUAL-SALARY NOT = ZERO
    MOVE ET-ANUAL-SALARY TO EM-ANUAL-SALARY.
```

---

### 6. Error Transaction Handling

Any transaction that cannot be matched or has an invalid code is written to the error file. The program checks the file status after each error write and halts if the write fails.

```cobol
WRITE ERROR-TRANSACTION FROM EMPLOYEE-TRANSACTION.
IF NOT ERRTRAN-SUCCESSFUL
    DISPLAY "WRITE ERROR ON ERRTRAN3 FOR EMPLOYE ID " ET-EMPLOYEE-ID
    SET ALL-RECORDS-PROCESSED TO TRUE
ELSE
    SET NEED-TRANSACTION TO TRUE.
```

---

### Resources

- [GeeksForGeeks – File Handling in COBOL](https://www.geeksforgeeks.org/cobol/file-handling-in-cobol/)
- Note: GeeksForGeeks is helpful for troubleshooting broken code. Stick to the textbooks (Chapters 12–16) when learning concepts for the first time.
