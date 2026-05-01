# The Sleeping Guardian: AI Agent with CIBA Authorization

A demonstration of **Client Initiated Backchannel Authentication (CIBA)** using a proactive AI agent that monitors stock markets and requests human authorization before executing high-value trades through an MCP (Model Context Protocol) server.

## The Story

**Meet Alice**, a high-net-worth investor who uses **Aurelius**, her personal AI agent. Aurelius's job is to watch the stock market 24/7 and protect Alice's portfolio from market crashes by identifying "Golden Buy" opportunities.

**The Conflict**: It's 3:00 AM. Alice is asleep. Suddenly, NVDA stock crashes 15% due to a market event. Aurelius detects this as a golden opportunity to buy shares at a discount. However, Alice has a security policy: **Any trade over $1,000 requires manual approval**.

**The Solution**: Aurelius cannot log Alice in. Instead, it uses **CIBA (Client Initiated Backchannel Authentication)** via WSO2 Identity Server (Asgardeo). Aurelius initiates a backchannel request, and Alice receives an SMS/Email notification. She taps a link, reviews the trade details, and approves it from her phone—all without leaving her bed. Aurelius receives the authorization token and executes the trade via a protected MCP Stock Trading Server.

## Architecture

This demo consists of four main components:

1. **Market Engine**: A synthetic stock market simulator that allows you to trigger controlled market events
2. **Aurelius Agent**: An LLM-powered AI agent that monitors prices and makes autonomous trading decisions
3. **MCP Stock Server**: A secured Model Context Protocol server that exposes stock trading tools with scope-based authorization
4. **CIBA Client**: Integration with WSO2 Identity Server (Asgardeo) for secure, asynchronous user authorization

```
┌─────────────────┐     ┌──────────────────────┐     ┌─────────────────┐
│  Market Engine  │────▶│  Aurelius Agent      │────▶│  CIBA Client    │
│  (Simulator)    │     │  (LLM + Monitoring)  │     │  (Asgardeo)     │
└─────────────────┘     └──────────────────────┘     └─────────────────┘
         │                       │                            │
         │                       │ Agent Token (stock:read)   │
         │                       │◄───────────────────────────┤
         │                       │                            │
         │                       │ Price drops!               │
         │                       │ Initiate CIBA request      │
         │                       │───────────────────────────▶│
         │                       │                            │
         │                       │                            ▼
         │                       │                   ┌─────────────────┐
         │                       │                   │ Email/SMS to    │
         │                       │                   │ Alice's Phone   │
         │                       │                   └─────────────────┘
         │                       │                            │
         │                       │                            ▼
         │                       │                   ┌─────────────────┐
         │                       │ OBO Token         │ User Approves   │
         │                       │◄──────────────────│ on Mobile       │
         │                       │ (stock:trade)     └─────────────────┘
         │                       │
         │                       ▼
         │              ┌─────────────────────┐
         └─────────────▶│  MCP Stock Server   │
                        │  (JWT Protected)    │
                        └─────────────────────┘
                                 │
                                 ▼
                        ┌─────────────────────┐
                        │   Trade Executed    │
                        │   Portfolio Updated │
                        └─────────────────────┘
```

## Features

- **Real-time Market Simulation**: Controllable stock price movements and market crashes
- **LLM-Powered AI Agent**: Uses Google Gemini to make intelligent trading decisions
- **MCP-Based Tool Access**: Agent interacts with protected stock trading tools via Model Context Protocol
- **Scope-Based Authorization**: Agent has `stock:read` permission; requires CIBA for `stock:trade`
- **Secure CIBA Authorization**: Human-in-the-loop approval via backchannel authentication
- **On-Behalf-Of Flow**: Agent acts on behalf of the user after approval
- **Live Dashboard**: Beautiful web interface showing market status and portfolio
- **Complete Audit Trail**: Full history of agent activities and decisions

## Prerequisites

Before starting, ensure you have:

