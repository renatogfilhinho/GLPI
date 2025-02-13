#  Tutorial: Configuração do GLPI com Docker

Este tutorial mostra como configurar o **GLPI** usando dois contêineres Docker:

- **`glpi-app`**: Contêiner com Apache2 para hospedar o GLPI.
- **`glpi-bd`**: Contêiner com MariaDB para armazenar os dados do GLPI.


##  1. Instalar o Docker e o Docker Compose
Se o Docker ainda não estiver instalado, execute:

```bash
sudo apt update && sudo apt install docker.io docker-compose -y
```

Ative e inicie o serviço:

```bash
sudo systemctl enable --now docker
```

---

##  2. Criar a Estrutura de Diretórios
Crie uma pasta para armazenar os arquivos do GLPI e do banco de dados:

```bash
mkdir -p ~/glpi/{app,db}
```

---

##  3. Criar o Arquivo `docker-compose.yml`
Crie um arquivo `docker-compose.yml` dentro da pasta `~/glpi` e adicione o seguinte conteúdo:

```yaml
version: '3.8'

services:
  glpi-bd:
    image: mariadb:latest
    container_name: glpi-bd
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: glpi
      MYSQL_USER: glpi_user
      MYSQL_PASSWORD: glpi_pass
    volumes:
      - ./db:/var/lib/mysql
    ports:
      - "3306:3306"

  glpi-app:
    image: debian:latest
    container_name: glpi-app
    restart: unless-stopped
    depends_on:
      - glpi-bd
    volumes:
      - ./app:/var/www/html/
    ports:
      - "80:80"
    command: >
      sh -c "apt update && apt install -y apache2 php php-mysqli php-curl php-gd php-intl php-ldap php-imap php-xml php-zip php-bz2 php-j son php-mbstring wget tar unzip && \
      wget https://github.com/glpi-project/glpi/releases/download/10.0.18/glpi-10.0.18.tgz -O /tmp/glpi.tgz && \
      tar -xzf /tmp/glpi.tgz -C /var/www/html/ && \
      rm -rf /tmp/glpi.tgz && \
      chown -R www-data:www-data /var/www/html/ && \

```

---

##  4. Executar o Docker Compose
Dentro da pasta `~/glpi`, execute:

```bash
cd ~/glpi
docker-compose up -d
```

Isso baixará as imagens necessárias e iniciará os contêineres.

---

##  5. Acessar o GLPI
Abra o navegador e acesse:

```
http://localhost/glpi
```

Ou, se estiver em outro dispositivo na mesma rede, use o IP da máquina onde está rodando o Docker.

---

##  6. Finalizar a Instalação do GLPI
Siga os passos na interface web do GLPI para configurar a conexão com o banco de dados (`glpi-bd`).

- **Host do banco de dados**: `glpi-bd`
- **Nome do banco**: `glpi`
- **Usuário**: `glpi_user`
- **Senha**: `glpi_pass`

Após a configuração, remova a pasta `install/` no contêiner:

```bash
docker exec -it glpi-app rm -rf /var/www/html/install
```

---

##  Conclusão
Agora, você tem o GLPI rodando em dois contêineres com persistência de dados! 🚀

### 🔍 Comandos úteis:

Verificar logs:
```bash
docker logs -f glpi-app
docker logs -f glpi-bd
```

Parar os contêineres:
```bash
docker-compose down
```

Reiniciar os contêineres:
```bash
docker-compose up -d
```




