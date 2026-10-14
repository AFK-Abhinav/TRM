# 📈 TRM - Trade Risk Management & Journaling Ecosystem

**TRM (Trade Risk Management)** is a high-performance, feature-packed Flutter application designed for stock, futures, and options traders. It serves as an all-in-one trading journal, position sizing calculator, analytics dashboard, and strategy backtesting tool—empowering traders to maintain strict risk control, eliminate emotional trading, and track performance with precision.

---

## 🎯 Key Capabilities & Highlights

- **🛡️ Disciplined Risk Sizing**: Calculate precise position sizes (quantity/lots) based on total account capital, risk percentage per trade, entry price, and stop loss.
- **📔 Automated Trade Journal**: Log trades with live estimated P&L preview, trade direction (Buy/Sell), lot size, trade date, and strategy notes.
- **📊 Real-time Performance Analytics**: Interactive bar charts (via `fl_chart`), win rate calculation, average win vs. average loss metrics, and monthly breakdown.
- **🔄 Smart Auth & Dynamic Navigation Wrapper**: Automated Firebase session handling and setup workflow check.
- **🧪 Strategy Backtesting Engine**: Pre-built Exponential Moving Average (EMA 9 vs EMA 20) crossover backtesting module to validate strategies before executing with real capital.
- **🎨 Ultra-Modern Cyberpunk Dark Theme**: Fluid 60fps animations, custom glowing cards, glassmorphism UI elements, and haptic feedback.

---

## 🏗️ Application Architecture & Navigation Flow

The app follows a stream-driven reactive architecture using **Firebase Auth**, **Cloud Firestore**, and **GetX**.

```
                         [ App Launch: lib/main.dart ]
                                       │
                                       ▼
                          [ lib/wrapper_page.dart ]
                                       │
                       ┌───────────────┴───────────────┐
                       ▼                               ▼
            [ Unauthenticated ]               [ Authenticated ]
                       │                               │
                       ▼                               ▼
              [ lib/pages/login_page.dart ]    [ Check Firestore: Setting/{uid} ]
                       │                               │
            ┌──────────┴──────────┐          ┌─────────┴─────────┐
            ▼                     ▼          ▼                   ▼
    [ signup_page.dart ] [ forgot_psd.dart ] [ No Settings ]   [ Profile Exists ]
                                             │                   │
                                             ▼                   ▼
                                    [ setting_page.dart ] [ home page.dart ]
```

### Flow Breakdown:
1. **`main.dart`**: Initializes `Firebase.initializeApp()` and launches `GetMaterialApp` targeting `wrapper_page`.
2. **`wrapper_page.dart`**: Listens to `FirebaseAuth.instance.authStateChanges()`.
   - If **Logged Out** $\rightarrow$ Renders `login_page`.
   - If **Logged In** $\rightarrow$ Fetches `/Setting/{user.uid}` from Firestore.
     - If settings do not exist $\rightarrow$ Redirects to `setting_page` to force initial profile setup.
     - If settings exist $\rightarrow$ Renders `home page` dashboard.

---

## 📁 Detailed Page-by-Page Explanation

### 1. 🔐 Authentication & Onboarding
- **`lib/wrapper_page.dart`**
  - **Purpose**: Central routing guard that seamlessly handles authentication states and user profile existence.
  - **Logic**: Uses nested `StreamBuilder`s to reactively switch screens when login state or Firestore profile data changes.

- **`lib/pages/login_page.dart`**
  - **Purpose**: User login screen.
  - **Features**: Email/Password authentication, Google Sign-In (`google_sign_in`), animated logo scaling, form validation, and direct navigation to Sign Up and Password Reset screens.

- **`lib/pages/signup_page.dart`**
  - **Purpose**: New trader registration.
  - **Features**: Creates new user credentials via Firebase Auth (`createUserWithEmailAndPassword`), validates matching passwords, and automatically updates session state.

- **`lib/pages/forgot_psd.dart`**
  - **Purpose**: Password recovery screen.
  - **Features**: Sends password reset links to registered email addresses via `sendPasswordResetEmail`.

- **`lib/pages/setting_page.dart`**
  - **Purpose**: Profile and risk parameter configuration screen.
  - **Inputs**:
    - **User Name**: Display name on dashboard.
    - **Trading Capital (₹)**: Total trading account capital.
    - **Risk Per Trade (%)**: Maximum risk percentage allowed per trade.
    - **Max Loss In Day (%)**: Daily loss limit percentage.
  - **Storage**: Saves configuration to Cloud Firestore under `/Setting/{uid}`.

