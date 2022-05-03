# reproducible_build_basic_exp_library

This is a proof of concept for a reproducible Rust build of a _library_ on Windows.

Reproducible Rust builds of _binaries_ on Windows do not currently work, as documented in [this GitHub issue](https://github.com/rust-lang/rust/issues/88982).

## Using

There are a few different ways you can use this proof of concept. I chose to use identical VMs in Azure (created using the Azure CLI).

To do it this way, you will need:
* An Azure Subscription
* The Azure CLI (I use it in PowerShell)
* Windows Remote Desk Protocol

First, login to your Azure subscription with:

```
az login
```

Then create your resource group (if you are not using an existing one):

```
az group create --name <RESOURCE GROUP NAME> --location centralus
```

### Your first VM

Then create your first VM (I chose to use a Windows 11 Cloud Workstation, but you could likely use WindowsServer or whatever Windows image you prefer)

```
az vm create --resource-group <RESOURCE GROUP NAME> --name <VM NAME> --image MicrosoftWindowsDesktop:windows11preview:win11-21h2-pro:22000.194.2109250206 --admin-username <ADMIN USERNAME> --admin-password <ADMIN PASSWORD> --size Standard_DC4s_v3 
```

Then, when the VM has completed spinning up, navigate to it using Azure Web Portal. Click "Connect" and select "RDP", which will download and RDP profile for you. Open the profile with Windows Remote Desk Protocol and log in to your VM using the admin username and password you defined for the VM.

Once you are on the VM, open a PowerShell window as Admin, then run these commands (this will take a bit to complete)

```
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
choco install visualcpp-build-tools --version 15.0.26228.20170424 --params "'/IncludeOptional'" -y
choco install rust-ms --version 1.60.0 -y
choco install git --version 2.36.0 -y
```

Close the PowerShell window and open a new one (it doesn't need to be Admin at this time), then run these commands to build a very base rust rlib.

```
git clone https://github.com/nellshamrell/reproducible_build_basic_exp_library.git
cd .\reproducible_build_basic_exp_library\
rustc .\src\lib.rs --crate-type rlib
```

Then get the SHA256 hash value of the rlib you just produced using:

```
Get-FileHash -Path .\liblib.rlib
```

Save this value somewhere.

### Your Second VM

To create your second VM, follow the exact same procedure you just did for your first VM (including the PowerShell commands, etc.)

When you get the SHA256 value of the rlib you produce on this machine with:

```
Get-FileHash -Path .\liblib.rlib
```

It should be the exact same value as the one created on your first VM.

### Clean Up

Remember to clean up your cloud infrastructure when you are done!

