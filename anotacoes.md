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

