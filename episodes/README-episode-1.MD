<h1 align="center">Workshop contruyendo una Api Rest con Node.js</h1>

<h4 align="center">
Node.js + Koa.js + Mongoose + EsLint
</h4>

# Tabla de Contenido

- [Tabla de Contenido](#tabla-de-contenido)
  - [Introducción](#introducci%c3%b3n)
  - [Episodios](#episodios)
  - [Stack](#stack)
  - [Estructura del Proyecto](#estructura-del-proyecto)
  - [Proposito de este Episodio](#proposito-de-este-episodio)
    - [Como lo haremos:](#como-lo-haremos)
  - [Requisitos](#requisitos)
  - [Pasos para implementar](#pasos-para-implementar)
  - [Probemos nuestra Api](#probemos-nuestra-api)
  - [Cofiguración de ESlint y Prettier](#cofiguraci%c3%b3n-de-eslint-y-prettier)
    - [EsLint](#eslint)
    - [Prettier](#prettier)

## Introducción
Este Workshop intenta ayudar a comprender como construir una Api Rest usando Node.js con la librería Koa.js. Dividiremos el taller en varias sesiones que llamaremos Episodios, en cada uno agregaremos algún complemento que nos ayudará a entender como ir armando nuestro proyecto de Api Rest.

## Episodios
En cada episodio iremos complementando nuestro proyecto.

- **Episodio 1**: Creando mi primera API rest en Node.js y MongoDB

## Stack
- [Koa](https://github.com/koajs/koa) - Middleware para node.js la cual usa funciones asincronas ES2017. Esta librería esta hecha por el equipo de express, y es más ligera que express.
- [Mongoose](https://mongoosejs.com/) - proporciona una solución directa basada en esquemas para modelar los datos de su aplicación con MongoDB
- [EsLint](https://eslint.org/) - ESLint es una herramienta para identificar e informar sobre ruptura de estandares encontrados en el código ECMAScript / JavaScript.
- [Prettier](https://prettier.io/) - Formateador de código, que nos ayuda a mantener limpio y alienado a los estandares.

## Estructura del Proyecto
```
|-- api-rest-nodejs-workshop
    |-- .editorconfig
    |-- .eslintignore
    |-- .eslintrc.yml
    |-- .gitignore
    |-- env-example.yaml
    |-- env.yaml
    |-- package-lock.json
    |-- package.json
    |-- README.md
    |-- src
        |-- routes.js
        |-- server.js
        |-- controllers
        |   |-- person.controller.js
        |-- models
        |   |-- person.model.js
        |-- repositories
        |   |-- person.repository.js
        |-- routes
        |   |-- person.route.js
        |-- schemas
        |-- utils
```

## Proposito de este Episodio
El objetivo principal, es implementar un endpoint que nos permita obtener un registro de una base de datos MongoDB, el endpoint final debe ser similar a http://localhost:3000/person/99 donde 99 es un parametro llamado index usado para recuperar una persona determinada de la base de datos de contactos de ejemplo.

### Como lo haremos:
Comenzaremos estructurando nuestro proyecto, inicializando el proyecto con npm, descargando las librerias npm que usaremos. También inicializaremos el repositorio git, el cual nos permitira tener una rama por cada episodio, de esta manera nuestro proyeco puede ir evolucionando sin perder el trabajo que vayamos construtendo en cada episodio.

Asimismo, crearemos las carpetas y los archivos de código necesarios para poner a andar nuestra api. 

Aunque puedes encontrar el proyecto es muy simple, esto tiene el proposito de ayudarte a comprender los pasos y la relación entre cada uno de los elemntos.

## Requisitos
Antes de empezar revisa ue cumplas con los siguientes requisitos:
- [Node.js](https://nodejs.org/) puede ser la version LTS o la actual
- [Git](https://git-scm.com/)
- [Visual Studio Code](https://code.visualstudio.com/)
- Extensión de VS Code [EditorConfig for VS Code](https://marketplace.visualstudio.com/items?itemName=EditorConfig.EditorConfig)
- Extensión de VS Code [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)
- Extensión de VS Code [Path Intellisense](https://marketplace.visualstudio.com/items?itemName=christian-kohler.path-intellisense)
- Extensión de VS Code [Visual Studio IntelliCode](https://marketplace.visualstudio.com/items?itemName=VisualStudioExptTeam.vscodeintellicode)
- Extensión de VS Code [YAML](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml)
- Extensión de VS Code [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)
- Extensión de VS Code [npm Intellisense](https://marketplace.visualstudio.com/items?itemName=christian-kohler.npm-intellisense)
- Extensión de VS Code [gitignore](https://marketplace.visualstudio.com/items?itemName=codezombiech.gitignore)

## Pasos para implementar
Bueno basta de tanta introducción ha llegado la hora de echar código :-)

1. Creemos la carpeta raíz de nuestro proyecto, sugiero algo como *api-rest-nodejs*
2. Abramos la carpeta con Visual Studio Code
3. Abre el terminal (puede ser de VS Code o el de tu preferencia pero debes dentro de la carpeta) e inicializa el nuevo proyecto npm, con el comando:

```bash
npm init -y
```
4. Instalemos las principales dependencias que usaremos (koa y algunas librerias de apoyo, moongoose para conectarnos a MongoDb, yenv para manejar la configuarción en un yaml y cross-env para facilitar el manejo de la variable de entorno NODE_ENV), ejecuta el siguiente comando:

```bash
npm i koa koa-router koa-bodyparser koa-logger koa-json mongoose yenv cross-env
```
6. Instalemos nodemon para que escuche si cambiamos nuestro codigo y se reinicie nuestro servidor node, pero la instalaremos como *dependencia de desarrollo* con el siguiente comando:
```bash
npm i --save-dev nodemon
```
7. Inicialicemos ahora un nuevo repositorio git para congelar el codigo de cada episodio, usando el comando:
```bash
git init
```
8. Ahora agreguemos el archivo .gitignore necesario para no subir al repositorio archivos innecsarios como los modulos npm, apoyate en el pluging gitignore que instalamos en los requisitos. Abre la Paleta de Comandos (*control + shift + p*) y escribe *gitignore*, y selecciona *Add gitignore* y luego escribe *node* en el cuadro de texto.
9. Agreguemos el archivo *.editorconfig*. Abre de nuevo la Paleta de Comandos (*control + shift + p*) y escribe *editor* y selecciona la opción *Generate .editorconfig*. Este archivo nos permite especificar por ejemplo cuantos espacios tendran nuestros tabs para identar nuestro código.
10. Editemos el archivo *.editorconfig* y cambia el valor de *indent_size* a **2**
11. Ok, ahora creemos las carpetas que usaremos para estructurar nuestro proyecto:
12. Crea la carpeta *src* en la raiz del proyecto
13. Dentro de la carpeta *src* creamos las siguientes carpetas vacias para ir estructurando nuestro proyecto:
  - models
  - repositories
  - controllers
  - routes
14. Dentro de la carpeta *models* creamos *person.model.js* con el siguiente código:
```javascript
/**
 * person.model.js
 * Nos permite gestionar los datos de la colección people de MongoDB
 */
const mongoose = require('mongoose')
const schema = new mongoose.Schema({
  index: {
    type: Number,
    required: true
  },
  age: {
    type: Number,
    required: true
  },
  eyeColor: {
    type: String,
    required: true
  },
  name: {
    type: String,
    required: true
  },
  gender: {
    type: String,
    required: true
  },
  company: {
    type: String,
    required: true
  },
  country: {
    type: String,
    required: true
  },
  email: {
    type: String,
    required: false
  },
  phone: {
    type: String,
    required: false
  },
  address: {
    type: String,
    required: false
  }
}, {
  collection: 'people'
})

const PersonModel = mongoose.model('PersonModel', schema)
module.exports = PersonModel

```
15. Dentro de la carpeta *repositories* creamos *person.repository.js* con el siguiente código:
```javascript
/**
 * person.repository.js
 * En esta clase expondremos los metodos de acceso a datos asociados a el modelo person.model
 */
const PersonModel = require('../models/person.model')
module.exports = class PersonRepository {
  /**
   * 
   * @param {object} filter: puede contener una o mas propiedades del documento en MongoDB por el cual se desea buscar 
   */
  async findOne(filter) {
    return await PersonModel.findOne(filter)
  }

  /**
   * 
   * @param {object} person: contiene las propiedades del documento que se desea guardar, 
   * deberia coincidir con la colección people, con excepción del al propiedad _id que la 
   * establece MongoDb automaticamente
   * @param {*} upsert: si es true (valor por defecto) indica que si no existe se inserte.
   */
  async save(person, upsert = true) {
    const filter = { index: person.index }
    const options = { upsert: upsert }
    await PersonModel.updateOne(filter, person, options)
  }
}


```
16. Dentro de la carpeta *controllers* creamos *person.controller.js* con el siguiente código:
```javascript
/**
 * person.controller.js
 * Responsable por recibir las solicitudes http desde el router person.route.js
 */
const PersonRepository = require('../repositories/person.repository')
const repository = new PersonRepository()

module.exports = class PersonController {
  /**
   * 
   * @param {object} ctx: contexto de koa que contiene los parameteros de la solicitud, en este caso 
   * desde el url de donde sacaremos el valor del parametro index (ctx.params.index)
   */
  async getByIndex(ctx) {
    const index = ctx.params.index && !isNaN(ctx.params.index) ? parseInt(ctx.params.index) : 0
    if (index > 0) {
      const filter = { index: index }
      const data = await repository.findOne(filter)
      if (data) {
        ctx.body = data
      } else {
        ctx.throw(404, `No se ha encontrado la persona con el indice ${index}`)
      }
    } else {
      ctx.throw(422, `Valor ${ctx.params.index} no soportado`)
    }
  }

  /**
   * 
   * @param {object} ctx: contexto de koa que contiene los parameteros de la solicitud, en este caso desde el body, 
   * obtendremos las propiedades de la persona a guardar a traves de ctx.request.body
   */
  async save(ctx) {
    const person = ctx.request.body
    await repository.save(person, true)
    ctx.body = person
  }
}

```
17. Dentro de la carpeta *routes* creamos *person.route.js* con el siguiente código:
```javascript
/**
 * person.route.js
 * Expone los puntos de entrada a traves de endpoints, es el encargado de recibir las solicitudes http que los usuarios o clientes del api nos envia.
 * puede contener diferentes rutas usando combinaciones de diferentes verbos http y parametros
 * por ejemplo:
 * router.get('person/byIndex', '/:index', controller.getByIndex) maneja la solicitudes desde person/99 donde 99 es el valor del parametro index
 */
const KoaRouter = require('koa-router')
const PersonController = require('../controllers/person.controller')
const router = new KoaRouter({ prefix: '/person' })
const controller = new PersonController()

// GET /person/29
router.get('person/byIndex', '/:index', controller.getByIndex)

// POST
router.post('person/post', '/',  controller.save)

module.exports = router


```
18. Ahora necesitaos un par de archivos más para poner andar el proyecto, creamos el archivo *routes.js* dentro de *src* con el siguiente codigo:
```javascript
/**
 * Expone una coleccion de todos los routes de nuestra api, a pesar de que aqui solo se se expone personRoute en la vida real debería exponer todos .
 */
const personRoute = require('./routes/person.route')
// aqui podria exponer todos los routes, por ejemplo module.exports = [personRoute, route2, route3, routen]
module.exports = [personRoute]

```
19. Ahora crearemos el archivo responsable por inicial la aplicación, creamos el archivo *server.js* dentro de *src*
```javascript
/**
 * server.js
 * Responsable por inciar nuestra api, inicializa koa con todos sus middleware y tambien inicialzia la conexión de bd
 */
const Koa = require('koa')
const json = require('koa-json')
const logger = require('koa-logger')
const bodyParser = require('koa-bodyparser')
const yenv = require('yenv')
const mongoose = require('mongoose')

const env = yenv()
const routes = require('./routes')

// Inicializar nuestro servidor usando koa (similar a express)
const app = new Koa()
// Inicializar los middleware
app
  .use(bodyParser())
  .use(json())
  .use(logger())

// cargar los routes que escucharan las peticiones http
routes.map(item => {
  app
    .use(item.routes())
    .use(item.allowedMethods())
})
// abrir la conexión con MongoDB
mongoose
  .connect(env.MONGODB_URL, { useNewUrlParser: true })
  .then(() => {
    // iniciar el servidor koa para que empiece a escuchar peticiones
    app.listen(env.PORT, () => {
      console.log(`Escuchando en el puerto ${env.PORT}`)
    })
  })
  .catch(error => {
    console.error(error)
  })

```
20. Agreguemos el archivo de configuración *env.yaml* a la raiz del proyecto, con el siguiente coódigo:
```yaml
development:
    PORT: 3000
    MONGODB_URL: "mongodb+srv://tu-usuario:tu-contraseña@cluster0-8hxu4.mongodb.net/contacts?retryWrites=true&w=majority"
production:
    PORT: 3000
    MONGODB_URL: "mmongodb+srv://tu-usuario:tu-contraseña@cluster0-8hxu4.mongodb.net/contacts?retryWrites=true&w=majority"

```
**Nota:** solicita el usuario y contraseña para que los reemplaces en el env.yaml
21. Ahora agregaremos el nombre del archivo *env.yaml* al archivo *.gitignore*, esto es una mejor practica, ya que nunca debemos exponer credenciales o datos sensibles a repositorios publicos. Simplemente abre el archivo y agrega en una linea nueva *env.yaml*

## Probemos nuestra Api
Bien hasta aquí hemos implementado el código para construir nuestra api, pero y como diablos puedo ponerla en marcha y probarla!!
1. Abrimos el archivo *package.json* que esta en la raiz y agrega dentro de la sección *scripts* la siguiente linea:
```json
"start": "cross-env NODE_ENV=development nodemon ./src/server.js",
```
2. Para iniciar nuestra aplicación ejecutamos el comando:
```bash
npm start
```
Si todo sale bien 
```bash
[nodemon] 2.0.4
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: js,mjs,json
[nodemon] starting `node ./src/server.js`
(node:13568) DeprecationWarning: current Server Discovery and Monitoring engine is deprecated, and will be removed in a future version. To use
the new Server Discover and Monitoring engine, pass option { useUnifiedTopology: true } to the MongoClient constructor.
Escuchando en el puerto 3000
```
3. Ahora solo nos queda probar una petición, puedes usar postman para enviar el request, o como es get puedes enviarla directamente por el browser
```
http://localhost:3000/person/29
```
4. Si hicimos todo bien, entonces veremos un json con los datos de la persona buscada:
```json
{
  "_id": "5dc88091ffc42b829564b160",
  "index": 29,
  "age": 36,
  "eyeColor": "green",
  "name": "Beach Rutledge",
  "gender": "male",
  "company": "URBANSHEE",
  "country": "PE",
  "email": "beachrutledge@urbanshee.com",
  "phone": "+1 (900) 521-2063",
  "address": "921 Karweg Place, Connerton, Arkansas, 3696"
}
```
## Cofiguración de ESlint y Prettier
### EsLint
1. Instalemos eslint como dependencia de desarrollo:
```bash
npm i --save-dev eslint
```
2. Inicialicemos eslint con el siguiente comando:
```bash
npx eslint --init
```
**Notas:** 
npx nos permite ejecutar scripts de paquetes que se encuentra dentro del proyecto (carpeta node_modules).
Cuando ejecutamos el comando se nos presenta una serie de preguntas que debemos elegir, para efectos de este demos, seleccionar los siguiente:
- To check syntax, find problems, and enforce code style
- CommonJs
- Node of these
- Does your project use TypeScript (N)
- Desmarcar Browser () y marcar Node (x)
- Use a popular style guide
- Estilo: (Standart)
- (Formato del archivo de configuración) (YAML)
- Instalar dependencias necesarass (Y)
  
### Prettier
Recuerda haber instalado el plugging de prettier (mencionado en los requisitos)
3. Instalamos Prettier como dependencia de desarrollo:
```bash
npm i --save-dev prettier
```
4. Creemos el archivo *.prettierrc.js* en la raiz del proyecto y lo editamos con el siguiente codigo:
```javascript
module.exports = {
  "endOfLine": "lf",
  semi: false,
  trailingComma: "all",
  singleQuote: true,
  printWidth: 120,
  tabWidth: 2,
  endOfLine: "auto"
};
```
5. Editemos el archivo de configuración de eslint *.eslintrc.yml* para que tenga los plugins y las extensiones de prettier, reemplaza el contenido por:
```yaml
env:
  commonjs: true
  es6: true
  node: true
extends:
  - standard
  - eslint:recommended
  - prettier
  - plugin:prettier/recommended
globals:
  Atomics: readonly
  SharedArrayBuffer: readonly
parserOptions:
  ecmaVersion: 11
  sourceType: module
plugins:
  - prettier
rules: {}

```
6. Edita el archivo *package.json* y agrega las siguientes lineas a la sección *scripts*, las cuales nos permitiran ejecutar la comprobación si nuestro codigo cumple con los estandares y reglas de eslint:
```json
"lint:show": "eslint src/ -f stylish",
"lint:fix": "eslint --fix --ext .js .",
```
7. Revisemos si tenemos errores que no satisfagan las reglas de eslint, ejecutemos el comando:
```bash
npm run lint:show
```
**Nota**: si tenemos errores se mostraran como resultado en la consola, donde se nos indicaran el archivo y la linea.
8. Si tenemos errores, intentemos arreglarlos automaticamente ejecutando el script:
```bash
npm run lint:fix
```
9. Vuelve a ejecutar el comando: *npm run lint:show* y veras que o no tienes errores o bajo la cantidad
10. Probemos nuestro código de nuevo: ejecuta *npm start* y accede por postman o el browser a http://localhost:3000/person/29