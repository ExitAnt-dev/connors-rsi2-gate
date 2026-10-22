# ExitAnt Connors RSI2 — Gate.io

Deploy this automated trading bot to **your own Render account** as a background worker. The bot runs from a prebuilt Docker image;
this repository contains **only the Render Blueprint (`render.yaml`)** — no source code.

The bot trades **Larry Connors RSI2** or **Turtle + Connors RSI2 merge** on Gate.io USDT perpetual futures with exactly the rules of the
TanQuant backtest engine, on the exchange's own daily candles (UTC day; stocks/ETFs: US trading days only) for every indicator. Yahoo daily candles are used only to complete
the 200-day moving average of symbols with fewer than 200 exchange candles (recently listed).
Copy a TanQuant result with **“Copy bot setup”** and paste it into the bot to set everything up at once.

## Deploy to Render

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/ExitAnt-dev/connors-rsi2-gate)

```
https://render.com/deploy?repo=https://github.com/ExitAnt-dev/connors-rsi2-gate
```

1. Click the button (or open the link) and sign in to Render.
2. Render reads `render.yaml` and proposes **one background worker** with a **1 GB persistent disk** mounted at `/data`
   (settings, books and positions survive restarts and redeploys).
3. Enter the environment variables listed below. They are marked `sync: false`, so Render asks you for them on the deploy screen
   and stores them **only in your own Render account**.
4. Click **Apply** and wait for the first deploy to finish.
5. Open Telegram and send **`/start`** to your bot to begin setup.

## Before you deploy

- **Gate.io account registered through ExitAnt.** At every start the bot checks your Gate.io UID with the ExitAnt server; only the UID is sent, never your keys.
- **API key** with futures trading permission. The key must be allowed to read the account and trade USDT perpetual futures.
  **Never enable the withdrawal permission** — the bot never withdraws, and a key without it cannot move funds out of your account even if it leaks.
  *For the Turtle merge's cash vault* the key needs the **Wallets** permission (futures ↔ spot transfer).
- **Dual (long/short) position mode** in Gate.io USDT futures. The bot switches the account to it when no position or order is open; otherwise set it yourself in Gate.io → Futures → settings.
- If the API key has an **IP whitelist**, add your Render service's outbound IP addresses (Render → the service → Settings → Outbound IP addresses). Otherwise Gate.io rejects the bot's calls with `IP_FORBIDDEN`, and the bot tells you the server's current IP.
- USDT in your Gate.io **USDT futures** account.

## Environment variables

| Variable | What to enter |
|---|---|
| `API_KEY` | Gate.io API key |
| `SECRET_KEY` | Gate.io API secret |
| `TELEGRAM_BOT_TOKEN` | Bot token issued by [@BotFather](https://t.me/BotFather) |
| `TELEGRAM_CHAT_ID` | Your own Telegram chat ID — the bot ignores every other sender |

Nothing else to configure: the exchange, trading mode and state-file location are built into the image.

## What the bot does

- Decides once per daily candle, right after the exchange closes it (UTC midnight), and sends market orders at once (the backtest's “fill at the close”).
- Watches the stops itself around the clock and closes at market when a stop is reached —
  the stops are **not** resting orders on the exchange, so **keep the bot running**. Stopping the bot stops the stop-watching.
- Keeps one account per symbol (trading pocket + vault), like TanQuant's multi-asset backtest. The Turtle merge moves part of the profit to the
  vault (spot wallet) on a trend exit and reinjects it after a deep drawdown, by the USA Turtle rules.
- Never stops trading because of an exchange answer — a refused or failed request skips that symbol for that cycle and is retried.

⚠️ Futures trading carries a very high risk of loss. Backtest results do not guarantee future returns.
