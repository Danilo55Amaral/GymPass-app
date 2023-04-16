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

# Criando schema do Prisma

- Agora comto todo o ambiente de banco de dados pronto, criamos as tabelas 
restantes. 

- Dentro do arquivo de schema.prisma vamos criar de acordo com os nossos 
requisitos da aplicação, foi ciado mais dois models o para academia Gym e 
o checkIn ambos possuem id como uma string, chave primaria e o default uuid
em seguida eu criei os outros campos dessas tabelas.

- Dentro das tabelas os campos de latitude e longitude são colocados 
por que quando o usuário faz login na academia será necessário saber onde 
ele está localizado no mapa. Isso por que um sistema de controle de chekIn em 
academia possuem um controle bem rigoroso para que uma pessoa só consiga afetuar 
o ckeckIn se ela estiver próxima da academia.

- O @@map é utilizado para renomear as tabelas do banco.

- O campo created_at é do tipo DateTime seu valor default é now() por que
quando é um campo de data e é um campo onde será armazenado o CheckIn o now 
armazena o valor atual daquele registro.

- Se queremos saber se um chekIn ta validado ou não, podemos usar um boleano mas
uma estratégia interessante seria trocar esse booleano por um DateTime de forma
opcional com isso além de saber se está validado ou não da para saber também 
quando que ele foi validado, isso foi feito no campo validated_at 

model CheckIn {
  id  String @id @default(uuid())
  created_at DateTime @default(now())
  validated_at DateTime?

  @@map("check_ins")
}

model Gym {
  id String @id @default(uuid())
  title String
  description String? 
  phone String?
  latitude Decimal 
  longitude Decimal 

  @@map("gyms")
}

## Campo de senha do usuário Encriptado

- Dentro da tabela de usuários criamos um campo password porém esse campo precisa
ser encriptado por que caso alguém tenha acesso ao banco ou vaze a senha está 
encriptado.Esse campo foi chamado de password_hash por que ele vai gerar uma 
combinação de caracteres gerada a partir da senha. 

- Um fator importante é que aqui não utilizamos um algoritimo de criptografia 
por que a criptografia na maior parte das vezes vai e volta, nesse caso aqui 
utilizamos uma estratégia de Hashing que é diferente da criptografia.

## Hashing  
- No Hashing apenas se cria para ir não tem como voltar , desciptografar o dado
a partir do momento em que o Hashing foi gerado não tem mais como ele voltar ao 
estado original para saber qual a senha original que gerou esse hashing.

## Rodando as alterações no banco de dados

- Para rodar a migrate utilizamos o código abaixo esse comando ler o arquivo 
schema.prisma comparando com o banco de dados que está rodando, e ver quais 
alterações feitas no arquivo que ainda não estão atualizadas no banco de dados
utilizamos o dev por que estamos em ambiente de desenvolvimento mas quando estiver 
em ambiente de ptodução utilizamos o deploy. 

        npx prisma migrate dev 
        npx prisma migrate deploy

- PS - Lembre que para executar esse código o banco tem que está rodando ou seja 
rodando o container do Docker que contém o banco.

-Após rodar o comando ele vai perguntar quais alterações foram feitas.

- Ele deu um bug quando o comando foi executado por que o campo password_hash 
String está como obrigatorio porém já existem dados inseridos no banco, para
resolver esse bug basta colocar esse campo como opcional ou adicionar um @default
com uma string vazia.

- Após rodar o comando ele já cria a migrate dentro da pasta migrations já com 
o sql que cria todas as alterações.

## Testando 

- Basta rodar um  npx prisma studio 
- Ele vai abrir uma interface no navegador e podemos checar se as tabelas 
foram criadas no banco.

# Relacionamento entre tabelas

## Tipos de relacionamentos 

- 1-1 ==> Um dado de uma tebela vai se relacionar somente com um dado de uma 
outra tabela, geralmente se cria isso para separar os dados em duas tabelas.

- 1-N ==> Uma informação que está armazenada em uma tabela pode está relacionada
com vários registros que estão armazenados em outra tabela.

- N-n ==>  Um registro em uma tabela que pode está relacionado com vários 
registros de outra e um registro dessa outra tabela pode está relacionado com 
vários registros da tebala inicial.

## Chaves estrangeiras 

