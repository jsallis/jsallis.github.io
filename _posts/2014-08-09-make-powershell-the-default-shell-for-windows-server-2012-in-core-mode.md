---
layout: post
title:  "Make PowerShell the default shell for Windows Server 2012 in Core Mode"
date:   2014-08-09 20:07:15
categories: windows powershell
---

Logging into a server running Windows Server 2012 in Core Mode presents you with Cmd.exe as your default shell. PowerShell is a more logical choice for administration and you'll likely start it as your first command. So you might as well skip having to do that and make PowerShell your default login shell.

Just run the following command via PowerShell to modify the registry, log out and back in again, and you'll be greeted by PowerShell instead of Cmd.exe.

{% highlight powershell %}
Set-ItemProperty -Confirm -Path "Microsoft.PowerShell.Core\Registry::HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" -Name Shell -Value "PowerShell.exe -noExit"
{% endhighlight %}
