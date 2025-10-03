# SlipstreamAI Trading Engine

## Overview

SlipstreamAI is a self-optimizing, ROI-compounding cryptocurrency trading engine built with a deterministic, auditable architecture. The system runs continuous loops to find, vet, size, and manage crypto trades while maintaining strict safety constraints including a 20% cash vault floor. The platform features a biological "organ" architecture where each component serves a specific function, and includes an optional AI agent (STVN) for analysis and optimization suggestions.

The application is a full-stack TypeScript/Python hybrid with a React frontend, Express.js backend, and Python trading engine core. It integrates with Kraken exchange for crypto trading and uses PostgreSQL for persistent data storage.

## User Preferences

Preferred communication style: Simple, everyday language.

## Recent Updates (October 3, 2025)

### Real Kraken Account Integration
- **Balance Integration**: System now fetches and displays real account balances from Kraken API
- **Position Tracking**: Automatically fetches and syncs open positions from Kraken with unrealized P&L calculations
- **Database Persistence**: Real portfolio and position data persists across restarts (replaced mock data)
- **Live Trading Mode**: System operates with real funds ($57.63 current balance)

## System Architecture

### Frontend Architecture

**Framework & Build System**
- React 18 with TypeScript for type safety
- Vite as the build tool and development server
- TanStack Query (React Query) for server state management with disabled refetching (manual updates via WebSocket)
- Wouter for client-side routing (lightweight alternative to React Router)

**UI Component System**
- shadcn/ui components built on Radix UI primitives (accordion, dialog, dropdown, toast, etc.)
- Tailwind CSS for styling with custom design tokens
- Dark theme by default with CSS variables for theming
- "New York" style variant from shadcn/ui

**State Management**
- Server state via TanStack Query with custom `queryClient` configuration
- Real-time updates via WebSocket connection (`useWebSocket` hook)
- Local state using React hooks (useState, useEffect)
- Queries set to `staleTime: Infinity` and `refetchOnWindowFocus: false` for explicit control

**Dashboard Layout**
- Tab-based interface with four main panels: Cockpit, Research, History, STVN
- TopBar component showing portfolio summary, system metrics, and WebSocket connection status
- Real-time data updates without polling - driven by WebSocket events

### Backend Architecture

**Server Framework**
- Express.js with TypeScript running on Node.js
- HTTP server with WebSocket upgrade support on `/ws/trading` path
- Vite middleware for development with HMR
- Static file serving for production builds

**API Structure**
- RESTful endpoints under `/api` prefix for CRUD operations
- WebSocket server for real-time trading engine updates
- Routes handle portfolio, positions, signals, metrics, challengers, orders, and STVN data
- Storage abstraction layer (`IStorage` interface) for data access

**Python Trading Engine**
- Separate Python process spawned from Node.js server (`run.py`)
- Biological "organ" architecture with specialized modules:
  - **Lungs (🫁)**: Market data ingestion with freshness validation
  - **Liver (🫀)**: Portfolio state management and P&L tracking
  - **Retina (👁️)**: Trading universe filtering and symbol selection
  - **Neocortex (🧠)**: Signal generation with ROI-velocity scoring
  - **ImmuneShield (🛡️)**: Risk management and safety constraints
  - **ReflexArc (⚡)**: Signal filtering and reconciliation monitoring
  - **Vasculature (🩸)**: Capital allocation enforcing vault constraints
  - **MotorCortex (🤖)**: Order execution and TP/SL management
  - **ShadowLab (🧪)**: Strategy backtesting and challenger evaluation
  - **Pituitary (🧬)**: Strategy promotion and rollback system

**Process Communication**
- Python engine outputs JSON events to stdout
- Node.js server captures stdout and broadcasts via WebSocket
- Bidirectional communication for commands (kill switch, configuration updates)

### Data Storage Solutions

**Database: PostgreSQL with Drizzle ORM**
- Neon serverless PostgreSQL driver (`@neondatabase/serverless`)
- Drizzle ORM for type-safe database queries
- Drizzle-Zod integration for runtime schema validation
- Migrations stored in `./migrations` directory

**Database Schema**
- `users`: User authentication (id, username, password)
- `portfolios`: Portfolio state (equity, cash, deployed capital, unrealized/daily P&L, vault percent)
- `positions`: Open/closed positions with TP/SL, holding time, and P&L tracking
- `order_ledger`: Complete order history with fill prices, fees, and status
- `signals`: Trading signals with confidence, expected ROI, veto status
- `system_metrics`: Tick performance, veto rates, reconciliation diffs
- `challengers`: Strategy testing with fitness scores and capital allocation
- `stvn_logs`: AI agent observations, patches, and autonomy settings

**Session Management**
- PostgreSQL session store using `connect-pg-simple`
- Session data persisted to database for stateful user sessions

### Authentication & Authorization

**Session-Based Authentication**
- Express sessions with PostgreSQL backing
- User credentials stored with hashed passwords
- Session cookies for maintaining authenticated state
- Username/password authentication system

### External Dependencies

**Cryptocurrency Exchange**
- Kraken API integration (`kraken_client.py`)
- Public endpoints: Market data, ticker info, orderbook
- Private endpoints: Account balances, order placement, position management
- HMAC-SHA512 signature authentication
- Rate limiting per venue (300 requests/minute for Kraken)

**AI Services**
- OpenAI integration in STVN agent (`stvn_agent.py`) for:
  - System analysis and briefings
  - Anomaly detection
  - Code patch generation
  - Performance optimization suggestions

**Telegram Notifications** (Optional)
- Critical event alerting system (`telegram_notifier.py`)
- Alerts sent for:
  - Vault floor violations (cash drops below 20%)
  - Kill switch activation
  - Order execution errors
  - Daily loss limit breaches
  - Reconciliation failures
- Configuration via environment variables:
  - `TELEGRAM_BOT_TOKEN`: Bot API token from @BotFather
  - `TELEGRAM_CHAT_ID`: Target chat ID for notifications
- Smart alert throttling to prevent spam (one alert per event until resolved)

**WebSocket Communication**
- Native WebSocket server (ws library)
- Dual implementation: TypeScript (`websocket_manager.ts`) and Python (`websocket_manager.py`)
- Real-time broadcast of trading engine events to connected clients
- Dedicated path `/ws/trading` to avoid conflicts with Vite HMR

**Build & Development Tools**
- Vite plugins: runtime error modal, cartographer (Replit-specific), dev banner
- TypeScript compiler with strict mode enabled
- ESBuild for server bundling in production
- PostCSS with Tailwind CSS and Autoprefixer

**Key Configuration Constraints**
- Vault floor: Minimum 20% cash reserve (hard constraint)
- Risk limits: Daily loss limit (5%), drawdown limit (10%)
- Position limits: Max 8 positions, 1 per symbol, 15% max allocation per symbol
- Freshness: 5 second market data staleness limit
- Tick budget: 2 second execution time limit
- Cooldown: 30 minute cooldown between trades per symbol

**Determinism Features**
- Idempotent order execution (no duplicate orders)
- State hashing for reproducibility
- Append-only audit logs with rotation
- Same inputs produce same outputs
- Monotonic TP/SL adjustments