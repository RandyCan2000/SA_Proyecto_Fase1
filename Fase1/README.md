# SA Proyecto
## **Documentación Tecnica YoVotoApp**
### **Descipción general**
> YoVotoAPP es una aplicacion destinada para realizar votaciones desde cualquier lugar del mundo desde la comodidad de un dispositivo movil.

___
## **Arquitectura y diseño de la solución**

### ***Modelo Entidad Relacion YoVotoApp***
![ER](./assets/ER_SA_PROYECTO1%20-%20ER.png)

### ***Infraestructura YoVotoApp***
![ER](./assets/ER_SA_PROYECTO1%20-%20Infraestructura.png)

___
### **Comunicación entre los servicios**
> la comunicación entre los servidores sera completamente REST hacia cualquier microservicio diseñado para el aplicativo de YoVoto, para poder centralizar el lugar a donde se realizan las peticiones se planteo la solucion de utilizar un middleware encargado de redireccionar el trafico a las correspondientes servicios desarrollados en los lenguajes que nos ofrescan la mejor escalabilidad y estabilidad.
___
## **Diagrama de actividades del microservicio de autenticación**

### ***Login de usuario***
> Esta diagrama hace referencia al login de un usuario al sistema de YoVoto.

![Auth_Login](./assets/Auth/ER_SA_PROYECTO1%20-%20Diagrama_Micro_Auth_login.png)

### ***Generacion Codigo Seguridad***
> Este diagrama hace referencia al servicio que genera un codigo de seguridad y lo envia a el celular del usuario.

![Auth_GenCode](./assets/Auth/ER_SA_PROYECTO1%20-%20Diagrama_Micro_Auth_GenCode.png)

### ***Validación Codigo Seguridad***
> Este diagrama hace referencia al servicio que valida un codigo de seguridad y lo envia a el celular del usuario.

![Auth_ValCode](./assets/Auth/ER_SA_PROYECTO1%20-%20Diagrama_Micro_Auth_ValCode.png)

___
## **Diagrama de actividades de emisión de voto**

### ***Obtener Elecciones Activas***
> Este diagrama hace referencia al servicio que retorna las elecciones activas.

![Voto_GetEleccion](./assets/Voto/ER_SA_PROYECTO1%20-%20Diagrama_Micro_Voto_AllElecciones.png)

### ***Obtener Opciones Elecciones Activas***
> Este diagrama hace referencia al servicio que retorna las opciones de las elecciones activas.

![Voto_GetOpcionesEleccion](./assets/Voto/ER_SA_PROYECTO1%20-%20Diagrama_Micro_Voto_GetOpciones.png)

### ***Agregar voto***
> Este diagrama hace referencia al servicio que agrega los votos a las elecciones correspondientes.

![Voto_AddVoto](./assets/Voto/ER_SA_PROYECTO1%20-%20Diagrama_Micro_Voto_AddVoto.png)

### ***Votos En Tiempo Real***
> Este diagrama hace referencia al servicio que retorna la cantidad de votos en tiempo real.

![Voto_TimeRealVoto](./assets/Voto/ER_SA_PROYECTO1%20-%20Diagrama_Micro_Voto_TimeReal.png)

### ***Finalizar eleccion***
> Este diagrama hace referencia al servicio que guarda los resultados de las votaciones al finalizar.

![Voto_FinishEleccion](./assets/Voto/ER_SA_PROYECTO1%20-%20Diagrama_Micro_Voto_finishEleccion.png)

___
## **Diagrama de actividades del sistema de registro (enrollamiento) de ciudadanos**

### ***Registrar Usuario***
> Este diagrama hace referencia al servicio que guarda compara la informacion del usuario nuevo y la almacena en base de datos.

![Registro_AddUser](./assets/Registro/ER_SA_PROYECTO1%20-%20Diagrama_Micro_Registro_AddUsuario.png)

### ***Obtener Informacion***
> Este diagrama hace referencia al servicio que retorna la informacion del usuario logueado.

![Registro_GetInfo](./assets/Registro/ER_SA_PROYECTO1%20-%20Diagrama_Micro_Registro_getInfoUser.png)

### ***Actualizar Usuario***
> Este diagrama hace referencia al servicio que actualiza la informacion del usuario.

![Registro_UpdateInfo](./assets/Registro/ER_SA_PROYECTO1%20-%20Diagrama_Micro_Registro_Update.png)

___
## **Descripción de la seguridad de la aplicación**

> La seguridad de la aplicacion se realizara mediante la utilizacion de JWT para poder validar la identidad del usuario logueado, tambien para reforzar la seguridad y no ser victimas de la usurpasion de identidad se utilizara un sistema de 2FA el cual generara un codigo aleatorio el cual sera enviado al telefono asociado al usuario para que este lo ingrese al momento de realizar el inicio de sesion y pueda obtener el token generado por el sistema y poder utilizar el sistema de YoVoto con seguridad.

