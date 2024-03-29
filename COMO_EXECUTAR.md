# web service node

Este repositório consiste em um web service escrito em NodeJS, que serve de base para os outros web services do Mateus App.

## Requisitos para executar

Para executar este web service você precisa instalar:
- NodeJS, na versão LTS mais recente
- Yarn, como gerenciador de dependências

Com tudo instalado, para instalar as dependências do projeto, execute `yarn` na raiz do repositório.

Para executar o web service, execute:

``` yarn start ```

Proposta para a solução dos problemas:

1 - Dockernização da API:

1.1 Problema
	De acordo com o problema relatado na 1º situação entende-se que o encapsulamento da api em um container docker, com instruções de pre-configuração agiliza o processo de ambientação da máquina de desenvolvimento para o desenvolvedor com a utilização de poucos comandos para o mesmo realizar suas tarefas. 
	Dessa forma, a criação do script em docker-compose pode resolver convenientemente esse problema, contudo apenas a utilização do script em si nao resolve o problema de automatização de toda a estrutura de maquinas dos desenvolvedores de uma equipe que utilizará o mesmo ambiente. 

	- Instalar as ferramentas git, docker e docker compose:
    sudo apt-get -y install docker docker-compose git ansible

	- clonar o repositório da api:
    git clone https://paulocorreaslz@bitbucket.org/timecrm/paulo.git

	- acessar o diretorio paulo/ e executar o comando a seguir:
    make up // para subir o container
    make dow // para parar o container
    make logs // para ver os logs da execução do container

	obs: em caso de problemas, pode-se fazer necessário a instalação das bibliotecas a seguir:
    	sudo apt-get install build-essential

    Obs2: durante o processo de conteinerização o yarn baixa automaticamente as dependencias do projeto com o comando "yarn install" e em seguida, inicia o servidor com o coman
	Ao final, em caso de sucesso será exibida a mensagem da api "Servidor ouvindo na porta 3000", para testar basta acessar o endereço http://localhost:3000 em cada máquina

2 - Automatização da configuração do ambiente das maquinas de desenvolvimento

Para automatizar a configuração de instalação do ambiente de trabalho das máquinas de desenvolvimento se faz necessário o uso da ferramenta ansible. Com ela, pode-se determinar grupos de instalação para facilitar a configuração diversas máquinas ao mesmo tempo.

Para simular um ambiente de trabalho, será utilizado o Vagrant para criar máquinas virtuais simulando as máquinas de desenvolvimento com Ubuntu Limpo mencionado no item 1 deste teste. Com o vagrant teremos uma máquina virtual para cada desenvolvedor que poderá ser configurada de acordo com as necessidades para resolver o problemração. 
A utilização do Vagrant necessita de uma máquina principal para controlar a instalação dos nós filhos.

Configurando a máquina controladora (controlador pai):
2.1 - Em uma maquina Ubuntu limpa, utilizar o comando:
	 	sudo apt-get install ansible python vagrant 

2.2 - Subindo uma maquina (nó filho que será controlado) vagrant do tipo developer com ambiente configurado para simular o ambiente do programador.
		Obs: antes de rodar o comando do ansible, é necessário subir o vagrant para criar a maquina virtual com o comando : vagrant up 
		o comando subira duas maquinas que estao no arquivo hosts, que poderao ser acessadas por ssh para testar se foram devidamente configuradas. 

		em seguida, rodar o comando abaixo para instalar os pacotes.
		ansible-playbook provisioning.yml -i hosts 
		
		obs: o arquivo hosts possui as informações para subir uma maquina virtual, contem o ip, usuario de acesso para configuração das maquinas e a chave de acesso gerada pelo vagrant, conforme a seguir. ex: 172.20.1.4 ansible_user=vagrant ansible_ssh_private_key_file=".vagrant/machines/developer/virtualbox/private_key"
		Ao subir a máquina, verificar o caminho da ansible_ssh_private_key_file, pois pode mudar de maquina para maquina.
		obs2: a máquina developer que irá subir está configurada a partir do arquivo VagrantFile, onde ontem as informações de nome, sistema operacional, memoria, ip e podendo conter outras configurações.

		apos as maquinas levantadas, para acessa-las basta usar o usuario vagrant e senha vagrant com o comando:
		ssh vagrant@172.20.1.4
		ou
		ssh vagrant@172.20.1.5 

		uma vez dentro da maquina pode se testar os aplicativos instalados pelo provisionamento do ansible: docker, docker-compose, yarn, etc.

3 Instalando o AWS-CLI e Docker-machine para automatizar deploy na AWS

