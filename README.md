# Innovatech Chile — Frontend

Aplicación web desarrollada en **Python Flask** que permite gestionar usuarios mediante una interfaz visual conectada al Backend API.

## Stack tecnológico

| Componente | Tecnología |
|---|---|
| Frontend | Python 3.11 + Flask |
| Contenedor | Docker (multi-stage, usuario no-root) |
| Orquestación | Docker Compose |
| CI/CD | GitHub Actions → Docker Hub → EC2 |

## Estructura del repositorio

```
Front_Eval2/
├── app.py                          # Aplicación Flask principal
├── templates/                      # Plantillas HTML (Jinja2)
├── requirements.txt                # Dependencias Python
├── Dockerfile                      # Multi-stage build (builder + producción)
├── docker-compose.yml              # Stack completo: frontend + backend + db
├── .env.example                    # Variables de entorno requeridas
└── .github/
    └── workflows/
        └── deploy.yml              # Pipeline CI/CD (trigger: rama deploy)
```

## Variables de entorno

Copiar `.env.example` a `.env` y completar:

```bash
cp .env.example .env
```

| Variable | Descripción | Ejemplo |
|---|---|---|
| `SECRET_KEY` | Clave secreta Flask | `mi_clave_segura` |
| `PORT` | Puerto del servidor | `5000` |
| `DEBUG` | Modo debug | `false` |
| `BACKEND_URL` | URL del Backend API | `http://backend:3000` |
| `MYSQL_ROOT_PASSWORD` | Contraseña root MySQL | `rootpassword` |
| `MYSQL_DATABASE` | Nombre de la base de datos | `proyecto_db` |
| `MYSQL_USER` | Usuario de la BD | `appuser` |
| `MYSQL_PASSWORD` | Contraseña del usuario BD | `apppassword` |

## Ejecución local con Docker Compose

```bash
# 1. Clonar el repositorio
git clone <url-repo-frontend>
cd Front_Eval2

# 2. Configurar variables de entorno
cp .env.example .env
# Editar .env con tus valores

# 3. Levantar el stack completo (frontend + backend + db)
docker compose up -d

# 4. Verificar servicios
docker compose ps

# 5. Acceder a la aplicación
# http://localhost:5000
```

## Ejecución individual (solo frontend)

```bash
docker build -t innovatech-frontend .
docker run -d -p 5000:5000 --env-file .env innovatech-frontend
```

## Pipeline CI/CD

El pipeline se activa automáticamente al hacer push a la rama **deploy**:

```
push rama deploy
       │
       ▼
  [build-and-push]
  ├── Checkout código
  ├── Login Docker Hub
  ├── Build imagen (multi-stage)
  └── Push imagen (tags: latest + SHA commit)
       │
       ▼
     [deploy]
  ├── SSH a EC2 Frontend
  ├── Pull imagen nueva
  └── Restart contenedor con variables de entorno
```

### GitHub Secrets requeridos

Configurar en Settings → Secrets and variables → Actions:

| Secret | Descripción |
|---|---|
| `DOCKERHUB_USERNAME` | Usuario de Docker Hub |
| `DOCKERHUB_TOKEN` | Access token de Docker Hub |
| `EC2_FRONTEND_HOST` | IP pública de la instancia EC2 del frontend |
| `EC2_USER` | Usuario SSH de EC2 (ec2-user o ubuntu) |
| `EC2_SSH_KEY` | Clave privada SSH para conectarse a EC2 |
| `SECRET_KEY` | Clave secreta de Flask |
| `EC2_BACKEND_HOST` | IP privada de la instancia EC2 del backend |

## Persistencia de datos

Los datos de MySQL se almacenan en un **named volume** (`mysql_data`), garantizando que la información no se pierda al reiniciar contenedores. Se eligió named volume sobre bind mount por portabilidad entre entornos.

## Arquitectura de red

```
Internet
   │
   ▼ puerto 5000
[Frontend EC2] ──── frontend_network ────► [Backend EC2:3000]
                                                │
                                        backend_network
                                                │
                                           [MySQL:3306]
```

Solo el Frontend es accesible desde Internet. El Backend y la BD operan en redes internas privadas.
