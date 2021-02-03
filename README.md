# Deploy-command-line
Realizando um deploy na linha de comando (Windows)

Antes de realizar um deploy temos que obter um servidor..
Neste tutorial uso a plataforma google cloud para obter um servidor na nuvem gratuitamente com o bônus que a plataforma concede.

Após criar uma Virtual Machine no Google Cloud Plataform, em Compute Engine/Instâncias de VM, e deixar o IP estático, vamos acessar a maquina via ssh

Terminal git bash:
~ comando: ssh-keygen //Escolha o nome do arquivo da chave e senha ssh 
~ comando: eval $(ssh-agent) //Ativar o agente ssh (Este comando se repete toda vez que iniciamos o computador)
~ comando: ssh-add ~/.(Diretório onde se encontra a chave) //Adicionar  a chave ssh (Este comando se repete toda vez que iniciamos o computador)

Após a criação da chave ssh, adicionamos ela no Google Cloud Plataform, em Compute Engine/Metadados/Chaves SSH
E copiamos o IP do servidor em Compute Engine/Instâncias de VM.

Terminal git bash:
~ comando ssh seusuario@ipdoservidor ou ssh ipdoservidor

Pronto ! Agora estamos acessando a máquina que criamos dentro do servidor da Google !

Agora vamos preparar a máquina para utilizarmos...

Terminal do servidor:
~ comando: sudo apt update
~ comando: sudo apt upgrade


Agora vamos configurar o Git para enviar arquivos ao servidor

No Git Bash, dentro da pasta do projeto:
~ comando: git config --global user.email "seuemail@seudominio.com"
~ comando: git config --global user.name "Seu nome"
~ comando: git init
~ comando: nano .gitignore (escrever quais arquivos devemos ignorar no upload para o servidor)
~ comando: git add . (Para adicionar todos os arquivos)
~ comando: git commit -am 'commit inicial' 

Pronto ! Configuramos o git para o envio ao servidor, agora vamos enviar os arquivos para o servidor

~ comando: eval $(ssh-agent) //Ativar o agente ssh (Este comando se repete toda vez que iniciamos o computador)
~ comando: ssh-add ~/.(Diretório onde se encontra a chave) //Adicionar  a chave ssh (Este comando se repete toda vez que iniciamos o computador)
~ comando: ssh ipdoservidor

Dentro do servidor:
~ comando: mkdir repo-projeto (Repositório dentro do servidor opcional)
~ comando: mkdir projeto (Onde estaram os arquivos do nosso projeto)
~ comando: cd repo-projeto/ 
~ comando: git init --bare
~ comando: cd ..
~ comando: cd projeto/
~ comando: git init
~ comando: git remote add projeto /home/seuser/repo-projeto/

Dentro da sua máquina:
~ comando: git remote add projeto ipdoservidor:repo-projeto
~ comando: git push projeto master

Dentro do servidor, na pasta dos arquivos do seu projeto:
~ comando: git pull projeto master

Pronto ! Agora temos todos os arquivos do nosso projeto dentro do servidor !


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

Em seguida vamos configurar o nginx com um arquivo de configuração padrão criado:
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

Abra o script no VSCODE e substituia todos os camos que estamo escritos "seudominio/ip" para seu dominio ou ip do servidor.





