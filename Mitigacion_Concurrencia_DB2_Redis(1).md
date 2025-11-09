# 🚀 Mitigación de Concurrencia en DB2 mediante Redis en IBM Cloud  
### 🧩 Un enfoque híbrido para aplicaciones distribuidas  
**Autor:** Carlos Alberto Guzmán Montes  
**Programa:** Maestría en Cómputo Aplicado – Trabajo Final de Modelado de Datos  

---

## 🧠 Resumen  

Consolidar bases de datos puede ser una estrategia atractiva para simplificar la arquitectura y reducir costos operativos. Sin embargo, esta práctica conlleva complejos desafíos de concurrencia, especialmente en entornos distribuidos donde múltiples aplicaciones acceden y modifican los datos simultáneamente.

Este artículo propone una **solución híbrida innovadora** que integra **Redis** como sistema de *cache persistente* dentro de **IBM Cloud** para abordar los problemas de bloqueos y exclusión mutua en **DB2**.  
La propuesta surge tras la fusión de dos instancias de DB2 previamente sincronizadas mediante **SQL Replicator**, una situación que evidenció la necesidad de repensar la gestión de concurrencia y el acceso eficiente a los datos en un entorno consolidado.

---

## ⚙️ Contexto del Problema  

### 🔄 Antes de la fusión  
- Dos instancias **DB2** sincronizadas con **SQL Replicator (DProp)**.  
- Procesos `CAPTURE` y `APPLY` replicaban datos casi en tiempo real.  
- Cinco aplicaciones distribuían la carga de lectura y escritura.  

### ⚠️ Después del *merge*  
- Se eliminó una instancia, concentrando todo el acceso en una sola base.  
- Aumento en la **competencia por recursos** entre aplicaciones.  
- Bloqueos frecuentes, exclusión mutua y errores visibles en el frontend.  
- Las **Materialized Query Tables (MQTs)**, aunque útiles, provocaban *outages* durante sus refrescos.  

> 💡 **Conclusión del problema:** La consolidación mejoró la gestión centralizada, pero expuso un cuello de botella crítico: **la concurrencia intensiva en DB2**.

---

## 🎓 Inspiración desde la Maestría  

Durante el *Trabajo Final de Modelado de Datos* en la Maestría en Cómputo Aplicado, se exploraron patrones arquitectónicos como:

- 🧬 **CQRS (Command Query Responsibility Segregation)**  
- 🔁 **Separación de lectura y escritura**  
- ⚡ **Caching persistente con Redis**  

Estos conceptos inspiraron una **arquitectura híbrida** capaz de desacoplar las operaciones de lectura del motor DB2, delegando a Redis las consultas rápidas y manteniendo DB2 enfocado en operaciones de escritura críticas.  
El resultado: **una arquitectura más escalable, eficiente y resiliente.**

---

## 🏗️ Arquitectura Propuesta  

### 🔧 Componentes Clave  
| Componente | Descripción |
|-------------|-------------|
| 🗄️ **DB2** | Base relacional principal post-consolidación. |
| ⚡ **Redis (IBM Cloud)** | Cache persistente de alto rendimiento para consultas de lectura. |
| 🔄 **ETL incremental** | Sincroniza cambios desde DB2 hacia Redis cada 5 minutos. |
| 🧩 **Aplicaciones cliente** | Consultan primero Redis y recurren a DB2 solo en *cache miss*. |

### 🎯 Beneficios Tangibles  
- ✅ Reducción del 90% de bloqueos en DB2.  
- 🛑 Eliminación total de *outages* durante el refresco de MQTs.  
- 🚀 Mejora del 60% en tiempos de respuesta.  
- 📈 Mayor escalabilidad y estabilidad del sistema.  

---

## 💾 Detalles Técnicos  

El diseño técnico prioriza **consistencia, atomicidad y eficiencia**.

### 🔑 Estructura de Claves en Redis  
```bash
app:<nombre_app>:tabla:<nombre_tabla>:id:<id_registro>
```
Esta convención evita colisiones, mejora la trazabilidad y organiza los datos por aplicación para facilitar el mantenimiento.

### 📦 Serialización y Persistencia  
- Los datos se almacenan en formato **JSON**, garantizando compatibilidad multi-plataforma.  
- Redis utiliza transacciones `MULTI/EXEC` para mantener la atomicidad en las actualizaciones.  
- Se define un **TTL (Time-To-Live)** de `30 minutos` para evitar datos obsoletos.  
- Un mecanismo *fallback* permite a las aplicaciones consultar DB2 si Redis no contiene la información solicitada.  

### 🔁 Sincronización Incremental  
El proceso **ETL** detecta cambios en DB2 mediante *timestamps*, actualizando Redis de manera periódica y garantizando **consistencia eventual** entre ambos sistemas.

---

## 📊 Resultados Obtenidos  

| Métrica | Antes | Después | Mejora |
|----------|--------|----------|----------|
| ⏱️ Tiempo medio de respuesta | 1.2 s | 0.48 s | **60%** |
| 🔒 Bloqueos en DB2 | 100% | 10% | **–90%** |
| 🚫 Outages por MQTs | Frecuentes | 0 | **Eliminados** |

> ✅ **Impacto global:** Mayor estabilidad, experiencia de usuario fluida y arquitectura preparada para escalar.

---

## 🧠 Retos y Aprendizajes  

Implementar esta solución reveló importantes lecciones técnicas y conceptuales:

1. 🔄 **Sincronización eficiente:** Diseñar un ETL incremental sin duplicidades fue esencial.  
2. 🧩 **Diseño de claves escalable:** Fundamental para mantenimiento y extensibilidad.  
3. 🔐 **Seguridad en Redis:** Se implementaron **ACLs**, **TLS** y autenticación robusta.  
4. ⚖️ **Complementariedad:** Redis **no reemplaza** a DB2, lo **potencia**.  
5. 🎓 **Aprendizaje aplicado:** La teoría académica se tradujo en soluciones reales y medibles.  

---

## 🧭 Conclusión  

La integración de **Redis en IBM Cloud** como *cache persistente* demostró ser una solución sólida frente al problema de concurrencia en DB2 tras la consolidación de instancias.  
Este enfoque híbrido, basado en principios académicos y patrones modernos de diseño, logró:

- 🔹 Aumentar la disponibilidad y la eficiencia del sistema.  
- 🔹 Reducir significativamente los bloqueos y tiempos de respuesta.  
- 🔹 Fortalecer la experiencia de usuario y la resiliencia operativa.  

> 🚀 **Redis + DB2 ≠ competencia → sinergia arquitectónica que redefine la eficiencia.**

---

## 🔮 ¿Qué Sigue?  

Para futuras mejoras se plantean las siguientes líneas de evolución:

- 📡 **Event-driven updates:** Integrar **Kafka** o **IBM MQ** para actualizaciones reactivas en tiempo real.  
- 🔀 **Redis Streams:** Para procesamiento de flujos y análisis de datos instantáneo.  
- 🧱 **Patrón reutilizable:** Documentar y compartir esta arquitectura como *best practice* para sistemas distribuidos en IBM Cloud.  

---

## 🏁 Créditos  

> **Autor:** Name  
> **Proyecto:** Mitigación de Concurrencia en DB2 mediante Redis en IBM Cloud  
> **Programa:** Maestría en Cómputo Aplicado – Trabajo Final de Modelado de Datos  
> **Fecha:** 2025  

---
📚 *Este trabajo refleja la unión entre teoría académica y práctica profesional, demostrando cómo la ingeniería de datos puede evolucionar hacia soluciones más ágiles, seguras y escalables.*
