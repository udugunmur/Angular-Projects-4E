# Parte 1: Primeros Pasos con Angular e Inteligencia Artificial

## Capítulo 9: Flash POS: Construcción de una Aplicación de Punto de Venta Moderna con Gestión de Estado

### Sección: Introducción

El equipamiento tecnológico de los restaurantes modernos generalmente cubre todas las etapas del servicio al cliente, desde servir comida en las mesas hasta la preparación para llevar (*takeaway*). En el Capítulo 3, *EasyMenu: Creating a Table Order Management App*, creamos una aplicación para ayudar al personal de un restaurante a tomar pedidos de las mesas. En este capítulo, ampliaremos las capacidades del restaurante creando una aplicación de Punto de Venta (*Point of Sale* o POS) para clientes de comida para llevar.

Vamos a cubrir los siguientes temas:

- Instalación de PrimeNG
- Visualización de categorías
- Visualización de artículos
- Visualización del carrito de compras

---

### Sección: Requisitos Técnicos

Todos los ejemplos de código de este capítulo se pueden encontrar en las carpetas `ch01` y `ch09`. Puedes descargar el proyecto de ejemplo y el código de este libro siguiendo las instrucciones de la sección *Download the example code files* en el Prefacio de este libro.

Los archivos de código de este capítulo están incluidos en el paquete de código descargable. Utilizaremos el código de la carpeta `ch01` como nuestro punto de partida para construir la aplicación de este capítulo.

---

### Sección: Instalación de PrimeNG

PrimeNG es una suite popular para el framework Angular que incluye componentes de interfaz de usuario altamente personalizables. Cuenta con una rica colección de plantillas, tanto gratuitas como de pago, para ayudarte a comenzar en tu próximo proyecto. También admite temas integrados que puedes modificar según las necesidades de tu aplicación.

Como hacemos en todos los capítulos de este libro, utilizaremos la aplicación del Capítulo 1, *Angular AI Kick-Starter: Scaffolding Smart Apps with Copilot*, como nuestro punto de partida y agregaremos un componente toolbar de PrimeNG para mostrar el título del capítulo.

Cubrimos el proceso de instalación de PrimeNG en la sección *Instalación de PrimeNG* del Capítulo 6, *Studio BookMaster: Designing an AI-Enhanced Room Booking App*. Repite los pasos de esa sección para instalar la biblioteca para este capítulo.

Los ejemplos de este capítulo están construidos utilizando el tema Aura de la biblioteca PrimeNG.

En este capítulo, también utilizaremos PrimeIcons, una colección de iconos predeterminados para PrimeNG, para mejorar la experiencia de usuario de nuestra aplicación:

1. Ejecuta el siguiente comando para instalar el paquete `primeicons`:

```bash
npm install primeicons
```

2. Abre el archivo `styles.scss` y agrega la siguiente sentencia en la parte superior del archivo:

```scss
@import "primeicons/primeicons.css";
```

La sentencia anterior importará PrimeIcons globalmente en la aplicación.

Ahora modificaremos el componente de encabezado de la aplicación para adaptarlo a las necesidades del proyecto actual:

1. Abre el archivo `app.ts` y modifica el constructor de la siguiente manera:

```typescript
constructor() {
  this.chapterTitleService.setTitle(
    'Chapter 9: Flash POS'
  );
}
```

2. Abre el archivo `app.config.ts` e importa el tema Aura usando la siguiente sentencia:

```typescript
import Aura from '@primeuix/themes/aura';
```

3. Usa el array `providers` para proporcionar el tema globalmente a la aplicación:

```typescript
export const appConfig: ApplicationConfig = {
  providers: [
    provideBrowserGlobalErrorListeners(),
    provideRouter(routes),
    providePrimeNG({
      theme: {
        preset: Aura
      }
    })
  ]
};
```

4. Abre el archivo `styles.scss` y agrega los siguientes estilos para los selectores `html` y `body`:

