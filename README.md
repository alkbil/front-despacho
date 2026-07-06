# Frontend — Innovatech Chile

Interfaz de usuario para el sistema de gestión de ventas y despachos de Innovatech Chile.

## Tecnologías
- React + Vite
- TailwindCSS
- Nginx (servidor web en producción)
- Docker (multi-stage build)

## Arquitectura
El Frontend corre en un contenedor Docker con Nginx que actúa como proxy inverso hacia los Backends:
- `/ventas/` → Backend Ventas (puerto 8080)
- `/despachos/` → Backend Despachos (puerto 8081)

## Estructura del repositorio
front_despacho/
├── src/                    # Código fuente React
├── k8s/                    # Manifiestos Kubernetes
│   ├── namespace.yaml      # Namespace innovatech
│   ├── frontend-deployment.yaml
│   ├── frontend-service.yaml
│   └── frontend-hpa.yaml
├── .github/workflows/      # Pipeline CI/CD
│   └── deploy.yml
├── Dockerfile              # Multi-stage build
├── nginx.conf              # Configuración proxy inverso
└── docker-compose.yml      # Desarrollo local

## Pipeline CI/CD
El pipeline se activa automáticamente con push a `main` o `deploy`:
1. **Build** → Construye imagen Docker con multi-stage
2. **Push** → Publica imagen en Amazon ECR
3. **Apply** → Aplica manifiestos Kubernetes en EKS
4. **Deploy** → Ejecuta rollout restart en el cluster

## Infraestructura AWS
- **Cluster EKS:** `innovatech-eks` (us-east-1)
- **Namespace:** `innovatech`
- **ECR:** `830985015694.dkr.ecr.us-east-1.amazonaws.com/frontend-innovatech`
- **Service:** LoadBalancer público
- **HPA:** mín 2 réplicas, máx 5, umbral CPU 50%

## Secrets requeridos en GitHub
| Secret | Descripción |
|--------|-------------|
| `AWS_ACCESS_KEY_ID` | Credencial AWS |
| `AWS_SECRET_ACCESS_KEY` | Credencial AWS |
| `AWS_SESSION_TOKEN` | Token de sesión AWS |

## Despliegue local
```bash
docker-compose up --build
```

## Acceso público
http://a3d200fd83d1d4331b8e0719ab07f5eb-1428136049.us-east-1.elb.amazonaws.com

