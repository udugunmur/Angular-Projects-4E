# Parte 1: Primeros Pasos con Angular e Inteligencia Artificial

## Capítulo 4: SmartFactory Picker: Construcción de una Aplicación de Almacén Basada en Códigos QR

### Sección: Introducción

La digitalización se ha expandido a todos los aspectos de nuestra vida cotidiana, incluido el mundo industrial. Las fábricas están automatizando varios pasos de sus líneas de producción para acelerar los procesos y mejorar el control. La preparación de pedidos (*picking*) es un paso esencial en el proceso para garantizar que los clientes reciban los productos correctos.

En este capítulo, construiremos una utilidad para un sistema de gestión de almacenes (WMS - *Warehouse Management System*) que ayude a los usuarios a recolectar los artículos de un pedido. Utilizaremos códigos QR para identificar artículos individuales en el almacén y agregarlos a una lista de preparación de pedidos. Los códigos QR se utilizan ampliamente en la industria debido a su resistencia y compatibilidad con dispositivos móviles.

Vamos a cubrir los siguientes temas:

- Configuración de la página de artículos
- Generación de códigos QR
- Escaneo de artículos
- Creación de la lista de preparación de pedidos

---

### Sección: Requisitos Técnicos

Puedes descargar el proyecto de ejemplo y el código de este libro siguiendo las instrucciones de la sección *Download the example code files* en el Prefacio de este libro.

Los archivos de código de este capítulo están incluidos en el paquete de código descargable.

Utilizaremos el código de la carpeta `ch01` como punto de partida para construir la aplicación de este capítulo.

---

### Sección: Configuración de la Página de Artículos

La aplicación mostrará una lista de artículos provenientes de una API de backend. Cada artículo tendrá una descripción, una categoría y un precio. Configuraremos el router de Angular para navegar a la lista de artículos cuando se inicie la aplicación.

Utilizaremos la aplicación del Capítulo 1, *Angular AI Kick-Starter: Scaffolding Smart Apps with Copilot*, como punto de partida:

1. Abre el archivo `app.ts` y crea el siguiente constructor para establecer el título del capítulo:

```typescript
constructor() {
  this.chapterTitleService.setTitle(
    '[Chapter 4](https://subscription.packtpub.com/book/programming/9781806668472/4): SmartFactory Picker'
  );
}
```

2. Abre el archivo `app.html` y mueve el contenido de la etiqueta `style` dentro del archivo `app.scss`.
3. Reemplaza el contenido del elemento de párrafo con el siguiente texto: `Building a QR-Driven Warehouse App.`
4. Agrega un nuevo elemento de ancla en los enlaces sociales que contenga una imagen para un código QR, como la siguiente:

> *Figura 4.1 – Icono de código QR*

La imagen anterior muestra el icono SVG de un código QR. Puedes obtenerlo del repositorio de GitHub. Para acceder al enlace del repositorio, sigue los pasos de la sección "Download the example code files" en el Prefacio.

5. Ejecuta la aplicación usando el comando `ng serve` y navega a `localhost:4200`:

> *Figura 4.2 – Salida de la aplicación*

La lista de botones en el lado derecho de la imagen muestra enlaces útiles para el framework Angular. La utilizaremos como base para mostrar una lista de productos para nuestro caso de uso:

1. Ejecuta el siguiente comando para crear un nuevo componente para la lista de productos:

```bash
ng generate component product-list
```

2. Mueve el elemento `div` con la clase `pill-group` del archivo `app.html` al archivo `product-list.html`.
3. Abre el archivo `app.routes.ts` y agrega una nueva ruta:

```typescript
import { Routes } from '@angular/router';
import { ProductList } from './product-list/product-list';

export const routes: Routes = [
  {
    path: '',
    component: ProductList
  }
];
```

El objeto de configuración de ruta anterior activa el componente de lista de productos al iniciar la aplicación.

4. Mueve los selectores CSS `.pill` y `.pill-group` del archivo `app.scss` al archivo `product-list.scss`.
5. Abre el archivo `app.html` y mueve el selector `router-outlet` encima de los enlaces sociales.
6. Ejecuta la aplicación y verifica que la salida sea la misma que en la Figura 4.2.

La aplicación mostrará productos de la Fake Store API, una API REST gratuita ideal para la creación de prototipos y la enseñanza:

1. Ejecuta el siguiente comando para crear una interfaz para los objetos de producto:

```bash
ng generate interface product
```

2. Abre el archivo `product.ts` y agrega las siguientes propiedades a la interfaz `Product`:

```typescript
id: number;
title: string;
description: string;
price: number;
category: string;
```

3. Ejecuta el siguiente comando para crear un servicio que utilizaremos para obtener productos de la Fake Store API:

```bash
ng generate service products
```

4. Abre el archivo `products.ts` y agrega las siguientes sentencias de importación:

```typescript
import { HttpClient } from '@angular/common/http';
import { Product } from './product';
```

La clase `HttpClient` proporciona métodos para intercambiar datos con una API de backend.

5. Usa el método `inject` del paquete `@angular/core` para inyectar el cliente HTTP en el servicio:

```typescript
private http = inject(HttpClient);
```

6. Crea el siguiente método que inicia una petición GET a la Fake Store API y devuelve todos los productos:

```typescript
getAll() {
  return this.http.get<Product[]>(
    'https://fakestoreapi.com/products'
  );
}
```

El componente de lista de productos utilizará el servicio de Angular anterior para obtener y mostrar los productos disponibles:

1. Abre el archivo `product-list.ts` y modifica las sentencias de importación de la siguiente manera:

```typescript
import { Component, inject } from '@angular/core';
import { Products } from '../products';
import { toSignal } from '@angular/core/rxjs-interop';
import { SlicePipe } from '@angular/common';
```

Utilizaremos el método `toSignal` para convertir las llamadas HTTP de observables a signals. La clase `SlicePipe` nos permitirá mostrar únicamente un subconjunto de los productos de la API de backend.

Por simplicidad, trabajaremos con un subconjunto de datos de productos.

2. Agrega la clase `SlicePipe` al array `imports` del decorador del componente:

```typescript
imports: [SlicePipe]
```

3. Declara la siguiente propiedad en la clase del componente `ProductList`:

```typescript
products = toSignal(inject(Products).getAll());
```

4. Abre el archivo `product-list.html` y reemplaza su contenido con el siguiente código HTML:

```html
<div class="pill-group">
  @for (item of products() | slice:0:6; track item.title) {
    <a class="pill">
      <span>{{ item.title }}</span>
    </a>
  }
</div>
```

El código itera sobre la propiedad `products` y muestra los primeros seis artículos.

5. Abre el archivo `product-list.scss` y establece el ancho del selector `.pill-group` en `300px`:

```scss
.pill-group {
  display: flex;
  flex-direction: column;
  align-items: start;
  flex-wrap: wrap;
  gap: 1.25rem;
  width: 300px;
}
```

6. Ejecuta la aplicación y verifica que la salida de la aplicación sea la siguiente:

> *Figura 4.3 – Lista de productos*

Esta figura muestra los primeros seis productos de la Fake Store API.

Nuestra aplicación de fábrica inteligente muestra una lista de artículos del almacén. Cada artículo tiene características específicas, como la categoría y el precio. En la siguiente sección, construiremos una página dedicada para cada artículo que muestre estos detalles.

---

### Sección: Generación de Códigos QR

La aplicación mostrará los detalles del producto en una página independiente que se abrirá cuando el usuario seleccione un producto de la lista. Utilizaremos el enrutamiento para consultar la Fake Store API por ID de producto. La página mostrará el nombre, la categoría y el precio del producto. Los usuarios podrán generar un código QR en la página para su uso posterior.

Comenzaremos agregando un método apropiado en el servicio de productos y luego crearemos la página de detalles del producto:

1. Abre el archivo `products.ts` y agrega el siguiente método:

```typescript
getSingle(id: number) {
  return this.http.get<Product>(
    'https://fakestoreapi.com/products/' + id
  );
}
```

El método anterior consultará la Fake Store API para obtener un producto con un ID específico y devolverá los detalles completos del producto actual.

2. Ejecuta el siguiente comando para crear un resolver de Angular:

```bash
ng generate resolver product
```

Utilizaremos un resolver para obtener los detalles del producto antes de mostrar el componente respectivo.

