# SmartFix - Orquestacion

Docker Compose que levanta todo el sistema SmartFix (gestion de incidencias y
mantenimiento de equipos tecnologicos) en una sola red.

## Servicios

| Servicio | Repositorio | Puerto |
|---|---|---|
| auth-service | smartfix-auth-service_1 | 8081 |
| equipment-service | smartfix-equipment-service_2 | 8082 |
| incident-service | smartfix-incident-service_3 | 8083 |
| maintenance-service | smartfix-maintenance-service_4 | 8084 |
| frontend (Angular + nginx) | smartfix-frontend_5 | 4200 |
| mysql | imagen mysql:8.4 | interno |

## Uso

Clonar todos los repositorios en la misma carpeta:

    git clone https://github.com/karengvc/smartfix-auth-service_1.git
    git clone https://github.com/karengvc/smartfix-equipment-service_2.git
    git clone https://github.com/karengvc/smartfix-incident-service_3.git
    git clone https://github.com/karengvc/smartfix-maintenance-service_4.git
    git clone https://github.com/karengvc/smartfix-frontend_5.git
    git clone <este repositorio> smartfix-orchestration

Levantar:

    cd smartfix-orchestration
    docker compose up --build

Abrir http://localhost:4200

Para detener: `docker compose down` (agregar `-v` para borrar los datos).

## Red

Todos los contenedores comparten la red `smartfix-net`. El frontend (nginx)
reenvia `/api/auth`, `/api/equipos`, `/api/incidencias` y `/api/mantenimientos`
a cada microservicio por su nombre de contenedor. MySQL contiene una base de
datos por servicio (smartfix_auth, smartfix_equipment, smartfix_incidentes_db,
smartfix_maintenance).

La contrasena de MySQL se puede cambiar con la variable `DB_PASSWORD`.
