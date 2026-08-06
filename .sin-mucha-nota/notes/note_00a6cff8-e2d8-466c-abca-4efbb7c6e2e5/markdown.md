# BPAY-2: Dashboard de monitoreo Bendo

## Información General

| Campo | Valor |
|-------|-------|
| **ID** | BPAY-2 |
| **Tipo** | Story |
| **Estado** | TO DO |
| **Prioridad** | Medium |
| **Asignado a** | David Cuaical |
| **Reportado por** | Karla Altamirano |
| **Fecha de vencimiento** | 2026-08-21 |
| **Proyecto** | B Pay \| Corporativo |
| **Creado** | 2026-08-05 17:57:37 |
| **Actualizado** | 2026-08-06 12:56:14 |

---

## Descripción

**Como** usuario de negocio, operaciones, cumplimiento y financiero  
**Quiero** un tablero que muestre con el máximo nivel de detalle el estado de cada transacción y archivo a lo largo de todo el flujo (conciliación → liquidación → dispersión → facturación), diferenciado por frecuencia de pago, tipo de consumo y motivo de retraso  
**Para** poder auditar diariamente si se cumple el ANS/SLA por comercio, identificar exactamente dónde y por qué se detiene una transacción, y actuar antes de que impacte al PAC (Prestador de Servicios de Pago) o al negocio.

---

## 0. Reglas Transversales (Aplican a todo el tablero)

1. **Eliminar el indicador "Compensación a comercios"** como módulo independiente; su contenido se reubica dentro de "Conciliación de comercios".

2. **Búsqueda por Número de autorización**: se debe incorporar la búsqueda por número de autorización para que se realice la búsqueda de toda la trazabilidad de la transacción (aplica para toda la logica del tablero que se encuentra a nivel transaccional)

3. **Filtro de modelo**: agregar switch/selector para alternar entre **Modelo Agregador** y **Dispositivos**. Todo el tablero (los 4 módulos) debe poder consultarse bajo ambos modelos, de forma independiente (no sumados).

4. **Una sola frecuencia de pago (ANS) por comercio** a la vez, configurada en Backoffice. Esa frecuencia debe ser consistente y trazable en TODOS los procesos (conciliación, liquidación, dispersión, facturación). Ningún indicador puede mostrar a un comercio con dos frecuencias distintas simultáneamente (es decir el misma frecuencia de pago que tiene el comercio parametrizado en backoffice debe concindicir, con nombres de archivos txt o cualquier parte del proceso donde se use ese campo en gestor bendo).

5. **Corte de conciliación diaria: 19:30h.**
   - Transacción con fecha de disponibilidad **antes** de las 19:30 → cuenta dentro del ANS del día (ej. 24h = mismo día).
   - Transacción **después** de las 19:30 → NO debe sumar un día adicional a la frecuencia de pago real del comercio; debe generarse en un archivo con etiqueta diferenciada, ej. **"[Frecuencia]D - Nocturno"** (ej. "1D Nocturno"), pero conservando la frecuencia de pago original del comercio.

6. **Nombres de archivos TXT** deben reflejar únicamente frecuencias de pago que existan realmente configuradas en Backoffice (no deben aparecer "2D" o "4D" si no hay comercios con esa frecuencia habilitada). Esto aplica para todos los campos en donde se considere este dato no solo para los nombres de los archivos txt.

7. **Corte diario de disponibilidad de datos: ~13:00h** (hora referencial, ajustable). Si el usuario consulta el tablero con filtro "Hoy" antes de ese corte y no hay datos, **no debe considerarse una alerta/error**; debe mostrarse una nota informativa fija: _"El corte diario de información es a las 13:00h aproximadamente. Si no visualiza datos, el proceso aún no ha finalizado."_

8. **Umbrales de riesgo (a definir/confirmar con negocio, propuesta inicial):**

