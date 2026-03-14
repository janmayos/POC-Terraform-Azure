# Terraform Azure Linux VM

Este proyecto despliega una VM Linux en Azure con Terraform:
- Resource Group
- VNet/Subnet
- NSG con acceso SSH (puerto 22)
- Public IP `Standard` + `Static`
- NIC
- Storage Account para boot diagnostics
- VM Linux Ubuntu 22.04
- Llaves SSH generadas en Azure (publica y privada)

## 1. Prerequisitos

- Terraform instalado (v1.x recomendado)
- Azure CLI instalado
- Sesion iniciada en Azure

Comandos:

```powershell
az login
az account show
terraform version
```

## 2. Variables importantes

En [variables.tf](variables.tf):
- `resource_group_location` (default: `westus2`)
- `resource_group_name_prefix` (default: `rg`)
- `username` (default: `azureadmin`)
- `vm_size` (ajustable si hay restricciones de capacidad)

## 3. Desplegar infraestructura

Desde la raiz del proyecto:

```powershell
terraform init
terraform validate
terraform plan -out main.tfplan
terraform apply main.tfplan
```

## 4. Ver outputs

```powershell
terraform output
```

Outputs disponibles en [outputs.tf](outputs.tf):
- `resource_group_name`
- `public_ip_address`
- `ssh_private_key` (sensible)

Para ver solo la IP publica:

```powershell
terraform output -raw public_ip_address
```

## 5. Guardar clave privada SSH y conectarse (Windows/PowerShell)

1) Exportar clave privada a archivo:

```powershell
terraform output -raw ssh_private_key | Out-File -FilePath .\id_rsa_tf -Encoding ascii
```

2) Ajustar permisos del archivo:

```powershell
icacls .\id_rsa_tf /inheritance:r
icacls .\id_rsa_tf /grant:r "$($env:USERNAME):(R)"
```

3) Obtener IP y conectarse:

```powershell
$ip = terraform output -raw public_ip_address
ssh -i .\id_rsa_tf azureadmin@$ip
```

Si cambiaste `username`, reemplaza `azureadmin` en el comando SSH.

## 6. Flujo rapido (copia y pega)

Ejecuta todo en orden desde la raiz del proyecto:

```powershell
terraform init
terraform validate
terraform plan -out main.tfplan
terraform apply main.tfplan
terraform output -raw ssh_private_key | Out-File -FilePath .\id_rsa_tf -Encoding ascii
icacls .\id_rsa_tf /inheritance:r
icacls .\id_rsa_tf /grant:r "$($env:USERNAME):(R)"
$ip = terraform output -raw public_ip_address
ssh -i .\id_rsa_tf azureadmin@$ip
```

## 7. Comandos utiles

Ver VM(s) del recurso:

```powershell
$rg = terraform output -raw resource_group_name
az vm list --resource-group $rg -o table
```

Ver estado de Terraform:

```powershell
terraform show
```

## 8. Destruir infraestructura

```powershell
terraform plan -destroy -out main.destroy.tfplan
terraform apply main.destroy.tfplan
```

## 9. Notas de troubleshooting

- Si una SKU no esta disponible en la region, cambia `vm_size` en [variables.tf](variables.tf) y vuelve a ejecutar `plan/apply`.
- Si no puedes conectar por SSH, valida:
  - Que la IP publica exista (`terraform output -raw public_ip_address`)
  - Que el NSG permita puerto 22
  - Que uses la clave privada correcta (`ssh_private_key`)