---

### 2. 🏠 Main Dashboard
- **`lib/pages/home page.dart`**
  - **Purpose**: Main hub for trading metrics, quick tools, and recent activity.
  - **Key Components**:
    - **Greeting & Live Market Status Indicator**: Animated pulse badge indicating market open state.
    - **Trader Capital Card**: Displays registered capital, risk per trade, and daily max loss.
    - **Today's Overview (Live Stream)**: Real-time calculation of today's P&L, trade count, and win rate fetched from Firestore for `date == yyyy-MM-dd`.
    - **Quick Actions Grid**: One-tap access to *Risk Calculator*, *Add Trade*, *View Analytics*, *Option Calc*, and *Backtesting*.
    - **Recent Activity**: Displays the 3 most recently logged trades with color-coded profit/loss badges.

---

### 3. 📝 Trade Journaling & History
- **`lib/pages/Add_Trad.dart`**
  - **Purpose**: Form to log executed trades.
  - **Features**:
    - **Live P&L Preview Banner**: Instantly calculates estimated profit/loss as entry price, exit price, and lot size are typed.
    - **Instrument Name**: Uppercase formatted stock/symbol input.
    - **Direction Toggle**: Switch between **BUY** and **SELL**.
    - **Entry & Exit Prices**: Decimal number inputs with custom regex formatters.
    - **Lot / Quantity**: Integer lot/share size.
    - **Trade Date**: Interactive date picker defaulting to current date.
    - **Trade Notes**: Optional textarea to record trade setup, psychology, or market conditions.
    - **P&L Logic**:
      $$\text{Buy P\&L} = (\text{Exit Price} - \text{Entry Price}) \times \text{Lot Size}$$
      $$\text{Sell P\&L} = (\text{Entry Price} - \text{Exit Price}) \times \text{Lot Size}$$
  - **Storage**: Saves document into `/Notes` collection with `uid` and server timestamp.

- **`lib/pages/Total_trade.dart`**
  - **Purpose**: Comprehensive trade history log and management journal.
  - **Features**:
    - **Summary Strip**: Overview of Total Trades, Total Wins, Total Losses, and Net P&L.
    - **Filter Chips**: Filter trades by `All`, `Buy`, `Sell`, `Win`, or `Loss`.
    - **Sort Options**: Sort trade list by `Date`, `P/L`, or `Stock Name`.
    - **Expandable Trade Cards**: Tap any trade card to expand and view entry price, exit price, lot size, P&L, date, trade notes, and a **Delete Trade** button with confirmation dialog.

---

### 4. 📊 Analytics & Performance Dashboard
- **`lib/pages/View_Analytics.dart`**
  - **Purpose**: In-depth analytical reporting tool to inspect trading performance and profitability metrics.
  - **Key Features**:
    - **Hero Total P&L Banner**: Displays overall account net profit/loss with glowing visual accents.
    - **Stats Grid**:
      - Total Trades Count
      - Win Rate Percentage ($\frac{\text{Winning Trades}}{\text{Total Trades}} \times 100$)
      - Winning Trades vs. Losing Trades count
      - Average Winning Trade Amount
      - Average Losing Trade Amount
    - **Monthly P&L Bar Chart (`fl_chart`)**: Interactive bar chart comparing monthly net profits and losses.
    - **Monthly Summary Breakdown**: Detailed list of net results aggregated by month (`yyyy-MM`).

---

### 5. 🧮 Risk Management & Calculators
- **`lib/pages/Risk_calculator.dart`**
  - **Purpose**: Smart position sizing engine designed to prevent over-leveraging and maintain fixed risk per trade.
  - **Inputs**:
    - Trade Type (**BUY** or **SELL**)
    - Entry Price
    - Stop Loss Price
    - Target Price (Optional - auto-calculates 1:2 R:R if left empty)
  - **Outputs**:
    - **Position Size (Quantity/Lots)**:
      $$\text{Risk Amount} = \text{Capital} \times \frac{\text{Risk \%}}{100}$$
      $$\text{Quantity} = \left\lfloor \frac{\text{Risk Amount}}{|\text{Entry Price} - \text{Stop Loss}|} \right\rfloor$$
    - **Max Loss (₹)**
    - **Expected Profit (₹)**
    - **Risk-Reward (R:R) Ratio**: e.g., `1:2.5`
    - **Target Price (₹)**
  - **Smart Advisory Engine**: Displays warnings for excessive capital utilization (>50%) and provides advice based on the calculated Risk-Reward ratio.

