# WindowsServer Packages

Quick reference guide for installing and configuring essential development tools for Windows Server AWS environments.

## Packages

### AWS CLI v2

```bash
<powershell>
# Baixa o instalador oficial do AWS CLI v2 para uma pasta temporária
Invoke-WebRequest -Uri "https://awscli.amazonaws.com/AWSCLIV2.msi" -OutFile "$env:TEMP\AWSCLIV2.msi"

# Instala o AWS CLI de forma silenciosa e aguarda a conclusão
Start-Process msiexec.exe -ArgumentList "/i `"$env:TEMP\AWSCLIV2.msi`" /qn" -Wait
</powershell>
```

### Verify installation

```bash
aws --version
```

---
    
### IIS

```bash
<powershell>
Install-WindowsFeature -name Web-Server -IncludeManagementTools
</powershell>
```

