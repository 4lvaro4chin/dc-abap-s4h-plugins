# dc-abap-s4h-plugins

Marketplace de plugins de Claude Code de **Digital Core Pro** para programación ABAP en **SAP S/4HANA**.

## Plugins

### `dc-abap-s4h`

| Tipo | Nombre | Para qué |
|---|---|---|
| Skill | `abap-s4h-nomenclatura` | Nombres de objetos, variables y pantalla; prefijo configurable por cliente; límites de longitud |
| Skill | `abap-s4h-estandares` | Includes, patrón OO, sintaxis moderna, performance HANA, clean core, simplificación S/4HANA |
| Skill | `abap-s4h-alv-factory` | Reportes ALV con `cl_salv_table`, procesos con botón y cargas Excel con semáforo |
| Skill | `abap-s4h-selection-screen` | Include `_SEL` con todos los elementos de `SELECTION-SCREEN` |
| Skill | `abap-s4h-entregables` | Carpetas por RICEF, documento de configuración y transporte |
| Agente | `programador-abap-s4h` | Desarrolla a partir de la especificación funcional aplicando las skills |

## Instalación

En Claude Code:

```
/plugin marketplace add 4lvaro4chin/dc-abap-s4h-plugins
/plugin install dc-abap-s4h@dc-abap-s4h-plugins
```

Las skills quedan disponibles como `/dc-abap-s4h:abap-s4h-alv-factory`, etc. Claude también las activa solas cuando el pedido coincide con su descripción.

## Actualizar

Quien mantiene el plugin:
1. Edita las skills en `plugins/dc-abap-s4h/`.
2. Sube `version` en `plugins/dc-abap-s4h/.claude-plugin/plugin.json` (ej. `1.0.0` → `1.1.0`).
3. `git commit` + `git push`.

Quien lo usa:

```
/plugin marketplace update dc-abap-s4h-plugins
```

## Estructura

```
.claude-plugin/marketplace.json
plugins/dc-abap-s4h/
  .claude-plugin/plugin.json
  agents/programador-abap-s4h.md
  skills/<skill>/SKILL.md
  skills/<skill>/references/*.md
```
