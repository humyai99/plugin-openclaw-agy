---
name: agy-openclaw-proxy
version: 1.0.0
author: kun-it
license: MIT
description: Use when connecting agy to OpenClaw via OpenAI proxy.
metadata:
  hermes:
    tags:
      - agy
      - openclaw
      - openai-proxy
      - antigravity
    related_skills:
      - openclaw-operations
---

# agy → OpenClaw OpenAI Proxy

## When to Use
- เชื่อม agy (Antigravity) กับ OpenClaw ผ่าน proxy OpenAI-compatible
- แก้ปัญหา `UND_ERR_SOCKET` หรือ `TypeError: can only concatenate str (not "list")`
- ทดสอบ provider agy ใน OpenClaw

## Architecture
- `/home/administrator/scripts/agy_openai_proxy.py` — Python ThreadingHTTPServer on port 8020
- GET /v1/models — parses `agy models` (slug\tLabel, take first column)
- POST /v1/chat/completions — builds prompt from messages, spawns `agy -p` (timeout 600s)
- OpenClaw provider config in `/home/administrator/.openclaw/openclaw.json`: baseUrl `http://host.docker.internal:8020/v1`, apiKey `agy-local-proxy`, models gemini-3.8-flash-{low,medium,high}
- UFW: `ufw allow from 172.16.0.0/12 to any port 8020 proto tcp` (policy DROP blocks Docker bridge otherwise)

## Pitfalls (verified 2026-09-27)
1. **HTTP/1.1 required** — default BaseHTTPRequestHandler is HTTP/1.0 (closes connection per request). OpenClaw's fetch keeps sockets alive and reuses them → `UND_ERR_SOCKET` within ~20ms. Fix: `protocol_version = "HTTP/1.1"` on the handler class.
2. **content may be a list** — OpenClaw sends OpenAI content parts `[{"type":"text","text":...}]`. String concatenation `"User: " + content` raises `TypeError: can only concatenate str (not "list")`. Fix: `flatten_content()` that joins text parts.
3. **Restart proxy after code edits** — the running process keeps old code in memory; kill + restart, then re-test with a list-content POST before running an agent turn.
4. **Test from inside the container** — `docker exec openclaw-openclaw-gateway-1 node <test.js>`; use `host.docker.internal` (Docker's host gateway), not 127.0.0.1.

## Verification workflow
1. `curl http://localhost:8020/v1/models` (host)
2. `docker exec openclaw-openclaw-gateway-1 node /home/node/.openclaw/agy_post_test_list.js` (list-content POST from container)
3. `docker exec openclaw-openclaw-gateway-1 openclaw agent --agent main --message "สวัสดีครับ" --model agy/gemini-3.8-flash-medium --json --timeout 300`

## Data flow warning
agy routes requests to Google (Antigravity subscription). Sensitive data should stay on the local vllm/openthai provider (localhost:8000).