### ***Twillio***
> Twilio es una compañía que ofrece una plataforma de comunicaciones así como de servicios en la nube (CPaaS), ubicada en San Francisco, California. Twilio permite desarrollar aplicaciones que hagan y reciban llamadas, mensajes de texto, elaboren funciones de comunicación y registro, usando APIs, propias del servicio web.

___
## **Pipelines para los servicios**
```
variables:
  GIT_STRATEGY: clone

stages:          # List of stages for jobs, and their order of execution
  - server-dev-testing
  - integration-testing
  - test&build
  - configure
  - check
  - deploy

configure-runner-deploy:
  stage: server-dev-testing
  tags:
    - ansible
  script:
    - "pwd"
    - "sudo docker-compose -f ./docker-compose-dev.yaml down"
    - "sudo docker-compose -f ./docker-compose-dev.yaml build"
    - "sudo docker-compose -f ./docker-compose-dev.yaml up -d"
  only:
    - develop

test&build-server-middleware:    #port 5000
  image: docker:20
  stage: test&build
  services:
    - docker:dind
  variables:
    IMAGE_TAG: $CI_REGISTRY_IMAGE/SA/middleware:$CI_COMMIT_REF_SLUG
  before_script:
    - echo $CI_BUILD_TOKEN | docker login -u "$CI_REGISTRY_USER" --password-stdin $CI_REGISTRY
  script:
    - docker build -t $IMAGE_TAG ./Backend/middleware
    - docker push $IMAGE_TAG
  only:
    - develop

test&build-server-voto:    #port 5001
  image: docker:20
  stage: test&build
  services:
    - docker:dind
  variables:
    IMAGE_TAG: $CI_REGISTRY_IMAGE/SA/api-voto:$CI_COMMIT_REF_SLUG
  before_script:
    - echo $CI_BUILD_TOKEN | docker login -u "$CI_REGISTRY_USER" --password-stdin $CI_REGISTRY
  script:
    - docker build -t $IMAGE_TAG ./Backend/api-voto
    - docker push $IMAGE_TAG
  only:
    - develop

test&build-server-registro:    #port 5002
  image: docker:20
  stage: test&build
  services:
    - docker:dind
  variables:
    IMAGE_TAG: $CI_REGISTRY_IMAGE/SA/api-registro:$CI_COMMIT_REF_SLUG
  before_script:
    - echo $CI_BUILD_TOKEN | docker login -u "$CI_REGISTRY_USER" --password-stdin $CI_REGISTRY
  script:
    - docker build -t $IMAGE_TAG ./Backend/api-registro
    - docker push $IMAGE_TAG
  only:
    - develop

test&build-server-Auth:    #port 5003
  image: docker:20
  stage: test&build
  services:
    - docker:dind
  variables:
    IMAGE_TAG: $CI_REGISTRY_IMAGE/SA/api-auth:$CI_COMMIT_REF_SLUG
  before_script:
    - echo $CI_BUILD_TOKEN | docker login -u "$CI_REGISTRY_USER" --password-stdin $CI_REGISTRY
  script:
    - docker build -t $IMAGE_TAG ./Backend/api-Auth
    - docker push $IMAGE_TAG
  only:
    - develop

test&build-server-twilio:    #port 5008
  image: docker:20
  stage: test&build
  services:
    - docker:dind
  variables:
    IMAGE_TAG: $CI_REGISTRY_IMAGE/SA/api-twilio:$CI_COMMIT_REF_SLUG
  before_script:
    - echo $CI_BUILD_TOKEN | docker login -u "$CI_REGISTRY_USER" --password-stdin $CI_REGISTRY
  script:
    - docker build --build-arg API_PORT=5008 -t $IMAGE_TAG ./Backend/api-twilio
    - docker push $IMAGE_TAG
  only:
    - develop

test&build-server-frontend:    #port 80-3000
  image: docker:20
  stage: test&build
  services:
    - docker:dind
  variables:
    IMAGE_TAG: $CI_REGISTRY_IMAGE/SA/frontend:$CI_COMMIT_REF_SLUG
  before_script:
    - echo $CI_BUILD_TOKEN | docker login -u "$CI_REGISTRY_USER" --password-stdin $CI_REGISTRY
  script:
    - docker build -t $IMAGE_TAG ./Frontend/aydiscoweb
    - docker push $IMAGE_TAG
  only:
    - develop

test-selenium-grid:
  stage: integration-testing
  image: node:16-alpine
  services:
  - name: selenium/standalone-firefox
    alias: selenium-grid
  script:
    - cd Selenium/Grid
    - npm install
    - npm run start
    - echo "Pruebas de integración completadas :D"
  only:
    - feature/selenium-grid-flujo
    - feature/pruebas-integracion
    - develop

configure-runner:
  stage: configure
  tags:
    - ansible
  script:
    - "sudo apt update"
    - "sudo apt install software-properties-common"
    - "sudo add-apt-repository --yes --update ppa:ansible/ansible"
    - "sudo apt install ansible -y"
  only:
    - master

check-configure:
  stage: check
  tags:
    - ansible
  script:
    - "ansible --version"
    - "git checkout master"
    - "git pull"
    - "git branch"
    - "sudo chmod 600 id_rsa"
    - "sudo ansible -m ping all"
  only:
    - master

deploy-project:
  stage: deploy
  tags:
    - ansible
  script:
    - "sudo chmod 600 id_rsa"
    - "git checkout master"
    - "ansible-playbook playbook.yml --limit=production"
  only:
    - master
```

___
## **Contratos**

### **Microservicio autenticación (/Auth)**

#### ***/Login***

Microservicio destinado a validar contraseña y rol del usuario que desea ingresar a YoVotoAPP.

***Request***
```yml
#headers
{}
#body
{
  username: string,
  password: string,
  rol: string
}
```
***Response***
```yml
#headers
{}
#body
{
  statusCode: number,
  message: string
}
```
### *Parametros Entrada*

<div align="center">

|Parametro          |Detalle|
|:-------------------:|:---------------:|
| username        |DPI del usuario| 
| password           |Contraseña del usuario|
| rol           |Rol del usuario|

</div>

### *Parametros Salida*

<div align="center">

|Parametro          |Detalle|
|:-------------------:|:---------------:|
| StatusCode        |Retorna un valor numero que representa si la transaccion se realizo con exito| 
| message           |Retorna un texto dependiendo el codigo de status Code|

</div>

<br>

### *Codigos Error*
<div align="center">

|Parametro          |Valor        |Detalle        |
|:-------------------:|:-------------:|:---------------:|
| StatusCode        | 200         |Valor de exito |
| StatusCode        |400         |Valor de error |

</div>

#### ***/generateSecureCode***

Microservicio destinado a generar un codigo de seguridad y enviado al telefono celular del usuario.

***Request***
```yml
#headers
{}
#body
{
  username: string
}
```
***Response***
```yml
#headers
{}
#body
{
  statusCode: number,
  message: string
}
```
### *Parametros Entrada*

<div align="center">

|Parametro          |Detalle|
|:-------------------:|:---------------:|
| username        |DPI del usuario|

</div>

### *Parametros Salida*

<div align="center">

|Parametro          |Detalle|
|:-------------------:|:---------------:|
| StatusCode        |Retorna un valor numero que representa si la transaccion se realizo con exito| 
| message           |Retorna un texto dependiendo el codigo de status Code|

</div>

<br>

### *Codigos Error*
<div align="center">

|Parametro          |Valor        |Detalle        |
|:-------------------:|:-------------:|:---------------:|
| StatusCode        | 200         |Valor de exito |
| StatusCode        |400         |Valor de error |

</div>

#### ***/validateSecureCode***

Microservicio destinado a validar un codigo de seguridad y enviado al telefono celular del usuario.

***Request***
```yml
#headers
{}
#body
{
  username: string,
  codeSecure: string
}
```

***Response***

```yml
#headers
{}
#body
{
  statusCode: number
  message: string
  token: string
  userData: userData
}
```

### *Parametros Entrada*

<div align="center">

|Parametro          |Detalle|
|:-------------------:|:---------------:|
| username        |DPI del usuario|
| codeSecure        |codigo de seguridad enviado al telefono del usuario|

</div>

### *Parametros Salida*

<div align="center">

|Parametro          |Detalle|
|:-------------------:|:---------------:|
| StatusCode        |Retorna un valor numero que representa si la transaccion se realizo con exito| 
| message           |Retorna un texto dependiendo el codigo de status Code|
| Token             |Retorna un codigo unico con tiempo de vida limitado para poder hacer uso de todas las funcionalidad des de YoVotoApp|
|userData|Json con informacion del usuario|

</div>

<br>

### *Codigos Error*
<div align="center">

|Parametro          |Valor        |Detalle        |
|:-------------------:|:-------------:|:---------------:|
| StatusCode        | 200         |Valor de exito |
| StatusCode        |400         |Valor de error |

</div>

## **Modelos**

### *userData*

```yml
{
  username: string,
  name: string,
  phone: string,
  img: string,
  email: string,
  country: string,
  city: string
}
```