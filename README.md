# FiveM Framework Adapter Starter (ESX ↔ QBCore ↔ QBOX)

**Run one resource across ESX, QBCore, and QBOX** by hiding framework differences behind a tiny, stable interface: `FW.*`.
This starter ships production-ready adapters, a minimal demo, tests, and CI.

> Pillar article: https://rpscripts.net/converting-fivem-scripts/
> How to use this adapter: https://rpscripts.net/adapter-patterns/

---

## Why
- Frameworks diverge on **core access**, **player model**, **identifiers**, **money**, **inventory**, and **events**.
- Adapters keep business logic framework-agnostic. Swap the adapter, not your codebase.

---

## Features
- 🔌 **Auto-detect** ESX / QBCore / QBOX (`es_extended`, `qb-core`, `qbx_core`)
- 👤 Unified **Player** helpers (src → handle, name, ids)
- 🧰 **Job** helpers (name, grade, job-change hook)
- 💵 **Money** helpers (get/add/remove; accounts: `cash`, `bank`, optional `black_money`)
- 🎒 **Inventory** autodetect (prefers `ox_inventory`, falls back to framework inventory)
- 🔔 Simple **notify** hook (`fw:notify`) with `ox_lib` support
- ✅ Unit tests (`busted`) + lint (`luacheck`) + GitHub Actions
- 🧪 Demo command: `/fwbonus` (+$250 for `delivery` job)

---

## Requirements
One of:
- ESX (`es_extended`)
- QBCore (`qb-core`)
- QBOX (`qbx_core`)

Optional:
- `ox_inventory` (auto-used by adapters if present)
- `ox_lib` (for nicer notifications on the client)

---

## Install (as a resource)
1. Copy this folder to your server `resources/` as `fivemx-fw-adapter-starter`.
2. In `server.cfg`:
   ```cfg
   ensure fivemx-fw-adapter-starter
   ```
3. Start your framework(s) **before** this resource.

Try in-game:
- `/fwbonus` — pays $250 if your current job is `delivery`.
- On player load, gives two `water` items (via `ox_inventory` if available).

---

## Use in your own resource
Copy `shared/` (adapters + `fw.lua`) into your resource and add to `fxmanifest.lua`:
```lua
shared_scripts {
  'shared/adapters/*.lua',
  'shared/fw.lua'
}
```

Replace direct framework calls with the contract:
```lua
local p = FW.Player.getBySrc(source)
FW.Money.add(p, 'cash', 250, 'bonus')
FW.Inv.addItem(p, 'water', 1)
```

---

## Interface Contract (stable)
```lua
FW.meta.name() -> 'esx' | 'qbcore' | 'qbox'
FW.meta.has(resourceName) -> bool

FW.Player.getBySrc(src) -> framework player handle
FW.Player.getStateId(p) -> string  -- ESX: identifier; QB/QBOX: citizenid
FW.Player.getServerId(p) -> number
FW.Player.getName(p) -> string

FW.Job.getName(p) -> string
FW.Job.getGrade(p) -> number|string
FW.Job.onChange(function(src, oldJob, newJob))  -- best-effort

FW.Money.get(p, account) -> number
FW.Money.add(p, account, amount, reason?)
FW.Money.remove(p, account, amount, reason?)

FW.Inv.addItem(p, name, count, metadata?) -> bool
FW.Inv.removeItem(p, name, count, metadata?) -> bool

FW.Events.notify(src, message, kind?)  -- 'info'|'success'|'error'
FW.Events.onPlayerLoaded(function(src))  -- best-effort
```

### Inventory note
Servers vary (`qb-inventory`, `ox_inventory`, `qs-inventory`, etc.). This starter prefers `ox_inventory` when present and falls back to the framework inventory.

---

## Anti-Patterns → Fixes
- **Hard-coding ESX/QB/QBOX** across files → Call `FW.*` only; put framework access in adapters.
- **Long-lived player handles** → Re-resolve with `FW.Player.getBySrc` or key by `getStateId`.
- **Identifier confusion** (`identifier` vs `citizenid`) → Use neutral `state_id` in your DB.
- **Event name coupling** → Subscribe via `FW.Events.*` and relay if your fork differs.
- **Inventory assumptions** → Use `FW.Inv.*` with autodetect.

---

## Testing & CI
Local:
```bash
luarocks install luacheck
luarocks install busted
luacheck .
busted -v
```

CI (GitHub Actions) runs automatically on push/PR:
- `luacheck` (lint)
- `busted` (unit tests with stubbed exports)

---

## Contributing
See [`CONTRIBUTING.md`](./CONTRIBUTING.md). Keep the public surface minimal; add optional helpers in separate modules.

---

## License
MIT © fivemx