```scss
html {
  font-size: 14px;
  font-family: "Inter var", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
  font-feature-settings: "cv02", "cv03", "cv04", "cv11";
  line-height: normal;
}

body {
  background: var(--surface-ground);
  color: var(--text-color);
  padding: 1rem;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  --surface-border:#dfe7ef;
}
```

5. Ejecuta la aplicación usando el comando `ng serve` y navega a `http://localhost:4200`:

> *Figura 9.1 – Encabezado de la aplicación*

La imagen anterior muestra el encabezado de la aplicación, que consta del título del capítulo.

La biblioteca PrimeNG contiene una colección de componentes de alta calidad que podemos usar para la aplicación Flash POS.

El diseño típico de una aplicación POS se divide en tres áreas principales:

- La lista de categorías de artículos
- La lista de artículos para una categoría seleccionada
- El carrito de compras

En la siguiente sección, aprenderemos a utilizar el componente menu bar para mostrar las categorías de artículos.

---

### Sección: Visualización de Categorías

La lista de categorías de artículos disponibles es el punto de partida de la aplicación para los usuarios que desean agregar artículos específicos al carrito de compras. Diseñaremos el diseño para que contenga un botón para cada categoría, mostrando el nombre de la categoría. Los botones deben ser lo suficientemente grandes para que sean fácilmente accesibles mediante una pantalla táctil. Usaremos un componente de barra de menú de PrimeNG para diseñar la lista de categorías:

1. Obtén el archivo `db.json` de la carpeta `ch09` del repositorio de GitHub y cópialo en la carpeta raíz del proyecto actual. El archivo `db.json` representa una base de datos virtual que contiene las categorías junto con sus artículos. Para acceder al enlace del repositorio, sigue los pasos de la sección "Download the example code files" en el Prefacio.
2. Sigue los pasos de la sección *Visualización de la información del edificio* del Capítulo 7, *Expense Builder: Building an SSR Optimized Expense Tracker*, para ejecutar la base de datos utilizando la biblioteca JSON-Server.
3. Ejecuta el siguiente comando para crear un servicio para acceder a la base de datos virtual:

```bash
ng generate service data
```

4. Abre el archivo `data.ts` e importa el método `inject` y la clase `HttpClient`:

```typescript
import { Service, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
```

5. Declara una interfaz para definir la estructura de la categoría de artículos:

```typescript
interface Category {
  id: string;
  name: string;
}
```

6. Inyecta el cliente HTTP en la clase del servicio `Data`:

```typescript
private http = inject(HttpClient);
```

7. Crea el siguiente método para obtener las categorías de la base de datos:

```typescript
getCategories() {
  return this.http.get<Category[]>(
    'http://localhost:3000/categories'
  );
}
```

El método anterior inicia una solicitud GET al punto final `categories` de la API virtual.

Ahora que hemos creado el método para acceder a los datos de las categorías, podemos modificar el componente principal de la aplicación para mostrarlas:

1. Abre el archivo `app.ts` y agrega las siguientes sentencias de importación:

```typescript
import { toSignal } from '@angular/core/rxjs-interop';
import { map } from 'rxjs';
import { Menubar } from 'primeng/menubar';
import { ButtonDirective } from 'primeng/button';
import { Data } from './data';
```

Usaremos las clases `Menubar` y `ButtonDirective` para mostrar categorías como botones dentro de un componente de barra de menú. `toSignal` y `map` nos ayudarán a convertir las categorías en signals con un formato específico para que la barra de menú de PrimeNG pueda mostrarlas correctamente.

2. Agrega las clases de PrimeNG al array `imports` del decorador del componente:

```typescript
imports: [ChapterTitleComponent, Menubar, ButtonDirective]
```

3. Inyecta el servicio `Data` en la clase del componente `App`:

```typescript
private data = inject(Data);
```

4. Declara la variable `categories` de la siguiente manera:

```typescript
readonly categories = toSignal(
  this.data.getCategories().pipe(
    map(categories => categories.map(cat => {
      return { id: cat.id, label: cat.name };
    }))
  )
);
```

En el fragmento anterior, obtenemos las categorías de artículos del servicio de datos y las convertimos en elementos de menú utilizando el operador `map` de RxJS. La etiqueta del elemento de menú muestra el nombre de la categoría.

