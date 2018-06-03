---
title: "CRUD SQLite no Ionic 3"
date: "2016-01-08"
tags: [ionic 3, sqlite]
---

Neste post, iremos implementar algumas funções básicas do SQLite no ionic 3.

##  Instalações necessitarias:

1. ionic cordova plugin add cordova-sqlite-storage
2. npm install --save @ionic-native/sqlite

## Preparar aplicativo:

No `app.module.ts` coloque estes import's:

```ts
import { NgModule, ErrorHandler, LOCALE_ID } from '@angular/core';
import { HttpModule } from '@angular/http';
import { SQLite } from '@ionic-native/sqlite';
import 'rxjs/add/operator/map';
import 'rxjs/add/operator/toPromise';
...

@NgModule({
  ...
  imports: [
    HttpModule,
  ],
  ...
  providers: [
    SQLite,
  ]
 ...
```

## Criar componetes:

Vamos para o `terminal` e criar os providers:

```cmd
  $ ionic g provider database
  $ ionic g provider event
```

No `database.ts` vamos por este código para criação do Banco de dados e uma inserção para fins de teste além da função **getDB()** que é sempre utilizada para podermos mexer com o DB [ caso tenha alguma dúvida clique [aqui](http://www.fabricadecodigo.com/crud-sqlite-ionic/) ]:

```ts
import { Injectable } from '@angular/core';
import { SQLite, SQLiteObject } from '@ionic-native/sqlite';

@Injectable()
export class DatabaseProvider {

  constructor(private sqlite: SQLite) {  }

  public getDB() {
    return this.sqlite.create({
      name: 'infocli.db',
      location: 'default'
    });
  }

  public createDatabase() {
    return this.getDB()
      .then((db: SQLiteObject) => {
        this.createTables(db);
        this.insertDefaultItems(db);
      })
      .catch(e => console.log(e));
  }

  private createTables(db: SQLiteObject) {
    // Creating as tables
    db.sqlBatch([
      ['CREATE TABLE IF NOT EXISTS events (id integer primary key AUTOINCREMENT NOT NULL, title TEXT, startTime TEXT, endTime TEXT, allDay integer)'],])
      .then(() => console.log('Tabelas criadas'))
      .catch(e => console.error('Erro ao criar as tabelas', e));
  }

  private insertDefaultItems(db: SQLiteObject) {
    
    db.executeSql('select COUNT(id) as qtd from eventos', {})
    .then((data: any) => {
      //If there is no record
      if (data.rows.item(0).qtd == 0) {

        // insert data
        db.sqlBatch([
          ['insert into events (title, startTime, endTime, allDay) values (?,?,?,?)', 
          [ 'test', '2018-01-07T12:00:00-02:00', '2018-01-07T14:00:00-02:00', 0]]
        ])
        .then(() => console.log('event data included'))
        .catch(e => console.error('Error adding data', e));

      }
    })
    .catch(e => console.error('Error verifying qtd of events', e));
  }
}
```

Aqui temos a funções para montar a estrutura do `db`, sendo colocada como private para manter a segurança e evitar mudanças desnecessárias. Todas são executadas como *Promise* para não termos problemas com o tempo de resposta do **SQLite** evitando qualquer erro na manipulação dos dados.

Bom, para este banco ser criado quando o aplicativo for instalado temos que por ele no `app.component.ts` chamar a função **createDatabase()** :

```ts
...
import { DatabaseProvider } from '../providers/database/database';
...

export class MyApp {
  rootPage: any = null;

  constructor(dbProvider: DatabaseProvider) {
    platform.ready().then(() => {
      statusBar.styleDefault();
      dbProvider.createDatabase()
        .then(() => {
          this.openHomePage(splashScreen);
        })
        .catch(() => {
          this.openHomePage(splashScreen);
        });
    });
  }

  private openHomePage(splashScreen: SplashScreen) {
    splashScreen.hide();
    this.rootPage = HomePage;
  }
  ...
}
```

Finalmente podemos agora ir ao provider `event.ts` e importamos o arquivo `database.ts`:

```ts
...
import { DatabaseProvider } from '../database/database';
import 'rxjs/add/operator/map';

@Injectable()
export class EventsDaoProvider {

  constructor(private dbProvider: DatabaseProvider) {  }
...
}

```

Feito isso, podemos iniciar o CRUD:

```ts
public insert(ev: Event) {
    return this.dbProvider.getDB()
      .then((db: SQLiteObject) => {
        let sql = 'insert into Events (title, startTime, endTime, allDay) values (?, ?, ?, ?)';
        let data = [ev.title, ev.startTime, ev.endTime, ev.allDay ? 1 : 0];
        return db.executeSql(sql, data)
          .then((data: any) => {return true;})
          .catch((e) => console.error(e));
      })
      .catch((e) => console.error(e));
  }
```

A lógica a seguir é simples chamando a função **getDB()** podemos por dentro do *then* recebemos um `SQLiteObject` basicamente um objeto do SQLite onde podemos executar comando *sql* onde só precisa ser chamada a função **executeSql()** passando sempre duas variáveis, por convenção coloque o comando *sql* em uma variável com este nome e os dados também. Você pode tratar todas as funções com esta estrutura, claramente só adaptando o retorno para sua necessidade e mudando os conteúdos das variáveis.