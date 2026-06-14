# Cloudflare Skills

This directory contains skills imported from the official Cloudflare Skills repository.

Source: <https://github.com/cloudflare/skills>

## Included Skills

- agents-sdk
- cloudflare-email-service
- cloudflare
- durable-objects
- sandbox-sdk
- turnstile-spin
- web-perf
- workers-best-practices
- wrangler

## Updating

### Linux / macOS

```bash
git clone --depth 1 https://github.com/cloudflare/skills.git temp-cloudflare-skills && for skill in agents-sdk cloudflare-email-service cloudflare durable-objects sandbox-sdk turnstile-spin web-perf workers-best-practices wrangler; do rm -rf "skills/cloudflare/$skill" && cp -R "temp-cloudflare-skills/skills/$skill" skills/cloudflare/; done && rm -rf temp-cloudflare-skills
```

### Windows (PowerShell)

```powershell
git clone --depth 1 https://github.com/cloudflare/skills.git temp-cloudflare-skills; @('agents-sdk','cloudflare-email-service','cloudflare','durable-objects','sandbox-sdk','turnstile-spin','web-perf','workers-best-practices','wrangler') | ForEach-Object { Remove-Item "skills/cloudflare/$_" -Recurse -Force -ErrorAction SilentlyContinue; Copy-Item "temp-cloudflare-skills/skills/$_" -Destination "skills/cloudflare" -Recurse }; Remove-Item temp-cloudflare-skills -Recurse -Force
```

These commands replace the local copies of the imported Cloudflare skills with the latest versions from the upstream repository.

## License

Each skill retains its original license and attribution from the upstream Cloudflare Skills repository.

See the upstream repository for the latest license information and documentation.

> NOTE:
> These skills are maintained by Cloudflare and are periodically synced from the upstream repository. Changes made locally may be overwritten during updates.
