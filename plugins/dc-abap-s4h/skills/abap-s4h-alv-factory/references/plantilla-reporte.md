# Plantilla completa (reporte de datos con botón de proceso)

Ejemplo de referencia: stock por centro/almacén/material (`MARD`). Reemplazar tablas, campos y textos según la especificación funcional. Sintaxis ABAP moderna de S/4HANA.

> En S/4HANA `MARD` es un agregado calculado desde `MATDOC`; para volúmenes altos evaluar la CDS de stock liberada (ver `abap-s4h-estandares` → `references/simplificacion-s4h.md`). El ejemplo se mantiene porque ilustra el patrón.

## 3.1 Programa principal `ZDC[MOD]_[NOMBRE]`

```abap
************************************************************************
*                         INFORMACION GENERAL
************************************************************************
* Programa...: ZDC[MOD]_[NOMBRE]
* Tipo.......: Reporte ALV
* Módulo.....: [MOD]
* Descripción: [Descripción del reporte]
* Autor......: Programador AI Digital Core
* Fecha......: [DD.MM.AAAA]
* Transacción: ZDC[MOD][NN]
************************************************************************
* Agregar Status bar para nuevos botones
* Tx.: SE41
* Programa: SALV_DEMO_TABLE_LAYOUT
* Status: SALV_STANDARD
************************************************************************
REPORT zdc[mod]_[nombre].

INCLUDE zdc[mod]_[nombre]_top.
INCLUDE zdc[mod]_[nombre]_def.
INCLUDE zdc[mod]_[nombre]_imp.
INCLUDE zdc[mod]_[nombre]_sel.
INCLUDE zdc[mod]_[nombre]_mai.
INCLUDE zdc[mod]_[nombre]_f01.
INCLUDE zdc[mod]_[nombre]_txt.
```

## 3.2 Include `_TOP`

Solo la estructura dummy que referencian los `SELECT-OPTIONS`. Los datos de negocio viven en la clase.

```abap
*&---------------------------------------------------------------------*
*& Include ZDC[MOD]_[NOMBRE]_TOP
*&---------------------------------------------------------------------*
* Estructura de referencia para SELECT-OPTIONS (no almacena datos)
TYPES: BEGIN OF ty_selec,
         werks TYPE werks_d,
         lgort TYPE lgort_d,
         matnr TYPE matnr,
         mtart TYPE mtart,
       END OF ty_selec.

DATA: gs_selec TYPE ty_selec.
```

## 3.3 Include `_DEF`

