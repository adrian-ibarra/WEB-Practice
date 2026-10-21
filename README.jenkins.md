# Jenkins con Docker Desktop en Windows

Este laboratorio usa contenedores Linux y el socket de Docker Desktop.
Jenkins necesita tanto el socket como el cliente Docker. La imagen oficial
de Jenkins no incluye ese cliente; Dockerfile.jenkins lo incorpora.

Desde PowerShell, en el directorio de este repositorio:

```powershell
docker build -f Dockerfile.jenkins -t jenkins-with-docker:local .
docker run -d --name jenkins-master --restart unless-stopped --user root -p 8080:8080 -p 50000:50000 --mount type=volume,source=jenkins_home,target=/var/jenkins_home --mount type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock jenkins-with-docker:local
```

El comando de arranque requiere que no exista otro contenedor con el mismo
nombre. En este equipo, el contenedor anterior se conserva detenido como
`jenkins-master-before-docker-cli`; el nuevo reutiliza `jenkins_home`.
No inicies ambos contra ese volumen al mismo tiempo.

## Configuracion inicial

Abre http://localhost:8080 y completa el asistente si aparece Unlock Jenkins.
Consulta la clave directamente en tu terminal, sin compartirla en el chat:

```powershell
docker exec jenkins-master cat /var/jenkins_home/secrets/initialAdminPassword
```

Instala los plugins recomendados y crea tu usuario administrador. El job debe
ser de tipo Pipeline, con soporte Declarative y Git, y obtener el repositorio
en el workspace. Usa `Jenkinsfile` como ruta del script. Si el job usa GitHub,
los cambios locales deben estar publicados antes de ejecutar ese pipeline.
Esta preparacion no crea un job ni publica cambios en GitHub.

## Despliegue

El pipeline solicita AUTHOR y ENVIRONMENT, verifica Docker y los archivos web,
reemplaza `apache1`, limpia el volumen `jenkins_web`, copia `web/.` usando
`docker cp` y arranca Apache. Comprueba HTTP con reintentos desde Jenkins en
http://host.docker.internal:9000/.

Abre la web desde Windows en http://localhost:9000/.

`jenkins_web` es un volumen Docker, no la carpeta `C:\jenkins_web`.
No se necesita montar una carpeta de Windows en Jenkins: `docker cp` evita
que el motor Docker tenga que interpretar la ruta del workspace de Jenkins.
El volumen se limpia en cada despliegue; no almacenes datos manuales en el.
Los nombres de contenedor, volumen y puerto son fijos para un solo job local.

## Seguridad

El acceso al socket permite controlar Docker y sus volumenes. Jenkins se
ejecuta como root para acceder al socket en este laboratorio. No ejecutes
pipelines no confiables ni expongas esta instancia a redes no confiables.