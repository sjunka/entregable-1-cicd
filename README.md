# entregable-1-cicd

Taller Entregable 1 del curso Integración y Entrega Continua de Software (SI6008), Universidad EAFIT.

## Integrantes

Texto usado en `AUTOR`: `Sergio Junca`

| Nombre completo | Usuario de GitHub | Rol |
|---|---|---|
| Sergio Junca | [sjunka](https://github.com/sjunka) | Propietario del repositorio |

Entrega individual.

## Contenido

- `.github/workflows/mi_workflow.yml`: workflow que imprime un saludo, la fecha y hora, configura Python 3.13 y ejecuta el script.
- `mi_script.py`: imprime el autor, el sistema operativo del runner y la fecha y hora desde Python.

## Ejecución exitosa

- Ejecución por `push` a `main`: https://github.com/sjunka/entregable-1-cicd/actions/runs/42638782136
- Ejecución manual (`workflow_dispatch`): https://github.com/sjunka/entregable-1-cicd/actions/runs/42638838974

Salida del job `imprimir_info` en la ejecución por `push`:

```
Hola, soy Sergio Junca y este es mi primer workflow!
La fecha y hora actual es: Mon Nov  9 14:51:37 UTC 2026
Autor: Sergio Junca
Sistema Operativo: Linux 7.0.0-1019-azure
Fecha y hora desde Python: 2026-11-09 14:51:38.317138
```

## Respuestas

### 1. Eventos configurados

Configuré dos eventos:

- `push` sobre la rama `main`: el workflow corre cada vez que subo cambios a la rama principal. Así sé enseguida si lo que acabo de subir sigue funcionando, sin tener que acordarme de lanzarlo.
- `workflow_dispatch`: permite ejecutarlo a mano desde la pestaña Actions con el botón "Run workflow". Sirve para volver a correrlo sin hacer un commit, por ejemplo para revisar los logs o probar después de un fallo del runner.

No agregué `pull_request` porque en este repositorio trabajo solo y directo sobre `main`. En un proyecto con más personas sí lo agregaría, para validar los cambios antes de que se integren.

### 2. Sistema operativo del runner y relación con el shell

`ubuntu-latest` es una máquina virtual Linux que GitHub crea para cada ejecución. En mi ejecución el log del paso "Set up job" muestra la imagen `ubuntu-26.04` (Ubuntu 26.04.1 LTS), y el script de Python reporta `Linux`. La etiqueta `latest` apunta a la versión de Ubuntu que GitHub tenga como predeterminada en ese momento, así que puede cambiar con el tiempo.

Funciona aunque mi computador sea macOS porque el workflow no se ejecuta en mi máquina. Yo solo subo el archivo YAML al repositorio y GitHub lanza el job en sus servidores. Por eso el script local muestra `Darwin` y el mismo script en Actions muestra `Linux`.

El runner define qué shells hay disponibles. En Ubuntu el shell por defecto de los pasos `run` es Bash, y por eso funciona `$(date)`. Si cambiara a `windows-latest`, el shell por defecto sería PowerShell y ese comando tendría que escribirse distinto (por ejemplo `Get-Date`).

### 3. Acciones del Marketplace

- `actions/cache`: guarda carpetas entre ejecuciones, como las dependencias descargadas con pip o npm. Si el archivo de dependencias no cambia, las restaura en vez de descargarlas otra vez, y el pipeline termina más rápido.
- `docker/build-push-action`: construye una imagen Docker a partir de un `Dockerfile` y la puede publicar en un registro como Docker Hub o GitHub Container Registry. Es útil cuando la aplicación se despliega como contenedor.
- `softprops/action-gh-release`: crea un release en GitHub, normalmente al subir un tag de versión (por ejemplo `v1.0.0`), y adjunta archivos como binarios o paquetes. Sirve para automatizar la publicación de versiones.

### 4. Secretos en GitHub Actions

Un secreto es un valor sensible, como una contraseña, un token o una URL privada, que se guarda cifrado en la configuración del repositorio (Settings > Secrets and variables > Actions) y no en el código. El workflow lo recibe en el momento de la ejecución con la expresión `${{ secrets.NOMBRE }}`. Así el valor no queda en el historial de Git ni a la vista de quien lea el repositorio. GitHub intenta ocultarlo en los logs, pero igual no se debe imprimir.

Este workflow no necesita secretos. Si quisiera que al final avisara en un canal de Teams que la ejecución terminó, guardaría la URL del webhook como secreto `TEAMS_WEBHOOK_URL` y la pasaría al paso como variable de entorno:

```yaml
- name: Notificar a Teams
  run: python notificar.py
  env:
    TEAMS_WEBHOOK_URL: ${{ secrets.TEAMS_WEBHOOK_URL }}
```

El script leería el valor con `os.environ["TEAMS_WEBHOOK_URL"]` sin mostrarlo nunca en la salida.

### 5. Reflexión

Lo que más me llamó la atención es que basta un archivo YAML en el repositorio para que cada cambio se verifique solo, sin depender de que alguien se acuerde de hacerlo.

En mi trabajo diario automatizaría dos cosas:

- Correr las pruebas y el linter en cada pull request. Hoy eso depende de que cada persona lo haga en su máquina antes de subir los cambios, y a veces se olvida. Si el pipeline lo hace siempre, los errores se encuentran antes de que lleguen a `main` y la revisión de código se concentra en la lógica y no en detalles de formato.
- Revisar dependencias de forma programada, con un workflow semanal (evento `schedule`) que avise cuando haya paquetes desactualizados o con vulnerabilidades conocidas. Es una tarea que nadie hace con regularidad si es manual, y dejarla atrasada hace que las actualizaciones después sean más grandes y más riesgosas.
