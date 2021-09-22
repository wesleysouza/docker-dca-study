# Capítulo 01 - Fundamentos

## Namespaces

Fornecem isolamento para os containers, limitando seu acesso aos recursos do sistema e a outros namespaces. Isto significa, por exemplo, que um usuário root de um container é diferente de um usuário root da máquina hospedeira.

Dentro de um Namespace tem-se:
- PID: Process ID
- MNT: Mount Points
- IPC: Comunicação Inter Processos
- UTS: Unix Timesharing System (Kernel e Identificadores)
- NET: Networking

Com o isolamento, os sistemas em execução nos containers tem suas próprias árvores de processo, sistemas de arquivos, conexões de rede e muito mais.

## Cgroups

Os containers trabalham com cgroups (Control Groups) que fazem isolamento dos recursos físicos da máquina. Em geral os cgroups podem ser utilizados para controlar estes recursos tais como limites e reserva de CPU, limites e reserva de memória, dispositivos, etc…

- cpu: Divisão de CPU por containers.
- cpuset: CPU Masks, para limitar threads
- memory: Memória
- device: Dispositivos

## Instalação do Docker

Para instalar basta executar o comando abaixo:

~~~shell
sudo curl -fsSL https://get.docker.com | bash
~~~

Em sistemas RHEL será necessário iniciar o serviço após a instalação com os comandos:

~~~shell
sudo systemctl enable docker
sudo systemctl start docker
~~~

Após a conclusão da instalação, podemos configurar agora nosso usuário para fazer parte do grupo docker , isso garantirá que possa-mos executar os comandos do docker sem a necessidade de elevar os privilégios. Sendo assim, execute o comando:

~~~shell
sudo usermod -aG docker $USER
~~~

Pra finalizar a instalação, vamos adicionar o recurso de Bash Completion:

~~~shell
sudo curl https://raw.githubusercontent.com/docker/machine/v0.16.0/contrib/completion/bash/docker-machine.bash -o /etc/bash_completion.d/docker-machine
~~~

Testando a instalação:

~~~shell
docker container run --rm -it hello-world
~~~

## Componentes

Os componentes do Docker são o **Docker Client**, **Docker Deamon** e o **Docker Registry**:

- **Docker Client**: é o pacote docker-ce-cli ele fornece os comandos do lado do cliente, como por exemplo o comando docker container run , que irá interagir com o Docker Daemon.
- **Docker Deamon**: é o pacote docker-ce ele é o servidor propriamente dito, que receberá os comandos através do Docker Client e fornecerá os recursos de virtualização a nível de sistema operacio- nal.
- **Docker Registry**: O Docker Registry é o local de armazenamento de imagens Docker, normalmente o Docker hub, de onde o Docker Daemon receberá as imagens a serem executadas no processo de criação de um container.

## Comandos Essenciais

Comando de ajuda do docker:

~~~shel
docker --help
~~~

### Comandos de Gerenciamento

Os comandos de gerenciamento do docker são:
- docker container
- docker image
- docker network
- docker system
- docker volume

Para cada um desses comandos de gerenciamento existem vários subcomandos que podem ser executados.

### Executando Comandos

Coletando informações do ambiente:

~~~shell
docker system info
docker info
~~~

Ambos os comandos acima são equivalentes.

#### Obtendo informação com mais granularidade:

Para listar containers, imagens, redes e volumes no docker, utilizamos o comando docker <comando> ls:

- docker container ls - lista os containers em execução (adicionando parâmtro -a também mostra os que estão parados)
- docker image ls - lista as imagens
- docker network ls - lista as redes
- docker volume ls - lista os volumes

Para pesquisar uma imagem podemos utilizar o comando **docker search**, exemplo:

~~~shell
docker search debian
~~~

#### Baixando e executando imagens

Para efetuar o download de uma imagem utilizamos o comando **docker image pull**:

~~~shell
docker image pull debian
~~~

Para executar um container, utilizamos o comando docker **container run**:

~~~shell
docker container run -dit --name debian1 --hostname c1 debian
~~~

Descrição do comando:
- **docker container run (…) debian** - Executa um container, sendo o último parâmetro o nome da imagem a ser utilizada;
- **-dit** - Executa um container como processo (d = Detached), habilitando a interação com o container (i = Interactive) e disponibiliza um pseudo-TTY(t = TTY);
- **--name** - Define o nome do container;
- **--hostname** - Define o hostname do container.

Veja o seu novo container rodando com o comando:

~~~shell
docker container ls
~~~

Outros comandos:
- **docker contaniner attach debian1**: entrando no container debian1;
- **docker container start**: iniciando container debian1;
- **docker container stop**: parando container docker1;
- **docker container rm debian1**: removendo o container debian1

Obs.: O container deve estar parado para ser possível a remoção. Caso queira remover um container em execução use o parâmetro **-f**:

~~~shell
docker container rm -debian1
~~~

Obs.: Para sair do container sem <para-lo> utilize a sequencia de teclas: ** < CTRL > + < P > + < Q > **, chamada de READ ESCAPE SEQUENCE.

#### Verificando logs

Para verificar os logs do container utilizamos o comando **docker container logs**:

~~~shell
docker container logs debian1
~~~

#### Enviando arquivos para o container

Crie um arquito de teste na pasta atual para enviar para o container c1:

~~~shell
echo "Arquivo de teste" > /tmp/arquivo
docker container cp /tmp/arquivo c1:/tmp
~~~

O comando **docker container cp** copia um arquivo da maquina host para o container ou vice-versa.

#### Executando comandos dentro do container

O comando **docker container exec** executa um comando no container e envia o retorno na saída padrão (STDOUT) da máquina.

Obs.: **caso o container tenha sido iniciado com a opção -i.**

Exemplo:

~~~shell
docker container exec c1 ls -l /tmp
docker container exec c1 cat /tmp/arquivo
~~~

