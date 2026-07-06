# Infraestructura AWS — Innovatech Chile EP3

## Proveedor Cloud
- **Proveedor:** Amazon Web Services (AWS)
- **Región:** us-east-1 (N. Virginia)
- **Cuenta:** 830985015694

## VPC (Virtual Private Cloud)
| Parámetro | Valor |
|-----------|-------|
| Nombre | VPC-Innovatech-EP3 |
| ID | vpc-07d2e569138c72393 |
| CIDR | 10.0.0.0/16 |
| Region | us-east-1 |

## Subredes
| Nombre | ID | CIDR | Zona | Tipo |
|--------|----|------|------|------|
| Subred-Publica-1 | subnet-0cc101261080ea33e | 10.0.1.0/24 | us-east-1a | Pública |
| Subred-Publica-2 | subnet-05d486f921fea24e5 | 10.0.2.0/24 | us-east-1b | Pública |
| Subred-Privada-1 | subnet-0ac72d1d32edb448b | 10.0.3.0/24 | us-east-1a | Privada |
| Subred-Privada-2 | subnet-0c17e156c7dcf08d9 | 10.0.4.0/24 | us-east-1b | Privada |

## Gateways y Routing
| Componente | ID | Descripción |
|------------|----|-------------|
| Internet Gateway | igw-045bf061bd5378622 | Acceso a internet para subredes públicas |
| NAT Gateway | nat-0e41e484414bce7a7 | Salida a internet para subredes privadas |
| RT Pública | rtb-086c6d4ea8e63a077 | Ruta 0.0.0.0/0 → IGW |
| RT Privada | rtb-0fa9e22131e4494db | Ruta 0.0.0.0/0 → NAT |

## Cluster EKS
| Parámetro | Valor |
|-----------|-------|
| Nombre | innovatech-eks |
| Versión Kubernetes | 1.36 |
| Región | us-east-1 |
| Endpoint | Público y Privado |
| Namespace | innovatech |

## Node Group
| Parámetro | Valor |
|-----------|-------|
| Nombre | innovatech-node-eks |
| Tipo de instancia | t3.large |
| Tipo de capacidad | SPOT |
| Mínimo nodos | 1 |
| Máximo nodos | 3 |
| Deseado | 2 |
| AMI | Amazon Linux 2023 (x86_64) |

## Roles IAM
| Rol | ARN | Uso |
|-----|-----|-----|
| LabEksClusterRole | arn:aws:iam::830985015694:role/c215895a5456970l15679075t1w830985-LabEksClusterRole-8zrZtx1rn3TQ | Control plane EKS |
| LabEksNodeRole | arn:aws:iam::830985015694:role/c215895a5456970l15679075t1w830985015-LabEksNodeRole-yicQLTFYGixd | Nodos worker EKS |

## Amazon ECR (Registro de imágenes)
| Repositorio | URI |
|-------------|-----|
| frontend-innovatech | 830985015694.dkr.ecr.us-east-1.amazonaws.com/frontend-innovatech |
| backend-ventas-innovatech | 830985015694.dkr.ecr.us-east-1.amazonaws.com/backend-ventas-innovatech |
| backend-despachos-innovatech | 830985015694.dkr.ecr.us-east-1.amazonaws.com/backend-despachos-innovatech |

## Complementos EKS instalados
| Complemento | Versión | Descripción |
|-------------|---------|-------------|
| CoreDNS | v1.14.2-eksbuild.4 | DNS interno del cluster |
| kube-proxy | v1.36.0-eksbuild.7 | Red de servicios |
| vpc-cni | v1.21.2-eksbuild.2 | Red de pods |
| metrics-server | v0.8.1-eksbuild.11 | Métricas para HPA |

## Servicios Kubernetes desplegados
| Servicio | Tipo | Puerto | Descripción |
|---------|------|--------|-------------|
| frontend | LoadBalancer | 80 | Frontend React + Nginx |
| backend-ventas | ClusterIP | 8080 | API REST ventas |
| backend-despachos | ClusterIP | 8081 | API REST despachos |
| mysql | ClusterIP | 3306 | Base de datos MySQL |

## Autoscaling (HPA)
| Componente | Mín réplicas | Máx réplicas | Umbral CPU |
|------------|-------------|-------------|------------|
| frontend-hpa | 2 | 5 | 50% |
| backend-ventas-hpa | 2 | 5 | 50% |
| backend-despachos-hpa | 2 | 5 | 50% |

## URL pública
http://a3d200fd83d1d4331b8e0719ab07f5eb-1428136049.us-east-1.elb.amazonaws.com

## Arquitectura de red
Internet
↓
Internet Gateway (igw-045bf061bd5378622)
↓
LoadBalancer (ALB generado por EKS)
↓
Subredes Públicas (10.0.1.0/24, 10.0.2.0/24)
↓
Pods Frontend (Nginx proxy inverso)
↓ DNS interno Kubernetes
Pods Backend Ventas (10.0.3.0/24, 10.0.4.0/24)
Pods Backend Despachos
↓
Pod MySQL

## Principio de mínimo privilegio
- Frontend: único componente expuesto a internet via LoadBalancer
- Backends: accesibles solo internamente via ClusterIP
- MySQL: accesible solo dentro del cluster via ClusterIP
- Roles IAM: uso de roles predefinidos del laboratorio sin permisos adicionales