# BadSuccessor Automation Script

This PowerShell script automates the **BadSuccessor (Delegated Managed Service Account abuse)** attack using a low-privileged user in an Active Directory domain.

It creates:
- A new computer account
- A delegated Managed Service Account (dMSA)
- Grants impersonation to the dMSA
- Extracts TGT and TGS tickets using Rubeus
- Demonstrates pre- and post-access differences

---

## 🔧 Requirements

- **Rubeus.exe**
  - Must be present in the **current working directory**
  - Used to generate AES256 hashes, request TGT/TGS, and inject tickets

- **RSAT Active Directory Tools**
  - PowerShell commands like `New-ADComputer`, `New-ADServiceAccount`, `Set-ADServiceAccount`, etc.
  - Can be installed via:
    ```powershell
    Add-WindowsFeature RSAT-AD-PowerShell
    ```

- **Domain credentials with ability to create computer objects** in the specified OU

---

## 📁 File Structure
