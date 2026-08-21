# Parte 1: Primeros Pasos con Angular e Inteligencia Artificial

## Capítulo 2: IssueTracker Lite: Construcción de un Sistema Inteligente de Seguimiento de Errores

### Sección: Introducción

Una de las herramientas de Angular más populares para diseñar interfaces que recopilen datos del usuario son los formularios reactivos (*reactive forms*). Los desarrolladores de Angular prefieren los formularios reactivos porque trabajan con datos inmutables en un contexto reactivo. El Clarity Design System proporciona patrones de diseño y primitivas que se integran a la perfección con los formularios reactivos de Angular para ofrecer experiencias consistentes en aplicaciones de nivel empresarial.

Vamos a cubrir los siguientes temas:

- Instalación de Clarity Design System
- Visualización del resumen de incidencias
- Reporte de nuevas incidencias
- Resolución de incidencias

Al final de este capítulo, aprenderás a utilizar diferentes componentes de Clarity y a construir una aplicación de gestión de incidencias.

---

### Sección: Requisitos Técnicos

Puedes descargar el proyecto de ejemplo y el código de este libro siguiendo las instrucciones de la sección *Download the example code files* en el Prefacio de este libro.

Los archivos de código de este capítulo están incluidos en el paquete de código descargable.

Utilizaremos el código de la carpeta `ch01` como punto de partida para construir la aplicación de este capítulo.

---

### Sección: Instalación de Clarity Design System

Clarity es un sistema de diseño de código abierto que contiene:

- Un conjunto de directrices de UX y UI para crear aplicaciones web.
- Un framework propietario de HTML y CSS.
- Una colección de componentes de UI basados en Angular diseñados de acuerdo con las directrices anteriores.

En esta sección, aprenderás cómo instalar y configurar Clarity en la aplicación Angular desde la carpeta `ch01` de la sección Requisitos Técnicos:

1. Ejecuta el siguiente comando dentro del proyecto de Angular CLI para instalar Clarity:

```bash
npm install @cds/core @clr/angular @clr/ui
```

Si tienes problemas para ejecutar el comando anterior, consulta el archivo `CHANGELOG.md` del repositorio de GitHub del capítulo. Para acceder al enlace del repositorio, sigue los pasos de la sección "Download the example code files" en el Prefacio.

2. Abre el archivo `angular.json` y agrega los estilos CSS de Clarity en el array `styles`:

```json
"styles": [
  "node_modules/@cds/core/global.min.css",
  "node_modules/@cds/core/styles/theme.dark.min.css",
  "node_modules/@clr/ui/clr-ui.min.css",
  "src/styles.scss"
]
```

3. Clarity necesita el paquete de animaciones de Angular. Ejecuta el siguiente comando para instalarlo:

```bash
npm install @angular/animations
```

Si tienes problemas para ejecutar el comando anterior, consulta el archivo `CHANGELOG.md` del repositorio de GitHub del capítulo. Para acceder al enlace del repositorio, sigue los pasos de la sección "Download the example code files" en el Prefacio.

4. Abre el archivo `app.config.ts` e importa el método `provideAnimationsAsync`:

```typescript
import { provideAnimationsAsync } from '@angular/platform-browser/animations/async';
```

El método anterior ha quedado obsoleto desde Angular 20.2, pero la versión actual de Clarity todavía lo necesita.

5. Agrega las animaciones en los proveedores de la aplicación:

```typescript
export const appConfig: ApplicationConfig = {
  providers: [
    provideBrowserGlobalErrorListeners(),
    provideRouter(routes),
    provideAnimationsAsync()
  ]
};
```

6. Abre el archivo `app.ts` y crea el siguiente constructor para establecer el título del capítulo:

```typescript
constructor() {
  this.chapterTitleService.setTitle(
    '[Chapter 2](https://subscription.packtpub.com/book/programming/9781806668472/2): IssueTracker Lite'
  );
}
```

