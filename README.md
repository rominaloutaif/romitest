# calendar-backend

## Resumen:
 
Calendar-backend permite leer, mostrar y modificar eventos dentro de la Universidad.

## API Doc
Para ver el uso de las APIs usamos [swagger](https://swagger.io/tools/swagger-ui/)
este funciona también como cliente.

[Swagger:](http://localhost:[port_number]/swagger-ui.html) `http://localhost:[port_number]/swagger-ui.html`


### Autenticación en Docker

Para autenticarse en el registry de imágenes de docker de gitlab, seguir los siguientes pasos:

1- Obtener el personal access token en GitLab

2- Correr:

`docker login hub.ues21.edu.ar`

3- Ingresar el usuario de Gitlab y el token como password

4- Correr:

`docker pull hub.ues21.edu.ar/algarrobo/calendar-backend/calendar-backend:latest`

### Docker Compose

Creamos un docker-compose para poder fácilmente levantar la aplicación

Para: levantar el compose:

`docker-compose up --build -d`

para detener los containers:

`docker-compose stop`

### Contruir y correr el container de docker manualmente


Para construir una imagen local de calendar-backend: 

`docker build --network bridge -t calendar-backend .`

Finalmente, para levantarlo, solo hay que especificar el puerto del contenedor. i.e:

`docker run -d -p 8081:8080 calendar-backend`

Para detener el contenedor:

`docker stop <container_id>`


### CI / CD

Contamos un Pipeline de Integración continua provisto por el mismo [Gitlab](https://code.ues21.edu.ar/algarrobo/calendar-backend),
 en donde hosteamos el código fuente. El mismo se puede ver en: `https://code.ues21.edu.ar/algarrobo/calendar-backend/pipelines`
 
 los pasos del pipeline son: (ver [.gitlab-ci.yml](./.gitlab-ci.yml))
 *  Build: el código se compila
 *  Static Analisys: El código se inspecciona estáticamente usando un servidor de [sonarqube](https://www.sonarqube.org/), [AQUI](https://sonarqube.uesiglo21.edu.ar/dashboard?id=algarroboProject20)
 *  Tests Unitarios: Se corren los tests unitarios
 *  Test funcionales: se corren los tests de funcionales
 *  Dockerize: Se genera una nueva imagen de Docker. (ver [Dockerfile](./Dockerfile))
 *  Deploy: se despliega el container en el servidor de Kubernetes.
 *  Deploy stable: se despliega el container en el entorno stable de AWS (Sólo mediante la creación de un tag)
 
### Correr sonar localmente

Para hacer un analisis estático de código usando sonarquebe localmente, se puede hacer lo siguiente:
`docker run -d --name sonarqube -p 9000:9000 sonarqube`

`gradle sonar`

### Tests de Integracion

Usamos [Behave](https://behave.readthedocs.io/en/latest/) para los test de integración.

### Tests Funcionales

Usamos [Behave](https://behave.readthedocs.io/en/latest/) para los test funcionales.
Para correr los tests funcionales se usa una BD en memoria ([h2](https://www.h2database.com/html/main.html)).

Se puede levantar el entorno local para los tests funcionales con los siguientes comandos:

`docker-compose -f func-test-compose.yml up --build`

`cd functional-test`

`behave`

### Detalles técnicos

*   Utilizamos las buenas prácticas de ["12 factors"](https://12factor.net/es/) para aplicaciones en la nube
*   Los requests mandan [correlation ids](https://blog.rapid7.com/2016/12/23/the-value-of-correlation-ids/), el header usado es `X-Correlation-Id`
*   Nos basamos en la [guia de diseño de APIs de google](https://cloud.google.com/apis/design/), para diseñar y crear las APIs
*   Se utiliza este estandar de [calendar](https://tools.ietf.org/id/draft-ietf-calext-jscalendar-10.html)
*   [spring-boot 2.1.3](https://spring.io/projects/spring-boot) como web framework de desarrollo
*   [Gradle](https://gradle.org/) como herramienta de "build and automation"
*   Aquellos responses cuyo payload supere 1KB sera enviados [comprimidos](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Encoding)
*   Soporte de [E-tag](https://developer.mozilla.org/es/docs/Web/HTTP/Headers/ETag) para los recursos
*   La applicación esta diseñada para ser fácilmente [dockerizada](https://www.docker.com/why-docker)
