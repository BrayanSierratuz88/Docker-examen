# 🐳 Moodle con Docker Compose

Implementación de **Moodle** (plataforma de aprendizaje en línea) usando **Docker Compose** con base de datos **MariaDB**.

---

## 📁 Estructura del proyecto

prueba/ │ ├── docker-compose.yaml   # Archivo principal de configuración ├── php.ini               # Config opcional (si se copia del contenedor) └── README.md             # Documentación del proyecto


---

## ⚙️ 1. Configuración del archivo `docker-compose.yaml`

Copia y pega lo siguiente dentro del archivo `docker-compose.yaml`:

```yaml
services:
  mariadb:
    image: bitnami/mariadb:latest
    environment:
      - MARIADB_ROOT_PASSWORD=12345
      - MARIADB_DATABASE=jm_base
      - MARIADB_USER=jm
      - MARIADB_PASSWORD=1234
      - MARIADB_CHARACTER_SET=utf8mb4
      - MARIADB_COLLATE=utf8mb4_unicode_ci
    volumes:
      - 'mariadb_data:/bitnami/mariadb'

  moodle:
    image: docker.io/bitnamilegacy/moodle:4.3
    depends_on:
      - mariadb
    ports:
      - '8080:8080'
      - '8443:8443'
    environment:
      - MOODLE_DATABASE_HOST=mariadb
      - MOODLE_DATABASE_PORT_NUMBER=3306
      - MOODLE_DATABASE_NAME=jm_base
      - MOODLE_DATABASE_USER=jm
      - MOODLE_DATABASE_PASSWORD=1234
      - MOODLE_USERNAME=admin
      - MOODLE_PASSWORD="Admin123!"
      - MOODLE_EMAIL=admin@example.com
    volumes:
      - 'moodle_data:/bitnami/moodle'
      - 'moodledata_data:/bitnami/moodledata'

volumes:
  mariadb_data:
    driver: local
  moodle_data:
    driver: local
  moodledata_data:
    driver: local
🚀 2. Levantar el entorno
Ejecuta los siguientes comandos desde la carpeta del proyecto:

Bash

docker compose up -d
docker ps
Cuando ambos contenedores estén activos (mariadb y moodle), abre en tu navegador:

http://localhost:8080
o
IP:Port
🧠 Usuario por defecto:

Usuario: admin

Contraseña: "Admin123!"

⚒️ 3. Modificación del archivo php.ini
📍 Ubicación dentro del contenedor
/opt/bitnami/php/etc/php.ini
📤 Copiar desde el contenedor a tu máquina, para tener un backup
Bash

docker cp prueba-moodle-1:/opt/bitnami/php/etc/php.ini .
mv php.ini old-php.ini
📝 Editar parámetros recomendados
Ini, TOML

upload_max_filesize = 100M
post_max_size = 100M
max_execution_time = 300
o copiar el php.ini que esta en este repositorio que ya esta modificado

📥 Subir el archivo de nuevo al contenedor
Bash

docker cp php.ini prueba-moodle-1:/opt/bitnami/php/etc/php.ini
docker restart prueba-moodle-1
💡 También puedes entrar directamente:

Bash

docker exec -it prueba-moodle-1 bash
nano /opt/bitnami/php/etc/php.ini
🎓 4. Configuración inicial en Moodle
Inicia sesión con el usuario administrador.  
![Inicio de Moodle](img/imagen1.png)
![Inicio de Moodle2](docker/imagen2.png)
Desde el panel principal, selecciona “My Courses” → “Create Course”.  
![My Courses](docker/imagen3.png)
![Courses Form](docker/imagen4.png)

Completa los datos del curso y guarda.

👥 5. Gestión de usuarios y roles
➕ Crear un nuevo usuario
https://docs.moodle.org/400/en/Admin_quick_guide
![Users](docker/imagen5.png)
Ir a Site administration → Users → Add a new user  
![Users Form](docker/imagen6.png)

![Users Display](docker/imagen7.png)
Completar los datos y guardar.

🧩 Asignar roles
Entra a Home → Participants  
![Users Display Role](docker/imagen8.png)
Edita el usuario y asigna un rol (Teacher, Student, etc.)
![Display Roles](docker/imagen9.png)
📚 6. Inscribir usuarios a un curso
Accede al curso desde My Courses.  
![My Course](docker/imagen10.png)
Ve a Participants → Enrol users.  
![My Course](docker/imagen11.png)
Selecciona los usuarios y define su rol.  
![My Course](docker/imagen12.png)
Guarda con Enrol users.



docker volume prune
🧠 Notas útiles
Logs del docker completo:   bash   docker compose logs  

Logs de Moodle:   bash   docker compose logs -f moodle  

Logs de MariaDB:   bash   docker compose logs -f mariadb  

Entrar al contenedor:   bash   docker exec -it prueba-moodle-1 bash  

👨‍💻 Autor
Brayan Sierra   📘 Proyecto: Moodle en Docker con MariaDB   🖥️ Sistema base: Ubuntu Server 22.04   📅 Fecha: 2025-11-12
