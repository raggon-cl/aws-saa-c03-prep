# Banco de Preguntas — AWS SAA-C03

> Preguntas de práctica con respuestas justificadas.
> Formato: Enunciado → Respuesta correcta → Justificación → Descarte de opciones incorrectas → Clave para el examen.

---

## Pregunta 01 — Base de datos / Aurora

**Escenario:** Una empresa ejecuta una aplicación en instalaciones que usa tecnología de base de datos MySQL. La empresa está migrando la aplicación a AWS. La arquitectura actual muestra una elevada actividad de lectura en la base de datos durante el funcionamiento normal. Cada 4 horas, el equipo de desarrolladores realiza una copia completa de base de datos de producción para rellenar una base de datos en el entorno de almacenamiento provisional. Durante este período, los usuarios experimentan una latencia marcada para la aplicación. El equipo de desarrollo no puede usar el entorno de almacenamiento provisional hasta que el procedimiento no se complete. El arquitecto de soluciones tiene que recomendar una arquitectura sustituta que resuelva el problema de latencia de la aplicación. Además, la arquitectura sustituta tiene que permitir al equipo seguir usando el entorno de almacenamiento provisional sin demoras.

**Opciones:**
- A. Usar Amazon Aurora MySQL con réplicas Multi-AZ de Aurora para producción. Rellenar la base de datos de almacenamiento provisional mediante la implementación de una copia de seguridad y restauración que use la utilidad mysqldump.
- B. Usar Amazon Aurora MySQL con réplicas Multi-AZ de Aurora para producción. Usar la clonación de la base de datos para crear una base de datos de almacenamiento provisional.
- C. Usar Amazon RDS para MySQL con una implementación Multi-AZ y réplicas de lectura para producción. Usar la instancia en espera para la base de datos de almacenamiento provisional.
- D. Usar Amazon RDS para MySQL con una implementación Multi-AZ y réplicas de lectura para producción. Rellenar la base de datos de almacenamiento provisional mediante un proceso de copia de seguridad y restauración que use la utilidad mysqldump.

**✅ Respuesta: B**

**Justificación:** Aurora MySQL con réplicas Multi-AZ resuelve la alta lectura. La clonación de Aurora (Aurora Fast Clone) crea una copia de la base de datos en segundos usando copy-on-write, sin duplicar los datos físicamente ni impactar el rendimiento de producción.

**Descarte:**
- A: mysqldump genera carga en producción y es lento. Contradice el requisito de "sin demoras".
- C: El standby de Multi-AZ no es accesible para lectura ni clonación rápida. Solo sirve para failover.
- D: mysqldump tiene el mismo problema que A: proceso lento que genera carga adicional.

**🔑 Clave:** Aurora Fast Clone es la diferenciación crítica de Aurora frente a RDS estándar cuando el escenario menciona poblar entornos de prueba/staging de forma rápida y sin impacto en producción.

---

## Pregunta 02 — Red / VPC Endpoint

**Escenario:** Una empresa tiene una aplicación que se ejecuta en instancias de Amazon EC2 dentro de una subred privada en una VPC. La empresa utiliza un bucket de Amazon S3 para almacenar datos. La VPC contiene una puerta de enlace de NAT en una subred pública para acceder al bucket de S3. Para reducir sus costos, la empresa quiere reemplazar la puerta de enlace de NAT sin afectar a la seguridad ni la redundancia.

**Opciones:**
- A. Reemplazar la puerta de enlace de NAT por una instancia NAT.
- B. Reemplazar la puerta de enlace de NAT por una puerta de enlace de Internet.
- C. Reemplazar la puerta de enlace de NAT por un punto de enlace de VPC de puerta de enlace.
- D. Reemplazar la puerta de enlace de NAT por una conexión de AWS Direct Connect.

**✅ Respuesta: C**

**Justificación:** Un VPC Gateway Endpoint para S3 es completamente gratuito (sin costo por hora ni por GB transferido). El tráfico hacia S3 viaja por la red interna de AWS sin salir a internet. Se configura como una entrada en la route table de la subnet privada.

**Descarte:**
- A: NAT Instance reduce algo el costo pero introduce overhead operacional. No elimina el costo de transferencia.
- B: Internet Gateway requeriría IPs públicas o mover instancias a subnet pública. Viola el requisito de seguridad.
- D: Direct Connect es conectividad híbrida on-premises ↔ AWS. No aplica aquí y su costo es mucho mayor.

**🔑 Clave:** Siempre que EC2 en subnet privada acceda a S3 o DynamoDB pasando por NAT Gateway → la respuesta es VPC Gateway Endpoint. Es el patrón de optimización de costos más frecuente en el SAA-C03.

---

## Pregunta 03 — Migración de base de datos / AWS DMS

**Escenario:** Una empresa almacena datos en una base de datos relacional de Oracle en las instalaciones. La empresa necesita que los datos estén disponibles en Amazon Aurora PostgreSQL para su análisis. La empresa utiliza una conexión de AWS Site-to-Site VPN para conectar la red en las instalaciones a AWS. La empresa debe capturar los cambios que ocurren en la base de datos de origen durante la migración a Aurora PostgreSQL.

**Opciones:**
- A. Usar la Herramienta de conversión de esquemas de AWS (AWS SCT) para convertir el esquema de Oracle a un esquema de Aurora PostgreSQL. Usar la tarea de migración de carga completa de AWS Database Migration Service (AWS DMS) para migrar los datos.
- B. Usar AWS DataSync para migrar los datos a un bucket de Amazon S3. Importar los datos de S3 a Aurora PostgreSQL con la extensión aws_s3 de Aurora PostgreSQL.
- C. Usar la Herramienta de conversión de esquemas de AWS (AWS SCT) para convertir el esquema de Oracle a un esquema de Aurora PostgreSQL. Usar AWS Database Migration Service (AWS DMS) para migrar los datos existentes y replicar los cambios en curso.
- D. Usar un dispositivo de AWS Snowball para migrar los datos a un bucket de Amazon S3. Importar los datos de S3 a Aurora PostgreSQL con la extensión aws_s3 de Aurora PostgreSQL.

**✅ Respuesta: C**

**Justificación:** SCT convierte el esquema Oracle → Aurora PostgreSQL (motores heterogéneos). DMS con full load + CDC carga inicialmente todos los datos y luego replica en tiempo real los cambios del origen usando los redo logs de Oracle. La conectividad ya existe vía Site-to-Site VPN.

**Descarte:**
- A: SCT + DMS full load únicamente no captura los cambios durante la migración. Genera pérdida de datos en el cutover.
- B: DataSync es para transferencia de archivos (NFS, SMB, S3), no para migración de bases de datos relacionales.
- D: Snowball es para transferencias masivas offline. Aquí ya existe VPN. Además no captura CDC.

**🔑 Clave:** El patrón SCT + DMS con CDC es la respuesta canónica para migraciones heterogéneas de BD que requieren mínimo downtime y captura de cambios.

---

## Pregunta 04 — Seguridad / IAM Role

**Escenario:** Una empresa ejecuta una aplicación en instancias de Amazon EC2. La aplicación necesita acceder a una base de datos de Amazon RDS. La empresa quiere conectar las instancias de EC2 de manera segura a la base de datos de RDS siguiendo el principio de mínimo privilegio.

**Opciones:**
- A. Crear un usuario de IAM que tenga una política que conceda permisos administrativos. Utilizar las claves de acceso del usuario de IAM en las instancias de EC2 para acceder a la base de datos de RDS.
- B. Crear un usuario de IAM que tenga una política que conceda los permisos mínimos necesarios para acceder a la base de datos de RDS. Insertar las claves de acceso del usuario de IAM en las instancias de EC2 para acceder a la base de datos de RDS.
- C. Crear un rol de IAM que tenga una política que otorgue los permisos mínimos necesarios para acceder a la base de datos de RDS. Adjuntar la clave de acceso al rol de IAM y la clave secreta del rol de IAM al perfil de instancia de EC2.
- D. Crear un rol de IAM que tenga una política que otorgue los permisos mínimos necesarios para acceder a la base de datos de RDS. Adjuntar el rol de IAM a un perfil de instancia de EC2. Asociar el perfil de instancia a las instancias.

**✅ Respuesta: D**

**Justificación:** IAM Role + Instance Profile es el mecanismo nativo de AWS para otorgar permisos a instancias EC2 sin usar credenciales estáticas. Las credenciales son temporales y se rotan automáticamente vía el metadata service (169.254.169.254).

**Descarte:**
- A: Viola el principio de mínimo privilegio. Además, insertar access keys en instancias EC2 es una mala práctica (riesgo de exposición).
- B: Mejora los permisos pero sigue usando credenciales estáticas. Si una instancia se compromete, las credenciales quedan expuestas.
- C: Los roles no usan access keys y secret keys. Su valor es precisamente no usar credenciales estáticas.

**🔑 Clave:** EC2 necesita acceder a cualquier servicio AWS → siempre IAM Role + Instance Profile, nunca access keys embebidas.

---

## Pregunta 05 — Almacenamiento S3 / Storage Classes

**Escenario:** Una empresa quiere usar Amazon S3 para la copia secundaria de su conjunto de datos en sus instalaciones, a la que accedería en raras ocasiones. El costo de almacenamiento de la solución debe ser mínimo.

**Opciones:**
- A. S3 Standard
- B. S3 Intelligent-Tiering
- C. S3 Standard - Acceso poco frecuente (S3 Standard-IA)
- D. S3 One Zone - Acceso poco frecuente (S3 One Zone-IA)

**✅ Respuesta: C**

**Justificación:** S3 Standard-IA tiene costo de almacenamiento significativamente menor que S3 Standard y mantiene resiliencia multi-AZ (apropiada para una copia de seguridad). El acceso poco frecuente coincide con el patrón descrito.

