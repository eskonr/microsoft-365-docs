---
title: McAfee to Microsoft Defender for Endpoint - Setup
description: This is phase 2, Setup, for migrating from McAfee to Microsoft Defender for Endpoint.
keywords: migration, Microsoft Defender for Endpoint, edr
search.product: eADQiWindows 10XVcnh
search.appverid: met150
ms.prod: m365-security
ms.technology: mde
ms.mktglfcycl: deploy
ms.sitesec: library
ms.pagetype: security
ms.author: deniseb
author: denisebmsft
localization_priority: Normal
manager: dansimp
audience: ITPro
ms.collection: 
  - M365-security-compliance
  - m365solution-mcafeemigrate
  - m365solution-scenario
ms.topic: article
ms.custom: migrationguides
ms.date: 03/03/2021
ms.reviewer: jesquive, chventou, jonix, chriggs, owtho
---

# Migrate from McAfee - Phase 2: Set up Microsoft Defender for Endpoint

**Applies to:**
- [Microsoft Defender for Endpoint](https://go.microsoft.com/fwlink/p/?linkid=2154037)
- [Microsoft 365 Defender](https://go.microsoft.com/fwlink/?linkid=2118804)

|[![Phase 1: Prepare](images/phase-diagrams/prepare.png)](mcafee-to-microsoft-defender-prepare.md)<br/>[Phase 1: Prepare](mcafee-to-microsoft-defender-prepare.md) |![Phase 2: Set up](images/phase-diagrams/setup.png)<br/>Phase 2: Set up |[![Phase 3: Onboard](images/phase-diagrams/onboard.png)](mcafee-to-microsoft-defender-onboard.md)<br/>[Phase 3: Onboard](mcafee-to-microsoft-defender-onboard.md) |
|--|--|--|
||*You are here!* | |

**Welcome to the Setup phase of [migrating from McAfee Endpoint Security (McAfee) to Microsoft Defender for Endpoint](mcafee-to-microsoft-defender-migration.md#the-migration-process)**. This phase includes the following steps:
1. [Enable Microsoft Defender Antivirus and confirm it's in passive mode](#enable-microsoft-defender-antivirus-and-confirm-its-in-passive-mode).
2. [Get updates for Microsoft Defender Antivirus](#get-updates-for-microsoft-defender-antivirus).
3. [Add Microsoft Defender for Endpoint to the exclusion list for McAfee](#add-microsoft-defender-for-endpoint-to-the-exclusion-list-for-mcafee).
4. [Add McAfee to the exclusion list for Microsoft Defender Antivirus](#add-mcafee-to-the-exclusion-list-for-microsoft-defender-antivirus).
5. [Add McAfee to the exclusion list for Microsoft Defender for Endpoint](#add-mcafee-to-the-exclusion-list-for-microsoft-defender-for-endpoint).
6. [Set up your device groups, device collections, and organizational units](#set-up-your-device-groups-device-collections-and-organizational-units).
7. [Configure antimalware policies and real-time protection](#configure-antimalware-policies-and-real-time-protection).

## Enable Microsoft Defender Antivirus and confirm it's in passive mode

On certain versions of Windows, such as Windows Server, Microsoft Defender Antivirus might have been uninstalled or disabled when your McAfee solution was installed. This is because Microsoft Defender Antivirus does not enter passive or disabled mode when you install a third-party antivirus product, such as McAfee. (To learn more about this, see [Microsoft Defender Antivirus compatibility](microsoft-defender-antivirus-compatibility.md).)

This step of the migration process includes the following tasks:
- [Setting DisableAntiSpyware to false on Windows Server](#set-disableantispyware-to-false-on-windows-server)
- [Reinstalling Microsoft Defender Antivirus on Windows Server](#reinstall-microsoft-defender-antivirus-on-windows-server);
- [Setting Microsoft Defender Antivirus to passive mode on Windows Server](#set-microsoft-defender-antivirus-to-passive-mode-on-windows-server)
- [Enabling Microsoft Defender Antivirus on your Windows client devices](#enable-microsoft-defender-antivirus-on-your-windows-client-devices); and
- [Confirming that Microsoft Defender Antivirus is set to passive mode](#confirm-that-microsoft-defender-antivirus-is-in-passive-mode).  

### Set DisableAntiSpyware to false on Windows Server

The [DisableAntiSpyware](/windows-hardware/customize/desktop/unattend/security-malware-windows-defender-disableantispyware) registry key was used in the past to disable Microsoft Defender Antivirus, and deploy another antivirus product, such as McAfee. In general, you should not have this registry key on your Windows devices and endpoints; however, if you do have `DisableAntiSpyware` configured, here's how to set its value to false:

1. On your Windows Server device, open Registry Editor.

2. Navigate to `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Defender`.

3. In that folder, look for a DWORD entry called **DisableAntiSpyware**.

   - If you do not see that entry, you're all set.

   - If you do see **DisableAntiSpyware**, proceed to step 4.

4. Right-click the DisableAntiSpyware DWORD, and then choose **Modify**.

5. Set the value to `0`. (This sets the registry key's value to *false*.)

> [!TIP]
> To learn more about this registry key, see [DisableAntiSpyware](/windows-hardware/customize/desktop/unattend/security-malware-windows-defender-disableantispyware).

### Reinstall Microsoft Defender Antivirus on Windows Server

> [!NOTE]
> The following procedure applies only to endpoints or devices that are running the following versions of Windows:
> - Windows Server 2019
> - Windows Server, version 1803 (core-only mode)
> - Windows Server 2016

1. As a local administrator on the endpoint or device, open Windows PowerShell.

2. Run the following PowerShell cmdlets: <br/>
   
   `Dism /online /Get-FeatureInfo /FeatureName:Windows-Defender-Features` <br/>
   
   `Dism /online /Get-FeatureInfo /FeatureName:Windows-Defender` <br/>

> [!NOTE]
> When using the DISM command within a task sequence running PS, the following path to cmd.exe is required.
> Example:<br/>
> `c:\windows\sysnative\cmd.exe /c Dism /online /Get-FeatureInfo /FeatureName:Windows-Defender-Features`<br/>
> `c:\windows\sysnative\cmd.exe /c Dism /online /Get-FeatureInfo /FeatureName:Windows-Defender`<br/>

3. To verify Microsoft Defender Antivirus is running, use the following PowerShell cmdlet: <br/>
   
   `Get-Service -Name windefend`

#### Are you using Windows Server 2016?

If you're using Windows Server 2016 and are having trouble enabling Microsoft Defender Antivirus, use the following PowerShell cmdlet:

`mpcmdrun -wdenable`

> [!TIP]
> Still need help? See [Microsoft Defender Antivirus on Windows Server 2016 and 2019](microsoft-defender-antivirus-on-windows-server.md).

### Set Microsoft Defender Antivirus to passive mode on Windows Server

Because your organization is still using McAfee, you must set Microsoft Defender Antivirus to passive mode. That way, McAfee and Microsoft Defender Antivirus can run side by side until you have finished onboarding to Microsoft Defender for Endpoint.

1. Open Registry Editor, and then navigate to <br/>
   `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Advanced Threat Protection`.

2. Edit (or create) a DWORD entry called **ForcePassiveMode**, and specify the following settings:
   
   - Set the DWORD's value to **1**.
   
   - Under **Base**, select **Hexadecimal**.

> [!NOTE]
> You can use other methods to set the registry key, such as the following:
>- [Group Policy Preference](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn581922(v=ws.11))
>- [A package in Configuration Manager](/mem/configmgr/apps/deploy-use/packages-and-programs)

### Enable Microsoft Defender Antivirus on your Windows client devices

Because your organization has been using McAfee as your primary antivirus solution, Microsoft Defender Antivirus is most likely disabled on your organization's Windows devices. This step of the migration process involves enabling Microsoft Defender Antivirus. 

To enable Microsoft Defender Antivirus, we recommend using Intune. However, you can any of the methods that are listed in the following table:

|Method  |What to do  |
|---------|---------|
|[Intune](/mem/intune/fundamentals/tutorial-walkthrough-endpoint-manager) <p>**NOTE**: Intune is now Microsoft Endpoint Manager. |1. Go to the [Microsoft Endpoint Manager admin center](https://go.microsoft.com/fwlink/?linkid=2109431) and sign in.<p>2. Select **Devices** > **Configuration profiles**, and then select the profile type you want to configure. <br/>If you haven't yet created a **Device restrictions** profile type, or if you want to create a new one, see [Configure device restriction settings in Microsoft Intune](/intune/device-restrictions-configure).<p>3. Select **Properties**, and then select **Configuration settings: Edit**.<p>4. Expand **Microsoft Defender Antivirus**. <p>5. Enable **Cloud-delivered protection**.<p>6. In the **Prompt users before sample submission** dropdown, select **Send all samples automatically**.<p>7. In the **Detect potentially unwanted applications** dropdown, select **Enable** or **Audit**.<p>8. Select **Review + save**, and then choose **Save**.<p>For more information about Intune device profiles, including how to create and configure their settings, see [What are Microsoft Intune device profiles?](/intune/device-profiles).|
|Control Panel in Windows     |Follow the guidance here: [Turn on Microsoft Defender Antivirus](/mem/intune/user-help/turn-on-defender-windows). <p>**NOTE**: You might see *Windows Defender Antivirus* instead of *Microsoft Defender Antivirus* in some versions of Windows.        |
|[Advanced Group Policy Management](/microsoft-desktop-optimization-pack/agpm/) <br/>or<br/>[Group Policy Management Console](use-group-policy-microsoft-defender-antivirus.md)  |1. Go to `Computer configuration > Administrative templates > Windows components > Microsoft Defender Antivirus`. <p>2. Look for a policy called **Turn off Microsoft Defender Antivirus**.<br/> <br/>3. Choose **Edit policy setting**, and make sure that policy is disabled. This enables Microsoft Defender Antivirus. <p>**NOTE**: You might see *Windows Defender Antivirus* instead of *Microsoft Defender Antivirus* in some versions of Windows. |

### Confirm that Microsoft Defender Antivirus is in passive mode

Microsoft Defender Antivirus can run alongside McAfee if you set Microsoft Defender Antivirus to passive mode. You can use either Command Prompt or PowerShell to perform this task, as described in the following table:

|Method  |What to do  |
|---------|---------|
|Command Prompt     |1. On a Windows device, open Command Prompt as an administrator. <p>2. Type `sc query windefend`, and then press Enter.<p>3. Review the results to confirm that Microsoft Defender Antivirus is running in passive mode.         |
|PowerShell     |1. On a Windows device, open Windows PowerShell as an administrator.<p>2. Run the [Get-MpComputerStatus](/powershell/module/defender/Get-MpComputerStatus) cmdlet. <p>3. In the list of results, look for either **AMRunningMode: Passive Mode** or **AMRunningMode: SxS Passive Mode**.|

> [!NOTE]
> You might see *Windows Defender Antivirus* instead of *Microsoft Defender Antivirus* in some versions of Windows.

## Get updates for Microsoft Defender Antivirus

Keeping Microsoft Defender Antivirus up to date is critical to assure your devices have the latest technology and features needed to protect against new malware and attack techniques, even if Microsoft Defender Antivirus is running in [passive mode](microsoft-defender-antivirus-compatibility.md).

There are two types of updates related to keeping Microsoft Defender Antivirus up to date:
- Security intelligence updates
- Product updates

To get your updates, follow the guidance in [Manage Microsoft Defender Antivirus updates and apply baselines](manage-updates-baselines-microsoft-defender-antivirus.md).

## Add Microsoft Defender for Endpoint to the exclusion list for McAfee

This step of the setup process involves adding Microsoft Defender for Endpoint to the exclusion list for McAfee and any other security products your organization is using. 

> [!TIP]
> To get help configuring exclusions, refer to McAfee documentation, such as the following article: [McAfee Endpoint Security 10.5.0 - Threat Prevention Module Product Guide (McAfee ePolicy Orchestrator) - Windows: Configuring exclusions](https://docs.mcafee.com/bundle/endpoint-security-10.5.0-threat-prevention-product-guide-epolicy-orchestrator-windows/page/GUID-71C5FB4B-A143-43E6-8BF0-8B2C16ABE6DA.html).

The specific exclusions to configure depend on which version of Windows your endpoints or devices are running, and are listed in the following table:

|OS |Exclusions |
|--|--|
|- Windows 10, [version 1803](/windows/release-health/status-windows-10-1803) or later (See [Windows 10 release information](/windows/release-health/release-information))<br/>- Windows 10, version 1703 or [1709](/windows/release-health/status-windows-10-1709) with [KB4493441](https://support.microsoft.com/help/4493441) installed <br/>- [Windows Server 2019](/windows/release-health/status-windows-10-1809-and-windows-server-2019)<br/>- [Windows Server, version 1803](/windows-server/get-started/whats-new-in-windows-server-1803) |`C:\Program Files\Windows Defender Advanced Threat Protection\MsSense.exe`<p>`C:\Program Files\Windows Defender Advanced Threat Protection\SenseCncProxy.exe`<p>`C:\Program Files\Windows Defender Advanced Threat Protection\SenseSampleUploader.exe`<p>`C:\Program Files\Windows Defender Advanced Threat Protection\SenseIR.exe`<br/>  |
|- [Windows 8.1](/windows/release-health/status-windows-8.1-and-windows-server-2012-r2) <br/>- [Windows 7](/windows/release-health/status-windows-7-and-windows-server-2008-r2-sp1)<br/>- [Windows Server 2016](/windows/release-health/status-windows-10-1607-and-windows-server-2016)<br/>- [Windows Server 2012 R2](/windows/release-health/status-windows-8.1-and-windows-server-2012-r2)<br/>- [Windows Server 2008 R2 SP1](/windows/release-health/status-windows-7-and-windows-server-2008-r2-sp1) |`C:\Program Files\Microsoft Monitoring Agent\Agent\Health Service State\Monitoring Host Temporary Files 6\45\MsSenseS.exe`<p>**NOTE**: Where Monitoring Host Temporary Files 6\45 can be different numbered subfolders.<p>`C:\Program Files\Microsoft Monitoring Agent\Agent\AgentControlPanel.exe`<p>`C:\Program Files\Microsoft Monitoring Agent\Agent\HealthService.exe`<p>`C:\Program Files\Microsoft Monitoring Agent\Agent\HSLockdown.exe`<p>`C:\Program Files\Microsoft Monitoring Agent\Agent\MOMPerfSnapshotHelper.exe`<p>`C:\Program Files\Microsoft Monitoring Agent\Agent\MonitoringHost.exe`<p>`C:\Program Files\Microsoft Monitoring Agent\Agent\TestCloudConnection.exe` |

## Add McAfee to the exclusion list for Microsoft Defender Antivirus

During this step of the setup process, you add McAfee and your other security solutions to the Microsoft Defender Antivirus exclusion list. 

When you add [exclusions to Microsoft Defender Antivirus scans](configure-exclusions-microsoft-defender-antivirus.md), you should add path and process exclusions. Keep the following points in mind:
- Path exclusions exclude specific files and whatever those files access.
- Process exclusions exclude whatever a process touches, but does not exclude the process itself.
- If you list each executable (.exe) as both a path exclusion and a process exclusion, the process and whatever it touches are excluded.
- List your process exclusions using their full path and not by their name only. (The name-only method is less secure.)

You can choose from several methods to add your exclusions to Microsoft Defender Antivirus, as listed in the following table:

|Method | What to do|
|--|--|
|[Intune](/mem/intune/fundamentals/tutorial-walkthrough-endpoint-manager) <p>**NOTE**: Intune is now Microsoft Endpoint Manager. |1. Go to the [Microsoft Endpoint Manager admin center](https://go.microsoft.com/fwlink/?linkid=2109431) and sign in.<p>2. Select **Devices** > **Configuration profiles**, and then select the profile that you want to configure.<p>3. Under **Manage**, select **Properties**. <p>4. Select **Configuration settings: Edit**.<p>5. Expand **Microsoft Defender Antivirus**, and then expand **Microsoft Defender Antivirus Exclusions**.<p>6. Specify the files and folders, extensions, and processes to exclude from Microsoft Defender Antivirus scans. For reference, see [Microsoft Defender Antivirus exclusions](/mem/intune/configuration/device-restrictions-windows-10#microsoft-defender-antivirus-exclusions).<p>7. Choose **Review + save**, and then choose **Save**.  |
|[Microsoft Endpoint Configuration Manager](/mem/configmgr/) |1. Using the [Configuration Manager console](/mem/configmgr/core/servers/manage/admin-console), go to **Assets and Compliance** > **Endpoint Protection** > **Antimalware Policies**, and then select the policy that you want to modify. <p>2. Specify exclusion settings for files and folders, extensions, and processes to exclude from Microsoft Defender Antivirus scans. |
|[Group Policy Object](/previous-versions/windows/desktop/Policy/group-policy-objects) | 1. On your Group Policy management computer, open the [Group Policy Management Console](https://technet.microsoft.com/library/cc731212.aspx), right-click the Group Policy Object you want to configure and click **Edit**.<p>2. In the **Group Policy Management Editor**, go to **Computer configuration** and click **Administrative templates**.<p>3. Expand the tree to **Windows components > Microsoft Defender Antivirus > Exclusions**.<br/>**NOTE**: You might see *Windows Defender Antivirus* instead of *Microsoft Defender Antivirus* in some versions of Windows.<p>4. Double-click the **Path Exclusions** setting and add the exclusions.<br/>- Set the option to **Enabled**.<br/>- Under the **Options** section, click **Show...**.<br/>- Specify each folder on its own line under the **Value name** column.<br/>- If you specify a file, make sure to enter a fully qualified path to the file, including the drive letter, folder path, filename, and extension. Enter **0** in the **Value** column.<p>5. Click **OK**.<p>6. Double-click the **Extension Exclusions** setting and add the exclusions.<br/>- Set the option to **Enabled**.<br/>- Under the **Options** section, click **Show...**.<br/>- Enter each file extension on its own line under the **Value name** column.  Enter **0** in the **Value** column.<p>7. Click **OK**. |
|Local group policy object |1. On the endpoint or device, open the Local Group Policy Editor. <p>2. Go to **Computer Configuration** > **Administrative Templates** > **Windows Components** > **Microsoft Defender Antivirus** > **Exclusions**. <br/>**NOTE**: You might see *Windows Defender Antivirus* instead of *Microsoft Defender Antivirus* in some versions of Windows.<p>3. Specify your path and process exclusions. |
|Registry key |1. Export the following registry key: `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Defender\exclusions`.<p>2. Import the registry key. Here are two examples:<br/>- Local path: `regedit.exe /s c:\temp\ MDAV_Exclusion.reg` <br/>- Network share: `regedit.exe /s \\FileServer\ShareName\MDAV_Exclusion.reg` |

## Add McAfee to the exclusion list for Microsoft Defender for Endpoint

> [!IMPORTANT]
> In general, you shouldn't have to add exclusions for Defender for Endpoint, especially if you have already defined exclusions for Microsoft Defender Antivirus. However, if you run into issues where Microsoft Defender Antivirus won't stay in passive mode, perform the following task. Otherwise, skip this section and proceed to [Set up your device groups, device collections, and organizational units](#set-up-your-device-groups-device-collections-and-organizational-units).

To add exclusions to Microsoft Defender for Endpoint, you create [indicators](indicator-file.md).

1. Go to the Microsoft Defender Security Center ([https://aka.ms/MDATPportal](https://aka.ms/MDATPportal)) and sign in.

2. In the navigation pane, choose **Settings** > **Rules** > **Indicators**.

3.  On the **File hashes** tab, choose **Add indicator**.

3. On the **Indicator** tab, specify the following settings:
   - File hash (Need help? See [Find a file hash using CMPivot](#find-a-file-hash-using-cmpivot) in this article.)
   - Under **Expires on (UTC)**, choose **Never**.

4. On the **Action** tab, specify the following settings:
   - **Response Action**: **Allow**
   - Title and description

5. On the **Scope** tab, under **Device groups**, select either **All devices in my scope** or **Select from list**.

6. On the **Summary** tab, review the settings, and then click **Save**.

### Find a file hash using CMPivot

CMPivot is an in-console utility for Configuration Manager. CMPivot provides access to the real-time state of devices in your environment. It immediately runs a query on all currently connected devices in the target collection and returns the results. To learn more, see [CMPivot overview](/mem/configmgr/core/servers/manage/cmpivot-overview).

To use CMPivot to get your file hash, follow these steps:

1. Review the [prerequisites](/mem/configmgr/core/servers/manage/cmpivot#prerequisites).

2. [Start CMPivot](/mem/configmgr/core/servers/manage/cmpivot#start-cmpivot). 

3. Connect to Configuration Manager (`SCCM_ServerName.DomainName.com`).

4. Select the **Query** tab.
 
5. In the **Device Collection** list, and choose **All Systems (default)**.

6. In the query box, type the following query:<br/>

```kusto
File(c:\\windows\\notepad.exe)
| project Hash
```
> [!NOTE]
> In the query above, replace *notepad.exe* with the your third-party security product process name. 

## Set up your device groups, device collections, and organizational units

| Collection type | What to do |
|--|--|
|[Device groups](machine-groups.md) (formerly called machine groups) enable your security operations team to configure security capabilities, such as automated investigation and remediation.<p> Device groups are also useful for assigning access to those devices so that your security operations team can take remediation actions if needed. <p>Device groups are created in the Microsoft Defender Security Center. |1. Go to the Microsoft Defender Security Center ([https://aka.ms/MDATPportal](https://aka.ms/MDATPportal)).<p>2. In the navigation pane on the left, choose **Settings** > **Permissions** > **Device groups**.  <p>3. Choose **+ Add device group**.<p>4. Specify a name and description for the device group.<p>5. In the **Automation level** list, select an option. (We recommend **Full - remediate threats automatically**.) To learn more about the various automation levels, see [How threats are remediated](automated-investigations.md#how-threats-are-remediated).<p>6. Specify conditions for a matching rule to determine which devices belong to the device group. For example, you can choose a domain, OS versions, or even use [device tags](machine-tags.md). <p>7. On the **User access** tab, specify roles that should have access to the devices that are included in the device group. <p>8. Choose **Done**. |
|[Device collections](/mem/configmgr/core/clients/manage/collections/introduction-to-collections) enable your security operations team to manage applications, deploy compliance settings, or install software updates on the devices in your organization. <p>Device collections are created by using [Configuration Manager](/mem/configmgr/). |Follow the steps in [Create a collection](/mem/configmgr/core/clients/manage/collections/create-collections#bkmk_create). |
|[Organizational units](/azure/active-directory-domain-services/create-ou) enable you to logically group objects such as user accounts, service accounts, or computer accounts. You can then assign administrators to specific organizational units, and apply group policy to enforce targeted configuration settings.<p> Organizational units are defined in [Azure Active Directory Domain Services](/azure/active-directory-domain-services). | Follow the steps in [Create an Organizational Unit in an Azure Active Directory Domain Services managed domain](/azure/active-directory-domain-services/create-ou). |

## Configure antimalware policies and real-time protection

Using Configuration Manager and your device collection(s), configure your antimalware policies.

- See [Create and deploy antimalware policies for Endpoint Protection in Configuration Manager](/mem/configmgr/protect/deploy-use/endpoint-antimalware-policies).

- While you create and configure your antimalware policies, make sure to review the [real-time protection settings](/mem/configmgr/protect/deploy-use/endpoint-antimalware-policies#real-time-protection-settings) and [enable block at first sight](configure-block-at-first-sight-microsoft-defender-antivirus.md).

> [!TIP]
> You can deploy the policies before your organization's devices on onboarded.

## Next step

**Congratulations**! You have completed the Setup phase of [migrating from McAfee to Microsoft Defender for Endpoint](mcafee-to-microsoft-defender-migration.md#the-migration-process)!

- [Proceed to Phase 3: Onboard to Microsoft Defender for Endpoint](mcafee-to-microsoft-defender-onboard.md)