5. Abre el archivo `app.html` y agrega el siguiente fragmento HTML debajo del componente del título del capítulo:

```html
<p-menubar [model]="categories()">
</p-menubar>
```

El fragmento anterior creará un componente de menú con categorías.

6. Inserta el siguiente fragmento HTML dentro del elemento `<p-menubar>`:

```html
<ng-template #item let-category>
  <button class="btn-category" pButton>
    {{ category.label }}
  </button>
</ng-template>
```

La variable de referencia de plantilla `item` define la plantilla HTML de cada elemento de menú. En nuestro caso, representamos cada uno como un elemento de botón. La expresión `let-category` expone cada objeto de categoría a la variable `category` que podemos usar para mostrar el nombre de la categoría en el texto del botón.

7. Abre el archivo `app.scss` y agrega los siguientes estilos para el elemento de botón:

```scss
.btn-category {
  padding: 1rem;
  width: 8rem;
}
```

Los estilos CSS anteriores harán que los botones de categoría sean más grandes que el tamaño predeterminado. Por lo general, se accede a las aplicaciones POS mediante gestos táctiles, por lo que deben ser más grandes para que se puedan presionar fácilmente.

8. Ejecuta la aplicación y la API virtual, y navega a `http://localhost:4200`:

> *Figura 9.2 – Categorías de artículos*

La imagen anterior muestra las categorías de artículos de la base de datos virtual como botones.

La barra de menú de la biblioteca PrimeNG es un gran componente para visualizar las acciones principales en nuestra aplicación. Los usuarios pueden ver las categorías de artículos en la parte superior de la aplicación y pueden alternar entre ellas cómodamente. En la siguiente sección, aprenderemos a obtener artículos de una categoría seleccionada.

---

### Sección: Visualización de Artículos

La barra de menú de la aplicación contiene la lista de categorías de artículos disponibles. Los usuarios deben poder hacer clic en un botón en particular y ver los artículos que pertenecen a la categoría.

Los elementos de menú de la biblioteca PrimeNG exponen un evento `command` que podemos usar para ejecutar lógica personalizada. Utilizaremos el evento para obtener los artículos de la categoría seleccionada y mostrarlos en la página. Primero, agregaremos un nuevo método en el servicio Data para obtener artículos por categoría:

1. Ejecuta el siguiente comando para crear una interfaz para los objetos de artículos:

```bash
ng generate interface item
```

2. Abre el archivo `item.ts` y agrega las siguientes propiedades:

```typescript
id: string;
name: string;
price: number;
```

Las propiedades anteriores corresponden a las propiedades de un artículo de la base de datos virtual.

3. Abre el archivo `data.ts` e importa la interfaz `Item` y el operador de RxJS `map`:

```typescript
import { Item } from './item';
import { map } from 'rxjs';
```

Cada categoría en la base de datos virtual contiene artículos en la siguiente estructura:

```typescript
{
  id: string;
  data: Item[];
}
```

Necesitamos el operador `map` para extraer la propiedad `data` de la respuesta.

4. Crea el método `getItems` en la clase del servicio `Data`:

```typescript
getItems(categoryId: string) {
  return this.http.get<{ id: string, data: Item[] }>(
    'http://localhost:3000/items/' + categoryId
  ).pipe(
    map(items => items.data)
  );
}
```

El método anterior iniciará una solicitud GET al punto final `items` de la API, pasando el ID de la categoría a la URL de la solicitud.

Usaremos el método `getItems` cuando el usuario haga clic en una categoría específica del componente principal de la aplicación:

1. Importa la interfaz `Item` y el método `signal` en el archivo `app.ts`:

```typescript
import { Component, inject, signal } from '@angular/core';
import { Item } from './item';
```

2. Declara la propiedad `items` en la clase del componente `App` e inicialízala como un array vacío:

```typescript
readonly items = signal<Item[]>([]);
```

3. Crea el siguiente método que llama al método `getItems` del servicio de datos:

