# 2 - Join a Windows worker node to Kubernetes

## 1. Preparation

On the control plane node, run

```
./calicoctl ipam configure --strictaffinity=true --allow-version-mismatch
```

On your laptop

```
Set-VMProcessor -VMName w25f -ExposeVirtualizationExtensions $true
```

Turn on the Windows VM

On Widows VM

```
Install-WindowsFeature -Name containers
Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart
```

## 2. Install containerd

Ref. https://github.com/containerd/containerd/blob/main/docs/getting-started.md?source=post_page-----50b61e6891ad--------------------------------#installing-containerd-on-windows

```
# If containerd previously installed run:
Stop-Service containerd

# Download and extract desired containerd Windows binaries
$Version="1.7.13"	# update to your preferred version
$Arch = "amd64"	# arm64 also available
curl.exe -LO https://github.com/containerd/containerd/releases/download/v$Version/containerd-$Version-windows-$Arch.tar.gz
tar.exe xvf .\containerd-$Version-windows-amd64.tar.gz

# Copy
Copy-Item -Path .\bin -Destination $Env:ProgramFiles\containerd -Recurse -Force

# add the binaries (containerd.exe, ctr.exe) in $env:Path
$Path = [Environment]::GetEnvironmentVariable("PATH", "Machine") + [IO.Path]::PathSeparator + "$Env:ProgramFiles\containerd"
[Environment]::SetEnvironmentVariable( "Path", $Path, "Machine")
# reload path, so you don't have to open a new PS terminal later if needed
$Env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")

# configure
containerd.exe config default | Out-File $Env:ProgramFiles\containerd\config.toml -Encoding ascii
# Review the configuration. Depending on setup you may want to adjust:
# - the sandbox_image (Kubernetes pause image)
# - cni bin_dir and conf_dir locations
Get-Content $Env:ProgramFiles\containerd\config.toml

# Register and start service
containerd.exe --register-service
Start-Service containerd
```

## 3. Install Calico

On Windows node

```
mkdir c:\k
mkdir 'C:\Program Files\containerd\cni\bin'
```

```
scp jakapong@172.23.163.83:/home/jakapong/.kube/config c:\k
```

![alt text](image-11.png)

```
Invoke-WebRequest https://github.com/projectcalico/calico/releases/download/v3.29.1/install-calico-windows.ps1 -OutFile c:\install-calico-windows.ps1

.\install-calico-windows.ps1 -KubeVersion 1.32.1 -CalicoBackend vxlan

Get-Service -Name CalicoNode
Get-Service -Name CalicoFelix
```

## 4. Install Kubernetes tools

```
C:\CalicoWindows\kubernetes\install-kube-services.ps1

Get-Service -Name kubelet
Get-Service -Name kube-proxy
```

## 5. Join a Windows worker not


