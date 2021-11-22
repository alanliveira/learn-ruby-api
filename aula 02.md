# Mesadão

Esse projeto tem como objetivo criar um sistema de receitas e despesas para o gerenciamento das mesadas. Nessa Fase fase o aluno irá aprender os seguintes tópicos.

- Uso de máquinas virtuais ou subsistemas linux
- instalação de programa pelo basj/terminal
- **Criação de api com rails**
- **Modelagem de dados**
- **Autentificação de usuários**
- **Trabalhando com as requisições**
- Testes de software unitários e requisoções
- Documentação de api com Postman ou Insomnia

## Instalação

> No window se você instalou o ruby pelo programa ruby installer, recomento que baixa versões com o dev kit superior a 2.3

- mostrar a instalação do mysql e config no db

## Criação de API com Rails

Uma API _application programming interface_ é um conjunto de rotinas e padrões que visa em facilitar a comunicação dos dados de varias aplicações no mundo da programação.

Imaginamos o discord, um aplicativo muito popular entre os alunos da supergeeks que utilizamos para comunicar na hora do jogo, ele está disponivel em várias plataformas (Android, IOS e WEB) e todos carregando as mesma conversas, mídia de sua conta. Mas como em plataformas diferentes o discord sabe quem é você e quais são as suas informações, sabendo que na WEB trabamos muito com HTML, CSS e JS, no IOS utilizamos a plataforma XCode com a linguagem swift e no Android utilizamos o Android studio com a linguagem de programação kotlin.

Pois ao que entra a API, ela será um servidor de internet que irá prover as informações para os aplicativos, é nele que estará armazenado os dados de sua aplicação des de imagens, conversas, vídeos e tb as call que você faz para jogar o amongus.

Felizmente podemos utilizar o rails para criar uma API e utolozar todo o profeito que o framework traz. Para isso vamos nos certificar que o ruby está instalado na máquina e se o rails está instalado na máquina.

```sh
ruby -v # ruby 3.0.2p107 (2021-07-07 revision 0db68f0233) [x86_64-linux]
rails -v # Rails 6.1.4.1
```

Após a verificação vamos criar um projeto rails utilizando o comando new utilizando a flag _--api_ e a principio vamos ignorar os teste com a flag _----skip-test_, vamos tambem utilizar o banco de dados mysql e para isso vamos definir com a flag _--database=mysql_

```sh
rails new mesadao --api --skip-test --database=mysql
```

> A fim de curiosidade, caso você tenha dúvida em que comando utilizar voce poderá utilizar o comando **rails new --help** para listar os comandos

## Modelando a api

A nossa API será um sistema que irá controlar receitas/despesas de nossas mesadas, para isso precisamos planejar os dados que iremos manupular,

A receita será um ganho que eu tive ao receber uma mesada, ou seja dinheiro que ganhamos por ser comportado, mas tambem pode ser adquirida por outros meios, como ganhos subs da twitch. Então será preciso que **descrevamos** de onde veio o **dinheiro** e que **dia** foi recebido, com essas informações podemos abstrair os dados e colocar em um banco de dados com os respectivos tipos:

**GAINS**

| campo       | tipo     | nulo  | chave |
| ----------- | -------- | ----- | :---: |
| id          | integer  | false |  PK   |
| description | string   | false |   -   |
| value       | float    | false |   -   |
| date        | date     | false |   -   |
| created_at  | datetime | false |   -   |
| updated_at  | datetime | false |   -   |

Para criar essa tabela vamos utilizar o rails para além de criar as tabelas, criar a model responsável pela manipulação da tabela:

```sh
rails g model Gain description:string value:float date:date
```

> Os campos ID, created_at e updated_ate serão criados pelo próprio rails

Após a criação vamos entender o que são as despesas, elas serão todo os gastos que terei com a minha mesada, por exemplo caso você queira comprar um skin no fortnite ou comprar um Playstation 5, então ele valor será a despesa e deve ser anotado em uma tabela a parte, a estrutura será similar a receitas mas a finalidade será diferente

**EXPENSES**

| campo       | tipo     | nulo  | chave |
| ----------- | -------- | ----- | :---: |
| id          | integer  | false |  PK   |
| description | string   | false |   -   |
| value       | float    | false |   -   |
| date        | date     | false |   -   |
| created_at  | datetime | false |   -   |
| updated_at  | datetime | false |   -   |

para criar a despesas vamos criar uma model no rails

```sh
rails g model Expense description:string value:float date:date
```

## Autentificação de usuários

Com as tabelas criadas vamos pensar que os nossos irmão queiram tambem utilizar o nosso sistema, porem do jeito que o nossa api está ela não será capaz de distinguir que receita e despesa e de cada um, causando um problema muito grande para nós.

Na fase passada trabalhamos com uma gem muito interessante chamada devise, que serve para criar um sistema de login em pouco passos, então vamos ir no arquivo **GemFile** pra adicionar essa gem

