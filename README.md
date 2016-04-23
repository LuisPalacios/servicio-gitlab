# Introducción


Este repositorio configura gitlab community edition, basado en el [proyecto GitLab](https://gitlab.com/gitlab-org/gitlab-ce/tree/master/docker) Community Edition.


# Instalación

Utilizo el contenedor oficial [gitlab/gitlab-ce](https://hub.docker.com/r/gitlab/gitlab-ce/) y la [documentación oficial](https://hub.docker.com/r/gitlab/gitlab-ce/). En el fichero fig-template.yml tienes un ejemplo para la ejecución a través de fig.


# Documentación

Para poder ejecutar y configurar este contenedor recomiendo consultar la documentación oficial, aquí tienes algunos tips sobre cómo lo he utilizado en mi caso.


## Notas rápidas


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

Como decía en las notas rápidas, la primera vez que instalé GitLab lo hice con la versión que había en ese momento (8.4.3) pero dos semanas después sacaron la versión 8.5. ¿funcionaría la actualización? ¿sería tan sencillo como hacer un pull de la nueva imagen?. 

A continuación encontrarás los pasos que he realizado para actualizar. He utilizado lo mismo para pasar a posteriores versiones, de forma secuencial he ido actualizando a la 8.6, 8.7, etc... 

Parar el contenedor:

    luis@aplicacionix ~ $ cd /Apps/docker/servicio-gitlab
    luis@aplicacionix /Apps/docker/servicio-gitlab $ fig stop
    
    (( recordar que utilizo FIG, el equivalente sería "docker stop gitlab" ))

Hacer un backup de los datos :-)
     
    tar cfz /share/DATOS_BACKUP/baul/Apps/2016-02-22.Apps_data_gitlab.tgz Apps/data/gitlab
    
Eliminar el contenedor:

    luis@aplicacionix /Apps/docker/servicio-gitlab $ docker rm gitlab
    luis@aplicacionix /Apps/docker/servicio-gitlab $ docker rmi gitlab/gitlab-ce
    
Pull de la nueva imagen: 

    luis@aplicacionix /Apps/docker/servicio-gitlab $ docker pull gitlab/gitlab-ce:latest

Rearranco la nueva imagen: 

    luis@aplicacionix ~ $ cd /Apps/docker/servicio-gitlab
    luis@aplicacionix /Apps/docker/servicio-gitlab $ fig up -d

### ¿Qué pasa si falla?
Pues tendrás que investigar caso por caso, a modo de ejemplo al actualizar a la versión 8.5 vi que fallaba por lo que utilicé el comando "docker logs xxxxxxxx" donde descubrí que intentaba crear un fichero de log en el subdirectorio logs/reconfigure/... y que dicho directorio NO existía, así que simplemente creé manualmente el directorio y listo... 

    cd Apps/data/gilab/logs
    /Apps/data/gilab/logs $ mkdir reconfigure
    /Apps/data/gilab/logs $ cd /Apps/docker/servicio-gitlab
    /Apps/docker/servicio-gitlab $ fig up -d 

Tampoco está de más que mires el [repositorio](https://hub.docker.com/r/gitlab/gitlab-ce/) a ver si hay comentarios de otros usuarios. 
    
### ¿Qué pasa con los datos?

El propio contenedor tiene en cuenta la existencia de los datos de la versión anterior, base de datos, etc. por lo que se encarga de las correspondientes actualizaciones. No hay que hacer nada. Simplemente acceder de nuevo vía Web y deberías tener tu servicio disponible. 
Quitando el tema del directorio "reconfigure" que tuve que crear a mano al migrar a la versión 8.5, el resto ha ido como la seda... Funciona perfectamente en las migraciones. 


