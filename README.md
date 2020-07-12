# Tutorial Amplify (backend) + Vue (frontend). ToDo fullstack app con autentificación
### ¿Qué desarrollarás con este tutorial?
Este tutorial, te guiará para establecer un backend e integrarlo en tu web app. Crearás una app de "ToDo" con la API de GraphQL para almacenar y recibir los items en la base de datos en la nube, también recivir actualizaciones de estos items en tiempo real.

En el siguiente enlace se elige el framework con el que se va a realizar el proyecto de ejemplo. (Mejor si se elige VUE).

https://docs.amplify.aws/start

Una vez seleccionado el framework pasamos a "Start the Tutorial 🚀".

### Requisitos previos
Antes de empezar, necesitamos comprobar que tenemos las siguientes erramientas instaladas en nuestro sistema:
>- [Node.js v10.x o superior](https://nodejs.org/es/).
>- [NPM V5.x o superior](https://www.npmjs.com/get-npm).
>- [git v2.14.1 o superior](https://git-scm.com/downloads).
>>Desde la terminal ejecutar los siguientes comandos para comprobar:
>>- `node -v`
>>- `npm -v`
>>- `git --version`

Una vez hechas la comprobaciones, necesitamos registrarnos en AWS siguiendo el siguiente enlace https://go.aws/3etk3Sw 

### Instalar y configurar el CLI de Amplify
La *Amplify Command Line Interface* __(CLI)__ es una herramienta para crear servicios en la nube de AWS para nuestras APP. 

#### Intrucciones
Abrir una terminal (ya sea en VisualStudio Code o en otra).

Ejecutar los siguientes comandos (en sudo si se trabaja en linux):
- `npm install -g @aws-amplify/cli` (Instalar la CLI)
- `amplify configure` (Configurar Amplify, te pedirá que inicies sesión en AWS Console)
   - Una vez iniciada la sesión, Amplify CLI te pedirá crear un usuario IAM (Identity and Access Management).

   -  Seleccionar los campos indicados:
      ~~~ 
      Specify the AWS Region  
      ? region:  eu-west-1      
      Specify the username of the new IAM user:      
      ? user name:  amplify-user
      Complete the user creation using the AWS console 
      ~~~
      
Crea un usuario con **AdministratorAccess** en tu cuenta para gestionar los recursos de AWS como AppSync, Cognito...

![Ejemplo GIF](https://docs.amplify.aws/images/user-creation.gif)

Una vez tenemos el usuario creado, Amplify CLI preguntará una **accessKeyId** y **secretAccessKey** para conectar con Amplify CLI con el usuario IAM creado anteriormente.

~~~
Enter the access key of the newly created user:
? accessKeyId:  # TU_ACCESS_KEY_ID
? secretAccessKey:  # TU_SECRET_ACCESS_KEY
This would update/create the AWS Profile in your local machine
? Profile Name:  # (default)

Successfully set up the new user.
~~~ 
 
### Establecer el proyecto fullstack
### Creación de nuestra app de VUE

Usaremos VUE CLI para crear nuestro proyecto.
En la terminal:
~~~
npm install -g @vue/cli
vue create myamplifyproject
cd myamplifyproject
~~~

Ejecutamos nuestra app: 
~~~
npm install
npm run serve
~~~

### Iniciamos nuestro nuevo backend
Ahora que tenemos una app de VUE ejecutada, es el momento para usar Amplify para poder crear los servicios de backend necesarios para nuestra app. Desde la raíz de nuestro proyecto (e.j Desktop/myamplifyproject, ejecutamos: 

`amplify init`

Cuando inicializas Amplify, tendremos que indicar cierta información sobre nuestra app:

~~~
Enter a name for the project (todo)

# All AWS services you provision for your app are grouped into an "environment"
# A common naming convention is dev, staging, and production
Enter a name for the environment (dev)

# Sometimes the CLI will prompt you to edit a file, it will use this editor to open those files.
Choose your default editor

# Amplify supports JavaScript (Web & React Native), iOS, and Android apps
Choose the type of app that you're building (javascript)

What JavaScript framework are you using (vue)

Source directory path (src)

Distribution directory path (dist)

Build command (npm run-script build)

Start command (npm run-script serve)

# This is the profile you created with the `amplify configure` command in the introduction step.
Do you want to use an AWS profile
~~~

Cuando inicializas un proyecto de Amplify, ocurre todo esto:
- Crea un un directorio llamado **amplify** que guarda la definición de nuestro backend.
- Crea un archivo llamado **aws-exports.js** en el directorio /src, que contiene la configuración para los servicios que creas con Amplify. Así es como el cliente de Amplify es capaz de obtener información sobre tus servicios de backend.
- Modifica el fichero .gitignore, añadiendo archivos generados en la lista para ignorar.
- Se crea un proyecto en la nube automáticamente en la consola de AWS Amplify, la cual se puede acceder ejecutando `amplify console`. La consola proporciona una lista de entornos de backend, *deep links* a recursos aprovisionados por categoría de Amplify, estado de implementaciones recientes e instrucciones sobre cómo promover, clonar, extraer y eliminar recursos de backend.

### Instalación de las librerias de Amplify

El primer paso para usar Amplify en el cliente, es instalar las dependencias necesarias:

`npm install aws-amplify @aws-amplify/ui-vue`

El paquete `@aws-amplify/ui-vue` es un set componentes UI específicos de VUE que hace fácil la integración de funcionalidades, como por ejemplo la autentificación.

### Preparación del frontend

A continuación, necesitamos configurar Amplify en el cliente, para poder interactuar con nuestros servicios de backend.

Abrimos **src/main.js** y le añadimos el siguiente código justo debajo del último import:
~~~
import Amplify from 'aws-amplify';
import '@aws-amplify/ui-vue';
import aws_exports from './aws-exports';

Amplify.configure(aws_exports);
~~~ 

Amplify ya está configurado. Si se realizan cambios usando Amplify CLI, la configuración de **aws-exports.js** se actualizará automáticamente. 

## Conectar la API y la base de datos a nuestra APP

Para este proyecto, necesitamos: 
- Una lista de ToDos
- La habilidad de crear/actualizar/eliminar un ToDo

### Modelar los datos con GraphQL Tranform

Teniendo en cuenta estos requisitos, necesitaremos poder consultar la API para obtener una lista de ToDos y también proporcionar una forma de enviar actualizaciones a la API para crear, actualizar y eliminar ToDos. En GraphQL usamos `type` para definir esa entidad, así:

~~~
type Todo {
  id: ID!
  name: String!
  description: String!
}
~~~

Debido a que estamos usando Amplify, podemos usar el lenguaje de definición de esquema GraphQL (SDL) y las directivas de Amplify personalizadas para definir nuestros requisitos de backend para nuestra API. Luego, la biblioteca GraphQL Transform convierte nuestra definición SDL en un conjunto de plantillas AWS CloudFormation completamente descriptivas que implementan nuestro modelo de datos.

~~~
type Todo @model {
  id: ID!
  name: String!
  description: String!
}
~~~

La directiva `@model` le permite a Amplify saber que pretendemos que este `type` tenga datos que deben almacenarse. Esto creará una tabla DynamoDB para nosotros y hará que todas las operaciones GraphQL estén disponibles en la API.

## Creación de GraphQL API y base de datos

Ahora que los datos están moldeados, es la hora de crear GraphQL API. Desde la raíz del proyecto, ejecutamos lo siguiente:

`amplify add api`

Seleccionamos los valores por defecto: 

~~~
? Please select from one of the below mentioned services:
# GraphQL
? Provide API name:
# myapi
? Choose the default authorization type for the API:
# API Key
? Enter a description for the API key:
# demo
? After how many days from now the API key should expire:
# 7 (or your preferred expiration)
? Do you want to configure advanced settings for the GraphQL API:
# No
? Do you have an annotated GraphQL schema? 
# No
? Do you want a guided schema creation? 
# Yes
? What best describes your project: 
# Single object with fields
? Do you want to edit the schema now? 
# Yes
~~~ 

La CLI debería abrir este esquema de GraphQL en nuestro editor (si no es el caso, lo abrimos manualmente).

**amplify/backend/api/myapi/schema.graphql**

~~~
type Todo @model {
  id: ID!
  name: String!
  description: String
}
~~~

El esquema que hemos generado, es para una ToDo app.

La biblioteca de transformación GraphQL proporciona directivas personalizadas que podemos usar en nuestro esquema que nos permiten definir modelos de datos, configurar reglas de autenticación y autorización, configurar funciones sin servidor como solucionadores y más.

Desde la línea de comando, presionamos Eneter para aceptar el esquema y continuar con los siguientes pasos.

## Deploy de nuestra GraphQL API

Ahora que la API se ha creado, necesitamos hacer push de nuestra configuración actualizada en la nube para que podamos hacer deploy de nuestra API: 

`amplify push`

## Generación del código de frontend para GraphQL API

Cuando ejecutamos `amplify push`, tendrá la opción de tener todas las operaciones GraphQL encontradas en su esquema generadas automáticamente en código. Elegimos las siguientes opciones:

~~~
Do you want to generate code for your newly created GraphQL API (Yes)
Choose the code generation language target (javascript)
Enter the file name pattern of graphql queries, mutations and subscriptions (src/graphql/**/*.js)
Do you want to generate/update all possible GraphQL operations - queries, mutations and subscriptions (Yes)
Enter maximum statement depth [increase from default if your schema is deeply nested] (2)
~~~

A continuación, ejecutamos el siguiente comando para comprobar el status de Amplify:

`amplify status`

Esto nos dará el estado actual del proyecto Amplify, incluido el entorno actual, cualquier categoría que se haya creado y en qué estado se encuentran esas categorías. Debería ser similar a esto:

~~~
Current Environment: dev

| Category | Resource name | Operation | Provider plugin   |
| -------- | ------------- | --------- | ----------------- |
| Api      | myapi         | No Change | awscloudformation |
~~~

### Probamos nuestra API

Para ver los servicios implementados en nuestro proyecto en cualquier momento en la consola de AWS Amplify, ejecutamos el siguiente comando.

`amplify console`

Esto abrirá nuestro proyecto Amplify en la consola de servicio de AWS. Elegimos la pestaña API para ver su API AppSync. Al hacer clic en la API, se abrirá la consola de AWS AppSync, donde puede ejecutar consultas, mutaciones o suscripciones en el servidor y ver los cambios en su aplicación cliente.

## Connectar frontend con API

Abrimos **App.vue**.

### Escribiendo datos con mutaciones GraphQL

Para crear un nuevo ToDo en la base de datos, usamos la operación `API.graphql()` con la mutación `createTodo` y pasamos los datos a escribir.

~~~
<template>
  <div id="app">
    <h1>Todo App</h1>
    <input type="text" v-model="name" placeholder="Todo name">
    <input type="text" v-model="description" placeholder="Todo description">
    <button v-on:click="createTodo">Create Todo</button>
  </div>
</template>

<script>
import { API } from 'aws-amplify';
import { createTodo } from './graphql/mutations';

export default {
  name: 'app',
  data() {
    return {
      name: '',
      description: ''
    }
  },
  methods: {
    async createTodo() {
      const { name, description } = this;
      if (!name || !description) return;
      const todo = { name, description };
      await API.graphql({
        query: createTodo,
        variables: {input: todo},
      });
      this.name = '';
      this.description = '';
    }
  }
};
</script>
~~~

### Leyendo datos con las consultas de GraphQL

Para mostrar los datos, actualizamos `App.vue` para enumerar todos los elementos de la base de datos importando `listTodos` y luego utilizando el método de VUE `created()` para actualizar la página cuando se ejecuta una consulta al cargar la página:

~~~
<template>
  <div id="app">
    <h1>Todo App</h1>
    <input type="text" v-model="name" placeholder="Todo name">
    <input type="text" v-model="description" placeholder="Todo description">
    <button v-on:click="createTodo">Create Todo</button>
    <div v-for="item in todos" :key="item.id">
      <h3>{{ item.name }}</h3>
      <p>{{ item.description }}</p>
    </div>
  </div>
</template>

<script>
import { API } from 'aws-amplify';
import { createTodo } from './graphql/mutations';
import { listTodos } from './graphql/queries';

export default {
  name: 'App',
  async created() {
    this.getTodos();
  },
  data() {
    return {
      name: '',
      description: '',
      todos: []
    }
  },
  methods: {
    async createTodo() {
      const { name, description } = this;
      if (!name || !description) return;
      const todo = { name, description };
      this.todos = [...this.todos, todo];
      await API.graphql({
        query: createTodo,
        variables: {input: todo},
      });
      this.name = '';
      this.description = '';
    },
    async getTodos() {
      const todos = await API.graphql({
        query: listTodos
      });
      this.todos = todos.data.listTodos.items;
    }
  }
}
</script>
~~~

### Datos en tiempo real con suscripciones de GraphQL 

Ahora, si deseamos suscribir datos, importamos la suscripción `onCreateTodo` y creamos una nueva suscripción agregando una nueva suscripción con API.graphql() de esta manera:

~~~
<script>
// other imports
import { onCreateTodo } from './graphql/subscriptions';

export default {
  // other functions and properties
  created(){
    this.getTodos();
    this.subscribe();
  },
  methods: {
    // other methods
    subscribe() {
      API.graphql({ query: onCreateTodo })
        .subscribe({
          next: (eventData) => {
            let todo = eventData.value.data.onCreateTodo;
            if (this.todos.some(item => item.name === todo.name)) return; // remove duplications
            this.todos = [...this.todos, todo];
          }
        });
    }
  }
};
</script>
~~~

## Añadimos autentificación

A continuación, añadiremos autentificación a nuestra app.

### Autentificación con Amplify

Amplify Framework utiliza Amazon Cognito como el principal proveedor de autenticación. Amazon Cognito es un sólido servicio de directorio de usuarios que maneja el registro de usuarios, la autenticación, la recuperación de cuentas y otras operaciones. En este tutorial, aprenderemos cómo agregar autenticación a nustra aplicación utilizando Amazon Cognito y el inicio de sesión con nombre de usuario/contraseña.

### Creamos el servicio de auntentificación

`amplify add auth´

~~~
? Do you want to use the default authentication and security configuration? Default configuration
? How do you want users to be able to sign in? Username
? Do you want to configure advanced settings?  No, I am done.
~~~

Para hacer deploy del servicio: 

`amplify push`

`? Are you sure you want to continue? Y`

Ahora, el servicio de autenticación se ha implementado y podemos comenzar a usarlo. Para ver los servicios desplegados en nuestro proyecto en cualquier momento, vamos a Amplify Console ejecutando el siguiente comando:

`amplify console`

### Creamos el UI del login

Ahora que tenemos nuestro servicio de autenticación implementado en AWS, es hora de agregar autenticación a nuestra aplicación Vue. Crear el flujo de inicio de sesión puede ser bastante difícil y lento para hacerlo bien. Afortunadamente, Amplify Framework tiene un componente de autenticación de UI que podemos usar que nos proporcionará todo el flujo de autenticación, utilizando nuestra configuración especificada en nuestro archivo aws-exports.js.

Abrimos **src/App.vue** y hacemos los siguientes cambios.

~~~
<template>
  <div id="app">
    <amplify-authenticator>
    <h1>Todo App</h1>
    <input type="text" v-model="name" placeholder="Todo name">
    <input type="text" v-model="description" placeholder="Todo description">
    <button v-on:click="createTodo">Create Todo</button>
    <div v-for="item in todos" :key="item.id">
      <h3>{{ item.name }}</h3>
      <p>{{ item.description }}</p>
    </div>
    <amplify-sign-out></amplify-sign-out>
  </amplify-authenticator>
  </div>
</template>
~~~

Ejecutamos la app, para ver nuestro servicio de autentificación trabajando: 

`npm run serve`

Ahora deberíamos ver que la aplicación se carga con autenticación que permite a los usuarios registrarse e iniciar sesión.

En este ejemplo, hemos usado la biblioteca Amplify VueUI y el componente amplify-authenticator para comenzar a trabajar rápidamente con un flujo de autenticación del mundo real.

También podemos personalizar este componente para agregar o eliminar campos, actualizar estilos u otras configuraciones.

Además del `amplify-authenticator`, puede crear flujos de autenticación personalizados utilizando la clase `Auth`.

`Auth` tiene más de 30 métodos, incluidos signUp, signIn, forgetPassword y signOut que nos permiten un control total sobre todos los aspectos del flujo de autenticación del usuario.

En la siguiente sección, alojaremos nuestra aplicación en Amplify Console, un servicio de alojamiento completo con un CDN disponible a nivel mundial, implementaciones atómicas, dominios personalizados fáciles y CI/CD.

## Deploy y hostear nuestra app

Ahora que tenemos nuestra app terminada, es hora de hacerla disponible en la web.

### Añadir hosting a nuestra app

Podemos implementar manualmente nuestra aplicación web o configurar la implementación continua automática. En esta guía cubriremos cómo implementar y alojar manualmente nuestra aplicación web estática para compartirla rápidamente con otros.

Desde la raíz de nuestro proyecto, ejecutamos el siguiente comando: 

`amplify add hosting`

~~~
? Select the plugin module to execute: # Hosting with Amplify Console (Managed hosting with custom domains, Continuous deployment)
? Choose a type: # Manual Deployment
~~~

### Publicamos nuestra app

Para publicar nuestra app, ejecutamos el siguiente comando: 

`amplify publish`

Después de publicar, la terminal mostrará la URL de nuestra aplicación alojada en un dominio `amplifyapp.com`. Siempre que tenga cambios adicionales para publicar, simplemente vuelvemos a ejecutar el comando `amplify publish`.

Si obtenemos un error "AccessDenied" dentro de un documento XML, tenemos que asegurarnos de que `DistributionDir` esté configurado en el directorio correcto en `amplify/.config/project-config.json` y luego vuelvemos a ejecutar `amplify publish`.

Para ver nuestra aplicación y la configuración de hosting en la Consola Amplify, ejecutamos: 

`amplify console`

Ya hemos terminado el tutorial 🥳