- Aqui criamos as chaves estrangeiras do projeto, o checkIn sempre vai está 
associado com usuário é necessário ter um user_id na tabela checkIn com o 
tipo de campo do id de user isso por que o id do usuário vai ficar armazenado 
dentro do user_id, a mesma coisa vai ser com a tabela de Gym isso por que o 
usuário faz um checkin em uma academia por isso é necessário também ter um  
gym_id que será o id da academia.

model CheckIn {
  id           String    @id @default(uuid())
  created_at   DateTime  @default(now())
  validated_at DateTime?

  user_id String
  gym_id  String

  @@map("check_ins")
}

- Se for feito apenas dessa maneira o prisma não vai entender que esses campos 
se tratam de chaves estrangeiras e por isso é necessário utilizar o @relation, 
com a extensão do prisma no vscode ele facilita muito a criação de 
relacionamentos, eu vou dize que o chekIn está relacionado com o user que é do 
tipo User isso nada mais é que um nome que damos ao relacionamento ==> user

user User 

- como eu estou utilizando a extenção do prisma no momento em que salvo esse 
código ele vai gerar automaticamente o código de relacionamento.

  user   User   @relation(fields: [userId], references: [id])
  userId String

- Note que ele já cria o relacionamento ele utilizao @relation e ele está falando 
para criar uma chave estrangeira que referencia a coluna id dentro de User com 
o campo userId deste checkIn eu posso substtuir a sintaxe do userId por user_id

- Ele também já adiciona na tabela de usuarios que ele já pode ter varios checkIns 
note que ele traz um array por que pode ter varios checkIns.

     CheckIns       CheckIn[] 

- Eu fiz a mesma coisa para a academa gym Gym salvando ele também já criou o 
código do relacionamento.

- Após salvaro código e criar todos os relacionamentos rodamos a migrate para executar o código no banco. 
      npx prisma migrate dev

- Após rodar o código e fazer as alterações no banco de dados eu posso abrir 
o prisma studio e ver que ele vai ter a nova coluna user_id, gym_id além disso 
o prisma vai mostrar também colunas com os nomes dos relacionamentos que foram
feitos com aqui o user e o gym, se for na tabela de usuários note que também tem
uma coluna que da para ver os checkIns dele. 

- Em seguida foram feitos testes cadastrando usuarios e academias e fazendo 
checkins o prisma studio ajuda bastante nesses testes.

# Caso de uso e design patterns 

# Criação de um usuário 

- Aqui criamos rota de criação de cadastro de um novo uauário, utilizando alguns 
design patterns e alguns principios do SOLID, criamos uma rota bem simple de 
criação de um usuário dentro do back end.

- Dentro do meu app.ts eu criei uma rota post na qual dei o nome de users 
passando uma função que recebe request e replay, dentro dessa função eu vou 
cadastrar um usuario,  estamos utilizando o zod e vamos usar para fazer a 
validação desse usuário, isso para que o usuário não envie dados errados a 
partir dessa rota, com o zod eu passo como se fosse uma tipagem que valida esses 
dados que vou colocar, no email também existe uma validação do zod para email e 
posso adicionar também.

- Em seguida eu pegoesses dados e utilizo um parse para fazer um request.body
para fazer a validação, o parse vai dar um throw automatico em caso de erro 
nehum código posterior vai ser executado, caso a validação falhe.

- Logo abaixo eu pego esses dados e faço a criação de um usuário, foi utilizado 
um await, antes eu separei a conexão do banco criando uma pasta separada dentro de 
src chamada lib com um arquivo chamado prisma.ts nesse arquivo eu exporto uma 
constante chamada prisma que é a conexão com o banco eu devo passar o PrismaClient 
e devo importa-lo. 

- De volta a minha rota eu utilizo no await esse prisma que acabei de criar dentro 
da pasta lib, devo importa-lo, note que quando utilizo um ponto e dou um crtl + 
espaço ele já encontra todas as tabelas domeu banco de dados, passei ta tabela 
user e um objeto create passando os dados data. 

- Como foi utilizado um await é necessário fazer que esse metodo seja async 
no final é retornado um status 201 com uma send resposta vazia.

