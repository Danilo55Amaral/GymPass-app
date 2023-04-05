# API Node.js com SOLID

# Definindo requisitos e regras 

## GymPass style app

# Criando projeto Node.js

- Para criar a estrutura do projeto em Node.js eu rodei o comando abaixo e ele vai 
criar o package.json

        npm init -y 

- Crei a pasta src e dentro dessa pasta eu crio um server.ts que é o servidor HTTP
eu instalo o typeScript junto com o Types/node e também o tsx o tsx vai ser para 
executar o código em TS pois o node só entende JS. Também adicionei o tsup que é 
a biblioteca de build da aplicação para quando for colocar em produção gerar a build.

    npm i typescript @types/node tsx tsup -D 

- Após instalar essas libs executo o npx tsc --init para criar o arquivo tsconfig.json
eu posso mudar o padrão es2016 para "target": "es2020". 

- Dentro do arquivo server.ts vou criar a estancia do servidor utilizando o Fastify 
e para isso eu devo instalar o Fastify rodando o comando abaixo 

    npm i fastify 

- Também foi criado o arquivo app.ts onde deve ser importado o fastify e nesse arquivo 
é criada a aplicação utilizando o fastify() e também é exportada.

import fastify from "fastify";

export const app = fastify()

- Dentro do server.ts é feita a importação desse app e em cima de app deve-se executar 
o método listen() dentro desse método é passada a porta da aplicação, uma dica legal é 
quando criar uma aplicação com fastify utilizar o host: '0.0.0.0' isso faz com que essa 
aplicação no fututo seja acessivel por fronts que estejam consumindo essa aplicação,
caso esse host não seja utilizado pode haver alguns problemas na hora de cocnectar essa 
api com um front por ex.

- Eu posso utilizar o then() por que o listen vai retornar uma promise e com isso pode 
aguardar o servidor subir ouvindo a porta e quando ele terminar eu dou um console.log 
com uma mensagem HTTP Server Running! 

import { app } from "./app";

app.listen({
    host: '0.0.0.0',
    port: 3333,
}).then(() => {
    console.log('HTTP Server Running!')
})

- Também ja criei o arquivo gitignore e coloquei a pasta node_modules e a pasta build

- Dentro do package.json já deixei configurado alguns scripts o start:dev onde vai 
executar o tsx que é a lib que vai executar a aplicação, passei também o watch para 
a aplicação ficar observando mudanças e executar de forma automatica toda vez que ocorrer
e passando o caminho do meu server rodando esse comando ele vai rodar o servidor que criei. Com isso basta rodar o comando  npm run start:dev para estartar o servidor.

"start:dev": "tsx watch src/server.ts"

- Também criei um script para executar a build onde passei para ele executar o tsup 
na pasta src e vou utilizar --out-dir passando uma pasta para onde quero que seja 
direcionada a build. 

"build": "tsup src --out-dir build" 

- Criei também um script chamado start que ao invés de startar o servidor para 
desenvolvimento ele vai startar para produção executando a nossa build.

"start": "node build/server.js"

# Usando versões exatas do NPM

- Quando o projeto estiver rodando em produção umas das coisas importantes é sempre 
atualizar as dependencias do projeto, isso é importante por questão de segurança ou 
performance, por isso é normal ao decorrer do tempo querer evoluir as dependencias,
não tem como pegar e sair atualizando todas as dependencias pois isso poderia trazer 
alguns problemas, por isso é importante quando construir o projeto instalar as versões
exatas das dependencias, isso faz com que alguns bots do git-hub como o renovate que 
fica tentando atualizar  as dependencias do projeto, esses bots também rodam os testes 
da aplicação para verificar se a aplicação continua funcionando sem problemas após  
atualizar as dependencias, se não houver problemas ele cria um pull request dando uma 
sugestão que podemos atualizar as dependencias.

- Para fazer isso eu crio um arquivo .npmrc e dentro desse arquivo eu passo 
save-exact=true é importante fazer isso logo no inicio antes de instalar as 
libs e coisas do projeto.

# Carregando veriáveis de ambiente 

