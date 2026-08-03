# Microsoft Entra ID and On-Premises Active Directory Synchronization

**Microsoft Entra ID | Active Directory Domain Services | Password Hash Synchronization | PowerShell**

## Project Overview

I configured hybrid identity synchronization between an on-premises Active Directory Domain Services environment and Microsoft Entra ID using Microsoft Entra Connect Sync.

I prepared the Windows Server for secure communication, installed Microsoft Entra Connect, configured Password Hash Synchronization using Express Settings, connected the on-premises AD DS forest to the Microsoft Entra tenant, initiated the first synchronization cycle, and verified that the directory users appeared in Microsoft Entra ID.

This project demonstrates practical experience with hybrid identity, directory synchronization, administrative authentication, User Principal Name planning, and synchronization validation.

## Business Scenario

An organization manages employee accounts in an on-premises Active Directory domain but also needs those identities available in Microsoft cloud services.

Creating separate cloud accounts would increase administrative work and could lead to inconsistent identities. Microsoft Entra Connect Sync provides a centralized approach by synchronizing identity information from AD DS to Microsoft Entra ID.

## Architecture

`On-Premises AD DS → Microsoft Entra Connect Sync → Microsoft Entra ID`

Microsoft Entra Connect acts as the synchronization bridge between the local Active Directory forest and the cloud tenant.

## Project Objectives

- Prepare the synchronization server for TLS 1.2 communication.
- Download and install Microsoft Entra Connect Sync.
- Connect Microsoft Entra ID and the on-premises AD DS forest.
- Configure Password Hash Synchronization.
- Review the relationship between on-premises UPN suffixes and verified cloud domains.
- Start the initial directory synchronization.
- Verify synchronized user objects in Microsoft Entra ID.
- Document installation and identity-design considerations.

## Lab Environment

| Component | Technology |
| --- | --- |
| On-premises directory | Active Directory Domain Services |
| On-premises forest | `adeniyi.com` |
| Cloud directory | Microsoft Entra ID |
| Synchronization tool | Microsoft Entra Connect Sync |
| Sign-in method | Password Hash Synchronization |
| Installation type | Express Settings |
| Server administration | Windows PowerShell |
| Cloud administration | Microsoft Entra admin center |

## Skills Demonstrated

- Hybrid identity implementation
- Microsoft Entra Connect installation
- Active Directory and Entra ID integration
- Password Hash Synchronization
- TLS 1.2 and Windows registry configuration
- PowerShell administration
- Identity and access administration
- UPN suffix and verified-domain planning
- Synchronization troubleshooting
- Cloud-directory validation

## Implementation

### 1. Prepared the Synchronization Server

I opened Windows PowerShell as an administrator on the Windows Server that would host Microsoft Entra Connect Sync.

<p align="center">
  <img src="https://i.imgur.com/s7SsVsp.png" width="700" alt="Opening Windows PowerShell as an administrator on the synchronization server">
</p>

I then configured the required .NET Framework and SCHANNEL registry settings for TLS 1.2.

```powershell
# TLS 1.2 Registry Key

New-Item 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -Force | Out-Null

New-ItemProperty -path 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -name 'SystemDefaultTlsVersions' -value '1' -PropertyType 'DWord' -Force | Out-Null

New-ItemProperty -path 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -name 'SchUseStrongCrypto' -value '1' -PropertyType 'DWord' -Force | Out-Null

New-Item 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -Force | Out-Null

New-ItemProperty -path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -name 'SystemDefaultTlsVersions' -value '1' -PropertyType 'DWord' -Force | Out-Null

New-ItemProperty -path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -name 'SchUseStrongCrypto' -value '1' -PropertyType 'DWord' -Force | Out-Null

New-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -Force | Out-Null

New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -name 'Enabled' -value '1' -PropertyType 'DWord' -Force | Out-Null

New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -name 'DisabledByDefault' -value 0 -PropertyType 'DWord' -Force | Out-Null

New-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -Force | Out-Null

New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -name 'Enabled' -value '1' -PropertyType 'DWord' -Force | Out-Null

New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -name 'DisabledByDefault' -value 0 -PropertyType 'DWord' -Force | Out-Null

Write-Host 'TLS 1.2 has been enabled.'
```


