## 04 - Ticket #003 — Printer Setup & Deployment

The Marketing department received a new network printer that needed to be deployed to all Marketing users automatically. The Print and Document Services role was installed on DC, the printer was configured and shared, and a GPO preference was created to map the printer to all Marketing users at login.

---

## Step 1: Install the Print and Document Services Role

The Print and Document Services role was installed on DC to enable printer sharing and centralized print management. This role is required before any printers can be shared over the network.

```powershell
Install-WindowsFeature -Name Print-Services -IncludeManagementTools
```

<img width="1918" height="840" alt="install print server" src="https://github.com/user-attachments/assets/6cb8a66c-751a-483e-8854-6baa6c650534" />

---

## Step 2: Create the Printer Port

A printer port was created pointing to the network printer's IP address. The port is the connection point between the print server and the physical printer device.

```powershell
Add-PrinterPort -Name "IP_192.168.1.100" -PrinterHostAddress "192.168.1.100"
```

<img width="1918" height="877" alt="add printer port and add printer" src="https://github.com/user-attachments/assets/d83be759-109a-4283-8b3c-f049c0cc5b6b" />


---

## Step 3: Add the Printer

The printer object was created on top of the port using the Generic / Text Only driver.

```powershell
Add-Printer -Name "Marketing-Printer" -DriverName "Generic / Text Only" -PortName "IP_192.168.1.100"
```

**Troubleshooting Note:** Microsoft Print To PDF was initially used as the driver but does not support network sharing. The Generic / Text Only driver was installed as a replacement since it supports sharing over the network.

<img width="1917" height="837" alt="adding printer server port driver" src="https://github.com/user-attachments/assets/71fc2d57-336a-40b4-b583-86bd08530d38" />


---

## Step 4: Share the Printer

The printer was shared via the Print Management GUI so it could be accessed over the network. PowerShell's `Set-Printer` command returned an error during sharing so the GUI was used as an alternative to complete the task.

*Print Management → Print Servers → DC → Printers → Marketing-Printer → Properties → Sharing → Share this printer → Share Name: MarketingPrinter*

<img width="1595" height="832" alt="printershare print management console" src="https://github.com/user-attachments/assets/9e87e3db-018e-45c5-9388-0b7392d5103b" />


---

## Step 5: Deploy the Printer via GPO

A GPO preference was configured to automatically map the Marketing printer to all users in the Marketing OU at login. Using GPO for printer deployment eliminates the need for users to manually install the printer on each machine they use.

The printer deployment was configured in Group Policy Management Console:

*User Configuration → Preferences → Control Panel Settings → Printers → New → Shared Printer*
- **Action:** Create
- **Share path:** \\DC\MarketingPrinter

<img width="1837" height="761" alt="set printer as default for marketprinter" src="https://github.com/user-attachments/assets/2dee37ab-880a-4cdd-bce5-113abd58539b" />


---

## Step 6: Push the GPO to the Client

`gpupdate /force` was run directly on Client01 to trigger an immediate Group Policy refresh. Printer preferences apply at user login — the user must log off and back on after the GPO is pushed to receive the mapped printer.

```powershell
Invoke-GPUpdate -Computer "Client01" -Force
```

<img width="1905" height="858" alt="Invoke gpudpate for print server" src="https://github.com/user-attachments/assets/8b77d6e2-5017-4c88-bf5b-08227635889f" />


---

## Summary

| Task | Action |
|---|---|
| Install Print Services | Install-WindowsFeature -Name Print-Services |
| Create Printer Port | Add-PrinterPort pointing to printer IP address |
| Add Printer | Add-Printer with Generic / Text Only driver |
| Share Printer | Print Management GUI — Share this printer as MarketingPrinter |
| Deploy via GPO | GPMC — User Configuration → Preferences → Printers → Shared Printer |
| Push GPO | Invoke-GPUpdate -Computer Client01 -Force |

Printer GPO preferences apply at user login — users must log off and back on to receive the mapped printer. Always verify the driver supports network sharing before attempting to share. GPO software deployment applies at startup while GPO printer preferences apply at user login — understanding this difference prevents unnecessary troubleshooting.