| Nivel | Regla propuesta | Color | Acción |
|-------|-----------------|-------|--------|
| Normal | No conciliados ≤ 5% del total del día, dentro de ANS vigente | Verde | Ninguna |
| En atención | No conciliados entre >5% y ≤10%, o ANS 24h vencido 1 día | Amarillo | Notificación interna (correo), sin escalar |
| Crítico | No conciliados > 10%, o ANS 24h vencido ≥2 días, o **no llegó ningún archivo SFTP del día** | Rojo | Alerta visual + notificación automática (correo) |

9. **Leyenda fija visible en el tablero** (tooltip o pie de página):
   - _Conciliado_: la transacción existe en el archivo de Diners, existe en Geopagos y está registrada en Backoffice.
   - _No conciliado_: falta en al menos uno de los tres orígenes (Diners, Geopagos, Backoffice); puede deberse a que aún no se ha recibido el pago o no está en el estado de cuenta.

10. Todas las tarjetas/indicadores numéricos del tablero deben recalcularse en función de los filtros seleccionados en la parte superior (rango de fechas, modelo, frecuencia de pago).

---

## 1. Módulo: Conciliación de comercios

### 1.1 Indicadores principales

- **Conciliado** (número total de transacciones/comercios conciliados en el periodo filtrado).
- **No conciliado** (número total de transacciones/comercios no conciliados en el periodo filtrado).

### 1.2 Desagregación obligatoria (2 niveles de agrupación) — para AMBOS indicadores

1. **Nivel 1 – Fecha de disponibilidad, plan de pagos comercios**
2. **Nivel 2 – Tipo de consumo:** Presente / No presente.

Ejemplo de desagregación esperada al hacer clic en "No conciliado":

```
No conciliado: 1,133

 ├─ plan de pagos 1
 │    ├─ Consumo presente: XX
 │    └─ Consumo no presente: XX
 ├─ plan de pagos 2
 │    ├─ Consumo presente: XX
 │    └─ Consumo no presente: XX
 └─ plan de pagos 3
      ├─ Consumo presente: XX
      └─ Consumo no presente: XX
```

### 1.3 Días de vencimiento de ANS (para No conciliados) — detalle obligatorio, no opcional

- Cada transacción/agrupación "No conciliada" debe mostrar **cuántos días lleva vencido su ANS específico**, calculado como: fecha actual – fecha límite de disponibilidad según su frecuencia de pago.
- Solo las de **24 horas** vencidas cuentan como **incumplimiento de SLA / impacto directo al PAC** y deben marcarse con alerta crítica (rojo) desde el día 1 de vencimiento.
- Las de **48h y 72h** aún no vencidas dentro de su propio plazo **NO deben mostrarse como "fuera de ANS"**; solo pasan a alerta cuando superan su propio plazo (48h → día 3+, 72h → día 4+).
- Columna sugerida en la tabla de detalle: Días vencido ANS con semaforización:
  - 0 días (dentro de plazo): sin color / gris.
  - 1 día vencido (solo aplica a 24h): amarillo.
  - 2+ días vencido: rojo.
  - Regla equivalente debe aplicarse a 48h (vencido desde el día 3) y 72h (vencido desde el día 4), replicando la misma lógica de semaforización.

### 1.4 No conciliados: detalle exhaustivo

Cada registro dentro de "No conciliado" debe permitir ver, como mínimo:

- Comercio (RUC/razón social).
- Frecuencia de pago (ANS) asignada.
- Tipo de consumo (presente/no presente).
- Fecha de la transacción original.
- Fecha límite de disponibilidad (según su ANS).
- Días vencido de ANS (ver 1.3).
- Motivo de no conciliación: _no existe en archivo Diners_ / _no existe en Backoffice_ / _no existe en estado de cuenta_ / _pendiente ambos_.
- Estado: _Aún no se recibe pago_ / _Aún no está en estado de cuenta_ (marcado con asterisco o ícono informativo).
- Indicador de si fue **regularizado** posteriormente (ver 1.5) y en qué fecha.

