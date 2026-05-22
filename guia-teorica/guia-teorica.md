# Guía Teórica — AWS Certified Solutions Architect Associate (SAA-C03)

> Marco teórico completo organizado por dominio del examen.
> Elaborado por **Rodrigo Aguilar** — Cloud Architect & AWS Authorized Instructor LATAM.

---

## Dominios del Examen

| Dominio | Descripción | Peso |
|---------|-------------|------|
| 1 | Diseño de Arquitecturas Seguras | ~30% |
| 2 | Diseño de Arquitecturas Resilientes | ~26% |
| 3 | Diseño de Arquitecturas de Alto Rendimiento | ~24% |
| 4 | Diseño de Arquitecturas Rentables | ~20% |

---

## Dominio 1 — Diseño de Arquitecturas Seguras

### 1.1 IAM — Identidad y Acceso

**IAM Role + Instance Profile (EC2)**
- Mecanismo nativo de AWS para otorgar permisos a instancias EC2 sin credenciales estáticas.
- Las credenciales son temporales y se rotan automáticamente vía el metadata service (`169.254.169.254`).
- **Regla absoluta:** EC2 necesita acceder a cualquier servicio AWS → IAM Role + Instance Profile, **nunca** access keys embebidas.

**Task IAM Role (ECS/EKS)**
- Para tareas ECS o pods EKS, el permiso se asigna en la task definition como Task IAM Role.
- El task role aplica solo a ese task específico (mínimo privilegio por contenedor).
- **Diferencia clave:** Instance profile otorga permisos a todos los contenedores del host; task role aplica solo al contenedor específico.

**SCPs — Service Control Policies**
- Aplicadas a OUs (Organizational Units) en AWS Organizations.
- Funciona como guardrail: ningún usuario ni rol en las cuentas puede exceder los permisos que la SCP permite.
- Aplicada en el root OU → afecta **todas** las cuentas de la organización automáticamente.
- **Caso de uso:** Restringir servicios o acciones en múltiples cuentas desde un único punto de control centralizado.

**IAM Access Analyzer**
- Analiza políticas IAM y de recursos para identificar accesos excesivos o no intencionados.
- Se puede configurar a nivel de AWS Organizations para analizar todas las cuentas desde un punto centralizado.
- Genera findings con accesos problemáticos (mínimo overhead administrativo).
- **Diferencia:** Network Access Analyzer analiza conectividad de red, no permisos IAM.

### 1.2 Cifrado y Gestión de Secretos

**SSE-KMS con Customer Managed Keys (CMK)**
- Cifrado en reposo para S3 con control granular sobre quién puede usar las claves.
- Auditoría completa vía CloudTrail y rotación programable.
- El equipo de compliance administra las políticas de claves de KMS.

**Forzar HTTPS en S3**
- Usar condición `aws:SecureTransport: false` → Deny en la bucket policy.
- S3 ya usa HTTPS nativo; ACM **no** se asocia directamente a buckets S3 (sí a ALB, CloudFront, API Gateway).

**AWS Secrets Manager**
- Gestión segura de credenciales con rotación automática nativa para RDS.
- Sin código personalizado: genera nueva contraseña, la actualiza en RDS y actualiza el secreto.
- **Diferencia crítica vs SSM Parameter Store:** Secrets Manager tiene rotación automática integrada para RDS; SSM Parameter Store no.
- **Regla:** RDS + rotación automática de credenciales → siempre Secrets Manager.

**KMS (Key Management Service)**
- Gestiona claves de cifrado, **no** credenciales de aplicación (username/password).
- No tiene concepto de rotación de credenciales de BD.

**Cifrado RDS completo**
- En reposo: KMS managed keys (se habilita al crear la instancia).
- En tránsito: SSL/TLS nativo de RDS (certificado gestionado por AWS).
- ACM gestiona certificados para conexiones EC2 → RDS vía SSL/TLS.
- VPN es para conectividad entre redes, no para cifrar tráfico dentro de una VPC.

### 1.3 Seguridad en S3

**Amazon Macie**
- Descubre y clasifica datos sensibles en S3 (PII, PHI, credenciales).
- **No cifra datos**. Es una herramienta de descubrimiento, no de cifrado.

