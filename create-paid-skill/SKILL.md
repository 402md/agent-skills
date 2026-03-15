---
name: create-paid-skill
displayName: Create Paid AI Skill (SKILL.md + x402)
description: >-
  Use this skill when the user asks to "create a paid skill", "monetize an API",
  "create a SKILL.md", "set up x402 payments", "make a paid API for agents",
  "create an agent skill with payments", "build a skill for skills.sh",
  "convert OpenAPI to SKILL.md", "create an MCP tool", "create an A2A agent card",
  or wants to create a service that AI agents can discover and pay for autonomously.
  Guides the user step-by-step through creating a SKILL.md file following the
  SKILL.md Specification v2.0 (RFC), including payment configuration, endpoint
  definitions, agent-ready documentation, and publishing.
version: 1.0.0
author: 402md
license: MIT
type: SKILL
tags:
  - skill-creation
  - monetization
  - x402
  - payments
  - api
  - agents
  - skillmd
  - mcp
  - a2a
  - openapi
category: developer-tools
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - WebFetch
---

# Create a Paid AI Skill (SKILL.md + x402)

You are a specialist in creating SKILL.md files following the **SKILL.md Specification v2.0**.
SKILL.md is the open standard for describing capabilities that AI agents can discover,
understand, and pay for autonomously using the x402 payment protocol.

**Specification:** https://github.com/402md/skillmd/blob/main/SPEC.md
**Reference implementation:** `npm install @402md/skillmd`

## Design Goals (from the spec)

1. **Human-readable** — A SKILL.md is a markdown file. Developers read it, edit it, commit it.
2. **Machine-parseable** — YAML frontmatter provides structured metadata for agents and tools.
3. **Framework-agnostic** — Works with MCP, A2A, OpenAPI, LangChain, or any agent framework.
4. **Payment-native** — First-class support for pricing, payment networks, and non-custodial settlement.
5. **Backwards-compatible** — Any valid Anthropic Claude Code skill is a valid SKILL.md.
6. **Extensible** — Unknown frontmatter fields are preserved, not rejected (`additionalProperties: true`).

## File Format (2)

A SKILL.md file consists of two parts separated by `---` delimiters:

```
---
[YAML frontmatter]
---

[Markdown body]
```

- **Frontmatter** — YAML between the opening and closing `---`. Contains structured metadata.
- **Body** — Markdown after the closing `---`. Contains documentation, instructions, or both.
- **Encoding** — UTF-8. Both LF and CRLF line endings are accepted.
- **File name** — MUST be `SKILL.md` (case-sensitive).

Frontmatter extraction regex: `^---\r?\n([\s\S]*?)\r?\n---\r?\n?([\s\S]*)$`

---

## Step-by-Step Process

When the user asks to create a skill, follow these steps IN ORDER:

### Step 1: Determine the Skill Profile (5)

Ask the user what kind of skill they want. There are THREE profiles:

#### Profile A: Paid API (most common)

For services that charge per API call via x402.
**Required fields:** `name`, `description`, `base_url`, `payment`, `endpoints`

```yaml
---
name: web-scraper
description: Real-time web scraping with structured output
base_url: https://api.scraper.dev
type: API
payment:
  networks: [stellar]
  asset: USDC
  payTo: GABC...XYZ
endpoints:
  - path: /v1/scrape
    method: POST
    description: Scrape a URL
    priceUsdc: "0.005"
---
```

#### Profile B: Agent Skill (Claude Code compatible)

For pure agent instructions — no paid endpoints. Compatible with Anthropic's format.
**Required fields:** `name`, `description`

```yaml
---
name: code-reviewer
description: >-
  This skill should be used when the user asks to "review code",
  "check for bugs", "audit this file", or mentions code quality.
version: 1.0.0
type: SKILL
allowed-tools: [Read, Grep, Glob]
---

# Code Reviewer

When reviewing code, follow these steps:
1. Read the file completely before commenting
2. Check for security vulnerabilities (OWASP top 10)
3. Verify error handling at boundaries
4. Suggest improvements only when asked
```

