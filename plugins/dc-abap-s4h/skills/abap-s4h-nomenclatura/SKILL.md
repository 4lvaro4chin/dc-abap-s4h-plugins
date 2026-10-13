---
name: abap-s4h-nomenclatura
description: Convención de nombres Digital Core para objetos de desarrollo ABAP en SAP S/4HANA (programas, includes, clases, diccionario, CDS, RAP, OData, BAdIs, paquetes, transacciones) y para variables, parámetros y elementos de pantalla. Incluye límites de longitud de SAP y prefijo configurable por cliente. Usar antes de proponer o crear cualquier objeto Z, al escribir la lista de objetos de una especificación o al revisar si un nombre es correcto.
---

# Nomenclatura ABAP S/4HANA — Digital Core

Fuente única de la regla de nombres. Las demás skills del plugin la referencian; si hay contradicción, manda esta.

## 1. Prefijo

Patrón base: `Z` + `[PFX]` + `[MOD]`

| Elemento | Valor | Ejemplo |
|---|---|---|
| `[PFX]` | `DC` por defecto. Si el cliente tiene estándar propio (ej. `PR`, `KM`), usar el suyo. | `ZDC…`, `ZPR…` |
| `[MOD]` | Módulo SAP en 2 letras: `MM`, `SD`, `FI`, `CO`, `PP`, `PM`, `QM`, `PS`, `WM`, `HR`; `CA` para transversal | `ZDCMM…` |

**Cómo decidir el prefijo:** usar el que indique la especificación funcional o el usuario. Si nadie lo indica, usar `DC` y decirlo explícitamente en la entrega. Nunca mezclar dos prefijos en un mismo requerimiento.

En el resto de este documento `ZDC` representa `Z[PFX]`.

## 2. Objetos de repositorio

| Objeto | Patrón | Ejemplo | Long. máx. |
|---|---|---|---|
| Paquete | `ZDC_[MOD]` o `ZDC_[MOD]_[PROYECTO]` | `ZDC_MM_PRODAC` | 30 |
| Programa (report) | `ZDC[MOD]_[NOMBRE]` | `ZDCMM_STOCK_VALORIZADO` | 40 (recomendado ≤ 30) |
| Include de programa | `[PROGRAMA]_[SUFIJO]` (ver §3) | `ZDCMM_STOCK_VALORIZADO_TOP` | 40 |
| Programa de carga | `ZDC[MOD]_CONV_[NOMBRE]` | `ZDCMM_CONV_MATERIALES` | 40 |
| Transacción | `ZDC[MOD][NN]` | `ZDCMM01` | 20 |
| Clase global | `ZCL_DC[MOD]_[DESC]` | `ZCL_DCSD_PRICING_ENGINE` | 30 |
| Interfaz global | `ZIF_DC[MOD]_[DESC]` | `ZIF_DCSD_PRICING` | 30 |
| Clase de excepción | `ZCX_DC[MOD]_[DESC]` | `ZCX_DCSD_PRICING` | 30 |
| Grupo de funciones | `ZDC[MOD]_[DESC]` | `ZDCMM_INTERFAZ_WMS` | 26 |
| Módulo de funciones | `ZDC[MOD]_FM_[DESC]` | `ZDCMM_FM_GET_STOCK` | 30 |
| Clase de mensajes | `ZDC_[MOD]` | `ZDC_SD` | 20 |
| Objeto de autorización | `ZDC_[MOD][NN]` | `ZDC_SD01` | 10 |

## 3. Includes de un report

| Sufijo | Contenido |
|---|---|
| `_TOP` | Tipos, constantes y estructura de referencia para `SELECT-OPTIONS` |
| `_DEF` | `CLASS … DEFINITION` |
| `_IMP` | `CLASS … IMPLEMENTATION` |
| `_SEL` | Pantalla de selección |
| `_MAI` | Eventos (`INITIALIZATION`, `AT SELECTION-SCREEN`, `START-OF-SELECTION`) |
| `_F01` | Stub vacío (no se usan `FORM`) |
| `_TXT` | Catálogo de textos (comentarios) |

El orden de los `INCLUDE` en el programa principal lo define la skill `abap-s4h-estandares`.

## 4. Diccionario de datos

