## 07 - Ticket #006 — Permission Denied on Shared Folder

A user suddenly lost access to the Marketing shared folder they had been using without issue. The troubleshooting process covered both layers of Windows permissions — Share and NTFS — as well as group membership verification and security token behavior. The root cause was identified as a stale security token rather than a permissions misconfiguration.

---

## Step 1: Check Group Membership

The first step was to verify the user was still a member of the Marketing security group. Group membership controls access to shared resources — if a user is removed from the group their access is revoked immediately regardless of their account status.

```powershell
Get-ADPrincipalGroupMembership -Identity smitchell | Select Name
```

<img width="1912" height="851" alt="get membership sarah" src="https://github.com/user-attachments/assets/06c24c02-332c-4577-88aa-5b0421b42517" />


---

## Step 2: Check NTFS Permissions

NTFS permissions control access at the file system level on the actual folder on disk. These were checked to confirm the Marketing group had the correct access rights on the folder itself.

```powershell
Get-Acl -Path "C:\Shares\Marketing" | Format-List
```

<img width="1917" height="892" alt="Get-Acl permissions for marketing" src="https://github.com/user-attachments/assets/fa16ea28-8acf-405c-bc4c-cc3ee9c14b5e" />


---

## Step 3: Check Share Permissions

Share permissions control access to the folder over the network. Both Share and NTFS permissions must allow access — if either layer denies, the user receives Access Denied. Share permissions were checked to confirm the Marketing group had the correct network access.

```powershell
Get-SmbShareAccess -Name "MarketingShare"
```

<img width="1915" height="558" alt="get-smbshare for share permissions for sarah access denited" src="https://github.com/user-attachments/assets/7a26d19c-e1a6-4e0d-b4af-4548a7f1de6f" />


---

## Step 4: Verify Share Connectivity

With both permission layers confirmed as correct, the share was tested directly from Client01 to verify it was reachable over the network. Active SMB sessions were also checked from DC to confirm the client was connecting to the server.

```powershell
# Run on Client01
Test-Path -Path "\\DC\MarketingShare"

# Run on DC
Get-SmbSession
```

<img width="1901" height="907" alt="testing to see if share shows true on client" src="https://github.com/user-attachments/assets/0bccec7d-ca40-4453-8cd2-9aca74aecf40" />
<img width="1917" height="857" alt="Get-smbsession checking the session for permission cache" src="https://github.com/user-attachments/assets/33d9b22c-3adb-4924-bd0f-a34b1371f2a9" />


---

## Step 5: Identify Root Cause — Stale Security Token

With group membership, NTFS permissions, share permissions, and network connectivity all confirmed correct, the root cause was identified as a **stale security token**.

When a user logs in, Windows creates a security token containing all their group memberships at that moment. If group membership changes while the user is already logged in, the token does not update until they log off and back on — even if the permissions are correctly configured in Active Directory.

The fix was simple — the user logged off and logged back on, refreshing their security token to include the current group memberships. Access to the Marketing share was restored immediately.

![User logged off and back on — security token refreshed and access to MarketingShare restored](images/05-access-restored.png)

---

## Summary

| Step | Command Used | Finding |
|---|---|---|
| Check group membership | Get-ADPrincipalGroupMembership | User was in the Marketing group — not the cause |
| Check NTFS permissions | Get-Acl | Permissions correctly configured — not the cause |
| Check share permissions | Get-SmbShareAccess | Marketing group had Read access — not the cause |
| Test share connectivity | Test-Path and Get-SmbSession | Share reachable over the network — not the cause |
| Root cause identified | — | Stale security token — resolved by log off and log on |

**Key lesson:** Always ask the user to log off and back on before modifying any permissions. Security tokens are created at login and do not update mid-session — this resolves the majority of Access Denied tickets in domain environments without any permission changes at all.

**Windows permission layers:**

| Layer | Controls | Checked With |
|---|---|---|
| Share permissions | Network access to the shared folder | Get-SmbShareAccess |
| NTFS permissions | File system access on the actual folder | Get-Acl |

Both layers must allow access. If either layer denies, the user receives Access Denied — always check both before drawing conclusions.
