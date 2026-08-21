# Parte 1: Primeros Pasos con Angular e Inteligencia Artificial

## Capítulo 5: CityPass Parking: Desarrollo de un Validador de Estacionamiento Impulsado por IA

### Sección: Introducción

El tráfico es un problema común en áreas urbanas densamente pobladas, especialmente en las grandes ciudades. El volumen de vehículos en la carretera es significativo y las personas en los centros de las ciudades a menudo tienen dificultades para encontrar estacionamiento. Para abordar este problema, los ayuntamientos designan plazas de estacionamiento donde los conductores pueden emitir un ticket mediante el pago de una tarifa y durante una duración específica.

En este capítulo, crearemos una aplicación que el personal municipal utilizará para verificar si el ticket de un automóvil es válido. Podrán comprobar si se ha pagado la tarifa y por cuánto tiempo. Las tecnologías avanzadas de mapas e inteligencia artificial ayudarán a los usuarios a acceder a información precisa y ofrecer resultados eficientes.

Vamos a cubrir los siguientes temas:

- Instalación de NG-ZORRO
- Adición manual de nuevos tickets
- Adición de nuevos tickets con IA
- Visualización de coches registrados
- Revisión de detalles de tickets

---

### Sección: Requisitos Técnicos

Puedes descargar el proyecto de ejemplo y el código de este libro siguiendo las instrucciones de la sección *Download the example code files* en el Prefacio de este libro.

Los archivos de código de este capítulo están incluidos en el paquete de código descargable.

---

### Sección: Instalación de NG-ZORRO

La biblioteca NG-ZORRO es una biblioteca de componentes de interfaz de usuario que proporciona una colección de componentes de nivel empresarial, incluidos controles de selección de fecha y hora para trabajar con objetos de fecha y hora.

En esta sección, aprenderás a instalar y configurar NG-ZORRO en una aplicación Angular. Utilizaremos la aplicación del Capítulo 1, *Angular AI Kick-Starter: Scaffolding Smart Apps with Copilot*, como punto de partida.

Comencemos instalando NG-ZORRO:

1. Ejecuta el siguiente comando dentro del proyecto de Angular CLI para instalar NG-ZORRO:

```bash
ng add ng-zorro-antd
```

El comando anterior iniciará un proceso que te guiará al agregar NG-ZORRO. El proceso implica hacer preguntas para proporcionar más contexto en la CLI sobre la aplicación que deseas construir.

Si tienes problemas para ejecutar el comando anterior, consulta el archivo `CHANGELOG.md` del repositorio de GitHub del capítulo. Para acceder al enlace del repositorio, sigue los pasos de la sección "Download the example code files" en el Prefacio.

2. La primera pregunta es si deseas habilitar la carga dinámica de iconos. Presiona ENTER para omitir este paso, ya que no utilizaremos iconos en la aplicación.
3. La siguiente pregunta es si usaremos un archivo de tema personalizado. Presiona ENTER para usar los temas integrados.
4. Presiona ENTER para seleccionar la configuración regional `en_US`.
5. En la última pregunta, podemos elegir una plantilla para nuestra aplicación. NG-ZORRO admite las siguientes plantillas prediseñadas:

- **sidemenu:** Muestra un menú de barra lateral a la izquierda con el contenido principal a la derecha.
- **topnav:** Muestra un menú en la parte superior con el contenido principal en la parte inferior.

Selecciona **topnav** y presiona ENTER. La Angular CLI modifica la estructura del proyecto Angular en función de la plantilla seleccionada.

6. NG-ZORRO necesita el paquete de animaciones de Angular. Ejecuta el siguiente comando para instalarlo:

```bash
npm install @angular/animations
```

7. Abre el archivo `app.html` y reemplaza el contenido del elemento `h1` con el siguiente texto: `CityPass Parking`.

No utilizaremos el componente de título del capítulo en este capítulo porque NG-ZORRO ya lo ha configurado para nosotros. Puedes eliminar todos los archivos de título del capítulo del proyecto si lo deseas.

8. Ejecuta la aplicación usando el comando `ng serve` y navega a `localhost:4200`:

> *Figura 5.1 – Salida de la aplicación*

La imagen anterior muestra el texto predeterminado del componente principal de la aplicación.

Instalar NG-ZORRO con una plantilla prediseñada cambia el diseño de nuestra aplicación y agrega contenido predefinido, como se muestra en la figura anterior. La página principal muestra:

- Un encabezado con el título del capítulo y tres enlaces
- Un mensaje de bienvenida

Actualizaremos los enlaces y el mensaje de bienvenida para que coincidan con el tipo de aplicación que estamos construyendo.

1. Abre el archivo `app.html` y reemplaza el texto del segundo y tercer elemento `li` de la siguiente manera:

```html
<li nz-menu-item>Tickets</li>
<li nz-menu-item>Cars</li>
```

Conectaremos los enlaces más adelante utilizando Angular Router para cargar las características correspondientes de la aplicación.

2. Abre el archivo `welcome.html` y reemplaza su contenido con el siguiente código HTML:

```html
<h1>Welcome to CityPass Parking</h1>
<p>Here you can:</p>
<u>
  <li>Add new tickets</li>
  <li>Validate tickets</li>
  <li>Review cars</li>
</u>
```

El archivo anterior es la plantilla del componente de bienvenida. La aplicación lo carga de forma predeterminada como se indica en el archivo `app.routes.ts`:

```typescript
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: '',
    pathMatch: 'full',
    redirectTo: '/welcome'
  },
  {
    path: 'welcome',
    loadChildren: () => import('./pages/welcome/welcome.routes')
      .then(m => m.WELCOME_ROUTES)
  }
];
```

De hecho, la aplicación no carga el componente de bienvenida directamente. Utiliza el método `loadChildren` para cargar las rutas secundarias desde la variable `WELCOME_ROUTES`, que luego carga el componente como se especifica en el archivo `welcome.routes.ts`:

```typescript
import { Routes } from '@angular/router';
import { Welcome } from './welcome';

export const WELCOME_ROUTES: Routes = [
  {
    path: '',
    component: Welcome
  },
];
```

La biblioteca NG-ZORRO utiliza una estructura de proyecto basada en opiniones para las características de la aplicación cubiertas en el resto de este capítulo. Cada carpeta es una funcionalidad que contiene un archivo de rutas junto con un componente principal. En la siguiente sección, implementaremos la primera funcionalidad de la aplicación, permitiendo a los usuarios agregar nuevos tickets de estacionamiento al sistema.