```abap
*&---------------------------------------------------------------------*
*& Include ZDC[MOD]_[NOMBRE]_DEF
*&---------------------------------------------------------------------*
CLASS lcl_reporte DEFINITION.
  PUBLIC SECTION.
*----------------------------------------------------------------------*
* TIPOS
*----------------------------------------------------------------------*
    TYPES: BEGIN OF ty_options,
             aggre TYPE xfeld,
             sorts TYPE xfeld,
             color TYPE xfeld,
           END OF ty_options.

    TYPES: BEGIN OF ty_output,
             werks      TYPE werks_d,
             name1      TYPE name1,
             lgort      TYPE lgort_d,
             lgobe      TYPE lgobe,
             matnr      TYPE matnr,
             maktx      TYPE maktx,
             mtart      TYPE mtart,
             meins      TYPE meins,
             labst      TYPE labst,
             set_colors TYPE lvc_t_scol,   " Columna de colores por fila/celda
           END OF ty_output.

    TYPES: ty_werks_rng TYPE RANGE OF werks_d,
           ty_lgort_rng TYPE RANGE OF lgort_d,
           ty_matnr_rng TYPE RANGE OF matnr,
           ty_mtart_rng TYPE RANGE OF mtart.

    TYPES: tab_output TYPE STANDARD TABLE OF ty_output WITH DEFAULT KEY.

*----------------------------------------------------------------------*
* CONSTANTES
*----------------------------------------------------------------------*
    CONSTANTS: gc_icon_inactive TYPE icon_d VALUE '@BZ@',
               gc_icon_success  TYPE icon_d VALUE '@5B@',
               gc_icon_error    TYPE icon_d VALUE '@5C@',
               gc_icon_warning  TYPE icon_d VALUE '@5D@'.

    CONSTANTS: gc_fcode_proceso TYPE salv_de_function VALUE '&PROCESO'.

*----------------------------------------------------------------------*
* MÉTODOS PÚBLICOS
*----------------------------------------------------------------------*
    CLASS-METHODS:
      match_code_variant RETURNING VALUE(pe_variant) TYPE slis_vari.

    METHODS:
      constructor IMPORTING pi_werks TYPE ty_werks_rng OPTIONAL
                            pi_lgort TYPE ty_lgort_rng OPTIONAL
                            pi_matnr TYPE ty_matnr_rng OPTIONAL
                            pi_mtart TYPE ty_mtart_rng OPTIONAL
                            pi_aggre TYPE xfeld OPTIONAL
                            pi_sorts TYPE xfeld OPTIONAL
                            pi_color TYPE xfeld OPTIONAL.

    METHODS:
      pro_obtener_datos EXCEPTIONS no_data.

    METHODS:
      mostrar_alv IMPORTING pi_variant TYPE slis_vari.

  PRIVATE SECTION.
*----------------------------------------------------------------------*
* ATRIBUTOS
*----------------------------------------------------------------------*
    DATA: mt_output TYPE tab_output.

    DATA: mt_werks_rng TYPE ty_werks_rng,
          mt_lgort_rng TYPE ty_lgort_rng,
          mt_matnr_rng TYPE ty_matnr_rng,
          mt_mtart_rng TYPE ty_mtart_rng.

    DATA: ms_options TYPE ty_options.

    DATA: mo_alv TYPE REF TO cl_salv_table.

*----------------------------------------------------------------------*
* MÉTODOS PRIVADOS
*----------------------------------------------------------------------*
    METHODS:
      set_pf_status CHANGING pi_alv TYPE REF TO cl_salv_table.

    METHODS:
      set_layout IMPORTING pi_variant TYPE slis_vari
                 CHANGING  pi_alv     TYPE REF TO cl_salv_table.

    METHODS:
      get_handle_events CHANGING pi_alv TYPE REF TO cl_salv_table.

    METHODS:
      on_user_command FOR EVENT added_function OF cl_salv_events
        IMPORTING e_salv_function.

    METHODS:
      on_link_click FOR EVENT link_click OF cl_salv_events_table
        IMPORTING row column.

    METHODS:
      set_display_setting CHANGING pi_alv TYPE REF TO cl_salv_table.

    METHODS:
      set_column_setting CHANGING pi_alv TYPE REF TO cl_salv_table.

    METHODS:
      set_hotspot CHANGING pi_alv TYPE REF TO cl_salv_table.

    METHODS:
      set_selections CHANGING pi_alv TYPE REF TO cl_salv_table.

    METHODS:
      set_aggregations CHANGING pi_alv TYPE REF TO cl_salv_table.

    METHODS:
      set_sorts CHANGING pi_alv TYPE REF TO cl_salv_table.

    METHODS:
      set_colors CHANGING pi_alv TYPE REF TO cl_salv_table.

    METHODS:
      pro_ejecutar_proceso IMPORTING pi_alv TYPE REF TO cl_salv_table.
ENDCLASS.
```

## 3.4 Include `_IMP`

