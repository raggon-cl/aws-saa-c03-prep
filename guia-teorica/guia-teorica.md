# Guía Teórica — AWS Certified Solutions Architect Associate (SAA-C03)

> Marco teórico completo organizado por dominio del examen.
> Elaborado por **Rodrigo Aguilar** — Cloud Architect & AWS Authorized Instructor LATAM.

---

## Dominios del Examen

| Dominio | Descripción | Peso | Frecuencia |
|---------|-------------|------|-----------|
| D1 | Diseño de Arquitecturas Seguras | ~30% | ★★★★★ |
| D2 | Diseño de Arquitecturas Resilientes | ~26% | ★★★★☆ |
| D3 | Diseño de Arquitecturas de Alto Rendimiento | ~24% | ★★★★☆ |
| D4 | Diseño de Arquitecturas Rentables | ~20% | ★★★☆☆ |

> **Cómo usar esta guía:** cada sección incluye etiquetas `[D1]`–`[D4]` para indicar el dominio y `★` para la frecuencia de aparición en el examen (★ = baja, ★★★ = media, ★★★★★ = alta).

---

## Dominio 1 — Diseño de Arquitecturas Seguras `[D1]`

### 1.1 IAM — Identidad y Acceso `[D1]` ★★★★★

**IAM Role + Instance Profile (EC2)**
- Mecanismo nativo de AWS para otorgar permisos a instancias EC2 sin credenciales estáticas.
- Las credenciales son temporales y se rotan automáticamente vía el metadata service (`169.254.169.254`).
- **Regla absoluta:** EC2 necesita acceder a cualquier servicio AWS → IAM Role + Instance Profile, **nunca** access keys embebidas.

**Task IAM Role (ECS/EKS)**
- Para tareas ECS o pods EKS, el permiso se asigna en la task definition como Task IAM Role.
- El task role aplica solo a ese task específico (mínimo privilegio por contenedor).
- **Diferencia clave:** Instance profile otorga permisos a **todos** los contenedores del host; task role aplica solo al contenedor específico.

**SCPs — Service Control Policies**
- Aplicadas a OUs (Organizational Units) en AWS Organizations.
- Funciona como guardrail: ningún usuario ni rol en las cuentas puede exceder los permisos que la SCP permite.
- Aplicada en el root OU → afecta **todas** las cuentas automáticamente.
- Las nuevas cuentas heredan la SCP sin configuración adicional.
- **Caso de uso:** Restringir servicios o acciones en múltiples cuentas desde un único punto de control centralizado.

**IAM Access Analyzer** `[D1]` ★★★
- Analiza políticas IAM y de recursos para identificar accesos excesivos o no intencionados.
- Se configura a nivel de AWS Organizations para analizar todas las cuentas desde un punto centralizado.
- **Diferencia:** Network Access Analyzer analiza conectividad de red VPC, no permisos IAM.

### 1.2 Cifrado y Gestión de Secretos `[D1]` ★★★★★

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
- Lambda recupera el secreto en runtime mediante la API de Secrets Manager.
- **Diferencia crítica vs SSM Parameter Store:** Secrets Manager tiene rotación automática integrada para RDS; SSM Parameter Store no.
- **Regla:** RDS + rotación automática de credenciales → siempre Secrets Manager.

**KMS (Key Management Service)**
- Gestiona claves de cifrado, **no** credenciales de aplicación (username/password).
- No tiene concepto de rotación de credenciales de BD.

**Cifrado RDS completo**
- En reposo: KMS managed keys (se habilita al crear la instancia, no modificable después).
- En tránsito: SSL/TLS nativo de RDS.
- ACM gestiona certificados para conexiones EC2 → RDS vía SSL/TLS.
- VPN es para conectividad entre redes, no para cifrar tráfico dentro de una VPC.

### 1.3 Seguridad en S3 `[D1]` ★★★

**Amazon Macie**
- Descubre y clasifica datos sensibles en S3 (PII, PHI, credenciales).
- **No cifra datos.** Es una herramienta de descubrimiento, no de cifrado.

**Amazon GuardDuty**
- Detección de amenazas mediante análisis de logs (CloudTrail, VPC Flow Logs, DNS).
- No es la fuente autoritativa de eventos específicos de creación/eliminación de usuarios IAM.

**Amazon Inspector**
- Evalúa vulnerabilidades en EC2, imágenes ECR y funciones Lambda.
- **No analiza** políticas IAM ni monitorea eventos de gestión de usuarios.

### 1.4 Autenticación de Usuarios — Amazon Cognito `[D1]` ★★★★

| Componente | Función |
|------------|---------|
| **User Pool** | Autenticación: directorio de usuarios, login/signup, MFA |
| **Identity Pool** | Autorización: intercambia token del User Pool por credenciales AWS temporales |

- **Regla:** User Pool = quién eres. Identity Pool = qué puedes hacer en AWS.
- Patrón para acceso a S3 desde app: User Pool (login) → Identity Pool (credenciales IAM temporales) → S3.

### 1.5 Monitoreo y Auditoría `[D1]` ★★★★

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
- Capa de gobierno sobre AWS Organizations con guardrails pre-configurados.
- Account drift notifications detectan cambios en la jerarquía de OUs automáticamente.
- Las notificaciones se envían vía SNS.
- **Caso de uso:** Detectar cambios en jerarquía de OUs en entornos multi-cuenta.

### 1.6 Elastic Load Balancing `[D1/D3]` ★★★★★

| Tipo | Capa | Protocolo | Casos de uso |
|------|------|-----------|-------------|
| **ALB** | 7 (Aplicación) | HTTP/HTTPS/gRPC | Microservicios, enrutamiento por path/host |
| **NLB** | 4 (Transporte) | TCP/UDP/TLS | VoIP, gaming, baja latencia extrema, IP estática |
| **GWLB** | 3 (Red) | IP | Appliances de red (firewalls, IDS/IPS) |

