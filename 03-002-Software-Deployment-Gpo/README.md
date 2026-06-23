## 03 - Ticket #002 — Software Deployment via GPO

The IT department needed 7-Zip deployed to all machines in the IT OU without touching each machine individually. A software distribution share was created on DC, the MSI was downloaded, and a GPO was configured to push the installation automatically at machine startup.

---

## Step 1: Create the Software Distribution Share

A dedicated shared folder was created on DC to host software packages for deployment. Domain Computers were granted Read access so client machines can pull the MSI during GPO processing at startup.

```powershell
New-Item -Path "C:\Shares\Software" -ItemType Directory
New-SmbShare -Name "Software" -Path "C:\Shares\Software" -ReadAccess "Domain Computers"
```

<img width="1641" height="841" alt="software deployment folder created" src="https://github.com/user-attachments/assets/eccfd68d-b7c6-47d9-9cdc-e667ceb35bb0" />
<img width="1917" height="641" alt="software deployment share in share folder" src="https://github.com/user-attachments/assets/bc8861fb-4573-415b-99ef-6db5516b4f68" />


---

## Step 2: Download the 7-Zip MSI

The 7-Zip MSI was downloaded directly to the software share using `Invoke-WebRequest`. GPO software deployment requires an MSI file — standard .exe installers cannot be deployed silently through Group Policy because they require user interaction.

```powershell
Invoke-WebRequest -Uri "https://www.7-zip.org/a/7z2301-x64.msi" -OutFile "C:\Shares\Software\7zip.msi"
```

<img width="1917" height="867" alt="invoke msi file for software deploy" src="https://github.com/user-attachments/assets/17b4eadb-6ddb-4e85-9a5e-88832bc79b41" />


---

## Step 3: Create and Link the Deployment GPO

A new GPO was created and linked to the IT OU so it only applies to machines in that container. Scoping the GPO to a specific OU prevents the software from deploying to machines outside the IT department.

```powershell
New-GPO -Name "Deploy 7-Zip"
New-GPLink -Name "Deploy 7-Zip" -Target "OU=IT,DC=mydomain,DC=com"
```

The software installation was configured inside the GPO using Group Policy Management Console:

*Computer Configuration → Policies → Software Settings → Software Installation → New Package → \\DC\Software\7zip.msi → Assigned*

<img width="1917" height="851" alt="gpo creation and link software deployment" src="https://github.com/user-attachments/assets/87e100db-09df-472c-b35c-b7a2f84e4003" />


---

## Step 4: Push the GPO to the Client

`Invoke-GPUpdate` was used to remotely force the client to pull the latest Group Policy from the domain controller. This triggers the policy refresh without waiting for the next scheduled background refresh.

```powershell
Invoke-GPUpdate -Computer "Client01" -Force
```

After the GPO was pushed, Client01 was restarted so the software installation could process at startup. GPO software deployment applies at machine startup — a client restart is always required after the policy is pushed.

<img width="1917" height="891" alt="invoke gpupdate for software deployment" src="https://github.com/user-attachments/assets/942a995e-b5fe-4014-bfc5-5dbac5ac957a" />


---

## Step 5: Verify the Installation

After restarting Client01, the GPO processed at startup and installed 7-Zip silently in the background. The installation was verified using `Get-Package` to confirm the software was present on the machine.

```powershell
Get-Package -Name "7-Zip*"
```

<img width="1595" height="882" alt="Verified software deployement" src="https://github.com/user-attachments/assets/ce5eab66-dad0-44e8-9517-5ff0f9ade16e" />


---

## Summary

| Task | Command Used |
|---|---|
| Create software share | New-SmbShare with Domain Computers Read access |
| Download MSI | Invoke-WebRequest to pull MSI directly to the share |
| Create GPO | New-GPO and New-GPLink scoped to IT OU |
| Configure software install | GPMC — Computer Configuration → Software Settings → Assigned |
| Push GPO | Invoke-GPUpdate -Computer Client01 -Force |
| Verify installation | Get-Package -Name "7-Zip*" |

GPO software deployment requires MSI files — .exe installers cannot be deployed silently through Group Policy. Always scope the GPO to the correct OU, restart the client after pushing the policy, and verify the computer account is in the correct OU if the GPO is not applying as expected.
