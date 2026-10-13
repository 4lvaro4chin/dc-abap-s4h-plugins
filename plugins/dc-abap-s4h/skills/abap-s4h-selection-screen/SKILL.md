---
name: abap-s4h-selection-screen
description: Genera el include _SEL (pantalla de selección) de reports ABAP en SAP S/4HANA con todos los elementos de SELECTION-SCREEN (parameters, select-options, checkbox, radiobutton, listbox, bloques, líneas, comentarios, pushbuttons, pestañas y control dinámico con MODIF ID). Usar al crear o modificar la pantalla de selección de un report.
---

# ABAP Selection Screen (_SEL) — S/4HANA

Esta skill genera el include `_SEL` (pantalla de selección) de reports ABAP en SAP S/4HANA.
Los nombres de parámetros, bloques y textos siguen la skill `abap-s4h-nomenclatura` (§8 y §9); la estructura de includes, la skill `abap-s4h-estandares`.

> Los nombres de `PARAMETERS` y `SELECT-OPTIONS` tienen **máximo 8 caracteres**. Los textos de selección, máximo 30.

## Referencia Completa de Elementos SELECTION-SCREEN

### 1. PARAMETERS - Campos de entrada simples

#### Campo basico obligatorio
```abap
PARAMETERS p_campo(10) TYPE c OBLIGATORY.
```

#### Longitud visible (VISIBLE LENGTH)
Controla el ancho del campo en pantalla independiente de la longitud real del dato:
```abap
PARAMETERS: p1(10) TYPE c VISIBLE LENGTH 1,
            p2(10) TYPE c VISIBLE LENGTH 5,
            p3(10) TYPE c VISIBLE LENGTH 10.
```

#### Checkbox
Casilla de verificacion. Usar `DEFAULT 'X'` para marcar por defecto y `USER-COMMAND` para disparar eventos:
```abap
PARAMETERS: cb_opt1 AS CHECKBOX USER-COMMAND uc_flag,
            cb_opt2 AS CHECKBOX DEFAULT 'X' USER-COMMAND uc_flag.
```

#### Radiobutton
Botones de opcion agrupados con `GROUP`. Solo uno puede estar seleccionado por grupo:
```abap
PARAMETERS: r1 RADIOBUTTON GROUP rad1,
            r2 RADIOBUTTON GROUP rad1 DEFAULT 'X',
            r3 RADIOBUTTON GROUP rad1.
SELECTION-SCREEN SKIP 1.
PARAMETERS: s1 RADIOBUTTON GROUP rad2,
            s2 RADIOBUTTON GROUP rad2,
            s3 RADIOBUTTON GROUP rad2 DEFAULT 'X'.
```

#### Listbox (dropdown)
Campo desplegable con valores del dominio o search help:
```abap
PARAMETERS p_carrid TYPE spfli-carrid
                    AS LISTBOX VISIBLE LENGTH 20 DEFAULT 'LH'.
```

#### MODIF ID - Identificador para control dinamico
Permite agrupar campos para habilitarlos/deshabilitarlos dinamicamente:
```abap
PARAMETERS: p_largo(30) MODIF ID a,
            p_corto(2)  MODIF ID b,
            p_fijo(5).
```

### 2. SELECT-OPTIONS - Rangos de seleccion

Campos de rango (desde/hasta) con opciones multiples. El `FOR` referencia un componente de la estructura `gs_selec` declarada en `_TOP`:
```abap
" En include TOP
TYPES: BEGIN OF ty_selec,
         matnr TYPE matnr,
         werks TYPE werks_d,
       END OF ty_selec.

DATA: gs_selec TYPE ty_selec.

" En include SEL
SELECT-OPTIONS:
  s_matnr FOR gs_selec-matnr,                 " Material
  s_werks FOR gs_selec-werks.                 " Centro
```
> **IMPORTANTE**: no usar `FOR lcl_clase=>atributo`. Todos los campos de rango salen de `gs_selec`, que es la única variable global del programa.

### 3. Elementos de formato

