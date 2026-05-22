# AWS Certified Solutions Architect — SAA-C03

![AWS](https://img.shields.io/badge/AWS-Certified-FF9900?style=for-the-badge&logo=amazonwebservices&logoColor=white)
![Solutions Architect](https://img.shields.io/badge/Solutions-Architect-232F3E?style=for-the-badge&logo=amazonwebservices&logoColor=white)
![Idioma](https://img.shields.io/badge/Idioma-Español-blue?style=for-the-badge)
![Estado](https://img.shields.io/badge/Estado-Activo-brightgreen?style=for-the-badge)

> Material de preparación para el examen **AWS Certified Solutions Architect – Associate (SAA-C03)** en español.
> Elaborado por **Rodrigo Aguilar** — Cloud Architect & AWS Authorized Instructor LATAM.

---

## 📋 Contenido del Repositorio

```
aws-saa-c03-prep/
├── README.md                              ← Este archivo
└── guia-teorica/                          ← Marco teórico completo SAA-C03
```

---

## 🗺️ Dominios del Examen SAA-C03

| Dominio | Descripción | Peso |
|---------|-------------|------|
| 1 | Diseño de Arquitecturas Seguras | ~30% |
| 2 | Diseño de Arquitecturas Resilientes | ~26% |
| 3 | Diseño de Arquitecturas de Alto Rendimiento | ~24% |
| 4 | Diseño de Arquitecturas Rentables | ~20% |

---

## 🔑 Patrones de Respuesta Frecuentes

### Almacenamiento
| Escenario | Servicio Correcto |
|-----------|-------------------|
| Archivos compartidos entre EC2 Windows | Amazon FSx for Windows File Server |
| Archivos compartidos entre EC2 Linux | Amazon EFS |
| Alta performance / HPC / gaming | Amazon FSx for Lustre |
| Backup centralizado multi-servicio | AWS Backup |
| Datos NFS on-premises → AWS | S3 File Gateway |
| Block storage on-premises → AWS | Volume Gateway |
| Datos fríos, acceso inmediato | S3 Glacier Instant Retrieval |
| Datos con acceso impredecible | S3 Intelligent-Tiering |
| Copia secundaria, acceso infrecuente | S3 Standard-IA |

### Base de Datos
| Escenario | Servicio Correcto |
|-----------|-------------------|
| Migración heterogénea con CDC | AWS SCT + AWS DMS |
| Poblar entorno staging sin impacto | Aurora Fast Clone |
| Connection pooling para RDS | Amazon RDS Proxy |
| Datos de series de tiempo / IoT | Amazon Timestream |
| Session state de aplicaciones web | ElastiCache Redis |
| Retención backups > 35 días | AWS Backup |
| Compartir snapshot cifrado entre cuentas | Copia snapshot + CMK key policy |
| Dev/Test intermitente (PostgreSQL) | Aurora Serverless v2 |

### Red y Conectividad
| Escenario | Servicio Correcto |
|-----------|-------------------|
| EC2 privado → S3 sin internet | VPC Gateway Endpoint |
| EC2 privado → internet saliente | NAT Gateway (en subnet pública) |
| On-premises → AWS temporal/barato | Site-to-Site VPN |
| Usuario remoto → VPC | AWS Client VPN |
| Multi-cuenta, restricciones centralizadas | SCP en AWS Organizations |
| DR multiregional Active-Passive | Route 53 Failover Routing Policy |

### Compute y Escalado
| Escenario | Servicio Correcto |
|-----------|-------------------|
| Carga predecible y recurrente | ASG Scheduled Action |
| Carga impredecible | ASG Dynamic Scaling |
| EC2 + Fargate en Savings Plan | Compute Savings Plan (no EC2 Instance SP) |
| EKS sin gestionar nodos | AWS Fargate para EKS |
| Separar hardware entre instancias | Spread Placement Group |
| Procesamiento máximo throughput entre nodos | Cluster Placement Group |

### Seguridad e IAM
| Escenario | Servicio Correcto |
|-----------|-------------------|
| EC2 accede a servicios AWS | IAM Role + Instance Profile |
| ECS task accede a servicios AWS | IAM Task Role |
| Credenciales BD con rotación automática | AWS Secrets Manager |
| Forzar HTTPS en S3 | Bucket policy aws:SecureTransport |
| Cifrado en reposo S3 con control de claves | SSE-KMS (CMK) |
| Auditar permisos IAM multi-cuenta | IAM Access Analyzer |
| Detectar cambios en recursos | AWS Config |
| Notificar eventos de API | CloudTrail → EventBridge → SNS |

### Messaging y Streaming
| Escenario | Servicio Correcto |
|-----------|-------------------|
| Orden garantizado de mensajes | SQS FIFO |
| Múltiples consumidores Kinesis sin compartir throughput | Enhanced Fan-Out |
| Cola SQS acumulada → escalar consumidores | ASG basado en ApproximateNumberOfMessagesVisible |
| Procesamiento duplicado en SQS | Aumentar visibility timeout (ChangeMessageVisibility) |
| RabbitMQ/ActiveMQ administrado | Amazon MQ |

---

## 🎯 Claves Conceptuales para el Examen

### Los "siempre" del SAA-C03

- **Siempre** que EC2 necesite acceder a servicios AWS → IAM Role + Instance Profile, nunca access keys
- **Siempre** que ECS task necesite acceder a servicios AWS → Task IAM Role, nunca instance profile
- **Siempre** que EC2 privado acceda a S3/DynamoDB → VPC Gateway Endpoint, no NAT Gateway
- **Siempre** que haya migración heterogénea de BD con CDC → AWS SCT + AWS DMS
- **Siempre** que el escenario diga "mínimo overhead" + backup multi-servicio → AWS Backup
- **Siempre** que haya rotación automática de credenciales BD → AWS Secrets Manager (no SSM Parameter Store)
- **Siempre** que se mencione RabbitMQ/ActiveMQ → Amazon MQ
- **Siempre** que aplicación en EKS deba enrutar por path → AWS Load Balancer Controller + ALB

### Diferencias clave que el examen pone a prueba

| Par confuso | Diferencia |
|-------------|------------|
| ALB vs NLB | ALB = HTTP/HTTPS capa 7; NLB = TCP/UDP capa 4 (VoIP, gaming) |
| SQS FIFO vs Standard | FIFO = orden garantizado; Standard = mayor throughput sin orden |
| Secrets Manager vs SSM Parameter Store | Secrets Manager tiene rotación nativa para RDS |
| One Zone-IA vs Standard-IA | One Zone = 1 AZ (datos reproducibles); Standard = multi-AZ |
| Glacier Instant vs Flexible | Instant = milisegundos; Flexible = minutos a horas |
| Cognito User Pool vs Identity Pool | User Pool = autenticación; Identity Pool = credenciales AWS |
| EC2 Instance SP vs Compute SP | Compute SP cubre EC2 + Fargate + Lambda |
| Client VPN vs Site-to-Site VPN | Client = usuario→VPC; Site-to-Site = red→red |
| gp2 vs gp3 | gp3 = IOPS independiente del tamaño, más barato |

---

## 🚀 Cómo usar este repositorio

1. **Repaso rápido**: Usa las tablas de referencia de este README como cheatsheet
2. **Patrones débiles**: Identifica los dominios donde tienes dudas y repasa los patrones correspondientes

---

## 📄 Licencia

Material elaborado para uso educativo. Basado en la guía oficial del examen AWS Certified Solutions Architect – Associate (SAA-C03).

> **Nota:** Este repositorio es material de estudio independiente, no afiliado ni endorsado por Amazon Web Services. AWS y los nombres de certificaciones son marcas registradas de Amazon.com, Inc.

---

*Última actualización: Mayo 2026*
