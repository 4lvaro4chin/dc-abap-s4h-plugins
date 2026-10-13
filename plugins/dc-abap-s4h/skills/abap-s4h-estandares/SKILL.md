---
name: abap-s4h-estandares
description: Estándares de programación ABAP Digital Core para SAP S/4HANA — estructura modular de includes, patrón OO con clase local, sintaxis ABAP moderna (7.50+), catálogo de textos, cabecera, performance sobre HANA, manejo de errores, seguridad, clean core y regla de verificación de objetos SAP estándar. Usar siempre antes de escribir o revisar código ABAP para S/4HANA (reports, clases, ampliaciones, cargas).
---

# Estándares de programación ABAP — SAP S/4HANA

Aplica a S/4HANA on-premise y Private Cloud (1610 a 2023). Para nombres de objetos y variables usar la skill `abap-s4h-nomenclatura`.

## 1. Estructura modular de un report (obligatoria)

Ningún report se genera en un solo archivo. Cada include es un archivo independiente en `docs/[ID]/Code/`.

**Orden de los `INCLUDE` en el programa principal:**

```abap
REPORT zdc[mod]_[nombre].

INCLUDE zdc[mod]_[nombre]_top.  " Tipos, constantes y estructura de referencia
INCLUDE zdc[mod]_[nombre]_def.  " Definición de clases locales
INCLUDE zdc[mod]_[nombre]_imp.  " Implementación de clases locales
INCLUDE zdc[mod]_[nombre]_sel.  " Pantalla de selección
INCLUDE zdc[mod]_[nombre]_mai.  " Eventos
INCLUDE zdc[mod]_[nombre]_f01.  " Stub vacío
INCLUDE zdc[mod]_[nombre]_txt.  " Catálogo de textos (solo comentarios)
```

- La clase va antes de `_SEL` y `_MAI` porque `_SEL` llama a sus métodos estáticos (F4) y `_MAI` la instancia.
- El programa principal solo contiene la cabecera, `REPORT` y los `INCLUDE`.
- `_F01` existe para mantener la estructura estándar, pero queda vacío: **no se usan `FORM`/`PERFORM`**.

## 2. Patrón OO del report (obligatorio)

Toda la lógica vive en una clase local con instancia y constructor.

**`_TOP`** — solo la estructura de referencia para los `SELECT-OPTIONS` y tipos compartidos. Sin datos de negocio:

```abap
TYPES: BEGIN OF ty_selec,
         werks TYPE werks_d,
         matnr TYPE matnr,
       END OF ty_selec.

DATA: gs_selec TYPE ty_selec.
```

**`_SEL`** — los `SELECT-OPTIONS` usan los componentes de `gs_selec`:

```abap
SELECT-OPTIONS: s_werks FOR gs_selec-werks,
                s_matnr FOR gs_selec-matnr.
```

**`_DEF`** — constructor que recibe la pantalla; datos en la sección privada:

```abap
CLASS lcl_reporte DEFINITION.
  PUBLIC SECTION.
    TYPES: ty_werks_rng TYPE RANGE OF werks_d,
           ty_matnr_rng TYPE RANGE OF matnr.

    METHODS constructor
      IMPORTING it_werks_rng TYPE ty_werks_rng OPTIONAL
                it_matnr_rng TYPE ty_matnr_rng OPTIONAL.
    METHODS ejecutar.

  PRIVATE SECTION.
    DATA: mt_werks_rng TYPE ty_werks_rng,
          mt_matnr_rng TYPE ty_matnr_rng,
          mt_output    TYPE STANDARD TABLE OF ... WITH EMPTY KEY.
ENDCLASS.
```

**`_MAI`** — instancia con `NEW` y delega:

```abap
START-OF-SELECTION.
  DATA(go_reporte) = NEW lcl_reporte( it_werks_rng = s_werks[]
                                      it_matnr_rng = s_matnr[] ).
  go_reporte->ejecutar( ).
```

Reglas del patrón:

1. Métodos de instancia para toda la lógica de negocio. `CLASS-METHODS` solo para utilitarios sin estado (F4, validaciones previas a instanciar).
2. La clase nunca lee `s_xxx` / `p_xxx` directamente: los recibe por el constructor.
3. No hay variables globales de negocio (`gt_*`, `gv_*`). La única global es `gs_selec` (y la referencia al objeto en `_MAI`).
4. Métodos cortos y con una sola responsabilidad; separar lectura de datos, lógica y presentación.
5. Los event handlers son métodos de la propia clase (`SET HANDLER me->on_...`).
6. Para ALV, usar el patrón completo de la skill `abap-s4h-alv-factory`. Para la pantalla, la skill `abap-s4h-selection-screen`.