#### SKIP - Salto de renglones
Inserta lineas en blanco verticales entre elementos:
```abap
PARAMETERS field1(10) TYPE c.
SELECTION-SCREEN SKIP 1.          " 1 linea en blanco
PARAMETERS field2(10) TYPE c.
SELECTION-SCREEN SKIP 3.          " 3 lineas en blanco
PARAMETERS field3(10) TYPE c.
```

#### ULINE - Lineas de separacion
Dibuja lineas horizontales para separar secciones. Sintaxis: `/posicion(longitud)`:
```abap
PARAMETERS field1(10) TYPE c.
SELECTION-SCREEN ULINE /1(5).       " Linea corta desde posicion 1

PARAMETERS field2(10) TYPE c.
SELECTION-SCREEN ULINE /1(20).      " Linea media desde posicion 1

PARAMETERS field3(10) TYPE c.
SELECTION-SCREEN ULINE /22(21).     " Linea desde posicion 22

" Linea completa dentro de BEGIN OF LINE
SELECTION-SCREEN BEGIN OF LINE.
SELECTION-SCREEN ULINE (79).
SELECTION-SCREEN END OF LINE.
```

#### COMMENT - Etiquetas de texto
Texto asociado a campos. Sintaxis: `posicion(longitud) variable`:
```abap
SELECTION-SCREEN BEGIN OF LINE.
SELECTION-SCREEN COMMENT 1(20) text-021 FOR FIELD p_campo.
SELECTION-SCREEN POSITION 21.
PARAMETERS: p_campo TYPE char5.
SELECTION-SCREEN END OF LINE.
```
> Los textos de COMMENT se inicializan en el evento INITIALIZATION si usan variables (ej: `tx_label = 'Mi texto'.`).

#### POSITION - Posicionamiento absoluto
Permite ubicar un campo en una posicion horizontal especifica dentro de `BEGIN OF LINE`:
```abap
SELECTION-SCREEN BEGIN OF LINE.
SELECTION-SCREEN COMMENT 1(20) text-021 FOR FIELD po1.
SELECTION-SCREEN POSITION 21.
PARAMETERS: po1 TYPE char5.
PARAMETERS: po2 TYPE char5.
SELECTION-SCREEN COMMENT 45(20) text-022 FOR FIELD po3.
PARAMETERS: po3 TYPE char5.
SELECTION-SCREEN END OF LINE.
```

### 4. PUSHBUTTON - Botones interactivos

Boton que dispara un USER-COMMAND procesable en `AT SELECTION-SCREEN`:
```abap
SELECTION-SCREEN PUSHBUTTON 10(20) text-020 USER-COMMAND uc_crear.
```

Botones en linea horizontal:
```abap
SELECTION-SCREEN BEGIN OF LINE.
SELECTION-SCREEN PUSHBUTTON (20) pb_sall USER-COMMAND uc_sall.
SELECTION-SCREEN PUSHBUTTON (20) pb_dall USER-COMMAND uc_dall.
SELECTION-SCREEN END OF LINE.
```
> Los textos de PUSHBUTTON con variables se inicializan en INITIALIZATION:
> ```abap
> INITIALIZATION.
>   pb_sall = 'Seleccionar Todas'.
>   pb_dall = 'Deseleccionar Todas'.
> ```

El manejo del boton se realiza en AT SELECTION-SCREEN:
```abap
AT SELECTION-SCREEN.
  CASE sy-ucomm.
    WHEN 'UC_SALL'.
      " Logica al presionar boton
    WHEN 'UC_DALL'.
      " Logica al presionar boton
  ENDCASE.
```

### 5. Bloques (BLOCK)

Contenedor visual con marco y titulo:
```abap
SELECTION-SCREEN BEGIN OF BLOCK b01 WITH FRAME TITLE text-b01.
  PARAMETERS: p_campo1 TYPE char10,
              p_campo2 TYPE char20.
SELECTION-SCREEN END OF BLOCK b01.
```
> El texto del titulo (`text-b01`) se define como text element en SE38.

### 6. Campos alineados en linea (BEGIN OF LINE)

