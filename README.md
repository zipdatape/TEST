# ARQUITECTURA PROPUESTA - MIGRACIÓN A LA NUBE OROCOM

## 🏗️ ARQUITECTURA DETALLADA RECOMENDADA

### Diagrama Principal de Arquitectura

```mermaid
graph TB
    subgraph "EXTERNAL"
        WEB[🌐 Internet]
    end
    
    subgraph "ON-PREMISES"
        FORTIGATE[🛡️ FortiGate Firewall<br/>IP: 192.168.1.1]
        USERS[👥 Usuarios de Dominio<br/>50+ usuarios]
        NAS[💾 NAS Storage<br/>27TB - Migración]
    end
    
    subgraph "GOOGLE CLOUD PLATFORM"
        subgraph "VPC: oro-com-vpc (10.0.0.0/16)"
            subgraph "Public Subnet: 10.0.1.0/24"
                IGW[🌐 Internet Gateway]
                NAT[🔄 Cloud NAT<br/>IP: 10.0.1.1]
                BASTION[🔑 Bastion Host<br/>e2-micro<br/>10.0.1.10]
            end
            
            subgraph "Private Subnet: 10.0.2.0/24"
                AD[🏢 Windows Server AD<br/>e2-standard-2<br/>2vCPU, 8GB RAM<br/>10.0.2.10<br/>Domain Controller]
                SPRING[☕ Spring Application<br/>e2-standard-2<br/>2vCPU, 8GB RAM<br/>10.0.2.20<br/>Sistema Contable]
            end
            
            subgraph "Storage Layer"
                CS[📦 Cloud Storage<br/>27TB<br/>Lifecycle Policies<br/>Backup Automático]
                FS[📁 Filestore<br/>10TB<br/>Shared Storage<br/>NFS/SMB]
                SQL[🗄️ Cloud SQL<br/>SQL Server<br/>db-standard-2<br/>10.0.2.30]
            end
            
            subgraph "Security & Monitoring"
                FW[🔒 Firewall Rules<br/>Security Groups]
                MON[📊 Cloud Monitoring<br/>Alertas y Métricas]
                BACKUP[💾 Cloud Backup<br/>Automated Backups<br/>Cross-Region]
                IAM[👤 IAM & Access Control]
            end
        end
    end
    
    %% Connections
    WEB --> FORTIGATE
    FORTIGATE --> IGW
    IGW --> NAT
    IGW --> BASTION
    
    NAT --> AD
    NAT --> SPRING
    
    AD --> CS
    AD --> FS
    SPRING --> SQL
    SPRING --> FS
    
    FW --> AD
    FW --> SPRING
    FW --> BASTION
    
    MON --> AD
    MON --> SPRING
    MON --> SQL
    
    BACKUP --> CS
    BACKUP --> FS
    BACKUP --> SQL
    
    AD -.-> USERS
    SPRING -.-> USERS
    
    NAS -.-> CS
    
    %% Styling
    classDef external fill:#95a5a6,stroke:#7f8c8d,color:#fff
    classDef onprem fill:#e74c3c,stroke:#c0392b,color:#fff
    classDef cloud fill:#3498db,stroke:#2980b9,color:#fff
    classDef storage fill:#f39c12,stroke:#e67e22,color:#fff
    classDef security fill:#9b59b6,stroke:#8e44ad,color:#fff
    classDef apps fill:#2ecc71,stroke:#27ae60,color:#fff
    
    class WEB external
    class FORTIGATE,USERS,NAS onprem
    class IGW,NAT,BASTION cloud
    class CS,FS,SQL storage
    class FW,MON,BACKUP,IAM security
    class AD,SPRING apps
```

## 🔄 FLUJO DE DATOS Y CONECTIVIDAD