**Descarte:**
- A: Más caro en almacenamiento. Adecuado para acceso frecuente, no para este caso.
- B: Intelligent-Tiering tiene costo de monitoreo por objeto. Si el acceso es conocidamente infrecuente, añade costo innecesario.
- D: One Zone-IA almacena en una sola AZ. Para una copia de seguridad, perder resiliencia multi-AZ es un riesgo inaceptable.

**🔑 Clave:** La distinción entre C y D gira en torno a si los datos son reproducibles fácilmente (One Zone-IA) o si son la única/principal copia de seguridad (Standard-IA).

---

## Pregunta 06 — Auto Scaling / Scheduled Action

**Escenario:** Una empresa tiene una carga de trabajo grande que se ejecuta todos los viernes a la noche, en instancias EC2 en dos zonas de disponibilidad en us-east-1. Normalmente, la empresa no ejecuta más de dos instancias en todo momento. Sin embargo, la empresa quiere escalar a seis instancias todos los viernes para tener una carga de trabajo mayor que se repite con regularidad. ¿Qué solución cumplirá estos requisitos con la MENOR sobrecarga operativa?

**Opciones:**
- A. Crear un recordatorio en Amazon EventBridge para escalar las instancias.
- B. Crear un grupo de escalado automático que tenga una acción programada.
- C. Crear un grupo de escalado automático que use escalado manual.
- D. Crear un grupo de escalado automático que use escalado automático.

**✅ Respuesta: B**

**Justificación:** La Scheduled Action del ASG escala proactivamente antes del pico conocido. Se configura una vez con expresión cron (ej. `0 20 * * 5`). Cero intervención manual posterior → mínimo overhead operativo. Define `min=6, max=6, desired=6` para los viernes y revierte al terminar.

**Descarte:**
- A: EventBridge para escalar instancias directamente añade complejidad innecesaria (necesita Lambda + llamadas a API). El ASG ya tiene esta capacidad nativa.
- C: Escalado manual requiere que alguien se acuerde cada viernes. Máximo overhead operativo.
- D: Dynamic scaling es válido para cargas impredecibles, pero subóptimo aquí por ser reactivo.

**🔑 Clave:** Carga predecible y recurrente → ASG con Scheduled Action. Dynamic scaling es para cargas impredecibles.

---

## Pregunta 07 — Almacenamiento EBS / gp3

**Escenario:** Una empresa de comercio electrónico apoya una base de datos PostgreSQL en una instancia de Amazon EC2. La base de datos almacena los datos en volúmenes de Amazon Elastic Block Store. La empresa periódicamente ejecuta un script en la base de datos para informar nuevos servidores. El script que se ejecuta en la base de datos afecta negativamente al rendimiento de una aplicación crítica. La empresa necesita mejorar el rendimiento de la aplicación con un mínimo de 15.000 IOPS. El rendimiento de IOPS del disco que sea independiente de la capacidad de almacenamiento en disco.

**Opciones:**
- A. Configurar los volúmenes EBS de SSD de uso general (gp2). Aprovisionar un volumen de 5 TB.
- B. Configurar los volúmenes EBS de SSD (io1) de IOPS aprovisionadas. Aprovisionar 15.000 IOPS.
- C. Configurar los volúmenes EBS de SSD de uso general (gp3). Aprovisionar 15.000 IOPS.
- D. Configurar los volúmenes magnéticos de EBS para lograr el máximo de IOPS.

**✅ Respuesta: C**

**Justificación:** gp3 permite provisionar hasta 16.000 IOPS de forma independiente al tamaño del volumen. Es ~20% más barato que gp2 y significativamente más barato que io1. Con gp3 se pueden tener 15.000 IOPS sin pagar por TBs innecesarios.

**Descarte:**
- A: En gp2, para obtener 15.000 IOPS se necesitan ~5 TB (3 IOPS/GB). Pagar por almacenamiento que no se necesita solo para alcanzar las IOPS.
- B: io1 es más caro que gp3 para el mismo nivel de IOPS. Solo se justifica con más de 16.000 IOPS o latencia sub-milisegundo consistente.
- D: Volúmenes magnéticos tienen cientos de IOPS como máximo. No alcanzan 15.000 IOPS.

**🔑 Clave:** Desde 2021, gp3 reemplaza a gp2 como la opción por defecto de costo-rendimiento. io1/io2 solo si se requieren IOPS > 16.000 o ratio IOPS:GB superior a 500:1.

---

## Pregunta 08 — Seguridad / Secrets Manager

**Escenario:** Una empresa ejecuta una función Node.js en un servidor en su centro de datos en las instalaciones. El centro de datos almacena datos en una base de datos de PostgreSQL. La empresa desea migrar la aplicación a AWS y reemplazar el servidor de la aplicación Node.js con AWS Lambda. También quiere usar Amazon RDS para PostgreSQL y asegurarse de que las credenciales de la base de datos se administren de forma segura.

**Opciones:**
- A. Almacenar las credenciales de la base de datos como un parámetro en el almacén de parámetros de AWS Systems Manager. Configurar el almacén de parámetros para rotar los secretos cada 30 días. Actualizar la función Lambda para recuperar las credenciales del parámetro.
- B. Almacenar las credenciales de la base de datos como un secreto en AWS Secrets Manager. Configurar AWS Secrets Manager para rotar automáticamente las credenciales cada 30 días. Actualizar la función Lambda para recuperar las credenciales del secreto.
- C. Almacenar las credenciales de la base de datos como una variable del entorno Lambda cifrada. Escribir una función Lambda personalizada para rotar las credenciales. Programar la función Lambda para que se ejecute cada 30 días.
- D. Almacenar las credenciales de la base de datos como clave en AWS Key Management Service (AWS KMS). Configurar la rotación automática de la clave. Actualizar la función Lambda para recuperar las credenciales de la clave de KMS.

**✅ Respuesta: B**

**Justificación:** AWS Secrets Manager tiene rotación automática nativa con RDS. Sin código personalizado, gestiona el ciclo completo: genera nueva contraseña, la actualiza en RDS y actualiza el secreto. Lambda recupera el secreto en runtime sin credenciales hardcodeadas.

**Descarte:**
- A: SSM Parameter Store no tiene rotación automática nativa para credenciales de RDS. Habría que construir una Lambda personalizada → mayor overhead.
- C: Variables de entorno Lambda con Lambda personalizada para rotar es desarrollo y mantenimiento adicional innecesario.
- D: KMS gestiona claves de cifrado, no credenciales de aplicación. No tiene concepto de username/password.

**🔑 Clave:** La distinción crítica entre A y B es la rotación automática nativa para RDS. Secrets Manager la tiene integrada; SSM Parameter Store no. Cuando el escenario menciona RDS + rotación automática → siempre Secrets Manager.

---

## Pregunta 09 — Alta Disponibilidad / Route 53

**Escenario:** Un equipo de soluciones está diseñando una estrategia de recuperación de desastres (DR) multiregional para una empresa. La empresa ejecuta una aplicación en instancias de Amazon EC2 en grupos de escalado automático que están detrás de un balanceador de carga de aplicación (ALB). La aplicación debe responder a las consultas de DNS de la región secundaria si la aplicación de la región principal falla. Solo una región debe atender el tráfico a la vez.

**Opciones:**
- A. Crear un punto de conexión de salida en Amazon Route 53 Resolver. Crear reglas de reenvío que determinen cómo se reenvían las consultas a los solucionadores de DNS de la red. Asociar las reglas a las VPC de cada región.
- B. Crear registros de DNS principales y secundarios en Amazon Route 53. Configurar las comprobaciones de estado y una política de conmutación por error. Asociar el perfil a las VPC de cada región.
- C. Crear una política de tráfico en Amazon Route 53. Utilizar una política de enrutamiento de geolocalización y un tipo de valor de ponderación para dirigir la carga de aplicación del ELB.
- D. Crear un perfil de Amazon Route 53. Asociar los recursos de DNS al perfil. Asociar el perfil a las VPC de cada región.

**✅ Respuesta: B**

**Justificación:** Route 53 Failover Routing Policy con health checks implementa Active-Passive DR. Dos registros DNS (Primary y Secondary) apuntando a sus respectivos ALBs. Cuando el health check del Primary falla, Route 53 responde automáticamente con el Secondary. Zero intervención manual.

**Descarte:**
- A: Route 53 Resolver es para resolución DNS híbrida on-premises ↔ VPC, no para DR multiregional.
- C: La geolocalización enruta según origen geográfico del usuario, no según estado de salud del endpoint.
- D: Route 53 Profiles es para compartir configuraciones DNS entre VPCs. No es mecanismo de failover entre regiones.

**🔑 Clave:** Active-Passive DR multiregional → Route 53 Failover Routing Policy + Health Checks. Si ambas regiones atienden tráfico simultáneamente → Active-Active con weighted o latency routing.

---

## Pregunta 10 — Almacenamiento / VPC Endpoint para BD

**Escenario:** Una empresa planea implementar una plataforma de procesamiento de datos en AWS. La plataforma de procesamiento de datos se basa en PostgreSQL. La empresa almacena los datos que la plataforma debe procesar en sus instalaciones. Para cumplir con las regulaciones, la empresa no puede migrar los datos a la nube. Sin embargo, la empresa quiere utilizar las soluciones de análisis de datos administradas por AWS. La empresa utiliza una conexión de AWS Site-to-Site VPN para conectar la red en las instalaciones a AWS.

**Opciones:**
- A. Crear una base de datos de Amazon RDS para PostgreSQL en una VPC. Crear un punto de conexión de VPC de la interfaz para conectar la base de datos PostgreSQL en las instalaciones de la empresa a la base de datos de RDS para PostgreSQL.
- B. Crear instancias de Amazon EC2 en un grupo de escalado automático en AWS Outposts. Instalar el software de análisis de datos PostgreSQL en las instancias.
- C. Crear un clúster de Amazon EMR en AWS Outposts. Conectar el clúster de EMR a la base de datos PostgreSQL en las instalaciones de la empresa para procesar los datos de forma local.
- D. Crear un clúster de Amazon EMR en una VPC. Conectar el clúster de EMR a Amazon RDS para SQL Server con un servicio para conectarse a la plataforma de procesamiento de datos de la empresa.