```rb
# file: GemFile
gem 'devise'
```

Depois de salvao o arquivo GemFile precisamos instalar no nosso sistema operacional.

```sh
bundle install
```

> Para verificar se a gem foi instalada execute o comando **rails generate** e procure pela sessão do devise

Após a sua instalação vamos instalar o devise em nossa API e criar um nível de acesso para usuários

```sh
rails g devise:install
rails g devise User
```

Agora com o devise instalado na nossa api precisamos fazer os relacionamentos do usuário com as tabelas receitas e despesas, Para isso iremos criar duas migrações, uma que irá fazer o relacionamento do usuário a receita e outra que irá fazer o relacionamento do usuário á despesas

```sh
rails g migration AddUserToGains user:references
rails g migration AddUserToExpenses user:references
```

Com todas as migrations prontas precisamos executa-las

```sh
rails db:migrate
```

Vamos fazer os relacionamentos das despesas/receitas com isso temos que pensar se nosso usuário vai criar mais de um receita ou despesas? nessa caso simm então vamos abrir a model app/models/user e colocar que ela será **has_many**, ou seja um usuário poderá fazer mais de uma receuta ou despesa

```rb
# file: app/models/user.rb

class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable

  has_many :expenses
  has_many :gains
end
```

Agora precisamos ir na **app/models/expense.rb** para colocar o relacionamento **belongs_to**, onde uma despesa pertence a um usuário

```rb
# file: app/models/expense.rb

class Expense < ApplicationRecord
  belongs_to :user
end
```

A mesma coisa na model **app/models/gain.rb**

```rb
# file: app/models/gain.rb

class Gain < ApplicationRecord
  belongs_to :user
end
```

## Trabalahndo com as requisições

Até agora trabalhamos muito na camada da model de nossa API, e como o objetivo dela é fazer a comunicação com outras plataformas, precisamos pensar em uma maneira de fazer essa comunicação, por sorte podemos utilizar o protocolo HTTP, muito utilizado na internet, para isso teremos que trabalhar com as rotas.

O HTTP para ser utilizado precisamos utilizar uma URL, as rotas e seus verbos. A

**URL**: _(Uniform Resource Locator)_ é o endereço da nossa página, por exemplo: **http://supergeeks.com.br**,, no nosso caso como estamos na nossa máquina e estamso desenvolvendo iremos utilizar o endereço **http://localhost:3000**, neste caso devemos especificar a porta de comunicação, pq o nossos sistema operacional não é destinado para servidores.

**Rotas**: Vem depois da URL e serve para navegar em diversos locais de api,exemplo **/sobre.html**

**verbos**: São as operações que a requisição ira fazer no servidor, por padrão todas as Páginas de internet utilizar o _GET_, que diz ao servidor que iremos pegar informações da página e existe mais de um verbo

| verbo  | descriçao                                       |
| ------ | ----------------------------------------------- |
| GET    | pega as informações no servidor                 |
| POST   | Cria um novo conteudo                           |
| PUT    | Altera todo o conteudo                          |
| PATCH  | Altera uma informação específica de um conteudo |
| DELETE | Apaga um coteudo                                |

No rails as rotas estão ligadas diretamente aos controller da nossa aplicação, então iremos precisar criar as controller para receitas e despesas. Mas e se no futuro eu queira fazer uma nova versão com mais recursos e padrões que existem no mercado de programação e os seus parentes não atualizarem nenhuma das plataforma que eles acessão? Isso com certeza vai querar tudo, então é uma boa prática nós colocar uma versão para cada rota que trabalhamos, para isso na pasta controller vamos criar uma outra pasta chamada API e dentro da pasta API vamos cruar a pasta v1, tambem podemos criar a pasta pelo terminal

```sh
mkdir -p app/controllers/api/v1
```

Com a pasta criada vamos criar as controllers de receitas e despesas

```sh
rails g controller api/v1/gains
rails g controller api/v1/expenses
```

Com as controller criadas, vamos abrir o arquivos de rotas que está em _config/routes.rb_ e precisamos fazer o mapeamento da rota e definir o modo de como acessamos a controller

```rb
# File: config/routes.rb

Rails.application.routes.draw do
  devise_for :users
  # For details on the DSL available within this file, see https://guides.rubyonrails.org/routing.html
  namespace :api do # faz a referência a pasta api dentro da controller
    namespace :v1 do # faz a referencia a pasta v1 dentro da api
      resources :gains # faz o mapeamento dos verbos das rotas de gains
      resources :expenses # faz o mapeamento dos verbos das rotas de expenses
      resources :users # faz o mapeamento dos verbos das rotas de users, nesse caso já mapeado pelo devise
    end
  end
end
```

Para verificar se tudo deu certo execute o comando abaixo e veja se tem rotas que começem com _/api/v1/\*_

```sh
rails routes
```

Com isso temos ja a nossa comunicação feita, mas ainda falta colocar as ações da requisição e melhorar a nossas rotas, coisa para a próxima aula
