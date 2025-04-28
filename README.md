📣 Discord Notify Action
Send beautiful Discord notifications from your GitHub Actions workflows when a build succeeds, fails, or is cancelled!

This Action posts an embedded message to your Discord server with clear statuses, colors, timestamps, and links to the build logs.

✨ Features
✅ Green Success message

❌ Red Failure message

⚠️ Yellow Cancelled message

📎 Clickable link to the GitHub Action run

🕒 Timestamp of build

👤 Actor and branch included in footer

🛡️ Simple and secure (uses GitHub Secrets for the webhook)

📥 Inputs

Name	Description	Required	Default
status	Build status (success, failure, cancelled)	No	failure
webhook	Discord Webhook URL	Yes	N/A
🚀 Usage
1. Add your Discord webhook URL as a GitHub Secret
Go to your Discord channel → Edit Channel → Integrations → Create Webhook → Copy Webhook URL.

Add it to your GitHub repo under Settings → Secrets → Actions as DISCORD_WEBHOOK_URL.

2. Example Workflow
yaml
Copy
Edit
name: Build and Notify

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: windows-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Build and Test
        run: |
          dotnet build YourSolution.sln
          dotnet test YourTests.sln

    finally:
      - name: Notify Discord
        uses: your-org/discord-notify-action@v1
        with:
          status: ${{ job.status }}
          webhook: ${{ secrets.DISCORD_WEBHOOK_URL }}
🔖 Example Discord Message
✅ Build Succeeded
All checks passed successfully! 🎉
(Link to view logs)
Triggered by @username on main
🕒 Timestamp shown automatically

📌 Notes
This action automatically formats the message color and content based on the job status.

If status is not passed, it defaults to failure to avoid silent failures.

Works on Linux, Windows, and macOS runners.

📄 License
MIT License

🎯 Quick Install
yaml
Copy
Edit
- uses: your-org/discord-notify-action@v1
  with:
    status: ${{ job.status }}
    webhook: ${{ secrets.DISCORD_WEBHOOK_URL }}
🧙‍♂️ That's it!
Now your GitHub Actions will talk directly to Discord like a boss.
Get notified instantly on success, failure, or cancellation 🚀