Para previsualizar la aplicación, ejecuta el comando `ng serve` y navega a `localhost:4200`:

> *Figura 2.1 – Salida de la aplicación*

La instalación de Clarity cambia el estilo de la página como se muestra en la figura anterior.

La página principal ahora muestra el título del capítulo actual. Refactorizaremos nuestra aplicación para mostrar el título en un encabezado de componente de Clarity:

1. Abre el archivo `app.html` y reemplaza su contenido con el siguiente código HTML:

```html
<clr-main-container>
  <chapter-title [chapterTitle]="title()" />
</clr-main-container>
```

El elemento `clr-main-container` es un componente de diseño (*layout*) de Clarity que actúa como el elemento anfitrión de la aplicación.

Los componentes de Clarity de la colección de Angular comienzan con el prefijo `clr`.

El componente de encabezado que utilizaremos debe residir dentro de un componente de diseño.

2. Abre el archivo `app.ts` y agrega la siguiente sentencia de importación:

```typescript
import { ClrLayoutModule } from '@clr/angular';
```

La clase `ClrLayoutModule` contiene el componente `<clr-main-container>` que necesitamos para alojar el encabezado.

3. Agrega la clase en el array `imports` del decorador del componente:

```typescript
imports: [ChapterTitleComponent, ClrLayoutModule]
```

4. Abre el archivo `chapter-title.component.ts` y agrega la siguiente sentencia de importación:

```typescript
import { ClrNavigationModule } from '@clr/angular';
```

La clase `ClrNavigationModule` contiene el componente de encabezado que necesitamos para mostrar el título del capítulo.

5. Reemplaza la propiedad `template` del decorador del componente con el siguiente código HTML:

```html
<clr-header>
  <div class="branding">
    <a href="javascript://" class="nav-link">
      <span class="title">{{chapterTitle()}}</span>
    </a>
  </div>
</clr-header>
```

El `clr-header` es un componente de Clarity que representa el encabezado de una aplicación Angular.

6. Agrega la clase `ClrNavigationModule` al array `imports` del decorador del componente:

```typescript
imports: [ClrNavigationModule]
```

Verifica que la salida de la aplicación se vea como la siguiente:

> *Figura 2.2 – Encabezado de la aplicación*

El diseño de la aplicación ahora sigue el sistema de diseño Clarity. Cuenta con un contenedor principal con un encabezado que muestra el título del capítulo.

En la siguiente sección, aprenderemos cómo interactuar con el contenido principal y mostrar una lista de incidencias.

---

### Sección: Visualización del Resumen de Incidencias

El objetivo principal de la aplicación es gestionar y realizar un seguimiento de las incidencias. Los usuarios deberían ver una lista de incidencias pendientes cuando abran la aplicación en su navegador. Completaremos las siguientes tareas para implementar la lista de incidencias:

- Gestión del estado de la aplicación
- Visualización de incidencias pendientes

Comenzaremos implementando un mecanismo para almacenar todas las incidencias de la aplicación.

#### Gestión del estado de la aplicación

Las signals en Angular son la forma perfecta de gestionar el estado de la aplicación. Crearemos un servicio que mantenga los datos de las incidencias mediante signals y los proporcione en diferentes partes de la aplicación:

1. Ejecuta el siguiente comando para crear el servicio de Angular:

```bash
ng generate service issues
```

2. Ejecuta el siguiente comando para crear una interfaz de TypeScript para los datos de las incidencias:

```bash
ng generate interface issue
```

3. Abre el archivo `issue.ts` y agrega las siguientes propiedades a la interfaz `Issue`:

```typescript
issueNo: number;
title: string;
description: string;
priority: 'low' | 'high';
type: 'Feature' | 'Bug';
completed: boolean;
```

La propiedad `completed` indica que una incidencia ha sido resuelta.

4. Abre el archivo `issues.ts` e importa la interfaz `Issue`:

```typescript
import { Issue } from './issue';
```

5. Crea la siguiente propiedad en la clase `Issues` para almacenar los datos de las incidencias:

```typescript
readonly issues = signal<Issue[]>([]);
```

En el código anterior, inicializamos la propiedad `issues` como un array vacío utilizando el método `signal` del paquete `@angular/core`. Si deseas comenzar con datos de ejemplo, puedes usar el archivo `mock-issues.ts` de la carpeta `public` que existe en el material de GitHub de este capítulo. Para acceder al enlace del repositorio, sigue los pasos de la sección "Download the example code files" en el Prefacio. Importa el archivo usando el siguiente fragmento:

```typescript
import { issueData } from '../../public/mock-issues';
```

El servicio anterior es responsable de almacenar los datos de las incidencias en la aplicación. En la siguiente sección, aprenderemos a utilizar un componente data grid de Clarity para mostrar y filtrar incidencias.

#### Visualización de incidencias pendientes

El data grid es un componente adecuado para mostrar datos tabulares. Clarity contiene un componente data grid que proporciona capacidades de filtrado y ordenación listas para usar. Crearemos un componente para usar el data grid y mostrar una lista de incidencias pendientes:

1. Ejecuta el siguiente comando para generar el componente:

```bash
ng generate component issue-list
```

2. Abre el archivo `issue-list.ts` e importa la clase del servicio `Issues`:

```typescript
import { Issues } from '../issues';
```

3. Inyecta el servicio `Issues` utilizando la siguiente declaración en la clase `IssueList`:

```typescript
private issuesService = inject(Issues);
```

Debes importar el método `inject` del paquete `@angular/core`.

4. Agrega la siguiente propiedad para obtener las incidencias pendientes del servicio:

```typescript
protected readonly issues = computed(() => {
  const data = this.issuesService.issues();
  return data.filter(i => !i.completed);
});
```

El código anterior utiliza el método `computed` para derivar una nueva lista de incidencias a partir de los datos iniciales. La propiedad `issues` contiene únicamente incidencias pendientes. Debes importar el método `computed` del paquete `@angular/core`.

El proceso anterior define la lógica de presentación de la lista de incidencias. Debemos interactuar con la plantilla del componente para conectar el HTML con los datos:

1. Agrega la siguiente sentencia de importación para usar el componente data grid de Clarity:

```typescript
import { ClrDatagridModule } from '@clr/angular';
```

2. Agrega la clase `ClrDatagridModule` en el array `imports` del decorador del componente:

```typescript
imports: [ClrDatagridModule]
```

3. Abre el archivo `issue-list.html` y agrega el siguiente fragmento para definir el data grid:

```html
<clr-datagrid></clr-datagrid>
```

4. Inserta los siguientes selectores `clr-dg-column` dentro de la cuadrícula para definir las columnas:

```html
<clr-dg-column [clrDgField]="'issueNo'" [clrDgColType]="'number'">
  Issue No
</clr-dg-column>
<clr-dg-column [clrDgField]="'type'">
  Type
</clr-dg-column>
<clr-dg-column [clrDgField]="'title'">
  Title
</clr-dg-column>
<clr-dg-column [clrDgField]="'description'">
  Description
</clr-dg-column>
<clr-dg-column [clrDgField]="'priority'">
  Priority
</clr-dg-column>
```

La directiva `clrDgField` vincula el nombre de la propiedad de la incidencia con la columna y proporciona capacidades de ordenación y filtrado. La ordenación funciona de forma predeterminada con contenido basado en cadenas de texto. Si queremos ordenar por un tipo primitivo diferente, debemos usar la directiva `clrDgColType` y especificar el tipo.

5. Inserta el siguiente fragmento debajo de las columnas de la cuadrícula para definir las filas:

```html
<clr-dg-row *clrDgItems="let issue of issues()">
</clr-dg-row>
```