app.post('/users', async (request, reply) => {
    const registerBodySchema = z.object({
        name: z.string(),
        email: z.string().email(),
        password: z.string().min(6),
    })

    const { name, email, password } = registerBodySchema.parse(request.body)

    await prisma.user.create({
        data: {
            name,
            email,
            password_hash: password,
        },
    })

    return reply.status(201).send()
})

 - Voutestar rodando a aplicação com o comando npm run start:dev

 - Com a aplicação rodando podemos testar essa rota no insomnia, dentro foi 
 criado uma New Request Collection para testar as rotas desse projeto, chamei 
 de Ignite API SOLID Node.js , eu crio uma nova http request vou chamar de 
 Create user, do tipo POST, Vou passar a minha url da rota, selecionar para 
 enviar um JSON, em seguida eu escrevo um Json com os dados da minha rota e 
 testo.

 - Após o teste posso checar também na base de dados se o novo usuário foi 
 inserido na base de dados.

 - Uma coisa interessante é que na configuração de conexão do prisma podemos 
 passar um objeto de configurações e um desses objetos de configurações é o log
 dentro desse objeto da para passar vários tipos de logs, aqui foi colocado 
 o query esse log faz com que quando um usuário for criado ele vai mostrar no 
 log da aplicação as queries que foram geradas. Essas Queryes o prisma faz de 
 forma automatizada.

 - Para habilitar esse log apenas em ambiente de desenvolvimento eu vou utilizar 
 o env passando que se o NODE_ENV for dev então p log vai ser query se não vai ser 
 vazio.

export const prisma = new PrismaClient({
    log: env.NODE_ENV == 'dev' ? ['query'] : [], 
})

# Controller de registro

- É interessante separar toda estrutura de código existente em cada rota da 
aplicação, que está no arquivo app.ts, cada parte que a aplicação toca no momento 
em que ela faz uma operação seja banco de dados,validação, requisição e resposta 
ou qualquer outra coisa, tudo isso é interessante separar em mais arquivos, aqui 
separamos de uma forma que a manuntenção desse código seja beneficiada.

## Controller 

- vem de MVC - Model / View / Controller 

- O Controller é o nome dado para a função que lida com a entrada de dados 
de uma requisição HTTP e devolve uma resposta de alguma forma. 

- Geralmente o Controller está associado a alguns desses framework como Fastify,
express, Nest e note que essas ferramentas são Contrllers. 

- Foi pego a função de dentro de app que é o Controller que é a função que recebe 
a requisição e devolve uma resposta, essa função foi movida para outra pasta, 
dentro de src foi criada uma pasta chamada http e tudo que for relacionada a 
requisição, resposta e toda a parte http da aplicação vai ficar nessa pasta, 
dentro foi criada uma outra pasta chamada controllers e dentro foi criado o 
arquivo do primeiro controller onde foi colocada essa função que foi copiada de app
esse arquivo foi chamado de register.ts e dentro desse arquivo eu exporto essa
função, como está em outro arquivo é necessário fazer algumas importações na parte 
de request e reply como esse arquivo não recohece esses dados deve-se importar o 
FastifyRequest e o FastifyReply e passar como tipagem.

import { prisma } from "@/lib/prisma";
import { FastifyRequest, FastifyReply } from "fastify";
import { z } from "zod";

export async function register(request: FastifyRequest, reply: FastifyReply) {
    const registerBodySchema = z.object({
        name: z.string(),
        email: z.string().email(),
        password: z.string().min(6),
    })

    const { name, email, password } = registerBodySchema.parse(request.body)

    await prisma.user.create({
        data: {
            name,
            email,
            password_hash: password,
        },
    })

    return reply.status(201).send()
}

- No arquivo de app no local onde estava a função podemos importar o register como
segundo parametro para post.  

- As rotas da aplicação estão todas dentro do arquivo app e isso não é o ideal 
por que a aplicação pode crescer bastante e atingir 200, 300 rotas por isso 
é interessante separar isso também, dentro de http foicriado um arquivo chamado 
routes.ts e movemos para la a rota da aplicação. Esse routes.ts vai funcionar como 
um pluguin do Fastify e por isso ele precisa ser uma função que chamei de appRoutes
essa função recebe o app do Fastify e seu tipo é FastifyInstance e dentro dessa 
função eu colo minhas rotas.

import { FastifyInstance } from "fastify";
import { register } from "./controllers/register";

