# Guía de Despliegue con Docker Registry Privado

Esta guía documenta el proceso para configurar un Docker Registry privado protegido con autenticación básica y SSL, así como el flujo de trabajo para construir, subir y desplegar imágenes en producción.

## 1. Requisitos Previos

* **VPS** con Docker y Docker Compose instalados.
* **Subdominio** apuntando al VPS (ej. `hub.midominio.com`).
* **Proxy Inverso** (Nginx, Traefik, Caddy) configurado con **SSL** (HTTPS) que redirija el tráfico del subdominio al puerto `5000` del contenedor.
* **Importante:** Al usar un proxy con SSL válido, **NO** es necesario configurar `insecure-registries` en el `daemon.json`.

---

## 2. Configuración del Registry (En el VPS)

### Paso A: Generar Credenciales

Necesitamos `htpasswd` para crear el archivo de autenticación.

1.  Instalar utilidades (Ubuntu/Debian):
    ```bash
    sudo apt update && sudo apt install apache2-utils -y
    ```

2.  Crear directorio y archivo de contraseñas:
    ```bash
    mkdir -p ~/registry/auth
    # Reemplaza 'miusuario' por tu usuario deseado. Te pedirá la contraseña.
    htpasswd -Bc ~/registry/auth/htpasswd miusuario
    ```

### Paso B: Iniciar el Contenedor del Registry

Ejecuta el siguiente comando para levantar el registry con autenticación habilitada.

```bash
docker run -d \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  -v ~/registry/auth:/auth \
  -v registry_data:/var/lib/registry \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" \
  registry:2
```

---

## 3. Flujo de Trabajo (Desde tu PC Local)

### Paso A: Autenticación

Antes de subir imágenes, debes iniciar sesión en tu máquina local.

```bash
docker login hub.midominio.com
# Usa las credenciales creadas en el paso 2A
```

### Paso B: Construir y Subir Imagen (Push)

Cuando tengas cambios listos en tu código:

1.  **Construir la imagen:**
    * **IMPORTANTE:** No usar `https://` ni puertos en el tag. El formato es estrictamente `dominio/imagen:tag`.

    ```bash
    # Sintaxis: docker build -t DOMINIO/NOMBRE_PROYECTO:TAG .
    docker build -t [hub.midominio.com/mi-app:v1](https://hub.midominio.com/mi-app:v1) .
    ```

2.  **Subir al registry:**
    ```bash
    docker push [hub.midominio.com/mi-app:v1](https://hub.midominio.com/mi-app:v1)
    ```

---

## 4. Despliegue en Producción (En el VPS)

### Paso A: Autenticación en el Servidor

El servidor también necesita permiso para descargar las imágenes privadas.

```bash
docker login hub.midominio.com
```

### Paso B: Archivo `docker-compose.prod.yml`

Crea este archivo en tu servidor para el despliegue.

**Diferencias clave con desarrollo:**

* No usar `build:`.
* Usar `image:` apuntando al registry privado.
* `restart: always`.

```yaml
version: '3.8'

services:
  api:
    image: [hub.midominio.com/mi-app:v1](https://hub.midominio.com/mi-app:v1)
    container_name: mi_app_prod
    restart: always
    env_file:
      - .env
    ports:
      - "${PORT:-3000}:80"
    networks:
      - app_net
    # Opcional: depends_on db, redis, etc.

networks:
  app_net:
    driver: bridge
```

### Paso C: Ejecutar

```bash
docker compose -f docker-compose.prod.yml up -d
```

---

## 5. Mantenimiento y Actualizaciones

Para actualizar la aplicación en producción:

1.  **Local:** Cambiar código, construir versión `v2` y hacer push.
    ```bash
    docker build -t [hub.midominio.com/mi-app:v2](https://hub.midominio.com/mi-app:v2) .
    docker push [hub.midominio.com/mi-app:v2](https://hub.midominio.com/mi-app:v2)
    ```
2.  **VPS:** Editar `docker-compose.prod.yml` cambiando el tag a `v2`.
3.  **VPS:** Aplicar cambios.
    ```bash
    docker compose -f docker-compose.prod.yml up -d
    ```