**ALB — Conceptos clave**
- Enrutamiento por path (`/api/*`), por host (`api.ejemplo.com`), por cabecera HTTP.
- Target groups: instancias EC2, IPs, funciones Lambda, contenedores ECS.
- Sticky sessions: mantiene afinidad cliente-instancia (problemático para HA → externalizar sesión a Redis).
- Integración nativa con AWS WAF para protección de capa 7.

**NLB — Conceptos clave**
- Preserva la IP de origen del cliente (ALB no lo hace sin cabecera `X-Forwarded-For`).
- IP estática por AZ (útil para whitelisting en firewalls de terceros).
- Soporte para TLS offloading.

**Security Group del ALB — Patrón:**
```
Internet → ALB (inbound 443 desde 0.0.0.0/0)
ALB → EC2  (outbound 443 + puerto de health check hacia SG de instancias)
```
- Los health checks los inicia el ALB hacia las instancias (tráfico **saliente** del ALB).

### 1.7 AWS WAF y Shield `[D1]` ★★★

**AWS WAF**
- Firewall de aplicaciones web (capa 7).
- Protege contra SQLi, XSS, bots, geo-blocking.
- Se adjunta a ALB, CloudFront, API Gateway o AppSync.

**AWS Shield**
- Shield Standard: protección DDoS básica gratuita para todos los clientes.
- Shield Advanced: protección DDoS avanzada con soporte 24/7 del equipo DDoS Response Team.

---

## Dominio 2 — Diseño de Arquitecturas Resilientes `[D2]`

### 2.1 Alta Disponibilidad — Patrón Base `[D2]` ★★★★★

El patrón estándar de HA en AWS requiere:
- Mínimo 2 Availability Zones.
- NAT Gateway por AZ (no compartido entre AZs).
- ALB en subnets **públicas**; compute (EC2) y datos (RDS) en subnets **privadas**.
- RDS Multi-AZ para alta disponibilidad de base de datos.
- Auto Scaling Group para escalado y recuperación automática de EC2.

```
Internet
   ↓
 IGW
   ↓
ALB  (subnet pública AZ-a)   ALB  (subnet pública AZ-b)
   ↓                              ↓
EC2 ASG (subnet privada AZ-a)  EC2 ASG (subnet privada AZ-b)
   ↓                              ↓
       RDS Multi-AZ Primary ↔ RDS Standby
       (subnet privada AZ-a)   (subnet privada AZ-b)

EC2 → NAT GW (AZ-a) → IGW → Internet
EC2 → NAT GW (AZ-b) → IGW → Internet
```

### 2.2 RDS — Modos de Despliegue `[D2]` ★★★★★

| Modo | Propósito | ¿Lectura? | ¿Failover automático? |
|------|-----------|-----------|----------------------|
| **Single-AZ** | Dev/Test | No | No |
| **Multi-AZ** | Producción HA | **No** (standby no accesible) | Sí (~60-120 seg) |
| **Read Replica** | Reporting/Lectura | Sí | No automático |
| **Multi-AZ + Read Replica** | Producción HA + escala lectura | Sí | Sí |

- **Regla crítica:** El standby de Multi-AZ **no** es accesible para lectura. Solo sirve para failover.
- Para descargar consultas de reporting → Read Replica.
- Para poblar entornos staging/test sin impacto → Aurora Fast Clone.

### 2.3 Amazon Aurora `[D2]` ★★★★

**Aurora Fast Clone**
- Crea una copia de la base de datos en segundos usando copy-on-write.
- No duplica datos físicamente → no impacta producción ni consume almacenamiento adicional hasta que se modifica.
- **Caso de uso:** Poblar entornos de desarrollo/staging de forma rápida y sin impacto.

**Aurora vs RDS MySQL/PostgreSQL**

| Característica | RDS MySQL/PostgreSQL | Aurora |
|----------------|---------------------|--------|
| Read Replicas máx. | 5 | 15 |
| Replicación | Asíncrona | Asíncrona (ms) |
| Storage | Fijo/autoextensible | Automático hasta 128 TB |
| Fast Clone | No | Sí |
| Serverless | No | Sí (v2) |

**Aurora Serverless v2**
- Escala automáticamente en fracciones de ACU según demanda.
- Ideal para cargas intermitentes: dev/test, reportes, APIs con tráfico variable.

### 2.4 Route 53 — Políticas de Enrutamiento `[D2]` ★★★★

| Política | Uso |
|----------|-----|
| **Failover** | DR Active-Passive multiregional |
| **Weighted** | Canary releases, distribución proporcional |
| **Latency** | Enrutar al endpoint con menor latencia para el usuario |
| **Geolocation** | Enrutar según país/continente del usuario |
| **Multivalue Answer** | Hasta 8 registros saludables, similar a DNS round-robin |
| **IP-based** | Enrutar según bloque CIDR de origen |

- **DR Active-Passive multiregional:** Route 53 Failover Routing Policy + Health Checks.
- **DR Active-Active multiregional:** Weighted o Latency Routing.
- Route 53 Resolver: resolución DNS híbrida on-premises ↔ VPC (no es failover).
- Route 53 Profiles: comparte configuraciones DNS entre VPCs (no es failover entre regiones).

### 2.5 Amazon MQ `[D2]` ★★★

- Servicio administrado de broker de mensajería compatible con **RabbitMQ** y **ActiveMQ**.
- Par active/standby en dos AZs con failover automático.
- **Regla:** Cuando el escenario mencione RabbitMQ o ActiveMQ → Amazon MQ (menor overhead que EC2).

### 2.6 AWS Backup `[D2]` ★★★

- Servicio centralizado de backup para múltiples servicios AWS: RDS, EBS, EFS, FSx, DynamoDB, S3, EC2.
- Políticas de retención unificadas desde un único punto.
- Backup Vault Lock: protección WORM (Write Once Read Many) para cumplimiento normativo.
- **Casos de uso:**
  - Retención de backups RDS > 35 días (límite de snapshots nativos).
  - Backup multi-servicio con mínimo overhead operativo.
  - Cumplimiento normativo con registros inmutables.

### 2.7 AWS Organizations y Control Tower `[D2]` ★★★

