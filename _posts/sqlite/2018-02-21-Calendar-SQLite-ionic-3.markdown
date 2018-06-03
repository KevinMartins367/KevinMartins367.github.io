---
title: "Calendario com SQLite no Ionic 3"
date: "2018-02-21"
tags: [ionic 3, sqlite, directive]
---

Neste post, iremos implementar algumas funções basicas do SQLite no ionic 3 utilizando um directive de calendario.

**Índice**  

1. [Instalações necessárias](#id1)
1. [Preparar aplicativo](#id2)
1. [Criar componentes:](#id3)
1. [Métodos:](#id4)
+ [Get()](#id5)
+ [List()](#id6)
+ [Create()](#id7)
+ [Delete()](#id8)
+ [Edit()](#id9)


### [DEMO](http://KevinMartins367.github.io/Calendar-Sqlite-Ionic-3/demo/)

## <a name="id1"> Instalações necessárias: </a>

1. npm install ionic2-calendar --save [ para mais informações clique [aqui](https://www.npmjs.com/package/ionic2-calendar) ]
1. npm install moment --save  [ para mais informações clique [aqui](https://momentjs.com/) ]
1. npm install intl@1.2.5 --save

## <a name="id2"> Preparar aplicativo: </a>

No `app.module.ts` coloque estes import's:

```ts
import { NgModule, ErrorHandler, LOCALE_ID } from '@angular/core';
import { HttpModule } from '@angular/http';
import { NgCalendarModule } from 'ionic2-calendar';
import { SQLite } from '@ionic-native/sqlite';
import 'rxjs/add/operator/map';
import 'rxjs/add/operator/toPromise';
...

@NgModule({
  ...
  imports: [
    HttpModule,
    NgCalendarModule,
  ],
  ...
  providers: [
    {provide: ErrorHandler, useClass: IonicErrorHandler},
    { provide: LOCALE_ID, useValue: 'pt-Br' },  
    SQLite,
  ]
 ...
```

OBS: Caso os nomes dos meses não sejam traduzidos para sua língua sugiro por dentro dos import’s do `@NgModule` um array com os nomes como abaixo

```ts
    IonicModule.forRoot(MyApp, {
        monthNames: ['janeiro', 'fevereiro', 'mar\u00e7o', ... ],
        monthShortNames: ['jan', 'fev', 'mar', ... ],
        dayNames: ['domingo', 'segunda-feira', 'ter\u00e7a-feira', ... ],
        dayShortNames: ['dom', 'seg', 'ter', ... ],
    })
```

Avalie qual pode se encaixar melhor no seu projeto!

## <a name="id3"> Criar componentes: </a>

Vamos para o `terminal` e criar as telas e os providers:

```cmd
  $ ionic g page Calendar  <!-- escolha o nome da sua preferencia -->
  $ ionic g page event-modal  <!-- escolha o nome da sua preferencia -->
  $ ionic g provider database
  $ ionic g provider event
```

Pegue o exemplo de CRUD neste post [aqui](http://KevinMartins367.github.io/2018/CRUD-SQLite-ionic-3/) 

## <a name="id4"> Métodos: </a>

Agora com um CRUD já feito podemos chamar cada função nas nossas telas primeiramente vamos para a **calendar** (ou o nome que você preferiu), primeiro item que vamos mexer é colocar o *calendar* para funcionar sendo assim vamos no `.html` e colocamos a tag:

```html
  <calendar [eventSource]="eventSource"
            [noEventsLabel]="noEventsLabel"
            [calendarMode]="calendar.mode"
            [currentDate]="calendar.currentDate"
            [locale]="calendar.locale"
            [monthviewEventDetailTemplate]="template"
            (onEventSelected)="onEventSelected($event)"
            (onTitleChanged)="onViewTitleChanged($event)"
            (onTimeSelected)="onTimeSelected($event)"
            step="30"
            class="calendar">
  </calendar>
```

Agora no `.ts` podemos pôr os eventos e variáveis necessárias:

```ts
...
import { IonicPage, NavController, NavParams, ModalController, AlertController  } from 'ionic-angular';
import * as moment from 'moment';
import { EventProvider } from '../../providers/event/event';]
...

export class CalendarPage {

  eventSource = [];
  viewTitle: string;
  noEventsLabel: string = "Não há eventos listados";
  selectedDay = new Date();
  calendar = {
    mode: 'month',
    locale: 'pt-br',
    currentDate: new Date()
  };

  constructor(public navCtrl: NavController, public navParams: NavParams, private modalCtrl: ModalController, private alertCtrl: AlertController, 
    public ep: EventProvider) {
    moment.locale('pt-br');   
    this.get();
  }
```

###  <a name="id5">Get()</a>

Dentro do constructor chamamos a função do `moment` locale para termos ele trabalhando no idioma em que estamos, mude ele para a sua necessidade. E também estamos iniciando a função **get()** que vamos criar agora:

```ts
get(){
  this.ep.getAll()
  .then((e: any[]) => {
    for (let i = 0; i < e.length; i++) {
      let ev = { id: e[i].id, title:e[i].title, startTime: new Date(e[i].startTime), endTime: new Date(e[i].endTime), allDay: e[i].allDay, notification: e[i].notification , subText: null };
      this.eventSource.push(ev);
    }
    console.log(this.eventSource);
    this.list(null);
  })
  .catch((e) => {console.error(e);});
}
```

Ela chama a promise **getAll()** do provider `event.ts` e manipulamos o objeto retornado **e** onde temos um `for` que corre todos os conteúdo de nosso objeto, colocando cada item na variável local **ev** e por fim dando um **push** no nosso array global **eventSource** ao termino do `for` coloquei o *console.log()* só para termos certeza de como esta nosso array e ao fim do *then* temos nosso tratamento de erros, o *catch*.

### <a name="id6">List() </a>

Antes do *catch* temos outra função sendo chamada a **list()** sendo incluída com o valor *null* vou demostrar agora o porquê:

```ts

  list(data){
    
    let eventData = data;
    let events = this.eventSource;

    if (data != null){
      eventData.startTime = new Date(data.startTime);
      eventData.endTime = new Date(data.endTime);
      events.push(eventData);
    }

    for (let i = 0; i < events.length; i++) {

      let element = events[i];
     
      if(moment(element.startTime).format('L') == moment(element.endTime).format('L')){
        if(element.allDay == true){
          element.subText = 'O dia todo';
        }else{
          element.subText = moment(element.startTime).format('LTS');
        }
      } else if(parseInt(moment(element.endTime).format('DD')) > parseInt(moment(element.startTime).format('DD'))){

        if(element.allDay == true){
          element.subText = "O dia todo    " + moment(element.startTime).format('L') +" || " + moment(element.endTime).format('L');
        }else{
          element.subText = moment(element.startTime).format('L') +' || ' + moment(element.endTime).format('L');
        }
      }
    }

    this.eventSource = [];
    setTimeout(() => {
      this.eventSource = events;
    });
  }
```

Esta função é para listar os eventos de cada dia dando uma informação rápida sobre o mesmo que é o item *subText*. Caso, a variável `data` venha como *null* significa que já temos um *subText* para todos os eventos listados tornado a checagem pelo `for` desnecessária (esta logica foi usada para evitar lentidão no app e possíveis erros)

Voltamos ao `.html` e vamos adicionar os componentes para esta função fazer sentido:

```html
<ng-template #template let-selectedDate="selectedDate.events" let-noEventsLabel="noEventsLabel">
    <ion-list *ngIf="selectedDate.length > 0" no-lines class="event-list" padding>
      <ion-list *ngFor="let event of selectedDate">
        <ion-item-sliding>
          <ion-item ion-item detail-push (click)="onEventSelected(event)" class="event-button">
            <ion-icon name="calendar" item-start></ion-icon>
            <h1>{{event.title}}</h1>
            <p>{{event.subText}} </p>
          </ion-item>
          <ion-item-options side="right">
            <button ion-button color="btnee" (click)="delete(event)">
              delete
            </button>
          </ion-item-options>
          <ion-item-options side="left">
            <button ion-button color="btnee" (click)="edit(event)">
              edit
            </button>
          </ion-item-options>
        </ion-item-sliding>
      </ion-list>
    </ion-list>
    <ion-title *ngIf="selectedDate.length == 0"><br>{{noEventsLabel}}</ion-title>
  </ng-template>
```

Temos um *ion-list* com um `if` da variável **selectedDate** que é uma variável do próprio **calendar**, por isso ela ainda não tinha sido mencionada, ela se comunica com o array **eventSource** pegando na função **get()**  o que torna possível validarmos a existência com um **> 0**  assim criando outro *ion-list* para exibirmos os itens separados pela **list()** mostrando o *title* e o *subText* de cada evento cada um com dois *button*, um para **delete()** e outro para **edit()** (ambas funções que serão mostradas no próximo passos), e caso não tenha nenhum evento listado para o dia selecionado é mostrado o conteúdo da variável **noEventsLabel**. Isso tudo só é possível sendo colocado dentro da tag *ng-template* caso contrário ocorrerá erro na execução.

### <a name="id7"> Create() </a>

Antes disso vamos ver a tela onde podemos criar novos eventos:

```ts
  addEvent() {
    let modal = this.modalCtrl.create('EventModalPage', {selectedDay: this.selectedDay});
    modal.present();
    modal.onDidDismiss(data => {
      if (data) {
        this.ep.insert(data)
        .then((res: any) => {
          this.list(data);
        })
        .catch((e) => {console.error(e);});
      }
    });
  }
```

Para criação temos que chamar a promise **insert()** do provider `event.ts` que recebe o valor de `data` onde temos a sulução do *then* que chama a outra função **list()** passando o `data` novamente e temos nosso tratamento de erros, o *catch*. Mas antes disso temos que explicar de onde vem o objeto `data`, ele é criado na *page* event-modal que chamamos pelo **modalCtrl** (caso, tenha algum problema em fazer esta chamada apague o import do event-modal no `app.component.ts`). E vamos por o codigo nesta *page* no `.html`:

```html
<ion-header>
  <ion-navbar>
    <ion-buttons end>
      <button ion-button icon-only (click)="cancel()">
        <ion-icon name="close"></ion-icon>
      </button>
    </ion-buttons>
    <ion-title>Adicionar Evento</ion-title>
  </ion-navbar>
</ion-header>
 
<ion-content>
  <ion-list>
    <ion-item>
      <ion-input type="text" placeholder="Titulo" [(ngModel)]="event.title"></ion-input>
    </ion-item>
 
    <ion-item>
      <ion-label>Inicio</ion-label>
      <ion-datetime color="bordas" displayFormat="DD/MM/YYYY HH:mm" pickerFormat="MMM D:HH:mm" [(ngModel)]="event.startTime" [min]="minDate"
      cancelText="Cancelar"
      doneText="Concluir"></ion-datetime>
    </ion-item>
 
    <ion-item>
      <ion-label>Fim</ion-label>
      <ion-datetime color="bordas" displayFormat="DD/MM/YYYY HH:mm" pickerFormat="MMM D:HH:mm" [(ngModel)]="event.endTime" [min]="event.startTime"
      cancelText="Cancelar"
      doneText="Concluir"></ion-datetime>
    </ion-item>
 
    <ion-item>
      <ion-label>alarme</ion-label>
      <ion-datetime displayFormat="HH:mm" pickerFormat="HH:mm" hourValues="0,1,2" minuteValues="0,1,15,30,45" [(ngModel)]="event.notification"
      cancelText="Cancelar"   
      doneText="Concluir" 
      ></ion-datetime>
    </ion-item>
    
    <ion-item>
      <ion-label>Dia inteiro?</ion-label>
      <ion-checkbox [(ngModel)]="event.allDay"></ion-checkbox>
    </ion-item>
  </ion-list>
 
  <div>
    <button ion-button block round (click)="save()">
      Concluir
    </button>
  </div>
  
</ion-content>
```

Neste arquivo usamos a tag *ion-datetime*, caso não esteja familiarizado com este componente vá na [documentação](https://ionicframework.com/docs/api/components/datetime/DateTime/) do ionic. Nesta tela criamos um Objeto event que será o retorno para `data` no `.ts` do **calendar** onde vamos manipular ele para enviar para o provider.

Mas neste momento temos que colocar algumas coisas no `.ts` deste modal:

```ts
import { Component } from '@angular/core';
import { IonicPage, NavController, NavParams, ViewController } from 'ionic-angular';
import * as moment from 'moment';
...

export class EventModalPage {

  event = { id:null, title:null, startTime: moment(new Date()).format(), endTime: moment(new Date()).format(), allDay: false, notification: "00:00", subText: null };
  minDate = moment(new Date()).format();

  constructor(public navCtrl: NavController, public navParams: NavParams, public viewCtrl: ViewController) {
    
    moment.locale('pt-br');

    if(this.navParams.get('ev')){
      let ev = this.navParams.get('ev');
      
      this.event.id = ev.id;
      this.event.allDay = ev.allDay;
      this.event.endTime = moment(ev.endTime).format();
      this.event.startTime = moment(ev.startTime).format();
      this.event.subText = ev.subText;
      this.event.notification = ev.notification;
      this.event.title = ev.title;
    }else{
      let preselectedDate = moment(new Date()).format();
      this.event.startTime = preselectedDate;
      this.event.endTime = preselectedDate;
    }
    
  }

  cancel() {
    this.viewCtrl.dismiss();
  }
 
  save() {
    this.viewCtrl.dismiss(this.event);
  }
}
``` 

Aqui montamos o Objeto event que usamos no `.html` anterior e temos um `if` que fará sentido nas funções de **delete()** e **edit()** por já termos um objeto definido agora vamos para a próxima função.

### <a name="id8"> Delete(): </a>

Nesta função temos que receber duas variáveis um Objeto `ev` e uma booleana que por padrão vem como *false*:

```ts
  delete(ev, act: boolean = false){
    
    var idx = this.eventSource.indexOf(ev, 0);
    if (idx != -1) {
      this.eventSource.splice(idx, 1); 
    }
    if(act == false){
      this.ep.remove(ev.id)
      .then((res: any) => {
        this.list(null);
      })
      .catch((e) => {console.error(e);});
    } 
  }
```

Primeiro verificamos a quantia de itens no **eventSource**, para depois usarmos o método splice do `array` para remover o item selecionado. Chamamos a promise **remove()** onde enviamos unicamente o id para ser resolvida. Além de passarmos o valor `null` para função **list()**.

### <a name="id9"> Edit(): </a>

Para esta só necessita enviar o objeto de `event`

```ts
  edit(ev){
    let modal = this.modalCtrl.create('EventModalPage', {ev: ev});
    modal.present();
    modal.onDidDismiss(data => {
      
      if (data) {
        this.ep.update(data)
        .then((res: any) => {
          this.delete(ev, true);
          this.list(data);
        })
        .catch((e) => {console.error(e);});
      }
    });
  }
```

Nesta função chamamos novamente o modal para podermos manipular os dados do *event* direto para o outro. Retomando o `if` como estamos enviando *event* como *ev* ele recebe um dado e igualamos ao *event* do modal. E executando a promise **update()** para alterar tudo também chamamos a funções de **delete()** e **list()**.

Na **delete()** removemos o item do array selecionado do **eventSource** e no **list()** inserimos o item com os dados atualizados.