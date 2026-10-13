# Cambios del modelo de datos en S/4HANA que afectan al código

Referencia rápida para no programar sobre estructuras de ECC. Es un resumen, no reemplaza la **Simplification List** de la versión exacta del cliente: ante la duda, verificar ahí o en ADT.

## Regla general

- Las tablas antiguas que SAP mantiene como **vistas de compatibilidad** se pueden **leer**, pero son más lentas que la tabla nueva y **nunca se escriben**.
- Para código nuevo, leer de la tabla nueva o, mejor, de la CDS liberada `I_*` del objeto de negocio.

## Logística — Inventario (MM-IM)

| ECC | S/4HANA | Qué hacer |
|---|---|---|
| `MKPF` + `MSEG` | `MATDOC` | Leer `MATDOC` o la CDS de documentos de material. `MKPF`/`MSEG` solo por compatibilidad. |
| Agregados de stock (`MARD`, `MCHB`, `MKOL`, `MSKA`…) | Calculados desde `MATDOC` | Lectura posible; para volúmenes grandes preferir las CDS de stock (`I_MaterialStock…`, verificar nombre en el sistema). |
| Movimientos vía BDC (MB1A, MB1B…) | Transacciones MB* retiradas | `BAPI_GOODSMVT_CREATE`. |

## Logística — Ventas (SD)

| ECC | S/4HANA | Qué hacer |
|---|---|---|
| `VBUK` / `VBUP` (status) | Campos de status en `VBAK`/`VBAP`, `LIKP`/`LIPS`, `VBRK`/`VBRP` | Leer el status desde la cabecera/posición del propio documento. |
| `KONV` (condiciones del documento) | `PRCD_ELEMENTS` | Leer `PRCD_ELEMENTS`. |
| Gestión de crédito FI-AR (`KNKK`, `S066`, `S067`) | SAP Credit Management (FIN-FSCM-CR) | Usar las tablas/APIs de Credit Management (`UKM*`). |
| Rappels SD (acuerdos de rappel) | Settlement Management (contratos de condiciones) | No desarrollar sobre el rappel clásico. |
| Comercio exterior SD-FT | SAP GTS | Validar con el arquitecto. |

## Maestros

| ECC | S/4HANA | Qué hacer |
|---|---|---|
| Clientes y proveedores con XD01 / XK01 | **Business Partner** obligatorio (CVI) | Crear y modificar vía BP (APIs de Business Partner). `KNA1`/`LFA1` siguen existiendo para lectura. |
| `MATNR` de 18 caracteres | `MATNR` de **40** caracteres | No usar offsets fijos ni `CHAR18`; tipar siempre con `matnr`. Revisar interfaces con archivos de ancho fijo. |

## Finanzas y Controlling

| ECC | S/4HANA | Qué hacer |
|---|---|---|
| `BSIS`, `BSAS`, `BSID`, `BSAD`, `BSIK`, `BSAK` | Índices sustituidos por vistas sobre `ACDOCA`/`BSEG` | Lectura por compatibilidad; para reporting nuevo usar `ACDOCA` o las CDS `I_JournalEntry…` (verificar nombre). |
| `GLT0`, `FAGLFLEXT`, `FAGLFLEXA` (totales y líneas del nuevo libro mayor) | `ACDOCA` (Universal Journal) | Leer `ACDOCA`. |
| `COEP`, `COSP`, `COSS` (CO) | `ACDOCA` para costes reales | Leer `ACDOCA`. |
| `ANLP`, `ANEP` (activos fijos) | `ACDOCA` | Leer `ACDOCA`. |
| Ledger de materiales opcional | Material Ledger **obligatorio** | Considerar `CKMLHD` / `CKMLCR` y las valoraciones en múltiples monedas. |

## Formularios y salidas

| ECC | S/4HANA | Qué hacer |
|---|---|---|
| SAPScript | No para desarrollos nuevos | Adobe Forms. |
| Control de mensajes NAST | Output Management (BRF+) en pedidos de compra, facturas y otros | Verificar qué gestión de salidas tiene activa el cliente para el documento antes de diseñar el formulario. |
| SmartForms | Soportado, pero no estratégico | Adobe Forms para formularios nuevos, salvo que el cliente mantenga NAST y SmartForms. |

## Almacén

| ECC | S/4HANA | Qué hacer |
|---|---|---|
| LE-WM (`LTAK`, `LTAP`, `LQUA`) | EWM embebido o decentralizado | Confirmar con el arquitecto qué solución de almacén usa el cliente antes de desarrollar. |