- **Python 3.8 or higher**
- **WSO2 Asgardeo Account**: Sign up at https://console.asgardeo.io
- **Google AI API Key**: For Gemini LLM (get from https://aistudio.google.com/app/apikey)
- **Email Access**: For receiving CIBA approval notifications

## Quick Start Summary

If you want a quick overview before diving into details:

### Setup Checklist

**Asgardeo Configuration** (Detailed in "WSO2 Asgardeo Setup" section):
- [ ] Create "Stock MCP Server" application (for token validation)
- [ ] Create "Aurelius Agent" application (for agent auth)
- [ ] Create API Resource with scopes: `stock:read`, `stock:trade`, `stock:admin`
- [ ] Authorize API Resource to both applications
- [ ] Create agent identity and note AGENT_ID + AGENT_SECRET
- [ ] Create user (Alice) with verified email
- [ ] Create and assign `stock_trader` role to both agent and user
- [ ] Enable CIBA grant type and App-Native Authentication in Agent app
- [ ] Disable "Public Client" in both applications

**Local Setup** (Detailed in "Installation" section):
- [ ] Configure `mcp-stock-server/.env` with MCP server app credentials
- [ ] Configure main `.env` with agent app credentials + API keys
- [ ] Start MCP Stock Server: `cd mcp-stock-server && python main.py`
- [ ] Start main app: `python app.py`
- [ ] Access dashboard at http://localhost:5001

**Running the Demo** (Detailed in "Conducting the Demo" section):
- [ ] Trigger market crash in dashboard
- [ ] Watch agent detect opportunity and request CIBA authorization
- [ ] Check email for approval request
- [ ] Approve the trade
- [ ] Watch trade execute successfully

### Time Estimate
- Asgardeo setup: 20-30 minutes
- Installation: 10 minutes
- Running demo: 5 minutes
- **Total: ~45 minutes**

## Installation

### 1. Clone and Navigate

```bash
cd sleeping-guardian
```

### 2. Set Up MCP Stock Server

The MCP Stock Server must be configured and running first.

#### Install MCP Server Dependencies

```bash
cd mcp-stock-server
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

#### Configure MCP Server Environment

```bash
cp .env.example .env
```

Edit `mcp-stock-server/.env` with your **Stock MCP Server** application credentials from Step 11:

```bash
# Asgardeo/WSO2 Identity Server Configuration
AUTH_ISSUER=https://api.asgardeo.io/t/YOUR_ORG/oauth2/token
CLIENT_ID=<stock-mcp-server-client-id>
JWKS_URL=https://api.asgardeo.io/t/YOUR_ORG/oauth2/jwks

# MCP Server Configuration
MCP_SERVER_PORT=8200
```

**Important Notes**:
- Replace `YOUR_ORG` with your actual Asgardeo organization name
- Use the `CLIENT_ID` from the **Stock MCP Server** application (not the Agent application)
- The `AUTH_ISSUER` should end with `/oauth2/token`
- The `JWKS_URL` should end with `/oauth2/jwks`

#### Start the MCP Server

```bash
python main.py
```

Keep this terminal running. You should see:

```
================================================================================
Stock Trading MCP Server
================================================================================
Port: 8200
Issuer: https://api.asgardeo.io/t/YOUR_ORG/oauth2/token
Client ID: <your-client-id>
================================================================================

Available Scopes:
  - stock:read   : Read market data and portfolio
  - stock:trade  : Execute buy/sell trades
  - stock:admin  : Update market prices (simulation)
================================================================================
```

### 3. Set Up Main Application

Open a **new terminal** and navigate back to the sleeping-guardian directory.

#### Create Virtual Environment

```bash
cd ..  # Back to sleeping-guardian directory
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

#### Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure Main Application Environment

```bash
cp .env.example .env
```

Edit `.env` and fill in your credentials from Step 11:

```bash
# ---------------------------------------
# Asgardeo OAuth2 Configuration
# ---------------------------------------
ASGARDEO_BASE_URL=https://api.asgardeo.io/t/YOUR_ORG
CLIENT_ID=<aurelius-agent-client-id>
CLIENT_SECRET=<aurelius-agent-client-secret>
REDIRECT_URI=http://localhost:5001/callback

# ---------------------------------------
# Asgardeo Agent Credentials
# ---------------------------------------
AGENT_ID=<agent-id-from-agent-identity>
AGENT_SECRET=<agent-secret-from-agent-identity>

# ---------------------------------------
# Google Gemini API Key
# ---------------------------------------
GOOGLE_AI_API_KEY=<your-google-ai-api-key>

# LLM model used by the agent
MODEL_NAME=gemini-2.5-flash

# ---------------------------------------
# CIBA Configuration
# ---------------------------------------
CIBA_NOTIFICATION_CHANNEL=email

# ---------------------------------------
# Sleeping Guardian Configuration
# ---------------------------------------
# Investor email for CIBA notifications (Alice's verified email)
INVESTOR_EMAIL=<alice-email-address>

# Stock simulation settings
STOCK_SYMBOL=NVDA
INITIAL_STOCK_PRICE=150.00
PRICE_THRESHOLD=140.00

# Trading settings
TRADE_AMOUNT=5000.00
INITIAL_BALANCE=10000.00

# MCP Stock Server URL
STOCK_MCP_SERVER_URL=http://localhost:8200/mcp

# Flask app port
PORT=5001
FLASK_DEBUG=False
```

**Important Configuration Notes**:
- `CLIENT_ID` and `CLIENT_SECRET`: Use credentials from **Aurelius Agent** application
- `AGENT_ID` and `AGENT_SECRET`: Use credentials from the agent identity you created
- `INVESTOR_EMAIL`: Must be the verified email of the user (Alice)
- `GOOGLE_AI_API_KEY`: Get from https://aistudio.google.com/app/apikey
- `STOCK_MCP_SERVER_URL`: Must point to running MCP server (default: `http://localhost:8200/mcp`)
- Replace `YOUR_ORG` with your Asgardeo organization name

## WSO2 Asgardeo Setup (Complete Guide)

This section provides step-by-step instructions to configure all required components in Asgardeo.

### Step 1: Create the MCP Client Application

This application represents the MCP Stock Trading Server that will validate JWT tokens.

1. Log in to **Asgardeo Console** at https://console.asgardeo.io
2. Navigate to **Applications** → **New Application**
3. Select **Standard-Based Application**
4. Configure basic settings:
   - **Name**: `Stock MCP Server`
   - **Protocol**: OAuth2/OpenID Connect
5. Click **Register**

#### Configure Protocol Settings

1. Go to the **Protocol** tab
2. **Allowed Grant Types**:
   - Uncheck all grant types (MCP server only validates tokens, doesn't issue them)
3. **Public Client**:
   - **DISABLE** this option (very important - this is a confidential client)
4. **Access Token Configuration**:
   - Token type: JWT
   - User access token expiry: 3600 seconds
   - Application access token expiry: 3600 seconds
5. **Authorized Redirect URLs**:
   - Add: `http://localhost:8200/callback`
   - (Note: This is required even though MCP doesn't use redirect flow)
6. Click **Update**

#### Note Client Credentials

1. In the **General** tab, note down:
   - **Client ID**: You'll need this for both MCP server and agent app
   - **Client Secret**: Click "Show" and copy it
2. Save these values for later use

### Step 2: Create Roles for Stock Trading

Roles will define who can perform which actions on the stock trading system.

1. Navigate to **User Management** → **Roles**
2. Click **New Role**
3. Create role: `stock_trader`
   - **Role Name**: `stock_trader`
   - **Description**: Users who can read market data and execute trades
4. Click **Next** and then **Finish**

5. Create role: `stock_viewer`
   - **Role Name**: `stock_viewer`
   - **Description**: Users who can only view market data
6. Click **Next** and then **Finish**

### Step 3: Create Scopes for the MCP Server

Scopes define granular permissions for accessing stock trading tools.

1. Navigate to **API Resources** → **Create API Resource**
2. **Name**: `Stock Trading API`
3. **Identifier**: `stock-api`
4. **Description**: API for stock market data and trading operations

5. Add the following **Scopes**:
   - **Scope 1**:
     - Display Name: `Read Stock Data`
     - Scope Name: `stock:read`
     - Description: Permission to read market prices and portfolio
   - **Scope 2**:
     - Display Name: `Trade Stocks`
     - Scope Name: `stock:trade`
     - Description: Permission to buy and sell stocks
   - **Scope 3**:
     - Display Name: `Admin Access`
     - Scope Name: `stock:admin`
     - Description: Permission to update market prices (simulation only)

6. Click **Create**

### Step 4: Authorize API Resource to MCP Application

Link the scopes to the MCP Server application so tokens can include these scopes.

1. Go to **Applications** → **Stock MCP Server**
2. Navigate to **API Authorization** tab
3. Click **Authorize an API Resource**
4. Select **Stock Trading API**
5. **Authorized Scopes**: Select all:
   - `stock:read`
   - `stock:trade`
   - `stock:admin`
   - `openid` (should be pre-selected)
6. Click **Authorize**

### Step 5: Create the Agent Application

This application represents the Aurelius AI Agent that will request tokens.

1. Navigate to **Applications** → **New Application**
2. Select **Standard-Based Application**
3. Configure basic settings:
   - **Name**: `Aurelius Agent`
   - **Protocol**: OAuth2/OpenID Connect
4. Click **Register**

#### Configure Agent Application Protocol

1. Go to the **Protocol** tab
2. **Allowed Grant Types**: Enable the following:
   - **Client Credentials** (for agent's own token)
   - **CIBA (Client Initiated Backchannel Authentication)** (for on-behalf-of tokens)
3. **Public Client**:
   - **DISABLE** this option
4. **Application Native Authentication**:
   - **ENABLE** this option (allows agent identity to authenticate)
5. **CIBA Configuration**:
   - **Notification Channel**: Email (or SMS if configured)
   - **Token Delivery Method**: Poll
   - **Binding Message**: Enabled
6. **Authorized Redirect URLs**:
   - Add: `http://localhost:5001/callback`
7. Click **Update**

#### Authorize API Resources for Agent

1. Still in **Aurelius Agent** application
2. Navigate to **API Authorization** tab
3. Click **Authorize an API Resource**
4. Select **Stock Trading API**
5. **Authorized Scopes**: Select:
   - `stock:read`
   - `stock:trade`
   - `openid`
6. Click **Authorize**

#### Note Agent Application Credentials

1. In the **General** tab, note:
   - **Client ID**: This is your `CLIENT_ID` for .env
   - **Client Secret**: This is your `CLIENT_SECRET` for .env

### Step 6: Create an Agent Identity

Agent identities are service accounts that can authenticate using app-native authentication.

1. Navigate to **User Management** → **Agents** (or **Applications** → **Aurelius Agent** → **Agents** tab)
2. Click **New Agent**
3. Configure:
   - **Agent Name**: `aurelius-trading-agent`
   - **Description**: AI agent for autonomous stock portfolio management
4. Click **Create**
5. **Important**: Copy the **Agent ID** and **Agent Secret** immediately (they won't be shown again)
   - **Agent ID**: This is your `AGENT_ID` for .env
   - **Agent Secret**: This is your `AGENT_SECRET` for .env

### Step 7: Assign Roles to Agent

Grant the agent identity appropriate permissions.

1. Go to **User Management** → **Agents**
2. Find and click on `aurelius-trading-agent`
3. Navigate to **Roles** tab
4. Click **Assign Roles**
5. Select:
   - `stock_trader` (so agent can execute trades when authorized)
6. Click **Assign**

### Step 8: Create a User (Alice - The Investor)

Create the end-user who will receive CIBA approval requests.

1. Navigate to **User Management** → **Users**
2. Click **Add User**
3. Configure user details:
   - **Username**: `alice` (or any username you prefer)
   - **Email**: Use your real email address (you'll receive approval notifications here)
   - **First Name**: Alice
   - **Last Name**: Investor
   - **Password**: Set a password
4. **Email Verification**: Make sure to verify the email address
   - Check your inbox for verification email
   - Click the verification link
5. Click **Finish**

### Step 9: Assign Roles to User

Grant the user appropriate trading permissions.

1. Go to **User Management** → **Users**
2. Find and click on the user you just created (Alice)
3. Navigate to **Roles** tab
4. Click **Assign Roles**
5. Select:
   - `stock_trader` (allows user to approve trading operations)
6. Click **Assign**

### Step 10: Configure CIBA Notification Settings

Ensure email notifications are properly configured.

1. Navigate to **Settings** → **Account Settings** → **Email Provider**
2. If using default email provider, it should already be configured
3. For production, configure SMTP settings:
   - **SMTP Host**: Your SMTP server
   - **SMTP Port**: Usually 587 or 465
   - **Username**: SMTP username
   - **Password**: SMTP password
   - **From Address**: The email address notifications will come from
4. Click **Update**

### Step 11: Gather Configuration Values

Before moving to installation, collect all these values:

From **Stock MCP Server** application:
- `CLIENT_ID` (for MCP server .env)
- `CLIENT_SECRET` (for MCP server .env)

From **Aurelius Agent** application:
- `CLIENT_ID` (for main .env - different from MCP server)
- `CLIENT_SECRET` (for main .env)

From **Agent Identity**:
- `AGENT_ID`
- `AGENT_SECRET`

From **User**:
- `INVESTOR_EMAIL` (the email address you used for Alice)

From **Asgardeo Tenant**:
- Organization name (from URL: `https://console.asgardeo.io/t/YOUR_ORG/`)
- Construct these URLs:
  - `ASGARDEO_BASE_URL`: `https://api.asgardeo.io/t/YOUR_ORG`
  - `AUTH_ISSUER`: `https://api.asgardeo.io/t/YOUR_ORG/oauth2/token`
  - `JWKS_URL`: `https://api.asgardeo.io/t/YOUR_ORG/oauth2/jwks`

## Running the Demo

### Prerequisites Check

Before running, ensure:
1. MCP Stock Server is running on port 8200 (from Step 2)
2. You have configured both `.env` files correctly
3. Your Asgardeo user email is verified

### 1. Start the Main Application

In your sleeping-guardian terminal (with virtual environment activated):

```bash
python app.py
```

You should see:

```
====================================================================================================
🏦 AURELIUS - The Sleeping Guardian (LLM-Powered with MCP)
====================================================================================================
Investor: alice@example.com
Stock Symbol: NVDA
Initial Price: $150.00
Price Threshold: $140.00
Trade Amount: $5000.00
CIBA Channel: email
LLM Model: gemini-2.5-flash
MCP Server: http://localhost:8200/mcp
====================================================================================================

🌐 Dashboard: http://localhost:5001
⚠️  NOTE: Make sure MCP Stock Server is running on port 8200!
   Start it with: cd mcp-stock-server && python main.py
```

If you see an error about missing MCP server, go back and ensure Step 2 is completed.

### 2. Open the Dashboard

Navigate to `http://localhost:5001` in your browser. You'll see:

- **Market Status**: Current stock price and market conditions
- **Portfolio**: Your current balance and holdings
- **Agent Activity**: Real-time log of Aurelius's decisions
- **CIBA Status**: Current authorization state

### 3. Verify Agent Initialization

In the terminal, you should see:

```
[CIBA Client] ✓ Agent token obtained successfully
[Aurelius] Agent initialized with stock:read scope
[Aurelius] Monitoring NVDA for prices below $140.00
```

This confirms:
- The agent successfully authenticated with Asgardeo
- The agent received a token with `stock:read` scope
- The agent can read market data but cannot trade yet

## Conducting the Demo

Follow these steps for a compelling demonstration of CIBA with On-Behalf-Of flow:

### Step 1: Show the Normal State

- Point out the market price hovering around $150
- Explain that Aurelius has authenticated using **App-Native Authentication** with agent credentials
- The agent currently has a token with **`stock:read`** scope only
- Agent can monitor prices but **cannot execute trades** without user authorization

### Step 2: Trigger the Market Crash

- In the dashboard, click the **"Trigger Market Crash"** button
- Watch the price start dropping: $148... $145... $142...
- The agent is continuously monitoring via the MCP server's `get_market_price` tool

### Step 3: The AI Detects Opportunity

When the price drops below $140 (the threshold), the LLM-powered agent will:

1. Detect the golden buy opportunity
2. Attempt to execute a trade using the MCP server's `buy_stock` tool
3. Receive an "insufficient_scope" error because it only has `stock:read`

The console will show:

```
[Aurelius] 🚨 GOLDEN BUY OPPORTUNITY DETECTED!
[Aurelius] Current price: $139.45 (below threshold $140.00)
[Aurelius] Attempting to execute trade...
[MCP Server] ❌ SCOPE CHECK FAILED
  Required: ['stock:trade', 'stock:read']
  Available: ['stock:read', 'openid']
  Missing: ['stock:trade']
  → CIBA authorization needed!
```

### Step 4: CIBA Request Initiated

The agent initiates a CIBA (Client Initiated Backchannel Authentication) request:

```
================================================================================
[CIBA] Initiating authorization request
[CIBA] User: alice@example.com
[CIBA] Channel: email
[CIBA] 🔍 REQUESTED SCOPES: ['openid', 'stock:read', 'stock:trade']
================================================================================

[15:30:45] 📩 Authorization request sent via email
[15:30:45] ⏳ Waiting up to 300s for user approval...
[15:30:45] Request ID: abc-123-xyz
```

### Step 5: Check Your Email

**This is the critical part - real CIBA notification:**

1. **Check your email inbox** (the email you configured as `INVESTOR_EMAIL`)
2. You'll receive an email from Asgardeo with subject like:
   - "Authentication Request" or "Approval Required"
3. The email will contain:
   - A description of what's being requested
   - The binding message: "AI agent requests permission to buy X shares of NVDA at $139.45 for $5000.00"
   - An **Approve** button/link
   - A **Deny** button/link

### Step 6: Approve the Trade

1. Click the **Approve** link in the email
2. You'll be redirected to an Asgardeo authentication page
3. If not already logged in, enter Alice's credentials
4. Review the consent screen showing:
   - Application: Aurelius Agent
   - Requested scopes: `stock:read`, `stock:trade`
   - Binding message with trade details
5. Click **Allow** or **Approve**

### Step 7: Watch the Authorization Complete

Back in the terminal, you'll see:

```
[15:30:52] [DEBUG] CIBA polling completed!
[15:30:52] ✓ Authorization approved!
[15:30:52] ✓ On-behalf-of token obtained

================================================================================
[CIBA] 🔍 SCOPE VALIDATION
[CIBA] REQUESTED: ['openid', 'stock:read', 'stock:trade']
[CIBA] RECEIVED : ['openid', 'stock:read', 'stock:trade']
[CIBA] TOKEN SUB: <alice-user-id>
[CIBA] TOKEN AUT: CIBA
[CIBA] ✅ All requested scopes granted
================================================================================
```

Key points to highlight:
- The agent now has an **On-Behalf-Of (OBO) token**
- The token has the user's identity (`sub` claim = Alice)
- The token includes the elevated `stock:trade` scope
- The authentication method (`aut` claim) is "CIBA"

### Step 8: Trade Execution via MCP

The agent retries the trade with the new OBO token:

```
[Aurelius] Executing trade with OBO token...
[MCP Server] ✅ SCOPE CHECK PASSED
[BUY_STOCK] ✅ Scope check passed for user <alice-user-id>
[TRADE] User <alice-user-id> bought 35 NVDA @ $139.45
```

### Step 9: View the Results

- Switch back to the **Dashboard** (`http://localhost:5001`)
- You'll see:
  - **Balance**: Decreased by ~$5,000
  - **Holdings**: Shows 35 shares of NVDA
  - **Trade History**: New entry showing the purchase
  - **Agent Activity**: Complete log of the entire flow

### Step 10 (Optional): Verify in MCP Server Logs

Check the MCP server terminal to see the complete flow:

```
[JWT VALIDATION SUCCESS]
  Subject (sub): <alice-user-id>
  Auth Method (aut): CIBA
  🔍 Token Scopes: ['openid', 'stock:read', 'stock:trade']

[SCOPE CHECK] Required: ['stock:trade', 'stock:read']
[SCOPE CHECK] Available: ['openid', 'stock:read', 'stock:trade']
[SCOPE CHECK] ✅ All required scopes present
```

## Key Demo Points to Highlight

### Security Architecture

1. **No User Credentials Stored**: Agent never has access to Alice's password
2. **App-Native Authentication**: Agent authenticates using its own identity (agent credentials)
3. **Principle of Least Privilege**: Agent starts with minimal `stock:read` scope
4. **Just-In-Time Authorization**: Elevated permissions (`stock:trade`) granted only when needed
5. **Scope-Based Access Control**: MCP server enforces granular permissions per tool
6. **JWT Validation**: MCP server validates all tokens using JWKS from Asgardeo
7. **Audit Trail**: Complete log of authorization requests and token claims

### CIBA (Client Initiated Backchannel Authentication) Benefits

1. **Asynchronous Flow**: Alice doesn't need to be at her computer - can approve from anywhere
2. **User-Friendly**: Approval via familiar channels (Email/SMS) instead of complex flows
3. **Secure Backchannel**: Tokens delivered via backchannel, not through browser redirects
4. **Context-Rich Approval**: Binding message shows exactly what action requires approval
5. **Timeout Protection**: Requests automatically expire if not approved within time limit
6. **Mobile-First**: Perfect for scenarios where user is on-the-go

### On-Behalf-Of (OBO) Flow

1. **User Identity Preserved**: OBO token contains user's identity (`sub` claim)
2. **Elevated Privileges**: Token includes scopes that agent alone doesn't have
3. **Audit Attribution**: All trades are attributed to the user, not the agent
4. **Time-Limited**: OBO tokens expire, forcing re-authorization for new operations
5. **Delegated Authorization**: Agent acts on behalf of user, not as the user

### MCP (Model Context Protocol) Integration

1. **Standardized Interface**: Tools exposed via MCP standard for LLM integration
2. **Fine-Grained Authorization**: Each tool can require different scopes
3. **Scope Enforcement**: Server validates scopes before executing any tool
4. **Token-Based Security**: All requests require valid JWT tokens
5. **User-Specific Data**: Portfolio and trades isolated per user (based on `sub` claim)

### AI Agent Capabilities

1. **LLM-Powered Decisions**: Uses Google Gemini for intelligent trading logic
2. **24/7 Monitoring**: Continuously watches market without human intervention
3. **Autonomous Decision Making**: Can decide when to request authorization
4. **Fast Reaction**: Can execute trades within seconds once approved
5. **Human Oversight**: Cannot perform sensitive operations without explicit user approval
6. **Transparent Operations**: All decisions and actions logged for review

## MCP Tools Available

The MCP Stock Server exposes the following tools to authorized agents:

### Read-Only Tools (require `stock:read` scope)

1. **`get_market_price(symbol: str)`**
   - Get current market price for a stock symbol
   - Returns: symbol, price, name, change

2. **`list_available_stocks()`**
   - List all available stocks for trading
   - Returns: Array of stock data

3. **`get_my_portfolio()`**
   - Get current user's portfolio (user identified by `sub` claim in token)
   - Returns: balance, holdings, total_value, created_at

4. **`get_trade_history()`**
   - Get current user's trade history
   - Returns: Array of past trades

### Trading Tools (require `stock:trade` + `stock:read` scopes)

5. **`buy_stock(symbol: str, shares: int)`**
   - Buy shares of a stock
   - Validates sufficient balance
   - Returns: Trade confirmation

6. **`sell_stock(symbol: str, shares: int)`**
   - Sell shares of a stock
   - Validates sufficient holdings
   - Returns: Trade confirmation

### Admin Tools (require `stock:admin` scope)

7. **`update_market_price(symbol: str, new_price: float)`**
   - Update market price for simulation
   - Used by market engine
   - Returns: Updated market data

## Dashboard API Endpoints

The Flask application provides these HTTP endpoints:

### Web Pages
- `GET /` - Main dashboard with market, portfolio, and activity log

### API Endpoints
- `GET /api/market` - Current market data (JSON)
- `GET /api/portfolio` - Portfolio status (JSON)
- `GET /api/status` - Complete system status including CIBA state (JSON)

### Control Endpoints
- `GET /crash` - Trigger market crash event
- `GET /stabilize` - Stabilize market to normal state
- `GET /reset` - Reset demo to initial state (balance, portfolio, market)

## Environment Variables Reference

### MCP Stock Server `.env` (in `mcp-stock-server/` directory)

| Variable | Description | Example |
|----------|-------------|---------|
| `AUTH_ISSUER` | Asgardeo token issuer URL | `https://api.asgardeo.io/t/YOUR_ORG/oauth2/token` |
| `CLIENT_ID` | Stock MCP Server application Client ID | `abc123...` |
| `JWKS_URL` | Asgardeo JWKS endpoint for JWT validation | `https://api.asgardeo.io/t/YOUR_ORG/oauth2/jwks` |
| `MCP_SERVER_PORT` | Port for MCP server | `8200` |

### Main Application `.env` (in root `sleeping-guardian/` directory)

#### Required Authentication Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `ASGARDEO_BASE_URL` | Asgardeo base URL for your organization | `https://api.asgardeo.io/t/YOUR_ORG` |
| `CLIENT_ID` | Aurelius Agent application Client ID | `xyz789...` |
| `CLIENT_SECRET` | Aurelius Agent application Client Secret | `secret123...` |
| `AGENT_ID` | Agent identity ID | `agent-uuid-here` |
| `AGENT_SECRET` | Agent identity secret | `agent-secret-here` |
| `REDIRECT_URI` | OAuth redirect URI (required by SDK) | `http://localhost:5001/callback` |

#### Required User Configuration

| Variable | Description | Example |
|----------|-------------|---------|
| `INVESTOR_EMAIL` | User's verified email address | `alice@example.com` |
| `CIBA_NOTIFICATION_CHANNEL` | CIBA notification method | `email` or `sms` |

#### Required LLM Configuration

| Variable | Description | Example |
|----------|-------------|---------|
| `GOOGLE_AI_API_KEY` | Google AI API key for Gemini | `AIza...` |
| `MODEL_NAME` | Gemini model to use | `gemini-2.5-flash` |

#### Required MCP Configuration

| Variable | Description | Example |
|----------|-------------|---------|
| `STOCK_MCP_SERVER_URL` | URL of running MCP Stock Server | `http://localhost:8200/mcp` |

#### Optional Trading Parameters

| Variable | Description | Default |
|----------|-------------|---------|
| `STOCK_SYMBOL` | Stock symbol to trade | `NVDA` |
| `INITIAL_STOCK_PRICE` | Starting stock price | `150.00` |
| `PRICE_THRESHOLD` | Price below which to trigger buy | `140.00` |
| `TRADE_AMOUNT` | Amount to invest per trade | `5000.00` |
| `INITIAL_BALANCE` | Starting portfolio balance | `10000.00` |
| `PORT` | Flask app port | `5001` |
| `FLASK_DEBUG` | Enable Flask debug mode | `False` |

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

### MCP Server Connection Issues

**Error**: `Connection refused to http://localhost:8200`

**Solutions**:
- Ensure MCP Stock Server is running: `cd mcp-stock-server && python main.py`
- Verify port 8200 is not in use by another application
- Check `STOCK_MCP_SERVER_URL` in main `.env` file matches the MCP server port

### MCP Server Token Validation Fails

**Error**: `Token validation failed` or `401 Unauthorized`

**Solutions**:
- Verify `CLIENT_ID` in `mcp-stock-server/.env` matches the **Stock MCP Server** application (not the Agent app)
- Check that `AUTH_ISSUER` and `JWKS_URL` are correctly formatted:
  - `AUTH_ISSUER` should end with `/oauth2/token`
  - `JWKS_URL` should end with `/oauth2/jwks`
- Ensure organization name in URLs matches your actual Asgardeo organization

### Agent Authentication Fails

**Error**: `Failed to get agent token`

**Solutions**:
- Verify `AGENT_ID` and `AGENT_SECRET` in main `.env` are correct
- Ensure the agent identity exists in Asgardeo under **User Management** → **Agents**
- Check that **App Native Authentication** is enabled in the **Aurelius Agent** application
- Verify agent has `stock_trader` role assigned

### CIBA Request Fails

**Error**: `Authorization failed` or `insufficient_scope`

**Solutions**:
- Verify `CLIENT_ID` and `CLIENT_SECRET` in main `.env` are from **Aurelius Agent** application
- Ensure **CIBA grant type** is enabled in the Aurelius Agent application Protocol settings
- Verify the agent has API authorization for scopes: `stock:read`, `stock:trade`, `openid`
- Check that user (Alice) exists and has `stock_trader` role
- Ensure `INVESTOR_EMAIL` matches the user's verified email in Asgardeo

### No CIBA Notification Received

**Solutions**:
- **Check spam/junk folder**: Asgardeo emails may be filtered
- **Verify email**: Ensure the user's email is verified in Asgardeo (check user profile)
- **Check notification channel**: Verify `CIBA_NOTIFICATION_CHANNEL=email` in .env
- **Email provider**: Ensure Asgardeo email provider is configured (Settings → Account Settings → Email Provider)
- **Test email**: Try sending a password reset email to the user to verify email delivery works

### Scope Validation Fails

**Error**: `Missing scopes: ['stock:trade']`

**Solutions**:
1. Verify API Resource authorization:
   - Go to **Aurelius Agent** app → **API Authorization**
   - Ensure **Stock Trading API** is authorized with `stock:read`, `stock:trade`, `openid`
2. Check that **Stock Trading API** resource has the scopes defined
3. Verify agent is requesting scopes correctly in the CIBA request
4. Check token scopes in console logs - compare REQUESTED vs RECEIVED scopes

### Agent Cannot Execute Trades

**Error**: `PermissionError: insufficient_scope`

**Solutions**:
- This is expected behavior! Agent should only get `stock:trade` after CIBA approval
- If error persists after approval:
  - Check that CIBA approval was successful (look for "✅ All requested scopes granted" in logs)
  - Verify OBO token was obtained
  - Check MCP server logs for scope validation details
  - Ensure user (Alice) has `stock_trader` role assigned

### Google AI API Issues

**Error**: `Google AI API key not found` or `Invalid API key`

**Solutions**:
- Get API key from https://aistudio.google.com/app/apikey
- Verify `GOOGLE_AI_API_KEY` is set in main `.env` file (not in MCP server .env)
- Check for typos or extra spaces in the API key
- Ensure API key has access to Gemini models

### Port Already in Use

**Error**: `Port 5001 already in use` or `Port 8200 already in use`

**Solutions**:
- Find and kill the process using the port:
  ```bash
  # On macOS/Linux
  lsof -i :5001  # or :8200
  kill <PID>

  # On Windows
  netstat -ano | findstr :5001
  taskkill /PID <PID> /F
  ```
- Or change the port in `.env`:
  - Main app: Change `PORT=5001` to another port
  - MCP server: Change `MCP_SERVER_PORT=8200` in mcp-stock-server/.env

### Public Client Error

**Error**: `Public client cannot use client_secret` or `Invalid client`

**Solutions**:
- Ensure **Public Client** is DISABLED in both applications:
  - Stock MCP Server application → Protocol tab → Public Client = OFF
  - Aurelius Agent application → Protocol tab → Public Client = OFF
- Both applications must be confidential clients

### Role Assignment Issues

**Solutions**:
- Verify user has `stock_trader` role:
  - Go to **User Management** → **Users** → Select user → **Roles** tab
- Verify agent has `stock_trader` role:
  - Go to **User Management** → **Agents** → Select agent → **Roles** tab
- If roles are missing, assign them following Steps 7 and 9 of the Asgardeo Setup

## Architecture Details

### Market Engine (`market_engine.py`)

- Simulates realistic price movements
- Supports different market states: STABLE, VOLATILE, CRASHING, RECOVERING
- Thread-safe price updates
- Historical price tracking
- Pushes price updates to MCP server via unauthenticated endpoint

### Aurelius Agent (`aurelius_agent_llm.py`)

- LLM-powered decision making using Google Gemini
- Asynchronous monitoring loop
- Connects to MCP Stock Server as an MCP client
- Uses agent token (with `stock:read`) for market monitoring
- Detects price drops below threshold using LLM analysis
- Initiates CIBA requests when elevated permissions needed
- Receives On-Behalf-Of tokens with `stock:trade` scope
- Executes approved trades via MCP tools
- Maintains state across monitoring cycles

### CIBA Client (`ciba_client.py`)

- Manages Asgardeo authentication
- Implements App-Native Authentication for agent identity
- Handles complete CIBA flow: initiate → poll → token
- Provides On-Behalf-Of (OBO) tokens with user identity
- Validates scopes in received tokens
- Error handling and retries for network issues
- Token expiration management

### MCP Stock Server (`mcp-stock-server/main.py`)

- FastMCP-based Model Context Protocol server
- JWT token validation using Asgardeo JWKS
- Scope-based authorization per tool
- User-specific data isolation (keyed by `sub` claim)
- Exposes trading tools to authorized agents
- In-memory portfolio and trade history management
- Real-time market data synchronization

## Understanding the Complete Flow

This section explains the end-to-end authentication and authorization flow:

### 1. Initial Agent Authentication (App-Native Authentication)

```
Aurelius Agent
    │
    ├─ Uses AGENT_ID + AGENT_SECRET
    │
    └─▶ Asgardeo /token endpoint
        │ Grant Type: client_credentials
        │ Scopes: [openid, stock:read]
        │
        └─▶ Returns: Agent Access Token (JWT)
            - sub: agent-app-id (application subject)
            - scope: "openid stock:read"
            - aud: <client-id>
```

**Key Points**:
- Agent authenticates as itself, not as a user
- Gets minimal scopes for monitoring only
- Cannot perform sensitive operations like trading

### 2. Monitoring Phase (Using Agent Token)

```
Aurelius Agent
    │ Has: Agent Token [stock:read]
    │
    └─▶ MCP Stock Server
        │ Tool: get_market_price("NVDA")
        │ Authorization: Bearer <agent-token>
        │
        ├─ MCP validates JWT signature using JWKS
        ├─ Extracts scopes: [stock:read]
        ├─ Checks: stock:read scope present? ✅
        │
        └─▶ Returns: { price: 139.45, symbol: "NVDA" }
```

**Key Points**:
- Agent can read market data continuously
- MCP server validates every request
- No user interaction needed for read operations

### 3. Trade Detection (Insufficient Scope)

```
Aurelius Agent (LLM Decision)
    │ Price: $139.45 < $140.00 threshold
    │ Decision: BUY 35 shares
    │
    └─▶ MCP Stock Server
        │ Tool: buy_stock("NVDA", 35)
        │ Authorization: Bearer <agent-token>
        │
        ├─ MCP validates token
        ├─ Extracts scopes: [stock:read, openid]
        ├─ Required scopes: [stock:trade, stock:read]
        ├─ Missing: [stock:trade]
        │
        └─▶ ❌ Error 403: insufficient_scope
            "Missing ['stock:trade']. Request authorization via CIBA."
```

**Key Points**:
- Agent detects opportunity but lacks permission
- MCP server enforces scope requirements
- Error message guides next action

### 4. CIBA Authorization Request

```
Aurelius Agent
    │
    └─▶ Asgardeo /bc-authorize endpoint
        │ login_hint: "alice@example.com"
        │ scope: "openid stock:read stock:trade"
        │ binding_message: "Buy 35 NVDA @ $139.45..."
        │ client_assertion: <agent-token>
        │
        └─▶ Asgardeo creates auth_req_id
            │
            └─▶ Sends Email to alice@example.com
                Subject: "Authentication Request"
                Body: Binding message + Approve/Deny links
```

**Key Points**:
- Agent initiates backchannel request
- User is notified asynchronously
- Binding message provides context

### 5. User Approval (CIBA Polling)

```
Alice receives email → Clicks Approve link
    │
    └─▶ Asgardeo Consent Page
        │ Shows: App, Scopes, Binding Message
        │ Alice clicks: "Allow"
        │
        └─▶ Asgardeo marks auth_req_id as approved

Meanwhile, Agent polls:
    │
    └─▶ Asgardeo /token endpoint (polling)
        │ auth_req_id: <request-id>
        │ grant_type: urn:openid:params:grant-type:ciba
        │
        └─▶ Returns: On-Behalf-Of Token (OBO Token)
            - sub: <alice-user-id> (user's subject!)
            - scope: "openid stock:read stock:trade"
            - aut: CIBA (authentication method)
            - act: { sub: <agent-app-id> } (actor claim)
            - aud: <client-id>
```

**Key Points**:
- Token contains USER identity, not agent identity
- All requested scopes granted
- Special claims indicate OBO flow (aut, act)

### 6. Trade Execution (With OBO Token)

```
Aurelius Agent
    │ Has: OBO Token [stock:trade, stock:read]
    │ sub: alice-user-id
    │
    └─▶ MCP Stock Server
        │ Tool: buy_stock("NVDA", 35)
        │ Authorization: Bearer <obo-token>
        │
        ├─ MCP validates JWT signature
        ├─ Extracts scopes: [stock:trade, stock:read, openid]
        ├─ Extracts user: alice-user-id (from sub claim)
        ├─ Checks: stock:trade scope present? ✅
        │
        ├─ Gets Alice's portfolio (keyed by alice-user-id)
        ├─ Validates: balance >= cost? ✅
        ├─ Executes trade
        ├─ Updates portfolio
        │
        └─▶ Returns: { status: "success", shares: 35, ... }
```

**Key Points**:
- Trade is attributed to Alice (from `sub` claim)
- MCP server uses user identity to isolate data
- Full audit trail: who (Alice) via what (Agent) did what (Buy NVDA)

### Flow Summary

```
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 1: Agent Auth (App-Native)                                │
│ Agent → Asgardeo → Agent Token [stock:read]                     │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 2: Monitoring (Agent Token)                               │
│ Agent → MCP Server → get_market_price() → Success               │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 3: Trade Attempt (Insufficient Scope)                     │
│ Agent → MCP Server → buy_stock() → 403 Forbidden                │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 4: CIBA Request                                           │
│ Agent → Asgardeo → /bc-authorize → Email to Alice               │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 5: User Approval                                          │
│ Alice → Email → Approve → Asgardeo → OBO Token [stock:trade]    │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 6: Trade Execution (OBO Token)                            │
│ Agent → MCP Server → buy_stock() → Success (as Alice)           │
└─────────────────────────────────────────────────────────────────┘
```

## Real-World Use Cases

This CIBA + OBO + MCP pattern is applicable to many scenarios:

### Financial Services
1. **Trading Bots**: Autonomous trading agents that require approval for high-value transactions
2. **Expense Management**: AI agents that process invoices pending manager approval
3. **Investment Advisors**: Robo-advisors that need client consent before portfolio rebalancing
4. **Fraud Prevention**: Systems that request user verification for suspicious transactions

### Healthcare
5. **Prescription Bots**: AI assistants that require doctor approval before prescribing medication
6. **Medical Records Access**: Agents accessing sensitive health records with patient consent
7. **Treatment Recommendations**: AI suggesting treatments that need physician authorization

### Enterprise
8. **Approval Workflows**: Automated systems requiring manager sign-off
9. **Resource Provisioning**: Infrastructure agents requesting approval for costly resources
10. **Data Access**: AI agents needing user approval to access sensitive data

### IoT and Smart Home
11. **Smart Thermostats**: Requesting approval before making expensive HVAC changes
12. **Security Systems**: Autonomous responses requiring homeowner confirmation
13. **Energy Management**: Agents optimizing consumption with owner oversight

### Autonomous Systems
14. **Delivery Robots**: Requesting human intervention for edge cases
15. **Autonomous Vehicles**: Seeking user input for unusual situations
16. **Drones**: Requiring approval for flight path changes

### Benefits for All Use Cases
- **Security**: User credentials never shared with agents
- **Auditability**: Complete trail of who authorized what
- **Flexibility**: User can approve from anywhere (mobile-friendly)
- **Granular Control**: Different permission levels for different actions
- **User Experience**: Async approval doesn't block agent operation

## Security Considerations

### Application Security Best Practices

1. **Never Hardcode Credentials**
   - Always use environment variables for sensitive data
   - Never commit `.env` files to version control
   - Use `.env.example` as templates only

2. **Scope Minimization**
   - Agent requests only minimum required scopes
   - Start with `stock:read`, request `stock:trade` only when needed
   - Never request `stock:admin` for production agents

3. **Token Management**
   - Tokens have limited lifetime (default: 3600 seconds)
   - OBO tokens expire and require re-authorization
   - Agent tokens should be refreshed before expiry

4. **Binding Messages**
   - Always include context in CIBA requests
   - Show exact action being requested
   - Include transaction details (amount, symbol, etc.)

5. **Audit Logging**
   - Track all authorization requests and outcomes
   - Log token claims for attribution
   - Monitor for unusual patterns

### Asgardeo Configuration Security

1. **Public Client Settings**
   - Always DISABLE "Public Client" for both applications
   - Use confidential clients with client secrets
   - Protect client secrets as you would passwords

2. **Grant Types**
   - Only enable required grant types
   - MCP Server: No grant types needed (validation only)
   - Agent: Only Client Credentials + CIBA

3. **Redirect URIs**
   - Use exact match URLs
   - Use HTTPS in production (HTTP only for local dev)
   - Avoid wildcard redirects

4. **CIBA Settings**
   - Set reasonable token expiry times
   - Use polling mode for token delivery
   - Configure timeouts for authorization requests

5. **Role-Based Access Control**
   - Assign minimal roles to users and agents
   - Use `stock_viewer` for read-only access
   - Use `stock_trader` only for approved traders
   - Never assign admin roles to regular users

### Production Deployment Considerations

1. **MCP Server Security**
   - Deploy MCP server with HTTPS
   - Use production-grade JWKS caching
   - Implement rate limiting
   - Add request logging and monitoring

2. **Network Security**
   - Use firewalls to restrict MCP server access
   - VPN or private network for agent-to-MCP communication
   - TLS for all external communications

3. **Email Security**
   - Use authenticated SMTP for CIBA notifications
   - Configure SPF/DKIM for email authenticity
   - Use branded emails to prevent phishing

4. **Monitoring and Alerts**
   - Monitor failed authorization attempts
   - Alert on unusual trading patterns
   - Track token validation failures
   - Log all scope escalation requests

5. **Data Protection**
   - Encrypt sensitive data at rest
   - Use secure random generators for IDs
   - Implement data retention policies
   - Regular security audits

### OWASP Top 10 Considerations

- **Injection**: MCP server validates all input parameters
- **Broken Authentication**: Use Asgardeo for centralized auth
- **Sensitive Data Exposure**: Never log tokens or secrets
- **XML External Entities**: Not applicable (using JSON)
- **Broken Access Control**: Scope-based authorization enforced
- **Security Misconfiguration**: Follow checklist above
- **XSS**: Sanitize all user inputs in dashboard
- **Insecure Deserialization**: Validate JWT signatures
- **Using Components with Known Vulnerabilities**: Keep dependencies updated
- **Insufficient Logging**: Comprehensive audit trail implemented

## Frequently Asked Questions (FAQ)

### General Questions

**Q: What is the difference between Agent Token and OBO Token?**

A:
- **Agent Token**: Issued to the agent identity using client credentials. Contains agent's identity, has limited scopes (`stock:read`).
- **OBO Token**: Issued after CIBA approval. Contains user's identity (`sub` = user ID), has elevated scopes (`stock:trade`), includes actor claim pointing to agent.

**Q: Why do I need two applications in Asgardeo?**

A:
- **Stock MCP Server**: Resource server that validates tokens. Doesn't issue tokens, only validates them.
- **Aurelius Agent**: Client application that requests and uses tokens. Needs Client Credentials + CIBA grant types.

**Q: Can I use a different LLM instead of Google Gemini?**

A: Yes! Modify `aurelius_agent_llm.py` to use any LLM that supports function calling (OpenAI, Claude, etc.). The MCP integration works with any LLM framework.

### Setup Questions

**Q: Do I need a paid Asgardeo account?**

A: No, the free tier supports this demo. You get sufficient API calls and features for development and testing.

**Q: Can I use WSO2 Identity Server instead of Asgardeo?**

A: Yes, the same setup works with on-premise WSO2 IS 7.0+. Update the URLs to point to your IS instance instead of `api.asgardeo.io`.

**Q: Why is my email not verified?**

A: Check your inbox (and spam folder) for the verification email from Asgardeo. Resend verification from user profile if needed.

**Q: Can I use SMS instead of email for CIBA?**

A: Yes, but you need to configure SMS provider in Asgardeo (Settings → Account Settings → SMS Provider). Then set `CIBA_NOTIFICATION_CHANNEL=sms`.

### Technical Questions

**Q: What happens if the user denies the CIBA request?**

A: The agent receives an `access_denied` error, logs it, and continues monitoring. No trade is executed.

**Q: How long do OBO tokens last?**

A: Default is 3600 seconds (1 hour). After expiry, a new CIBA request is needed for elevated operations.

**Q: Can multiple users use the same MCP server?**

A: Yes! The MCP server isolates data by user ID (from `sub` claim in token). Each user has their own portfolio.

**Q: Is the portfolio data persistent?**

A: No, it's stored in-memory for the demo. Restarting the MCP server resets all portfolios. For production, use a database.

**Q: Can I deploy this to production?**

A: This is a demo/proof-of-concept. For production:
- Use HTTPS for all services
- Add database persistence
- Implement rate limiting
- Add comprehensive error handling
- Use production LLM endpoints
- Set up monitoring and alerts
- Follow security best practices section

### CIBA Questions

**Q: What if the CIBA request times out?**

A: Default timeout is 300 seconds (5 minutes). After timeout, the request expires and the agent continues monitoring.

**Q: Can I approve from a different device?**

A: Yes! The email is sent to the user's address and can be approved from any device with email access.

**Q: Does CIBA work offline?**

A: No, CIBA requires internet connectivity to send notifications and poll for approval.

**Q: What is the "binding message" in CIBA?**

A: It's context provided to the user about what they're approving. Example: "Buy 35 shares of NVDA at $139.45 for $5000.00"

### MCP Questions

**Q: What is MCP (Model Context Protocol)?**

A: MCP is a standard protocol for connecting LLMs to external tools and data sources. It provides a secure, standardized way for agents to call functions.

**Q: Can I add more tools to the MCP server?**

A: Yes! Add new `@mcp.tool()` decorated functions in `main.py`. Define required scopes using `require_scopes()`.

**Q: Why does MCP server use JWT tokens?**

A: JWT tokens provide:
- Stateless authentication (no session storage)
- Scope-based authorization
- User identity propagation
- Signature verification via JWKS

**Q: Can I use the MCP server with other agents?**

A: Yes! Any agent that can obtain a valid JWT token from Asgardeo can use the MCP server.

### Troubleshooting Questions

**Q: I get "Connection refused" to port 8200**

A: Ensure MCP Stock Server is running. Start it with: `cd mcp-stock-server && python main.py`

**Q: Scopes are missing in the token**

A: Check:
1. API Resource has the scopes defined
2. Application has API Resource authorized with those scopes
3. CIBA request includes the scopes in the request

**Q: Agent can't authenticate**

A: Verify:
1. `AGENT_ID` and `AGENT_SECRET` are correct
2. Agent identity exists in Asgardeo
3. "App Native Authentication" is enabled in the Agent application

**Q: Where can I see detailed logs?**

A:
- Main app logs: Terminal where `python app.py` is running
- MCP server logs: Terminal where MCP server is running
- Asgardeo logs: Login to console → Logs → Audit Logs

## License

Copyright (c) 2025, WSO2 LLC. (http://www.wso2.com). All Rights Reserved.

This software is the property of WSO2 LLC. and its suppliers, if any. Dissemination of any information or reproduction of any material contained herein is strictly forbidden, unless permitted by WSO2 in accordance with the WSO2 Commercial License.

## Support

For issues or questions:
- WSO2 Documentation: https://wso2.com/asgardeo/docs/
- CIBA Specification: https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html

## Contributing

This is a demo application. For production use, additional security measures and error handling should be implemented.