**✅ Respuesta: A**

**Justificación:** RDS PostgreSQL en VPC + VPC Interface Endpoint (PrivateLink) permite que la plataforma on-premises se conecte a RDS de forma privada sin atravesar internet público. Los datos permanecen procesándose on-premises pero se conectan al servicio administrado de AWS.

**Descarte:**
- B: EC2 con PostgreSQL en Outposts no es un servicio administrado. No cumple el requisito de "soluciones de análisis administradas por AWS".
- C: EMR es para procesamiento de datos tipo Hadoop/Spark, no para administrar una base de datos PostgreSQL.
- D: Cambia el motor a SQL Server (viola el requisito de PostgreSQL) y no resuelve la restricción de datos on-premises.

**🔑 Clave:** "Datos deben permanecer on-premises" + "usar servicios administrados AWS" → PrivateLink/VPC Interface Endpoint como puente entre on-premises y el servicio administrado en AWS.

---

## Pregunta 11 — Almacenamiento Híbrido / Storage Gateway

**Escenario:** Una empresa está desarrollando una nueva aplicación web en AWS. La aplicación necesita consumir archivos de una aplicación heredada en las instalaciones de la empresa que ejecuta un proceso por lotes y envía aproximadamente 1 GB de datos cada noche a un montaje de archivos NFS. Un arquitecto de soluciones debe diseñar una solución de almacenamiento que requiera cambios mínimos en la aplicación heredada y mantenga los costos bajos.

**Opciones:**
- A. Implementar un Outpost en AWS Outposts en la ubicación en las instalaciones de la empresa donde está almacenada la aplicación heredada. Configurar la aplicación heredada y la aplicación web para almacenar y recuperar los archivos en Amazon S3 en Outpost.
- B. Implementar un volume gateway en AWS Storage Gateway en las instalaciones de la empresa. Dirigir la aplicación heredada al Volume Gateway. Configurar la aplicación web para que utilice el bucket de Amazon S3 que utiliza el Volume Gateway.
- C. Implementar un punto de conexión de interfaz de Amazon S3 en AWS. Volver a configurar la aplicación heredada para almacenar los archivos de forma directa en un punto de conexión de Amazon S3. Configurar la aplicación web para recuperar los archivos de Amazon S3.
- D. Implementar Amazon S3 File Gateway en las instalaciones de la empresa. Dirigir la aplicación heredada a File Gateway. Configurar la aplicación web para recuperar los archivos del bucket de S3 que usa File Gateway.

**✅ Respuesta: D**

**Justificación:** S3 File Gateway presenta una interfaz NFS/SMB a la aplicación heredada (cero cambios en la aplicación). Los archivos se almacenan automáticamente en S3. La aplicación web en AWS accede directamente al bucket S3. File Gateway incluye caché local para baja latencia.

**Descarte:**
- A: Outposts es costoso y con alto overhead operativo. No justificado para 1 GB/noche.
- B: Volume Gateway presenta volúmenes iSCSI, no NFS. La aplicación heredada usa NFS → habría que cambiarla.
- C: Requiere modificar la aplicación heredada para usar la API de S3. Viola el requisito de mínimos cambios.

**🔑 Clave:** Aplicación on-premises usa NFS o SMB + necesita almacenar en S3 + mínimos cambios → S3 File Gateway.

---

## Pregunta 12 — Red / VPN

**Escenario:** Una empresa planea migrar las cargas de trabajo de un centro de datos en sus instalaciones a AWS. La empresa quiere probar las cargas de trabajo migradas en AWS mediante la configuración de un entorno de pruebas temporal en una región de AWS. El entorno de pruebas incluye una única instancia de Amazon EC2 en la VPC. La empresa no tiene ningún requisito de ancho de banda o de alta tolerancia a errores para el entorno de pruebas. La empresa necesita establecer una conectividad bidireccional entre el centro de datos en sus instalaciones y el entorno de pruebas.

**Opciones:**
- A. Crear una conexión de AWS Direct Connect entre el centro de datos en las instalaciones de la empresa y AWS.
- B. Crear una conexión AWS Site-to-Site VPN entre el centro de datos en las instalaciones de la empresa y AWS.
- C. Crear múltiples conexiones AWS Site-to-Site VPN entre el centro de datos en las instalaciones de la empresa y AWS.
- D. Crear una conexión AWS Client VPN entre el centro de datos en las instalaciones de la empresa y AWS.

**✅ Respuesta: B**

**Justificación:** Site-to-Site VPN es rápida de implementar (minutos), tiene costo bajo (~$0.05/hora + GB), provee conectividad bidireccional cifrada. Para un entorno de pruebas temporal con una sola EC2, es completamente suficiente.

**Descarte:**
- A: Direct Connect tarda semanas y tiene costos fijos significativos. Completamente desproporcionado para un entorno de pruebas temporal.
- C: Múltiples Site-to-Site VPN añade redundancia. El escenario indica explícitamente que no hay requisitos de redundancia.
- D: Client VPN es para usuarios individuales (laptops) a la VPC, no para conectar dos redes.

**🔑 Clave:** Client VPN = usuario→VPC. Site-to-Site VPN = red→red. Direct Connect solo cuando hay requisitos de ancho de banda consistente, baja latencia o cumplimiento normativo que prohibe internet público.

---

## Pregunta 13 — Almacenamiento Compartido / FSx

**Escenario:** Una empresa tiene una aplicación basada en Windows que se tiene que migrar a AWS. La aplicación requiere el uso de un sistema compartido de archivos de Windows conectado a varias instancias de Windows de Amazon EC2 implementadas en distintas zonas de disponibilidad.

**Opciones:**
- A. Configurar AWS Storage Gateway en modo puerta de enlace de volumen. Montar el volumen para cada instancia de Windows.
- B. Configurar Amazon FSx para Windows File Server. Montar el sistema de archivos de Amazon FSx para cada instancia de Windows.
- C. Configurar un sistema de archivos mediante Amazon Elastic File System (Amazon EFS). Montar el sistema de archivos de EFS en cada instancia de Windows.
- D. Configurar un volumen de Amazon Elastic Block Store (Amazon EBS) con el tamaño necesario. Adjuntar cada instancia de EC2 al sistema de archivos dentro del volumen en cada instancia de Windows.

**✅ Respuesta: B**

**Justificación:** FSx for Windows File Server usa protocolo nativo SMB, es compatible con instancias EC2 Windows sin configuración adicional, soporta Multi-AZ con failover automático e integración con Active Directory.

**Descarte:**
- A: Volume Gateway presenta volúmenes iSCSI, no sistemas de archivos compartidos SMB. No sirve para compartir entre múltiples instancias simultáneamente.
- C: EFS usa protocolo NFS (nativo de Linux/Unix). Para cargas Windows, FSx es el servicio correcto.
- D: Los volúmenes EBS son de bloque y solo pueden adjuntarse a una instancia a la vez. No sirven para compartir entre múltiples instancias en distintas AZs.

**🔑 Clave:** Windows + SMB → FSx for Windows File Server. Linux + NFS → Amazon EFS. HPC + Lustre → FSx for Lustre.

---

## Pregunta 14 — EKS / Load Balancer

**Escenario:** Una empresa ejecuta una aplicación en contenedores mediante Amazon Elastic Kubernetes Service (Amazon EKS). La aplicación incluye microservicios que administran clientes y realizan pedidos. La empresa debe dirigir las solicitudes entrantes a los microservicios correspondientes.

**Opciones:**
- A. Utilizar el controlador del balanceador de carga de AWS para aprovisionar un Network Load Balancer.
- B. Utilizar el controlador del balanceador de carga de AWS para aprovisionar un Application Load Balancer.
- C. Utilizar una función de AWS Lambda para conectar las solicitudes a Amazon EKS.
- D. Utilizar Amazon API Gateway para conectar las solicitudes a Amazon EKS.

**✅ Respuesta: B**

**Justificación:** El enrutamiento por path hacia distintos microservicios es enrutamiento de capa 7 (HTTP/HTTPS), dominio del ALB. El AWS Load Balancer Controller crea automáticamente un ALB con las reglas de enrutamiento cuando se define un Ingress de Kubernetes. Es la integración nativa y más rentable.

**Descarte:**
- A: NLB opera en capa 4 (TCP/UDP). No tiene capacidad de enrutamiento por path o por host HTTP.
- C: Lambda añade latencia, complejidad y costo de invocación por request. No es la solución nativa.
- D: API Gateway tiene costo por millón de requests significativamente mayor que ALB y requiere configuración adicional (VPC Link).

**🔑 Clave:** EKS + enrutamiento HTTP entre microservicios → AWS Load Balancer Controller + ALB vía Ingress.

---

## Pregunta 15 — Savings Plans

**Escenario:** Una empresa quiere una solución de cómputo flexible que incluya Amazon EC2 instances y AWS Fargate. La empresa no quiere comprometerse a contratos de varios años.

**Opciones:**
- A. Purchase a 1-year EC2 Instance Savings Plan with the All Upfront option.
- B. Purchase a 1-year Compute Savings Plan with the No Upfront option.
- C. Purchase a 1-year Compute Savings Plan with the Partial Upfront option.
- D. Purchase a 1-year Compute Savings Plan with the All Upfront option.

**✅ Respuesta: B**

**Justificación:** Compute Savings Plan cubre EC2 (cualquier familia/región/OS) + Fargate + Lambda. No Upfront maximiza la flexibilidad financiera (cero desembolso inicial). El escenario pide flexibilidad y no compromiso multi-año → 1 año es el mínimo, No Upfront es el más flexible.

**Descarte:**
- A: EC2 Instance Savings Plan no cubre Fargate. Eliminada directamente.
- C: Partial Upfront requiere desembolso inicial parcial. Menos flexible que No Upfront.
- D: All Upfront es el menos flexible: pago total por adelantado.

**🔑 Clave:** Siempre que el escenario mencione EC2 + Fargate → solo Compute Savings Plan aplica. EC2 Instance Savings Plan nunca cubre Fargate.