Los resolvers proporcionan una buena experiencia de usuario cuando no se desea cargar el componente antes de que lleguen los datos. Por lo general, se utilizan con indicadores de carga (*spinners*) para señalar el progreso.

3. Abre el archivo `product-resolver.ts` y agrega las siguientes sentencias de importación:

```typescript
import { inject } from '@angular/core';
import { Products } from './products';
import { Product } from './product';
```

4. Modifica la función `productResolver` de la siguiente manera:

```typescript
export const productResolver: ResolveFn<Product> = route => {
  const id = Number(route.paramMap.get('id'));
  return inject(Products).getSingle(id);
};
```

El resolver lee el parámetro ID de la ruta activada, lo convierte en un número y lo pasa al método `getSingle`.

5. Ejecuta el siguiente comando para crear el componente para los detalles del producto:

```bash
ng generate component product-detail
```

6. Abre el archivo `app.routes.ts` y agrega las siguientes sentencias de importación:

```typescript
import { ProductDetail } from './product-detail/product-detail';
import { productResolver } from './product-resolver';
```

7. Agrega el siguiente objeto de configuración de ruta en la propiedad `routes` para activar el componente de detalles del producto:

```typescript
{
  path: ':id',
  component: ProductDetail,
  resolve: {
    product: productResolver
  }
}
```

Cuando el usuario navegue a un ID de producto específico, el resolver llamará a la Fake Store API usando ese ID y devolverá los detalles del producto al componente. Modificaremos el componente de lista de productos para que el usuario pueda navegar a un producto específico seleccionándolo:

1. Abre el archivo `product-list.ts` y agrega la siguiente sentencia de importación:

```typescript
import { RouterLink } from '@angular/router';
```

La clase `RouterLink` es una directiva de Angular que agrega capacidades de navegación a los elementos a los que se adjunta, y se utiliza frecuentemente con elementos de ancla.

2. Agrega la clase `RouterLink` al array `imports` del decorador del componente:

```typescript
imports: [SlicePipe, RouterLink]
```

3. Abre el archivo `product-list.html` y agrega el atributo `routerLink` al elemento de ancla:

```html
<div class="pill-group">
  @for (item of products() | slice:0:6; track item.title) {
    <a class="pill" [routerLink]="item.id.toString()">
      <span>{{ item.title }}</span>
    </a>
  }
</div>
```

Pasamos la propiedad `id` del artículo a la directiva `routerLink` y la convertimos en una cadena de texto porque los parámetros de ruta son siempre cadenas.

4. Abre el archivo `product-detail.ts` y agrega las siguientes sentencias de importación:

```typescript
import { ActivatedRoute } from '@angular/router';
import { Product } from '../product';
import { CurrencyPipe, UpperCasePipe } from '@angular/common';
```

`ActivatedRoute` es un servicio de Angular que proporciona acceso a la ruta activada actual. La clase `UpperCasePipe` convierte cualquier texto a mayúsculas.

5. Agrega las clases de la última sentencia de importación al array `imports` del decorador del componente:

```typescript
imports: [CurrencyPipe, UpperCasePipe]
```

6. Declara la siguiente propiedad de tipo signal en la clase del componente `ProductDetail`:

```typescript
product = signal<Product>(
  inject(ActivatedRoute).snapshot.data['product']
);
```

El componente inyecta el servicio `ActivatedRoute` y lee el valor del producto del resolver a través de la propiedad `snapshot.data`.

Los métodos `inject` y `signal` se pueden importar del paquete `@angular/core`.

7. Abre el archivo `product-detail.html` y reemplaza su contenido con el siguiente código HTML:

```html
<h2>{{product().title}}</h2>
<p>{{product().description}}</p>
<h4>{{product().price | currency}}</h4>
<i>{{product().category | uppercase}}</i>
```

Utilizamos varios elementos HTML para mostrar los detalles del producto. Mostramos el precio en formato de moneda y la categoría en mayúsculas.

8. Abre el archivo `product-detail.scss` y agrega los siguientes estilos CSS para definir el ancho de los elementos `p` y `h2`:

```scss
p, h2 {
  width: 400px;
}
```

9. Ejecuta la aplicación, selecciona un producto y verifica que la salida sea similar a la siguiente:

> *Figura 4.4 – Detalles del producto*

La figura anterior muestra los detalles del producto Solid Gold Petite Micropave.