3.1 Gerando as chaves Access key e Secret Access
  Para gerar a Access Key ID, acessar o painel do administrador da AWS (https://console.aws.amazon.com/iam/home?region=us-east-1#/security_credentials) e criar uma nova chave de acesso (Create new access key)
  Salvar as informações para serem utilizadas no item

3.2 Configurando o AWS-CLI para um perfil existente com acesso ao console da AWS.

3.2.1 No console da maquina local digitar:
	aws configure

	O Comando acima irá solicitar o Access Key, Acces Secret e a Região dos datacenters da AWS que deseja hospedar sua aplicação pelo mundo que foi gerado no item 3.1
	
3.3 Configurando o AWS-CLI por arquivo de configuração
	Para configurar utilizando os arquivo de configuração, após o AWS-CLI instalado na máquina, deve-se acessar/criar se nao houver o arquivo ~/.aws/credentials e adicionar as informações referentes as credenciais
	[default]
	aws_access_key_id=AKIAIOSFODNN7EXAMPLE
	aws_secret_access_key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

	Deve-se ainda acessar/criar se nao houver o arquivo ~/.aws/config e adicionar as informações da região do datacenter AWS a ser utilizado
	[default]
	region=us-west-2
	output=json

	Como referencia foi utiizado a documentação oficial de configuração da AWS em:
	https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html

3.4 Instalando o Docker-Machine para executar comandos Docker na instancia EC2 criada AWS.
	O docker-machine é responsável por acessar e rodar comandos diretamente dentro do docker da instancia AWS criada através do AWS-CLI.

	Para instalar o docker-machine no linux utilize os comandos a seguir, seguindo a documentação (https://docs.docker.com/machine/install-machine/):
	$ base=https://github.com/docker/machine/releases/download/v0.16.0 &&
	  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
	  sudo mv /tmp/docker-machine /usr/local/bin/docker-machine &&
	  chmod +x /usr/local/bin/docker-machine

4 Executando script para resolver o problema de deploy automatico de container docker na AWS
	
	Executar o comando:
	sh deploy.prod.sh

	Após a execução do comando acima, caso todos os passos tenham sidos realizados corretamente, será criada uma instância a partir dos arquivos dockerfile e docker-compose do tipo EC2 no perfil AWS configurado pelo comando do item 3.2.3 com a porta 3000 liberada.

5 O problema 2 consiste em melhorar o processo de deploy de aplicações desenvolvidas pelo time de 	desenvolvimento.
	Para aumentar a produtividade e realizar a automatização do processo de integração continua e deploy automatico, propoe-se a execução de um script, criado no item 4, que é responsavel por fazer o deploy do container docker que foi atualizado atraves do git em uma maquina virtual na AWS. 
	O Script é uma versão simplificada de como pode ser automatizado o processo de criação de uma instância de uma maquina virtual EC2 na AWS a partir de um container docker, contudo sua execução nesse cenário é manual. Porém, embora seja manual, o script pode fazer parte de qualquer uma das ferramentas de Integração continua mencionadas (Jenkins, TravisCI, CircleCI etc) como parte da esteira de processo de integração continua a partir da análise e monitoramento do repositório do sistema de controle de versao (git) dos e através de jobs que podem definir quais branches ou tags devem iniciar o processo de deploy.

	No anexo, Comparacao_de_ferramentas.doc, está descrita uma análise das vantagens e desvantagens de cada uma delas.

5.1 Acessando o painel EC2
		
	Uma vez que a aplicação foi implantada através do processo de deploy do script na nuvem da AWS, para descobrir o endereço IP da mesma, se faz necessário neste único momento acessar o painel do console EC2 da conta do usuário da AWS na qual a instância docker foi configurada pelo AWS-CLI no item 3 e seus subitens. Dessa forma, podemos ver o IP que foi atribuido a instancia docker da aplicação durante o processo de deploy. Para isto, acesse o link abaixo e utilize as credenciais do usuário para abrir o painel/console de gerenciamento de máquinas EC2.
	Comando: 
	https://console.aws.amazon.com/ec2/v2/home

5.2 Descobrindo o IP
	Para descobrir o IP, basta ir no menu lateral esquerdo "INSTANCES", sub menu "INSTANCES", será mostrada na parte direita diversas máquinas, se houver. Contudo, deverá haver uma máquina chamada "docker-node". 
	Para ver o IP no qual a AWS atribuiu para a mesma basta clicar no nome da maquina e na parte inferior do console será mostrada uma área com diversas informações da máquina. Localize a opção "IPv4 Public IP" que mostra o IP público da instância AWS. A mesma pode ser acessada pelo endereço DNS ou ou pelo subdominio gerado automaticamente "Public DNS (IPv4)"
	exemplo:
	Public DNS (IPv4): 	ec2-52-23-232-102.compute-1.amazonaws.com
	IPv4 Public IP: 52.23.232.102

5.3  Testando a aplicação online
	Para testar se a aplicação está online, basta utilizar uma das duas opções acima e digitar no navegador ou outra aplicação de sua escolha como Postman ou Insomnia a requisição GET a seguir:
	exemplo: http://ec2-52-23-232-102.compute-1.amazonaws.com:3000
			 http://52.23.232.102:3000
	
	Caso tudo tenha sido realizado com sucesso, o retorno do endpoint "/" do verbo http GET deverá ser "Hello World"

6 - Bônus (Prometheus e Grafana)
	Ademais as tecnologias já instaladas, foi adicionado uma pasta monitoramento para rodar como um módulo local, chamada monitoramento.
	Dentro desta pasta estão duas aplicações das mais utilizadas no mercado, o Prometheus e Grafana. 
	A primeira temo objetivo de buscar os dados de métricas nas apis das aplicações. O segundo tem como objetivo criar dashboard para melhor visualização desses dados. Foi adicionado um servido de teste de dados de metricas conhecido como Exporter, para gerar dados para o prometheus.
	
	Para levantar os serviços basta entrar na pasta "monitoramento" e digitar o comando:
		make up

	Para acessar:
		Prometheus - http://localhost:9090
		Grafana - http://localhost:4000
		node-exporter - http://localhost:9100

7 Orientações sobre o makefile
	O Makefile é um arquivo criado para abstrair os comandos do docker-compose e facilitar a forma de como subir, destruir e ver os logs dos serviços docker.
	Ele possui 3 comandos.
	make up - para criar os containers docker
	make down - para destruir os containers
	make logs - para ver os logs de um container que esta ativo


8 Tecnologias utilizadas:
	- docker 1.13.1
	- docker-compose 1.24.1 - 1.25.5
	- vagrant 2.2.6
	- ansible 2.5.3
	- docker-machine 0.16.0
	- AWS-CLI/2.0.14 Python/3.7.3 Linux/4.15.0-29deepin-generic botocore/2.0.0dev18
	- Prometheus:latest
	- Grafana:latest