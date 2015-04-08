---
layout: post
title: Global settings files in WiX
date: 2013-09-24 20:55:00 -700
tags:
- wix
---

Global settings files have several requirements:

1. On install, create the file if it does not exist.

2. Configure certain settings during installation/upgrade, and leave all other
   settings alone.

3. Never delete the settings file during installation, upgrade, or
   uninstallation.

With these requirements, we have the following fragment:

    <Component Id="settings.config" NeverOverwrite="yes" Permanent="yes">
        <File Source="path/to/default/settings.config" KeyPath="yes" />
    </Component>

    <Component Id="settings.config.Configure" KeyPath="yes" Guid="...">
        <util:XmlConfig File="[#settings.config]" .... Value="[PROPERTY1]" />
        <util:XmlConfig File="[#settings.config]" .... Value="[PROPERTY2]" />
    </Component>

Here's how this meets all the requirements:

1. The `settings.config` component creates the file if the component isn't
   installed when we start the installation.

2. During installation and upgrade, the second component will modify the
   existing `settings.config` that's on disk. The setting
   `Component/@NeverOverwrite` prevents us from overwriting the existing file
   on upgrade.

3. During uninstallation, the file is left in place with
   `Component/@Permanent`. This attribute also prevents the file from being
   removed even if `MajorUpgrade/@Schedule='afterInstallValidate'`.

4. The `XmlConfig` actions are in a separate component so they will run on
   upgrades even though `settings.config` isn't being updated.