| Objeto | Patrón | Ejemplo | Long. máx. |
|---|---|---|---|
| Tabla transparente | `ZDC[MOD]_[DESC]` | `ZDCMM_CUSTFLD` | **16** |
| Tabla draft (RAP) | `ZDC[MOD]_[DESC]_D` | `ZDCMM_CUSTFLD_D` | **16** |
| Estructura | `ZS_DC[MOD]_[DESC]` | `ZS_DCMM_STOCK_DATA` | 30 |
| Tipo tabla | `ZTT_DC[MOD]_[DESC]` | `ZTT_DCMM_STOCK_DATA` | 30 |
| Elemento de datos | `ZDE_DC[MOD]_[DESC]` | `ZDE_DCMM_CUSTOM_STATUS` | 30 |
| Dominio | `ZDO_DC[MOD]_[DESC]` | `ZDO_DCMM_STATUS_TYPE` | 30 |
| Ayuda de búsqueda | `ZSH_DC[MOD]_[DESC]` | `ZSH_DCMM_STATUS` | 30 |
| Objeto de bloqueo | `EZDC[MOD]_[DESC]` (siempre empieza con `EZ`) | `EZDCMM_CUSTFLD` | **16** |
| Append a tabla estándar | `ZAPP_DC[MOD]_[TABLA]` | `ZAPP_DCSD_VBAK` | 30 |
| Campo en append / include de extensión | `ZZ1_[CAMPO]` o `ZZ[CAMPO]` | `ZZ1_CANAL_WEB` | 30 |

> **Atención con las tablas:** el límite es 16 caracteres. `ZDCMM_CUSTOM_FIELDS` (19) **no es válido**; abreviar a `ZDCMM_CUSTFLD`.
> Los campos `ZZ1_` son los que genera Key User Extensibility (app *Custom Fields*); preferir esa vía cuando el objeto de negocio la soporta.

## 5. CDS y RAP

Sigue el modelo de datos virtual (VDM) de SAP con el prefijo Digital Core.

| Objeto | Patrón | Ejemplo | Long. máx. |
|---|---|---|---|
| CDS interface / básica | `ZI_DC[MOD]_[DESC]` | `ZI_DCMM_STOCK` | 30 |
| CDS root transaccional (RAP) | `ZR_DC[MOD]_[DESC]` | `ZR_DCMM_SOLICITUD` | 30 |
| CDS projection / consumo | `ZC_DC[MOD]_[DESC]` | `ZC_DCMM_SOLICITUD` | 30 |
| CDS de parámetros / privada | `ZP_DC[MOD]_[DESC]` | `ZP_DCMM_STOCK_AGG` | 30 |
| CDS abstracta (parámetros de acción) | `ZA_DC[MOD]_[DESC]` | `ZA_DCMM_APROBAR_PRM` | 30 |
| CDS de extensión | `ZE_DC[MOD]_[DESC]` | `ZE_DCMM_STOCK` | 30 |
| Metadata extension | mismo nombre que la CDS que anota | `ZC_DCMM_SOLICITUD` | 30 |
| Behavior definition | mismo nombre que la CDS root / projection | `ZR_DCMM_SOLICITUD` | 30 |
| Behavior pool (clase) | `ZBP_[NOMBRE_CDS_ROOT]` | `ZBP_R_DCMM_SOLICITUD` | 30 |
| Service definition | `ZUI_DC[MOD]_[DESC]` (UI) / `ZAPI_DC[MOD]_[DESC]` (API) | `ZUI_DCMM_SOLICITUD` | 30 |
| Service binding | service definition + `_O4` / `_O2` | `ZUI_DCMM_SOLICITUD_O4` | 30 |
| Proyecto SEGW (OData V2 clásico) | `ZDC[MOD]_[DESC]_SRV` | `ZDCMM_STOCK_SRV` | 30 |

> Si el nombre con prefijo no cabe, abreviar `[DESC]`, nunca el prefijo ni el módulo. Las CDS views clásicas (`DEFINE VIEW` con `@AbapCatalog.sqlViewName`) llevan además un SQL view de **16** caracteres: `ZV_DC[MOD]_[DESC]`. En S/4HANA 2020+ crear siempre `DEFINE VIEW ENTITY`, que no lo necesita.

## 6. Ampliaciones

| Objeto | Patrón | Ejemplo |
|---|---|---|
| Implementación de enhancement (spot / BAdI) | `ZEI_DC[MOD]_[DESC]` | `ZEI_DCSD_PRICING` |
| Implementación de BAdI | `ZBI_DC[MOD]_[DESC]` | `ZBI_DCSD_ORDER_CHECK` |
| Clase de implementación de BAdI | `ZCL_DC[MOD]_BADI_[DESC]` | `ZCL_DCSD_BADI_ORDER_CHECK` |
| Clase de lógica reutilizable llamada desde la ampliación | `ZCL_DC[MOD]_[DESC]` | `ZCL_DCSD_ORDER_RULES` |
| Formulario Adobe | `ZDC[MOD]_AF_[DESC]` | `ZDCSD_AF_FACTURA` |
| Interfaz de formulario Adobe | `ZDC[MOD]_AFI_[DESC]` | `ZDCSD_AFI_FACTURA` |

