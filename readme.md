# Despliegue de Inferencia (vLLM) y UI (Open Web UI) con Podman

**Autor:** De Anda Medina, René Rosendo

**Documento base:** *Implementación de vLLM con Open Web UI (GPU) [vllm-qwen-7b]*

**Repositorio asociado:** `[github.com/starcrash16/contenerizacion-vLLM]`

## Resumen

Este repositorio documenta el despliegue de un servidor de inferencia **vLLM** de alto rendimiento y una interfaz gráfica **Open Web UI**, orquestados mediante **Podman** y `podman-compose`.

El objetivo es servir el modelo `Qwen/Qwen1.5-7B-Chat` en hardware GPU NVIDIA (Ampere/Hopper), priorizando una arquitectura `rootless` para mayor seguridad. Esta implementación utiliza vLLM como un backend compatible con la API de OpenAI.

---

## Pila Tecnológica

La arquitectura de la solución se compone de:

* **Motor de Contenedores:** `Podman` (arquitectura `daemonless` y `rootless`).
* **Orquestación:** `podman-compose` (compatibilidad con sintaxis `compose.yml`).
* **Servidor de Inferencia:** `vllm/vllm-openai:latest` (expone API compatible con OpenAI en el puerto `8000`).
* **Interfaz Gráfica:** `ghcr.io/open-webui/open-webui:main` (frontend de chat).
* **Modelo:** `Qwen/Qwen1.5-7B-Chat` modelo oficial cargado desde Hugging Face.

---

## Implementación

El despliegue se gestiona a través del archivo `compose.yml` incluido en este repositorio.

### Requisitos Previos

El archivo `compose.yml` es compatible tanto con `podman-compose` como con `docker-compose`. Los requisitos varían ligeramente según el motor de contenedores seleccionado.

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
2.  Inicie la pila de servicios en segundo plano:

    ```bash
    podman-compose up -d
    ```

El primer inicio descargará las imágenes de los contenedores y el modelo `Qwen/Qwen1.5-7B-Chat`, que se almacenará en el volumen local.

---

## Configuración y Gestión

### Acceso y Configuración de la UI

1.  Acceda a la interfaz web en `http://localhost:3001`.
2.  Cree la cuenta de administrador local.
3.  Navegue a **Settings** -> **Models**.
4.  Añada manualmente el nombre del modelo servido por vLLM:
    ```
    Qwen/Qwen1.5-7B-Chat
    ```
5.  Guarde y seleccione el modelo en la interfaz principal de chat.

### Gestión de Servicios

Utilice los siguientes comandos de `podman-compose` para administrar la pila:

* **Verificar registros (logs):**
    ```bash
    # Monitorizar todos los servicios
    podman-compose logs -f
    
    # Monitorizar solo el servicio vLLM (útil para ver la carga del modelo)
    podman-compose logs -f vllm
    ```

* **Detener y eliminar contenedores:**
    ```bash
    # No elimina los volúmenes de datos persistentes
    podman-compose down
    ```

---

## Comparativa de Comandos (Podman vs Docker)

Este proyecto está configurado para `podman-compose`. La sintaxis es directamente compatible con `docker-compose` si se prefiere utilizar Docker.

| Acción | Comando Podman (Rootless) | Comando Docker |
| :--- | :--- | :--- |
| **Iniciar servicios** (segundo plano) | `podman-compose up -d` | `docker-compose up -d` |
| **Ver registros** (en tiempo real) | `podman-compose logs -f` | `docker-compose logs -f` |
| **Detener y eliminar contenedores** | `podman-compose down` | `docker-compose down` |
| **Listar contenedores en ejecución** | `podman ps` | `docker ps` |
