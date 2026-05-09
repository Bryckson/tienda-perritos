# Tienda Perritos 🐕

Aplicación de tienda de perritos con frontend, backend y base de datos usando Docker.

## Estructura del Proyecto

```
tienda-perritos/
├── backend/           # API Node.js
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
├── db/               # Base de datos
│   ├── Dockerfile
│   └── init.sql
├── frontend/         # Interfaz web
│   ├── Dockerfile
│   ├── app.js
│   ├── default.conf
│   ├── index.html
│   └── styles.css
└── docker-compose.yml
```

## Requisitos

- Docker
- Docker Compose

## Uso

### Ejecutar con Docker Compose

```bash
docker-compose up -d
```

### Acceder a la aplicación

- Frontend: http://localhost:3000
- Backend API: http://localhost:5000
- Base de datos: localhost:5432

## Servicios

### Frontend
- Servidor Nginx
- Aplicación web React/Vanilla JS
- Puerto: 3000

### Backend
- Node.js con Express
- API REST
- Puerto: 5000

### Base de Datos
- PostgreSQL
- Puerto: 5432

## Desarrollo

Para desarrollo local, consulta la documentación de cada servicio en su respectiva carpeta.