---

## Pregunta 16 — Monitoreo / AWS Config

**Escenario:** Una empresa ejecuta cargas de trabajo de producción en su cuenta de AWS. Varios equipos crean y mantienen las cargas de trabajo. La empresa necesita capturar los cambios en las configuraciones de los recursos como elementos de configuración sin cambios ni modificaciones en los recursos existentes.

**Opciones:**
- A. Usar AWS Config. Iniciar el registrador de configuración para que los recursos de AWS detecten cambios en la configuración de los recursos.
- B. Usar AWS CloudFormation. Iniciar la detección de desviaciones para capturar cambios en las configuraciones de los recursos.
- C. Usar Amazon Detective para detectar, analizar e investigar cambios en las configuraciones de los recursos.
- D. Usar AWS Audit Manager para capturar eventos de administración y eventos de servicio global para las configuraciones de los recursos.

**✅ Respuesta: A**

**Justificación:** AWS Config registra continuamente el estado de configuración de los recursos AWS, captura cada cambio como un configuration item, opera de forma completamente pasiva (no modifica recursos), y permite consultar el historial de configuración de cualquier recurso.

**Descarte:**
- B: CloudFormation drift detection solo aplica a recursos desplegados via CloudFormation y es una operación bajo demanda, no continua.
- C: Amazon Detective es para investigación de seguridad, no para rastrear cambios de configuración de recursos.
- D: AWS Audit Manager es para recopilar evidencia de cumplimiento normativo, no para tracking continuo de configuraciones.

**🔑 Clave:** AWS Config = visibilidad y historial de configuraciones de recursos. Referencia para preguntas sobre "¿quién cambió qué recurso y cuándo?" sin intervención sobre los recursos.

---

## Pregunta 17 — Seguridad / Cifrado S3

**Escenario:** Un hospital necesita almacenar registros de pacientes en un bucket de Amazon S3. El equipo de compliance del hospital debe asegurarse de que toda la información de salud personal (PHI) esté cifrada en tránsito y en reposo. El equipo de compliance también debe administrar el ciclo de vida de los datos en reposo.

**Opciones:**
- A. Crear un certificado SSL/TLS en AWS Certificate Manager (ACM). Asociar el certificado con Amazon S3. Configurar el cifrado predeterminado para cada bucket de S3 para usar el cifrado del lado del servidor con claves administradas por AWS (SSE-KMS). Asignar el equipo de compliance para administrar las políticas de claves de KMS.
- B. Usar la condición aws:SecureTransport en la política del bucket de S3 para permitir solo conexiones cifradas sobre HTTPS (TLS). Configurar el bucket de S3 para usar el cifrado del lado del servidor con claves administradas por Amazon S3 (SSE-S3). Asignar el equipo de compliance para administrar las claves SSE-S3.
- C. Usar la condición aws:SecureTransport en la política del bucket de S3 para permitir solo conexiones cifradas sobre HTTPS (TLS). Configurar el bucket de S3 para usar el cifrado del lado del servidor con claves de AWS KMS (SSE-KMS). Asignar el equipo de compliance para administrar las políticas de claves de KMS.
- D. Usar la condición aws:SecureTransport en la política del bucket de S3 para permitir solo conexiones cifradas sobre HTTPS (TLS). Usar Amazon Macie para cifrar los datos confidenciales que se almacenan en Amazon S3. Asignar el equipo de compliance para administrar Macie.

**✅ Respuesta: C**

**Justificación:** `aws:SecureTransport` en la bucket policy fuerza HTTPS (cifrado en tránsito). SSE-KMS con Customer Managed Keys da control granular al equipo de compliance sobre quién puede usar las claves, con auditoría completa vía CloudTrail y rotación programable.

**Descarte:**
- A: ACM gestiona certificados para ALB/CloudFront/API Gateway, no para buckets S3 directamente. S3 ya usa HTTPS nativo.
- B: SSE-S3 cifra en reposo pero las claves las gestiona completamente AWS. El compliance no tiene control sobre el ciclo de vida.
- D: Amazon Macie descubre y clasifica datos sensibles en S3, no los cifra. No cumple el requisito de cifrado en reposo.

**🔑 Clave:** Forzar HTTPS en S3 → `aws:SecureTransport: false` → Deny en bucket policy. Control de claves por el cliente → SSE-KMS con CMK.

---

## Pregunta 18 — Costos / Cost Allocation Tags

**Escenario:** Una empresa usa AWS para ejecutar sus cargas de trabajo. La empresa usa AWS Organizations para administrar múltiples cuentas de AWS. La empresa necesita identificar qué departamentos son responsables de costos específicos. Se crean nuevas cuentas constantemente en la estructura de Organizations. El pipeline CI/CD ya agrega la etiqueta department a todos los recursos de AWS. La empresa quiere usar un reporte de AWS Cost Explorer para identificar los costos de servicio por departamento desde todas las cuentas de AWS.

**Opciones:**
- A. Activar la etiqueta de asignación de costos aws:createdBy y la etiqueta de asignación de costos department en la cuenta de administración. Configurar Cost Explorer.
- B. Crear un nuevo reporte de costos y uso en Cost Explorer. Agrupar por la etiqueta de asignación de costos department. Aplicar un filtro para ver todas las cuentas vinculadas y servicios.
- C. Activar solo la etiqueta de asignación de costos department en la cuenta de administración.
- D. Crear un nuevo reporte de costos y uso en Cost Explorer. Agrupar por la etiqueta de asignación de costos department sin ningún otro filtro.

**✅ Respuesta: C y D**

**Justificación:** C: Los cost allocation tags deben activarse en la management account. Como CI/CD ya aplica el tag `department`, solo hay que activarlo. Activar `aws:createdBy` es innecesario para este objetivo. D: Agrupar por `department` sin filtros muestra el costo consolidado por departamento desde todas las cuentas. Los filtros por linked accounts fragmentarían la vista.

**Descarte:**
- A: Activar `aws:createdBy` además de `department` es innecesario y añade overhead.
- B: El filtro por linked accounts y servicios fragmenta la vista cuando el objetivo es visibilidad consolidada por departamento.

**🔑 Clave:** Cost allocation tags: activar en management account → aparecen en Cost Explorer → agrupar por ese tag. El CI/CD que ya aplica tags significa que no hay trabajo adicional de etiquetado.

---

## Pregunta 19 — Seguridad / Cognito + S3 Endpoint

**Escenario:** Una empresa aloja una aplicación en una subred privada. La empresa ha integrado la aplicación con Amazon Cognito. La empresa usa un Amazon Cognito user pool para autenticar a los usuarios. La empresa necesita modificar la aplicación para que pueda almacenar de forma segura documentos de usuario en Amazon S3.

**Opciones:**
- A. Crear un Amazon Cognito identity pool para generar tokens de acceso seguros de Amazon S3. Configurar la aplicación para usar el Cognito Identity Pool instance para almacenar la sesión.
- B. Usar el Amazon Cognito user pool existente para generar tokens de acceso de Amazon S3 cuando los usuarios inicien sesión correctamente.
- C. Crear un punto de conexión de VPC de Amazon S3 en la misma VPC donde la empresa aloja la aplicación.
- D. Crear una NAT gateway en la VPC donde la empresa aloja la aplicación. Asignar una política al bucket de S3 para denegar cualquier solicitud que no se inicie desde Amazon Cognito.

**✅ Respuesta: A y C**

**Justificación:** A: El Identity Pool intercambia el token del User Pool por credenciales temporales de IAM con permisos específicos a S3. Es el patrón estándar para que usuarios autenticados accedan a recursos AWS. C: La aplicación está en subnet privada. Un S3 Gateway Endpoint permite que acceda a S3 a través de la red interna de AWS sin salir a internet.

**Descarte:**
- B: Los User Pools gestionan autenticación de usuarios, no generan credenciales AWS para acceder a servicios como S3. Ese es el rol del Identity Pool.
- D: NAT Gateway permite salida a internet pero es innecesario y costoso si se usa VPC Endpoint. Una bucket policy basada en origen Cognito no es el mecanismo correcto.

**🔑 Clave:** User Pool = autenticación (login). Identity Pool = autorización (credenciales AWS temporales). Subnet privada + S3 → siempre VPC Gateway Endpoint.

---

## Pregunta 20 — Arquitectura / Two-Tier HA

**Escenario:** Una empresa ejecuta un sitio web de comercio electrónico de dos niveles. El nivel web consiste en un balanceador de carga de aplicación (ALB). El nivel de base de datos usa una instancia de Amazon RDS de base de datos. Las instancias de EC2 y las instancias de base de datos de RDS no deben estar expuestas a la internet pública. Las instancias de EC2 requieren acceso a internet para completar el procesamiento de pagos de pedidos a través de un servicio web de un tercero. La aplicación debe tener alta disponibilidad.

**Opciones:**
- A. Usar un grupo de Auto Scaling para lanzar las instancias de EC2 en subredes privadas. Implementar las instancias de base de datos de RDS Multi-AZ de RDS en subredes privadas.
- B. Configurar una VPC con dos subredes privadas y dos NAT gateways en todas las zonas de disponibilidad. Implementar un balanceador de carga de aplicación en las subredes privadas.
- C. Usar un grupo de Auto Scaling para lanzar las instancias de EC2 en subredes públicas en dos zonas de disponibilidad. Implementar las instancias de RDS Multi-AZ de base de datos en subredes privadas.
- D. Configurar una VPC con dos subredes públicas, dos subredes privadas y dos NAT gateways en las dos zonas de disponibilidad. Implementar un balanceador de carga de aplicación en las subredes públicas.
- E. Configurar una VPC con una subred pública en la que haya un balanceador de carga de aplicación en las subredes públicas.

**✅ Respuesta: A y D**

