# plugin-openclaw-agy

OpenAI-compatible proxy เชื่อม agy (Antigravity) กับ OpenClaw

## ไฟล์

- `scripts/agy_openai_proxy.py` — proxy พอร์ต 8020 (HTTP/1.1, flatten_content)
- `skills/SKILL.md` — สกิล agy-openclaw-proxy (architecture, pitfalls, verification)

## รัน

```bash
python3 scripts/agy_openai_proxy.py
```

## ทดสอบ

```bash
curl http://localhost:8020/v1/models
docker exec openclaw-openclaw-gateway-1 openclaw agent --agent main --message "สวัสดีครับ" --model agy/gemini-3.8-flash-medium --json --timeout 300
```

## ข้อควรระวัง

agy ส่งข้อมูลไป Google (Antigravity subscription) — ข้อมูลอ่อนไหวให้ใช้ provider vllm (openthai ท้องถิ่น) แทน
