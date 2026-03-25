---
name: claude-us-clash-config
description: Configure macOS Clash Verge so Claude and Anthropic-related traffic uses a dedicated US static proxy while other traffic keeps existing proxy/direct behavior. Use when setting up or updating a local machine for stable Claude access with IPRoyal or a similar static proxy.
---

# Claude US Clash Config

Use this skill when the goal is:

- keep existing Clash Verge subscriptions intact
- route `claude.ai` and related Anthropic domains through a dedicated US static proxy
- keep other traffic on the user's existing rules
- make terminal traffic follow local Clash rules instead of connecting directly to the proxy vendor

This skill assumes a macOS machine with Clash Verge Rev installed.

## Inputs to collect

Before editing anything, collect:

- `type` (`socks5` preferred, `http` acceptable)
- `host`
- `port`
- `username`
- `password`

Optional:

- extra domains that should also go through the dedicated Claude route

## Safety rules

Always follow these rules:

- Do not delete the user's existing subscription or main proxy group.
- Back up any file before editing it.
- Prefer editing Clash Verge enhancement files in `profiles/` when they exist.
- Do not make terminal traffic connect directly to IPRoyal or another proxy vendor.
- Terminal should connect only to local Clash, then let Clash rules decide routing.
- Validate YAML after every config edit.

## Expected Clash outcome

The target state is:

- node: `IPRoyal-US`
- group: `Claude-US`
- `Claude-US` options:
  - `IPRoyal-US`
  - `Proxy`
  - `DIRECT`

Only the following domains should go through `Claude-US` by default:

- `claude.ai`
- `anthropic.com`
- `claudeusercontent.com`
- `ipinfo.io` for testing
- `intercomcdn.com`
- `intercom.io`
- `intercomassets.com`
- `sentry.io`
- `segment.io`

Use `prepend`, not `append`, for these rules.

## Files to inspect first

Check these paths in order:

1. `~/Library/Application Support/io.github.clash-verge-rev.clash-verge-rev/profiles.yaml`
2. `~/Library/Application Support/io.github.clash-verge-rev.clash-verge-rev/profiles/`
3. `~/Library/Application Support/io.github.clash-verge-rev.clash-verge-rev/clash-verge.yaml`

If the active profile uses enhancement files, update these first:

- proxies enhancement file
- groups enhancement file
- rules enhancement file

Typical enhancement files look like:

- `po7tMfZFQw6V.yaml`
- `gVDxBcjmPiO4.yaml`
- `r6Nws8evMy8G.yaml`

## Edit pattern

### Proxies enhancement

Add a node like:

```yaml
append:
  - name: IPRoyal-US
    type: socks5
    server: 130.255.66.76
    port: 12324
    username: your_username
    password: your_password
```

### Groups enhancement

Add a group like:

```yaml
append:
  - name: Claude-US
    type: select
    proxies:
      - IPRoyal-US
      - Proxy
      - DIRECT
```

### Rules enhancement

Add high-priority rules like:

```yaml
prepend:
  - DOMAIN-SUFFIX,ipinfo.io,Claude-US
  - DOMAIN-SUFFIX,claude.ai,Claude-US
  - DOMAIN-SUFFIX,anthropic.com,Claude-US
  - DOMAIN-SUFFIX,claudeusercontent.com,Claude-US
  - DOMAIN-SUFFIX,intercomcdn.com,Claude-US
  - DOMAIN-SUFFIX,intercom.io,Claude-US
  - DOMAIN-SUFFIX,intercomassets.com,Claude-US
  - DOMAIN-SUFFIX,sentry.io,Claude-US
  - DOMAIN-SUFFIX,segment.io,Claude-US
```

## Terminal behavior

The terminal should follow Clash rules by pointing to local Clash only:

```bash
export http_proxy="http://127.0.0.1:7897"
export https_proxy="http://127.0.0.1:7897"
export all_proxy="socks5h://127.0.0.1:7897"
export HTTP_PROXY="$http_proxy"
export HTTPS_PROXY="$https_proxy"
export ALL_PROXY="$all_proxy"
```

Do not point shell variables directly to the static proxy provider.

## Verification steps

After edits:

1. validate YAML
2. refresh Clash Verge subscription
3. confirm `Claude-US` exists and is set to `IPRoyal-US`
4. run:

```bash
curl https://ipinfo.io
curl -I https://claude.ai
curl -I https://www.anthropic.com
```

Expected result:

- `ipinfo.io` shows the US static IP
- Claude and Anthropic domains respond successfully

## When to fall back

If the dedicated static proxy is unstable:

- keep `Claude-US` in place
- switch `Claude-US` to `Proxy` temporarily
- preserve the config so the user can switch back later

This is preferable to deleting the static proxy config entirely.