---

### Sección: Adición Manual de Nuevos Tickets

La creación de un nuevo ticket para una plaza de estacionamiento es la acción más básica de la aplicación. Debe ser accesible y eficiente para ingresar los detalles requeridos del ticket. Los usuarios crearán nuevos tickets a través del enlace Tickets en el encabezado de la aplicación ingresando la siguiente información:

- Número de matrícula (*plate number*)
- Fecha y hora de llegada
- Ubicación

Utilizaremos el componente de formulario de NG-ZORRO para diseñar un formulario que recopile los datos proporcionados. El flujo de trabajo para agregar nuevos tickets se puede resumir en las siguientes tareas:

- Creación de una página de funcionalidad
- Diseño de formularios con NG-ZORRO
- Integración de formularios basados en plantillas

Comenzaremos la implementación creando una nueva página de funcionalidad dentro de la base de código existente.

#### Creación de una página de funcionalidad

Seguiremos la estructura de NG-ZORRO y crearemos el nuevo componente dentro de la carpeta `pages`:

1. Crea el componente de tickets ejecutando el siguiente comando dentro de la carpeta `pages`:

```bash
ng generate component tickets
```

2. Crea el archivo `tickets.routes.ts` dentro de la carpeta `tickets` e introduce el siguiente fragmento:

```typescript
import { Routes } from '@angular/router';
import { Tickets } from './tickets';

export const TICKETS_ROUTES: Routes = [
  {
    path: '',
    component: Tickets
  }
];
```

El archivo anterior define la configuración de enrutamiento de la funcionalidad de tickets de la aplicación. En este archivo, podemos agregar rutas futuras, como mostrar una lista de tickets o editar los detalles de un ticket.

3. Abre el archivo `app.routes.ts` y agrega la siguiente ruta dentro del array `routes`:

```typescript
{
  path: 'tickets',
  loadChildren: () => import('./pages/tickets/tickets.routes')
    .then(m => m.TICKETS_ROUTES)
}
```

En la ruta anterior, utilizamos el método `loadChildren` para cargar de forma diferida (*lazily load*) la configuración para las rutas de tickets. La configuración de ruta de tickets cargará el componente `Tickets` de forma predeterminada, como lo indica la ruta de cadena vacía en el paso anterior.

4. Abre el archivo `app.ts` e importa la directiva `RouterLink` del paquete `@angular/router`:

```typescript
import { RouterOutlet, RouterLink } from '@angular/router';
```

5. Agrega la directiva en el array `imports` del decorador del componente:

```typescript
imports: [
  RouterOutlet,
  NzLayoutModule,
  NzMenuModule,
  RouterLink
]
```

6. Abre el archivo `app.html` y agrega la directiva `routerLink` al elemento de lista Tickets de la siguiente manera:

```html
<li nz-menu-item routerLink="/tickets">Tickets</li>
```

7. Ejecuta la aplicación con `ng serve` y navega a `http://localhost:4200`.
8. Haz clic en el enlace Tickets y verifica que la página muestre lo siguiente como contenido principal:

> *Figura 5.2 – Funcionalidad de Tickets*

La imagen anterior muestra el texto predeterminado del componente tickets.

La página de tickets muestra el texto predeterminado del componente y es accesible desde el encabezado de la aplicación. En la siguiente sección, comenzaremos a diseñar el formulario para recopilar los datos del ticket.

#### Diseño de formularios con NG-ZORRO

Utilizaremos el componente tickets que creamos en la sección anterior para diseñar el formulario que recopile los detalles del ticket del usuario:

1. Abre el archivo `tickets.ts` y agrega las siguientes sentencias de importación:

```typescript
import { NzFormModule } from 'ng-zorro-antd/form';
import { NzInputModule } from 'ng-zorro-antd/input';
```

La biblioteca NG-ZORRO incluye la clase `NzFormModule`, que nos ayuda a manejar formularios HTML en aplicaciones Angular. También necesitaremos la clase `NzInputModule` para definir los controles de formulario.

2. Agrega las clases anteriores al array `imports` del decorador del componente:

```typescript
imports: [NzFormModule, NzInputModule]
```

3. Abre el archivo `tickets.html` y reemplaza su contenido con el siguiente código HTML:

```html
<form nz-form></form>
```

El código anterior representa un elemento de formulario HTML que se renderizará como un formulario NG-ZORRO durante el tiempo de ejecución. El atributo `nz-form` indica que el formulario es un componente de formulario de NZ-ZORRO.

Un formulario en la biblioteca NZ-ZORRO consta de elementos de formulario (*form items*). Inserta el siguiente fragmento dentro del elemento `<form>` para agregar un elemento de formulario para el número de matrícula:

```html
<nz-form-item>
  <nz-form-label nzFor="plateNo">
    Plate number
  </nz-form-label>
  <nz-form-control>
    <input nz-input name="plateNo" id="plateNo" />
  </nz-form-control>
</nz-form-item>
```

Un elemento de formulario consta de una etiqueta y un control de formulario. El componente `nz-form-label` indica la etiqueta y `nz-form-control` el control de formulario real. El atributo `nz-input` indica que el elemento `<input>` es un componente de entrada NZ-ZORRO utilizado para el número de matrícula. Usamos el atributo `nzFor` para vincular la etiqueta al control a través del atributo `id`, de modo que los usuarios puedan enfocar el campo haciendo clic en la etiqueta.

4. Ejecuta la aplicación y haz clic en el enlace Tickets para previsualizar el nuevo formulario:

> *Figura 5.3 – Control de número de matrícula*