**AWS Organizations**
- Gestiona múltiples cuentas AWS en una jerarquía de OUs.
- SCPs aplicadas en el root OU afectan todas las cuentas automáticamente.
- **RAM (Resource Access Manager):** comparte recursos entre cuentas (subnets, Transit Gateway, Route 53 Resolver rules).

**AWS Control Tower**
- Capa de gobierno sobre Organizations con Landing Zone pre-configurada.
- Provisiona cuentas con guardrails (preventivos = SCPs, detectivos = Config rules).
- Detecta drift: cuentas/OUs que se desvían de la configuración esperada.

---

## Dominio 3 — Diseño de Arquitecturas de Alto Rendimiento `[D3]`

### 3.1 Compute — Auto Scaling `[D3]` ★★★★★

| Tipo de Scaling | Cuándo usar |
|-----------------|-------------|
| **Scheduled Action** | Carga predecible y recurrente (ej. viernes por la noche) |
| **Target Tracking** | Mantener una métrica en un valor objetivo (ej. CPU al 60%) |
| **Step Scaling** | Escalar en pasos según rangos de métricas |
| **Dynamic / Simple** | Respuesta a alarma de CloudWatch |

- Scheduled Action: configuración con expresión cron, cero intervención manual posterior.
- Métrica para escalar consumidores SQS → `ApproximateNumberOfMessagesVisible`.

### 3.2 AWS Lambda `[D3]` ★★★★

**Características clave**
- Máximo 15 minutos de ejecución por invocación.
- Memoria: 128 MB a 10.240 MB (la CPU escala proporcionalmente con la memoria).
- Cold start: latencia inicial al crear un nuevo execution environment (~100ms–1s según runtime).
- Lambda SnapStart (Java): pre-inicializa el execution environment para reducir cold start.

**Lambda en VPC**
- Lambda puede conectarse a recursos privados (RDS, ElastiCache) dentro de una VPC.
- Requiere configuración de subnet y security group.
- Usa ENI (Elastic Network Interface) para acceder a la VPC.
- Sin IP pública propia → necesita NAT Gateway para acceso a internet desde VPC.

**Lambda + RDS → RDS Proxy**
- Lambda escala a miles de instancias simultáneas.
- Cada instancia abre conexión a RDS → "too many connections".
- Solución: RDS Proxy actúa como pool de conexiones entre Lambda y RDS.

**Límites importantes**
- Concurrencia por defecto: 1.000 ejecuciones simultáneas por región (ajustable).
- Tamaño de payload: 6 MB síncrono, 256 KB asíncrono.
- Tamaño del paquete de despliegue: 50 MB (zip), 250 MB (descomprimido).

### 3.3 Amazon CloudFront `[D3]` ★★★★

- CDN (Content Delivery Network) global con más de 400 puntos de presencia (PoPs).
- Reduce latencia sirviendo contenido desde el edge más cercano al usuario.
- Integra con S3, ALB, EC2, API Gateway y orígenes HTTP externos.

**Características clave**
- **Origin Access Control (OAC):** permite que solo CloudFront acceda al bucket S3 (S3 privado).
- **Signed URLs / Signed Cookies:** acceso temporal a contenido protegido.
- **Cache behaviors:** reglas por path para diferentes orígenes o TTLs.
- **Lambda@Edge / CloudFront Functions:** lógica en el edge (modificar request/response, A/B testing).
- Integración con **AWS WAF** para protección de capa 7 en el edge.
- **Invalidaciones:** elimina objetos del cache antes de que expire el TTL.

**CloudFront vs S3 Transfer Acceleration**
- CloudFront: distribución global de contenido estático y dinámico con caché.
- S3 Transfer Acceleration: acelera uploads a S3 usando la red backbone de AWS (sin caché).

### 3.4 EC2 Placement Groups `[D3]` ★★★

| Tipo | Característica | Límite | Uso |
|------|----------------|--------|-----|
| **Cluster** | Mismo rack → baja latencia de red | Sin límite | HPC, procesamiento paralelo |
| **Spread** | Racks distintos → máximo aislamiento | 7 instancias/AZ | Aplicaciones críticas |
| **Partition** | Grupos en particiones separadas | 7 particiones/AZ | Hadoop, Kafka, Cassandra |

- **Regla:** "Evitar que nodos compartan hardware" → Spread. "Máximo rendimiento de red entre nodos" → Cluster.

### 3.5 EBS — Tipos de Volumen `[D3]` ★★★★

| Tipo | Categoría | IOPS máx | Throughput | Caso de uso |
|------|-----------|----------|------------|-------------|
| **gp3** | SSD general | 16.000 | 1.000 MB/s | Default para la mayoría de workloads |
| **gp2** | SSD general | 16.000 | 250 MB/s | Legacy (reemplazado por gp3) |
| **io2 Block Express** | SSD provisionado | 256.000 | 4.000 MB/s | Bases de datos de alta demanda |
| **io1** | SSD provisionado | 64.000 | 1.000 MB/s | Workloads I/O intensivos |
| **st1** | HDD throughput | 500 | 500 MB/s | Big data, logs, procesamiento secuencial |
| **sc1** | HDD frío | 250 | 250 MB/s | Archivos de acceso infrecuente |

- **gp3:** IOPS independiente del tamaño del volumen (~20% más barato que gp2).
- **gp2:** 3 IOPS/GB → para 15.000 IOPS se necesitan ~5 TB (pagar almacenamiento para obtener IOPS).
- **io1/io2:** solo si IOPS > 16.000 o ratio IOPS:GB > 500:1.
- Volúmenes magnéticos: cientos de IOPS como máximo, no alcanzan requisitos modernos.

### 3.6 Almacenamiento de Archivos Compartidos `[D3]` ★★★★

| Protocolo | Servicio AWS | SO | Características |
|-----------|-------------|-----|----------------|
| **SMB** | FSx for Windows File Server | Windows | AD integration, Multi-AZ, DFS |
| **NFS** | Amazon EFS | Linux | Elástico, multi-AZ, Performance/General mode |
| **Lustre** | FSx for Lustre | Linux | HPC, ML, 100+ GB/s, integración S3 |
| **OpenZFS** | FSx for OpenZFS | Linux | NFS compatible, snapshots, clones |
| **ONTAP** | FSx for NetApp ONTAP | Win/Linux | iSCSI + NFS + SMB, multi-protocolo |

