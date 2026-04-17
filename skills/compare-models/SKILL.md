---
name: compare-models
description: >
  Compare Replicate models by cost, speed, quality, and capabilities. Use when
  deciding between multiple models for a task, evaluating tradeoffs, or building
  model selection logic into an app.
---

# Comparing models on Replicate

When multiple models can do the same task, compare them on cost, speed, quality, and capabilities to pick the right one.

## Docs

- Reference: <https://replicate.com/docs/llms.txt>
- OpenAPI schema: <https://api.replicate.com/openapi.json>
- MCP server: <https://mcp.replicate.com>
- Per-model docs: `https://replicate.com/{owner}/{model}/llms.txt`

## Comparison workflow

1. Search or browse collections to build a shortlist of candidate models.
2. Fetch each model's schema to compare inputs, outputs, and capabilities.
3. Check pricing from model metadata or the Replicate website.
4. Run a small batch of test predictions to compare output quality.
5. Pick the model that best fits your constraints (cost, latency, quality).

## Fetching model metadata for comparison

```python
import replicate

models_to_compare = [
    "black-forest-labs/flux-2-klein-9b",
    "black-forest-labs/flux-schnell",
]

for model_id in models_to_compare:
    owner, name = model_id.split("/")
    model = replicate.models.get(owner, name)
    schema = model.latest_version.openapi_schema["components"]["schemas"]["Input"]
    input_names = list(schema["properties"].keys())
    print(f"{model_id}")
    print(f"  runs: {model.run_count}")
    print(f"  inputs: {', '.join(input_names[:5])}")
```

```javascript
const Replicate = require("replicate");
const replicate = new Replicate();

const models = [
  "black-forest-labs/flux-2-klein-9b",
  "black-forest-labs/flux-schnell",
];

for (const modelId of models) {
  const [owner, name] = modelId.split("/");
  const model = await replicate.models.get(owner, name);
  const schema =
    model.latest_version.openapi_schema.components.schemas.Input;
  const inputNames = Object.keys(schema.properties).slice(0, 5);
  console.log(`${modelId}`);
  console.log(`  runs: ${model.run_count}`);
  console.log(`  inputs: ${inputNames.join(", ")}`);
}
```

## Running test predictions side by side

Run the same input through multiple models to compare outputs:

```python
import replicate
import time

models = [
    "black-forest-labs/flux-2-klein-9b",
    "black-forest-labs/flux-schnell",
]

predictions = [
    replicate.predictions.create(
        model=model_id,
        input={"prompt": "a red panda in a bamboo forest", "num_outputs": 1},
    )
    for model_id in models
]

results = {}
while len(results) < len(predictions):
    time.sleep(1)
    for pred in predictions:
        if pred.id not in results:
            p = replicate.predictions.get(pred.id)
            if p.status in ("succeeded", "failed", "canceled"):
                results[pred.id] = p

for model_id, pred in zip(models, predictions):
    result = results[pred.id]
    elapsed = ""
    if result.metrics and result.metrics.get("predict_time"):
        elapsed = f" ({result.metrics['predict_time']:.1f}s)"
    print(f"{model_id}{elapsed}: {result.output}")
```

## What to compare

### Speed

- Check `metrics.predict_time` on completed predictions for actual inference time.
- Official models are always warm. Community models can cold-boot (seconds to minutes).
- For latency-sensitive apps, prefer models with fast inference and no cold start.

### Cost

- Official models have predictable per-run pricing.
- Community models charge by compute time (GPU-seconds).
- Run a few predictions and check the `metrics` field for actual cost data.
- Cheaper models often have quality tradeoffs. Test before committing.

### Quality

- Run the same prompts through each model and compare outputs visually.
- For image models, compare detail, text rendering, prompt adherence, and artifacts.
- For language models, compare coherence, accuracy, and instruction following.
- Quality is subjective. Match it to your use case, not a leaderboard.

### Capabilities

- Compare input schemas: does the model accept the inputs you need (reference images, masks, aspect ratios)?
- Check output formats: does it return the file type you need?
- Check if the model supports features like streaming, webhooks, or multi-image input.

## Key tradeoffs

| Priority        | Favor                           | Accept                           |
| --------------- | ------------------------------- | -------------------------------- |
| Lowest cost     | Smaller/distilled models        | Slower inference, lower quality  |
| Lowest latency  | Official models, schnell/turbo  | Higher cost per run              |
| Highest quality | Pro/max/quality variants        | Slower inference, higher cost    |
| Most control    | Models with ControlNet, masks   | More complex input setup         |

## Official vs community models

- Official models: always warm, stable APIs, predictable pricing, maintained by Replicate.
- Community models: may cold-boot, require version pinning, maintained by the author.
- If a community model meets your needs and an official model doesn't, consider creating a deployment for consistent uptime.

## Image model comparison reference

For image generation and editing specifically, see the [prompt-images](../prompt-images/SKILL.md) skill, which includes model selection tables for tasks like text rendering, style transfer, character consistency, inpainting, and more.
