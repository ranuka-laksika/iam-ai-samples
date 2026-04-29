# The Sleeping Guardian: AI Agent with CIBA Authorization

A demonstration of **Client Initiated Backchannel Authentication (CIBA)** using a proactive AI agent that monitors stock markets and requests human authorization before executing high-value trades.

## The Story

**Meet Alice**, a high-net-worth investor who uses **Aurelius**, her personal AI agent. Aurelius's job is to watch the stock market 24/7 and protect Alice's portfolio from market crashes by identifying "Golden Buy" opportunities.

**The Conflict**: It's 3:00 AM. Alice is asleep. Suddenly, NVDA stock crashes 15% due to a market event. Aurelius detects this as a golden opportunity to buy shares at a discount. However, Alice has a security policy: **Any trade over $1,000 requires manual approval**.

**The Solution**: Aurelius cannot log Alice in. Instead, it uses **CIBA (Client Initiated Backchannel Authentication)** via WSO2 Identity Server. Aurelius initiates a backchannel request, and Alice receives an SMS/Email notification. She taps a link, reviews the trade details, and approves it from her phone—all without leaving her bed. Aurelius receives the authorization token and executes the trade.

## Architecture

This demo consists of three main components:

1. **Market Engine**: A synthetic stock market simulator that allows you to trigger controlled market events
2. **Aurelius Agent**: A background AI agent that monitors prices and identifies trading opportunities
3. **CIBA Client**: Integration with WSO2 Identity Server for secure, asynchronous user authorization

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  Market Engine  │────▶│ Aurelius Agent   │────▶│  CIBA Client    │
│  (Simulator)    │     │  (Monitoring)    │     │  (WSO2 IS)      │
└─────────────────┘     └──────────────────┘     └─────────────────┘
                               │                           │
                               │                           ▼
                               │                  ┌─────────────────┐
                               │                  │ Email/SMS to    │
                               │                  │ Alice's Phone   │
                               │                  └─────────────────┘
                               │                           │
                               │                           ▼
                               │                  ┌─────────────────┐
                               └─────────────────▶│ User Approves   │
                                                  │ Trade Executes  │
                                                  └─────────────────┘
```

## Features

- **Real-time Market Simulation**: Controllable stock price movements and market crashes
- **Proactive AI Monitoring**: Aurelius continuously watches for trading opportunities
- **Secure CIBA Authorization**: Human-in-the-loop approval via backchannel authentication
- **Live Dashboard**: Beautiful web interface showing market status and portfolio
- **Mobile-First Approval**: Realistic phone interface for reviewing and approving trades
- **Complete Audit Trail**: Full history of agent activities and decisions

## Prerequisites

- Python 3.8 or higher
- WSO2 Identity Server (Asgardeo) account
- Configured CIBA application in WSO2 IS
- Email or SMS configured for CIBA notifications

## Installation

### 1. Clone and Navigate

```bash
cd sleeping-guardian
```

### 2. Create Virtual Environment

```bash
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure Environment

Copy the example environment file and configure it:

```bash
cp .env.example .env
```

Edit `.env` and fill in your credentials:

```bash
# WSO2 Asgardeo Configuration
ASGARDEO_BASE_URL=https://api.asgardeo.io/t/your-org
CLIENT_ID=your-client-id
CLIENT_SECRET=your-client-secret

# Agent Credentials
AGENT_ID=your-agent-id
AGENT_SECRET=your-agent-secret

# User Configuration
INVESTOR_EMAIL=alice@example.com

# CIBA Configuration
CIBA_NOTIFICATION_CHANNEL=email  # or "sms"
```

## WSO2 Identity Server Setup

### 1. Create an Application

1. Log in to your WSO2 Asgardeo console
2. Go to **Applications** → **New Application**
3. Choose **Standard-Based Application**
4. Configure the following:
   - **Name**: Aurelius Agent
   - **Grant Types**: Enable "CIBA", "Client Credentials"
   - **Scopes**: `openid`, `stock:read`, `stock:trade`