Cada producto de la aplicación tiene una página dedicada con toda la información necesaria. Utilizaremos Google Chrome para generar un código QR para el producto seleccionado:

1. Abre Google Chrome y haz clic en el menú de tres puntos.
2. Haz clic en la opción de menú **Cast, save, and share**.
3. Selecciona la opción **Create QR Code** del grupo Share.

El navegador abre el panel Scan QR Code:

> *Figura 4.5 – Código QR*

Google Chrome genera un código QR único para cada página de detalles del producto. Podemos copiarlo o descargarlo localmente para su procesamiento posterior, como imprimirlo.

En un escenario realista, imprimiremos códigos QR para todos los productos y los colocaremos en el estante físico correspondiente a cada uno. El usuario podrá escanear el código QR cuando tome el producto del estante. En la siguiente sección, aprenderemos a escanear un código QR y leer su contenido.

---

### Sección: Escaneo de Artículos

Necesitamos una cámara para escanear un código QR y leer su contenido. Los navegadores web pueden interactuar con un dispositivo de cámara mediante APIs nativas de JavaScript. La Media Streams API proporciona métodos adecuados para trabajar con datos de video transmitidos, como una señal de cámara en vivo. Sin embargo, no admite códigos QR de forma predeterminada.

En este capítulo, utilizaremos la biblioteca de Angular `ngx-scanner-qrcode`, que proporciona una forma conveniente de abrir la cámara y leer el valor de un código QR. La biblioteca es un wrapper de Angular para APIs nativas que elimina gran parte del código repetitivo necesario para interactuar con la cámara.

Crearemos un componente que se encargará de escanear artículos y agregarlos a una lista de preparación de pedidos:

1. Ejecuta el siguiente comando para generar el componente:

```bash
ng generate component picking
```

2. Abre el archivo `app.routes.ts` y agrega la siguiente sentencia de importación:

```typescript
import { Picking } from './picking/picking';
```

3. Agrega el siguiente objeto de configuración de ruta antes de la ruta de detalles del producto para activar el nuevo componente:

```typescript
{
  path: 'picking',
  component: Picking
}
```

El router de Angular analiza la configuración de enrutamiento en orden. Coloca las rutas más específicas antes de las menos específicas.

4. Abre el archivo `app.ts` y agrega la siguiente sentencia de importación:

```typescript
import { RouterLink } from '@angular/router';
```

5. Agrega la clase anterior al array `imports` del decorador del componente:

```typescript
imports: [RouterOutlet, ChapterTitleComponent, RouterLink]
```

6. Abre el archivo `app.html` y agrega la directiva `routerLink="picking"` al enlace del código QR.

La aplicación cargará el componente de picking cuando los usuarios hagan clic en el enlace del código QR en la página principal. Utilizaremos la biblioteca de escáner QR en el componente de picking:

1. Ejecuta el siguiente comando para instalar la biblioteca del escáner QR:

```bash
npm install ngx-scanner-qrcode
```

2. Abre el archivo `picking.ts` y agrega la siguiente sentencia de importación:

```typescript
import { NgxScannerQrcodeComponent } from 'ngx-scanner-qrcode';
```

3. Agrega la clase `NgxScannerQrcodeComponent` al array `imports` del decorador del componente:

```typescript
imports: [NgxScannerQrcodeComponent]
```

4. Declara las siguientes propiedades en la clase del componente `Picking`:

```typescript
readonly scanner = viewChild.required(
  NgxScannerQrcodeComponent
);
items = signal<string[]>([]);
```

La propiedad `scanner` contiene una instancia del componente del escáner QR. La aplicación leerá los valores del escáner y llenará la propiedad `items`. Usamos los métodos `viewChild` y `signal` del paquete `@angular/core`.

5. Agrega el siguiente constructor para interactuar con el escáner QR:

```typescript
constructor() {
  afterNextRender(() => {
  });
}
```

El método `afterNextRender` del paquete `@angular/core` nos permite ejecutar código personalizado después de que Angular inicializa el componente de picking.

6. Agrega el siguiente fragmento dentro del método `afterNextRender`:

```typescript
this.scanner().start();
this.scanner().data.subscribe(data => {
  if (data.length) {
    this.items.update(i => [...i, data[0].value]);
    this.scanner().data.next([]);
  }
});
```

