# Deploy-command-line
Realizando um deploy na linha de comando (Windows)

Antes de realizar um deploy temos que obter um servidor..
Neste tutorial uso a plataforma google cloud para obter um servidor na nuvem gratuitamente com o bônus que a plataforma concede.

Após criar uma Virtual Machine no Google Cloud Plataform, em Compute Engine/Instâncias de VM, e deixar o IP estático, vamos acessar a maquina via ssh

Terminal git bash:
~ comando: ssh-keygen //Escolha o nome do arquivo da chave e senha ssh 
~ comando: eval %(ssh-agent) //Ativar o agente ssh (Este comando se repete toda vez que iniciamos o computador)
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
