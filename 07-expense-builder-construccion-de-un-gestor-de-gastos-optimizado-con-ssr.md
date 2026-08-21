# Parte 1: Primeros Pasos con Angular e Inteligencia Artificial

## Capítulo 7: Expense Builder: Construcción de un Gestor de Gastos Optimizado con SSR

### Sección: Introducción

A medida que los desarrolladores agregan más funciones a las aplicaciones web, el tamaño de los paquetes (*bundles*) aumenta, lo que dificulta su carga por parte de los navegadores. Para superar el cuello de botella del rendimiento, nuestro objetivo es minimizar el código JavaScript final mediante el uso de renderizado del lado del servidor (*Server-Side Rendering* o SSR). Una aplicación habilitada para SSR carga parte del código en el servidor, minimizando el análisis y el procesamiento del lado del cliente. En algunos escenarios, podemos ejecutar aplicaciones SSR en el cliente sin JavaScript, una técnica conocida como Generación de Sitios Estáticos (*Static Site Generation* o SSG). SSG crea una versión HTML estática de una aplicación SSR que no requiere JavaScript para ejecutarse porque todas las dependencias se agregan en línea al HTML.

En este capítulo, aprenderemos a utilizar SSR en aplicaciones Angular mediante la creación de un gestor de gastos para un edificio con apartamentos independientes. Utilizaremos formularios de Angular con Angular Material para estructurar experiencias de interfaz de usuario dinámicas.

Vamos a cubrir los siguientes temas:

- Instalación de Angular Material
- Instalación de Angular SSR
- Visualización de la información del edificio
- Reporte de gastos

---

### Sección: Requisitos Técnicos

Puedes descargar el proyecto de ejemplo y el código de este libro siguiendo las instrucciones de la sección *Download the example code files* en el Prefacio de este libro.

Los archivos de código de este capítulo están incluidos en el paquete de código descargable.

---

### Sección: Instalación de Angular Material

La biblioteca Angular Material es una biblioteca de componentes de interfaz de usuario administrada por el equipo de Angular. Ofrece un amplio conjunto de componentes de interfaz de usuario, incluidos campos de formulario altamente personalizables para diseñar formularios.

Como hacemos en todos los capítulos de este libro, utilizaremos la aplicación del Capítulo 1, *Angular AI Kick-Starter: Scaffolding Smart Apps with Copilot*, como nuestro punto de partida y agregaremos un componente de encabezado de Angular Material para mostrar el título del capítulo.

Cubrimos el proceso de instalación de Angular Material en la sección *Instalación de Angular Material* del Capítulo 3, *EasyMenu: Creating a Table Order Management App*.

Los ejemplos de este capítulo están construidos utilizando la paleta de colores Magenta/Violet de la biblioteca Angular Material.

Modificaremos el componente de encabezado de la aplicación para adaptarlo a las necesidades del proyecto actual:

1. Abre el archivo `app.ts` y modifica el constructor de la siguiente manera:

```typescript
constructor() {
  this.chapterTitleService.setTitle(
    'Chapter 7: Expense Builder'
  );
}
```

2. Abre el archivo `app.html` y agrega el componente de salida del enrutador de la biblioteca de enrutador de Angular:

```html
<chapter-title [chapterTitle]="title()" />
<router-outlet />
```

3. Ejecuta la aplicación usando el comando `ng serve`, navega a `http://localhost:4200` y verifica que la salida sea la siguiente: `Chapter 7: Expense Builder`.
4. Abre el archivo `chapter-title.component.ts` y agrega las siguientes sentencias de importación:

```typescript
import { MatIconButton } from '@angular/material/button';
import { MatIcon } from '@angular/material/icon';
import { MatTooltip } from '@angular/material/tooltip';
```

Utilizaremos las clases anteriores para agregar un botón de acción con un icono en el encabezado de la aplicación. También incluirá un tooltip que proporciona contexto adicional sobre la acción.

5. Agrega el siguiente fragmento HTML debajo del elemento `<span>` en la propiedad `template` del decorador del componente:

```html
<a matIconButton matTooltip="Expenses">
  <mat-icon>apartment</mat-icon>
</a>
```

El fragmento anterior crea un botón con un icono y un tooltip adjunto.

6. Agrega las clases del paso 4 al array `imports`:

```typescript
imports: [MatToolbar, MatIconButton, MatIcon, MatTooltip]
```

7. Agrega la siguiente propiedad al decorador:

```typescript
styles: 'span { flex: 1 1 auto; }'
```

El estilo CSS anterior posicionará el botón de acción al final del encabezado de la aplicación.

8. Deberías ver la siguiente salida cuando la aplicación haya terminado de recargarse:

> *Figura 7.1 – Encabezado de la aplicación*

La imagen anterior muestra el encabezado de la aplicación, que consta del título de la aplicación y el botón de acción.

La biblioteca Angular Material proporciona una colección de componentes de alta calidad que podemos usar y personalizar según nuestras necesidades. Cada componente puede sobrescribir los estilos integrados de la paleta de colores seleccionada para que coincida con el color específico de tu marca. Por ejemplo, si agregamos el siguiente fragmento al selector `html` del archivo `styles.scss`, nos dará un encabezado con un fondo de color violeta con texto blanco y un icono blanco:

```scss
@include mat.toolbar-overrides((
  container-background-color: rgb(129, 0, 129),
  container-text-color: white
));
@include mat.icon-overrides((
  color: white
));
```

En la siguiente sección, aprenderemos a habilitar SSR en la aplicación y veremos la mejora de rendimiento al ejecutarla.

---

### Sección: Instalación de Angular SSR

El renderizado del lado del servidor afecta el rendimiento de la aplicación de las siguientes maneras:

- Acelera la carga de la aplicación renderizándola en el servidor y reduciendo el contenido entregado al cliente. El servidor entrega el HTML inicial al cliente, que se puede analizar y renderizar mientras se descarga el JavaScript.
- Mejora las métricas de Core Web Vitals (CWV) relacionadas con la velocidad de carga y la estabilidad de la interfaz de usuario.

Angular SSR es un conjunto de patrones de optimización de rendimiento para aplicaciones Angular, implementados a través del paquete `@angular/ssr`. Instala Angular SSR ejecutando el siguiente comando:

```bash
ng add @angular/ssr
```

El comando anterior descarga e instala todos los paquetes requeridos y crea los siguientes archivos:

- `main.server.ts`: Inicializa la aplicación en el servidor utilizando una configuración específica.
- `app.config.server.ts`: Contiene la configuración de la aplicación SSR que consiste en una versión combinada de los archivos de configuración de la aplicación cliente y servidor.
- `server.ts`: Configura e inicia un servidor Node.js Express que renderiza la aplicación Angular en el servidor.
- `app.routes.server.ts`: Define la configuración de enrutamiento para la aplicación SSR.

También realiza las siguientes modificaciones para compilar y ejecutar la aplicación en el servidor:

- Agrega las opciones necesarias en la sección de compilación del archivo `angular.json` para que la CLI de Angular pueda reconocer la configuración de SSR y ejecutar la aplicación.
- Agrega `provideClientHydration` en el archivo `app.config.ts` para habilitar la hidratación en la aplicación Angular.