**Justificación:** A: EC2 e RDS en subnets privadas (no expuestos a internet). ASG para HA y escalado. RDS Multi-AZ para HA de BD. D: 2 NAT Gateways (uno por AZ, para HA de salida a internet). ALB en subnets públicas recibe tráfico y lo reenvía a EC2 privadas. 2 AZs para alta disponibilidad completa.

**Descarte:**
- B: Solo una subnet pública y NAT Gateway compartido → single point of failure.
- C: EC2 en subnets públicas viola el requisito de no exposición a internet.
- E: No especifica arquitectura Multi-AZ completa.

**🔑 Clave:** El patrón de HA en AWS: mínimo 2 AZs, NAT Gateway por AZ (no compartido), ALB en subnets públicas, compute y datos en subnets privadas.

---

## Pregunta 21 — Seguridad / SCP

**Escenario:** Un equipo de seguridad quiere limitar el acceso a servicios o acciones en todas las cuentas AWS del equipo. Todas las cuentas pertenecen a una organización grande en AWS Organizations. La solución debe ser escalable y debe haber un único punto donde se puedan mantener los permisos.

**Opciones:**
- A. Crear una ACL para brindar acceso a los servicios o acciones.
- B. Crear un grupo de seguridad para permitir cuentas y adjuntarlo a los grupos de usuarios.
- C. Crear roles entre cuentas en cada cuenta para denegar acceso a los servicios o acciones.
- D. Crear una política de control de servicios en el root organizational unit para denegar acceso a los servicios o acciones.

**✅ Respuesta: D**

**Justificación:** Las SCPs aplicadas en el root OU afectan todas las cuentas hijas automáticamente. Un único punto de administración. Cuando se crea una nueva cuenta en la organización, hereda la SCP automáticamente. Ningún usuario ni rol en las cuentas puede exceder los permisos que la SCP permite.

**Descarte:**
- A: Las ACLs de S3 son para control de acceso a buckets/objetos, no para restringir servicios a nivel de cuenta.
- B: Security Groups controlan tráfico de red, no permisos IAM sobre servicios AWS.
- C: Crear roles en cada cuenta individualmente no es escalable y no tiene un punto centralizado de control.

**🔑 Clave:** SCP = guardrail centralizado para múltiples cuentas en AWS Organizations. Aplicada en el root OU → afecta todas las cuentas.

---

## Pregunta 22 — Red / Security Groups ALB

**Escenario:** Una empresa está diseñando una aplicación web con un balanceador de carga de aplicación (ALB) orientado a internet. La empresa necesita que el ALB reciba tráfico web HTTPS de la internet pública. El ALB también debe enviar solo tráfico HTTPS a los servidores de aplicaciones web en las instancias de Amazon EC2 en el puerto 443. El ALB debe realizar una comprobación de estado de los servidores de aplicaciones web sobre HTTPS en el puerto 8443.

**Opciones:**
- A. Allow HTTPS inbound traffic from 0.0.0.0/0 for port 443.
- B. Allow all outbound traffic to 0.0.0.0/0 for port 443.
- C. Allow HTTPS outbound traffic to the web application instances for port 443.
- D. Allow HTTPS inbound traffic from the web application instances for port 443.
- E. Allow HTTPS outbound traffic to the web application instances for the health check on port 8443.
- F. Allow HTTPS inbound traffic from the web application instances for the health check on port 8443.

**✅ Respuesta: A, C y E**

**Justificación:** A: ALB recibe HTTPS desde internet (inbound 443 desde 0.0.0.0/0). C: ALB envía tráfico a las instancias EC2 (outbound 443 hacia el Security Group de las instancias). E: Health checks los inicia el ALB hacia las instancias (outbound 8443).

**Descarte:**
- B: Demasiado amplio. El ALB no necesita enviar a cualquier destino en internet.
- D: El ALB no recibe tráfico entrante desde las instancias web. El flujo es ALB → instancias, no al revés.
- F: El health check lo inicia el ALB hacia las instancias (outbound), no al revés.

**🔑 Clave:** Para Security Groups de ALB: internet → ALB (inbound 443) y ALB → EC2 (outbound 443 + puerto de health check). Los health checks son tráfico saliente del ALB hacia las instancias.

---

## Pregunta 23 — Mensajería / Amazon MQ

**Escenario:** Una empresa ejecuta su aplicación de comercio electrónico en AWS. Cada nuevo pedido se publica como un mensaje en una cola de RabbitMQ que se ejecuta en una instancia de Amazon EC2 en una sola zona de disponibilidad. Los mensajes son procesados por una aplicación diferente que se ejecuta en una instancia EC2 separada. La base de datos PostgreSQL está en otro EC2. La empresa necesita rediseñar su arquitectura para ofrecer la mayor disponibilidad con la menor sobrecarga operativa.

**Opciones:**
- A. Migrar la cola a un par redundante (activo/en espera) de instancias de RabbitMQ de Amazon MQ. Crear un grupo de escalado automático Multi-AZ para EC2 que aloja la aplicación. Crear un despliegue de Amazon RDS Multi-AZ para PostgreSQL.
- B. Migrar la cola a un par redundante (activo/en espera) de instancias de RabbitMQ de Amazon MQ. Crear un grupo de escalado automático Multi-AZ para EC2 que aloja la aplicación. Migrar la base de datos para ejecutarse en una implementación Multi-AZ de Amazon RDS para PostgreSQL.
- C. Crear un grupo de escalado automático Multi-AZ para EC2 que aloja la cola de RabbitMQ. Crear otro grupo de Auto Scaling Multi-AZ para EC2 que aloja la aplicación. Migrar la base de datos para ejecutarse en una implementación Multi-AZ de Amazon RDS para PostgreSQL.
- D. Crear un grupo de escalado automático Multi-AZ para EC2 que aloja la cola de RabbitMQ. Crear otro grupo de Auto Scaling Multi-AZ para EC2 que aloja la aplicación. Crear un tercer grupo de Auto Scaling Multi-AZ para EC2 que aloja la base de datos PostgreSQL.

**✅ Respuesta: B**

**Justificación:** Amazon MQ con par active/standby gestiona RabbitMQ en dos AZs con failover automático (cero gestión de broker). Multi-AZ ASG para EC2 de la aplicación. RDS Multi-AZ para PostgreSQL elimina overhead de gestión de BD.

**Descarte:**
- A: Igual que B en estructura pero usa instancias de RDS para el ASG adicional en lugar de RDS Multi-AZ. Más overhead operativo.
- C y D: Proponen usar ASGs de EC2 para alojar RabbitMQ. Gestionar RabbitMQ en EC2 tiene overhead operativo significativo.

**🔑 Clave:** Cuando el escenario menciona RabbitMQ o ActiveMQ + menor overhead → Amazon MQ. Para PostgreSQL en EC2 → RDS Multi-AZ.

---

## Pregunta 24 — Notificaciones / CloudTrail + EventBridge

**Escenario:** Una empresa desea recibir una notificación por correo electrónico cuando se agreguen usuarios de IAM a una cuenta de AWS o se eliminen de ella.

**Opciones:**
- A. Activar Amazon Inspector. Crear una regla de Amazon EventBridge que responda a Amazon Inspector findings. Establecer el objetivo como un tema de Amazon Simple Notification Service (Amazon SNS). Establecer la dirección de correo electrónico de la empresa como suscriptora del tema SNS.
- B. Habilitar Amazon GuardDuty. Crear una regla de Amazon EventBridge que responda a GuardDuty findings. Configurar un event pattern de IamUserCreateUnauthorizedAccess. Establecer el objetivo como un tema SNS. Establecer la dirección de correo electrónico de la empresa como suscriptora del tema SNS.
- C. Habilitar Amazon Macie. Crear una regla de Amazon EventBridge que responda a Macie findings. Establecer el objetivo como un tema SNS. Establecer la dirección de correo electrónico de la empresa como suscriptora del tema SNS.
- D. Habilitar los eventos de administración en AWS CloudTrail. Crear una regla de Amazon EventBridge que use un event pattern para las acciones CreateUser y DeleteUser. Establecer el objetivo como un tema SNS. Establecer la dirección de correo electrónico de la empresa como suscriptora del tema SNS.

**✅ Respuesta: D**

**Justificación:** CloudTrail management events registra todas las llamadas a la API de IAM (CreateUser, DeleteUser). EventBridge filtra esos eventos con un event pattern específico. SNS entrega el evento como email. Es el patrón event-driven estándar: CloudTrail → EventBridge → SNS → Email.

**Descarte:**
- A: Amazon Inspector evalúa vulnerabilidades en EC2, ECR y Lambda. No monitorea eventos IAM.
- B: GuardDuty detecta amenazas de seguridad. No genera eventos específicos de CreateUser/DeleteUser de forma confiable.
- C: Amazon Macie descubre datos sensibles en S3. No tiene relación con eventos de gestión de usuarios IAM.

**🔑 Clave:** Notificar cuando ocurre una acción en AWS → CloudTrail (captura la API call) → EventBridge (filtra el evento) → SNS/Lambda (notifica). Pipeline de tres pasos.

---

## Pregunta 25 — ECS / Task IAM Role

**Escenario:** Una empresa ejecuta una aplicación como tarea en un clúster de Amazon Elastic Container Service (Amazon ECS). La aplicación necesita tener acceso de lectura y escritura a un grupo específico de buckets de Amazon S3. Los buckets de S3 están en la misma región de AWS y cuenta de AWS que el clúster de ECS. La empresa desea garantizar que la aplicación tenga acceso a los buckets de S3 siguiendo el principio de mínimo privilegio.

**Opciones:**
- A. Agregar una etiqueta a cada bucket. Crear una política de IAM que incluya una condición StringEquals que coincida con las etiquetas y valores de los buckets.
- B. Crear una política de IAM que incluya el nombre de recurso de Amazon (ARN) completo para cada bucket de S3.
- C. Adjuntar la política de IAM al rol de instancia de la tarea de ECS.
- D. Crear una política de IAM que incluya un nombre de recurso de Amazon (ARN) comodín que coincida con todas las combinaciones de los nombres de buckets de S3.
- E. Adjuntar la política de IAM al rol de tarea de la tarea de ECS.