Permite colocar multiples campos en la misma fila horizontal:
```abap
SELECTION-SCREEN BEGIN OF LINE.
  PARAMETERS: cb_opt1 AS CHECKBOX DEFAULT 'X'.
  SELECTION-SCREEN COMMENT (25) tx_opt1.
  PARAMETERS: cb_opt2 AS CHECKBOX DEFAULT 'X'.
  SELECTION-SCREEN COMMENT (25) tx_opt2.
SELECTION-SCREEN END OF LINE.
```
> Los textos de COMMENT con variables se inicializan en INITIALIZATION:
> ```abap
> INITIALIZATION.
>   tx_opt1 = 'Opcion 1'.
>   tx_opt2 = 'Opcion 2'.
> ```

### 7. Pestanas (TABBED BLOCK)

Crea solapas navegables. Cada pestana es un SCREEN AS SUBSCREEN:

```abap
" Primero definir las subscreens con su contenido
SELECTION-SCREEN BEGIN OF SCREEN 1100 AS SUBSCREEN.
  SELECTION-SCREEN BEGIN OF BLOCK b01 WITH FRAME TITLE text-b01.
    PARAMETERS campo1(15) TYPE c.
  SELECTION-SCREEN END OF BLOCK b01.
SELECTION-SCREEN END OF SCREEN 1100.

SELECTION-SCREEN BEGIN OF SCREEN 1200 AS SUBSCREEN.
  SELECTION-SCREEN BEGIN OF BLOCK b02 WITH FRAME TITLE text-b02.
    PARAMETERS campo2(5) TYPE c.
  SELECTION-SCREEN END OF BLOCK b02.
SELECTION-SCREEN END OF SCREEN 1200.

" Luego definir el tabbed block que las contiene
SELECTION-SCREEN BEGIN OF TABBED BLOCK tab_main FOR 15 LINES.
  SELECTION-SCREEN TAB (20) tab1 USER-COMMAND uc_tab1
    DEFAULT SCREEN 1100.
  SELECTION-SCREEN TAB (20) tab2 USER-COMMAND uc_tab2
    DEFAULT SCREEN 1200.
SELECTION-SCREEN END OF BLOCK tab_main.
```

Inicializar los textos de las pestanas:
```abap
INITIALIZATION.
  tab1 = 'Primera Solapa'.
  tab2 = 'Segunda Solapa'.
```

> **REGLAS IMPORTANTES para TABBED BLOCK:**
> - Las screens deben declararse con `AS SUBSCREEN`
> - Usar numeros de screen altos (1100, 1200, etc.) para evitar colisiones
> - El numero en `FOR n LINES` determina la altura del area de pestanas
> - Los textos de las pestanas (tab1, tab2) se asignan en INITIALIZATION
> - El TABBED BLOCK puede ir dentro o fuera de otro BLOCK
> - El orden de declaracion puede ser: primero las subscreens, luego el tabbed block, o viceversa

### 8. Control dinamico de pantalla

#### Ocultar/mostrar campos segun seleccion (screen-active)
Usa `MODIF ID` en la definicion y `screen-group1` en el evento `AT SELECTION-SCREEN OUTPUT`:
```abap
" En include SEL
PARAMETERS: p_largo(30) MODIF ID a,
            p_corto(2)  MODIF ID b.
PARAMETERS: p_a RADIOBUTTON GROUP rad5 USER-COMMAND act DEFAULT 'X',
            p_b RADIOBUTTON GROUP rad5.

" En include MAI
AT SELECTION-SCREEN OUTPUT.
  LOOP AT SCREEN.
    IF p_a = abap_true.
      IF screen-group1 = 'B'.
        screen-active = 0.         " Ocultar campo
      ENDIF.
    ELSEIF p_b = abap_true.
      IF screen-group1 = 'A'.
        screen-active = 0.         " Ocultar campo
      ENDIF.
    ENDIF.
    MODIFY SCREEN.
  ENDLOOP.
```

#### Campo grisado / solo lectura (screen-input)
Muestra el campo pero impide la edicion:
```abap
" En include SEL
PARAMETERS: p_campo(30) DEFAULT 'GRISADO'.

" En include MAI
AT SELECTION-SCREEN OUTPUT.
  LOOP AT SCREEN.
    IF screen-name = 'P_CAMPO'.
      screen-input = 0.            " Deshabilitar entrada (grisado)
    ENDIF.
    MODIFY SCREEN.
  ENDLOOP.
```

