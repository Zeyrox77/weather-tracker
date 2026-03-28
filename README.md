# 🌦️ Weather Tracker – Bern, Switzerland

> Automated daily weather logger for Bern. Powered by a self-hosted Docker container running on Ubuntu & Portainer.

---

## 📋 What it does

Every day around **noon (12:00 CET)**, a lightweight Docker container wakes up, fetches the current weather in Bern via [wttr.in](https://wttr.in), and appends a new entry to [`weather_log.md`](./weather_log.md). The result is then committed and pushed to this repository automatically — no manual input needed.

To keep the activity organic and avoid always pushing at the exact same second, the bot waits a **random delay of up to 2 hours** after noon before making the push.

---

## 📁 Repository Structure

```
weather-tracker/
├── README.md          ← You are here
└── weather_log.md     ← The growing daily weather log
```

---

## 📝 Log Format

Each entry in `weather_log.md` follows this format:

```
- 2026-03-27 12:24: The weather in Bern is +12°C Partly cloudy
```

---

## 🐳 How it works (Docker)

The bot runs as a Docker service defined in a `docker-compose.yml` file on a local Ubuntu server managed via **Portainer**.

### docker-compose.yml

```yaml
version: '3.8'
services:
  wetter-bot:
    image: alpine:latest
    container_name: github_weather_bot_bern
    restart: unless-stopped
    environment:
      # ENTER YOUR DATA HERE:
      - GIT_USER=YOUR_GITHUB_USERNAME
      - GIT_TOKEN=YOUR_PERSONAL_ACCESS_TOKEN
      - REPO=YOUR_REPO_NAME
    command: >
      /bin/sh -c "
      apk add --no-cache git curl tzdata &&
      cp /usr/share/zoneinfo/Europe/Zurich /etc/localtime &&
      rm -rf /repo &&
      git clone https://$${GIT_USER}:$${GIT_TOKEN}@github.com/$${GIT_USER}/$${REPO}.git /repo &&
      cd /repo &&
      git config user.name 'YOUR_BOT_NAME' &&
      git config user.email 'YOUR_GITHUB_NOREPLY_EMAIL' &&
      while true; do
        HOUR=$$(date +'%H') &&
        if [ \"$$HOUR\" -eq 12 ]; then
          DELAY=$$(awk 'BEGIN{srand(); print int(rand() * 7200)}') &&
          echo \"It is noon! Waiting $$DELAY seconds before pushing...\" &&
          sleep $$DELAY &&
          git pull --rebase &&

          DATE=$$(date +'%Y-%m-%d %H:%M') &&
          WEATHER=$$(curl -s 'wttr.in/Bern?format=%t+%C&lang=en') &&
          echo \"- $$DATE: The weather in Bern is $$WEATHER\" >> weather_log.md &&
          git add weather_log.md &&
          git commit -m \"Automatic weather update: $$DATE\" || echo 'No new changes.' &&
          git push &&
          echo 'Successfully pushed to GitHub!' &&

          echo 'Push done. Going to sleep until tomorrow...' &&
          sleep 72000;
        else
          echo \"Current hour: $$HOUR. Waiting for the noon window...\" &&
          sleep 1800;
        fi
      done
      "
```

### Setup

1. **Create a new repository** on GitHub (e.g. `weather-tracker`).
2. Generate a **GitHub Personal Access Token** with `repo` write permissions — see [GitHub Docs](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens).
3. Fill in your credentials in the `environment` section of the `docker-compose.yml`:

| Placeholder | Replace with |
|---|---|
| `YOUR_GITHUB_USERNAME` | Your GitHub username |
| `YOUR_PERSONAL_ACCESS_TOKEN` | Your GitHub PAT |
| `YOUR_REPO_NAME` | Your repository name (e.g. `weather-tracker`) |
| `YOUR_BOT_NAME` | Display name for Git commits (e.g. `My Weather Bot`) |
| `YOUR_GITHUB_NOREPLY_EMAIL` | Your GitHub no-reply email (e.g. `12345678+username@users.noreply.github.com`) |

4. Deploy via Portainer or run manually:

```bash
docker compose up -d
```

The container will start, set the timezone to `Europe/Zurich`, clone the repository, and begin its daily loop.

---

## ⚙️ Technical Details

| Detail | Value |
|---|---|
| Base image | `alpine:latest` |
| Timezone | `Europe/Zurich` (CET/CEST) |
| Weather source | [wttr.in](https://wttr.in/Bern) |
| Update schedule | Daily ~12:00, with random 0–2h offset |
| Restart policy | `unless-stopped` |

---

## 🔑 Required GitHub Token Permissions

When generating your Personal Access Token (PAT), the following scope is required:

- `repo` — Full control of private/public repositories (for push access)

> ⚠️ Never push your `docker-compose.yml` to a public repository — it contains your personal access token. Keep it only on your local server.

---

## 📊 Weather Log

See [`weather_log.md`](./weather_log.md) for the full archive of daily weather entries.

---

## 📜 License

This project is for personal use and learning purposes. Feel free to fork and adapt it for your own city or use case.

---

*Built with ☕ and a love for self-hosting — running 24/7 on a home Ubuntu server.*
