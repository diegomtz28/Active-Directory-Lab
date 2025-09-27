# 🖥️ Active Directory Lab  

This project simulates a small corporate Active Directory environment with OUs, users, groups, file shares, Group Policy Objects (GPOs), and security hardening. It also includes a real troubleshooting scenario where I accidentally locked out the Domain Administrator account and successfully recovered it.  

Built on **Windows Server 2022 (Domain Controller)** and **Windows 11 (domain-joined client)**.  

---


## 🛠️ Lab Setup  

- Installed and promoted Windows Server 2022 as Domain Controller (`corp.local`).  
- Created OU structure for HR, IT, and Sales.  
- Used built-in AD DS management tools (ADUC, GPMC, etc.) directly on the Domain Controller for configuration.  

📸 Screenshots:  
| DC Promotion | OU Structure |  
|--------------|--------------|  
| ![Promotion](https://github.com/diegomtz28/Active-Directory-Lab/blob/main/docs/images/promotingdc.png.webp) | ![OUs](https://github.com/diegomtz28/Active-Directory-Lab/blob/main/docs/images/OUorganization.png.webp) |  


---

## 👥 User & Group Management  

- Bulk-created 60 users using PowerShell automation.  
- Organized users into HR, IT, and Sales OUs.  
- Created security groups: `Human Resources`, `Help Desk`, `Sales Users`.  
- Delegated password reset rights to Help Desk.  

📸 Screenshots:  
| PowerShell Script | Users Created | Groups in ADUC |  
|-------------------|---------------|----------------|  
| ![Script](https://github.com/diegomtz28/Active-Directory-Lab/blob/main/docs/images/script%20for%2060%20users.png) | ![Users](https://github.com/diegomtz28/Active-Directory-Lab/blob/main/docs/images/Users%20Creatwd.png.webp) | ![Groups](https://github.com/diegomtz28/Active-Directory-Lab/blob/main/docs/images/groups%20created.png) |  

---

## 📂 File Shares & Drive Mapping  

- Created departmental shares (`HRShare`, `ITShare`, `SalesShare`).  
- Configured NTFS/share permissions for each group.  
- Applied drive mappings via Group Policy Preferences.  

📸 Screenshots:  
| Shared Folders | NTFS Permissions | Drive Mapping via GPO |  
|----------------|------------------|------------------------|  
| ![Shares](screenshots/shares-gpo/shares.png) | ![Permissions](https://github.com/diegomtz28/Active-Directory-Lab/blob/main/docs/images/NTFS%20permission%20set.png) | ![DriveMapping](screenshots/shares-gpo/drive-mapping.png) |  

---

## 🔒 Security Hardening with Group Policy  

- Created and linked `CORP Security Baseline` GPO.  
- Enforced Password Policy: **12+ char, complexity, 90-day expiration**.  
- Applied Account Lockout Policy: **3 attempts → 30-minute lockout**.  
- Disabled Guest account domain-wide.  
- Applied with `gpupdate /force` and verified via login tests.  

📸 Screenshots:  
| GPO Created | Password Policy | Account Lockout Policy | Guest Account Disabled | gpupdate /force |  
|-------------|-----------------|------------------------|------------------------|-----------------|  
| ![GPO](screenshots/security/gpo-created.png) | ![PassPolicy](screenshots/security/password-policy.png) | ![Lockout](screenshots/security/account-lockout.png) | ![GuestDisabled](screenshots/security/guest-disabled.png) | ![Gpupdate](screenshots/security/gpupdate.png) |  

---

## ⚠️ Troubleshooting: Domain Admin Password Reset  

While running a bulk password reset script, I accidentally included the **Domain Administrator** account, locking myself out.  

### 🔍 Problem  
- Could not log in with Domain Admin.  
- All accounts reset to the same default password.  

📸 Screenshots:  
| Failed Login | Lockout Message |  
|--------------|-----------------|  
| ![FailedLogin](screenshots/troubleshooting/failed-login.png) | ![Lockout](screenshots/troubleshooting/lockout.png) |  

---

### 🛠️ Recovery Process  
1. Logged into the DC using the **local Administrator** account.  
2. Opened **Active Directory Users and Computers (ADUC)**.  
3. Reset the Domain Admin password.  
4. Logged back in as Domain Admin successfully.  

📸 Screenshots:  
| Local Admin Login | Resetting in ADUC | Successful Login as Domain Admin |  
|-------------------|-------------------|----------------------------------|  
| ![LocalAdmin](screenshots/troubleshooting/local-admin.png) | ![Reset](screenshots/troubleshooting/aduc-reset.png) | ![Success](screenshots/troubleshooting/success-login.png) |  

---

### ✅ Preventive Fix  
I updated the script to **exclude privileged accounts**:  

```powershell
Get-ADUser -Filter * -SearchBase "OU=Corp Users,DC=corp,DC=local" |
Where-Object { $_.SamAccountName -notin @("Administrator","LabAdmin") } |
ForEach-Object {
    $newPass = ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force
    Set-ADAccountPassword $_ -Reset -NewPassword $newPass
    Set-ADUser $_ -ChangePasswordAtLogon $true
}