- FSx for Windows: integración nativa con Active Directory, Multi-AZ con failover automático.
- EFS: almacenamiento elástico NFS, multi-AZ, facturación por GB usado.

### 3.7 AWS Storage Gateway `[D3]` ★★★

| Tipo | Protocolo | Almacenamiento backend | Caso de uso |
|------|-----------|----------------------|-------------|
| **S3 File Gateway** | NFS/SMB | Amazon S3 | Archivos on-premises → S3 |
| **Volume Gateway** | iSCSI (bloque) | S3 + EBS snapshots | Block storage on-premises → AWS |
| **Tape Gateway** | iSCSI VTL | S3 Glacier | Reemplazo de cintas físicas |

- **Regla:** Aplicación on-premises usa NFS + necesita almacenar en S3 + mínimos cambios → S3 File Gateway.
- Volume Gateway: presenta volúmenes de bloque iSCSI, **no** sistemas de archivos compartidos.

### 3.8 Amazon Kinesis — Enhanced Fan-Out `[D3]` ★★★

- Kinesis Data Streams: cada shard tiene 2 MB/s de lectura compartida entre todos los consumidores.
- **Enhanced Fan-Out:** 2 MB/s por shard de forma **dedicada** a cada consumidor registrado.
- Usa HTTP/2 push (no polling) → latencia ~70 ms vs ~200 ms en polling estándar.
- **Regla:** Múltiples consumidores de Kinesis que no deben compartir throughput → Enhanced Fan-Out.

### 3.9 Mensajería `[D3]` ★★★★★

**SQS — Standard vs FIFO**

| Característica | Standard | FIFO |
|----------------|----------|------|
| Orden | Best-effort | Garantizado |
| Throughput | Ilimitado | 300 msg/s (3.000 con batching) |
| Duplicados | Posibles | Exactly-once processing |
| Caso de uso | Máximo throughput | Orden estricto requerido |

- **Regla:** "Specific order that must be maintained" → siempre SQS FIFO.
- SNS **no** garantiza orden de entrega.

**Visibility Timeout**
- Tiempo en que un mensaje está invisible para otros consumidores mientras es procesado.
- Si el procesamiento tarda más que el timeout → mensaje vuelve a ser visible → duplicado.
- Solución: `ChangeMessageVisibility` para extender el timeout dinámicamente.
- **Regla:** Mensajes duplicados en SQS (sin duplicados en la cola) = visibility timeout muy corto.

**Dead Letter Queue (DLQ)**
- Recibe mensajes que fallaron N veces (configurable via `maxReceiveCount`).
- Permite inspeccionar mensajes problemáticos sin perderlos.
- Se puede configurar en colas Standard y FIFO.

**Amazon SNS — Fan-out**
- Publica un mensaje a múltiples suscriptores simultáneamente.
- Suscriptores: SQS, Lambda, HTTP/HTTPS, Email, SMS.
- SNS FIFO: orden garantizado, solo suscriptores SQS FIFO.
- Patrón SNS → SQS: desacoplamiento + durabilidad + múltiples consumidores.

**Amazon EventBridge**
- Bus de eventos serverless para integración entre servicios AWS y externos.
- Event patterns para filtrar eventos específicos (ej. cambios de estado EC2, eventos CloudTrail).
- Rules con targets: Lambda, SQS, SNS, Step Functions, API Gateway.

### 3.10 Bases de Datos Especializadas `[D3]` ★★★★

| Caso de uso | Servicio | Característica diferenciadora |
|-------------|---------|-------------------------------|
| Clave-valor / NoSQL | Amazon DynamoDB | Latencia de un dígito de ms a escala |
| Series de tiempo / IoT | Amazon Timestream | Optimizado para datos temporales |
| Grafos | Amazon Neptune | Traversals de grafos eficientes |
| Ledger inmutable | Amazon QLDB | Historial de cambios verificable |
| Documentos (MongoDB) | Amazon DocumentDB | Compatible con MongoDB API |
| En memoria / caché | ElastiCache Redis/Memcached | Microsegundos de latencia |

**DynamoDB TTL** `[D3]` ★★★
- Mecanismo nativo para expiración automática de ítems basada en tiempo.
- Sin costo adicional por TTL; los ítems se eliminan en background (puede tardar hasta 48h).
- **Caso de uso:** "Revocar/expirar algo después de N días" → TTL de DynamoDB.

**RDS Proxy** `[D3]` ★★★★
- Mantiene un pool de conexiones persistentes hacia RDS.
- Transparente para la aplicación: solo se cambia el endpoint, sin modificar código.
- **Casos de uso:** Lambda + RDS, muchas conexiones simultáneas, "too many connections".

**ElastiCache — Redis vs Memcached**

| Característica | Redis | Memcached |
|----------------|-------|-----------|
| Replicación | Sí | No |
| Persistencia | Sí | No |
| Multi-AZ / HA | Sí | No |
| Estructuras de datos | Strings, Hashes, Listas, Sets... | Solo strings |
| Caso de uso | Session state HA, leaderboards, pub/sub | Caché simple sin HA |

- **Regla:** Session state con HA → Redis. Memcached si los datos son completamente desechables.

### 3.11 Observabilidad `[D3]` ★★★★

**AWS X-Ray**
- Distributed tracing: rastrea cada request a través de todos los microservicios.
- Genera service map visual e identifica qué microservicio causa degradación.
- SDK disponible para Node.js, Python, Java, Go, .NET.

**CloudWatch Container Insights**
- Métricas a nivel de cluster, nodo, pod y contenedor en EKS y ECS.
- Métricas: CPU, memoria, red, disco.

**CloudWatch Logs Insights**
- Consultas interactivas sobre logs en CloudWatch.

**Combinación para microservicios en EKS/ECS:**
- Container Insights → métricas de infraestructura.
- X-Ray → trazabilidad entre microservicios.

---

## Dominio 4 — Diseño de Arquitecturas Rentables `[D4]`

