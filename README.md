# Introducción


Este repositorio configura gitlab community edition, basado en el [proyecto GitLab](https://gitlab.com/gitlab-org/gitlab-ce/tree/master/docker)


# Instalación

Utilizo el contenedor oficial [gitlab/gitlab-ce](https://hub.docker.com/r/gitlab/gitlab-ce/) y la [documentación oficial](https://hub.docker.com/r/gitlab/gitlab-ce/). En el fichero fig-template.yml tienes un ejemplo para la ejecución a través de fig.


# Documentación

Para poder ejecutar y configurar este contenedor recomiendo consultar la documentación oficial, aquí tienes algunos tips:


# Vistazo rápido

Además de la documentación, dejo aquí unas notas rápidas

Pull de la imagen

    docker pull gitlab/gitlab-ce:latest

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


# Puertos externos

Si utilizas puertos externos no olvides modificar el fichero gitlab.rb, tal como se define en su documentación. Un ejemplo:

En el fichero gitlab.rb

    # For HTTP
    external_url "http://gitlab.example.com"
    
    # For HTTPS
    external_url "https://gitlab.example.com"

    # Set gitlab_shell_ssh_port:
    gitlab_rails['gitlab_shell_ssh_port'] = 2289
  