**✅ Respuesta: B y E**

**Justificación:** B: Especificar los ARNs exactos de cada bucket es la implementación correcta de mínimo privilegio. Solo esos buckets quedan autorizados. E: El task IAM role es el mecanismo correcto para otorgar permisos AWS a un contenedor ECS. Se asigna en la task definition y las credenciales son accesibles dentro del contenedor.

**Descarte:**
- A: ABAC con tags es más complejo. Si se conocen los buckets específicos, ARNs directos son más simples.
- C: El instance profile del EC2 host otorga permisos a todos los contenedores de esa instancia. Viola mínimo privilegio.
- D: Un wildcard puede capturar buckets no intencionados. Viola mínimo privilegio.

**🔑 Clave:** ECS + permisos AWS → siempre Task IAM Role en la task definition, nunca instance profile. El task role aplica solo a ese task específico.

---

## Pregunta 26 — DR / Route 53 Failover

**Escenario:** Un arquitecto de soluciones de una empresa está diseñando una solución multiusuaria de AWS que utiliza AWS Organizations. El arquitecto de soluciones organiza las cuentas de la empresa en unidades organizativas (OUs). Necesita una solución que identifique cualquier cambio en la jerarquía de las OUs. La solución también debe notificar al equipo de operaciones de la empresa de cualquier cambio.

**Opciones:**
- A. Aprovisionar las cuentas de AWS mediante AWS Control Tower. Usar account drift notifications para identificar los cambios en la jerarquía de las OUs.
- B. Aprovisionar las cuentas de AWS mediante AWS Control Tower. Usar AWS Config aggregated rules para identificar los cambios en la jerarquía de las OUs.
- C. Utilizar AWS Service Catalog para crear cuentas en Organizations. Usar un AWS CloudTrail organization trail para identificar los cambios.
- D. Utilizar las plantillas de AWS CloudFormation para crear cuentas en Organizations. Usar la operación de detección de desviaciones en una pila para identificar los cambios en la jerarquía de las OUs.

**✅ Respuesta: A**

**Justificación:** AWS Control Tower gestiona el aprovisionamiento de cuentas en Organizations con guardrails. Account drift notifications detectan automáticamente cuando una cuenta o OU se desvía de la configuración esperada, incluyendo cambios en la jerarquía de OUs. Las notificaciones se envían automáticamente vía SNS.

**Descarte:**
- B: AWS Config con aggregated rules puede detectar cambios de configuración en recursos, pero no es la herramienta nativa para detectar cambios en la jerarquía de OUs específicamente.
- C: Service Catalog es para catálogos de productos de infraestructura aprobados. No detecta cambios en la jerarquía de OUs.
- D: CloudFormation drift detection identifica diferencias entre template y estado actual de recursos de CloudFormation, no cambios en OUs de Organizations.

**🔑 Clave:** Gobernar y detectar cambios en entornos multi-cuenta con OUs → AWS Control Tower. Drift detection es una capacidad nativa clave.

---

## Pregunta 27 — Mensajería / SQS FIFO

**Escenario:** Una empresa tiene un servicio que genera datos de eventos. La empresa quiere usar AWS para procesar los datos de eventos a medida que se reciben. Los datos se escriben en un orden específico que debe mantenerse a lo largo del procesamiento. La empresa quiere implementar una solución que minimice la sobrecarga operacional.

**Opciones:**
- A. Crear una cola de Amazon Simple Queue Service (Amazon SQS) FIFO para contener los mensajes. Configurar una función de AWS Lambda para procesar los mensajes desde la cola.
- B. Crear un tema de Amazon Simple Notification Service (Amazon SNS) para entregar notificaciones que contengan payloads para procesar. Configurar una función de AWS Lambda como suscriptora.
- C. Crear una cola de Amazon Simple Queue Service (Amazon SQS) estándar para contener los mensajes. Configurar una función de AWS Lambda para procesar mensajes desde la cola de forma independiente.
- D. Crear un tema de Amazon Simple Notification Service (Amazon SNS) para entregar notificaciones que contengan payloads para procesar. Configurar una cola de Amazon Simple Queue Service (Amazon SQS) como suscriptora.

**✅ Respuesta: A**

**Justificación:** SQS FIFO garantiza que los mensajes se entregan y procesan exactamente en el orden en que fueron enviados, con exactly-once processing. Lambda como consumidor de SQS FIFO es serverless y minimiza overhead operativo.

**Descarte:**
- B: SNS no garantiza orden de entrega. Los mensajes pueden llegar fuera de secuencia.
- C: SQS Standard no garantiza orden (best-effort ordering). Los mensajes pueden procesarse fuera de orden.
- D: SNS + SQS Standard. Doble violación: SNS sin orden + SQS Standard sin orden.

**🔑 Clave:** Orden garantizado de mensajes → SQS FIFO. SQS Standard = alto throughput sin orden. "Specific order that must be maintained" → siempre FIFO.

---

## Pregunta 28 — Compute / EC2 Placement Groups

**Escenario:** Una empresa está implementando una aplicación que procesa grandes cantidades de datos en paralelo. La empresa planea usar instancias de Amazon EC2 para la carga de trabajo. La arquitectura de red debe ser configurable para evitar que grupos de nodos compartan el mismo hardware subyacente.

**Opciones:**
- A. Ejecutar las instancias de EC2 en un grupo de ubicaciones diferencial (spread placement group).
- B. Agrupar las instancias de EC2 en cuentas separadas.
- C. Utilizar AWS Fargate para proporcionar cómputo a las instancias.
- D. Configurar los grupos de nodos administrados de Active Directory FS.

**✅ Respuesta: A**

**Justificación:** Spread Placement Group distribuye las instancias en racks de hardware distintos. Cada instancia queda en hardware separado con su propia fuente de energía y red. Garantiza que ningún grupo de nodos comparte el mismo hardware subyacente. Máximo 7 instancias por AZ.

**Descarte:**
- B: Las cuentas AWS son límites de facturación y administración, no de hardware físico.
- C: Fargate es cómputo serverless para contenedores. No aplica para EC2 con Placement Groups.
- D: Active Directory FS es para gestión de identidades, no para control de hardware subyacente.

**🔑 Clave:** Spread = máximo aislamiento de hardware. Cluster = máximo rendimiento de red entre instancias. Partition = grandes clusters distribuidos (Hadoop, Kafka).

---

## Pregunta 29 — Base de Datos / RDS Read Replica

**Escenario:** Una empresa aloja una base de datos que se ejecuta en una instancia de Amazon RDS implementada en múltiples zonas de disponibilidad. La empresa periódicamente ejecuta un script en la base de datos para informar nuevas entradas. El script que se ejecuta en la base de datos afecta negativamente al rendimiento de una aplicación crítica. La empresa necesita mejorar el rendimiento de la aplicación con un mínimo de sobrecarga operativa.

**Opciones:**
- A. Agregar funcionalidad al script para identificar la instancia que tiene el menor número de conexiones activas. Configurar el script para que lea desde esa instancia.
- B. Crear una réplica de lectura de la base de datos. Configurar el script para consultar solo la réplica de lectura para informar el total de entradas nuevas.
- C. Indicarle al equipo de desarrollo que exporte manualmente las nuevas entradas para el día en la base de datos al final de cada día.
- D. Usar Amazon ElastiCache para almacenar en caché las consultas comunes que el script ejecuta en la base de datos.

**✅ Respuesta: B**

**Justificación:** La Read Replica es una copia de solo lectura de la BD primaria. El script de reporting se redirige a la Read Replica sin tocar la instancia primaria. La aplicación crítica sigue usando la instancia primaria sin interferencia. Mínimo overhead: se crea la replica, se cambia el endpoint en el script.

**Descarte:**
- A: Lógica de routing personalizada en el script añade complejidad. En Multi-AZ el standby no es accesible para lectura.
- C: Proceso manual, propenso a errores humanos. Máximo overhead operativo.
- D: ElastiCache para un script de reporting que busca nuevas entradas (datos que cambian constantemente) tendría alta tasa de invalidación.

**🔑 Clave:** Separar cargas de trabajo de lectura intensiva (reporting, analytics) de la BD primaria → Read Replica. El standby de Multi-AZ es solo para failover, no para lectura.

---

## Pregunta 30 — Red / NAT Gateway

**Escenario:** Una empresa aloja una instancia de Amazon EC2 en una subred privada de una VPC nueva. La VPC también tiene una subred pública que tiene una ruta predeterminada establecida en una puerta de enlace de Internet. La subred privada no tiene acceso a Internet. La instancia de EC2 debe tener la capacidad de descargar actualizaciones de seguridad mensuales de un proveedor de software externo. Sin embargo, la empresa debe bloquear cualquier conexión que se inicie desde Internet.

**Opciones:**
- A. Configurar la tabla de enrutamiento de la subred privada para usar la puerta de enlace de Internet como ruta predeterminada.
- B. Crear una puerta de enlace NAT en la subred pública. Configurar la tabla de enrutamiento de la subred privada apuntando al NAT Gateway como ruta default.
- C. Crear una instancia de NAT en la subred privada. Configurar la tabla de enrutamiento de la subred privada para usar la instancia NAT como ruta predeterminada.
- D. Crear una instancia de NAT en la subred privada. Configurar la tabla de enrutamiento de la subred privada para usar la puerta de enlace de Internet como ruta predeterminada.

**✅ Respuesta: B**

**Justificación:** NAT Gateway en subnet pública traduce las solicitudes salientes (outbound) de las instancias privadas. La respuesta del vendor regresa pero nadie desde internet puede iniciar una conexión hacia la instancia. Route table de subnet privada: `0.0.0.0/0 → NAT Gateway`.

**Descarte:**
- A: Un Internet Gateway requiere IP pública en la instancia y permitiría conexiones entrantes desde internet.
- C: El NAT debe estar en la subnet pública para tener acceso al Internet Gateway. En subnet privada no puede alcanzar internet.
- D: Mismo problema que C: ubicación incorrecta del NAT Instance.