### 4.1 Clases de Almacenamiento S3 `[D4]` ★★★★★

| Clase | Acceso | AZs | Costo recuperación | Latencia | Caso de uso |
|-------|--------|-----|-------------------|----------|-------------|
| **Standard** | Frecuente | ≥3 | No | ms | Datos activos |
| **Intelligent-Tiering** | Variable | ≥3 | No | ms | Patrones impredecibles |
| **Standard-IA** | Infrecuente | ≥3 | Sí | ms | Backup multi-AZ esporádico |
| **One Zone-IA** | Infrecuente | 1 | Sí | ms | Datos reproducibles |
| **Glacier Instant Retrieval** | Raro | ≥3 | Sí | ms | Archivos con acceso ocasional inmediato |
| **Glacier Flexible Retrieval** | Archivos | ≥3 | Sí | min–12h | Archivos sin urgencia |
| **Glacier Deep Archive** | Archivos profundos | ≥3 | Sí | 12–48h | Archivos de largo plazo |

**Reglas clave:**
- Acceso impredecible + sin costo de recuperación + mínimo overhead → S3 Intelligent-Tiering.
- Copia de seguridad multi-AZ con acceso infrecuente → Standard-IA (no One Zone-IA).
- One Zone-IA solo para datos fácilmente reproducibles.
- Glacier Instant ≠ Glacier Flexible: Instant = milisegundos, Flexible = minutos a horas.

**Reglas de ciclo de vida (S3 Lifecycle)**
- Transición automática entre clases según tiempo de vida del objeto.
- Ejemplo: Standard (0–180 días) → Standard-IA (180–360 días) → Glacier Instant Retrieval (360+ días).
- Si el requisito incluye "acceso inmediato siempre" → Glacier Instant (no Flexible ni Deep Archive).

### 4.2 Savings Plans `[D4]` ★★★★

| Plan | Cubre | Alcance |
|------|-------|---------|
| **EC2 Instance SP** | EC2 de una familia/región específica | Más restrictivo, mayor descuento |
| **Compute SP** | EC2 (cualquier familia/región) + Fargate + Lambda | Más flexible |

- **Regla:** EC2 + Fargate → solo Compute Savings Plan (EC2 Instance SP **nunca** cubre Fargate).
- No Upfront = mayor flexibilidad financiera.
- All Upfront = mayor descuento pero menor flexibilidad.
- Compromisos disponibles: 1 año o 3 años (no "varios años" — el examen pone trampas aquí).

### 4.3 VPC Endpoints — Reducción de Costos `[D4]` ★★★★★

**Gateway Endpoint**
- Completamente gratuito (sin costo por hora ni por GB).
- Solo para **S3** y **DynamoDB**.
- Se configura como entrada en la route table de la subnet privada.
- **Regla:** EC2 privada accede a S3 o DynamoDB pasando por NAT Gateway → reemplazar con Gateway Endpoint.

**Interface Endpoint (PrivateLink)**
- Costo por hora (~$0.01/h) + por GB procesado.
- Para la mayoría de servicios AWS (SSM, Secrets Manager, ECR, CloudWatch, etc.).
- Permite conectar servicios on-premises vía Direct Connect o VPN a servicios AWS privados.

### 4.4 Conectividad de Red `[D4]` ★★★★

| Escenario | Servicio | Costo/Complejidad |
|-----------|---------|-------------------|
| Usuarios remotos → VPC | AWS Client VPN | Bajo |
| Red on-premises → VPC (temporal/barato) | Site-to-Site VPN | Bajo |
| Red on-premises → VPC (ancho de banda / baja latencia) | AWS Direct Connect | Alto |
| EC2 privada → internet (salida) | NAT Gateway (en subnet pública) | Medio |
| EC2 privada → S3/DynamoDB | VPC Gateway Endpoint | Gratuito |

- **Client VPN:** usuario individual (laptop) → VPC.
- **Site-to-Site VPN:** red → red (dos redes completas).
- **Direct Connect:** solo cuando hay requisitos de ancho de banda consistente, baja latencia o cumplimiento normativo que prohíbe internet público. Demora semanas en provisionarse.
- **NAT Gateway:** siempre en subnet **pública**; subnet privada apunta al NAT Gateway.

**Flujo NAT Gateway:**
```
EC2 privada → NAT GW (subnet pública) → IGW → Internet
```

### 4.5 Cost Allocation Tags `[D4]` ★★★

- Se activan en la **management account** de AWS Organizations.
- Después de activarse, aparecen como dimensiones en AWS Cost Explorer.
- Agrupar por tag en Cost Explorer sin filtros por linked account → visibilidad consolidada por dimensión.
- El CI/CD que ya aplica tags significa que no hay trabajo adicional de etiquetado.

---

## Migración a AWS `[D2/D3]`

### Los 7 Rs de Migración ★★★

| Estrategia | Descripción | Esfuerzo |
|------------|-------------|---------|
| **Retire** | Desconectar aplicaciones sin uso | Ninguno |
| **Retain** | Mantener on-premises (por ahora) | Ninguno |
| **Rehost** | Lift-and-shift (EC2) | Bajo |
| **Replatform** | Lift-tinker-and-shift (RDS, ElastiCache) | Medio |
| **Repurchase** | Mover a SaaS (Salesforce, ServiceNow) | Bajo-Medio |
| **Refactor** | Re-arquitecturar para cloud-native | Alto |
| **Relocate** | Mover VMware a VMware Cloud on AWS | Bajo |

### AWS SCT + AWS DMS ★★★★★

| Herramienta | Función |
|------------|---------|
| **AWS SCT** (Schema Conversion Tool) | Convierte el esquema de un motor a otro (heterogéneo) |
| **AWS DMS** (Database Migration Service) | Migra datos; con CDC replica cambios en curso |

**Patrón canónico para migración heterogénea:**
1. AWS SCT convierte el esquema (ej. Oracle → Aurora PostgreSQL).
2. AWS DMS con **full load + CDC** carga los datos iniciales y replica cambios en tiempo real.

