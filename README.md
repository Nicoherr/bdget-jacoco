# BDGet JaCoCo - Evaluación Parcial 2 DevOps

Microservicio REST de estudiantes desarrollado con Java 17 y Spring Boot. El proyecto integra contenedores, pruebas automatizadas, análisis de seguridad, orquestación y despliegue continuo.

## Requisitos

- Java 17 para ejecutar Maven localmente.
- Docker Engine y Docker Compose v2 para levantar el entorno completo.

## Ejecutar la aplicación con contenedores

```sh
cp .env.example .env
```

Los valores de `.env.example` son exclusivamente para desarrollo local. Reemplázalos por credenciales seguras en cualquier entorno compartido o desplegado.

```sh
docker compose up --build --wait
```

La aplicación queda disponible en `http://localhost:8080`; la API CRUD está bajo `/students`. Compose inicia MySQL con almacenamiento persistente, dos réplicas de la aplicación y un proxy Nginx que distribuye las solicitudes. Los límites de CPU/memoria, comprobaciones de salud y dependencias de inicio se declaran en `docker-compose.yml`.

Para ajustar el número de réplicas:

```sh
docker compose up -d --scale app=3
```

Para detener los servicios sin borrar los datos:

```sh
docker compose down
```

Usa `docker compose down -v` solo si también quieres eliminar la base de datos local.

## Pruebas y cobertura

```sh
./mvnw --batch-mode clean verify
```

La suite de pruebas usa H2 en memoria para no depender de una base de datos externa. JaCoCo genera `target/site/jacoco/` y hace fallar `verify` si la cobertura de líneas cae por debajo del 50 %.

## Pipeline CI/CD

`.github/workflows/main.yml` ejecuta en pull requests y en cambios a `main`:

1. Pruebas Maven y control de cobertura JaCoCo.
2. Escaneo Snyk de dependencias con bloqueo para vulnerabilidades altas o críticas.
3. Análisis SonarCloud con el reporte JaCoCo y espera del Quality Gate cuando sus credenciales y variables estén configuradas.
4. Construcción de la imagen Docker y escaneo Trivy de vulnerabilidades altas o críticas.
5. Análisis estático CodeQL, revisión de nuevas dependencias en pull requests y actualizaciones semanales de Maven y GitHub Actions con Dependabot.
6. Publicación del reporte de cobertura como artefacto asociado al SHA del commit.
7. Despliegue por SSH a EC2 después de que las validaciones de seguridad y pruebas terminen correctamente.

En GitHub, abre **Settings → Secrets and variables → Actions**. Guarda los tokens `SNYK_TOKEN` y `SONAR_TOKEN` en **Secrets**; crea `SONAR_ORGANIZATION` y `SONAR_PROJECT_KEY` en **Variables** usando los identificadores de tu proyecto en SonarCloud. Hasta que los datos de SonarCloud estén completos, ese análisis muestra una advertencia y se omite; los demás controles siguen ejecutándose. No guardes los tokens en el código ni los compartas.

El despliegue requiere además los secretos `EC2`, `EC2_USER` y `EC2_SSH`. En la instancia, instala Docker Compose v2 y crea `~/bdget-jacoco/.env` con credenciales seguras para `MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER` y `MYSQL_PASSWORD`. Configura también el grupo de seguridad de EC2 para permitir el puerto indicado por `APP_PORT` (8080 por defecto). El pipeline no copia archivos `.env` ni wallets al servidor.

## Criterios de la rúbrica

- **IE1 — Contenedores:** imagen Java multi-stage, ejecución no privilegiada y configuración reproducible.
- **IE2 — Pruebas automatizadas:** CI ejecuta pruebas y bloquea la integración si la cobertura de líneas es menor al 50 %.
- **IE3 — Escalabilidad y seguridad:** límites de recursos, réplicas detrás de Nginx, CodeQL, Snyk, Dependabot, revisión de dependencias y escaneo de la imagen.
- **IE4 — Despliegue y trazabilidad:** despliegue desde `main` tras validaciones y etiqueta OCI de la imagen con el SHA del commit.
- **IE5 — Orquestación:** Docker Compose coordina aplicación, proxy y base de datos con persistencia, salud y orden de inicio.

## Declaración del uso de Inteligencia Artificial

En conformidad con las políticas académicas de Duoc UC:

- **Herramientas utilizadas:** Gemini y Claude (Anthropic).
- **Gemini:** asistencia técnica para depurar errores de permisos SSH, resolver incompatibilidades de Docker Buildx en Amazon Linux 2023 y revisar la sintaxis de `.github/workflows/main.yml`.
- **Claude:** análisis del repositorio y del pipeline CI/CD, identificación de tareas pendientes (SonarCloud, despliegue, Dependabot) y guía para configurar los secretos de GitHub Actions y el despliegue en AWS EC2.
- **Validación:** el código, la infraestructura en AWS y las pruebas fueron revisados y validados directamente por el equipo de estudiantes.

## Equipo

- Nicolás Herrera
- Guillermo Manriquez
- Ingeniería DevOps (DOY0101) — Duoc UC