## 3. Sintaxis ABAP moderna

En S/4HANA se usa sintaxis 7.50+ por defecto. Preferir:

| Usar | En lugar de |
|---|---|
| `DATA(lv_x) = ...`, `FIELD-SYMBOL(<fs>)` inline | Bloques `DATA` / `FIELD-SYMBOLS` separados sin necesidad |
| `NEW lcl_x( )` | `CREATE OBJECT` |
| `VALUE #( )`, `CORRESPONDING #( )`, `CONV #( )` | Asignaciones campo a campo |
| `COND #( )`, `SWITCH #( )` | `IF`/`CASE` solo para asignar un valor |
| String templates `|{ lv_a } { lv_b }|` | `CONCATENATE` |
| `line_exists( )`, `line_index( )`, `lt_tab[ key = ... ]` | `READ TABLE ... TRANSPORTING NO FIELDS` |
| `FOR ... IN` y `REDUCE` para construir tablas | `LOOP` + `APPEND` triviales |
| Open SQL estricto: lista con comas y host variables con `@` | Sintaxis sin `@` |
| `SELECT ... INTO TABLE @DATA(lt_x)` | Tipos declarados solo para el `SELECT` |
| `TYPE STANDARD TABLE OF ... WITH EMPTY KEY` | `WITH DEFAULT KEY` en tablas nuevas |

**Prohibido en S/4HANA:**

- `TABLES`, `OCCURS`, `WITH HEADER LINE`, `MOVE ... TO`, `COMPUTE`, `FORM`/`PERFORM`.
- `CALL TRANSACTION ... USING` (BDC) para crear o modificar datos de negocio. Usar BAPI o API liberada.
- SAPScript para formularios nuevos.
- Leer o escribir tablas eliminadas por la simplificación de S/4HANA (ver [references/simplificacion-s4h.md](references/simplificacion-s4h.md)).
- `UPDATE` / `INSERT` / `MODIFY` directo sobre tablas estándar SAP.
- Modificaciones al estándar (claves de acceso) sin aprobación escrita del arquitecto.
- Literales en el código para textos visibles al usuario.

## 4. Textos y mensajes

**En reports:** todos los mensajes se definen como elementos de texto `text-mNN` y se usan con `MESSAGE text-mNN TYPE 'S'/'W'/'E'`. El report no lleva `MESSAGE-ID` y no se crea clase de mensajes para él.

**En clases globales, RAP y ampliaciones:** usar clase de mensajes `ZDC_[MOD]` (SE91 / ADT), porque `T100` es lo que esperan `BAPIRET2`, los logs de aplicación y el framework RAP.

**Include `_TXT` (obligatorio en reports):** documenta, como comentarios, todo lo que hay que crear a mano en SE38 → *Elementos de texto*:

```abap
*======================================================================*
*  ELEMENTOS DE TEXTO (SE38 -> Ir a -> Elementos de texto)
*======================================================================*
*  ID     | Texto
*  -------|------------------------------------------------------------
*  B01    | Criterios de selección
*  M01    | No se encontraron registros con los criterios indicados
*
*======================================================================*
*  TEXTOS DE SELECCION (max. 30 caracteres)
*======================================================================*
*  Parametro | Texto
*  ----------|------------------------------------------------------------
*  S_WERKS   | Centro
*  S_MATNR   | Material
```

Los textos de selección tienen un máximo de 30 caracteres.

## 5. Cabecera

```abap
************************************************************************
*                         INFORMACION GENERAL
************************************************************************
* Programa...: ZDC[MOD]_[NOMBRE]
* Tipo.......: [Reporte ALV / Carga / Clase / Ampliación]
* Módulo.....: [MOD]
* Descripción: [Descripción breve]
* Requerim...: [ID-REQUERIMIENTO] / FS-[ID]
* Autor......: Programador AI Digital Core
* Fecha......: [DD.MM.AAAA]
* Transacción: ZDC[MOD][NN]
************************************************************************
```

Cada include lleva una cabecera corta: `*& Include ZDC[MOD]_[NOMBRE]_[SUF]`.

## 6. Performance sobre HANA

