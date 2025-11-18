# Sistema de Gestión de Usuarios - Arquitectura de Microservicios

> Aplicación Full-Stack desplegada en AWS EKS con arquitectura de microservicios


## Descripción

Sistema completo de gestión de usuarios implementado con arquitectura de microservicios, desplegado en Amazon EKS (Elastic Kubernetes Service). El proyecto incluye un frontend en React, un backend con Django REST Framework, un microservicio de notificaciones con Flask, y una base de datos PostgreSQL en AWS RDS.

**Características principales:**
- Arquitectura de microservicios escalable
- Orquestación con Kubernetes en AWS EKS
- Base de datos PostgreSQL gestionada (RDS)
- API REST expuesta mediante API Gateway
- Notificaciones por correo electrónico
- Alta disponibilidad con múltiples réplicas
- Seguridad implementada con SSDLC
- Contenedores Docker optimizados para producción

## Arquitectura del Sistema

```
Internet
   │
   ├─→ Frontend LoadBalancer → React App (2 réplicas)
   │
   └─→ API Gateway → Backend LoadBalancer → Django API (2 réplicas)
                                                 │
                                                 ├─→ RDS PostgreSQL
                                                 │
                                                 └─→ Notification Service (2 réplicas) → Gmail SMTP
```

### Componentes

| Componente | Tecnología | Puerto | Tipo de Servicio |
|------------|------------|--------|------------------|
| **Frontend** | React 18 + NGINX | 80 | LoadBalancer (público) |
| **Backend** | Django 4.2 + DRF | 8000 | LoadBalancer + API Gateway |
| **Notification Service** | Flask | 5000 | ClusterIP (privado) |
| **Base de Datos** | PostgreSQL 15.4 | 5432 | RDS (privada) |

## Estructura de Repositorios

Este proyecto está organizado en múltiples repositorios:

```
infraestructura/                    # Este repositorio
├── docs/                           # Documentación completa
├── diagrams/                       # Diagramas de arquitectura
└── scripts/                        # Scripts de utilidad

users-backend/                      # Backend Django REST API
├── users_project/                  # Configuración del proyecto
├── users/                          # App de usuarios
└── Dockerfile

users-frontend/                     # Frontend React
├── src/
├── public/
└── Dockerfile.prod

notification-service/               # Microservicio de notificaciones
├── app.py
└── Dockerfile

k8s-manifiests/                     # Manifiestos de Kubernetes
├── backend/
├── frontend/
└── notification/
```

