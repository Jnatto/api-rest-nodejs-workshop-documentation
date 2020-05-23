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

## Introducción
Este Workshop intenta ayudar a comprender como construir una Api Rest usando Node.js con la librería Koa.js. Dividiremos el taller en varias sesiones que llamaremos Episodios, en cada uno agregaremos algún complemento que nos ayudará a entender como ir armando nuestro proyecto de Api Rest.

## Episodios
En cada episodio iremos complementando nuestro proyecto.

- [Episodio #1 Creando mi primera API rest en Node.js y MongoDB](episodes/README-episode-1.MD)

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