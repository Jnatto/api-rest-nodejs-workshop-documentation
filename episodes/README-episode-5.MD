<h1 align="center">Workshop contruyendo una Api Rest con Node.js</h1>

<h2 align="center">
Episodio 5: Desplegar nuestro api rest en la nube de AWS (Segunda Parte).
</h2>

# Tabla de Contenido

- [Tabla de Contenido](#tabla-de-contenido)
  - [Objetivo General](#objetivo-general)
  - [Objetivos Específicos](#objetivos-específicos)
  - [Requisitos](#requisitos)
  - [Como lo haremos](#como-lo-haremos)
  - [Pasos para implementar](#pasos-para-implementar)
    - [Configuracion e Instalación del Framework Serverless](#configuracion-e-instalación-del-framework-serverless)
    - [Crear Proyecto](#crear-proyecto)

## Objetivo General

Conocer otras manera de desplegar en la nube de AWS.

## Objetivos Específicos

1. Usar el [Framework Serverless](https://www.serverless.com/framework/docs/) para desplegar via scripts nuestro Lambda y API Gateway.
2. Desplegar el proyecto implementado hasta el [Episodio 3](README-episode-3.MD) usando un enfoque más tradicional a través de [AWS EC2](https://docs.aws.amazon.com/es_es/AWSEC2/latest/UserGuide/concepts.html)

## Requisitos

:speech_balloon: **Importante:**

1. Necesitas tener una cuenta de AWS para poder desplegar, puedes crearla desde [aqui](https://aws.amazon.com/resources/create-account/), si ya dispones de una cuenta, debes logearte con un usuario que tengas los privilegios necesario para crear recursos en Api Gateway, Crear funciones Lambdas e incluso crear roles de IAM.

2. Implementar el código de la función Lambda (puedes encontrar los pasos en el Episodio 4 Seccion [Pasos para implementar/Implementar el código de la función Lambda](README-episode-4.MD#pasos-para-implementar).

## Como lo haremos

Este episodio tiene dos partes tal como se menciona en los objetivos especificos.

**Usando Serverless Framework para desplegar**. En el episodio anterior implementamos un Lambda el cual lo desplegamos manualmente, ahora lo que haremos será apoyarnos en el Framewrok Serverless para automatizar este proceso.

**Desplegar el Api usando un enfoque tradicional**. El proyecto que construimos hasta el Epidodio 3, lo desplegaremos en una maquina virtual usando el servicio AWS EC2.

## Pasos para implementar

### Configuracion e Instalación del Framework Serverless

1. Instalemos el framework Serverless via npm:

```bash
npm install -g serverless
```

2. Vas a necesitar un usuario con privilegios administrativos y tipo de acceso Programmatic access, si no lo tienes puede crearlo con los siguientes pasos:

- Ir al dashboard de [IAM](https://console.aws.amazon.com/iam) en el cual manejamos usuario, roles y grupos.
- Clic en la opción ***Users*** que aparece en el menú lateral izquierdo.
- Clic en el botón ***Add user***.
- Escribe un nombre en el campo ***User name*** (por ejemplo: *serverless-user*)
- Selecciona unicamente el check ***Programmatic access***
- En las opciones de ***Set permissions*** selecciona ***Attach existing policies directly***
- Selecciona ***AdministratorAccess***
- Clic en botón ***Next: tags***
- Clic en botón ***Next: Review***
- Clic en botón ***Create user***
  
:speech_balloon: **Importante:** Guarda las credenciales (Access key ID y Secret access key), puedes hacer clic en el botón Download csv, o hacer clic en el link Show debajo de la columna Secret access key


### Crear Proyecto

1. Abre el terminal y ubicate en la carpeta raiz donde quieres crear el proyecto.

2. Ejecuta el comando

```bash
serverless
```

3. Se nos mostraran una serie de preguntas:

- No project detected. Do you want to create a new one? (Y/n): ***Seleccionar Y***
- Seleccionar: ***AWS Node.js***
- What do you want to call this project?: ***serverless-lambda-api-demo***
- Would you like to enable this? (Y/n): ***n***

4. Ingresemos a esa carpeta desde el terminal

```bash
cd serverless-lambda-api-demo
```

5. Inicialicemos el proyecto npm

```bash
npm init -y
```
   
6. Si es la primera vez que usamos serverless en nuestra máquina, debemos configurar las credenciales de nuestra cuenta:

```bash
serverless config credentials --provider aws --key <your_access_key> --secret <your_secret_key>
```

:speech_balloon: **Importante:** Recuerda que el paso [Configuracion e Instalación del Framework Serverless](#configuracion-e-instalación-del-framework-serverless) obtuviste las credenciales, usalas para reemplazar <your_access_key> y <your_secret_key>

7. Para la implementación del código puedes copiar todos los archivos y subcarpetas dentro de la carpeta donde implementaste el codigo del ***Episodio 4***, si no tienes el código entonces ejecuta los pasos de dicha sección [Pasos para implementar/Implementar el código de la función Lambda](README-episode-4.MD#pasos-para-implementar)

8. Borremos el archivo handler.js de la raiz para no confundirnos, ya que el nuestro debe encontrarse dentro de la subcarpeta *src*.

9 Abrimos la carpeta del proyecto con *Visual Studio Code*
10. Si seguiste todos los pasos en ***Crear Función Lambda y Configurar API Gateway del episodio 4***, entonces elimina tanto el Lambda como el API en el API Gateway, ya que con el framework serverless desplegaremos estos recursos via comandos.

11. Configuremos el archivo ***serverless.yaml*** el cual nos permitirá desplegar por linea de comandos nuestro lambda y la configuración en el API Gateway:

- Ubica la sección ***functions*** (linea 60) y asegurate de que este el siguiente codigo:

```yaml
functions:
  serverless-lambda-api:
    name: ${self:service}-${self:provider.stage}-serverless-lambda-api
    handler: src/handler.router
    timeout: 30
    events:
      - http:
          path: 'v1/{proxy+}'
          method: any
```

12. Listo llegó la hora de Desplegar

```bash
serverless deploy
```

Si hicimos los pasos correctamente entonces las solicitudes serán exitosas :smiley:
