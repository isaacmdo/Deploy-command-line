# Deploy-command-line
## Realizando um deploy na linha de comando (Windows)

#### Antes de realizar um deploy temos que obter um servidor..
#### Neste tutorial uso a plataforma google cloud para obter um servidor na nuvem gratuitamente com o bônus que a plataforma concede.
#### Após criar uma Virtual Machine no Google Cloud Plataform, em Compute Engine/Instâncias de VM, e deixar o IP estático, vamos acessar a maquina via ssh.

### Terminal git-bash:
* git-bash: ssh-keygen //Escolha o nome do arquivo da chave e senha ssh 
* git-bash: eval $(ssh-agent) //Ativar o agente ssh (Este comando se repete toda vez que iniciamos o computador)
* git-bash: ssh-add ~/.(Diretório onde se encontra a chave) //Adicionar  a chave ssh (Este comando se repete toda vez que iniciamos o computador)

Após a criação da chave ssh, adicionamos ela no Google Cloud Plataform, em Compute Engine/Metadados/Chaves SSH
E copiamos o IP do servidor em Compute Engine/Instâncias de VM.

Terminal git bash:
* git-bash: ssh seusuario@ipdoservidor ou ssh ipdoservidor

Pronto ! Agora estamos acessando a máquina que criamos dentro do servidor da Google !

Agora vamos preparar a máquina para utilizarmos...

Terminal do servidor:
* bash: sudo apt update
* bash: sudo apt upgrade


### Agora vamos configurar o Git para enviar arquivos ao servidor

#### No Git Bash, dentro da pasta do projeto:
* git-bash: git config --global user.email "seuemail@seudominio.com"
* git-bash: git config --global user.name "Seu nome"
* git-bash: git init
* git-bash: nano .gitignore (escrever quais arquivos devemos ignorar no upload para o servidor)
* git-bash: git add . (Para adicionar todos os arquivos)
* git-bash: git commit -am 'commit inicial' 

### Pronto ! Configuramos o git para o envio ao servidor, agora vamos enviar os arquivos para o servidor

* git-bash: eval $(ssh-agent) //Ativar o agente ssh (Este comando se repete toda vez que iniciamos o computador)
* git-bash: ssh-add ~/.(Diretório onde se encontra a chave) //Adicionar  a chave ssh (Este comando se repete toda vez que iniciamos o computador)
* git-bash: ssh ipdoservidor

#### Dentro do servidor:
* bash: mkdir repo-projeto (Repositório dentro do servidor opcional)
* bash: mkdir projeto (Onde estaram os arquivos do nosso projeto)
* bash: cd repo-projeto/ 
* bash: git init --bare
* bash: cd ..
* bash: cd projeto/
* bash: git init
* bash: git remote add projeto /home/seuser/repo-projeto/

#### Dentro da sua máquina:
* git-bash: git remote add projeto ipdoservidor:repo-projeto
* git-bash: git push projeto master

#### Dentro do servidor, na pasta dos arquivos do seu projeto:
* bash: git pull projeto master

### Pronto ! Agora temos todos os arquivos do nosso projeto dentro do servidor !


______________________________________________________________________________________________
A partir de agora toda vez que fizermos alguma alteração no nosso projeto, digitamos:

Na nossa máquina dentro da pasta master:
~ comando: git add .
~ comando: git commit -am 'A mensagem que queremos adicionar'
~ comando: git push projeto master
(Assim seguindo o processo do tutorial enviamos os arquivos para a pasta repo-projeto dentro do servidor)

Dentro do servidor:
~ comando: cd projeto/
~ comando: git pull agenda master (Aqui colocamos as alterações em produção)
________________________________________________________________________________________________

Com os arquivos do projeto no servidor, vamos configurar o servidor e a aplicação para uso.

Na pasta no projeto vamo instalar o nodejs e subir as dependências:
~ comando: sudo apt install curl -y
~ comando: curl -sL https://deb.nodesource.com/setup_12.x | sudo bash -
~ comando: sudo apt install nodejs -y
~ comando: npm i

Instalar Pm2 para gerenciar a aplicação:
~ comando: sudo npm i -g pm2
~ comando: pm2 start ~/caminhodoprojeto/server.js --name Projeto
(Obs.: Caso seu banco utilize um arquivo .env, crie agr)

Agora vamos configurar o Pm2 para startar nossa aplicação mesmo após correr uma reinicialização da maquina:
~ comando: pm2 startup
Copie e cole no terminal o comando de saída sujerido pelo Pm2
Agora para testar se a aplicação está funcionando digitamos:
~ comando: npm start
(Se houver algum erro, você terá que debuggar)
Para checkar se o pm2 está funcionado:
~ comando: curl http://localhost:3000


Vamos instalar o nginx no servidor:
~ comando: sudo apt install nginx
~ comando: sudo systemctl status (Para verificar se o nginx está rodando)

Em seguida vamos configurar o nginx com um arquivo de configuração padrão criado (Copie em um arquivo txt na SUA maquina para editar posteriormente):



  	server {
	listen 80;
	listen [::]:80;

	# Add index.php to the list if you are using PHP
	index index.html index.htm index.nginx-debian.html index.php;

	server_name seudominio/ip;

	location = /favicon.ico { access_log off; log_not_found off; }
  
	location / {
		proxy_pass http://localhost:3000;
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection 'upgrade';
		proxy_set_header Host $host;
		proxy_cache_bypass $http_upgrade;
	}

	# deny access to .htaccess files, if Apache's document root
	# concurs with nginx's one
	#
	location ~ /\.ht {
		deny all;
	}

	location ~ /\. {
		access_log off;
		log_not_found off;
		deny all;
	}

	gzip on;
	gzip_disable "msie6";

	gzip_comp_level 6;
	gzip_min_length 1100;
	gzip_buffers 4 32k;
	gzip_proxied any;
	gzip_types
		text/plain
		text/css
		text/js
		text/xml
		text/javascript
		application/javascript
		application/x-javascript
		application/json
		application/xml
		application/rss+xml
		image/svg+xml;

	access_log off;
	#access_log  /var/log/nginx/seudominio/ip-access.log;
	error_log   /var/log/nginx/seudominio/ip-error.log;

	#include /etc/nginx/common/protect.conf;
	}