export async function appRoutes(app: FastifyInstance) {
    app.post('/users', register)
}

- Dentro de app eu chamo o register e dentro passo o appRoutes.

import fastify from "fastify";
import { appRoutes } from "./http/routes";

export const app = fastify()

app.register(appRoutes)

- Após salvar eu reinicio o servidor e podemos testar criando um novo usuário no 
insonmia.

- Foi feita também a separação de código de dentro do controller que criamos no 
arquivo register.ts, isso por que temos que enxergar a aplicação em camadas 

# Hash da senha e validação 

## BcryptJs

- Foi instalado o Bcrypt rodando o seguinte comando   npm i bcryptjs  

- Como essa biblioteca não é escrita em TypeScript énecessário instalar esse
pacote separadamente com o comando    npm i -D @types/bcryptjs

- Esse hash vai gerar um valor que é impossivel de reverter ao formato original 
esse dado preciso ser gerado com base em alguma coisa que fica no segundo paramtro
do metodo.

- Essa é a biblioteca mais comum no node para fazer hash de senhas aqui 
dentro do arquivo de register.ts foi importado de dentro de bcryptjs o 
modulo hash , em seguida foicriado o password_hash de forma separada 
utilizando o metodo has que foi importado do bcrypt esse metodo retorna 
uma promise e por isso podemos utiliza um await para aguardar esse metodo
o primeiro parametro que é enviado para o hash é a senha de usuario e o segundo 
paramtro pode ser o 'asd' que é algo unico e aleatorio e a senha pode ser gerada 
a partir dele,  ou pode ser passado um numero de rounds isso pode ser entendido
como o número de vezes que vai ser gerado o hash, cada round que é colocado a mais
fica mais dificil para que esse hash seja quebrado ou descoberto, porém para casa
round que for colocado a mais ficará mais pesado para a aplicação gerar o hash
dessa senha.

import { hash } from "bcryptjs";

const password_hash = await hash(password, 6)

- PS - Se esse hash é usado para alguma coisa que vai acotnecer muitas vezes ou 
seja muitas requisições não podemos colocar um número alto de rounds no hash como
6 por exemplos por que vai pesar na aplicação, vai gastar muito processamento.
na maioria das aplicações web 6 rounds é utilizado.

- Podemos testar se nosso algoritimo de criptografia ta funcionando criando um 
novo usuario no isomnia e depois checando o campo de senha na base de dados e ver
se foi gerado hash da senha.

- Esse hash não pode ser descriptografado isso não é possivel, para checar se o 
hash é valido, quando o usuário fazer login com a senha essa senha vai ser pega e 
vai gerar um hash também e vai ser comparado se esse hash gerado é o mesmo que 
está salvo no banco de dados. Ou seja não é possivel fazer o caminho inverso e 
descriptografar mas é possivel durante o login gerar o hash novamente e ver se
eles batem.

## Resolvendo problema de email duplicado

- Note que se for cadastrado um usuario com o email que já existe vai da um erro 
500, para resolver isso é bom fazer uma checagem para ver se tem um usuario com o 
mesmo email, para isso eu criei uma contante userWithSameEmail e utilizei o metodo
findUnique() do proprio prisma, esse metodo procura um registro unico na tabela 
ai podemos passar um where para filtrar qual campo quer procurar o registro 
é importante saber que esse metodo só consegue buscar registros em campos que 
são ou unique ou chaves primarias, sem seguida eu crio um if estabelecendo que
se já exixtir um usuario com o mesmo email eu posso retornar um status 409 que
é para conflito quando se tem dados duplicados.

const userWithSameEmail = await prisma.user.findUnique({
        where: {
            email,
        },
    })

    if (userWithSameEmail) {
        return reply.status(409).send()
    }

# Caso de uso de registro

- Aqui é separada do controler a parte que é igual independente de que ela 
vai ser executada por que ela pode ser usada em outras coisas futuramente ela  
pode ser separada deixando o código mais limpo e de fácil manutenção. 

- Para isso foi criada uma pasta chamada use-cases que é para casos de usos 
dentro dessa pasta foi criado um arquivo chamado register.ts dentre desse arquivo 
foi criada uma função e foipassada a parte que queremos separar do outro arquivo 
de register do controller. É importante fazer as importações necessárias e 
transformar essa função em uma função async. 

