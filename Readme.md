# Ruby on Rails e Docker - parte 01.
Crie um aplicativo RoR 7 com Docker pela primeira vez.

Nesta lição você poderá visualizar a configuração feita para criação de aplicações Ruby on Rails 7 utilizando Docker. Antes de continuar, certifique-se de ter o Docker instalado em seu computador.



Observações.

Se você estiver usando Windows, recomendo instalar o WSL2 para que o processo não demore muito. Se você usa Linux, o processo de criação de contêineres será muito mais rápido. Neste link https://learn.microsoft.com/es-es/windows/wsl/install você encontrará mais informações sobre como instalar o WSL2 no Windows.



Desenvolvimento.



1. Crie uma pasta e nomeie-a como quiser. Dentro da pasta, crie um arquivo chamado Dockerfile e cole a seguinte configuração. Na linha WORKDIR, digite o nome desejado. Este diretório será criado no contêiner.

FROM ruby:3.0.4

WORKDIR /myapp

RUN apt-get update -qq && apt-get install -y nodejs postgresql-client imagemagick libvips



2. Instale a extensão VSCode Dev Containers.


3. Digite ctrl + shift + p e selecione a opção Reconstruir sem cache e reabrir no container.

3.1. De Dockfile ok

4. O container começará a ser criado, então você terá que esperar até que ele termine.

5. Abra um novo terminal e ele será aberto dentro do contêiner, mas não no seu computador local. Crie um aplicativo Ruby on Rails usando o comando rails new myapp.

Você pode adicionar o banco de dados usando o comando rails new myapp -d=postgresql

# Ruby on Rails e Docker - parte 02.
## Aplicativo Dockerize Ruby on Rails 7.

Agora, neste ponto você deve ter seu aplicativo Ruby on Rails 7 gerado.

Entramos no aplicativo a partir do nosso terminal local, mas não do contêiner, então você terá que fechar o VSCode, abrir um terminal no seu computador local, entrar na pasta do aplicativo e abrir o VSCode a partir daí.

Desenvolvimento.

1. Crie um Dockerfile na raiz da aplicação. Podem ser solicitadas permissões de edição se você estiver usando Linux, então abra um novo terminal e digite o seguinte comando: sudo chown -R nomedeusuário:nomedeusuário . (nome de usuário é o seu nome de usuário do Linux).

FROM ruby:3.0.4

WORKDIR /myapp

RUN apt-get update -qq && apt-get install -y nodejs postgresql-client imagemagick libvips

COPY Gemfile .

COPY Gemfile.lock .

RUN bundle install

COPY . .

COPY entrypoint.sh /usr/bin/

RUN chmod +x /usr/bin/entrypoint.sh

ENTRYPOINT ["entrypoint.sh"]

2. Crie um arquivo chamado entrypoint.sh na raiz do aplicativo e adicione a seguinte configuração. Tenha cuidado com os comentários, pois eles podem levar a erros inesperados.

#!/bin/bash
set -e
 
# Remove a potentially pre-existing server.pid for Rails.
rm -f /myapp/tmp/pids/server.pid
 
# Then exec the container's main process (what's set as CMD in the Dockerfile).
exec "$@"

3. Crie um arquivo docker-compose.yml para registrar o contêiner Postgres junto com a aplicação; e adicione a seguinte configuração.

versão: "3.8"
  Serviços:
    base de dados:
      imagem: postgres
      volumes:
        - ./tmp/db:/var/lib/postgresql/data
      ambiente:
        POSTGRES_USERNAME: postgres
        POSTGRES_PASSWORD: senha
 
    desenvolvimento:
      construir:
        contexto: ./
        dockerfile: Dockerfile
      depende de:
        -base de dados
        
4. Na raiz da pasta do projeto rails pressionamos ctrl + shift + p e selecionamos a opção Adicionar arquivos de configuração do Dev Container....

5. Selecionamos a opção "From 'docker-compose.yml'" e o serviço de desenvolvimento. Será criada uma pasta chamada .devcontainers e na linha 19 substituímos o nome padrão pelo nome que registramos em nosso Dockerfile na instrução WORKDIR.

6. Atualizamos a configuração do arquivo docker-compose.yml localizado na mesma pasta.

7. Para finalizar salvamos e digitamos ctrl + shift + p e selecionamos a opção Reconstruir sem cache e Reabrir no Container.

8. Esperamos que termine e adicione a configuração de conexão ao banco de dados contêiner adicionando a seguinte configuração. O nome do host deve ser igual ao nome do serviço que registramos em nosso arquivo docker-compose.yml que criamos no início do processo na pasta principal do projeto.