#### Propiedades de SCREEN disponibles
| Propiedad | Valores | Descripcion |
|-----------|---------|-------------|
| `screen-active` | 0 / 1 | Ocultar (0) o mostrar (1) el campo |
| `screen-input` | 0 / 1 | Solo lectura (0) o editable (1) |
| `screen-name` | string | Nombre del campo en mayusculas |
| `screen-group1` | char3 | Grupo asignado por MODIF ID |

### 9. Eventos relacionados con Selection Screen (en include MAI)

| Evento | Descripcion |
|--------|-------------|
| `INITIALIZATION` | Asignar valores por defecto, textos de pestanas, botones y comments |
| `AT SELECTION-SCREEN OUTPUT` | Control dinamico: ocultar/mostrar/grisar campos (PBO) |
| `AT SELECTION-SCREEN` | Validaciones y manejo de USER-COMMAND (PAI) |
| `AT SELECTION-SCREEN ON campo` | Validacion especifica de un campo |
| `AT SELECTION-SCREEN ON VALUE-REQUEST FOR campo` | F4 help personalizado |
| `AT SELECTION-SCREEN ON HELP-REQUEST FOR campo` | F1 help personalizado |

## Reglas de Generacion del Include _SEL

1. **Siempre generar dentro de un BLOCK con titulo** - no dejar campos sueltos
2. **SELECT-OPTIONS referencian `gs_selec`** declarada en TOP, nunca atributos de clase
3. **Pestanas (TABBED BLOCK)**: usar `AS SUBSCREEN`, numeros de screen >= 1100
4. **Checkboxes en columnas**: usar `BEGIN OF LINE` con `COMMENT` para las etiquetas
5. **Botones**: usar PUSHBUTTON con USER-COMMAND, inicializar texto en INITIALIZATION
6. **Control dinamico**: usar MODIF ID en SEL, logica en MAI (AT SELECTION-SCREEN OUTPUT)
7. **Nombres cortos y con prefijo**: `p_`, `s_`, `rb_`, `cb_`, `pb_`, maximo 8 caracteres. En la logica de eventos (MAI) se usa sintaxis moderna (`abap_true`, inline declarations, etc.)
8. **Textos**: referenciar text elements con `text-xxx`, no literales con `'texto'(xxx)`
9. **La logica de eventos** (INITIALIZATION, AT SELECTION-SCREEN, etc.) va en el include MAI, no en SEL
10. **Documentar todos los textos** en el include TXT (text elements, textos de seleccion, etc.)

## Patron de Generacion

Al generar un include _SEL, seguir esta estructura:

```
_SEL
├── TABBED BLOCK (si hay muchos campos, agrupar en pestanas)
│   ├── SCREEN AS SUBSCREEN (pestana 1)
│   │   ├── BLOCK con FRAME (grupo logico 1)
│   │   │   └── PARAMETERS / SELECT-OPTIONS
│   │   └── BLOCK con FRAME (grupo logico 2)
│   │       └── PARAMETERS / SELECT-OPTIONS
│   └── SCREEN AS SUBSCREEN (pestana 2)
│       ├── BLOCK con FRAME (grupo logico 3)
│       │   ├── PUSHBUTTON (acciones)
│       │   └── PARAMETERS con BEGIN OF LINE (checkboxes en columnas)
│       └── BLOCK con FRAME (grupo logico 4)
│           └── PARAMETERS / RADIOBUTTON
└── (si pocos campos, usar BLOCKs directos sin pestanas)
```

Cada generacion del _SEL debe ir acompanada de:
- Componentes en `gs_selec` (include TOP) para cada SELECT-OPTIONS
- Textos de pestanas, botones y comments en INITIALIZATION (include MAI)
- Logica de control dinamico en AT SELECTION-SCREEN OUTPUT (include MAI)
- Validaciones en AT SELECTION-SCREEN (include MAI)
- Documentacion de textos en include TXT
