
# Internet Health Monitor – SPA (Adaptive Probe, Configurable Bounds, 2s Timeout)

A zero‑dependency, single‑file web app to monitor internet reachability by probing a configurable URL. It adapts its probe frequency within **configurable bounds** (default: **10s–30s**), displays a live **2‑hour history chart**, and reports an **overall status** based on the last **5 minutes**.

---

## ✅ What it does

- **Connectivity probe:** Calls a target URL you set (default: `https://www.yahoo.com/favicon.ico`).
- **Adaptive cadence (configurable bounds):**
  - **More often on failures** (shorter interval).
  - **Less often on successes** (longer interval).
  - **Never faster than your Min interval** and **never slower than your Max interval**.
- **Timeout policy:** A probe **fails** if no response within **2 seconds**.
- **Status window:** Computes **Healthy / Unstable / Down** based on the last **5 minutes**.
- **History:** Keeps the last **2 hours** of results in a **stepped chart** (success=1, failure=0).
- **Persistence:** Stores your **Probe URL** and **cadence limits** in the browser’s **localStorage**.

---

## 🚀 Quick Start

1. Open the HTML page in a modern browser (Chrome/Edge/Firefox).
2. Set the **Probe URL**:
   - Type a domain (e.g., `bing.com`) or a full link (e.g., `http://www.gstatic.com/generate_204`).
   - Click **Apply** to start probing it immediately.
3. (Optional) Configure cadence **limits**:
   - Enter **Min interval (seconds)** and **Max interval (seconds)**.
   - Click **Apply limits**.
4. Use **Test now** to trigger a probe (respects the Min interval guard).
5. Use **Clear history** to wipe local data (rolling 2‑hour window).

> For verbose diagnostics, append:  
> `?log=debug&trace=1&net=1`  
> Example: `internet-monitor.html?log=debug&trace=1&net=1`

---

## 🧭 Controls & UI

- **Probe URL**: Text field at the top.  
  - Bare domains are normalized to `https://` and default to `/favicon.ico` if no path is provided.
  - The request includes a **cache‑buster** query param to avoid cached responses.
- **Apply**: Saves the URL and restarts probing (cadence resets to the midpoint of your configured bounds).
- **Min/Max interval (seconds)**: Configurable bounds for adaptive cadence (default **10s** min, **30s** max).  
  - Enforced guardrails: **no run faster than Min** and **no run slower than Max**.
- **Apply limits**: Saves your cadence bounds, clamps the current interval, and continues probing.
- **Test now**: Immediately triggers a probe (throttled to not violate Min interval).
- **Clear history**: Wipes stored history.
- **Status badge**: Shows **Healthy / Unstable / Down**.
- **History chart**: Stepped 0/1 line over the past **2 hours**. **Hover** for timestamp & latency tooltip.

---

## 🔧 Probe Logic (High Level)

- Uses **`fetch(url, { mode: 'no-cors' })`** with an **AbortController** timeout:
  - If the request **completes** before the timeout, the result is **Success** (opaque response is acceptable for reachability).
  - If the request **aborts** at **2 seconds** (or throws a network error), the result is **Failure**.
- Works for:
  - **Small public assets** (e.g., `/favicon.ico`).
  - **204 No Content** endpoints (e.g., `http://www.gstatic.com/generate_204`), which are frequently used for connectivity checks.

> `no-cors` ensures cross‑origin requests complete without exposing response contents—sufficient to confirm reachability while respecting browser security constraints.

---

## ⏱️ Cadence (Adaptive + Configurable Guardrails)

- **Initial interval:** Midpoint between your **Min** and **Max** (default midpoint: 20s).
- **On Failure:** Interval **tightens** (about 30% shorter), clamped to **≥ Min**.
- **On Success:** Interval **backs off** (about 18% longer + 0.5s), clamped to **≤ Max**.
- **Hard guardrail:** Scheduler blocks any run **faster** than **Min** (applies to both automatic runs and “Test now”).

---

## 🧮 Status Calculation (5‑minute window)

- Evaluates the last **5 minutes** of results:
  - **Healthy:** Success rate ≥ **90%**
  - **Unstable:** 60–89%
  - **Down:** < 60%
- The **status badge** updates with these labels and colors.
- The **last probe** text shows “Internet is working” or “Internet is bad”.

---

## 📈 History Chart (2‑hour rolling window)

- X-axis: Relative time across the last **2 hours**, grid every **10 minutes**.
- Y-axis: **1 = Success**, **0 = Failure** (stepped line).
- **Hover tooltip**: Shows **timestamp** and **duration (ms)** of the nearest data point.
- Redraws when the window is resized.

> The chart is a custom **Canvas** rendering with **no external libraries**, so it works in restricted environments (no CDNs needed).

---

## 💾 Persistence

- **History**: Stored in `localStorage` under a dedicated key. Old points are trimmed beyond **2 hours**.
- **Settings**: Stores your **Probe URL**, **Min interval**, and **Max interval**.

> In strict privacy modes, localStorage may be unavailable.

---

## ✅ Recommended URLs

- Public, small assets like **`/favicon.ico`** for speed and reliability.
- Captive portal endpoints like **`http://www.gstatic.com/generate_204`** (returns **204 No Content**)—fully supported by the `fetch`‑based probe.

---

## 🧩 Troubleshooting

- **No persistence**: Check privacy mode or enterprise policies blocking localStorage.
- **Unexpected failures**: Confirm the endpoint is public (no auth) and reachable; consider testing with a known-fast CDN endpoint.
- **Probe “works in browser but shows Failure here”**: If the endpoint demands credentials or blocks cross-origin, the `fetch` may fail. Prefer public test endpoints.

---

## ⚙️ Config Summary

- **Min interval (default):** 10s (editable)
- **Max interval (default):** 30s (editable)
- **Timeout:** 2s
- **History window:** 2 hours
- **Status window:** 5 minutes
- **Adaptive rules:** Tighten on Failure; Back off on Success (within your configured bounds)

---

## 🔍 Optional Diagnostics

Append query params for logging:

- `?log=debug` — verbose logs
- `&trace=1` — grouped logs (`runOnce()`, `updateUI()`, `probeOnce()`)
- `&net=1` — times network operations

Example:  
`internet-monitor.html?log=debug&trace=1&net=1`

---

## ✨ Great for

- Quick incident validation of connectivity.
- Lightweight monitoring without agents or servers.
- Visual evidence of intermittent issues (chart + success %).
- Sharing with teams or attaching to tickets.

---

*Save the HTML locally and run. No external dependencies. Configurable cadence bounds and URL, persisted on your device.*
