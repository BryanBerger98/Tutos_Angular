# Tuto Drag and Drop - Angular

Prérequis :
- Maîtriser Angular
- Nous utiliserons Bootstrap 4 pour le style

Partons du principe que nous avons déjà un projet Angular et des éléments en base de données.

## C'est parti !

Première étape, installer ``@angular/cdk``, une dépendance qui nous permettra entre autres d'utiliser le module ``DropDropModule``.

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

Une fois ceci fait, nous pouvons appliquer le drag and drop à tous les endroits où nous en avons besoin ! Le module met à disposition des méthodes et des directives qui nous permettront de facilement mettre en place cette fonctionnalité.

### Attention !
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

- ``cdkDrag`` : Permet de rendre un élément draggable.
- ``cdkDropList`` : Permet d' imposer une zone dans laquelle un élément peut être déplacé.
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

Ces méthodes peuvent être exécutées suite à l'émission d'un événement que l'on récupère à l'intérieur d'une méthode comme celle-ci dans le TypeScript :
```javascript
onTaskDrop(event: CdkDragDrop<any[]>) {}
```

Voyons comment le mettre en application maintenant.

## Application

```html
<div class="container py-5">
  <div class="row" cdkDropListGroup>
    <div class="col-sm-12 col-md-6 col-lg-4" *ngFor="let taskGroup of taskGroups" cdkDropList [cdkDropListData]="taskGroup.tasks" (cdkDropListDropped)="onTaskDrop($event)">

      <div class="bg-light shadow rounded p-3">
        <h2 class="font-weight-normal text-center">{{ taskGroup.title }}</h2>
        <div class="bg-white shadow my-3 p-4" *ngFor="let task of taskGroup.tasks" cdkDrag>
          <h4 class="font-weight-normal">{{ task.title }}</h4>
          <p class="lead mb-0">{{ task.description }}</p>
        </div>
      </div>

    </div>
  </div>
</div>
```
J'ai ici un template HTML dans le quel je souhaite pouvoir modifier l'emplacement des éléments à l'intérieur des listes mais aussi déplacer ces éléments d'une liste à une autre.

Pour cela, j'ai besoin de créer un groupe de liste. Chose que je fais à l'aide de la directive ``cdkDropListGroup`` en haut du fichier dans une ``div``.

Ensuite, j'affiche mes différentes listes (qui sont en fait mes ``taskGroups`` pour "groupes de tâches") avec un ``*ngFor`` et j'applique plusieurs directives.

Comme annoncé plus haut, ``cdkDropList`` me permet d'imposer une zone dans laquelle un élément peut être déplacé. Je ne souhaite pas que mes éléments puissent être déplacés en dehors de mes listes.

Je lie ensuite mes données à mes listes dans le DOM grâce à ceci : ``[cdkDropListData]="taskGroup.tasks"``. De cette façon je vais pouvoir récupérer dans le type script les données transmises par l'évènement drop.

D'ailleurs quand on parle du loup ! La directive ``(cdkDropListDropped)="onTaskDrop($event)"`` va exécuter la méthode ``onTaskDrop($event)`` via l'event binding. L'évènement qui la déclanchera sera le drop d'un élément. En paramètre, on indique d'ailleurs l'événement.

Il faut ensuite que j'indique à mon code les éléments qui peuvent être déplacés. Je fais ceci grâce à la directive ``cdkDrag`` que vous trouverez à côté du ``*ngFor`` qui affiche les tâches.

Voici la partie TypeScript:
```javascript
onTaskDrop(event: CdkDragDrop<any[]>) {
    if (event.previousContainer === event.container) {
      moveItemInArray(event.container.data, event.previousIndex, event.currentIndex);
    } else {
      transferArrayItem(event.previousContainer.data,
        event.container.data,
        event.previousIndex,
        event.currentIndex);
    }
  }
```
Vous remarquez qu'il s'agit de la méthode que nous avions placé en valeur de la directive ``(cdkDropListDropped)``. Lorsqu'un élément est dropped, il execute cette méthode dans laquelle on trouve une condition :

- Si le conteneur source est le même que le conteneur cible alors cela signifie que nous sommes toujours dans le même conteneur et donc que nous souhaitons seulement déplacer un élément à l'intérieur d'une liste. Dans ce cas on utilise la méthode ``moveItemInArray()``.
- Sinon, le conteneur source est différent du conteneur cible, cela veut donc dire que nous souhaitons transférer un élément d'un liste à une autre. Nous utiliserons alors la méthode ``transferArrayItem()``.

### Info supplémentaire

La directive ``cdkDragHandle`` permet d'imposer une partie d'un élément draggable comme étant la "poignée" à saisir pour déplacer cet élément. De cette façon, il faudra obligatoirement maintenir le clic sur cette partie de l'élément pour le déplacer un nul part ailleurs.

Une autre façon de créer des groupe de liste est de le faire via leurs IDs comme ceci :
```html
<div class="col-sm-12 col-md-6 col-lg-4" *ngFor="let taskGroup of taskGroups">

      <div class="bg-light shadow rounded p-3" cdkDropList [id]="taskGroup.id" [cdkDropListData]="taskGroup.tasks" [cdkDropListConnectedTo]="taskGroupsIds" (cdkDropListDropped)="onTaskDrop($event)">
        <h2 class="font-weight-normal text-center">{{ taskGroup.title }}</h2>
        <div class="bg-white shadow my-3 p-4" *ngFor="let task of taskGroup.tasks" cdkDrag>
          <h4 class="font-weight-normal">{{ task.title }}</h4>
          <p class="lead mb-0">{{ task.description }}</p>
        </div>
      </div>

    </div>
```
Vous constatez que j'utilise le property binding pour appliquer un id à chacune des listes en utilisant la directive ``[id]`` et en plus passant en valeur l'id de chaque groupe de tâches.

Ensuite, il faut "connecter" nos listes entre elles en utilisant ``[cdkDropListConnectedTo]`` avec pour valeur un tableau des IDs de chaque liste. Pour cela j'ai créé un tableau ``taskGroupsIds`` :
```javascript
 taskGroupsIds: any[] = [];
 ```
 Puis à l'initilisation du composant, lorsque nous chargeons les listes de tâches, je push les IDs de chaque liste avec un ``forEach`` comme ceci :
 ```javascript
ngOnInit() {
    this.taskGroupsSubscription = this.tasksService.taskGroupsSubject.subscribe(
      (taskGroups: any[]) => {
        this.taskGroups = taskGroups;
        
        taskGroups.forEach(function (child) {
          this.taskGroupsIds.push(child.id);
          console.log(child.id);
        }.bind(this));
        
      }
    );
    this.tasksService.emitTaskGroups();
  }
 ```
Petite parenthèse : le ``.bind(this)`` n'est souvent pas obligatoire. Je l'ai appliqué par précaution au cas où il ne reconnaîtrai pas le tableau ``taskGroupsIds`` déclaré plus haut.

En espérant que ce tutoriel vous aura été utile ;)

Si vous souhaitez en savoir plus, voici le lien vers la documentation d'Angular Material : <https://material.angular.io/cdk/drag-drop/overview>