**🔑 Clave:** NAT Gateway siempre va en la subnet pública. Subnet privada apunta al NAT Gateway. NAT Gateway apunta al IGW. Flujo: EC2 privada → NAT GW (pública) → IGW → Internet.

---

## Pregunta 31 — Base de Datos / DynamoDB TTL

**Escenario:** Una empresa está diseñando una aplicación de trading de acciones que proporciona información comercial a los clientes. La empresa necesita una solución para revocar el acceso a los usuarios inactivos que no han iniciado sesión en la aplicación durante más de 180 días.

**Opciones:**
- A. Usar un proveedor de OpenID Connect (OIDC) para autenticar a los usuarios. Almacenar la información de inicio de sesión de los usuarios en Amazon RDS. Crear un trabajo de AWS Glue DataBrew que compruebe si la fecha del último inicio de sesión de un usuario es más de 180 días en el pasado. Actualizar la función Lambda para recuperar las credenciales del parámetro.
- B. Usar Amazon Cognito para autenticar a los usuarios. Almacenar los datos de tiempo de inicio de sesión de los usuarios en Amazon DynamoDB. Programar un evento de Amazon EventBridge para ejecutar una función de AWS Lambda cada 180 días para desactivar a los usuarios cuya fecha del último inicio de sesión sea más de 180 días en el pasado.
- C. Usar Amazon Cognito para autenticar a los usuarios. Almacenar los metadatos de inicio de sesión del usuario en Amazon DocumentDB (con compatibilidad con MongoDB). Definir un TTL de 180 días para el atributo de inicio de sesión. Configurar un rastreador de AWS Glue para comprobar los usuarios que tienen un TTL de inicio de sesión caducado e invocar una función de AWS Lambda para desactivar a los usuarios.
- D. Usar un proveedor de identidad (IdP) basado en Kerberos para autenticar a los usuarios. Almacenar los metadatos de inicio de sesión del usuario en un bucket cifrado de Amazon S3. Configurar un rastreador de AWS Glue para catalogar los metadatos de inicio de sesión del usuario. Usar Amazon Athena para consultar el catálogo de datos para los últimos detalles de inicio de sesión. Eliminar los datos que tengan más de 180 días.

**✅ Respuesta: C**

**Justificación:** Cognito para autenticación (servicio nativo AWS). DynamoDB almacena metadata del último login con TTL de 180 días. Cuando el TTL expira, DynamoDB marca el ítem. AWS Glue crawler detecta los registros con TTL expirado. Lambda desactiva el usuario en Cognito. El TTL es el mecanismo clave para detección automática de inactividad.

**Descarte:**
- A: Glue DataBrew es para preparación visual de datos, no para este flujo operativo. RDS añade overhead innecesario.
- B: CloudTrail registra eventos de API, no eventos de login de usuarios Cognito directamente. Proceso diario tiene mayor latencia.
- D: Kerberos es para autenticación on-premises/enterprise. Athena sobre S3 es innecesariamente complejo.

**🔑 Clave:** DynamoDB TTL es el mecanismo nativo para expiración automática de datos basada en tiempo. "Expirar o revocar algo después de N días" → TTL de DynamoDB.

---

## Pregunta 32 — Base de Datos / RDS Proxy

**Escenario:** Una empresa tiene una aplicación de procesamiento de transacciones respaldada por una base de datos MySQL de Amazon RDS. Cuando aumenta la carga en la aplicación, se genera un gran número de conexiones a la base de datos, lo que provoca latencia en las transacciones de la base de datos. Un arquitecto de soluciones determina que la causa de la latencia es el deficiente manejo de conexiones por parte de la aplicación. El arquitecto de soluciones no puede modificar el código de la aplicación. La empresa necesita mejorar la administración de las conexiones para mejorar el rendimiento de la base de datos durante la carga.

**Opciones:**
- A. Actualizar la instancia de base de datos a un tipo de instancia mayor para manejar un gran número de conexiones de base de datos.
- B. Configurar el escalado automático del almacenamiento de Amazon RDS para incrementar dinámicamente las IOPS aprovisionadas.
- C. Utilizar el proxy de Amazon RDS para agrupar y compartir conexiones de base de datos.
- D. Convertir la instancia de base de datos a un despliegue Multi-AZ.

**✅ Respuesta: C**

**Justificación:** RDS Proxy mantiene un pool de conexiones hacia RDS. En lugar de que cada request abra y cierre una conexión directamente, el Proxy reutiliza conexiones existentes. Es transparente para la aplicación: solo se cambia el endpoint de conexión al del RDS Proxy sin modificar el código.

**Descarte:**
- A: Escalar verticalmente puede aliviar síntomas pero no resuelve la causa raíz: el manejo de conexiones.
- B: El problema es de conexiones, no de throughput de almacenamiento. IOPS no tiene relación con la latencia causada por conexiones.
- D: Multi-AZ provee HA y failover automático pero no mejora el rendimiento de conexiones en condiciones normales.

**🔑 Clave:** RDS Proxy = connection pooling. Siempre que el escenario mencione muchas conexiones, Lambda accediendo a RDS, o "too many connections" → RDS Proxy.

---

## Pregunta 33 — Almacenamiento / S3 Lifecycle

**Escenario:** Una empresa almacena una gran cantidad de archivos de imágenes en un bucket de Amazon S3. Las imágenes son solo accesibles durante 180 días. Sin embargo, la empresa debe poder acceder a las imágenes de forma inmediata cuando sea necesario. La empresa utilizará el almacenamiento de S3 Standard para los primeros 180 días. La empresa necesita configurar las reglas de S3 Lifecycle para manejar las etapas de ciclo de vida restantes de los archivos.

**Opciones:**
- A. Transición a S3 One Zone-Infrequent Access (S3 One Zone-IA) después de 180 días. Transición a S3 Glacier Instant Retrieval después de 360 días.
- B. Transición a S3 One Zone-Infrequent Access (S3 One Zone-IA) después de 180 días. Transición a S3 Glacier Flexible Retrieval después de 360 días.
- C. Transición a S3 Standard-Infrequent Access (S3 Standard-IA) después de 180 días. Transición a S3 Glacier Instant Retrieval después de 360 días.
- D. Transición a S3 Standard-Infrequent Access (S3 Standard-IA) después de 180 días. Transición a S3 Glacier Flexible Retrieval después de 360 días.

**✅ Respuesta: C**

**Justificación:** Standard-IA (días 180-360): acceso infrecuente pero disponibilidad inmediata, multi-AZ (cumple el requisito de HA/redundancia). Glacier Instant Retrieval (día 360+): recuperación en milisegundos (cumple "acceso inmediato cuando sea necesario"), multi-AZ, costo menor.

**Descarte:**
- A y B: One Zone-IA almacena en una sola AZ. El escenario exige alta disponibilidad y redundancia durante todo el ciclo de vida.
- D: Glacier Flexible Retrieval tiene tiempos de recuperación de minutos a horas. El escenario requiere acceso inmediato.

**🔑 Clave:** Glacier Instant = milisegundos (acceso inmediato). Glacier Flexible = minutos a horas. Glacier Deep Archive = 12-48 horas.

---

## Pregunta 34 — Seguridad / Cifrado RDS

**Escenario:** Una empresa de servicios financieros lanza una nueva aplicación en AWS para manejar datos financieros sensibles. La aplicación se despliega en instancias EC2. La base de datos es Amazon RDS for MySQL. Las políticas de seguridad exigen que los datos se cifren en reposo y en tránsito. Menor overhead operativo.

**Opciones:**
- A. Configurar el cifrado en reposo para Amazon RDS for MySQL usando claves KMS managed keys. Configurar AWS Certificate Manager (ACM) SSL/TLS certificates para el cifrado en tránsito.
- B. Configurar el cifrado en reposo para Amazon RDS for MySQL usando claves KMS managed keys. Configurar AWS Managed Microsoft AD para el cifrado en tránsito.
- C. Implementar el cifrado de datos a nivel de aplicación de terceros antes de almacenar datos en Amazon RDS for MySQL. Configurar AWS Certificate Manager (ACM) SSL/TLS certificates para el cifrado en tránsito.
- D. Configurar el cifrado en reposo para Amazon RDS for MySQL usando claves KMS managed keys. Configurar una VPN connection para habilitar conectividad privada para cifrar los datos en tránsito.

**✅ Respuesta: A**

**Justificación:** KMS managed keys + RDS encryption habilita cifrado en reposo con un checkbox (mínimo overhead). ACM SSL/TLS para conexiones EC2 → RDS cifra el tránsito. RDS for MySQL soporta SSL/TLS nativo. ACM gestiona certificados automáticamente.

**Descarte:**
- B: Microsoft AD es un servicio de directorio, no un mecanismo de cifrado de tránsito.
- C: Cifrado a nivel de aplicación de terceros añade complejidad innecesaria cuando RDS tiene cifrado nativo.
- D: VPN es para conectividad de red entre redes diferentes, no para cifrar tránsito entre EC2 y RDS dentro de la misma VPC.

**🔑 Clave:** Cifrado RDS completo → KMS en reposo + SSL/TLS en tránsito. Ambos son nativos y de mínimo overhead.

---

## Pregunta 35 — Kinesis / Enhanced Fan-Out

**Escenario:** Una empresa tiene múltiples consumidores que comparten datos de Amazon Kinesis Data Streams. La empresa quiere compartir el rendimiento entre todos los consumidores. La empresa debe prevenir que un consumidor monopolice el rendimiento del stream. Cada consumidor debe tener su propio rendimiento de lectura y no debe competir con otros consumidores.

**Opciones:**
- A. Configurar enhanced fan-out para permitir que cada consumidor reciba su propio throughput dedicado.
- B. Aumentar la cantidad de particiones en el stream.
- C. Configurar la administración de los shards dentro de los consumidores.
- D. Enviar los datos a una cola de Amazon Simple Queue Service (Amazon SQS) antes de distribuir los datos a los consumidores.