```abap
*&---------------------------------------------------------------------*
*& Include ZDC[MOD]_[NOMBRE]_IMP
*&---------------------------------------------------------------------*
CLASS lcl_reporte IMPLEMENTATION.

  METHOD match_code_variant.
    DATA: ls_layout_in  TYPE disvariant,
          ls_layout_out TYPE disvariant,
          lv_exit       TYPE c.

    ls_layout_in-report = sy-repid.

    CALL FUNCTION 'LVC_VARIANT_F4'
      EXPORTING
        is_variant = ls_layout_in
        i_save     = 'A'
      IMPORTING
        e_exit     = lv_exit
        es_variant = ls_layout_out
      EXCEPTIONS
        not_found  = 1
        OTHERS     = 2.

    IF sy-subrc NE 0.
      MESSAGE ID sy-msgid TYPE sy-msgty NUMBER sy-msgno
              WITH sy-msgv1 sy-msgv2 sy-msgv3 sy-msgv4.
    ELSEIF lv_exit NE abap_true.
      pe_variant = ls_layout_out-variant.
    ENDIF.
  ENDMETHOD.

  METHOD constructor.
*   Parámetros de la pantalla de selección → atributos privados
    mt_werks_rng = pi_werks.
    mt_lgort_rng = pi_lgort.
    mt_matnr_rng = pi_matnr.
    mt_mtart_rng = pi_mtart.

    ms_options-aggre = pi_aggre.
    ms_options-sorts = pi_sorts.
    ms_options-color = pi_color.
  ENDMETHOD.

  METHOD pro_obtener_datos.
    SELECT a~werks, c~name1, a~lgort, d~lgobe, a~matnr, e~maktx, a~labst,
           b~mtart, b~meins
      INTO TABLE @DATA(lt_data)
      FROM mard AS a
     INNER JOIN mara  AS b ON b~matnr EQ a~matnr
     INNER JOIN t001w AS c ON c~werks EQ a~werks
     INNER JOIN t001l AS d ON d~werks EQ a~werks
                          AND d~lgort EQ a~lgort
     INNER JOIN makt  AS e ON e~matnr EQ a~matnr
     WHERE a~matnr IN @mt_matnr_rng
       AND a~werks IN @mt_werks_rng
       AND a~lgort IN @mt_lgort_rng
       AND b~mtart IN @mt_mtart_rng
       AND e~spras EQ @sy-langu.

    IF sy-subrc NE 0.
      MESSAGE text-m01 TYPE 'S' DISPLAY LIKE 'E'.
      RAISE no_data.
    ENDIF.

    LOOP AT lt_data INTO DATA(ls_data).
      APPEND INITIAL LINE TO mt_output ASSIGNING FIELD-SYMBOL(<fs_output>).
      MOVE-CORRESPONDING ls_data TO <fs_output>.
    ENDLOOP.
  ENDMETHOD.

  METHOD mostrar_alv.
    DATA: lx_msg TYPE REF TO cx_salv_msg.

    TRY.
        cl_salv_table=>factory(
          IMPORTING
            r_salv_table = mo_alv
          CHANGING
            t_table      = mt_output ).
      CATCH cx_salv_msg INTO lx_msg.
        MESSAGE lx_msg->get_text( ) TYPE 'S' DISPLAY LIKE 'E'.
        RETURN.
    ENDTRY.

    set_pf_status( CHANGING pi_alv = mo_alv ).
    set_layout( EXPORTING pi_variant = pi_variant CHANGING pi_alv = mo_alv ).
    get_handle_events( CHANGING pi_alv = mo_alv ).
    set_display_setting( CHANGING pi_alv = mo_alv ).
    set_column_setting( CHANGING pi_alv = mo_alv ).
    set_hotspot( CHANGING pi_alv = mo_alv ).
    set_selections( CHANGING pi_alv = mo_alv ).

    IF ms_options-aggre IS NOT INITIAL.
      set_aggregations( CHANGING pi_alv = mo_alv ).
    ENDIF.

    IF ms_options-sorts IS NOT INITIAL.
      set_sorts( CHANGING pi_alv = mo_alv ).
    ENDIF.

    IF ms_options-color IS NOT INITIAL.
      set_colors( CHANGING pi_alv = mo_alv ).
    ENDIF.

    mo_alv->display( ).
  ENDMETHOD.

  METHOD set_pf_status.
*   Status copiado de SALV_DEMO_TABLE_LAYOUT / SALV_STANDARD (SE41) con botones propios
    pi_alv->set_screen_status( pfstatus      = 'SALV_STANDARD'
                               report        = sy-repid
                               set_functions = pi_alv->c_functions_all ).

*   Alternativa sin botones propios (no requiere status en SE41):
*    DATA(lo_functions) = pi_alv->get_functions( ).
*    lo_functions->set_all( abap_true ).
  ENDMETHOD.

  METHOD set_layout.
    DATA: lo_layout  TYPE REF TO cl_salv_layout,
          ls_key     TYPE salv_s_layout_key,
          lv_variant TYPE slis_vari.

    lo_layout = pi_alv->get_layout( ).

    ls_key-report = sy-repid.
    lo_layout->set_key( ls_key ).

*   Permite grabar variantes de usuario y globales
    lo_layout->set_save_restriction( if_salv_c_layout=>restrict_none ).

    IF pi_variant IS NOT INITIAL.
      lv_variant = pi_variant.
      lo_layout->set_initial_layout( lv_variant ).
    ENDIF.
  ENDMETHOD.

  METHOD get_handle_events.
    DATA: lo_events TYPE REF TO cl_salv_events_table.

    lo_events = pi_alv->get_event( ).

    SET HANDLER me->on_user_command FOR lo_events.
    SET HANDLER me->on_link_click   FOR lo_events.
  ENDMETHOD.

  METHOD on_user_command.
    CASE e_salv_function.
      WHEN gc_fcode_proceso.
        pro_ejecutar_proceso( pi_alv = mo_alv ).
    ENDCASE.
  ENDMETHOD.

  METHOD on_link_click.
    READ TABLE mt_output INTO DATA(ls_output) INDEX row.
    IF sy-subrc EQ 0.
      CASE column.
        WHEN 'MATNR'.
          SET PARAMETER ID 'MAT' FIELD ls_output-matnr.
          CALL TRANSACTION 'MM03' AND SKIP FIRST SCREEN.
      ENDCASE.
    ENDIF.
  ENDMETHOD.

  METHOD set_display_setting.
    DATA: lo_display TYPE REF TO cl_salv_display_settings.

    lo_display = pi_alv->get_display_settings( ).

    lo_display->set_striped_pattern( abap_true ).
    lo_display->set_list_header( text-t01 ).
  ENDMETHOD.

  METHOD set_column_setting.
    DATA: lo_columns TYPE REF TO cl_salv_columns,
          lo_column  TYPE REF TO cl_salv_column.

    lo_columns = pi_alv->get_columns( ).
    lo_columns->set_optimize( abap_true ).

*   Texto de cabecera de columna (para campos sin elemento de datos o con texto distinto)
    TRY.
        lo_column = lo_columns->get_column( 'MAKTX' ).
        lo_column->set_long_text( text-c01 ).
        lo_column->set_medium_text( text-c02 ).
        lo_column->set_short_text( text-c03 ).
      CATCH cx_salv_not_found.
    ENDTRY.

*   Ocultar columnas
    TRY.
        lo_column = lo_columns->get_column( 'NAME1' ).
        lo_column->set_visible( abap_false ).
      CATCH cx_salv_not_found.
    ENDTRY.

    TRY.
        lo_column = lo_columns->get_column( 'LGOBE' ).
        lo_column->set_visible( abap_false ).
      CATCH cx_salv_not_found.
    ENDTRY.

*   La columna técnica de colores nunca se muestra
    TRY.
        lo_column = lo_columns->get_column( 'SET_COLORS' ).
        lo_column->set_technical( abap_true ).
      CATCH cx_salv_not_found.
    ENDTRY.
  ENDMETHOD.

  METHOD set_hotspot.
    DATA: lo_columns TYPE REF TO cl_salv_columns_table,
          lo_column  TYPE REF TO cl_salv_column_table.

    lo_columns = pi_alv->get_columns( ).

    TRY.
        lo_column ?= lo_columns->get_column( 'MATNR' ).
        TRY.
            lo_column->set_cell_type( if_salv_c_cell_type=>hotspot ).
          CATCH cx_salv_data_error.
        ENDTRY.
      CATCH cx_salv_not_found.
    ENDTRY.
  ENDMETHOD.

  METHOD set_selections.
    DATA: lo_selections TYPE REF TO cl_salv_selections.

    lo_selections = pi_alv->get_selections( ).

*   row_column: muestra la columna de marcado y permite seleccionar filas y columnas
    lo_selections->set_selection_mode( if_salv_c_selection_mode=>row_column ).
  ENDMETHOD.

  METHOD set_aggregations.
    DATA: lo_aggrs TYPE REF TO cl_salv_aggregations.

    lo_aggrs = pi_alv->get_aggregations( ).

    TRY.
        lo_aggrs->add_aggregation( columnname  = 'LABST'
                                   aggregation = if_salv_c_aggregation=>total ).
      CATCH cx_salv_data_error cx_salv_not_found cx_salv_existing.
    ENDTRY.

*   Totales arriba de los ítems (omitir para mostrarlos al final)
    lo_aggrs->set_aggregation_before_items( ).
  ENDMETHOD.

  METHOD set_sorts.
    DATA: lo_sorts TYPE REF TO cl_salv_sorts.

    lo_sorts = pi_alv->get_sorts( ).
    lo_sorts->clear( ).

    TRY.
        lo_sorts->add_sort( columnname = 'WERKS'
                            position   = 1
                            subtotal   = abap_true
                            sequence   = if_salv_c_sort=>sort_up ).
      CATCH cx_salv_not_found cx_salv_data_error cx_salv_existing.
    ENDTRY.

    TRY.
        lo_sorts->add_sort( columnname = 'MTART'
                            position   = 2
                            subtotal   = abap_true
                            sequence   = if_salv_c_sort=>sort_up ).
      CATCH cx_salv_not_found cx_salv_data_error cx_salv_existing.
    ENDTRY.
  ENDMETHOD.

  METHOD set_colors.
    INCLUDE <color>.

    DATA: lo_columns TYPE REF TO cl_salv_columns_table,
          lo_column  TYPE REF TO cl_salv_column_table,
          ls_color   TYPE lvc_s_colo,
          lt_colors  TYPE lvc_t_scol.

    lo_columns = pi_alv->get_columns( ).

*   a) Colorear toda una columna
    TRY.
        lo_column ?= lo_columns->get_column( 'MEINS' ).
        ls_color-col = col_heading.
        lo_column->set_color( ls_color ).
      CATCH cx_salv_not_found.
    ENDTRY.

*   b) Colorear fila completa o celda según condición de negocio
    LOOP AT mt_output ASSIGNING FIELD-SYMBOL(<fs_output>).
      CLEAR lt_colors.

      IF <fs_output>-labst GT 0.
*       fname informado → solo la celda
        APPEND INITIAL LINE TO lt_colors ASSIGNING FIELD-SYMBOL(<fs_color>).
        <fs_color>-fname     = 'LABST'.
        <fs_color>-color-col = col_positive.
        <fs_color>-color-int = 0.
        <fs_color>-color-inv = 0.
      ELSE.
*       fname vacío → toda la fila
        APPEND INITIAL LINE TO lt_colors ASSIGNING <fs_color>.
        <fs_color>-color-col = col_negative.
        <fs_color>-color-int = 0.
        <fs_color>-color-inv = 0.
      ENDIF.

      <fs_output>-set_colors = lt_colors.
    ENDLOOP.

    TRY.
        lo_columns->set_color_column( 'SET_COLORS' ).
      CATCH cx_salv_data_error.                          "#EC NO_HANDLER
    ENDTRY.
  ENDMETHOD.

  METHOD pro_ejecutar_proceso.
    DATA: lo_selections TYPE REF TO cl_salv_selections,
          lo_columns    TYPE REF TO cl_salv_columns,
          lt_rows       TYPE salv_t_row,
          lv_answer     TYPE c.

    lo_selections = pi_alv->get_selections( ).
    lt_rows = lo_selections->get_selected_rows( ).

    IF lines( lt_rows ) EQ 0.
      MESSAGE text-m02 TYPE 'S' DISPLAY LIKE 'E'.
      RETURN.
    ENDIF.

    CALL FUNCTION 'POPUP_TO_CONFIRM'
      EXPORTING
        titlebar       = text-p01
        text_button_1  = text-p02
        text_button_2  = text-p03
        text_question  = text-p04
        default_button = '2'
      IMPORTING
        answer         = lv_answer
      EXCEPTIONS
        text_not_found = 1.

    CHECK lv_answer EQ '1'.

    LOOP AT lt_rows INTO DATA(lv_row).
      READ TABLE mt_output ASSIGNING FIELD-SYMBOL(<fs_output>) INDEX lv_row.
      CHECK sy-subrc EQ 0.
*     Lógica de negocio sobre la fila marcada
*     ...
    ENDLOOP.

*   Reoptimizar anchos y refrescar la grilla con los datos modificados
    lo_columns = pi_alv->get_columns( ).
    lo_columns->set_optimize( abap_true ).
    pi_alv->refresh( ).
  ENDMETHOD.

ENDCLASS.
```