- Também foi necessário criar uma interface para receber alguns dados como name, 
email e password.

import { prisma } from "@/lib/prisma";
import { hash } from "bcryptjs";

interface RegisterUseCaseRequest {
    name: string
    email: string
    password: string
}

export async function registerUseCase({
    name,
    email,
    password,
}: RegisterUseCaseRequest) {
    const password_hash = await hash(password, 6)

    const userWithSameEmail = await prisma.user.findUnique({
        where: {
            email,
        },
    })

    if (userWithSameEmail) {
        throw new Error('E-mail alread exists.')
    }

    await prisma.user.create({
        data: {
            name,
            email,
            password_hash,
        },
    })
}

-Não faz sentido utilizar aqui dentro o reply por que ele é algo especifico da 
parte HTTP por isso essa parte não faz sentido aqui dentro já que o objetivo foi 
separar essa parte do código, por isso vamos reescrever essa parte de outra forma 
utilizando um trow , porém dentro do Register na parte de contrller quando é chamado esse caso de uso da para colocar ele por um try catch e no catch do erro
colocar a parte do reply com o status 409.

  try {
        await registerUseCase({
            name,
            email,
            password,
        })
    } catch (err) {
        return reply.status(409).send()
    }

- O que foi feito aqui foi separar a parte lógica da fucionalidade em si da parte
que é especifica da camada de HTTP com isso em qualquer lugar da aplicação for 
necessário criar um usuário basta chamar o caso de uso que foi criado. 

- A motivação de ter separado o caso de uso foi para conseguir reaproveitar a 
lógica de criação do usuário entre várias funcionalidades da aplicação que 
precisam criar um usuário, todas elas vão chamar o mesmo caso de uso que lida com 
toda a parte comum entre todos os momentos em que for necessário criar um usuario 
na aplicação.

## Repository Pattern 

- Esse patern serve para abstrair a parte de conexão de requisições que são feitas
ao banco de dados, toda essa parte de comunicação com o banco de dados em um 
arquivo separado. 

- Foi criada uma pasta chamada repositories dentro dela foi criado umarquivo 
chamado prisma mais o nome da tabela do banco de dados que vou utilizar assim
prisma-users-repository.ts. 

- Dentro desse arquuivo eu criei e exportei uma classe e dentro dela vou ter
vários metodos que vão interceptar e vão ser as portas de entrada para qualquer
operação que for ser feita no banco de dados ou seja todas as operações do banco 
de dados sempre vão passar pelos respositorios. 

- Foi criado um metodo create que recebe data(alguns dados) e ele executa o código 
de criação de usuário que foi copiado dentro do register que cria o usuário no 
banco de dados, para tipar os dados desse metodo eu posso utilizar um recursodo 
proprio prisma e utilizar o Prisma.UserCreateInput, com isso dá também para passar
o data inteiro sem precisar especificar o name, email e o passowrd, isso pode ser
colocado em uma variavel e depois retornar essa variável de dentro e com isso o 
caso de uso que chamou esse metodo se ele quiser trabalhar com o usuário recem 
criado ele pode.

export class PrismaUsersRepository {
    async create(data: Prisma.UserCreateInput) {
        const user = await prisma.user.create({
            data,
        })

        return user
    }
}

- Após isso voltamos ao register e necessário criar variavel para instanciar 
essa classe, em seguida eu passo o metodo create para essa variavel com os dados 
de criação de usuário, esse metodo é uma promise e por isso utilizamos um await.

 const prismaUsersRepository = new PrismaUsersRepository()

    await prismaUsersRepository.create({
        name,
        email,
        password_hash,
    }) 

- Podemos testar se está funcionando criando um novo usuário no insomina e 
verificando na base de dados se o usuário foi criado.

# Inversão de dependências 

- Umas das motivações para utilizar repositorios é conseguir ter a possibilidade 
de conseguir migrar de ferramenta de banco de dados ou qualquer outra coisa 
que se esteja utilizando na aplicação.

## SOLID

- São 5 principios na programação que foi criado no Clean Code. 

## D - Dependency Inversion Principle 

- Esse principio cabe bem nessa situação que mexemos na aplicação que o caso de 
uso de registro de usuários ele possue uma dependencia no repositorio do Prisma 
o caso de uso depende do repositorio do Prisma se esse repositorio do Prisma 
deixar de existir o caso de uso para de funcionar por isso é uma dependencia. 

