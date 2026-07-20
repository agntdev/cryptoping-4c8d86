# CryptoPing — Bot specification

**Archetype:** finance

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

CryptoPing is a Telegram bot that allows users to monitor cryptocurrency prices through customizable alerts and a daily digest. Users can set price-threshold and percentage-change alerts for specific coins, manage a private watchlist, and configure quiet hours and notification preferences. The bot owner can view aggregated statistics about user activity and alert triggers.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Retail crypto traders
- Crypto holders
- Non-technical users

## Success criteria

- Users can set and receive accurate price alerts
- Daily digest is delivered at user-specified time
- Quiet hours suppress notifications as configured
- Owner dashboard shows accurate user and alert statistics

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu and begin onboarding
- **/price** (command, actor: user, command: /price) — Check current price of a specific coin or all watchlist coins
  - inputs: ticker (optional)
  - outputs: Price information message
- **Watchlist** (button, actor: user, callback: watchlist:open) — Open the watchlist management menu
  - inputs: none
  - outputs: Watchlist menu with inline buttons
- **Set Alert** (button, actor: user, callback: alert:start) — Begin setting up a new alert
  - inputs: none
  - outputs: Alert setup wizard
- **Configure Notifications** (button, actor: user, callback: notifications:open) — Configure quiet hours and digest settings
  - inputs: none
  - outputs: Notification settings menu

## Flows

### Onboarding
_Trigger:_ /start

1. Welcome message
2. Ask for timezone selection
3. Confirm onboarding completion

_Data touched:_ User profile

### Watchlist Management
_Trigger:_ watchlist:open

1. Display watchlist items with inline controls
2. Add/remove coins via buttons or input
3. Update watchlist state

_Data touched:_ Watchlist item

### Price Alert Setup
_Trigger:_ alert:start

1. Select alert type (price threshold or percent change)
2. Configure alert parameters
3. Confirm and save alert

_Data touched:_ Watchlist item

### Price Check
_Trigger:_ /price

1. Parse ticker parameter
2. Fetch current price
3. Display price with timestamp

_Data touched:_ Price sample

### Daily Digest
_Trigger:_ digest:scheduled

1. Compile current prices for watchlist
2. Identify notable price changes
3. Send formatted digest message

_Data touched:_ Price sample, User profile

### Owner Dashboard
_Trigger:_ owner:dashboard

1. Verify owner identity
2. Fetch and display user count
3. Display top alerts by trigger count

_Data touched:_ User profile, Watchlist item

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User profile** _(retention: persistent)_ — Stores user preferences and settings
  - fields: Telegram ID, timezone, quiet hours start/end, digest time, notification preferences, cooldown settings
- **Watchlist item** _(retention: persistent)_ — Represents a cryptocurrency being monitored by a user
  - fields: ticker, display name, alert rules, last-notified timestamp, cooldown state
- **Price sample** _(retention: persistent)_ — Cached price data for alert evaluation
  - fields: ticker, timestamp, price, source status
- **Owner stats** _(retention: persistent)_ — Aggregated statistics for the bot owner
  - fields: user count, alert trigger counters by rule/ticker

## Integrations

- **Telegram** (required) — Bot API messaging and notifications
- **Crypto price API** (required) — Fetch current cryptocurrency prices
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- View owner dashboard with user count and top alerts
- Configure bot-wide settings (if needed in future)

## Notifications

- Price threshold alerts
- Percentage change alerts
- Daily digest
- Error notifications for price API failures

## Permissions & privacy

- All user data is private and not shared
- Watchlists and alerts are user-specific
- Owner can only view aggregated statistics, not individual user data

## Edge cases

- Handling unknown or misspelled tickers with suggestions
- Price API failures with retry logic
- Alerts during quiet hours are queued for later delivery
- Cooldown expiration after quiet hours

## Required tests

- Verify price alerts trigger at correct thresholds
- Test daily digest delivery at scheduled time
- Validate quiet hours suppression and post-quiet delivery
- Confirm owner dashboard shows accurate statistics

## Assumptions

- Price API is reliable with reasonable rate limits
- Users will primarily use USD for price thresholds
- Most users will use common tickers like BTC, ETH, TON
- Owner will handle future monetization decisions separately
