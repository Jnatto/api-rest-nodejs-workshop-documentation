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
    - [Configuración e Instalación del Framework Serverless](#configuración-e-instalación-del-framework-serverless)
    - [Crear Proyecto Serverless](#crear-proyecto-serverless)
    - [Eliminar los recursos desplegados por serverless](#eliminar-los-recursos-desplegados-por-serverless)
    - [Desplegar el Api usando un enfoque tradicional con EC2](#desplegar-el-api-usando-un-enfoque-tradicional-con-ec2)
      - [Crear la máquina virtual en EC2](#crear-la-máquina-virtual-en-ec2)
      - [Conectarse via SSH a la Instancia](#conectarse-via-ssh-a-la-instancia)
      - [Configurar el software necesario en la Instancia](#configurar-el-software-necesario-en-la-instancia)
      - [Descargar y configura el código fuente](#descargar-y-configura-el-código-fuente)
    - [Eliminar la instancia de EC2](#eliminar-la-instancia-de-ec2)
  - [Links de Interés](#links-de-interés)
    - [Otros Comandos Útiles de Serverless Framework](#otros-comandos-útiles-de-serverless-framework)
    - [Alternativas a Serverless Framework en AWS](#alternativas-a-serverless-framework-en-aws)

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

**Usando Serverless Framework para desplegar**. En el episodio anterior implementamos un Lambda el cual lo desplegamos manualmente, ahora lo que haremos será apoyarnos en el Framework Serverless para automatizar este proceso.

**Desplegar el Api usando un enfoque tradicional**. El proyecto que construimos hasta el Epidodio 3, lo desplegaremos en una maquina virtual usando el servicio AWS EC2.

## Pasos para implementar

### Configuración e Instalación del Framework Serverless

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

### Crear Proyecto Serverless

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

- Asegúrate de que el contenido de tu archivo serverless.yaml sea igual al siguiente fragmento:

```yaml
service: serverless-lambda-api-demo
provider:
  name: aws
  runtime: nodejs12.x
  stage: dev
  region: us-east-1
functions:
  serverless-lambda-api:
    name: ${self:service}-${self:provider.stage}-serverless-lambda-api
    handler: src/handler.router
    timeout: 30
    events:
      - http:
          path: 'people/{proxy+}'
          method: any
    environment:
     NODE_ENV: development
```

12. Listo llegó la hora de Desplegar

```bash
serverless deploy
```
13. Si el comando anterior es exitoso verás un mensaje en la consola con la configuración creada, copia el url del ***endpoints***, debe ser algo similar a:

```bash
endpoints:
  ANY - https://kynsv88n72.execute-api.us-east-1.amazonaws.com/dev/people/{proxy+}
```
:speech_balloon: **Nota:** Usa ese valor del url para hacer tus pruebas.

19. Ahora podemos usar este URL en ***Postman*** y prueba las siguientes tramas:
:speech_balloon: **Nota:** Recuerda sustituir el el texto tu_url_aqui por el url que visualizas en el paso 18.

- GET tu_url_aqui/dev/people/100
- GET tu_url_aqui/dev/people?eyeColor=green&gender=male
- POST tu_url_aqui/dev/people/

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

Si hicimos los pasos correctamente entonces las solicitudes serán exitosas :smiley:

### Eliminar los recursos desplegados por serverless

Para eliminar los recursos desplegados (Lambda, API Gateway y Bucket S3), usa el siguiente comando del cli de serverless:

```bash
serverless remove
```

### Desplegar el Api usando un enfoque tradicional con EC2

Esta sección corresponde al *Objetivo Especifico 2*: Desplegar el proyecto implementado hasta el [Episodio 3](README-episode-3.MD) usando un enfoque más tradicional a través de [AWS EC2](https://docs.aws.amazon.com/es_es/AWSEC2/latest/UserGuide/concepts.html)

EC2 es el servicio de maquinas virtuales de AWS.

#### Crear la máquina virtual en EC2

1. Navega al [Dashboard de EC2](https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Home:)
2. Haz clic en el botón naranja ***Launch Instance*** y vuelve a seleccionar Launch Instance
3. Selecciona la opción ***Amazon Linux 2 AMI (HVM), SSD Volume Type*** haciendo clic en el botón ***Select***
4. Deja la opción por defecto en Instance Type (t2.micro)
5. Haz clic en el botón ***Next:Configure Instance Details***
6. Haz clic en el botón ***Next:Add Storage***
7. Haz clic en el botón ***Next:Add Tags***
8. Haz clic en el botón ***Next:Configure Security Group***
9. Asegurate de que en este paso este seleccionada la opción ***Create a new security group***
10. Escribe un mombre para el nuevo security group, por ejemplo sgApiRest
11. Opcionalmente puedes escribir una descripción para que se te haga facil reconocer su configuración, por ejemplo: Grupo de seguridad para habilitar acceso ssh y http
12. Haz clic en Add Rule para agregarle permisos al puerto http 80 desde cualquier ip de internet.
13. En el dropdownlist de Type selecciona ***HTTP***
14. En Source elije ***Anywhere***
15. Haz clic en el botón ***Review and Launch***
16. Clic en ***Launch***
17. Aparecerá un pop up en donde debes seleccionar ***Create a new key pair***, ponle un nombre y haz clic en el botón Download Key Pair. **Importante:** Si no descargas el archivo PEM no podrás conectarte via SSH.
18. Haz clic en el botón ***View Instances***
19. Esperar que el valor del campo Instance State esté en running.

#### Conectarse via SSH a la Instancia

:speech_balloon: **Importante:** Para conectarse via SSH depende del Sistema Operativo que utilices, si usas Linux o Mac puedes hacerlo usando el terminal y el comando ssh. En caso de Windows puedes usar la aplicación [Putty](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html).

Para más información pude visitar el link https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstances.html?icmpid=docs_ec2_console

En mi caso uso Linux por lo que usaré los pasos para Linux.

1. Desde la [lista de Instancias](https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Instances:) haz clic en el botón Connect que aparece en la parte superior.
2. Desde alli verás algunas instrucciones que te ayudarán a conectarte.
3. Si estas en Linux abre desde el terminal la carpeta donde descargaste el archivo de certificado **.pem**
4. Fijate que desde la ventana modal que se abrió en el paso 3 te indica el nombre de tu arhivo de certificados, ejeuta el comando que te indica, por ejemplo:

```bash
chmod 400 my-pem.pem
```

:speech_balloon: **Nota:** El comando *chmod 400 archivo* asegura que el archivo tenga permisos de lectura para el usuario propietario, es un requisito de seguridad para ssh.

5. Copia en comando que te aparece en la ventana modal en la sección de ***Example***: por ejemplo:

```bash
ssh -i "my-pem.pem" ec2-user@ec2-78-87-208-99.compute-1.amazonaws.com
```

6. Escribe ***yes***

:speech_balloon: **Nota:** Si ejecutaste correctamente todos los pasos despues de escribir *yes* en el paso 6, ya te encuentras conectado a la instancia.

#### Configurar el software necesario en la Instancia

1. Ejecuta el comando

```bash
sudo yum update -y
```

2. Instalemos ***Git*** con el comando:

```bash
sudo yum install git
```

3. Instalar ***node.js 12.x***

```bash
sudo yum install -y gcc-c++ make
```

```bash
curl -sL https://rpm.nodesource.com/setup_12.x | sudo -E bash -
```

```bash
sudo yum install -y nodejs
```

4. Comprueba la versiones de node y npm

```bash
node -v
```

```bash
npm -v
```

#### Descargar y configura el código fuente

1. Clonemos el repositorio que contiene el último codigod del ***Episodio 3**, si no lo tienes a la mano usa el siguiente:

```bash
git clone --single-branch --branch episodio3 https://github.com/jriverox/workshop-node.git
```

2. Ingresa a la carpeta del codigo

```bash
cd workshop-node
```

3. Instalar las dependencias

```bash
cd npm i
```

4. Edita el archivo env.yaml deberás cambiar el puerto (***PORT***) a 80 y el *usuario* y la *contraseña* por credenciales validas o por una cadena de conexión valida.

```bash
vim env.yaml
```

5. Presiona la tecla insert
6. Modifica el archivo
7. Para salir y guardar, presiona la tecla ***Escape*** y luego ***:***
8. Posteriormente escribe ***wq***
9. Finalmente para probar levanta la aplicación:

```bash
sudo npm start
```

10. Para probar necesitarás copiar desde la consola de AWS EC2, seleccionando la instancia que estas ejecutando para ese ejercicio y desde la pestaña Description copia el valor del campo ***Public DNS (IPv4)****, utilizalo para acceder desde POSTMAN, por ejemplo haz un GET ec2-100-25-157-122.compute-1.amazonaws.com/person/100

### Eliminar la instancia de EC2

Una vez que hayas realizado el ejercicio asegurate de terminar la instancia o al menos detenerla.

1. Desde la consola de [AWS EC2](https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Instances:)
2. Selecciona la instancia (asegurate de que el check este seleccionado)
3. Haz clic en el botón ***Actions***
4. Selecciona la opción Instance State y luego Terminate

## Links de Interés

### Otros Comandos Útiles de Serverless Framework

- [serverless package](https://www.serverless.com/framework/docs/providers/aws/cli-reference/package/) Este comando te permite generar el epaquetado (.zip) del Lambda manualmente.
- [serverless invoke](https://www.serverless.com/framework/docs/providers/aws/cli-reference/invoke/) Permite incokar a la función Lambda, pero debes considerar que debes pasarle el objeto esperado en el paraetro event.
- [serverless invoke local](https://www.serverless.com/framework/docs/providers/aws/cli-reference/invoke-local/) Permite incokar a la función Lambda, pero debes considerar que debes pasarle el objeto esperado en el paraetro event.

### Alternativas a Serverless Framework en AWS

- [AWS Serverless Application Model](https://aws.amazon.com/serverless/sam/) Se trata de un cli similar al serverless framework que nos permite gestionar los Lambdas.
- [CloudFormation](https://docs.aws.amazon.com/es_es/AWSCloudFormation/latest/UserGuide/Welcome.html) Es un servicio de AWS que nos permite la automatización de la gestion de recursos en AWS, por ejemplo podriamos crear un stack que via cli podamos crear y configurar varios recursos.
