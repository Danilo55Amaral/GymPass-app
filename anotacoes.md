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

