# Quick Test Guide for Qwen local router

## Web UI: http://localhost:7860

| Try This Query | Expected Model |
|----------------|----------------|
| "Hello, how are you?" | nemotron-nano-9b (chit_chat) |
| "Solve x^2 + 5x + 6 = 0" | gpt-5-chat (hard_question) |
| "What's in this image?" | nemotron-12b-vl (image) |

## Terminal (curl)

```bash
curl -s -X POST http://localhost:8001/sfc_router/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"messages": [{"role": "user", "content": "YOUR QUERY HERE"}]}' | jq -r '.choices[0].message.content'
```