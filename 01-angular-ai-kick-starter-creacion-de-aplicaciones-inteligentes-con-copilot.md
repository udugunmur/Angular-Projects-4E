# Parte 1: Primeros Pasos con Angular e Inteligencia Artificial

## Capítulo 1: Angular AI Kick-Starter: Creación de Aplicaciones Inteligentes con Copilot

### Sección: Introducción

La herramienta más popular para crear y compilar aplicaciones de Angular es la interfaz de línea de comandos de Angular (CLI), que automatiza numerosas tareas de desarrollo, incluyendo la estructura inicial (*scaffolding*), las pruebas y el despliegue de aplicaciones Angular. Los desarrolladores web utilizan herramientas de inteligencia artificial (IA) para la codificación asistida con el fin de completar tareas como la aplicación de mejores prácticas y la búsqueda de documentación.

Al final de este capítulo, podrás utilizar asistentes de codificación de IA en un proyecto de Angular para mejorar tu experiencia como desarrollador y hacer que tu flujo de trabajo en Angular sea más eficiente. Aprenderás a utilizar GitHub Copilot para construir una aplicación Angular utilizando las características más recientes y las prácticas modernas del framework Angular.

Vamos a cubrir los siguientes temas:

- Creación de una aplicación Angular
- Herramientas de Angular en VS Code
- Interacción con GitHub Copilot

> [!NOTE]
> **Su compra incluye una copia gratuita en PDF + paquete de código**
> 
> Su compra incluye una copia en PDF sin DRM de este libro, el paquete de código y extras exclusivos adicionales. Consulte la sección de beneficios gratuitos con su libro en el Prefacio para desbloquearlos al instante y maximizar su aprendizaje.

---

### Sección: Requisitos Técnicos

Puedes descargar el proyecto de ejemplo y el código de este libro siguiendo las instrucciones de la sección *Download the example code files* en el Prefacio de este libro.

Los archivos de código de este capítulo están incluidos en el paquete de código descargable.

---

### Sección: Creación de una Aplicación Angular

Cada capítulo del libro requiere configurar una nueva aplicación Angular. La herramienta estándar para crear una aplicación Angular es la Angular CLI. Es un paquete de npm que proporciona una herramienta de línea de comandos para generar la estructura y ejecutar una aplicación Angular. En la siguiente sección, aprenderemos cómo instalarla.

#### Instalación de la Angular CLI

Abre una ventana de terminal y ejecuta el siguiente comando para instalar la Angular CLI globalmente en tu máquina:

```bash
npm install -g @angular/cli
```

El comando anterior instalará la última versión de la Angular CLI, publicada en el registro de npm por el equipo de Angular.

