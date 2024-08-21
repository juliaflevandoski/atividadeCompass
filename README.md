# Atividade Compass

## AWS

Descrever os passos nas AWS
Aqui está a documentação adaptada com base nos comandos que você usou:

---

# Documentação para Configuração de Ambiente AWS e Linux

## Requisitos AWS

### 1. Gerar uma chave pública para acesso ao ambiente

1. **Abra o console da AWS** e vá para o serviço **EC2**.
2. No painel de navegação, selecione **Key Pairs** e clique em **Create Key Pair**.
3. Escolha um **nome** para a chave e **formato** (.pem para Linux, .ppk para Windows).
4. **Baixe** a chave e **guarde** em um local seguro, pois você precisará dela para acessar sua instância.

### 2. Criar uma instância EC2

1. No console EC2, clique em **Launch Instance**.
2. Escolha a **Amazon Linux 2 AMI**.
3. Selecione o tipo de instância **t3.small**.
4. Clique em **Next: Configure Instance Details** e ajuste conforme necessário.
5. Clique em **Next: Add Storage** e defina o **tamanho do SSD** como 16 GB.
6. Clique em **Next: Add Tags** e adicione tags conforme necessário.
7. Clique em **Next: Configure Security Group** e adicione as regras de segurança:
   - Tipo: SSH, Porta: 22, Protocolo: TCP
   - Tipo: NFS, Porta: 111, Protocolo: TCP e UDP
   - Tipo: NFS, Porta: 2049, Protocolo: TCP e UDP
   - Tipo: HTTP, Porta: 80, Protocolo: TCP
   - Tipo: HTTPS, Porta: 443, Protocolo: TCP
8. Selecione a chave criada anteriormente em **Key Pair** e clique em **Launch Instance**.

### 3. Gerar um Elastic IP e anexar à instância EC2

1. No console EC2, vá para **Elastic IPs** e clique em **Allocate Elastic IP address**.
2. Selecione o **Elastic IP** e clique em **Actions** > **Associate Elastic IP address**.
3. Escolha a instância EC2 criada e clique em **Associate**.

## Requisitos no Linux

### 1. Atualizar o sistema e instalar os pacotes necessários

```bash
# Buscar atualizações disponíveis
sudo yum update

# Instalar atualizações
sudo yum upgrade -y

# Instalar NFS, HTTPD (Apache) e mod_ssl para suporte HTTPS
sudo yum install nfs-utils httpd mod_ssl -y
```

### 2. Configuração do fuso horário e criação de certificados HTTPS

```bash
# Modificar o fuso horário para São Paulo
sudo timedatectl set-timezone America/Sao_Paulo

# Criar certificados HTTPS
openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/httpd.key -x509 -days 3650 -out /etc/pki/tls/certs/httpd.crt
```

### 3. Configuração do Apache

```bash
# Criar diretório para arquivos HTTPS
mkdir /var/www/https

# Criar uma página HTML para requisições HTTP
sudo bash -c 'echo "<!DOCTYPE html><html lang=\"en\"><head><meta charset=\"UTF-8\"><meta name=\"viewport\" content=\"width=device-width, initial-scale=1.0\"><title>Simple HTML</title><style>body {font-family: Arial, sans-serif; margin: 0; padding: 0; background-color: #f4f4f4; color: #333;} header {background-color: #333; color: white; padding: 1em;} main {padding: 2em;} h1 {font-size: 2em; margin-bottom: 0.5em;} p {font-size: 1.2em; line-height: 1.6; text-align: justify;} footer {background-color: #333; color: white; text-align: center; padding: 1em; position: fixed; bottom: 0; width: 100%;}</style></head><body><header><h1>Some Title HTTP!</h1></header><main><h2>Welcome!</h2><p>This is a basic HTML template with some CSS styling. You can customize the content and styles as needed.</p><p>The header and footer sections are styled to be distinct, with the footer remaining fixed at the bottom of the page.</p></main><footer><span>&copy; 2024 My Website</span></footer></body></html>" > /var/www/html/index.html'

# Criar uma página HTML para requisições HTTPS
sudo bash -c 'echo "<!DOCTYPE html><html lang=\"en\"><head><meta charset=\"UTF-8\"><meta name=\"viewport\" content=\"width=device-width, initial-scale=1.0\"><title>Simple HTML</title><style>body {font-family: Arial, sans-serif; margin: 0; padding: 0; background-color: #f4f4f4; color: #333;} header {background-color: #333; color: white; padding: 1em;} main {padding: 2em;} h1 {font-size: 2em; margin-bottom: 0.5em;} p {font-size: 1.2em; line-height: 1.6; text-align: justify;} footer {background-color: #333; color: white; text-align: center; padding: 1em; position: fixed; bottom: 0; width: 100%;}</style></head><body><header><h1>Some Title HTTPS!</h1></header><main><h2>Welcome!</h2><p>This is a basic HTML template with some CSS styling. You can customize the content and styles as needed.</p><p>The header and footer sections are styled to be distinct, with the footer remaining fixed at the bottom of the page.</p></main><footer><span>&copy; 2024 My Website</span></footer></body></html>" > /var/www/html/index.html'
```

