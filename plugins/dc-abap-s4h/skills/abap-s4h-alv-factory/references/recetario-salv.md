# Recetario de opciones `cl_salv_table`

Cada receta va en su propio método `set_*`. Todas las llamadas `get_column` van dentro de `TRY … CATCH cx_salv_not_found`.

| Necesidad | Objeto | Código |
|---|---|---|
| Todas las funciones estándar (sin botones propios) | `cl_salv_functions_list` | `pi_alv->get_functions( )->set_all( abap_true )` |
| Botones propios | `cl_salv_table` | `set_screen_status( pfstatus = 'SALV_STANDARD' report = sy-repid set_functions = c_functions_all )` |
| Título del listado | `cl_salv_display_settings` | `set_list_header( text-t01 )` |
| Filas rayadas | `cl_salv_display_settings` | `set_striped_pattern( abap_true )` |
| Optimizar ancho | `cl_salv_columns` | `set_optimize( abap_true )` |
| Renombrar cabecera | `cl_salv_column` | `set_long_text / set_medium_text / set_short_text` |
| Ocultar columna | `cl_salv_column` | `set_visible( abap_false )` |
| Columna técnica (nunca visible ni en layout) | `cl_salv_column` | `set_technical( abap_true )` |
| Posición de columna | `cl_salv_columns` | `set_column_position( columnname = 'X' position = 1 )` |
| Clave (fijar a la izquierda) | `cl_salv_column_table` | `set_key( abap_true )` |
| Hotspot | `cl_salv_column_table` | `set_cell_type( if_salv_c_cell_type=>hotspot )` |
| Checkbox visual | `cl_salv_column_table` | `set_cell_type( if_salv_c_cell_type=>checkbox )` |
| Icono en columna | `cl_salv_column_table` | `set_icon( if_salv_c_bool_sap=>true )` (campo tipo `icon_d`) |
| Referencia a moneda / unidad | `cl_salv_column_table` | `set_currency_column( 'WAERS' )` / `set_quantity_column( 'MEINS' )` |
| Total | `cl_salv_aggregations` | `add_aggregation( columnname = 'X' aggregation = if_salv_c_aggregation=>total )` |
| Totales arriba | `cl_salv_aggregations` | `set_aggregation_before_items( )` |
| Orden + subtotal | `cl_salv_sorts` | `add_sort( columnname = 'X' position = n subtotal = abap_true sequence = if_salv_c_sort=>sort_up )` |
| Color de columna | `cl_salv_column_table` | `set_color( ls_color )` con `lvc_s_colo` |
| Color fila / celda | `cl_salv_columns_table` | Campo `lvc_t_scol` en la salida + `set_color_column( 'SET_COLORS' )` |
| Selección de filas | `cl_salv_selections` | `set_selection_mode( if_salv_c_selection_mode=>row_column )` |
| Filas marcadas | `cl_salv_selections` | `get_selected_rows( )` → `salv_t_row` |
| Variante de layout | `cl_salv_layout` | `set_key / set_save_restriction / set_initial_layout` |
| Refrescar tras modificar datos | `cl_salv_table` | `refresh( )` (con `refresh_mode = if_salv_c_refresh=>full` si cambian columnas) |
| Cabecera Top-of-page | `cl_salv_form_layout_grid` | `set_top_of_list( lo_grid )` |

## Modos de selección disponibles

| Constante `if_salv_c_selection_mode` | Comportamiento |
|---|---|
| `none` | Sin selección |
| `single` | Una fila, sin columna de marcado |
| `multiple` | Varias filas, sin columna de marcado |
| `cell` | Celdas individuales |
| `row_column` | Columna de marcado a la izquierda; filas y columnas (**el estándar del patrón**) |

## Colores (`INCLUDE <color>`)

| Constante | Nº | Uso habitual |
|---|---|---|
| `col_heading` | 1 | Cabecera / columnas clave |
| `col_normal` | 2 | Neutro |
| `col_total` | 3 | Totales |
| `col_key` | 4 | Clave |
| `col_positive` | 5 | Verde: OK / stock disponible |
| `col_negative` | 6 | Rojo: error / faltante |
| `col_group` | 7 | Agrupación |

En `lvc_s_scol`: `fname` vacío colorea toda la fila; `fname = 'CAMPO'` colorea solo esa celda. `int` = intensificado (0/1), `inv` = inverso (0/1).

## Eventos de `cl_salv_events_table`

| Evento | Parámetros | Uso |
|---|---|---|
| `added_function` | `e_salv_function` | Botones propios del status |
| `link_click` | `row`, `column` | Hotspot |
| `double_click` | `row`, `column` | Navegación alternativa sin hotspot |
| `before_salv_function` / `after_salv_function` | `e_salv_function` | Interceptar funciones estándar (ej. `&REFRESH`) |
| `top_of_page` / `end_of_page` | `r_top_of_page` | Cabecera/pie dinámicos |
