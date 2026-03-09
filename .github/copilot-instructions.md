# NostalgiaForInfinity Bot — Copilot Instructions

## Project Overview
This is a Freqtrade crypto trading bot running **NostalgiaForInfinityX7** strategy on Bybit futures.

## Deployment
- The bot runs on a remote server. Connection details are stored in Copilot repo memory (`/memories/repo/nfi-bot-server.md`). Read that file before performing any server operations.
- Server config files (`user_data/config-private.json`) are gitignored — sync them manually via `docker cp` (see update procedure in repo memory).

## Config Architecture
The bot uses a layered config system:
1. `user_data/config.json` loads base configs via `add_config_files`
2. `user_data/config-private.json` is loaded last and overrides everything
3. Docker `.env` on the server overrides config values (telegram, API creds)

Key parameters are in `config-private.json`:
- `nfi_parameters.system_v3_1_stop_threshold_doom_futures` — doom stoploss threshold (currently 0.35)
- `pair_blacklist` — must be kept in sync with upstream `configs/blacklist-bybit.json`

## Update Workflow
1. Fetch upstream: `git fetch upstream && git merge upstream/main`
2. Check for new blacklist entries and sync to `config-private.json`
3. Push: `git push origin main`
4. Deploy to server (see repo memory for full procedure)
5. Restart: `docker compose restart`

## Important Notes
- Strategy version is in `NostalgiaForInfinityX7.py` (`return "v17.x.xxx"`)
- Upstream repo: `iterativv/NostalgiaForInfinity`
- Never modify strategy `.py` files — only configs
- `user_data/config-private.json` contains exchange API keys — never commit to git
