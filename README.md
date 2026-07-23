# idleryu-announcement-v3

Static JSON backend for the 1dleryu X TikTok extension's announcement banner.
No framework, no build step — Vercel serves the `.json` files as-is.

## Deploy

```
npm i -g vercel   # skip if already installed
vercel --prod
```

Or push this folder to a GitHub repo and import it at vercel.com/new.

## Endpoints after deploy

- `https://<your-project>.vercel.app/announcement.json`
- `https://<your-project>.vercel.app/config.json`
- `https://<your-project>.vercel.app/changelog.json`
- `https://<your-project>.vercel.app/version.json`

`vercel.json` sets `Access-Control-Allow-Origin: *` so the extension can
fetch these from any context, and caches at the edge for 60s so a push
goes live almost immediately without you needing to purge anything.

If you deploy under a different project name, update
`ANNOUNCEMENT_BASE_URL` in the extension's `popup.js` and the matching
entry in `manifest.json`'s `host_permissions` to point at the new domain.

---

## File schemas

### `announcement.json`

| Field          | Type    | Notes |
|----------------|---------|-------|
| `enabled`      | bool    | Optional. `false` hides the banner entirely. |
| `id`           | string  | Unique per announcement. Used to remember dismissal — change it whenever you want a *new* announcement to show again even to users who dismissed an old one. |
| `type`         | string  | `"update_required"` renders a **non-dismissible** banner with no close button; it re-checks the installed version against `version.json`'s `minimumRequiredVersion` every time the popup opens and only disappears once that's satisfied. Any other value (e.g. `"info"`, `"feature"`) renders a normal dismissible banner with a close (×) button. |
| `badge`        | string  | Small pill label above the title. Omit/leave blank to hide it. |
| `headline`     | string  | Main title. |
| `subtext`      | string  | Body copy. |
| `versionFrom`  | string  | Optional. Shown as `"v3 → v4"` in the header if both `versionFrom`/`versionTo` are set. |
| `versionTo`    | string  | Optional, paired with `versionFrom`. |
| `showChangelog`| bool    | Optional. If `true` (and `type` isn't `update_required`), renders the `changelog.json` version list inside the card. |
| `cta.label`    | string  | Button text. Button is hidden if `cta.label`/`cta.url` are both missing and `config.json`'s `downloadUrl` isn't set. |
| `cta.url`      | string  | Button destination. Falls back to `config.json`'s `downloadUrl` if omitted. |
| `footnote`     | string  | Small text at the bottom of the card. Falls back to `config.json`'s `branding.studio` if omitted. |

### `config.json`

| Field                 | Notes |
|-----------------------|-------|
| `downloadUrl`         | Fallback CTA URL if `announcement.json`'s `cta.url` is absent. |
| `branding.name`       | Not currently rendered directly; reserved for future use. |
| `branding.studio`     | Fallback footnote text. |
| `theme.*`              | Applied live as CSS custom properties on the banner card (background, card, border, text, textMuted, textDim, accent, accentSoft, accentLine). Any hex color or `rgba()` string works. |
| `font.family`         | Google Font family name. Loaded on the fly via a `<link>` tag; falls back silently to the default UI font if it can't load. |
| `font.weights`        | Array of weights to request from Google Fonts (e.g. `[400, 700]`). |

### `changelog.json`

```json
{
  "versions": [
    { "version": "v4", "date": "2026-07-23", "changes": ["..."] }
  ]
}
```
Only entries with a non-empty `changes` array are rendered. Only shown when `announcement.json` sets `"showChangelog": true` and `type` isn't `"update_required"`.

### `version.json`

| Field                     | Notes |
|---------------------------|-------|
| `extension`               | Informational only. |
| `latestVersion`           | Informational only. |
| `previousVersion`         | Informational only. |
| `minimumRequiredVersion`  | Compared against the installed extension's manifest version (leading `v` stripped) to decide when an `update_required` announcement can stop showing. |
| `forceUpdate`             | Informational — the actual gating is driven by `announcement.json`'s `type` field, not this flag. |

### `vercel.json`

Deployment-only config (CORS headers + edge cache). Never fetched by the extension at runtime.
