# aptos-cli-choco-publish

This repository automatically packages and publishes the [Aptos CLI](https://github.com/aptos-labs/aptos-core) to [Chocolatey](https://community.chocolatey.org/packages/aptos).

---

## Two Workflows — Which One Should I Use?

There are two GitHub Actions workflows in this repository. They work together: one watches for new releases, the other does the actual publishing.

### 1. `check-update.yaml` — Automated Version Check (runs itself)

| Property | Detail |
|----------|--------|
| **Triggered by** | Automatically every day at 02:00 UTC, or manually via **Actions → Check Aptos CLI update → Run workflow** |
| **What it does** | Looks up the latest stable Aptos CLI release in `aptos-labs/aptos-core`, then checks whether that version already exists on Chocolatey. If it does **not** exist yet, it automatically dispatches `cli-publish.yaml` to publish it. |
| **Do I need to run this manually?** | **No** — it runs on a daily schedule. You would only trigger it manually if you want to force an immediate check without waiting for the next scheduled run. |

### 2. `cli-publish.yaml` — Chocolatey Publisher (the real worker)

| Property | Detail |
|----------|--------|
| **Triggered by** | Automatically by `check-update.yaml` when a new version is detected, **or** manually via **Actions → Checkout Aptos CLI Version → Run workflow** (you must supply the version, e.g. `v5.1.0`) |
| **What it does** | Clones `aptos-labs/aptos-core` at the specified release tag, downloads the Windows binary, computes its checksum, packs a `.nupkg`, and pushes it to Chocolatey. |
| **Do I need to run this manually?** | **Only** if you want to publish a specific version right now without waiting for the daily check, or if you need to re-publish a version that the automatic check missed. |

---

## Summary

```
Daily cron (02:00 UTC)
        │
        ▼
check-update.yaml
  ├── Latest aptos-cli version already on Chocolatey?
  │     YES → "No update needed", workflow ends.
  │
  └──   NO  → dispatches cli-publish.yaml with the new version
                        │
                        ▼
              cli-publish.yaml
              builds & pushes the Chocolatey package
```

**Normal operation:** you don't need to touch either workflow. `check-update.yaml` runs every day and calls `cli-publish.yaml` automatically whenever a new Aptos CLI release appears.

**Manual publish:** if you need to publish a specific version immediately, run **`cli-publish.yaml`** directly from the Actions tab and enter the version number.

---

## Required Secret

| Secret | Description |
|--------|-------------|
| `CHOCO_API_KEY` | Your Chocolatey API key, used by `cli-publish.yaml` to push the package. Set it under **Settings → Secrets and variables → Actions**. |