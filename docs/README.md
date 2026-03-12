# Student Account COBOL Documentation

This document describes the COBOL programs in `src/cobol` and how they work together to manage a student account balance.

## Overview

The system is split into three COBOL programs:

- `main.cob` provides the user menu and dispatches actions.
- `operations.cob` performs account operations (view total, credit, debit).
- `data.cob` stores and returns the current account balance.

The programs communicate through `CALL ... USING` and operation codes.

## File Purposes

### `src/cobol/main.cob` (`PROGRAM-ID. MainProgram`)

Purpose:
- Acts as the entry point and interactive menu for account management.

Key logic:
- Repeats until user selects Exit.
- Accepts one menu choice (`1-4`).
- Translates user choices into operation codes and calls `Operations`:
  - `1` -> `TOTAL `
  - `2` -> `CREDIT`
  - `3` -> `DEBIT `
- Handles invalid menu choices with a validation message.

### `src/cobol/operations.cob` (`PROGRAM-ID. Operations`)

Purpose:
- Contains business operations that modify or display student account balances.

Key logic:
- Receives an operation code from `MainProgram`.
- Calls `DataProgram` with:
  - `READ` to fetch current balance.
  - `WRITE` to persist updated balance.
- Supported operations:
  - `TOTAL `: displays current balance.
  - `CREDIT`: prompts for amount, adds to balance, saves new balance.
  - `DEBIT `: prompts for amount, subtracts only if sufficient funds exist.

### `src/cobol/data.cob` (`PROGRAM-ID. DataProgram`)

Purpose:
- Encapsulates balance storage and read/write access.

Key logic:
- Maintains `STORAGE-BALANCE` in working storage.
- Receives operation via linkage (`PASSED-OPERATION`) and balance by reference.
- Supports two internal operations:
  - `READ`: copies stored balance to caller.
  - `WRITE`: updates stored balance from caller.

## Key Functions and Interactions

1. User selects an action in `MainProgram`.
2. `MainProgram` calls `Operations` with a 6-character operation token.
3. `Operations` calls `DataProgram` to retrieve and/or store balance.
4. Updated balance is displayed and retained for future operations.

## Business Rules for Student Accounts

The current implementation enforces these rules:

- Starting balance defaults to `1000.00`.
- Balance and amount precision is two decimal places (`PIC 9(6)V99`).
- Debit transactions are blocked when requested amount exceeds current balance.
- Credit transactions always increase the current balance.
- Balance changes persist only through `DataProgram` `WRITE` calls.
- Menu loop ends only when user chooses option `4` (Exit).

## Important Technical Notes

- Operation codes are fixed-length (`PIC X(6)`), so some values include trailing spaces (for example `TOTAL ` and `DEBIT `).
- No explicit validation is implemented for negative input amounts; behavior depends on runtime input handling and should be treated as a potential enhancement area.
- Storage is in-memory (working storage), not file- or database-backed.

## Suggested Enhancements

- Add validation to reject negative or zero transaction amounts.
- Add student identifier support to manage multiple student accounts.
- Persist balances to a file or database for durability across runs.
- Add audit logging for each credit/debit operation.
