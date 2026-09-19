# My Narrative — Version History & Deployment Tracker

## Tagging Convention

Every push/deploy MUST follow this pattern:

```
v{MAJOR}.{MINOR}.{PATCH}-{stage}
```

### Version Numbers
- **MAJOR** — Breaking changes, new architecture, database migrations
- **MINOR** — New features, new endpoints, new integrations
- **PATCH** — Bug fixes, config changes, dependency updates

### Stage Tags (appended after version)
- `-dev` — Local development, not deployed
- `-staging` — Deployed to staging/preview
- `-prod` — Deployed to production
- `-rollback` — Reverted to previous version

### Commit Message Convention
```
{type}: {description}

type:
  feat     — New feature
  fix      — Bug fix
  security — Security hardening
  docs     — Documentation only
  refactor — Code restructure (no behavior change)
  perf     — Performance improvement
  test     — Adding tests
  chore    — Build, config, dependency updates
  MILESTONE — Major milestone (use sparingly)
```

---

## Deployment Log

### v2.2.0-prod — Fashion Intelligence + Look Carousel (Sep 20, 2026)

**Feature release: multi-stage recommendation engine, price intelligence, swipeable look carousel, geolocation weather, pinned VTON restored.**

| Component | Version | URL | Status |
|---|---|---|---|
| Fly.io API | v2.2.0 | `https://drishti-api.fly.dev` | Running |
| Fly.io Secondary | v2.2.0 | `https://drishti-api-v2.fly.dev` | Running |
| Shopify Theme | v2.2.0 | `https://mynarrative.store` | Live |
| Widget | v2.2.0 | Floating widget on store | Live |

**Backend (api/):**
- `feat: fashion intelligence engine — slot queries, attribute enrichment, weighted match scoring`
- `feat: price intelligence layer — live percentile budget bands, shopping levels, algorithmic brand ranking`
- `feat: multi-stage recommendation pipeline with diversity selection and look labels`
- `feat: geolocation weather — /api/weather/current accepts lat/lon`
- `feat: VTON seed pass-through for artifact re-rolls`
- `fix: restore pinned VTON model version (latest produces extra-limb artifacts)`
- `fix: client-side price filtering — SerpApi min/max_price returns zero with gl=in`
- `fix: keep production machine warm — auto-stop disabled, single machine`

**Theme (theme/):**
- `feat: magic-first wizard v5 — landing, photo, occasion, look carousel`
- `feat: swipeable look carousel — one VTON per screen, Save Outfit, Regenerate`
- `feat: Style DNA personalization — styles, occasions, total-look budget, brands, digital closet`
- `feat: returning-user stylist — wardrobe / mixed / new-outfit modes`
- `feat: geolocation weather request during look creation`
- `feat: compare prices & shop with per-retailer rows and best-price badges`
- `fix: remove location bar from results; weather is a silent ranking signal`
- `fix: garment candidate rail in consultant widget`

**Secondary API (api-secondary/):**
- `feat: marketplace scrapers — Amazon + Flipkart with Google Shopping parser`
- `fix: add httpx dependency; region ams`

**Critical pinned artifacts:**
- VTON model version: `0e122964dd5d7fce695da14e9206f8dd48c0c5595ecb7e3cf1a4078701fb2665` (do not change)

---

### v2.1.0-prod — VTON LOCKED (Sep 15, 2026)
**MILESTONE: VTON system permanently frozen.**

| Component | Version | URL | Status |
|---|---|---|---|
| Fly.io API | v2.1.0 | `https://drishti-api-v2.fly.dev` | Running |
| Vercel API | v2.1.0 | `https://drishti-api-blond.vercel.app` | Ready |
| Shopify Theme | v2.1.0 | `https://mynarrative.store` | Live |
| Widget | v2.1.0 | Floating widget on store | Live |

**Commits:**
- API: `b048cf7` docs: VTON_LOCKED.md
- API: `650621f` MILESTONE: Lock VTON system
- API: `5a1cd05` fix: migrate urllib→requests, lazy env vars
- Theme: `e1426f8` MILESTONE: Lock VTON
- Theme: `239fc4c` Fix VTON: 3 frontend bugs

**Tags:**
- API: `v2.1.0-vton-locked`
- Theme: `v2.1.0-vton-locked`, `v4.0-vton-working`, `v4.1-vton-ux-polish`

**What's locked:**
- VTON model: `prunaai/p-image-try-on` (5.8s)
- Fallback: `cuuupid/idm-vton` (category-specific)
- Patterns: data URI download, garment extraction, image proxy, 429 retry
- See `VTON_LOCKED.md` for full details

---

### v2.0.0-prod — B2B + Security (Sep 14, 2026)
**B2B Cross-Brand Syndicate platform with hardened security.**

**Commits:**
- `b447023` feat: B2B Cross-Brand Syndicate platform
- `4f35274` security: fix module-level env vars, health check auth, RLS
- `d44b2b8` feat: World-class recommendation engine — 7-stage pipeline

---

### v1.0.0-prod — Initial Deployment (Sep 8, 2026)
**First working production deployment.**

**Commits:**
- `a6bd611` initial deployment

---

## Future Releases

1. Commit with type prefix and update the affected component
2. Deploy preview where applicable (Vercel: `vercel --yes`)
3. Test on the preview URL
4. Deploy production (Fly: `fly deploy`, Vercel: `vercel --prod --yes`, Shopify: asset push)
5. Tag: `git tag -a v{X}.{Y}.{Z}-prod -m "description"`
6. Push: `git push origin main --tags`
7. Update this VERSION.md with the new entry

### Recommended Next Versions
- `v2.3.0` — Brand catalog expansion + affiliate monetization
- `v2.4.0` — Card-offer price engine (effective vs listed price)
- `v3.0.0` — Major: user accounts, style DNA sync, cross-device profiles
