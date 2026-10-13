---
name: abap-s4h-entregables
description: Estructura de carpetas y archivos del código ABAP por tipo RICEF en SAP S/4HANA (reports, interfaces, cargas, ampliaciones, formularios, CDS/RAP), plantilla del documento de configuración (CONFIG) e instrucciones de transporte y activación post-transporte. Usar al entregar código ABAP de un requerimiento o al preparar su configuración y transporte.
---

# Entregables de desarrollo ABAP — S/4HANA

Todos los entregables de un requerimiento viven en `docs/[ID-REQUERIMIENTO]/`. El código va en la subcarpeta `Code/`.
El `[ID-REQUERIMIENTO]` (ej. `PRODAC-SD003`) llega en el pedido. Si el proyecto define otra ruta, usar la del proyecto.

Nombres de objetos y extensiones de archivo: skill `abap-s4h-nomenclatura` (§10).

## 1. Estructura de `Code/` por tipo RICEF

### Report (R)
```
docs/[ID]/Code/
  ├── zdc[mod]_[nombre].abap          → Programa principal (cabecera + REPORT + INCLUDEs)
  ├── zdc[mod]_[nombre]_top.abap      → Tipos y estructura gs_selec
  ├── zdc[mod]_[nombre]_def.abap      → Definición de clases
  ├── zdc[mod]_[nombre]_imp.abap      → Implementación de clases
  ├── zdc[mod]_[nombre]_sel.abap      → Pantalla de selección
  ├── zdc[mod]_[nombre]_mai.abap      → Eventos
  ├── zdc[mod]_[nombre]_f01.abap      → Stub vacío
  └── zdc[mod]_[nombre]_txt.abap      → Catálogo de textos
```

### Interface (I)
```
docs/[ID]/Code/
  ├── zcl_dc[mod]_[nombre].abap       → Clase de procesamiento
  ├── zdc[mod]_fm_[nombre].abap       → Módulo de funciones RFC (si aplica)
  ├── zi_dc[mod]_[nombre].ddls.asddls → CDS de exposición (si aplica)
  ├── zui_dc[mod]_[nombre].srvd.srvdsrv / zapi_… → Service definition OData (si aplica)
  └── zdc[mod]_idoc_[nombre].abap     → Procesamiento IDoc (si aplica)
```

### Conversion (C)
```
docs/[ID]/Code/
  ├── zdc[mod]_conv_[nombre].abap        → Programa principal + includes (_TOP … _TXT, igual que R)
  ├── zdc[mod]_conv_[nombre]_tmpl.md     → Descripción de la plantilla de carga (columnas, formato)
  └── zdc[mod]_conv_[nombre]_map.md      → Mapeo campo origen → campo SAP / parámetro BAPI
```
Antes de programar una carga, confirmar que el **Migration Cockpit** (LTMC / *Migrate Your Data*) no cubre el objeto.

### Enhancement (E)
```
docs/[ID]/Code/
  ├── zcl_dc[mod]_badi_[nombre].abap  → Clase de implementación de BAdI
  ├── zcl_dc[mod]_[nombre].abap       → Lógica reutilizable llamada desde la BAdI
  └── zei_dc[mod]_[nombre].md         → Enhancement implementation: spot, BAdI, filtros y activación
```
No se usan user exits (SMOD/CMOD) ni enhancement points implícitos en desarrollos nuevos, salvo que no exista BAdI y el arquitecto lo apruebe.

### Form (F)
```
docs/[ID]/Code/
  ├── zdc[mod]_afi_[nombre].md        → Interfaz del formulario Adobe (parámetros, código de inicialización)
  ├── zdc[mod]_af_[nombre]_layout.md  → Descripción del layout (secciones, campos, condiciones)
  └── zcl_dc[mod]_[nombre]_out.abap   → Clase/programa de impresión o lógica de Output Management
```

### CDS / RAP (aplicaciones Fiori y APIs)
```
docs/[ID]/Code/
  ├── zdc[mod]_[nombre].md                    → Tabla(s) persistente(s) y draft
  ├── zr_dc[mod]_[nombre].ddls.asddls         → CDS root
  ├── zr_dc[mod]_[nombre].bdef.asbdef         → Behavior definition
  ├── zbp_r_dc[mod]_[nombre].abap             → Behavior pool
  ├── zc_dc[mod]_[nombre].ddls.asddls         → Projection
  ├── zc_dc[mod]_[nombre].bdef.asbdef         → Behavior projection
  ├── zc_dc[mod]_[nombre].ddlx.asddlxs        → Metadata extension (UI)
  ├── zui_dc[mod]_[nombre].srvd.srvdsrv       → Service definition
  └── zui_dc[mod]_[nombre]_o4.md              → Service binding (tipo, versión OData, publicación)
```

## 2. Entrega del código

Para cada archivo:
1. Código completo y que compile; nunca pseudocódigo.
2. Comentarios en la lógica compleja, con referencia a la regla de negocio (`" RN-003`).
3. Objetos SAP no confirmados marcados con `TODO` / `PENDIENTE` (regla de verificación de `abap-s4h-estandares`).
4. Al final, una tabla con todos los objetos creados: nombre, tipo, paquete y nivel de clean core.

## 3. Documento de configuración

**Ruta:** `docs/[ID]/CONFIG-[ID]-configuracion.md`

```markdown
## Pasos de Configuración

### Paso 1: [Título]
- **Transacción / App**: [SPRO / SM30 / SE93 / SE41 / app Fiori]
- **Ruta IMG**: [ruta completa si aplica]
- **Vista/Tabla**: [nombre]
- **Acción**: [Crear/Modificar entrada]
- **Valores**:
  | Campo | Valor | Descripción |
  |-------|-------|-------------|
  | [campo1] | [valor1] | [desc] |
```

Incluir siempre lo que el código necesita y no se transporta solo: transacción (SE93), status GUI (SE41), elementos de texto (si se crean a mano), objeto de log (SLG0), variantes y jobs (SM36).

## 4. Instrucciones de transporte

Documentar:
- **Orden workbench**: objetos de repositorio (paquete, diccionario, CDS, clases, programas, servicios).
- **Orden customizing**: entradas de configuración.
- **Secuencia de importación** cuando hay dependencias (diccionario antes que programas; CDS antes que behavior y servicio).
- **Actividades post-transporte** en S/4HANA:

| Objeto | Actividad | Dónde |
|---|---|---|
| Servicio OData V2 (SEGW o binding V2) | Registrar y activar el servicio | `/IWFND/MAINT_SERVICE` |
| Servicio OData V4 (binding V4) | Publicar el grupo de servicios | `/IWFND/V4_ADMIN` |
| Service binding RAP | Publicar en el sistema destino si no viaja publicado | ADT o la transacción correspondiente según versión |
| App Fiori | Catálogo, grupo/espacio y rol en el launchpad | Launchpad Designer / Content Manager, `PFCG` |
| Transacción | Asignar a roles | `PFCG` |
| Jobs | Programar con variante | `SM36` |

Verificar los nombres de transacciones de administración en la versión del cliente; varían entre releases.
