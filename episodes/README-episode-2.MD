<h1 align="center">Workshop contruyendo una Api Rest con Node.js</h1>

<h2 align="center">
Episodio 2: Implementando validaciones de request y mejores prácticas de manejo de errores.
</h2>

# Tabla de Contenido

- [Tabla de Contenido](#tabla-de-contenido)
  - [Objetivo General](#objetivo-general)
  - [Objetivos Específicos](#objetivos-específicos)
    - [Como lo haremos:](#como-lo-haremos)
  - [Pasos para implementar](#pasos-para-implementar)

## Objetivo General

Implementando validaciones de request y mejores prácticas de manejo de errores.

## Objetivos Específicos

1. Implementar validaciones de los requests http usando joi.
2. Centralizar el manejo de errores usando mejores practicas.
3. Reto: Queremos que nuestro modelo tenga ahora una nueva propiedad compleja (objeto json), llamada preferences, la cual tenga las propiedades hobby (string, opcional) y color (string, opcional). Entonces, deberás modificar:
   1. La persistencia, es decir el modelo para que guarde esa propiedad en la misma colección.
   2. Implementar la validación en el post que llama al save, asegurate que el color unicamente pueda recibir las palabras: rojo, azul, verde, si es un color diferente debes devolver un error 422.

### Como lo haremos

Para validar los requests que llegan a nuestros routes, utilizaremos @hapi/joi, el cual nos permite crear esquemas con cada parametro que recibamos ya en cualquiera de las partes de cada request (header, body, params y query), porsteriormente crearemos una función que nos permita pasarle el esquema que intenamos validar y usarlo como middleware en el route antes de llamar a nuestro controlador.

En cuanto al manejo de errores, usaremos la librería winston para logear nuestros errores, aunque podriamos usar cualquier otra librería esto de hecho no es lo más relevante, ya que lo importante es mandejar de manera centralizada nuestros errores e incluso disponer de nuetra logica de negocio para manejar los errores consistentemente.

Te invito a darle un vistazo a este [excelente documento de mejores parcticas](https://github.com/goldbergyoni/nodebestpractices#2-error-handling-practices)

## Pasos para implementar

### Validar los request usando @hapi/Joi

1.Creemos otra rama a partir del codigo del episodio anterior:

``` bash
git checkout -b episodio-2
```

2.Instalemos la librería <kbd>Joi</kbd> definir esquemas de validacion:

``` bash
npm i @hapi/joi
```

3.Crea la carpeta <kbd>utils</kbd> dentro de src/

4.Dentro de la carpeta utils crea el archivo <kbd>schema-validator.js</kbd> con el siguiente codigo:

```javascript
const validateRequest = (contextPart, label, schema, options) => {
  if (!schema) return
  const { error } = schema.validate(contextPart, options)
  if (error) {
    throw new Error(`Invalid ${label} - ${error.message}`)
  }
}
// middleware de validacion que usaremos para validar los request pasandole un determinado esquema
const validate = (schema) => (ctx, next) => {
  try {
    validateRequest(ctx.headers, 'Headers', schema.headers, { allowUnknown: true })
    validateRequest(ctx.params, 'URL Parameters', schema.params)
    validateRequest(ctx.query, 'URL Query', schema.query)
    if (ctx.request.body) {
      validateRequest(ctx.request.body, 'Request Body', schema.body)
    }
    return next()
  } catch (error) {
    ctx.throw(422, error.message)
  }
}

module.exports = validate

```

5.Creemos la carpeta <kbd>schemas</kbd> dentro de src/

6.Creemos el archivo <kbd>person.schema.js</kbd> con el siguiente codigo:

```javascript
const Joi = require('@hapi/joi')

// esquema de validacion que usaremos en el route GET person/:index
const byIndex = Joi.object().keys({
  index: Joi.number().min(1).required(),
})

// esquema de validacion que usaremos en el route POST person/
const post = Joi.object().keys({
  index: Joi.number().min(1).required(),
  age: Joi.number().min(5).max(100).required(),
  eyeColor: Joi.string().valid('black', 'blue', 'green', 'brown', 'grey').required(),
  name: Joi.string().required(),
  gender: Joi.string().valid('male', 'female').required(),
  company: Joi.string().required(),
  country: Joi.string().length(2).uppercase().required(),
  email: Joi.string().email().required(),
  phone: Joi.string().required(),
  address: Joi.string().required(),
})

module.exports = {
  post,
  byIndex
}

```

7.Ahora ya que en los pasos anteriores tenemos nuestro esquema de validacion y niestro middleware que podemos usar en koa, tendremos que editar el archivo <kbd>routes/person.route.js</kbd> para agregar las referencias y utilizarlo en los routes, el archivo final debe quedar asi:

:speech_balloon: **Nota:** : el código siguiente no contiene la ruta que implementaste en el reto del episodio 1, por lo que respalda esa linea para agregarla luego de copiar el siguiente fragmento.

``` javascript
/**
 * person.route.js
 * Expone los puntos de entrada a traves de endpoints, es el encargado de recibir las solicitudes http que los usuarios o clientes del api nos envia.
 * puede contener diferentes rutas usando combinaciones de diferentes verbos http y parametros
 * por ejemplo:
 * router.get('person/byIndex', '/:index', controller.getByIndex) maneja la solicitudes desde person/99 donde 99 es el valor del parametro index
 */
const KoaRouter = require('koa-router')
const PersonController = require('../controllers/person.controller')
// referenciamos los esquemas declarados
const personSchemas = require('../schemas/person.schema')
// referenciamos el middleware de validacion de esquemas
const schemaValidator = require('../utils/schema-validator')

const router = new KoaRouter({ prefix: '/person' })
const controller = new PersonController()

// creamos una instancia del validador pasandole la parte del request que queremos validar en este caso (params) y el esquema apropiado
const byIndexValidator = schemaValidator({ params: personSchemas.byIndex })

// creamos una instancia del validador pasandole la parte del request que queremos validar en este caso (body) y el esquema apropiado
const postValidator = schemaValidator({ body: personSchemas.post })

// GET /person/29
router.get('person/byIndex', '/:index', byIndexValidator, controller.getByIndex)

// POST
router.post('person/post', '/', postValidator, controller.save)


module.exports = router
```

### Manejo de errores

1.Instalemos la librería winston para logear los errores en un archivo de texto:

:speech_balloon: **Nota:** : incluso con winston puedes usar la librerria winston-elasticsearch de su ecosistema para logear en elasticsearch.

``` bash
npm i winston

```
2. 