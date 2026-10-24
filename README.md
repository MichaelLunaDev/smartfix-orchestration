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
| datos-ejemplo | imagen mysql:8.4 (corre una vez y termina) | - |

## Uso

Clonar todos los repositorios en la misma carpeta:

    git clone https://github.com/karengvc/smartfix-auth-service_1.git
    git clone https://github.com/karengvc/smartfix-equipment-service_2.git
    git clone https://github.com/karengvc/smartfix-incident-service_3.git
    git clone https://github.com/karengvc/smartfix-maintenance-service_4.git
    git clone https://github.com/karengvc/smartfix-frontend_5.git
    git clone https://github.com/MichaelLunaDev/smartfix-orchestration.git

Levantar:

    cd smartfix-orchestration
    docker compose up --build

Abrir http://localhost:4200

Para detener: `docker compose down`. Para borrar tambien los datos y empezar de
cero: `docker compose down -v`.

## Seguridad y roles

El servicio auth-service entrega un token JWT al iniciar sesion. El frontend lo
envia en cada peticion y cada servicio lo valida con la clave compartida
`JWT_SECRET` (variable de entorno).

| Rol | Puede hacer |
|---|---|
| ADMINISTRADOR | Todo: recibir equipos en mostrador, asignar y reasignar tecnicos, corregir estados, administrar equipos y usuarios |
| TECNICO | Ver los casos en espera y tomar uno (no puede soltarlo), avanzar solo sus casos, registrar mantenimientos |
| USUARIO (cliente) | Registrar sus equipos, reportar fallas y seguir el avance. Solo ve lo suyo |

## Flujo de un caso

1. El equipo llega **en linea** (el cliente reporta la falla) o **en mostrador** (el administrador lo recibe). Queda en estado `RECIBIDO`, en espera.
2. Un tecnico lo **toma** y pasa a `EN_ANALISIS`. Desde ahi solo el administrador puede reasignarlo.
3. El tecnico lo avanza a `EN_REPARACION` y luego a `FINALIZADO`.
4. Si el cliente lo devuelve porque la falla sigue, indica el motivo y el caso pasa a `DEVUELTO` (vuelve a la fila de espera).
5. Si el equipo quedo bien pero tiene otro problema distinto, el cliente reporta un caso nuevo.

Cada cambio de estado pide confirmacion y queda en el historial.

## Comunicacion entre servicios

Al registrar o cambiar de equipo una incidencia, incident-service consulta a
equipment-service con un cliente Feign (por la red `smartfix-net`). Si el equipo
no existe, esta dado de baja o no pertenece al cliente, el caso se rechaza. El token del usuario se
reenvia en esa llamada.

## Datos de ejemplo

El contenedor `datos-ejemplo` carga una sola vez usuarios, equipos, incidencias y
mantenimientos de prueba (archivo `db/seed.sql`).

| Cuenta | Contrasena | Rol |
|---|---|---|
| admin@smartfix.com | Admin123 | ADMINISTRADOR |
| luis@smartfix.com, ana@smartfix.com, carlos@smartfix.com | Tecnico123 | TECNICO |
| 15 clientes: maria, jorge, lucia, diego, valeria, andres, camila, renzo, sofia, rodrigo, fiorella, mateo, daniela, piero, ximena (todos @smartfix.com) | Cliente123 | USUARIO |

Los datos incluyen 44 equipos (2 a 4 por cliente) y 31 casos en todos los estados.

## Red

Todos los contenedores comparten la red `smartfix-net`. El frontend (nginx)
reenvia `/api/auth`, `/api/equipos`, `/api/incidencias` y `/api/mantenimientos`
a cada microservicio por su nombre de contenedor. MySQL contiene una base de
datos por servicio (smartfix_auth, smartfix_equipment, smartfix_incidentes_db,
smartfix_maintenance).

La contrasena de MySQL se puede cambiar con la variable `DB_PASSWORD` y la clave
del token con `JWT_SECRET`.
