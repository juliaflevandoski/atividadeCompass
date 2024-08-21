Aqui está a documentação detalhada para seu repositório no GitHub, cobrindo a configuração da infraestrutura na AWS e o ambiente Linux. Você pode usar este conteúdo diretamente no seu repositório GitHub para descrever o processo completo.

---

# Documentação do Projeto AWS e Linux

---

## Configuração da Infraestrutura na AWS

### Criação da VPC

1. **Nome da VPC**: Definimos o nome da VPC para facilitar a identificação.
2. **Bloco CIDR**: Escolhemos um bloco de endereçamento IPv4 para a VPC, por exemplo, `10.0.0.0/16`.
3. **Criação**: Criamos a VPC usando o console da AWS, CLI, ou Terraform, conforme sua preferência.

### Criação das Subnets

1. **Nome da Subnet**: Definimos um nome para cada subnet criada.
2. **Associação à VPC**: Associamos cada subnet à VPC previamente criada.
3. **Zona de Disponibilidade (AZ)**: Selecionamos a zona de disponibilidade (por exemplo, `us-east-1a`) para a subnet.
4. **Bloco CIDR da Subnet**: Definimos o bloco de endereçamento IPv4 para cada subnet, como `10.0.1.0/24`.

### Criação do Internet Gateway

1. **Criação do Internet Gateway**: Criamos um Internet Gateway na AWS.
2. **Associação à VPC**: Associamos o Internet Gateway à VPC, permitindo comunicação da VPC com a internet.

### Configuração da Route Table

1. **Criação da Route Table**: Criamos uma nova tabela de rotas ou utilizamos a tabela padrão.
2. **Adição de Rota**: Adicionamos uma rota que direciona todo o tráfego (0.0.0.0/0) para o Internet Gateway.
3. **Associação da Subnet à Route Table**: Associamos a subnet pública à tabela de rotas configurada para permitir o tráfego de saída para a internet.

### Criação da Instância EC2

1. **Escolha da AMI**: Selecionamos o Amazon Linux 2 como o sistema operacional para a instância.
2. **Tipo de Instância**: Escolhemos `t3.small` como o tipo de instância, o que oferece um equilíbrio entre performance e custo.
3. **Par de Chaves (Key Pair)**: Criamos ou selecionamos um par de chaves para acesso SSH seguro à instância.
4. **Configuração de Armazenamento**: Configuramos o armazenamento da instância, utilizando 16 GB de SSD.
5. **Criação da Instância**: Finalizamos a configuração e lançamos a instância EC2.

### Configuração das Regras de Segurança

1. **Configuração do Security Group**: Criamos ou utilizamos um Security Group existente.
2. **Liberação das Portas**: Adicionamos regras para liberar as portas necessárias:
   - Porta 22/TCP: Para acesso SSH.
   - Porta 80/TCP: Para tráfego HTTP.
   - Porta 443/TCP: Para tráfego HTTPS.
   - Porta 111/TCP e UDP, 2049/TCP e UDP: Para o serviço NFS.

---

## Configuração do Ambiente Linux

### Atualização e Instalação de Pacotes

```bash
sudo yum update

sudo yum upgrade -y

sudo yum install nfs-utils httpd mod_ssl -y
```

### Configuração do Fuso Horário e Certificados HTTPS

```bash
sudo timedatectl set-timezone America/Sao_Paulo

openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/httpd.key -x509 -days 3650 -out /etc/pki/tls/certs/httpd.crt
```

### Configuração do Apache