**Amazon GuardDuty**
- Detección de amenazas mediante análisis de logs (CloudTrail, VPC Flow Logs, DNS).
- No es la fuente autoritativa de eventos específicos de creación/eliminación de usuarios IAM.

**Amazon Inspector**
- Evalúa vulnerabilidades en EC2, imágenes ECR y funciones Lambda.
- **No analiza** políticas IAM ni monitorea eventos de gestión de usuarios.

### 1.4 Autenticación de Usuarios

**Amazon Cognito**

| Componente | Función |
|------------|---------|
| **User Pool** | Autenticación (directorio de usuarios, login/signup) |
| **Identity Pool** | Autorización (intercambia token del User Pool por credenciales AWS temporales) |

- **Regla:** User Pool = quien eres. Identity Pool = qué puedes hacer en AWS.
- Patrón para acceso a S3 desde app: User Pool (login) → Identity Pool (credenciales IAM temporales) → S3.

### 1.5 Monitoreo y Auditoría

**AWS Config**
- Registra continuamente el estado de configuración de recursos AWS.
- Captura cada cambio como un configuration item de forma completamente pasiva (no modifica recursos).
- Permite consultar el historial de configuración: "¿quién cambió qué recurso y cuándo?".
- **Diferencia vs CloudFormation drift detection:** Config es continuo y abarca todos los recursos; drift detection es bajo demanda y solo para stacks de CloudFormation.

**AWS CloudTrail + EventBridge + SNS**
- CloudTrail captura todas las llamadas a la API de AWS (management events).
- EventBridge filtra eventos específicos con event patterns (ej. `CreateUser`, `DeleteUser`).
- SNS entrega notificaciones (email, SMS, HTTP).
- **Patrón de tres pasos:** CloudTrail → EventBridge → SNS/Lambda.

**AWS Control Tower**
- Gestiona el aprovisionamiento de cuentas en Organizations con guardrails.
- Account drift notifications detectan automáticamente cambios en la jerarquía de OUs.
- Las notificaciones se envían automáticamente vía SNS.
- **Caso de uso:** Detectar cambios en jerarquía de OUs en entornos multi-cuenta.

---

## Dominio 2 — Diseño de Arquitecturas Resilientes

### 2.1 Alta Disponibilidad — Patrón Base

El patrón estándar de HA en AWS requiere:
- Mínimo 2 Availability Zones.
- NAT Gateway por AZ (no compartido entre AZs).
- ALB en subnets **públicas**; compute (EC2) y datos (RDS) en subnets **privadas**.
- RDS Multi-AZ para alta disponibilidad de base de datos.
- Auto Scaling Group para escalado y recuperación automática de EC2.

```
Internet → IGW → ALB (subnet pública, 2 AZs)
                    ↓
               EC2 en ASG (subnet privada, 2 AZs)
                    ↓
               RDS Multi-AZ (subnet privada, 2 AZs)

EC2 → NAT GW (subnet pública) → IGW → Internet (salida)
```

### 2.2 RDS — Modos de Despliegue

| Modo | Propósito | ¿Lectura? | ¿Failover? |
|------|-----------|-----------|------------|
| **Single-AZ** | Dev/Test | No | No |
| **Multi-AZ** | Producción HA | **No** (standby no accesible) | Sí, automático |
| **Read Replica** | Lectura/Reporting | Sí | No automático |
| **Multi-AZ + Read Replica** | Producción HA + escala de lectura | Sí | Sí |

- **Regla crítica:** El standby de Multi-AZ **no** es accesible para lectura. Solo sirve para failover.
- Para descargar consultas de reporting → Read Replica.
- Para poblar entornos staging/test sin impacto → Aurora Fast Clone.

### 2.3 Amazon Aurora

**Aurora Fast Clone**
- Crea una copia de la base de datos en segundos usando copy-on-write.
- No duplica datos físicamente → no impacta producción.
- **Caso de uso:** Poblar entornos de desarrollo/staging de forma rápida y sin impacto.

**Aurora vs RDS MySQL/PostgreSQL**
- Aurora es compatible con MySQL y PostgreSQL pero con arquitectura de almacenamiento propia.
- Mayor throughput, replicación más rápida, hasta 15 Read Replicas.
- Aurora Serverless v2 para cargas intermitentes (dev/test PostgreSQL, por ejemplo).

