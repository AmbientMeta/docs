---
title: Quickstart
description: Get your first API call working
---

## 1. Get your API key

Sign up at [ambientmeta.com](https://ambientmeta.com) and copy your API key from the dashboard.

## 2. Install the SDK

```bash
pip install ambientmeta
```

## 3. Sanitize some text

```python
from ambientmeta import AmbientMeta

client = AmbientMeta(api_key="am_live_your_key_here")

# Sanitize text before sending to an LLM
result = client.sanitize("Email John Smith at john@acme.com about the project")

print(result.sanitized)
# "Email [PERSON_1] at [EMAIL_ADDRESS_1] about the project"

print(result.session_id)
# "ses_abc123..."
```

## 4. Call your LLM (with safe text)

```python
import openai

response = openai.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": result.sanitized}]
)

llm_response = response.choices[0].message.content
```

## 5. Rehydrate the response

```python
final = client.rehydrate(llm_response, result.session_id)

print(final.text)
# Original names and emails restored
```

<Tip>
**That's it!** Your LLM never saw the real PII. The original data stayed in your control the entire time.
</Tip>

## Using curl

### Sanitize

```bash
curl -X POST https://api.ambientmeta.com/v1/sanitize \
  -H "X-API-Key: am_live_xxx" \
  -H "Content-Type: application/json" \
  -d '{"text": "Email John Smith at john@acme.com about the merger"}'
```

Response:

```json
{
  "sanitized": "Email [PERSON_1] at [EMAIL_ADDRESS_1] about the merger",
  "session_id": "ses_a1b2c3d4e5f6",
  "redacted": false,
  "entities_found": 2,
  "entities": [
    {"placeholder": "[PERSON_1]", "type": "PERSON", "confidence": 0.97, "start": 6, "end": 16},
    {"placeholder": "[EMAIL_ADDRESS_1]", "type": "EMAIL_ADDRESS", "confidence": 0.99, "start": 20, "end": 33}
  ],
  "processing_ms": 12.3
}
```

### Rehydrate

```bash
curl -X POST https://api.ambientmeta.com/v1/rehydrate \
  -H "X-API-Key: am_live_xxx" \
  -H "Content-Type: application/json" \
  -d '{"text": "I will contact [PERSON_1] at [EMAIL_ADDRESS_1] tomorrow.", "session_id": "ses_a1b2c3d4e5f6"}'
```

Response:

```json
{
  "text": "I will contact John Smith at john@acme.com tomorrow.",
  "entities_restored": 2,
  "processing_ms": 3.1
}
```

## Next Steps

- [Full sanitize API reference](/api-reference/sanitize)
- [Submit corrections](/api-reference/feedback) to improve detection accuracy
- [Create custom patterns](/api-reference/patterns) for org-specific data
- [View insights](/api-reference/insights) and resolve conflicts
- Use with [LangChain](/guides/langchain) or [LlamaIndex](/guides/llamaindex)
- [Python SDK documentation](/sdks/python)