- No princípio da inversão de dependencia isso muda um pouco em como o caso de 
uso ou qualquer outro arquivo da aplicação tenha acesso a suas dependencias, esse
principio se inverte a ordem de como a dependencia vai chegar nesse caso de uso
ou seja ai invés do caso de uso estanciar as dependencias que ele precisa, ele 
vai receber essas dependencias como parametro. 

- Geralmente utilizamos classes para isso, foi criada uma classe RegisterUseCase
dentro do meu register.ts de use-cases e foi pego todo o metodo registerUseCase e 
foi colocado nessa classe também foi dado outro nome ao metodo.

- Caca classe de caso de uso vai ter apenas um unico metodo, a classe pode ter um 
constructor e dentro dele pode ser recebida as dependncias ou seja ao invés da 
classe estanciar as dependencias ela vai receber como parametro e por isso que 
é inversão de depencia.

- Dentro do constructor vai ser recebido o repository e por isso colocamos 
userRepository , quando queresmo que um parametro que estamos recebendo no 
constructor automaticamente vire uma propriedade da classe basta colocar 
private ou public, posso colocar por enquanto any como tipagem.

- Posso remover a parte que estou instanciando o repositorio. 

- E no meu metodo create eu vou receber o usersRepository.

export class RegisterUseCase {
    constructor(private usersRepository: any) {}
    async execute({
        name,
        email,
        password,
    }: RegisterUseCaseRequest) {
        const password_hash = await hash(password, 6)
    
        const userWithSameEmail = await prisma.user.findUnique({
            where: {
                email,
            },
        })
    
        if (userWithSameEmail) {
            throw new Error('E-mail alread exists.')
        }
    
        await this.usersRepository.create({
            name,
            email,
            password_hash,
        })
    }
}

- Essa classe deve ser exportada 

- Note que nessa classe agora ela não está mais associada ao prisma e agora o 
arquivo que precisar desse caso de uso no caso aqui é o register.ts do controller
Essa classe vai ser usada nesse arquivo e para isso ela precisa ser instanciada 
e é necessário passar como parametro quais são as depencias, aqui é pego o 
prismaUsersRepository  e é passado como parametro para o registerUseCase e 
utilizamos o metodo execute.

 try {
        const usersRepository= new PrismaUsersRepository()
        const registerUseCase = new RegisterUseCase(usersRepository)

        await registerUseCase.execute({
            name,
            email,
            password,
        })
    } catch (err) {
        return reply.status(409).send()
    }

    return reply.status(201).send()

# Interface do repositório

- Note que dentro do caso de uso no arquivo register a tiágem do userRepository 
ainda está como any, e não queremos utilzar tipagem do prisma nesse arquivo pois 
a ideia é ele ser indenpende, para resolver isso foi criado um arquivo de tipagem
dentro de repositories chamado users-repository.ts, dentro desse arquivo eu 
construir uma interface que vai dizer quais metodos vão existir no repositorio 
e quais parametros eles recebem, essa interface deve ser exportada.

import { Prisma, User } from "@prisma/client";

export interface UsersRepository {
    create(data: Prisma.UserCreateInput): Promise<User>
}

- Vontando ao caso de uso no arquivo de register eu vou utilizar essa interface 
para tipar o constructor.

constructor(private usersRepository: UsersRepository) {}

- Note que dentro do arquivo register do controller se o metodo create não for
implementado ele não dará nenhum erro porém esse metodo é necessário, por isso 
dentro do repositorio que queremos que siga essa interface devemos implementar 
por isso dentro do arquivo prisma-users-repository.ts eu implemento essa interface
também.

export class PrismaUsersRepository implements UsersRepository {
    async create(data: Prisma.UserCreateInput) {
        const user = await prisma.user.create({
            data,
        })

        return user
    }
}

- Dessa forma se o metodo create não for informado ele vai dar erro pedindo que 
seja implementado.

- A pasta repositories foi organizada e dentro dela foi criada uma pasta chamada 
prisma e todos os repositorios especificos do prisma foi jogado para essa pasta.

- Foi feito um novo teste no insomnia para ver se a aplicação continua 
cadastrando normalmente.

