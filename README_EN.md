[🇷 Русский](README.md) · [🇬🇧 English](README_EN.md)

# 🛠 GMod Server Helper

> How heavy is your collection? And what should go into `workshop.lua`? The helper answers both.

**🔴 Live demo:** https://duckflit.github.io/gmod_helper/

A set of free in-browser tools for Garry's Mod server admins.
No backend, no API keys, no signup: the whole app is a single `index.html`
you can open locally or publish to GitHub Pages.

<!-- When you add a screenshot, uncomment and put the file at docs/screen.png
![Screenshot](docs/screen.png)
-->

## 🧰 Tools

| Tool | Status | What it does |
|---|---|---|
| **Collection Helper** | ✅ working | Per-addon size, total weight, heaviest addon, sorting and CSV export |
| **Workshop.lua Generator** | ✅ working | Generates a ready-to-use `workshop.lua` with `resource.AddWorkshop(...)` lines from a collection — with a download button |
| Conflict checker | 🔜 soon | Model/material/entity conflict detection |
| server.cfg generator | 🔜 soon | Ready-made commented config |
| Load estimation | 🔜 soon | Collection load estimation for the server |

UI languages — **Russian and English** (toggle in the sidebar, the choice is remembered).

## 🚀 Usage

1. Open https://duckflit.github.io/gmod_helper/ (or a local `index.html`).
2. Paste a collection link or its ID (e.g. `2812330501`).
3. Press **“Analyze”** to get sizes and stats,
   or **“Generate”** to get a `workshop.lua` file.

Example of the generated file:

```lua
-- workshop.lua · GMod Server Helper
-- Collection: https://steamcommunity.com/sharedfiles/filedetails/?id=2812330501
-- Addons: 299 · 2025-08-24

resource.AddWorkshop("3690602885") -- Пособие по атмосфернагагой физике
resource.AddWorkshop("1604765873") -- !*$%? ERRORS
resource.AddWorkshop("246756300") -- 3D Stream Radio
```

Drop the file into `garrysmod/lua/autorun/server/workshop.lua` — the server pulls the addons on start.

## ⚙️ How it works

- The collection page is fetched through public CORS proxies and parsed right in the browser (`DOMParser`).
- Sizes come from the public Steam Web API (`ISteamRemoteStorage/GetPublishedFileDetails`) —
  via a **POST request**: Steam now requires POST, and the helper handles that.
- If one proxy is down, others are tried automatically; every step is visible in the “Debug log”.
- Hidden, banned and deleted addons have no public size — they are tagged separately and excluded from the total.

## 🖥 Run & deploy

```bash
# Locally — just open the file
index.html

# GitHub Pages
git add . && git commit -m "gmod server helper" && git push
# Settings → Pages → Branch: main → Save
```

## 🔐 Your own CORS proxy (optional)

Public proxies go down sometimes. For 100% stability, deploy your own on
Cloudflare Workers (free, 15 lines):

<details>
<summary>Worker code</summary>

```js
export default {
  async fetch(req){
    const url = new URL(req.url);
    if (req.method === 'OPTIONS'){
      return new Response(null, { status: 204, headers: {
        'Access-Control-Allow-Origin': '*', 'Access-Control-Allow-Headers': '*',
        'Access-Control-Allow-Methods': 'GET,POST,OPTIONS' } });
    }
    if (url.pathname !== '/proxy')
      return new Response('proxy alive', { headers: { 'Access-Control-Allow-Origin': '*' } });
    const target = url.searchParams.get('url');
    if (!target) return new Response('missing url', { status: 400 });
    const res = await fetch(target, {
      method: req.method,
      body: (req.method === 'POST' || req.method === 'PUT') ? req.body : undefined,
      headers: { 'Content-Type': req.headers.get('Content-Type') || 'application/x-www-form-urlencoded' },
    });
    return new Response(res.body, { status: res.status, headers: {
      'Access-Control-Allow-Origin': '*',
      'Content-Type': res.headers.get('Content-Type') || 'application/json' } });
  },
};
```

</details>

Then put the worker URL into `index.html`: `const MY_PROXY = 'https://name.workers.dev';`

## 🗂 Structure

```
gmod_helper/
├── index.html      # the whole app: styles, markup, logic
├── README.md       # Russian version
└── README_EN.md    # you are here (EN)
```

## 🗺 Roadmap

- [ ] Addon conflict checker
- [ ] server.cfg generator
- [ ] Collection load estimation
- [ ] Saved collections (localStorage)

## ⚠️ Disclaimer

This project is not affiliated with Valve Corporation or Steam. All data is taken
from public Steam pages and the Steam Web API.
