# Moodle + MariaDB (Docker Compose)

Implementación rápida de **Moodle** usando **Docker Compose** con **MariaDB**.
Estructura del proyecto y archivos listos para ejecutar en Ubuntu Server / Windows / macOS.

---

## Estructura del proyecto

```
prueba/
├── docker-compose.yml     # Archivo principal de Docker Compose (USAR ESTE)
├── php.ini                # (opcional) php.ini personalizado a copiar al contenedor
├── .env                   # (opcional) variables sensibles (contraseñas)
└── README.md              # Este archivo
```

> Nota: si tus imágenes (por ejemplo `Docker/Imagen1.PNG`) no se muestran en la vista previa, asegúrate de que la carpeta y las rutas sean correctas (ej. `Docker/Imagen1.PNG`). Las rutas en markdown son relativas al README.md.

---

# docker-compose.yml (lista para pegar)

Copia exactamente lo siguiente a `docker-compose.yml` en la carpeta `prueba/`:

```yaml
version: "3.8"

services:
  mariadb:
    image: bitnami/mariadb:latest
    container_name: prueba_mariadb
    restart: unless-stopped
    environment:
      - MARIADB_ROOT_PASSWORD=12345
      - MARIADB_DATABASE=jm_base
      - MARIADB_USER=jm
      - MARIADB_PASSWORD=1234
      - MARIADB_CHARACTER_SET=utf8mb4
      - MARIADB_COLLATE=utf8mb4_unicode_ci
    volumes:
      - mariadb_data:/bitnami/mariadb
    healthcheck:
      test: ["CMD-SHELL", "mysqladmin ping -h 127.0.0.1 -u root -p$MARIADB_ROOT_PASSWORD >/dev/null 2>&1 || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 5

  moodle:
    image: bitnami/moodle:4.3
    container_name: prueba_moodle
    depends_on:
      mariadb:
        condition: service_healthy
    restart: unless-stopped
    ports:
      - "8080:8080"   # HTTP
      - "8443:8443"   # HTTPS (si la imagen lo proporciona)
    environment:
      - MOODLE_DATABASE_HOST=mariadb
      - MOODLE_DATABASE_PORT_NUMBER=3306
      - MOODLE_DATABASE_NAME=jm_base
      - MOODLE_DATABASE_USER=jm
      - MOODLE_DATABASE_PASSWORD=1234
      - MOODLE_USERNAME=admin
      - MOODLE_PASSWORD=Admin123!
      - MOODLE_EMAIL=admin@example.com
      - BITNAMI_DEBUG=true
    volumes:
      - moodle_data:/bitnami/moodle
      - moodledata_data:/bitnami/moodledata

volumes:
  mariadb_data:
    driver: local
  moodle_data:
    driver: local
  moodledata_data:
    driver: local
```

> Explicaciones rápidas:
>
> * `depends_on` con `condition: service_healthy` asegura que MariaDB esté lista antes de iniciar Moodle (necesita healthcheck).
> * `container_name` hace más sencillo usar `docker cp` o `docker exec`.
> * Ajusta contraseñas/usuarios en `.env` o directamente en el `docker-compose.yml` según tu política de seguridad.

---

# Levantar el entorno

Desde la carpeta `prueba/` ejecuta:

```bash
# (opcional) si usas Compose v2 en Linux/Windows:
docker compose up -d

# Ver contenedores funcionando
docker ps
```

Accede en tu navegador a:

* [http://localhost:8080](http://localhost:8080)
* o `http://<IP_del_servidor>:8080` si está en un servidor remoto

Contenedores esperados:

* `prueba_mariadb`
* `prueba_moodle`

---

# Credenciales por defecto (según `docker-compose.yml` anterior)

* Usuario: `admin`
* Contraseña: `Admin123!`
* Email admin: `admin@example.com`

> **Cambia inmediatamente** la contraseña del admin desde la interfaz de Moodle si el despliegue es accesible públicamente.

---

# php.ini — parámetros recomendados (opcional)

Si quieres ajustar `php.ini`, puedes usar este bloque como referencia y copiarlo dentro del contenedor:

```ini
; Valores recomendados para Moodle (ejemplo)
upload_max_filesize = 100M
post_max_size = 100M
max_execution_time = 300
memory_limit = 512M
date.timezone = "America/Merida"
```

**Copiar / restaurar `php.ini` desde/hacia el contenedor**

```bash
# Obtener php.ini desde el contenedor a tu máquina (usa el container_name arriba: prueba_moodle)
docker cp prueba_moodle:/opt/bitnami/php/etc/php.ini ./php.ini

# (opcional) renombra backup
mv php.ini php.ini.bak

# Edita php.ini en tu máquina -> luego sube:
docker cp php.ini prueba_moodle:/opt/bitnami/php/etc/php.ini

# Reinicia el contenedor para aplicar cambios
docker restart prueba_moodle
```

> Si tu `container_name` es distinto, reemplázalo por el que corresponda. En Docker Compose por defecto el nombre puede ser `prueba_moodle_1` si no especificas `container_name`.

---

# Permisos de `moodledata`

Si Moodle no puede escribir en `moodledata`, revisa permisos. Con los volúmenes nombrados en docker-compose normalmente se gestiona bien, pero en despliegues bind-mount (ruta del host) asegúrate:

```bash
# ejemplo si usas bind-mount a /srv/moodledata (NO en config actual, es solo referencia)
sudo chown -R 1001:1001 /srv/moodledata    # 1001 es UID/GID típico en imágenes bitnami
sudo chmod -R 770 /srv/moodledata
```

---

# Crear curso / usuarios — guía rápida (desde UI)

1. Inicia sesión con admin.
2. Panel principal → **My courses** → **Create course**.
3. Rellena los datos y guarda.
4. Site administration → Users → Add a new user (o en Site administration → Courses → Enrol users desde el curso).
5. Asignar roles (Teacher, Student) desde Participants → Edit → Assign role.

(En el README no incluyo imágenes, coloca las tuyas en `Docker/Imagen1.PNG` y referencia desde markdown si quieres que aparezcan.)

---

# Tips y soluciones a problemas comunes

* **Contenedores no se inician**: `docker logs prueba_moodle` y `docker logs prueba_mariadb` para revisar errores.
* **MariaDB rechaza conexión**: revisa variables `MARIADB_*` y que la DB y usuario existan; elimina volumes si necesitas reiniciar DB (¡pérdida de datos!).
* **Imágenes en README no se muestran**: rutas relativas. Si README está en `prueba/` y las imágenes en `prueba/Docker/Imagen1.PNG` usa `![alt](Docker/Imagen1.PNG)`.
* **Cambiar puertos**: modifica `ports` en `docker-compose.yml` si `8080` ya está en uso.
* **Backups**: Haz backup del volumen de mariadb con `docker run --rm -v mariadb_data:/data -v $(pwd):/backup alpine tar czf /backup/mariadb_backup.tgz -C /data .`

---

# Seguridad (recomendaciones breves)

* No dejes contraseñas por defecto en producción.
* Usa `.env` y `docker secret` para datos sensibles en entornos más serios.
* Configura HTTPS real (proxy inverso: nginx / traefik) si expones Moodle a internet.
* Habilita copias de seguridad periódicas de la base de datos y `moodledata`.

---

# Ejemplo de `.env` (opcional)

Crea un archivo `.env` y reemplaza contraseñas en `docker-compose.yml` por variables referenciadas si prefieres no tenerlas en texto plano.

```
MARIADB_ROOT_PASSWORD=TU_ROOT_PASSWORD_SEGURO
MARIADB_DATABASE=jm_base
MARIADB_USER=jm
MARIADB_PASSWORD=TU_PASS_BD
MOODLE_PASSWORD=Admin123!
```

Y en `docker-compose.yml` usar `${MARIADB_ROOT_PASSWORD}` etc.

---

## Autor / proyecto

**Brayan Sierra** — *Proyecto: Moodle en Docker con MariaDB*
Sistema base: Ubuntu Server 22.04
Fecha: 2025-11-12

