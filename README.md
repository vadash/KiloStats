<div align="center">

[![KiloStats Banner](https://capsule-render.vercel.app/api?type=waving&color=8bd11f&height=220&section=header&text=KiloStats&fontSize=90&fontColor=ffffff&animation=fadeIn&fontAlignY=38&desc=Community-driven%20hourly%20benchmarking%20of%20every%20free%20model%20on%20the%20Kilo%20AI%20Gateway&descSize=20&descAlignY=60&descAlign=50)](#)

[![CI](https://github.com/vadash/KiloStats/actions/workflows/benchmark.yml/badge.svg)](https://github.com/vadash/KiloStats/actions)
[![Models](https://img.shields.io/badge/models-free%20catalog-blue?style=flat-square)](https://api.kilo.ai/api/gateway/models)
[![License: MIT](https://img.shields.io/badge/license-MIT-yellow?style=flat-square)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat-square)](https://github.com/vadash/KiloStats/pulls)
[![Stars](https://img.shields.io/github/stars/vadash/KiloStats?style=flat-square&color=gold)](https://github.com/vadash/KiloStats/stargazers)

<br/>

> Community-driven hourly benchmarking of every free model on the Kilo AI Gateway. No servers, no API key required.

<br/>

**[🚀 View Live Dashboard](#) · [📖 Docs](#-quick-start) · [🤝 Contribute](#-contributing) · [💬 Discussions](https://github.com/vadash/KiloStats/discussions)**

</div>

---

## ✨ What is KiloStats?

KiloStats benchmarks the free models on the [Kilo AI Gateway](https://kilo.ai) once an hour with GitHub Actions and shows the results on a dashboard.

There's no hardcoded model list. Each run fetches whatever the gateway is currently offering for free from `https://api.kilo.ai/api/gateway/models` and benchmarks all of it. Kilo rotates that catalog roughly every minute, so the set drifts over time and the benchmark follows it.

The only real constraint is the gateway's 200 requests/hour per-IP free-tier limit, which the ~10-15 request per run stays well under. You don't need a key, though you can set one if you want a dedicated quota.

<div align="center">

| 🏎️ Hourly benchmarks | 📊 Interactive charts | 🔁 No infra | 🌍 Open source |
|:---:|:---:|:---:|:---:|
| Automatic via GitHub Actions | Response time, throughput, trends | Static site, free CI | Fork and self-host quickly |

</div>

---

## ⚡ Quick Start

### 1. Fork & Clone

```bash
git clone https://github.com/vadash/KiloStats.git
cd KiloStats
```

### 2. API key (optional)

For free models you don't need one. If you want a dedicated quota:

```bash
export KILO_API_KEY=your_optional_key_here
```

### 3. Deploy the dashboard

| Platform | Steps |
|----------|-------|
| **Cloudflare Pages** | Connect repo in [Cloudflare Pages](https://pages.cloudflare.com/) |
| **GitHub Pages** | Settings → Pages → Deploy from `main` |
| **Netlify / Vercel** | Connect repo for auto-deploy |

### 4. Run your first benchmark

In **Actions**, pick **"Benchmark Kilo Gateway Free Models"** and select **Run workflow**. The dashboard updates every hour.

---

## 📊 Dashboard Features

<div align="center">

| Tab | What you get |
|-----|-------------|
| **📊 Overview** | 5 animated KPI cards · success trend charts · top-10 speed & throughput bars · model reliability pills |
| **🏆 Leaderboard** | Composite score rankings · sortable columns · SVG sparklines · trend indicators (↑↓→) · provider chips |
| **🔬 Explorer** | Per-model deep dive · response time history chart · error breakdown donut · availability heatmap |
| **⏱ Timeline** | Filterable run history (All / 24h / 48h / 7d) · expandable run cards with full per-model detail |
| **⚔️ Compare** | Head-to-head overlay chart · win-rate stats · side-by-side metric comparison |

</div>

---

## 🤖 Benchmarked Models

There is no list to maintain. Each run calls `https://api.kilo.ai/api/gateway/models`, keeps everything with `isFree: true` that supports both `max_tokens` and `temperature`, and benchmarks that. The catalog rotates roughly every minute, so the exact set changes between runs.

The auto-router `kilo-auto/free` is included whenever it's free. Providers seen in the free tier so far:

| Provider | Example model id |
|----------|------------------|
| **Kilo Auto** | `kilo-auto/free` |
| **StepFun** | `stepfun/step-3.7-flash:free` |
| **InclusionAI** | `inclusionai/ling-3.0-flash:free` |
| **Poolside** | `poolside/laguna-s-2.1:free` |
| **Cohere** | `cohere/north-mini-code:free` |
| **NVIDIA** | `nvidia/nemotron-3-super-120b-a12b:free` |
| **OpenRouter** | `openrouter/free` |
| **KwaiPilot** | `kwaipilot/kat-coder-pro-v2.5:free` |

---

## 🏗️ How It Works

```
┌──────────────────── GitHub Actions (every hour) ──────────────────────┐
│                                                                               │
│   fetch_free_models() ── GET api.kilo.ai/api/gateway/models ──▶ queue    │
│                                                                               │
│   ┌─────────────────────┐        ┌─────────────────────┐                    │
│   │  Worker 1           │        │  Worker 2           │  (run in parallel) │
│   │  free model subset  │        │  free model subset  │                    │
│   └──────────┬──────────┘        └──────────┬──────────┘                    │
│              └──────────────┬───────────────┘                               │
│                    ┌────────▼────────┐                                       │
│                    │  Merge + commit │  → history.db updated in repo         │
│                    └─────────────────┘                                       │
└───────────────────────────────────────────────────────────────────────────── ┘
                                     │
                          ┌──────────▼──────────┐
                          │  Static Dashboard   │  rebuilds on each push
                          │  (Pages / Netlify)  │
                          └─────────────────────┘
```

Two parallel workers roughly halve the run time.

---

## 🛠️ Customization

<details>
<summary><b>Change the benchmark prompt</b></summary>

Edit `PROMPT` in `scripts/test_models.py`:
```python
PROMPT = "Your custom prompt here"
```
</details>

<details>
<summary><b>Add or remove models</b></summary>

There is no static list. `test_models.py` fetches the free catalog live via `fetch_free_models()` from `https://api.kilo.ai/api/gateway/models`. Restrict the set by filtering inside `fetch_free_models()` (say, by provider prefix or model id).
</details>

<details>
<summary><b>Change the schedule</b></summary>

Edit `.github/workflows/benchmark.yml`:
```yaml
- cron: '0 */6 * * *'  # Every 6 hours instead of every hour
```
</details>

<details>
<summary><b>Run locally</b></summary>

```bash
# Serve the dashboard
python3 -m http.server 8000
# Open http://localhost:8000

# Run benchmarks manually, no env vars needed
python3 scripts/test_models.py

# Or with an optional key for a dedicated quota
KILO_API_KEY=your_key_here python3 scripts/test_models.py
```
</details>

---

## 📦 Data Storage

`history.db` is a SQLite database committed to the repo and treated as the source of truth. The browser loads it through [sql.js](https://sql.js.org/) (WebAssembly) and runs all queries client-side. `scripts/results.json` and `results-worker*.json` are temporary per-job artifacts and are never committed.

```sql
runs          (id, timestamp, prompt, success_count, total_models, fastest_model, fastest_time)
model_results (run_id, model, success, error, response_time, tokens_generated, total_tokens, response)
```

Benchmark parameters: `temperature: 0.7` · `top_p: 0.9` · `max_tokens: 1000`, posted to the OpenAI-compatible endpoint `https://api.kilo.ai/api/gateway/chat/completions`.

---

## 🤝 Contributing

Fork the repo, cut a `feat/` or `fix/` branch, open a PR against `main`. Things that are useful:

- New chart types or dashboard widgets
- More provider metadata in `js/constants.js` (`PROVIDER_META`)
- Translations / i18n
- Bug fixes and performance work
- Docs improvements

Skim the open [Issues](https://github.com/vadash/KiloStats/issues) first to avoid duplicating effort.

---

## 🔗 Resources

- [Kilo AI Gateway](https://kilo.ai)
- [Kilo Models API](https://api.kilo.ai/api/gateway/models)
- [GitHub Actions Docs](https://docs.github.com/en/actions)
- [sql.js, SQLite in the browser](https://sql.js.org/)

---

## 📄 License

MIT. See [`LICENSE`](LICENSE).

---

<div align="center">

Made with ❤️ for the open-source AI community · [⭐ Star this repo](https://github.com/vadash/KiloStats) if you find it useful!

[![footer](https://capsule-render.vercel.app/api?type=waving&color=8bd11f&height=100&section=footer)](#)

</div>