### 2.4 Route 53 — Políticas de Enrutamiento

| Política | Uso |
|----------|-----|
| **Failover** | DR Active-Passive multiregional |
| **Weighted** | Distribución proporcional de tráfico (canary releases) |
| **Latency** | Enrutar al endpoint con menor latencia para el usuario |
| **Geolocation** | Enrutar según país/continente del usuario |
| **Health Checks** | Determinar si un endpoint está disponible |

- **DR Active-Passive multiregional:** Route 53 Failover Routing Policy + Health Checks.
- **DR Active-Active multiregional:** Weighted o Latency Routing.
- Route 53 Resolver: resolución DNS híbrida on-premises ↔ VPC (no es mecanismo de failover).
- Route 53 Profiles: comparte configuraciones DNS entre VPCs (no es failover entre regiones).

### 2.5 Amazon MQ

- Servicio administrado de broker de mensajería compatible con **RabbitMQ** y **ActiveMQ**.
- Par active/standby en dos AZs con failover automático.
- **Regla:** Cuando el escenario mencione RabbitMQ o ActiveMQ → Amazon MQ (menor overhead que EC2).

### 2.6 AWS Control Tower y Organizations

**AWS Organizations**
- Gestiona múltiples cuentas AWS en una jerarquía de OUs.
- SCPs aplicadas en el root OU afectan todas las cuentas automáticamente.

**AWS Control Tower**
- Capa de gobierno sobre Organizations.
- Provisiona cuentas con guardrails pre-configurados.
- Detecta drift (cambios en OUs, cuentas que se desvían de configuración esperada).

---

## Dominio 3 — Diseño de Arquitecturas de Alto Rendimiento

### 3.1 Compute — Auto Scaling

| Tipo de Scaling | Cuándo usar |
|-----------------|-------------|
| **Scheduled Action** | Carga predecible y recurrente (ej. viernes por la noche) |
| **Dynamic Scaling** | Carga impredecible (reacciona a métricas en tiempo real) |
| **Target Tracking** | Mantener una métrica en un valor objetivo (ej. CPU al 60%) |
| **Step Scaling** | Escalar en pasos según rangos de métricas |

- Scheduled Action: se configura con expresión cron, cero intervención manual posterior.
- Métrica para escalar consumidores SQS: `ApproximateNumberOfMessagesVisible`.

### 3.2 EC2 Placement Groups

| Tipo | Característica | Uso |
|------|----------------|-----|
| **Cluster** | Instancias juntas en mismo rack → baja latencia de red | HPC, procesamiento paralelo |
| **Spread** | Instancias en racks distintos → máximo aislamiento de hardware | Aplicaciones críticas, máx. 7 instancias/AZ |
| **Partition** | Grupos de instancias en particiones separadas | Hadoop, Kafka, Cassandra |

- **Regla:** "Evitar que nodos compartan hardware" → Spread Placement Group.

### 3.3 EBS — Tipos de Volumen

| Tipo | Característica | IOPS máx |
|------|----------------|----------|
| **gp2** | IOPS ligado al tamaño (3 IOPS/GB) | 16.000 |
| **gp3** | IOPS independiente del tamaño | 16.000 |
| **io1/io2** | IOPS aprovisionadas, ratio hasta 500:1 | 64.000 |
| **st1** | HDD optimizado para throughput | Bajo |
| **sc1** | HDD frío, menor costo | Bajo |

- **Regla:** Desde 2021, gp3 reemplaza a gp2 como opción default de costo-rendimiento.
- io1/io2 solo si se requieren IOPS > 16.000 o ratio IOPS:GB superior a 500:1.
- gp3 permite IOPS independientes del tamaño: no hay que pagar por almacenamiento para obtener IOPS.

### 3.4 Almacenamiento de Archivos Compartidos

| Protocolo | Servicio AWS | Sistema Operativo |
|-----------|-------------|-------------------|
| **SMB** | FSx for Windows File Server | Windows |
| **NFS** | Amazon EFS | Linux |
| **Lustre** | FSx for Lustre | Linux (HPC) |