> **Note:** The `SKILL` type makes `payment` and `endpoints` optional. The body follows
> Anthropic convention — imperative form, step-by-step procedures.

#### Profile C: Hybrid (Paid API + Agent Instructions)

A paid API that also provides agent instructions for how to use it.
**Required fields:** `name`, `description`, `base_url`, `payment`, `endpoints`

```yaml
---
name: weather-api
description: >-
  This skill should be used when the user asks about "weather",
  "temperature", "forecast", or needs meteorological data.
  Real-time weather data for any location worldwide.
base_url: https://api.weatherco.com
type: API
version: 1.0.0
payment:
  networks: [stellar, base]
  asset: USDC
  payTo: GABC...XYZ
endpoints:
  - path: /v1/current
    method: POST
    description: Get current weather
    priceUsdc: "0.001"
    inputSchema:
      type: object
      properties:
        location:
          type: string
      required: [location]
tags: [weather, geolocation]
---

# Weather API

When the user asks about weather:
1. Call `/v1/current` with the location ($0.001 per call)
2. If they want a forecast, call `/v1/forecast` ($0.005)
3. Always show the cost before making the call
```

### Step 2: Gather Information

Based on the profile, ask the user:

**For all profiles:**
1. **What does your skill/API do?**
2. **Who is the author?**

**For Paid API and Hybrid (Profile A & C) — also ask:**
3. **What endpoints does it have?** (method, path, what each does)
4. **What should each call cost?** (in USDC — see Pricing Guidelines below)
5. **What blockchain network?** Options: `stellar`, `base`, `stellar-testnet`, `base-sepolia`
6. **What is your wallet address?** (Stellar G-address for Stellar, EVM 0x-address for Base)
7. **Do you also have an EVM address?** (for `payToEvm` fallback — optional)
8. **What is the base URL?** (e.g., `https://api.myservice.com`)
9. **Do you have an SLA?** (e.g., `"99.9%"` — optional)
10. **What is your rate limit?** (e.g., `"1000/hour"` — optional)
11. **Do you have a sandbox/free test endpoint?** (URL — optional)

**For Agent Skill (Profile B) — also ask:**
3. **What tools should the agent be allowed to use?** (e.g., `Read`, `Bash`, `Write`, `Grep`, `Glob`, `WebFetch`)
4. **What trigger phrases should invoke this skill?** (for Claude Code auto-invocation)

If the user already has an OpenAPI spec, read it first to extract endpoints automatically using the `generateFromOpenAPI()` function from `@402md/skillmd`.

### Step 3: Generate the SKILL.md

#### Complete Frontmatter Schema (3)

Generate the frontmatter using ALL applicable fields from the spec:

```yaml
---
#3.1 Core Fields (required)
name: {kebab-case-name}                    # ^[a-z0-9][a-z0-9_-]*$ — max 100 chars
description: >-                            # max 2000 chars
  This skill should be used when the user asks to "{trigger phrase 1}",
  "{trigger phrase 2}", "{trigger phrase 3}", or needs {general description}.
  {One-line summary of what the API does}.
version: 1.0.0                             # semver (recommended)

#3.2 Identity Fields (optional)
displayName: {Human Readable Name}         # max 200 chars, falls back to name
author: {author-name}                      # max 100 chars
license: {MIT | proprietary | etc}

#3.3 Classification Fields
type: API                                  # API | SAAS | PRODUCT | SERVICE | SUBSCRIPTION | CONTENT | SKILL
tags: [{tag1}, {tag2}, {tag3}]             # max 20 items, for discovery
category: {primary-category}               # for marketplace listing

#3.4 Service Fields (for APIs)
base_url: {https://api.example.com}        # REQUIRED when endpoints present
sla: "99.9%"                               # uptime guarantee (optional)
rateLimit: "1000/hour"                     # rate limit info (optional)
sandbox: {https://sandbox.example.com}     # free test endpoint URL (optional)

#3.5 Payment Fields (REQUIRED when endpoints present, optional for type: SKILL)
payment:
  networks:                                # min 1 required
    - stellar                              # stellar | stellar-testnet | base | base-sepolia
    - base
  asset: USDC                              # default: USDC
  payTo: {G-address or 0x-address}         # primary recipient (REQUIRED)
  payToEvm: {0x-address}                   # EVM fallback (optional, ^0x[a-fA-F0-9]{40}$)
  facilitator: https://facilitator.402.md  # payment verification service (optional)

#3.6 Endpoint Fields (REQUIRED for API types)
endpoints:
  - path: /v1/{resource}                   # MUST start with /
    method: POST                           # GET | POST | PUT | DELETE | PATCH
    description: {What this endpoint does}
    priceUsdc: "{price}"                   # decimal string: ^\d+(\.\d+)?$
    inputSchema:                           # JSON Schema (optional)
      type: object
      properties:
        {param}:
          type: string
          description: {param description}
      required: [{required-params}]
    outputSchema:                          # JSON Schema (optional)
      type: object
      properties:
        {field}:
          type: string

#3.7 Agent Integration Fields
allowed-tools: [Read, Write, Bash]         # tools the skill can use (Anthropic compatible)

#3.8 Extensibility — any additional fields are preserved
# x-custom-field: "value"
# mcp_server: "@402md/mcp"
# marketplace:
#   featured: true
#   verified: true
---
```