Abra o script no VSCODE e substituia todos os camos que estamo escritos "seudominio/ip" para seu dominio ou ip do servidor e salve.

Agora com as alterações realizadas no arquivo, copie tudo e vamos para o seguinte procedimento:
~ comando: sudo nano /etc/nginx/sites-enabled/ipservidor

Depois de colar o arquivo de configuração, salve, e saia do nano. Digite:
~ comando: cd /etc/nginx/sites-enabled/
~ comando: sudo rm default
~ comando: sudo nginx -t
(Se a saída do comando acima mostrar algum erro, reveja o arquivo de configuração criado)
~ comando: sudo systemctl restart nginx

Agora acessando no navegador o dominio/ipdoservidor, terá acesso a aplicação, mas a aplicação está em http.
Obs.: Um comportamneto padrão dos navegadores e redirecionar a pagina para https, e isto quebraria o caminha da aplicação, para resolver este problema
vamos aplicar o seguinte passo para adicionar uma TLS(Cripytografia de segurança), mas para isto teremos que ter um dominio...Caso não tenha um dominio, não podemos 
aplicar a configuração e prosseguir com o tutorial.

~ comando: sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
~ comando: sudo apt-get install certbot
~ comando: sudo service nginx stop
~ comando: sudo certbot certonly --standalone -d seudominio.com.br
~ Coloque um email valido e aceite

Formate o arquivo substituindo tudo que está escrito seudominio para seu dominio:


		# O servidor não vai responder via IP
		server {
		  listen 80 default_server;
		  server_name _;
		  return 404;
		}

		# Redireciona para HTTPS
		server {
			listen 80;
			listen [::]:80;
		  server_name seudominio;
		  return 301 https://$host$request_uri;
		}

		# HTTPS
		server {
			listen 443 ssl http2;
			listen [::]:443 ssl http2;

			server_name seudominio;

			# O servidor só vai responder pra este domínio
		  if ($host != "seudominio") {
		    return 404;
		  }
	
		ssl_certificate /etc/letsencrypt/live/seudominio/fullchain.pem; # managed by Certbot
		ssl_certificate_key /etc/letsencrypt/live/seudominio/privkey.pem; # managed by Certbot
		ssl_trusted_certificate /etc/letsencrypt/live/seudominio/chain.pem;

		# Improve HTTPS performance with session resumption
		ssl_session_cache shared:SSL:10m;
		ssl_session_timeout 5m;

		# Enable server-side protection against BEAST attacks
		ssl_prefer_server_ciphers on;
		ssl_ciphers ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5;

		# Disable SSLv3
		ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

		# Diffie-Hellman parameter for DHE ciphersuites
		# $ sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 4096
		ssl_dhparam /etc/ssl/certs/dhparam.pem;

		# Enable HSTS (https://developer.mozilla.org/en-US/docs/Security/HTTP_Strict_Transport_Security)
		add_header Strict-Transport-Security "max-age=63072000; includeSubdomains";

		# Enable OCSP stapling (http://blog.mozilla.org/security/2013/07/29/ocsp-stapling-in-firefox)
		ssl_stapling on;
		ssl_stapling_verify on;
		resolver 8.8.8.8 8.8.4.4 valid=300s;
		resolver_timeout 5s;

		# Add index.php to the list if you are using PHP
		index index.html index.htm index.nginx-debian.html index.php;

		location = /favicon.ico { access_log off; log_not_found off; }

		location / {
			proxy_pass http://localhost:3000;
			proxy_http_version 1.1;
			proxy_set_header Upgrade $http_upgrade;
			proxy_set_header Connection 'upgrade';
			proxy_set_header Host $host;
			proxy_cache_bypass $http_upgrade;
		}

		# deny access to .htaccess files, if Apache's document root
		# concurs with nginx's one
		#
		location ~ /\.ht {
			deny all;
		}

		location ~ /\. {
			access_log off;
			log_not_found off;
			deny all;
		}

		gzip on;
		gzip_disable "msie6";

		gzip_comp_level 6;
		gzip_min_length 1100;
		gzip_buffers 4 32k;
		gzip_proxied any;
		gzip_types
			text/plain
			text/css
			text/js
			text/xml
			text/javascript
			application/javascript
			application/x-javascript
			application/json
			application/xml
			application/rss+xml
			image/svg+xml;

		access_log off;
		#access_log  /var/log/nginx/seudominio-access.log;
		error_log   /var/log/nginx/seudominio-error.log;

		#include /etc/nginx/common/protect.conf;
	}


~ comando: cd /etc/nginx/sites-enabled/
~ comando: sudo rm ipdoservidor
~ comando: sudo nano meudominio 
~ Cole o arquivo de configuração nginx e salve
~ comando: sudo service nginx start

Pronto ! Estamos com o certificado SSL.
Após vencer o ser certificado usamos:
~ comando: sudo certbot renew


Edição do app pelo Git (Windows):

* git-bash: ssh $eval(agent)
* git-bash: ssh-add ~/.ssh/suachave
* git-bash: git add .
* git-bash: git commit -am 'Nova versão'
* git-bash: git push projeto master
* git-bash: ssh <ip/dominio> "git -C /home/seuser/projeto/ pull projeto master"