- FSx for Windows: integración nativa con Active Directory, Multi-AZ con failover automático.
- EFS: almacenamiento elástico NFS, multi-AZ, solo Linux.
- FSx for Lustre: sistema de archivos de alta velocidad para HPC, gaming, ML.

### 3.5 AWS Storage Gateway

| Tipo | Protocolo | Almacenamiento backend |
|------|-----------|----------------------|
| **S3 File Gateway** | NFS/SMB | Amazon S3 |
| **Volume Gateway** | iSCSI (bloque) | S3 + EBS (snapshots) |
| **Tape Gateway** | iSCSI VTL | S3 Glacier |

- **Regla:** Aplicación on-premises usa NFS + necesita almacenar en S3 + mínimos cambios → S3 File Gateway.
- Volume Gateway: presenta volúmenes de bloque iSCSI, no sistemas de archivos compartidos.

### 3.6 Amazon Kinesis — Enhanced Fan-Out

- Kinesis Data Streams: cada shard tiene 2 MB/s de lectura compartida entre todos los consumidores.
- **Enhanced Fan-Out:** asigna 2 MB/s por shard de forma **dedicada** a cada consumidor registrado.
- Usa HTTP/2 push (no polling) → latencia ~70 ms.
- **Regla:** Múltiples consumidores de Kinesis que no deben compartir throughput → Enhanced Fan-Out.

### 3.7 Mensajería

**SQS — Standard vs FIFO**

| Característica | Standard | FIFO |
|----------------|----------|------|
| Orden | Best-effort | Garantizado |
| Throughput | Ilimitado | 300 msg/s (3.000 con batching) |
| Duplicados | Posibles | Exactly-once |
| Caso de uso | Máximo throughput | Orden estricto |

- **Regla:** "Specific order that must be maintained" → siempre SQS FIFO.
- SNS no garantiza orden de entrega.

**Visibility Timeout**
- Tiempo en que un mensaje está invisible para otros consumidores mientras es procesado.
- Si el procesamiento tarda más que el timeout, el mensaje vuelve a ser visible → procesamiento duplicado.
- Solución: `ChangeMessageVisibility` para extender el timeout.
- **Regla:** Mensajes duplicados en SQS (sin duplicados en la cola) = visibility timeout muy corto.

**Amazon MQ**
- RabbitMQ/ActiveMQ administrado con menor overhead operativo que EC2 propio.

### 3.8 Bases de Datos Especializadas

| Caso de uso | Servicio |
|-------------|---------|
| Series de tiempo / IoT | Amazon Timestream |
| Grafos (redes sociales, recomendaciones) | Amazon Neptune |
| Ledger inmutable | Amazon QLDB |
| Documentos (MongoDB compatible) | Amazon DocumentDB |
| Clave-valor / NoSQL | Amazon DynamoDB |

**DynamoDB TTL**
- Mecanismo nativo para expiración automática de ítems basada en tiempo.
- Sin costo adicional por TTL; los ítems se eliminan en background.
- **Caso de uso:** "Revocar/expirar algo después de N días" → TTL de DynamoDB.

**RDS Proxy**
- Mantiene un pool de conexiones persistentes hacia RDS.
- Transparente para la aplicación: solo se cambia el endpoint.
- **Caso de uso:** Muchas conexiones simultáneas, Lambda + RDS, "too many connections".

### 3.9 Observabilidad

**AWS X-Ray**
- Distributed tracing: rastrea cada request a través de todos los microservicios.
- Genera service map visual e identifica qué microservicio causa degradación.

**CloudWatch Container Insights**
- Recopila métricas a nivel de cluster, nodo, pod y contenedor en EKS y ECS.
- Métricas: CPU, memoria, red, disco.

**Combinación para microservicios en EKS/ECS:**
- Container Insights → métricas de infraestructura.
- X-Ray → trazabilidad entre microservicios.

---

## Dominio 4 — Diseño de Arquitecturas Rentables

### 4.1 Clases de Almacenamiento S3