#### Markdown Body (4)

The body is **the most important part**. The frontmatter provides metadata for tooling,
but the body is what an AI agent actually reads to understand how to use the skill.

> **Key insight from the spec:** No LLM has been trained on x402. The body MUST teach
> the agent what x402 is, how payment works, and how to call each endpoint — step by step,
> with examples.

**For Paid API (Profile A) and Hybrid (Profile C), write the body in this order:**

```markdown
# {Skill Name}

{One paragraph: what it does, when to use it, no API keys needed.}

## Payment Protocol (x402)

This API uses the x402 payment protocol. No API keys or accounts needed.
Payment is per-request in USDC on {network}.

**How it works:**

1. Make a normal HTTP request to any endpoint
2. The server responds with HTTP 402 (Payment Required) and a JSON body
   containing payment requirements (amount, recipient address, network)
3. Sign a USDC transfer authorization with your wallet (non-custodial,
   no funds leave your wallet until verified)
4. Retry the same request with the signed payment in the `X-PAYMENT` header
5. The server verifies the payment, settles on-chain, and returns the result

**Payment properties:**
- **Non-custodial**: The agent signs locally. No private key leaves the wallet.
- **Atomic**: On Stellar, settlement uses multi-op transactions. On Base, a Solidity contract splits in a single transaction.
- **Gasless**: The facilitator covers gas fees. Agent and seller pay zero gas.

**If using @402md/mcp:** Payment is handled automatically. Just call the
endpoint via `use_skill` and the MCP server handles steps 2-5.

**If calling directly:**
\`\`\`typescript
import { x402Fetch } from '@402md/x402'

const response = await x402Fetch('{base_url}{path}', {
  method: 'POST',
  body: JSON.stringify({ /* params */ }),
  stellarSecret: process.env.STELLAR_SECRET,
  network: '{network}'
})
\`\`\`

## Authentication

No API keys, no accounts, no registration. Authentication IS the payment.
Each request is independently paid via x402. The agent's wallet signature
serves as both authentication and payment authorization.

## Endpoints

### {Endpoint Name}

**{METHOD} {path}** — ${price} USDC

{Description of what this endpoint does.}

**Request:**
\`\`\`json
POST {base_url}{path}
Content-Type: application/json

{
  "param": "value"
}
\`\`\`

**Response (200):**
\`\`\`json
{
  "field": "value"
}
\`\`\`

**Parameters:**
- `param` (required): Description.
- `optionalParam`: Description. Default: `value`.

## Workflow

### {Common Use Case}

1. Call `{METHOD} {path}` with `{ "param": "value" }` (${price})
2. {Next step}
3. Always tell the user the cost before making the call
4. Wait for user confirmation, then execute

### {Another Use Case}

1. Call endpoint A first
2. Use result from A to call endpoint B
3. Total cost = $X — confirm with user first

## Error Handling

| Status | Meaning | What to do |
|--------|---------|------------|
| 400 | Bad request | Check input format. Show the error message to the user. |
| 402 | Payment required | Normal x402 flow — handled automatically by @402md/x402 or MCP. |
| 403 | Insufficient funds | Tell user: "Wallet balance too low. Fund your wallet to continue." |
| 404 | Not found | The resource doesn't exist. Don't retry. |
| 429 | Rate limited | Wait 60 seconds and retry once. If still 429, tell the user. |
| 500 | Server error | Retry once after 5 seconds. If still failing, tell the user. |
| 504 | Timeout | The request timed out. Retry once or suggest simpler parameters. |

## Pricing Summary

| Endpoint | Method | Price | Description |
|----------|--------|-------|-------------|
| `{path}` | {METHOD} | ${price} | {description} |

**Cost examples:**
- {Common operation}: ${cost}
- {Batch operation}: ${cost} × {count}

## Sandbox (Free Testing)

Test endpoint for free (limited):
\`\`\`
{METHOD} {sandbox_url}{path}
\`\`\`
No payment required. Use this to verify your integration works before making paid calls.
```