La directiva `clrDgItems` itera sobre la propiedad `issues` y crea una fila para cada una.

6. Inserta los siguientes selectores `clr-dg-cell` dentro de las filas del grid para definir las celdas:

```html
<clr-dg-cell>{{issue.issueNo}}</clr-dg-cell>
<clr-dg-cell>{{issue.type}}</clr-dg-cell>
<clr-dg-cell>{{issue.title}}</clr-dg-cell>
<clr-dg-cell>{{issue.description}}</clr-dg-cell>
<clr-dg-cell>
  <span class="label" [class.label-danger]="issue.priority === 'high'">
    {{issue.priority}}
  </span>
</clr-dg-cell>
```

Cada celda muestra el valor de la columna a la que pertenece en la cuadrícula. En la última celda, agregamos la clase `label-danger` cuando una incidencia tiene prioridad alta para indicar su importancia.

7. Agrega el siguiente fragmento debajo del elemento `<clr-dg-row>` para definir el pie de página de la cuadrícula que muestra el número total de incidencias:

```html
<clr-dg-footer>{{issues().length}} issues</clr-dg-footer>
```

La configuración del componente data grid es compleja y modular. Puedes añadir funciones más avanzadas visitando [https://clarity.design/documentation/datagrid](https://clarity.design/documentation/datagrid).

Podemos previsualizar la nueva funcionalidad de nuestra aplicación agregándola al componente principal de la aplicación:

1. Abre el archivo `app.html` y agrega el siguiente fragmento debajo del selector `chapter-title`:

```html
<div class="content-container">
  <div class="content-area">
    <app-issue-list />
  </div>
</div>
```

El fragmento anterior muestra la lista de incidencias en el área de contenido del contenedor principal.

2. Abre el archivo `app.ts` e importa la clase del componente `IssueList`:

```typescript
import { IssueList } from './issue-list/issue-list';
```

3. Agrega la clase en el array `imports` del decorador del componente:

```typescript
imports: [
  ChapterTitleComponent,
  ClrLayoutModule,
  IssueList
]
```

Si ejecutamos nuestra aplicación Angular utilizando `ng serve`, la salida se verá de la siguiente manera:

> *Figura 2.3 – Incidencias pendientes*

En la figura anterior, la aplicación utiliza datos de muestra del archivo `mock-issues.ts`.

Has alcanzado un punto en el que puedes mostrar el estado de la aplicación en un componente data grid. Hasta ahora, hemos utilizado datos simulados para construir el estado de la aplicación. En la siguiente sección, aprenderás a utilizar formularios para reportar nuevas incidencias.

---

### Sección: Reporte de Nuevas Incidencias

La aplicación puede mostrar datos de incidencias, pero carece de un mecanismo para agregar nuevos datos. En esta sección, construiremos un flujo de trabajo de usuario para reportar nuevas incidencias. Utilizaremos formularios reactivos para diseñar una interfaz de usuario destinada a recopilar los detalles de las incidencias. La implementación de la nueva funcionalidad implica las siguientes tareas:

- Diseño del formulario
- Validación de la entrada del usuario
- Adición de nuevas incidencias

Comenzaremos con la tarea principal de diseñar un formulario reactivo para introducir los detalles de la incidencia.

#### Diseño del formulario

El reportero de incidencias será un componente con un formulario reactivo. El formulario permitirá al usuario ingresar detalles de la incidencia y guardarlos en el estado de la aplicación:

1. Ejecuta el siguiente comando para crear el componente:

```bash
ng generate component issue-reporter
```

2. Abre el archivo `issue-reporter.ts` y agrega las siguientes sentencias de importación del paquete de formularios reactivos:

```typescript
import { FormControl, FormGroup, ReactiveFormsModule } from '@angular/forms';
```

3. Agrega la clase `ReactiveFormsModule` al array `imports` del decorador del componente:

```typescript
imports: [ReactiveFormsModule]
```

4. Define la estructura del formulario utilizando la siguiente propiedad:

```typescript
form = new FormGroup({
  title: new FormControl(''),
  description: new FormControl(''),
  priority: new FormControl(''),
  type: new FormControl(''),
});
```

Hemos diseñado los elementos centrales del formulario y debemos asociarlos con los elementos HTML correspondientes:

1. En el archivo `issue-reporter.ts`, agrega la siguiente sentencia para importar los componentes relacionados con formularios de la biblioteca Clarity:

```typescript
import { ClrFormsModule } from '@clr/angular';
```

2. Agrega la clase `ClrFormsModule` en el array `imports` del decorador del componente:

```typescript
imports: [ReactiveFormsModule, ClrFormsModule]
```

3. Abre el archivo `issue-reporter.html` y reemplaza su contenido con el siguiente código HTML:

```html
<form clrForm [formGroup]="form"></form>
```

En el código anterior, utilizamos la directiva `clrForm` para indicar que el elemento de formulario es un componente de formulario de Clarity.

4. Inserta el siguiente fragmento dentro del selector de formulario para crear un componente de entrada para el título de la incidencia:

```html
<clr-input-container>
  <label>Title</label>
  <input clrInput formControlName="title" />
</clr-input-container>
```

5. Inserta el siguiente fragmento después del componente de entrada para crear un control de área de texto para la descripción de la incidencia:

```html
<clr-textarea-container>
  <label>Description</label>
  <textarea clrTextarea formControlName="description">
  </textarea>
</clr-textarea-container>
```

6. Inserta el siguiente fragmento después del componente de área de texto para crear un control de grupo de botones de opción (*radio buttons*) para la prioridad de la incidencia:

```html
<clr-radio-container clrInline>
  <label>Priority</label>
  <clr-radio-wrapper>
    <input type="radio" value="low" clrRadio formControlName="priority" />
    <label>Low</label>
  </clr-radio-wrapper>
  <clr-radio-wrapper>
    <input type="radio" value="high" clrRadio formControlName="priority" />
    <label>High</label>
  </clr-radio-wrapper>
</clr-radio-container>
```

El elemento `clr-radio-container` es el contenedor de grupo para controles de radio en Clarity.

7. El siguiente código representa el control desplegable para el tipo de incidencia. Agrégalo debajo del grupo de botones de opción:

```html
<clr-select-container>
  <label>Type</label>
  <select clrSelect formControlName="type">
    <option value="Feature">Feature</option>
    <option value="Bug">Bug</option>
  </select>
</clr-select-container>
```

8. Agrega un elemento de botón debajo del componente de selección para enviar los detalles del formulario:

```html
<button class="btn btn-primary">Create</button>
```

Podemos previsualizar el nuevo formulario agregándolo al componente principal de la aplicación encima de la lista de incidencias:

1. Abre el archivo `app.html` y agrega el reportero de incidencias encima del selector `app-issue-list`:

```html
<div class="content-area">
  <app-issue-reporter />
  <app-issue-list />
</div>
```

2. Abre el archivo `app.ts` e importa la clase del componente `IssueReporter`:

```typescript
import { IssueReporter } from './issue-reporter/issue-reporter';
```

3. Agrega la clase en el array `imports` del decorador del componente:

```typescript
imports: [
  ChapterTitleComponent,
  ClrLayoutModule,
  IssueList,
  IssueReporter
]
```

La página principal de la aplicación debería mostrar el nuevo formulario de la siguiente manera:

> *Figura 2.4 – Reportero de incidencias*

Utilizamos la biblioteca Clarity para diseñar una interfaz de usuario sencilla y ergonómica para reportar nuevas incidencias.

En la siguiente sección, mejoraremos la experiencia de usuario (UX) del formulario aplicando validaciones a los controles de usuario.

#### Validación de la entrada del usuario

Una buena UX en los formularios guía a los usuarios para completar todos los detalles necesarios y previene errores accidentales. En esta sección, aplicaremos validaciones en el formulario de reporte de incidencias para:

- Mostrar qué campos no deben estar vacíos
- Evitar que el usuario envíe campos vacíos

Utilizaremos los mecanismos de validación de los formularios reactivos con los componentes de formulario de Clarity:

1. Abre el archivo `issue-reporter.ts` e importa la clase `Validators` del paquete de formularios:

```typescript
import { FormControl, FormGroup, ReactiveFormsModule, Validators } from '@angular/forms';
```

2. Marca todos los controles excepto la descripción como requeridos utilizando la propiedad `required`:

```typescript
form = new FormGroup({
  title: new FormControl('', Validators.required),
  description: new FormControl(''),
  priority: new FormControl('', Validators.required),
  type: new FormControl('', Validators.required),
});
```

3. Abre el archivo `issue-reporter.html` y agrega el siguiente componente debajo del elemento HTML de entrada:

```html
<clr-control-error>Title is required</clr-control-error>
```

El elemento `clr-control-error` es un componente de Clarity que se conecta con un contenedor de control, como `clr-input-container`. Muestra un error cuando el control de formulario contenedor no es válido.

4. Agrega un componente de error de control para cada campo requerido restante y asígnale un mensaje adecuado.

5. Ejecuta la aplicación, haz clic dentro del campo Title y luego haz clic fuera del formulario:

> *Figura 2.5 – Validación de título*

El mensaje de error aparece debajo de los campos, indicando que aún no hemos ingresado ningún valor. Los mensajes de validación en la biblioteca Clarity se indican mediante texto rojo y un icono de exclamación en el control de formulario.

Intenta hacer clic en el botón Create y verifica que la aplicación muestre todos los mensajes de validación.

Mejoramos la UX de la aplicación utilizando la validación de formularios con los componentes de error de control de Clarity. La validación garantiza la integridad de los datos, permitiendo a los usuarios ingresar datos con confianza. En la última sección, aprenderemos a recopilar la entrada del usuario y guardarla en el estado de la aplicación.

#### Adición de nuevas incidencias

El reportero de incidencias recopilará datos, los guardará en el estado de la aplicación y la nueva incidencia aparecerá en la lista. Utilizaremos el proceso de envío de formularios HTML para iniciar el proceso:

1. Abre el archivo `issues.ts` y agrega el siguiente método que añade una nueva incidencia al array `issues`:

```typescript
create(issue: Issue) {
  this.issues.update(issues => {
    issue.issueNo = this.issues().length + 1;
    return [...issues, issue];
  });
}
```

El método `create` asigna un nuevo `issueNo` en función del número total de incidencias.

2. Abre el archivo `issue-reporter.html` y agrega la siguiente vinculación al evento `click` del botón:

```html
<button class="btn btn-primary" (click)="create()">
  Create
</button>
```

El reportero de incidencias contiene solo un elemento de botón HTML, por lo que el formulario se enviará cuando se haga clic en el botón.

3. Abre el archivo `issue-reporter.ts` e importa el servicio de incidencias y la interfaz correspondiente:

```typescript
import { Issues } from '../issues';
import { Issue } from '../issue';
```

4. Inyecta la clase del servicio `Issues` en el componente usando la siguiente propiedad:

```typescript
private issuesService = inject(Issues);
```

Importa el método `inject` del paquete `@angular/core`.

5. Agrega el siguiente método a la clase del componente `IssueReporter`:

```typescript
create() {
  if (!this.form.valid) {
    return;
  }
  this.issuesService.create(this.form.value as Issue);
}
```

Crear una nueva incidencia es aceptable solo si el formulario es válido. Si el formulario no tiene errores, llamamos al método `create`, pasando la propiedad `value` del formulario como un tipo `Issue`.

Intenta reportar una nueva incidencia y verifica que se muestre en la lista de incidencias.

Hemos alcanzado otro hito en el desarrollo del gestor de incidencias. La aplicación puede reportar y mostrar nuevas incidencias, ayudando a los usuarios y aumentando la visibilidad de las tareas. En la siguiente sección, proporcionaremos a los usuarios controles de mantenimiento para cerrar incidencias.

---

### Sección: Resolución de Incidencias

El ciclo de vida de una incidencia es específico y, en algún momento, debe cerrarse. Construiremos un flujo de trabajo que permita a los usuarios resolver incidencias de la lista. La aplicación solicitará confirmación antes de resolver una incidencia mediante un cuadro de diálogo modal:

1. Ejecuta el siguiente comando para crear el componente que alojará el cuadro de diálogo modal:

```bash
ng generate component confirm
```

2. Abre el archivo `confirm.ts` e importa el componente de diálogo del paquete Clarity:

```typescript
import { ClrModalModule } from '@clr/angular';
```

3. Agrega la clase `ClrModalModule` en el array `imports` del decorador del componente:

```typescript
imports: [ClrModalModule]
```

4. Importa los símbolos `Component`, `input` y `output` del paquete `@angular/core`:

```typescript
import { Component, input, output } from '@angular/core';
```

5. Crea los siguientes enlaces de entrada y salida en la clase del componente `Confirm`:

```typescript
readonly issueNo = input<number>();
readonly confirmed = output<boolean>();
```

Utilizaremos la propiedad `issueNo` para mostrar el número de incidencia que deseamos resolver. Utilizaremos el evento `confirmed` para indicar si el usuario confirmó la resolución de la incidencia.

La lógica de presentación del componente es el primer paso del flujo de trabajo para resolver incidencias. Agregaremos los componentes necesarios de Clarity en la plantilla para que funcione:

1. Abre el archivo `confirm.html` y reemplaza su contenido con el siguiente código HTML:

```html
<clr-modal [clrModalOpen]="issueNo() !== undefined">
</clr-modal>
```

El selector `clr-modal` corresponde al componente modal de Clarity. El modal se abrirá cuando se establezca la propiedad `issueNo`.

2. Agrega el siguiente fragmento dentro del componente modal para definir su título:

```html
<h3 class="modal-title">
  Resolve Issue # {{issueNo()}}
</h3>
```

3. Inserta el siguiente fragmento después del título para definir el contenido principal del modal:

```html
<div class="modal-body">
  <p>Are you sure you want to close the issue?</p>
</div>
```

4. Agrega un elemento HTML `div` después del contenido principal del modal:

```html
<div class="modal-footer">
  <button class="btn btn-outline" (click)="confirmed.emit(false)">
    Cancel
  </button>
  <button class="btn btn-danger" (click)="confirmed.emit(true)">
    Yes, continue
  </button>
</div>
```

El fragmento anterior define las acciones del modal con dos botones. Cada botón activa el evento de salida `confirmed`.

La lista de incidencias será el punto de entrada principal del flujo de trabajo para resolver una incidencia. Agregaremos el modal de confirmación en el componente de lista para que se abra cuando los usuarios interactúen con las incidencias:

1. Abre el archivo `issues.ts` y crea el siguiente método que cerrará la incidencia y actualizará el estado de la aplicación:

```typescript
resolve(no: number) {
  const i = this.issues().findIndex(i => i.issueNo === no);
  this.issues.update(issues => {
    issues[i].completed = true;
    return [...issues];
  });
}
```

El método `resolve` busca el índice de la incidencia que necesita actualizarse según su número y establece la propiedad `completed` en `true`.

2. Abre el archivo `issue-list.ts` y declara la siguiente propiedad de tipo signal:

```typescript
readonly selected = signal<number | undefined>(undefined);
```

Importa el método `signal` del paquete `@angular/core`. La propiedad `selected` contendrá el número de la incidencia seleccionada para resolver.

3. Crea el siguiente método en la clase del componente `IssueList`:

```typescript
complete(confirmed: boolean) {
  if(confirmed) {
    this.issuesService.resolve(this.selected()!);
  }
  this.selected.set(undefined);
}
```

El modal activará el método anterior de acuerdo con la retroalimentación del usuario. Resolverá la incidencia tras la aprobación y restablecerá la incidencia seleccionada en todos los casos.

4. Agrega la siguiente sentencia para importar el componente de confirmación:

```typescript
import { Confirm } from '../confirm/confirm';
```

5. Agrega la clase `Confirm` al array `imports` del decorador del componente:

```typescript
imports: [ClrDatagridModule, Confirm]
```

6. Abre el archivo `issue-list.html` y agrega el componente de confirmación al final del archivo de plantilla:

```html
<app-confirm [issueNo]="selected()" (confirmed)="complete($event)" />
```

7. Agrega el siguiente fragmento HTML al comienzo del elemento de fila del data grid:

```html
<clr-dg-action-overflow>
  <button class="action-item" (click)="selected.set(issue.issueNo)">
    Resolve
  </button>
</clr-dg-action-overflow>
```

El componente `clr-dg-action-overflow` de Clarity agrega un menú desplegable en cada fila de la tabla. El menú contiene un único botón para establecer la propiedad `selected` en el número de la incidencia actual cuando se hace clic en él.

Todos los pasos necesarios del flujo de trabajo para resolver una incidencia están listos. Veamos una guía paso a paso de cómo funciona:

1. Ejecuta `ng serve`, si aún no estás ejecutando la aplicación, y navega a `http://localhost:4200`.
2. Si aún no tienes datos de incidencias, utiliza el formulario del reportero de incidencias para crear una nueva incidencia.
3. Haz clic en el menú de acciones de una fila y selecciona **Resolve**. El menú es el icono de tres puntos verticales junto a la columna Issue No:

> *Figura 2.6 – Menú de resolución*

4. En el cuadro de diálogo de confirmación que aparece, haz clic en el botón **YES, CONTINUE**:

> *Figura 2.7 – Cuadro de diálogo de resolución de incidencia*

5. Después de hacer clic en el botón, el cuadro de diálogo se cerrará y la incidencia ya no debería ser visible en la lista.

¡IssueTracker Lite v1.0 ya está listo para producción! Resolver incidencias en la aplicación es importante si queremos mantener nuestra aplicación actualizada. El cuadro de diálogo de confirmación y el menú contextual del data grid garantizan una buena experiencia de usuario (UX) durante el uso de la aplicación.

---

### Sección: Resumen

En este capítulo, aprendimos cómo construir una aplicación web sencilla para realizar un seguimiento de errores y solicitudes de funcionalidades. Profundizamos en el Clarity Design System, desde el diseño del layout de la aplicación, pasando por la visualización de datos de incidencias en formato tabular, hasta la gestión de la retroalimentación del usuario. Utilizamos el paquete de formularios reactivos de Angular para recopilar la entrada del usuario y proporcionar mensajes de ayuda significativos. En el siguiente capítulo, exploraremos Angular Material y Google Firebase mientras construimos una aplicación web para gestionar pedidos de menú.

---

### Sección: Ejercicios

Utiliza la API de formularios reactivos para restablecer los campos del reportero de incidencias cada vez que agreguemos una nueva incidencia en el estado de la aplicación.

---

### Sección: Lecturas Complementarias

- **Clarity Design System:** [https://clarity.design](https://clarity.design/)
- **Reactive forms:** [https://angular.dev/guide/forms/reactive-forms](https://angular.dev/guide/forms/reactive-forms)
- **Validating reactive forms:** [https://angular.dev/guide/forms/form-validation#validating-input-in-reactive-forms](https://angular.dev/guide/forms/form-validation#validating-input-in-reactive-forms)
