# Capítulo 02 - Imagens

## Dockerhub

O dockerhub é um repositório remoto de imagens que oferece os seguintes serviços:
- Hospedagem e imagens Docker;
- Autenticação de usuário;
- Automatização do processo de construção de imagens através
de triggers (webhooks);
- Integração com o Github e Bitbucket.

### Fazendo login no Dockerhub pelo terminal

Para fazer o login:

~~~shell
docker login -u <usuario_dockerhub>
~~~

Verificando a criação do arquivo de autorização:

~~~shell
cat ~/.docker/config.json
~~~

Encerrando o login:

~~~shell
docker logout
~~~

## Docker Image

### Gerenciando imagens no Docker

Liste as imagens e verifique o histórico de comandos utilizados para sua contrução:

~~~shell
docker image ls
docker image history debian
~~~

O comando **docker image history** mostra as camadas que compoem uma imagem. 

Para inspecionar uma imagem utilizamos o comando **docker image inspect**.

~~~shell
docker image inspect debian
~~~

Com o **inspect** é possível obter informações detalhadas do container.

#### Criando um novo container e salvando as mdificações com o comando **commit**

Crie o container e instale novos pacotes:

~~~shell
docker container run -dit --name servidor-debian debian
docker container exec servidor-debian apt-get update
docker container exec servidor-debian apt-get install nginx -y
~~~

Após instalar os pacotes, podemos criar a nova imagem (nova camada) com o comando **commit**:

~~~shell
docker container commit servidor-debian webserver-nginx
docker image ls
~~~

Obs.: O comando docker container commit <container> <imagem> cria uma imagem a partir de alterações realizadas em um container, este procedimento não é o recomendado para este fim, mais a frente veremos outras soluções.

Para salvar a imagem podemos utilizar o parâmetro **save**:

~~~shell
docker image save webserver-nginx -o imagem-webserver-nginx.tar
~~~

Verificando o diretório atual é possível observar que a imagem foi salva no arquivo **imagem-webserver-nginx.tar**.

Para ver o tamanho e o nome da imagem gerada utilize o comando abaixo:

~~~shell
du -sh imagem-webserver-nginx.tar 
~~~

Após realizar esses procedimento remova o container e a imagem:

~~~shell
docker container rm -f servidor-debian
docker image rm webserver-nginx
docker image ls
~~~

#### Observações sobre o linux

Comandos du e df
- **du**: O comando du, sem qualquer opção e sem nome de arquivo ou diretório, fornece a quantidade de espaço ocupada por cada subdiretório que se encontra hierarquicamente abaixo do diretório atual.
- **df**: O comando df exibe informações sobre os espaços, livres e ocupados, das partições.

#### Carregando imagem com o comando **load**

~~~shell
docker image load -i imagem-webserver-nginx.tar
docker image ls
~~~

Testando a imagem carregada:

~~~shell
docker container run -dit --name webserver webserver-nginx
docker container ls
~~~

#### Removendo todos os containers

~~~shell
docker container rm -f $(docker container ls -aq)
docker container ls
~~~

No comando acima, estamos passando através de um subshell o comando docker container ls -aq que lista todos os containers por id, sendo assim, será feita a remoção de todos os containers.

## Dockerfile

O Dockerfile é um arquivo de instruções de como deve ser gerada a imagem Docker.

As imagens são criadas a partir do Dockerfile com o comando **docker build**.

### Sintaxe

| Parâmetro | Valor                                       |
|-----------|---------------------------------------------|
| FROM      | Distribuição:Versão                         |
| COPY      | Arquivo_Local Caminho_Absoluto_no_Container |
| RUN       | Comando                                     |
| EXPOSE    | Porta do serviço                            |
| CMD       | Comando executado ao iniciar o Container    |

Obs.: O nome do arquivo deve ser Dockerfile.

Boa prática 1: Utilizar caixa alta nos parâmetros e caixa baixa nos valores.
Boa prática 2: Utilize o **#** para fazer comentários que facilitem o entendimento.

Definições:

- **FROM**: Inicializa um novo estágio de compilação e define a imagem de base para instruções subsequentes;
- **COPY**: Copia arquivos ou diretórios de origem local adicionando-os a imagem do container;
- **ADD**: Similar ao parâmetro COPY porém possibilita que a origem seja uma URL bem como a alteração de permissionamento ao adicionar os arquivos a imagem do container;
- **RUN**: Executa os comandos em uma nova camada na parte superior a imagem atual, é uma boa prática combinar diversos comandos em um unico RUN utilizando de ; e && para a combinação, assim criando apenas uma camada;
- **EXPOSE**: Informa ao docker a porta na qual o container estará escutando enquanto estiver sendo executado, é possível especificar portas TCP e UDP, caso não seja declarado o tipo de porta, o padrão (TCP) é assumido;
- **CMD**: Só pode existir uma unica instrução deste tipo em um arquivo, o propósito desta instrução é prover os padrõespara a execução do container, podendo ser um executável ouaté mesmo opções para o executável definido na instrução**ENTRYPOINT**;
- **ENTRYPOINT**: Possibilita configurar o container para rodar como um executável, o comando docker run <image> inicializará o container em seu entrypoint somado ao CMD se existente.

### Criando Dockerfiles

#### Primeiro Dockerfile:

Escreva o conteúdo abaixo no arquivo:

~~~Dockerfile
FROM alpine
ENTRYPOINT ["echo"]
CMD ["--help"]
~~~

Para criar essa imagem utilize o comando **docker image build**:

~~~shell
docker image build -t echo-container .
docker image ls
~~~

A opção **-t** significa TAG, após o nome da **imagem:tag** informamos qual o diretório que o Dockerfile se encontra. Para utilizer o diretório atual (PWD) use o ponto **( . )**.

Execute o container com a imagem criada:

~~~shell
docker container run --rm -it echo-container
~~~

Utilizando a opção **-rm** informamos que o container será apagado após cumprir seu papel.

Executando e passando um parâmetro:

~~~shel
docker container run --rm -it echo-container Container DevOps
~~~

Ao passar um parâmetro após o nome da imagem estamos alterando o CMD do container, anteriormente echo –help para echo Container DevOps.

#### Dockerfile de um servidor WEB

~~~Dockerfile
FROM debian
RUN apt-get update && apt-get install wget git apache2 -y
EXPOSE 80
CMD ["apachectl", "-D", "FOREGROUND"]
~~~

Criando a imagem:

~~~shell
docker image build -t webserver .
docker image ls
~~~

#### Enviando a imagem para o Dockerhub

Para enviar a imagem para o Dockerhub é necessário executar 3 passos:
- docker login
- docker image tag
- docker push

##### Login no Dockerhub

Faça o login com o comando:
~~~shell
docker login -u <usuario_dockerhub>
~~~

##### Crie a tag da imagem

Precisaos que a tag da imagem siga o padrão: **usuario/imagem:versao**.

~~~shell
docker image tag echo-container <usuario_dockerhub>/echo-container:latest
docker image tag webserver <usuario_dockerhub>/webserver
~~~

Obs.: Caso não seja informada uma versão, o docker entende que a versão
trata-se da latest.

##### Enviando para o Dockerhub

~~~shell
docker image push <usuario_dockerhub>/echo-container
docker image push <usuario_dockerhub>/webserver
~~~

Ao finalizar o procedimento, faça logout em sua conta do dockerhub:

~~~shell
docker logout
~~~

### Melhores práticas com o Dockerfile

Primeiras recomendações:
- Os containers devem ser tão **efêmeros** quanto possível, isso quer dizer que o con-
tainer deve poder ser parado e/ou destruido a qualquer momento, e reconstruido ou substituido com o mínimo de configuração ou atualização.
- Os containers devem subir de maneira stateless (sem estado).

#### Entendendo o contexto de Build

O **build context** é o diretório onde o comando **docker image build** é executado Nesse diretório só pode conter o conteúdo necessário para a criação da imagem pois, esse contaúdo será enviado para o docker deamon.

##### Excluíndo arquivos do build

Para excluir os arquivos que não naõ importantes para o build, podemos criar um arquivo **.dockerignore**. Esse arquivo contêm os padrões de exclusão e é semelhante aos do **.gitignore**, possibilitando que ignoremos arquivos no build sem modificar o nosso repositório.

Criando arquivo .dockerignore:

~~~shell
vim context/.dockerignore
~~~

Conteúdo do arquivo:

~~~shell
# Comentario: Ignorando arquivos do diretorio log
log
~~~

Criando a imagem:

~~~shell
docker image build --no-cache -t exemplo:v4 -f image/Dockerfile context
~~~