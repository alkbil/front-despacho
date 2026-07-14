# React + Vite

This template provides a minimal setup to get React working in Vite with HMR and some ESLint rules.

Currently, two official plugins are available:

- [@vitejs/plugin-react](https://github.com/vitejs/vite-plugin-react/blob/main/packages/plugin-react/README.md) uses [Babel](https://babeljs.io/) for Fast Refresh
- [@vitejs/plugin-react-swc](https://github.com/vitejs/vite-plugin-react-swc) uses [SWC](https://swc.rs/) for Fast Refresh

## Arquitectura de Despliegue

![Arquitectura Innovatech](./arquitectura.png)

El sistema está compuesto por un Frontend (React/Vite + NGINX), dos microservicios Backend
(Spring Boot: Ventas y Despachos) y una base de datos MySQL, desplegados en un clúster
Amazon EKS dentro de una VPC dedicada con subredes públicas y privadas. El acceso público
se realiza mediante un Network Load Balancer; los backends y la base de datos son
accesibles únicamente desde la red interna del clúster.

## Desarrollo local con Docker Compose

Este repositorio incluye un `docker-compose.yml` que levanta los 4 servicios de forma
local (frontend, backend-ventas, backend-despachos, mysql), útil para desarrollo y pruebas
sin necesidad de desplegar en la nube.
cd C:\Users\Usuario\Documents\Innovatech-EP3\front-despacho
cd C:\Users\Usuario\Documents\Innovatech-EP3\front-despacho

@"

## Arquitectura de Despliegue

![Arquitectura Innovatech](./arquitectura.png)

El sistema esta compuesto por un Frontend (React/Vite + NGINX), dos microservicios Backend
(Spring Boot: Ventas y Despachos) y una base de datos MySQL, desplegados en un cluster
Amazon EKS dentro de una VPC dedicada con subredes publicas y privadas. El acceso publico
se realiza mediante un Network Load Balancer; los backends y la base de datos son
accesibles unicamente desde la red interna del cluster.

## Desarrollo local con Docker Compose

Este repositorio incluye un docker-compose.yml que levanta los 4 servicios de forma
local (frontend, backend-ventas, backend-despachos, mysql).

docker-compose up --build

- Frontend: http://localhost
- Backend Ventas: http://localhost:8080/swagger-ui.html
- Backend Despachos: http://localhost:8081/swagger-ui.html
- MySQL: localhost:3306

## Despliegue en AWS EKS

- Cluster: Innovatech-eks (Kubernetes v1.36, EKS Auto Mode)
- Namespace: innovatech
- Registro de imagenes: Amazon ECR (frontend-innovatech, backend-ventas-innovatech, backend-despachos-innovatech)
- Manifiestos: ver carpeta k8s/

### Evidencia de funcionamiento

Pods corriendo en el namespace innovatech:

NAME                                 READY   STATUS    RESTARTS   AGE
backend-despachos-5bf7c99c97-b4f6p   1/1     Running   2          98s
backend-despachos-5bf7c99c97-lvkp7   1/1     Running   2          98s
backend-despachos-5bf7c99c97-mt7jk   1/1     Running   2          98s
backend-ventas-6969747957-5jctj      1/1     Running   2          97s
backend-ventas-6969747957-m8hrc      1/1     Running   2          97s
backend-ventas-6969747957-wr47r      1/1     Running   2          97s
frontend-6ffc85887d-4gkqf            1/1     Running   0          97s
frontend-6ffc85887d-xvqft            1/1     Running   0          97s
mysql-6f9f96bc4c-mq7lm               1/1     Running   0          42s

Servicio del Frontend con IP publica:

NAME       TYPE           EXTERNAL-IP
frontend   LoadBalancer   k8s-innovate-frontend-ef0a573955-21eb6b2f69bcb20c.elb.us-east-1.amazonaws.com

Comunicacion Frontend -> Backend -> Base de Datos:
Validada creando datos de prueba reales via Swagger UI:
- POST /api/v1/ventas -> 201 Created (idVenta: 1)
- POST /api/v1/despachos -> 201 Created (idDespacho: 1, idCompra: 1)

## Observabilidad: Logs y Metricas

### Logs de la aplicacion
Se verifico el funcionamiento mediante kubectl logs, detectando y corrigiendo un error
de conexion JDBC (Public Key Retrieval is not allowed) causado por el metodo de
autenticacion por defecto de MySQL 8, resuelto agregando
--default-authentication-plugin=mysql_native_password en el Deployment de MySQL.

### Metricas de escalado (HPA)

NAME                    REFERENCE                      TARGETS              MINPODS   MAXPODS   REPLICAS
backend-despachos-hpa   Deployment/backend-despachos   cpu: unknown/50%   1         3         3
backend-ventas-hpa      Deployment/backend-ventas      cpu: unknown/50%   1         3         3
frontend-hpa            Deployment/frontend            cpu: 1%/50%          2         5         2

### Logs del cluster (CloudWatch)
El cluster tiene habilitado logging completo (api, audit, authenticator, controllerManager,
scheduler) hacia Amazon CloudWatch Logs (/aws/eks/Innovatech-eks/cluster).

## CI/CD Pipeline

El workflow de GitHub Actions ejecuta build -> test -> build de imagen Docker -> push a
Amazon ECR (etiquetada con el SHA del commit) -> despliegue en EKS mediante kubectl apply.
Las credenciales se gestionan mediante GitHub Secrets.