- CDC (Change Data Capture) usa redo logs/transaction logs del origen.
- **Regla:** Migración heterogénea de BD con mínimo downtime → AWS SCT + AWS DMS con CDC.
- Full load únicamente (sin CDC) → pérdida de datos durante el cutover.

### AWS DataSync ★★★

- Transferencia de **archivos** entre on-premises y AWS (NFS, SMB → S3, EFS, FSx).
- **No es** para migración de bases de datos relacionales (eso es DMS).
- **No es** para montar un filesystem on-premises en AWS con mínimos cambios (eso es Storage Gateway).
- Caso de uso: migración masiva de datos de archivos, transferencia recurrente programada.

### AWS Snow Family ★★★

| Servicio | Capacidad | Caso de uso |
|---------|-----------|-------------|
| **Snowcone** | 8 TB HDD / 14 TB SSD | Edge computing, entornos austeros |
| **Snowball Edge Storage** | 80 TB | Migración offline de datos grandes |
| **Snowball Edge Compute** | 80 TB + 52 vCPUs | Edge computing con procesamiento |
| **Snowmobile** | 100 PB | Migración de exabytes |

- **Usar cuando:** ancho de banda limitado o el tiempo de transferencia por red > 1 semana.
- **No usar cuando:** ya existe VPN/Direct Connect (usar DataSync o DMS según el caso).
- Snowball **no captura CDC** — no es válido para migraciones con cambios continuos.

### AWS Migration Hub ★★

- Panel centralizado para rastrear el progreso de migraciones.
- Integra con AWS Application Migration Service (MGN) y DMS.
- No realiza migraciones por sí mismo; es una herramienta de visibilidad.

---

## Arquitecturas de Referencia

### Arquitectura Serverless Típica

```
Usuarios
   ↓
Amazon CloudFront (caché + WAF)
   ↓
Amazon API Gateway (throttling, autenticación, enrutamiento)
   ↓
AWS Lambda (lógica de negocio)
   ↓         ↓          ↓
DynamoDB   S3        SQS → Lambda (procesamiento async)
```

**Servicios clave:**
- CloudFront: reduce latencia y protege con WAF.
- API Gateway: gestiona throttling, API keys, autenticación JWT/Cognito.
- Lambda: sin servidores, escala automáticamente.
- DynamoDB: base de datos serverless de baja latencia.
- SQS: desacopla el procesamiento asíncrono.

### Arquitectura de Procesamiento de Datos

```
Fuentes de datos (IoT, apps, logs)
   ↓
Amazon Kinesis Data Streams
   ↓                    ↓
AWS Lambda          Amazon Kinesis
(procesamiento      Data Firehose
 en tiempo real)         ↓
                    Amazon S3 (data lake)
                         ↓
                    AWS Glue (ETL / Catalog)
                         ↓
                    Amazon Athena (SQL sobre S3)
                         ↓
                    Amazon QuickSight (BI / visualización)
```

### Arquitectura Multi-Cuenta con Organizations

```
Management Account
├── SCPs (guardrails globales)
├── AWS Control Tower (governance)
└── OUs
    ├── OU-Security
    │   └── Security Account (GuardDuty, Config Aggregator, CloudTrail Org)
    ├── OU-Shared
    │   └── Shared Services Account (Active Directory, Transit Gateway)
    ├── OU-Prod
    │   ├── Prod Account A
    │   └── Prod Account B
    └── OU-Dev
        └── Dev Accounts
```

**Servicios clave:**
- SCPs: restricciones en el root OU se heredan por todas las cuentas.
- AWS RAM: comparte recursos (Transit Gateway, subnets) entre cuentas sin peering complejo.
- CloudTrail Organization Trail: centraliza logs de todas las cuentas en S3 de la cuenta de seguridad.
- AWS Config Aggregator: consolida compliance de todas las cuentas.

### Arquitectura DR Multiregional Active-Passive

```
Región Primaria (us-east-1)          Región Secundaria (us-west-2)
ALB → EC2 ASG → RDS Primary    ←→   ALB → EC2 ASG → RDS Read Replica
                                          (promoverse a primary en failover)

Route 53 Failover Routing Policy
├── Primary: ALB us-east-1 (health check activo)
└── Secondary: ALB us-west-2 (en espera)

S3 Cross-Region Replication para datos de objetos
AWS Backup con vault en región secundaria
```

---

## Anti-Patrones — Trampas Frecuentes del Examen

> Estos son los errores más comunes que el examen pone a prueba deliberadamente. Memorizar estos anti-patrones permite eliminar opciones incorrectas rápidamente.

| Anti-patrón | Por qué es incorrecto | Solución correcta |
|-------------|----------------------|-------------------|
| Usar NAT Gateway cuando el destino es S3/DynamoDB | NAT Gateway tiene costo y tráfico sale de VPC | VPC Gateway Endpoint (gratuito) |
| Usar SSM Parameter Store para credenciales RDS con rotación | SSM no tiene rotación automática nativa para RDS | AWS Secrets Manager |
| Usar ElastiCache Memcached para session state con HA | Memcached no replica ni tiene Multi-AZ | ElastiCache Redis |
| Leer desde el standby de RDS Multi-AZ | El standby no es accesible para lectura | RDS Read Replica |
| Usar Snowball cuando ya existe VPN/Direct Connect | Snowball es para transferencias offline sin red | AWS DataSync o DMS |
| Usar mysqldump para migrar BD con cambios continuos | mysqldump es lento y no captura CDC | AWS DMS con full load + CDC |
| Usar EC2 Instance Savings Plan cuando hay Fargate | EC2 Instance SP no cubre Fargate | Compute Savings Plan |
| Usar ACM para cifrar el tráfico hacia un bucket S3 | ACM se asocia a ALB/CloudFront, no a S3 directamente | `aws:SecureTransport` en bucket policy |
| Usar Internet Gateway en subnet privada para salida | IGW requiere IP pública; viola seguridad | NAT Gateway en subnet pública |
| Usar CloudFormation drift detection para monitoreo continuo | Drift detection es bajo demanda, no continuo | AWS Config |
| Usar instance profile para dar permisos a un task ECS | Instance profile afecta a todos los contenedores del host | Task IAM Role en la task definition |
| Usar SQS Standard cuando se requiere orden de mensajes | Standard no garantiza orden | SQS FIFO |
| Usar Macie para cifrar datos PHI en S3 | Macie descubre datos, no los cifra | SSE-KMS con CMK |
| Usar NAT Gateway para alta disponibilidad con un solo NAT | Un solo NAT es SPOF: si la AZ falla, se pierde salida | NAT Gateway por AZ |
| Usar geolocation routing para DR | Geolocation enruta por origen geográfico, no por salud | Route 53 Failover Routing + Health Checks |
| Usar CloudTrail para monitorear rendimiento de microservicios | CloudTrail registra API calls del plano de control | AWS X-Ray + CloudWatch Container Insights |
| Usar Amazon Inspector para revisar permisos IAM excesivos | Inspector evalúa vulnerabilidades de SO/paquetes | IAM Access Analyzer |

