# Blazor-Next.js---single-port-via-YARP
Blazor_Nextjs : Single-port Blazor Server + Next.js (App Router) via YARP



# Blazor_Nextjs
**Single-port Blazor Server + Next.js (App Router) via YARP**  
This project shows how to host a Next.js app **under `/next`** inside a Blazor Server app and serve everything from **one port** using **YARP** as a reverse proxy.  
Session for the Next demo is kept **in server memory** and identified by an `X-Session-Id` header (no cookies).

---

## ✨ What you get
- **Blazor Server (.NET 8)** as the main host
- **Next.js 14 (App Router)** mounted at **`/next`**
- **YARP 2.x** reverse proxy, single port (no CORS needed)
- **Dev auto‑start**: a hosted service runs `npm install/ci` and `npm run dev`
- **Health check** at **`/healthz`** (ASP.NET pings the Next API)
- Demo pages on Next:
  - `/next` — Counter using memory session (header only, no cookies)
  - `/next/todos` — Simple Todo list (memory per session)

---

## 📦 Requirements
- **.NET SDK 8.0+**
- **Node.js LTS** (with `npm`)  
- Windows/macOS/Linux (dev https is fine; use `-k` with curl if cert is untrusted)

---

## 🗂 Project layout (important parts)

Blazor_Nextjs-Level-2_v3/
- Blazor_Nextjs-Level-2.csproj      (.NET 8 web project; YARP package)
- Program.cs                         (Adds YARP and maps /next/** to Next server)
- Dev/NextDevHostedService.cs        (Dev-only: npm install/ci + npm run dev)
- Pages, Shared, Data, Services, wwwroot
- next/ (Next.js app, basePath '/next')
  - app/page.tsx                     (Counter demo)
  - app/todos/page.tsx               (Todos demo)
  - app/api/session/route.ts         (GET/POST; memory session)
  - app/api/todos/route.ts           (GET/POST/DELETE; memory store)
  - app/api/ping/route.ts            (Health response)
  - next.config.mjs                  (basePath '/next')
  - package.json, tsconfig.json
- scripts/
  - build-next-prod.cmd
  - start-next-prod.cmd

---

## ⚙️ How it works
- **YARP** proxies `/next/{**catchall}` to `NEXT_URL` (default `http://localhost:3000`).
- **Dev**: Hosted service auto-runs Next dev; just `dotnet run`.
- **Prod**: Build & start Next, set `NEXT_URL`, then run Blazor.
- **Health**: `/healthz` calls `NEXT_URL/next/api/ping` → 200 if Next OK.

---

## ▶️ Run in Development
dotnet restore
dotnet run
# https://localhost:<port>/
# https://localhost:<port>/next
# https://localhost:<port>/next/todos
# https://localhost:<port>/healthz

---

## 🚀 Run in Production (basic)
scripts\build-next-prod.cmd
scripts\start-next-prod.cmd        # Next: http://localhost:3000
set NEXT_URL=http://localhost:3000
dotnet run --configuration Release
# Browse via the Blazor port: /next, /next/todos, /healthz

---

## ✅ Test Plan

A) Smoke (UI)
- Home → Open Next root → +1 / Reset buttons work
- Home → Open Next Todos → can add items

B) Session (no cookies)
- /next: +1 a few times, Reset, F5 (value persists)
- /next: open Incognito → counter resets (new session)
- /next/todos: add items; refresh keeps them; Incognito shows empty

C) API with curl (use -k if cert untrusted)
curl -k https://localhost:<port>/healthz
curl -k https://localhost:<port>/next/api/session
curl -k -H "x-session-id: demo" https://localhost:<port>/next/api/session
curl -k -H "x-session-id: demo" -H "content-type: application/json" -d "{"op":"inc"}" https://localhost:<port>/next/api/session
curl -k -H "x-session-id: demo" https://localhost:<port>/next/api/todos
curl -k -H "x-session-id: demo" -H "content-type: application/json" -d "{"title":"Write README"}" https://localhost:<port>/next/api/todos
# Delete: curl -k -X DELETE -H "x-session-id: demo" "https://localhost:<port>/next/api/todos?id=<id>"

D) HMR/WebSocket (dev)
- Keep /next open → edit next/app/page.tsx → save → page hot-reloads

E) Failure
- Stop Next → /healthz returns 503; start again → 200

---

## 🔧 Code highlights
Program.cs: YARP routes without transforms; UseWebSockets(); MapReverseProxy().
next/next.config.mjs: basePath '/next'.

---

## 🐞 Troubleshooting
- InvalidOperationException for X-Forwarded → remove transforms (fixed in v3)
- /healthz 503 → Next not running / NEXT_URL wrong
- Dev HTTPS warnings → dotnet dev-certs https --trust or curl -k
- Next dev fails → install Node/npm; try npm run dev in next/

---


## 📜 License
MIT (or your preference).

## ✅ Status
v3: transforms removed; namespaces unified; health + hosted service working.
