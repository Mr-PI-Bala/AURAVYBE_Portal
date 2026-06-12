# BOOTSTRAPPING.md

MERIT repository lifecycle: **structure first**, **bootstrap second**, **mXout / mXin** for collaborative edit and sync.

All operator commands run through **one script**: `scripts/merit.ps1` (canonical copy in `merit-private-vault`, runtime in `%USERPROFILE%\HumanBala\scripts\`).

Set **`MERIT_PERSONA`** in `.env.local` (see `cfg/merit-personas.json` in the private vault).

## Quick reference

| Step | Command | Purpose |
|------|---------|---------|
| 0 | (manual or `-ScaffoldMissing`) | MERIT folder layout |
| 1 | `merit.ps1 bootstrap` | First-time Git + GitHub setup |
| 2 | `merit.ps1 mXout` | Lock file/dir (recursive) + pull from remote |
| 3 | *(edit locally)* | Exclusive work on locked paths |
| 4 | `merit.ps1 mXin` | Commit + push + release locks |
| * | `merit.ps1 bootstrap -Status` | Health check anytime |

```powershell
.\scripts\merit.ps1 help
.\scripts\merit.ps1 bootstrap
.\scripts\merit.ps1 mXout -Path "{Name} docs\theme.md"
.\scripts\merit.ps1 mXin
```

---

## 0. Before You Run Bootstrap

`merit.ps1 bootstrap` assumes the **MERIT golden-standard layout** (section I.A) exists at the repo root.

| Expectation | Required? | Notes |
|-------------|-----------|-------|
| `core/`, `scripts/`, `tests/`, `cfg/`, **`{Name} docs/`**, `output/` | **Yes** | Directory skeleton |
| `README.md`, `VERSION`, `CHANGELOG.md` | Recommended | `-ScaffoldMissing` can create placeholders |
| `AGENTS.md`, `.env.example` | Recommended | L1/L2 pointer and secrets template |
| `run_[project].py`, `test_[project].py` | Recommended | Two-entry-point pattern |
| `ops/` | Optional | Holds `ops/locks/` for mXout file locks |
| `.env.local`, `.git/`, GitHub remote | **No** | Created by bootstrap |

**Machine prerequisites:** Git, GitHub CLI (`gh`), network.

---

## 1. Bootstrap — `merit.ps1 bootstrap`

```powershell
.\scripts\merit.ps1 bootstrap
.\scripts\merit.ps1 bootstrap -Status
.\scripts\merit.ps1 bootstrap -Sync
```

---

## 2. Collaborative Edit — mXout / mXin

See `merit-private-vault/docs/MERIT_VAULT.md` and L1 MERIT.instructions §XI.D.

---

## 3. Baseline Release — `merit.ps1 release`

| Bump | Who may run (MERIT) |
|------|---------------------|
| **patch** | Routine validated closeout |
| **minor** | **Human Bala** approval |
| **major** | **Human Bala** approval |

---

## 4. Confirmation Codes (section II.F)

| Answer | Meaning |
|--------|---------|
| **ya** | Yes — this item only |
| **na** | No — skip |
| **ay** | All yes — remaining items |
| **an** | All no — skip remaining |
