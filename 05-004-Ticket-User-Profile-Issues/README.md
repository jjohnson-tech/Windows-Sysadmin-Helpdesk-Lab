## 05 - Ticket #004 — User Profile Issues

A user's desktop settings, mapped drives, and personal files were wiped after their workstation was reimaged. The root cause was a local profile stored only on the machine — which was lost during the reimage. A roaming profile was configured so the user's settings are stored on the server and follow them to any machine they log into.

---

## Step 1: Understand the Root Cause

When a machine is reimaged, everything stored locally is wiped — including the user's local profile. A local profile contains the user's desktop settings, Documents folder, AppData, and mapped drive configurations. If the profile is not stored on a server, all of this is permanently lost during a reimage.

The solution is a **roaming profile** — the user's profile is stored on a network share on DC instead of the local machine. Every time the user logs in, their profile syncs from the server to the machine. Every time they log out, it syncs back. This means their settings follow them to any machine and survive reimaging.

---

## Step 2: Create the Profiles Share on DC

A dedicated Profiles share was created on DC to store roaming profile data for domain users. Domain Users were granted Full Control so each user can read and write their own profile data to the server.

```powershell
New-Item -Path "C:\Shares\Profiles" -ItemType Directory

New-SmbShare -Name "Profiles" `
  -Path "C:\Shares\Profiles" `

Grant-SmbShareAccess -Name "Profiles" -AccountName "Domain Users" -AccessRight Full Force
```

<img width="1916" height="876" alt="profiles folder created" src="https://github.com/user-attachments/assets/cbc6f4b3-b6cd-462b-8e39-90c84c38267a" />
<img width="1166" height="587" alt="smb share profiles in profiles folder" src="https://github.com/user-attachments/assets/4a17f2e6-bcca-4445-b25f-ec8981d1ded4" />
<img width="1897" height="847" alt="grant smb access for profile shares" src="https://github.com/user-attachments/assets/06ab7b6b-4939-44ab-b815-224c67ac3158" />


---

## Step 3: Set the Roaming Profile Path on the User Account

The user's AD account was updated with a roaming profile path pointing to their dedicated folder on the Profiles share. Windows automatically creates the profile folder on first login and syncs the user's settings to the server going forward.

```powershell
Set-ADUser -Identity smitchell -ProfilePath "\\DC\Profiles\smitchell"
```

The profile path was verified to confirm it was set correctly on the account:

```powershell
Get-ADUser -Identity smitchell -Properties ProfilePath | Select Name, ProfilePath
```

<img width="1918" height="803" alt="set smitchell for profile share" src="https://github.com/user-attachments/assets/33c26279-04d8-4a80-8a45-2e9036232368" />
<img width="1918" height="672" alt="profile path confirmation for sarah" src="https://github.com/user-attachments/assets/a5a4feff-be58-4071-b8b2-ef251253f02b" />


---

## Step 4: Verify Profile Folder Creation After First Login

After the profile path was set, the user logged into Client01 for the first time. Windows automatically created the profile folder on the server. The folder was verified on DC to confirm the roaming profile was working correctly.

```powershell
Get-ChildItem -Path "C:\Profiles"
```

---

## Summary

| Task | Command Used |
|---|---|
| Create Profiles share | New-SmbShare with Domain Users Full Control |
| Set profile path | Set-ADUser -ProfilePath \\DC\Profiles\smitchell |
| Verify profile path | Get-ADUser -Properties ProfilePath |
| Confirm folder creation | Get-ChildItem -Path C:\Profiles |

**Common causes of profile issues in domain environments:**

| Issue | Cause |
|---|---|
| Profile wiped after reimage | Local profile — not stored on server |
| Temporary profile at login | Network issue, corrupted profile, or insufficient share permissions |
| Settings not following user | Profile path not set on the AD account |
| Profile not syncing | Domain Users missing Full Control on the Profiles share |

Roaming profiles store user settings on the server so they follow the user to any machine. Always configure roaming profiles for domain users in enterprise environments to prevent data loss during hardware changes or reimaging.
