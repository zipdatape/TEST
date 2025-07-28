# DIAGRAMA DE MIGRACIÓN A LA NUBE - OROCOM

## Arquitectura de Migración Propuesta

```mermaid
graph TB
    subgraph "Infraestructura Actual"
        NAS[NAS Storage<br/>27TB]
        AD_LOCAL[Windows Server AD<br/>Local]
        SPRING_LOCAL[Spring App<br/>Sistema Contable]
        FORTIGATE[FortiGate<br/>Firewall]
        USERS[Usuarios de Dominio]
    end
    
    subgraph "Proceso de Migración"
        MIGRATION[8 Semanas<br/>Migración Gradual]
    end
    
    subgraph "Infraestructura en la Nube"
        subgraph "VPC Cloud (10.0.0.0/16)"
            subgraph "Public Subnet (10.0.1.0/24)"
                IGW[Internet Gateway]
                NAT[NAT Gateway]
                BASTION[Bastion Host]
            end
            
            subgraph "Private Subnet (10.0.2.0/24)"
                AD_CLOUD[Windows Server AD<br/>Domain Controller]
                SPRING_CLOUD[Spring Application<br/>Sistema Contable]
            end
            
            subgraph "Storage Layer"
                S3[Cloud Storage<br/>27TB]
                EFS[File Storage<br/>10TB]
                RDS[Database<br/>SQL Server]
            end
            
            subgraph "Security & Monitoring"
                SG[Security Groups]
                MON[Cloud Monitoring]
                BACKUP[Automated Backups]
            end
        end
    end
    
    NAS --> MIGRATION
    AD_LOCAL --> MIGRATION
    SPRING_LOCAL --> MIGRATION
    FORTIGATE --> IGW
    MIGRATION --> AD_CLOUD
    MIGRATION --> SPRING_CLOUD
    MIGRATION --> S3
    MIGRATION --> EFS
    MIGRATION --> RDS
    
    IGW --> NAT
    NAT --> AD_CLOUD
    NAT --> SPRING_CLOUD
    
    AD_CLOUD --> S3
    AD_CLOUD --> EFS
    SPRING_CLOUD --> RDS
    SPRING_CLOUD --> EFS
    
    SG --> AD_CLOUD
    SG --> SPRING_CLOUD
    SG --> BASTION
    
    MON --> AD_CLOUD
    MON --> SPRING_CLOUD
    BACKUP --> S3
    BACKUP --> EFS
    BACKUP --> RDS
    
    AD_CLOUD -.-> USERS
    SPRING_CLOUD -.-> USERS
    
    classDef current fill:#e74c3c,stroke:#c0392b,color:#fff
    classDef cloud fill:#3498db,stroke:#2980b9,color:#fff
    classDef migration fill:#f39c12,stroke:#e67e22,color:#fff
    classDef storage fill:#2ecc71,stroke:#27ae60,color:#fff
    classDef security fill:#9b59b6,stroke:#8e44ad,color:#fff
    
    class NAS,AD_LOCAL,SPRING_LOCAL,FORTIGATE current
    class AD_CLOUD,SPRING_CLOUD,IGW,NAT,BASTION cloud
    class MIGRATION migration
    class S3,EFS,RDS storage
    class SG,MON,BACKUP security
```

## Comparación de Costos

```mermaid
graph LR
    subgraph "Costos Mensuales"
        AWS[AWS<br/>$1,448.50]
        GCP[Google Cloud<br/>$1,253.50]
        AZURE[Microsoft Azure<br/>$1,400.00]
        CURRENT[Infraestructura Actual<br/>$2,500.00]
    end
    
    subgraph "Ahorro Anual"
        SAVINGS_GCP[Ahorro con GCP<br/>$14,958]
        SAVINGS_AWS[Ahorro con AWS<br/>$12,618]
        SAVINGS_AZURE[Ahorro con Azure<br/>$13,200]
    end
    
    CURRENT --> SAVINGS_GCP
    CURRENT --> SAVINGS_AWS
    CURRENT --> SAVINGS_AZURE
    
    classDef gcp fill:#4285f4,stroke:#2980b9,color:#fff
    classDef aws fill:#ff9900,stroke:#e67e22,color:#fff
    classDef azure fill:#00a1f1,stroke:#2980b9,color:#fff
    classDef current fill:#e74c3c,stroke:#c0392b,color:#fff
    
    class GCP,SAVINGS_GCP gcp
    class AWS,SAVINGS_AWS aws
    class AZURE,SAVINGS_AZURE azure
    class CURRENT current
```

