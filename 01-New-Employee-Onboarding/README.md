## 01 - New Employee Onboarding

A new employee, Sarah Mitchell, was joining the Marketing department. The goal was to provision her complete domain account from scratch using PowerShell — including AD user creation, security group setup, network share access, mapped drive via GPO, and roaming profile configuration. All steps were completed without touching the GUI.

---

## Step 1: Create the AD User Account

Used `New-ADUser` to create Sarah's account and place it directly into the Marketing OU. The `-Enabled $true` flag activates the account immediately so the user can log in without any additional steps.

```powershell
New-ADUser `
  -Name "Sarah Mitchell" `
  -GivenName "Sarah" `
  -Surname "Mitchell" `
  -SamAccountName "smitchell" `
  -UserPrincipalName "smitchell@mydomain.com" `
  -Path "OU=Marketing,DC=mydomain,DC=com" `
  -AccountPassword (ConvertTo-SecureString "TempPass123!" -AsPlainText -Force) `
  -Enabled $true
```

<img width="1918" height="867" alt="creating sarah user" src="https://github.com/user-attachments/assets/07d32577-e948-4934-84a5-48e75a0a5cbc" />


---

## Step 2: Verify Account Placement

Checked the `DistinguishedName` property to confirm Sarah's account landed in the correct Marketing OU. This is an important verification step to ensure GPOs and group permissions will apply correctly.

```powershell
Get-ADUser -Identity smitchell -Properties DistinguishedName | Select DistinguishedName
```

<img width="1917" height="883" alt="user sarah confirmation creation" src="https://github.com/user-attachments/assets/b32bcbbe-5004-4f88-94f4-ee6d0b918411" />


---

## Step 3: Force Password Change at Next Login

Configured the account to require a password change on first login so only the user knows their final password — a security best practice after any new account creation.

```powershell
Set-ADUser -Identity smitchell -ChangePasswordAtLogon $true
```

<img width="1887" height="862" alt="sarah set password change at logon" src="https://github.com/user-attachments/assets/7a26707a-7751-49f2-9ff5-d9a08728eedf" />


---

## Step 4: Create the Marketing Security Group

Created a dedicated security group for the Marketing department. Security groups are used to assign permissions to shared resources at the group level — access management scales easily as the team grows.

```powershell
New-ADGroup `
  -Name "Marketing" `
  -GroupScope Global `
  -GroupCategory Security `
  -Path "OU=Marketing,DC=mydomain,DC=com"
```

<img width="1918" height="886" alt="creating marketing security group" src="https://github.com/user-attachments/assets/bccceebb-6069-4284-b948-fb0a7bd440fc" />


---

## Step 5: Add Sarah to the Marketing Group

Added Sarah's account to the Marketing security group. Group membership controls her access to shared resources — any permissions granted to the Marketing group automatically apply to her account.

```powershell
Add-ADGroupMember -Identity "Marketing" -Members "smitchell"
```

<img width="1918" height="876" alt="add sarah to marketing security group" src="https://github.com/user-attachments/assets/68d7fd28-1794-448e-a0b7-b657bcf87d55" />


---

## Step 6: Create and Share the Marketing Folder

Created a dedicated folder on DC to serve as the Marketing department's shared network location. Shared it over the network using `New-SmbShare` and set permissions — Full Control for Domain Admins and Read for the Marketing group.

```powershell
New-Item -Path "C:\Shares\Marketing" -ItemType Directory
New-SmbShare -Name "MarketingShare" -Path "C:\Shares\Marketing" -FullAccess "Domain Admins" -ReadAccess "Marketing"
Get-SmbShareAccess -Name "MarketingShare"
```

<img width="1918" height="925" alt="creating market folder" src="https://github.com/user-attachments/assets/801968d8-2695-400d-8607-95da59f9bdc1" />
<img width="1918" height="867" alt="grant smb mydomain" src="https://github.com/user-attachments/assets/bb155565-097d-449b-9828-7d9d1ae20a15" />


---

## Step 7: Map the Network Drive on the Client

Mapped the M: drive on Client01 pointing to the Marketing share on DC. The `-Persist` flag ensures the mapping survives reboots and session changes.

```powershell
New-PSDrive -Name "M" -PSProvider FileSystem -Root "\\DC\MarketingShare" -Persist
Get-PSDrive -Name "M"
```

<img width="1703" height="880" alt="mapped share on client machine" src="https://github.com/user-attachments/assets/6e7e72f7-94ac-4b3c-bdae-064acfe31272" />
<img width="1600" height="873" alt="mapped drive confirmation" src="https://github.com/user-attachments/assets/9003f95e-4d10-4d00-a3f3-27c5ac515eab" />


