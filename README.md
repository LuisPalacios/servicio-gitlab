# Introducción


Este es el repositorio que he utilizado para crear un contenedor Docker con [GitLab Community Edition](https://gitlab.com/gitlab-org/gitlab-ce/tree/master/docker).


# Instalación

Utilizo el contenedor oficial [gitlab/gitlab-ce](https://hub.docker.com/r/gitlab/gitlab-ce/), en el fichero fig-template.yml tienes un ejemplo para la ejecución a través de fig.


# Documentación

Para poder ejecutar y configurar este contenedor recomiendo consultar la [documentación oficial](https://hub.docker.com/r/gitlab/gitlab-ce/), aquí dejo unos apuntes rápidos sobre mi instalación. 


## Apuntes y notas


Pull de la imagen

    docker pull gitlab/gitlab-ce:latest   ((Nota: En mi caso instalé la versión 8.4.3))

Arranque del contenedor, en mi caso utilizo FIG (ver fig-tempalte.xml para un ejemplo), así que utilizo el comando fig

    fig up -d

Una vez arrancado conectar con http://X.X.X.X/ y emplear el usuario root con contraseña 5iveL!fe.

Arrancar una shell dentro del contenedor para configurarlo si lo necesitas

    docker exec -it gitlab /bin/bash
    
Ver estatus y logs

    docker logs -f gitlab


Para actualizar a la última versión

    docker stop gitlab
    docker rm gitlab
    docker pull gitlab/gitlab-ce:latest
    fig up -d


## Puertos externos

Si utilizas puertos externos no olvides modificar el fichero gitlab.rb, tal como se define en su documentación. Un ejemplo:

En el fichero gitlab.rb

    # For HTTP
    external_url "http://gitlab.example.com"
    
    # For HTTPS
    external_url "https://gitlab.example.com"

    # Set gitlab_shell_ssh_port:
    gitlab_rails['gitlab_shell_ssh_port'] = 2289


## Actualización

La primera vez que instalé GitLab lo hice con la versión que había en ese momento (8.4.3) pero dos semanas después sacaron la versión 8.5. ¿funcionaría la actualización? ¿sería tan sencillo como hacer un pull de la nueva imagen?. 

La respuesta es que sí, más abajo tienes los pasos que sigo cada vez que actualizo. He utilizado lo mismo para pasar a posteriores versiones, de forma secuencial he ido actualizando a la 8.6, 8.7, etc.

Lo primero que hago es consultar el número de la versión actual y apuntármelo para usarlo luego, en este ejemplo estoy pasando de la versión 8.7.5 a la versión 8.10.5

    http://gitlab.midominio.com/help 
    Verás algo parecido a: GitLab Community Edition 8.7.5 0e8b7d8

Paro el contenedor:

    luis@aplicacionix ~ $ cd /Apps/docker/servicio-gitlab
    luis@aplicacionix /Apps/docker/servicio-gitlab $ fig stop
    
    (( recordar que utilizo FIG, el equivalente sería "docker stop gitlab" ))

Hacer un backup de los datos (como root)

    luis@aplicacionix ~ # cd /
    luis@aplicacionix ~ # tar cfz /share/DATOS_BACKUP/baul/Apps/2016-08-15.Apps_data_gitlab.tgz Apps/data/gitlab

Hacer una copia de la IMAGEN. En vez de copiarla lo que hago es etiquetarla y así me quedo con una copia :-). Si la migración falla estrepitosamente tengo forma de volver a la imagen anterior (8.7.5) con los datos del backup que acabo de hacer. Busco el repositorio gitlab/gitlab-ce y me apunto el ID de la imagen para luego etiquetarlo:

    luis@aplicacionix $ docker images | grep gitlab   (El ID será del tipo: 98dfe52bcfd2)
    luis@aplicacionix $ docker tag <ID> gitlab/gitlab-ce:8.7.5  (8.7.5, lo averigué en el primer paso)
    
Eliminar el CONTENEDOR. Buscar el contenedor gitlab/gitlab-ce:latest, copiar el ID para usarlo a continuación

    luis@aplicacionix $ docker ps -a | grep gitlab
    luis@aplicacionix $ docker rm <container_ID>
    
Pull de la nueva imagen, en el siguiente ejemplo observarás la nueva versión 'latest' y la anterior '8.7.5'

    luis@aplicacionix $ docker pull gitlab/gitlab-ce:latest
    luis@aplicacionix ~ $ docker images | grep gitlab
    gitlab/gitlab-ce           latest              4d00a46c96e2        5 days ago          1.154 GB
    gitlab/gitlab-ce           8.7.5               98dfe52bcfd2        3 months ago        1.185 GB

Rearranco la nueva imagen: 

    luis@aplicacionix ~ $ cd /Apps/docker/servicio-gitlab
    luis@aplicacionix /Apps/docker/servicio-gitlab $ fig up -d
    luis@aplicacionix /Apps/docker/servicio-gitlab $ fig logs    (Para ver qué va haciendo.. ojo que tarda un buen rato)

Comprueba que todo va perfecto, que puedes conectar, que reconoce todos tus repositorios, etc. Si todo va bien puedes eliminar la imagen antigua para ahorrar espacio:

    luis@aplicacionix $ docker rmi gitlab/gitlab-ce:8.7.5


### ¿Qué pasa si falla?

Tienes dos opciones. La primera es que no vaya nada y no consigas migrar. Restaura los datos antiguos, renombra la IMAGEN a la que le pusiste el TAG y vuelve a arrancar el servicio. Estás como estabas.  Mira en los foros, reporta el problema, investiga...

La segunda opción es intentar arreglarlo, tendrás que investigar caso por caso, a modo de ejemplo al actualizar a la versión 8.5 vi que fallaba por lo que utilicé el comando "docker logs xxxxxxxx" donde descubrí que intentaba crear un fichero de log en el subdirectorio logs/reconfigure/... y que dicho directorio NO existía, así que simplemente creé manualmente el directorio y listo... 

    cd Apps/data/gilab/logs
    /Apps/data/gilab/logs $ mkdir reconfigure
    /Apps/data/gilab/logs $ cd /Apps/docker/servicio-gitlab
    /Apps/docker/servicio-gitlab $ fig up -d 

Tampoco está de más que mires el [repositorio](https://hub.docker.com/r/gitlab/gitlab-ce/) a ver si hay comentarios de otros usuarios. 

### ¿Qué pasa con los datos?

El propio contenedor tiene en cuenta la existencia de los datos de la versión anterior, base de datos, etc. por lo que se encarga de las correspondientes actualizaciones. No hay que hacer nada. Simplemente acceder de nuevo vía Web y deberías tener tu servicio disponible. 
Quitando el tema del directorio "reconfigure" que tuve que crear a mano al migrar a la versión 8.5, el resto ha ido como la seda... Funciona perfectamente en las migraciones. 