I confirmed that the script completed and displayed the TLS 1.2 success message.

<p align="center">
  <img src="https://i.imgur.com/nbr63c5.png" width="700" alt="PowerShell output confirming that TLS 1.2 was enabled">
</p>



### 2. Downloaded Microsoft Entra Connect Sync

From the Microsoft Entra admin center, I opened **Microsoft Entra Connect** and selected **Connect Sync**.

<p align="center">
  <img src="https://i.imgur.com/tWWoY8s.png" width="700" alt="Opening Microsoft Entra Connect from the Microsoft Entra admin center">
</p>

I selected **Download Microsoft Entra Connect** and saved the installation package to the Windows Server.

<p align="center">
  <img src="https://i.imgur.com/0tu6Muj.png" width="700" alt="Downloading Microsoft Entra Connect Sync from the Connect Sync page">
</p>

### 3. Installed Microsoft Entra Connect

I located the downloaded `AzureADConnect` installer and launched it with administrative privileges.

<p align="center">
  <img src="https://i.imgur.com/9OuWQsC.png" width="700" alt="Microsoft Entra Connect installation package on the Windows Server">
</p>

The setup wizard copied the required Microsoft Entra Connect files to the server.

<p align="center">
  <img src="https://i.imgur.com/cnX0rde.png" width="700" alt="Microsoft Entra Connect installation in progress">
</p>

#### Installation Troubleshooting

An initial setup attempt displayed a message stating that the installation had been interrupted.

<p align="center">
  <img src="https://i.imgur.com/XKNneoA.png" width="700" alt="Initial Microsoft Entra Connect setup interruption">
</p>

I did not treat the interrupted wizard as a successful installation. After checking the server prerequisites and rerunning the setup, I confirmed that Microsoft Entra Connect opened successfully and continued with the configuration.

### 4. Selected Express Settings

I opened Microsoft Entra Connect, accepted the license terms and privacy notice, and continued to the configuration wizard.

<p align="center">
  <img src="https://i.imgur.com/2l0mpG4.png" width="700" alt="Accepting the Microsoft Entra Connect license terms">
</p>

Because this was a single-forest lab environment, I selected **Use express settings**.

<p align="center">
  <img src="https://i.imgur.com/gin1nDO.png" width="700" alt="Selecting Express Settings in Microsoft Entra Connect">
</p>

Express Settings configured:

- Synchronization for the current AD DS forest
- Password Hash Synchronization
- Initial synchronization
- Synchronization of supported identity attributes
- Automatic upgrade

### 5. Connected Microsoft Entra ID

On the **Connect to Microsoft Entra ID** page, I entered the credentials for an account with the required hybrid identity administration permissions.

<p align="center">
  <img src="https://i.imgur.com/5myYPYL.png" width="700" alt="Connecting Microsoft Entra Connect to Microsoft Entra ID">
</p>

### 6. Connected Active Directory Domain Services

On the **Connect to AD DS** page, I entered the credentials for an account with Enterprise Administrator permissions in the on-premises forest.

<p align="center">
  <img src="https://i.imgur.com/DqnEFzf.png" width="700" alt="Connecting Microsoft Entra Connect to Active Directory Domain Services">
</p>

This allowed Microsoft Entra Connect to read the on-premises directory and configure the required synchronization components.

### 7. Reviewed the UPN Suffix Configuration

The Microsoft Entra sign-in configuration page showed that the on-premises UPN suffix `adeniyi.com` had not been added as a verified custom domain in Microsoft Entra ID.

<p align="center">
  <img src="https://i.imgur.com/WwcyTpt.png" width="700" alt="Microsoft Entra sign-in configuration showing an unmatched UPN suffix">