En el fragmento anterior, abrimos el escáner de códigos QR y comenzamos a escuchar valores utilizando el observable `data`. Cuando hay datos disponibles, los agregamos a la propiedad `items` y limpiamos los datos anteriores del escáner.

7. Abre el archivo `picking.html` y reemplaza su contenido con el siguiente código HTML:

```html
<ngx-scanner-qrcode />
@for (item of items(); track $index) {
  <p>{{item}}</p>
}
```

El componente `ngx-scanner-qrcode` es un elemento de marcador de posición para la señal de la cámara. La aplicación mostrará los valores del código QR en una lista de elementos de párrafo.

Veamos cómo funciona el flujo de trabajo de escaneo en la práctica ejecutando la aplicación:

1. Haz clic en el enlace del código QR en la página principal.
2. El navegador solicitará tu permiso para usar la cámara.
3. La aplicación abrirá la cámara encima de los enlaces sociales al otorgar el permiso:

> *Figura 4.6 – Página de preparación de pedidos (picking)*

El fondo negro en la imagen anterior es un marcador de posición para la cámara que mostrará la señal en vivo durante el tiempo de ejecución.

4. Coloca el código QR que creamos en la sección anterior dentro del panel de la cámara. Espera hasta que la aplicación reconozca el código y deberías ver la siguiente salida debajo del panel de la cámara: `http://localhost:4200/6`

También escucharás un sonido y el código QR se convertirá en un cuadro verde con una etiqueta roja que muestra el código real.

La aplicación puede escanear códigos de barras QR y mostrar sus valores en una lista dentro del componente de picking. Si experimentas con el escaneo, notarás que la lista de elementos crece a medida que el escáner lee el mismo código QR. En la siguiente sección, aprenderemos cómo excluir códigos QR duplicados y cómo convertirlos en productos reales.

---

### Sección: Creación de la Lista de Preparación de Pedidos

Una lista de preparación de pedidos (*picking list*) en un WMS es una lista digital o escrita de productos de un pedido específico de un cliente. El personal del almacén recorre las estanterías según el pedido del cliente, busca los artículos requeridos, escanea sus códigos de barras y los coloca en un contenedor digital. En esta sección, modificaremos el componente de picking para que se comporte como una lista de preparación de pedidos.

Primero, debemos configurarlo para que reconozca códigos QR únicos, ya que la cámara es una señal en vivo y puede detectar el mismo código QR repetidamente:

1. Abre el archivo `picking.ts` y crea el siguiente método en la clase del componente `Picking`:

```typescript
private getProduct(code: string) {
}
```

El parámetro `code` del método será el código QR leído por el escáner.

2. Mueve el fragmento de TypeScript rodeado por la sentencia `if` del callback `data` del escáner dentro del método:

```typescript
private getProduct(code: string) {
  this.items.update(i => [...i, code]);
  this.scanner().data.next([]);
}
```

En el método anterior, reemplazamos `data[0].value` del método `update` con el parámetro `code`.

3. Modifica el método del callback `data` del escáner de la siguiente manera:

```typescript
this.scanner().data.subscribe(data => {
  if (data.length) {
    this.getProduct(data[0].value);
  }
});
```

4. Ejecuta la aplicación y escanea el código QR varias veces. La salida debería mostrar el siguiente texto varias veces: `http://localhost:4200/6`

5. Modifica el método `getProduct` agregando la siguiente sentencia `if`:

```typescript
private getProduct(code: string) {
  if (!this.items().includes(code)) {
    this.items.update(i => [...i, code]);
    this.scanner().data.next([]);
  }
}
```

El fragmento anterior garantiza que la aplicación agregue el código a la lista de elementos solo si aún no existe.

6. Escanea el código QR nuevamente y verifica que la lista muestre su valor solo una vez. La salida del escáner es una transmisión en tiempo real. La aplicación identifica el código QR varias veces, como lo indica el sonido, pero lo agrega a la lista solo una vez.
7. Intenta escanear un código QR diferente y verifica que se agregue a la lista.

La aplicación puede diferenciar entre el mismo código QR, de modo que no agregamos el mismo artículo a la lista varias veces.

El siguiente paso es hacer que el diseño de picking sea consistente con el resto de la aplicación:

1. Abre el archivo `picking.html` y agrega el siguiente elemento `h2` después del selector `ngx-scanner-qrcode`:

```html
<h2>Picking List</h2>
```

2. Agrega un elemento `div` que rodee la lista de elementos:

```html
<div class="picking-list">
  @for (item of items(); track $index) {
    <p>{{item}}</p>
  }
</div>
```

3. Agrega la clase `picking-item` en el elemento de párrafo:

```html
<p class="picking-item">
  {{item}}
</p>
```

4. Abre el archivo `picking.scss` y agrega el siguiente estilo CSS para el selector `.picking-list`:

```scss
.picking-list {
  display: flex;
  flex-direction: column;
  align-items: start;
  flex-wrap: wrap;
  width: 300px;
}
```

5. Agrega también un estilo CSS para el selector `.picking-item`:

```scss
.picking-item {
  display: flex;
  align-items: center;
  background: color-mix(
    in srgb,
    var(--pill-accent) 5%,
    transparent
  );
  color: var(--pill-accent);
  padding-inline: 0.75rem;
  padding-block: 0.375rem;
  border-radius: 2.75rem;
  font-family: var(--inter-font);
  font-size: 0.875rem;
  font-weight: 500;
}
```

La salida de la lista de picking debería verse como la siguiente:

> *Figura 4.7 – Lista de preparación de pedidos (picking)*

El componente muestra los elementos de la lista de picking en azul con un fondo transparente, siguiendo el diseño de la aplicación.

Ahora podemos integrarnos con la Fake Store API y obtener los detalles del producto de acuerdo con el valor del código QR:

1. Abre el archivo `picking.ts` y agrega la siguiente sentencia de importación:

```typescript
import { Products } from '../products';
```

2. Usa el método `inject` del paquete `@angular/core` para inyectar la clase del servicio `Products`:

```typescript
private productsService = inject(Products);
```

3. Agrega el siguiente fragmento al comienzo del método `getProduct`:

```typescript
const id = code.substring(code.lastIndexOf('/') + 1);
```

La variable `id` extrae el texto después del último carácter `/` del código QR, que ya sabemos que representa un ID de producto.

4. Utiliza la variable `id` para llamar al método `getSingle` del servicio de productos:

```typescript
this.productsService.getSingle(Number(id)).subscribe(p => {
});
```

5. Mueve el resto del método `getProduct` dentro del callback `subscribe` del método `getSingle` y modifícalo para agregar el título del producto dentro de la lista de elementos:

```typescript
this.productsService.getSingle(Number(id)).subscribe(p => {
  if (!this.items().includes(p.title)) {
    this.items.update(i => [...i, p.title]);
    this.scanner().data.next([]);
  }
});
```

6. Ejecuta la aplicación, escanea el mismo código QR y verifica que la lista de picking muestre el título del producto similar a lo siguiente:

> *Figura 4.8 – Lista de preparación de pedidos (picking)*

La lista de picking muestra el título de cada producto correspondiente a su código QR. Los usuarios pueden continuar escaneando códigos QR hasta que creen la lista de productos requerida.

---

### Sección: Resumen

En este capítulo, aprendimos cómo crear una utilidad de escaneo de códigos QR que ayuda a un almacén a recolectar productos para su preparación de manera eficiente y sin errores. Integramos la aplicación con la Fake Store API para obtener datos simulados para los productos de nuestro almacén. Exploramos cómo interactuar con el dispositivo de cámara utilizando una biblioteca de terceros para leer y decodificar códigos QR. En el siguiente capítulo, construiremos una aplicación que controla plazas de estacionamiento.

---

### Sección: Ejercicios

Modifica el componente de la lista de picking para mostrar el número total de artículos y el coste total.

---

### Sección: Lecturas Complementarias

- **Fake Store API:** [https://fakestoreapi.com](https://fakestoreapi.com/)
- **Media Streams API:** [https://developer.mozilla.org/docs/Web/API/Media_Capture_and_Streams_API](https://developer.mozilla.org/docs/Web/API/Media_Capture_and_Streams_API)
- **Ngx-scanner-qrcode:** [https://github.com/id1945/ngx-scanner-qrcode](https://github.com/id1945/ngx-scanner-qrcode)