### 1.5 Regularizaciones (trazabilidad histórica)

- Cuando un registro "No conciliado" pasa a "Conciliado" en un día posterior, el tablero debe registrar la fecha real en que se regularizó (no solo restarlo del contador del día siguiente).
- El gráfico histórico (ver 1.6) debe permitir distinguir, dentro de "No conciliados", cuáles ya fueron regularizados posteriormente vs. cuáles siguen pendientes a la fecha de consulta.

### 1.6 Gráfico histórico (últimos 30 días)

- Debe mostrar **dos series separadas**: Conciliados y No conciliados (actualmente solo muestra una).
- Debe incluir un filtro/toggle para ver una serie, la otra, o ambas superpuestas.
- Al hacer clic sobre un punto del gráfico de "No conciliados" de un día pasado, debe desplegar el detalle de cuántos de esos casos ya fueron regularizados y en qué fecha.

### 1.7 Tracking/embudo hacia el siguiente proceso

- Debe mostrarse, para el periodo filtrado: Total transacciones recibidas → No conciliadas (quedan retenidas aquí) → Conciliadas (avanzan a Liquidación de valores).
- Ejemplo real mencionado en reunión: de 1,341 no conciliadas, solo 8,700 (aprox.) avanzan al siguiente proceso — este tipo de resumen numérico debe quedar visible como parte de este módulo (no en Liquidación).

### 1.8 Filtros del módulo

- Rango de fechas (default: Hoy).
- Modelo: Agregador / Dispositivos.
- Frecuencia de pago: 24h / 48h / 72h / Todas.
- Tipo de consumo: Presente / No presente / Todos.

---

## 2. Módulo: Liquidación de valores

### 2.1 Cambio de nomenclatura

- Renombrar el estado actual **"Liquidado"** → **"Generado en TXT"** (evita confusión con el estado final de pago).

### 2.2 Indicadores principales

- **Generado en TXT**: total de transacciones que forman parte del archivo TXT a liquidar.
- **Generado en TXT pero no pagado**: transacciones cuyo comercio tiene pendiente de validar el certificado bancario (aún no se les paga aunque ya estén en el TXT).
- **No generado en TXT** (renombrar desde "no liquidado"): transacciones conciliadas que, por alguna causa, no entraron al TXT del día.

### 2.3 Detalle obligatorio de "No generado en TXT" — por motivo

Debe existir un desglose exhaustivo por causa, cada una con su propio contador:

| Motivo | Descripción |
|--------|-------------|
| Sin validación de certificado bancario | Institución financiera nueva o certificado no regularizado |
| Retención de fondos | Transacción retenida por proceso de monitoreo/control |
| Monitoreo de fraude | Con su propio ANS de retención (ej. "vencida 3 días") — **debe mostrar días vencido igual que en el módulo 1** |
| Contracargos | Con su propio ANS de retención (ej. "vencida 50 días") — **debe mostrar días vencido** |
| Solicitud de débito | Transacción con débito asociado pendiente |
| No conciliado | No pasó el filtro de conciliación del módulo 1 (enlace directo al detalle de ese módulo) |

**Nota:** Retenciones de fondos, monitoreo de fraude y contracargos manejan **su propio ANS de retención**, independiente del ANS de la frecuencia de pago. Debe replicarse aquí la misma lógica de "días vencido" del punto 1.3, pero calculada contra el ANS de retención correspondiente.

### 2.4 Filtro previo obligatorio

- Todo registro "No conciliado" (módulo 1) debe excluirse automáticamente del flujo de Liquidación (no debe aparecer contabilizado ni en "Generado en TXT" ni en "No generado en TXT" por otro motivo distinto a "No conciliado").

### 2.5 Filtros del módulo

- Rango de fechas, Modelo (Agregador/Dispositivos), Frecuencia de pago, Motivo de no generación en TXT.

---

