# 🧠 Proyecto Users API – Arquitectura de Microservicios en AWS EKS

## 📘 Descripción General

Este proyecto implementa una **arquitectura de microservicios** utilizando **Django (backend)**, **Flask (microservicio de notificaciones)** y **React (frontend)**.  
Todo el sistema se encuentra **contenedorizado con Docker** y puede desplegarse tanto de forma local mediante `docker-compose`, como en la nube (AWS EKS).

El objetivo principal es construir una infraestructura modular, escalable y automatizable para la **gestión de usuarios** y el **envío de notificaciones automáticas por correo** al administrador.

---

## 🧱 Estructura del Proyecto

```
users_api/
│
├── backend/                  # API principal en Django + PostgreSQL
│   ├── users/                # App principal
│   ├── users_project/        # Configuración global
│   ├── Dockerfile            # Imagen del backend
│   ├── requirements.txt      # Dependencias de Django
│   ├── manage.py
│   └── .env
│
├── notification_service/     # Microservicio de notificaciones (Flask)
│   ├── app.py
│   ├── requirements.txt
│   └── Dockerfile
│
├── frontend/                 # Interfaz en React
│   ├── src/
│   ├── public/
│   ├── Dockerfile
│   └── package.json
│
├── docker-compose.yml        # Orquestación local
└── README.md                 # Este archivo
```

---

## ⚙️ Tecnologías Utilizadas

| Capa | Tecnología | Descripción |
|------|-------------|-------------|
| Backend | **Django REST Framework** | API para CRUD de usuarios |
| Microservicio | **Flask** | Envío de correos automáticos |
| Frontend | **React.js** | Interfaz para gestión de usuarios |
| Base de datos | **PostgreSQL** | Almacenamiento persistente |
| Infraestructura | **Docker & Docker Compose** | Contenedorización y orquestación local |
| Despliegue | **AWS EKS / ECR** | Infraestructura escalable en la nube |

---

## 🚀 Ejecución Local con Docker Compose

### 1️⃣ Clonar el repositorio
```bash
git clone https://github.com/<tu-usuario>/<nombre-repo>.git
cd users_api
```

### 2️⃣ Construir los contenedores
```bash
docker compose build
```

### 3️⃣ Levantar los servicios
```bash
docker compose up
```

📍 Por defecto, los servicios se ejecutan en:
- **Frontend:** http://localhost:3000  
- **Backend (API):** http://localhost:8000/api/users/  
- **Microservicio Flask:** http://localhost:5000/notify  
- **PostgreSQL:** puerto 5432 interno en Docker  

---

## 💾 Variables de Entorno

Cada módulo posee su propio archivo `.env`.  
Ejemplo para el **backend (.env)**:

```env
DEBUG=True
DB_NAME=usersdb
DB_USER=admin
DB_PASSWORD=admin123
DB_HOST=db
DB_PORT=5432

EMAIL_BACKEND=django.core.mail.backends.smtp.EmailBackend
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USE_TLS=True
EMAIL_HOST_USER=tu_correo@gmail.com
EMAIL_HOST_PASSWORD=tu_contraseña_de_aplicacion
DEFAULT_FROM_EMAIL=tu_correo@gmail.com
ADMIN_EMAIL=admin@empresa.com
NOTIFICATION_SERVICE_URL=http://notification_service:5000
```

---

## 🧩 Funcionalidades

### ✅ Backend (Django)
- Creación, listado y eliminación de usuarios.  
- Comunicación con el microservicio Flask.  
- Envío automático de correo de confirmación al administrador.  
- Base de datos PostgreSQL persistente.

### ✅ Microservicio de Notificaciones (Flask)
- Recibe las notificaciones del backend.  
- Imprime logs y envía correos mediante SMTP.  
- Corre en puerto `5000`.

### ✅ Frontend (React)
- Formulario para registrar usuarios.  
- Tabla de usuarios con búsqueda y filtrado dinámico.  
- Opción para eliminar usuarios.  
- Conexión en tiempo real al backend Django.

---

## 🧰 Comandos útiles

| Acción | Comando |
|--------|----------|
| Construir imágenes | `docker compose build` |
| Iniciar servicios | `docker compose up` |
| Detener servicios | `docker compose down` |
| Ver logs del backend | `docker logs users_api` |
| Entrar al contenedor backend | `docker exec -it users_api bash` |

---

## ☁️ Próximos pasos: Despliegue en AWS EKS

- Subir imágenes al **Amazon Elastic Container Registry (ECR)**  
- Crear clúster y despliegue en **EKS**  
- Implementar balanceo con **Load Balancer**  
- Monitorear logs con **AWS CloudWatch**

---

## 👨‍💻 Autor

**Nicolás Ferreira**  
📍 UTEC - Tecnólogo en Informática  
Materia: *Administración de Infraestructuras*  
Año: 2025

---

> 💬 Proyecto académico basado en una arquitectura de microservicios contenedorizados, con comunicación entre capas y despliegue en entorno cloud (AWS).
