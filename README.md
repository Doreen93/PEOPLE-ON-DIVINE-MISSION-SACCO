# PEOPLE-ON-DIVINE-MISSION-SACCO-- Create SACCO database
CREATE DATABASE sacco_db;
USE sacco_db;

-- Members Table
CREATE TABLE members (
    member_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone_number VARCHAR(20) UNIQUE NOT NULL,
    national_id VARCHAR(20) UNIQUE NOT NULL,
    date_of_birth DATE NOT NULL,
    address TEXT,
    join_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) DEFAULT 'active' CHECK (status IN ('active', 'inactive', 'suspended'))
);

-- Savings Table
CREATE TABLE savings (
    saving_id SERIAL PRIMARY KEY,
    member_id INT REFERENCES members(member_id) ON DELETE CASCADE,
    amount DECIMAL(12,2) NOT NULL,
    transaction_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    transaction_type VARCHAR(20) CHECK (transaction_type IN ('deposit', 'withdrawal')),
    balance DECIMAL(12,2) NOT NULL
);

-- Loans Table
CREATE TABLE loans (
    loan_id SERIAL PRIMARY KEY,
    member_id INT REFERENCES members(member_id) ON DELETE CASCADE,
    loan_amount DECIMAL(12,2) NOT NULL,
    interest_rate DECIMAL(5,2) NOT NULL,
    duration_months INT NOT NULL,
    status VARCHAR(20) CHECK (status IN ('pending', 'approved', 'rejected', 'active', 'cleared')),
    issue_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    due_date DATE
);

-- Loan Repayments Table
CREATE TABLE loan_repayments (
    repayment_id SERIAL PRIMARY KEY,
    loan_id INT REFERENCES loans(loan_id) ON DELETE CASCADE,
    member_id INT REFERENCES members(member_id) ON DELETE CASCADE,
    amount_paid DECIMAL(12,2) NOT NULL,
    payment_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    balance DECIMAL(12,2) NOT NULL
);

-- Transactions Table
CREATE TABLE transactions (
    transaction_id SERIAL PRIMARY KEY,
    member_id INT REFERENCES members(member_id) ON DELETE CASCADE,
    transaction_type VARCHAR(20) CHECK (transaction_type IN ('deposit', 'withdrawal', 'loan_disbursement', 'loan_repayment')),
    amount DECIMAL(12,2) NOT NULL,
    transaction_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Shares Table
CREATE TABLE shares (
    share_id SERIAL PRIMARY KEY,
    member_id INT REFERENCES members(member_id) ON DELETE CASCADE,
    shares_count INT NOT NULL,
    share_value DECIMAL(12,2) NOT NULL,
    total_value DECIMAL(12,2) GENERATED ALWAYS AS (shares_count * share_value) STORED,
    purchase_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Guarantors Table
CREATE TABLE guarantors (
    guarantor_id SERIAL PRIMARY KEY,
    loan_id INT REFERENCES loans(loan_id) ON DELETE CASCADE,
    member_id INT REFERENCES members(member_id) ON DELETE CASCADE,
    guarantee_amount DECIMAL(12,2) NOT NULL
);

-- Users Table (For system admins & staff)
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    role VARCHAR(20) CHECK (role IN ('admin', 'accountant', 'loan_officer', 'member')),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes for faster queries
CREATE INDEX idx_member_email ON members(email);
CREATE INDEX idx_member_phone ON members(phone_number);
CREATE INDEX idx_transaction_member ON transactions(member_id);
CREATE INDEX idx_loans_member ON loans(member_id);
CREATE INDEX idx_savings_member ON savings(member_id);

-- Foreign Key Constraints
ALTER TABLE savings ADD CONSTRAINT fk_savings_member FOREIGN KEY (member_id) REFERENCES members(member_id) ON DELETE CASCADE;
ALTER TABLE loans ADD CONSTRAINT fk_loans_member FOREIGN KEY (member_id) REFERENCES members(member_id) ON DELETE CASCADE;
ALTER TABLE loan_repayments ADD CONSTRAINT fk_repayments_loan FOREIGN KEY (loan_id) REFERENCES loans(loan_id) ON DELETE CASCADE;
ALTER TABLE loan_repayments ADD CONSTRAINT fk_repayments_member FOREIGN KEY (member_id) REFERENCES members(member_id) ON DELETE CASCADE;
ALTER TABLE transactions ADD CONSTRAINT fk_transactions_member FOREIGN KEY (member_id) REFERENCES members(member_id) ON DELETE CASCADE;
ALTER TABLE shares ADD CONSTRAINT fk_shares_member FOREIGN KEY (member_id) REFERENCES members(member_id) ON DELETE CASCADE;
ALTER TABLE guarantors ADD CONSTRAINT fk_guarantors_loan FOREIGN KEY (loan_id) REFERENCES loans(loan_id) ON DELETE CASCADE;
ALTER TABLE guarantors ADD CONSTRAINT fk_guarantors_member FOREIGN KEY (member_id) REFERENCES members(member_id) ON DELETE CASCADE;


