# Discord Notify Action

A composite GitHub Action that posts the status of a build, deploy or publish to a Discord channel through a webhook.

The message is a Discord embed: what ran and how it ended, a link to the run's logs, the repository and branch, who triggered it and a timestamp.

## Features

- Green message for a success, red for a failure, yellow for a cancelled run, orange for a successful prod deploy
- Link to the GitHub Actions run
- UTC timestamp
- Repository and Branch fields (plus Environment and Version when set); the footer names who triggered the run
- The webhook comes from a GitHub secret

## Quick Install

```yaml
- uses: LawlGames/discord-notify-action@7f3455a745620b6d6cbd263c7c84826fd5de0107 # v3
  if: always()
  with:
    status: ${{ job.status }}
    webhook: ${{ secrets.DISCORD_WEBHOOK_URL }}
```

A deploy:

```yaml
- uses: LawlGames/discord-notify-action@7f3455a745620b6d6cbd263c7c84826fd5de0107 # v3
  if: always()
  with:
    status: ${{ job.status }}
    webhook: ${{ secrets.DISCORD_WEBHOOK_URL }}
    title: Deploy
    environment: ${{ inputs.environment }}
    version: ${{ inputs.image_version }}
```

The step must run with `if: always()`. A step after a failed step is skipped by default, so without it a failure or a cancellation is never sent. In a separate job, use `needs:` with `if: always()` and pass `${{ needs.<job>.result }}` as `status`.

## Inputs

| Name | Description | Required | Default |
|---|---|---|---|
| `status` | Job status: `success`, `failure` or `cancelled`. Any other value counts as failed | No | `failure` |
| `webhook` | Discord webhook URL. When empty the step warns and sends nothing | Yes | |
| `title` | What ran: Build, Deploy, Site publish | No | `Build` |
| `environment` | Environment it ran against (`dev`, `prod`). Goes in the title ("Deploy to PROD succeeded") and a field. A success on an environment that starts with `prod` gets an orange bar | No | `""` |
| `version` | Version built or deployed, shown as a field | No | `""` |
| `details` | Extra Discord Markdown under the status line | No | `""` |

## Versions

`title`, `environment`, `version` and `details` are the v3 inputs. There is no `v3` tag: callers pin the commit `7f3455a745620b6d6cbd263c7c84826fd5de0107` with a `# v3` comment. The tags are `v1` and `v2`, which take `status` and `webhook` only. lg-issues and nuget-pulumi-core use `@v2`.

v2 inputs work unchanged on v3. The message text differs ("Build succeeded", "All steps passed."), and an empty webhook skips the message instead of failing the step.

## Usage

### 1. Add your Discord webhook URL as a GitHub secret

In Discord, go to the channel, then Edit Channel > Integrations > Create Webhook > Copy Webhook URL.

Add it to the GitHub repo under Settings > Secrets and variables > Actions as `DISCORD_WEBHOOK_URL`.

### 2. Example workflow

```yaml
name: Build and Notify

on:
  push:
    branches: [master]
  pull_request:
    branches: [master]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Build and Test
        run: |
          dotnet build YourSolution.sln
          dotnet test YourSolution.sln

      - name: Notify Discord
        if: always()
        uses: LawlGames/discord-notify-action@7f3455a745620b6d6cbd263c7c84826fd5de0107 # v3
        with:
          status: ${{ job.status }}
          webhook: ${{ secrets.DISCORD_WEBHOOK_URL }}
```

## What the message shows

- **Title**: ✅ `<title> succeeded`, ⚠️ `<title> cancelled` or ❌ `<title> failed`. With `environment` set it reads `<title> to <ENVIRONMENT>`, the environment in upper case: "Deploy to PROD succeeded".
- **Description**: "All steps passed. 🎉", "The workflow was cancelled." or "There were errors. ❗". Then `details` on its own line when set, then a `[View Logs]` link to the run.
- **Fields**: Repository and Branch always, Environment and Version when set.
- **Footer**: "Triggered by `<actor>`" and a UTC timestamp.
- **Colour**: green for a success, yellow for cancelled, red for failed. A success whose `environment` starts with `prod` or `Prod` is orange.

Only `success` and `cancelled` are recognised. Any other `status` (`skipped`, for example) is reported as failed.

## Example Discord Message

A prod deploy, with an orange bar:

```
✅ Deploy to PROD succeeded
All steps passed. 🎉

[View Logs]

Repository           Environment   Version   Branch
LawlGames/lg-work    prod          1.2.3     master

Triggered by <actor> • <time of the run>
```

## Used in LawlGames

The webhook is each repo's `DISCORD_WEBHOOK_URL` secret.

- **Service deploys.** The `deploy` job of `deploy.yml` in lg-studio, lg-work, lg-telemetry, lg-updates, lg-creators, lg-competitive-insights and lg-lambda-template ends with the deploy step above under `if: always()`: `title: Deploy`, `environment: ${{ github.event.inputs.environment }}` and `version: ${{ github.event.inputs.image_version }}`.
- **Site publishes.** The kit's reusable publish workflow (`lg-studio-kit/.github/workflows/publish.yml`) has a `notify` job that posts "Site publish" with the published and unchanged sites as `details`. It sends only when the caller passes the secret. Game and site repos pass it from their `studio-publish.yml`; unset means no message.

  ```yaml
  secrets:
    DISCORD_WEBHOOK_URL: ${{ secrets.DISCORD_WEBHOOK_URL }}
  ```

- **Codex publish.** ashlit's `codex.yml` posts "Codex publish (GitHub Pages)" with the page URL as `details`.

## Notes

- The colour and text of the message follow the job status.
- If `status` is not passed, it defaults to `failure`, so a missing status is never reported as a success.
- An empty or unset `webhook` skips the message with a warning and never fails the job.
- A Discord error (a deleted webhook, a rate limit) doesn't fail the step either. Check the step log if a message is missing.
- `title`, `details` and the other values are JSON-escaped, so quotes and multi-line `details` are safe.
- Works on Linux, Windows, and macOS runners.

## License

MIT License