```typescript
private getItemsByCategory(category: string) {
  this.data.getItems(category).subscribe(items => {
    this.items.set(items);
  });
}
```

El método anterior establecerá el resultado del método `getItems` en la propiedad signal `items`.

4. Agrega la propiedad `command` en la signal `categories` de la siguiente manera:

```typescript
readonly categories = toSignal(
  this.data.getCategories().pipe(
    map(categories => categories.map(cat => {
      return {
        id: cat.id,
        label: cat.name,
        command: () => this.getItemsByCategory(cat.id)
      };
    }))
  )
);
```

La propiedad `command` define un método de devolución de llamada que se ejecutará cuando los usuarios hagan clic en una categoría. La implementación del método en nuestro caso llamará al método `getItemsByCategory`.

5. Abre el archivo `app.html` e inserta el siguiente código HTML debajo del componente menubar:

```html
<div class="grid">
  @for(item of items(); track $index) {
    <div class="col-2">
      <button pButton class="btn-item" severity="info">
        {{ item.name }}
      </button>
    </div>
  }
</div>
```

En el fragmento anterior, iteramos sobre el array `items` y mostramos un botón para cada artículo. La severidad `info` le da un color distintivo al botón de acuerdo con el tema seleccionado de PrimeNG.

6. Abre el archivo `app.scss` y agrega los siguientes estilos para el fragmento HTML anterior:

```scss
.grid {
  padding-top: 1.5rem;
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
}

.col-2 {
  flex: 0 0 auto;
  padding: .5rem;
  width: 12%;
}

.btn-item {
  padding: 3rem;
  width: 100%;
  max-height: 2rem;
}
```

Los estilos CSS anteriores mostrarán los botones de los artículos en un diseño de cuadrícula.

7. Ejecuta la aplicación y navega a `http://localhost:4200`. Asegúrate de que la API del servidor todavía se esté ejecutando.
8. Selecciona la categoría Fish para ver sus artículos:

> *Figura 9.3 – Artículos de la categoría Fish*

La imagen anterior muestra los artículos de la categoría Fish como botones.

El usuario puede ver los artículos disponibles de una categoría seleccionada en un diseño de cuadrícula, lo cual resulta muy cómodo para las aplicaciones POS. El tamaño de cada botón que representa un artículo es lo suficientemente grande y tiene un espacio adecuado para que el usuario pueda hacer clic fácilmente en él. En la siguiente sección, aprenderemos cómo agregar artículos a un carrito de compras y calcular el costo total.

---

### Sección: Visualización del Carrito de Compras

Los usuarios deben poder seleccionar un artículo de una categoría particular y agregarlo al carrito de compras. Podrán agregar el mismo artículo varias veces haciendo clic en el botón correspondiente. Una indicación en el encabezado de la aplicación mostrará el número total de artículos en el carrito de compras. Los usuarios también podrán ver el contenido del carrito de compras haciendo clic en la indicación.

Agregaremos un botón en el encabezado de la aplicación para mostrar el número total de artículos en el carrito de compras:

1. Abre el archivo `app.ts` y agrega la siguiente sentencia de importación:

```typescript
import { OverlayBadge } from 'primeng/overlaybadge';
```

La clase `OverlayBadge` contiene un componente de insignia que usaremos para mostrar el número total de artículos.

2. Agrega la clase anterior al array `imports` del decorador del componente:

```typescript
imports: [
  ChapterTitleComponent,
  Menubar,
  ButtonDirective,
  OverlayBadge
]
```

3. Crea la siguiente propiedad en la clase del componente `App`:

```typescript
readonly selectedItems = signal<Item[]>([]);
```

Utilizaremos la propiedad `selectedItems` para almacenar los artículos del carrito de compras.

4. Crea un método para agregar artículos al carrito:

```typescript
addToCart(item: Item) {
  this.selectedItems.update(items => [...items, item]);
}
```

El método `addToCart` utiliza el método `update` para agregar un nuevo artículo al array `selectedItems`.

5. Abre el archivo `app.html` y agrega el siguiente fragmento HTML debajo del elemento `ng-template`:

