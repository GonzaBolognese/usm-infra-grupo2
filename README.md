## 🎯 Objetivos de las Actividades

### **Actividad 1 (Obligatoria)**: Agregar Columna Distribuidor
Agregar la columna `distribuidor` a todos los archivos CSV antes de cargarlos a BigQuery/Data Warehouse.

### **Actividad 2 (Opcional/Avanzada)**: Implementar Subida a Google Cloud Storage
Implementar el método `_upload_to_gcs()` para subir automáticamente los archivos generados a un bucket de GCS.

---

## 📋 ACTIVIDAD 1: Columna Distribuidor (OBLIGATORIA)

### ❓ ¿Por qué agregar la columna distribuidor?

#### 1. **Particionado Eficiente en BigQuery**
```sql
-- Con la columna distribuidor puedes hacer:
SELECT * FROM ventas WHERE distribuidor = 1 AND fecha_cierre = '2024-01-01'

-- Sin la columna, tendrías que hacer:
SELECT * FROM ventas WHERE _FILE_NAME LIKE '%Distribuidor_1%'
```

#### 2. **Mejores Consultas y Filtros**
- Filtrar por distribuidor es instantáneo
- Agregaciones por distribuidor son más eficientes  
- Joins entre tablas son más simples

#### 3. **Particionado de Tablas**
```sql
-- BigQuery puede particionar automáticamente:
CREATE TABLE dataset.ventas_partitioned
PARTITION BY distribuidor
AS SELECT * FROM ventas_raw
```

#### 4. **Control de Acceso**
- Diferentes equipos pueden acceder solo a su distribuidor
- Seguridad a nivel de fila basada en distribuidor
- Auditoría más granular

### 📍 Dónde Agregar la Columna

Busca estos comentarios en `generador.py`:

#### 1. Archivo de Ventas (línea ~299)
```python
ventas.append({
    'sucursal': cliente.sucursal,
    'cliente': cliente.id_cliente,
    'fecha_cierre': fecha.strftime('%Y-%m-%d'),
    'sku': producto_sku,
    'venta_unidades': venta['cantidad'],
    'venta_importe': venta['importe'],
    'condicion_venta': cliente.condicion_venta
    # TODO ESTUDIANTES: Agregar columna 'distribuidor' aquí
    # 'distribuidor': distribuidor
})
```

**✅ Solución:**
```python
ventas.append({
    'sucursal': cliente.sucursal,
    'cliente': cliente.id_cliente,
    'fecha_cierre': fecha.strftime('%Y-%m-%d'),
    'sku': producto_sku,
    'venta_unidades': venta['cantidad'],
    'venta_importe': venta['importe'],
    'condicion_venta': cliente.condicion_venta,
    'distribuidor': distribuidor  # ← AGREGAR ESTA LÍNEA
})
```

#### 2. Archivo de Stock (línea ~320)
```python
stock_record = {
    'sucursal': distribuidor * 100 + 1,
    'fecha_cierre': fecha_str,
    'sku': sku,
    'producto': PRODUCTOS[sku]['nombre'],
    'stock': info['cantidad'],
    'unidad': info['unidad']
    # TODO ESTUDIANTES: Agregar columna 'distribuidor' aquí
    # 'distribuidor': distribuidor
}
```

**✅ Solución:**
```python
stock_record = {
    'sucursal': distribuidor * 100 + 1,
    'fecha_cierre': fecha_str,
    'sku': sku,
    'producto': PRODUCTOS[sku]['nombre'],
    'stock': info['cantidad'],
    'unidad': info['unidad'],
    'distribuidor': distribuidor  # ← AGREGAR ESTA LÍNEA
}
```

#### 3. Archivo Maestro (línea ~340)
```python
maestro_record = {
    'sucursal': cliente.sucursal,
    'cliente': cliente.id_cliente,
    # ... otros campos ...
    'tipo_negocio': cliente.tipo_negocio
    # TODO ESTUDIANTES: Agregar columna 'distribuidor' aquí
    # 'distribuidor': distribuidor
}
```

**✅ Solución:**
```python
maestro_record = {
    'sucursal': cliente.sucursal,
    'cliente': cliente.id_cliente,
    # ... otros campos ...
    'tipo_negocio': cliente.tipo_negocio,
    'distribuidor': distribuidor  # ← AGREGAR ESTA LÍNEA
}
```

---

## ☁️ ACTIVIDAD 2: Google Cloud Storage (OPCIONAL)

### 🎯 Objetivo
Implementar el método `_upload_to_gcs()` para subir automáticamente los archivos CSV generados a un bucket de Google Cloud Storage.