</p>

For this lab, I continued without matching every UPN suffix to a verified domain. This allowed the directory objects to synchronize, but Microsoft Entra ID used the tenant’s default `onmicrosoft.com` suffix for the affected cloud UPNs.


### 8. Started the Initial Synchronization

On the **Ready to configure** page, I reviewed the selected configuration and left **Start the synchronization process when configuration completes** enabled.

<p align="center">
  <img src="https://i.imgur.com/JWn6QYE.png" width="700" alt="Microsoft Entra Connect ready-to-configure page">
</p>

I selected **Install**, which:

- Installed the synchronization engine
- Configured the Microsoft Entra connector
- Configured the AD DS connector
- Enabled Password Hash Synchronization
- Enabled automatic upgrade
- Started the initial synchronization cycle

### 9. Confirmed Configuration Completion

The wizard displayed **Configuration complete**, confirming that Microsoft Entra Connect had been configured and the synchronization process had been initiated.

<p align="center">
  <img src="https://i.imgur.com/rISwTyF.png" width="700" alt="Microsoft Entra Connect configuration-complete page">
</p>

The completion screen also identified that Active Directory Recycle Bin was not enabled in the lab forest.


### 10. Verified Synchronized Users

In the Microsoft Entra admin center, I opened **Microsoft Entra ID > Users** and confirmed that the on-premises directory users appeared in the cloud tenant.

<p align="center">
  <img src="https://i.imgur.com/MSWBAWQ.png" width="700" alt="Synchronized Active Directory users displayed in Microsoft Entra ID">
</p>

The **On-premises sync enabled** value confirmed that the highlighted accounts originated from the local Active Directory environment.

## Validation Results

I confirmed that:

- TLS 1.2 was enabled on the synchronization server.
- Microsoft Entra Connect was installed and configured.
- The Entra tenant connection completed successfully.
- The on-premises AD DS forest connection completed successfully.
- Password Hash Synchronization was enabled.
- The initial synchronization cycle was started.
- On-premises user objects appeared in Microsoft Entra ID.
- Synchronized users were identified as originating from on-premises AD DS.
- The unmatched UPN-suffix warning was reviewed and documented.

## Troubleshooting Reference

| Issue | Recommended check |
| --- | --- |
| Setup wizard is interrupted | Confirm TLS 1.2, server prerequisites, local administrator access, and network connectivity before rerunning the installer |
| Cloud authentication fails | Confirm the Microsoft Entra account has the required hybrid identity administration permissions |
| AD DS authentication fails | Confirm the account has Enterprise Administrator permissions and the domain name is entered correctly |
| UPN suffix displays `Not Added` | Add and verify the corresponding custom domain in Microsoft Entra ID or update the on-premises UPN suffix |
| Users do not appear in Entra ID | Check the synchronization service, directory scope, connector status, and network connectivity |
| Users synchronize but cannot sign in | Review the cloud UPN, verified-domain status, and Password Hash Synchronization status |

## Key Takeaways

This project strengthened my understanding of how on-premises identities are extended into Microsoft Entra ID.

Successful hybrid identity configuration depends on more than installing the synchronization tool. The server must meet security prerequisites, both directories must be authenticated correctly, the sign-in domain must be planned, and the synchronized objects must be validated in the cloud tenant.

The project also demonstrated the importance of distinguishing between successful object synchronization and a complete sign-in experience. Directory objects can synchronize even when the on-premises UPN suffix is not verified, but users may receive a different cloud UPN until the custom domain is added and verified.


## Related Projects

- [Active Directory Implementation](https://github.com/AdeniyiAdesakin/Active-Directory-Implementation)
- [Active Directory Domain Services and Windows Client Integration](https://github.com/AdeniyiAdesakin/Install-Active-Directory-Domain-Services-and-Join-Client-s-Computer-to-Active-Directory)

