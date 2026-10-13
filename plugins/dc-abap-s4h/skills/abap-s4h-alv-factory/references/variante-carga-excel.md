# Variante B: proceso masivo desde Excel con semáforo (modelo `ZOSGE_CREAR_USUARIO`)

Se usa cuando el programa **carga un archivo**, lo valida, lo muestra en ALV con una columna de estado (icono) y ejecuta una BAPI por cada fila marcada. Todo el patrón de `plantilla-reporte.md` se mantiene; cambia lo siguiente.

## 4.1 Diferencias en la definición de la clase

```abap
*   Tipo de entrada (columnas del Excel en el mismo orden que el archivo)
    TYPES: BEGIN OF ty_input,
             bname      TYPE xubname,
             name_first TYPE ad_namefir,
             name_last  TYPE ad_namelas,
             smtp_addr  TYPE ad_smtpadr,
           END OF ty_input.
    TYPES: tab_input TYPE STANDARD TABLE OF ty_input WITH DEFAULT KEY.

*   Tipo de salida: SIEMPRE con icono de estado en la primera columna y mensaje al final
    TYPES: BEGIN OF ty_output,
             icono      TYPE icon_d,
             bname      TYPE xubname,
             name_first TYPE ad_namefir,
             name_last  TYPE ad_namelas,
             name_text  TYPE ad_namtext,
             smtp_addr  TYPE ad_smtpadr,
             message    TYPE bapi_msg,
           END OF ty_output.

*   Métodos adicionales
    CLASS-METHODS:
      match_code_file RETURNING VALUE(pe_filxls) TYPE localfile.

    METHODS:
      check_file        IMPORTING  pi_filxls TYPE localfile
                        EXCEPTIONS no_data,
      upload_file_excel IMPORTING  pi_filxls TYPE localfile
                        EXPORTING  pe_input  TYPE tab_input
                        EXCEPTIONS no_data,
      mostrar_progreso  IMPORTING  pi_percentage TYPE numc3
                                   pi_text       TYPE string,
      pro_ejecutar_proceso IMPORTING pi_function TYPE char20
                                     pi_alv      TYPE REF TO cl_salv_table,
      pro_crear_usuario CHANGING pe_output TYPE ty_output.
```

Constantes de estado (mismas en ambos modelos):

| Constante | Icono | Significado |
|---|---|---|
| `gc_icon_inactive` | `@BZ@` | Pendiente de procesar |
| `gc_icon_success` | `@5B@` | Procesado OK |
| `gc_icon_error` | `@5C@` | Error (no se reprocesa) |
| `gc_icon_warning` | `@5D@` | Advertencia de validación previa (ej. registro ya existe) |

## 4.2 Pantalla de selección

```abap
SELECTION-SCREEN BEGIN OF BLOCK b01 WITH FRAME TITLE TEXT-b01.
  PARAMETERS: p_filxls TYPE localfile DEFAULT 'C:\*.xls' OBLIGATORY.
  PARAMETERS: p_varian TYPE disvariant-variant NO-DISPLAY.
SELECTION-SCREEN END OF BLOCK b01.

AT SELECTION-SCREEN ON VALUE-REQUEST FOR p_filxls.
  p_filxls = lcl_reporte=>match_code_file( ).
```

## 4.3 Métodos de carga

