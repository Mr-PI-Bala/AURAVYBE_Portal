# AURAVYBE_Portal â€” Agent bootstrap

## Instruction chain (read in order)

1. **Private Vault** `github.com/AgentDraven/merit-private-vault` â€” L1/L2 SSOT in `instructions/` (**private**)
2. `%USERPROFILE%\HumanBala\MERIT.instructions` â€” L1 runtime (deployed via `merit.ps1 setup -Machine`)
3. `%USERPROFILE%\HumanBala\MrPIBala.instructions` â€” persona L2 (Mr-PI-Bala)
4. `AURAVYBE_Portal.instructions` â€” project specifics (L3, repo root) when present
5. `AURAVYBE_Portal docs/INDEX.md` â€” documentation front door when present

**Repo pointers:** `AURAVYBE_Portal docs/MERIT.instructions` links to HumanBala â€” do not fork L1 policy in this repo.

## Closeout discipline

Every completed scope follows MERIT **Â§G** hygiene â†’ **Â§VIII.F** (validate, commit, annotated tag, push) â†’ **Â§H** **3-3** in chat.

| Step | Action |
|------|--------|
| Hygiene | Stage only intended files; run applicable validation |
| Git | `.\scripts\merit.ps1 mXin -Message "<summary>"` |
| Report | **3-3** â€” **Done** Â· **State** Â· **Next** (â‰¤3 bullets each) |

Refresh toolchain: `merit.ps1 setup -Machine` (from vault or HumanBala).

## Operator commands (daily)

```powershell
.\scripts\merit.ps1              # context card
.\scripts\merit.ps1 mXout -Path index.html
.\scripts\merit.ps1 mXin -Message "feat: example closeout"
```

Rare: `setup` Â· `eXout`/`eXin` Â· `publish` Â· `doctor` â€” see vault `docs/MERIT_VAULT.md`.

Onboard new repos: `merit.ps1 setup -Onboard -Project auravybe-portal -Persona Mr-PI-Bala`.

