<h1 align="center">Workshop contruyendo una Api Rest con Node.js</h1>

<h2 align="center">
Episodio 3: Documentando y probando nuestra api.
</h2>

# Tabla de Contenido

- [Tabla de Contenido](#tabla-de-contenido)
  - [Objetivo General](#objetivo-general)
  - [Objetivos Específicos](#objetivos-específicos)
    - [Como lo haremos:](#como-lo-haremos)
  - [Pasos para implementar](#pasos-para-implementar)

## Objetivo General

Documentar y probar nuestra api.

## Objetivos Específicos

1. Documentar nuestra api usando swagger.
2. Implementar Unit Tests.
3, Implementar Integration Tests.
3. Reto: Implementa una prueba unitaria.

### Como lo haremos

**Para documentar** nuestra api usaremos swagger especificamente el par de librerias *swagger2* y *swagger2-koa*. Swagger nos permite exponer en una página la especificación de los endpoints con cada uno de los parametros que reciben u objetos de las respuestas.

**Implementar los test**, principalmente usaremos el framework *jest* y lo complementaremos con *supertest* para las pruebas de integración del api y la librería *@shopify/jest-koa-mocks"* la cual nos permite crear mokcs de el objet ctx de koa (el cual representa el request http).

Si quieres profundizar en el tema de test, te dejo dos excelentes artículos:

- [Node.js & JavaScript Testing Best Practices (2020)](https://medium.com/@me_37286/yoni-goldberg-javascript-nodejs-testing-best-practices-2b98924c9347)
- [Testing And Overall Quality Practices](https://github.com/goldbergyoni/nodebestpractices#4-testing-and-overall-quality-practices)

## Pasos para implementar

### Documentar con Swagger

1. Iniciemos abriendo el proyecto donde los dejamos en el episodio 2, incluso sugiero crear una rama episo3 apratir del episodio2.
2. Instalemos las librerías npm:

``` bash
npm i swagger2 swagger2-koa
```

3.Ahora crearemos el archivo <kbd>api-docs.js</kbd> dentro de la carpeta *src/utils*

```javascript
// src/utils/api-docs.js
const path = require('path')
const swagger = require('swagger2')
const { ui } = require('swagger2-koa')
// indicar la ruta fisica del archivo docs.yaml utils/docs.yaml
const file = path.join(__dirname, 'docs.yaml')
const document = swagger.loadDocumentSync(file)
// exponer la documentación en la ruta /docs
module.exports = ui(document, '/docs')

```

4.Creemos ahora el archivo <kbd>docs.yaml</kbd> dentro de la carpeta *src/utils*

:speech_balloon: **Nota:** El archivo docs.yaml contiene toda la documentación de nuestra api, mediante el framewrok swagger (Open Api) y su especificación definiremos cada unos de los endpoints que expondremos en nuestra api REST.

Recomiendo darle una lectura al siguiente [link](https://swagger.io/docs/specification/2-0/basic-structure/) para conocer más acerca de la especificación.

```yaml
swagger: "2.0"
info:
  title: "API Rest demo usando node.js + koa.js"
  version: "1.0"
  description: "API Rest de ejemplo usando koa, mongoose"
basePath: /
schemes:
  - http
paths:
  /person/{index}:
    get:
      tags:
        - Person
      summary: "Permite obtener los datos de una persona por el campo index"
      parameters:
        - in: path
          name: index
          type: integer
          required: true
          description: "Identificador unico de la persona"
      produces:
        - application/json
      responses:
        "200":
          description: OK
          schema:
            $ref: "#/definitions/Person"
        "404":
          description: "Recurso inexistente"
          schema:
            $ref: "#/definitions/AppError"
        "422":
          description: "Formato incorrecto"
          schema:
            $ref: "#/definitions/AppError"
        "500":
          description: "Error desconocido"
          schema:
            $ref: "#/definitions/AppError"
  /person:
    post:
      summary: "Permite guardar los datos de una nueva persona o actualizar lo de una existente"
      parameters:
        - in: body
          required: true
          name: Person
          description: "Datos de la persona"
          schema:
            $ref: "#/definitions/Person"
      responses:
        200:
          description: Actualizado
          schema:
            $ref: "#/definitions/Person"
        201:
          description: Creado
          schema:
            $ref: "#/definitions/Person"
        422:
          description: "Formato incorrecto"
          schema:
            $ref: "#/definitions/AppError"
        500:
          description: "Error desconocido"
          schema:
            $ref: "#/definitions/AppError"
      deprecated: false
definitions:
  Person:
    description: "Representa los datos de un contacto o persona"
    type: object
    properties:
      index:
        description: "Valor numerico que identifica a a la persona"
        type: integer
      age:
        description: "Edad de la persona"
        type: integer
      eyeColor:
        description: "Color de ojos"
        type: string
        enum:
          - black
          - blue
          - green
          - brown
          - grey
      name:
        description: "Nombre y Apellido"
        type: string
      gender:
        description: "Genero"
        type: string
        enum:
          - male
          - female
      company:
        description: "Compañía para la que labora"
        type: string
      country:
        description: "Pais de origen"
        type: string
      email:
        description: "Dirección decorreo electrónico"
        type: string
      phone:
        description: "Número telefónico"
        type: string
      address:
        description: "Dirección de habitación"
        type: string
  AppError:
    type: object
    properties:
      message:
        description: "Mensaje del error"
        type: string
      name:
        description: "Identificador del Error"
        type: string
        enum:
          - InvalidInputError
          - UnauthorizedError
          - OperationNotAllowedError
          - NotFoundError
          - DuplicateItemError
          - ConflictError
          - BadFormatError
          - UnknownError
      status:
        description: "Codigo Http"
        type: integer
      date:
        description:  "Fecha en la que ocurrió el error"
        type: string
      isOperational:
        description: "Si es verdadero indica que es un error controlado"
        type: boolean
      innerException:
        description: "Error original"
      expose:
        description: "Flag que indica se debe mostrar en la respuesta http"
        type: boolean
```

5.Actualizar el archivo server.js para agregar a koa el middleware que creamos en el paso 3

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
const apiError = require('./utils/api-error')
const LogManager = require('./utils/logging/log-manager')
// Inicializar nuestro servidor usando koa (similar a express)
const app = new Koa()
const logManager = new LogManager()

// LINEA AGREGADA: referenciar al middleware que expondra la documentación
const docs = require('./utils/api-docs')

// Inicializar los middleware
// LINEA MODIFICADA: agregamos docs dentro de la cadena de middlwares generales
app.use(bodyParser()).use(json()).use(logger()).use(apiError).use(docs)

// cargar los routes que escucharan las peticiones http
routes.map((item) => {
  app.use(item.routes()).use(item.allowedMethods())
})

app.on('error', (err, ctx) => {
  console.error('logging error')
  const isOperationalError = logManager.error(err)
  if (!isOperationalError) {
    process.exit(1)
  }
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
  .catch((error) => {
    console.error(error)
  })
```

6.Ahora nos toca probar nuestra documentación, inicia con

```bash
npm start
```

7.Navega al link http://localhost:3000/docs

### Implementar los Tests con Jest

Antes de empezar te describo las librerías que vamos a usar en esta sección:

- [Jest](https://jestjs.io/docs/en/getting-started) Jest es un framework para testing muy popular en javascript creado por Facebook

- [supertest](https://github.com/visionmedia/supertest) Supertest es un modulo de npm el cual nos sirve para probar de manera automatizada los endpoints de un Api Rest. Esta librería la usaremos para los integration tests.

- [jest-koa-mocks](https://www.npmjs.com/package/@shopify/jest-koa-mocks?activeTab=readme) Esta libreria es usada para implementar unit testing, nos permite crear mocks del contexto de koa, es decir del parámetro ctx que reciben los middleware y controlador, recuerda que este objeto ctx representa la solicitud http y contiene todos los parámetros (query string, path o body), asi como el body de la respuesta que retornaremos.

- [eslint-plugin-jest](https://www.npmjs.com/package/eslint-plugin-jest) Plugin de eslint para Jest.

1.Instalemos las librerias necesarias para crear nuestros tests.

``` bash
npm i --save-dev jest supertest @shopify/jest-koa-mocks eslint-plugin-jest
```

2.Creemos el archivo <kbd>app.js</kbd> dentro de la carpeta *src/*

```javascript
// src/app.js
const Koa = require('koa')
const json = require('koa-json')
const logger = require('koa-logger')
const bodyParser = require('koa-bodyparser')
const yenv = require('yenv')
const mongoose = require('mongoose')

const env = yenv()
const routes = require('./routes')
const apiError = require('./utils/api-error')
const LogManager = require('./utils/logging/log-manager')
const docs = require('./utils/api-docs')
// Inicializar nuestro servidor usando koa (similar a express)
const app = new Koa()
const logManager = new LogManager()
// Inicializar los middleware
app.use(bodyParser()).use(json()).use(logger()).use(apiError).use(docs)

// cargar los routes que escucharan las peticiones http
routes.map((item) => {
  app.use(item.routes()).use(item.allowedMethods())
})

app.on('error', (err, ctx) => {
  const isOperationalError = logManager.error(err)
  if (!isOperationalError) {
    process.exit(1)
  }
})

mongoose.connect(env.MONGODB_URL, { useNewUrlParser: true, useUnifiedTopology: true })
mongoose.connection.on('error', console.error.bind(console, 'MongoDB Connection Error...'))

module.exports = app

```

3.Ahora **modificaremos** el archivo <kbd>server.js</kbd> que esta en la misma carpeta.

```javascript
// src/server.js
const yenv = require('yenv')
const app = require('./app')
const env = yenv()
app.listen(env.PORT, () => {
  console.log(`Escuchando en el puerto ${env.PORT}`)
})

```

:speech_balloon: **Nota:** Lo que hemos hecho en los pasos 2 y 3 es necesario en nuestros test, ya que es una mejor práctica recomendada separar el archivo que representa la aplicación (app.js) que contiene la configuración de como debe inicializar la aplciación. Por otro lado, server.js hace referencia a app.js y unicamente se encarga de iniciarla con app.listen. La razón principal de hacer esta separación es permitir que nuestras pruebas de integración, especificamente supertest pueda hacer uso sin necesidad de que este levantado el servidor.

4.Creemos el archivo <kbd>jest.config.js</kbd> en la raiz del proyecto, el cual contiene la configuración de Jest.

```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'node',
}

```

:speech_balloon: **Nota:** Esta configuración es **super importante** ya que el environment por defecto de jest es jsdom el cual esta asociado más a frontend, pero como estamos trabajando con node en backend debemos especificarlo. Te dejo 2 links recomendados para que le des una chequeada. En la documentación de moongose se indica que es extremadamente relevante.

- [testEnvironment](https://jestjs.io/docs/en/configuration#testenvironment-string)
- [Testing Mongoose with Jest](https://mongoosejs.com/docs/jest.html)

5.Editemos el archivo <kbd>.eslintrc.yaml</kbd> el cual está en la raiz del proyecto. Agregaremos a jest como plugin.

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
plugins:
  - prettier
  - jest
rules: {}

```

6.Creemos en la raiz del proyecto la carpeta ***\_\_tests__***

7.Creeemos dentro la carpeta ***person*** dentro de la carpeta ***\_\_tests__***

8.Creemos el archivo <kbd>person.int.test.js<kbd> dentro de la carpeta creada en el paso anterior. Aqui vamos a implementar nuestros integrations tests.

```javascript
//__tests__/person/person.int.test.js
const supertest = require('supertest')
const mongoose = require('mongoose')
// referenciar el archivo de la aplicacion
const app = require('../../src/app')

describe('Person Service', () => {
  afterAll((done) => {
    mongoose.disconnect()
    done()
  })

  describe('Get Person by Index', () => {
    it('When pass an index of existing person, should return person data successfully', async () => {
      const personIndex = 1000
      const response = await supertest(app.callback()).get(`/person/${personIndex}`)

      expect(response.status).toBe(200)
      expect(response.body.index).toBe(personIndex)
    })

    it('When pass an index of non-existing person. should return 404 error', async () => {
      const personIndex = 200000
      const response = await supertest(app.callback()).get(`/person/${personIndex}`)

      expect(response.status).toBe(404)
    })

    it('When pass an index of non-existing person. should return 422 error', async () => {
      const personIndex = 'xxxxx'
      const response = await supertest(app.callback()).get(`/person/${personIndex}`)

      expect(response.status).toBe(422)
    })
  })

  describe('Save Person', () => {
    it('When try to save a person that already exists. Should update successfully', async () => {
      const person = {
        index: 1000,
        address: '921 Karweg Place, Connerton, Arkansas, 3696',
        age: 36,
        company: 'BELCORP',
        country: 'PE',
        email: 'beachrutledge@urbanshee.com',
        eyeColor: 'black',
        gender: 'male',
        name: 'Pepe Trueno',
        phone: '+1 (900) 521-2063',
      }
      const response = await supertest(app.callback()).post('/person').send(person).set('Accept', 'application/json')

      expect(response.status).toBe(200)
    })

    it('When try to save a person without required property. Should return 422 status', async () => {
      const person = {
        address: '921 Karweg Place, Connerton, Arkansas, 3696',
        company: 'BELCORP',
        country: 'PE',
        email: 'beachrutledge@urbanshee.com',
        eyeColor: 'black',
        gender: 'male',
        name: 'Pepe Trueno',
        phone: '+1 (900) 521-2063',
      }

      const response = await supertest(app.callback()).post('/person/').send(person)

      expect(response.status).toBe(422)
    })

    it('When try to save a person with malformed request json body. Should return 422 status', async () => {
      const person = {
        x: 'aaaaa',
      }

      const response = await supertest(app.callback()).post('/person/').send(person)

      expect(response.status).toBe(422)
    })
  })
})

```

9.Ahora creemos el archivo <kbd>person.unit.test.js</kbd> que contendra los unit test de las funcionalidades de personController.

```javascript
//__tests__/person/person.unit.test.js
const { createMockContext } = require('@shopify/jest-koa-mocks')
const PersonController = require('../../src/controllers/person.controller')
const PersonModel = require('../../src/models/person.model')
const AppError = require('../../src/utils/logging/app-error')
const person = {
  _id: '5ec928506f54870b72479654',
  index: 1000,
  address: '921 Karweg Place, Connerton, Arkansas, 3696',
  age: 99,
  company: 'Belcorp',
  country: 'PE',
  email: 'pepe.trueno@email.com',
  eyeColor: 'brown',
  gender: 'male',
  name: 'Pepe Trueno',
  phone: '+1 (900) 521-2063',
}

let ctx = {}
let controller

describe('Person Service', () => {
  beforeAll(() => {
    process.env.NODE_ENV = 'development'
    controller = new PersonController()
  })

  describe('Get person by Index', () => {
    beforeEach(() => {
      PersonModel.findOne = jest.fn()
    })

    it('When pass an index of existing person, should return person data successfully ', async () => {
      // Arrage
      PersonModel.findOne.mockReturnValue(person)
      ctx = createMockContext({
        method: 'GET',
        // url: '/person/1000',
        // statusCode: 200,
        customProperties: { params: { index: 1000 } },
        state: person,
      })
      // Act
      await controller.getByIndex(ctx)
      // Assert
      expect(ctx.status).toBe(200)
      expect(ctx.body).toBe(person)
    })

    it('When pass an index of non-existing person. should return 404 error', async () => {
      PersonModel.findOne.mockReturnValue(null)
      ctx = createMockContext({
        method: 'GET',
        // url: '/person/1000',
        statusCode: 404,
        customProperties: { params: { index: 200000 } },
        state: null,
      })

      // Acts
      await expect(controller.getByIndex(ctx)).rejects.toThrow()

      try {
        await controller.getByIndex(ctx)
      } catch (error) {
        expect(error).toBeInstanceOf(AppError)
        expect(error.isOperational).toBeTruthy()
        expect(error.status).toBe(404)
      }
    })
  })
})

```

:speech_balloon: **Nota:** Dejo unos links de referencias claves para entender la diferencia y el objetivo de las pruebas unitarias y las de inegración. Tambien entender la razón de los unit tests y el paradigma TDD (Tes Driven Development)

- [Los diferentes tipos de tests](https://programacionymas.com/blog/tipos-de-testing-en-desarrollo-de-software)
- [TDD como metodología de diseño de software](https://www.paradigmadigital.com/dev/tdd-como-metodologia-de-diseno-de-software/)
- [Simulando las dependencias en las pruebas unitarias. Dummies vs Stubs vs Mocks vs Spies vs Fakes](https://www.javiergarzas.com/2015/09/dummies-vs-stubs-vs-mocks-vs-spies-vs-fakes.html)

10.Editar archivo package.json y agregar los scripts test y test:cover:
:speech_balloon: **Nota:** Asegurate de que la sección **scripts** del archivo package.json tengo el siguiente fragmento

```json
"scripts": {
    "start": "cross-env NODE_ENV=development nodemon ./src/server.js",
    "lint:show": "eslint src/ -f stylish",
    "lint:fix": "eslint --fix --ext .js .",
    "test": "cross-env NODE_ENV=development jest",
    "test:cover": "cross-env NODE_ENV=development jest --coverage"
  },
```

11.Si seguimos correctamente lso pasos podemos iniciar nuestras pruebas, con:

- npm run test

- npm run test:cover

:speech_balloon: **Nota:** test y test:cover ejecutan ambos bloques de pruebas (unitarias y de integración), pero test:cover muestra la covertura de nuestras pruebas en relación a la totalidad de lineas de codigo.
