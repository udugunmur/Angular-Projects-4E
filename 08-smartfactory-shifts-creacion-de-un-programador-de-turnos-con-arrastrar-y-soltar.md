# Parte 1: Primeros Pasos con Angular e Inteligencia Artificial

## Capítulo 8: SmartFactory Shifts: Creación de un Programador de Turnos con Arrastrar y Soltar

### Sección: Introducción

El trabajo de un gerente en una gran unidad fabril es exigente en términos de carga de trabajo y presión. La gestión de turnos requiere un gran esfuerzo para asignar a los empleados correctamente de modo que puedan ser eficaces. Los gerentes deben tener una visión general de alto nivel de las franjas horarias de trabajo y del personal disponible para trabajar eficientemente.

En este capítulo, construiremos una aplicación de programación de turnos para gerentes centrada en la velocidad y la experiencia de usuario. Utilizaremos una vista de calendario de la biblioteca Kendo UI para asignar empleados mediante una experiencia de arrastrar y soltar (*drag-and-drop*).

Vamos a cubrir los siguientes temas:

- Instalación de Kendo UI
- Adición de nuevos empleados
- Asignación de empleados a un turno
- Movimiento de empleados entre turnos

---

### Sección: Requisitos Técnicos

Todos los ejemplos de código de este capítulo se pueden encontrar en las carpetas `ch01` y `ch08` en GitHub. Puedes descargar el proyecto de ejemplo y el código de este libro siguiendo las instrucciones de la sección *Download the example code files* en el Prefacio de este libro.

Los archivos de código de este capítulo están incluidos en el paquete de código descargable. Utilizaremos el código de la carpeta `ch01` como punto de partida para construir la aplicación de este capítulo.

---

### Sección: Instalación de Kendo UI

Kendo UI es una suite de componentes web nativos creados por Telerik. Cada componente tiene una API coherente y es altamente personalizable. La versión de Angular de Kendo UI contiene un componente scheduler que podemos usar para construir nuestra aplicación.

