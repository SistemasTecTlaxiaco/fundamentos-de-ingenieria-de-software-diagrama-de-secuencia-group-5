# Práctica 10: Documento Integrador — Modelo de Dominio, Secuencias y Arquitectura
**Sistema de Gestión de Información de Proyectos Agroecológicos de CEDICAM (SIGIPA-CEDICAM)**

| Concepto | Datos de la Entrega |
| :--- | :--- |
| **Institución** | Tecnológico Nacional de México · Instituto Tecnológico de Tlaxiaco |
| **Programa Educativo** | Ingeniería en Sistemas Computacionales |
| **Asignatura** | Fundamentos de Ingeniería de Software (SCC-1007) · Unidad 4: Modelo de análisis (Sprint 4) |
| **Organización / Cliente** | CEDICAM - Centro de Desarrollo Integral Campesino de la Mixteca (Coordinador Técnico) |
| **Integrantes del Equipo** | Evelyn Sosa José, Irwing Sael Feria Caballero, Ebenezer Acevedo Nicolas, Jazmin Aguilar Rojas |
| **Grupo y Docente** | Grupo 5BS · Docente: Ing. José Alfredo Román Cruz |
| **Periodo** | Agosto - Diciembre 2026 |
| **Entregables del Repositorio** | `modelo/OpenHub-Equipo.qea`, PNGs en `docs/analisis/`, `docs/analisis/documento-integrador.md` |

---

## Introducción

Este documento integrador consolida los modelos de negocio (Unidad 2), la especificación de requisitos validados (Unidad 3) y la representación técnica orientada a objetos (Unidad 4) del sistema **SIGIPA-CEDICAM**. Su objetivo es demostrar la consistencia estructural y dinámica entre el modelo conceptual del dominio, los escenarios de los casos de uso mediante diagramas de secuencia, el diagrama de clases actualizado y la arquitectura de software en cuatro capas, asegurando el funcionamiento *offline* en dispositivos Android de gama baja para los promotores comunitarios de CEDICAM.

---

## 1. Modelo de Dominio

El modelo de dominio recoge exclusivamente los conceptos fundamentales del negocio extraídos del diagrama BPMN mejorado (Unidad 2), las entrevistas de elicitación (Práctica 7) y el glosario del DRS v1.0, empleando la terminología exacta utilizada por los técnicos y comunidades de CEDICAM. **Se excluyen por completo clases tecnológicas o de software** (como `Pantalla`, `Controlador`, `BaseDeDatos` o `SQLite`).

### 1.1 Conceptos, Atributos Conceptuales y Reglas de Asociación

De acuerdo con la notación de análisis conceptual, los atributos del modelo de dominio se expresan sin tipos de datos primitivos, sin operaciones y sin símbolos de visibilidad. Cada asociación cuenta con nombre descriptivo y multiplicidad en ambos extremos para leerse en voz alta como una regla operativa de la organización:

| Concepto del Dominio | Atributos Conceptuales | Asociaciones y Reglas de Lectura en Voz Alta (Organización) |
| :--- | :--- | :--- |
| **`Proyecto`** | `nombre`, `descripcion`, `fechaInicio`, `fechaFin` | • Un `Proyecto` (`1..*`) **integra** a uno o muchos `Participante` (`1..*`).<br>• Cada `Proyecto` (`1`) contempla una o más `ObraConservacion` (`1..*`). |
| **`Comunidad`** | `nombre`, `municipio`, `zonaRecargaHidrica`, `referenciaMapa` | • Cada `Comunidad` (`1`) **ubica_en** cero o muchos `Proyecto` (`0..*`).<br>• Una `Comunidad` (`1`) es el lugar al que **pertenece_a** cero o muchos `Participante` (`0..*`). |
| **`Participante`** | `nombre`, `rol`, `contacto` | • Cada `Participante` (`1..*`) participa en uno o varios `Proyecto` (`1..*`).<br>• Cada `Participante` (`0..*`) reside en exactamente una `Comunidad` (`1`). |
| **`ObraConservacion`** | `tipo`, `ubicacionReferencia`, `fechaRegistro`, `estadoActual`, `responsableTecnico` | • Toda `ObraConservacion` (`1..*`) **pertenece_a_proyecto** (`1`).<br>• Cada `ObraConservacion` (`0..*`) **se_construye_en** exactamente una `Comunidad` (`1`). |
| **`Capacitacion`** | `tema`, `fecha`, `numAsistentes`, `facilitador` | • Una `Capacitacion` (`0..*`) se imparte en una `Comunidad` (`1`) con asistencia de `Participante` (`1..*`). |
| **`TecnicaAgroecologica`** | `nombrePractica`, `fechaAplicacion`, `resultado` | • Una `TecnicaAgroecologica` (`0..*`) se documenta en una `Comunidad` (`1`). |