La hidratación es el proceso de restaurar la aplicación SSR en el cliente. Puedes encontrar más detalles en [https://angular.dev/guide/hydration](https://angular.dev/guide/hydration).

Ahora que hemos instalado Angular SSR en nuestra aplicación, veamos cómo usarlo. Crearemos un nuevo componente de Angular que será la página de inicio de nuestra aplicación:

1. Crea el nuevo componente usando el siguiente comando:

```bash
ng generate component welcome
```

2. Abre el archivo `welcome.html` y reemplaza su contenido con el siguiente código HTML:

```html
<h1>Welcome to Expense Builder</h1>
<p>
  An application that helps managers to track building and apartment expenses
</p>
```

El fragmento anterior muestra un mensaje de bienvenida al usuario.

3. Abre el archivo `app.routes.ts` e importa la clase `Welcome`:

```typescript
import { Welcome } from './welcome/welcome';
```

4. Agrega un nuevo objeto de definición de ruta en el array `routes`:

```typescript
export const routes: Routes = [
  { path: '', component: Welcome }
];
```

La aplicación activará el componente `Welcome` de forma predeterminada al iniciarse.

5. Abre el archivo `app.routes.server.ts` y agrega un nuevo objeto de definición de ruta en el array `serverRoutes`:

```typescript
export const serverRoutes: ServerRoute[] = [
  { path: '**', renderMode: RenderMode.Prerender },
  { path: '', renderMode: RenderMode.Prerender }
];
```

La nueva ruta le indica a Angular que prerenderice la ruta predeterminada que agregamos en el paso anterior. La prerenderización agregará la plantilla del componente en línea en el archivo `index.html` de la aplicación sin esperar a que el navegador analice y cargue el código JavaScript del componente.

6. Abre el archivo `app.html` y rodea el componente `<router-outlet>` en un elemento `div`:

```html
<div class="container">
  <router-outlet />
</div>
```

7. Abre el archivo `app.scss` y agrega el siguiente estilo CSS:

```scss
.container {
  margin: 1rem;
}
```

El estilo anterior agrega un margen alrededor del componente `<router-outlet>`.

Veremos cómo SSR afecta a una aplicación Angular en términos de rendimiento mediante la ejecución de un experimento:

1. Ejecuta la aplicación usando el comando `ng serve` y navega a `http://localhost:4200` usando Google Chrome.
2. Abre las herramientas de desarrollo, selecciona la pestaña Lighthouse y deberías ver lo siguiente:

> *Figura 7.2 – Pestaña Lighthouse*

Lighthouse es una herramienta para medir varios aspectos de rendimiento de una página web, incluidas las métricas de CWV. Google Chrome tiene una versión integrada de Lighthouse que podemos usar para comparar nuestra aplicación.

3. Selecciona la opción **Desktop** en la sección Device, marca solo la opción **Performance** en la sección Categories y haz clic en el botón **Analyze page load**. Google Chrome analizará la aplicación y mostrará una puntuación global:

> *Figura 7.3 – Informe de Lighthouse*

El informe en la imagen anterior muestra la puntuación de rendimiento general en las métricas de CWV para nuestra aplicación.

La puntuación general de rendimiento es una estimación y puede variar según las capacidades de tu computadora y las extensiones del navegador instaladas. Es preferible ejecutar la prueba en modo incógnito para simular un entorno más realista.

4. Abre una nueva ventana de terminal y compila la aplicación usando el siguiente comando:

```bash
ng build
```

5. Ejecuta el siguiente comando para ejecutar la aplicación en producción:

```bash
npm run serve:ssr:my-app
```

El comando anterior alojará la aplicación en un servidor Express Node.js en el puerto 4000.

6. Navega a `http://localhost:4000` y genera un nuevo informe usando Lighthouse:

> *Figura 7.4 – Informe de Lighthouse para SSR*

Como se muestra en la imagen anterior, la puntuación general ha mejorado con la prerenderización porque enviamos menos código JavaScript al cliente.

En la siguiente sección, comenzaremos a agregar nuevas características mostrando la información del edificio en la aplicación.

---

### Sección: Visualización de la Información del Edificio

La aplicación Expense Builder permite a los usuarios reportar los gastos operativos del edificio que administran. La información del edificio, como la dirección y el número de apartamentos, es fija. Almacenaremos esa información en un entorno virtual utilizando la biblioteca JSON-Server, que permite a los desarrolladores crear una API REST para creación de prototipos.

Instalaremos y configuraremos la biblioteca JSON-Server con los datos del edificio:

1. Instala la biblioteca usando el siguiente comando:

```bash
npm install json-server
```

El comando anterior instala el paquete `json-server`, que incluye todas las herramientas necesarias para utilizar la biblioteca.

2. Crea un archivo `db.json` en la carpeta raíz del proyecto Angular y agrega el siguiente contenido:

```json
{
  "$schema": "./node_modules/json-server/schema.json"
}
```

El archivo anterior representa una base de datos virtual para nuestros datos. La propiedad `$schema` especifica el esquema JSON de la biblioteca.

3. Agrega el siguiente fragmento debajo de la propiedad `$schema`:

```json
"buildings": [
  {
    "id": "1",
    "address": "Athens, Greece",
    "apartments": 10
  }
]
```

El array `buildings` contiene la lista de edificios de los que es responsable el usuario. Cada edificio tiene un identificador único, su dirección y el número de apartamentos.

4. Abre el archivo `package.json` y agrega la siguiente entrada en la propiedad `scripts`:

```json
"start:server": "json-server db.json"
```

El script anterior inicia el servidor JSON utilizando la base de datos virtual que creamos en el paso anterior.

5. Ejecuta el siguiente comando para iniciar el servidor:

```bash
npm run start:server
```

El comando anterior inicia un servidor web en el puerto 3000 y expone el array `buildings` como un punto final de API.

6. Navega a `http://localhost:3000/buildings`, y deberías ver la siguiente salida:

```json
[
  {
    "id": "1",
    "address": "Athens, Greece",
    "apartments": 10
  }
]
```

7. Ahora intenta navegar a `http://localhost:3000/buildings/1`, y deberías ver el objeto del único registro en la base de datos:

```json
{
  "id": "1",
  "address": "Athens, Greece",
  "apartments": 10
}
```

JSON-Server genera métodos de puntos finales basados en el esquema de datos que proporcionas. Consultaremos los puntos finales desde nuestra aplicación para obtener datos:

1. Crearemos un nuevo servicio de Angular para acceder a los datos del edificio usando el siguiente comando:

```bash
ng generate service data
```

2. Abre el archivo `data.ts` e importa el cliente HTTP y el método `inject`:

```typescript
import { Service, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
```

3. Crea la siguiente interfaz encima del decorador `@Service`:

```typescript
interface Building {
  address: string;
  apartments: number;
}
```

La interfaz anterior define la estructura del edificio tal como está en la API.

4. Inyecta el cliente HTTP en la clase del servicio `Data`:

```typescript
private http = inject(HttpClient);
```

5. Crea el siguiente método:

```typescript
getBuilding(id: string) {
  return this.http.get<Building>(
    'http://localhost:3000/buildings/' + id
  );
}
```

El método `getBuilding` inicia una solicitud GET a la API para recuperar información de un ID de edificio específico.

Crearemos un nuevo componente para mostrar la información de un edificio específico en la página principal de la aplicación. Obtendremos la información del edificio utilizando el método `getBuilding` del servicio de datos y pasando el ID del edificio:

1. Primero, crea el componente usando el siguiente comando:

```bash
ng generate component expenses
```

2. Abre el archivo `expenses.ts` y modifica las sentencias de importación de la siguiente manera:

```typescript
import { Component, inject } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
import { toSignal } from '@angular/core/rxjs-interop';
import { Data } from '../data';
import { MatCardModule } from '@angular/material/card';
```

La clase `ActivatedRoute` proporciona acceso a la información sobre la ruta actualmente activa en la aplicación. El método `toSignal` nos permite convertir el observable del servicio Data en una signal. La clase `MatCardModule` contiene todos los componentes necesarios para mostrar datos en un diseño de tarjeta.

3. Agrega la clase `MatCardModule` al array `imports` del decorador del componente:

```typescript
imports: [MatCardModule]
```

4. Inyecta los servicios `ActivatedRoute` y `Data` en la clase del componente `Expenses`:

```typescript
private route = inject(ActivatedRoute);
private data = inject(Data);
```

5. Convierte el observable `paramMap` en una signal declarando la siguiente propiedad:

```typescript
params = toSignal(this.route.paramMap);
```

La propiedad `paramMap` de la ruta activada devuelve un observable que podemos usar para recuperar valores de parámetros de URL.

6. Declara la propiedad `building` que utiliza la propiedad `params` para extraer el ID del edificio y pasarlo al servicio de datos:

```typescript
building = toSignal(
  this.data.getBuilding(this.params()?.get('id')!)
);
```

En el fragmento anterior, convertimos el observable devuelto por el método `getBuilding` en una signal porque queremos usar signals para definir los datos del componente.

7. Abre el archivo `expenses.html` y reemplaza su contenido con el siguiente código HTML:

```html
<mat-card>
  <mat-card-header>
    <mat-card-title>
      {{ building()?.address }}
    </mat-card-title>
    <mat-card-subtitle>
      {{ building()?.apartments }} apartments
    </mat-card-subtitle>
  </mat-card-header>
  <mat-card-content></mat-card-content>
</mat-card>
```

Diseñamos la página de gastos utilizando el componente de tarjeta de Angular Material, que es el elemento `<mat-card>`. El elemento `<mat-card-header>` representa el encabezado de la tarjeta y muestra la dirección y el número total de apartamentos. El elemento `<mat-card-content>` es el área de contenido principal de la tarjeta, donde agregaremos los formularios de gastos más adelante.

El componente expenses puede obtener y mostrar información del edificio desde la API. Ajustaremos el encabezado de la aplicación para activar el componente al hacer clic en el botón Expenses:

1. Abre el archivo `app.routes.ts` e importa el componente:

```typescript
import { Expenses } from './expenses/expenses';
```

2. Agrega un nuevo objeto de definición de ruta en el array `routes`:

```typescript
{ path: ':id', component: Expenses }
```

La ruta anterior activa el componente `Expenses` cuando el usuario navega a una URL como `http://localhost:4200/1`, donde `1` es el ID del edificio.

3. Abre el archivo `chapter-title.component.ts` y agrega la siguiente sentencia de importación:

```typescript
import { RouterLink } from '@angular/router';
```

4. Agrega la clase `RouterLink` al array `imports` del decorador del componente:

```typescript
imports: [
  MatToolbar,
  MatIconButton,
  MatIcon,
  MatTooltip,
  RouterLink
]
```

5. Agrega la directiva `routerLink` al elemento de anclaje de la plantilla:

```html
<a matIconButton matTooltip="Expenses" routerLink="1">
  <mat-icon>apartment</mat-icon>
</a>
```

En el fragmento anterior, configuramos el enlace para vincular directamente al único edificio que existe en la base de datos. Un enfoque más dinámico sería permitir a los usuarios seleccionar el edificio de una lista.

6. Abre el archivo `app.routes.server.ts` y agrega la siguiente ruta del servidor:

```typescript
{ path: ':id', renderMode: RenderMode.Server }
```

El fragmento anterior renderizará el contenido de la ruta en el servidor.

7. Inicia la aplicación usando el comando `ng serve` y navega a `http://localhost:4200`.
8. Haz clic en el botón del encabezado de la aplicación y deberías ver la siguiente salida:

> *Figura 7.5 – Información del edificio*

La imagen anterior muestra la información del edificio en un diseño de tarjeta con Angular Material.

La tarjeta del edificio actualmente no contiene contenido. En la siguiente sección, agregaremos la interfaz de usuario requerida para reportar gastos.

---

### Sección: Reporte de Gastos

Los gastos totales de un edificio constan de los gastos estándar que se aplican globalmente al edificio y los gastos de cada apartamento. El reporte de gastos se realiza cada mes. Agregaremos un selector de fechas de la biblioteca Angular Material para que los usuarios puedan seleccionar la fecha del informe:

1. Abre el archivo `app.config.ts` y agrega la siguiente sentencia de importación:

```typescript
import { provideNativeDateAdapter } from '@angular/material/core';
```

El método `provideNativeDateAdapter` configura el selector de fechas para usar una implementación de fecha nativa y ajustes de configuración regional.

2. Agrega el método anterior al array `providers` de la configuración de la aplicación:

```typescript
export const appConfig: ApplicationConfig = {
  providers: [
    provideBrowserGlobalErrorListeners(),
    provideRouter(routes),
    provideClientHydration(withEventReplay()),
    provideNativeDateAdapter()
  ]
};
```

3. Abre el archivo `expenses.ts` y agrega las siguientes sentencias de importación:

```typescript
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatDatepickerModule } from '@angular/material/datepicker';
import { MatInput } from '@angular/material/input';
```

Necesitamos todas las clases anteriores del paquete `@angular/material` para usar el componente selector de fechas.

4. Agrega las clases al array `imports` del decorador del componente:

```typescript
imports: [
  MatCardModule,
  MatFormFieldModule,
  MatDatepickerModule,
  MatInput
]
```

5. Abre el archivo `expenses.html` y agrega el siguiente fragmento dentro del elemento `<mat-card-content>`:

```html
<mat-form-field>
  <mat-label>Choose a date</mat-label>
  <input matInput [matDatepicker]="picker">
  <mat-datepicker-toggle matIconSuffix [for]="picker" />
  <mat-datepicker #picker startView="year" />
</mat-form-field>
```

En el fragmento anterior, definimos un campo de formulario usando el elemento `<mat-form-field>`. El campo de formulario consta de una etiqueta, una entrada y un componente selector de fechas. Podemos abrir el selector de fechas haciendo clic en el elemento de entrada o utilizando el componente `<mat-datepicker-toggle>`, que es un botón con un icono de calendario. El atributo `startView` en el componente `<mat-datepicker>` configura el selector de fechas para mostrar inicialmente los meses del año actual.

6. Abre el archivo `expenses.scss` y agrega el siguiente estilo:

```scss
mat-card-content {
  margin-top: 1rem;
}
```

El estilo CSS anterior dará espacio entre el contenido principal de la tarjeta y su encabezado.

7. Inicia la aplicación, navega a `http://localhost:4000/1` y verifica que la salida se vea de la siguiente manera:

> *Figura 7.6 – Selección de fecha*

Como podemos ver en la imagen anterior, los usuarios podrán reportar los gastos mensualmente utilizando el control selector de fechas al comienzo del proceso. Implementaremos la funcionalidad de reporte de gastos en dos pasos:

- Reporte de gastos estándar
- Reporte de gastos por apartamento

La funcionalidad consta de dos formularios dentro del área de contenido principal de la tarjeta. Comenzaremos con los gastos estándar en la siguiente sección.

#### Reporte de gastos estándar

Los gastos típicos de un edificio son los siguientes:

- Electricidad
- Consumo de agua
- Limpieza
- Mantenimiento del ascensor
- Calefacción

Agregaremos un formulario dentro del contenido de la tarjeta para ingresar los gastos estándar. El formulario incluirá un campo de entrada para cada tipo de gasto:

1. Abre el archivo `expenses.ts` y agrega la siguiente sentencia de importación:

```typescript
import { MatExpansionModule } from '@angular/material/expansion';
```

La clase `MatExpansionModule` contiene una colección de componentes que podemos usar para crear paneles plegables.

2. Agrega la clase anterior al array `imports` del decorador del componente:

```typescript
imports: [
  MatCardModule,
  MatFormFieldModule,
  MatDatepickerModule,
  MatInput,
  MatExpansionModule
]
```

3. Abre el archivo `expenses.html` y agrega el siguiente código HTML debajo del campo de formulario de fecha:

```html
<mat-expansion-panel>
  <mat-expansion-panel-header>
    <mat-panel-title>Standard expenses</mat-panel-title>
  </mat-expansion-panel-header>
</mat-expansion-panel>
```

El fragmento anterior crea un panel de expansión con un encabezado.

4. Agrega un campo de formulario debajo del encabezado del panel para el gasto de Electricidad (*Electricity*):

```html
<mat-form-field>
  <mat-label>Electricity</mat-label>
  <input matInput>
</mat-form-field>
```

5. Repite el paso anterior para todos los tipos de gastos.
6. Abre el archivo `expenses.scss` y agrega el siguiente estilo:

```scss
mat-expansion-panel mat-form-field {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}
```

El estilo CSS anterior mostrará los campos del formulario en un diseño flexbox, uno debajo del otro.

7. Ejecuta la aplicación y verifica que la salida de la página de gastos se vea de la siguiente manera:

> *Figura 7.7 – Página de gastos con panel de expansión*

La imagen anterior muestra el panel de expansión para los gastos estándar debajo del selector de fechas.

8. Haz clic en el panel Standard expenses y verifica que veas los siguientes campos:

> *Figura 7.8 – Campos de gastos estándar*

Los campos no están conectados actualmente a una fuente de datos. Crearemos un formulario reactivo para vincular los campos. Usaremos formularios reactivos en este caso porque queremos reaccionar al ingresar un valor en cada campo para calcular la cantidad total de gastos:

1. Abre el archivo `expenses.ts` y agrega las siguientes sentencias de importación:

```typescript
import { FormControl, FormGroup, ReactiveFormsModule } from '@angular/forms';
```

2. Agrega la clase `ReactiveFormsModule` al array `imports` del decorador del componente:

```typescript
imports: [
  MatCardModule,
  MatFormFieldModule,
  MatDatepickerModule,
  MatInput,
  MatExpansionModule,
  ReactiveFormsModule
]
```

3. Crea el formulario reactivo en la clase del componente `Expenses`:

```typescript
form = new FormGroup({
  electricity: new FormControl(0),
  water: new FormControl(0),
  cleaning: new FormControl(0),
  elevator: new FormControl(0),
  heating: new FormControl(0)
});
```

Cada control de formulario en la instancia de `FormGroup` tiene un valor predeterminado de `0`.

4. Abre el archivo `expenses.html`, crea un elemento `<form>` alrededor de los campos del formulario y vincúlalo al formulario reactivo:

```html
<form [formGroup]="form">
```

5. Vincula cada elemento de entrada con el control de formulario adecuado de la siguiente manera:

```html
<input matInput formControlName="electricity">
```

El valor de `formControlName` debe coincidir con el nombre del control de formulario.

6. La página de gastos ahora debería verse de la siguiente manera:

> *Figura 7.9 – Formulario de gastos estándar*

En la imagen anterior, cada campo de gasto se inicializa con un valor de 0.

El formulario de gastos estándar permite a los usuarios ingresar una cantidad para cada tipo de gasto. Será útil mostrar la cantidad total de gastos convenientemente.

Calcularemos la cantidad cuando el usuario ingrese un valor en los campos de gastos y la mostraremos en el encabezado del panel de expansión:

1. Abre el archivo `expenses.ts` e importa el método `signal` y la clase `CurrencyPipe`:

```typescript
import { Component, inject, signal } from '@angular/core';
import { CurrencyPipe } from '@angular/common';
```

La clase `CurrencyPipe` formatea un valor de texto como una moneda según la configuración regional.

2. Importa también el método `takeUntilDestroyed` del paquete `rxjs-interop`:

```typescript
import { toSignal, takeUntilDestroyed } from '@angular/core/rxjs-interop';
```

El método anterior nos permite cancelar la suscripción a un observable cuando se destruye el componente actual.

3. Agrega la clase `CurrencyPipe` al array `imports` del decorador del componente:

```typescript
imports: [
  MatCardModule,
  MatFormFieldModule,
  MatDatepickerModule,
  MatInput,
  MatExpansionModule,
  ReactiveFormsModule,
  CurrencyPipe
]
```

4. Declara la siguiente propiedad en la clase del componente `Expenses` utilizando el método `signal` del paquete `@angular/core`:

```typescript
readonly total = signal(0);
```

5. Agrega el siguiente constructor:

```typescript
constructor() {
  this.form.valueChanges.pipe(takeUntilDestroyed())
    .subscribe(values => {
      const amount = Object.values(values).flatMap(v => {
        return Number(v);
      });
      this.total.set(amount.reduce((x, y) => x + y, 0));
    }
  );
}
```

En el código anterior, escuchamos los cambios de valor del grupo de formularios. Exportamos los valores del objeto de grupo de formularios al array `amount` y calculamos su total.

6. Abre el archivo `expenses.html` y agrega el siguiente fragmento debajo del componente de título del panel Standard expenses:

```html
<mat-panel-description>
  {{ total() | currency }}
</mat-panel-description>
```

El componente `<mat-panel-description>` representa la descripción del panel y muestra la cantidad total.

7. Ejecuta la aplicación, ingresa algunos valores en los campos de gastos y verifica que el panel muestre la cantidad total:

> *Figura 7.10 – Cantidad total de gastos estándar*

En la imagen anterior, el importe total de los gastos es de $178.80.

La aplicación puede reportar gastos estándar utilizando un diseño de Angular Material y mostrar la cantidad total con fines de validación. En la siguiente sección, crearemos el formulario para reportar los gastos de los apartamentos.

#### Reporte de gastos por apartamento

Como aprendimos, el conteo de apartamentos para el edificio proviene de la API. Usaremos formularios basados en plantillas para construir un formulario dinámico que muestre un campo de entrada para cada apartamento:

1. Agrega un nuevo panel de expansión debajo del existente con el siguiente código:

```html
<mat-expansion-panel>
  <mat-expansion-panel-header>
    <mat-panel-title>Apartment expenses</mat-panel-title>
  </mat-expansion-panel-header>
</mat-expansion-panel>
```

2. Combina los paneles de expansión envolviéndolos en un elemento `<mat-accordion>` que nos permita expandir uno a la vez.
3. Agrega el atributo `multi` al acordeón para que podamos abrir los paneles de forma independiente:

```html
<mat-accordion multi>
```

4. Ejecuta la aplicación, navega a la página de gastos y verifica que el nuevo panel se muestre de la siguiente manera:

> *Figura 7.11 – Acordeón de gastos*

En la imagen anterior, los paneles de expansión están agrupados y podemos abrir cada uno independientemente del otro.

Usaremos formularios basados en plantillas para diseñar el formulario para reportar los gastos de los apartamentos:

1. Abre el archivo `expenses.ts` e importa el método `linkedSignal` y la clase `FormsModule`:

```typescript
import { Component, inject, signal, linkedSignal } from '@angular/core';
import { FormsModule } from '@angular/forms';
```

El método `linkedSignal` se utiliza para crear una signal grabable que depende del valor de otra signal.

2. Agrega la clase `FormsModule` al array `imports` del decorador del componente:

```typescript
imports: [
  MatCardModule,
  MatFormFieldModule,
  MatDatepickerModule,
  MatInput,
  MatExpansionModule,
  ReactiveFormsModule,
  CurrencyPipe,
  FormsModule
]
```

3. Crea la siguiente propiedad en la clase del componente `Expenses`:

```typescript
readonly aExpenses = linkedSignal(() => {
  return Array.from(
    { length: this.building()?.apartments! },
    () => 0
  );
});
```

El fragmento anterior crea una signal de array con una longitud igual al conteo de apartamentos e inicializa cada elemento en 0.

4. Agrega un método para actualizar los gastos usando el siguiente código:

```typescript
updateExpense(apartment: number, amount: string) {
  this.aExpenses.update(expenses => {
    expenses[apartment] = Number(amount);
    return [...expenses];
  });
}
```

En el fragmento anterior, actualizamos el array de la signal cambiando el valor del array en el índice específico que representa el apartamento actual.

5. Abre el archivo `expenses.html` y agrega el siguiente código HTML debajo del encabezado del segundo panel:

```html
@for (expense of aExpenses(); track $index) {
  <mat-form-field>
    <mat-label>Apartment {{ $index + 1 }}</mat-label>
    <input matInput [name]="$index.toString()" [ngModel]="expense" (ngModelChange)="updateExpense($index, $event)">
  </mat-form-field>
}
```

En el fragmento anterior, iteramos sobre el array `aExpenses` y creamos un componente de campo de formulario para cada elemento. La propiedad `$index` representa el número de apartamento actual. Cada campo de formulario tiene una etiqueta que muestra el número de apartamento y un elemento de entrada. El nombre del elemento de entrada es el número de apartamento y el valor es el elemento actual del array. Cuando el modelo subyacente del elemento de entrada cambia, llamamos al método `updateExpense` para actualizar el valor correspondiente en el array.

6. Navega a la página de gastos de la aplicación y haz clic en el panel Apartment expenses para ver los campos de entrada:

> *Figura 7.12 – Formulario de gastos de apartamentos*

La imagen anterior muestra solo un subconjunto de los campos de gastos de apartamentos.

La última pieza del formulario de gastos de apartamentos es mostrar la cantidad total de gastos, similar a la sección anterior:

1. Agrega una descripción para el segundo panel usando el siguiente fragmento:

```html
<mat-panel-description>
  {{ aTotal() | currency }}
</mat-panel-description>
```

2. Abre el archivo `expenses.ts` e importa el método signal `computed`:

```typescript
import { Component, inject, signal, linkedSignal, computed } from '@angular/core';
```

3. Crea la propiedad `aTotal` en la clase del componente `Expenses`:

```typescript
readonly aTotal = computed(() => {
  return this.aExpenses().reduce((x, y) => x + y, 0);
});
```

La signal anterior calcula la suma de todos los valores del array de gastos de apartamentos.

4. Intenta ingresar valores en los campos de gastos de apartamentos y verifica que la aplicación muestre el total correctamente, similar a lo siguiente:

> *Figura 7.13 – Cantidad total de gastos de apartamentos*

La imagen anterior muestra el importe total para los apartamentos 1 y 2.

Con el formulario de gastos de apartamentos, el administrador tiene el control total de los gastos reportados del edificio.

---

### Sección: Resumen

En este capítulo, aprendimos cómo aprovechar las técnicas de SSR para mejorar el rendimiento de una aplicación Angular. Exploramos cómo configurar Angular SSR y qué archivos se requieren para compilar una aplicación SSR. Experimentamos con Lighthouse para ver cómo una aplicación SSR nos ayuda en términos de métricas de CWV. Finalmente, combinamos el poder de los formularios de Angular con Angular Material para diseñar un creador de gastos que enfatizó la simplicidad y la experiencia de usuario de una aplicación Angular.

En el siguiente capítulo, aprenderemos a utilizar la biblioteca Kendo UI y construiremos una aplicación de gestión de turnos de fábrica.

---

### Sección: Ejercicios

Modifica el enlace en el encabezado de la aplicación para navegar a un nuevo componente que mostrará los edificios disponibles. Los usuarios podrán seleccionar un edificio y navegar a la página de gastos respectiva. El nuevo componente debe utilizar el modo de prerenderización en SSR.

---

### Sección: Lecturas Complementarias

- **Angular SSR:** [https://angular.dev/guide/ssr](https://angular.dev/guide/ssr)
- **Angular Material:** [https://material.angular.dev](https://material.angular.dev/)
- **JSON-Server:** [https://www.npmjs.com/package/json-server](https://www.npmjs.com/package/json-server)
- **Core Web Vitals:** [https://web.dev/explore/learn-core-web-vitals](https://web.dev/explore/learn-core-web-vitals)