### 2. Configure CIBA Settings

1. In your application settings, go to **Protocol** tab
2. Enable **CIBA (Backchannel Authentication)**
3. Configure notification settings:
   - **Channel**: Email or SMS
   - **Token Delivery Method**: Poll
   - **Binding Message**: Enabled

### 3. Create an Agent Identity

1. Go to **Users & Roles** → **Agents**
2. Create a new agent:
   - **Name**: Aurelius Trading Agent
   - **Description**: AI agent for portfolio management
3. Note the Agent ID and Agent Secret

### 4. Configure User for CIBA

1. Ensure the user (Alice) has a verified email or phone number
2. The user should have permissions for the scopes: `stock:read`, `stock:trade`

## Running the Demo

### 1. Start the Application

```bash
python app.py
```

You should see:

```
================================================================================
🏦 AURELIUS - The Sleeping Guardian
================================================================================
Investor: alice@example.com
Stock Symbol: NVDA
Initial Price: $150.00
Price Threshold: $140.00
Trade Amount: $5000.00
CIBA Channel: email
================================================================================

🌐 Dashboard: http://localhost:5000
📱 Mobile View: http://localhost:5000/phone
```

### 2. Open the Dashboard

Navigate to `http://localhost:5000` in your browser. You'll see:

- **Market Status**: Current stock price and market conditions
- **Portfolio**: Your current balance and holdings
- **Agent Activity**: Real-time log of Aurelius's decisions

### 3. Open Mobile View

In a separate window or device, open `http://localhost:5000/phone`. This simulates Alice's mobile phone where CIBA notifications will appear.

## Conducting the Demo

Follow these steps for a compelling presentation:

### Step 1: Show the Normal State

- Point out the market price hovering around $150
- Explain that Aurelius is using **M2M credentials** with `stock:read` permission
- The agent is monitoring but **cannot trade** without user approval

### Step 2: Trigger the Event

- Click the **"⚠️ TRIGGER MARKET CRASH"** button
- Watch the price start dropping: $148... $145... $142...

### Step 3: The AI Reaction

- When the price hits $139 (below the threshold), Aurelius springs into action
- The console will show:
  ```
  [Aurelius] 🚨 GOLDEN BUY OPPORTUNITY DETECTED!
  [Aurelius] 📉 Current price: $139.45
  [CIBA] Initiating authorization request...
  [CIBA] 📩 Authorization request sent via email
  ```

### Step 4: The Human Interaction

- Switch to the **Mobile View** (`/phone`)
- You'll see a notification appear on the "phone screen"
- The notification shows:
  - Stock symbol and current price
  - Number of shares to buy
  - Total transaction amount
  - Security notice about CIBA

### Step 5: The Authorization

**In a real scenario**, Alice would receive this on her actual phone via Email/SMS. For the demo:

1. Click **"✓ Approve Trade"** on the mobile view
2. You'll see a success confirmation

### Step 6: The Result

- Switch back to the **Dashboard**
- Watch the trade execute:
  - Balance decreases by ~$5,000
  - Shares owned increases
  - Market stabilizes/recovers
  - Activity log shows the completed trade

## Key Demo Points to Highlight

### Security Benefits

1. **No Stored Credentials**: Aurelius doesn't have Alice's password
2. **Limited Scope**: Agent can only read market data without approval
3. **Just-In-Time Authorization**: Trading permission granted only when needed
4. **Binding Message**: Alice sees exactly what she's approving
5. **Audit Trail**: Complete log of all authorization requests

### CIBA Advantages

1. **Asynchronous**: Alice doesn't need to be at her computer
2. **User-Friendly**: Approval via familiar channels (Email/SMS)
3. **Secure**: Token delivery via backchannel, not front channel
4. **Context-Rich**: Binding message provides transaction details
5. **Timeout Protection**: Requests expire if not approved

### AI Agent Benefits

