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

:speech_balloon: **Nota:** El archivo docs.yaml contiene toda la documentación de nuestra api, mediante el framewrok swagger y su especificación definiremos cada unos de los endpoints que expondremos en nuestra api REST.

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
