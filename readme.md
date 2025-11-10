# Despliegue de Inferencia (vLLM) y UI (Open Web UI) con Podman

**Autor:** De Anda Medina, René Rosendo

**Documento base:** *Implementación de vLLM con Open Web UI (GPU) [vllm-qwen-7b]*

**Repositorio asociado:** `[github.com/starcrash16/contenerizacion-vLLM]`

---

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
* **Modelo:** `Qwen/Qwen1.5-7B-Chat` (SML <10B) cargado desde Hugging Face.

---

## Implementación

El despliegue se gestiona a través del archivo `compose.yml` incluido en este repositorio.

### Requisitos Previos

1.  `podman` instalado.
2.  `podman-compose` instalado.
3.  NVIDIA Container Toolkit configurado para Podman.
4.  Creación de los directorios de volúmenes persistentes:
    * `/datos/contenedores/lcid-vllm-modelos` (caché de modelos).
    * `/datos/contenedores/lcid-webui-vllm` (datos de la UI).
5.  Asignación de permisos adecuados en los directorios para el usuario `rootless` que ejecuta Podman.

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
3.  Navegue a **Settings** ➡️ **Models**.
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
