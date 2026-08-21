# Parte 1: Primeros Pasos con Angular e Inteligencia Artificial

## Capítulo 10: NotesAI Desktop: Creación de un Editor de Notas Impulsado por IA

### Sección: Introducción

El ecosistema web ha progresado mucho durante los últimos años para cerrar la brecha entre las aplicaciones web y las de escritorio. Los desarrolladores pueden usar Web APIs modernas para interactuar con un sistema de archivos nativo o comunicarse con un modelo de IA en el navegador.

En este capítulo, construiremos una aplicación web para tomar notas. Los usuarios podrán guardar notas en el sistema de archivos local y utilizar la IA para proporcionar un resumen del contenido.

Vamos a cubrir los siguientes temas:

- Instalación de ngx-wig
- Persistencia de datos en el sistema de archivos
- Aumento de contenido con IA

---

### Sección: Requisitos Técnicos

Puedes descargar el proyecto de ejemplo y el código de este libro siguiendo las instrucciones de la sección *Download the example code files* en el Prefacio de este libro.

Los archivos de código de este capítulo están incluidos en el paquete de código descargable.

---

### Sección: Instalación de ngx-wig

La biblioteca `ngx-wig` es un editor WYSIWYG ligero para Angular sin dependencias externas. Contiene una barra de herramientas con botones de comando que los desarrolladores pueden ampliar y personalizar según sus necesidades. La barra de herramientas proporciona los siguientes comandos integrados:

- Lista con viñetas / numerada
- Negrita (*Bold*)
- Cursiva (*Italic*)
- Subrayado (*Underlined*)
- Enlaces URL

Crearemos una nueva aplicación de Angular que mostrará el editor `ngx-wig`:

1. Ejecuta el siguiente comando de Angular CLI para crear una nueva aplicación llamada `ainotes`:

```bash
ng new ainotes
```

Sigue las instrucciones de la sección *Scaffolding a new application* en el Capítulo 1, *Angular AI Kick-Starter: Scaffolding Smart Apps with Copilot*, para completar el proceso de creación de la aplicación.

2. Instala la biblioteca `ngx-wig` dentro de la carpeta `ainotes`:

```bash
npm install ngx-wig
```

3. Abre la carpeta `ainotes` con tu IDE y abre el archivo `app.ts`.
4. Agrega la siguiente sentencia de importación en la parte superior del archivo:

```typescript
import { NgxWigModule } from 'ngx-wig';
```

La clase `NgxWigModule` contiene el componente editor que agregaremos a nuestra aplicación.

5. Agrega la clase anterior al array `imports` del decorador del componente:

```typescript
imports: [NgxWigModule]
```

6. Elimina todas las ocurrencias de la clase `RouterOutlet` porque no usaremos enrutamiento en nuestra aplicación.
7. Abre el archivo `app.html` y reemplaza su contenido con el siguiente código HTML:

```html
<ngx-wig placeholder="What are your thoughts today?" />
```

8. Abre el archivo `styles.scss` y agrega los siguientes estilos:

```scss
html, body {
  margin: 0;
  width: 100%;
  height: 100%;
}

.ng-wig, .nw-editor-container, .nw-editor {
  display: flex;
  flex-direction: column;
  height: 100%;
  overflow: hidden;
}
```

Los estilos CSS anteriores harán que el editor llene toda la ventana del navegador.

9. Ejecuta la aplicación usando el comando `ng serve` y navega a `http://localhost:4200`:

> *Figura 10.1 – Editor ngx-wig*

La imagen anterior muestra el editor ngx-wig con los botones básicos de la barra de herramientas y un marcador de posición (*placeholder*).

La aplicación muestra un editor WYSIWYG con funcionalidad básica. En la siguiente sección, ampliaremos el editor para guardar su contenido en el sistema de archivos local.

---

### Sección: Persistencia de Datos en el Sistema de Archivos

Los navegadores pueden usar la File System API para interactuar con el sistema de archivos nativo. Agregaremos un botón en la barra de herramientas de ngx-wig que obtendrá el contenido del editor y lo guardará localmente utilizando la File System API. Primero, aprenderemos cómo agregar un nuevo botón a la barra de herramientas:

1. Crea un nuevo archivo llamado `save-button.ts` dentro de la carpeta `app`.
2. Abre el archivo y agrega el siguiente contenido:

```typescript
export const save = {
  title: 'Save',
  styleClass: 'nw-button',
  icon: 'icon-save'
};
```

El fragmento anterior define la estructura de un botón de la biblioteca ngx-wig:

- `title`: Muestra el texto como un tooltip cuando pasamos el cursor sobre el botón.
- `styleClass`: Aplica los estilos CSS del botón desde la biblioteca.
- `icon`: Define el icono del botón. El valor de la propiedad `icon` es un estilo CSS que definiremos manualmente en el siguiente paso.