| Clase | Acceso | AZs | Costo recuperación | Caso de uso |
|-------|--------|-----|-------------------|-------------|
| **Standard** | Frecuente | ≥3 | No | Datos activos |
| **Intelligent-Tiering** | Variable/desconocido | ≥3 | No | Patrones impredecibles |
| **Standard-IA** | Infrecuente | ≥3 | Sí | Backup multi-AZ, acceso esporádico |
| **One Zone-IA** | Infrecuente | 1 | Sí | Datos reproducibles, menor costo |
| **Glacier Instant Retrieval** | Raro, acceso inmediato | ≥3 | Sí | Archivos con acceso ocasional inmediato |
| **Glacier Flexible Retrieval** | Archivos | ≥3 | Sí | 1-12 horas de recuperación |
| **Glacier Deep Archive** | Archivos profundos | ≥3 | Sí | 12-48 horas de recuperación |

**Reglas clave:**
- Acceso impredecible + sin costo de recuperación + mínimo overhead → S3 Intelligent-Tiering.
- Copia de seguridad multi-AZ con acceso infrecuente → Standard-IA (no One Zone-IA).
- One Zone-IA solo para datos fácilmente reproducibles (menor resiliencia).
- Glacier Instant ≠ Glacier Flexible: Instant = milisegundos, Flexible = minutos a horas.

**Reglas de ciclo de vida (S3 Lifecycle)**
- Transición automática entre clases según tiempo de vida del objeto.
- Ejemplo: Standard (0-180 días) → Standard-IA (180-360 días) → Glacier Instant Retrieval (360+ días).
- Si el requisito incluye "acceso inmediato siempre" → Glacier Instant (no Flexible).

### 4.2 Savings Plans

| Plan | Cubre | Compromiso |
|------|-------|------------|
| **EC2 Instance Savings Plan** | EC2 de una familia/región específica | 1 o 3 años |
| **Compute Savings Plan** | EC2 (cualquier familia/región) + Fargate + Lambda | 1 o 3 años |

- **Regla:** Escenario con EC2 + Fargate → solo Compute Savings Plan aplica (EC2 Instance SP **nunca** cubre Fargate).
- No Upfront = mayor flexibilidad financiera (cero desembolso inicial).
- All Upfront = mayor descuento pero menos flexibilidad.

### 4.3 VPC Endpoints — Reducción de Costos

**Gateway Endpoint**
- Completamente gratuito (sin costo por hora ni por GB).
- Solo para S3 y DynamoDB.
- Se configura como entrada en la route table de la subnet privada.
- **Regla:** EC2 privada accede a S3 o DynamoDB pasando por NAT Gateway → reemplazar con VPC Gateway Endpoint.

**Interface Endpoint (PrivateLink)**
- Costo por hora + por GB procesado.
- Para la mayoría de servicios AWS (SSM, Secrets Manager, ECR, CloudWatch, etc.).
- Permite conectar servicios on-premises vía Direct Connect o VPN a servicios AWS privados.

### 4.4 Conectividad de Red

| Escenario | Servicio | Costo/Complejidad |
|-----------|---------|-------------------|
| Usuarios remotos → VPC | AWS Client VPN | Bajo |
| Red on-premises → VPC (temporal/barato) | Site-to-Site VPN | Bajo |
| Red on-premises → VPC (ancho de banda/baja latencia) | AWS Direct Connect | Alto |
| EC2 privada → internet (salida) | NAT Gateway (en subnet pública) | Medio |
| EC2 privada → S3/DynamoDB | VPC Gateway Endpoint | Gratuito |

- **Client VPN:** usuario individual (laptop) → VPC.
- **Site-to-Site VPN:** red → red (dos redes completas).
- **Direct Connect:** solo cuando hay requisitos de ancho de banda consistente, baja latencia o cumplimiento normativo.
- **NAT Gateway:** siempre en subnet **pública**; subnet privada apunta al NAT Gateway.

### 4.5 Cost Allocation Tags

- Se activan en la **management account** de AWS Organizations.
- Después de activarse, aparecen como dimensiones en AWS Cost Explorer.
- Agrupar por tag en Cost Explorer (sin filtros por linked account) → visibilidad consolidada por dimensión.
- El CI/CD que ya aplica tags significa que no hay trabajo adicional de etiquetado.

---

## Migración de Bases de Datos

### AWS SCT + AWS DMS

| Herramienta | Función |
|------------|---------|
| **AWS SCT** (Schema Conversion Tool) | Convierte el esquema de un motor a otro (heterogéneo) |
| **AWS DMS** (Database Migration Service) | Migra datos; con CDC replica cambios en curso |

**Patrón canónico para migración heterogénea:**
1. AWS SCT convierte el esquema (ej. Oracle → Aurora PostgreSQL).
2. AWS DMS con **full load + CDC** carga los datos iniciales y replica cambios en tiempo real.

- CDC (Change Data Capture) usa redo logs/transaction logs del origen.
- **Regla:** Migración heterogénea de BD con mínimo downtime → AWS SCT + AWS DMS con CDC.
- Full load únicamente (sin CDC) genera pérdida de datos durante el cutover.
- DataSync: transferencia de **archivos** (NFS, SMB, S3), no de bases de datos relacionales.
- Snowball: transferencias masivas offline; no captura CDC.

---

## Arquitecturas Híbridas

### AWS Outposts
- Infraestructura AWS física instalada en el data center del cliente.
- Alto costo y overhead operativo significativo.
- Casos justificados: latencia ultra-baja local, requisitos de residencia de datos estrictos.
- No es la solución para 1 GB/noche de datos NFS (use Storage Gateway en su lugar).

### AWS PrivateLink (VPC Interface Endpoint)
- Conecta on-premises (vía Direct Connect o VPN) a servicios administrados de AWS de forma privada.
- Los datos permanecen en la red de AWS, nunca en internet público.
- **Caso de uso:** "Datos deben permanecer on-premises" + "usar servicios administrados AWS" → PrivateLink como puente.

---

## Patrones Frecuentes en el Examen

### Almacenamiento
| Escenario | Solución |
|-----------|---------|
| Archivos compartidos entre EC2 Windows | Amazon FSx for Windows File Server |
| Archivos compartidos entre EC2 Linux | Amazon EFS |
| HPC / gaming / ML | Amazon FSx for Lustre |
| Datos NFS on-premises → AWS mínimos cambios | S3 File Gateway |
| Block storage on-premises → AWS | Volume Gateway (iSCSI) |
| Backup multi-servicio mínimo overhead | AWS Backup |
| Datos con acceso impredecible sin costo recuperación | S3 Intelligent-Tiering |
| Copia secundaria acceso infrecuente multi-AZ | S3 Standard-IA |
| Archivos con acceso inmediato ocasional | S3 Glacier Instant Retrieval |

### Base de Datos
| Escenario | Solución |
|-----------|---------|
| Migración heterogénea con CDC | AWS SCT + AWS DMS |
| Poblar staging sin impacto en producción | Aurora Fast Clone |
| Connection pooling / muchas conexiones | Amazon RDS Proxy |
| Expirar datos automáticamente después de N días | DynamoDB TTL |
| Session state de aplicaciones web | ElastiCache Redis |
| Cargas de trabajo intermitentes PostgreSQL | Aurora Serverless v2 |
| Separar carga de reporting de BD primaria | RDS Read Replica |

### Red y Conectividad
| Escenario | Solución |
|-----------|---------|
| EC2 privada → S3/DynamoDB sin internet | VPC Gateway Endpoint (gratuito) |
| EC2 privada → internet saliente | NAT Gateway (en subnet pública) |
| On-premises → AWS temporal/barato | Site-to-Site VPN |
| Usuario remoto → VPC | AWS Client VPN |
| Multi-cuenta, restricciones centralizadas | SCP en AWS Organizations |
| DR multiregional Active-Passive | Route 53 Failover Routing + Health Checks |

### Compute y Escalado
| Escenario | Solución |
|-----------|---------|
| Carga predecible y recurrente | ASG Scheduled Action |
| Carga impredecible | ASG Dynamic Scaling |
| EC2 + Fargate en Savings Plan | Compute Savings Plan |
| Separar hardware entre instancias | Spread Placement Group |
| Máximo throughput de red entre nodos | Cluster Placement Group |
| IOPS independiente del tamaño del volumen | EBS gp3 |

### Seguridad e IAM
| Escenario | Solución |
|-----------|---------|
| EC2 accede a servicios AWS | IAM Role + Instance Profile |
| ECS task accede a servicios AWS | IAM Task Role |
| Credenciales BD con rotación automática | AWS Secrets Manager |
| Forzar HTTPS en S3 | Bucket policy `aws:SecureTransport` |
| Cifrado en reposo S3 con control de claves | SSE-KMS (CMK) |
| Auditar permisos IAM multi-cuenta | IAM Access Analyzer |
| Detectar cambios en configuración de recursos | AWS Config |
| Detectar cambios en jerarquía de OUs | AWS Control Tower (drift notifications) |
| Notificar eventos de API | CloudTrail → EventBridge → SNS |

