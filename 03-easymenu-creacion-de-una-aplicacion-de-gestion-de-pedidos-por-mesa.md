# Parte 1: Primeros Pasos con Angular e Inteligencia Artificial

## Capítulo 3: EasyMenu: Creación de una Aplicación de Gestión de Pedidos por Mesa

### Sección: Introducción

La elección de una solución de almacenamiento para una aplicación Angular es un proceso desafiante que depende de las necesidades del negocio. Los arquitectos de soluciones deben seleccionar un proveedor de almacenamiento adecuado en función de los requisitos de velocidad, disponibilidad, latencia y escalabilidad del proyecto. Cloud Firestore es una opción perfecta para aplicaciones de línea de negocio (*line-of-business*), como restaurantes y almacenes, donde los usuarios deben interactuar con los datos en tiempo real.

En este capítulo, construiremos una aplicación para ayudar al personal de un restaurante a tomar pedidos rápidamente para que el equipo de cocina pueda prepararlos. Utilizaremos Cloud Firestore para almacenar datos y ponerlos a disposición de los usuarios al instante. También utilizaremos la biblioteca Angular Material para ofrecer una experiencia de usuario fluida y rápida. La combinación de la velocidad de Firestore y la simplicidad de Angular Material nos permitirá crear una aplicación que a los usuarios les encantará utilizar.

Vamos a cubrir los siguientes temas:

- Configuración del almacenamiento de datos
- Instalación de Angular Material
- Visualización de mesas
- Gestión de pedidos

---

### Sección: Requisitos Técnicos

Puedes descargar el proyecto de ejemplo y el código de este libro siguiendo las instrucciones de la sección *Download the example code files* en el Prefacio de este libro.

Los archivos de código de este capítulo están incluidos en el paquete de código descargable.

Utilizaremos el código de la carpeta `ch01` como punto de partida para construir la aplicación de este capítulo.

---

### Sección: Configuración del Almacenamiento de Datos

Cloud Firestore es una potente base de datos de documentos NoSQL que puede proporcionar actualizaciones en tiempo real a los clientes conectados. Es una solución de almacenamiento robusta y escalable que sincroniza datos entre dispositivos. Utilizaremos la consola de Firebase para configurar Firestore para nuestra aplicación. Primero, debemos crear un proyecto de Firebase en la consola:

1. Navega a [https://console.firebase.google.com](https://console.firebase.google.com/). Necesitarás una cuenta de Google para iniciar sesión en la consola de Firebase.
2. Selecciona la opción **Create a new Firebase project**.
3. Escribe `easymenu` en el campo Project name y haz clic en el botón **Continue**.
4. Haz clic en **Continue** en la página AI assistance for your Firebase project. La asistencia de IA no es obligatoria para nuestra aplicación, pero se recomienda en caso de que desees agregar funciones inteligentes en el futuro.
5. Desactiva la opción **Enable Google Analytics for this project** y haz clic en el botón **Create project**. Analytics no es necesario para nuestra aplicación, pero puedes habilitarlo más adelante si deseas agregar funciones de monitoreo.
6. Espera a que se complete el proceso de creación del proyecto y haz clic en el botón **Continue**.

Ahora que tenemos un proyecto de Firebase, podemos agregar el servicio Firestore:

1. Expande la barra lateral izquierda y selecciona la opción **Firestore** del menú Databases and storage.
2. Haz clic en el botón **Create database** para iniciar el asistente de creación de base de datos.
3. Selecciona una edición de base de datos en el paso Select edition del asistente y haz clic en el botón **Next**. La opción recomendada para proyectos con conjuntos de datos pequeños, como los de este capítulo, es la edición Standard.
4. Selecciona una ubicación cercana a ti en el paso Database ID and location del asistente y haz clic en el botón **Next**.
5. Selecciona el modo de trabajo de la base de datos en el paso Configure del asistente y haz clic en el botón **Create**. La opción recomendada es el modo de prueba (*test mode*) durante el desarrollo de la aplicación.

Firebase agregará Firestore al proyecto actual de Firebase y creará una base de datos inicialmente vacía. La base de datos almacenará una lista de mesas de restaurante junto con los detalles de los pedidos.

De acuerdo con las especificaciones de la aplicación, una mesa solo puede tener un pedido activo en cualquier momento dado.

Utilizaremos la vista de panel de Firestore para agregar mesas a la base de datos:

1. Haz clic en el enlace **Start collection** para crear una nueva colección en la base de datos.
2. Escribe `tables` en el campo Collection ID y haz clic en el botón **Next**.
3. Introduce el número `1` en el campo Document ID, elimina cualquier campo adicional y haz clic en el botón **Save**. El campo Document ID representa el número de mesa.
4. Utiliza el enlace **Add document** para agregar más mesas a la colección.

La configuración de Cloud Firestore es simple y rápida, lo que te permite ponerte en marcha rápidamente con una solución de almacenamiento de datos robusta. En la siguiente sección, aprenderemos cómo agregar una biblioteca de componentes de interfaz de usuario para diseñar la apariencia visual de la aplicación.

---

### Sección: Instalación de Angular Material

La biblioteca Angular Material es una biblioteca de componentes de interfaz de usuario mantenida por el equipo de Angular. Cuenta con una colección completa de componentes de UI, que incluye una vista de cuadrícula flexible y un cuadro de diálogo que utilizaremos para visualizar mesas y solicitar acciones al usuario en la aplicación.

En esta sección, aprenderás a instalar y configurar Angular Material en una aplicación Angular. Utilizaremos la aplicación del Capítulo 1, *Angular AI Kick-Starter: Scaffolding Smart Apps with Copilot*, como punto de partida y agregaremos un componente de encabezado de Angular Material para mostrar el título del capítulo.

Comencemos instalando Angular Material:

1. Inicia el proceso de instalación ejecutando el siguiente comando dentro del proyecto de Angular CLI:

```bash
ng add @angular/material
```

El comando anterior iniciará un proceso que te guiará al agregar Angular Material. El proceso implica hacer preguntas para proporcionar más contexto en la CLI sobre la aplicación que deseas construir.

El proceso preguntará qué paleta de colores nos gustaría usar en nuestra aplicación. Angular Material admite las siguientes paletas prediseñadas:

- Azure/Blue
- Rose/Red
- Magenta/Violet
- Cyan/Orange

Usa las flechas del teclado para resaltar una opción y presiona Enter. Los ejemplos de este capítulo se han creado con la paleta de colores Azure/Blue.

2. Abre el archivo `app.ts` y crea el siguiente constructor para establecer el título del capítulo:

```typescript
constructor() {
  this.chapterTitleService.setTitle(
    '[Chapter 3](https://subscription.packtpub.com/book/programming/9781806668472/3): EasyMenu'
  );
}
```

3. Ejecuta la aplicación usando el comando `ng serve` y navega a `localhost:4200`:

> *Figura 3.1 – Salida de la aplicación*

La instalación de Angular Material cambia el estilo de la página como se muestra en la figura anterior. La página principal ahora muestra el título del capítulo actual. Mostraremos el título en el encabezado de la aplicación:

1. Abre el archivo `app.html` y reemplaza su contenido con el siguiente código HTML:

```html
<chapter-title [chapterTitle]="title()" />
```

2. Abre el archivo `chapter-title.component.ts` y agrega la siguiente sentencia de importación:

```typescript
import { MatToolbar } from '@angular/material/toolbar';
```

La clase `MatToolbar` contiene el componente de encabezado que necesitamos para mostrar el título del capítulo.

3. Reemplaza la propiedad `template` del decorador del componente con el siguiente código HTML:

```html
<mat-toolbar>
  <span>{{ chapterTitle() }}</span>
</mat-toolbar>
```

El `mat-toolbar` es un componente de Angular Material que representa el encabezado de una aplicación Angular. Los componentes de Angular Material comienzan con el prefijo `mat`.

4. Agrega la clase `MatToolbar` al array `imports` del decorador del componente:

```typescript
imports: [MatToolbar]
```

Verifica que la salida de la aplicación muestre lo siguiente: `[Chapter 3](https://subscription.packtpub.com/book/programming/9781806668472/3): EasyMenu`

El diseño de la aplicación ahora sigue las directrices de Angular Material. Cuenta con un encabezado que muestra el título del capítulo. En la siguiente sección, aprenderemos a interactuar con Cloud Firestore y a gestionar las mesas del restaurante.

---

### Sección: Visualización de Mesas

La familia de productos de Firebase proporciona SDKs para varias plataformas de desarrollo, frameworks, herramientas y bibliotecas que desean interactuar con sus servicios. Puedes encontrar una lista completa en [https://firebase.google.com/docs/libraries](https://firebase.google.com/docs/libraries).

En esta sección, aprenderemos a utilizar el SDK web de Firebase para interactuar con Firestore y mostrar las mesas disponibles.

Las aplicaciones de Angular pueden utilizar los siguientes SDKs web de Firebase:

- API de JavaScript de Firebase
- Biblioteca AngularFire

La biblioteca AngularFire es un wrapper de Angular que proporciona métodos convenientes para utilizar la API de JavaScript de Firebase. Podemos instalarla en nuestra aplicación utilizando la Angular CLI:

1. Inicia el proceso de instalación de la biblioteca AngularFire mediante el siguiente comando:

```bash
ng add @angular/fire
```

El proceso implica hacer preguntas para proporcionar más contexto en la CLI sobre el proyecto de Firebase que creamos en la sección Configuración del almacenamiento de datos.

Si tienes problemas para ejecutar el comando anterior, consulta el archivo `CHANGELOG.md` del repositorio de GitHub del capítulo. Para acceder al enlace del repositorio, sigue los pasos de la sección "Download the example code files" en el Prefacio.

2. La primera pregunta es sobre qué características de Firebase nos gustaría usar en nuestra aplicación. Selecciona la opción **Firestore** y presiona Enter para continuar.
3. En la siguiente pregunta, la CLI preguntará si queremos usar Gemini con Firebase. Aunque no utilizaremos Gemini en esta aplicación, se recomienda habilitarlo si planeas agregar funciones de IA en una versión futura, como sugerir artículos populares y resumir pedidos para el personal de cocina. Presiona Enter para habilitar Gemini en Firebase.
4. En la siguiente pregunta, podemos optar por permitir que Firebase recopile datos de uso y análisis de la aplicación.
5. La Angular CLI nos solicitará autenticarnos con Firebase antes de continuar. Si ya estás usando Firebase en tu sistema, te pedirá que selecciones la cuenta que utilizas.
6. En la siguiente pregunta, selecciona la opción **easymenu** de la lista de proyectos y presiona Enter.
7. En la última pregunta, debemos asociar el proyecto de Firebase con una aplicación. Asegúrate de que la opción **[CREATE NEW APP]** esté seleccionada y presiona Enter.
8. Introduce `easymenu` como nombre de la aplicación y presiona Enter.

La Angular CLI descarga e instala la biblioteca AngularFire y crea la configuración de Firebase en `app.config.ts`.

Si la configuración de Firebase contiene algún error, consulta el archivo `CHANGELOG.md` del repositorio de GitHub del capítulo. Para acceder al enlace del repositorio, sigue los pasos de la sección "Download the example code files" en el Prefacio.

Utilizaremos la Angular CLI para crear un componente que muestre las mesas desde Firestore:

1. Ejecuta el siguiente comando para crear el componente:

```bash
ng generate component table-list
```

2. Abre el archivo `table-list.ts` y agrega las siguientes sentencias de importación:

```typescript
import { MatGridListModule } from '@angular/material/grid-list';
import { MatCardModule } from '@angular/material/card';
import { collection, collectionData, Firestore } from '@angular/fire/firestore';
```

La salida de la aplicación mostrará las mesas como componentes de tarjetas con un diseño de componente de cuadrícula de la biblioteca Angular Material.

La clase `Firestore` es un servicio de Angular que inyecta Firestore en el componente. Los métodos `collection` y `collectionData` son asistentes para obtener una colección junto con sus datos desde Firestore.

3. Agrega las clases anteriores en el array `imports` del decorador del componente:

```typescript
imports: [MatGridListModule, MatCardModule]
```

4. Usa el método `inject` del paquete `@angular/core` para inyectar el servicio `Firestore` en la clase `TableList`:

```typescript
private firestore = inject(Firestore);
```

5. Agrega las siguientes propiedades para obtener los datos de las mesas de la base de datos de Firestore:

```typescript
private tableCol = collection(this.firestore, 'tables');
readonly tables = toSignal(collectionData(this.tableCol));
```

El método `collection` acepta el nombre de la colección de Firebase como segundo parámetro. Crea la colección y la pasa al método `collectionData`.

Utilizamos el método `toSignal` del paquete `@angular/core/rxjs-interop` para convertir el observable devuelto por el método `collectionData` en una signal.

6. Abre el archivo `table-list.html` y reemplaza su contenido con el siguiente código HTML:

```html
<mat-grid-list cols="2" rowHeight="350px">
  @for (table of tables() ; track idx; let idx = $index) {
    @let tableId = idx + 1;
    <mat-grid-tile>
    </mat-grid-tile>
  }
</mat-grid-list>
```

El `mat-grid-list` define un componente de cuadrícula con dos columnas y una altura de fila específica. La cuadrícula contiene un componente `mat-grid-tile` para cada mesa de la colección. Calculamos el ID de la mesa a partir del índice de la mesa actual utilizando un bloque `@for`.

7. Inserta el siguiente fragmento dentro del componente de celda de cuadrícula:

```html
<mat-card>
  <mat-card-header>
    <mat-card-title># {{tableId}}</mat-card-title>
  </mat-card-header>
</mat-card>
```

Cada celda de cuadrícula contiene un componente de tarjeta con un encabezado, lo que permite que la cuadrícula se ajuste en función del número total de mesas. El componente de encabezado contiene un componente `mat-card-title` para mostrar el número de mesa.

8. Abre el archivo `table-list.scss` y agrega el siguiente código CSS para dar estilo al componente de tarjeta:

```scss
mat-card {
  position: absolute;
  top: 1.5rem;
  left: 1.5rem;
  right: 1.5rem;
  bottom: 1.5rem;
}
```

El componente de lista de mesas ya está listo para mostrar nuestras mesas desde la base de datos de Firestore. Debemos agregarlo al componente principal de la aplicación:

1. Abre el archivo `app.html` y agrega el siguiente fragmento debajo del selector `chapter-title`:

```html
<app-table-list />
```

2. Abre el archivo `app.ts` e importa la clase del componente `TableList`:

```typescript
import { TableList } from './table-list/table-list';
```

3. Agrega la clase en el array `imports` del decorador del componente:

```typescript
imports: [ChapterTitleComponent, TableList]
```

Si ejecutamos nuestra aplicación Angular utilizando `ng serve`, la salida se verá de la siguiente manera:

> *Figura 3.2 – Lista de mesas*

La salida de la aplicación depende de cuántas mesas hayas agregado a la base de datos. La figura anterior muestra un restaurante con cuatro mesas.

Nuestra aplicación tiene una base sólida que nos permite integrarnos con Firestore y mostrar los datos almacenados. La página principal de la aplicación muestra cada mesa mediante una representación en tarjeta dentro de un diseño de cuadrícula. En la siguiente sección, aprenderemos cómo agregar pedidos y gestionar los elementos de su menú.

---

### Sección: Gestión de Pedidos

Cada mesa de la aplicación solo puede tener un pedido activo a la vez. El flujo de trabajo del usuario comienza creando el pedido y agregando elementos del menú. Durante el proceso, los usuarios pueden agregar más artículos al pedido. Cuando el cliente está listo para pagar, el usuario completará el pedido y la mesa estará disponible para uno nuevo. El flujo de trabajo se puede resumir en los siguientes pasos:

- Creación de pedidos
- Gestión de artículos
- Guardado de pedidos

Comenzaremos desarrollando el flujo de trabajo para crear un nuevo pedido en una mesa.

#### Creación de pedidos

El proceso para tomar un nuevo pedido debe ser rápido y contener mínimas distracciones en la interfaz de usuario de la aplicación. Un cuadro de diálogo modal satisface ambos requisitos porque:

- Se puede abrir tocando la tarjeta de la lista de mesas.
- Atrae la atención del usuario mostrando el contenido en primer plano y posicionando el resto en segundo plano.

Comenzaremos a crear el cuadro de diálogo modal:

1. Crea el nuevo componente que alojará el diálogo mediante el siguiente comando:

```bash
ng generate component order
```

2. Abre el archivo `order.ts` y agrega las siguientes sentencias de importación:

```typescript
import { MatListModule } from '@angular/material/list';
import { MatButton } from '@angular/material/button';
import { MatDivider } from '@angular/material/divider';
import { MAT_DIALOG_DATA, MatDialogModule } from '@angular/material/dialog';
```

Cada una de las clases anteriores representa un componente de Angular Material que necesitaremos para el diálogo modal. El token `MAT_DIALOG_DATA` proporciona acceso a los datos pasados al diálogo.

3. Agrega las clases anteriores al array `imports` del decorador del componente:

```typescript
imports: [
  MatDialogModule,
  MatListModule,
  MatButton,
  MatDivider
]
```

4. Copia el archivo `menu.ts` de la sección del repositorio de GitHub en la carpeta `app` e impórtalo. Para acceder al enlace del repositorio, sigue los pasos de la sección "Download the example code files" en el Prefacio.

```typescript
import { menu } from '../menu';
```

El archivo contiene datos de ejemplo para el menú del restaurante. Expone los datos como un array de objetos donde cada objeto representa una categoría con elementos del menú como los siguientes:

```json
{
  "name": "Starters",
  "items": [
    { "name": "Baked Feta Cheese", "price": 5.5 },
    { "name": "Fried Zucchini Sticks", "price": 5 },
    { "name": "Fried Potatoes", "price": 4.5 },
    { "name": "Tzatziki", "price": 4.5 }
  ]
}
```

En el fragmento anterior, la propiedad `name` es la categoría y la propiedad `items` son los elementos del menú de la categoría. Cada artículo tiene un nombre y un precio.

5. Agrega las siguientes propiedades a la clase del componente `Order`:

```typescript
data = inject(MAT_DIALOG_DATA);
menu = menu;
```

Usamos el método `inject` del paquete `@angular/core` para acceder a los datos pasados al diálogo. También creamos una propiedad `menu` para vincular el menú del restaurante a la plantilla del componente.

6. Abre el archivo `order.html` y reemplaza su contenido con el siguiente elemento `h2`:

```html
<h2 mat-dialog-title>
  # {{data}}
</h2>
```

La directiva `mat-dialog-title` representa el título del diálogo.

7. Inserta el siguiente fragmento HTML debajo del título:

```html
<mat-dialog-content></mat-dialog-content>
```

El selector `mat-dialog-content` es el anfitrión del contenido principal del diálogo.

8. Inserta el siguiente fragmento HTML dentro del contenido principal:

```html
<mat-list>
  @for (m of menu; track $index) {
    <div mat-subheader>{{m.name}}</div>
    @for (item of m.items; track $index) {
      <mat-list-item>{{item.name}}</mat-list-item>
    }
    <mat-divider />
  }
</mat-list>
```

El selector `mat-list` es un componente de lista de Angular Material que muestra los elementos del menú agrupados por categoría. La directiva `mat-subheader` muestra la categoría del menú y el componente `mat-list-item` muestra los artículos de la categoría. Usamos el componente `mat-divider` para agregar una línea divisoria entre cada categoría.

9. Inserta el siguiente fragmento después del contenido principal para definir las acciones del diálogo:

```html
<mat-dialog-actions>
  <button mat-button mat-dialog-close>Cancel</button>
  <button mat-flat-button>Ok</button>
</mat-dialog-actions>
```

El componente `mat-dialog-actions` incluye dos botones. El botón Cancel contiene la directiva `mat-dialog-close` para cerrar el diálogo. El botón Ok cerrará el diálogo y devolverá los artículos del pedido al componente invocador, como aprenderemos en la siguiente sección.

El diálogo modal muestra la mesa seleccionada y los elementos disponibles del menú. Conectaremos el diálogo para que se abra desde la lista de mesas:

1. Abre el archivo `table-list.ts` y agrega las siguientes sentencias de importación:

```typescript
import { Order } from '../order/order';
import { MatDialog } from '@angular/material/dialog';
```

La clase `Order` es el componente de diálogo y la clase `MatDialog` es un servicio de Angular Material para abrir diálogos.

2. Crea la siguiente propiedad en la clase del componente `TableList` para inyectar el servicio de diálogo:

```typescript
private dialog = inject(MatDialog);
```

3. Agrega el siguiente método para abrir el componente de diálogo de pedido:

```typescript
select(no: number) {
  this.dialog.open(Order, {
    width: '500px',
    data: no
  });
}
```

Usamos el método `open` del servicio de diálogo para abrir el componente de diálogo. También pasamos un objeto de configuración que define el ancho de la ventana y los datos que deseamos pasar al componente.

4. Abre el archivo `table-list.html` y agrega el siguiente evento de clic en el componente `mat-grid-tile`:

```html
<mat-grid-tile (click)="select(tableId)">
  <mat-card>
    <mat-card-header>
      <mat-card-title># {{tableId}}</mat-card-title>
    </mat-card-header>
  </mat-card>
</mat-grid-tile>
```

La aplicación llamará al método `select` cuando hagamos clic en una mesa en particular.

5. Ejecuta la aplicación y haz clic en la tarjeta de una mesa para abrir el menú:

> *Figura 3.3 – Diálogo de pedido*

El diálogo muestra el número de mesa seleccionado en el encabezado y los elementos del menú en el cuerpo. Los usuarios pueden desplazarse por el menú para ver los artículos disponibles. También contiene botones de acción en el pie de página del diálogo.

Los diálogos son un componente central de la aplicación porque los utilizaremos para completar casi todas las acciones. El diálogo de pedido muestra una mesa seleccionada y los elementos del menú disponibles. En la siguiente sección, aprenderemos cómo agregar o eliminar artículos de un pedido en particular.

#### Gestión de artículos

El diálogo de pedido muestra el número de mesa actual y los elementos del menú disponibles. El menú es una lista de solo lectura que muestra los artículos y sus categorías. En esta sección, crearemos un nuevo componente que permita a los usuarios agregar o eliminar artículos del menú al pedido seleccionado:

1. Ejecuta el siguiente comando para crear una interfaz que describa la estructura de los artículos del pedido:

```bash
ng generate interface item
```

2. Abre el archivo `item.ts` y agrega las siguientes propiedades a la interfaz `Item`:

```typescript
export interface Item {
  name: string;
  qty: number;
}
```

La propiedad `name` representa el nombre del elemento del menú.

3. Ejecuta el siguiente comando para crear el componente de artículo de pedido:

```bash
ng generate component order-item
```

4. Abre el archivo `order-item.ts` y agrega las siguientes sentencias de importación:

```typescript
import { MatIconButton } from '@angular/material/button';
import { MatIcon } from '@angular/material/icon';
```

El componente contendrá dos botones con iconos para aumentar o disminuir la cantidad del artículo en el pedido.

5. Agrega las clases anteriores en el array `imports` del decorador del componente:

```typescript
imports: [MatIconButton, MatIcon]
```

6. Declara las siguientes propiedades en la clase del componente `OrderItem`:

```typescript
readonly name = input('');
readonly qty = model(0);
```

Usamos los métodos `model` e `input` del paquete npm `@angular/core` para obtener el nombre del artículo actual y modificar su cantidad.

7. Agrega los siguientes métodos a la clase del componente `OrderItem` para incrementar y decrementar la cantidad del artículo:

```typescript
add() {
  this.qty.update(qty => qty + 1);
}

subtract() {
  this.qty.update(qty => qty - 1);
}
```

8. Abre el archivo `order-item.html` y reemplaza su contenido con el siguiente fragmento HTML:

```html
<p [class.selected]="qty()">{{name()}}</p>
<button mat-icon-button>
  <mat-icon (click)="add()">add</mat-icon>
</button>
<span>{{qty()}}</span>
<button (click)="subtract()" [disabled]="!qty()" mat-icon-button>
  <mat-icon>remove</mat-icon>
</button>
```

La plantilla del componente muestra el nombre del artículo y la cantidad, e incluye dos botones para aumentar o disminuir la cantidad. Este último se deshabilita cuando la cantidad es cero. La aplicación agrega la clase CSS `selected` para resaltar los artículos que se han agregado al menos una vez al pedido.

9. Abre el archivo `order-item.scss` y agrega los siguientes estilos CSS:

```scss
:host {
  display: flex;
}

p {
  flex: 1;
  margin: 7px;
}

span {
  margin: 7px;
}

.selected {
  background: #00adf1;
}
```

En el archivo anterior, hemos agregado estilos para los elementos de nombre y cantidad. También hemos agregado un color de fondo para que el artículo seleccionado se destaque del resto de los elementos del menú.

Para usar el componente de artículo de pedido, debemos agregarlo al diálogo de pedido:

1. Abre el archivo `order.ts` y agrega la siguiente sentencia de importación:

```typescript
import { OrderItem } from '../order-item/order-item';
```

2. Agrega la clase `OrderItem` al array `imports` del decorador del componente:

```typescript
imports: [
  MatDialogModule,
  MatListModule,
  MatButton,
  MatDivider,
  OrderItem
]
```

3. Abre el archivo `order.html` y reemplaza el contenido del componente `mat-list-item` con el siguiente código HTML:

```html
<app-order-item [name]="item.name" />
```

4. Ejecuta la aplicación y selecciona una mesa. El diálogo de pedido debería verse como el siguiente:

> *Figura 3.4 – Diálogo de pedido*

El diálogo de pedido muestra todos los elementos del menú, cada uno con botones de control para aumentar o disminuir su cantidad.

5. Haz clic en el botón de aumento en un artículo y la aplicación lo resaltará agregando un color de fondo:

> *Figura 3.5 – Diálogo de pedido con artículos seleccionados*

Hemos alcanzado un punto en el que nuestro proceso de toma de pedidos es completamente funcional. Los usuarios pueden seleccionar una mesa y agregar artículos del menú del restaurante al pedido actual. En la siguiente sección, completaremos el flujo de trabajo de gestión de pedidos guardando los artículos seleccionados en la base de datos de Firestore.

#### Guardado de pedidos

Cuando el usuario esté listo para guardar un pedido, hará clic en el botón Ok del diálogo de pedido y la aplicación enviará los detalles del pedido a la base de datos de Firestore para su procesamiento posterior.

El componente de diálogo de pedido es responsable de recopilar los artículos del pedido y enviarlos al componente de lista de mesas:

1. Abre el archivo `order.ts` e importa el método `viewChildren`:

```typescript
import { Component, inject, viewChildren } from '@angular/core';
```

2. Declara la siguiente propiedad en la clase del componente `Order`:

```typescript
private readonly orderItems = viewChildren(OrderItem);
```

La propiedad `orderItems` devuelve todas las instancias del componente `OrderItem`.

3. Importa la clase `MatDialogRef` de la biblioteca Angular Material:

```typescript
import { MAT_DIALOG_DATA, MatDialogModule, MatDialogRef } from '@angular/material/dialog';
```

La clase `MatDialogRef` es un servicio de Angular que nos brinda acceso al componente de diálogo subyacente.

4. Inyecta el servicio en la clase del componente:

```typescript
private dialogRef = inject(MatDialogRef);
```

5. Agrega el siguiente método que cierra el diálogo y devuelve los artículos del pedido al componente invocador:

```typescript
ok() {
  const items = this.orderItems()
    .filter(item => item.qty())
    .map(i => {
      return {
        name: i.name(),
        qty: i.qty()
      }
    });
  this.dialogRef.close(items);
}
```

El método `ok` itera sobre las instancias del componente `OrderItem` y devuelve una nueva colección de objetos de artículos de pedido. En lugar de pasar el artículo de pedido completo, lo convertimos en un objeto plano más simple para reducir la huella de datos en Firestore.

6. Abre el archivo `order.html` y vincula el método al evento `click` del segundo botón:

```html
<mat-dialog-actions>
  <button mat-button mat-dialog-close>Cancel</button>
  <button mat-flat-button (click)="ok()">Ok</button>
</mat-dialog-actions>
```

El componente de lista de mesas puede escuchar el evento de cierre del diálogo y obtener los artículos del pedido para guardarlos en Firestore:

1. Abre el archivo `table-list.ts` e importa los métodos `doc` y `updateDoc` de la biblioteca AngularFire:

```typescript
import { collection, collectionData, Firestore, doc, updateDoc } from '@angular/fire/firestore';
```

Necesitaremos ambos métodos para guardar datos en Firestore.

2. Suscríbete al observable `afterClosed` del diálogo abierto:

```typescript
select(no: number) {
  this.dialog.open(Order, {
    width: '500px',
    data: no
  }).afterClosed().subscribe(items => {
    if (items) {
    }
  });
}
```

Debemos verificar si el diálogo devuelve algún artículo, ya que el usuario puede cerrarlo mediante el botón Cancel.

3. Ajusta el callback de suscripción del observable `afterClosed` de la siguiente manera:

```typescript
async items => {
  if (items) {
    const tableDoc = doc(this.tableCol, no.toString());
    await updateDoc(tableDoc, {items});
  }
}
```

En el fragmento anterior, usamos el método `doc` para obtener el documento de Firestore para el número de mesa seleccionado. Luego llamamos al método `updateDoc`, pasando los artículos del pedido al documento de la mesa para que podamos guardarlo en la base de datos.

4. Ejecuta la aplicación y agrega un elemento de Baked Feta Cheese, Tzatziki y Greek Salad a la primera mesa.
5. Navega a la opción Firestore en la consola de Firebase.
6. Selecciona la primera mesa y deberías ver los artículos del pedido en el panel de la colección:

> *Figura 3.6 – Colección de Firestore*

La colección `items` es una lista de objetos `OrderItem`. En la imagen anterior, la mesa contiene tres artículos.

El último paso del flujo de trabajo de pedidos ya está completo. Los usuarios pueden guardar el pedido de una mesa seleccionada en la base de datos de Firestore para que otras aplicaciones puedan procesarlo, como el personal de cocina que prepara el pedido. De hecho, podríamos usar la misma aplicación porque los datos se guardan en tiempo real y cada nuevo pedido aparecería automáticamente en todas las aplicaciones conectadas.

---

### Sección: Resumen

En este capítulo, aprendimos a construir un sistema de gestión de pedidos por mesa que ayuda al personal del restaurante a trabajar de manera más eficiente. Comprendimos cómo podemos beneficiarnos de Cloud Firestore para que los datos estén disponibles en diferentes secciones del restaurante en tiempo real. Diseñamos la interfaz de usuario de la aplicación utilizando la biblioteca Angular Material, proporcionando una experiencia de usuario (UX) receptiva y consistente. En el siguiente capítulo, cambiaremos a la industria de gestión de almacenes y aprenderemos a construir una aplicación para la preparación de pedidos (*picking*).

---

### Sección: Ejercicios

Utiliza la biblioteca AngularFire para mostrar un resumen del pedido, incluido el coste total, dentro de la tarjeta de la mesa.

---

### Sección: Lecturas Complementarias

- **Cloud Firestore:** [https://firebase.google.com/products/firestore](https://firebase.google.com/products/firestore)
- **AngularFire:** [https://github.com/angular/angularfire](https://github.com/angular/angularfire)
- **Componentes de Angular Material:** [https://material.angular.dev/components/categories](https://material.angular.dev/components/categories)