1. **24/7 Monitoring**: Never misses an opportunity
2. **Fast Reaction**: Can act within seconds when approved
3. **Human Oversight**: Cannot execute without explicit permission
4. **Transparent**: Shows reasoning and actions in activity log

## API Endpoints

The application provides several API endpoints for integration:

- `GET /` - Main dashboard
- `GET /phone` - Mobile phone view
- `GET /api/market` - Current market data (JSON)
- `GET /api/portfolio` - Portfolio status (JSON)
- `GET /api/status` - Complete system status (JSON)
- `GET /crash` - Trigger market crash
- `GET /stabilize` - Stabilize market
- `GET /approve` - Approve pending trade
- `GET /deny` - Deny pending trade
- `GET /reset` - Reset demo to initial state

## Customization

### Adjust Trading Parameters

Edit `.env` to customize:

```bash
# Market behavior
INITIAL_STOCK_PRICE=150.00
PRICE_THRESHOLD=140.00  # When to trigger buy alert

# Trading amounts
TRADE_AMOUNT=5000.00
INITIAL_BALANCE=10000.00
```

### Change Stock Symbol

```bash
STOCK_SYMBOL=AAPL  # or any other symbol
```

### Switch Notification Channel

```bash
CIBA_NOTIFICATION_CHANNEL=sms  # Change from email to SMS
```

## Troubleshooting

### CIBA Request Fails

- **Check credentials**: Ensure `CLIENT_ID`, `CLIENT_SECRET`, `AGENT_ID`, and `AGENT_SECRET` are correct
- **Verify scopes**: Make sure the agent has permission to request required scopes
- **Check user email**: Ensure `INVESTOR_EMAIL` matches a verified user in WSO2 IS

### No Notification Received

- **Verify channel**: Check that Email/SMS is properly configured in WSO2 IS
- **Check spam folder**: Email notifications might be filtered
- **Verify user contact**: Ensure the user has a verified email/phone number

### Agent Not Monitoring

- Check console for errors
- Ensure all environment variables are set
- Verify network connectivity to WSO2 IS

## Architecture Details

### Market Engine (`market_engine.py`)

- Simulates realistic price movements
- Supports different market states: STABLE, VOLATILE, CRASHING, RECOVERING
- Thread-safe price updates
- Historical price tracking

### Aurelius Agent (`aurelius_agent.py`)

- Asynchronous monitoring loop
- Detects price drops below threshold
- Initiates CIBA requests
- Executes approved trades
- Maintains portfolio state

### CIBA Client (`ciba_client.py`)

- Manages WSO2 IS authentication
- Handles CIBA flow: initiate → poll → token
- Provides On-Behalf-Of (OBO) tokens
- Error handling and retries

## Real-World Use Cases

This pattern is applicable to:

1. **Financial Trading Bots**: Require approval for high-value transactions
2. **Expense Approval Systems**: Agents process transactions pending manager approval
3. **Healthcare Bots**: AI assistants require doctor approval for prescriptions
4. **IoT Devices**: Smart home devices requesting permission for costly actions
5. **Autonomous Vehicles**: Requesting human override in edge cases

## Security Considerations

- **Never hardcode credentials**: Always use environment variables
- **Scope Minimization**: Agent requests only required scopes
- **Token Expiry**: Tokens have limited lifetime
- **Binding Messages**: Always show what action requires approval
- **Audit Logging**: Track all authorization requests and outcomes

## License

Copyright (c) 2025, WSO2 LLC. (http://www.wso2.com). All Rights Reserved.

This software is the property of WSO2 LLC. and its suppliers, if any. Dissemination of any information or reproduction of any material contained herein is strictly forbidden, unless permitted by WSO2 in accordance with the WSO2 Commercial License.

## Support

For issues or questions:
- WSO2 Documentation: https://wso2.com/asgardeo/docs/
- CIBA Specification: https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html

## Contributing

This is a demo application. For production use, additional security measures and error handling should be implemented.