3. Abre el archivo `app.scss` y agrega un estilo CSS para el selector `icon-save` usando la siguiente sintaxis:

```scss
::ng-deep .icon-save {
  background: "A custom image."
}
```

En el estilo anterior, reemplaza el marcador de posición `"A custom image."` con la ruta de un archivo de imagen de tu elección. Alternativamente, obtén el valor correspondiente del repositorio de GitHub. Para acceder al enlace del repositorio, sigue los pasos de la sección "Download the example code files" en el Prefacio.

4. Abre el archivo `app.ts` e importa la variable `save` del archivo `save-button`:

```typescript
import { save } from './save-button';
```

5. Modifica también la sentencia de importación de la biblioteca `ngx-wig` de la siguiente manera:

```typescript
import { BUTTONS, DEFAULT_LIBRARY_BUTTONS, NgxWigModule } from 'ngx-wig';
```

La clase `BUTTONS` es un token de inyección que podemos usar para proporcionar botones a la barra de herramientas. La variable `DEFAULT_LIBRARY_BUTTONS` es el conjunto de botones predeterminado de la barra de herramientas.

6. Usa el array `providers` en el decorador del componente para agregar el botón de guardar a la barra de herramientas:

```typescript
providers: [
  {
    provide: BUTTONS,
    multi: true,
    useValue: {
      ...DEFAULT_LIBRARY_BUTTONS,
      save
    }
  }
]
```

7. Ejecuta la aplicación y verifica que la barra de herramientas contenga el nuevo botón:

> *Figura 10.2 – Barra de herramientas del editor*

La imagen anterior muestra el nuevo botón con un icono de guardar al final de la barra de herramientas.

Invocaremos la File System API al hacer clic en el nuevo botón para guardar el contenido del editor en el sistema de archivos local:

1. Abre el archivo `save-button.ts` y agrega la siguiente sentencia de importación:

```typescript
import { NgxWigComponent } from 'ngx-wig';
```

La clase `NgxWigComponent` es el componente principal de la biblioteca `ngx-wig` que contiene el editor.

2. Agrega una propiedad `command` en el objeto `save` de la siguiente manera:

```typescript
export const save = {
  title: 'Save',
  styleClass: 'nw-button',
  icon: 'icon-save',
  command: async (editor: NgxWigComponent) => {
  }
};
```

La propiedad anterior define la lógica de ejecución al hacer clic en el botón de guardar de la barra de herramientas. La variable `editor` que pasamos como parámetro es la instancia del componente editor actual.

3. Agrega la siguiente línea dentro del cuerpo de la función flecha `command`:

```typescript
const picker = await editor['window'].showSaveFilePicker();
```

La File System API está disponible desde el objeto global `window`. El componente editor expone una instancia del objeto `window` que podemos usar. En el fragmento anterior, llamamos a su método `showSaveFilePicker` para mostrar un cuadro de diálogo de guardado al usuario.

4. Inserta el siguiente fragmento debajo de la línea anterior:

```typescript
const stream = await picker.createWritable();
await stream.write(editor.content());
await stream.close();
```

En el fragmento anterior, usamos un flujo de escritura (*writable stream*) para crear un archivo a partir de la elección del usuario, escribir el contenido del editor y cerrar el archivo.

5. Ejecuta la aplicación usando el comando `ng serve --ssl` y navega a `https://localhost:4200`. La File System API solo es accesible en un contexto de navegador seguro mediante SSL.

Dependiendo de la configuración de seguridad de tu Google Chrome, puede notificarte que la conexión a localhost no es privada porque no estamos usando una clave SSL real:

> *Figura 10.3 – Advertencia de seguridad*

Haz clic en el botón Configuración avanzada (*Advanced*) y sigue las instrucciones para omitir la advertencia y continuar hacia la aplicación.

6. Ingresa algo de texto en el área del editor y haz clic en el botón Save.
7. En el cuadro de diálogo para guardar archivos, elige dónde deseas guardar el archivo, escribe un nombre de archivo y guárdalo.

La aplicación interactuará con el sistema de archivos local y guardará el archivo en la ubicación seleccionada. Ahora tienes un editor sencillo para organizar tus notas personales. Las notas pueden variar desde pequeños textos de recordatorio hasta pensamientos extensos y estructurados. En la siguiente sección, aprenderemos cómo agregar una función de resumen para notas extensas utilizando las capacidades de la API del navegador.

---

### Sección: Aumento de Contenido con IA

La característica principal de nuestra aplicación es que puede usar el sistema de archivos y trabajar sin conexión. Los avances recientes en el ecosistema de los navegadores nos permiten usar la IA en el navegador sin necesidad de un servicio externo para interactuar con los modelos de IA. Los navegadores exponen APIs nativas que los desarrolladores pueden utilizar para comunicarse localmente con un modelo de IA y agregar capacidades agénticas a sus aplicaciones.

