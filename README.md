# Creación de aplicación web en contenedores con Docker

Bienvenidos a la guía para la creación de aplicaciones webs en Azure Cointainers con Docker, este resumen practico esta hecho gracias a la documentación oficial de Microsoft https://learn.microsoft.com/en-us/training/modules/intro-to-containers/ Realizada para practicar para la renovación de la certificación **AZ-104** a **Marzo de 2024**.

- Conviene echarle un vistazo a la guía con comandos de Docker realizada anteriormente: https://github.com/Ramonperu/DockersFundamentals


Recordemos que para realizar todo esto hemos de tener activada la virtualización en Bios, WSL instalado en Windows y activar lo siguiente dentro de las características de Windows, si no es probable que nos genere un error al iniciar Docker Desktop:

<img align="center" src="/img/1ºimagenn.PNG"  />

## Ejercicio: Recuperar una imagen de Docker existente e implementarla localmente

En este ejercicio, extraerá una imagen de Docker Hub y la ejecutará. Examinará el estado local de Docker para ayudar a comprender los elementos que se implementan. Finalmente, eliminarás el contenedor y la imagen de tu computadora.

Iniciamos Docker Desktop, si no al ejecutar los comandos nos dará un error:

```cmd
docker pull mcr.microsoft.com/dotnet/samples:aspnetapp
```

<img align="center" src="/img/2ºimagenn.PNG"  />

Comprobamos que tenemos la imagen en local:

```cmd
docker image ls
```

<img align="center" src="/img/3ºimagenn.PNG"  />

Ejecutamos el siguiente comando:

```cmd
docker run -d -p 8080:8080 mcr.microsoft.com/dotnet/samples:aspnetapp
```

La opción *-d* es para ejecutarla como una aplicación en segundo plano y no interactiva. El indicador *-p* sirve para asignar el puerto 8080 en el contenedor creado al puerto 8080 localmente, este comando nos devuelve esto

<img align="center" src="/img/4ºimagenn.PNG"  />

Y comprobamos que se este ejecutando localmente accediendo al localhost con el puerto 8080

 http://localhost:8080  Donde se podrá ver algo asi

<img align="center" src="/img/5ºimagenn.PNG"  />

### Examinar el contenedor en el registro local de Docker

Para ver los contenedores en ejecución ejecutamos un:

```cmd
docker ps
```

<img align="center" src="/img/6ºimagenn.PNG"  />

Para pararlo hemos de hacer un:

```cmd
docker container stop jovial_cori
```

Para comprobarlo hay dos maneras, mediante comando y accediendo al local host el cual nos devolverá un error de Conexión rechazada.

Si ejecutamos el siguiente comando podremos ver en **Status** que el contendor no esta en funcionamiento:

```cmd
docker ps -a
```

<img align="center" src="/img/7ºimagenn.PNG"  />

### Eliminar el contenedor y la imagen del registro local

Para eliminarlo ejecutaremos un, tras esto lo comprobamos con el comando docker ps:

```cmd
docker container rm jovial_cori
```

<img align="center" src="/img/8ºimagenn.PNG"  />

Ahora vamos a proceder a localizar la imagen y a borrarla:

Listamos:

```cmd
docker image ls
```

<img align="center" src="/img/9ºimagenn.PNG"  />

Borramos:

```cmd
docker image rm mcr.microsoft.com/dotnet/samples:aspnetapp
```

Comprobamos que se encuentra borrado:

<img align="center" src="/img/10ºimagenn.PNG"  />



# Ejercicio: personaliza una imagen de Docker para ejecutar tu propia aplicación web

Creación de un DockerFile para la aplicación web, ejecutamos Docker Desktop, abrimos el cmd y clonamos el siguiente repositorio, tras esto accedemos a el:

```cmd
git clone https://github.com/MicrosoftDocs/mslearn-hotel-reservation-system.git
```

<img align="center" src="/img/11ºimagenn.PNG"  />

Tras esto vamos a ejecutar el siguiente comando:

```cmd
copy NUL Dockerfile
```

Después de esto vamos a crear un archivo mediante el comando 

```cmd
notepad Dockerfile
```

 Donde se introducira lo siguiente:

```
FROM mcr.microsoft.com/dotnet/core/sdk:2.2
WORKDIR /src
COPY ["/HotelReservationSystem/HotelReservationSystem.csproj", "HotelReservationSystem/"]
COPY ["/HotelReservationSystemTypes/HotelReservationSystemTypes.csproj", "HotelReservationSystemTypes/"]
RUN dotnet restore "HotelReservationSystem/HotelReservationSystem.csproj"
```

