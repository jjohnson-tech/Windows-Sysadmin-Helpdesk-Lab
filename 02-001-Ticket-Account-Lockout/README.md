## 02 - Ticket #001 — Account Lockout

A user was locked out of their domain account after multiple failed login attempts. The task was to investigate the lockout, unlock the account, reset the password, and verify the user could log back in — all performed via PowerShell on DC.

---

## Step 1: Check the Domain Lockout Policy

Before taking any action, the domain lockout policy was reviewed to understand the threshold that triggered the lockout and how long the lockout duration lasts.

```powershell
Get-ADDefaultDomainPasswordPolicy | Select LockoutThreshold, LockoutDuration, LockoutObservationWindow
```

<img width="1918" height="823" alt="password lockout policy observation" src="https://github.com/user-attachments/assets/1cc6de37-a8a3-4aac-a025-d0d1e4cc69dd" />


---

## Step 2: Confirm the Account Was Locked

`Search-ADAccount` was used to find all currently locked accounts in the domain. This confirms which account triggered the lockout before taking any action.

```powershell
Search-ADAccount -LockedOut 
```

The `LockedOut` property was also queried directly on the affected account to confirm its status:

```powershell
Get-ADUser -Identity smitchell -Properties LockedOut | Select Name, LockedOut
```

<img width="1918" height="815" alt="identify lockedout accounts" src="https://github.com/user-attachments/assets/15e2d030-2593-49f6-9a43-893328c82d83" />


---

## Step 3: Unlock the Account

`Unlock-ADAccount` was used to immediately restore access to the locked account without resetting the password.

```powershell
Unlock-ADAccount -Identity smitchell
```

After unlocking, the `LockedOut` property was queried again to verify the account was successfully unlocked:

```powershell
Get-ADUser -Identity smitchell -Properties LockedOut | Select Name, LockedOut
```

---

## Step 4: Reset the Password and Force Change at Next Login

A temporary password was set on the account and the user was required to change it at next login. This ensures only the user knows their final password — a security best practice after any account unlock.

```powershell
Set-ADAccountPassword -Identity smitchell `
  -Reset `
  -NewPassword (ConvertTo-SecureString "TempPass123!" -AsPlainText -Force)

Set-ADUser -Identity smitchell -ChangePasswordAtLogon $true
```

<img width="1906" height="812" alt="reset password and changepassword for sara" src="https://github.com/user-attachments/assets/755c1cbc-f5ce-4e27-bca8-c778930c574c" />


---

## Summary

| Task | Command Used |
|---|---|
| Check lockout policy | Get-ADDefaultDomainPasswordPolicy |
| Find locked accounts | Search-ADAccount -LockedOut |
| Confirm lockout status | Get-ADUser -Properties LockedOut |
| Unlock the account | Unlock-ADAccount |
| Reset password | Set-ADAccountPassword -Reset |
| Force password change | Set-ADUser -ChangePasswordAtLogon $true |

Account lockouts are one of the most frequent helpdesk tickets in domain environments. Always verify the lockout policy first to understand the threshold, confirm the locked account before unlocking, and always force a password change after any reset so only the user knows their final credentials. Verify the user can log in successfully before closing the ticket.