---

## Límites y Umbrales Numéricos

> El examen SAA-C03 incluye preguntas donde los números son determinantes para elegir la opción correcta.

### Cómputo

| Servicio / Recurso | Límite |
|--------------------|--------|
| Lambda — duración máxima | 15 minutos |
| Lambda — memoria | 128 MB – 10.240 MB |
| Lambda — concurrencia default por región | 1.000 ejecuciones simultáneas |
| Lambda — payload síncrono | 6 MB |
| Lambda — payload asíncrono | 256 KB |
| Lambda — paquete de despliegue (zip) | 50 MB |
| EC2 Spread Placement Group | 7 instancias por AZ |
| EC2 Partition Placement Group | 7 particiones por AZ |

### Almacenamiento S3

| Recurso | Límite |
|---------|--------|
| Tamaño mínimo objeto (Standard-IA / One Zone-IA / Glacier) | 128 KB mínimo de facturación |
| S3 multipart upload — recomendado desde | 100 MB |
| S3 multipart upload — obligatorio desde | 5 GB |
| S3 tamaño máximo de objeto | 5 TB |
| Glacier Instant Retrieval | Milisegundos |
| Glacier Flexible Retrieval | 1–12 horas (Expedited: 1–5 min, Standard: 3–5h, Bulk: 5–12h) |
| Glacier Deep Archive | 12–48 horas |

### EBS

| Tipo | IOPS máx | Throughput máx |
|------|----------|---------------|
| gp3 | 16.000 | 1.000 MB/s |
| gp2 | 16.000 | 250 MB/s |
| io1 | 64.000 | 1.000 MB/s |
| io2 Block Express | 256.000 | 4.000 MB/s |

### Mensajería

| Recurso | Límite |
|---------|--------|
| SQS — tamaño máximo de mensaje | 256 KB |
| SQS — retention period máximo | 14 días |
| SQS FIFO — throughput estándar | 300 msg/s |
| SQS FIFO — throughput con batching | 3.000 msg/s |
| SQS — visibility timeout máximo | 12 horas |
| SNS — tamaño máximo de mensaje | 256 KB |
| Kinesis Data Streams — shard write | 1 MB/s o 1.000 registros/s |
| Kinesis Data Streams — shard read | 2 MB/s (shared entre consumidores) |
| Kinesis Enhanced Fan-Out — shard read | 2 MB/s **por consumidor** |

### Base de Datos

| Recurso | Límite |
|---------|--------|
| RDS Multi-AZ — tiempo de failover | 60–120 segundos |
| RDS — máximo Read Replicas | 5 (RDS MySQL/PostgreSQL) |
| Aurora — máximo Read Replicas | 15 |
| Aurora — storage máximo | 128 TB |
| DynamoDB — tamaño máximo de ítem | 400 KB |
| DynamoDB — partición: RCU máx | 3.000 RCU |
| DynamoDB — partición: WCU máx | 1.000 WCU |
| ElastiCache Redis — max nodos por cluster | 6 (1 primario + 5 réplicas) |
| RDS snapshots automáticos — retención máxima | 35 días |

### Red

| Recurso | Límite |
|---------|--------|
| VPC — máximo de subnets | 200 por VPC |
| VPC — máximo de security groups por instancia | 5 |
| Security Group — reglas de entrada máx | 60 por SG |
| Transit Gateway — VPCs adjuntas | 5.000 |
| Direct Connect — velocidades disponibles | 1, 10, 100 Gbps (hosted: desde 50 Mbps) |
| Site-to-Site VPN — throughput máx | ~1.25 Gbps por túnel |

---

## Diferencias Clave que el Examen Evalúa

| Par confuso | Diferencia |
|-------------|------------|
| ALB vs NLB | ALB = HTTP/HTTPS capa 7 (path routing); NLB = TCP/UDP capa 4 (VoIP, gaming, IP estática) |
| NLB vs GWLB | NLB = balanceo de tráfico; GWLB = inserción de appliances de red (firewalls, IDS) |
| SQS FIFO vs Standard | FIFO = orden garantizado, 300 msg/s; Standard = ilimitado sin orden |
| Secrets Manager vs SSM Parameter Store | Secrets Manager = rotación automática nativa RDS; SSM = sin rotación nativa |
| Standard-IA vs One Zone-IA | Standard = multi-AZ; One Zone = 1 AZ (solo datos reproducibles) |
| Glacier Instant vs Flexible vs Deep Archive | Instant = ms; Flexible = 1-12h; Deep Archive = 12-48h |
| Cognito User Pool vs Identity Pool | User Pool = autenticación; Identity Pool = credenciales AWS temporales |
| EC2 Instance SP vs Compute SP | Compute SP cubre EC2 + Fargate + Lambda; EC2 Instance SP solo EC2 familia/región fija |
| Client VPN vs Site-to-Site VPN | Client = usuario→VPC; Site-to-Site = red on-premises→VPC |
| gp2 vs gp3 | gp3 = IOPS independiente del tamaño, más barato; gp2 = 3 IOPS/GB |
| Multi-AZ standby vs Read Replica | Multi-AZ = failover (no lectura); Read Replica = lectura (no failover automático) |
| Macie vs Inspector vs GuardDuty | Macie = datos sensibles S3; Inspector = vulnerabilidades EC2/ECR/Lambda; GuardDuty = amenazas activas |
| Config vs CloudTrail | Config = estado/historial de configuración de recursos; CloudTrail = llamadas a API |
| Gateway Endpoint vs Interface Endpoint | Gateway = S3/DynamoDB gratuito; Interface = otros servicios con costo/hora |
| Spread vs Cluster Placement Group | Spread = aislamiento hardware (máx 7/AZ); Cluster = rendimiento máx de red (HPC) |
| DataSync vs Storage Gateway | DataSync = migración/transferencia de archivos; Storage Gateway = extensión on-premises continua |
| DataSync vs DMS | DataSync = archivos; DMS = bases de datos relacionales |
| CloudFront vs S3 Transfer Acceleration | CloudFront = distribución con caché; Transfer Acceleration = uploads a S3 sin caché |
| Redis vs Memcached | Redis = HA, replicación, persistencia; Memcached = caché simple sin HA |