- Este código tiene comandos para recuperar una imagen que contiene el SDK de .NET Core Framework. Los archivos de proyecto para la aplicación web ( `HotelReservationSystem.csproj`) y el proyecto de biblioteca ( `HotelReservationSystemTypes.csproj`) se copian en la carpeta /src del contenedor. El `dotnet restore`comando descarga las dependencias requeridas por estos proyectos desde NuGet.

```
COPY . .
WORKDIR "/src/HotelReservationSystem"
RUN dotnet build "HotelReservationSystem.csproj" -c Release -o /app
```

- Estos comandos copian el código fuente de la aplicación web en el contenedor y luego ejecutan el comando dotnet build para compilar la aplicación. Las DLL resultantes se escriben en la carpeta /app del contenedor.

```
RUN dotnet publish "HotelReservationSystem.csproj" -c Release -o /app
```

- El `dotnet publish`comando copia los ejecutables del sitio web en una nueva carpeta y elimina los archivos provisionales. Los archivos de esta carpeta se pueden implementar en un sitio web.

```
EXPOSE 80
WORKDIR /app
ENTRYPOINT ["dotnet", "HotelReservationSystem.dll"]
```

- El primer comando abre el puerto 80 en el contenedor. El segundo comando se mueve a la `/app`carpeta que contiene la versión publicada de la aplicación web. El comando final especifica que cuando se ejecuta el contenedor debe ejecutar el comando `dotnet HotelReservationSystem.dll`. Esta biblioteca contiene el código compilado para la aplicación web.

Tras introducir todo esto en el mismo orden guardaremos el archivo.

**ATENCION** hay que guardarlo como Todos los archivos y sin extensión y dentro de la carpeta SRC dentro de `mslearn-hotel-reservation-system` para que funcione correctamente.

**Si por un casual no funcionara o no detectara el archivo DockerFile podemos ir a la carpeta a mano y darle el formato y el contenido necesario*

## Construya e implemente la imagen usando Dockerfile

Ejecutaremos el siguiente comando y comprobaremos que la imagen se haya creado con el nombre de reservation system

```
docker build -t reservationsystem .
```

<img align="center" src="/img/12ºimagenn.PNG"  />

Listamos la imagenes:

```
docker image list
```

<img align="center" src="/img/13ºimagenn.PNG"  />

Y ahora a comprobar que funcione, abrimos el puerto 8080 e iniciamos:

```
docker run -p 8080:80 -d --name reservations reservationsystem
```

<img align="center" src="/img/14ºimagenn.PNG"  />

Si variamos el link podemos ver las distintas características de cada reserva

<img align="center" src="/img/15ºimagenn.PNG"  />



Comprobamos el estado del contenedor:

<img align="center" src="/img/16ºimagenn.PNG"  />

Y detenemos y eliminamos:

<img align="center" src="/img/17ºimagenn.PNG"  />

## Ejercicio: Implementación de una imagen de Docker en una instancia de contenedor de Azure

En el ejercicio anterior, empaquetamos y probamos la aplicación web como una imagen de Docker local. Ahora deseamos usarr el resultado de ese ejercicio y hacer que la aplicación web esté disponible globalmente. Para lograr esta disponibilidad, ejecutaremos la imagen como una instancia de contenedor de Azure.

En este ejercicio,se aprenderá a reconstruir la imagen de la aplicación web y cargarla en Azure Container Registry.Se utilizará el servicio Azure Container Instance para ejecutar la imagen

Vamos a proceder a crear un contenedor desde el portal

<img align="center" src="/img/18ºimagenn.PNG"  />

Rellenamos los datos basicos con lo siguiente

| Configuración                | Valor                                                        |
| :--------------------------- | :----------------------------------------------------------- |
| **Detalles del proyecto**    |                                                              |
| Suscripción                  | Suscripcion del learn                                        |
| Grupo de recursos            | Seleccionamos el grupo, que se encuentre en nuestra suscripcion del learn |
| **Detalles de la instancia** |                                                              |
| Nombre de registro           | Seleccione un nombre de su elección. El nombre del registro debe ser único dentro de Azure y contener entre 5 y 50 caracteres alfanuméricos. |
| Ubicación                    | Seleccione una ubicación que esté cerca de usted.            |
| SKU                          | **Estándar**                                                 |

Que queda así:

<img align="center" src="/img/19ºimagenn.PNG"  />

Revisamos y creamos, esperamos la validación final y accedemos al recurso.