### 📦 Preparación

#### 1. Instalar Dependencias
```bash
pip install google-cloud-storage
```

#### 2. Configurar Credenciales
```bash
# Opción 1: Variable de entorno
export GOOGLE_APPLICATION_CREDENTIALS="path/to/credentials.json"

# Opción 2: Autenticación con gcloud
gcloud auth application-default login
```

#### 3. Crear Bucket GCS
```bash
gsutil mb gs://mi-bucket-universidad
```

### 🔧 Implementación

Busca el método `_upload_to_gcs()` en `generador.py` (línea ~XX) y reemplaza el contenido:

**🚀 Código a Implementar:**
```python
def _upload_to_gcs(self, archivos, bucket_name, distribuidor):
    """
    Sube archivos a Google Cloud Storage.
    """
    try:
        from google.cloud import storage
        
        # Crear cliente GCS
        client = storage.Client()
        bucket = client.bucket(bucket_name)
        
        # Subir cada archivo
        for archivo in archivos:
            if os.path.exists(archivo):
                # Estructura: data/distribuidor_X/archivo.csv
                blob_name = f"data/distribuidor_{distribuidor}/{os.path.basename(archivo)}"
                blob = bucket.blob(blob_name)
                
                # Subir archivo
                blob.upload_from_filename(archivo)
                print(f"    ☁️ Subido a GCS: {blob_name}")
                
    except ImportError:
        print("    ⚠️ google-cloud-storage no instalado")
    except Exception as e:
        print(f"    ❌ Error subiendo a GCS: {str(e)}")
```

### 🔌 Activar Subida GCS

Busca el comentario en el bucle principal (línea ~XX):

```python
# 🎯 ACTIVIDAD ADICIONAL PARA ESTUDIANTES:
# Implementar subida a Google Cloud Storage aquí
# Ejemplo de uso:
# archivos_a_subir = [archivo_ventas, archivo_stock]
# if dia == 0:  # Agregar maestro solo el primer día
#     archivos_a_subir.append(archivo_maestro)
# self._upload_to_gcs(archivos_a_subir, 'nombre-bucket', distribuidor)
```

**✅ Solución:**
```python
# Subir archivos a GCS (implementado por estudiantes)
archivos_a_subir = [archivo_ventas, archivo_stock]
if dia == 0:  # Agregar maestro solo el primer día
    archivos_a_subir.append(archivo_maestro)
self._upload_to_gcs(archivos_a_subir, 'mi-bucket-universidad', distribuidor)
```

### 🏗️ Estructura Resultante en GCS
```
mi-bucket-universidad/
├── data/
│   ├── distribuidor_1/
│   │   ├── Venta_Clientes_2024-01-01.csv
│   │   ├── StockPeriodo_2024-01-01.csv
│   │   └── Maestro_2024-01-01.csv
│   ├── distribuidor_2/
│   │   ├── Venta_Clientes_2024-01-01.csv
│   │   └── ...
│   └── distribuidor_3/
└── ...
```

---

## 🧪 Verificando las Implementaciones

### 1. Ejecutar el generador
```bash
python generador.py
```

### 2. Verificar Actividad 1: Columna Distribuidor
```bash
# Verificar headers de archivos generados
head -1 output/Archivos_VentaClientes/Distribuidor_1/Venta_Clientes_*.csv
head -1 output/Archivos_Stock/Distribuidor_1/StockPeriodo_*.csv  
head -1 output/Archivos_Maestro/Distribuidor_1/Maestro_*.csv
```

**✅ Deberías ver:**
```
sucursal,cliente,fecha_cierre,sku,venta_unidades,venta_importe,condicion_venta,distribuidor
sucursal,fecha_cierre,sku,producto,stock,unidad,distribuidor
sucursal,cliente,ciudad,provincia,estado,...,distribuidor
```

### 3. Verificar Actividad 2: Subida GCS
```bash
# Listar archivos en GCS
gsutil ls -r gs://mi-bucket-universidad/

# Debería mostrar:
# gs://mi-bucket-universidad/data/distribuidor_1/Venta_Clientes_2024-XX-XX.csv
# gs://mi-bucket-universidad/data/distribuidor_1/StockPeriodo_2024-XX-XX.csv
# etc.
```

---

## 📊 Cargando a BigQuery

