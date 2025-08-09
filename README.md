# BankingAuthSystem Prototype

## Overview
This prototype is a banking authentication system built with .NET 6. It includes a server (`BankingAuthSystem.Server`) that handles user registration, login, OTP verification, money transfers, and fraud detection, a WPF client (`BankingAuthSystem`) for the user interface, and a test project (`BankingAuthSystem.Tests`) for unit testing. The system uses MySQL (via XAMPP) for data storage.

## Prerequisites
- **XAMPP**: For running MySQL and phpMyAdmin.
- **.NET 6 SDK**: For building and running the server, WPF app, and tests.
- **Visual Studio 2022**: Recommended IDE for running the solution.
- **MySQL Connector/NET**: Included via NuGet in the project (`MySql.Data` package).

## Setup Instructions

### 1. Install XAMPP
1. Download and install XAMPP from [https://www.apachefriends.org/](https://www.apachefriends.org/).
2. Start the XAMPP Control Panel as Administrator.
3. Configure MySQL and Apache:
   - Configure MySQL to run on port 3307 (default might be 3306):
     - Edit `C:\xampp\mysql\bin\my.ini`.
     - Under `[mysqld]`, set `port=3307`.
     - Save and restart MySQL if running.
   - Configure Apache to run on port 8081 (default might be 80):
     - Edit `C:\xampp\apache\conf\httpd.conf`.
     - Find `Listen 80` and change to `Listen 8081`.
     - Save and restart Apache if running.
4. Start MySQL and Apache in XAMPP Control Panel.

### 2. Set Up the Database
1. Open phpMyAdmin at `http://localhost:8081/phpmyadmin`.
2. Create a database named `banking_db`.
3. Run the following SQL to create the required tables:

   ```sql
   -- Users Table
   CREATE TABLE Users (
      UserId INT AUTO_INCREMENT PRIMARY KEY,
      Username VARCHAR(50) NOT NULL,
      HashedPassword VARCHAR(255) NOT NULL,
      Email VARCHAR(100) NOT NULL,
      TrustedCountries TEXT DEFAULT NULL,
      AccountBalance DECIMAL(15,2) DEFAULT 0.00,
      OtpCode VARCHAR(6) DEFAULT NULL,
      OtpExpiry DATETIME DEFAULT NULL,
      LoginAttempts INT DEFAULT 0,
      LastLoginAttempt DATETIME DEFAULT NULL
   );

   -- FraudLog Table
   CREATE TABLE FraudLog (
      LogId INT AUTO_INCREMENT PRIMARY KEY,
      Username VARCHAR(50) NOT NULL,
      IpAddress VARCHAR(45) NOT NULL,
      AttemptTime DATETIME DEFAULT CURRENT_TIMESTAMP,
      IsLocked TINYINT(1) DEFAULT 0,
      FOREIGN KEY (Username) REFERENCES Users(Username) ON DELETE CASCADE
   );

   -- Transactions Table
   CREATE TABLE Transactions (
      TransactionId INT AUTO_INCREMENT PRIMARY KEY,
      SenderUsername VARCHAR(50) NOT NULL,
      ReceiverUsername VARCHAR(50) NOT NULL,
      Amount DECIMAL(15,2) DEFAULT 0.00,
      TransactionDate DATETIME DEFAULT CURRENT_TIMESTAMP,
      Description TEXT,
      FOREIGN KEY (SenderUsername) REFERENCES Users(Username) ON DELETE CASCADE,
      FOREIGN KEY (ReceiverUsername) REFERENCES Users(Username) ON DELETE CASCADE
   );

   -- FraudAlerts Table
   CREATE TABLE FraudAlerts (
      AlertId INT AUTO_INCREMENT PRIMARY KEY,
      TransactionId INT NOT NULL,
      Reason TEXT NOT NULL,
      AlertDate DATETIME DEFAULT CURRENT_TIMESTAMP,
      FOREIGN KEY (TransactionId) REFERENCES Transactions(TransactionId) ON DELETE CASCADE
   );
   ```

### 3. Configure the Server
1. Open the solution (`BankingAuthSystem.sln`) in Visual Studio 2022.
2. Open `BankingAuthSystem.Server/appsettings.json` and ensure the MySQL connection string matches your setup:
   ```json
   {
     "ConnectionStrings": {
       "MySql": "Server=localhost;Port=3307;Database=banking_db;User=root;Password=;"
     }
   }
   ```
   - Update `Password=` if you set a root password in MySQL.

3. Build the `BankingAuthSystem.Server` project to restore NuGet packages (e.g., `MySql.Data`).

### 4. Run the Project
1. **Start the Server**:
   - In Visual Studio, set `BankingAuthSystem.Server` as the startup project.
   - Run the project (F5). It should start an HTTP server (e.g., on `http://localhost:5000`).
   - Check the console for "Connection successful" to confirm the database connection.

2. **Start the WPF Client**:
   - Set `BankingAuthSystem` as the startup project.
   - Run the project (F5). The login window should appear.
   - Register a new user via the WPF app:
     - In the Login page click "Create account" to initiate registration.
     - Enter a username (e.g., `testuser`), email (e.g., `testuser@example.com`), trusted countries (e.g., `UK,USA`), and a password (e.g., `TestPass123`).
     - Follow the OTP verification process to complete registration.
   - Log in with the registered credentials.

### 5. Test the Features
- **Login**: Use the registered user (e.g., `testuser`/`TestPass123`) to log in.
- **OTP Verification**: After login, enter the OTP sent to the server console.
- **Dashboard**: View account balance and transaction history.
- **Transfer Money**: Transfer money to another user (e.g., register another user like `logan` and transfer £50).
- **Fraud Detection**: Transfers to users in untrusted countries will log alerts in the `FraudAlerts` table.
- **Run Tests**: Open `BankingAuthSystem.Tests`, build, and run tests in Test Explorer to verify functionality.

## Notes
- Ensure MySQL runs on port 3307 and Apache on port 8081 to avoid conflicts.
- If you encounter database errors (e.g., "Table doesn't exist in engine"), recreate the tables using the SQL provided.
- The system uses PBKDF2 for password hashing (16-byte salt, 32-byte hash, 100,000 iterations, SHA256).