## 3.5 Include `_SEL`

```abap
*&---------------------------------------------------------------------*
*& Include ZDC[MOD]_[NOMBRE]_SEL
*&---------------------------------------------------------------------*
SELECTION-SCREEN BEGIN OF BLOCK b01 WITH FRAME TITLE TEXT-b01.
  SELECT-OPTIONS: s_werks FOR gs_selec-werks,
                  s_lgort FOR gs_selec-lgort,
                  s_matnr FOR gs_selec-matnr.
  SELECTION-SCREEN SKIP 1.
  SELECT-OPTIONS: s_mtart FOR gs_selec-mtart.
  PARAMETERS:     p_varian TYPE disvariant-variant.
SELECTION-SCREEN END OF BLOCK b01.

SELECTION-SCREEN BEGIN OF BLOCK b02 WITH FRAME TITLE TEXT-b02.
  PARAMETERS: p_aggre AS CHECKBOX,
              p_sorts AS CHECKBOX,
              p_color AS CHECKBOX.
SELECTION-SCREEN END OF BLOCK b02.

AT SELECTION-SCREEN ON VALUE-REQUEST FOR p_varian.
  p_varian = lcl_reporte=>match_code_variant( ).
```

Los checkboxes `p_aggre / p_sorts / p_color` son del programa modelo para demostrar las opciones. En un desarrollo real se **eliminan** y se decide en la especificación funcional si el ALV lleva totales, subtotales o colores de forma fija.

