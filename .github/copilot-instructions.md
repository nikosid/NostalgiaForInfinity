# NostalgiaForInfinity Bot — Copilot Instructions

## Project Overview
This is a Freqtrade crypto trading bot running **NostalgiaForInfinityX7** strategy on Bybit futures (dry run mode).

## Production Server
- **Host:** `forge4.nikosid.com`
- **SSH:** `ssh freqtrade@forge4.nikosid.com` (key auth, no password)
- **Bot directory:** `~/bot3-NostalgiaForInfinity`
- **Container:** `NFI_X7_Bybit_bybit_futures-NostalgiaForInfinityX7` (image: `freqtradeorg/freqtrade:stable`)
- **API port:** 8090
- **File permissions:** repo files owned by `freqtrade:freqtrade`, but `user_data/` files owned by `ubuntu:ubuntu` (644) — use `docker cp` to update configs inside the container (direct `cp` won't work).

## Config Architecture
The bot uses a layered config system:
1. `user_data/config.json` loads base configs via `add_config_files`:
   - `../configs/trading_mode-futures.json`
   - `../configs/pairlist-volume-bybit-usdt.json`
   - `../configs/blacklist-bybit.json`
   - `../configs/exampleconfig.json`
   - `config-private.json` (loaded last, overrides all)
2. Docker `.env` on the server overrides config values (telegram, API creds)

Key parameters in `config-private.json`:
- `nfi_advanced_mode: true` — required for overriding non-safe parameters
- `nfi_parameters.system_v3_1_stop_threshold_doom_futures` — doom stoploss threshold (currently 0.35, default 0.70)
- `nfi_parameters.system_v3_1_stop_threshold_futures_rebuy` — rebuy doom threshold (currently 0.70, default 1.40)
- `pair_blacklist` — must be kept in sync with upstream `configs/blacklist-bybit.json` + our custom additions

## Update Workflow (upstream sync)
1. `git fetch upstream && git merge upstream/main`
2. Check for new blacklist entries: `git diff HEAD~1 -- configs/blacklist-bybit.json`
3. Sync new entries to `user_data/config-private.json` blacklist
4. `git push origin main`
5. Deploy to server (see below)

## Production Deployment
```bash
# 1. Pull code on server
ssh freqtrade@forge4.nikosid.com "cd bot3-NostalgiaForInfinity && git pull origin main"

# 2. If config-private.json changed — copy via docker cp (not scp directly due to permissions)
scp user_data/config-private.json freqtrade@forge4.nikosid.com:/tmp/config-private.json
ssh freqtrade@forge4.nikosid.com "cd bot3-NostalgiaForInfinity && docker cp /tmp/config-private.json NFI_X7_Bybit_bybit_futures-NostalgiaForInfinityX7:/freqtrade/user_data/config-private.json"

# 3. Restart and verify
ssh freqtrade@forge4.nikosid.com "cd bot3-NostalgiaForInfinity && docker compose restart && sleep 20 && docker compose ps"
```
The bot should show status `(healthy)` within ~20–30 seconds after restart.

## Checking Logs
```bash
# Recent logs
ssh freqtrade@forge4.nikosid.com "cd bot3-NostalgiaForInfinity && docker compose logs --tail 50"

# Verify parameters loaded
ssh freqtrade@forge4.nikosid.com "cd bot3-NostalgiaForInfinity && docker compose logs" | grep "changed from"
```

## Important Notes
- Strategy version is in `NostalgiaForInfinityX7.py` (`return "v17.x.xxx"`)
- Upstream repo: `iterativv/NostalgiaForInfinity`
- Never modify strategy `.py` files — only configs
- `user_data/config-private.json` contains exchange API keys — **never commit to git**
- Git remotes: `origin` = `nikosid/NostalgiaForInfinity` (fork), `upstream` = `iterativv/NostalgiaForInfinity` (official)