- **`lib/pages/Quick_Option_clac.dart`**
  - **Purpose**: Quick Option pricing and premium calculation utility.

---

### 6. 🧪 Strategy Backtesting Engine
- **`lib/pages/Backtesting/backtesting.dart`**
  - **Purpose**: Algorithmic strategy testing module.
  - **Features**:
    - Implements an **EMA Crossover Strategy** (EMA 9 vs. EMA 20).
    - Calculates Exponential Moving Averages dynamically over closing price series:
      $$\text{Multiplier } k = \frac{2}{\text{Period} + 1}$$
      $$\text{EMA}_{\text{today}} = (\text{Price}_{\text{today}} \times k) + (\text{EMA}_{\text{yesterday}} \times (1 - k))$$
    - Simulates trade executions on EMA crossover signals.
    - Displays overall backtest stats: Total Trades, Winning Trades, Losing Trades, Win Rate %, and Total Profit.
    - Displays an itemized list of all backtested trades with entry price, exit price, trade type, and profit.

---

## 🗄️ Database Schema (Firebase Firestore)

### Collection: `Setting`
*Document ID*: `{user.uid}`

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `User` | `String` | Trader display name |
| `Capital` | `String` | Account capital amount (e.g. "100000") |
| `Risk per trade` | `String` | Allowed risk percentage per trade (e.g. "1.5") |
| `Max loss in day` | `String` | Allowed max daily loss percentage (e.g. "3.0") |
| `uid` | `String` | Firebase Auth user ID |
| `timestamp` | `FieldValue` | Profile creation/modification server timestamp |

### Collection: `Notes`
*Document ID*: Auto-generated

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `uid` | `String` | Owner user ID |
| `Stockname` | `String` | Stock or index symbol name (e.g. "NIFTY", "RELIANCE") |
| `type` | `String` | Direction: `"Buy"` or `"Sell"` |
| `EntryPrice` | `String` | Trade entry price |
| `ExitPrice` | `String` | Trade exit price |
| `lot_size` | `String` | Number of lots or quantity |
| `date` | `String` | Trade date formatted as `yyyy-MM-dd` |
| `profitOrLoss` | `double` | Net P&L for the trade |
| `Note` | `String` | Additional notes or trade commentary |
| `timestamp` | `FieldValue` | Server creation timestamp |

---

## 🛠️ Tech Stack & Dependencies

| Library / Tool | Version | Purpose |
| :--- | :--- | :--- |
| **Flutter SDK** | `^3.9.0` | Cross-platform UI toolkit |
| **GetX** | `^4.7.3` | High-performance navigation & state management |
| **Firebase Core** | `^4.3.0` | Firebase integration foundation |
| **Firebase Auth** | `^6.1.3` | User authentication & session management |
| **Cloud Firestore** | `^6.1.1` | Real-time cloud database |
| **Google Sign-In** | `^6.2.1` | OAuth2 Google account authentication |
| **FL Chart** | `^0.68.0` | Interactive charts for trade analytics |
| **Intl** | `^0.19.0` | Date formatting and number parsing |
| **Pinput** | `^6.0.1` | PIN / OTP input widgets |
| **CSV** | `^5.0.2` | CSV file parsing for strategy backtesting data |

---

## ⚙️ Setup & Installation Instructions

### 1. Prerequisites
- [Flutter SDK](https://docs.flutter.dev/get-started/install) (v3.19.0 or higher)
- [Android Studio](https://developer.android.com/studio) or VS Code with Flutter extension
- Firebase Project configured on [Firebase Console](https://console.firebase.google.com/)

### 2. Clone & Install
```bash
# Clone repository
git clone https://github.com/your-username/trm.git

# Move into project directory
cd trm

# Install dependencies
flutter pub get
```

### 3. Firebase Configuration
1. Register an Android app in your Firebase Console using package name found in `android/app/build.gradle`.
2. Download `google-services.json` and place it in `android/app/google-services.json`.
3. Enable **Email/Password** and **Google** sign-in methods under Firebase Authentication.
4. Enable **Cloud Firestore** in test or production mode.

### 4. Run Application
```bash
# Run on connected device or emulator
flutter run
```

---

## 👥 Audience & Project Presentation

This README file is structured to give developers, investors, and technical reviewers an immediate, comprehensive understanding of TRM's architecture, trading logic, risk features, database schema, and source code layout.
