# Instrucciones de instalación Aptos

Saludos a todos, amigos. Hoy os voy a contar cómo poner un nodo Aptos para intentar entrar en AIT2.

 - Guía oficial: https://aptos.dev/nodes/validator-node/run-validator-node-using-docker
 - Más información sobre el evento: https://medium.com/aptoslabs/welcome-to-aptos-incentivized-testnet-2-af26e2fd69a7

Los requisitos recomendados para el servidor son los siguientes 

 - CPU: 4 núcleos
 - Memoria: 8 GB de RAM
 - 300 GB de espacio en disco

## ¡IMPORTANTE!

Si obtiene el error 859 al registrarse, comience por configurar el puerto Api 80 en lugar del 8080

Si el error sigue produciéndose, detenga el nodo con la eliminación de la base de datos (`docker compose down -v`). A continuación, realice los siguientes pasos:
 - Crear archivo layout.yaml 
 - Generar un archivo de génesis 
 - Ejecuta el nodo (`docker compose up -d`)

## Secuencia de instalación:

### 1. Actualizar el servidor e instalar los paquetes necesarios
```
sudo apt update && sudo apt upgrade -y
sudo apt install make clang pkg-config libssl-dev build-essential git unzip gcc chrony curl jq ncdu bsdmainutils htop net-tools lsof fail2ban wget -y
```
### 2. Introduzca las variables y especifíquelas en el perfil BASH
```
APTOS_MONIKER="El nombre de su nodo"
APTOS_IP="Dirección IP de su nodo"

echo 'export APTOS_IP='${APTOS_IP=} >> $HOME/.bash_profile
echo 'export APTOS_MONIKER='${APTOS_MONIKER=} >> $HOME/.bash_profile
source $HOME/.bash_profile
```
### 3. Instalación de Docker y Docker-Compose
```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
curl -SL https://github.com/docker/compose/releases/download/v2.5.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```
### 4. Descargue y mueva el archivo ejecutable al directorio de trabajo
```
wget -qO aptos-cli.zip https://github.com/aptos-labs/aptos-core/releases/download/aptos-cli-0.2.0/aptos-cli-0.2.0-Ubuntu-x86_64.zip
unzip -o aptos-cli.zip
chmod +x aptos
mv aptos /usr/local/bin 
```
### 5. Cree una carpeta de proyecto (`.aptos`) y descargue los archivos de configuración en ella
```
mkdir -p $HOME/.aptos
cd $HOME/.aptos
wget -O $HOME/.aptos/docker-compose.yaml https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/aptos-node/docker-compose.yaml
wget -O $HOME/.aptos/validator.yaml https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/aptos-node/validator.yaml
```
### 6. Generar claves en el directorio de trabajo y luego configurar el validador
```
aptos genesis generate-keys --output-dir $HOME/.aptos
aptos genesis set-validator-configuration  --keys-dir $HOME/.aptos --local-repository-dir $HOME/.aptos  --username $APTOS_MONIKER  --validator-host $APTOS_IP:6180
```
### 7. Crear `layout.yaml`
```
echo "---
root_key: \"F22409A93D1CD12D2FC92B5F8EB84CDCD24C348E32B3E7A720F3D2E288E63394\"
users:
  - \"$APTOS_MONIKER\"
chain_id: 40
min_stake: 0
max_stake: 100000
min_lockup_duration_secs: 0
max_lockup_duration_secs: 2592000
epoch_duration_secs: 86400
initial_lockup_timestamp: 1656615600
min_price_per_gas_unit: 1
allow_new_validators: true" >layout.yaml
```
### 8. Descargue e instale el Framework
```
wget -O $HOME/.aptos/framework.zip https://github.com/aptos-labs/aptos-core/releases/download/aptos-framework-v0.2.0/framework.zip
unzip -o framework.zip
```
### 9. Generar un archivo de génesis
```
aptos genesis generate-genesis --local-repository-dir $HOME/.aptos --output-dir $HOME/.aptos
```
### 10. Lanzamiento de un contenedor con Docker
```
docker compose up -d
```
### 11.1 Compruebe los registros de Docker
```
docker logs -f aptos-validator-1
```
### 11.2 Comprobar el estado de la sincronización
```
curl 127.0.0.1:9101/metrics 2> /dev/null | grep aptos_state_sync_version | grep type
```
### 11.3 Comando para la salida de los datos necesarios para completar [el formulario](https://community.aptoslabs.com/)
```
cat $HOME/.aptos/$APTOS_MONIKER.yaml
```
