# ProjetoIntegrador_AcademiaNeon
Projeto Integrador para a Academia Neon DevOps - Grupo DevOpers - Utilizando Ansible, Docker, Jenkins, AWS

## Sobre o Projeto Integrador
Introdução: Será fornecido um APP feito em NodeJS que escreve e lê arquivos no S3. As configurações necessárias para esse app funcionar serão definidas por variáveis de ambiente (environment)

O objetivo: Criar as duas máquinas de app na AWS junto com uma máquina que conterá o Jenkins (para facilitar a evolução
fora da sala de aula). Com o jenkins no ar elas deverão construir as pipelines clonando o projeto, executando os testes e configurando 
com as devidas variáveis de ambiente e por fim publicando no ambiente de destino: Prod ou Homolog

Resultados Esperados: Com os aplicativos NodeJS no ar as alunas devem observar a url /healthcheck de cada um deles para ver se foram ou não configurados com sucesso. Deverão testar a ação de upload de imagens da aplicação que foi fornecida e se estiver com os tokens corretos do S3 fará o upload com sucesso.

## Arquitetura

![Arquitetura](docs/DevOpers_ArquiteturaPI.jpg)

## Provisionando EC2 + S3 + IAM + ECR na AWS
Partindo do presuposto que seu ambiente de trabalho já está preparado com as devidas ferramentas instaladas e configuradas, seguir os seguintes passos para criar uma infra na AWS. Caso não tenha um ambiente de trabalho por favor verificar o repositório Academia_Neon_DevOps.

O objetivo é criar um ambiente de Homologação e Produção + um servidor que rodará o Jenkins.
* Ao ser executado o item abaixo irá criar uma VPC e todos os itens obrigatórios relacionados a ela (por exemplo, subnet, security group, entre outros), logo após irá gerar uma key-pair e salvar. Logo após irá criar uma EC2 para o Jenkins e um ECR (Elastic Container Registry), em seguida uma EC2 para Produção, um user e um bucket S3. E por último criará uma EC2 para Homologação, um user e um bucket S3

* * `$ ansible-playbook playbooks/aws_provisioning.yml`

O script acima utiliza os seguintes scripts: aws_provisioning_vpc.yml, aws_provisioning_jenkins.yml, aws_provisioning_producao.yml, aws_provisioning_homolog.yml

* Para validar se foi criado acessar o console da AWS ou executar o seguinte comando:
* * `$ ansible-inventory --graph aws_ec2`

## Configurando Maquinas EC2 
Após todo o ambiente criado na AWS, precisamos configurar as máquinas, todas as 3 EC2 criadas terão os pacotes atualizados e os seguintes itens instalados via ansible: docker, awscli, java, python, entre outros. Já a EC2 do Jenkins vamos instalar também o ansible e o jenkins. 

* Atualizar pacotes e instalar awscli, java, python - Esse script foi preparado para atualizar as 3 EC2 ao ser executado...
* * `$ ansible-playbook playbooks/config_all-ec2.yml`

* Instalar docker - Esse script foi preparado para atualizar as 3 EC2 ao ser executado...
* * `$ ansible-playbook playbooks/install_docker_all-ec2.yml`

* Instalar ansible - Esse script foi preparado para atualizar apenas a EC2 do Jenkins ao ser executado...
* * `$ ansible-playbook playbooks/install_ansible_ec2-jenkins.yml`

* Instalar jenkins - Esse script foi preparado para atualizar apenas a EC2 do Jenkins ao ser executado...
* * `$ ansible-playbook playbooks/install_jenkins_ec2-jenkins.yml`

## Configurando Jenkins 
Instalar plugins Jenkins que vamos utilizar em nosso pipeline.
Acessar a url de acordo com o nome ou ip publico gerado na AWS, para isso será necessário entrar no console.

Após logar no Jenkins com usuário e senha definidos no playbook install_jenkins_ec2-jenkins.yml, ir em Gerenciar Jenkins → Gerenciar de Plugins → Disponíveis → procurar e selecionar os seguintes itens:
    1.pipeline
    2.docker pipeline
    3.ssh
    4.github
    5.github api
    6.amazon ecr

Acessar a EC2 via ssh conforme exemplo abaixo (é possivel pegar esse comando no console da AWS também) para validar se o usuário jenkins está incluso no grupo docker do Linux (id jenkins):
ssh -i <chave que salvou>.pem ubuntu@<nome ou ip publico>   

Caso o usuário não esteja executar o comando abaixo para incluir e depois reiniciar o serviço.
* Verificar usuário
* * `$ id jenkins` 
* Incluir o usuário no grupo
* * `$ sudo addgroup jenkins docker` 
* Parar o serviço do jenkins
* * `$ sudo service jenkins stop`
* Subir o serviço do jenkins
* * `$ sudo service jenkins start`
* Validar o serviço do jenkins
* * `$ sudo service jenkins status`

Configurar nodes Homolog e Produção para a execução do pipeline
Gerenciar Jenkins → Gerenciar nós → novo nó

Adicionar Credenciais para acessar as instâncias
Credentials → Add Credentials
Global e SSH username + private key

Adicionar Credenciais para acessar AWS ECR
Credentials → Add Credentials
AWS Credentials + Global + ID e Private Key AWS

## Pipeline Jenkins
Após realizar todas as configurações é possível criar um Job Pipeline que irá no repositório git e fará todo o deploy através do Jenkinsfile.

## APP NodeJS utilizado
Foi realizado um fork do repositório https://github.com/bgsouza/digitalhouse-devops-app.git para https://github.com/andresavs/digitalhouse-devops-app.git pois teriamos que customizar o arquivo Jenksfile.

## Documentação Oficial

* https://www.ansible.com/
* https://github.com/
* https://www.docker.com/
* https://www.jenkins.io/
* https://aws.amazon.com/pt/
