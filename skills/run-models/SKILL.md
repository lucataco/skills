---
name: run-models
description: >
  Run AI models on Replicate via predictions, webhooks, and streaming. Use when
  building apps that call Replicate models, handling file I/O, running concurrent
  predictions, or chaining multiple models together.
---

# Running models on Replicate

Models on Replicate are run by creating predictions. A prediction goes through these states: `starting` -> `processing` -> `succeeded` / `failed` / `canceled`.

## Docs

- Reference: <https://replicate.com/docs/llms.txt>
- OpenAPI schema: <https://api.replicate.com/openapi.json>
- MCP server: <https://mcp.replicate.com>
- Per-model docs: `https://replicate.com/{owner}/{model}/llms.txt`

## Key rules

- Always fetch the model schema first. Even popular models change their interfaces.
- Validate inputs against schema constraints: check `minimum`, `maximum`, `enum` values.
- Don't set optional inputs without reason. Let the model's defaults work.
- Use HTTPS URLs for file inputs. Base64 works but is slower.
- Run predictions concurrently. Don't wait for one to finish before starting the next.
- Output file URLs expire in 1 hour. Back them up to R2/S3 if you need to keep them.

## The `replicate.run()` convenience wrapper

The simplest way to run a model. Creates a prediction, waits for it, and returns the output.

```python
import replicate

output = replicate.run(
    "black-forest-labs/flux-2-klein-9b",
    input={"prompt": "a red panda in a bamboo forest", "num_outputs": 1},
)
for item in output:
    print(item.url)
```

```javascript
const Replicate = require("replicate");
const replicate = new Replicate();

const output = await replicate.run("black-forest-labs/flux-2-klein-9b", {
  input: { prompt: "a red panda in a bamboo forest", num_outputs: 1 },
});
console.log(output);
```

## Create a prediction (async)

For more control, create a prediction and poll or use webhooks.

```bash
curl -s -X POST \
  -H "Authorization: Bearer $REPLICATE_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"version": "black-forest-labs/flux-2-klein-9b", "input": {"prompt": "a red panda in a bamboo forest", "num_outputs": 1}}' \
  https://api.replicate.com/v1/predictions | jq '{id, status}'
```

```python
import replicate

prediction = replicate.predictions.create(
    model="black-forest-labs/flux-2-klein-9b",
    input={"prompt": "a red panda in a bamboo forest", "num_outputs": 1},
)
print(prediction.id, prediction.status)
```

```javascript
const Replicate = require("replicate");
const replicate = new Replicate();

const prediction = await replicate.predictions.create({
  model: "black-forest-labs/flux-2-klein-9b",
  input: { prompt: "a red panda in a bamboo forest", num_outputs: 1 },
});
console.log(prediction.id, prediction.status);
```

## Polling for results

```python
import replicate
import time

prediction = replicate.predictions.create(
    model="black-forest-labs/flux-2-klein-9b",
    input={"prompt": "a red panda in a bamboo forest", "num_outputs": 1},
)
while prediction.status not in ("succeeded", "failed", "canceled"):
    time.sleep(1)
    prediction = replicate.predictions.get(prediction.id)

print(prediction.status)
if prediction.output:
    for url in prediction.output:
        print(url)
```

```javascript
const Replicate = require("replicate");
const replicate = new Replicate();

let prediction = await replicate.predictions.create({
  model: "black-forest-labs/flux-2-klein-9b",
  input: { prompt: "a red panda in a bamboo forest", num_outputs: 1 },
});
while (!["succeeded", "failed", "canceled"].includes(prediction.status)) {
  await new Promise((r) => setTimeout(r, 1000));
  prediction = await replicate.predictions.get(prediction.id);
}
console.log(prediction.status);
if (prediction.output) {
  for (const url of prediction.output) {
    console.log(url);
  }
}
```

## Synchronous mode with `Prefer: wait`

Hold the connection open until the prediction finishes (up to 60 seconds).

```bash
curl -s -X POST \
  -H "Authorization: Bearer $REPLICATE_API_TOKEN" \
  -H "Content-Type: application/json" \
  -H "Prefer: wait=60" \
  -d '{"version": "black-forest-labs/flux-2-klein-9b", "input": {"prompt": "a red panda in a bamboo forest", "num_outputs": 1}}' \
  https://api.replicate.com/v1/predictions | jq '{id, status, output}'
```

If the model doesn't finish in time, the response returns the prediction in its current state. Poll `urls.get` for the final result.

## File inputs

Prefer HTTPS URLs. Output URLs from one prediction can be passed directly as file inputs to the next.

```python
import replicate

output = replicate.run(
    "black-forest-labs/flux-kontext-pro",
    input={
        "prompt": "make it look like a watercolor painting",
        "input_image": "https://picsum.photos/id/237/200/300.jpg",
    },
)
print(output.url)
```

## Concurrent predictions

Start all predictions you can, then collect results. Don't run them serially.

```python
import replicate
import time

prompts = [
    "a red panda eating a bamboo sandwich",
    "a blue parrot riding a bicycle",
    "a green iguana playing chess",
]

predictions = [
    replicate.predictions.create(
        model="black-forest-labs/flux-2-klein-9b",
        input={"prompt": p, "num_outputs": 1},
    )
    for p in prompts
]

results = {}
while len(results) < len(predictions):
    time.sleep(1)
    for pred in predictions:
        if pred.id not in results:
            p = replicate.predictions.get(pred.id)
            if p.status in ("succeeded", "failed", "canceled"):
                results[pred.id] = p

for pred in results.values():
    print(pred.status, pred.output)
```

