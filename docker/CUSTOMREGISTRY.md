Guía de Despliegue con Docker Registry PrivadoEsta guía documenta el proceso para configurar un Docker Registry privado protegido con autenticación básica y SSL, así como el flujo de trabajo para construir, subir y desplegar imágenes en producción.1. Requisitos PreviosVPS con Docker y Docker Compose instalados.Subdominio apuntando al VPS (ej. hub.midominio.com).Proxy Inverso (Nginx, Traefik, Caddy) configurado con SSL (HTTPS) que redirija el tráfico del subdominio al puerto 5000 del contenedor.Nota: Al usar SSL, NO es necesario configurar insecure-registries en el daemon.json.2. Configuración del Registry (En el VPS)Paso A: Generar CredencialesNecesitamos htpasswd para crear el archivo de autenticación.Instalar utilidades (Ubuntu/Debian):sudo apt update && sudo apt install apache2-utils -y
Crear directorio y archivo de contraseñas:mkdir -p ~/docker-registry/auth
# Reemplaza 'miusuario' por tu usuario deseado
htpasswd -Bc ~/docker-registry/auth/htpasswd miusuario
(Ingresa la contraseña cuando se te solicite).Paso B: Iniciar el Contenedor del RegistryEjecuta el siguiente comando para levantar el registry con autenticación habilitada.docker run -d \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  -v ~/docker-registry/auth:/auth \
  -v registry_data:/var/lib/registry \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" \
  registry:2
3. Flujo de Trabajo (Desde tu PC Local)Paso A: AutenticaciónAntes de subir imágenes, debes iniciar sesión. Esto solo se hace una vez (o si cambias de credenciales).docker login hub.midominio.com
Paso B: Construir y Subir Imagen (Push)Cuando tengas cambios listos en tu código:Construir la imagen:IMPORTANTE: No usar https:// ni puertos en el tag. Solo dominio/imagen:tag.# Sintaxis: docker build -t DOMINIO/NOMBRE_PROYECTO:TAG .
docker build -t [hub.midominio.com/mi-app:v1](https://hub.midominio.com/mi-app:v1) .
Subir al registry:docker push [hub.midominio.com/mi-app:v1](https://hub.midominio.com/mi-app:v1)
4. Despliegue en Producción (En el VPS)Paso A: Autenticación en el ServidorEl servidor también necesita permiso para descargar las imágenes.docker login hub.midominio.com
Paso B: Archivo docker-compose.prod.ymlCrea este archivo en tu servidor para el despliegue.Diferencias clave con desarrollo:No usar build:.Usar image: apuntando al registry privado.restart: always.version: '3.8'

services:
  app:
    image: [hub.midominio.com/mi-app:v1](https://hub.midominio.com/mi-app:v1)
    container_name: mi_app_prod
    restart: always
    env_file:
      - .env
    ports:
      - "80:80"
    networks:
      - app_net

networks:
  app_net:
    driver: bridge
Paso C: Ejecutardocker compose -f docker-compose.prod.yml up -d
5. Actualizaciones FuturasPara actualizar la aplicación en producción:Local: Cambiar código, construir versión v2 y hacer push.docker build -t [hub.midominio.com/mi-app:v2](https://hub.midominio.com/mi-app:v2) .
docker push [hub.midominio.com/mi-app:v2](https://hub.midominio.com/mi-app:v2)
VPS: Editar docker-compose.prod.yml cambiando el tag a v2.VPS: Aplicar cambios.docker compose -f docker-compose.prod.yml up -d