![Diagrama del Modelo de Dominio - SIGIPA-CEDICAM](modelo-dominio.png)
*Figura 1. Modelo de dominio conceptual del sistema SIGIPA-CEDICAM (Paquete `«Modelo de dominio»`).*

---

## 2. Diagramas de Secuencia basados en Casos de Uso

Se elaboraron dos diagramas de secuencia detallados que traducen paso a paso las especificaciones de los casos de uso de la Actividad 3, incluyendo tanto el flujo principal como los flujos alternos críticos mediante fragmentos combinados UML (`alt`, `opt`, `loop`). Ambas secuencias aplican el patrón arquitectónico **Frontera – Control – Entidad**, garantizando que el actor interactúe únicamente con los objetos de frontera.

### 2.1 Escenario 1: Registro de un Nuevo Proyecto Agroecológico (`CU-01`)
* **Requisitos trazados:** `RF-01`, `RF-06`, `RNF-01`.
* **Líneas de vida participantes:**
  1. `promotor : PromotorComunitario` *(Actor)*
  2. `vistaProyecto : FrmRegistroProyecto` *(Frontera)*
  3. `ctrlProyecto : CtrlProyecto` *(Control)*
  4. `proyecto : Proyecto` *(Entidad)*
* **Secuencia de mensajes y orden de ejecución:**
  1. El `promotor` envía `capturarDatosProyecto(nombre, descripcion, fechaInicio, fechaFin)` hacia `vistaProyecto`.
  2. `vistaProyecto` delega la solicitud al controlador mediante `solicitarRegistroProyecto(datos)`.
  3. `ctrlProyecto` invoca la operación de verificación `validarFechas(fechaInicio, fechaFin)` sobre `proyecto`.
  4. **Fragmento combinado `alt` (Validación de fechas y campos obligatorios):**
     * **Guarda `[fechaFin < fechaInicio || nombre == ""]`:** `proyecto` retorna `false`; `ctrlProyecto` envía `mostrarErrorValidacion("Verifique las fechas o campos obligatorios")` hacia `vistaProyecto`, la cual notifica al `promotor` mediante mensaje punteado de respuesta.
     * **Guarda `[else]`:** `ctrlProyecto` ejecuta `registrar(nombre, fechaInicio)` sobre `proyecto`, persiste el registro localmente en SQLite, recibe confirmación `true` y responde a `vistaProyecto` con `notificarRegistroExitoso()`.

![Diagrama de Secuencia CU-01 - Registrar Proyecto](secuencia-cu01-proyecto.png)
*Figura 2. Diagrama de secuencia para el caso de uso `CU-01: Registrar Proyecto Agroecológico`.*

---

### 2.2 Escenario 2: Captura de Obra de Conservación y Sincronización Offline (`CU-02`)
* **Requisitos trazados:** `RF-02`, `RF-06`, `RF-10`, `RNF-01`.
* **Líneas de vida participantes:**
  1. `promotor : PromotorComunitario` *(Actor)*
  2. `vistaObra : FrmObraConservacion` *(Frontera)*
  3. `ctrlObra : CtrlObra` *(Control)*
  4. `obra : ObraConservacion` *(Entidad)*
