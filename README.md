# GymPass style app

## RFs (Requisitos funcionais) 

- são Funcionalidades da aplicação o que é possivel o usuário fazer ma aplicação. 

- [ ] Deve ser possível se cadastrar;
- [ ] Deve ser possível se autenticar;
- [ ] Deve ser possível obter o perfil de um usuário logado;
- [ ] Deve ser possível obter o número de check-ins realizados pelo usuário logado;
- [ ] Deve ser possível o usuário obter seu histórico de check-ins;
- [ ] Deve ser possível o usuário buscar academias próximas;
- [ ] Deve ser possível o usuário buscar academias pelo nome;
- [ ] Deve ser possível o usuário realizar check-in em uma academia;
- [ ] Deve ser possível validar o check-in de um usuário;
- [ ] Deve ser possível cadastrar uma academia;


## RNs (Regras de negócio)

- São caminhos que cada requisito pode tomar, que condições serão aplicadas para cada RFs, sempre está associada ao RF.

- [ ] O usuário não deve poder se cadastrar com um e-mail duplicado;
- [ ] O usuário não pode fazer 2 check-ins no mesmo dia;
- [ ] O usuário não pode fazer check-in se não estiver perto (100m) da academia;
- [ ] O check-in só pode ser validado até 20 minutos após criado;
- [ ] O check-in só podeser validado por administradores;
- [ ] A academia só pode ser cadastrada por administradores;

## RNFs (Requisitos não-funcionais)

- São requisitos que não partem do cliente, o usuário não vai ter controle desses 
requisitos, são requisitos mais técnicos como qual banco de dados devo utilizar, 
quais estratégias serão utilizadas, tecnologias.

- [ ] A senha do usuário precisa estar criptografada;
- [ ] Os dados da aplicação precisam estar persistidos em um banco PostgresSQL;
- [ ] Todas listas de dados precisam estár paginadas com 20 itens por página;
- [ ] o usuário deve ser identificado por um JWT (JSON Web Token);