```mermaid
flowchart TD
    subgraph "Flujo de Usuarios"
        USERS[👥 Usuarios] --> VPN[🔗 VPN Connection]
        VPN --> FORTIGATE[🛡️ FortiGate]
        FORTIGATE --> CLOUD[☁️ Cloud VPC]
    end
    
    subgraph "Flujo de Aplicaciones"
        CLOUD --> AD[🏢 Active Directory]
        CLOUD --> SPRING[☕ Spring App]
        
        AD --> STORAGE[💾 Storage Services]
        SPRING --> STORAGE
        
        STORAGE --> BACKUP[💾 Backup Services]
        STORAGE --> MONITORING[📊 Monitoring]
    end
    
    subgraph "Flujo de Seguridad"
        SECURITY[🔒 Security Layer] --> AD
        SECURITY --> SPRING
        SECURITY --> STORAGE
    end
    
    classDef users fill:#34495e,stroke:#2c3e50,color:#fff
    classDef network fill:#3498db,stroke:#2980b9,color:#fff
    classDef apps fill:#2ecc71,stroke:#27ae60,color:#fff
    classDef storage fill:#f39c12,stroke:#e67e22,color:#fff
    classDef security fill:#9b59b6,stroke:#8e44ad,color:#fff
    
    class USERS,VPN users
    class FORTIGATE,CLOUD network
    class AD,SPRING apps
    class STORAGE,BACKUP storage
    class SECURITY,MONITORING security
```

## 📊 ESPECIFICACIONES TÉCNICAS DETALLADAS

### 🖥️ **Servidores Virtuales**

| Componente | Especificación | Costo Mensual | Propósito |
|------------|----------------|---------------|-----------|
| **Windows Server AD** | e2-standard-2<br/>2vCPU, 8GB RAM<br/>Windows Server 2019 | $65.00 | Domain Controller, Gestión de usuarios |
| **Spring Application** | e2-standard-2<br/>2vCPU, 8GB RAM<br/>Ubuntu 20.04 LTS | $65.00 | Sistema contable Spring |
| **Bastion Host** | e2-micro<br/>2vCPU, 1GB RAM<br/>Ubuntu 20.04 LTS | $6.50 | Acceso seguro a servidores privados |

### 💾 **Almacenamiento**

| Servicio | Capacidad | Tipo | Costo Mensual | Propósito |
|----------|-----------|------|---------------|-----------|
| **Cloud Storage** | 27TB | Standard | $540.00 | Almacenamiento principal, reemplaza NAS |
| **Filestore** | 10TB | Basic | $250.00 | Archivos compartidos, NFS/SMB |
| **Cloud SQL** | 100GB | SQL Server | $140.00 | Base de datos del sistema |

### 🔒 **Seguridad y Redes**

| Componente | Especificación | Propósito |
|------------|----------------|-----------|
| **VPC** | 10.0.0.0/16 | Red virtual privada |
| **Public Subnet** | 10.0.1.0/24 | Servicios públicos (NAT, Bastion) |
| **Private Subnet** | 10.0.2.0/24 | Servidores de aplicaciones |
| **Firewall Rules** | Reglas específicas por servicio | Control de acceso |
| **IAM** | Roles y permisos granulares | Gestión de identidades |

## 🚀 PLAN DE IMPLEMENTACIÓN

```mermaid
gantt
    title Plan de Implementación - Arquitectura Cloud OROCOM
    dateFormat  YYYY-MM-DD
    section Fase 1: Preparación
    Configuración GCP Project     :done, gcp, 2025-08-01, 3d
    Configuración VPC y Redes     :done, vpc, 2025-08-04, 3d
    Configuración IAM             :done, iam, 2025-08-07, 3d
    Configuración Firewall        :done, fw, 2025-08-10, 3d
    
    section Fase 2: Infraestructura Base
    Despliegue Windows Server AD  :active, ad, 2025-08-13, 5d
    Configuración Active Directory :active, adconfig, 2025-08-18, 3d
    Configuración Cloud Storage   :active, storage, 2025-08-21, 3d
    Configuración Filestore       :active, filestore, 2025-08-24, 2d
    
    section Fase 3: Aplicaciones
    Despliegue Spring Application :spring, 2025-08-26, 5d
    Configuración Cloud SQL       :sql, 2025-08-31, 3d
    Migración de Datos            :migration, 2025-09-03, 7d
    
    section Fase 4: Integración
    Configuración VPN             :vpn, 2025-09-10, 3d
    Configuración DNS             :dns, 2025-09-13, 2d
    Pruebas de Conectividad       :testing, 2025-09-15, 3d
    
    section Fase 5: Corte
    Validación Final              :validation, 2025-09-18, 2d
    Corte de Servicios            :cutover, 2025-09-20, 1d
    Monitoreo Post-Migración      :monitoring, 2025-09-21, 7d
```

