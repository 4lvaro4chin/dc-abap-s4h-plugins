---
name: abap-s4h-alv-factory
description: Genera reportes ALV en ABAP para SAP S/4HANA con cl_salv_table=>factory siguiendo el patrón modelo Digital Core (clase local lcl_reporte por includes DEF/IMP, variante de layout, hotspot, botones propios en status SALV_STANDARD, totales, subtotales, colores, selección de filas y proceso masivo con BAPI y semáforo). Usar cuando haya que crear un reporte ALV, un programa de proceso con grilla o una carga masiva desde Excel con resultado en ALV.
---

# ABAP ALV Factory (cl_salv_table) — S/4HANA

Patrón estándar Digital Core para reportes ALV con `cl_salv_table=>factory`. Está derivado de los programas modelo `ZOSGE_MODELO_ALV_FACTORY` (reporte de datos) y `ZOSGE_CREAR_USUARIO` (proceso masivo desde Excel), probados en productivo.

Cubre:
- Reporte ALV de consulta (RICEF **R**).
- Programa de proceso que muestra datos y ejecuta acciones sobre las filas marcadas (botón propio).
- Carga masiva (RICEF **C**) con resultado en ALV y semáforo de estado.

Antes de generar, aplicar también:
- `abap-s4h-nomenclatura` — nombres de objetos y variables.
- `abap-s4h-estandares` — estructura de includes, sintaxis y reglas generales.
- `abap-s4h-selection-screen` — si la pantalla de selección tiene más que rangos simples.

## Archivos de esta skill

| Archivo | Cuándo leerlo |
|---|---|
| [references/plantilla-reporte.md](references/plantilla-reporte.md) | Siempre. Código completo de los includes de un reporte con botón de proceso. |
| [references/variante-carga-excel.md](references/variante-carga-excel.md) | Si el programa carga un archivo Excel y procesa filas con una BAPI. |
| [references/recetario-salv.md](references/recetario-salv.md) | Para cualquier opción de `cl_salv_table` que no esté en la plantilla (posición de columna, checkbox, iconos, top-of-page, eventos, colores). |

---

## 1. Arquitectura del patrón

```
ZDC[MOD]_[NOMBRE]            → Cabecera + REPORT + INCLUDEs
ZDC[MOD]_[NOMBRE]_TOP        → Estructura gs_selec para SELECT-OPTIONS
ZDC[MOD]_[NOMBRE]_DEF        → CLASS lcl_reporte DEFINITION
ZDC[MOD]_[NOMBRE]_IMP        → CLASS lcl_reporte IMPLEMENTATION
ZDC[MOD]_[NOMBRE]_SEL        → Pantalla de selección + F4 de variante
ZDC[MOD]_[NOMBRE]_MAI        → START-OF-SELECTION (instancia, obtiene datos, muestra ALV)
ZDC[MOD]_[NOMBRE]_F01        → Stub vacío
ZDC[MOD]_[NOMBRE]_TXT        → Catálogo de textos
```

**Orden de los INCLUDE:** `TOP → DEF → IMP → SEL → MAI → F01 → TXT`. La clase se define antes de que `_SEL` llame a su F4 y `_MAI` la instancie.

### Responsabilidades de `lcl_reporte`

| Método | Visibilidad | Responsabilidad |
|---|---|---|
| `match_code_variant` | CLASS-METHOD público | F4 de variante de layout (`LVC_VARIANT_F4`) |
| `constructor` | público | Recibe rangos y flags de la pantalla y los guarda en atributos privados |
| `pro_obtener_datos` | público | `SELECT` + consolidación en `mt_output`. Lanza la excepción clásica `no_data` |
| `mostrar_alv` | público | Crea el ALV con `factory` y encadena todos los `set_*` |
| `set_pf_status` | privado | Status GUI con botones propios o funciones estándar |
| `set_layout` | privado | Clave de layout, restricción de grabado, variante inicial |
| `get_handle_events` | privado | Registra `on_user_command` y `on_link_click` |
| `on_user_command` | privado (handler) | `CASE` por código de función de los botones propios |
| `on_link_click` | privado (handler) | Navegación por hotspot (`SET PARAMETER` + `CALL TRANSACTION`) |
| `set_display_setting` | privado | Título del listado y filas rayadas |
| `set_column_setting` | privado | Optimizar ancho, textos de cabecera, ocultar columnas |
| `set_hotspot` | privado | Columnas con tipo de celda hotspot |
| `set_selections` | privado | Modo de selección de filas |
| `set_aggregations` | privado | Totales |
| `set_sorts` | privado | Orden y subtotales |
| `set_colors` | privado | Color de columna completa y de fila/celda |
| `pro_ejecutar_proceso` | privado | Lee filas marcadas, confirma con popup, ejecuta y refresca |

**Regla:** un `set_*` por aspecto del ALV. Nunca mezclar columnas con ordenación o colores en el mismo método.

> Los parámetros de los métodos de `lcl_reporte` usan `pi_` / `pe_` (heredado del modelo). Mantenerlo en todo el programa; ver la excepción en `abap-s4h-nomenclatura` §7.

---

## 2. Prerrequisito: status GUI `SALV_STANDARD` (solo si hay botones propios)

`cl_salv_table` en pantalla completa **no permite agregar botones por código**. Para tener botones propios se copia el status estándar al programa Z y se le añaden los códigos de función.

