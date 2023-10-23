<p align="center">
  <a href="" rel="noopener">
 <img width=400px height=100px src="https://upload.wikimedia.org/wikipedia/commons/thumb/4/45/Logo_CompassoUOL_Positivo.png/1200px-Logo_CompassoUOL_Positivo.png" alt="Project logo"></a>
</p>

<h1 align="center">Conexão entre duas VMs, uma com MariaDB e outra com Wordpress, utilizando NFS para salvar os estáticos. </h1>
<p align="center"><i> Criando uma conexão entre duas VMs, onde a VM01 está utilizando o MariaDB e a VM02 está com o Wordpress e servidor apache. Além disso, os estáticos do wordpress serão salvos em uma pasta compartilhada (NFS).</i></p>

## Desafios anteriores:
- [Desafio 1](https://github.com/CleitonOS/compass-linux-desafio1)
- [Desafio 2](https://github.com/CleitonOS/compass-linux-desafio2)

## 📝 Tabela de conteúdos
- [Instalando tudo que é necessário](#step1)
- [](#step2)
- [](#step3)


## 🖥️ Instalando tudo que é necessário (Passo 1)<a name = "step1"></a>

### Atualizando o sistema
- Execute o seguinte comando:
    ```
    sudo dnf update
    ```
    
- Se houver atualizações do kernel, considere reiniciar a máquina com:

    ```
    sudo reboot
    ```

### Instalando MariaDB na VM01

1. Adicinando o repositório do MariaDB ao Oracle Linux

    ```
    curl -LsS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash -s -- --mariadb-server-version="mariadb-10.6"
    ```

2. Instalando os pacotes do maria DB

    ```
    sudo dnf module reset mariadb -y
    sudo dnf -y install MariaDB-server MariaDB-client MariaDB-backup
    ```

- Inicie e habilide o serviço MariaDB:

    ```
    sudo systemctl enable --now mariadb
    ```

- Confira o status do serviço (Veja se o serviço está ativo):

    ```
    systemctl status mariadb
    ```

3. Faça a instalação segura do banco de dados MariaDB

- Execute o script de proteção do banco de dados para remover o banco de dados de teste e definir a senha root]

    ```
    mariadb-secure-installation
    ```

- Depois faça login como root para verificar se a senha definida está funcionando:

    ```
    mysql -u root -p
    ```

    <img src="./Screenshots/login-mariadb.png" width="60%">
</br>

- Insira os seguintes comandos no MariaDB:

    ```
    CREATE DATABASE wordpress;
    CREATE USER 'wordpressuser'@'IP_VM02' IDENTIFIED BY 'sua_senha';
    GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'IP_VM02';
    FLUSH PRIVILEGES;
    EXIT;
    ```
    - Observação: Não esqueça de substituir 'IP_VM02' pelo IP da sua segunda máquina que está com o Wordpress instalado, além disso dê um nome ao seu usuário (USER) e crie sua senha (sua_senha).


### Instalando Wordpress, Apache e PHP na VM02

1. Começando pelo **Apache**, instalando Apache HTTP Server
    
    ```
    sudo yum install httpd
    ```

- Inicie o serviço e configure-o para iniciar na inicialização

    ```
    sudo systemctl start httpd
    sudo systemctl enable httpd
    ```
<br/>

2. Instalando **PHP** e os módulos necessários

    ```
    dnf install php-fpm php-cli php-json php-gd php-mbstring php-pdo php-xml php-mysqlnd php-pecl-zip curl -y
    ```
    
- Inicie o PHP-FPM

    ```
    sudo systemctl start php-fpm
    sudo systemctl enable php-fpm
    ```
<br/>


3. Baixando e descompactando o **Wordpress**

- Mova para a pasta de instalação

    ```
    cd /var/www/html
    ```

- Baixe o Wordpress

    ```
    curl https://wordpress.org/latest.tar.gz --output wordpress.tar.gz
    ```

- Descompacte o arquivo

    ```
    tar xf wordpress.tar.gz
    ```

- Vá para a pasta do Wordpress

    ```
    cd /var/www/html/wordpress
    ```

- Crie um arquivo de configuração para o Wordpress

    ```
    sudo cp wp-config-sample.php wp-config.php
    sudo nano wp-config.php
    ```

- Lembra das informações que você preencheu ao criar um Database no MariaDB? Então, iremos utilizá-las agora:
    ```
    define('DB_NAME', 'wordpress');
    define('DB_USER', 'wordpressuser');
    define('DB_PASSWORD', 'sua_senha');
    define('DB_HOST', 'IP_VM01');
    ```
    - Não esqueça de colocar as informações que você definiu, as informações acima servem apenas como exemplo.