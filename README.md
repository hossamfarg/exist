
# Internet Health Monitor – SPA (Adaptive Probe, 10–30s, 2s Timeout)

A zero-dependency, single‑file web app that monitors internet reachability by probing a configurable URL. It adapts probe frequency between **10s and 30s** based on recent results, shows a live **2‑hour history chart**, and reports an **overall status** based on the last **5 minutes**.

---

## ✅ What it does

- **Connectivity probe:** Calls a target URL (default: `https://www.yahoo.com/favicon.ico`).
- **Dynamic cadence (bounded):**
  - **More often on failures** (shorter interval).
  - **Less often on success** (longer interval).
  - **Never faster than 10 seconds** and **never slower than 30 seconds**.
- **Timeout policy:** A test **fails** if no response is received within **2 seconds**.
- **Status window:** Computes **Healthy / Unstable / Down** based on the last **5 minutes**.
- **History:** Keeps the last **2 hours** of results and renders them in a **stepped chart** (success=1, failure=0).
- **Persistence:** History and the chosen URL are stored in your browser’s **localStorage**.

---

## 🚀 Quick Start

1. Open the HTML file in a modern browser (Chrome/Edge/Firefox).
2. (Optional) Enter a different **Probe URL** and click **Apply**.
3. Watch the **status badge**, **5‑min success rate**, and the **history chart** update live.
4. Use **Test now** to trigger a run (respecting the 10s guardrail).
5. Use **Clear history** to reset local data for the past 2 hours.

> For deeper console logs, open with query flags:  
> `file:///.../internet-monitor.html?log=debug&trace=1&net=1`

---

## 🧭 Controls & UI

- **Probe URL**: Text field at the top (e.g., `example.com`, `https://example.com/favicon.ico`, `http://www.gstatic.com/generate_204`).  
  - Bare domains are normalized to `https://` and default to `/favicon.ico` if no path is provided.
  - The exact request includes a cache‑buster query parameter.
- **Apply**: Saves the URL and restarts probing (cadence resets to mid‑point).
- **Test now**: Manually triggers a probe (will not exceed 1 per 10s).
- **Clear history**: Wipes stored history (2‑hour window).
- **Status badge**: Shows **Healthy / Unstable / Down** with color.
- **History chart**: Stepped 0/1 line over the past 2 hours; **hover** to see timestamp & latency.

---

## 🔧 Probe Logic (High‑level)

- Uses **`fetch(url, { mode: 'no-cors' })` + `AbortController`**:
  - Treats **successful completion** of the request (even opaque cross‑origin) as **Success**.
  - Treats **abort at 2s** or **network error** as **Failure**.
- Works with:
  - Image/favicon endpoints (e.g., `/favicon.ico`).
  - **204 No Content** endpoints (e.g., `http://www.gstatic.com/generate_204`) that don’t return bytes.

> Why `no-cors`? It allows cross‑origin requests without reading the body/status, which is sufficient to validate basic reachability and respects browser security.

---

## ⏱️ Cadence (Adaptive, with Guardrails)

- **Initial interval:** 20s (mid‑range).
- **On Failure:** Interval **tightens** (×0.70), min **10s**.
- **On Success:** Interval **backs off** (×1.18 + 500ms), max **30s**.
- **Hard guardrail:** The scheduler prevents **any** run faster than **10s** (including “Test now”).

---

## 🧮 Status Calculation (5 minutes)

- Window: Last **5 minutes** of results.
- **Healthy:** Success rate ≥ **90%**  
- **Unstable:** 60–89%  
- **Down:** < 60%

Shown as:
- **“Internet is working”** for the last probe result = Success.
- **“Internet is bad”** for the last probe result = Failure.
- Overall label (Healthy/Unstable/Down) reflects the windowed rate.

---

## 📈 History Chart (2 hours)

- X-axis: Relative time across the last **2 hours**, grid every **10 minutes**.
- Y-axis: **1 = Success**, **0 = Failure** (stepped line).
- **Hover** shows a tooltip with **timestamp** + **Success/Failure** + **duration (ms)**.
- Redraws on window resize.

> The chart is rendered via **HTML Canvas** (no external libraries), so it works in restricted environments with blocked CDNs.

---

## 💾 Persistence

- **History**: Stored in `localStorage` under a dedicated key; kept for the last **2 hours** (rolling window).
- **Settings**: The **Probe URL** persists so you don’t have to re-enter it after reloads.

> If the browser blocks third‑party storage or you use a strict privacy mode, local persistence may be disabled.

---

## ✅ Recommended URLs

- **Small static assets** like `/favicon.ico` (fast and cache‑friendly).
- **Captive portal-style endpoints** like `http://www.gstatic.com/generate_204` (returns **204 No Content**; fetch approach handles it correctly).

> If an authenticated or blocked endpoint is used, expect **Failure** results.

---

## 🧩 Troubleshooting

- **No chart / errors about Chart.js**: Not applicable — the page uses a custom Canvas chart (no external libs).
- **No persistence**: Check if storage is blocked (InPrivate, tracking protection, enterprise policy).
- **Unexpected timeouts**: Verify outbound network and DNS, and consider adjusting the target URL to a nearby CDN or known-fast endpoint.
- **Probe URL “works in browser but fails here”**: If the page is protected or returns non-public content, `no-cors` fetch may still resolve; if it truly blocks, you’ll see a Failure. Prefer public assets/204 endpoints.

---

## 🔐 Privacy & Security

- The app runs **entirely in your browser**.
- It **does not** send your history or settings to any server.
- The probe is a standard **HTTP(S) request** to the configured URL.
- Use public, non-sensitive endpoints to avoid accidental disclosure in logs.

---

## ⚙️ Config Summary

- **Min interval:** 10s  
- **Max interval:** 30s  
- **Timeout:** 2s (probe counts as Failure if exceeded)  
- **History window:** 2 hours  
- **Status window:** 5 minutes  
- **Adaptive rules:** Tighten on Failure, back off on Success (bounded above)

---

## 🧰 Power-user logging

Append query params for diagnostics:

- `?log=debug` — verbose logs (`info|warn|error|debug`)
- `&trace=1` — grouped sections (`runOnce()`, `updateUI()`, `probeOnce()`)
- `&net=1` — adds `console.time` around network operations

Example:  
`internet-monitor.html?log=debug&trace=1&net=1`

---

## 📄 Change the Probe URL

1. Enter a domain or full URL in **Probe URL**:
   - `bing.com` → normalized to `https://bing.com/favicon.ico`
   - `http://www.gstatic.com/generate_204` → used as-is (with cache-busting)
2. Click **Apply** to start using it immediately.
3. The interval resets to the middle (20s) and adapts from there.

---

## ✨ What this is great for

- Quick connectivity validations during incidents.
- Lightweight monitoring without installing agents or servers.
- On-call triage to prove intermittent connectivity (chart + success rate).
- Attaching a **CSV export** (optional enhancement available) to tickets.

---

*Authored for internal support use. No external dependencies. Save the HTML locally and run.*  
