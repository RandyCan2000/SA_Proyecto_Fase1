# SA Proyecto
## **Documentación Tecnica YoVotoApp**
### **Descipción general**
> YoVotoAPP es una aplicacion destinada para realizar votaciones desde cualquier lugar del mundo desde la comodidad de un dispositivo movil.

___
## **Arquitectura y diseño de la solución**

### ***Modelo Entidad Relacion YoVotoApp***
<div align="center">

![ER](./assets/ER_SA_PROYECTO1%20-%20ER.png)

</div>

### ***Infraestructura YoVotoApp***

<div align="center">

![ER](./assets/ER_SA_PROYECTO1%20-%20Infraestructura.png)

</div>

___
### **Comunicación entre los servicios**
> la comunicación entre los servidores sera completamente REST hacia cualquier microservicio diseñado para el aplicativo de YoVoto, para poder centralizar el lugar a donde se realizan las peticiones se planteo la solucion de utilizar un middleware encargado de redireccionar el trafico a las correspondientes servicios desarrollados en los lenguajes que nos ofrescan la mejor escalabilidad y estabilidad.
___
## **Diagrama de actividades del microservicio de autenticación**

### ***Login de usuario***
> Esta diagrama hace referencia al login de un usuario al sistema de YoVoto.

<div align="center">

![Auth_Login](./assets/Auth/ER_SA_PROYECTO1%20-%20Diagrama_Micro_Auth_login.png)

</div>

### ***Generacion Codigo Seguridad***
> Este diagrama hace referencia al servicio que genera un codigo de seguridad y lo envia a el celular del usuario.

<div align="center">

![Auth_GenCode](./assets/Auth/ER_SA_PROYECTO1%20-%20Diagrama_Micro_Auth_GenCode.png)

</div>

### ***Validación Codigo Seguridad***
> Este diagrama hace referencia al servicio que valida un codigo de seguridad y lo envia a el celular del usuario.

<div align="center">

![Auth_ValCode](./assets/Auth/ER_SA_PROYECTO1%20-%20Diagrama_Micro_Auth_ValCode.png)

</div>

___
## **Diagrama de actividades de emisión de voto**

### ***Obtener Elecciones Activas***
> Este diagrama hace referencia al servicio que retorna las elecciones activas.

<div align="center">

![Voto_GetEleccion](./assets/Voto/ER_SA_PROYECTO1%20-%20Diagrama_Micro_Voto_AllElecciones.png)

</div>

### ***Obtener Opciones Elecciones Activas***
> Este diagrama hace referencia al servicio que retorna las opciones de las elecciones activas.

<div align="center">

![Voto_GetOpcionesEleccion](./assets/Voto/ER_SA_PROYECTO1%20-%20Diagrama_Micro_Voto_GetOpciones.png)

</div>

### ***Agregar voto***
> Este diagrama hace referencia al servicio que agrega los votos a las elecciones correspondientes.

<div align="center">

![Voto_AddVoto](./assets/Voto/ER_SA_PROYECTO1%20-%20Diagrama_Micro_Voto_AddVoto.png)

</div>

### ***Votos En Tiempo Real***
> Este diagrama hace referencia al servicio que retorna la cantidad de votos en tiempo real.

<div align="center">

![Voto_TimeRealVoto](./assets/Voto/ER_SA_PROYECTO1%20-%20Diagrama_Micro_Voto_TimeReal.png)

</div>

### ***Finalizar eleccion***
> Este diagrama hace referencia al servicio que guarda los resultados de las votaciones al finalizar.

<div align="center">

![Voto_FinishEleccion](./assets/Voto/ER_SA_PROYECTO1%20-%20Diagrama_Micro_Voto_finishEleccion.png)

</div>

___
## **Diagrama de actividades del sistema de registro (enrollamiento) de ciudadanos**

### ***Registrar Usuario***
> Este diagrama hace referencia al servicio que guarda compara la informacion del usuario nuevo y la almacena en base de datos.

<div align="center">

![Registro_AddUser](./assets/Registro/ER_SA_PROYECTO1%20-%20Diagrama_Micro_Registro_AddUsuario.png)

</div>

### ***Obtener Informacion***
> Este diagrama hace referencia al servicio que retorna la informacion del usuario logueado.

<div align="center">

![Registro_GetInfo](./assets/Registro/ER_SA_PROYECTO1%20-%20Diagrama_Micro_Registro_getInfoUser.png)

</div>

