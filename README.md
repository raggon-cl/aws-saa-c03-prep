# AWS Certified Solutions Architect — SAA-C03

![AWS](https://img.shields.io/badge/AWS-Certified-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Solutions Architect](https://img.shields.io/badge/Solutions_Architect-Associate-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![Idioma](https://img.shields.io/badge/Idioma-Español-blue?style=for-the-badge)
![Estado](https://img.shields.io/badge/Estado-Activo-brightgreen?style=for-the-badge)

> Material de preparación completo para el examen **AWS Certified Solutions Architect Associate (SAA-C03)** en español.  
> Elaborado por **Rodrigo Aguilar** — Cloud Architect & AWS Authorized Instructor LATAM.

---

## Contenido del Repositorio

```
aws-saa-c03-prep/
├── README.md                          ← Este archivo
├── guia-teorica/
│   └── sustento-teorico-completo.md  ← Marco teórico completo SAA-C03
└── practica/
    └── preguntas-practica.md         ← Set de preguntas de práctica
```

---

## Dominios del Examen SAA-C03

Según la guía oficial de AWS, el examen cubre:

| Dominio | Descripción | Peso aproximado |
|---------|-------------|----------------|
| 1 | Diseño de Arquitecturas Seguras | 30% |
| 2 | Diseño de Arquitecturas Resilientes | 26% |
| 3 | Diseño de Arquitecturas de Alto Rendimiento | 24% |
| 4 | Diseño de Arquitecturas Optimizadas en Costo | 20% |

---

## Servicios AWS Clave — Referencia Rápida

| Necesidad | Servicio Correcto |
|-----------|------------------|
| Balanceo de carga HTTP/HTTPS | Application Load Balancer (ALB) |
| Balanceo de carga TCP/UDP | Network Load Balancer (NLB) |
| Distribución de contenido global | Amazon CloudFront |
| DNS y enrutamiento de tráfico | Amazon Route 53 |
| Red privada virtual | Amazon VPC |
| Almacenamiento de objetos | Amazon S3 |
| Almacenamiento de bloques | Amazon EBS |
| Almacenamiento de archivos compartido | Amazon EFS |
| Base de datos relacional gestionada | Amazon RDS |
| Base de datos NoSQL | Amazon DynamoDB |
| Caché en memoria | Amazon ElastiCache |
| Cola de mensajes | Amazon SQS |
| Mensajería pub/sub | Amazon SNS |
| Procesamiento de eventos sin servidor | AWS Lambda |
| Contenedores gestionados | Amazon ECS / EKS |
| Escalado automático | Auto Scaling Groups |
| Identidad y acceso | AWS IAM |
| Cifrado de datos | AWS KMS |
| Monitoreo y métricas | Amazon CloudWatch |
| Auditoría de API calls | AWS CloudTrail |
| Infraestructura como código | AWS CloudFormation |
| Migración de datos | AWS DMS / Snowball |
| Conectividad híbrida | AWS Direct Connect / VPN |
| Data warehouse | Amazon Redshift |
| Búsqueda | Amazon OpenSearch |

---

## Patrones de Arquitectura — Identificación Rápida

| Señal en el enunciado | Solución |
|----------------------|---------|
| "Alta disponibilidad en múltiples zonas" | Multi-AZ + ALB + Auto Scaling |
| "Recuperación ante desastres mínima" | Pilot Light o Warm Standby |
| "Contenido estático a usuarios globales" | S3 + CloudFront |
| "Desacoplamiento de microservicios" | SQS o SNS |
| "Procesamiento sin servidor" | Lambda + API Gateway |
| "Base de datos escalable globalmente" | DynamoDB Global Tables |
| "Datos relacionales con failover automático" | RDS Multi-AZ |
| "Reducir latencia de base de datos" | ElastiCache (Redis/Memcached) |
| "Acceso privado a S3 desde VPC" | VPC Gateway Endpoint |
| "Conectar on-premise con AWS" | Direct Connect o Site-to-Site VPN |
| "Transferencia masiva de datos" | AWS Snowball / Snowmobile |
| "Análisis de logs en tiempo real" | Kinesis Data Streams |
| "Menor costo para cargas intermitentes" | EC2 Spot Instances |
| "Menor costo para cargas predecibles" | EC2 Reserved Instances |

---

## Como usar este repositorio

1. **Lectura teórica**: Comienza con `guia-teorica/sustento-teorico-completo.md`
2. **Práctica**: Usa `practica/preguntas-practica.md` para evaluar tu comprensión
3. **Repaso rápido**: Usa las tablas de este README como cheatsheet antes del examen

---

## Licencia

Material elaborado para uso educativo. Basado en la guía oficial del examen AWS Certified Solutions Architect Associate (SAA-C03).

> **Nota:** Este repositorio es material de estudio independiente, no afiliado ni endorsado por Amazon Web Services. AWS y los nombres de certificaciones son marcas registradas de Amazon.com, Inc.

---

*Última actualización: Mayo 2026*
