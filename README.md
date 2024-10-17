# Guess Game Utilizando Docker

O objetivo deste projeto foi criar uma estrutura em docker que comportasse um projeto existente já desenvolvido, o Guess Game, que foi criado para rodar em uma maquina local. Para que fosse possivel adaptar
esse projeto para Docker, foi necessário dividir o programa em três containers, que se comunicam entre si.

## Instalações necessárias


* Docker: Ferramenta de containerização para rodar aplicações de maneira isolada.
* Docker Compose: Ferramenta para definir e rodar multi-containers Docker, permitindo o uso de um arquivo docker-compose.yml para orquestrar os serviços.
1. Instalação no Linux
* Passo 1: Instalar o Docker

    * Atualize o gerenciador de pacotes:

            sudo apt update

    * Instale pacotes necessários:

            sudo apt install apt-transport-https ca-certificates curl software-properties-common

    * Adicione a chave GPG oficial do Docker:

            curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

    * Adicione o repositório do Docker:

            echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee/etc/apt/sources.list.d/docker.list > /dev/null

    * Instale o Docker:

            sudo apt update
            sudo apt install docker-ce
    * Verifique se o Docker foi instalado corretamente:

            sudo systemctl status docker

* Passo 2: Instalar o Docker Compose

    * Baixe a versão mais recente do Docker Compose:

              sudo curl -L "https://github.com/docker/compose/releases/download/v2.10.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
              
    * Dê permissões de execução:

                sudo chmod +x /usr/local/bin/docker-compose
   
    * Verifique a versão instalada:

                docker-compose --version

2. Instalação no Windows
* Passo 1: Instalar o Docker Desktop

    * Acesse Docker Desktop para Windows e baixe o instalador.
    
    * Execute o instalador e siga os passos da instalação. Certifique-se de que a opção de instalar o Docker Compose está selecionada.
    
    * Reinicie o computador, caso necessário.

* Passo 2: Verificar a Instalação

    * Após reiniciar, abra o Docker Desktop e verifique se o Docker está rodando.
    
    * Abra o Prompt de Comando ou PowerShell e execute o seguinte comando para verificar a instalação do Docker Compose:

            docker-compose --version

3. Instalação no macOS
* Passo 1: Instalar o Docker Desktop

    * Baixe o Docker Desktop para Mac a partir de Docker Desktop for Mac.

    * Abra o arquivo .dmg baixado e arraste o ícone do Docker para a pasta de Aplicativos.

    * Abra o Docker Desktop a partir da pasta de Aplicativos e siga as instruções para completar a instalação.
    
    * Uma vez instalado, o Docker Compose também estará disponível.

* Passo 2: Verificar a Instalação

    * Abra o Terminal e execute o seguinte comando para verificar se o Docker Compose está corretamente instalado:

            docker-compose --version

## Configuração dos Containers 

* Container do Banco de Dados: utilização da imagem do postgres para rodar um banco de dados que possui seu proprio volume separado para garantir a segurança dos dados armazenados. esse banco guarda as informações relativas
as respostas do guess game e o ID dos games. O Dockerfile do banco segue os passos de configuração dados pelo projeto original.
    
* Container do Backend: Comunica o banco sobre os dados que devem ser armazenados que foram passados pelo Frontend, assim como a geração do ID da sessão do game. Essa container possui o próprio Dockerfile que segue as seguintes instruções:
    * Instalar as dependencias citadas no documento do projeto requeriments.txt. 
    * Copiar o codigo referente ao backend original do programa;
    * configurar o container para rodar na porta 4000;
    * rodar o arquivo start-backend.sh, responsavel pela contrução do backend do projeto original