## 3. Módulo: Dispersión de fondos

### 3.1 Switch de vistas (obligatorio)

Un control tipo switch en la parte superior con dos opciones:

- **Vista Transaccional** (vista por defecto al entrar al módulo).
- **Vista por Pagos/Archivo**.

### 3.2 Vista Transaccional (default)

Para cada transacción que pasó de Liquidación:

- Archivo TXT en el que fue incluida.
- Estado del archivo en el banco: No existe en banco / Iniciado (cargado, no aprobado) / Aprobado (en tránsito, esperando confirmación — hasta 72h en operaciones interbancarias) / Procesado (banco confirmó todo el lote).
- No debe mostrar cantidad de pagos agregados (eso es exclusivo de la vista por archivo).

### 3.3 Vista por Pagos/Archivo

Estructura de navegación en 3 niveles (drill-down):

1. **Archivo TXT generado** (ej. "Archivo 3D — 244 comercios").
2. Clic → **Comercios incluidos en ese archivo** con su monto total a pagar cada uno.
3. Clic sobre un comercio → **Detalle de las liquidaciones/transacciones individuales** que componen ese pago consolidado.

### 3.4 Monitor de pagos (sección separada dentro del módulo)

- Tabla por archivo con: fecha de carga, fecha de generación, estado en banco.
- Debe diferenciar "pagos sin confirmar del mismo banco" vs "pagos sin confirmar de otros bancos" (con sus respectivas cantidades), tal como se mencionó en la reunión (ej. "mismo banco: 0, otros bancos: 100, 15, 5, 29").

### 3.5 Notificaciones — indicador independiente

- **No debe mezclarse** "pago confirmado" con "pago notificado". Deben ser dos estados separados:
  - Pago confirmado / no confirmado (banco).
  - Notificación enviada / pendiente (puede encolarse sin que signifique que el pago no se realizó).

### 3.6 Búsqueda puntual

- Buscador combinable por **RUC** y/o **número de autorización**:
  - Solo RUC → muestra todas las transacciones de ese comercio.
  - RUC + autorización → muestra el detalle específico de esa transacción y su estado a lo largo de toda la cadena (conciliación → liquidación → dispersión).

### 3.7 Transacciones rebotadas (embebidas en este módulo, no como sección aparte)

- Motivos a listar: cédula incorrecta, RUC incorrecto, cuenta inactiva, cuenta bloqueada, u otro motivo de rechazo bancario.
- Estado de gestión: **"Sin gestión"** (rebotó hace X días y operaciones/negocio no ha actuado — debe mostrar días transcurridos desde el rebote) vs **"Regularizado"** (ya se generó un nuevo TXT tras resolver la novedad).
- El indicador final de Dispersión debe sumar: Dispersado (confirmado) + Pendiente por dispersar (en proceso de confirmación) + Rebotado (sin gestión / regularizado) = total que debía dispersarse.

### 3.8 Dispersión por retención de fondos / solicitudes de débito liberadas

- Debe existir un sub-apartado separado del flujo normal para las transacciones que provienen de una retención de fondos ya liberada.
- Estas manejan **su propio ANS**, distinto al ANS de la frecuencia de pago estándar, y **no deben afectar ni mezclarse con el cálculo del SLA normal**.

### 3.9 Filtros del módulo

- Rango de fechas, Modelo, Frecuencia de pago, Estado bancario del archivo, Tipo (flujo normal / retención liberada).

---

## 4. Módulo: Facturación y conciliación final

### 4.1 Base de cálculo

- Este módulo debe alimentarse **únicamente** de transacciones ya **dispersadas y confirmadas** (módulo 3), NO del total de liquidadas (módulo 2). Esto corrige la inconsistencia detectada en la reunión, donde el total no cuadraba porque se incluían no dispersadas.

### 4.2 Cortes mensuales (3 cortes — validar fechas exactas con Ana antes de desarrollo)