### Mensajería y Streaming
| Escenario | Solución |
|-----------|---------|
| Orden garantizado de mensajes | SQS FIFO |
| Múltiples consumidores Kinesis sin compartir throughput | Enhanced Fan-Out |
| Cola SQS acumulada → escalar consumidores | ASG basado en `ApproximateNumberOfMessagesVisible` |
| Procesamiento duplicado en SQS | Aumentar visibility timeout (`ChangeMessageVisibility`) |
| RabbitMQ/ActiveMQ administrado | Amazon MQ |

---

## Diferencias Clave que el Examen Evalúa

| Par confuso | Diferencia |
|-------------|------------|
| ALB vs NLB | ALB = HTTP/HTTPS capa 7; NLB = TCP/UDP capa 4 (VoIP, gaming) |
| SQS FIFO vs Standard | FIFO = orden garantizado; Standard = mayor throughput sin orden |
| Secrets Manager vs SSM Parameter Store | Secrets Manager tiene rotación nativa para RDS |
| Standard-IA vs One Zone-IA | One Zone = 1 AZ (datos reproducibles); Standard = multi-AZ |
| Glacier Instant vs Flexible | Instant = milisegundos; Flexible = minutos a horas |
| Cognito User Pool vs Identity Pool | User Pool = autenticación; Identity Pool = credenciales AWS |
| EC2 Instance SP vs Compute SP | Compute SP cubre EC2 + Fargate + Lambda |
| Client VPN vs Site-to-Site VPN | Client = usuario→VPC; Site-to-Site = red→red |
| gp2 vs gp3 | gp3 = IOPS independiente del tamaño, más barato |
| Multi-AZ standby vs Read Replica | Multi-AZ standby = failover (no lectura); Read Replica = lectura |
| Macie vs Inspector vs GuardDuty | Macie = datos sensibles S3; Inspector = vulnerabilidades EC2/ECR; GuardDuty = amenazas activas |
| Config vs CloudTrail | Config = estado de recursos; CloudTrail = llamadas a API |
| Gateway Endpoint vs Interface Endpoint | Gateway = S3/DynamoDB gratuito; Interface = otros servicios con costo |
| Spread vs Cluster Placement Group | Spread = aislamiento hardware; Cluster = máximo rendimiento de red |

---

## Los "Siempre" del SAA-C03

- **Siempre** que EC2 necesite acceder a servicios AWS → IAM Role + Instance Profile, nunca access keys.
- **Siempre** que ECS task necesite acceder a servicios AWS → Task IAM Role, nunca instance profile.
- **Siempre** que EC2 privado acceda a S3/DynamoDB → VPC Gateway Endpoint, no NAT Gateway.
- **Siempre** que haya migración heterogénea de BD con CDC → AWS SCT + AWS DMS.
- **Siempre** que el escenario diga "mínimo overhead" + backup multi-servicio → AWS Backup.
- **Siempre** que haya rotación automática de credenciales BD → AWS Secrets Manager (no SSM Parameter Store).
- **Siempre** que se mencione RabbitMQ/ActiveMQ → Amazon MQ.
- **Siempre** que aplicación en EKS deba enrutar por path → AWS Load Balancer Controller + ALB.
- **Siempre** que el escenario mencione EC2 + Fargate en Savings Plan → Compute Savings Plan.
- **Siempre** que haya "orden garantizado" de mensajes → SQS FIFO.
- **Siempre** que haya "mensajes duplicados" en SQS (sin duplicados en cola) → aumentar visibility timeout.
- **Siempre** que NAT Gateway esté presente y el destino sea S3/DynamoDB → reemplazar con VPC Gateway Endpoint.
- **Siempre** que el escenario pida DR Active-Passive multiregional → Route 53 Failover Routing + Health Checks.

---

*Última actualización: Mayo 2026 | Basado en 40 preguntas del banco SAA-C03*
