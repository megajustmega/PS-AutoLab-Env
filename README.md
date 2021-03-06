# PSAutoLab

[![PSGallery Version](https://img.shields.io/powershellgallery/v/PSAutolab.png?style=for-the-badge&logo=powershell&label=PowerShell%20Gallery)](https://www.powershellgallery.com/packages/PSAutolab/) [![PSGallery Downloads](https://img.shields.io/powershellgallery/dt/PSAutolab.png?style=for-the-badge&label=Downloads)](https://www.powershellgallery.com/packages/PSAutoLab/)

This project serves as a set of "wrapper" commands that utilize the [Lability](https://github.com/VirtualEngine/Lability) module which is a terrific tool for creating a lab environment of Windows based systems.
The downside is that it is a difficult module for less experienced PowerShell users.
The configurations and control commands for the Hyper-V virtual machines are written in PowerShell using Desired State Configuration (DSC) and deployed via Lability.
If you feel sufficiently skilled, you can skip using this project and use the Lability module on your own.
Note that the Lability module is not owned or managed by Pluralsight.
This project and all files are released under an MIT License - meaning you can copy and use as your own, modify, borrow, steal - whatever you want.

**While this project is under the Pluralsight banner, it is offered AS-IS as a free tool with no official support from Pluralsight.
Pluralsight makes no guarantees or warranties.
This project is intended to be used for educational purposes only.**

## Requirements

This tool currently supports running on either Windows Server 2016 (not fully tested) or a __Windows 10__ client that supports virtualization.
Windows 10 Pro or Enterprise should be sufficient.
This module will not work and and is unsupported on Windows 10 Home or any Student edition.
It is assumed you will be installing this on a Windows 10 desktop using Windows PowerShell 5.1.

The host computer must have the following:

* Windows PowerShell 5.1
* An internet connection
* Minimum 16GB of RAM (32GB is recommended)
* Minimum 100GB free disk space preferably on a fast SSD device.
* PowerShell Remoting enabled

You must have administrator access and be able to update TrustedHosts.
If you are in a corporate environment, these settings may be locked down or restricted.
If this applies to you, this module may not work for you.

**__This module and configurations have NOT been tested running from PowerShell Core or PowerShell 7 and is not supported at this time.__**

## Aliases and Language

While this module follows proper naming conventions, the commands you will typically use employ aliases that use non-standard verbs such as `Run-Lab`.
This is to avoid conflicts with commands in the Lability module and to maintain backwards compatibility.
You can use the aliases or the full function name.
All references in this document use the aliases.

## Previous Versions

If you installed previous versions of this module, and struggled, hopefully this version will be an improvement.
To avoid any other complications, it is STRONGLY recommended that you manually remove the old version which is most likely under `C:\Program Files\WindowsPowerShell\Modules\PSAutoLab`.
You can run a command like:

```powershell
PS C:\> Get-Module PSAutolab -ListAvailable | Select-Object Path
```

To identify the module location.
Use this information to delete the PSAutolab folder.

**The previous version was not installed using PowerShell's module cmdlets so it can't be updated or removed except manually.**

## Installation

This project has been published to the PowerShell Gallery.
It is recommended that you have at least version 2.2 of the `PowerShellGet` module which handles module installations.

Open an elevated PowerShell prompt and run:

```powershell
PS C:\> Install-Module PSAutoLab
```

If prompted, answer yes to update the nuget version and to install from an untrusted repository, unless you've already marked the PSGallery as trusted.
If you have an old copy from before Pluralsight took ownership you will get an error.
Manually remove the old module files and try again.

*Do not download or use any of the release packages on Github.
Only install this module from the PowerShell Gallery.*

See the [Changelog](./changelog.txt) for update details.

**DO NOT run this module on any mission-critical production system.**

### Note for VMware Users

This project is designed to work with Hyper-V.
If you are going to build a Host VM of Server 2016 or Windows 10, In the general settings for your VM, you must change the OS type to Hyper-V(Unsupported) or the Host Hyper-V will not work!

## Setup

The first time you use this module, you will need to configure the local machine or host.

Open an elevated PowerShell session and run:

```powershell
PS C:\> Setup-Host
```

This will install and configure the Lability module and install the Hyper-V feature if it is missing.
By default, all AutoLab files will be stored under C:\AutoLab, which the setup process will create.
If you prefer to use a different drive, you can specify it during setup.

```powershell
PS C:\> Setup-Host -DestinationPath D:\AutoLab
```

You will be prompted to reboot, which you should do especially if setup had to add Hyper-V.

## Creating a Lab

Lab information is stored under the AutoLab Configurations folder, which is C:\AutoLab\Configurations by default.
Open an elevated PowerShell prompt and change location to the desired configuration folder.
View the `Instructions.md` and/or readme files in the folder to learn more about the configuration.

The first time you setup a lab, Lability will download evaluation versions of required operating systems in ISO format.
This may take some time depending on your Internet bandwidth.
The downloads only happen when the required ISO is not found.
When you wipe and rebuild a lab it won't download files a second time.

Once the lab is created you can use the module commands for managing it.
Or you can manage individual virtual machines using the Hyper-V manager or cmdlets.

*It is assumed that you will only have one lab configuration created at a time.*

### Manual Setup

Most, if not all, configurations should follow the same manual process.
Run each command after the previous one has completed.

* `Setup-Lab`
* `Run-Lab`
* `Enable-Internet`

To verify that all virtual machines are properly configured you can run `Validate-Lab`.
This will invoke a set of tests and loop until everything passes.
Due to the nature of DSC and complexity of some configurations this could take up to 60 minutes.
You can use `Ctrl+C` to break out of the testing loop at any time.
You can manually run the test one time to see the current state of the configuration.

```powershell
PS C:\Autolab\Configurations\SingleServer\> Invoke-Pester VMValidate.test.ps1
```

This can be useful for troubleshooting.

### Unattended Setup

As an alternative, you can setup a lab environment with minimal prompting.

```powershell
PS C:\Autolab\Configurations\SingleServer\> Unattend-Lab
```

Assuming you don't need to install a newer version of nuget, you can leave the setup alone.
It will run all of the manual steps for you.
Beginning in version 4.3.0 you also have the option to run the unattend process in a PowerShell background job.

```powershell
PS C:\Autolab\Configurations\SingleServer\> unattend-lab -asjob
```

### Stopping a Lab

To stop the lab VMs, change to the configuration folder in an elevated Windows PowerShell session and run:

```powershell
PS C:\Autolab\Configurations\SingleServer\> Shutdown-Lab
```

You can also use the Hyper-V manager or cmdlets to shut down virtual machines.
If your lab contains a domain controller such as DOM1 or DC1, that should be the last virtual machine to shut down.

### Starting a Lab

The setup process will leave the virtual machines running.
If you have stopped the lab and need to start it, change to the configuration folder in an elevated Windows PowerShell session and run:

```powershell
PS C:\Autolab\Configurations\SingleServer\> Run-Lab
```

You can also use the Hyper-V manager or cmdlets to start virtual machines.
If your lab contains a domain controller such as DOM1 or DC1, that should be the first virtual machine to start up.

### Lab Checkpoints

You can snapshot the entire lab very easily.
Change to the configuration folder in an elevated Windows PowerShell session and run:

```powershell
PS C:\Autolab\Configurations\SingleServer\> Snapshot-Lab
```

To quickly rebuild the labs from the checkpoint, run:

```powershell
PS C:\Autolab\Configurations\SingleServer\> Refresh-Lab
```

### To Remove a Lab

To destroy the lab completely, change to the configuration folder in an elevated Windows PowerShell session and run:

```powershell
PS C:\Autolab\Configurations\SingleServer\> Wipe-Lab
```

This will remove the virtual machines and DSC configuration files.
If you intend to rebuild the lab or another configuration, you can keep the LabNat virtual switch.

## Windows Updates

When you build an lab, you are creating Windows virtual machines based on evaluation software.
You might still want to make sure the virtual machines are up to date with security patches and updates.
You can use `Update-Lab``to invoke Windows update on all lab members.
This can be a time consuming process, so you have an option to run the updates as a background job.
Just be sure not to close your PowerShell session before the jobs complete.

```powershell
PS C:\Autolab\Configurations\PowerShellLab> update-lab -AsJob

Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
18     WUUpdate        RemoteJob       Running       True            DOM1                  WUUpdate
21     WUUpdate        RemoteJob       Running       True            SRV1                  WUUpdate
24     WUUpdate        RemoteJob       Running       True            SRV2                  WUUpdate
27     WUUpdate        RemoteJob       Running       True            SRV3                  WUUpdate
30     WUUpdate        RemoteJob       Running       True            WIN10                 WUUpdate

PS C:\Autolab\Configurations\PowerShellLab> receive-job -id 27 -Keep
[11/22/2019 12:05:43] Found 5 updates to install on SRV3
[11/22/2019 12:25:13] Update process complete on SRV3
WARNING: SRV3 requires a reboot
```

Run the update process as a background job. Use the PowerShell job cmdlets to manage.

## Updating PSAutolab

As this module is updated over time, new configurations may be added, or bugs fixed in existing configurations.
There may also be new Lability updates.
Use PowerShell to check for new versions:

```powershell
PS C:\> Find-Module PSAutoLab
```

And update:

```powershell
PS C:\> Update-Module PSAutoLab
```

If you update, it is recommended that you update the computer running AutoLab.

```powershell
PS C:\> Refresh-Host
```

This will update Lability if required and copy all new configuration files to your AutoLab\Configurations folder.
It will NOT delete any files.

## Troubleshooting

The commands and configurations in this module are not foolproof.
During testing a lab configuration will run quickly and without error on one Windows 10 desktop but fail or take much longer on a different Windows 10 desktop.
Most setups should be complete in under an hour.
If validation is failing, manually run the validation test in the configuration folder.

```powershell
PS C:\Autolab\Configurations\SingleServer\> Invoke-Pester VMValidate.test.ps1
```

Take note of which virtual machines are generating errors.
Verify the virtual machine is running in Hyper-V.
On occasion for reasons still undetermined, sometimes a virtual machine will shutdown and not reboot.
This often happens with the client nodes of the lab configuration.
Verify that all virtual machines are running and manually start those that have stopped using the Hyper-V manager or cmdlets.

Sometimes even if the virtual machine is running, manually shutting it down and restarting it can resolve the problem.
Remember to wait at least 5 minutes before manually running the validation test again when restarting any virtual machine.

As a last resort, manually break out of any testing loop, wipe the lab and start all-over.

If you *still* are having problems, wipe the lab and try a different configuration.
This will help determine if the problem is with the configuration or a larger compatibility problem.

At this point, you can open an issue in this repository.
Open an elevated PowerShell prompt and run Get-PSAutoLabSetting which will provide useful information.
Copy and paste the results into a new issue along with any error messages you are seeing.

## Known Issues

### I get an error trying to update Lability

If you try to run `Refresh-Host` you might see an error about a certificate mismatch.
Between v0.18.0 and v0.19.0 the Lability module changed code signing certificates.
If you encounter this problem, run `Refresh-Host -SkipPublisherCheck`.

### Multiple DSC Resources

Due to what is probably a bug in the current implementation of Desired State Configuration in Windows, if you have multiple versions of the same resource, a previous version might be used instead of the required on.
You might especially see this with the xNetworking module and the xIPAddress resource.
If you have any version older than 5.7.0.0 you might encounter problems.
Run this command to see what you have installed:

```powershell
PS C:\> Get-DSCResource xIPAddress
```

If you have older versions of the module, uninstall them if you can.

```powershell
PS C:\> Uninstall-Module xNetworking -RequiredVersion 3.0.0.0
```

It is recommended that you restart your PowerShell session and try the lab setup again.

## Acknowledgments

This module is a continuation of the work done by Jason Helmick and Melissa (Missy) Januszko, whose efforts are greatly appreciated.
Beginning with v4.0.0, this module is unrelated to any projects Jason or Missy may be developing under similar names.

## Road Map

These are some of the items that are being considered for future updates:

* This project will need additional documentation
* While Lability currently is for Windows only, it would be nice to deploy a Linux VM
* Integrate the [PostSetup](.\Configurations\PowerShellLab\PostSetup\README.md) tools from the PowerShellLab configuration
* Offer an easy way to customize a lab configuration such as time zone, node names and operating systems.

A complete list of enhancements can be found in [Issues](https://github.com/pluralsight/PS-AutoLab-Env/issues).

Last Updated 2020-02-06 15:22:37Z UTC