```abap
  METHOD match_code_file.
    CALL FUNCTION 'KD_GET_FILENAME_ON_F4'
      CHANGING
        file_name = pe_filxls.
  ENDMETHOD.

  METHOD mostrar_progreso.
    CALL FUNCTION 'SAPGUI_PROGRESS_INDICATOR'
      EXPORTING
        percentage = pi_percentage
        text       = pi_text.
  ENDMETHOD.

  METHOD check_file.
    DATA: lv_file   TYPE string,
          lv_result TYPE c.

    lv_file = pi_filxls.

    cl_gui_frontend_services=>file_exist(
      EXPORTING  file                 = lv_file
      RECEIVING  result               = lv_result
      EXCEPTIONS cntl_error           = 1
                 error_no_gui         = 2
                 wrong_parameter      = 3
                 not_supported_by_gui = 4
                 OTHERS               = 5 ).

    IF lv_result IS INITIAL.
      RAISE no_data.
    ENDIF.
  ENDMETHOD.

  METHOD upload_file_excel.
    DATA: lv_fname  TYPE rlgrap-filename,
          lt_excel  TYPE STANDARD TABLE OF alsmex_tabline,
          lv_numtot TYPE numc10,
          lv_numreg TYPE numc10,
          lv_porcen TYPE numc3,
          lo_line   TYPE REF TO data.

    FIELD-SYMBOLS: <fs_excel> TYPE alsmex_tabline,
                   <fs_input> TYPE any,
                   <fs_val>   TYPE any.

    mostrar_progreso( pi_percentage = 100 pi_text = text-g01 ).

    lv_fname = pi_filxls.

*   Fila 1 = cabecera; columnas según ty_input; ajustar i_end_col / i_end_row
    CALL FUNCTION 'ALSM_EXCEL_TO_INTERNAL_TABLE'
      EXPORTING
        filename                = lv_fname
        i_begin_col             = 1
        i_begin_row             = 2
        i_end_col               = 4
        i_end_row               = 1000
      TABLES
        intern                  = lt_excel
      EXCEPTIONS
        inconsistent_parameters = 1
        upload_ole              = 2
        OTHERS                  = 3.

    IF sy-subrc NE 0.
      IF sy-msgty IS NOT INITIAL.
        MESSAGE ID sy-msgid TYPE 'S' NUMBER sy-msgno
                WITH sy-msgv1 sy-msgv2 sy-msgv3 sy-msgv4 DISPLAY LIKE 'E'.
      ENDIF.
      RAISE no_data.
    ENDIF.

    CREATE DATA lo_line LIKE LINE OF pe_input.
    ASSIGN lo_line->* TO <fs_input>.

    lv_numtot = lines( lt_excel ).

*   ALSM devuelve una línea por celda (row, col, value): se pivotea a ty_input
*   por posición de componente, así el mapeo es 1 columna Excel = 1 componente
    LOOP AT lt_excel ASSIGNING <fs_excel>.
      ADD 1 TO lv_numreg.
      lv_porcen = ( lv_numreg * 100 / lv_numtot ).
      mostrar_progreso( pi_percentage = lv_porcen pi_text = text-g01 ).

      AT NEW row.
        APPEND INITIAL LINE TO pe_input ASSIGNING <fs_input>.
      ENDAT.

      ASSIGN COMPONENT <fs_excel>-col OF STRUCTURE <fs_input> TO <fs_val>.
      IF sy-subrc EQ 0.
        <fs_val> = <fs_excel>-value.
      ENDIF.

      AT END OF row.
        UNASSIGN <fs_input>.
      ENDAT.
    ENDLOOP.

    IF lines( pe_input ) EQ 0.
      RAISE no_data.
    ENDIF.
  ENDMETHOD.
```

## 4.4 `pro_obtener_datos` con validación previa y semáforo

```abap
  METHOD pro_obtener_datos.
    DATA: lt_input TYPE tab_input.

    check_file( EXPORTING pi_filxls = mv_filxls EXCEPTIONS no_data = 1 ).
    IF sy-subrc NE 0.
      MESSAGE text-m03 TYPE 'S' DISPLAY LIKE 'E'.    " El archivo indicado no existe
      RAISE no_data.
    ENDIF.

    upload_file_excel( EXPORTING pi_filxls = mv_filxls
                       IMPORTING pe_input  = lt_input
                       EXCEPTIONS no_data  = 1 ).
    IF sy-subrc NE 0.
      MESSAGE text-m04 TYPE 'S' DISPLAY LIKE 'E'.    " Los datos cargados no son válidos
      RAISE no_data.
    ENDIF.

*   Validación masiva contra maestro (un solo SELECT, nunca dentro del LOOP)
    DATA(lt_input_tmp) = lt_input.
    SORT lt_input_tmp BY bname.
    DELETE ADJACENT DUPLICATES FROM lt_input_tmp COMPARING bname.

    IF lt_input_tmp IS NOT INITIAL.
      SELECT bname FROM usr02
        FOR ALL ENTRIES IN @lt_input_tmp
        WHERE bname EQ @lt_input_tmp-bname
        INTO TABLE @DATA(lt_usr02).
    ENDIF.

    LOOP AT lt_input INTO DATA(ls_input).
      APPEND INITIAL LINE TO mt_output ASSIGNING FIELD-SYMBOL(<fs_output>).
      MOVE-CORRESPONDING ls_input TO <fs_output>.
      <fs_output>-name_text = |{ <fs_output>-name_first } { <fs_output>-name_last }|.

      IF line_exists( lt_usr02[ bname = <fs_output>-bname ] ).
        <fs_output>-icono   = gc_icon_warning.
        <fs_output>-message = text-m05.              " Usuario indicado ya existe
      ENDIF.

      IF <fs_output>-icono IS INITIAL.
        <fs_output>-icono = gc_icon_inactive.
      ENDIF.
    ENDLOOP.
  ENDMETHOD.
```

