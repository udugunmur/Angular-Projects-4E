# Parte 1: Primeros Pasos con Angular e Inteligencia Artificial

## Capítulo 6: Studio BookMaster: Diseño de una Aplicación de Reserva de Salas Mejorada con IA

### Sección: Introducción

Vivimos en una era donde la IA afecta muchos aspectos de nuestra vida cotidiana. El enfoque principal de la IA se centra actualmente en guiar a los usuarios, pero pronto cambiará hacia la automatización de tareas más complejas. Los usuarios buscan ayuda de asistentes de IA al comprar en línea, buscar una tienda local o verificar un vuelo.

Ya hemos visto cómo integrar la IA en el lado del cliente de una aplicación en el Capítulo 5, *CityPass Parking: Developing an AI-Enabled Parking Validator*. En este capítulo, aprenderemos a utilizar la IA en el backend de una aplicación full-stack mejorando un sistema de reservas en línea para estudios de música. Construiremos una aplicación de reserva completa que permite a los usuarios realizar nuevas reservas utilizando un calendario y recibir un correo electrónico de confirmación. También aprenderemos a mejorar el proceso de reserva con asistencia de IA.

Vamos a cubrir los siguientes temas:

- Creación del backend
- Integración del acceso a la base de datos
- Instalación de PrimeNG
- Creación de reservas
- Visualización de salas disponibles
- Envío de mensajes de confirmación
- Reservas mediante IA

---

### Sección: Requisitos Técnicos

Todos los ejemplos de código para este capítulo se pueden encontrar en las carpetas `ch01` y `ch06` en GitHub. Puedes descargar el proyecto de ejemplo y el código de este libro siguiendo las instrucciones de la sección *Download the example code files* en el Prefacio de este libro.

Los archivos de código de este capítulo están incluidos en el paquete de código descargable.

---

### Sección: Creación del Backend

Construiremos un sistema de reservas completo con NestJS como solución de backend y Angular como frontend:

- **Frontend:** Usaremos Angular con la biblioteca PrimeNG para diseñar la interfaz de usuario de la aplicación y la página de reservas. El frontend interactuará con el backend para crear nuevas reservas.
- **Backend:** Usaremos NestJS para diseñar la API de la aplicación que será responsable de obtener datos del frontend y guardar nuevas reservas en una base de datos MongoDB.

NestJS es un framework de Node.js para crear aplicaciones y APIs del lado del servidor escalables y eficientes. Es popular entre los desarrolladores de Angular porque está escrito en TypeScript y su sintaxis y patrones arquitectónicos se asemejan a los de Angular.

Construiremos el sistema backend en una carpeta separada del frontend:

1. Crea una carpeta llamada `bookmaster` en tu sistema local.
2. Abre una ventana de terminal e instala NestJS con el siguiente comando:

```bash
npm install -g @nestjs/cli
```

El comando anterior instala la CLI de NestJS, una interfaz de línea de comandos para estructurar (*scaffolding*), compilar y probar aplicaciones NestJS.

3. Usa la CLI de NestJS para crear una nueva aplicación usando el siguiente comando dentro de la carpeta `bookmaster`:

```bash
nest new server
```

La CLI te preguntará qué gestor de paquetes deseas utilizar. Selecciona tu preferencia y presiona Enter. La CLI de NestJS descarga e instala todos los paquetes necesarios y crea archivos predeterminados para tu aplicación de servidor. Después de terminar, la carpeta `server` contendrá una aplicación básica de NestJS.

4. Ejecuta el siguiente comando desde una ventana de terminal dentro de la carpeta `server` para ejecutar tu aplicación:

```bash
nest start
```

La CLI de NestJS compila la aplicación e inicia un servidor web para alojarla. Alternativamente, puedes ejecutar `nest start --watch` para mantener la aplicación en ejecución y recargarla mientras realizas cambios en el código fuente.

Todos los comandos de terminal para la aplicación backend deben ejecutarse dentro de la carpeta `server` del proyecto actual.

5. Después de una compilación exitosa, puedes obtener una vista previa de la aplicación abriendo tu navegador y navegando a `http://localhost:3000`. Deberías ver la siguiente salida: `Hello World!`

Ahora tenemos una aplicación básica de NestJS que servirá como backend para nuestro sistema de reservas. Las aplicaciones NestJS son similares a las aplicaciones Angular en términos de sintaxis y arquitectura:

- El módulo principal en una aplicación NestJS es el archivo `app.module.ts`:

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

La clase `AppModule` orquesta todas las partes de la aplicación, incluidos otros módulos, proveedores y controladores.

- Los controladores son responsables de comunicarse con los clientes e intercambiar mensajes a través de métodos de API, como GET y POST. El archivo `app.controller.ts` contiene el controlador básico de la aplicación:

```typescript
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
```

La clase `AppController` contiene una única operación GET que devuelve el resultado del método `getHello`.

- Los controladores, al igual que los componentes de Angular, dependen de servicios para interactuar con sistemas externos como bases de datos. El archivo `app.service.ts` define el método `getHello`:

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    return 'Hello World!';
  }
}
```

El método de la clase `AppService` es la fuente del texto que se muestra en el navegador.

- El archivo `main.ts` utiliza la clase `AppModule` para arrancar (*bootstrap*) la aplicación NestJS:

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

La aplicación escucha de forma predeterminada en el puerto 3000, pero podemos especificar un puerto alternativo en tiempo de ejecución utilizando la variable de entorno `PORT`.

Ahora tienes una comprensión básica de una aplicación NestJS y podemos comenzar a construir la solución de reservas sobre ella. La característica principal de la solución son las reservas, que serán un módulo separado con el controlador y servicio que lo acompañan:

1. Ejecuta el siguiente comando para crear la funcionalidad de reservas:

```bash
nest generate resource reservations
```

El comando anterior creará automáticamente el módulo, el controlador y el servicio para la nueva funcionalidad.

2. En la primera pregunta, presiona ENTER para seleccionar **REST API** como la capa de transporte a utilizar. La aplicación Angular utilizará métodos de API REST para comunicarse con el servidor.
3. En la siguiente pregunta, presiona ENTER para indicarle a la CLI de NestJS que cree puntos de entrada CRUD para la aplicación. Los puntos de entrada CRUD utilizan Objetos de Transferencia de Datos (*Data Transfer Objects* o DTOs) para definir las estructuras de datos intercambiadas entre el cliente y el servidor. Son un contrato entre las dos partes que garantiza la seguridad de tipos y el formato correcto de los datos intercambiados.
4. La CLI creará todos los archivos y dependencias requeridos dentro de la carpeta `reservations`. Abre el archivo `create-reservation.dto.ts` y modifícalo de la siguiente manera:

```typescript
export class CreateReservationDto {
  readonly name: string;
  readonly start: string;
  readonly room: number;
  readonly email: string;
}
```

La clase `CreateReservationDto` define las propiedades de una nueva reserva. El cliente debe adherirse a la estructura anterior para enviar una nueva reserva.

El archivo `update-reservation.dto.ts` define la estructura para actualizar una reserva. Los DTOs que actualizan objetos suelen incluir todas las propiedades requeridas para crear un nuevo objeto y marcarlas como opcionales, utilizando la clase `PartialType`.

5. Inicia la aplicación usando el comando `nest start` y navega a `http://localhost:3000/reservations`. Deberías ver la siguiente salida: `This action returns all reservations`.

El mensaje anterior proviene del método `findAll` del archivo `reservations.service.ts`:

```typescript
findAll() {
  return `This action returns all reservations`;
}
```

Cualquier cliente que admita llamadas a API REST puede utilizar la aplicación backend y usar datos de reservas del sistema de reservas. En la siguiente sección, aprenderemos a manejar datos y persistirlos en una base de datos.

---

### Sección: Integración del Acceso a la Base de Datos

NestJS admite cualquier proveedor de bases de datos, ya sea SQL o NoSQL, según tus necesidades. Puede integrarse directamente con un controlador nativo de Node.js adecuado para la base de datos específica o utilizar una biblioteca de integración para una abstracción de nivel superior. Para este proyecto, utilizaremos MongoDB, una popular base de datos NoSQL que requiere una configuración mínima y se integra a la perfección con las aplicaciones NestJS. La flexibilidad de MongoDB y la ausencia de un tipo de esquema estricto la convierten en una solución atractiva para este proyecto, donde no tenemos datos relacionales.

Utilizaremos la Community Edition de MongoDB, que es una versión autogestionada de la base de datos que instalamos localmente:

1. Navega a [https://www.mongodb.com/try/download/community](https://www.mongodb.com/try/download/community).
2. Descarga la última versión de MongoDB según tu sistema operativo.
3. Ejecuta el archivo descargado y sigue las instrucciones para instalar MongoDB.

Utilizaremos la biblioteca de modelado de objetos Mongoose para integrar MongoDB en la aplicación de reservas. Instala Mongoose mediante el siguiente comando de terminal:

```bash
npm install @nestjs/mongoose mongoose
```

NestJS mantiene un paquete dedicado `@nestjs/mongoose` que requiere la biblioteca nativa `mongoose`.

1. Abre el archivo `app.module.ts` e importa la clase `MongooseModule`:

```typescript
import { MongooseModule } from '@nestjs/mongoose';
```

2. Agrega la clase anterior al array `imports` del decorador del módulo:

```typescript
imports: [
  ReservationsModule,
  MongooseModule.forRoot('mongodb://127.0.0.1/studio')
]
```

El método `forRoot` acepta la cadena de conexión utilizada para conectarse a la instancia local de MongoDB. La parte `studio` representa el nombre de la base de datos.

La aplicación tiene acceso a la base de datos local de MongoDB, pero actualmente no contiene ningún esquema. Crearemos el esquema de base de datos para las reservas:

1. Abre el archivo `reservation.entity.ts` y agrega las siguientes sentencias de importación:

```typescript
import { Prop, Schema, SchemaFactory } from '@nestjs/mongoose';
```

La clase `Reservation` es responsable de definir una entidad de reserva de la base de datos.

2. Agrega el decorador `@Schema` encima de la clase TypeScript:

```typescript
@Schema()
export class Reservation {}
```

El decorador anterior mapea la entidad a una colección de MongoDB.

3. Agrega las siguientes propiedades a la clase `Reservation`:

```typescript
@Prop()
name: string;

@Prop()
start: string;

@Prop()
room: number;

@Prop()
email: string;
```

La estructura anterior define el documento subyacente de MongoDB contenido en la colección `reservation`. El decorador `@Prop` marca las propiedades del documento en la sintaxis de MongoDB.

4. Convierte la clase en un esquema con el siguiente comando para que podamos usarla a través del sistema de inyección de dependencias de NestJS:

```typescript
export const ReservationSchema = SchemaFactory.createForClass(Reservation);
```

Podemos utilizar el esquema de reserva en el archivo `reservations.service.ts` para devolver datos reales de la base de datos:

1. Abre el archivo `reservations.module.ts` e importa el modelo `Reservation` junto con su esquema:

```typescript
import { MongooseModule } from '@nestjs/mongoose';
import { Reservation, ReservationSchema } from './entities/reservation.entity';
```

2. Agrega la siguiente propiedad `imports` al decorador del módulo:

```typescript
imports: [MongooseModule.forFeature([
  { name: Reservation.name, schema: ReservationSchema }
])]
```

El método `forFeature` registra el modelo de reserva con el módulo actual.

3. Abre el archivo del servicio y agrega las siguientes sentencias de importación:

```typescript
import { Reservation } from './entities/reservation.entity';
import { InjectModel } from '@nestjs/mongoose';
import { Model } from 'mongoose';
```

4. Crea un constructor que utilice todas las clases anteriores de la siguiente manera:

```typescript
constructor(
  @InjectModel(Reservation.name) private reservationModel: Model<Reservation>
) {}
```

Usamos el decorador `@InjectModel` para inyectar el modelo `Reservation` en el servicio.

5. Reemplaza el contenido del método `findAll` con el siguiente fragmento:

```typescript
return this.reservationModel.find().exec();
```

El fragmento anterior consulta la colección `reservation` en la base de datos y devuelve todos los registros.

6. Ejecuta la aplicación con la CLI de NestJS y navega a `http://localhost:3000/reservations` para verificar que la salida del navegador sea la siguiente: `[]`.

La API devuelve un array vacío porque todavía no hay reservas en la base de datos. El array vacío también indica una conexión exitosa a la base de datos.

Si utilizas una interfaz gráfica de usuario para conectarte a MongoDB, como MongoDB Compass, verás la base de datos `studio` que contiene la colección `reservations`:

> *Figura 6.1 – MongoDB Compass*

La colección de MongoDB es el nombre del modelo en minúsculas, terminado en una *s*.

La instalación de MongoDB te solicita que configures MongoDB Compass, lo cual es opcional. Alternativamente, puedes descargarlo manualmente en [https://www.mongodb.com/try/download/compass](https://www.mongodb.com/try/download/compass).

Hemos configurado el backend de nuestra aplicación para utilizar una base de datos para la persistencia de datos. En la siguiente sección, comenzaremos a implementar la parte frontend de la aplicación de reservas.

---

### Sección: Instalación de PrimeNG

PrimeNG es una suite popular para el framework Angular que incluye componentes de interfaz de usuario altamente personalizables. Usaremos el componente stepper que se comporta como un asistente de interfaz de usuario (*wizard*) y nos permitirá interpretar el proceso de reserva en pasos separados y distintos. La biblioteca cuenta con una rica colección de plantillas, tanto gratuitas como de pago, para ayudarte a comenzar en tu próximo proyecto. También admite temas integrados que puedes modificar según las necesidades de tu aplicación. Usaremos el tema integrado Nora que ha sido diseñado específicamente para aplicaciones listas para entornos empresariales.

En esta sección, aprenderás a instalar y configurar PrimeNG en la aplicación de reservas:

1. Copia la carpeta del proyecto del Capítulo 1, *Angular AI Kick-Starter: Scaffolding Smart Apps with Copilot*, dentro de la carpeta raíz `bookmaster` y cámbiale el nombre a `client`.
2. Instala PrimeNG ejecutando el siguiente comando dentro de la carpeta `client` desde una ventana de terminal:

```bash
npm install primeng @primeuix/themes
```

El comando anterior instalará PrimeNG y sus temas integrados.

Todos los comandos de terminal para la aplicación frontend deben ejecutarse dentro de la carpeta `client` del proyecto actual. Si tienes problemas al ejecutar el comando anterior, consulta el archivo `CHANGELOG.md` del repositorio de GitHub del capítulo. Para acceder al enlace del repositorio, sigue los pasos de la sección "Download the example code files" en el Prefacio.

3. Abre el archivo `app.config.ts` y agrega las siguientes sentencias de importación:

```typescript
import { providePrimeNG } from 'primeng/config';
import Nora from '@primeuix/themes/nora';
```

La última sentencia en el fragmento anterior importa el tema Nora. PrimeNG admite los siguientes temas integrados:

- **Aura:** Un tema con opiniones de PrimeTek, la empresa detrás de PrimeNG.
- **Material:** Un tema inspirado en Google Material Design.
- **Lara:** Un tema adaptado a Bootstrap.
- **Nora:** Un tema destinado a aplicaciones de nivel empresarial.

4. Usa el array `providers` para proporcionar el tema globalmente a la aplicación:

```typescript
export const appConfig: ApplicationConfig = {
  providers: [
    provideBrowserGlobalErrorListeners(),
    provideRouter(routes),
    providePrimeNG({
      theme: {
        preset: Nora
      }
    })
  ]
};
```

5. Abre el archivo `app.ts` y crea el siguiente constructor para establecer el título del capítulo:

```typescript
constructor() {
  this.chapterTitleService.setTitle(
    'Chapter 6: Studio BookMaster'
  );
}
```

6. Ejecuta la aplicación usando el comando `ng serve` y navega a `localhost:4200`:

> *Figura 6.2 – Salida de la aplicación*

Instalar PrimeNG cambia el estilo de la página como se muestra en la figura anterior. La página principal ahora muestra el título del capítulo actual.

Los temas integrados de PrimeNG admiten modos oscuro y claro. La imagen anterior variará según el modo de tu sistema. Puedes aprender a establecer un valor predeterminado en [https://primeng.org/theming/styled#darkmode](https://primeng.org/theming/styled#darkmode).

Modificaremos la aplicación para mostrar el título en un componente toolbar de PrimeNG:

1. Abre el archivo `app.html` y reemplaza su contenido con el siguiente código HTML:

```html
<chapter-title [chapterTitle]="title()" />
```

2. Abre el archivo `chapter-title.component.ts` y agrega la siguiente sentencia de importación:

```typescript
import { ToolbarModule } from 'primeng/toolbar';
```

La clase `ToolbarModule` contiene el componente de encabezado que mostrará el título del capítulo.

3. Reemplaza la propiedad `template` del decorador del componente con el siguiente código HTML:

```html
<p-toolbar>
  <span>{{ chapterTitle() }}</span>
</p-toolbar>
```

El elemento `<p-toolbar>` es un componente PrimeNG que representa una barra de herramientas. Los componentes de PrimeNG comienzan con el prefijo `p`.

4. Agrega la clase `ToolbarModule` al array `imports` del decorador del componente:

```typescript
imports: [ToolbarModule]
```

5. Abre el archivo `styles.scss` y agrega la siguiente configuración para usar fuentes personalizadas distintas a las predeterminadas del sistema:

```scss
html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol';
}
```

6. Verifica que la salida de la aplicación muestre lo siguiente:

> *Figura 6.3 – Encabezado de la aplicación*

El diseño de la aplicación ahora sigue el diseño de PrimeNG. Cuenta con un encabezado que muestra el título del capítulo. El componente de encabezado consta de un borde que puede agrupar otros componentes, como botones e iconos.

En la siguiente sección, aprenderemos a utilizar más componentes de PrimeNG y diseñaremos un formulario de reserva.

---

### Sección: Creación de Reservas

El proceso para realizar una nueva reserva debe ser claro y fácil para los usuarios de la aplicación. El sistema utilizará un calendario para seleccionar una fecha de reserva y mostrará campos de entrada adicionales para recopilar los datos requeridos, como el nombre y el correo electrónico. Separaremos el proceso de reserva en pasos distintos utilizando el componente stepper de PrimeNG, que se comporta como un asistente de interfaz de usuario.

El componente stepper para la reserva de estudio contendrá los siguientes pasos:

- Nombre completo y correo electrónico del usuario
- Fecha y hora de la reserva
- Sala de estudio

Comencemos a construir el asistente de reservas:

1. Primero, ejecutaremos el siguiente comando de Angular CLI que creará un componente para alojar el asistente de reservas:

```bash
ng generate component booking
```

2. Abre el archivo `booking.ts` y agrega la siguiente sentencia de importación:

```typescript
import { StepperModule } from 'primeng/stepper';
```

La clase `StepperModule` contiene todos los elementos necesarios para crear un componente stepper en PrimeNG.

3. Agrega la clase `StepperModule` al array `imports` del decorador del componente:

```typescript
imports: [StepperModule]
```

4. Abre el archivo `booking.html` y reemplaza su contenido con el código HTML:

```html
<p-stepper [value]="1">
  <p-step-list>
    <p-step [value]="1">Basic info</p-step>
    <p-step [value]="2">Date/Time</p-step>
    <p-step [value]="3">Room</p-step>
  </p-step-list>
  <p-step-panels>
    <p-step-panel [value]="1"></p-step-panel>
    <p-step-panel [value]="2"></p-step-panel>
    <p-step-panel [value]="3"></p-step-panel>
  </p-step-panels>
</p-stepper>
```

La etiqueta `<p-stepper>` indica el componente stepper y la propiedad `value` define el paso seleccionado. La etiqueta `<p-step-list>` define los pasos individuales del stepper. Cada paso es una etiqueta `<p-step>` que contiene el valor y el nombre del paso. La etiqueta `<p-step-panels>` define una lista de componentes `<p-step-panel>` que alojan el contenido HTML de cada paso.

5. Abre el archivo `app.ts` e importa la clase del componente booking:

```typescript
import { Booking } from './booking/booking';
```

6. Agrega la clase `Booking` en el array `imports` del decorador del componente:

```typescript
imports: [ChapterTitleComponent, Booking]
```

También puedes eliminar la clase `RouterOutlet`, ya que no utilizaremos el Router de Angular en este capítulo.

7. Abre el archivo `app.html` y agrega el componente booking a la plantilla:

```html
<chapter-title [chapterTitle]="title()" />
<app-booking />
```

8. Ejecuta la aplicación con el comando `ng serve` y previsualízala en `http://localhost:4200`:

> *Figura 6.4 – Pasos de la reserva*

La imagen anterior muestra los pasos del asistente de reservas. Cada paso actualmente no contiene campos. Primero, construiremos el paso Información básica (*Basic info*):

1. Abre el archivo `booking.ts` y agrega las siguientes sentencias de importación:

```typescript
import { FormsModule } from '@angular/forms';
import { InputTextModule } from 'primeng/inputtext';
```

Necesitamos `FormsModule` para usar formularios de plantilla de Angular e `InputTextModule` para dar estilo a los campos de entrada como componentes de texto de PrimeNG.

2. Agrega las clases anteriores al array `imports` del decorador del componente:

```typescript
imports: [StepperModule, FormsModule, InputTextModule]
```

3. Importa el método `model` del paquete npm `@angular/core` para definir los campos de entrada del asistente como signals:

```typescript
import { Component, model } from '@angular/core';
```

4. Agrega las siguientes propiedades a la clase `Booking`:

```typescript
readonly name = model('');
readonly email = model('');
```

Las propiedades anteriores definen el nombre completo y el correo electrónico del usuario.

5. Abre el archivo `booking.html` e inserta el siguiente fragmento dentro de la primera etiqueta `<p-step-panel>`:

```html
<ng-template #content>
  <div class="container">
    <div class="control">
      <label for="name">Name</label>
      <input [(ngModel)]="name" pInputText id="name" fluid />
    </div>
    <div class="control">
      <label for="email">Email</label>
      <input [(ngModel)]="email" pInputText id="email" fluid />
    </div>
  </div>
</ng-template>
```

En el fragmento anterior, definimos dos elementos HTML de entrada que se vinculan a las propiedades signal subyacentes a través de `ngModel`. El atributo `pInputText` indica que son componentes de entrada de PrimeNG y el atributo `fluid` indica que se expanden a través del ancho completo de su contenedor. La variable de referencia de plantilla `content` garantiza que los controles de entrada aparezcan cuando seleccionamos el primer paso del asistente de reserva.

6. Abre el archivo `booking.scss` y agrega estilos para las clases `container` y `control`:

```scss
.container {
  max-width: 20rem;
}

.control {
  padding: 0.25rem;
  * {
    margin: 0.25rem;
  }
}
```

Las clases anteriores garantizan que la aplicación muestre los campos de entrada verticalmente con un espaciado adecuado.

7. Ejecuta la aplicación y verifica que el paso Basic info se vea de la siguiente manera:

> *Figura 6.5 – Paso de información básica*

Para el siguiente paso del asistente, utilizaremos un conjunto diferente de componentes de PrimeNG para definir la fecha y hora de la reserva:

1. Importa las siguientes clases en el archivo `booking.ts`:

```typescript
import { DatePickerModule } from 'primeng/datepicker';
import { SelectButtonModule } from 'primeng/selectbutton';
```

La clase `DatePickerModule` contiene componentes PrimeNG para la manipulación de fechas. Usaremos la clase `SelectButtonModule` para representar la hora de reserva como una lista de botones, lo que permite a los usuarios seleccionar solo uno.

2. Agrega las clases anteriores al array `imports` del decorador del componente:

```typescript
imports: [
  StepperModule,
  FormsModule,
  InputTextModule,
  DatePickerModule,
  SelectButtonModule
]
```

3. Define las siguientes propiedades signal en la clase del componente `Booking`:

```typescript
readonly date = model('');
readonly time = model('');
readonly slots = signal([
  '12:00','13:00','14:00','15:00','16:00','17:00','18:00'
]);
```

Las propiedades anteriores utilizan los métodos `model` y `signal` del paquete `@angular/core` para representar la fecha y hora de la reserva, así como las franjas horarias disponibles.

Definimos las franjas horarias en línea por simplicidad. Alternativamente, podríamos obtenerlas dinámicamente desde la API según la disponibilidad de las salas. Para este proyecto, asumiremos que la duración máxima que un usuario puede reservar una sala es de 1 hora.

4. Inserta el siguiente fragmento HTML dentro de la segunda etiqueta `<p-step-panel>`:

```html
<ng-template #content>
  <div class="date-container">
    <p-datepicker [(ngModel)]="date" [inline]="true" />
    <p-selectbutton [options]="slots()" [(ngModel)]="time" />
  </div>
</ng-template>
```

El fragmento anterior envuelve los componentes datepicker y selectbutton en un elemento `<div>`. Vinculamos el selector de fechas a la propiedad `date` y lo configuramos en `inline` para que siempre se muestre. Definimos las opciones para el componente selectbutton como las franjas horarias y vinculamos su valor a la propiedad `time`.

5. Abre el archivo `booking.scss` y agrega los siguientes estilos para los nuevos componentes que usamos:

```scss
.date-container {
  display: flex;
  flex-direction: row;
  gap: 2rem;
  max-width: 600px;
}

.p-selectbutton {
  flex-direction: column;
  flex-grow: 0.5;
}

::ng-deep .p-selectbutton .p-togglebutton {
  border-width: 1px !important;
}
```

Las clases anteriores alinearán el selector de fechas y el botón de selección en una sola fila. Los selectores `.p-selectbutton` y `.p-togglebutton` son clases integradas de PrimeNG para el componente selectbutton y alinearán las franjas horarias verticalmente.

Sobrescribimos el estilo `border-width` con la palabra clave `!important` porque el componente selectbutton no admite la orientación vertical de forma predeterminada.

6. Ejecuta la aplicación y selecciona el paso Date/Time del asistente de reservas para previsualizar el nuevo paso:

> *Figura 6.6 – Paso de fecha y hora*

La imagen anterior muestra el selector de fechas y el selector de franjas horarias para la nueva reserva.

El último paso incluirá un campo desplegable para que los usuarios seleccionen la sala de reserva. Usaremos el componente select de la biblioteca PrimeNG:

1. Importa la clase `SelectModule` en el archivo `booking.ts`:

```typescript
import { SelectModule } from 'primeng/select';
```

2. Agrega la clase anterior al array `imports` del decorador del componente:

```typescript
imports: [
  StepperModule,
  FormsModule,
  InputTextModule,
  DatePickerModule,
  SelectButtonModule,
  SelectModule
]
```

3. Agrega la siguiente propiedad signal que representa la sala seleccionada:

```typescript
readonly room = signal<number | undefined>(undefined);
```

4. Inserta el siguiente fragmento HTML dentro de la última etiqueta `<p-step-panel>`:

```html
<ng-template #content>
  <div class="control">
    <label>Room no</label>
    <p-select [options]="[1,2,3]" [(ngModel)]="room" />
  </div>
</ng-template>
```

En el fragmento anterior, asumimos que el estudio tiene 3 salas.

5. Ejecuta la aplicación, selecciona el paso Room y abre el campo desplegable para previsualizar su contenido:

> *Figura 6.7 – Paso de sala*

El asistente de reservas contiene todos los campos necesarios para crear una nueva reserva. Agregaremos un botón que recopila todos los datos de entrada y los envía a la API backend para almacenarlos en la base de datos.

Primero, crearemos el servicio Angular que comunica los datos a la API backend utilizando el cliente HTTP:

1. Crea el servicio mediante el comando de Angular CLI a continuación:

```bash
ng generate service data
```

2. Abre el archivo `data.ts` e importa los símbolos `inject` y `HttpClient`:

```typescript
import { Service, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
```

3. Inyecta el cliente HTTP en la clase del servicio `Data` con la siguiente propiedad:

```typescript
private http = inject(HttpClient);
```

4. Agrega el método `create` para enviar datos de reserva a la API backend:

```typescript
create(
  name: string,
  email: string,
  date: string,
  room: number
) {
  return this.http.post('http://localhost:3000/reservations', {
    name,
    email,
    start: date,
    room
  });
}
```

El método anterior emite una solicitud POST a la URL de la API, enviando los datos de reserva.

Ahora, agregaremos un botón en el último paso del asistente que utiliza el servicio Angular:

1. Agrega las siguientes sentencias de importación en el archivo `booking.ts`:

```typescript
import { ButtonModule } from 'primeng/button';
import { Data } from '../data';
```

2. Agrega la clase `ButtonModule` en el array `imports` del decorador del componente:

```typescript
imports: [
  StepperModule,
  FormsModule,
  InputTextModule,
  DatePickerModule,
  SelectButtonModule,
  SelectModule,
  ButtonModule
]
```

3. Inyecta el servicio `Data` en la clase del componente `Booking` usando el método `inject` del paquete `@angular/core`:

```typescript
private data = inject(Data);
```

4. Agrega la siguiente propiedad signal que utiliza el método `computed` del paquete `@angular/core`:

```typescript
readonly bookingDate = computed(() => {
  const d = new Date(this.date());
  const offset = d.getTime() - d.getTimezoneOffset() * 60000;
  const offdate = new Date(offset);
  const t = Number(this.time().substring(0, this.time().indexOf(':')));
  offdate.setHours(t);
  return offdate.toString();
});
```

La signal `bookingDate` recibe la selección de fecha de la interfaz de usuario como un objeto `Date` nativo que no contiene la hora local real. Calcula la hora en la variable `offset` y luego crea un nuevo objeto de fecha que contiene la hora local real. Finalmente, analiza la selección de hora eliminando la parte `:00` de la cadena de franja horaria.

5. Crea el siguiente método que la aplicación ejecuta cuando el usuario presiona el botón Save:

```typescript
save() {
  this.data.create(
    this.name(),
    this.email(),
    this.bookingDate(),
    this.room()!
  ).subscribe();
}
```

El método `save` recopila los datos de reserva del asistente y los pasa al método de servicio `create`.

6. Abre el archivo `booking.html` y agrega un componente de botón PrimeNG en el último paso del asistente, después del componente select, de la siguiente manera:

```html
<div class="action">
  <p-button label="Save" size="small" (click)="save()" />
</div>
```

El botón llamará al método `save` al hacer clic.

7. Agrega el siguiente estilo CSS al archivo `booking.scss`:

```scss
.action {
  display: flex;
  flex-direction: row;
  justify-content: end;
}
```

El estilo anterior alineará el botón a la derecha del paso del asistente.

8. Abre la carpeta `server` dentro de VSCode y modifica el archivo `main.ts` de la siguiente manera:

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableCors();
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

Habilitamos el intercambio de recursos de origen cruzado (*Cross-Origin Resource Sharing* o CORS) mediante el método `enableCors` para permitir el intercambio de datos entre las aplicaciones frontend y backend.

CORS es un mecanismo HTTP que se utiliza para controlar el acceso a los recursos en un servidor. Lee más sobre CORS en [https://docs.nestjs.com/security/cors](https://docs.nestjs.com/security/cors). Debemos habilitarlo porque se ejecutan en diferentes puertos en la máquina local: Angular en el 4200 y NestJS en el 3000.

9. Abre el archivo `reservation.service.ts` y reemplaza el contenido del método `create` con el siguiente código:

```typescript
const res = new this.reservationModel(createReservationDto);
return res.save();
```

El fragmento anterior crea una nueva entidad de reserva a partir del `createReservationDto`, la guarda en la base de datos y devuelve el resultado a quien realizó la llamada. El resultado será la entidad de la base de datos creada por la operación.

La lógica para crear nuevas reservas en ambas partes de la aplicación ya está implementada. Podemos reservar una nueva cita siguiendo los pasos a continuación:

1. Inicia el backend ejecutando el comando `nest start`.
2. Inicia el frontend ejecutando el comando `ng serve`.
3. Navega a `http://localhost:4200` y completa todos los pasos requeridos del asistente.
4. Haz clic en el botón Save en el último paso y verifica que los datos de la reserva se guarden en la base de datos navegando a `http://localhost:3000/reservations` o usando una herramienta GUI como MongoDB Compass a continuación:

> *Figura 6.8 – Datos de reserva de muestra*

Hemos alcanzado un hito importante en la aplicación Studio BookMaster. Nuestros usuarios pueden usar la aplicación para reservar una sala y el estudio puede ver la reserva al instante.

En el proyecto actual, no implementaremos una aplicación de back-office para el personal del estudio. Nuestro enfoque principal estará en el lado del consumidor.

En la siguiente sección, mejoraremos la experiencia de usuario de la aplicación mostrando la lista de selección de salas según la disponibilidad horaria.

---

### Sección: Visualización de Salas Disponibles

La aplicación permite a los usuarios seleccionar cualquier sala, lo que puede provocar conflictos cuando dos usuarios reservan la misma sala durante el mismo período de tiempo. Para abordar esta posibilidad y mejorar la experiencia de usuario, modificaremos la aplicación para que el frontend recupere la lista de salas dinámicamente desde la API.

La API encontrará qué salas están disponibles en el período de tiempo seleccionado:

1. Navega a la carpeta `server` del proyecto actual y abre el archivo `reservations.controller.ts`.
2. Agrega una nueva ruta a la clase `ReservationsController` con los siguientes contenidos:

```typescript
@Post('rooms')
findRooms(@Body() roomDto: { start: string }) {
  return this.reservationsService.findRooms(roomDto.start);
}
```

El método `findRooms` recupera el parámetro `start` del cuerpo de la solicitud, que representa la fecha y hora de la reserva.

3. Abre el archivo `reservations.service.ts` y agrega el método `findRooms` respectivo:

```typescript
async findRooms(start: string) {
  const res = await this.reservationModel.findOne({ start });
  return [1, 2, 3].filter(room => room !== res?.room);
}
```

El método anterior comprueba si hay una reserva para la fecha y hora de inicio específicas y filtra la lista de salas según la sala reservada.

La implementación del método es simple y asume que si ha comenzado una reserva en una sala, ya no se puede reservar la misma sala. El escenario anterior sería más preciso si la aplicación tuviera una función para liberar una sala asignada.

Refactorizaremos la parte frontend de la aplicación para que use la nueva ruta para recuperar las salas disponibles. Reemplazaremos la lista estática en el tercer paso del asistente con la lista de salas de la API:

1. Navega a la carpeta `client` del proyecto y abre el archivo `data.ts`.
2. Crea la siguiente variable en la clase del servicio `Data`:

```typescript
private apiUrl = 'http://localhost:3000';
```

Creamos una variable separada para la URL de la API para que podamos reutilizarla en nuestro servicio.

3. Modifica el método `create` para usar la nueva variable:

```typescript
return this.http.post(this.apiUrl + '/reservations', {
  name,
  email,
  start: date,
  room
});
```

4. Crea un nuevo método `getRooms` con el siguiente contenido:

```typescript
getRooms(date: string) {
  return this.http.post<number[]>(
    this.apiUrl + '/reservations/rooms',
    { start: date }
  );
}
```

El método anterior llama a la nueva ruta de la API y devuelve una lista de números de sala.

5. Abre el archivo `booking.ts` y crea la siguiente propiedad en la clase del componente `Booking`:

```typescript
readonly rooms = signal<number[]>([]);
```

6. Crea un nuevo método que recupere las salas disponibles para la fecha de reserva seleccionada:

```typescript
getRooms() {
  this.data.getRooms(this.bookingDate()).subscribe(
    rooms => this.rooms.set(rooms)
  );
}
```

El método anterior se suscribe al método `getRooms` y asigna el resultado a la propiedad `rooms` del componente.

7. Abre el archivo `booking.html` y agrega el siguiente evento `click` al último elemento `<p-step>`:

```html
<p-step [value]="3" (click)="getRooms()">Room</p-step>
```

El componente llamará al método `getRooms` cada vez que seleccionemos el paso Room.

8. Vincula las opciones del componente select a la propiedad `rooms`:

```html
<p-select [options]="rooms()" [(ngModel)]="room" />
```

El componente select mostrará los números de sala provenientes de la API.

Cuando el usuario haga clic en el paso Room, la aplicación mostrará solo las salas que están libres en la fecha y hora seleccionadas. Luego, el usuario puede hacer clic en el botón Save para guardar la reserva en la base de datos. Sin embargo, la aplicación no proporciona ningún comentario al usuario. En la siguiente sección, aprenderemos a enviar un mensaje de confirmación al usuario al completar con éxito el proceso de reserva.

---

### Sección: Envío de Mensajes de Confirmación

El primer paso del asistente de reserva pide a los usuarios que introduzcan su nombre completo y dirección de correo electrónico. La API utilizará la dirección de correo electrónico para enviar un mensaje de confirmación tras completar con éxito el proceso de reserva con Nodemailer. Es una biblioteca popular para enviar correos electrónicos a través de Node.js y es compatible con el entorno NestJS mediante la biblioteca `@nestjs-modules/mailer`.

Utilizaremos la biblioteca Nodemailer en el módulo de reservas de la aplicación backend:

1. Navega a la carpeta `server` e instala `nodemailer` y la biblioteca respectiva de NestJS mediante el siguiente comando:

```bash
npm install @nestjs-modules/mailer nodemailer
```

2. Abre el archivo `app.module.ts` y agrega la siguiente sentencia de importación:

```typescript
import { MailerModule } from '@nestjs-modules/mailer';
```

La clase `MailerModule` proporciona servicios para configurar y utilizar la biblioteca Nodemailer para NestJS.

3. Registra Nodemailer en el módulo de la aplicación agregando el siguiente fragmento en el array `imports` del decorador del módulo:

```typescript
MailerModule.forRoot({
  transport: connection
})
```

El método `forRoot` acepta un objeto de parámetros con una propiedad `transport`. La propiedad anterior representa la conexión a tu proveedor de correo electrónico. Puede ser un proveedor personalizado o uno de los integrados, como Gmail u Outlook.

Lee más en [https://nodemailer.com/smtp](https://nodemailer.com/smtp) para crear un método de transporte personalizado.

En el código fuente que encontrarás en la sección Requisitos técnicos, estamos usando Gmail:

```typescript
const connection = {
  service: 'gmail',
  auth: {
    user: '',
    pass: ''
  }
};
```

La opción `user` es tu cuenta de Gmail y la opción `pass` es una contraseña de aplicación de Gmail que puedes crear en [https://myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords).

Por motivos de simplicidad, almacenamos las credenciales de correo electrónico dentro del código fuente. En un entorno de producción, no deberías hacer eso por razones de seguridad. Alternativamente, guárdalas en variables de entorno o dentro de un archivo `.env` fuera del código de la aplicación. Aprenderemos a almacenar datos sensibles en archivos de entorno con NestJS en la siguiente sección.

4. Abre el archivo `reservations.service.ts` y agrega la siguiente sentencia de importación:

```typescript
import { MailerService } from '@nestjs-modules/mailer';
```

La clase `MailerService` expone un método para enviar mensajes de correo electrónico.

5. Inyecta `MailerService` en el constructor de la clase del servicio:

```typescript
constructor(
  @InjectModel(Reservation.name) private reservationModel: Model<Reservation>,
  private mailer: MailerService
) {}
```

6. Modifica el método `create` de la siguiente manera:

```typescript
async create(createReservationDto: CreateReservationDto) {
  const res = new this.reservationModel(createReservationDto);
  await res.save();
  await this.mailer.sendMail({
    to: createReservationDto.email,
    subject: 'Booking Confirmed ',
    text: `You have booked room ${createReservationDto.room} for ${createReservationDto.start} `
  });
  return { message: 'Booking confirmed' };
}
```

Usamos el método `sendMail` de la clase `MailerService` para enviar un mensaje al usuario que realizó la reserva. Acepta un objeto de opciones como parámetro que define la dirección de correo electrónico del usuario, el asunto del mensaje y el texto del mensaje.

`sendMail` también admite archivos HTML y de plantilla utilizando las propiedades `html` y `template` en el objeto de opciones. El método anterior también devuelve un objeto con una propiedad `message` que podemos mostrar en la aplicación frontend.

Cuando el usuario hace clic en el botón Save, Angular envía los datos de reserva a la API y el backend envía un mensaje de correo electrónico al usuario. Sin embargo, el usuario no puede saber si la reserva se realizó correctamente en el frontend. Usaremos el resultado del método `create` para mostrar un mensaje de información al usuario:

1. Navega a la carpeta `client`, abre el archivo `data.ts` y modifica la solicitud POST en el método `create` de la siguiente manera:

```typescript
return this.http.post<{ message: string }>(this.apiUrl + '/reservations', {
  name,
  email,
  start: date,
  room
});
```

La solicitud POST devuelve un objeto que coincide con el tipo de resultado de la llamada al método de la API.

2. Abre el archivo `booking.ts` y agrega la siguiente sentencia de importación:

```typescript
import { MessageModule } from 'primeng/message';
```

La clase `MessageModule` incluye un componente para mostrar mensajes en línea con PrimeNG.

3. Agrega la clase anterior al array `imports` del decorador del componente:

```typescript
imports: [
  StepperModule,
  FormsModule,
  InputTextModule,
  DatePickerModule,
  SelectButtonModule,
  SelectModule,
  ButtonModule,
  MessageModule
]
```

4. Define una propiedad signal en la clase del componente `Booking` que almacenará el resultado de la API:

```typescript
readonly msg = signal('');
```

5. Modifica el método `save` de la siguiente manera:

```typescript
save() {
  this.data.create(
    this.name(),
    this.email(),
    this.bookingDate(),
    this.room()!
  ).subscribe(result => this.msg.set(result.message));
}
```

Establecemos el mensaje de la llamada al método de la API en el valor de la propiedad signal `msg`.

6. Abre el archivo `booking.html` y reemplaza el elemento `<div>` que contiene la clase `action` con el siguiente fragmento:

```html
@if (msg()) {
  <p-message>{{msg()}}</p-message>
} @else {
  <div class="action">
    <p-button label="Save" size="small" (click)="save()" />
  </div>
}
```

Cuando `msg` tiene un valor, lo mostramos envolviéndolo dentro de un elemento `<p-message>`. De lo contrario, mostramos el botón Save.

Todas las partes de la aplicación están listas para enviar un mensaje de correo electrónico y mostrar una confirmación al usuario. Repite los siguientes pasos para obtener una vista previa de la nueva funcionalidad:

1. Ejecuta el frontend y el backend de la aplicación utilizando los comandos respectivos.
2. Navega a `http://localhost:4200` y completa todos los pasos en el asistente de reservas.
3. Haz clic en el botón Save para reservar tu cita.

La aplicación debería enviar un mensaje a la dirección de correo electrónico que proporcionaste con un aspecto similar al siguiente:

> *Figura 6.9 – Mensaje de correo electrónico*

La figura anterior muestra un correo electrónico de confirmación para una reserva en la sala 2. También mostrará un mensaje de confirmación al usuario:

> *Figura 6.10 – Confirmación de reserva*

La imagen anterior muestra el mensaje que devuelve la API tras una reserva exitosa.

La confirmación y el mensaje de correo electrónico completan el flujo de trabajo de reservas de nuestra aplicación. Los usuarios controlan todo el ciclo de vida de la reserva, desde ingresar los detalles necesarios hasta recibir una confirmación en su bandeja de entrada. En la siguiente sección, brindaremos a los usuarios la capacidad de interactuar con un agente de IA para completar la reserva.

---

### Sección: Reservas Mediante IA

En el capítulo anterior, aprendimos cómo integrar un asistente de IA en una aplicación Angular utilizando Firebase AI Logic en el cliente. En este capítulo, utilizaremos Google GenKit para integrar IA en una API de backend, lo que permitirá a los usuarios crear una reserva con un solo prompt. Agregar un asistente de IA a un sistema de reservas tiene los siguientes beneficios:

- Los usuarios pueden interactuar con el sistema de reservas utilizando una interfaz de chat familiar.
- Los desarrolladores pueden ofrecer una experiencia unificada con un solo prompt en lugar de diseñar una interfaz de usuario separada para cada necesidad empresarial.

GenKit es una biblioteca de código abierto que nos permite crear experiencias de IA en aplicaciones full-stack. Proporciona herramientas para interactuar con cualquier Modelo de Lenguaje Extenso (*Large Language Model* o LLM) de manera abstracta desde una aplicación Node.js.

La integración de IA en la aplicación se puede analizar en las siguientes tareas:

- Adición de GenKit a la API
- Creación de un asistente de IA

Comenzaremos aprendiendo cómo agregar GenKit a la aplicación NestJS.

#### Adición de GenKit a la API

Instalaremos y configuraremos GenKit en la parte backend de la aplicación:

1. Instala los paquetes de GenKit ejecutando el siguiente comando dentro de la carpeta `server` del proyecto:

```bash
npm install genkit @genkit-ai/google-genai
```

El comando anterior instala la biblioteca GenKit y la API de Gemini.

Aunque GenKit es un producto de Google, puede funcionar con otros proveedores de modelos de IA. Lee más en [https://genkit.dev/docs/models](https://genkit.dev/docs/models).

2. Navega a [https://aistudio.google.com/apikey](https://aistudio.google.com/apikey) y obtén una clave API de Gemini de Google AI Studio. Google AI Studio es una plataforma donde construyes y pruebas varias experiencias de IA de Google, encuentras documentación relevante y monitoreas tus proyectos de IA. Puedes crear una nueva clave de Gemini para un proyecto de Gemini existente o crear un nuevo proyecto desde cero.
3. Crea un archivo de entorno `.env` en la raíz de la carpeta `server` y agrega la clave API de Gemini en el siguiente formato:

```text
GEMINI_API_KEY=Key
```

En el fragmento anterior, reemplaza `Key` con el valor de tu clave API de Gemini.

4. Ejecuta el siguiente comando para instalar el paquete `@nestjs/config`:

```bash
npm install @nestjs/config
```

El paquete anterior contiene módulos y servicios para leer archivos de entorno en una aplicación NestJS.

5. Abre el archivo `app.module.ts` y agrega la siguiente sentencia de importación:

```typescript
import { ConfigModule } from '@nestjs/config';
```

Usaremos la clase `ConfigModule` para configurar archivos de entorno en nuestra aplicación.

6. Agrega `ConfigModule` en el array `imports` del decorador del módulo:

```typescript
imports: [
  ReservationsModule,
  MongooseModule.forRoot('mongodb://127.0.0.1/studio'),
  MailerModule.forRoot({ transport: connection }),
  ConfigModule.forRoot()
]
```

Usamos el método `forRoot` para cargar el archivo `.env` en nuestra aplicación y exportar la clave API de Gemini como una variable de entorno.

La aplicación backend tiene todas las dependencias requeridas para comunicarse con Gemini. Incluye la biblioteca GenKit que usaremos para interactuar con Gemini y expone la clave API de Gemini que GenKit necesita. En la siguiente sección, crearemos la interfaz de usuario necesaria para comunicarnos con la IA.

#### Creación de un Asistente de IA

Crearemos un nuevo servicio NestJS que se encargará de la comunicación con Gemini:

1. Crea un nuevo servicio ejecutando el siguiente comando dentro de la carpeta `reservations`:

```bash
nest generate service ai
```

El comando anterior creará el servicio en la carpeta `ai` y lo registrará en el módulo de reservas.

2. Abre el archivo `ai.service.ts` y agrega las siguientes sentencias de importación:

```typescript
import { Genkit, genkit, ToolAction, z } from 'genkit';
import { googleAI } from '@genkit-ai/google-genai';
import { ReservationsService } from '../reservations.service';
```

Los tipos `Genkit` y `ToolAction` definen una instancia de GenKit y sus herramientas. `genkit` es un método que crea una instancia de GenKit y `z` es la biblioteca Zod que garantiza una definición de datos de esquema con seguridad de tipos. `googleAI` es el sistema de complementos de Gemini Developer API. También necesitamos la clase `ReservationsService` para crear la nueva reserva.

3. Define las siguientes propiedades en la clase `AiService`:

```typescript
private ai: Genkit;
private tool: ToolAction;
```

4. Crea un constructor e inyecta la clase `ReservationService`:

```typescript
constructor(
  private reservationsService: ReservationsService
) {}
```

5. Agrega el siguiente contenido al cuerpo del constructor:

```typescript
this.ai = genkit({
  plugins: [googleAI()],
  model: googleAI.model('gemini-2.5-flash')
});
```

El constructor crea una nueva instancia de GenKit utilizando el método `genkit`. La propiedad `plugins` inicializa Gemini utilizando la clave API del archivo de entorno. La propiedad `model` indica el modelo de Gemini que queremos utilizar.

6. Inserta el siguiente fragmento debajo de la inicialización de GenKit:

```typescript
this.tool = this.ai.defineTool({
  name: 'createReservation',
  description: 'Creates a new reservation',
  inputSchema: z.object({
    name: z.string(),
    email: z.string(),
    start: z.string(),
    room: z.number()
  }),
  outputSchema: z.string()
}, async input => {
  const rooms = await this.reservationsService.findRooms(
    input.start
  );
  if (!rooms.includes(input.room)) {
    return `Room ${input.room} is unavailable`;
  }
  const res = await this.reservationsService.create(input);
  return res.message;
});
```

En el fragmento anterior, usamos el método `defineTool` para agregar una nueva herramienta a GenKit.

Las herramientas son capacidades que permiten a la IA interactuar con sistemas externos como bases de datos o APIs. Una herramienta tiene un nombre, una descripción, un esquema de entrada/salida y una lógica personalizada para ejecutar.

En nuestro caso, crearemos una herramienta que permita a GenKit realizar nuevas reservas. El `inputSchema` define el formato de los datos de entrada que coincide con el DTO de una nueva reserva. La lógica de la herramienta extrae los datos de entrada y comprueba la disponibilidad de las salas. Si la sala no está disponible para reservar, devuelve un mensaje de error; de lo contrario, procede a crear la reserva.

7. Agrega el siguiente método `ask` en la clase `AiService`:

```typescript
async ask(prompt: string) {
  const resp = await this.ai.generate({
    prompt,
    tools: [this.tool]
  });
  const msg = resp.messages.filter(m => m.role === 'tool');
  return { message: msg[0].content[0].toolResponse?.output };
}
```

El método anterior combina el prompt de texto con la herramienta de IA, lo ejecuta y recupera la respuesta del modelo. El array `messages` de la respuesta contiene todos los mensajes intercambiados con el modelo. Filtramos el array para encontrar solo aquellos que provienen de la herramienta seleccionada. Finalmente, extraemos el mensaje real de la propiedad `toolResponse`.

8. Abre el archivo `reservations.controller.ts` e importa la clase `AiService`:

```typescript
import { AiService } from './ai/ai.service';
```

9. Inyecta el servicio en la clase `ReservationsController`:

```typescript
constructor(
  private readonly reservationsService: ReservationsService,
  private readonly aiService: AiService
) {}
```

10. Crea una nueva ruta POST:

```typescript
@Post('ask')
ask(@Body() askDto: { prompt: string }) {
  return this.aiService.ask(askDto.prompt);
}
```

El método anterior recupera el prompt del cuerpo de la solicitud y lo pasa como parámetro al método `ask` del `aiService`.

Modificaremos la aplicación frontend para usar la nueva ruta como una forma alternativa de crear una nueva reserva:

1. Navega a la carpeta `client` y agrega el siguiente método en el archivo `data.ts`:

```typescript
ask(prompt: string) {
  return this.http.post(this.apiUrl + '/reservations/ask', { prompt });
}
```

El método anterior utiliza la nueva ruta de la API para enviar un prompt de usuario al asistente de IA.

2. Abre el archivo `chapter-title.component.ts` y agrega las siguientes sentencias de importación:

```typescript
import { ButtonModule } from 'primeng/button';
import { PopoverModule } from 'primeng/popover';
```

Agregaremos un botón en el encabezado de la aplicación que abrirá un componente popover de PrimeNG utilizando la clase `PopoverModule`. El popover incluirá un área de texto para describir los detalles de la reserva.

3. Agrega las clases anteriores al array `imports` del decorador del componente:

```typescript
imports: [ToolbarModule, ButtonModule, PopoverModule]
```

4. Agrega el siguiente código HTML debajo del elemento `<span>` de la plantilla del componente:

```html
<ng-template #end>
  <p-button label="AI Assistant" (click)="op.toggle($event)" size="small" />
  <p-popover #op>
    <div class="content">
      <textarea rows="5" cols="30" pTextarea></textarea>
      <p-button label="Submit" />
    </div>
  </p-popover>
</ng-template>
```

En el fragmento anterior, definimos un elemento `<ng-template>` y lo alineamos al final de la barra de herramientas de la aplicación. El elemento contiene un botón pequeño que muestra un componente popover cuando se hace clic en él. El popover contiene un elemento `<textarea>` y un botón Submit que usaremos más adelante para enviar los detalles de la reserva a la API para que el modelo de IA los procese.

5. Agrega una propiedad `styles` en el decorador del componente:

```typescript
styles: `
  .content {
    display: flex;
    flex-direction: column;
    gap: 1rem;
    align-items: end;
  }
`
```

Los estilos CSS anteriores alinean el botón del popover debajo del área de texto.

6. Ejecuta la aplicación frontend usando el comando `ng serve` y navega a `http://localhost:4200`. Deberías ver el siguiente encabezado de aplicación:

> *Figura 6.11 – Encabezado de aplicación con botón de acción*

La figura anterior muestra el encabezado de la aplicación con el botón de acción AI Assistant.

7. Haz clic en el botón y deberías ver la siguiente ventana emergente (*popover*):

> *Figura 6.12 – Asistente de IA*

Modificaremos el componente para que la aplicación envíe el contenido del componente de área de texto a la API para que sea procesado por el modelo de IA:

1. Importa la clase `Data` y el método `inject` en el archivo del componente:

```typescript
import { ChangeDetectionStrategy, Component, input, inject } from '@angular/core';
import { Data } from './data';
```

2. Inyecta el servicio `Data` en la clase `ChapterTitleComponent`:

```typescript
private data = inject(Data);
```

3. Crea el siguiente método que llama al método `ask` del servicio de datos:

```typescript
submit(text: string) {
  this.data.ask(text).subscribe();
}
```

4. Agrega una variable de referencia de plantilla al elemento `<textarea>`:

```html
<textarea rows="5" cols="30" pTextarea #prompt></textarea>
```

5. Pasa el valor de la variable `prompt` al método `submit` cuando el usuario haga clic en el botón:

```html
<p-button label="Submit" (click)="submit(prompt.value)" />
```

Podemos ver cómo usar el asistente de IA en la aplicación Studio BookMaster ejecutando el siguiente flujo de trabajo:

1. Ejecuta las partes frontend y backend de la aplicación.
2. Navega a `http://localhost:4200` con tu navegador.
3. Abre las herramientas de desarrollo del navegador y selecciona la pestaña Network.
4. Haz clic en el botón AI Assistant en el encabezado de la aplicación.
5. Ingresa el siguiente texto dentro del área de texto y haz clic en el botón Submit:

```text
Make a reservation with the following details:
Name: Test user
Email: <A valid email>
Date: 27/1/2026 13:00
Room: 2
```

En el texto anterior, puedes reemplazar los detalles de la reserva con tus propias elecciones. Asegúrate de que la dirección de correo electrónico sea válida para recibir el mensaje de confirmación en tu bandeja de entrada.

6. Busca la solicitud POST en la pestaña Network y revisa su respuesta. Debería ser la siguiente: `message: "Booking Confirmed!"`.
7. Intenta hacer clic en el botón Submit nuevamente y revisa la respuesta de la nueva solicitud HTTP: `message: "I am sorry, room 2 is unavailable at that time"`.

Los usuarios pueden hacer una reserva con un simple prompt, omitiendo los pasos del asistente. La asistencia de IA se extiende más allá del formato de prompt que usamos; se anima a los usuarios a ser creativos y diseñar sus propios prompts.

La tecnología de asistencia de IA proporciona automatizaciones en los sistemas de reservas, eliminando la necesidad de que los desarrolladores diseñen una interfaz de usuario separada para cada tarea. La comprobación de la disponibilidad de las salas, la selección de fechas y el suministro de información del usuario se pueden realizar con una sola interfaz de chat. Por el contrario, una aplicación de back-office para gestionar reservas, donde los usuarios deben completar ciertas tareas, requeriría una interfaz de usuario diseñada específicamente.

---

### Sección: Resumen

En este capítulo, aprendimos cómo construir una aplicación full-stack con capacidades de IA. Exploramos PrimeNG, que es una biblioteca moderna de componentes de interfaz de usuario que muchos desarrolladores utilizan para crear aplicaciones empresariales. Sus componentes nos ayudaron a diseñar una interfaz de usuario hermosa y fácil de usar para reservar salas en un estudio de música.

La aplicación frontend fue respaldada por una API REST construida con NestJS, un framework de backend que muchos desarrolladores de Angular prefieren por su similitud con Angular. NestJS es compatible con muchas bibliotecas, incluidas la configuración de aplicaciones y la comunicación por correo electrónico. También aprendimos cómo usar MongoDB para la interacción con bases de datos y cómo integrar GenKit con NestJS para crear experiencias de IA en el backend de nuestra aplicación.

En el próximo capítulo, exploraremos las características de renderizado del lado del servidor (*Server-Side Rendering* o SSR) en Angular mediante la creación de una aplicación de seguimiento de gastos.

---

### Sección: Ejercicios

Utiliza PrimeNG para mostrar un mensaje emergente (*toast message*) que muestre el resultado del envío del asistente de IA.

---

### Sección: Lecturas Complementarias

- **PrimeNG:** [https://primeng.org](https://primeng.org/)
- **NestJS:** [https://nestjs.com](https://nestjs.com/)
- **MongoDB:** [https://www.mongodb.com](https://www.mongodb.com/)
- **Mongoose:** [https://mongoosejs.com](https://mongoosejs.com/)
- **Nodemailer:** [https://nodemailer.com](https://nodemailer.com/)
- **GenKit:** [https://genkit.dev](https://genkit.dev/)
- **Google AI Studio:** [https://aistudio.google.com](https://aistudio.google.com/)