```html
<button pButton severity="secondary" (click)="isCartVisible.set(true)">
  <p-overlaybadge [value]="selectedItems().length" badgeSize="large">
    <i class="pi pi-shopping-cart"></i>
  </p-overlaybadge>
</button>
```

El código anterior agregará un elemento de botón con una indicación superpuesta grande y un icono de carrito de compras. El valor de la indicación mostrará el número total de artículos en el carrito.

6. Agrega una vinculación de evento de clic al botón de cada artículo de categoría:

```html
<button pButton class="btn-item" severity="info" (click)="addToCart(item)">
  {{ item.name }}
</button>
```

El código anterior agregará el artículo al carrito de compras al hacer clic en el botón correspondiente.

7. Abre el archivo `app.scss` y agrega un estilo para el icono del carrito de compras:

```scss
i {
  font-size: 2rem;
}
```

El estilo CSS anterior hará que el icono del carrito de compras sea más grande para que encaje bien con el resto de la interfaz de usuario.

8. Asegúrate de que la aplicación se esté ejecutando, selecciona algunos artículos y verifica que el icono de indicación refleje su número total:

> *Figura 9.4 – Encabezado de la aplicación con artículos del carrito de compras*

La imagen anterior muestra el encabezado de la aplicación y el botón del carrito de compras con cuatro artículos en total.

El botón del carrito de compras actualmente muestra el número total de artículos. Crearemos un nuevo componente para mostrar los detalles de cada artículo cuando el usuario haga clic en el botón:

1. Crea el componente del carrito usando el siguiente comando:

```bash
ng generate component cart
```

2. Abre el archivo `app.ts` e importa la clase `Cart`:

```typescript
import { Cart } from './cart/cart';
```

3. Agrega la clase al array `imports` del decorador del componente:

```typescript
imports: [
  ChapterTitleComponent,
  Menubar,
  ButtonDirective,
  OverlayBadge,
  Cart
]
```

4. Agrega la siguiente propiedad a la clase del componente `App`:

```typescript
readonly isCartVisible = signal(false);
```

La propiedad `isCartVisible` alternará la visibilidad del componente del carrito.

5. Abre el archivo `app.html` y agrega el componente del carrito debajo del elemento `div` que muestra los botones de categoría:

```html
<app-cart [items]="selectedItems()" [(visible)]="isCartVisible" />
```

Pasaremos los artículos que hemos agregado al carrito de compras en el componente del carrito utilizando el enlace de propiedad `items`.

6. Abre el archivo `cart.ts` y modifica las sentencias de importación de la siguiente manera:

```typescript
import { Component, input, model } from '@angular/core';
import { Item } from '../item';
```

7. Declara las siguientes propiedades de entrada en la clase del componente `Cart`:

```typescript
readonly visible = model(false);
readonly items = input<Item[]>([]);
```

El componente del carrito mostrará los artículos seleccionados junto con sus cantidades y el costo total. Diseñaremos la interfaz de usuario del componente del carrito como una barra lateral en el lado derecho de la pantalla:

1. Agrega las siguientes sentencias de importación en el archivo `cart.ts`:

```typescript
import { Drawer } from 'primeng/drawer';
import { TableModule } from 'primeng/table';
```

La clase `Drawer` contiene un componente de barra lateral que se abre y se cierra como un cajón. `TableModule` nos permitirá usar componentes para mostrar los artículos del carrito y su costo total en formato tabular.

2. Crea una interfaz para definir los artículos del carrito de compras:

```typescript
interface CartItem {
  name: string;
  qty: number;
}
```

Usaremos la interfaz `CartItem` para mostrar los artículos en el carrito de compras.

3. Agrega las clases `Drawer` y `TableModule` al array `imports` del decorador del componente:

```typescript
imports: [Drawer, TableModule]
```

4. Usa el método `computed` del paquete `@angular/core` para calcular la cantidad total por artículo en la clase del componente `Cart`:

```typescript
readonly cart = computed(() => {
  const cart: CartItem[] = [];
  this.items().forEach(item => {
    const existed = cart.find(i => i.name === item.name);
    if (!existed) {
      cart.push({ name: item.name, qty: 1 });
    } else {
      existed.qty += 1;
    }
  });
  return cart;
});
```

