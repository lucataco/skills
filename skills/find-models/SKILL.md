---
name: find-models
description: >
  Find AI models on Replicate. Use when you need to discover models for a task
  like image generation, video, speech, or text. Covers search, collections,
  model schemas, and how to pick the right model.
---

# Finding models on Replicate

Replicate hosts thousands of AI models. Use the search API and curated collections to find the right one.

## Docs

- Reference: <https://replicate.com/docs/llms.txt>
- OpenAPI schema: <https://api.replicate.com/openapi.json>
- MCP server: <https://mcp.replicate.com>
- Per-model docs: `https://replicate.com/{owner}/{model}/llms.txt`
- Set `Accept: text/markdown` when requesting docs pages for Markdown responses.

## Search API

The search endpoint finds models, collections, and docs by text query:

```bash
curl -s -H "Authorization: Bearer $REPLICATE_API_TOKEN" \
  "https://api.replicate.com/v1/search?query=text+to+speech" | jq '[.models[:3][] | {name: .model.name, owner: .model.owner, runs: .model.run_count}]'
```

```python
import replicate

page = replicate.models.search("text to speech")
for model in page.results[:3]:
    print(f"{model.owner}/{model.name} — {model.run_count} runs")
    print(f"  {model.description}")
```

```javascript
const Replicate = require("replicate");
const replicate = new Replicate();

const page = await replicate.models.search("text to speech");
for (const model of page.results.slice(0, 3)) {
  console.log(`${model.owner}/${model.name} — ${model.run_count} runs`);
  console.log(`  ${model.description}`);
}
```

Search returns metadata including `tags`, `generated_description`, and `run_count`. It also returns matching collections alongside model results.

## Collections

Collections are curated groups of models maintained by Replicate staff. They're a good way to discover vetted models for specific tasks.

### List all collections

```bash
curl -s -H "Authorization: Bearer $REPLICATE_API_TOKEN" \
  https://api.replicate.com/v1/collections | jq '[.results[] | {slug, name}]'
```

```python
import replicate

page = replicate.collections.list()
for collection in page.results:
    print(f"{collection.slug}: {collection.name} — {collection.description}")
```

```javascript
const Replicate = require("replicate");
const replicate = new Replicate();

const page = await replicate.collections.list();
for (const collection of page.results) {
  console.log(
    `${collection.slug}: ${collection.name} — ${collection.description}`,
  );
}
```

### Get a collection

```bash
curl -s -H "Authorization: Bearer $REPLICATE_API_TOKEN" \
  https://api.replicate.com/v1/collections/text-to-image | jq '{name, slug, description, model_count: (.models | length)}'
```

```python
import replicate

collection = replicate.collections.get("text-to-image")
print(f"{collection.name}: {collection.description}")
for model in collection.models[:3]:
    print(f"  {model.owner}/{model.name}")
```

```javascript
const Replicate = require("replicate");
const replicate = new Replicate();

const collection = await replicate.collections.get("text-to-image");
console.log(`${collection.name}: ${collection.description}`);
for (const model of collection.models.slice(0, 3)) {
  console.log(`  ${model.owner}/${model.name}`);
}
```

### The `official` collection

The `official` collection contains models that are always warm, have stable APIs, and predictable per-run pricing. Always prefer official models when available.

## Reading model schemas

Every model exposes its input/output schema via the models API. Always fetch the schema before running a model, as schemas change.

```bash
curl -s -H "Authorization: Bearer $REPLICATE_API_TOKEN" \
  https://api.replicate.com/v1/models/black-forest-labs/flux-2-klein-9b \
  | jq '.latest_version.openapi_schema.components.schemas.Input.properties | to_entries[] | {name: .key, type: .value.type, description: .value.description}'
```

```python
import replicate

model = replicate.models.get("black-forest-labs", "flux-schnell")
input_schema = model.latest_version.openapi_schema["components"]["schemas"]["Input"]
for name, prop in input_schema["properties"].items():
    required = name in input_schema.get("required", [])
    print(f"{'*' if required else ' '} {name}: {prop.get('type', '?')} — {prop.get('description', '')}")
```

```javascript
const Replicate = require("replicate");
const replicate = new Replicate();

const model = await replicate.models.get("black-forest-labs", "flux-schnell");
const inputSchema =
  model.latest_version.openapi_schema.components.schemas.Input;
const required = inputSchema.required || [];
for (const [name, prop] of Object.entries(inputSchema.properties)) {
  console.log(
    `${required.includes(name) ? "*" : " "} ${name}: ${prop.type || "?"} — ${prop.description || ""}`,
  );
}
```

The schema path is: `model.latest_version.openapi_schema.components.schemas.Input.properties`

Each property may include: `type`, `description`, `default`, `minimum`/`maximum`, `enum`, `format` (e.g. `uri` for file inputs), and `x-order`.

## How to pick the right model

- Prefer official models. They're always warm (no cold boot), have stable APIs, and predictable pricing.
- Prefer the latest version. If search returns Kling 2.5 and Kling 3.0, use Kling 3.
- Run count can be misleading. Old models accumulate runs over time but may be outdated. A model with 10M runs from 2023 is likely worse than a model with 100K runs from 2025.
- Prefer recently released models. The AI space moves fast.
- Check model tags. Search results include tags like `image-generation`, `video`, `audio` to help filter.
- Use collections to narrow the shortlist before deep comparison.
- Avoid listing all models via API. It's a firehose. Use targeted queries.

## Model identifiers

- **Official models**: `owner/name` (e.g. `black-forest-labs/flux-2-klein-9b`). Routes to the latest version automatically.
- **Community models**: `owner/name:version_id`. You must pin a specific version. Community models can cold-boot and take time to start.

## Listing model versions

Each model can have multiple versions. Only community models expose a version list.

```bash
curl -s -H "Authorization: Bearer $REPLICATE_API_TOKEN" \
  https://api.replicate.com/v1/models/stability-ai/sdxl/versions | jq '[.results[:3][] | {id: .id, created_at: .created_at}]'
```

```python
import replicate

model = replicate.models.get("stability-ai", "sdxl")
versions = model.versions.list()
for v in versions.results[:3]:
    print(f"{v.id} — created {v.created_at}")
```

```javascript
const Replicate = require("replicate");
const replicate = new Replicate();

const versions = await replicate.models.versions.list("stability-ai", "sdxl");
for (const v of versions.results.slice(0, 3)) {
  console.log(`${v.id} — created ${v.created_at}`);
}
```
