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
    container_name: github_wetter_bot_bern
    restart: unless-stopped
    environment:
      - GIT_USER=your_github_username
      - GIT_TOKEN=your_personal_access_token
      - REPO=weather-tracker
    command: >
      /bin/sh -c "
      apk add --no-cache git curl tzdata &&
      cp /usr/share/zoneinfo/Europe/Zurich /etc/localtime &&
      rm -rf /repo &&
      git clone https://$${GIT_USER}:$${GIT_TOKEN}@github.com/$${GIT_USER}/$${REPO}.git /repo &&
      cd /repo &&
      git config user.name 'Zeyrox Weather Bot' &&
      git config user.email '141281496+Zeyrox77@users.noreply.github.com' &&
      while true; do
        HOUR=$$(date +'%H') &&
        if [ \"$$HOUR\" -eq 12 ]; then
          DELAY=$$(awk 'BEGIN{srand(); print int(rand() * 7200)}') &&
          echo \"It is noon! Waiting $$DELAY seconds before pushing...\" &&
          sleep $$DELAY &&

          DATUM=$$(date +'%Y-%m-%d %H:%M') &&
          WETTER=$$(curl -s 'wttr.in/Bern?format=%t+%C&lang=en') &&
          echo \"- $$DATUM: The weather in Bern is $$WETTER\" >> weather_log.md &&
          git add weather_log.md &&
          git commit -m \"Automatic weather update: $$DATUM\" || echo 'No new changes.' &&
          git push &&
          echo 'Successfully pushed to GitHub!' &&

          sleep 72000;
        else
          echo \"Current hour: $$HOUR. Waiting for the noon window...\" &&
          sleep 1800;
        fi
      done
      "
```

### Setup

1. **Clone this repository** or create a new one named `weather-tracker` on GitHub.
2. Generate a **GitHub Personal Access Token** with `repo` write permissions.
3. Fill in your credentials in the `environment` section of the `docker-compose.yml`.
4. Deploy via Portainer or run manually:

```bash
docker compose up -d
```

The container will start, set the timezone to `Europe/Zurich`, clone the repository, and begin its daily loop immediately.

---

## ⚙️ Technical Details

| Detail | Value |
|---|---|
| Base image | `alpine:latest` |
| Timezone | `Europe/Zurich` (CET/CEST) |
| Weather source | [wttr.in](https://wttr.in/Bern) |
| Update schedule | Daily ~12:00, with random ±2h offset |
| Restart policy | `unless-stopped` |

---

## 🔑 Required GitHub Token Permissions

When generating your Personal Access Token (PAT), the following scope is required:

- `repo` — Full control of private/public repositories (for push access)

> ⚠️ Never commit your token directly into the `docker-compose.yml`. Use environment variables or a `.env` file.

---

## 📊 Weather Log

See [`weather_log.md`](./weather_log.md) for the full archive of daily weather entries.

---

## 📜 License

This project is for personal use and learning purposes. Feel free to fork and adapt it for your own city or use case.

---

*Built with ☕ and a love for self-hosting — running 24/7 on a home Ubuntu server.*