**✅ Respuesta: A**

**Justificación:** Enhanced Fan-Out asigna 2 MB/s por shard de forma dedicada a cada consumidor registrado, independientemente de cuántos otros consumidores existan. Usa HTTP/2 push (no polling), reduciendo latencia a ~70ms. Los consumidores no compiten entre sí.

**Descarte:**
- B: Más shards aumenta el throughput total pero si múltiples consumidores leen del mismo shard, siguen compitiendo por los 2 MB/s de ese shard.
- C: No existe un mecanismo de "shard management" dentro del consumidor que resuelva la competencia de throughput.
- D: SQS es una cola donde cada mensaje es consumido por un solo consumidor. No sirve para que múltiples consumidores reciban los mismos datos.

**🔑 Clave:** Múltiples consumidores de Kinesis sin compartir throughput → Enhanced Fan-Out.

---

## Pregunta 36 — Mensajería / SQS Visibility Timeout

**Escenario:** Una empresa aloja una aplicación en varias instancias de Amazon EC2. La aplicación procesa mensajes de una cola de Amazon SQS, escribe en una tabla de Amazon RDS y elimina el mensaje de la cola. La cola de SQS no tiene ningún mensaje duplicado. En ocasiones, se encuentran registros duplicados en la tabla de RDS.

**Opciones:**
- A. Usar la llamada a la API CreateQueue para crear una nueva cola.
- B. Usar la llamada a la API AddPermission para agregar los permisos adecuados.
- C. Usar la llamada a la API ReceiveMessage para establecer un tiempo de espera apropiado.
- D. Usar la llamada a la API ChangeMessageVisibility para aumentar el timeout de visibilidad.

**✅ Respuesta: D**

**Justificación:** El visibility timeout está configurado muy corto. El procesamiento tarda más que el timeout, el mensaje vuelve a ser visible y otra instancia lo procesa generando el duplicado. `ChangeMessageVisibility` extiende el timeout mientras se procesa, dando más tiempo para completar y eliminar el mensaje.

**Descarte:**
- A: CreateQueue crea una nueva cola. No resuelve el procesamiento duplicado.
- B: AddPermission gestiona permisos de acceso. No relacionado con el problema.
- C: ReceiveMessage con wait time configura long polling (reduce llamadas vacías). No afecta el visibility timeout.

**🔑 Clave:** Mensajes duplicados en SQS cuando la cola no tiene duplicados = visibility timeout muy corto. Solución: aumentar el visibility timeout.

---

## Pregunta 37 — Observabilidad / X-Ray + Container Insights

**Escenario:** Una empresa está construyendo una aplicación basada en microservicios que se implementará en Amazon Elastic Kubernetes Service (Amazon EKS). Los microservicios interactuarán entre sí. La empresa quiere asegurarse de que la aplicación sea observable para identificar problemas de rendimiento en el futuro.

**Opciones:**
- A. Configurar la aplicación para utilizar Amazon ElastiCache para reducir el número de solicitudes que se envían a los microservicios.
- B. Configurar la información de los contenedores de Amazon CloudWatch Container Insights para recopilar métricas del clúster de EKS. Configurar AWS X-Ray para trazar las solicitudes entre los microservicios.
- C. Configurar AWS CloudTrail para revisar las llamadas a la API. Construir un panel de Amazon QuickSight para observar las interacciones del microservicio.
- D. Utilizar AWS Trusted Advisor para entender el rendimiento de la aplicación.

**✅ Respuesta: B**

**Justificación:** CloudWatch Container Insights recopila métricas a nivel de cluster, nodo, pod y container en EKS (CPU, memoria, red). AWS X-Ray provee distributed tracing — rastrea cada request a través de todos los microservicios, genera service map visual y identifica qué microservicio causa degradación.

**Descarte:**
- A: ElastiCache es caché de datos, no una herramienta de observabilidad.
- C: CloudTrail registra API calls de AWS (plano de control), no el tráfico entre microservicios de la aplicación.
- D: Trusted Advisor revisa configuraciones AWS contra best practices. No monitorea rendimiento de aplicaciones en tiempo real.

**🔑 Clave:** Observabilidad en EKS/ECS/microservicios → Container Insights (métricas) + X-Ray (distributed tracing).

---

## Pregunta 38 — Seguridad / IAM Access Analyzer

**Escenario:** Una empresa ejecuta todas sus aplicaciones comerciales en la nube de AWS. La empresa usa AWS Organizations para administrar múltiples cuentas de AWS. Un arquitecto de soluciones debe revisar todos los permisos que se conceden a los usuarios de IAM para determinar qué usuarios de IAM tienen más permisos de los necesarios. La solución debe cumplir estos requisitos con la MENOR sobrecarga administrativa.

**Opciones:**
- A. Utilizar el analizador de acceso de la red de AWS para revisar todos los permisos de acceso en las cuentas de AWS de la empresa.
- B. Crear una alarma de AWS CloudWatch que se active cuando un usuario de IAM crea o modifica recursos en una cuenta de AWS.
- C. Utilizar AWS Identity and Access Management (IAM) Access Analyzer para revisar todos los permisos de los recursos y cuentas de la empresa.
- D. Utilizar Amazon Inspector para encontrar vulnerabilidades en las políticas de IAM existentes.

**✅ Respuesta: C**

**Justificación:** IAM Access Analyzer analiza políticas IAM y de recursos para identificar accesos excesivos o no intencionados. Se puede configurar a nivel de AWS Organizations para analizar todas las cuentas miembro desde un punto centralizado. Genera findings con accesos problemáticos. Mínimo overhead administrativo.

**Descarte:**
- A: Network Access Analyzer analiza el acceso de red (conectividad entre recursos VPC). No tiene relación con permisos IAM.
- B: CloudWatch alarms son reactivas. No pueden determinar qué usuarios tienen más permisos de los necesarios.
- D: Inspector evalúa vulnerabilidades en EC2 y contenedores ECR. No analiza políticas IAM.

**🔑 Clave:** Revisar y auditar permisos IAM en múltiples cuentas → IAM Access Analyzer.

---

## Pregunta 39 — Alta Disponibilidad / Session State

**Escenario:** Una empresa ejecuta una aplicación web en instancias de Amazon EC2 en un grupo de escalado automático detrás de un balanceador de carga de aplicación (ALB). La empresa ha habilitado sticky sessions para el ALB. Cada servidor web actualmente aloja el estado de sesión del usuario. La empresa quiere garantizar alta disponibilidad y evitar la pérdida del estado de sesión en caso de un fallo de un servidor web.

**Opciones:**
- A. Usar una instancia de Amazon ElastiCache (Memcached) para almacenar los datos de sesión. Actualizar la aplicación para usar la instancia de ElastiCache para almacenar el estado de la sesión.
- B. Usar un caché de Amazon ElastiCache (Redis OSS) para almacenar el estado de la sesión. Actualizar la aplicación para usar la instancia de ElastiCache para almacenar el estado de la sesión.
- C. Usar un volumen en caché de Volume Gateway de AWS Storage Gateway para almacenar datos de sesión.
- D. Usar Amazon DynamoDB con lecturas eventualmente consistentes predeterminadas para almacenar los datos de sesión. Configurar la aplicación con un TTL de 24 horas para los registros de sesión.

**✅ Respuesta: B**

**Justificación:** ElastiCache Redis es el estándar para session store en AWS. In-memory de baja latencia, altamente disponible con Multi-AZ. Si una instancia EC2 falla, el session state persiste en Redis. Con esto, se pueden eliminar las sticky sessions del ALB — cualquier instancia puede atender cualquier request.

**Descarte:**
- A: Memcached no soporta replicación ni persistencia. Si el nodo falla, los datos de sesión se pierden. Para session state con HA, Redis es siempre la elección correcta.
- C: Storage Gateway Volume Gateway es para almacenamiento de bloque híbrido. No es un session store.
- D: DynamoDB tiene mayor latencia que Redis (milisegundos vs microsegundos) y mayor costo para datos transientes de sesión.

**🔑 Clave:** Session state externalizado en AWS → ElastiCache Redis. Redis > Memcached cuando se necesita HA, replicación o persistencia.

---

## Pregunta 40 — S3 / Intelligent-Tiering

**Escenario:** Una empresa está desarrollando una aplicación web que permitirá almacenar una gran cantidad de imágenes en Amazon S3. Los usuarios accederán a las imágenes durante lapsos de tiempo variables. La empresa busca lo siguiente:
- Retener todas las imágenes
- No incurrir en costos de recuperación
- Tener almacenamiento de administración mínima
- Tener acceso a las imágenes sin impacto en el tiempo de recuperación

**Opciones:**
- A. Implementar S3 Intelligent-Tiering.
- B. Implementar el análisis de clases de almacenamiento de S3.
- C. Implementar una política de ciclo de vida de S3 para mover datos a S3 Standard-Infrequent Access (S3 Standard-IA).
- D. Implementar una política de ciclo de vida de S3 para mover datos a S3 One Zone-Infrequent Access (S3 One Zone-IA).

**✅ Respuesta: A**

**Justificación:** S3 Intelligent-Tiering mueve objetos automáticamente entre tiers según patrones de acceso (mínima administración). No cobra por recuperación. Los objetos en Frequent Access tier tienen la misma latencia que S3 Standard. Diseñado exactamente para patrones de acceso variables o desconocidos.

**Descarte:**
- B: Storage Class Analysis es una herramienta de análisis que recomienda cuándo mover objetos. No es una solución de almacenamiento en sí misma y requiere configurar Lifecycle policies manualmente.
- C: S3-IA cobra un costo de recuperación por GB. Viola el requisito de "no incurrir en costos de recuperación".
- D: Same problema de costo de recuperación que C. Además, One Zone-IA tiene menor resiliencia.

**🔑 Clave:** Acceso impredecible o variable + sin costo de recuperación + mínimo overhead → S3 Intelligent-Tiering.

---

*Última actualización: Mayo 2026 | Total: 40 preguntas*
