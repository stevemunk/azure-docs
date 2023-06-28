---
title: Use multimedia redirection on Azure Virtual Desktop - Azure
description: How to use multimedia redirection on Azure Virtual Desktop.
author: dknappettmsft
ms.topic: how-to
ms.date: 05/10/2023
ms.author: daknappe
manager: femila
---
# Use multimedia redirection on Azure Virtual Desktop

This article will show you how to use multimedia redirection (MMR) for Azure Virtual Desktop with Microsoft Edge or Google Chrome browsers. For more information about how multimedia redirection works, see [Understanding multimedia redirection for Azure Virtual Desktop](multimedia-redirection-intro.md).

## Prerequisites

Before you can use multimedia redirection on Azure Virtual Desktop, you'll need the following things:

- An Azure Virtual Desktop deployment.
- Microsoft Edge or Google Chrome installed on your session hosts.
- Windows Desktop client, version 1.2.3916 or later on Windows 11, Windows 10, or Windows 10 IoT Enterprise devices. This includes the multimedia redirection plugin (`C:\Program Files\Remote Desktop\MsMmrDVCPlugin.dll`), which is required on the client device. Your device must meet the [hardware requirements for Teams on a Windows PC](/microsoftteams/hardware-requirements-for-the-teams-app#hardware-requirements-for-teams-on-a-windows-pc/).
- Microsoft Visual C++ Redistributable 2015-2022, version 14.32.31332.0 or later installed on your session hosts and Windows client devices. You can download the latest version from [Microsoft Visual C++ Redistributable latest supported downloads](/cpp/windows/latest-supported-vc-redist).

## Install the multimedia redirection extension

For multimedia redirection to work, there are two parts to install on your session hosts: the host component and the browser extension for Edge or Chrome. You install the host component and browser extension from an MSI file, and you can also get and install the browser extension from Microsoft Edge Add-ons or the Chrome Web Store, depending on which browser you're using.

### Install the host component

To install the host component on your session hosts, you can install the MSI manually on each session host or use your enterprise deployment tool with `msiexec`. To install the MSI manually, you'll need to:

1. Sign in to a session host as a local administrator.

1. Download the [MMR host MSI installer](https://aka.ms/avdmmr/msi).

1. Open the file that you downloaded to run the setup wizard.

1. Follow the prompts. Once it's completed, select **Finish**.

### Install the browser extension

Next, you'll need to install the browser extension on session hosts where you already have Edge or Chrome available. When you install the host component, it also installs the browser extension. In order to use multimedia redirection, they'll need to enable the extension:

1. Sign in to Azure Virtual Desktop and open Edge or Chrome.

1. When opening the browser, after a short while users will see a prompt that says **New Extension added**, then they should select **Turn on extension**. Users should also pin the extension so that they can see from the icon if multimedia redirection is connected.

   :::image type="content" source="./media/mmr-extension-enable.png" alt-text="A screenshot of the prompt to enable the extension.":::

   >[!IMPORTANT]
   >If the user selects **Remove extension**, it will be removed from the browser and they will need to add it from Microsoft Edge Add-ons or the Chrome Web Store. To install it again, see [Installing the browser extension manually](#install-the-browser-extension-manually).

You can also automate installing the browser extension from Microsoft Edge Add-ons or the Chrome Web Store for all users by [using Group Policy](#install-the-browser-extension-using-group-policy).

Using Group Policy has the following benefits:

- You can install the extension silently and without user interaction.
- You can restrict which websites use multimedia redirection.
- You can pin the extension icon in Google Chrome by default.

#### Install the browser extension manually

If you need to install the browser extension separately, you can download it from Microsoft Edge Add-ons or the Chrome Web Store.

To install the multimedia redirection extension manually, follow these steps:

1. Sign in to Azure Virtual Desktop.

1. In your browser, open one of the following links, depending on which browser you're using:

   - For **Microsoft Edge**: [Microsoft Multimedia Redirection Extension](https://microsoftedge.microsoft.com/addons/detail/wvd-multimedia-redirectio/joeclbldhdmoijbaagobkhlpfjglcihd)

   - For **Google Chrome**: [Microsoft Multimedia Redirection Extension](https://chrome.google.com/webstore/detail/wvd-multimedia-redirectio/lfmemoeeciijgkjkgbgikoonlkabmlno)

1. Install the extension by selecting **Get** (for Microsoft Edge) or **Add to Chrome** (for Google Chrome), then at the additional prompt, select **Add extension**. Once the installation is finished, you'll see a confirmation message saying that you've successfully added the extension.

#### Install the browser extension using Group Policy

You can install the multimedia redirection extension using Group Policy, either centrally from your domain for session hosts that are joined to an Active Directory (AD) domain, or using the Local Group Policy Editor for each session host. This process will change depending on which browser you're using.

# [Edge](#tab/edge)

1. Download and install the Microsoft Edge administrative template by following the directions in [Configure Microsoft Edge policy settings on Windows devices](/deployedge/configure-microsoft-edge#1-download-and-install-the-microsoft-edge-administrative-template)

1. Next, decide whether you want to configure Group Policy centrally from your domain or locally for each session host:
   
   - To configure it from an AD Domain, open the **Group Policy Management Console** (GPMC) and create or edit a policy that targets your session hosts.
   
   - To configure it locally, open the **Local Group Policy Editor** on the session host.

1. Go to **Computer Configuration** > **Administrative Templates** > **Microsoft Edge** > **Extensions**.

1. Open the policy setting **Configure extension management settings** and set it to **Enabled**.

1. In the field for **Configure extension management settings**, enter the following:

   ```json
   { "joeclbldhdmoijbaagobkhlpfjglcihd": { "installation_mode": "force_installed", "update_url": "https://edge.microsoft.com/extensionwebstorebase/v1/crx" } }
   ```

   You can specify additional parameters to allow or block specific domains. For example, to only allow *youtube.com*, enter the following:

   ```json
   { "joeclbldhdmoijbaagobkhlpfjglcihd": { "installation_mode": "force_installed", "runtime_allowed_hosts": [ "*://*.youtube.com" ], "runtime_blocked_hosts": [ "*://*" ], "update_url": "https://edge.microsoft.com/extensionwebstorebase/v1/crx" } }
   ```

1. Apply the changes by running the following command in Command Prompt or PowerShell on each session host:

   ```cmd
   gpupdate /force
   ```

# [Google Chrome](#tab/google-chrome)

1. Download and install the Google Chrome administrative template by following the instructions in [Set Chrome Browser policies on managed PCs](https://support.google.com/chrome/a/answer/187202#zippy=%2Cwindows)

1. Next, decide whether you want to configure Group Policy centrally from your domain or locally for each session host:
   
   - To configure it from an AD Domain, open the **Group Policy Management Console** (GPMC) and create or edit a policy that targets your session hosts.
   
   - To configure it locally, open the **Local Group Policy Editor** on the session host.

1. Go to **Computer Configuration** > **Administrative Templates** > **Google** > **Google Chrome** > **Extensions**.

1. Open the policy setting **Extension management settings** and set it to **Enabled**.

1. In the field for **Extension management settings**, enter the following:

   ```json
   { "lfmemoeeciijgkjkgbgikoonlkabmlno": { "installation_mode": "force_installed", "update_url": "https://clients2.google.com/service/update2/crx" } }
   ```

   You can specify additional parameters to allow or block specific domains. For example, to only allow *youtube.com* and pin the extension to the toolbar, enter the following:

   ```json
   { "lfmemoeeciijgkjkgbgikoonlkabmlno": { "installation_mode": "force_installed", "runtime_allowed_hosts": [ "*://*.youtube.com" ], "runtime_blocked_hosts": [ "*://*" ], "toolbar_pin": "force_pinned", "update_url": "https://clients2.google.com/service/update2/crx" } }
   ```

1. Apply the changes by running the following command in Command Prompt or PowerShell on each session host:

   ```cmd
   gpupdate /force
   ```

---

## Check the extension status

Once you've installed the extension, you can check its status by visiting a website with media content, such as one from the list at [Websites that work with multimedia redirection](multimedia-redirection-intro.md#websites-that-work-with-multimedia-redirection), and hovering your mouse cursor over [the multimedia redirection extension icon](multimedia-redirection-intro.md#the-multimedia-redirection-status-icon) in the extension bar on the top-right corner of your browser. A message will appear and tell you about the current status, as shown in the following screenshot.

:::image type="content" source="./media/mmr-extension-status-popup.png" alt-text="A screenshot of the MMR extension in the Microsoft Edge extension bar.":::

Another way you can check the extension status is by selecting the extension icon, then you'll see a list of **Features supported on this website** with a green check mark if the website supports that feature.

## Teams live events

To use multimedia redirection with Teams live events:

1. Sign in to Azure Virtual Desktop.

1. Open the link to the Teams live event in either the Edge or Chrome browser.

1. Make sure you can see a green play icon as part of the [multimedia redirection status icon](multimedia-redirection-intro.md#the-multimedia-redirection-status-icon). If the green play icon is there, MMR is enabled for Teams live events.

1. Select **Watch on the web instead**. The Teams live event should automatically start playing in your browser. Make sure you only select **Watch on the web instead**, as shown in the following screenshot. If you use the native Teams app, MMR won't work.

   :::image type="content" source="./media/teams-live-events.png" alt-text="A screenshot of the 'Watch the live event in Microsoft Teams' page. The status icon and 'watch on the web instead' options are highlighted in red.":::

## Enable video playback for all sites

Multimedia redirection is currently limited to the sites listed in [Websites that work with multimedia redirection](multimedia-redirection-intro.md#websites-that-work-with-multimedia-redirection) by default. However, you can enable video playback for all sites to allow you to test the feature with other websites. To enable video playback for all sites:

1. Select the extension icon in your browser.

1. Select **Show Advanced Settings**.

1. Toggle **Enable video playback for all sites(beta)** to **on**.

## Redirected video outlines

Redirected video outlines will allow you to highlight the currently redirected video elements. When this is enabled, you will see a bright highlighted border around the video element that is being redirected. To enable redirected video outlines:

1. Select the extension icon in your browser.

1. Select **Show Advanced Settings**.

1. Toggle **Redirected video outlines** to **on**. You will need to refresh the webpage for the change to take effect.

### Video status overlay

When you enable video status overlay, you'll see a short message at the top of the video player that indicates the redirection status of the current video. The message will disappear after five seconds.  To enable video status overlay:

1. Select the extension icon in your browser.

2. Select **Show Advanced Settings**.

3. Toggle **Video Status Overlay** to **on**. You'll need to refresh the webpage for the change to take effect.

## Next steps

For more information about multimedia redirection and how it works, see [What is multimedia redirection for Azure Virtual Desktop?](multimedia-redirection-intro.md).

To troubleshoot issues or view known issues, see [our troubleshooting article](troubleshoot-multimedia-redirection.md).

If you're interested in learning more about using Teams for Azure Virtual Desktop, check out [Teams for Azure Virtual Desktop](teams-on-avd.md).