Utilizaremos la Summarizer API para pedirle a un modelo de IA que proporcione un resumen rápido de nuestras notas. Agregaremos un nuevo botón en la barra de herramientas para usar la Summarizer API:

1. Crea un nuevo archivo llamado `brief-button.ts` dentro de la carpeta `app` y agrega el siguiente contenido:

```typescript
import { NgxWigComponent } from 'ngx-wig';

export const brief = {
  title: 'Summarize',
  styleClass: 'nw-button',
  icon: 'icon-ai',
  command: async (editor: NgxWigComponent) => {
  }
};
```

2. Abre el archivo `app.ts` e importa la variable `brief`:

```typescript
import { brief } from './brief-button';
```

3. Agrega la variable anterior al array `providers` del decorador del componente de la siguiente manera:

```typescript
providers: [
  {
    provide: BUTTONS,
    multi: true,
    useValue: {
      ...DEFAULT_LIBRARY_BUTTONS,
      save,
      brief
    }
  }
]
```

4. Crea un nuevo estilo para el nuevo icono dentro del archivo `app.scss`, como hicimos en la sección anterior, o usa uno existente del repositorio de GitHub. Para acceder al enlace del repositorio, sigue los pasos de la sección "Download the example code files" en el Prefacio.
5. Ejecuta la aplicación y verifica que el nuevo botón aparezca en la barra de herramientas:

> *Figura 10.4 – Barra de herramientas del editor*

La imagen anterior muestra el nuevo botón junto al botón de guardar con un icono personalizado.

El nuevo botón aún no contiene ninguna lógica. Agregaremos código en su propiedad `command` para interactuar con la Summarizer API:

1. Abre el archivo `brief-button.ts` y agrega la siguiente sentencia dentro del cuerpo de la función flecha `command`:

```typescript
const original = editor.content();
```

Mostraremos el resumen encima del contenido original, por lo que debemos mantener las notas iniciales en la memoria.

2. Crea una nueva instancia de la Summarizer API agregando la siguiente línea:

```typescript
const briefer = await editor['window'].Summarizer.create();
```

3. Pasa el siguiente código como parámetro al método `create`:

```typescript
{
  monitor: (m: any) => {
    m.addEventListener('downloadprogress', (e: any) => {
      const progress = Math.floor(e.loaded * 100);
      editor.writeValue(
        `Thinking...${progress}%<hr>${original}`
      );
    });
  }
}
```

La Summarizer API requiere descargar un modelo de IA una vez al principio. Algunos modelos son bastante grandes y tardan un tiempo en descargarse.

La Summarizer API proporciona la propiedad `monitor` que usamos para informar el progreso en nuestra aplicación. En el fragmento anterior, mostramos el progreso encima del contenido del editor separado por una línea horizontal.

> *Figura 10.5 – Progreso de descarga*

La imagen anterior muestra el progreso de la descarga encima del contenido original.

4. Agrega las siguientes líneas debajo de la creación del resumidor:

```typescript
const brief = await briefer.summarize(editor.content());
editor.writeValue(brief + '<hr>' + original);
```

El código anterior pasa el contenido del editor al resumidor y espera hasta que haya una respuesta de la API. La aplicación finalmente muestra el resumen encima del contenido original, como lo hizo con el progreso de la descarga.

> *Figura 10.6 – Resumen del contenido*

La imagen anterior muestra el resumen sobre el contenido original, separado por una línea horizontal. Los usuarios pueden pegar un texto grande dentro del editor y obtener un resumen rápido sin tener que revisar todo el contenido.

---

### Sección: Resumen

En este capítulo, exploramos cómo usar las APIs nativas de la web y construir una aplicación para tomar notas sin conexión, como una de escritorio. Usamos la biblioteca `ngx-wig` para diseñar el área principal del editor y aprendimos cómo usar la File System API para guardar su contenido en el sistema de archivos local. También usamos la Summarizer API para interactuar con un modelo de IA en el navegador y resumir texto.

Este capítulo no utiliza la Summarizer API al azar. Resume un viaje increíble de este libro a través de Angular y muchas herramientas y bibliotecas modernas. El ecosistema de Angular es enorme, y hay mucho más para que explores en la comunidad. ¡Usa tu creatividad y Angular para crear aplicaciones excelentes y escalables!

---

### Sección: Ejercicios

Agrega un nuevo botón en la barra de herramientas para seleccionar un archivo específico del sistema de archivos local y cargar su contenido en el editor.

---

### Sección: Lecturas Complementarias

- **ngx-wig:** [https://github.com/stevermeister/ngx-wig](https://github.com/stevermeister/ngx-wig)
- **File System API:** [https://developer.mozilla.org/docs/Web/API/File_System_API](https://developer.mozilla.org/docs/Web/API/File_System_API)
- **Summarizer API:** [https://developer.mozilla.org/docs/Web/API/Summarizer_API](https://developer.mozilla.org/docs/Web/API/Summarizer_API)