- Como não queremos que o caso de uso não tenha nenhuma referencia ao prisma e
seja completamente independente também movemos a parte do metodo de buscar um 
usuário pelo email lá para dentro do repositorio, para começar dentro da 
interface no arquivo users-repository.ts  eu crio um novo metodo dentro da 
interface chamado findByEmail que é para encontrar por Email, esse metodo 
recebe email e devolve através de uma Promise um usuário se ele não encontrar 
um usuário ele retorna nulo.

export interface UsersRepository {
    findByEmail(email: string): Promise<User | null>
    create(data: Prisma.UserCreateInput): Promise<User>
}

- Agora Voltando ao repositorio do prisma como ele está implementando a 
interface ele já vai gerar um erro pedindo para ser implementado o metodo 
que falta.  Eu apenas implemento esse metodo passando a parte do código que busca
o usuário por email que queremos separar do nosso caso de uso. Esse metodo vira 
async e eu passo essa parte do código que quero mover, posso renomear para user.

export class PrismaUsersRepository implements UsersRepository {
    async findByEmail(email: string) {
        const user = await prisma.user.findUnique({
            where: {
                email,
            },
        })

        return user
    }
    
    async create(data: Prisma.UserCreateInput) {
        const user = await prisma.user.create({
            data,
        })

        return user
    }
}

- Agora dentro do meu caso de uso eu apenas coloco para utilizar o metodo do 
repositorio.

const userWithSameEmail = await this.usersRepository.findByEmail(email)

- Agora note que dentro do caso de uso não tem mais nenhuma dependencia do 
prisma. Fizemos mais um teste no insomnia para checar se está funcionando 
normalmente.

# Lidando com erros do use case 

- Uma das coisas que podem ser melhoradas na aplicação é a parte de erros 
que são originados de dentro dos casos de usos, no momento só existe um throw
new Error passando uma mensagem de erro, dentro do controller existe apenas um
try catch por volta de todo o use case e para todo o tipo de erro está sendo 
retornado um status 409 . 

- Isso pode ser ruim no futuro quando a aplicação crescer por que é comum os 
casos de uso, serviços a funcionalidades da aplicação, terem vários tipos de 
condicionais, validações e possiveis erros que podem ocorrer e serem retornados
é interessante ter uma alguma forma de dentro do controller para fazer alguma 
tratativa para cada tipo de erro diferente conseguir ter uma resposta diferente 
um status code diferente para cada um dos erros.

- Dentro da pasta de use-cases criamos uma nova pasta chamada errors e dentro dela
foi criado um arquivo para cada tipo de erro que possa ter na aplicação. foi 
criado um arquivo para esse erro que já estamos tratando, o arquivo criado se 
chama user-already-exists-error.ts 

- Dentro desse arquivo eu exporto uma classe que chamei de UserAlreadyExistsError
essa classe extends a classe Error tradicional do JavaScript, dentro dessa classe
tem um constructor que chama um metodo super que é o metodo constructor da classe
error, nese metodo super podemos enviar uma mensagem.

export class UserAlreadyExistsError extends Error {
    constructor() {
        super('E-mail already exists.')
    }
}

- Com isso voltamos ao caso de uso e ao invés de da um trow new Error vamos 
utilizar o throw passando a classe que foi criada.

 if (userWithSameEmail) {
     throw new UserAlreadyExistsError()
}

- Agora dentro do controller na parte do catch fazemos a condição que se o err 
for uma instancia da classe UserAlreadyExistsError vai ser retornado o status 409
se não por enquanto podemos retornar um erro status 500.

if (err instanceof UserAlreadyExistsError) {
            return reply.status(409).send()
        }

        return reply.status(500).send()

- Após isso podemos testar no insomnia. 

- Com isso temos uma maneira de tratar cada tipo de erro dentro do controller 
e retornar um status diferente.

- Eu posso melhorar ainda mais passando dentro do send um message passando a 
mensagem.

 if (err instanceof UserAlreadyExistsError) {
            return reply.status(409).send({ message: err.message })
        }

# Handler de erros global

- Uma das coisas que ainda falta é a parte de lidar com outros tipos de erros 
na aplicação, como se acontecer algum tipo de erro dentro do caso de uso que 
não é um erro conhecido ou  um erro que não foi tratado, note que temos um 
status 500 para outros tipos de erro, porém não traz informações. 