* **Secuencia de mensajes y orden de ejecución:**
  1. El `promotor` ingresa los datos físicos de la intervención (`terrazas`, `zanja_infiltracion`, `bordo`, `presa_gaviones`) en `vistaObra` mediante `registrarDatosObra(tipo, ubicacionReferencia, responsableTecnico)`.
  2. `vistaObra` invoca `procesarNuevaObra(datos)` en `ctrlObra`.
  3. `ctrlObra` llama a `crear(tipo, ubicacionReferencia)` en la entidad `obra`, almacenando el registro localmente.
  4. **Fragmento combinado `loop` (Adjuntar evidencias de ubicación):**
     * **Guarda `[existenReferenciasMapa == true]`:** `ctrlObra` invoca `adjuntarReferenciaEspacial(rutaImagen)` repetidamente hasta completar las referencias ligeras de campo.
  5. **Fragmento combinado `opt` (Sincronización diferida según conectividad):**
     * **Guarda `[conexionInternet == true]`:** `ctrlObra` ejecuta `sincronizarOffline()` sobre `obra` para respaldar el registro en el servidor institucional sin bloquear la operación en campo.
  6. `vistaObra` devuelve confirmación visual de guardado exitoso al `promotor`.

![Diagrama de Secuencia CU-02 - Registrar Obra de Conservación](secuencia-cu02-obra.png)
*Figura 3. Diagrama de secuencia para el caso de uso `CU-02: Registrar Obra de Conservación de Suelo y Agua`.*

---

## 3. Consistencia con el Diagrama de Clases

Para garantizar la integridad del modelo de análisis entre la Práctica 9 y la Práctica 10, se auditó cada línea de vida de entidad y cada mensaje recibido en los diagramas de secuencia contra el diagrama de clases en `modelo/OpenHub-Equipo.qea`.

### 3.1 Actualizaciones Incorporadas al Diagrama de Clases
Todas las operaciones invocadas sobre las entidades en los diagramas de secuencia existen ahora explícitamente en sus respectivas clases con visibilidad, parámetros y tipos de retorno:

| Clase Actualizada | Operación Agregada / Verificada | Tipo de Retorno | Justificación desde los Diagramas de Secuencia |
| :--- | :--- | :--- | :--- |
| **`Proyecto`** | `+ validarFechas(fInicio: Date, fFin: Date)` | `Boolean` | Requerida por el fragmento `alt` en `CU-01` para validar la coherencia cronológica antes de persistir el proyecto. |
| **`Proyecto`** | `+ registrar(nombre: String, fInicio: Date)` | `Boolean` | Confirmada en la rama `[else]` del escenario `CU-01`. |
| **`ObraConservacion`** | `+ adjuntarReferenciaEspacial(ruta: String)` | `Boolean` | Incorporada para soportar el bloque repetitivo `loop` de evidencias de campo en `CU-02`. |
| **`ObraConservacion`** | `+ sincronizarOffline()` | `Boolean` | Agregada para ejecutar la sincronización condicionada dentro del fragmento `opt` en `CU-02`. |

---

## 4. Modelo de Arquitectura en UML

La estructura lógica del sistema SIGIPA-CEDICAM se organizó mediante un **Diagrama de Paquetes de 4 Capas** en Enterprise Architect, respetando estrictamente el principio de **dependencia unidireccional hacia abajo** (ninguna capa inferior depende de una superior ni se saltan niveles intermedios):

```
+-------------------------------------------------------+
|                1. CAPA DE PRESENTACIÓN                |
|   FrmRegistroProyecto, FrmObraConservacion, Vistas    |
+-------------------------------------------------------+
                           |  «use»
                           v
+-------------------------------------------------------+
|              2. CAPA DE APLICACIÓN / API              |
|        CtrlProyecto, CtrlObra, Servicios de Caso      |
+-------------------------------------------------------+
                           |  «use»
                           v
+-------------------------------------------------------+
|                  3. CAPA DE DOMINIO                   |
| Proyecto, Comunidad, Participante, ObraConservacion   |
+-------------------------------------------------------+
                           |  «use»
                           v
+-------------------------------------------------------+
|            4. CAPA DE INTEGRACIÓN / DATOS             |
|   SQLiteLocalAdapter [REAL], SyncExportAPI [SIMULADO] |
+-------------------------------------------------------+
```

