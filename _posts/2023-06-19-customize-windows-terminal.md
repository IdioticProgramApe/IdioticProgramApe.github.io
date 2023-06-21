---
title: Windows Terminal Customization
author: ipa
date: 2023-06-19
categories: [Miscellaneous]
tags: [tools]
img_path: /tools/
---

## Prerequisites

Here are some tools need installing before officially start our customization:

- Windows Terminal: can be downloaded via Microsoft Store
- PowerShell: can be also downloaded via Microsoft Store
- Linux: can use `wsl --install -d Ubuntu` to install it via terminal otherwise as always Microsoft Store, there are other distributions as well.
- Cascadia Cove Nerd fonts: get them from https://www.nerdfonts.com/
- Terminal Icons: 
  - to install: `Install-Module -Name Terminal-Icons -Repository PSGallery`
  - to import: `Import-Module -Name Terminal-Icons`, in the ps profile files

- Z:
  - to install: Install-Module -Name Z -AllowClobber
  - to import: `Import-Module -Name Z`, in the ps profile files

- Oh my Posh!: follow the doc at https://ohmyposh.dev/docs/installation

## Settings

1. Open up the Setting tab from Terminal, the new powershell should be available, if not try relaunch the terminal:

   ![terminal customization settings](terminal_customization_settings.png){: width="392" height="219" }
   _Windows Terminal Settings_

2. Hide the default windows power shell from the profiles that terminal can access from by setting ***hidden*** to `true`:

3. (Optional) Run windows terminal powershellcore as Administrator

4. Open the `$Profile` file for the current powershell, copy the content from [here](https://gist.github.com/shanselman/25f5550ad186189e0e68916c6d7f44c3) to that file.

5. Browse in [themes](https://ohmyposh.dev/docs/themes), pick one and replace the url (local absolute path) used for oh-my-posh initialization in the profile file with the just picked one.

6. reload profile with `. $PROFILE`

## Reference

For more details, here is the original video [link](https://www.youtube.com/watch?v=VT2L1SXFq9U).