**Enlaces a los repositorios:**
- [Backend (Django)](https://github.com/NicoBonilla373/users-backend)
- [Frontend (React)](https://github.com/NicoBonilla373/users-frontend)
- [Notification Service (Flask)](https://github.com/NicoBonilla373/notification-service)
- [Manifiestos Kubernetes](https://github.com/NicoBonilla373/k8s-manifiests)

## Stack Tecnológico

### Frontend
- **React 18** - Librería de UI
- **Axios** - Cliente HTTP
- **NGINX Alpine** - Servidor web (producción)

### Backend
- **Django 4.2** - Framework web
- **Django REST Framework 3.14** - API REST
- **PostgreSQL 15.4** - Base de datos
- **Python 3.11** - Lenguaje de programación

### Notification Service
- **Flask 3.0** - Framework web ligero
- **SMTP (Gmail)** - Envío de emails
- **Python 3.11** - Lenguaje de programación

### Infraestructura
- **AWS EKS** - Kubernetes gestionado
- **AWS RDS** - PostgreSQL gestionado
- **AWS API Gateway** - Gateway de APIs
- **AWS ECR** - Registro de contenedores
- **Docker** - Contenedorización
- **Kubernetes 1.34** - Orquestación de contenedores

## Requisitos Previos

- Cuenta de AWS (AWS Academy Learner Lab o cuenta regular)
- AWS CLI configurado
- kubectl instalado
- Docker instalado
- Git

## Instalación y Despliegue

### 1. Clonar el repositorio principal

```bash
git clone https://github.com/NicoBonilla373/infraestructura.git
cd infraestructura
```

### 2. Configurar AWS CLI

```bash
# Configurar credenciales de AWS
aws configure set aws_access_key_id [TU_ACCESS_KEY]
aws configure set aws_secret_access_key [TU_SECRET_KEY]
aws configure set aws_session_token [TU_SESSION_TOKEN]
aws configure set region us-east-1

# Verificar configuración
aws sts get-caller-identity
```

### 3. Crear Base de Datos RDS

```bash
# Variables
VPC_ID=$(aws ec2 describe-vpcs --filters "Name=isDefault,Values=true" --query 'Vpcs[0].VpcId' --output text)

# Crear DB Subnet Group
aws rds create-db-subnet-group \
  --db-subnet-group-name users-db-subnet-group \
  --db-subnet-group-description "Subnet group for users database" \
  --subnet-ids [SUBNET_ID_1] [SUBNET_ID_2]

# Crear instancia RDS
aws rds create-db-instance \
  --db-instance-identifier users-db \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --engine-version 15.4 \
  --master-username masteruser \
  --master-user-password [TU_PASSWORD] \
  --allocated-storage 20
```

### 4. Crear Cluster EKS

Desde la consola de AWS:
1. Ir a **EKS** → **Clusters** → **Create cluster**
2. Nombre: `users-app-cluster`
3. Versión: `1.34`
4. Configurar VPC y subnets
5. Crear node group con 2 nodos `t3.medium`

Configurar kubectl:
```bash
aws eks update-kubeconfig --region us-east-1 --name users-app-cluster
kubectl get nodes
```

### 5. Construir y Subir Imágenes Docker

```bash
# Login a ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin [ACCOUNT_ID].dkr.ecr.us-east-1.amazonaws.com

# Construir imágenes (ver README de cada repositorio)
# Backend
cd users-backend
docker build -t users-backend:latest .
docker tag users-backend:latest [ACCOUNT_ID].dkr.ecr.us-east-1.amazonaws.com/users-backend:latest
docker push [ACCOUNT_ID].dkr.ecr.us-east-1.amazonaws.com/users-backend:latest

# Frontend
cd users-frontend
docker build -f Dockerfile.prod -t users-frontend:prod .
docker tag users-frontend:prod [ACCOUNT_ID].dkr.ecr.us-east-1.amazonaws.com/users-frontend:prod
docker push [ACCOUNT_ID].dkr.ecr.us-east-1.amazonaws.com/users-frontend:prod

# Notification Service
cd notification-service
docker build -t notification-service:latest .
docker tag notification-service:latest [ACCOUNT_ID].dkr.ecr.us-east-1.amazonaws.com/notification-service:latest
docker push [ACCOUNT_ID].dkr.ecr.us-east-1.amazonaws.com/notification-service:latest
```

### 6. Desplegar en Kubernetes

```bash
# Crear namespace
kubectl create namespace users-app

# Crear secrets
kubectl create secret generic db-credentials \
  --from-literal=DB_HOST=[RDS_ENDPOINT] \
  --from-literal=DB_PORT=5432 \
  --from-literal=DB_NAME=postgres \
  --from-literal=DB_USER=masteruser \
  --from-literal=DB_PASSWORD=[PASSWORD] \
  -n users-app

kubectl create secret generic smtp-credentials \
  --from-literal=SMTP_HOST=smtp.gmail.com \
  --from-literal=SMTP_PORT=587 \
  --from-literal=SMTP_USER=[TU_EMAIL] \
  --from-literal=SMTP_PASSWORD=[APP_PASSWORD] \
  -n users-app

# Aplicar manifiestos
kubectl apply -f k8s-manifiests/backend-env-config.yaml
kubectl apply -f k8s-manifiests/backend/
kubectl apply -f k8s-manifiests/frontend/
kubectl apply -f k8s-manifiests/notification/

# Verificar despliegue
kubectl get pods -n users-app
kubectl get svc -n users-app
```

### 7. Configurar API Gateway

```bash
# Ver el script completo en docs/api-gateway-setup.sh
# Crear API REST
API_ID=$(aws apigateway create-rest-api \
  --name users-api \
  --endpoint-configuration types=REGIONAL \
  --query 'id' --output text)

# Configurar recursos y métodos (ver documentación completa)
```

## URLs de Acceso

### Producción (AWS)

**Frontend:**
```
http://k8s-usersapp-usersfro-[ID].elb.us-east-1.amazonaws.com
```

**API Gateway:**
```
https://[API_ID].execute-api.us-east-1.amazonaws.com/prod/users
```

## Monitoreo y Logs

```bash
# Ver estado de pods
kubectl get pods -n users-app

# Ver logs de un servicio
kubectl logs -f deployment/users-backend -n users-app

# Ver logs de notificaciones
kubectl logs -l app=notification-service -n users-app | grep EMAIL

# Ver eventos
kubectl get events -n users-app --sort-by='.lastTimestamp'

# Ver uso de recursos
kubectl top pods -n users-app
kubectl top nodes
```

## Seguridad

El proyecto implementa el ciclo de desarrollo seguro (SSDLC) con las siguientes herramientas:

| Tipo | Herramienta | Descripción |
|------|-------------|-------------|
| **SAST** | Bandit | Análisis estático de código Python |
| **SCA** | Safety, npm audit | Análisis de dependencias vulnerables |
| **Container** | Trivy | Escaneo de vulnerabilidades en imágenes Docker |
| **DAST** | Nikto | Análisis dinámico de aplicación web |

### Ejecutar Análisis de Seguridad

```bash
# SAST - Backend
cd users-backend
bandit -r . -f txt -o bandit-report.txt

# SCA - Backend
safety check

# SCA - Frontend
cd users-frontend
npm audit

# Container Scanning
trivy image [ACCOUNT_ID].dkr.ecr.us-east-1.amazonaws.com/users-backend:latest

# DAST
nikto -h http://[FRONTEND_URL]
```

## Pruebas

### Crear un usuario

```bash
curl -X POST https://[API_ID].execute-api.us-east-1.amazonaws.com/prod/users \
  -H "Content-Type: application/json" \
  -d '{
    "nombre": "Juan Pérez",
    "email": "juan@example.com",
    "telefono": "099123456"
  }'
```

### Listar usuarios

```bash
curl https://[API_ID].execute-api.us-east-1.amazonaws.com/prod/users
```

## Troubleshooting

### Pods en ImagePullBackOff
```bash
# Verificar permisos del node role
NODE_ROLE=$(aws iam list-roles --query "Roles[?contains(RoleName, 'LabEksNodeRole')].RoleName" --output text)
aws iam attach-role-policy \
  --role-name $NODE_ROLE \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly
```

### Backend no se conecta a RDS
```bash
# Verificar security group
NODE_SG=$(aws ec2 describe-instances \
  --filters "Name=tag:eks:cluster-name,Values=users-app-cluster" \
  --query 'Reservations[0].Instances[0].SecurityGroups[0].GroupId' \
  --output text)

aws ec2 authorize-security-group-ingress \
  --group-id [RDS_SG_ID] \
  --protocol tcp \
  --port 5432 \
  --source-group $NODE_SG
```

### Frontend no se conecta al Backend
```bash
# Actualizar config.js con la URL del API Gateway
kubectl rollout restart deployment/users-frontend -n users-app
```



## Autores

- **Nicolás Bonilla** - *Desarrollo e implementación* - [NicoBonilla373](https://github.com/NicoBonilla373)

## Institución

**UTEC - Universidad Tecnológica**
- Curso: Administración de Infraestructuras
- Docentes: Israel Bellizzi y Leonardo Sellanes
- Año: 2025


Link del proyecto: [https://github.com/NicoBonilla373/infraestructura](https://github.com/NicoBonilla373/infraestructura)