## 7. Variables, parámetros y atributos

Prefijo = **ámbito** + **tipo**.

| Ámbito | Letra | | Tipo | Letra |
|---|---|---|---|---|
| Local (método) | `l` | | Variable elemental | `v` |
| Global (programa) | `g` | | Estructura | `s` |
| Atributo de instancia | `m` | | Tabla interna | `t` |
| Parámetro IMPORTING | `i` | | Referencia a objeto | `o` |
| Parámetro EXPORTING | `e` | | Referencia a dato | `r` |
| Parámetro CHANGING | `c` | | Excepción | `x` |
| Parámetro RETURNING | `r` | | | |

Ejemplos: `lv_matnr`, `ls_vbak`, `lt_items`, `lo_alv`, `lx_error`, `mt_output`, `iv_werks`, `et_return`, `rv_result`.

| Elemento | Patrón | Ejemplo |
|---|---|---|
| Constante local / global / de clase | `lc_` / `gc_` / `gc_` | `gc_icon_success` |
| Tipo local | `ty_` | `ty_output` |
| Tipo tabla local | `tt_` (o `tab_` en el patrón ALV) | `tt_output` |
| Tipo rango | `ty_[campo]_rng` | `ty_matnr_rng` |
| Field-symbol | `<fs_[nombre]>` | `<fs_output>` |
| Clase local | `lcl_` | `lcl_reporte` |
| Interfaz local | `lif_` | `lif_strategy` |
| Clase de test | `ltc_` | `ltc_reporte` |

> **Excepción heredada:** el patrón `abap-s4h-alv-factory` usa `pi_` / `pe_` para los parámetros de sus métodos (`pi_werks`, `pe_variant`) porque viene del programa modelo productivo. Dentro de un programa ALV, mantener `pi_`/`pe_` en todos los métodos de `lcl_reporte`; no mezclar con `iv_`/`ev_`.

## 8. Pantalla de selección

Los nombres de `PARAMETERS` y `SELECT-OPTIONS` tienen **máximo 8 caracteres**.

| Elemento | Patrón | Ejemplo |
|---|---|---|
| Parámetro | `p_[campo]` | `p_bukrs` |
| Select-option | `s_[campo]` | `s_matnr` |
| Radiobutton | `rb_[op]` | `rb_detal` |
| Checkbox | `cb_[op]` | `cb_test` |
| Pushbutton | `pb_[acción]` | `pb_sall` |
| Comment | `tx_[desc]` | `tx_opt1` |
| Bloque | `b[NN]` | `b01` |
| Pestaña / tabbed block | `tab[N]` / `tab_main` | `tab1` |
| User-command | `uc_[acción]` | `uc_flag` |
| MODIF ID | 3 caracteres | `M01` |

## 9. Textos

| Uso | ID | Ejemplo |
|---|---|---|
| Título de bloque | `Bnn` | `text-b01` |
| Título de listado | `Tnn` | `text-t01` |
| Cabecera de columna ALV | `Cnn` | `text-c01` |
| Mensaje al usuario | `Mnn` | `text-m01` |
| Popup | `Pnn` | `text-p01` |
| Indicador de progreso | `Gnn` | `text-g01` |

## 10. Archivos generados

En `docs/[ID-REQUERIMIENTO]/Code/`, el nombre del archivo es el nombre técnico en minúsculas con extensión según el tipo:

| Tipo | Extensión | Ejemplo |
|---|---|---|
| Report / include / clase / FM | `.abap` | `zdcmm_stock_valorizado_top.abap` |
| CDS / view entity | `.ddls.asddls` | `zi_dcmm_stock.ddls.asddls` |
| Metadata extension | `.ddlx.asddlxs` | `zc_dcmm_solicitud.ddlx.asddlxs` |
| Behavior definition | `.bdef.asbdef` | `zr_dcmm_solicitud.bdef.asbdef` |
| Service definition | `.srvd.srvdsrv` | `zui_dcmm_solicitud.srvd.srvdsrv` |
| Tabla / estructura / elemento de datos | `.md` con la definición de campos | `zdcmm_custfld.md` |

## Checklist

- [ ] Prefijo único en todo el requerimiento (`DC` u el del cliente, declarado en la entrega).
- [ ] Todos los nombres dentro del límite de longitud (tablas y objetos de bloqueo ≤ 16).
- [ ] Parámetros de pantalla ≤ 8 caracteres.
- [ ] Variables con prefijo ámbito + tipo; ALV con `pi_`/`pe_` de forma consistente.
- [ ] CDS nuevas como `DEFINE VIEW ENTITY`, con el prefijo VDM que corresponde a su capa.
