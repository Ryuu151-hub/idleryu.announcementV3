# Deploy

```
npm i -g vercel   # skip if already installed
vercel --prod
```

Or push this folder to a GitHub repo and import it at vercel.com/new. No framework, no build step — Vercel serves it as static files automatically.

## Endpoints after deploy

- `https://<your-project>.vercel.app/announcement.json`
- `https://<your-project>.vercel.app/version.json`
- `https://<your-project>.vercel.app/config.json`
- `https://<your-project>.vercel.app/changelog.json`

`vercel.json` sets `Access-Control-Allow-Origin: *` so the extension can fetch these from any context, and caches at the edge for 60s so a push goes live almost immediately without you needing to purge anything.