## Plan de Migración por Fases

```mermaid
gantt
    title Plan de Migración a la Nube - OROCOM
    dateFormat  YYYY-MM-DD
    section Fase 1: Preparación
    Configuración AWS/GCP Account    :done, config, 2025-08-01, 14d
    Diseño de Red                    :done, network, 2025-08-01, 14d
    Configuración IAM                :done, iam, 2025-08-01, 14d
    
    section Fase 2: Infraestructura Base
    Windows Server AD                :active, ad, 2025-08-15, 14d
    Cloud Storage                    :active, storage, 2025-08-15, 14d
    Database Setup                   :active, db, 2025-08-15, 14d
    
    section Fase 3: Aplicaciones
    Spring Application               :spring, 2025-08-29, 14d
    Migración de Datos               :data, 2025-08-29, 14d
    Configuración DNS                :dns, 2025-08-29, 14d
    
    section Fase 4: Pruebas y Corte
    Testing                          :test, 2025-09-12, 7d
    Validación                       :validation, 2025-09-12, 7d
    Corte de Servicios               :cutover, 2025-09-19, 7d
```

## Flujo de Datos en la Arquitectura Final

```mermaid
flowchart TD
    USERS[Usuarios] --> FORTIGATE[FortiGate Firewall]
    FORTIGATE --> IGW[Internet Gateway]
    IGW --> NAT[NAT Gateway]
    
    NAT --> AD[Windows Server AD]
    NAT --> SPRING[Spring Application]
    
    AD --> S3[Cloud Storage 27TB]
    AD --> EFS[File Storage 10TB]
    SPRING --> RDS[Database SQL Server]
    SPRING --> EFS
    
    S3 --> BACKUP[Automated Backups]
    EFS --> BACKUP
    RDS --> BACKUP
    
    AD --> MON[Cloud Monitoring]
    SPRING --> MON
    
    MON --> ALERTS[Alertas y Notificaciones]
    BACKUP --> DR[Disaster Recovery]
    
    classDef users fill:#34495e,stroke:#2c3e50,color:#fff
    classDef network fill:#3498db,stroke:#2980b9,color:#fff
    classDef apps fill:#2ecc71,stroke:#27ae60,color:#fff
    classDef storage fill:#f39c12,stroke:#e67e22,color:#fff
    classDef monitoring fill:#9b59b6,stroke:#8e44ad,color:#fff
    
    class USERS users
    class FORTIGATE,IGW,NAT network
    class AD,SPRING apps
    class S3,EFS,RDS storage
    class MON,BACKUP,ALERTS,DR monitoring
```

## Recomendación Final

### 🏆 **Google Cloud Platform (GCP)**

**Ventajas principales:**
- ✅ **Costo más bajo**: $1,253.50/mes
- ✅ **Ahorro anual**: $14,958 USD
- ✅ **Mejor rendimiento de red**
- ✅ **Herramientas de análisis avanzadas**
- ✅ **Escalabilidad automática**

**Servicios GCP recomendados:**
- **Compute Engine**: Windows Server AD y Spring Application
- **Cloud Storage**: 27TB de almacenamiento
- **Filestore**: 10TB para archivos compartidos
- **Cloud SQL**: Base de datos SQL Server
- **Cloud Monitoring**: Monitoreo y alertas
- **Cloud Backup**: Backups automáticos

---

*Diagrama generado el: 28 de Julio 2025*
*Basado en inventario de servidores OROCOM* 
