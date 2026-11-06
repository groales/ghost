# Ghost con Docker Compose

Este repositorio despliega una instancia sencilla de **Ghost** usando Docker Compose y base de datos **SQLite** (persistida en volumen local).

## ¿Qué es Ghost?

Ghost es una plataforma de publicación enfocada en blogs, newsletters y sitios de contenido.  
Incluye panel de administración, editor moderno y gestión de usuarios en una solución ligera.

## Arquitectura de este despliegue

La configuración está en [compose.yaml](C:/Users/gustavo.roales/Developer/projects/docker/ghost/compose.yaml) y define:

- **Servicio:** `ghost`
- **Imagen:** `ghost:latest`
- **Puerto publicado:** `2368:2368`
- **Persistencia:** `./content:/var/lib/ghost/content`
- **Base de datos:** SQLite (`content/data/ghost.db`)
- **Entorno:** `NODE_ENV=production`
- **Red:** red externa `proxy` como red por defecto

## Requisitos previos

- Docker
- Docker Compose (plugin `docker compose`)
- Red externa `proxy` creada en Docker

Si la red `proxy` no existe, créala:

```bash
docker network create proxy
```

## Despliegue

Desde la raíz del repositorio:

```bash
docker compose up -d
```

Verifica el estado:

```bash
docker compose ps
docker compose logs -f ghost
```

Acceso local:

- Sitio: `http://localhost:2368`
- Alta inicial de admin (primera ejecución): `http://localhost:2368/ghost`

## Configuración inicial de Ghost (primer arranque)

Después de levantar el contenedor por primera vez:

1. Abre `http://localhost:2368/ghost`.
2. Completa el asistente de Ghost:
   - **Site title**: nombre público del sitio.
   - **Full name**: nombre del usuario administrador.
   - **Email address**: email del administrador.
   - **Password**: contraseña del administrador.
3. Pulsa **Create account** para terminar el alta.
4. Inicia sesión en `http://localhost:2368/ghost` con ese usuario.

Notas:

- Este asistente se muestra solo una vez por instalación (mientras se conserve [content/](C:/Users/gustavo.roales/Developer/projects/docker/ghost/content)).
- Si luego cambias la URL pública (dominio/proxy), actualiza el valor `url` en [compose.yaml](C:/Users/gustavo.roales/Developer/projects/docker/ghost/compose.yaml) y reinicia con `docker compose up -d`.

## Datos persistentes

Todo el contenido y la base de datos SQLite quedan en [content/](C:/Users/gustavo.roales/Developer/projects/docker/ghost/content), por lo que sobreviven a reinicios y recreación del contenedor.

## Parar y eliminar el despliegue

```bash
docker compose down
```

> Esto detiene y elimina el contenedor, pero mantiene los datos del directorio `content/`.

## Notas de configuración

- La URL base actual está configurada como `http://localhost:2368`.
- Si se publica detrás de un proxy inverso o con dominio propio, actualiza `url` en [compose.yaml](C:/Users/gustavo.roales/Developer/projects/docker/ghost/compose.yaml).

## Despliegue en producción (dominio + HTTPS)

Para publicar Ghost en Internet con TLS:

1. Define un dominio, por ejemplo: `blog.midominio.com`.
2. En [compose.yaml](ghost/compose.yaml), cambia:
   - `url` a `https://blog.midominio.com`
3. Conecta Ghost a tu proxy inverso (Traefik, Nginx Proxy Manager, Caddy, Nginx, etc.) usando la red `proxy`.
4. Emite/renueva certificados TLS en el proxy (Let's Encrypt o similar).

### Ajuste recomendado de exposición de puertos

Si Ghost quedará detrás de proxy inverso, evita exponerlo públicamente en el host.

- Opción actual (válida para local): `2368:2368`
- Recomendado en producción:
  - Eliminar `ports` y usar solo red interna con el proxy, o

```

### Arranque en producción

```bash
docker compose pull
docker compose up -d
docker compose ps
```

### Comprobaciones

- La home responde en `https://blog.midominio.com`
- El admin responde en `https://blog.midominio.com/ghost`
- Ghost inicia sin errores:

```bash
docker compose logs -f ghost
```

### Operación recomendada

- Backups periódicos del directorio [content/](ghost/content).
- Controlar actualizaciones de imagen (`ghost:latest` actualiza automáticamente al hacer pull).  
  Si prefieres estabilidad estricta, usa una etiqueta fija de versión.
