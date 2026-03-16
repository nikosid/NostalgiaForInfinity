# NostalgiaForInfinity Bot - OpenCode Project Rules

## Project Overview
This repository contains a Freqtrade crypto trading bot using the NostalgiaForInfinityX7 strategy on Bybit futures (dry run mode).

Primary working model for this repo:
- Prefer config changes over strategy code changes.
- Keep edits minimal, targeted, and easy to review.

## Hard Safety Rules
- Never modify strategy files (`NostalgiaForInfinityX.py`, `NostalgiaForInfinityX2.py`, ..., `NostalgiaForInfinityX7.py`) unless the user explicitly asks for it.
- Never commit secrets. In particular, never commit `user_data/config-private.json`.
- Treat API keys, exchange credentials, and Telegram tokens as sensitive.
- Do not run destructive git commands (`git reset --hard`, `git checkout --`, force-push) unless explicitly requested.
- Ask for confirmation before any production-impacting operation (remote restart, deploy, or config copy to live container).

## Production Server
- Host: `forge4.nikosid.com`
- SSH: `ssh freqtrade@forge4.nikosid.com` (key auth)
- Bot directory: `~/bot3-NostalgiaForInfinity`
- Container: `NFI_X7_Bybit_bybit_futures-NostalgiaForInfinityX7`
- Image: `freqtradeorg/freqtrade:stable`
- API port: `8090`

Permission caveat:
- Repo files are owned by `freqtrade:freqtrade`.
- Some `user_data/` files are owned by `ubuntu:ubuntu`.
- If `config-private.json` needs updating on server, use `docker cp` into the container.

## Config Architecture
Main config flow:
1. `user_data/config.json` loads base configs through `add_config_files`:
   - `../configs/trading_mode-futures.json`
   - `../configs/pairlist-volume-bybit-usdt.json`
   - `../configs/blacklist-bybit.json`
   - `../configs/exampleconfig.json`
   - `config-private.json` (loaded last, overrides all)
2. Docker `.env` on server can override config values (including Telegram and API credentials).

Key private parameters in `user_data/config-private.json`:
- `nfi_advanced_mode: true` (required for overriding non-safe parameters)
- `nfi_parameters.system_v3_1_stop_threshold_doom_futures`
- `nfi_parameters.system_v3_1_stop_threshold_futures_rebuy`
- `pair_blacklist` (must be kept in sync with upstream blacklist + custom additions)

## Upstream Sync Workflow
When updating from upstream (`iterativv/NostalgiaForInfinity`):
1. `git fetch upstream && git merge upstream/main`
2. Check blacklist changes: `git diff HEAD~1 -- configs/blacklist-bybit.json`
3. Mirror new blacklist entries into `user_data/config-private.json` if needed
4. `git push origin main`

Git remotes:
- `origin`: `nikosid/NostalgiaForInfinity` (fork)
- `upstream`: `iterativv/NostalgiaForInfinity` (official)

## Production Deployment Runbook
Use only after explicit confirmation.

```bash
# 1) Pull code on server
ssh freqtrade@forge4.nikosid.com "cd bot3-NostalgiaForInfinity && git pull origin main"

# 2) If config-private.json changed, copy via container
scp user_data/config-private.json freqtrade@forge4.nikosid.com:/tmp/config-private.json
ssh freqtrade@forge4.nikosid.com "cd bot3-NostalgiaForInfinity && docker cp /tmp/config-private.json NFI_X7_Bybit_bybit_futures-NostalgiaForInfinityX7:/freqtrade/user_data/config-private.json"

# 3) Restart and verify health
ssh freqtrade@forge4.nikosid.com "cd bot3-NostalgiaForInfinity && docker compose restart && sleep 20 && docker compose ps"
```

Expected result:
- Bot container becomes `(healthy)` in about 20-30 seconds.

## Log Checks
```bash
# Recent logs
ssh freqtrade@forge4.nikosid.com "cd bot3-NostalgiaForInfinity && docker compose logs --tail 50"

# Verify overridden parameters loaded
ssh freqtrade@forge4.nikosid.com "cd bot3-NostalgiaForInfinity && docker compose logs" | grep "changed from"
```

## Change Policy
- Prefer safe, incremental diffs.
- Preserve existing file style and structure.
- When changing configs, explain expected runtime impact.
- For risky or ambiguous changes, stop and ask before proceeding.
