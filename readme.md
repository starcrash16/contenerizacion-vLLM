# Despliegue de Inferencia vLLM y UI con Contenedores (Podman/Docker)

**Autor:** De Anda Medina, René Rosendo

**Documento base:** *Implementación de vLLM con Open Web UI (GPU) [vllm-qwen-7b]*

**Repositorio asociado:** `github.com/starcrash16/contenerizacion-vLLM`

---

## Resumen

Este repositorio documenta el despliegue de un servidor de inferencia **vLLM** de alto rendimiento y una interfaz gráfica **Open Web UI**. La pila se orquesta mediante un único archivo `compose.yml` compatible tanto con **Podman** (`podman-compose`) como con **Docker** (`docker-compose`).

El objetivo es servir el modelo `Qwen/Qwen1.5-7B-Chat` en hardware GPU NVIDIA (Ampere/Hopper). Se prioriza el uso de Podman por su arquitectura `rootless` (sin root), que mejora la seguridad, aunque la implementación es totalmente funcional con Docker. Esta solución utiliza vLLM como un backend compatible con la API de OpenAI.

---

## Arquitectura de la Solución

La pila tecnológica se compone de los siguientes elementos:

* **Motor de Contenedores:** `Podman` (recomendado) o `Docker`.
* **Orquestación:** `podman-compose` o `docker-compose`.
* **Archivo de Definición:** `compose.yml` (compatible con ambos orquestadores).
* **Servidor de Inferencia:** `vllm/vllm-openai:latest` (expone API compatible con OpenAI en el puerto `8000`).
* **Interfaz Gráfica:** `ghcr.io/open-webui/open-webui:main` (frontend de chat).
* **Modelo:** `Qwen/Qwen1.5-7B-Chat` (modelo oficial cargado desde Hugging Face).

---

## Implementación

El despliegue se gestiona a través del archivo `compose.yml` incluido en este repositorio.

### Requisitos Previos

El archivo `compose.yml` es agnóstico al motor. Los requisitos varían ligeramente según la herramienta seleccionada.

**Requisitos Comunes (Para ambos motores):**

1.  **Directorios de Volúmenes:** Se deben crear las siguientes rutas en el *host* para los datos persistentes:
    * `/datos/contenedores/lcid-vllm-modelos` (para el caché de modelos).
    * `/datos/contenedores/lcid-webui-vllm` (para los datos de la UI).
2.  **Soporte de GPU:** El **NVIDIA Container Toolkit** debe estar instalado y configurado para el motor de contenedores que se vaya a utilizar.

**Requisitos Específicos (Elija una opción):**

**Opción A: Podman (Rootless)**

* `podman` instalado.
* `podman-compose` instalado (ej. `pip install podman-compose`).
* **Permisos:** Los directorios de volúmenes (`/datos/contenedores/...`) deben ser propiedad del usuario `rootless` que ejecutará los contenedores.

**Opción B: Docker**

* `docker` instalado.
* `docker-compose` instalado (generalmente como un plugin de Docker, `docker compose`).
* **Permisos:** El usuario debe tener permisos para comunicarse con el *daemon* de Docker (generalmente, perteneciendo al grupo `docker` o usando `sudo`).

### Ejecución de los Servicios

1.  Clone este repositorio y navegue al directorio raíz.
2.  Inicie la pila de servicios en segundo plano utilizando el comando correspondiente a su motor de contenedores.

    *Para Podman:*
    ```bash
    podman-compose up -d
    ```

    *Para Docker:*
    ```bash
    docker-compose up -d
    ```

La primera ejecución descargará las imágenes de los contenedores y el modelo `Qwen/Qwen1.5-7B-Chat`, que se almacenará en el volumen local persistente.

---

## Configuración y Gestión

### Acceso y Configuración de la UI

Estos pasos son idénticos independientemente del motor de contenedores utilizado.

1.  Acceda a la interfaz web en `http://localhost:3001`.
2.  Cree la cuenta de administrador local.
3.  Navegue a **Settings** -> **Models**.
4.  Añada manualmente el nombre del modelo servido por vLLM:
    ```
    Qwen/Qwen1.5-7B-Chat
    ```
5.  Guarde y seleccione el modelo en la interfaz principal de chat.

### Gestión de Servicios y Comandos

Toda la gestión de la pila (ver registros, detener servicios, etc.) se realiza con el orquestador elegido. La sintaxis de los comandos es intercambiable, como se detalla en la siguiente tabla.

| Acción | Comando Podman (Rootless) | Comando Docker |
| :--- | :--- | :--- |
| **Iniciar servicios** (segundo plano) | `podman-compose up -d` | `docker-compose up -d` |
| **Ver registros** (en tiempo real) | `podman-compose logs -f` | `docker-compose logs -f` |
| **Ver registros de un servicio** | `podman-compose logs -f vllm` | `docker-compose logs -f vllm` |
| **Detener y eliminar contenedores** | `podman-compose down` | `docker-compose down` |
| **Listar contenedores en ejecución** | `podman ps` | `docker ps` |