### 4.1 Clasificación de Componentes Reales y Simulados
* **Componentes Reales:** La persistencia local en base de datos ligera **SQLite (`SQLiteLocalAdapter`)**, las vistas móviles de captura y la lógica de validación de dominio están implementadas de forma **real** para operar sin conexión a internet en campo (`RNF-01`).
* **Componentes Simulados:** El conector de sincronización remota en la nube y el motor de generación automática de reportes consolidados hacia donantes (`SyncExportAPI`) se modelan e integran como componentes **simulados** mediante archivos locales CSV/PDF en esta etapa del Sprint 4.

![Diagrama de Arquitectura de Paquetes - SIGIPA-CEDICAM](arquitectura-capas.png)
*Figura 4. Diagrama de paquetes en UML mostrando las 4 capas arquitectónicas, asignación de clases y dependencias descendentes.*

---

## 5. Integración de Negocio y Requisitos (Trazabilidad)

El siguiente mapa de trazabilidad vincula los procesos de negocio BPMN (Unidad 2), los requisitos del DRS v1.0 (Unidad 3), los casos de uso (Actividad 3), las clases del dominio y las secuencias dinámicas:

| Proceso BPMN (Unidad 2) | Requisitos DRS | Caso de Uso | Clases del Dominio / Análisis | Diagrama de Secuencia | Capa Arquitectónica |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Gestión de proyectos agroecológicos** | `RF-01`, `RF-06`, `RNF-01` | `CU-01: Registrar Proyecto` | `Proyecto`, `Comunidad`, `Participante` | `secuencia-cu01-proyecto` | Presentación, Aplicación, Dominio, Integración |
| **Conservación de suelo y agua** | `RF-02`, `RF-06`, `RF-10` | `CU-02: Registrar Obra` | `ObraConservacion`, `Proyecto`, `Comunidad` | `secuencia-cu02-obra` | Presentación, Aplicación, Dominio, Integración |
| **Registro de padrón y líderes** | `RF-01`, `RF-04`, `RF-08` | `CU-03: Alta de Participante` | `Participante`, `Comunidad`, `Proyecto` | `secuencia-cu03-participante` | Presentación, Aplicación, Dominio |
| **Consulta histórica y territorial** | `RF-05`, `RF-07`, `RF-09`, `RNF-07` | `CU-05: Consultar Historial` | `Comunidad`, `Capacitacion`, `TecnicaAgroecologica` | `secuencia-cu05-historial` | Presentación, Aplicación, Dominio |

---

## 6. Presentación y Validación con la Organización (Minuta)

### Minuta de Sesión de Validación Operativa
* **Fecha y Duración:** 15 de noviembre de 2026 · Duración: 10 minutos.
* **Modalidad:** Sesión virtual de presentación con prototipo visual y recorrido de escenarios en lenguaje cotidiano de la organización.
* **Asistentes:**
  * **Por la Organización:** Coordinador Técnico de CEDICAM *(se omite nombre completo para proteger datos personales en repositorio público)*.
  * **Por el Equipo de Desarrollo (TecNM Campus Tlaxiaco):** Irwing Sael Feria Caballero (eduScrum Master), Evelyn Sosa José, Ebenezer Acevedo Nicolas y Jazmin Aguilar Rojas.

### Preguntas de Validación y Respuestas de la Organización
1. **¿Los nombres utilizados en el modelo (`Comunidad`, `ObraConservacion`, `Participante`, `Proyecto`) coinciden con la forma en que los promotores registran la información en campo?**
   * *Respuesta:* Sí, son exactamente los términos que usamos en nuestras bitácoras comunitarias y reportes técnicos; cualquier promotor los entiende sin explicación adicional.