1. **Llevar el trabajo a la base de datos:** `JOIN`, agregaciones (`SUM`, `COUNT`, `GROUP BY`) y filtros en el `SELECT` o en una CDS, no en `LOOP`.
2. Nunca `SELECT *`; listar solo los campos usados.
3. Nunca `SELECT` dentro de `LOOP`. Si se necesita, `FOR ALL ENTRIES` con chequeo previo de tabla no vacía, o mejor un `JOIN`.
4. Para lecturas reutilizables o complejas, crear una CDS view entity (`ZI_…`) en vez de repetir el `JOIN` en varios programas.
5. Leer de las vistas CDS liberadas por SAP (`I_*`) cuando existan para el objeto de negocio.
6. Tablas internas grandes con clave ordenada o hash (`SORTED`/`HASHED`) cuando se leen por clave dentro de un `LOOP`.
7. AMDP solo cuando una CDS no alcanza, y con justificación en la entrega.

## 7. Manejo de errores y logs

- Excepciones basadas en clases (`TRY … CATCH cx_…`). Clases propias como `ZCX_DC[MOD]_…`.
- Cada BAPI se cierra con `BAPI_TRANSACTION_COMMIT` (`wait = 'X'`) o `BAPI_TRANSACTION_ROLLBACK`, y el resultado (`BAPIRET2`) se refleja al usuario.
- Procesos en fondo: registrar en el log de aplicación (SLG1), con objeto/subobjeto creados en SLG0.
- `RETURN` y `CHECK` para cortar el flujo; nunca `EXIT` fuera de bucles.

## 8. Seguridad

- `AUTHORITY-CHECK` antes de mostrar o modificar datos sensibles (sociedad, centro, organización de ventas…).
- Al construir `WHERE` dinámicos, escapar valores con `cl_abap_dyn_prg` para evitar inyección SQL.
- No escribir datos personales ni credenciales en logs.

## 9. Clean core

Antes de elegir el tipo de ampliación, seguir este orden de preferencia:

1. **Estándar / configuración** — si se cubre, no se programa.
2. **Key User Extensibility** — campos (`ZZ1_`), lógica (*Custom Logic*), CDS personalizadas.
3. **Developer extensibility con APIs liberadas** — ABAP Cloud, RAP, CDS sobre `I_*`, BAdIs liberadas.
4. **ABAP clásico** con APIs clásicas (BAPI, BAdI clásicas, enhancement spots) — aceptable, documentando el motivo.
5. **Objetos internos SAP o modificaciones** — solo con aprobación del arquitecto y registro en la entrega.

En cada entrega, indicar en qué nivel queda cada objeto y por qué no se pudo usar uno más limpio.

## 10. Objetos SAP estándar — regla de verificación (crítica)

**Nunca inventar ni asumir nombres de objetos SAP estándar**: BAPIs y FMs, tablas, CDS `I_*`, clases y métodos, parámetros de interfaz, transacciones y parámetros SPA/GPA.

| Caso | Qué hacer |
|---|---|
| Objeto ampliamente conocido (`BAPI_TRANSACTION_COMMIT`, `CL_SALV_TABLE`, `VBAK`, `I_PRODUCT`) | Usarlo sin advertencia |
| Plausible pero no confirmado (nombre o parámetros) | Usarlo con `" TODO: verificar en SE37 / SE24 / ADT antes de activar` junto a cada parámetro dudoso |
| Desconocido | Comentario `" PENDIENTE:` con opciones a investigar y avisar al arquitecto en la entrega |

Presentar como estándar un objeto inventado genera código que no compila y pérdida de tiempo en el cliente.

## Checklist

- [ ] Includes completos y en el orden de la §1.
- [ ] Clase local con constructor; sin `FORM`, sin globales de negocio.
- [ ] Sintaxis moderna; nada de la lista de prohibidos.
- [ ] Ninguna tabla o campo eliminado por la simplificación de S/4HANA.
- [ ] Textos en `_TXT`; mensajes `text-mNN` en reports o clase `ZDC_[MOD]` en clases.
- [ ] `SELECT` sin `*`, sin `SELECT` en `LOOP`, con trabajo empujado a la BD.
- [ ] `AUTHORITY-CHECK` donde corresponde.
- [ ] Nivel de clean core declarado por objeto.
- [ ] Objetos SAP dudosos marcados con `TODO` / `PENDIENTE`.
