# Vulnerability Report — System Security Audit
**DecodeLabs Industrial Training Kit — Project 4**
**Author:** Kawtar Moustaghfir
**Date:** August 1, 2026
**System:** Windows 11 (KAWTAR) — HP Laptop

---

## Audit Methodology

Four-step checklist following the DecodeLabs System Vulnerability Checklist:
- Step 1: Identity & Access (passwords, MFA)
- Step 2: Software & Patch Status
- Step 3: Human Perimeter (screen lock, guest accounts, privilege)
- Step 4: Network & Endpoint Hygiene (firewall, disk encryption)

---

## Section 1 — Findings

### Finding 1 — Disk Encryption Disabled (BitLocker Off)
| Field | Detail |
|-------|--------|
| **Risk Level** | 🔴 High |
| **CVSS Score** | 7.4 |
| **CVSS Vector** | Physical access, no authentication required |
| **Affected Asset** | C: drive (237 GB, OS volume) |
| **Evidence** | `manage-bde -status` → Protection désactivée, 0% chiffré |

**Description:**
The system drive is completely unencrypted. In the event of physical theft or loss of the laptop, an attacker can remove the drive and read all data — including credentials, project files, and sensitive documents — without needing the Windows login password.

**Proof (command output):**

Volume C: []
[Volume du système d'exploitation]
Version de BitLocker : Aucun
État de la conversion : Intégralement déchiffré
Pourcentage chiffré : 0,0%
État de la protection : Protection désactivée


---

### Findings: No Issues Detected

| Check | Result |
|-------|--------|
| Windows Update | ✅ Last update: July 30, 2026 (KB5101684) |
| Firewall (Domain/Private/Public) | ✅ Active, BlockInbound on all profiles |
| Guest account | ✅ Does not exist |
| Screen lock timeout | ✅ 3 minutes (under 5-minute threshold) |
| Password policy | ✅ Password required on user account |
| User privilege | ✅ Standard user (Utilisateurs), not admin |

---

## Section 2 — Remediation

### Finding 1 — BitLocker Remediation Attempt

Three remediation methods were attempted:

**Attempt 1 — TPM-based encryption:**
```powershell
manage-bde -on C: -RecoveryPassword
```
Result: `ERREUR 0x8028008b — TPM 2.0: Handle incorrect`
Cause: TPM 2.0 chip is present but not correctly initialized.

**Attempt 2 — Password-based encryption:**
```powershell
manage-bde -on C: -Password
```
Result: `ERREUR 0x8031006a — Les paramètres de stratégie de groupe ne permettent pas la création d'un mot de passe`
Cause: Group Policy blocks non-TPM BitLocker methods.

**Attempt 3 — PowerShell method:**
```powershell
Enable-BitLocker -MountPoint "C:" -EncryptionMethod Aes256 -UsedSpaceOnly -SkipHardwareTest -RecoveryPasswordProtector
```
Result: Same TPM 2.0 error (0x8028008B)

**Root cause analysis:**
Two blocking conditions exist simultaneously:
1. TPM 2.0 chip is misconfigured at the hardware/BIOS level
2. Group Policy prevents password-based fallback (likely a domain/OEM policy)

**Recommended remediation path:**
1. Enter BIOS → Security → TPM → Clear and re-initialize TPM
2. Or: Via Group Policy Editor (`gpedit.msc`) → Computer Configuration → Administrative Templates → Windows Components → BitLocker Drive Encryption → Operating System Drives → Enable "Require additional authentication at startup" with "Allow BitLocker without a compatible TPM" checked
3. Then re-run: `manage-bde -on C: -RecoveryPassword`

---

## Section 3 — Risk Summary & Remediation Roadmap

| # | Finding | Risk | Status |
|---|---------|------|--------|
| 1 | BitLocker disabled | 🔴 High | ⚠ Remediation blocked by TPM/GPO — fix requires BIOS access |

**Overall security posture:** Moderate. The system is well-maintained (recent patches, active firewall, proper screen lock, no guest account) but the lack of disk encryption is a significant gap, especially for a laptop that can be physically stolen.

---

## Compliance Mapping

| Standard | Relevant Control | Status |
|----------|-----------------|--------|
| CIS Controls v8 | Control 3.6 — Encrypt data on end-user devices | ❌ Fail |
| ISO 27001:2022 | Annex A 8.5 — Secure authentication | ✅ Pass |
| NIST 800-63B | Password & MFA standards | ✅ Pass |

---

*DecodeLabs — Industrial Training Kit · Project 4: System Vulnerability Checklist*