### 4. Configuração do NFS

```bash
# Criar diretório para NFS e subdiretório com nome
sudo mkdir /srv/nfs_share
sudo mkdir /srv/nfs_share/cristian

# Alterar o dono do diretório para 'nobody'
sudo chown -R nobody:nobody /srv/nfs_share/

# Modificar permissões para leitura e escrita para todos
sudo chmod 666 -R /srv/nfs_share

# Adicionar configuração ao arquivo de exportações do NFS
sudo bash -c 'echo "/srv/nfs_share 0.0.0.0/0(rw,all_squash)" >> /etc/exports'

# Iniciar e habilitar o serviço NFS
sudo systemctl start nfs-server
sudo systemctl enable nfs-server

# Aplicar as configurações do NFS
sudo exportfs -rav
```

### 5. Configuração e execução do Apache

```bash
# Iniciar e habilitar o serviço Apache
sudo systemctl start httpd.service
sudo systemctl enable httpd.service
```

### 6. Criar e configurar o script de verificação do Apache

```bash
# Criar o script para verificar o status do HTTPD
sudo bash -c 'echo "#!/bin/bash

PROCESS_NAME=\"httpd\"
POPULAR_NAME=\"APACHE\"

CURRENT_DATE=\$(date +%d/%m/%y-%H:%M:%S)

PROCESS_ACTIVE=\$(systemctl is-active \"\$PROCESS_NAME\")

if [ \"\$PROCESS_ACTIVE\" = \"active\" ]; then
  echo \"[\${CURRENT_DATE}] - Process \"\${POPULAR_NAME}\" is ACTIVE at the moment!\" >> /srv/nfs_share/cristian/ONLINE.log
else
  echo \"[\${CURRENT_DATE}] - Process \"\${POPULAR_NAME}\" is INACTIVE at the moment!\" >> /srv/nfs_share/cristian/OFFLINE.log
fi" > /usr/bin/httpd_check.sh'

# Tornar o script executável
sudo chmod +x /usr/bin/httpd_check.sh
```

### 7. Configuração do serviço e timer para o script de verificação

```bash
# Criar o serviço systemd para o script
sudo bash -c 'echo "[Unit]
Description=Verify if HTTPD is active

[Service]
Type=oneshot
ExecStart=/usr/bin/httpd_check.sh

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/httpd_check.service'

# Criar o timer systemd para o serviço
sudo bash -c 'echo "[Unit]
Description=\"Timer for HTTPD CHECK service\"

[Timer]
OnBootSec=5min
OnUnitActiveSec=5min
AccuracySec=1s

[Install]
WantedBy=timers.target" > /etc/systemd/system/httpd_check.timer'

# Recarregar configurações do systemd e habilitar o timer
sudo systemctl daemon-reload
sudo systemctl enable httpd_check.timer
sudo systemctl status httpd_check.timer
```

## Versionamento e Documentação

1. **Inicialize um repositório Git** no diretório de trabalho:
   ```bash
   git init
   ```

2. Adicione os arquivos ao repositório:
   ```bash
   git add /usr/bin/httpd_check.sh
   git add /etc/systemd/system/httpd_check.service
   git add

## Linux

Descrever os passos no Linux


**asdfsadf**

*asdfasdf*

# Titulo 1

## Titulo 2

### Titulo 3



```bash
echo oi
```