- Foi criado um arquivo .env para configurar as variaveis de ambientes, também foi 
criando um arquivo .env.example que serve para quando outra pessoa baixar o projeto 
ela saber quais variaveis de ambiente serão necessárias serem criadas, isso por que 
o arquivo .env é adicionado ao gitIgnore.

- Para carregar essas variáveis de ambiente é necessário instalar uma biblioteca 
chamada dotenv rodando o comando   npm i dotenv  essa biblioteca vai carregar 
esse arquivo .env e vai transformar em variaveis de ambiente dentro do node.js. 

npm i dotenv

- Para fazer esse carregamento dessas variaveis de ambiente de uma forma mais organizada 
foi criada uma pasta chamada env com um arquivo index.ts dentro desse arquivo eu carrego 
as variaveis de ambiente importando o dotenv/config, em seguida dentro desse arquivo 
foi feita a validação dessas variaveis de ambiente, para fazer essa validação também
foi utilizado uma lib que é o zod para isso precisa ser instalado rodando o comando 

npm i zod 

- Deve ser importando o z de dentro de zod para poder fazer todas as validações 
necessárias, em seguida foi criada uma variavel envSchema que recebe z.object 
isso por que quando o dotenv carrega as variaveis de ambiente todos os valores 
vem dentro de um objeto, inicialmente eu validei a variavel NODE_ENV e essa variavel
precisa ser um entre algumas outras opções e por isso foi utilizado um enum e dentro 
desse anum foi passado as opções validas para esse NODE_ENV, Foi passado um default 
com um valor caso essa variavel não seja passada.

import "dotenv/config";
import { z } from "zod";

const envSchema = z.object({
    NODE_ENV: z.enum(['dev', 'test', 'production']).default('dev'),
})

- Outra variavel ambiente comum é a PORT que é a porta do servidor alguns serviços
de hospedagem setam essa variavel de forma automatizada por isso essa variavel é 
necessária para que a aplicação seja informada qual porta será utilizada na hora de 
subir o servidor, nessa variavel utilizamos o coerce que pega algum dado de qualquer 
formato e converte para um formato escolhido coerce.number() , também posso utilizar 
um dafault passando algum valor padrão caso a porta não seja encontrada.

const envSchema = z.object({
    NODE_ENV: z.enum(['dev', 'test', 'production']).default('dev'),
    PORT: z.coerce.number().default(3333),
})

- Depois de criado a envSchema é feita a validação em si , foi criado uma const 
chamada _env passando o envSchema e para executar a validação é utilizado o 
metodo safeParse passando o local onde ficam as variaveis de ambiente que é o 
process.env

const _env = envSchema.safeParse(process.env)

- Para verificar se a validação deu certo ou não eu crio um teste se _env não teve 
sucesso por isso utilizo o success se isso for falso quer dizer que houve algum erro 
e posso retornar um consol.error com uma mensagem de erro, em seguida é passado o 
_env com error passando format que é um metodo que vaipegar todos os erros que ocorreram 
no processo de validação e formatar eles de uma forma mais simples , depois foi passado 
um trow new Error isso por que toda vez que um código entra em um trow nenhum código 
seguinte será mais executado. Isso faz com que a plicação não execute caso der algum 
erro nas variaveis de ambiente.

if (_env.success == false) {
    console.error('Invalid environment variables', _env.error.format())

    throw new Error('Invalid environment variables.')
}

- Caso a aplicação passe nesse if e seja um sucesso exportamos uma variavel env 
que recebe _env setando os dados data das variaveis de ambiente.

export const env = _env.data

- Em seguida eu vou no server e utilizo a variavel PORT e rodo o servidor para ver 
se está funcionando. PS - é necessário importar o env 

import { env } from "./env";

app.listen({
    host: '0.0.0.0',
    port: env.PORT,
}).then(() => {
    console.log('HTTP Server Running!')
})

# Criando eliases de importação 