* Container do Frontend: Utiliza Nginx para servir o framework do projeto base, onde também ocorre o uso de proxy reverso e balanceamento de carga das estâncias do Backend. Assim como Backend, possui o proprio Dockerfile, onde:
    * Copia os arquivos de configuração existentes do projeto para a instalação das dependencias;
    * Instala essas dependencias;
    * Usa uma varivel de ambiente que será passada para o codigo Frontend que localiza onde o Backend está rodando;
    * Copia a parte da aplicação do frontend para o container;
    * Construi a aplicação de produção e copia para o diretorio padrão de aplicação do Nginx;
    * Copia também o arquivo de configuração do Nginx que estabelece o proxy reverso e o balanceamento de carga do Backend para o diretorio padrão de configuração;
    * Estabele a porta onde o Nginx rodará a aplicação.

Essa estrutura foi contruida usando docker-compose, onde cada parte possui seu proprio dockerfile, que utiliza instruções sobre como cada um deve ser configurado.
Assim cada parte da estrutura pode ser atualizada e subida individualmente se necessario, e se alguma delas apresentar falha, seu container reinicia automaticamente.

 ## Configuração do Nginx

 O Nginx serve arquivos estáticos para as rotas normais (/) e encaminha requisições da rota /guess_game para um conjunto de servidores backend localizados na porta 4000, fazendo balanceamento de carga entre eles. Ele também configura permissões para requisições CORS, garantindo que o frontend possa interagir com a API backend:
 
* Escutar na porta 80:
    * O servidor está configurado para ouvir requisições HTTP na porta 80 (mas que no docker-compose é configurado para estar disponivel na porta 3000), que é a porta padrão para HTTP.
    * O servidor de nome configurado é localhost (server_name localhost;), então ele tratará requisições feitas para esse domínio;
      
* Servir arquivos estáticos:
    * A diretiva location / indica que qualquer requisição feita à raiz (/) do servidor será tratada servindo arquivos estáticos a partir do diretório /usr/share/nginx/html.
    * O comando try_files $uri $uri/ /index.html; tenta encontrar o arquivo solicitado pelo cliente;
* Proxy para o backend:
    * A diretiva location /guess_game define um caminho específico para as requisições relacionadas ao jogo. Quando uma requisição é feita para este caminho, o Nginx a encaminha (faz um proxy) para o backend configurado no grupo de servidores chamado backend_pool (proxy_pass http://backend_pool;);
* Cabeçalhos de CORS (Cross-Origin Resource Sharing): As diretivas add_header são usadas para permitir o compartilhamento de recursos entre diferentes origens. Neste caso:
    * 'Access-Control-Allow-Origin' '*' permite que qualquer domínio acesse o recurso;
    *'Access-Control-Allow-Methods' 'GET, POST, OPTIONS' define os métodos HTTP permitidos;
    * 'Access-Control-Allow-Headers' 'Content-Type, Authorization' especifica os cabeçalhos permitidos nas requisições.
* Requisições de preflight:
    * O bloco if ($request_method = OPTIONS) trata as requisições OPTIONS, que são enviadas automaticamente pelos navegadores antes de uma requisição real, para verificar se o servidor permite a operação de CORS. Se o método for OPTIONS, o Nginx retorna um status 200, confirmando que a operação é permitida.
* Definição do Pool de Servidores do Backend:
    * upstream backend_pool: Define um grupo de servidores chamado backend_pool, que o Nginx utilizará para balancear a carga das requisições;
    * ip_hash: Garante que as requisições de um mesmo cliente (com o mesmo endereço IP) sempre vão para o mesmo servidor. Isso é útil para manter a sessão de um usuário sempre no mesmo servidor;
    * server 127.0.0.1:4000; e server localhost:4000;: Define os servidores do pool que estão rodando localmente na porta 4000.


 ## Como rodar a estrutura

Para rodar a estrutura completa:

    docker-compose up --build

Para rodar apenas um dos containers:

    docker-compose build <nome_do_serviço>

* O nome do serviço no caso se refere ao nome contido no docker-compose.
* Quando for preciso atualizar o conteudo de um do containers, é possivel editar o arquivo e usar o mesmo comando usado para rodar o container especifico, sem a necessidade de derrobar toda a arquitetura.

Para parar completamente a estrutura:

        docker compose down


        



 
