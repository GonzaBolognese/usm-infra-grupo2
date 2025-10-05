# Trabajo Práctico Final – Infraestructura en Ciencia de Datos  
### Sistema de Venta Indirecta con Distribuidoras  
**Universidad Nacional de San Martín (UNSAM)**  
**Profesor:** Leandro E. Lucero  
**Alumno:** Gonzalo Ariel Bolognese  
**Año:** 2025  

---

## ANÁLISIS

El presente trabajo tiene como objetivo aplicar los conceptos aprendidos en la materia **Infraestructura en Ciencia de Datos** para el diseño de una arquitectura de procesamiento y análisis de datos en la nube.  

El caso práctico aborda el desarrollo de un **Sistema de Venta Indirecta con Distribuidoras**, en el cual cada distribuidor trabaja con sus propios sistemas de gestión y reporta información a una plataforma central de análisis.  

**Objetivos principales:**
- Centralizar los datos de ventas, stock, clientes y deuda provenientes de distintas distribuidoras.  
- Implementar un flujo automatizado de carga y procesamiento de datos utilizando servicios de **Google Cloud Platform (GCP)**.  
- Diseñar una infraestructura **escalable, virtualizada y segura** que permita análisis y visualizaciones en tiempo real mediante dashboards.  

**Problemas detectados:**
- Fuentes de datos heterogéneas y dispersas.  
- Procesos manuales de consolidación de reportes.  
- Necesidad de una plataforma flexible que soporte crecimiento anual del 20% (Clase 3).  

---

## DESARROLLO

### 1. Arquitectura General del Sistema

El sistema se estructura en tres niveles principales:

1. **Redes Distribuidoras Locales:**  
   Cada distribuidora cuenta con su propio “Sistema Ventanas” y “Agente Argentina Ideal”, que generan datos de:
   - Clientes  
   - Ventas  
   - Stock  
   - Deuda  

   Estos datos se exportan periódicamente y son subidos a un **bucket en Google Cloud Storage**.

2. **Sistema Central – Venta Indirecta:**  
   El sistema central recibe la información consolidada de todas las distribuidoras y la almacena en un **Data Warehouse**.  
   Posteriormente, los datos son transformados en **Data Mart** y visualizados mediante un **Dashboard** (Looker Studio).  

3. **Usuarios Analíticos:**  
   - Analista de Ventas y Marketing  
   - Analista de Finanzas  
   - Analista de Planificación de Suministros  

   Cada uno accede a vistas específicas del Data Mart para la toma de decisiones.

---

### 2. Proceso de Generación y Carga de Datos

Para simular las fuentes de información, se utilizó el script **`generador.py`**, que crea datasets de ventas y clientes para cada distribuidora.  
Los archivos generados se almacenaron en el **Google Drive** y luego fueron cargados en un **bucket de Cloud Storage** dentro del proyecto de GCP.  

Este bucket actúa como **zona de aterrizaje de datos (landing zone)**, desde donde una instancia de **Compute Engine** ejecuta procesos ETL (Extract, Transform, Load).

---

### 3. Servicios Utilizados en Google Cloud Platform (GCP)

| Componente | Servicio GCP | Categoría | Función |
|-------------|---------------|-----------|----------|
| **Almacenamiento de datos brutos** | Cloud Storage | Storage Service | Guarda los datasets generados por `generador.py`. |
| **Procesamiento y transformación (ETL)** | Compute Engine | IaaS – Computing Service | Ejecuta scripts de limpieza y carga hacia BigQuery. |
| **Almacenamiento analítico** | BigQuery | Big Data Service | Actúa como Data Warehouse, permitiendo consultas SQL sobre grandes volúmenes de datos. |
| **Visualización** | Looker Studio (Data Studio) | Visualización / BI | Construcción de dashboards con indicadores clave. |
| **Orquestación (opcional)** | Cloud Dataflow / Cloud Composer | PaaS – Big Data | Automatiza los flujos de carga periódica. |

> **Referencia:**  
> Clase 3 – *Servicios y herramientas de Engineer Computer*, diap. 10–15: “Computing Service y Storage Service”.  
> Clase 5 – *Introducción a GCP*, diap. 20–30: “BigQuery, Dataflow, Datalab, AI Platform”.

---

### 4. Tipos de Máquinas y Recursos en Compute Engine

Según el tipo de carga de trabajo, se definieron las siguientes opciones de instancias virtuales:

| Tipo de máquina | Familia | Uso recomendado | Características principales |
|------------------|----------|------------------|------------------------------|
| **E2-standard** | Uso general | Procesos ETL livianos y pruebas | Bajo costo, hasta 32 vCPU y 128 GB de RAM. |
| **N2-standard** | Uso general | Consolidación de datos de varias distribuidoras | Escalable hasta 128 vCPU y 8 GB por vCPU. |
| **A2-highgpu** | Optimizada para aceleradores (GPU NVIDIA A100) | Entrenamiento o inferencia de modelos ML | Alta capacidad de cómputo paralelo y 80 GB HBM2. |

> **Referencia:**  
> Documentación “Tipos de máquinas GPU – Compute Engine” (Google Cloud, 2024)  
> Guía comparativa de familias de máquinas – Compute Engine (2024).

---

### 5. Virtualización y Escalabilidad

El uso de **Compute Engine** permite ejecutar máquinas virtuales en una infraestructura de nube distribuida, sin necesidad de hardware físico.  
Estas VMs están basadas en tecnología de **hipervisores tipo 1** (bare metal), lo que garantiza aislamiento, rendimiento y seguridad (Clase 2, diap. 27).  

**Ventajas principales:**
- Escalabilidad automática ante picos de demanda.  
- Pago por uso (modelo OPEX vs CAPEX).  
- Balanceo de carga y alta disponibilidad (Clase 5).  
- Recuperación ante desastres y replicación automática.  

La arquitectura propuesta puede escalar horizontalmente añadiendo más instancias por cada nueva distribuidora.

---

### 6. Flujo de Datos Resumido

1. Distribuidoras locales generan y exportan datos.  
2. Archivos subidos al **bucket de Cloud Storage**.  
3. **Compute Engine** procesa y limpia los datos (Python / ETL).  
4. Datos cargados a **BigQuery (Data Warehouse)**.  
5. Generación de **Data Mart** con vistas temáticas (Ventas, Stock, Deuda).  
6. Visualización en **Looker Studio** mediante conexión directa a BigQuery.  

---

## RESPUESTA

El sistema propuesto de **Venta Indirecta con Distribuidoras** permite integrar de forma eficiente la información de diferentes redes comerciales mediante una arquitectura **en la nube, virtualizada y escalable**.  

**Principales beneficios:**
- Centralización de datos críticos de negocio.  
- Reducción de tiempos de procesamiento y errores manuales.  
- Mayor capacidad analítica gracias a BigQuery y Looker Studio.  
- Elasticidad para incorporar nuevas distribuidoras con mínima configuración.  
- Bajo costo inicial y alta disponibilidad gracias a la infraestructura de Google Cloud.

**Comparación Cloud vs On-Premise (según Clase 5):**
| Criterio | On-Premise | Cloud Computing (GCP) |
|-----------|-------------|-----------------------|
| Costo inicial | Alto (CAPEX) | Nulo (modelo OPEX) |
| Escalabilidad | Limitada | Automática |
| Mantenimiento | Local y manual | Gestionado por Google |
| Seguridad | Local y dependiente del hardware | Multi-factor, replicación y cifrado |
| Elasticidad | Estática | Dinámica y bajo demanda |

---

## FUENTES

- **Clase 1 – Introducción a la Arquitectura del Computador**  
  Conceptos de CPU, memoria y jerarquía de almacenamiento.  
- **Clase 2 – Arquitectura del Computador II**  
  Virtualización, tipos de hipervisores y ventajas de la nube.  
- **Clase 3 – Servicios y herramientas de Engineer Computer**  
  Compute Engine, tipos de máquinas, GPU y casos de uso.  
- **Clase 4 – Redes**  
  Conectividad, redes virtuales y segmentación (VPC).  
- **Clase 5 – Introducción a GCP**  
  Modelos de servicio (IaaS, PaaS, SaaS), BigQuery, Dataflow, IAM.  
- **Documentación oficial de Google Cloud (Compute Engine, 2024):**  
  - *Tipos de máquinas GPU*  
  - *Guía comparativa de familias de máquinas*

---

## CONCLUSIÓN

La arquitectura desarrollada demuestra la aplicación integral de los conceptos de **infraestructura de ciencia de datos**, **virtualización** y **cloud computing** vistos en clase.  
El sistema de **Venta Indirecta con Distribuidoras** consolida datos dispersos, optimiza recursos y permite a la organización contar con información analítica confiable y actualizada.  

Su diseño en **Google Cloud Platform** garantiza escalabilidad, disponibilidad y seguridad, cumpliendo con los objetivos de una infraestructura moderna de ciencia de datos.

---


# 4. Ejecutar análisis
bq query "SELECT distribuidor, SUM(venta_importe) FROM dataset.ventas GROUP BY 1"
``` 