En este libro, utilizamos el gestor de paquetes npm debido a su adopción generalizada en la comunidad de desarrollo web. Si estás utilizando un gestor de paquetes diferente, visita [https://angular.dev/tools/cli/setup-local#install-the-angular-cli](https://angular.dev/tools/cli/setup-local#install-the-angular-cli) para obtener más información sobre cómo instalarlo.

Para verificar que has instalado la versión 22, ejecuta el siguiente comando en la misma ventana de terminal:

```bash
ng version
```

La salida del comando anterior debería indicar que la Angular CLI instalada es la versión 22.

Los ejemplos demostrados en este libro han sido desarrollados y probados con la versión 22 de la Angular CLI.

Si tienes una versión diferente instalada, ejecuta el siguiente comando para instalar la correcta:

```bash
npm install -g @angular/cli@22
```

Tu entorno local ya está configurado con la Angular CLI y estás listo para comenzar a construir aplicaciones Angular. En la siguiente sección, utilizarás la Angular CLI para crear una nueva aplicación.

#### Creación de la estructura de una nueva aplicación

Utilizamos la Angular CLI ejecutando el ejecutable `ng` desde una ventana de terminal. Acepta una opción como parámetro que define el tipo de tarea que queremos completar mediante la Angular CLI.

Puedes ver una lista completa de las opciones admitidas en [https://angular.dev/cli](https://angular.dev/cli).

La opción `new` indica que queremos crear la estructura de una nueva aplicación Angular:

Ejecuta el siguiente comando en una ventana de terminal:

```bash
ng new my-app
```

El comando anterior iniciará un proceso que te guiará en la creación de una nueva aplicación Angular llamada `my-app`. El proceso implica hacer preguntas para proporcionar más contexto en la CLI sobre la aplicación que deseas construir.

La primera pregunta es sobre qué formato de hoja de estilos nos gustaría usar en nuestra aplicación. La Angular CLI admite los siguientes formatos:

- CSS
- Tailwind CSS
- Sass
- Less

Usa las flechas del teclado para resaltar la opción **Sass (SCSS)** y presiona Enter.

La siguiente pregunta es si queremos configurar la aplicación para el renderizado del lado del servidor (SSR). Presiona Enter para omitir este paso. Construiremos una aplicación habilitada para SSR en el Capítulo 7, *Expense Builder: Building an SSR-Optimized Expense Tracker*.

La última pregunta nos solicita agregar un asistente de codificación de IA al proyecto de Angular. La Angular CLI admite una amplia colección de herramientas de IA. Selecciona la opción **GitHub Copilot** y presiona Enter.

La Angular CLI descarga e instala todos los paquetes necesarios y crea los archivos predeterminados para tu aplicación Angular. Al finalizar, la carpeta `my-app` contendrá una aplicación básica de Angular.

Ejecuta el siguiente comando desde una ventana de terminal dentro de la carpeta `my-app` para ejecutar tu aplicación:

```bash
ng serve
```

La Angular CLI compila la aplicación Angular e inicia un servidor web para alojarla. Tras una compilación exitosa, puedes previsualizar la aplicación abriendo tu navegador y navegando a `http://localhost:4200`.

Ahora tenemos una aplicación Angular básica que servirá como base para los proyectos que construiremos en los siguientes capítulos. En la siguiente sección, aprenderemos sobre las herramientas necesarias para desarrollar nuestros proyectos de Angular.

---

### Sección: Herramientas de Angular en VS Code

Una de las herramientas más populares para desarrollar con Angular es Visual Studio Code (VS Code). Contiene una rica colección de extensiones que podemos instalar para mejorar la experiencia de desarrollo con el framework Angular. Podemos buscar e instalar extensiones de Angular desde el menú Extensions en la barra de herramientas de VS Code. En esta sección, utilizaremos Profiles para instalar una colección seleccionada de extensiones de Angular.

Añadimos perfiles desde el menú Manage:

1. Haz clic en el botón con el icono de engranaje para acceder al menú Manage.
2. Selecciona la opción **Profiles** en el menú emergente.
3. Haz clic en la flecha hacia abajo en el botón **New Profile** y selecciona la opción **From Template | Angular**. VS Code agrega un nuevo perfil que contiene configuraciones y extensiones útiles para el desarrollo con Angular.
4. Haz clic en el botón **Create** para iniciar la instalación del perfil de Angular. Dependiendo de tu configuración de VS Code, es posible que se te solicite permitir la instalación de extensiones de terceros.

El proceso instalará extensiones y configurará VS Code de acuerdo con el perfil de Angular. El perfil se puede habilitar de las siguientes maneras:

- **Automáticamente:** Marca la opción *Use this profile as the default for new windows* para habilitarlo para todas las ventanas de VS Code de forma predeterminada.
- **Manualmente:** Usa el menú Manage para cambiar al perfil de Angular.

Se recomienda utilizar el enfoque manual si utilizas VS Code para desarrollar con otras pilas tecnológicas además de Angular.

VS Code activa las extensiones instaladas cuando habilitas el perfil de Angular. Los proyectos que construiremos en los siguientes capítulos no las utilizan todas. Las más comunes son las siguientes:

- **Angular Language Service:** Desarrollado y mantenido por el equipo de Angular, ofrece características como autocompletado de código, navegación y diagnóstico efectivo de errores dentro de las plantillas y clases de Angular.
- **Angular Schematics:** Proporciona una forma de crear diferentes partes de una aplicación Angular a través de una interfaz de usuario intuitiva.
- **EditorConfig for VS Code:** Puede anular configuraciones de VS Code como la sangría y el espaciado mediante un archivo de configuración local.
- **Material Icon Theme:** Extiende los iconos predeterminados de VS Code añadiendo nuevos iconos basados en Material Design Icons y el framework Angular.

VS Code utiliza las extensiones restantes del perfil de Angular según las necesidades específicas de una aplicación Angular. Por ejemplo, la extensión Jest es útil si utilizas el ejecutor de pruebas Jest para pruebas unitarias. Del mismo modo, la extensión ESLint proporciona capacidades de análisis estático (linting) a una aplicación cuando la biblioteca ESLint está instalada.

Ahora disponemos de una configuración básica de VS Code que nos ayudará a desarrollar proyectos de Angular con confianza. En la siguiente sección, aprenderemos cómo aplicar las mejores prácticas en el desarrollo de Angular utilizando tecnología asistida por IA.

---

### Sección: Interacción con GitHub Copilot

La versión más reciente de VS Code incluye una integración nativa de GitHub Copilot. Aprenderemos a utilizar Copilot como un programador en pareja de IA y a desarrollar aplicaciones Angular de las siguientes maneras:

- **Más inteligente:** Proporcionando instrucciones teniendo en cuenta las mejores prácticas de Angular.
- **Más rápido:** Construyendo diferentes partes de la aplicación Angular mediante *prompts*.

En las siguientes secciones, exploraremos ambos conceptos a medida que continuamos desarrollando nuestra aplicación Angular.

#### Personalización de instrucciones

Los agentes de IA utilizan instrucciones para definir un contexto de trabajo y operar dentro de límites específicos. Las instrucciones definen el rol y las capacidades del agente en el contexto actual. La Angular CLI creó un archivo de instrucciones para Copilot al crear la aplicación:

1. Abre VS Code y selecciona **File | Open Folder…** en el menú principal.
2. Navega hasta la carpeta `my-app` y selecciónala. VS Code cargará el proyecto asociado de la Angular CLI.
3. Expande la carpeta `.github` y selecciona el archivo `copilot-instructions.md`. El archivo Markdown contiene instrucciones que ayudan a Copilot a entender Angular.

La parte superior del archivo define el rol del agente de IA:

```markdown
You are an expert in TypeScript, Angular, and scalable web application development. You write functional, maintainable, performant, and accessible code following Angular and TypeScript best practices.
```

Cada encabezado del archivo representa una capacidad diferente del agente. Por ejemplo, el agente debe adherirse a las siguientes mejores prácticas de TypeScript:

```markdown
## TypeScript Best Practices
- Use strict type checking
- Prefer type inference when the type is obvious
- Avoid the `any` type; use `unknown` when type is uncertain
```

Podemos personalizar el archivo de instrucciones según nuestras necesidades. Agrega lo siguiente en el encabezado State Management:

```markdown
- Define signal properties as `readonly`
```

La instrucción anterior guiará a Copilot para declarar signals con la palabra clave `readonly`.

El formato Markdown del archivo de instrucciones lo hace accesible para todos los desarrolladores. Ahora comprendes lo fácil que es extenderlo y compartirlo con otros miembros de tu equipo de desarrollo. En la siguiente sección, aprenderemos cómo Copilot utiliza las instrucciones para interactuar con nuestra aplicación Angular.

#### Desarrollo mediante prompts

GitHub Copilot está preinstalado y habilitado de forma predeterminada en la versión más reciente de VS Code. Se abre automáticamente en el panel lateral **CHAT** cuando cargamos una aplicación Angular.

Si el panel no está abierto, puedes usar el botón **Toggle Chat** en la barra superior cerca del campo de búsqueda.

El panel CHAT presenta una pantalla de bienvenida y un cuadro de entrada que nos permite interactuar con Copilot mediante prompts:

> *Figura 1.1 – GitHub Copilot*

Primero, utilizaremos Copilot para refactorizar la base de código existente de la aplicación Angular.

Los agentes de IA no son deterministas, lo que significa que sus respuestas pueden variar en cada ejecución. Los pasos que seguirás en esta sección pueden dar como resultado una salida diferente a la descrita en la sección.

Crearemos una nueva propiedad en la clase del componente `App` para los títulos de los capítulos:

Introduce la siguiente frase en el cuadro de entrada de Copilot y haz clic en el botón **Send**:

```text
Create a signal in the App component for storing the chapter title with the following value: [Chapter 1](https://subscription.packtpub.com/book/programming/9781806668472/1): Angular AI Kick-Starter.
```

Debes iniciar sesión en Copilot antes de comenzar a interactuar con él. Selecciona un método de inicio de sesión en el siguiente cuadro de diálogo:

> *Figura 1.2 – Iniciar sesión para usar GitHub Copilot*

Una vez completado el proceso de inicio de sesión, Copilot crea una propiedad `chapterTitle` en la clase del componente `App`.

Copilot continúa nuestro prompt con una sugerencia para actualizar el título del capítulo en la plantilla del componente. Introduce el siguiente prompt:

```text
Update the title in the template according to the chapter title.
```

Copilot cambia la línea en el archivo de plantilla que mostraba la propiedad `title` por lo siguiente:

```html
<h1>{{ chapterTitle() }}</h1>
```

Ejecuta la aplicación usando el comando `ng serve` y navega a `localhost:4200`:

> *Figura 1.3 – Salida de la aplicación*

Has aprendido a refactorizar una aplicación Angular existente sin necesidad de interactuar directamente con el código. Copilot hizo todo el trabajo requerido mediante prompts.

Podemos utilizar Copilot para tareas más complejas, como la creación de nuevos componentes y servicios:

1. Haz clic en el botón más ubicado en el encabezado del panel CHAT para crear una nueva sesión de Copilot.
2. Introduce el siguiente prompt en el cuadro de entrada:

```text
Create a component responsible for displaying the chapter title and use it in the App component.
```

El prompt anterior creará un nuevo componente basado en las mejores prácticas de Components del archivo de instrucciones.

El nuevo componente pasa el valor de la propiedad `chapterTitle` utilizando una signal de entrada (*input signal*). Examina el navegador para verificar que la salida de la aplicación sigue siendo la misma.

Ahora entiendes cuánto más rápido es desarrollar aplicaciones Angular con Copilot. Aprendimos cómo crear un componente sin utilizar la Angular CLI. Puedes utilizar el mismo proceso para generar diferentes partes de una aplicación Angular, como servicios y directivas.

---

### Sección: Resumen

En este capítulo, aprendimos cómo construir una aplicación Angular con tecnología asistida por IA. Iniciamos el proyecto instalando la Angular CLI y creando la estructura de una nueva aplicación. Creamos un perfil de VS Code para aprovechar las extensiones de Angular durante el desarrollo. Utilizamos el chat integrado para que GitHub Copilot nos asista al escribir código nuevo o refactorizar código existente.

Ahora disponemos de todas las herramientas necesarias para construir nuestra primera aplicación del mundo real en el próximo capítulo, en el cual desarrollaremos un sistema de seguimiento para gestionar y reportar incidencias.

---

### Sección: Ejercicios

Utiliza GitHub Copilot para crear un servicio de Angular que contenga el título del capítulo actual. Modifica el componente `App` para que utilice el servicio y pase el título del capítulo al componente respectivo.

Intenta experimentar con diferentes prompts para ver cómo varían los resultados en cada uno.

---

### Sección: Lecturas Complementarias

- **Angular CLI:** [https://angular.dev/tools/cli](https://angular.dev/tools/cli)
- **Profiles:** [https://code.visualstudio.com/docs/configure/profiles](https://code.visualstudio.com/docs/configure/profiles)
- **Angular AI prompts:** [https://angular.dev/ai/develop-with-ai](https://angular.dev/ai/develop-with-ai)
- **GitHub Copilot:** [https://github.com/features/copilot](https://github.com/features/copilot)