---

## Step 8: Create and Link the Marketing Drive Mapping GPO

Created a GPO to automatically map the M: drive for all Marketing users at login. Deploying drive mappings through GPO eliminates manual configuration — any user in the Marketing OU receives the mapped drive automatically on any machine they log into.

```powershell
New-GPO -Name "Marketing Drive Mapping"
New-GPLink -Name "Marketing Drive Mapping" -Target "OU=Marketing,DC=mydomain,DC=com"
```

The drive mapping preference was configured inside the GPO using Group Policy Management Console:

*User Configuration → Preferences → Windows Settings → Drive Maps → New → Mapped Drive*
- **Action:** Create
- **Location:** \\DC\MarketingShare
- **Drive Letter:** M

<img width="1897" height="833" alt="created market shariing gpo" src="https://github.com/user-attachments/assets/d2aa90a8-5c5b-41c4-85be-f3679330b37f" />
<img width="1915" height="627" alt="linked gpo to OU" src="https://github.com/user-attachments/assets/326b4d2e-146d-4873-b5cd-f2b2f82c959c" />
<img width="1911" height="873" alt="gp management creating map" src="https://github.com/user-attachments/assets/20bd5082-6b99-4e2f-a5ad-28253e7e677d" />

---

## Step 9: Push GPO and Verify Application

Pushed the GPO to Client01 remotely using `Invoke-GPUpdate` and generated a Resultant Set of Policy report to confirm the Marketing Drive Mapping GPO was successfully applied.

```powershell
Invoke-GPUpdate -Computer "Client01" -Force
Get-GPResultantSetOfPolicy -Computer "Client01" -ReportType Html -Path "C:\Reports\RSoP.html"
```

**Troubleshooting Note:** `Invoke-GPUpdate` initially failed because the Remote Scheduled Tasks Management firewall rule was disabled on Client01. It was enabled using the following command before retrying:

```powershell
Enable-NetFirewallRule -DisplayName "Remote Scheduled Tasks Management (RPC)"
```

<img width="1913" height="910" alt="invoke share drive to client" src="https://github.com/user-attachments/assets/7d5810d3-5d55-468b-aaae-83e38273f877" />
<img width="1918" height="863" alt="gpresulant for mapped drive gpo" src="https://github.com/user-attachments/assets/375a2f38-8cdf-4332-b972-35f8b73116b2" />


---

## Step 10: Configure the Roaming Profile

Created a Profiles share on DC and configured Sarah's account with a roaming profile path. Roaming profiles store user settings on the server so they follow the user to any machine they log into — preventing data loss if a machine is reimaged.

```powershell
New-Item -Path "C:\Profiles" -ItemType Directory
New-SmbShare -Name "Profiles" -Path "C:\Profiles" -FullAccess "Domain Users"
Set-ADUser -Identity smitchell -ProfilePath "\\DC\Profiles\smitchell"
Get-ADUser -Identity smitchell -Properties ProfilePath | Select ProfilePath
```

<img width="1916" height="876" alt="profiles folder created" src="https://github.com/user-attachments/assets/c565470d-f003-41cf-b290-94c87aa46a0a" />
<img width="1166" height="587" alt="smb share profiles in profiles folder" src="https://github.com/user-attachments/assets/d5831845-2665-47ce-b909-03aa3cf29a85" />
<img width="1918" height="803" alt="set smitchell for profile share" src="https://github.com/user-attachments/assets/85269bfc-e0c1-4eb7-9e50-8754b06e0770" />
<img width="1918" height="672" alt="profile path confirmation for sarah" src="https://github.com/user-attachments/assets/f91ccea0-9dbd-40e8-9043-57786fd287f3" />

---

## Summary

| Task | Action |
|---|---|
| Create AD User | New-ADUser with correct OU placement and enabled flag |
| Verify Placement | Confirmed DistinguishedName pointed to Marketing OU |
| Force Password Change | Set ChangePasswordAtLogon to True |
| Create Security Group | New-ADGroup with Global scope and Security category |
| Add to Group | Add-ADGroupMember and verified with Get-ADGroupMember |
| Create SMB Share | New-SmbShare with correct permissions for Marketing group |
| Map Network Drive | New-PSDrive with -Persist flag on Client01 |
| Create Drive Mapping GPO | New-GPO, New-GPLink, and configured preference in GPMC |
| Push and Verify GPO | Invoke-GPUpdate and RSoP report confirmed application |
| Configure Roaming Profile | Profiles share created and ProfilePath set on user account |

This onboarding workflow demonstrates a complete end-to-end new hire provisioning process performed entirely in PowerShell — no GUI required.