## 3.6 Include `_MAI`

```abap
*&---------------------------------------------------------------------*
*& Include ZDC[MOD]_[NOMBRE]_MAI
*&---------------------------------------------------------------------*
START-OF-SELECTION.
  DATA: go_reporte TYPE REF TO lcl_reporte.

  go_reporte = NEW lcl_reporte( pi_werks = s_werks[]
                                pi_lgort = s_lgort[]
                                pi_matnr = s_matnr[]
                                pi_mtart = s_mtart[]
                                pi_aggre = p_aggre
                                pi_sorts = p_sorts
                                pi_color = p_color ).

  go_reporte->pro_obtener_datos( EXCEPTIONS no_data = 1 ).

  IF sy-subrc EQ 0.
    go_reporte->mostrar_alv( p_varian ).
  ENDIF.
```

## 3.7 Include `_TXT` (catálogo de textos a documentar)

| ID | Tipo | Texto |
|---|---|---|
| `B01` | Text symbol | Criterios de selección |
| `B02` | Text symbol | Opciones de visualización |
| `T01` | Text symbol | Título del listado |
| `C01` / `C02` / `C03` | Text symbol | Texto largo / medio / corto de la columna renombrada |
| `M01` | Text symbol | No se encontraron datos para los parámetros ingresados |
| `M02` | Text symbol | Antes de ejecutar la función es necesario marcar las entradas |
| `P01` | Text symbol | Confirmar |
| `P02` / `P03` | Text symbol | Sí / No |
| `P04` | Text symbol | Se ejecutará el proceso. ¿Desea continuar? |
| `S_WERKS`… | Selection text | Etiquetas de la pantalla de selección |