**For Agent Skill (Profile B), write the body as imperative instructions:**

```markdown
# {Skill Name}

{One paragraph: what this skill does.}

When {trigger condition}, follow these steps:
1. {Step 1}
2. {Step 2}
3. {Step 3}

## {Section for specific procedure}

1. {Detailed step}
2. {Detailed step}
```

**For Hybrid (Profile C), combine both:** Start with the API documentation (like Profile A)
but add agent-specific workflow instructions telling the agent exactly what to do and when.

#### Body Length Guidelines (4.7)

| Skill complexity | Recommended body length |
|-----------------|------------------------|
| Simple (1-2 endpoints) | 500-1,000 words |
| Medium (3-5 endpoints) | 1,000-3,000 words |
| Complex (6+ endpoints, workflows) | 3,000-8,000 words |
| Platform API (many features) | 5,000-15,000 words |

**Err on the side of more detail.** An agent with too much context wastes tokens.
An agent with too little context makes wrong API calls, wastes money, and fails.

### Step 4: Validate the SKILL.md (9)

After generating, validate using the @402md/skillmd library:

```typescript
import { validateSkillMd } from '@402md/skillmd'
import { readFileSync } from 'fs'

const content = readFileSync('SKILL.md', 'utf-8')
const result = validateSkillMd(content)

if (!result.valid) {
  console.error('Errors:', result.errors)
}
if (result.warnings.length > 0) {
  console.warn('Warnings:', result.warnings)
}
```

Or validate manually against these rules:

**Errors (MUST fix):**

| Rule | Field | Description |
|------|-------|-------------|
| `MISSING_NAME` | `name` | Name is required |
| `INVALID_NAME` | `name` | Must match `^[a-z0-9][a-z0-9_-]*$` |
| `MISSING_DESCRIPTION` | `description` | Description is required |
| `MISSING_BASE_URL` | `base_url` | Required when endpoints are present |
| `INVALID_BASE_URL` | `base_url` | Must be a valid URL |
| `MISSING_PAYMENT` | `payment` | Required when endpoints are present |
| `MISSING_NETWORKS` | `payment.networks` | At least one network required |
| `INVALID_NETWORK` | `payment.networks[]` | Must be: stellar, stellar-testnet, base, base-sepolia |
| `MISSING_PAY_TO` | `payment.payTo` | Recipient address required |
| `MISSING_ENDPOINTS` | `endpoints` | At least one endpoint required (unless `type: SKILL`) |
| `INVALID_PATH` | `endpoints[].path` | Must start with `/` |
| `INVALID_METHOD` | `endpoints[].method` | Must be: GET, POST, PUT, DELETE, PATCH |
| `INVALID_PRICE` | `endpoints[].priceUsdc` | Must match `^\d+(\.\d+)?$` |
| `DUPLICATE_ENDPOINT` | `endpoints[]` | Duplicate `{method} {path}` not allowed |

**Warnings (SHOULD fix):**