### ***Actualizar Usuario***
> Este diagrama hace referencia al servicio que actualiza la informacion del usuario.

<div align="center">

![Registro_UpdateInfo](./assets/Registro/ER_SA_PROYECTO1%20-%20Diagrama_Micro_Registro_Update.png)

</div>

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
  statusCode: number,
  message: string,
  token: string,
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

___
### **Microservicio Registro (/Registry)**

#### ***/addUser***

Microservicio destinado a guardar la informacion de un usuario nuevo que desee utilizar la aplicacion YoVotoApp.

***Request***
```yml
#headers
{
  token: string
}
#body
{
  newUser: userData,
  password: string
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
| newUser        |modelo con la informacion del usuario nuevo|
| password        |contraseña que utilizar el usuario nuevo para poder ingresar a YoVotoApp|

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

#### ***/addElection***

Microservicio destinado a guardar la informacion de una eleccion nuevo que desee realizarce mediante la aplicacion YoVotoApp.

***Request***
```yml
#headers
{
  token: string
}
#body
{
  newElection: election
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
| newElection        |modelo con la informacion de la nueva elección|

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

#### ***/getUserInfo***

Microservicio destinado a retornar la información del usuario que utiliza YoVotoApp.

***Request***
```yml
#headers
{
  token: string
}
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
  message: string,
  user: userData
}
```

### *Parametros Entrada*

<div align="center">

|Parametro          |Detalle|
|:-------------------:|:---------------:|
| username        |numero de DPI del usuario que se desea obtener la informacion|

</div>

### *Parametros Salida*

<div align="center">

|Parametro          |Detalle|
|:-------------------:|:---------------:|
| StatusCode        |Retorna un valor numero que representa si la transaccion se realizo con exito| 
| message           |Retorna un texto dependiendo el codigo de status Code|
|user               |Retorna un json con la informacion almacenado en base de datos de YoVotoApp|

</div>

<br>

### *Codigos Error*
<div align="center">

|Parametro          |Valor        |Detalle        |
|:-------------------:|:-------------:|:---------------:|
| StatusCode        | 200         |Valor de exito |
| StatusCode        |400         |Valor de error |

</div>

#### ***/updateUser***

Microservicio destinado para la actualización de la información del usuario logueado en la aplicacion YoVotoApp.

***Request***
```yml
#headers
{
  token: string
}
#body
{
  userUpdate: userData
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
| userUpdate        |modelo con la informacion del usuario para guardar en base de datos|

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

___
### **Microservicio Votaciones (/Votes)**

#### ***/allElectionsActive***

Microservicio destinado a retornar todas las elecciones activas dentro de la aplicacion YoVotoApp.

***Request***
```yml
#headers
{
  token: string
}
#body
{}
```

***Response***

```yml
#headers
{}
#body
{
  statusCode: number,
  message: string,
  elections: election[]
}
```

### *Parametros Entrada*

<div align="center">

|Parametro          |Detalle|
|:-------------------:|:---------------:|
| | |

</div>

### *Parametros Salida*

<div align="center">

|Parametro          |Detalle|
|:-------------------:|:---------------:|
| StatusCode        |Retorna un valor numero que representa si la transaccion se realizo con exito| 
| message           |Retorna un texto dependiendo el codigo de status Code|
|elections          |Retorna un arreglo con las elecciones activas|

</div>

<br>

### *Codigos Error*
<div align="center">

|Parametro          |Valor        |Detalle        |
|:-------------------:|:-------------:|:---------------:|
| StatusCode        | 200         |Valor de exito |
| StatusCode        |400         |Valor de error |

</div>

#### ***/allElections***

Microservicio destinado a retornar todas las elecciones activas y no activas dentro de la aplicacion YoVotoApp.

***Request***
```yml
#headers
{
  token: string
}
#body
{}
```

***Response***

```yml
#headers
{}
#body
{
  statusCode: number,
  message: string,
  elections: election[]
}
```

### *Parametros Entrada*

<div align="center">

|Parametro          |Detalle|
|:-------------------:|:---------------:|
| | |

</div>

### *Parametros Salida*

<div align="center">

|Parametro          |Detalle|
|:-------------------:|:---------------:|
| StatusCode        |Retorna un valor numero que representa si la transaccion se realizo con exito| 
| message           |Retorna un texto dependiendo el codigo de status Code|
|elections          |Retorna un arreglo con las elecciones activas y no activas|

</div>

<br>

### *Codigos Error*
<div align="center">

|Parametro          |Valor        |Detalle        |
|:-------------------:|:-------------:|:---------------:|
| StatusCode        | 200         |Valor de exito |
| StatusCode        |400         |Valor de error |

</div>

#### ***/getOptions***

Microservicio destinado a retornar las opciones de una eleccion en especifico.

***Request***
```yml
#headers
{
  token: string
}
#body
{
  idElection: number
}
```

***Response***

```yml
#headers
{}
#body
{
  statusCode: number,
  message: string,
  options: option[]
}
```

### *Parametros Entrada*

<div align="center">

|Parametro          |Detalle|
|:-------------------:|:---------------:|
|idElection |id de la eleccion que se desean saber sus opciones|

</div>

### *Parametros Salida*

<div align="center">

|Parametro          |Detalle|
|:-------------------:|:---------------:|
| StatusCode        |Retorna un valor numero que representa si la transaccion se realizo con exito| 
| message           |Retorna un texto dependiendo el codigo de status Code|
|options            |Retorna un arreglo con las opciones de la eleccion correspondiente|

</div>

<br>

### *Codigos Error*
<div align="center">

|Parametro          |Valor        |Detalle        |
|:-------------------:|:-------------:|:---------------:|
| StatusCode        | 200         |Valor de exito |
| StatusCode        |400         |Valor de error |

</div>

#### ***/addVote***

Microservicio destinado a guardar el voto de un usuario para una eleccion en YoVotoApp.

***Request***
```yml
#headers
{
  token: string
}
#body
{
  idElection: number,
  idUser?: number,
  idOption: number,
  date: date 
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
|idElection |id de la eleccion a la que se votara|
|idUser |id del usuario, es un valor opcional|
|idOption |id de la opcion que escogio el usuario para votar|
|date |fecha y hora que se realizo el voto|

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

#### ***/votesTimeReal***

Microservicio destinado que retorna la cantidad de votos por opcion de una eleccion en tiempo real.

***Request***
```yml
#headers
{
  token: string
}
#body
{
  idElection: number
}
```

***Response***

```yml
#headers
{}
#body
{
  statusCode: number,
  message: string,
  votes: votesResult[]
}
```

### *Parametros Entrada*

<div align="center">

|Parametro          |Detalle|
|:-------------------:|:---------------:|
|idElection |id de la eleccion a la que se votara|

</div>

### *Parametros Salida*

<div align="center">

|Parametro          |Detalle|
|:-------------------:|:---------------:|
| StatusCode        |Retorna un valor numero que representa si la transaccion se realizo con exito| 
| message           |Retorna un texto dependiendo el codigo de status Code|
|votes              |Retorna un arreglo con las opciones de la eleccion y su cantidad de votos en el momento|

</div>

<br>

### *Codigos Error*
<div align="center">

|Parametro          |Valor        |Detalle        |
|:-------------------:|:-------------:|:---------------:|
| StatusCode        | 200         |Valor de exito |
| StatusCode        |400         |Valor de error |

</div>

#### ***/finishElection***

Microservicio destinado a concluir y guardar en base de datos los resultados finales de una votación.

***Request***
```yml
#headers
{
  token: string
}
#body
{
  idElection: number
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
|idElection |id de la eleccion a la que se votara|

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

___
## **Modelos**

### *userData*

```yml
{
  username: string,
  name: string,
  phone: string,
  img: string,
  dpiImg: string,
  email: string,
  country: string,
  city: string,
  idRol: number
}
```

### *election*

```yml
{
  id: number,
  title: string,
  description: string,
  dateStart: date,
  dateFinish: date,
  options: option[]
}
```

### *option*

```yml
{
  title: string,
  img: string,
  metaData: metaData[]
}
```

### *metaData*
```yml
{
  clave: string,
  valor: string
}
```

### *votesResult*
```yml
{
  option: option,
  result: number
}
```

___
## **Json File Renap**

```yml
{
  users: [
    {
      username: int,
      name: string,
      phone: string,
      img: string,
      dpiImg: string,
      email: string,
      country: string,
      city: string
    }
  ]
}
```

### ***Ejemplo***
```yml
{
  users: [
    {
      username: 3005122870101,
      name: "Randy Alexander Can Ajuchan",
      phone: "54875373",
      img: "http://img.jpg",
      dpiImg: "http://img2.jpg",
      email: "canalex210@gmail.com",
      country: "Guatemala",
      city: "Guatemala"
    }
    .
    .
    .
  ]
}
```