---

## Los "Siempre" del SAA-C03

- **Siempre** que EC2 necesite acceder a servicios AWS → IAM Role + Instance Profile, nunca access keys.
- **Siempre** que ECS task necesite acceder a servicios AWS → Task IAM Role, nunca instance profile.
- **Siempre** que EC2 privado acceda a S3/DynamoDB → VPC Gateway Endpoint, no NAT Gateway.
- **Siempre** que haya migración heterogénea de BD con CDC → AWS SCT + AWS DMS.
- **Siempre** que haya rotación automática de credenciales BD → AWS Secrets Manager (no SSM Parameter Store).
- **Siempre** que se mencione RabbitMQ/ActiveMQ → Amazon MQ.
- **Siempre** que aplicación en EKS deba enrutar por path → AWS Load Balancer Controller + ALB.
- **Siempre** que el escenario mencione EC2 + Fargate en Savings Plan → Compute Savings Plan.
- **Siempre** que haya "orden garantizado" de mensajes → SQS FIFO.
- **Siempre** que haya "mensajes duplicados" en SQS (sin duplicados en cola) → aumentar visibility timeout.
- **Siempre** que NAT Gateway esté presente y el destino sea S3/DynamoDB → reemplazar con VPC Gateway Endpoint.
- **Siempre** que el escenario pida DR Active-Passive multiregional → Route 53 Failover Routing + Health Checks.
- **Siempre** que haya retención de backups RDS > 35 días → AWS Backup.
- **Siempre** que múltiples consumidores de Kinesis necesiten throughput independiente → Enhanced Fan-Out.
- **Siempre** que se mencione "datos deben permanecer on-premises" + "servicios administrados AWS" → VPC Interface Endpoint (PrivateLink).
- **Siempre** que el escenario diga "mínimo overhead" + backup multi-servicio → AWS Backup.
- **Siempre** que haya Lambda + RDS con "too many connections" → RDS Proxy.
- **Siempre** que se necesite CDN + protección WAF en el edge → CloudFront + WAF.

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
| Migración offline con ancho de banda limitado | AWS Snowball Edge |
| Transferencia recurrente de archivos on-premises → S3 | AWS DataSync |

### Base de Datos

| Escenario | Solución |
|-----------|---------|
| Migración heterogénea con CDC | AWS SCT + AWS DMS |
| Poblar staging sin impacto en producción | Aurora Fast Clone |
| Connection pooling / muchas conexiones | Amazon RDS Proxy |
| Lambda + RDS "too many connections" | Amazon RDS Proxy |
| Expirar datos automáticamente después de N días | DynamoDB TTL |
| Session state de aplicaciones web | ElastiCache Redis |
| Cargas intermitentes PostgreSQL | Aurora Serverless v2 |
| Separar carga de reporting de BD primaria | RDS Read Replica |
| Retención backups > 35 días | AWS Backup |

### Red y Conectividad

| Escenario | Solución |
|-----------|---------|
| EC2 privada → S3/DynamoDB sin internet | VPC Gateway Endpoint (gratuito) |
| EC2 privada → internet saliente | NAT Gateway (en subnet pública) |
| On-premises → AWS temporal/barato | Site-to-Site VPN |
| Usuario remoto → VPC | AWS Client VPN |
| Multi-cuenta restricciones centralizadas | SCP en AWS Organizations |
| DR multiregional Active-Passive | Route 53 Failover Routing + Health Checks |
| Compartir recursos entre cuentas AWS | AWS RAM (Resource Access Manager) |
| Múltiples VPCs + on-premises centralizado | AWS Transit Gateway |

### Compute y Escalado

| Escenario | Solución |
|-----------|---------|
| Carga predecible y recurrente | ASG Scheduled Action |
| Carga impredecible | ASG Dynamic/Target Tracking Scaling |
| EC2 + Fargate en Savings Plan | Compute Savings Plan |
| Separar hardware entre instancias | Spread Placement Group |
| Máximo throughput de red entre nodos | Cluster Placement Group |
| IOPS independiente del tamaño del volumen | EBS gp3 |
| Función sin servidor de larga ejecución (< 15 min) | AWS Lambda |
| Contenedores sin gestionar infraestructura | AWS Fargate |

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
| Protección DDoS capa 7 + WAF | CloudFront + AWS WAF |

### Mensajería y Streaming

| Escenario | Solución |
|-----------|---------|
| Orden garantizado de mensajes | SQS FIFO |
| Múltiples consumidores Kinesis sin compartir throughput | Enhanced Fan-Out |
| Cola SQS acumulada → escalar consumidores | ASG basado en `ApproximateNumberOfMessagesVisible` |
| Procesamiento duplicado en SQS | Aumentar visibility timeout (`ChangeMessageVisibility`) |
| RabbitMQ/ActiveMQ administrado | Amazon MQ |
| Fan-out a múltiples suscriptores | Amazon SNS |
| Eventos entre servicios AWS y externos | Amazon EventBridge |

---

*Última actualización: Mayo 2026 | Basado en 40 preguntas del banco SAA-C03*
