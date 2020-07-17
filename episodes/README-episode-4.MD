<h1 align="center">Workshop contruyendo una Api Rest con Node.js</h1>

<h2 align="center">
Episodio 4: Desplegar nuestro api rest en la nube.
</h2>

# Tabla de Contenido

- [Tabla de Contenido](#tabla-de-contenido)
  - [Objetivo General](#objetivo-general)
  - [Objetivos Específicos](#objetivos-específicos)
    - [Requisitos:](#requisitos)
    - [Como lo haremos:](#como-lo-haremos)
  - [Pasos para implementar](#pasos-para-implementar)
    - [Implementar el código de la función Lambda:](#implementar-el-codigo-de-la-funcion-Lambda)
    - [Crear Función Lambda:](#crear-funcion-lambda)
    - [Configurar API Gateway:](#configurar-api-gateway)

## Objetivo General

Desplegar nuestro api rest en la nube usando AWS Api Gateway + Lambda + Lambda Api

## Objetivos Específicos

1.Implementar el código del api rest preprarlo para que sea ejecutado a travpes de una función Lambda.
2.Crear una función Lamnda y desplegar nuestro código.
3.Crear un Api en AWS Api Gateway.
4.Probar nuestro api a través de postman.

### Requisitos

Necesitas tener una cuenta de AWS para poder desplegar, puedes crearla desde [aqui](https://aws.amazon.com/resources/create-account/), si ya dispones de una cuenta, debes logearte con un usuario que tengas los privilegios necesario para crear recursos en Api Gateway, Crear funciones Lambdas e incluso crear roles de IAM.

### Como lo haremos

En los episodios anteriores hemos implementado nuestro proyecto con una estructura ideal para desplegar ya sea en una instancia de **AWS EC2** (como una maquina virtual) o también podriamos dockerizar nuestro proyecto y desplegarlo a través de **AWS ECS** (Elastic Container Service).

Sin embargo, en este episodio lo que haremos es desplegar una versión simplificada de nuestra api usando un enfoque *serverless*.

**Implementar el código de la función Lambda**, esta vez implementaremos un proyecto nuevo utilizando algunos elemntos de los epidosios anteriores.

**Desplegar la función Lambda**, luego debemos crear la una función Lambda basado en el código del proyecto.

**Expondremos nuestra Api usando AWS API Gateway**, de esta manera podremos disponer de los endpoints y poder acceder via http.

## Pasos para implementar

### Implementar el código de la función Lambda

1.Creemos una carpeta llamada *lambda-api-demo*
2.Desde el terminal ingresemos dentro de la carpta

``` bash
cd lambda-api-demo
```

3.Inicialicemos el proyecyo con npm

``` bash
npm init -y
```

4.Instalemos los módulos npm que vamos a usar

``` bash
npm i lambda-api yenv mongoose @hapi/joi
```

5.Abramos la carpeta con Visual Studio Code

6.Creemos el archivo env.yaml con la cadena de conexión de mongodb

``` yaml
development:
  MONGODB_URL: "mongodb+srv://<usuario aqui>:<password aqui>@cluster0-8hxu4.mongodb.net/contacts?retryWrites=true&w=majority"

``` 

7.Creemos la carpeta *src*

8.Creemos el archivo <kbd>handler.js</kbd>

```javascript
// src/utils/api-docs.js
const yenv = require('yenv');
const api = require('lambda-api')({ version: 'v1.0', base: 'people' });
const dbConnector = require('./utils/db-connector');
const PersonRepository = require('./person.repository');
const personSchemas = require('./person.schema')
const schemaValidator = require('./utils/schema-validator')
const byIndexValidator = schemaValidator({ params: personSchemas.byIndex })
const postValidator = schemaValidator({ body: personSchemas.post })

const env = yenv();
let cachedConnection;
let repository;

api.get('/:index', byIndexValidator, async (req, res) => {
  const index = req.params.index;
  const person = await repository.findOne({ index: index});
  if (person) {
    res.json(person);
  } else {
    res.error(404, `no existe la persona ${index} no existe`);
  }
});

api.get('/', async (req, res) => {
  const people = await repository.find(req.query);
  if (people && people.length > 0) {
    res.json(people);
  } else {
    res.error(404, 'no se han encontrado registros.');
  }
});

api.post('/', postValidator, async (req, res) => {
  const person = req.body;
  await repository.save(person, true);
  res.json(person);
});

api.use((err, req, res, next) => {
  if (err) {
    console.error(err);
  }
  next();
})

// Punto de entrada de la función Lambda
module.exports.router = async (event, context) => {
  context.callbackWaitsForEmptyEventLoop = false;
  const mongoOptions = { useNewUrlParser: true, useUnifiedTopology: true, bufferCommands: false, bufferMaxEntries: 0 };
  cachedConnection = await dbConnector.connect(env.MONGODB_URL, cachedConnection, mongoOptions);
  repository = new PersonRepository(cachedConnection);
  // Run the request
  return await api.run(event, context);
};

```

9.Creemos el archivo <kbd>person.schema.js</kbd> que usaremos para validar los request de los endpoints

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
  preferences: Joi.object({
    hobby: Joi.string(),
    color: Joi.string().valid('rojo', 'azul', 'verde'),
  }),
})

module.exports = {
  post,
  byIndex,
}

```

10.Creemos el archivo <kbd>person.model.js</kbd> para mapear con la colección de mongodb que usaremos

```javascript
/**
 * person.model.js
 * Nos permite gestionar los datos de la colección people de MongoDB
 */
const mongoose = require('mongoose');
const schema = new mongoose.Schema(
  {
    index: {
      type: Number,
      required: true,
    },
    age: {
      type: Number,
      required: true,
    },
    eyeColor: {
      type: String,
      required: true,
    },
    name: {
      type: String,
      required: true,
    },
    gender: {
      type: String,
      required: true,
    },
    company: {
      type: String,
      required: true,
    },
    country: {
      type: String,
      required: true,
    },
    email: {
      type: String,
      required: false,
    },
    phone: {
      type: String,
      required: false,
    },
    address: {
      type: String,
      required: false,
    },
    preferences: {
      hobby: {
        type: String,
        required: false,
      },
      color: {
        type: String,
        required: false,
      },
    },
  },
  {
    collection: 'people',
  },
)

const getModel = (connection) => {
  const model = connection.model('PersonModel', schema);
  return model;
}

module.exports = {
  getModel
}

```

11.Agreguemos el archivo <kbd>person.repository.js</kbd>

```javascript
const PersonModel = require('./person.model');

module.exports = class PersonRepository {
  constructor(connection) {
    // this.connection = connection;
    this.model = PersonModel.getModel(connection);
  }
  /**
   *
   * @param {object} filter: puede contener una o mas propiedades del documento en MongoDB por el cual se desea buscar
   */
  async findOne(filter) {
    return await this.model.findOne(filter);
  }

  // /**
  //  *
  //  * @param {object} person: contiene las propiedades del documento que se desea guardar,
  //  * deberia coincidir con la colección people, con excepción del al propiedad _id que la
  //  * establece MongoDb automaticamente
  //  * @param {*} upsert: si es true (valor por defecto) indica que si no existe se inserte.
  //  */
  async save(person, upsert = true) {
    const filter = { index: person.index }
    const options = { upsert: upsert }
    await this.model.updateOne(filter, person, options)
  }

  async find(filter, page = 1, size = 10) {
    const skip = (page - 1) * size
    return await this.model.find(filter, null, { skip: skip, limit: size })
  }
}

```

12.Creemos la carpeta *utils* dentro de la carpeta *src*

13.Creemos el archivo <kbd>db-connector.js</kbd> el cual usaremos para abrir la conexión con mongodb

```javascript
const mongoose = require('mongoose');

module.exports.connect = async (uri, cachedConnection, options) => {
  if (cachedConnection) {
    console.log('=> using cached database instance')
    return Promise.resolve(cachedConnection)
  }

  cachedConnection = await mongoose.createConnection(uri, options)
  console.log('=> new database instance')
  return cachedConnection
}

```

14.Creemos el archivo <kbd>schema-validator.js</kbd> el cual usaremos para validar los request mediante los esquemas definos en person.schema.js

```javascript
const validateRequest = (contextPart, label, schema, options) => {
  if (!schema) return
  const { error } = schema.validate(contextPart, options)
  if (error) {
    throw new Error(`Invalid ${label} - ${error.message}`)
  }
}
// middleware de validacion que usaremos para validar los request pasandole un determinado esquema
const validate = (schema) => (req, res, next) => {
  try {
    validateRequest(req.headers, 'Headers', schema.headers, { allowUnknown: true })
    validateRequest(req.params, 'URL Parameters', schema.params)
    validateRequest(req.query, 'URL Query', schema.query)
    if (req.body) {
      validateRequest(req.body, 'Request Body', schema.body)
    }
    return next()
  } catch (error) {
    res.error(422, error);
  }
}

module.exports = validate

```

### Crear Función Lambda

1.Abrimos la [consola de AWS](https://console.aws.amazon.com/) e inicia sesión con un usuario con los privilegios necesarios (recuerda qhaber leído la seccion Requisitos)

2.Asegurate de utilizar la region US East N Virginia (us-east-1)

3.En la caja de texto Find Services, escribe Lambda, y selecciona la opción Lambda.

4.Desde el dashboard de Lambda (o menú lateral funciones) haz clic en el botón ***Create function****

5.Dejar seleccionada la opción Autor from scratch

6.En function name coloca el nombre ***lambda-api-rest***

7.En Runtine deja seleccionado Node.js 12.x

8.Haz clic en el botón ***Create function***

9.Ingresa en la carpeta rapiz de tu código y comprime el contenido de todos los archivos y carpetas, usa cualquier nombre pero el formato debe ser .zip

10.Desde la consola de AWS dentro de la función que acabas de crear, ubica el botón*Actions*que esta en la parte superior izquierda del código de la función, elije*Upload a .zip file*, deberas subir el zip generado en el paso anterior

11.Por defecto el punto de entrada de la función es index.handler, debemos cambiarlo a ***src/handler.router***, busca la sección Edit basic settings y haz clic en el botón edit y cambia el handler y también en Timeout asegurate que al menos este en 45 segundos en lugar de 3

12.Haz clic en el botón Save

13.Asegurate de que el archivo env.yaml tenga la cadena de conexión usuario y contraseña correcta

14.Busca la sección Environment variables y haz clic en el botón Manage environment variables y luego en Add environment variables, en key escribe ***NODE_ENV*** y en Value ***development***

15.Haz clic en*Save*

### Configurar API Gateway

1.Abre el dashboar de [AWS API Gateway](https://console.aws.amazon.com/apigateway/main/apis?region=us-east-1)

2.Haz clic en el botón *Create API*

3.Selecciona el botpon Build de la opción ***REST API***

4.En la sección Choose the protocol elije ***REST***

5.En Create new API selecciona ***New API***

6.En Settings escribe un nombre para el API

7.En Endpoint Type debe estar seleccionada la opción Regional

8.Haz clic en el botón ***Create API***

9.Haz clic en el botón *Actions* y elije ***Create Resource***

10.Selecciona el check de ***proxy resource***

11.Selecciona el check ***Enable API Gateway CORS***

12.Haz clic en el botón ***Create Resource***

13.Ahora en el método *ANY* (que aparece debajo del recurso {proxy+}) asegurate de que este seleccionado ***Lambda Function Proxy*** en la opción *Integration Type*

14.En Lambda Region debes elegir us-east-1 o la misma region donde creaste tu lambda.

15.En Lambda Function escribe el nombre la función Lambda que ceraste en la sección anterior ***lamba-api-rest***

16.Haz clic en el botón Save y en el pop up de permisos haz clic en ***OK***

17.Nuevamente haz clic en el botón *Actions* y selecciona ***Deploy API*** elije New Stage ponle el nombre *PRD* y haz clic en *Deploy*

18.Fijate que ahora debes visualizar el link del del ambiente que acabas de crear, debe aparecer en la parte superior de la pantalla como algo similar a Invoke URL: https://dvntyal156.execute-api.us-east-1.amazonaws.com/PRD

19.Usa es URL en postman y prueba las siguientes tramas:

- GET https://dvntyal156.execute-api.us-east-1.amazonaws.com/PRD/people/100
- GET https://dvntyal156.execute-api.us-east-1.amazonaws.com/PRD/people?eyeColor=green&gender=male
- POST https://dvntyal156.execute-api.us-east-1.amazonaws.com/PRD/people/

```json
{
    "index": 1000,
    "address": "calle xxxx",
    "age": 36,
    "company": "BELCORP",
    "country": "PE",
    "email": "beachrutledge@urbanshee.com",
    "eyeColor": "black",
    "gender": "male",
    "name": "Pepe Trueno",
    "phone": "+1 (900) 521-2063"
}
```

Si hicimos los pasos correctamente entonces las solicitudes serán exitosas.