| Rule | Field | Description |
|------|-------|-------------|
| `MISSING_VERSION` | `version` | Recommended for published skills |
| `INVALID_VERSION` | `version` | Should follow semver |
| `MISSING_TAGS` | `tags` | Recommended for discovery |
| `TOO_MANY_TAGS` | `tags` | Max 20 tags |
| `UNRECOGNIZED_ADDRESS` | `payment.payTo` | Address doesn't match known formats |

### Step 5: Set Up Directory Structure (8)

Depending on the use case, recommend one of these structures:

**Standalone (single skill):**
```
project/
└── SKILL.md
```

**Skill Package (with supporting files):**
```
skill-name/
├── SKILL.md              # Required
├── references/           # Loaded on demand
│   ├── api-docs.md
│   └── examples.md
├── scripts/              # Executable utilities
│   └── validate.sh
└── assets/               # Output resources
    └── template.html
```

**Monorepo (API + agent skills):**
```
project/
├── SKILL.md              # Root skill (the API itself)
└── skills/               # Agent skills that use the API
    ├── weather-helper/
    │   └── SKILL.md
    └── forecast-analyst/
        └── SKILL.md
```

### Step 6: Conversions (6)

Tell the user about available format conversions:

#### SKILL.md → MCP Tool Definitions

Each endpoint becomes an MCP tool. Tool name: `{skill-name}_{path}` (e.g., `weather-api_v1_current`).

```typescript
import { parseSkillMd, toMcpToolDefinitions } from '@402md/skillmd'
const manifest = parseSkillMd(content)
const tools = toMcpToolDefinitions(manifest)
// Each tool has: name, description (with price), inputSchema
```

#### SKILL.md → A2A Agent Card

Converts to Google A2A Protocol Agent Card v0.3.0:

```typescript
import { parseSkillMd, toAgentCard } from '@402md/skillmd'
const manifest = parseSkillMd(content)
const card = toAgentCard(manifest)
// Serve at GET /.well-known/agent-card.json
```

Mapping:
| SKILL.md | Agent Card |
|----------|------------|
| `name` | `humanReadableId` |
| `displayName` \|\| `name` | `name` |
| `description` | `description` |
| `base_url` | `url` |
| `version` | `agentVersion` |
| `author` | `provider.name` |
| `endpoints[]` | `skills[]` |
| `payment` | `authSchemes[{ scheme: "x402" }]` |

#### SKILL.md → OpenAPI 3.0

Each endpoint becomes a path operation. A 402 response is added with the price.

```typescript
import { parseSkillMd, toOpenAPI } from '@402md/skillmd'
const manifest = parseSkillMd(content)
const spec = toOpenAPI(manifest)
```

#### OpenAPI → SKILL.md

Convert an existing OpenAPI spec into a SKILL.md:

```typescript
import { generateFromOpenAPI } from '@402md/skillmd'
const manifest = generateFromOpenAPI(openApiSpec, {
  networks: ['stellar'],
  asset: 'USDC',
  payTo: 'GABC...XYZ'
}, {
  pricingMap: {
    'POST /v1/scrape': '0.005',
    'POST /v1/screenshot': '0.01'
  }
})
```

### Step 7: Explain Publishing Options

Tell the user about these publishing/discovery channels (13):

1. **GitHub Repository** (for skills.sh):
   - Create a public repo with the SKILL.md at the root or inside `skills/{skill-name}/`
   - Anyone can install with: `npx skills add {owner}/{repo}`
   - It will appear on https://skills.sh for discovery
   - Initialize with: `npx skills init {name}`

2. **npm Package**:
   - Include SKILL.md at the package root
   - `npm install @{scope}/{skill-name}`

3. **402.md Marketplace**:
   - Register at https://402.md/marketplace
   - Discoverable via the `@402md/mcp` → `search_skills` tool

4. **Direct URL**:
   - Host SKILL.md at `https://yourdomain.com/SKILL.md`
   - Agents can fetch and parse it directly

5. **A2A Discovery**:
   - Serve `GET /.well-known/agent-card.json` (converted from SKILL.md)
   - Compatible with Google A2A protocol

6. **File System**:
   - Place in `./SKILL.md` or `./skills/*/SKILL.md`
   - Agents discover locally