- Quando o projeto tem muitas pastas e fazemos importações elas podem ficar muito 
longas dificultando alguém ler esse código, uma forma de melhorar isso é ir dentro 
do arquivo tsconfig.json e configuramos uma opção chamada "baseUrl": "./" ele indica 
qual o diretório rayz da aplicação , eu informo também o "paths": {},  dentro dele 
eu defino o seguinte ==> quando uma importação que começa com @/ for digitada 
tudo que vier depois da barra o vsCode vai entender como se tivesse sendo feita uma
importação em ./src/* ou seja toda vez que for importado em qualquer arquivo da 
aplicação e ela começar com @/ tudo que for digitado após vai ser tratado como se 
tivesse sendo feita uma importação direta da pasta src ==>  "@/*": ["./src/*"] 

 "baseUrl": "./",                                 
    "paths": {
      "@/*": ["./src/*"]
    }, 

# Fundamentos do Prisma ORM

## ORM (Object Relational Mapping)

- A partir de agora trabalhamos com banco de dados e ao invés de utilizar um query 
builder como o knex utilizamos um ORM que são ferramentas que trazem um alto nível de 
abstração, na parte de trabalhar com banco de dados, ou seja traz um leque de ferramentas
maior para se trabalhar com dba, a idéia do ORM é mapear as tabelas do banco de dados 
para estruturas de dentro do código que se consiga representar essas tabelas como classes

- O Prisma é o melhor ORM por que ele diminui bastante o trabalho e principalmente a 
parte de duplicidade quando se trabalha com banco de dados e também tem uma integração 
muito boa com o TypeScript, outro recurso muito legal é que ele faz toda a parte de 
migrations de forma automatizada.

- Para iniciar o Prisma deve ser instalado rodando o comando abaixo como dependencia de
desenvolvimento.

    npm i prisma -D 

- Executando npx prisma -h eu tenho acesso a uma série de comandos que posso utilizar 
com o Prisma para fazer operações comuns no db 

- O comando abaixo vai inicializar a parte de banco de dados da aplicação, parte de 
conexão com o banco de dados.

    npx prisma init

- Note que após inicializar ele criou uma pasta prisma com um arquivo schema.prisma 
é importante instalar a extenção do Prisma, é importante também no editor 
settings.json colocar o comando abaixo para facilitar a formatação do arquivo do 
Schema do prisma.

## Prisma: Configurando extensão no VSCode

1. Instale a extensão Prisma no seu Visual Studio Code.
2. Abra a Paleta de Comandos: Se estiver no Windows ou Linux: CTRL + SHIFT + P
3. Abra as configurações em JSON buscando por: 
  - Se o seu VSCode estiver em português: Abrir as Configurações do usuário (JSON)
  - Se o seu VSCode estiver em inglês: Open User Settings (JSON)
4. Adicione dentro do JSON o código abaixo:

"[prisma]": {
  "editor.defaultFormatter": "Prisma.prisma",
  "editor.formatOnSave": true
},

- O Schema dentro do prisma é a representação das tabelas que temos no banco de dados 
o prisma suporta vários banco de dados e ele suporta esse conceito de tabela de uma 
forma diferente, por isso o prima ele na cria tabelas mas cria model que é uma estrutura
onde vai ser armazenado os dados, no model damos a ele um nome sempre iniciando em 
maiúsculas e nunca no plural, por ex queremos criar uma tabela para usuário então 
criamos um model User, mas caso seja necessário trocar por exemplo depois o nome da 
tabela , temos recuusus como o @@mao("users") que por exemplo troco o nome por users. 

- Em seguida eu crio as coluna da minha tabela como id por exemplo, note que se eu errar 
algo como colocar string ao invés de String o próprio prisma vai me corrigir.
Note que ao utilizar o @ ele vai trazer algumas configurações a nível de coluna quando
tiver criando as colunas da tabela, quando se coloca dois @@ vai está sendo configurada 
a tabela, o @id que é obrigatório em cada tabela que é a chave primária, o @default 
pode ser passado 3 opções uuid() que segue a especificação do uuid que vai gerar um id 
longo e universalmente único, o cuid() tem a mesma idéia porém é um pouco mais curto.

# UUID()

- Na maioria das vezes se utiliza para transformar rotas com ids que ficam publicamente
expostos isso dificulta alguém ficar passando ids aleatórios e identificar usuários.

- Abaixo foi criado o promeiro model:  

model User {
  id    String @id @default(uuid())
  name  String
  email String @unique

  @@map("users")
}

- Note que como ainda não foi feita a conexão e nem criado o banco de dados esse 
comando não funciona é só um exemplo.

## É necessário criar a tipagem do Schema 

- Esse comando abaixo cria de forma automatizada a tipagem desse Schema, essa tipagem 
é a integração do typescript para que o código entenda que exista a tabela criada 
e que ela possue seus campos.

        npx prisma generate  

- Uma forma de testar se funcionou é dentro de node_modules procurar a pasta .prisma 
em seguida a pasta client abrir o arquivo index.d.ts e note que dentro desse arquivo o 
model que foi criado vai estar lá.

- Em seguida é necessário também instalar o client do prisma rodando o comando abaixo:
    npm i @prisma/client 

- Note que essa dependencia a cima é instalada de produção pois será usada para acessar 
o banco de dados. 

## Testando 

- Dentro de app.ts só para validar se está funcionando eu importei o PrismaClient 
eu eu crio uma conexão com o banco de dados estanciando esse PrismaClient e quando 
eu passo prisma. ele já vai trazer user que é a tabela que criamos quando eu utilizo 
um ponto no user ele tras todos os metodos que podem ser executados nesse model, eu 
fiz um teste com um create. 

import fastify from "fastify";
import { PrismaClient } from "@prisma/client";

export const app = fastify()

const prisma = new PrismaClient()

prisma.user.create({
    data: {
        name: 'Danilo Amaral',
        email: 'danilo123amaral.com.br',
    },
})

# Fundamentos do Docker 

https://www.docker.com

- Umas das ferramentas muito utilizadas e indispensável principalmente no back end 
principalmente também na parte de Deploy da aplicação, utilizamos para desenvolvimento 
da aplicação para instalar banco de dados na maquina sem precisar instalar na própria 
maquina, é mais utilizado ainda em ambiente de produção por que o Docker facilita 
bastante o deploy. 

- Umas das idéias de se utilizar o Docker é conseguir por exemplo subir um banco de 
dados para a aplicação, o banco esteja rodando na máquina mas a qualquer momento poder
excluir de forma facil aquele banco com todos os dados, arquivos de instalação, residuos
nada fica na maquina, fica tudo em Containers Virtuais.

# PostgreSQL com Docker 

link para instalação do Docker:  https://docs.docker.com/get-docker/

## Dockerhub 

- O dockerHub é um repositório de imagens do Docker  link:  https://hub.docker.com
- Imagens no docker são como receitas, são scripts que são preconfigurados para executar
coisas como banco de dados por exemplo, essas imagens são serviços que precisam ser 
executados para colocar na maquina, rodamos essas imagens no docker.

- nessa aplicação utilizamos uma imagem da bitnami/postgresql, foi optado por essa da 
bitnami por que ela ja vem trabalhada com uma parte de segurança.

- Aqui vamos rodar o postgres na máquina, no terminal e executamos o comando abaixo 
        docker run --name nome-que-quero-dar-a-imagem imagem-que-vai-ser-utilizada

- exemplo --> docker run --name postgresql bitnami/postgresql:latest 
PS- o latest é opcional

- Existe uma forma demelhorar a criação dessa imagem, podemos adicionar algumas 
variveis de ambiente para facilitar ainda mais nosso trabalho de conexão do banco 
de dados com a api, se olhar na documentação da imagem podemos ver algumas variaveis de
ambiente que podemos passar no momento de criação da imagem. utilizamos o prefixo 
-e para passar essas variaveis, inclusive já dá até para criar também um banco de dados
também.

- Podemos passar o -p para direcionar a porta do postgres que está rodando seja 
direcionada para a porta do host do navegador com isso quando for acessar a mesma 
porta no navegador vai está acessando a mesma porta dentro do container.

    docker run --name api-solid-pg -e POSTGRESQL_USERNAME=docker -e POSTGRESQL_PASSWORD=docker -e POSTGRESQL_DATABASE=apisolid -p 5432:5432  bitnami/postgresql

- Após isso ele vai subir o banco de dados criado ao container do docker.

## Alguns comandos

- O comando docker ps retorna todos os comando que estão rodando

- O comando docer ps -a ele vai mostrar todos os containers que foram criados em 
algum momento

## Rodando o container 

- Para rodar o comando basta utilizar o comando start passando o id ou o nome do 
container que criamos 

  docker start nome-do-container 

- Para parar o container basta rodar o comando stop 
    docker stop nome-do-container 

## Deletando um Container

- Para deletar um container basta rodar o comano rm 
    docker rm nome-do-container

## vendo logs do container

- docker logs nome-do-container

- para manter os logs e ficar monitorando ==>  docker logs nome-do-container -f 

## Validando o acesso ao banco dentro do container 

- Dentro do arquivo de .env na variavel de DATABASE_URL colocamod os dados de 
acesso ao banco 

- subistituir esses dados abaixo: 
DATABASE_URL="postgresql://johndoe:randompassword@localhost:5432/mydb?schema=public"

- por esses abaixo: 
DATABASE_URL="postgresql://docker:docker@localhost:5432/apisolid?schema=public"

- Apenas subistiuimos os dados de login e senha e nome do banco pelo do banco do 
nosso container.
- lembre que para conectar o container deve estar rodando.

- Para testar essa conexão no terminal eu rodei o seguinte comando abaixo: 
    npx prisma migrate dev 

- Esse comando vai no schema.prisma e vai pegar todas as tabelas que estão dentro desse
arquivo ou campos ou quellquer tipo de alteração que tenha sido feita nesse arquivo 
que ainda não foi refletido no banco de dados, quando rodamos esse comando ele vai 
identificar todas as alterções e vai pedir um nome para essa migration nessa descrição 
deve ser colocado tudo que foi feito desde a ultima vez que esse comando foi rodado. 

- Note que após rodar esse comando ele criou uma pasta dentr de prisma chamada 
migrations, dentro dessa pasta ele criou outra pasta que tem a timestamp atual 
que é quando essa migration foi executada, o nome da migration, e dentro dessa 
pasta um arquivo chamado migration.sql que é o sql que cria a alteração que 
foi feita na migration.

## Prisma Studio 
 - É um recurso do prisma que gera uma interface no navegador onde conseguimos 
 visualizar as tabelas do nosso banco de dados. 

 - Para utilizar basta rodar o comando abaixo 
    npx prisma studio

# utilizando o Docker Compose

- O Docker Composer é um arquivo criado na raiz do projeto, esse arquivo utiliza a 
extensão .yml, o arquivo docker-compose.yml esse arquivo vai ditar quais são todos
os Containers que a aplicação precisa criar para que ela funcione, edentro desse 
arquivo é necessário informar a versão da sitaxe do compose vai ser utilizada, em 
seguida os serviços que são os nomes dos containers, também deve se indicar qual 
imagem o serviço está utilizando, em seguida as portas, em seguida se coloca 
environment que são as variaveis de ambientes utilizadas.

version: '3'

services:
  api-solid-pg:
    image: bitnami/postgresql
    ports:
      - 5432:5432
    environment: 
      - POSTGRESQL_USERNAME=docker
      - POSTGRESQL_PASSWORD=docker
      - POSTGRESQL_DATABASE=apisolid

- Com isso quando for necessário baixar o projeto de um repositorio remoto por 
exemplo basta rodar o composer para ele criar o banco com o docker com o seguinte 
comando abaixo:

        docker compose up -d   ou   docker compose up

- Esse comando vai rodar todod os containers que estão dentro desse arquivo do composer de uma vez e para parar de rodar basta executar o comando abaixo que 
ele também para de rodar todos os containers de uma única vez. Esse comando abaixo
não só para de executar como também vai deletar todos os containers.

    docker compose down 

- Para apenas parar de executar todos os containers sem deletar basta rodar o 
comando abaixo: 

    docker compose stop 

