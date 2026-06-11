# AURAVYBE â€” Agent bootstrap

## Instruction chain (read in order)

1. **Private Vault** `github.com/AgentDraven/merit-private-vault` â€” L1/L2 in `instructions/` (**private**)
2. `%USERPROFILE%\HumanBala\` â€” runtime copy via `merit-private-vault/scripts/install.ps1 sync`
3. `%USERPROFILE%\HumanBala\MrPIBala.instructions` â€” persona L2 (Mr-PI-Bala)
4. `AURAVYBE.instructions` â€” project specifics (L3, repo root) when present
5. `AURAVYBE docs/INDEX.md` â€” documentation front door when present

**Repo pointers:** `AURAVYBE docs/MERIT.instructions` links to HumanBala â€” do not fork L1 policy in this repo.

## Closeout discipline

Every completed scope follows MERIT **Â§G** hygiene â†’ **Â§VIII.F** (validate, commit, annotated tag, push) â†’ **Â§H** **3-3** in chat.

| Step | Action |
|------|--------|
| Hygiene | Stage only intended files; run applicable validation |
| Git | `.\scripts\merit.ps1 mXin -Message "<summary>"` |
| Report | **3-3** â€” **Done** Â· **State** Â· **Next** (â‰¤3 bullets each) |

Refresh toolchain: `merit-private-vault/scripts/install.ps1 sync` or `%USERPROFILE%\HumanBala\install.ps1 sync`.

## Operator commands

```powershell
.\scripts\merit.ps1 help
.\scripts\merit.ps1 bootstrap -Status
.\scripts\merit.ps1 mXout -Path index.html
.\scripts\merit.ps1 mXin -Message "feat: example closeout"
```

Onboard new MERIT repos: `merit-private-vault/scripts/onboard-repo.ps1 -RepoPath <path> -Project auravybe -Persona Mr-PI-Bala`.