## 💰 ANÁLISIS DE COSTOS DETALLADO

### 📈 **Costos Mensuales por Categoría**

```mermaid
pie title Distribución de Costos Mensuales - GCP
    "Compute Engine" : 136.50
    "Cloud Storage" : 540.00
    "Filestore" : 250.00
    "Cloud SQL" : 140.00
    "Networking" : 40.00
    "Monitoring & Backup" : 57.00
    "Data Transfer" : 90.00
```

### 💵 **Comparación de Costos Anuales**

| Concepto | Infraestructura Actual | GCP | Ahorro |
|----------|------------------------|-----|--------|
| **Servidores** | $15,000 | $1,638 | $13,362 |
| **Almacenamiento** | $8,000 | $9,480 | -$1,480 |
| **Mantenimiento** | $12,000 | $0 | $12,000 |
| **Energía** | $3,000 | $0 | $3,000 |
| **Licencias** | $6,000 | $1,680 | $4,320 |
| **TOTAL ANUAL** | **$44,000** | **$12,798** | **$31,202** |

## 🔧 CONFIGURACIÓN TÉCNICA

### 🌐 **Configuración de Red**

```yaml
VPC Configuration:
  Name: oro-com-vpc
  CIDR: 10.0.0.0/16
  Subnets:
    - Name: public-subnet
      CIDR: 10.0.1.0/24
      Region: us-central1
    - Name: private-subnet
      CIDR: 10.0.2.0/24
      Region: us-central1
  
  Firewall Rules:
    - Name: allow-ssh
      Ports: 22
      Source: 0.0.0.0/0
    - Name: allow-rdp
      Ports: 3389
      Source: 10.0.1.0/24
    - Name: allow-spring-app
      Ports: 8080, 8443
      Source: 10.0.2.0/24
```

### 🔐 **Configuración de Seguridad**

```yaml
IAM Roles:
  - Name: oro-com-admin
    Permissions: Owner
    Members: [admin@orocom.com]
  
  - Name: oro-com-developer
    Permissions: Editor
    Members: [dev@orocom.com]
  
  - Name: oro-com-viewer
    Permissions: Viewer
    Members: [support@orocom.com]

Security Policies:
  - Enforce MFA for all users
  - Regular security audits
  - Automated vulnerability scanning
  - Encrypted data at rest and in transit
```

## 📋 CHECKLIST DE IMPLEMENTACIÓN

### ✅ **Preparación (Semana 1-2)**
- [ ] Crear cuenta GCP con billing habilitado
- [ ] Configurar VPC y subnets
- [ ] Configurar IAM roles y políticas
- [ ] Configurar firewall rules
- [ ] Configurar Cloud NAT

### ✅ **Infraestructura Base (Semana 3-4)**
- [ ] Desplegar Windows Server AD
- [ ] Configurar Active Directory
- [ ] Configurar Cloud Storage buckets
- [ ] Configurar Filestore
- [ ] Configurar Cloud SQL

### ✅ **Aplicaciones (Semana 5-6)**
- [ ] Desplegar Spring Application
- [ ] Migrar aplicación contable
- [ ] Configurar conexiones a base de datos
- [ ] Migrar datos del NAS
- [ ] Configurar DNS

### ✅ **Integración (Semana 7-8)**
- [ ] Configurar VPN con FortiGate
- [ ] Pruebas de conectividad
- [ ] Validación de aplicaciones
- [ ] Corte de servicios locales
- [ ] Monitoreo post-migración

## 🎯 BENEFICIOS ESPERADOS

### 💡 **Beneficios Inmediatos**
- ✅ **Ahorro de costos**: $31,202 anuales
- ✅ **Eliminación de mantenimiento de hardware**
- ✅ **Escalabilidad automática**
- ✅ **Backups automáticos**

### 🚀 **Beneficios a Largo Plazo**
- ✅ **Disponibilidad 99.9%**
- ✅ **Recuperación ante desastres**
- ✅ **Acceso remoto global**
- ✅ **Actualizaciones automáticas**

---

*Arquitectura propuesta generada el: 28 de Julio 2025*
*Basada en inventario de servidores OROCOM*
*Recomendación: Google Cloud Platform* 
