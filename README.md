# BadSuccessor Automation Script

This PowerShell script automates the **BadSuccessor** attack technique, which leverages **Delegated Managed Service Accounts (dMSA)** to escalate privileges in Active Directory environments. It simplifies the attack by programmatically:

- Creating a machine account (computer object)
- Creating a dMSA
- Granting permissions
- Setting the required delegation fields
- Requesting TGT and TGS using Rubeus
- Demonstrating access before and after impersonation

---

## 🔍 Recon

- [Akamai Recon Script](https://github.com/akamai/BadSuccessor)
-  ```powershell
      .\Get-BadSuccessorOUPermissions.ps1
   ```

---

## 🔧 Requirements

### ✅ Tools
- `Rubeus.exe`
  - Latest version - Required to generate AES256 hash, request TGT/TGS, use DMSA authentication and inject tickets
  - [GitHub - Rubeus](https://github.com/GhostPack/Rubeus)

- **RSAT: Active Directory Tools**
  - Required for AD PowerShell cmdlets like `New-ADComputer`, `Set-ADServiceAccount`, etc.
  - [Latest ADModule](https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2025-ps)
  - Install with:
    ```powershell
    Get-WindowsCapability -Name RSAT* -Online | Select-Object DisplayName, State
    Add-WindowsCapability -Online -Name "Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0"
    ```

---

### Ensure `Rubeus.exe` is in your Windows Path.

---

## 🚀 Usage

Run the script with the following parameters:

```powershell
.\Invoke-BadSuccessor.ps1 `
  -Domain talokan.local `
  -OU BadOU `
  -LinkTargetDN "CN=Administrator,CN=Users,DC=talokan,DC=local" `
  -LowPrivUser "TALOKAN\namor" `
  -TargetHost "DC6.Talokan.local"
```

---

## ⚙️ Parameters

| Parameter       | Description                                                                 |
|----------------|-----------------------------------------------------------------------------|
| `-Domain`       | Your target domain FQDN (e.g., talokan.local)                               |
| `-OU`           | Organizational Unit name where the computer/dMSA will be created (e.g., BadOU) |
| `-LinkTargetDN` | The DN of the high-privileged account to impersonate                         |
| `-LowPrivUser`  | A user with low privileges but `CreateChild` permissions on the OU          |
| `-TargetHost`   | Host to test access before/after impersonation (e.g., DC6.Talokan.local)     |

---

## 🛠️ What It Does

1. **Create a new machine account** using a fixed password
2. **Derive the AES256 hash** using Rubeus
3. **Create a dMSA** that allows that machine to request its password
4. **Grant GenericAll** on the dMSA to the specified user
5. **Set impersonation fields** (`msDS-ManagedAccountPrecededByLink`, `msDS-DelegatedMSAState`)
6. **Use Rubeus** to:
   - Request a TGT for the computer
   - Use S4U2Self to get a TGS as the dMSA
7. **Run access tests** (e.g., `dir \\<host>\c$`) before and after the impersonation

Each step includes an **interactive pause** so you can observe or validate changes in real-time.

---

## 🧪 Example Output Flow

- Machine account: `BadMachine1234`
- dMSA: `BadDMSA1234`
- TGT acquired via `Rubeus.exe asktgt`
- TGS acquired via `Rubeus.exe asktgs /dmsa`
- Access shown pre- and post-attack to demonstrate impact

---

## 🛡 Disclaimer

> This script is intended for **educational purposes** and **authorized penetration testing** only. Do **not** run this against systems you do not own or have explicit permission to test. Misuse of this script may violate laws and result in severe consequences.

---

## 📎 References

- [Akamia Blog](https://www.akamai.com/blog/security-research/abusing-dmsa-for-privilege-escalation-in-active-directory)
- [Rubeus GitHub](https://github.com/GhostPack/Rubeus)

---