```bash
mkdir /var/www/https

# Criar uma página HTML para requisições HTTP
sudo bash -c 'echo "<!DOCTYPE html><html lang=\"en\"><head><meta charset=\"UTF-8\"><meta name=\"viewport\" content=\"width=device-width, initial-scale=1.0\"><title>Simple HTML</title><style>body {font-family: Arial, sans-serif; margin: 0; padding: 0; background-color: #f4f4f4; color: #333;} header {background-color: #333; color: white; padding: 1em;} main {padding: 2em;} h1 {font-size: 2em; margin-bottom: 0.5em;} p {font-size: 1.2em; line-height: 1.6; text-align: justify;} footer {background-color: #333; color: white; text-align: center; padding: 1em; position: fixed; bottom: 0; width: 100%;}</style></head><body><header><h1>Some Title HTTP!</h1></header><main><h2>Welcome!</h2><p>This is a basic HTML template with some CSS styling. You can customize the content and styles as needed.</p><p>The header and footer sections are styled to be distinct, with the footer remaining fixed at the bottom of the page.</p></main><footer><span>&copy; 2024 My Website</span></footer></body></html>" > /var/www/html/index.html'

# Criar uma página HTML para requisições HTTPS
sudo bash -c 'echo "<!DOCTYPE html><html lang=\"en\"><head><meta charset=\"UTF-8\"><meta name=\"viewport\" content=\"width=device-width, initial-scale=1.0\"><title>Simple HTML</title><style>body {font-family: Arial, sans-serif; margin: 0; padding: 0; background-color: #f4f4f4; color: #333;} header {background-color: #333; color: white; padding: 1em;} main {padding: 2em;} h1 {font-size: 2em; margin-bottom: 0.5em;} p {font-size: 1.2em; line-height: 1.6; text-align: justify;} footer {background-color: #333; color: white; text-align: center; padding: 1em; position: fixed; bottom: 0; width: 100%;}</style></head><body><header><h1>Some Title HTTPS!</h1></header><main><h2>Welcome!</h2><p>This is a basic HTML template with some CSS styling. You can customize the content and styles as needed.</p><p>The header and footer sections are styled to be distinct, with the footer remaining fixed at the bottom of the page.</p></main><footer><span>&copy; 2024 My Website</span></footer></body></html>" > /var/www/html/index.html'
```

### Configuração do NFS

```bash
sudo mkdir /srv/nfs
sudo mkdir /srv/nfs/julia

sudo chown -R nobody:nobody /srv/nfs/

sudo chmod 666 -R /srv/nfs_share

sudo bash -c 'echo "/srv/nfs_share 0.0.0.0/0(rw,all_squash)" >> /etc/exports'

sudo systemctl start nfs-server
sudo systemctl enable nfs-server

sudo exportfs -rav
```

### Criação e Configuração do Script de Verificação

```bash
sudo bash -c 'echo "#!/bin/bash

PROCESS_NAME=\"httpd\"
POPULAR_NAME=\"APACHE\"

CURRENT_DATE=\$(date +%d/%m/%y-%H:%M:%S)

PROCESS_ACTIVE=\$(systemctl is-active \"\$PROCESS_NAME\")

if [ \"\$PROCESS_ACTIVE\" = \"active\" ]; then
  echo \"[\${CURRENT_DATE}] - Process \"\${POPULAR_NAME}\" is ACTIVE\" >> /srv/nfs/julia/ONLINE.log
else
  echo \"[\${CURRENT_DATE}] - Process \"\${POPULAR_NAME}\" is INACTIVE\" >> /srv/nfs/julia/OFFLINE.log
fi" > /usr/bin/httpd_check.sh'

sudo chmod +x /usr/bin/httpd_check.sh
```

### Configuração do Serviço e Timer para o Script

```bash
sudo bash -c 'echo "[Unit]
Description=Verify if HTTPD is active

[Service]
Type=oneshot
ExecStart=/usr/bin/httpd_check.sh

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/httpd_check.service'

sudo bash -c 'echo "[Unit]
Description=\"Timer for HTTPD CHECK service\"

[Timer]
OnBootSec=5min
OnUnitActiveSec=5min
AccuracySec=1s

[Install]
WantedBy=timers.target" > /etc/systemd/system/httpd_check.timer'

sudo systemctl daemon-reload
sudo systemctl enable httpd_check.timer
sudo systemctl status httpd_check.timer
```

---