```javascript
const Replicate = require("replicate");
const replicate = new Replicate();

const prompts = [
  "a red panda eating a bamboo sandwich",
  "a blue parrot riding a bicycle",
  "a green iguana playing chess",
];

const predictions = await Promise.all(
  prompts.map((prompt) =>
    replicate.predictions.create({
      model: "black-forest-labs/flux-2-klein-9b",
      input: { prompt, num_outputs: 1 },
    }),
  ),
);

const poll = async (id) => {
  let pred = await replicate.predictions.get(id);
  while (!["succeeded", "failed", "canceled"].includes(pred.status)) {
    await new Promise((r) => setTimeout(r, 1000));
    pred = await replicate.predictions.get(id);
  }
  return pred;
};

const results = await Promise.all(predictions.map((p) => poll(p.id)));
for (const result of results) {
  console.log(result.status, result.output);
}
```

## Webhooks

Set `webhook` on the prediction to receive a POST when it completes.

```bash
curl -s -X POST \
  -H "Authorization: Bearer $REPLICATE_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "version": "black-forest-labs/flux-2-klein-9b",
    "input": {"prompt": "a red panda in a bamboo forest", "num_outputs": 1},
    "webhook": "https://example.com/webhook",
    "webhook_events_filter": ["completed"]
  }' \
  https://api.replicate.com/v1/predictions | jq '{id, status}'
```

Webhook events: `start`, `output`, `logs`, `completed`. Validate signatures using the `Webhook-ID`, `Webhook-Timestamp`, and `Webhook-Signature` headers. Get the signing secret from `GET /v1/webhooks/default/secret`.

## Cancel a prediction

```python
import replicate

prediction = replicate.predictions.create(
    model="black-forest-labs/flux-2-klein-9b",
    input={"prompt": "a red panda in a bamboo forest", "num_outputs": 1},
)
cancelled = replicate.predictions.cancel(prediction.id)
print(cancelled.status)
```

```javascript
const Replicate = require("replicate");
const replicate = new Replicate();

const prediction = await replicate.predictions.create({
  model: "black-forest-labs/flux-2-klein-9b",
  input: { prompt: "a red panda in a bamboo forest", num_outputs: 1 },
});
const cancelled = await replicate.predictions.cancel(prediction.id);
console.log(cancelled.status);
```

## Prediction lifetime (auto-cancel)

Set `lifetime` to cancel predictions that run too long. Accepts `30s`, `5m`, `1h`, etc.

```bash
curl -s -X POST \
  -H "Authorization: Bearer $REPLICATE_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "version": "black-forest-labs/flux-2-klein-9b",
    "input": {"prompt": "a red panda in a bamboo forest", "num_outputs": 1},
    "lifetime": "5m"
  }' \
  https://api.replicate.com/v1/predictions | jq '{id, status}'
```

## Streaming (SSE)

Language models that support streaming include a `stream` URL in the response.

```python
import replicate

prediction = replicate.predictions.create(
    model="meta/meta-llama-3-8b-instruct",
    input={"prompt": "write a haiku about mountains"},
    stream=True,
)
for event in prediction.stream():
    print(str(event), end="", flush=True)
print()
```

## Async Python

```python
import asyncio
import replicate

async def main():
    output = await replicate.async_run(
        "black-forest-labs/flux-2-klein-9b",
        input={"prompt": "a red panda in a bamboo forest", "num_outputs": 1},
    )
    for item in output:
        print(item.url)

asyncio.run(main())
```

## Multi-model workflows

Chain models by passing output URLs as file inputs to the next model.

```python
import replicate

image_output = replicate.run(
    "black-forest-labs/flux-2-klein-9b",
    input={"prompt": "a serene mountain lake at dawn"},
)
image_url = image_output[0].url

caption_output = replicate.run(
    "andreasjansson/blip-2:f677695e5e89f8b236e52ecd1d3f01beb44c34606419bcc19345e046d8f786f9",
    input={"image": image_url, "question": "What is in this image?"},
)
print(caption_output)
```

## HTTP API quick reference

| Endpoint                                   | Method | Purpose                    |
| ------------------------------------------ | ------ | -------------------------- |
| `/v1/predictions`                          | POST   | Create prediction (any model) |
| `/v1/predictions/{id}`                     | GET    | Poll for result            |
| `/v1/predictions/{id}/cancel`              | POST   | Cancel a prediction        |
| `/v1/predictions`                          | GET    | List your predictions      |
| `/v1/models/{owner}/{name}`                | GET    | Get model and schema       |
| `/v1/models/{owner}/{name}/versions`       | GET    | List model versions        |
| `/v1/search?query={query}`                 | GET    | Search models              |
| `/v1/collections`                          | GET    | List collections           |
| `/v1/collections/{slug}`                   | GET    | Get collection with models |
| `/v1/webhooks/default/secret`              | GET    | Get webhook signing secret |

Auth: `Authorization: Bearer $REPLICATE_API_TOKEN` on all requests. Base URL: `https://api.replicate.com`.
