# RakshaPay — SIM Swap Attack Detection

### A Risk-Based Security Platform for Safer Digital Banking

RakshaPay is a demonstration banking application designed to detect and prevent SIM swap attacks through real-time fraud risk assessment and risk-based authentication.

A SIM swap attack occurs when a fraudster convinces a telecom provider to transfer a victim's phone number to another SIM card. This can allow attackers to intercept SMS-based OTPs and gain unauthorised access to banking accounts.

RakshaPay uses a hybrid risk engine combining rule-based detection and machine learning to evaluate login attempts and financial transactions. Based on the calculated risk score, the system allows access, requests additional verification, or blocks suspicious activity.

**Live Demo:** [https://rakshapay-sim-swap-detection.onrender.com/](https://rakshapay-sim-swap-detection.onrender.com/)

**Mobile Application:** [RakshaPay Mobile](https://github.com/rmonal-dev/RakshaPay-Mobile)

**API Documentation:** [FastAPI Swagger Docs](https://rakshapay-sim-swap-detection.onrender.com/api/docs)

---

## Problem Statement

Traditional banking authentication often relies heavily on SMS-based OTP verification. However, SIM swap attacks can compromise this authentication mechanism.

RakshaPay addresses this vulnerability by evaluating multiple risk signals before allowing sensitive operations, helping identify suspicious login attempts and potentially fraudulent transactions.

## How the Detection Works

The risk engine evaluates multiple signals during login attempts and transactions:

- Recent SIM swap activity.
- Device familiarity (known or new device).
- Transaction amount.
- IP address origin.
- Login time.
- Recent failed login attempts.
- Account age.
- Number of transactions within the last 24 hours.

### Hybrid Risk Engine

Two detection layers contribute to the final risk score.

**1. Rule-Based Detection**

Applies predefined risk scores to suspicious patterns, including:

- Recent SIM swap activity.
- Login from an unknown device.
- Unusually high-value transactions.
- Multiple failed login attempts.

**2. Machine Learning Detection**

An XGBoost classifier estimates the probability of fraudulent activity using the evaluated risk signals.

The final risk score is calculated using a 60/40 weighted combination of rule-based scoring and machine learning output, capped at 100.

A SIM swap detected within the last hour acts as a hard override and blocks the request.

### Risk Decision Thresholds

| Risk Score   | Decision  | Action                                  |
| ------------ | --------- | --------------------------------------- |
| Below 35     | ALLOW     | Normal authentication flow              |
| 35–69        | CHALLENGE | Additional verification required        |
| 70 and above | BLOCK     | Access denied and fraud alert generated |

The same risk engine evaluates money transfers. Blocked transfers are recorded with a BLOCK status and generate a `TXN_BLOCKED` alert.

## Key Features

- Real-time SIM swap detection during login and transfers.
- Hybrid rule-based and machine learning risk assessment.
- Risk-based authentication with ALLOW, CHALLENGE and BLOCK decisions.
- Live risk assessment meter on the login page.
- OTP-based verification.
- Account lockout after five failed login attempts for 15 minutes.
- Rate limiting on sensitive API endpoints.
- Protected administrative login.
- Admin dashboard with fraud logs, alerts and statistics.
- SIM swap attack simulator for demonstration.
- Audit trail of login attempts and transactions.
- PostgreSQL support through database configuration, including Supabase, with SQLite for local development.
- Shared backend for web and Flutter mobile applications.
- Interactive API documentation through Swagger.

## Technology Stack

| Layer                | Technology                           |
| -------------------- | ------------------------------------ |
| Backend              | Python 3.11, FastAPI, Uvicorn        |
| Machine Learning     | XGBoost, scikit-learn, NumPy, pandas |
| Authentication       | JWT, python-jose, HMAC-SHA256, OTP   |
| Database             | SQLAlchemy ORM, PostgreSQL, SQLite   |
| Security             | slowapi, Pydantic, account lockout   |
| Frontend             | HTML5, CSS3, Vanilla JavaScript      |
| Testing              | pytest                               |
| SIM Swap Integration | Vonage Network API                   |
| Deployment           | Render                               |
| Database Hosting     | Supabase PostgreSQL when configured  |

## System Architecture

RakshaPay follows a client-server architecture in which the web and mobile applications communicate with a common FastAPI backend.

```text
                RakshaPay Platform
                        |
               +--------+--------+
               |                 |
            Web App          Flutter App
               |                 |
               +--------+--------+
                        |
                   REST APIs
                        |
                 FastAPI Backend
                        |
             +----------+----------+
             |                     |
       Authentication          Risk Engine
             |                     |
             +----------+----------+
                        |
                  Database Layer
                        |
               PostgreSQL / SQLite
```

The backend handles authentication, risk assessment, transaction processing and database operations. Both clients are designed to access shared application data.

## Running Locally

### Prerequisites

- Python 3.11
- pip
- Git

### 1. Clone the Repository

```bash
git clone https://github.com/rmonal-dev/RakshaPay-Sim-Swap-Detection.git

cd RakshaPay-Sim-Swap-Detection
```

### 2. Create a Virtual Environment

```bash
python3.11 -m venv venv
```

Activate it:

**Windows:**

```bash
venv\Scripts\activate
```

**Linux / macOS:**

```bash
source venv/bin/activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Train the Machine Learning Model

```bash
python3.11 ml/train_model.py
```

This generates the model files required by the risk engine:

- `fraud_model.pkl`
- `scaler.pkl`
- `model_meta.json`

### 5. Start the Server

```bash
python3.11 main.py
```

Open the application at:

`http://localhost:8000`

### Environment Variables

Create a `.env` file in the project root.

```env
DATABASE_URL=postgresql+psycopg2://USER:PASSWORD@HOST:5432/postgres

JWT_SECRET=your-strong-random-secret
APP_ENV=development

ADMIN_PASSWORD=choose-a-strong-password

VONAGE_APPLICATION_ID=
VONAGE_PRIVATE_KEY_PATH=./vonage_private.key
VONAGE_ENVIRONMENT=sandbox
```

For local development, SQLite is used when `DATABASE_URL` is not configured. PostgreSQL can be configured through the database connection string.

Never commit `.env` files, private keys or database credentials.

## Demo Walkthrough

### Normal Login

1. Register a new account at `/register`.
2. Log in using the registered credentials.
3. The risk engine evaluates the login attempt.
4. Complete OTP verification when required.
5. Access the dashboard after successful authentication.
6. Navigate to `/transfer` to perform a transaction.

### Simulating a SIM Swap Attack

1. Open `/admin`.
2. Sign in using the administrator password.
3. Select a user and click **Simulate**.
4. Attempt to log in using that user's account.
5. Observe the elevated risk score and blocked login.
6. Check the generated fraud alert in the admin dashboard.

### Resetting the Simulation

Use the **Reset** option in the admin panel to reset the simulated SIM swap status.

## Project Structure

```text
RakshaPay-Sim-Swap-Detection/
│
├── main.py
├── models.py
├── auth.py
├── risk_engine.py
├── vonage_client.py
├── config.py
│
├── ml/
│   ├── train_model.py
│   ├── fraud_model.pkl
│   ├── scaler.pkl
│   └── model_meta.json
│
├── pages/
│   ├── login.html
│   ├── register.html
│   ├── dashboard.html
│   ├── transfer.html
│   ├── admin_login.html
│   └── admin.html
│
├── tests/
│   ├── test_auth.py
│   ├── test_risk.py
│   └── test_api.py
│
├── requirements.txt
├── runtime.txt
├── Procfile
└── run.sh
```

## API Endpoints

| Method | Endpoint                   | Description                           |
| ------ | -------------------------- | ------------------------------------- |
| POST   | `/api/register`            | Create an account                     |
| POST   | `/api/login`               | Login with risk assessment            |
| POST   | `/api/verify-otp`          | Verify OTP and receive JWT            |
| POST   | `/api/admin/login`         | Administrator login                   |
| POST   | `/api/admin/logout`        | Administrator logout                  |
| GET    | `/api/admin/session`       | Check administrator session           |
| GET    | `/api/user/me`             | Retrieve authenticated user profile   |
| GET    | `/api/user/{id}`           | Retrieve user profile                 |
| POST   | `/api/transfer`            | Initiate a risk-scored transfer       |
| GET    | `/api/admin/users`         | Retrieve users (admin)                |
| POST   | `/api/admin/simulate-swap` | Simulate SIM swap (admin)             |
| POST   | `/api/admin/reset-swap`    | Reset SIM swap status (admin)         |
| GET    | `/api/admin/fraud-logs`    | Retrieve fraud logs (admin)           |
| GET    | `/api/admin/alerts`        | Retrieve fraud alerts (admin)         |
| GET    | `/api/admin/stats`         | Retrieve dashboard statistics (admin) |
| GET    | `/api/admin/model-meta`    | Retrieve ML model metrics (admin)     |
| GET    | `/api/health`              | Check backend health                  |

**Interactive API Documentation:** [Open Swagger UI](https://rakshapay-sim-swap-detection.onrender.com/api/docs)

## Mobile Application Integration

RakshaPay also provides a Flutter-based mobile application connected to the shared backend.

- **Mobile Repository:** [RakshaPay Mobile](https://github.com/rmonal-dev/RakshaPay-Mobile)
- **Android Releases:** [Download RakshaPay Mobile](https://github.com/rmonal-dev/RakshaPay-Mobile/releases)

The mobile application provides login, OTP verification, MPIN protection, dashboard access and money transfers. Administrative controls remain available through the web application.

## Vonage Integration

The `vonage_client.py` module integrates with the Vonage Network SIM Swap API.

When the required application credentials and private key are configured, the application can perform carrier-level SIM swap checks.

Without these credentials, the application uses its local simulation or sandbox fallback for demonstration purposes.

The risk scoring and blocking logic remains part of the backend risk engine.

## Known Limitations

- The machine learning model is trained on synthetic data. Its reported AUC/F1 score of 1.0 reflects the characteristics of that synthetic dataset and should not be interpreted as real-world fraud detection performance.
- Demo OTP values are displayed in the application instead of being delivered through a separate communication channel.
- The web client supports an `X-User-Id` header for user identification. Production deployments should enforce JWT-based authorization consistently for all protected operations.
- HMAC-SHA256 password hashing is used in the documented implementation; Argon2 or bcrypt would be preferable for production password storage.
- Real carrier-level SIM swap checks require valid Vonage credentials and carrier support.
- The Render free tier may introduce cold starts after periods of inactivity.

## Future Enhancements

- Integration with real-world fraud datasets.
- Improved machine learning models trained on representative transaction data.
- Stronger authentication and device verification.
- Biometric authentication.
- Push notifications for suspicious activities.
- Enhanced fraud analytics and monitoring.

---

_RakshaPay — Building Safer Digital Payment Experiences._