Antes de usar Kendo UI, navega a [https://www.telerik.com/kendo-angular-ui/components/licensing](https://www.telerik.com/kendo-angular-ui/components/licensing) y sigue las instrucciones para configurar una licencia.

Si ya tienes una licencia de Kendo UI, no necesitas obtener una nueva. Todas las versiones de Kendo UI, incluida la versión de prueba que utilizaremos en este proyecto, requieren una clave de licencia.

En esta sección, aprenderás a instalar y configurar Kendo UI en la aplicación Angular desde la carpeta `ch01` de la sección Requisitos técnicos:

1. Ejecuta el siguiente comando para instalar la biblioteca Zone.js:

```bash
npm install zone.js
```

Las aplicaciones de Angular CLI son sin zonas (*zoneless*) de forma predeterminada a partir de Angular 21 en adelante, pero Kendo UI aún no es totalmente compatible con zoneless. Por lo tanto, debemos instalarla por separado como una dependencia con el comando anterior.

2. Abre el archivo `angular.json` y agrega una sección `polyfills` en las opciones de compilación de la sección `architect`:

```json
"polyfills": [
  "zone.js"
]
```

Usamos la sección de polyfills en la configuración de Angular para incluir características que no son compatibles de forma nativa con los navegadores modernos.

3. Abre el archivo `app.config.ts` e importa los métodos `provideZoneChangeDetection` y `provideAnimationsAsync`:

```typescript
import { ApplicationConfig, provideBrowserGlobalErrorListeners, provideZoneChangeDetection } from '@angular/core';
import { provideAnimationsAsync } from '@angular/platform-browser/animations/async';
```

La biblioteca Kendo UI requiere animaciones para algunos componentes, como los menús desplegables. El método `provideAnimationsAsync` habilita las animaciones globalmente en la aplicación.

Las animaciones integradas de Angular están obsoletas desde Angular 20.2. Sin embargo, algunas bibliotecas, como Kendo UI, todavía las requieren para funcionar correctamente. Las versiones futuras de Kendo UI eventualmente necesitarán eliminarlas también para mantenerse al día con las características web modernas.

4. Agrega los métodos anteriores al array `providers` de la configuración de la aplicación:

```typescript
providers: [
  provideBrowserGlobalErrorListeners(),
  provideRouter(routes),
  provideZoneChangeDetection(),
  provideAnimationsAsync()
]
```

Comenzaremos a usar la biblioteca Kendo UI creando el encabezado de la aplicación:

1. Ejecuta el siguiente comando para instalar el paquete de navegación de Kendo UI:

```bash
ng add @progress/kendo-angular-navigation
```

La biblioteca Kendo UI consta de muchos componentes individuales que residen dentro de paquetes (*bundles*), y debemos instalarlos por separado. En el comando anterior, instalamos el paquete de navegación que contiene un componente para el encabezado de la aplicación.

Si tienes problemas para ejecutar el comando anterior, consulta el archivo `CHANGELOG.md` del repositorio de GitHub del capítulo. Para acceder al enlace del repositorio, sigue los pasos de la sección "Download the example code files" en el Prefacio.

2. Abre el archivo `app.html` y reemplaza su contenido con el siguiente código HTML:

```html
<chapter-title [chapterTitle]="title()" />
<router-outlet />
```

3. Abre el archivo `app.ts` y crea el siguiente constructor para establecer el título del capítulo:

```typescript
constructor() {
  this.chapterTitleService.setTitle(
    'Chapter 8: SmartFactory Shifts'
  );
}
```

4. Abre el archivo `chapter-title.component.ts` y agrega la siguiente sentencia de importación:

```typescript
import { KENDO_APPBAR } from '@progress/kendo-angular-navigation';
```

La variable `KENDO_APPBAR` contiene el componente que utilizaremos para crear una barra de aplicación.

5. Reemplaza el valor de la propiedad `template` en el decorador del componente con el siguiente fragmento HTML:

```html
<kendo-appbar>
  <kendo-appbar-section>
    <h1 class="title">{{ chapterTitle() }}</h1>
  </kendo-appbar-section>
</kendo-appbar>
```

La barra de aplicación utiliza el elemento `kendo-appbar-section` para mostrar un título. Los componentes de Kendo UI comienzan con el prefijo `kendo`.

6. Agrega la variable `KENDO_APPBAR` al array `imports`:

```typescript
imports: [KENDO_APPBAR]
```

7. Ejecuta la aplicación usando el comando `ng serve` y navega a `http://localhost:4200`:

> *Figura 8.1 – Encabezado de la aplicación*

En la imagen anterior, vemos el encabezado de la aplicación cuando navegamos a la página de inicio.

El diseño de la aplicación utiliza Kendo UI para Angular para visualizar la interfaz de usuario. Muestra un encabezado que presenta el título del capítulo. En la siguiente sección, aprenderemos a interactuar con más componentes de la biblioteca Kendo UI implementando una función para agregar nuevos empleados.

---

### Sección: Adición de Nuevos Empleados

Los gerentes de turno deben poder agregar nuevos empleados para que luego puedan asignarlos a un turno específico. Un empleado tendrá los siguientes campos:

- Nombre completo
- Edad
- Puesto de trabajo (*job*)

La elección de los campos anteriores se realizó para mantener la aplicación lo más simple posible.

La función de creación de empleados se puede dividir en las siguientes tareas:

- Configuración del almacenamiento
- Diseño del creador de empleados (*employee builder*)

Comenzaremos implementando una solución de almacenamiento para conservar los datos de los empleados.

#### Configuración del almacenamiento

Utilizaremos el almacenamiento local (*local storage*) del navegador para guardar los datos de los empleados. Necesitaremos un servicio de Angular que actúe como envoltorio alrededor del almacenamiento local:

1. Primero, ejecuta el siguiente comando para crear una interfaz para los objetos de los empleados:

```bash
ng generate interface employee
```

2. Agrega las siguientes propiedades a la interfaz `Employee`:

```typescript
name: string;
age: number;
job: string;
```

3. Ahora, crea el servicio usando el siguiente comando:

```bash
ng generate service employees
```

4. Abre el archivo `employees.ts` e importa el token de inyección `DOCUMENT` y el método `inject`:

```typescript
import { Service, DOCUMENT, inject } from '@angular/core';
```

`DOCUMENT` nos da acceso al objeto global de documento a través del sistema de inyección de dependencias de Angular.

5. Agrega otra sentencia de importación para usar la interfaz del paso 1:

```typescript
import { Employee } from './employee';
```

6. Crea la siguiente propiedad dentro de la clase del servicio `Employees`:

```typescript
private storage = inject(DOCUMENT).defaultView!.localStorage;
```

En el código anterior, recuperamos el objeto de almacenamiento local de la vista predeterminada del documento actual.

7. Agrega un método para crear un nuevo empleado:

```typescript
create(employee: Employee) {
  const employees = this.storage.getItem('sfs') ?? '[]';
  this.storage.setItem('sfs', JSON.stringify(
    [...JSON.parse(employees), employee])
  );
}
```

El método `create` lee los empleados existentes del almacenamiento local como un valor de cadena y añade los nuevos datos del empleado convirtiéndolos en una cadena mediante el método `stringify`.

Diseñaremos un formulario usando Kendo UI para ingresar datos de empleados por parte del usuario y guardarlos en el almacenamiento local utilizando el servicio anterior. Se podrá acceder al formulario mediante un enlace desde el encabezado de la aplicación:

1. Abre el archivo `chapter-title.component.ts` y agrega la siguiente sentencia de importación:

```typescript
import { RouterLink } from '@angular/router';
```

Utilizaremos el router de Angular para navegar al componente del formulario de empleados.

2. Agrega una nueva sección de barra de aplicación en la plantilla del componente con el siguiente código:

```html
<kendo-appbar-section>
  <ul>
    <li>
      <a routerLink="employees">Employees</a>
    </li>
  </ul>
</kendo-appbar-section>
```

La lista desordenada contendrá un enlace para cada funcionalidad de la aplicación. Actualmente, contiene un elemento de anclaje que apunta a la ruta `employees` que definiremos más adelante.

3. Agrega la clase `RouterLink` al array `imports`:

```typescript
imports: [KENDO_APPBAR, RouterLink]
```

4. Da estilo al componente agregando la siguiente propiedad al decorador:

```scss
styles: `
  ul {
    list-style-type: none;
  }
  a {
    font-size: 1rem;
    text-decoration: none;
    color: navy;
    &:hover {
      color: gray;
    }
  }
`
```

Los estilos CSS anteriores eliminan el estilo predeterminado de la lista desordenada y del elemento de anclaje.

5. Ejecuta el siguiente comando para crear un nuevo componente para la creación de empleados:

```bash
ng generate component employee-builder
```

6. Abre el archivo `app.routes.ts` e importa la clase `EmployeeBuilder`:

```typescript
import { EmployeeBuilder } from './employee-builder/employee-builder';
```

7. Agrega una nueva ruta para activar el componente:

```typescript
export const routes: Routes = [
  { path: 'employees', component: EmployeeBuilder }
];
```

8. Ejecuta la aplicación, haz clic en el enlace Employees y deberías ver la siguiente salida en la página: `employee-builder works!`.

La aplicación proporciona a los usuarios acceso al componente creador de empleados. En la siguiente sección, crearemos la interfaz de usuario del creador de empleados.

#### Diseño del creador de empleados

Utilizaremos la biblioteca Kendo UI para enriquecer el componente con campos de entrada para ingresar los datos requeridos de un empleado:

1. Abre el archivo `employee-builder.ts` e importa el método `model` y la clase `FormsModule`:

```typescript
import { Component, model } from '@angular/core';
import { FormsModule } from '@angular/forms';
```

2. Agrega la clase `FormsModule` al array `imports` del decorador del componente:

```typescript
imports: [FormsModule]
```

3. Crea las siguientes propiedades en la clase del componente `EmployeeBuilder`:

```typescript
readonly name = model('');
readonly age = model<number | undefined>(undefined);
readonly job = model('');
```

4. Instala los paquetes de entradas y etiquetas de Kendo UI con los siguientes comandos:

```bash
ng add @progress/kendo-angular-inputs
ng add @progress/kendo-angular-label
```

Usaremos un componente de etiqueta para cada entrada que agreguemos al componente.

Si tienes problemas para ejecutar los comandos anteriores, consulta el archivo `CHANGELOG.md` del repositorio de GitHub del capítulo. Para acceder al enlace del repositorio, sigue los pasos de la sección "Download the example code files" en el Prefacio.

5. Importa los paquetes anteriores:

```typescript
import { KENDO_INPUTS } from '@progress/kendo-angular-inputs';
import { KENDO_LABEL } from '@progress/kendo-angular-label';
```

6. Agrega las clases de los paquetes al array `imports` del decorador del componente:

```typescript
imports: [KENDO_INPUTS, KENDO_LABEL]
```

7. Abre el archivo `employee-builder.html` y reemplaza su contenido con el siguiente código HTML:

```html
<div class="container">
  <kendo-label text="Full Name">
    <kendo-textbox [(ngModel)]="name" />
  </kendo-label>
</div>
```

El elemento `kendo-textbox` representa un campo de entrada. Lo rodeamos con un elemento `kendo-label` para mostrar una etiqueta encima de la entrada.

8. Agrega un nuevo elemento `div` para definir una entrada numérica para la edad del empleado:

```html
<div class="container">
  <kendo-label text="Age (18-65)">
    <kendo-numerictextbox format="n0" [autoCorrect]="true" [min]="18" [max]="65" [(ngModel)]="age" />
  </kendo-label>
</div>
```

Formateamos el cuadro de texto numérico para que no use puntos decimales y restringimos su valor de 18 a 65. La propiedad `autoCorrect` le indica a Kendo UI que corrija el valor automáticamente cuando el usuario ingresa un valor no válido, como caracteres o un valor que viola el rango aceptado.

Para el puesto de trabajo del empleado, utilizaremos un componente diferente a un cuadro de texto. El trabajo de un empleado define su especialidad en la fábrica, como electricista o fontanero, que es fija. Por lo tanto, usaremos un componente desplegable con valores fijos:

1. Instala el paquete desplegable con el siguiente comando:

```bash
ng add @progress/kendo-angular-dropdowns
```

Si tienes problemas para ejecutar el comando anterior, consulta el archivo `CHANGELOG.md` del repositorio de GitHub del capítulo. Para acceder al enlace del repositorio, sigue los pasos de la sección "Download the example code files" en el Prefacio.

2. Abre el archivo `employee-builder.ts` y agrega la siguiente sentencia de importación:

```typescript
import { KENDO_DROPDOWNS } from '@progress/kendo-angular-dropdowns';
```

La sentencia anterior importa el paquete que contiene un componente de lista desplegable.

3. Agrega la variable `KENDO_DROPDOWNS` al array `imports` del decorador del componente:

```typescript
imports: [
  FormsModule,
  KENDO_INPUTS,
  KENDO_LABEL,
  KENDO_DROPDOWNS
]
```

4. Abre el archivo `employee-builder.html` y agrega un combobox con una etiqueta debajo de la entrada de edad:

```html
<kendo-label text="Job">
  <kendo-combobox [data]="['Electrician', 'Technician', 'Plumber']" [(ngModel)]="job" />
</kendo-label>
```

5. Abre el archivo `employee-builder.scss` y agrega los siguientes estilos CSS:

```scss
.container {
  margin: 1rem;
  display: flex;
  gap: 2rem;
  & * {
    flex: 1;
  }
}
```

Los estilos anteriores mostrarán cada elemento div en un diseño flexbox y harán que sus elementos secundarios ocupen todo el espacio de manera uniforme.

6. Ejecuta la aplicación y navega a la página de empleados:

> *Figura 8.2 – Creador de empleados*

Como se muestra en la imagen anterior, el creador de empleados tiene todos los campos de entrada que corresponden a los datos de información del empleado. Agregaremos un botón al componente para recopilar los datos y guardarlos en el almacenamiento local:

1. Ejecuta el siguiente comando para instalar el paquete de botones de Kendo UI:

```bash
ng add @progress/kendo-angular-buttons
```

Si tienes problemas para ejecutar el comando anterior, consulta el archivo `CHANGELOG.md` del repositorio de GitHub del capítulo. Para acceder al enlace del repositorio, sigue los pasos de la sección "Download the example code files" en el Prefacio.

2. Abre el archivo `employee-builder.ts` e importa los métodos signal `inject` y `computed`:

```typescript
import { Component, model, computed, inject } from '@angular/core';
```

3. Importa el servicio `Employees` y la interfaz `Employee`:

```typescript
import { Employees } from '../employees';
import { Employee } from '../employee';
```

4. Importa el paquete de botones de la biblioteca Kendo UI:

```typescript
import { KENDO_BUTTONS } from '@progress/kendo-angular-buttons';
```

5. Agrega el paquete de botones al array `imports` del decorador del componente:

```typescript
imports: [
  FormsModule,
  KENDO_INPUTS,
  KENDO_LABEL,
  KENDO_DROPDOWNS,
  KENDO_BUTTONS
]
```

6. Inyecta el servicio `Employees` en la clase del componente `EmployeeBuilder`:

```typescript
private employeesService = inject(Employees);
```

7. Crea una propiedad `employee` de la siguiente manera:

```typescript
readonly employee = computed<Employee>(() => {
  return {
    name: this.name(),
    age: this.age()!,
    job: this.job()
  };
});
```

En el fragmento anterior, usamos el método `computed` para derivar el valor de la variable `employee` a partir de las propiedades respectivas.

8. Crea un método `save` para llamar al método `create` usando los detalles del empleado:

```typescript
save() {
  this.employeesService.create(this.employee());
}
```

9. Abre la plantilla del componente e inserta el siguiente fragmento para agregar un elemento de botón:

```html
<button kendoButton (click)="save()">Save</button>
```

El elemento de botón activa el método `save` cuando se hace clic en él.

El creador de empleados se comunica con el almacenamiento local del navegador y guarda la información de un nuevo empleado. Utilizaremos la información de los empleados para seleccionar empleados de una lista y agregarlos a un turno de fábrica. En la siguiente sección, construiremos un calendario para asignar empleados.

---

### Sección: Asignación de Empleados a un Turno

La mejor manera de visualizar los turnos de trabajo es utilizando una vista de calendario. El calendario puede representar el cronograma y los recursos de trabajo, como los empleados, de una manera conveniente para los gerentes.

La biblioteca Kendo UI contiene los siguientes componentes que proporcionan una vista de calendario:

- **Calendar:** Un calendario visual que permite a los usuarios seleccionar una sola fecha.
- **Scheduler:** Un calendario completamente equipado con estilo Outlook.

Para el propósito de nuestra aplicación, usaremos el scheduler porque necesitamos ver el tiempo y los empleados en una vista unificada. Crearemos un nuevo componente de Angular que alojará el componente scheduler:

1. Crea el componente usando el siguiente comando de Angular CLI:

```bash
ng generate component shifts
```

2. Abre el archivo `chapter-title.component.ts` y agrega un nuevo elemento de anclaje en el elemento de lista desordenada de la plantilla del componente:

```html
<ul>
  <li>
    <a routerLink="employees">Employees</a>
  </li>
  <li>
    <a routerLink="shifts">Shifts</a>
  </li>
</ul>
```

3. Agrega un nuevo estilo CSS a la propiedad de estilos del componente:

```scss
li {
  display: inline-block;
  margin: 0.5rem;
}
```

El estilo anterior alineará los elementos de la lista horizontalmente.

4. Importa el componente `Shifts` en el archivo `app.routes.ts`:

```typescript
import { Shifts } from './shifts/shifts';
```

5. Agrega una nueva ruta en la propiedad `routes` para navegar al componente:

```typescript
export const routes: Routes = [
  { path: 'employees', component: EmployeeBuilder },
  { path: 'shifts', component: Shifts }
];
```

6. Ejecuta la aplicación con el comando `ng serve` y navega a `http://localhost:4200`:

> *Figura 8.3 – Encabezado de la aplicación*

La imagen anterior muestra el nuevo enlace Shifts en el encabezado de la aplicación.

7. Haz clic en el nuevo enlace y verifica que la página muestre la siguiente salida: `shifts works!`.

El componente de turnos es la característica central de la aplicación y funcionará de acuerdo con las siguientes especificaciones:

- Los gerentes pueden organizar los turnos hasta con una semana de anticipación.
- Los horarios de trabajo de la fábrica son de 8:00 AM a 4:00 PM.
- La fábrica está cerrada los fines de semana.

Primero, agregaremos un componente scheduler de Kendo UI a la página de turnos:

1. Instala el componente scheduler usando el siguiente comando:

```bash
ng add @progress/kendo-angular-scheduler
```

Si tienes problemas para ejecutar el comando anterior, consulta el archivo `CHANGELOG.md` del repositorio de GitHub del capítulo. Para acceder al enlace del repositorio, sigue los pasos de la sección "Download the example code files" en el Prefacio.

2. Abre el archivo `shifts.ts` y agrega las siguientes sentencias de importación:

```typescript
import { KENDO_SCHEDULER } from '@progress/kendo-angular-scheduler';
import { Day } from '@progress/kendo-date-math';
```

`KENDO_SCHEDULER` contiene todos los componentes necesarios para trabajar con un componente scheduler en Kendo UI. La enumeración `Day` representa los días de la semana.

3. Agrega el paquete del scheduler al array `imports` del decorador del componente:

```typescript
imports: [KENDO_SCHEDULER]
```

4. Crea la siguiente variable en la clase del componente `Shifts`:

```typescript
hiddenDays = [Day.Saturday, Day.Sunday];
```

La variable `hiddenDays` define una lista de días que utilizaremos para ocultar del componente scheduler porque los empleados no trabajarán los fines de semana.

5. Abre el archivo `shifts.html` y reemplaza su contenido con el siguiente código HTML:

```html
<kendo-scheduler>
  <kendo-scheduler-day-view />
  <kendo-scheduler-week-view />
</kendo-scheduler>
```

El componente scheduler puede mostrar datos en diferentes vistas, como mes, semana y año. Cuando queramos usarlo, debemos pasar al menos una vista. En el fragmento anterior, configuramos el scheduler para mostrar la vista del día actual y de la semana para que los gerentes puedan ver los turnos hasta el final de la semana.

6. Modifica el elemento `kendo-scheduler` agregando las siguientes propiedades y atributos:

```html
[showWorkHours]="true"
[allDaySlot]="false"
workDayStart="08:00"
workDayEnd="16:00"
[weekStart]="1"
[hiddenDays]="hiddenDays"
```

Las propiedades `workDayStart`, `workDayEnd` y `showWorkHours` configuran el scheduler para mostrar solo las horas de trabajo de la fábrica. La propiedad `allDaySlot` evita que el scheduler asigne a un empleado a un turno de día completo. `weekStart` configura el scheduler para que comience desde el lunes, y `hiddenDays` ocultará el sábado y el domingo.

7. Ejecuta la aplicación y navega a la página de turnos:

> *Figura 8.4 – Vista de día del scheduler*

La imagen anterior muestra un componente scheduler que abarca de 8:00 AM a 4:00 PM para el día actual. La línea alrededor de la 1:00 PM define la hora actual. El componente también tiene una barra de herramientas para alternar entre las vistas de día y semana.

8. Haz clic en el botón Week para cambiar a la vista semanal:

> *Figura 8.5 – Vista semanal del scheduler*

Como se muestra en la imagen anterior, el diseño del componente scheduler se ajusta a los horarios y días laborables de la fábrica.

Haremos que el scheduler sea interactivo permitiendo a los gerentes agregar un empleado a una franja horaria específica. El componente scheduler puede crear eventos para definir un turno para un empleado en particular:

1. Abre el archivo `shifts.ts` y agrega la siguiente sentencia de importación:

```typescript
import { FormControl, FormGroup, ReactiveFormsModule } from '@angular/forms';
```

Conectaremos el componente scheduler con un formulario de Angular para crear un nuevo turno cuando los usuarios hagan clic en el componente. El componente scheduler actualmente solo admite el módulo de formularios reactivos del framework Angular.

2. Agrega la clase `ReactiveFormsModule` en el array `imports` del decorador del componente:

```typescript
imports: [KENDO_SCHEDULER, ReactiveFormsModule]
```

3. Crea las siguientes propiedades en la clase del componente `Shifts`:

```typescript
shifts = [];
form!: FormGroup;
```

La propiedad `shifts` almacenará una lista de todos los turnos de trabajo que crea el componente scheduler, y la propiedad `form` será un formulario reactivo que agregará un empleado a un turno específico.

4. Agrega un nuevo método con el siguiente contenido:

```typescript
createShift(args: CreateFormGroupArgs) {
  const dataItem = args.dataItem;
  this.form = new FormGroup({
    title: new FormControl(''),
    start: new FormControl(dataItem.start),
    end: new FormControl(dataItem.end)
  });
  return this.form;
}
```

El método anterior crea un formulario reactivo que contiene las propiedades básicas para un evento del scheduler. El evento precompleta la duración de la propiedad `dataItem` que se establece cuando el usuario selecciona una franja horaria específica en el scheduler.

5. Importa el tipo `CreateFormGroupArgs` del paquete `@progress/kendo-angular-scheduler`.
6. Vincula la clase del componente `Shifts` con el formulario mediante un constructor:

```typescript
constructor() {
  this.createShift = this.createShift.bind(this);
}
```

7. Abre el archivo `shifts.html` y agrega las siguientes propiedades al elemento `kendo-scheduler`:

```html
[kendoSchedulerBinding]="shifts"
[kendoSchedulerReactiveEditing]="createShift"
```

La primera propiedad vincula el componente scheduler a la propiedad `shifts` y guarda nuevos eventos en el array. La segunda define el formulario que el scheduler debe usar para crear nuevos eventos.

8. Ejecuta la aplicación y realiza un doble clic en una franja horaria específica en el scheduler:

> *Figura 8.6 – Formulario de nuevo evento*

La imagen anterior muestra el formulario para crear un nuevo evento de 11:00 AM a 11:30 AM.

9. Agrega un título en el campo Title y haz clic en el botón Save:

> *Figura 8.7 – Nuevo evento*

La imagen anterior muestra el nuevo evento con un espacio asignado en el scheduler.

El componente scheduler funciona como un calendario normal. Queremos conectar el nuevo formulario de eventos con los datos de los empleados para asignar un empleado a un evento en particular. Agregaremos una lista desplegable en el formulario de eventos para que el usuario pueda seleccionar un empleado:

1. Abre el archivo `employees.ts` y crea el siguiente método:

```typescript
getAll(): Employee[] {
  return JSON.parse(this.storage.getItem('sfs')!);
}
```

El método anterior obtiene los datos de los empleados del almacenamiento local del navegador.

2. Importa el método `inject` y la clase `Employees` en el archivo `shifts.ts`:

```typescript
import { Component, inject } from '@angular/core';
import { Employees } from '../employees';
```

3. Declara la siguiente propiedad en la clase del componente `Shifts`:

```typescript
resources = [
  {
    name: 'Employee',
    field: 'employee',
    valueField: 'name',
    textField: 'name',
    data: inject(Employees).getAll()
  }
];
```

La variable `resources` declara una lista de objetos de recursos que el componente scheduler puede asignar a un evento:

- `name`: La etiqueta de la lista desplegable.
- `field`: El nombre del control de formulario que usaremos más adelante para los datos del empleado.
- `valueField`: La propiedad del empleado que estableceremos en la propiedad `field`.
- `textField`: El texto de cada elemento de la lista desplegable.
- `data`: Los datos de los empleados para rellenar la lista desplegable.

Puedes tener más de un tipo de recurso dentro del array, como salas.

4. Agrega un nuevo control de formulario a la propiedad `form` con el mismo nombre que usaste en la propiedad `field` del recurso:

```typescript
this.form = new FormGroup({
  title: new FormControl(''),
  start: new FormControl(dataItem.start),
  end: new FormControl(dataItem.end),
  employee: new FormControl('')
});
```

5. Agrega la propiedad `resources` en el componente scheduler del archivo `shifts.html` de la siguiente manera:

```html
[resources]="resources"
```

6. Guarda todos los cambios, espera a que se recargue la aplicación y selecciona una franja horaria del scheduler:

> *Figura 8.8 – Formulario de nuevo evento con datos de empleados*

El formulario de eventos en la imagen anterior contiene una lista desplegable que muestra la lista de nuestros empleados.

Los gerentes de turno tienen una descripción general de alto nivel de todos los turnos en una fábrica utilizando el scheduler. Pueden asignar a un empleado para que trabaje en un período de tiempo específico usando el formulario de eventos y explorar los turnos en vistas de día y semana. En la siguiente sección, mejoraremos la experiencia de usuario de la aplicación permitiendo a los gerentes asignar empleados más rápido arrastrándolos directamente desde una lista al componente de calendario.

---

### Sección: Movimiento de Empleados Entre Turnos

El flujo de trabajo para asignar un empleado a una duración de turno específica consta de los siguientes pasos:

1. Seleccionar una franja horaria específica del componente scheduler.
2. Modificar la duración del tiempo según sea necesario.
3. Seleccionar un empleado de la lista.
4. Hacer clic en el botón Save en el formulario de eventos.

Mejoraremos el flujo de trabajo permitiendo a los gerentes de turno arrastrar al empleado de una lista y soltarlo en la franja horaria específica. El nuevo formulario de evento se abrirá automáticamente, con el campo Employee completado automáticamente. Usaremos un componente de vista de lista de la biblioteca Kendo UI para mostrar la lista de empleados:

1. Ejecuta el siguiente comando para instalar los paquetes de listview y layout:

```bash
ng add @progress/kendo-angular-listview
ng add @progress/kendo-angular-layout
```

El paquete `@progress/kendo-angular-layout` contiene componentes de diseño, incluida una tarjeta que usaremos para mostrar al empleado en una lista.

Si tienes problemas para ejecutar los comandos anteriores, consulta el archivo `CHANGELOG.md` del repositorio de GitHub del capítulo. Para acceder al enlace del repositorio, sigue los pasos de la sección "Download the example code files" en el Prefacio.

2. Abre el archivo `shifts.ts` e importa el paquete listview, el paquete layout y el método `signal`:

```typescript
import { Component, inject, signal } from '@angular/core';
import { KENDO_LISTVIEW } from '@progress/kendo-angular-listview';
import { KENDO_LAYOUT } from '@progress/kendo-angular-layout';
```

3. Agrega los paquetes al array `imports` del decorador del componente:

```typescript
imports: [
  KENDO_SCHEDULER,
  ReactiveFormsModule,
  KENDO_LISTVIEW,
  KENDO_LAYOUT
]
```

4. Declara la siguiente variable que devuelve los datos de los empleados:

```typescript
employees = signal(inject(Employees).getAll());
```

5. Refactoriza la propiedad `resources` para que use la nueva variable:

```typescript
resources = [
  {
    name: 'Employee',
    field: 'employee',
    valueField: 'name',
    textField: 'name',
    data: this.employees()
  }
];
```

6. Abre el archivo `shifts.html` y rodea el elemento `kendo-scheduler` con un elemento `div` que tenga una clase `container`.
7. Agrega el siguiente fragmento HTML encima del componente scheduler:

```html
<kendo-listview [data]="employees()">
  <ng-template kendoListViewItemTemplate let-employee="dataItem">
    <kendo-card width="200px">
      <kendo-card-header>
        <h1 kendoCardTitle>{{ employee.name }}</h1>
        <p kendoCardSubtitle>{{ employee.job }}</p>
      </kendo-card-header>
    </kendo-card>
  </ng-template>
</kendo-listview>
```

En el código anterior, usamos un componente listview para mostrar datos de la variable `employees`. El atributo `kendoListViewItemTemplate` define el contenido de cada elemento en la vista de lista. Usamos un componente card de 200 px de ancho para mostrar el nombre y el puesto de trabajo de cada empleado.

8. Abre el archivo `shifts.scss` y agrega los siguientes estilos:

```scss
.container {
  display: flex;
}

kendo-listview {
  margin-right: 1rem;
  background: transparent;
}

kendo-card {
  margin-top: 2rem;
}
```

Los estilos CSS anteriores posicionan el scheduler y el listview en una sola fila, uno al lado del otro.

9. Ejecuta la aplicación y navega a la página de turnos:

> *Figura 8.9 – Lista de empleados*

La página de turnos debe mostrar la lista de empleados en el lado izquierdo de la salida de la aplicación, como se muestra en la imagen anterior.

La biblioteca Kendo UI contiene una práctica colección de utilidades para enriquecer nuestras aplicaciones con capacidades de arrastrar y soltar. Las utilizaremos para conectar el formulario de eventos del componente scheduler con la lista de empleados:

1. Importa el paquete de arrastrar y soltar en el archivo `shifts.ts`:

```typescript
import { KENDO_DRAGANDDROP } from '@progress/kendo-angular-utils';
```

2. Agrega el paquete al array `imports` del decorador del componente:

```typescript
imports: [
  KENDO_SCHEDULER,
  ReactiveFormsModule,
  KENDO_LISTVIEW,
  KENDO_LAYOUT,
  KENDO_DRAGANDDROP
]
```

3. Declara una propiedad en la clase del componente `Shifts` para almacenar el nombre del empleado seleccionado de la lista:

```typescript
selected = signal('');
```

4. Abre el archivo `shifts.html` y agrega los siguientes atributos al elemento `kendo-listview`:

```html
kendoDragTargetContainer dragTargetFilter=".k-listview-item"
```

El fragmento anterior define que podemos arrastrar elementos desde el componente listview.

5. Agrega el siguiente atributo al elemento `kendo-scheduler`:

```html
kendoDropTargetContainer
```

El atributo anterior define que podemos soltar elementos en el componente scheduler.

6. Agrega el siguiente fragmento HTML debajo del componente scheduler:

```html
<ng-template #info>
  <p class="info">{{ selected() }}</p>
</ng-template>
```

La plantilla HTML anterior mostrará la información del empleado seleccionado mientras arrastramos al empleado desde el componente listview.

7. Conecta la plantilla `info` con el componente listview agregando las siguientes vinculaciones al elemento `kendo-listview`:

```html
[hint]="{ hintTemplate: info }"
(onDragStart)="
  selected.set($event.dragTarget.outerText.split('\n')[0])
"
```

La vinculación de la propiedad `hint` define la referencia de plantilla que corresponde a la información del empleado seleccionado del paso anterior. La vinculación de evento `onDragStart` establece el nombre del empleado seleccionado cuando el usuario comienza a arrastrar a un empleado de la lista.

8. Abre el archivo `shifts.scss` para agregar estilos para la plantilla info:

```scss
.info {
  display: flex;
  width: 100px;
  height: 40px;
  padding: 4px 24px;
  align-items: center;
  border-radius: 4px;
  background-color: rgb(133, 133, 247);
}
```

El estilo CSS anterior mostrará el nombre del empleado dentro de un cuadro con esquinas redondeadas.

Para abrir el formulario de evento cuando soltamos una tarjeta de empleado en el componente scheduler, debemos implementar la vinculación de evento `onDrop`:

1. Importa `SchedulerComponent`, `DropTargetEvent` y `viewChild` en el archivo `shifts.ts`:

```typescript
import { Component, inject, signal, viewChild } from '@angular/core';
import { CreateFormGroupArgs, KENDO_SCHEDULER, SchedulerComponent } from '@progress/kendo-angular-scheduler';
import { KENDO_DRAGANDDROP, DropTargetEvent } from '@progress/kendo-angular-utils';
```

2. Declara la siguiente variable en la clase del componente `Shifts`:

```typescript
private readonly scheduler = viewChild(SchedulerComponent);
```

La variable anterior nos dará acceso a la instancia del componente scheduler.

3. Crea un método reutilizable para construir el formulario reactivo para nuevos eventos:

```typescript
private buildForm(start: Date, end: Date, name: string) {
  this.form = new FormGroup({
    title: new FormControl(name),
    start: new FormControl(start),
    end: new FormControl(end),
    employee: new FormControl(name)
  });
}
```

En el método anterior, construimos el título del evento a partir del parámetro del nombre del empleado.

4. Refactoriza el método `createShift` en consecuencia:

```typescript
createShift(args: CreateFormGroupArgs) {
  const dataItem = args.dataItem;
  this.buildForm(dataItem.start, dataItem.end, '');
  return this.form;
}
```

5. Crea el método `addShift` que abre el nuevo formulario de eventos según la fecha seleccionada:

```typescript
addShift(evt: DropTargetEvent) {
  const start = new Date(
    evt.dropTarget.getAttribute('date')!
  );
  const end = new Date(start.getTime() + 30 * 60000);
  this.buildForm(start, end, this.selected());
  this.scheduler()?.addEvent(this.form);
}
```

El fragmento anterior obtiene el valor de fecha del objetivo donde se soltó el elemento que definiremos en el siguiente paso. Calcula la fecha de finalización, construye el formulario reactivo y llama al método `addEvent` del scheduler para abrir el formulario.

6. Abre el archivo `shifts.html` y agrega el siguiente fragmento HTML dentro del elemento `kendo-scheduler`:

```html
<ng-template kendoSchedulerTimeSlotTemplate let-date="date">
  <div class="drop-zone" [attr.date]="date"></div>
</ng-template>
```

El HTML anterior define el elemento HTML de destino donde podemos soltar las tarjetas de los empleados después de arrastrarlas desde la lista de empleados. El atributo `date` contendrá la fecha seleccionada del scheduler.

7. Agrega los siguientes atributos al elemento `kendo-scheduler`:

```html
dropTargetFilter=".drop-zone" (onDrop)="addShift($event)"
```

El primer atributo define el elemento de la zona de colocación y la vinculación de evento llama al método `addShift` cuando soltamos la tarjeta de empleado dentro de la zona.

8. Agrega un estilo CSS para la zona de colocación dentro del archivo `shifts.scss`:

```scss
.drop-zone {
  height: 100%;
}
```

La aplicación permite a los gerentes de turno asignar empleados rápidamente a un turno en particular. Utiliza la función de arrastrar y soltar de Kendo UI para asignar empleados de una lista a un evento particular en el componente de calendario.

---

### Sección: Resumen

En este capítulo, aprendimos cómo construir un calendario interactivo para gerentes de turnos de fábrica utilizando la biblioteca Kendo UI. Exploramos cómo usar los componentes de navegación y crear el encabezado principal de la aplicación con enlaces de menú. Creamos un servicio que se comunica con el almacenamiento local del navegador para conservar los datos. Aprendimos a obtener datos del almacenamiento y presentarlos en una vista de lista. Finalmente, exploramos cómo programar turnos de trabajo utilizando el componente scheduler. En el próximo capítulo, construiremos una aplicación de Punto de Venta (POS) para el sector minorista.

---

### Sección: Ejercicios

Agrega un color para cada nuevo empleado y utilízalo en la lista de empleados y en el evento del scheduler para diferenciar a los distintos trabajadores de turno en el scheduler.

---

### Sección: Lecturas Complementarias

- **Kendo UI para Angular:** [https://www.telerik.com/kendo-angular-ui](https://www.telerik.com/kendo-angular-ui)
- **Acceso a document en Angular:** [https://angular.dev/guide/ssr#accessing-document-via-di](https://angular.dev/guide/ssr#accessing-document-via-di)
- **Local storage:** [https://developer.mozilla.org/docs/Web/API/Window/localStorage](https://developer.mozilla.org/docs/Web/API/Window/localStorage)
