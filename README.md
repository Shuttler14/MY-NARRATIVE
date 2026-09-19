# MY NARRATIVE

**AI-powered fashion stylist platform** — see yourself in real outfits, styled by AI, shoppable across India's fashion marketplaces.

Production: [mynarrative.store](https://mynarrative.store) · API: `https://drishti-api.fly.dev`

---

## What This Is

MY NARRATIVE turns a single photo into a personalized styling experience:

1. **User uploads a photo** → AI analyzes body profile
2. **Picks an occasion** (Everyday / Work / Date / Party / Travel / Wedding)
3. **AI generates complete looks** — tops, bottoms, shoes, accessories from live marketplace inventory (Amazon, Myntra, Flipkart, AJIO, Meesho)
4. **Virtual Try-On** renders the look on the user's own photo
5. **Compare prices & shop** — direct product links with live prices

### Product Principles

- **Anonymous users experience the magic. Registered users experience the intelligence.**
- First-time visitors get a full look with zero signup — personalization is offered *after* the wow moment
- Returning users flow straight into the stylist: occasion → style mode → budget → brands → look
- The recommendation engine owns intelligence, not inventory: Google Shopping is the discovery layer, our fashion knowledge graph decides what actually suits the user

---

## Repository Structure

```
MY-NARRATIVE/
├── api/              FastAPI backend — recommendation engine + VTON + pricing
├── api-secondary/    Flask secondary API — marketplace scrapers + stylist pipeline
├── theme/            Shopify theme — including the AI Stylist Wizard v5
│   └── assets/       MN-stylist-wizard-v4.js (readable) + .min.js (deployed build)
└── VERSION.md        Deployment history & versioning convention
```

### `api/` — Core Backend (FastAPI)

Deployed to Fly.io as `drishti-api` (region: sin).

| Service | Purpose |
|---|---|
| `api/services/fashion_engine.py` | Fashion intelligence — slot queries, attribute enrichment, weighted match scoring, diversity selection |
| `api/services/price_intelligence.py` | Dynamic price bands from live distributions, shopping levels, brand ranking |
| `api/services/marketplace_reco.py` | Multi-stage recommendation pipeline |
| `api/services/price_scraper.py` | Google Shopping via SerpApi + marketplace scrapers |
| `api/services/vton_replicate.py` | Virtual try-on via Replicate (`prunaai/p-image-try-on`, **pinned version**) |
| `api/services/garment_extract.py` | rembg garment isolation for marketplace photos |
| `api/routers/reco.py` | `/api/reco/outfits`, `/api/reco/brands` |
| `api/routers/vton.py` | `/api/vton/try-on`, `/api/vton/upload-person`, `/api/vton/extract-garment` |
| `api/routers/weather.py` | `/api/weather/current` (city or geolocation lat/lon) |

### `api-secondary/` — Secondary API (Flask)

Deployed to Fly.io as `drishti-api-v2`. Contains marketplace scrapers (Amazon, Flipkart) and the legacy stylist pipeline used for brand search.

### `theme/` — Shopify Theme

The stylist experience lives in `assets/MN-stylist-wizard-v4.js`:

- **New-user flow**: Landing → Photo → Occasion → Creating theater → **Look carousel** → optional personalization (Style DNA, digital closet)
- **Returning-user flow**: Occasion → Style mode (my wardrobe / mixed / new) → Total-look budget → Brand preference → Look
- **Look carousel**: one VTON per screen, swipeable, Save Outfit, Regenerate (re-rolls VTON seed), items rail, Compare Prices & Shop
- **Geolocation weather**: permission asked at "Create my look" — feeds temperature/conditions into the recommendation payload for fabric-aware styling

**Deploy**: push `assets/MN-stylist-wizard-v4.min.js` content as the theme's `MN-stylist-wizard-v4.js` asset, plus `MN-stylist-wizard-v4.css`, via Shopify CLI or admin.

---

## Recommendation Pipeline

```
USER INTENT (occasion, style, gender, budget, weather, body)
        │
        ▼
SLOT QUERY GENERATION ── "women formal shirt", "men chinos", "heels"
        │
        ▼
GOOGLE SHOPPING (SerpApi) — 4 parallel slot queries, cached 24h
        │
        ▼
HARD FILTERS ── gender, kids, price band (client-side, SerpApi native price
                filters break gl=in queries — see commit history)
        │
        ▼
FASHION ENRICHMENT ── category, colour family, fabric, fit, pattern, VTON-safety
        │
        ▼
MATCH SCORING ── style 25% · occasion 20% · fit 15% · colour 10% · budget 10%
                 quality 10% · popularity 5% · brand 5%
        │
        ▼
DIVERSITY SELECTION ── slot quotas, near-duplicate suppression, look labels
        │
        ▼
VTON ── pinned prunaai/p-image-try-on version (seed-controlled regeneration)
```

### Budget Intelligence

Static price slabs are replaced with **live distributions**:

- **Shopping levels** (Value / Contemporary / Premium / Luxury) map to percentile windows of the *current* result pool — "luxury" for party dresses ≠ "luxury" for sneakers
- **Continuous budgets** — slider ranges and "around ₹X ± tolerance" both supported
- **Quick picks** — budget chips generated from the live price percentiles
- **Brand ranking** — algorithmic (relevance, price affinity, quality, popularity, coverage), never hardcoded to price tiers

---

## Critical Constraints (Do Not Break)

1. **VTON model version is pinned** (`0e122964...`). The "latest" version of `prunaai/p-image-try-on` produces artifacts (extra limbs). Never switch back to the model-latest endpoint.
2. **Images for VTON are downloaded as data URIs** before sending to Replicate — marketplace CDNs and Google Shopping thumbnails are not reliably fetchable by Replicate.
3. **SerpApi `min_price`/`max_price` return zero results with `gl=in`** — price filtering happens client-side after fetching.
4. **Never commit `.env`** — all secrets live in Fly.io secrets and Vercel env vars.

---

## Local Development

```bash
# Backend
cd api
pip install -r requirements.txt
uvicorn api.main:app --reload

# Required env vars (see api/.env.example)
REPLICATE_API_TOKEN, SERPAPI_KEY, DATABASE_URL, OPENAI_API_KEY, R2_* credentials
```

## Deployment

| Component | Command |
|---|---|
| Main API | `cd api && fly deploy` (app: `drishti-api`) |
| Secondary API | `cd api-secondary && fly deploy` (app: `drishti-api-v2`) |
| Shopify theme | Push `theme/assets/*` via Shopify CLI |
| Vercel functions | `cd api && vercel --prod --yes` |

See [VERSION.md](VERSION.md) for the full deployment log and tagging convention.