## 4.5 Varios botones y ejecución por fila con BAPI

```abap
  METHOD on_user_command.
    CASE e_salv_function.
      WHEN '&PROCESO'.
        pro_ejecutar_proceso( pi_function = 'CREAR'   pi_alv = mo_alv ).
      WHEN '&SAP_ALL'.
        pro_ejecutar_proceso( pi_function = 'SAP_ALL' pi_alv = mo_alv ).
    ENDCASE.
  ENDMETHOD.

  METHOD pro_ejecutar_proceso.
    " ... obtener lt_rows y POPUP_TO_CONFIRM igual que en plantilla-reporte.md §3.4 ...

    LOOP AT lt_rows INTO DATA(lv_row).
      READ TABLE mt_output ASSIGNING FIELD-SYMBOL(<fs_output>) INDEX lv_row.
      CHECK sy-subrc EQ 0.
      CHECK <fs_output>-icono NE gc_icon_error.        " Nunca reprocesar errores

      CASE pi_function.
        WHEN 'CREAR'.
          CHECK <fs_output>-icono NE gc_icon_warning.  " No crear si ya existe
          pro_crear_usuario( CHANGING pe_output = <fs_output> ).
        WHEN 'SAP_ALL'.
          pro_asignar_perfil( CHANGING pe_output = <fs_output> ).
      ENDCASE.
    ENDLOOP.

    lo_columns = pi_alv->get_columns( ).
    lo_columns->set_optimize( abap_true ).
    pi_alv->refresh( ).
  ENDMETHOD.

  METHOD pro_crear_usuario.
    DATA: ls_logondata TYPE bapilogond,
          ls_address   TYPE bapiaddr3,
          ls_password  TYPE bapipwd,
          lt_return    TYPE TABLE OF bapiret2.

    ls_logondata-ustyp   = 'A'.                        " Usuario de diálogo
    ls_address-firstname = pe_output-name_first.
    ls_address-lastname  = pe_output-name_last.
    ls_address-e_mail    = pe_output-smtp_addr.

    CALL FUNCTION 'BAPI_USER_CREATE1'
      EXPORTING
        username           = pe_output-bname
        logondata          = ls_logondata
        password           = ls_password
        address            = ls_address
        generate_pwd       = abap_true
      IMPORTING
        generated_password = ls_password
      TABLES
        return             = lt_return.

*   Patrón de cierre: 1 BAPI = 1 COMMIT o 1 ROLLBACK, y el resultado se refleja en la fila
    READ TABLE lt_return INTO DATA(ls_return) WITH KEY type = 'E'.
    IF sy-subrc EQ 0.
      pe_output-icono   = gc_icon_error.
      pe_output-message = ls_return-message.
      CALL FUNCTION 'BAPI_TRANSACTION_ROLLBACK'.
    ELSE.
      pe_output-icono   = gc_icon_success.
      pe_output-message = text-m06.                    " Procesado correctamente
      CALL FUNCTION 'BAPI_TRANSACTION_COMMIT'
        EXPORTING
          wait = 'X'.
    ENDIF.
  ENDMETHOD.
```

En `set_column_setting` de esta variante se renombra la columna `ICONO` con textos *Estado* y el hotspot se pone sobre la clave del objeto (`BNAME` → `SU01` vía `SET PARAMETER ID 'XUS'`).

## 4.6 Lectura de Excel en S/4HANA

`ALSM_EXCEL_TO_INTERNAL_TABLE`, `KD_GET_FILENAME_ON_F4`, `SAPGUI_PROGRESS_INDICATOR` y `POPUP_TO_CONFIRM` funcionan en S/4HANA on-premise. `ALSM_*` requiere OLE en el frontend (SAP GUI para Windows): si el cliente lo prohíbe o la carga corre en fondo, leer el `.xlsx` con `cl_fdt_xl_spreadsheet` y marcar la llamada con `" TODO: verificar` según la regla de verificación de objetos estándar.