- Dentro do arquivo de register.ts no controllers da aplicação a parte que 
retorna um status 500 vou substituir por um trow err note que com isso a 
aplicação melhora a tratativa desse erro com o fastify que já identifica
porém da para melhorar mais. 

try {
        const usersRepository= new PrismaUsersRepository()
        const registerUseCase = new RegisterUseCase(usersRepository)

        await registerUseCase.execute({
            name,
            email,
            password,
        })
    } catch (err) {
        if (err instanceof UserAlreadyExistsError) {
            return reply.status(409).send({ message: err.message })
        }

        throw err 
    }

    return reply.status(201).send()

- Dentro do arquivo app.ts na raiz da aplicação foi chamada uma função chamada 
setErrorHandler na qual recebe uma função essa função tem acesso ao error e 
também recebe a requisição request e a reply, em seguida foimontado um if que 
se o error for uma instancia de ZodError ele irá retornar um status 400 com um 
send passando uma message, e um issues passando error.format que é um metodo que 
o zod utiliza para formatar os erros.

app.setErrorHandler((error, _request, reply) => {
    if (error instanceof ZodError) {
        return reply
        .status(400)
        .send({ message: 'Validation error.', issues: error.format() })
    }

    return reply.status(500).send({ message: 'Internal server error.' })
})

- Foi montado um outro if que se o NODE_ENV for diferente de production ele vai 
dar um concole.error passando o error com isso quando der algum erro ele vai 
retornar no terminal  informações do erro.

app.setErrorHandler((error, _request, reply) => {
    if (error instanceof ZodError) {
        return reply
        .status(400)
        .send({ message: 'Validation error.', issues: error.format() })
    }

    if (env.NODE_ENV !== 'production') {
        console.error(error)
    }
    return reply.status(500).send({ message: 'Internal server error.' })
})

- Ps Como o request é um parametro que não está sendo utilizado colocamos ele 
com um _ para indicar isso.

# Design Patterns e Testes 

# Configurando o Vitest

- Testes dentro do back end não é algo opcional é necessário se trabalhar com 
testes desde o inicio, quando se está trabalhando nas regras de negocio em si
os testes são fundamentais. 

## Vitest 

- É a ferramenta que utilizamos para escrever os testes para instalar basta rodar 
o comando abaixo   

    npm i vitest -D 

- Podemos já instalar com um pluguim que vai ser utilizado com issopara instalar 
basta adicionar 

    npm i vitest vite-tsconfig-paths - D 

- Após instalar criamos na raiz do projeto um arquivo chamado vite.config.ts 
dentro desse arquivo é necessário exportar uma função como padrão chamada 
defineConfig e essa função importamos do vitest/config , essa função recebe 
parametros e aqui vamos utilizar o plugin, e nesse plugin foi passado o 
tsconfigPaths que importamos do vite-tsconfig-paths  isso é necessário 
para o Vitest entender todos os tipos de importações que fazemos aqui tipo com o
@ e por isso também instalamos o pluguin  junto ao Vitest.

import { defineConfig } from  "vitest/config";
import tsconfigPaths from "vite-tsconfig-paths"; 

export default defineConfig({
    plugins: [tsconfigPaths()],
})

## Criando Scripts para rodar os testes 

- Dentro do package.json criamos um script para rodar os testes, foi configurando
dois scripts "test" para rodar os testes de um única vez e "test:watch" para ficar
monitorando e rodando sempre que uma alteração for feita ele vai executar de 
forma automatica os testes.

    "test": "vitest run",
    "test:watch": "vitest"

- Para testar dentro da pasta de use-cases eu crio um arquivo chamado 
register.spec.ts que é o arquivo de teste dentre desse arquivo foi  importado
o test e chamamos para criar o primeiro teste, o nome do teste é chack if it 
works que é testando se está tudo funcionando. Nesse teste eu escrevi que 
espero que 2 + 2 seja igual a 4 

import {expect, test } from "vitest"

test('check if it works', () => {
    expect(2 + 2).toBe(4)
})

- Em seguida executamos o teste para ver se está funcionando. 

## Comando para rodar os testes
    npm run test 

## Comando para rodar os testes e ficar monitorando 
    npm run test:watch 