El formulario muestra la etiqueta al lado del control porque utiliza un diseño horizontal de forma predeterminada. Puedes cambiarlo seleccionando un diseño de [https://ng.ant.design/#components-form-demo-layout](https://ng.ant.design/#components-form-demo-layout).

Ahora podemos agregar los controles restantes para la fecha y hora de llegada y la ubicación en el formulario:

1. Abre el archivo `tickets.ts` y agrega la siguiente sentencia de importación:

```typescript
import { NzDatePickerModule } from 'ng-zorro-antd/date-picker';
```

La clase `NzDatePickerModule` proporciona un conjunto de componentes para trabajar con fechas y horas.

2. Agrega la clase anterior al array `imports` del decorador del componente:

```typescript
imports: [NzFormModule, NzInputModule, NzDatePickerModule]
```

3. Abre el archivo `tickets.html` y agrega el siguiente fragmento HTML después del elemento de formulario existente:

```html
<nz-form-item>
  <nz-form-label>Arrival</nz-form-label>
  <nz-form-control>
    <nz-date-picker name="arrival" nzShowTime />
  </nz-form-control>
</nz-form-item>
```

El elemento de formulario anterior utiliza el componente `nz-date-picker` para mostrar un control de selección de fecha. El atributo `nzShowTime` agrega la selección de hora al selector de fechas.

4. Agrega el siguiente fragmento HTML después del elemento de formulario de fecha para crear un control para la ubicación del estacionamiento:

```html
<nz-form-item>
  <nz-form-label nzFor="location">
    Location
  </nz-form-label>
  <nz-form-control>
    <input nz-input name="location" id="location" />
  </nz-form-control>
</nz-form-item>
```

5. Ejecuta la aplicación y navega al enlace Tickets:

> *Figura 5.4 – Formulario de tickets*

La imagen anterior muestra todos los controles de formulario necesarios para crear un ticket.

6. Haz clic en el control Arrival para mostrar el selector de fechas:

> *Figura 5.5 – Selector de fechas*

El selector de fechas que se muestra en la imagen selecciona el día actual de forma predeterminada.

7. Selecciona una fecha y hora, y observa el valor mostrado en el control Arrival:

> *Figura 5.6 – Control de llegada*

La imagen anterior muestra la fecha y hora seleccionadas en un formato predefinido. Puedes modificarlo leyendo la guía de documentación en [https://ng.ant.design/#components-date-picker-demo-format](https://ng.ant.design/#components-date-picker-demo-format).

Nuestro formulario incluye todos los campos que utilizaremos para recopilar la entrada del usuario. Integraremos el componente de formulario NG-ZORRO con formularios de Angular para capturar los datos del formulario en la siguiente sección.

#### Integración de formularios basados en plantillas

Utilizaremos formularios basados en plantillas (*template-driven forms*) de Angular, que son muy sencillos de configurar y son ideales para formularios pequeños como el formulario de tickets:

1. Abre el archivo `tickets.ts` y agrega la siguiente sentencia de importación:

```typescript
import { FormsModule } from '@angular/forms';
```

La clase `FormsModule` contiene componentes y directivas para trabajar con formularios basados en plantillas en Angular.

2. Agrega la clase `FormsModule` al array `imports` del decorador del componente:

```typescript
imports: [
  NzFormModule,
  NzInputModule,
  NzDatePickerModule,
  FormsModule
]
```

3. Define las siguientes propiedades en la clase del componente `Tickets`:

```typescript
readonly plateNo = model('');
readonly arrival = model(new Date());
readonly location = model('');
```

Usamos el método `model` del paquete `@angular/core` para definir una signal para cada control de formulario. También establecemos el valor inicial de `arrival` en la fecha y hora actuales.

4. Crea una directiva `ngModel` para cada control de formulario y asígnala a la propiedad signal respectiva:

```html
<input nz-input name="plateNo" id="plateNo" [(ngModel)]="plateNo" />
<nz-date-picker name="arrival" nzShowTime [(ngModel)]="arrival" />
<input nz-input name="location" id="location" [(ngModel)]="location" />
```

5. Agrega el atributo `required` en todos los controles de formulario del paso anterior porque un ticket debe tener valores para todas las propiedades para ser válido.
6. Utiliza el atributo `nzRequired` en los componentes `nz-form-label` de la siguiente manera:

```html
<nz-form-label nzFor="plateNo" nzRequired>
  Plate number
</nz-form-label>
<nz-form-label nzRequired>Arrival</nz-form-label>
<nz-form-label nzFor="location" nzRequired>
  Location
</nz-form-label>
```

El atributo anterior muestra un asterisco rojo junto a la etiqueta, indicando que el control es obligatorio.

7. Ejecuta la aplicación y verifica que la salida del formulario se vea de la siguiente manera:

> *Figura 5.7 – Formulario de tickets*

Todos los campos en el formulario de tickets tienen un asterisco rojo delante de sus etiquetas.

Para procesar los datos del formulario, agregaremos un botón para enviar el formulario e invocar un método en la clase del componente:

1. Abre el archivo `tickets.ts` y agrega la siguiente sentencia de importación:

```typescript
import { NzButtonModule } from 'ng-zorro-antd/button';
```

La clase `NzButtonModule` proporciona componentes y directivas de Angular para botones HTML.

2. Agrega la clase `NzButtonModule` al array `imports` del decorador del componente:

```typescript
imports: [
  NzFormModule,
  NzInputModule,
  NzDatePickerModule,
  FormsModule,
  NzButtonModule
]
```

3. Crea el siguiente método en la clase del componente que registre los valores del ticket en la consola del navegador:

```typescript
add() {
  console.table([{
    'plateNo': this.plateNo(),
    'arrival': this.arrival(),
    'location': this.location()
  }]);
}
```

Usamos el método `console.table` para mostrar los valores del ticket en una tabla. Más adelante en el capítulo, agregaremos un servicio para conservar los datos en memoria.

4. Abre el archivo `tickets.html` y agrega la directiva `ngForm` al elemento de formulario:

```html
<form nz-form #ticketForm="ngForm">
```

Definimos la variable de referencia de plantilla `ticketForm` y la asignamos a la directiva `ngForm` para que podamos acceder a las propiedades del formulario más adelante en la plantilla.

5. Vincula el evento de envío del formulario al método del componente `add`:

```html
<form nz-form #ticketForm="ngForm" (ngSubmit)="add()">
```

La aplicación ejecutará el método `add` cuando se envíe el formulario.

6. Agrega el siguiente elemento de formulario al final del formulario:

```html
<nz-form-item>
  <nz-form-control>
    <button nz-button nzType="primary" type="submit" [disabled]="ticketForm.invalid">
      Submit
    </button>
  </nz-form-control>
</nz-form-item>
```

Las acciones de formulario con botones también se envuelven de manera similar a los controles que usamos anteriormente. El atributo `nz-button` representa un componente de botón y el atributo `nzType` define su tema, incluido el color.

NG-ZORRO admite la personalización de temas siguiendo la guía en [https://ng.ant.design/docs/customize-theme/en](https://ng.ant.design/docs/customize-theme/en).

Deshabilitamos el botón cuando el formulario no es válido, lo que ocurre cuando algún control del formulario está vacío. Además, establecemos su tipo en `submit` para que el formulario se envíe cuando el usuario haga clic en él.

7. Navega al menú Tickets mientras la aplicación se está ejecutando y se debería mostrar el botón Submit:

> *Figura 5.8 – Formulario de tickets con acción*

El botón Submit se deshabilita cuando algunos campos obligatorios se dejan en blanco.

8. Ingresa datos en los campos requeridos, haz clic en el botón Submit y verifica que la salida de la consola sea similar a la siguiente:

> *Figura 5.9 – Registro de la aplicación*

La imagen anterior muestra datos de ticket de ejemplo.

Nuestra aplicación utiliza Angular Forms y la biblioteca NG-ZORRO para permitir a los usuarios crear nuevos tickets de estacionamiento mediante formularios HTML. En la siguiente sección, automatizaremos el proceso permitiendo a los usuarios enviar nuevos tickets con un asistente de IA.

---

### Sección: Adición de Nuevos Tickets con IA

Los formularios web que requieren datos extensos o complejos pueden beneficiarse de la tecnología asistida por IA mediante la creación de aplicaciones agénticas (*agentic apps*). Una aplicación agéntica utiliza un agente de IA para automatizar diferentes tareas. En nuestro caso, el campo de ubicación en el formulario de tickets debe almacenar datos en un formato adecuado para el mapeo. Podemos usar un agente de IA para automatizar la creación de tickets y gestionar los datos de ubicación en consecuencia.

Integraremos la API de Gemini en la aplicación y agregaremos capacidades de agente para crear nuevos tickets. Utilizaremos el SDK de Firebase AI Logic, que nos permite interactuar con Gemini directamente desde una aplicación del lado del cliente.

En el próximo capítulo, aprenderemos a utilizar la API de Gemini desde una aplicación de backend en el servidor.

El proceso que seguiremos para implementar la creación de tickets con IA consta de las siguientes tareas:

- Configuración de la infraestructura de IA
- Uso de Firebase AI Logic
- Diseño del creador con IA (*AI Creator*)

Antes de continuar, extraigamos el método de creación de tickets en un servicio dedicado de Angular para que el agente de IA pueda utilizarlo:

1. Crea un servicio de estacionamiento ejecutando el siguiente comando dentro de la carpeta `app`:

```bash
ng generate service parking
```

2. Abre el archivo `parking.ts` y crea un método para registrar los detalles del ticket en la consola del navegador:

```typescript
createTicket(plate: string, arrival: Date, loc: string) {
  console.table([{
    'plateNo': plate,
    'arrival': arrival,
    'location': loc
  }]);
}
```

3. Abre el archivo `tickets.ts` e importa la clase `Parking`:

```typescript
import { Parking } from '../../parking';
```

4. Inyecta el servicio en la clase del componente `Tickets` utilizando el método `inject` del paquete `@angular/core`:

```typescript
private parkingService = inject(Parking);
```

5. Reemplaza el contenido del método `add` con el siguiente fragmento:

```typescript
this.parkingService.createTicket(
  this.plateNo(),
  this.arrival(),
  this.location()
);
```

El servicio de estacionamiento es responsable de gestionar los tickets de los vehículos. Firebase AI Logic interactuará con el servicio mediante un método llamado llamada a funciones (*function calling*).

En la llamada a funciones, la aplicación expone los métodos requeridos como herramientas (*tools*). Un agente de IA puede descubrir herramientas y ejecutarlas para interactuar con la aplicación.

Primero, utilizaremos la consola de Firebase para configurar todos los requisitos previos necesarios para la IA.

#### Configuración de la infraestructura de IA

Crearemos un nuevo proyecto de Firebase y lo conectaremos a nuestra aplicación:

1. Navega a [https://console.firebase.google.com/project/_/ailogic](https://console.firebase.google.com/project/_/ailogic). Necesitarás una cuenta de Google para iniciar sesión en la consola de Firebase.
2. Selecciona la opción **Create a new Firebase project**.
3. Escribe `citypass` en el campo Project name y haz clic en el botón **Continue**.
4. Asegúrate de que la opción **Enable Gemini in Firebase** esté habilitada y haz clic en **Continue** en la página AI assistance for your Firebase project.
5. Desactiva la opción **Enable Google Analytics for this project** y haz clic en el botón **Create project**. Analytics no es necesario para nuestra aplicación, pero puedes habilitarlo más adelante si deseas agregar funciones de monitoreo.
6. Espera a que se complete el proceso de creación del proyecto y haz clic en el botón **Continue**.

Ahora que tenemos un proyecto de Firebase, podemos configurar Firebase AI Logic:

1. Haz clic en el botón **Get started** en la página de bienvenida del servicio Firebase AI Logic.
2. Haz clic en el botón **Get started with this API** en el panel Gemini Developer API. Gemini Developer API es perfecta para usuarios principiantes que desean comenzar sin coste.
3. Haz clic en el botón **Enable API** en el paso Enable the Gemini Developer API del asistente para habilitar Gemini.
4. Elige si deseas habilitar el monitoreo de IA de forma opcional en el siguiente paso del asistente y haz clic en el botón **Continue**.
5. Selecciona la opción **Web**, indicada por el icono de código, en el paso Add an app to start para crear una aplicación web de Firebase. La aplicación web nos permitirá asociar nuestra aplicación Angular con los servicios de Firebase.
6. Escribe `citypass` en el campo App nickname y haz clic en el botón **Register app**. Asegúrate de anotar la variable `firebaseConfig` generada por Firebase, ya que la usaremos más adelante.
7. Haz clic en el botón **Continue to the console** para completar la creación de la aplicación web.
8. Haz clic en el botón **Continue** en el paso del asistente Add the Firebase AI Logic SDK para cerrar el asistente y navegar al panel de control de Firebase AI Logic.

El proyecto de Firebase incluye capacidades de IA habilitadas por la configuración de Firebase AI Logic. Aprenderemos a usar el SDK en la siguiente sección.

#### Uso de Firebase AI Logic

Podemos utilizar la API modular web del SDK de Firebase AI Logic para interactuar con la IA en nuestra aplicación Angular:

1. Instala Firebase en el proyecto actual de Angular CLI mediante el siguiente comando:

```bash
npm install firebase
```

2. Abre el archivo `app.config.ts` y agrega la siguiente sentencia de importación:

```typescript
import { initializeApp } from 'firebase/app';
```

3. Copia el objeto `firebaseConfig` de la configuración de Firebase AI Logic.
4. Crea la siguiente variable que inicializa Firebase con la configuración proporcionada de AI Logic:

```typescript
const firebaseApp = initializeApp(firebaseConfig);
```

5. Agrega el siguiente fragmento al array `providers` del objeto `appConfig`:

```typescript
{ provide: 'FIREBASE_APP', useValue: firebaseApp }
```

El proveedor anterior hará que la aplicación de Firebase esté disponible para la aplicación Angular a través de la inyección de dependencias.

Firebase ofrece un amplio conjunto de servicios en la nube que podemos usar en una aplicación web. La inicialización de la aplicación de Firebase conecta la aplicación Angular con estos servicios. La aplicación Angular interactúa con un servicio de Firebase a través de una API predefinida. En nuestro caso, interactuaremos con la API web del SDK de Firebase AI Logic:

1. Abre el archivo `parking.ts` y agrega las siguientes sentencias de importación de la biblioteca de Firebase:

```typescript
import { ChatSession, FunctionDeclarationsTool, getAI, getGenerativeModel, Schema } from 'firebase/ai';
import { FirebaseApp } from 'firebase/app';
```

Aprenderemos qué hace cada una de las clases anteriores en los siguientes pasos, a medida que las usemos.

2. Declara la siguiente propiedad en la clase del servicio `Parking`:

```typescript
private readonly chat: ChatSession;
```

La propiedad `chat` nos permite intercambiar mensajes y mantener un historial con Gemini.

3. Agrega un constructor para inyectar el token `FIREBASE_APP`:

```typescript
constructor(
  @Inject('FIREBASE_APP') firebaseApp: FirebaseApp
) {}
```

Usamos el decorador `@Inject` del paquete `@angular/core` para inyectar el objeto `firebaseApp` en el servicio.

4. Crea la variable `toolset` en el constructor para definir las herramientas que están disponibles para Gemini:

```typescript
const toolset: FunctionDeclarationsTool = {
  functionDeclarations: []
};
```

Cada herramienta contiene una propiedad `functionDeclarations` que define las funciones que Gemini puede utilizar.

5. Agrega la siguiente declaración de función para agregar nuevos tickets:

```typescript
{
  name: 'addTicket',
  description: 'Add one ticket',
  parameters: Schema.object({
    properties: {
      plateNo: Schema.string(),
      arrival: Schema.string(),
      location: Schema.string()
    }
  })
}
```

Una declaración de función contiene metadatos sobre la función, como su nombre y descripción. Usamos el nombre de la respuesta del chat para determinar qué método de servicio ejecutar. También enumera los parámetros que acepta la función dentro del objeto `parameters`. El objeto `properties` contiene cada parámetro como un par clave-valor, donde la clave es el nombre del parámetro y el valor es el tipo de parámetro.

6. Define la siguiente variable que describe el contexto del agente de IA:

```typescript
const instructions = `
  Welcome to citypass.
  You are a superstar agent for this car parking validator.
  You will assist users by submitting parking tickets.
  You can convert date phrases to ISO strings and act as a geocode service to convert a location or address to coordinates long/lat.
`;
```

Proporcionar al agente de IA instrucciones detalladas es fundamental, ya que podrá gestionar los mensajes con mayor precisión. Con las instrucciones anteriores, el agente puede convertir una frase de fecha, como *today* o *tomorrow*, en un objeto `Date` de JavaScript. Además, podrá convertir una dirección en coordenadas, lo cual es necesario para mostrar los vehículos en el mapa más adelante.

7. Usa el método `getAI` para inicializar la Gemini Developer API para la aplicación actual de Firebase:

```typescript
const ai = getAI(firebaseApp);
```

8. Crea un modelo generativo utilizando el siguiente método:

```typescript
const model = getGenerativeModel(ai, {
  model: 'gemini-2.5-flash',
  systemInstruction: instructions,
  tools: [toolset]
});
```

El método anterior acepta la instancia de Gemini Developer API y un conjunto de opciones como parámetros. Las opciones definen el modelo que queremos utilizar, el contexto del sistema y la lista de herramientas compatibles.

9. Finalmente, inicializa una sesión de chat con el siguiente código:

```typescript
this.chat = model.startChat();
```

El agente de IA puede escuchar mensajes de la aplicación Angular y responder con la función adecuada a ejecutar. En la siguiente sección, diseñaremos una interfaz de usuario para interactuar con el agente de IA.

#### Diseño del creador con IA

Modificaremos el formulario de tickets para que los usuarios puedan enviar un mensaje al agente de IA desde la interfaz de usuario:

1. Abre el archivo `tickets.html` y agrega un nuevo botón cerca del botón Submit:

```html
<button nz-button type="button" nzType="link">
  AI Creator
</button>
```

El botón AI Creator aparecerá junto al botón Submit como un enlace. El tipo de botón se establece en `button`, por lo que no activa el envío del formulario al hacer clic.

2. Agrega los siguientes estilos al archivo `tickets.scss` para tener espacio entre los botones:

```scss
[nz-button] {
  margin-right: 8px;
  margin-bottom: 12px;
}
```

3. Abre el archivo `tickets.ts` y agrega la siguiente sentencia de importación:

```typescript
import { NzModalModule } from 'ng-zorro-antd/modal';
```

La clase `NzModalModule` proporciona un cuadro de diálogo modal que podemos usar para escribir el prompt para el agente de IA.

4. Agrega la clase anterior al array `imports` del decorador del componente:

```typescript
imports: [
  NzFormModule,
  NzInputModule,
  NzDatePickerModule,
  FormsModule,
  NzButtonModule,
  NzModalModule
]
```

5. Crea las siguientes propiedades de tipo signal en la clase del componente `Tickets`:

```typescript
readonly isVisible = model(false);
readonly prompt = model('');
```

La propiedad `isVisible` controlará la visibilidad del modal y la propiedad `prompt` se vinculará al prompt del usuario para el agente de IA.

6. Define el siguiente método que utilizaremos para enviar el prompt del usuario al agente de IA:

```typescript
async ok() {
  await this.parkingService.ask(this.prompt());
}
```

El método `ask` aún no existe en el servicio `Parking`. Debemos usar `async/await` para llamarlo porque la comunicación con el agente de IA se basará en promesas.

7. Abre el archivo `tickets.html` y agrega la siguiente vinculación al evento `click` del botón AI Creator:

```html
<button nz-button nzType="link" type="button" (click)="isVisible.set(true)">
  AI Creator
</button>
```

El modal se mostrará cuando hagamos clic en el botón.

8. Inserta el siguiente fragmento HTML al final de la plantilla para agregar un componente modal:

```html
<nz-modal [(nzVisible)]="isVisible" nzTitle="AI Creator" (nzOnCancel)="isVisible.set(false)" (nzOnOk)="ok()">
  <ng-container *nzModalContent>
    <textarea [(ngModel)]="prompt" rows="3" placeholder="Enter ticket details" nz-input>
    </textarea>
  </ng-container>
</nz-modal>
```

El componente modal de NG-ZORRO consta de las siguientes partes:

- `nzVisible`: Controla la visibilidad del modal.
- `nzTitle`: Muestra un título en el encabezado.
- `nzOnCancel`: Se ejecuta cuando el usuario descarta el modal usando el botón de cerrar o cancelar.
- `nzOnOk`: Se ejecuta cuando el usuario hace clic en el botón de aceptar.
- `*nzModalContent`: Define el contenido principal del modal. En nuestro caso, contiene un elemento `<textarea>` que se vincula a la propiedad `prompt`.

La única pieza que falta en la integración de la IA es implementar el método `ask`, que se encargará de comunicarse con Gemini:

1. Abre el archivo `parking.ts` y crea el siguiente método:

```typescript
async ask(prompt: string) {}
```

El método acepta un parámetro de tipo string que pasaremos a Gemini.

2. Llama al método `sendMessage` dentro del método `ask` de la siguiente manera:

```typescript
let result = await this.chat.sendMessage(prompt);
```

En el fragmento anterior, pasamos el prompt a la instancia de chat y guardamos el resultado en una variable local.

3. Extrae la propiedad `functionCalls` de la respuesta del chat:

```typescript
const calls = result.response.functionCalls();
```

El agente de IA procesará el prompt y determinará qué llamadas a funciones coinciden con él de acuerdo con las herramientas definidas. El objeto de respuesta contendrá una lista de todas las funciones.

4. Inserta la siguiente sentencia condicional al final del método:

```typescript
if (calls && calls[0].name === 'addTicket') {
  const args = calls[0].args as Record<string, string>;
  this.createTicket(
    args['plateNo'],
    new Date(args['arrival']),
    args['location']
  );
}
```

Las respuestas de la IA son impredecibles y pueden contener errores. Debemos validar la respuesta del agente de IA antes de procesarla. La sentencia condicional verifica si la respuesta es una lista y que el primer elemento coincide con el nombre de la declaración de función utilizado en el constructor.

Si la respuesta es correcta, extraemos los argumentos de la llamada a la función y los pasamos al método `createTicket` como parámetros.

Para verificar que la comunicación con el agente de IA funciona, intentemos crear un ticket con el editor de IA:

1. Ejecuta la aplicación con `ng serve` y navega a [http://localhost:4200](http://localhost:4200/).
2. Haz clic en el enlace Tickets y verifica que el formulario de tickets se vea como el siguiente:

> *Figura 5.10 – Formulario de tickets*

3. Haz clic en el enlace AI Creator y aparecerá el siguiente modal:

> *Figura 5.11 – AI Creator*

La imagen anterior muestra los campos del modal AI Creator.

4. Introduce un prompt adecuado en la entrada de texto, como el siguiente, y haz clic en el botón OK:

```text
Add ABC123 arrived on 22/12/2025 at 1:00 pm in Athens, Greece
```

5. Observa la salida en la consola del navegador:

> *Figura 5.12 – Salida de consola*

La imagen anterior muestra datos de tickets de ejemplo. La salida puede no coincidir exactamente con la de tu caso. Intenta experimentar con diferentes prompts para ayudar al modelo a producir la salida deseada.

La integración de la API de Gemini permite a los usuarios interactuar con nuestra aplicación y agregar tickets a través de una interfaz de texto conversacional. Los usuarios pueden especificar la ubicación de la plaza de estacionamiento como una dirección y Gemini la convertirá en coordenadas. En la siguiente sección, aprenderemos a representar las coordenadas del ticket en un mapa.

---

### Sección: Visualización de Coches Registrados

Los usuarios de la aplicación CityPass Parking deben poder revisar los detalles de los tickets con fines de validación. Es posible que un vehículo haya excedido la duración permitida para una plaza de estacionamiento específica. La forma más conveniente de mostrar los tickets es usar un mapa y guiar a los usuarios a las ubicaciones adecuadas. Utilizaremos Google Maps para mostrar los vehículos registrados por ubicación.

En la versión actual de la aplicación, registramos los tickets en la consola del navegador. Debemos conservarlos en una ubicación de almacenamiento para poder mostrarlos más adelante en un mapa. Los guardaremos en memoria utilizando la API de Signals por simplicidad:

1. Define una interfaz para los tickets utilizando el siguiente comando de Angular CLI:

```bash
ng generate interface ticket
```

2. Abre el archivo `ticket.ts` y agrega las siguientes propiedades a la interfaz:

```typescript
plateNo: string;
arrival: Date;
location: string;
```

3. Abre el archivo `parking.ts` e importa el método `signal` y la interfaz `Ticket`:

```typescript
import { Inject, Service, signal } from '@angular/core';
import { Ticket } from './ticket';
```

4. Crea la siguiente propiedad en la clase del servicio `Parking` para conservar los nuevos tickets:

```typescript
readonly tickets = signal<Ticket[]>([]);
```

5. Modifica el método `createTicket` para que agregue un nuevo ticket al array `tickets` después de registrarlo en la consola:

```typescript
createTicket(plate: string, arrival: Date, loc: string) {
  const ticket: Ticket = {
    plateNo: plate,
    arrival: arrival,
    location: loc
  };
  console.table([ticket]);
  this.tickets.update(tickets => [
    ...tickets,
    ticket
  ]);
}
```

Google Maps utilizará el array `tickets` para cargar los detalles del ticket en el mapa. Los usuarios podrán acceder a él a través de la opción Cars en el encabezado de la aplicación. Seguiremos el mismo enfoque que en la sección Adición manual de nuevos tickets para crear una página de funcionalidad para mostrar vehículos:

1. Navega a la carpeta `pages` y ejecuta el siguiente comando:

```bash
ng generate component cars
```

2. Crea un archivo llamado `cars.routes.ts` con el siguiente contenido:

```typescript
import { Routes } from '@angular/router';
import { Cars } from './cars';

export const CARS_ROUTES: Routes = [
  {
    path: '',
    component: Cars
  }
];
```

3. Agrega las rutas de `cars` en el archivo de configuración de enrutamiento principal `app.routes.ts`:

```typescript
{
  path: 'cars',
  loadChildren: () => import('./pages/cars/cars.routes')
    .then(m => m.CARS_ROUTES)
}
```

4. Finalmente, abre el archivo `app.html` y agrega la ruta `cars` al enlace Cars:

```html
<li nz-menu-item routerLink="/cars">Cars</li>
```

El componente `cars` alojará una instancia de Google Maps para mostrar las ubicaciones de los tickets:

1. Usa el comando `ng add` para instalar la biblioteca de Google Maps para Angular:

```bash
ng add @angular/google-maps
```

2. Abre el archivo `index.html` y agrega el fragmento descrito en [https://github.com/angular/components/tree/main/src/google-maps#loading-the-api](https://github.com/angular/components/tree/main/src/google-maps#loading-the-api) después de la etiqueta `<app-root>`. Si aún no tienes una clave API de Google Maps, no la incluyas en el fragmento. Aún puedes usar el mapa, pero mostrará una marca de agua indicando que es solo para fines de desarrollo.
3. Abre el archivo `cars.ts` y agrega la siguiente sentencia de importación:

```typescript
import { GoogleMap } from '@angular/google-maps';
```

La clase `GoogleMap` es un componente de Angular para renderizar Google Maps.

4. Agrega la clase `GoogleMap` en el array `imports` del decorador del componente:

```typescript
imports: [GoogleMap]
```

5. Crea la siguiente propiedad para definir el centro y el nivel de zoom de Google Maps en nuestra aplicación:

```typescript
options: google.maps.MapOptions = {
  center: {
    lat: 37.98,
    lng: 23.72
  },
  zoom: 9
};
```

Las opciones anteriores centrarán el mapa en Atenas, Grecia. Puedes definir las coordenadas del centro según tu ubicación actual.

6. Abre el archivo `cars.html` y reemplaza su contenido con el siguiente código HTML:

```html
<google-map mapId="citypass" height="400px" width="600px" [options]="options">
</google-map>
```

El código anterior mostrará la instancia de Google Maps de `citypass` de 400 px de alto y 600 px de ancho.

7. Ejecuta la aplicación y haz clic en el enlace Cars en el encabezado de la aplicación para mostrar el mapa:

> *Figura 5.13 – Página de coches*

La imagen anterior muestra Google Maps.

La instancia de Google Maps está actualmente vacía. Agregaremos un marcador de mapa para cada nuevo ticket:

1. Abre el archivo `cars.ts` y modifica las sentencias de importación de la siguiente manera:

```typescript
import { Component, computed, inject } from '@angular/core';
import { GoogleMap, MapAdvancedMarker } from '@angular/google-maps';
import { Parking } from '../../parking';
```

La clase `MapAdvancedMarker` representa un componente de marcador en Google Maps. Un marcador de mapa no puede identificar objetos de ticket directamente, por lo que usaremos el método `computed` para derivar las posiciones de los marcadores a partir de los tickets de estacionamiento.

2. Agrega la clase `MapAdvancedMarker` al array `imports` del decorador del componente:

```typescript
imports: [GoogleMap, MapAdvancedMarker]
```

3. Inyecta el servicio de estacionamiento en la clase del componente `Cars`:

```typescript
private parkingService = inject(Parking);
```

4. Crea la siguiente propiedad signal que crea posiciones para los marcadores de mapa:

```typescript
positions = computed(() => {
  return this.parkingService.tickets().map(ticket => {
    const coords = ticket.location.split(',');
    return {
      lat: Number(coords[0]),
      lng: Number(coords[1])
    };
  });
});
```

El código anterior asume que la ubicación del ticket tiene la forma `latitud,longitud`. Itera sobre la propiedad `tickets` del servicio de estacionamiento, divide la ubicación, convierte cada parte en un número y devuelve un objeto que un marcador de Google Maps puede entender.

5. Abre el archivo `cars.html` y agrega el siguiente fragmento dentro del elemento `<google-map>`:

```html
@for (position of positions(); track position) {
  <map-advanced-marker [position]="position" />
}
```

El fragmento anterior itera sobre la propiedad `positions` y crea un componente de marcador de mapa en cada posición.

6. Haz clic en la opción Tickets en el encabezado de la aplicación y agrega algunos tickets, ya sea manualmente o usando AI Creator.
7. Navega al mapa y verifica que veas marcadores en las ubicaciones de tickets correspondientes, como las siguientes:

> *Figura 5.14 – Ubicaciones de los tickets*

La imagen muestra cómo Google Maps representará los tickets como marcadores.

Google Maps utiliza un icono de marca de posición para indicar la ubicación de un marcador en el mapa. El marcador actualmente no contiene información aparte de su ubicación. En la siguiente sección, aprenderemos cómo mostrar información adicional del ticket mientras interactuamos con el mapa.

---

### Sección: Revisión de Detalles de Tickets

Google Maps ofrece a los usuarios una descripción general rápida de todas las ubicaciones de estacionamiento. Pueden usarlo para navegar a un ticket y ver sus detalles rápidamente. Utilizaremos la API de Google Maps para mostrar los detalles del ticket sobre un marcador:

1. Abre el archivo `cars.ts` y modifica la propiedad `positions` para que devuelva un objeto personalizado que contenga el número de matrícula del automóvil junto con la ubicación del ticket:

```typescript
positions = computed(() => {
  return this.parkingService.tickets().map(ticket => {
    const coords = ticket.location.split(',');
    return {
      car: ticket.plateNo,
      location: {
        lat: Number(coords[0]),
        lng: Number(coords[1])
      }
    };
  });
});
```

Google Maps utiliza coordenadas y no puede almacenar metadatos de marcadores personalizados. Usamos la propiedad `car` para asociar un ticket con una ubicación de marcador porque cada vehículo tiene un número de matrícula único.

2. Abre el archivo `cars.html` y agrega una propiedad `title` al elemento `<map-advanced-marker>`:

```html
<map-advanced-marker [position]="position.location" [title]="position.car" />
```

La propiedad `title` muestra el número de matrícula del automóvil en un tooltip. También cambiamos el enlace `position` para usar la propiedad `location`.

3. Ejecuta la aplicación con el comando `ng serve` y agrega datos de tickets desde la página Tickets.
4. Navega a la página Cars y pasa el cursor sobre un marcador de mapa:

> *Figura 5.15 – Ubicación del ticket con número de matrícula*

La imagen anterior muestra el ticket para el automóvil con número de matrícula `ABC123`. El número de matrícula se muestra al pasar el cursor sobre el marcador.

El componente de marcador de mapa tiene capacidades limitadas para mostrar información detallada. La propiedad `title` es suficiente para proporcionar a los usuarios un consejo rápido sobre los datos subyacentes. Alternativamente, podemos usar el componente de ventana de información del mapa (*info window*) para mostrar datos personalizados como HTML. Es una ventana superpuesta que podemos mostrar cuando los usuarios hacen clic en un marcador de mapa:

1. Importa la clase `MapInfoWindow` en el archivo `cars.ts`:

```typescript
import { GoogleMap, MapAdvancedMarker, MapInfoWindow } from '@angular/google-maps';
```

2. Agrega la clase anterior al array `imports` del decorador del componente:

```typescript
imports: [GoogleMap, MapAdvancedMarker, MapInfoWindow]
```

3. Usamos el método `viewChild` del paquete `@angular/core` y creamos la siguiente propiedad en la clase del componente `Cars`:

```typescript
private readonly info = viewChild.required(MapInfoWindow);
```

La propiedad anterior nos dará acceso al componente de ventana de información del mapa.

4. Crea un método para mostrar la ventana de información del mapa:

```typescript
showTicket(marker: MapAdvancedMarker) {
  const ticket = this.parkingService.tickets().find(
    ticket => ticket.plateNo === marker.advancedMarker.title
  );
  this.info().open(
    marker,
    false,
    'Arrived at: ' + ticket?.arrival
  );
}
```

El método `showTicket` acepta el componente de marcador de mapa como parámetro para recuperar su título y posición en el DOM. Usamos el título, que representa el número de matrícula del automóvil, para encontrar los detalles específicos del ticket. La ventana de información pasa el componente de marcador al método `open` para que sepa dónde anclar la ventana en relación con el marcador. El tercer parámetro del método `open` especifica el contenido de la ventana; en este caso, muestra la fecha y hora de llegada del vehículo.

5. Abre el archivo `cars.html` y modifica el componente de marcador de mapa de la siguiente manera:

```html
<map-advanced-marker [position]="position.location" [title]="position.car" #marker="mapAdvancedMarker" (mapClick)="showTicket(marker)" />
```

En el fragmento anterior, definimos la variable de referencia de plantilla `marker` como un componente de marcador avanzado y la pasamos al método `showTicket` como parámetro. Vinculamos el método `showTicket` con el evento `mapClick` para mostrar la ventana de información cuando el usuario haga clic en un marcador.

6. Agrega el componente de ventana de información del mapa después del bloque `@for`:

```html
<map-info-window />
```

El componente de ventana de información del mapa también puede mostrar contenido HTML. En lugar de pasar su contenido a través del método `open`, podríamos mantener el ticket seleccionado en el estado del componente y usarlo de la siguiente manera:

```html
<map-info-window>
  <h1>Car {{ticket.plateNo}}</h1>
  <p>Arrived at: {{ticket.arrival | date }}</p>
</map-info-window>
```

7. Ejecuta la aplicación para agregar datos de tickets y haz clic en un marcador de mapa para ver sus detalles:

> *Figura 5.16 – Ventana de información del mapa*

La imagen muestra cómo Google Maps mostrará la información del ticket en una ventana.

La API de Google Maps mejora las aplicaciones web al aumentar la accesibilidad y agregar datos de geolocalización. Los usuarios de CityPass Parking pueden navegar al punto de referencia para encontrar ubicaciones de estacionamiento adecuadas y garantizar el uso correcto de las plazas de estacionamiento disponibles.

---

### Sección: Resumen

En este capítulo, construimos una aplicación para enviar datos de tickets para un validador de estacionamiento y hacerla accesible con tecnología de mapeo. Inicialmente, utilizamos formularios basados en plantillas de Angular para recopilar datos de entrada y nos aseguramos de que los usuarios los completaran correctamente mediante reglas de validación. También les permitimos ser más creativos al hacer que interactúen con un asistente de IA a través de Gemini para ingresar cómodamente los datos requeridos del ticket. Finalmente, utilizamos Google Maps para visualizar y mostrar los datos recopilados en un mapa. En el siguiente capítulo, aprenderemos cómo usar herramientas de IA desde el backend construyendo una aplicación de reservas en línea para un estudio de música.

---

### Sección: Ejercicios

Utiliza la biblioteca NG-ZORRO para mostrar una notificación cuando el usuario envíe un ticket con éxito. Para la entrada manual, asegúrate de que los campos del formulario se restablezcan a su estado inicial. Mientras usas AI Creator, deshabilita el botón Submit y limpia la entrada del prompt.

---

### Sección: Lecturas Complementarias

- **NG-ZORRO:** [https://ng.ant.design](https://ng.ant.design/)
- **Formularios basados en plantillas de Angular:** [https://angular.dev/guide/forms/template-driven-forms](https://angular.dev/guide/forms/template-driven-forms)
- **Primeros pasos con Firebase AI Logic:** [https://firebase.google.com/docs/ai-logic](https://firebase.google.com/docs/ai-logic)
- **Google Maps para Angular:** [https://github.com/angular/components/tree/main/src/google-maps](https://github.com/angular/components/tree/main/src/google-maps)