En el código anterior, iteramos sobre los artículos seleccionados del enlace de entrada y creamos un objeto de artículo del carrito. La cantidad del artículo del carrito se deriva de las apariciones de cada artículo en los datos de entrada.

5. Abre el archivo `cart.html` y reemplaza su contenido con el siguiente fragmento HTML:

```html
<p-drawer [(visible)]="visible" header="Cart" position="right">
  <p-table [value]="cart()">
    <ng-template #body let-item>
      <tr>
        <td>{{ item.name }}</td>
        <td>{{ item.qty }}</td>
      </tr>
    </ng-template>
  </p-table>
</p-drawer>
```

En el código anterior, agregamos un elemento `<p-drawer>` que representa el componente de barra lateral y lo configuramos para que aparezca en el lado derecho. Dentro de la barra lateral, agregamos un elemento `<p-table>` para mostrar los datos en formato tabular. La variable de referencia de plantilla `body` representa el cuerpo de la tabla que muestra el nombre y la cantidad del artículo del carrito.

6. Con la aplicación en ejecución, selecciona algunos artículos de las categorías y haz clic en el botón del carrito de compras para ver sus detalles:

> *Figura 9.5 – Carrito de compras*

La imagen anterior muestra los detalles del carrito de compras en la barra lateral derecha de la aplicación.

Calcularemos el costo total del carrito y lo mostraremos en el pie de página de la tabla del componente del carrito:

1. Abre el archivo `cart.ts` y declara la siguiente propiedad en la clase del componente `Cart`:

```typescript
readonly cost = computed(() => {
  return this.items()
    .map(item => item.price)
    .reduce((x: number, y: number) => x + y, 0);
});
```

En el fragmento anterior, iteramos sobre los artículos seleccionados y calculamos el costo total en función de su precio.

2. Abre el archivo `cart.html` y agrega el siguiente fragmento HTML dentro del cuerpo de la tabla como último elemento:

```html
<ng-template #footer>
  <tr class="font-bold">
    <td>Total</td>
    <td>{{ cost() }}</td>
  </tr>
</ng-template>
```

La variable de referencia de plantilla `footer` indica el contenido del pie de página de la tabla, que mostrará el costo total del carrito de compras.

3. Abre el archivo `cart.scss` y agrega un estilo para el pie de página de la tabla:

```scss
.font-bold {
  font-weight: 700;
}
```

El estilo CSS anterior hará que el contenido del pie de página de la tabla sea más grueso que el resto del contenido.

4. Actualiza el navegador, selecciona los mismos artículos que antes y haz clic en el botón del carrito:

> *Figura 9.6 – Carrito de compras con costo total*

La imagen anterior muestra el costo total de los artículos del carrito de compras.

El carrito de compras muestra los detalles del pedido y el costo total al instante. En el futuro, los usuarios pueden enviar los detalles del carrito de compras a una impresora térmica para imprimir un recibo del pedido.

---

### Sección: Resumen

En este capítulo, creamos una aplicación POS para ayudar al personal del restaurante a realizar pedidos para llevar. Usamos la biblioteca PrimeNG para diseñar una interfaz adecuada para pantallas táctiles con botones grandes. Aprendimos a usar una base de datos existente y mostrar artículos por categoría. También exploramos cómo usar las signals de Angular para calcular la cantidad y el costo total de cada artículo y mostrarlos en un diseño de carrito de compras. En el próximo capítulo, construiremos una aplicación sencilla para tomar notas con capacidades de IA.

---

### Sección: Ejercicios

Usa el campo `discount` del archivo `db.json` y calcula el descuento total del carrito de compras. Agrega un botón en el pie de página del carrito de compras que limpie el carrito de compras y cierre la barra lateral.

---

### Sección: Lecturas Complementarias

- **PrimeNG:** [https://primeng.org](https://primeng.org/)
- **JSON-Server:** [https://www.npmjs.com/package/json-server](https://www.npmjs.com/package/json-server)
