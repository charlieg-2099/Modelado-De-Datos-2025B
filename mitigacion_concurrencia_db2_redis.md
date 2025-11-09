
# Mitigación de Concurrencia en DB2 mediante Redis en IBM Cloud  
### Un enfoque híbrido para aplicaciones distribuidas  
**Autor:** Name  
**Maestría en Cómputo Aplicado – Trabajo Final de Modelado de Datos**

---

## 🧠 Resumen
La consolidación de bases de datos puede simplificar la arquitectura, pero también puede introducir problemas de concurrencia en sistemas distribuidos. Este artículo presenta una solución híbrida que utiliza Redis como sistema de cache en IBM Cloud para resolver bloqueos y exclusión mutua en DB2, tras la fusión de dos instancias previamente sincronizadas con SQL Replicator.

---

## 🧩 Contexto del Problema

### Antes del merge:
- Existían dos instancias DB2 sincronizadas mediante SQL Replicator (Dprop).
- Se utilizaban procesos de **CAPTURE** y **APPLY** para replicar datos en tiempo real.
- Cinco aplicaciones accedían a ambas bases, distribuyendo la carga de lectura/escritura.

### Después del merge:
- Se eliminó una instancia DB2, consolidando todo en una sola.
- Las aplicaciones comenzaron a competir por acceso a las mismas tablas.
- Se generaron bloqueos, exclusión mutua y errores en el frontend.
- El uso de **Materialized Query Tables (MQTs)** como solución parcial provocaba *outages* temporales al refrescarse.

---

## 🎓 Inspiración desde la Maestría
Durante el curso de Modelado de Datos, se exploraron patrones como:

- CQRS (Command Query Responsibility Segregation)
- Read/Write Separation
- Caching persistente con Redis

Estas ideas llevaron a proponer una arquitectura híbrida que desacoplara las consultas de lectura del motor DB2, utilizando Redis como sistema de cache.

---

## 🏗️ Arquitectura Propuesta

### Componentes principales:
- 🗄️ **DB2 (Post-Merge):** Base de datos relacional principal.  
- ⚡ **Redis (IBM Cloud):** Cache persistente para consultas de lectura.  
- 🔄 **ETL incremental:** Script que sincroniza datos entre DB2 y Redis.  
- 🧩 **Aplicaciones:** Modificadas para consultar Redis en lugar de DB2 cuando sea posible.

### Ventajas clave:
- ✅ Reducción de bloqueos en DB2.  
- 🛑 Eliminación de *outages* durante el REFRESH de MQTs.  
- 🚀 Mejora en la experiencia del usuario.  
- 📈 Mayor escalabilidad para futuras aplicaciones.

---

## ⚙️ Detalles Técnicos

### 🔑 Diseño de claves en Redis:
```
app:<nombre_app>:tabla:<nombre_tabla>:id:<id_registro>
```
- Acceso rápido por ID.  
- Segmentación por aplicación.  
- Evita colisiones y facilita mantenimiento.

### 📦 Serialización de datos:
- Uso de **JSON** para compatibilidad entre lenguajes.  
- Las aplicaciones pueden deserializar fácilmente los datos.

### ⏱️ Estrategia de actualización:
- ETL detecta cambios en DB2 mediante timestamps.  
- Actualiza Redis cada 5 minutos.  
- Uso de **Redis Transactions (MULTI/EXEC)** para garantizar atomicidad.

### 🧯 TTL y fallback:
- TTL de 30 minutos.  
- Si el dato no está en Redis, la aplicación consulta DB2 como respaldo.

---

## 📊 Resultados Obtenidos

- ✅ Reducción del **90%** en bloqueos de tabla.  
- 🚀 Mejora del **60%** en tiempo de respuesta.  
- 🛑 Cero *outages* durante el REFRESH de MQTs.  
- 🎯 Mayor estabilidad en el frontend.

---

## 🧠 Retos y Aprendizajes

### Desafíos enfrentados:
- Sincronización eficiente entre DB2 y Redis.  
- Diseño de claves escalable y mantenible.  
- Seguridad en Redis (ACLs, TLS, autenticación).  
- Evitar duplicidad y mantener consistencia eventual.

### Lecciones aprendidas:
- Redis no reemplaza DB2, pero lo complementa perfectamente.  
- La separación de responsabilidades mejora la escalabilidad.  
- La formación académica puede transformar directamente la práctica profesional.

---

## 🏁 Conclusión
La implementación de Redis como sistema de cache en IBM Cloud permitió resolver un problema crítico de concurrencia en DB2 tras la fusión de bases de datos. Esta solución híbrida, inspirada en conceptos académicos, demuestra cómo el diseño de datos y la arquitectura distribuida pueden mejorar significativamente la disponibilidad y el rendimiento de aplicaciones empresariales.

---

## 🔮 ¿Qué sigue?

- 🔁 Implementar actualizaciones *event-driven* con Kafka o IBM MQ.  
- 📊 Explorar Redis Streams para datos en tiempo real.  
- 📚 Publicar esta solución como patrón reutilizable en entornos similares.
