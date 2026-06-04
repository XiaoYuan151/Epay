# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

彩虹易支付 (Rainbow EasyPay) — a PHP-based aggregated payment gateway system supporting Alipay, WeChat Pay, QQ Pay, UnionPay, Douyin Pay, and 60+ payment channel plugins. No framework; plain PHP with a custom autoloader and PDO-based database layer.

## Development Environment

- PHP (no composer at root level; dependencies managed via `includes/vendor/` with composer in `includes/`)
- MySQL/MariaDB database with table prefix configurable in `config.php` (default: `pay`)
- Web server: Nginx or IIS (rewrite rules in `nginx.txt` and `IIS.txt`)
- Timezone: Asia/Shanghai

## Running Locally

1. Configure database credentials in `config.php`
2. Set up Nginx rewrite rules from `nginx.txt`
3. Run `/install/` to initialize the database, then place `install/install.lock`
4. Composer dependencies: `cd includes && composer install`

## Architecture

### Request Flow

- `submit.php` / `submit2.php` — payment submission entry points, delegate to `\lib\api\Pay::submit()`
- `pay.php` — handles payment callbacks/redirects via URL rewrite (`/pay/{plugin}/{trade_no}/`)
- `api.php` — merchant API (query orders, refund, settle) via `?act=` or rewritten `/api/{path}`
- `gateway.php` — legacy API gateway
- `cashier.php` — hosted cashier/checkout page

### Core Libraries (`includes/lib/`)

- `Payment.php` — sign generation/verification (MD5 and RSA)
- `Order.php` — order lifecycle (freeze, unfreeze, refund)
- `Channel.php` — payment channel configuration and routing
- `Plugin.php` — plugin discovery and loading
- `Cache.php` — system config caching (pre_config table)
- `PdoHelper.php` — database abstraction ($DB global)
- `Transfer.php` — merchant settlement/withdrawal
- `RiskCheck.php` — fraud/risk detection
- `ApiHelper.php` — API routing for `/api/{path}` URLs

### Plugin System

Each plugin lives in `plugins/{name}/` with a `{name}_plugin.php` entry file defining a class `{name}_plugin` with:
- `static $info` — metadata array: name, showname, author, types, transtypes, inputs, select
- Payment processing methods called by the core payment flow

### Key Globals

- `$DB` — PdoHelper instance
- `$conf` — system configuration array (from pre_config table, cached)
- `$CACHE` — Cache instance
- `SYS_KEY` — system-level secret key (from config)
- `DBQZ` — database table prefix

### Database Tables (prefixed with configurable prefix, default `pay_`)

- `pre_config` — key/value system settings
- `pre_order` — payment orders
- `pre_user` — merchants
- `pre_channel` — payment channels
- `pre_subchannel` — sub-channels
- `pre_settle` — settlement records
- `pre_type` — payment type definitions

### Directory Layout

- `admin/` — admin panel pages
- `user/` — merchant panel pages
- `includes/` — core system (common.php is the bootstrap)
- `plugins/` — payment channel plugins (blocked from web access)
- `template/` — frontend templates
- `paypage/` — payment page templates
- `assets/` — static assets
- `install/` — installer and SQL migrations

## Conventions

- Bootstrap: every page includes `includes/common.php` (sets up DB, config, session, functions)
- Pages that skip session set `$nosession = true` before requiring common.php
- Autoloading: `\lib\*` classes map to `includes/lib/*.php` via the custom Autoloader
- SQL uses prepared statements via PdoHelper for merchant-facing queries; admin queries use string interpolation with `daddslashes()`
- Config values are stored as rows in pre_config and accessed via the `$conf` array
- Version tracking: `VERSION` constant (app version) and `DB_VERSION` (schema version) in common.php