- Corte 1: día 15 de cada mes.
- Corte 2: día 28 (o fin de mes).
- Corte 3: primeros días del mes siguiente (corresponde al cierre del mes anterior).

### 4.3 Indicadores por corte

Para cada uno de los 3 cortes, mostrar:

- Cantidad de **agrupadores generados** vs **no generados**.
- Cantidad de **facturas de comisión** generadas.
- Cantidad de **comprobantes de retención** generados.
- Cantidad de **bajas** (asiento contable / reconciliación interna) generadas.

### 4.4 Indicador de cuadre (obligatorio)

- Comparar, por corte y rango de fechas:
  - **Monto esperado** = suma de comisiones de todas las transacciones dispersadas y confirmadas en ese rango (según comprobante de liquidación).
  - **Monto facturado** = suma real de las facturas de comisión generadas.
  - Repetir el mismo cuadre para **IVA/retención** (monto esperado según comprobante de liquidación vs. comprobantes de retención generados).
- Si el cuadre no coincide, debe permitirse navegar al detalle (aunque se reconoce que el nivel transaccional puede ser pesado de visualizar; priorizar el cuadre a nivel de monto total primero).

### 4.5 Filtros del módulo

- Corte (1, 2, 3), rango de fechas, modelo.

---

## 5. Diagrama de flujo general (representación visual en el tablero)

Debe corregirse el diagrama actual (las actividades mostradas hoy están mal descritas) y representar dos líneas de flujo:

**Línea principal:** Conciliación de comercios → Liquidación de valores → Dispersión de fondos (con "Transacciones rebotadas" embebido dentro de Dispersión, no como caja independiente).

**Línea secundaria (rama):** Liquidación de valores → Facturación y conciliación final (debe visualizarse claramente que esta rama NO depende de Dispersión de fondos, sino directamente de Liquidación).

Las actividades del flujo (texto descriptivo debajo de cada caja) deben simplificarse a alto nivel, ejemplo:

- Envío de archivo de capturas.
- Conciliación por Diners.
- Depósito de archivo de liquidación en SFTP.
- Generación de TXT.
- Carga y aprobación en banco.
- Confirmación/dispersión.
- Facturación y conciliación final.

---

## Subtareas

Esta historia de usuario tiene 11 subtareas asociadas:

| ID | Resumen | Estado | Prioridad |
|----|---------|--------|-----------|
| BPAY-3 | [Elaboración Convenio ARQ - INFRA] Dashboard de monitoreo Bendo | TO DO | Medium |
| BPAY-4 | [Planificación Desarrollo] Dashboard de monitoreo Bendo | TO DO | Medium |
| BPAY-5 | [Desarrollo Front] Dashboard de monitoreo Bendo | TO DO | Medium |
| BPAY-6 | [Desarrollo Back] Dashboard de monitoreo Bendo | TO DO | Medium |
| BPAY-7 | [Ejecución Pruebas QA] Dashboard de monitoreo Bendo | TO DO | Medium |
| BPAY-8 | [Corrección Pruebas QA] Dashboard de monitoreo Bendo | TO DO | Medium |
| BPAY-9 | [Ejecución Pruebas SGI-QA] Dashboard de monitoreo Bendo | TO DO | Medium |
| BPAY-10 | [Corrección Pruebas SGI] Dashboard de monitoreo Bendo | TO DO | Medium |
| BPAY-11 | [Pruebas UAT] Dashboard de monitoreo Bendo | TO DO | Medium |
| BPAY-12 | [Despliegue a Producción] Dashboard de monitoreo Bendo | TO DO | Medium |
| BPAY-13 | [Ejecución Pruebas SGI-PROD] Dashboard de monitoreo Bendo | TO DO | Medium |

---

## Comentarios

**Autor:** Karla Altamirano  
**Fecha:** 2026-08-06 12:56:14

> @David Cuaical por favor tu ayuda estimando esta HU, gracias.