2. **En el flujo de captura de una obra de conservación, ¿qué sucede cuando el promotor está en una localidad sin señal de internet?**
   * *Respuesta:* El sistema debe guardar primero toda la información en el teléfono y permitir sincronizarla después cuando el promotor regrese a la oficina o tenga señal.
3. **¿Falta algún dato indispensable en el registro de obras o proyectos para rendir cuentas a los comités comunitarios?**
   * *Respuesta:* Es muy importante que quede registrado explícitamente quién es el `responsableTecnico` que supervisó la obra en cada comunidad.

### Cambios Incorporados tras la Validación y Constancia de Aprobación
* **Ajuste aplicado:** Se confirmó e integró el atributo `responsableTecnico` en `ObraConservacion` dentro del modelo de dominio y en los parámetros del mensaje `registrarDatosObra()` del diagrama de secuencia `CU-02`. Asimismo, se ratificó el fragmento `opt [conexionInternet == true]` para garantizar el guardado local prioritario.
* **Estado de aprobación:** El Coordinador Técnico de CEDICAM validó y aprobó el modelo integrado al confirmar que refleja fielmente la operación de campo de la organización.

---

## 7. Entrega, Revisión Cruzada y Trabajo en Equipo

Todos los diagramas (Modelo de Dominio, Diagramas de Secuencia `CU-01` y `CU-02`, Diagrama de Clases actualizado y Diagrama de Arquitectura de 4 Capas) residen unificados dentro del archivo **`modelo/OpenHub-Equipo.qea`** y exportados en alta resolución en **`docs/analisis/`**.

### 7.1 Lista de Consistencia Aplicada por el Equipo Revisor Par

| Regla de Consistencia Verificada | Estado | Observación y Respuesta en el Pull Request |
| :--- | :---: | :--- |
| Cada línea de vida de entidad es una clase del dominio o del diagrama de clases. | ✅ | Verificado: `Proyecto` y `ObraConservacion` coinciden exactamente con el modelo de dominio y clases. |
| Cada mensaje que recibe un objeto existe como operación en su clase. | ✅ | Corregido en PR: Se agregaron `validarFechas()`, `adjuntarReferenciaEspacial()` y `sincronizarOffline()` al diagrama de clases. |
| Cada diagrama de secuencia sigue la especificación de un caso de uso. | ✅ | Verificado: `CU-01` y `CU-02` traducen paso a paso el flujo principal y alterno de la Actividad 3. |
| El orden de los mensajes coincide con el BPMN mejorado de la Unidad 2. | ✅ | Verificado: La secuencia de captura, validación y almacenamiento respeta el flujo operativo del BPMN. |
| Los nombres del dominio son los del glosario de la organización. | ✅ | Verificado: Se emplea vocabulario de CEDICAM sin clases técnicas en el modelo de dominio. |
| Cada elemento conserva su requisito de origen (`RF`, `RNF`, `RD`). | ✅ | Verificado: Todos los diagramas y tablas citan sus requisitos del DRS v1.0. |

### 7.2 Participación Verificable por Pull Request (eduScrum)

| Integrante | Contribución como Autor (PR) | Revisión Cruzada Realizada (PR) |
| :--- | :--- | :--- |
| **Evelyn Sosa José** | Creación del paquete `«Modelo de dominio»` y reglas de asociación en `.qea`. | Revisión de estructura Frontera-Control-Entidad en `CU-01` y `CU-02`. |
| **Irwing Sael Feria Caballero** | Elaboración de diagramas de secuencia `CU-01` y `CU-02` con fragmentos `alt`/`opt`/`loop` y minuta. | Revisión de arquitectura de 4 capas y dependencias descendentes. |
| **Ebenezer Acevedo Nicolas** | Construcción del diagrama de paquetes de arquitectura y actualización del diagrama de clases. | Verificación de consistencia entre mensajes de secuencia y operaciones de clase. |
| **Jazmin Aguilar Rojas** | Consolidación del documento integrador, matriz de trazabilidad y exportación de PNGs. | Revisión del modelo de dominio contra el glosario de CEDICAM y BPMN. |