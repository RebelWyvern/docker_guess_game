# Guess Game Utilizando Docker

O objetivo deste projeto foi criar uma estrutura em docker que comportasse um projeto existente já desenvolvido, o Guess Game, que foi criado para rodar em uma maquina local. Para que fosse possivel adaptar
esse projeto para Docker, foi necessário dividir o programa em três containers, que se comunicam entre si.

## Instalações necessárias

1. Docker: Como o projeto roda em Docker, é necessário te-lo instalado para rodar a estrutura;
2. Docker-compose: Nem sempre o docker-compose é instalado junto com Docker, então é essencial certificar que ele também está instalado;
3. (Opcional) Docker Desktop: Essa aplicação do Docker não é obrigatória, mas permite ver com mais detalhes a estrutura criada pelo Docker.

## Configuração dos Containers 

1. Container do Banco de Dados: utilização da imagem do postgres para rodar um banco de dados que possui seu proprio volume separado para garantir a segurança dos dados armazenados. esse banco guarda as informações relativas
as respostas do guess game e o ID dos games. O Dockerfile do banco segue os passos de configuração dados pelo projeto original.
    
3. Container do Backend: Comunica o banco sobre os dados que devem ser armazenados que foram passados pelo Frontend, assim como a geração do ID da sessão do game. Essa container possui o próprio Dockerfile que segue as seguintes instruções:
    * Instalar as dependencias citadas no documento do projeto requeriments.txt. 
    * Copiar o codigo referente ao backend original do programa;
    * configurar o container para rodar na porta 4000;
    * rodar o arquivo start-backend.sh, responsavel pela contrução do backend do projeto original
5. Container do Frontend: Utiliza Nginx para servir o framework do projeto base, onde também ocorre o uso de proxy reverso e balanceamento de carga das estâncias do Backend. Assim como Backend, possui o proprio Dockerfile, onde:
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

 ## Como rodar a estrutura

 