Ahora vamos a acceder a las Claves de acceso y activaremos la opción de usuario administrador:

<img align="center" src="/img/20ºimagenn.PNG"  />

Al clicar en Usuario administrador se nos desplegaran una serie de datos que hemos de anotar para después realizar la conexión y subida de datos

***En este ejercicio, habilitamos el acceso a la cuenta de administrador para que podamos cargar imágenes y probar el registro. En un entorno de producción, es importante deshabilitar el acceso a la cuenta de usuario administrador y usar Microsoft Entra ID Protection tan pronto como esté satisfecho de que el registro esté funcionando como se esperaba.***

Ahora nos vamos a nuestro simbolo del sistema, vamos a proceder a cambiar la etiqueta donde `<registry-name>` sera nuestro `ramonapp`:

```
docker tag reservationsystem:latest <registry-name>.azurecr.io/reservationsystem:latest

```



Comprobamos que se haya cambiado la etiqueta

<img align="center" src="/img/21ºimagenn.PNG"  />

Procedemos a realizar la conexión, para esto vamos a necesitar los datos, donde `<login-serve>` sera nuestro `ramonapp.azurecr.io`

Probablemente se nos pida la contraseña y es probable tambien que obtengamos una advertencia indicando que su aplicación no está registrada con Microsoft Entra ID.

```
docker login <login-server>
```

<img align="center" src="/img/22ºimagenn.PNG"  />

Y ahora procederemos a cargar la imagen con el siguiente comando, donde `<registry-name>` sera nuestro `ramonapp`:

```
docker push <registry-name>.azurecr.io/reservationsystem:latest
```

<img align="center" src="/img/23ºimagenn.PNG"  />

Cuando todo se encuentre en Pushed, podemos proceder a la verificacion del contenido del registro.

## Verificar el contenido del registro.

Volvemos al registro del contenedor->Servicios->Repositorios y comprobamos que contenga una imagen con la etiqueta Latest

<img align="center" src="/img/24ºimagenn.PNG"  />



## Cargue y ejecute una imagen usando Azure Container Instance

Hemos de Crear recurso-> Azure Container Instances(No confundir con el de registro) y completar la pestaña básicos con los siguientes datos

| Configuración                                        | Valor                                                        |
| :--------------------------------------------------- | :----------------------------------------------------------- |
| **Detalles del proyecto**                            |                                                              |
| Suscripción                                          | Seleccione su suscripción de Azure predeterminada en la que puede crear y administrar recursos. |
| Grupo de recursos                                    | Reutilice el grupo de recursos existente **learn-deploy-container-aci-rg** |
| **Detalles del contenedor**                          |                                                              |
| Nombre del contenedor                                | instancia del sistema hotelero                               |
| Región                                               | Utilice la ubicación predeterminada                          |
| Fuente de imagen                                     | Docker Hub u otro registro                                   |
| Tipo de imagen                                       | Privado                                                      |
| Imagen                                               | ramonapp.azurecr.io/reservationsystem:latest                 |
| Servidor de inicio de sesión de registro de imágenes | ramonapp.azurecr.io                                          |
| Nombre de usuario del registro de imágenes           | ramonapp                                                     |
| Contraseña de registro de imágenes                   | La pass que apuntamos al principio del todo                  |
| Tipo de sistema operativo                            | linux                                                        |
| Tamaño                                               | *Deje el tamaño* predeterminado establecido en **1 vcpu, 1,5 GiB de memoria, 0 gpus** |

<img align="center" src="/img/25ºimagenn.PNG"  />

Tras rellenar esto seguiremos a la pestaña de **Redes**

<img align="center" src="/img/26ºimagenn.PNG"  />

Después de esto pasaremos a las opciones avanzadas donde solo cambiaremos la directiva de incio lo demás se quedara en predeterminado

<img align="center" src="/img/27ºimagenn.PNG"  />

Por ultimo revisamos y creamos, nos dirigimos a descripcion general y hemos de fijarnos en lo siguiente: 

<img align="center" src="/img/28ºimagenn.PNG"  />

Y para comprobar que funcione accederemos a la url http://hotel.southcentralus.azurecontainer.io/api/reservations/1 donde podremos ver el contenido del JSON.



**NOTA IMPORTANTE**: Al realizar esta practica via sandbox de Microsoft learn llega un momento en el que no se permite la creación de Container Instances por temas de permisos, no he podido comprobar que funcione correctamente al no disponer de una suscripción real pero estos son todos los pasos a seguir.