### 1. Desde GCS (Recomendado)
```sql
-- Crear tabla particionada por distribuidor
CREATE TABLE `proyecto.distribucion_data.ventas`
(
  sucursal INT64,
  cliente INT64,
  fecha_cierre DATE,
  sku STRING,
  venta_unidades INT64,
  venta_importe FLOAT64,
  condicion_venta STRING,
  distribuidor INT64
)
PARTITION BY distribuidor;

-- Cargar desde GCS
LOAD DATA INTO `proyecto.distribucion_data.ventas`
FROM FILES (
  format = 'CSV',
  skip_leading_rows = 1,
  uris = ['gs://mi-bucket-universidad/data/*/Venta_Clientes_*.csv']
);
```

### 2. Desde Local
```bash
# Cargar usando bq command line
bq load --source_format=CSV --skip_leading_rows=1 \
  proyecto:distribucion_data.ventas \
  output/Archivos_VentaClientes/*/*.csv
```

---

## 📈 Consultas Optimizadas

### Ventas por Distribuidor
```sql
SELECT 
    distribuidor,
    COUNT(*) as total_ventas,
    SUM(venta_importe) as facturacion_total
FROM `proyecto.distribucion_data.ventas`
WHERE distribuidor = 1  -- Solo procesa partición 1
  AND fecha_cierre >= '2024-01-01'
GROUP BY distribuidor;
```

### Stock Crítico por Distribuidor
```sql
SELECT 
    distribuidor,
    sku,
    producto,
    stock
FROM `proyecto.distribucion_data.stock`
WHERE distribuidor = 2  -- Solo partición 2
  AND stock < 100
ORDER BY stock ASC;
```

### Análisis Cross-Distribuidor
```sql
-- Comparar performance entre distribuidores
SELECT 
    distribuidor,
    AVG(venta_importe) as ticket_promedio,
    COUNT(DISTINCT cliente) as clientes_unicos,
    SUM(venta_importe) as facturacion_total
FROM `proyecto.distribucion_data.ventas`
WHERE fecha_cierre >= '2024-01-01'
GROUP BY distribuidor
ORDER BY facturacion_total DESC;
```

---

## ✅ Checklist de Completitud

### Actividad 1: Columna Distribuidor
- [ ] Agregué columna `distribuidor` en ventas
- [ ] Agregué columna `distribuidor` en stock  
- [ ] Agregué columna `distribuidor` en maestro
- [ ] Ejecuté el generador sin errores
- [ ] Verifiqué que los CSV incluyan la nueva columna
- [ ] Los valores de distribuidor son correctos (1, 2, 3)

### Actividad 2: GCS Upload (Opcional)
- [ ] Instalé google-cloud-storage
- [ ] Configuré credenciales de GCS
- [ ] Creé bucket en GCS
- [ ] Implementé método _upload_to_gcs()
- [ ] Activé llamada al método en el bucle principal
- [ ] Verifiqué que archivos se suban a GCS correctamente
- [ ] La estructura de carpetas en GCS es correcta

### BigQuery Integration
- [ ] Cargué al menos un archivo en BigQuery
- [ ] Creé tabla particionada por distribuidor
- [ ] Ejecuté consultas con filtro por distribuidor
- [ ] Verifiqué mejora de performance vs tabla no particionada

---

## 🏆 Beneficios Obtenidos

1. **Performance**: Consultas 10x más rápidas con particionado
2. **Costos**: BigQuery solo procesa particiones necesarias
3. **Escalabilidad**: Pipeline automático local → GCS → BigQuery
4. **Mantenibilidad**: Código SQL más limpio y legible
5. **Profesional**: Experiencia con herramientas cloud reales

## 🤔 Preguntas de Reflexión

1. **¿Qué ventajas tiene usar GCS como intermediario vs subir directo a BigQuery?**
2. **¿Cómo implementarías particionado por fecha además de distribuidor?**
3. **¿Qué estrategia usarías para datos incremental vs full refresh?**
4. **¿Cómo manejarías errores en la subida a GCS en un entorno productivo?**

---

## 💡 Conceptos Avanzados

### Particionado Híbrido
```sql
CREATE TABLE ventas_optimizada
PARTITION BY distribuidor
CLUSTER BY fecha_cierre, cliente;
```

### Pipeline Completo
```bash
# 1. Generar datos
python generador.py

# 2. Subir a GCS (automático si implementaste)

# 3. Cargar a BigQuery
bq load --source_format=CSV --skip_leading_rows=1 \
  dataset.ventas gs://bucket/data/*/Venta_*.csv

# 4. Ejecutar análisis
bq query "SELECT distribuidor, SUM(venta_importe) FROM dataset.ventas GROUP BY 1"
``` 