Pasos (se documentan en las instrucciones de configuración de la entrega):

1. Transacción **SE41** → *Status* → *Copiar*.
   - Programa origen: `SALV_DEMO_TABLE_LAYOUT`, status origen: `SALV_STANDARD`.
   - Programa destino: `ZDC[MOD]_[NOMBRE]`, status destino: `SALV_STANDARD`.
2. Agregar en la *Barra de herramientas de aplicación* los botones propios, por ejemplo:

| Código función | Texto | Icono sugerido |
|---|---|---|
| `&PROCESO` | Ejecutar proceso | `ICON_EXECUTE_OBJECT` |
| `&SAP_ALL` | Asignar perfil | `ICON_ASSIGN` |

3. Activar el status. Los códigos empiezan con `&` para no colisionar con las funciones estándar del ALV.
4. En el código, `set_pf_status` usa `set_screen_status` con `set_functions = c_functions_all` para conservar las funciones estándar más las propias.

Si el reporte **no necesita botones propios**, no se copia el status; se usa `get_functions( )->set_all( abap_true )`.

Dejar siempre el recordatorio en la cabecera del programa principal:

```abap
************************************************************************
* Agregar Status bar para nuevos botones
* Tx.: SE41
* Programa: SALV_DEMO_TABLE_LAYOUT
* Status: SALV_STANDARD
************************************************************************
```

---

## 3. Reglas de generación

1. **Siempre `cl_salv_table=>factory`**, nunca `REUSE_ALV_GRID_DISPLAY` ni `cl_gui_alv_grid`, salvo que la especificación exija edición de celdas (en ese caso, escalar al arquitecto). Si el requerimiento pide una app web en lugar de SAP GUI, el ALV no aplica: evaluar Fiori Elements sobre CDS/RAP.
2. **Una clase local `lcl_reporte`** por programa, con los métodos de la tabla §1. No renombrar `pro_obtener_datos`, `mostrar_alv` ni los `set_*`: son el contrato reconocible del patrón.
3. **Datos en la sección privada** (`mt_output`, rangos, `ms_options`, `mo_alv`). La única global es `gs_selec` en `_TOP`.
4. **El constructor recibe los rangos** con `s_xxx[]` desde `_MAI`. La clase nunca accede a `s_xxx`.
5. **`pro_obtener_datos` levanta `no_data`** con mensaje `TYPE 'S' DISPLAY LIKE 'E'`, y `_MAI` solo muestra el ALV si `sy-subrc EQ 0`.
6. **La salida siempre incluye `set_colors TYPE lvc_t_scol`** aunque no se coloree al inicio (marcada como técnica); evita cambiar la estructura después.
7. **En programas de proceso, `icono TYPE icon_d` es la primera columna y `message TYPE bapi_msg` la última.** Estados: inactivo → éxito / error / advertencia. Nunca se reprocesan filas en error.
8. **Cada BAPI cierra con COMMIT (`wait = 'X'`) o ROLLBACK** dentro del mismo método, y el resultado se escribe en la fila.
9. **Botones propios** exigen el status `SALV_STANDARD` copiado en SE41 y códigos con `&`.
10. **Confirmación antes de procesar** con `POPUP_TO_CONFIRM` y validación de filas marcadas (`get_selected_rows`).
11. **Tras procesar**: `set_optimize( abap_true )` + `refresh( )`. Nunca volver a llamar a `factory` ni `display`.
12. **Todos los textos** vía `text-xxx` y documentados en `_TXT`. Ningún literal visible en el código.
13. **Hotspots** solo sobre claves de objetos con transacción de visualización (`MM03`, `SU01`, `VA03`, `ME23N`, etc.), con `SET PARAMETER ID` + `CALL TRANSACTION … AND SKIP FIRST SCREEN`.
14. **Modelo de datos S/4HANA**: no leer tablas eliminadas (`VBUK`, `VBUP`, `KONV`…) y preferir las tablas nuevas o las CDS `I_*` (ver `abap-s4h-estandares`).
15. **Excel**: fila 1 es cabecera (`i_begin_row = 2`), `i_end_col` igual al número de componentes de `ty_input`, y el orden de columnas del Excel es el orden de componentes del tipo.

---

## 4. Checklist de entrega

- [ ] Programa principal con cabecera INFORMACION GENERAL y recordatorio SE41 (si hay botones).
- [ ] Includes `_TOP / _DEF / _IMP / _SEL / _MAI / _F01 / _TXT` en `docs/[ID]/Code/` y en el orden de §1.
- [ ] `lcl_reporte` con constructor, `pro_obtener_datos`, `mostrar_alv` y los `set_*` necesarios.
- [ ] F4 de variante (`match_code_variant`) y layout con `set_key` + `restrict_none`.
- [ ] Hotspot con navegación implementada en `on_link_click`.
- [ ] Columna `SET_COLORS` marcada como técnica.
- [ ] Modo de selección `row_column` y `pro_ejecutar_proceso` con popup + validación de filas (si hay botón).
- [ ] Semáforo con las 4 constantes de icono (programas de proceso).
- [ ] COMMIT / ROLLBACK por fila procesada (programas de proceso).
- [ ] Todos los textos en `_TXT`; ningún literal en el código.
- [ ] Sin tablas eliminadas por la simplificación de S/4HANA.
- [ ] Instrucciones de configuración con transacción (SE93), status GUI (SE41) y variantes de layout.
