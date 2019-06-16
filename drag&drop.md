#Tuto Drag and Drop - Angular

Prérequis :
- Maîtriser Angular
- Nous utiliserons Bootstrap 4 pour le style

Partons du principe que nous avons déjà un projet Angular et des éléments en base de données.

##C'est parti !

Première étape, installer ``@angular/cdk``, une dépendance qui nous permettra en autres d'utiliser le module ``DropDropModule``.

```shell
npm i --save @angular/cdk
```

Ensuite, RDV dans ``app.module.ts``. Il faudra y ajouter ces deux lignes en haut du fichier :

```javascript
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { DragDropModule } from '@angular/cdk/drag-drop';
```
Pour enfin ajouter ``BrowserAnimationsModule`` et ``DragDropModule`` au tableau des imports un peu plus bas dans le fichier :
```javascript
imports: [
    BrowserAnimationsModule,
    DragDropModule
  ]
```

Une fois ceci fait, nous pouvons appliquer le drag and drop à tous les endroits où nous en avons besoin ! Le module met à disposition des méthodes et des directives qui nous permettrons de facilement mettre en place cette fonctionnalité.

###Attention !
Il est possible que vous ayez ce genre d'erreur à un moment ou un autre :
```shell
WARNING in ./node_modules/@angular/cdk/esm5/platform.es5.js 102:130-138
"export 'ɵɵinject' was not found in '@angular/core'
```
Pour la résoudre, il vous faudra simple désinstaller puis réinstaller ``@angular/core``:
```shell
npm uninstall @angular/core
npm install @angular/core
```

J'ai créé dans mon application Angular un fichier ``tasks.services.ts`` qui contient ma "base de données" :
```javascript
import { Injectable } from '@angular/core';
import { Subject } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class TasksService {

  taskGroups: any[] = [
    {
      title: "A faire",
      id: "todo",
      tasks: [
        {
          id: 0,
          title: "Première tache",
          description: "Voici ma première tache"
        },
        {
          id: 1,
          title: "Acheter du pain",
          description: "Aller acheter du pain à la boulangerie"
        },
        {
          id: 2,
          title: "Faire les courses",
          description: "Prendre du lait, des céréales et des bananes"
        }
      ]
    },
    {
      title: "En cours",
      id: "inProgress",
      tasks: [
        {
          id: 0,
          title: "Réparer la voiture",
          description: "Changer le phare avant gauche"
        },
        {
          id: 1,
          title: "Etendre le linge",
          description: "Etendre le linge"
        },
        {
          id: 2,
          title: "Préparer une quiche",
          description: "Voir la recette de la quiche"
        }
      ]
    },
    {
      title: "Terminé",
      id: "done",
      tasks: [
        {
          id: 0,
          title: "Mettre la table",
          description: "Mettre 8 couverts pour les invités"
        },
        {
          id: 1,
          title: "Passer l'aspirateur",
          description: "Nettoyage du rez de chaussée"
        }
      ]
    }
  ];
  
  taskGroupsSubject = new Subject<any[]>();

  constructor() { }

  emitTaskGroups() {
    this.taskGroupsSubject.next(this.taskGroups);
  }

}
```

Je récupère ensuite ces données dans mon composant de la manière suivante :
```javascript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { Subscription } from 'rxjs';
import { TasksService } from '../services/tasks.service';

@Component({
  selector: 'app-home',
  templateUrl: './home.component.html',
  styleUrls: ['./home.component.css']
})
export class HomeComponent implements OnInit, OnDestroy {

  taskGroups: any[];
  taskGroupsSubscription: Subscription;

  taskGroupsIds: any[];

  constructor(
    private tasksService: TasksService
  ) { }

  ngOnInit() {
    this.taskGroupsSubscription = this.tasksService.taskGroupsSubject.subscribe(
      (taskGroups: any[]) => {
        this.taskGroups = taskGroups;
        taskGroups.forEach(function (child) {
          this.taskGroupsIds.push(child);
          console.log(child);
        });
      }
    );
    this.tasksService.emitTaskGroups();
  }

  ngOnDestroy() {
    this.taskGroupsSubscription.unsubscribe();
  }

}
```

Quelques directives à connaître pour commencer :

- ``cdkDrag`` : Permet de rendre un élément draggable
- ``cdkDropList`` : Permet de imposer une zone dans laquelle un élément peut être déplacé.
- ``cdkDropListGroup`` : Permet de créer un groupe de zones dans lesquels des éléments peuvent être déplacés et/ou transférés. Un seconde méthode existe pour le faire.
- ``[cdkDropListData]`` : Permet de lier les données à une zone afin de pouvoir plus simplement les traiter.
- ``[cdkDropData]`` : Permet de lier les données à un élément dans le même but que ci-dessus.
- ``(cdkDropListDropped)`` : Exécute une méthode lorsqu'un élément est dopped.

Ensuite pour ce tuto, nous auront besoin de deux méthodes fournies par le module :

```javascript
moveItemInArray(event.container.data, event.previousIndex, event.currentIndex)
```
Cette méthode pend en paramètre les données envoyées par l'évenement ainsi que les index précédent et courant de l'évènement. De cette façon, elle va modifier l'état du tableau (pour déplacer les éléments) de l'ancien état vers le nouveau.

```javascript
transferArrayItem(event.previousContainer.data,
        event.container.data,
        event.previousIndex,
        event.currentIndex);
```
Celle-ci prend 4 paramètre : Les données de la source, les données de la cible, et les index de chacun. De cette façon, elle va pouvoir déplacer un élément d'un tableau à un autre.

Ces méthodes peuvent être exécutées suite à l'émission d'un événement que l'on récupère à l'intérieur d'une méthode comme celle-ci dans le typerscript :
```javascript
onTaskDrop(event: CdkDragDrop<any[]>) {}
```

Voyons comment le mettre en application maintenant.

## Application