### Step 8: Security Recommendations (14)

Always remind the user:

- **Payment amounts**: Agents SHOULD enforce budget limits before making payments
- **Private keys**: MUST be stored locally (e.g., `~/.402md/wallet.json` with mode `0600`). Private keys MUST NOT be transmitted.
- **Facilitator trust**: The facilitator settles payments but never holds funds. Settlement is atomic and on-chain verifiable.
- **Input validation**: Parsers MUST validate all fields before use. Untrusted SKILL.md files SHOULD be validated with `validateSkill()` before execution.
- **URL handling**: `base_url` and endpoint paths MUST be sanitized before constructing request URLs.

---

## Quick Reference Tables

### Skill Type Reference (3.3)

| Type | When to use | Endpoints required? | Payment required? |
|------|-------------|--------------------|--------------------|
| `API` | RESTful API with paid endpoints | Yes | Yes |
| `SAAS` | SaaS product with API access | Yes | Yes |
| `PRODUCT` | Digital product (dataset, model) | Yes | Yes |
| `SERVICE` | Human-in-the-loop or async | Yes | Yes |
| `SUBSCRIPTION` | Recurring access model | Yes | Yes |
| `CONTENT` | Static content (docs, reports) | Yes | Yes |
| `SKILL` | Pure agent instruction | No | No |

### Payment Network Reference (3.5)

| Network | Use for | Address format |
|---------|---------|---------------|
| `stellar` | Production — Stellar mainnet | G-address (56 chars) |
| `base` | Production — Base L2 (Coinbase) | 0x-address (42 chars) |
| `stellar-testnet` | Testing — free testnet USDC | G-address |
| `base-sepolia` | Testing — Base Sepolia testnet | 0x-address |

### Pricing Guidelines

| Service type | Typical price range |
|-------------|-------------------|
| Simple data lookup | $0.001 - $0.005 |
| Content extraction | $0.005 - $0.02 |
| AI/ML inference | $0.01 - $0.10 |
| Image generation | $0.02 - $0.20 |
| Complex processing | $0.05 - $0.50 |

Price low enough that agents don't hesitate, high enough to cover your costs.
USDC has 7 decimals on Stellar, 6 on Base.

### Anthropic Compatibility Matrix (11)

| Feature | Anthropic Skills | SKILL.md v2 | Status |
|---------|-----------------|-------------|--------|
| `name` | Required | Required | **Compatible** |
| `description` | Required (triggers) | Required (any) | **Superset** |
| `version` | Optional | Optional | **Compatible** |
| `license` | Optional | Optional | **Compatible** |
| `allowed-tools` | Optional | Optional | **Compatible** |
| `payment` | Not supported | Optional/Required | **Extension** |
| `endpoints` | Not supported | Optional/Required | **Extension** |
| `base_url` | Not supported | Conditional | **Extension** |
| `type` | Implicit (skill) | Explicit enum | **Extension** |
| Body as instructions | Yes | Yes | **Compatible** |
| `references/` dir | Supported | Supported | **Compatible** |
| `scripts/` dir | Supported | Supported | **Compatible** |

**Any valid Anthropic Claude Code skill is a valid SKILL.md.** The 402md parser
accepts Anthropic skills without modification.

---

## Important Rules

1. **Always show the full SKILL.md** to the user after generating it
2. **Always validate** before recommending publication
3. **Include concrete request/response examples** in every endpoint section — agents CANNOT use abstract descriptions (4.3)
4. **Include the x402 payment section** — no LLM has been trained on x402, the body is how agents learn (4.2)
5. **Use trigger phrases** in the description for Claude Code auto-invocation (3.1.1)
6. **Test with testnet first** — recommend `stellar-testnet` or `base-sepolia` for development
7. **Suggest a sandbox URL** if the user can provide a free test endpoint
8. **Respect body length guidelines** — more detail is better than less (4.7)
9. **Mention all 3 profiles** if the user is unsure which to use (5)
10. **Offer conversions** — MCP, A2A, OpenAPI are all available (6)
11. **Include security reminders** — budget limits, key storage, input validation (14)
