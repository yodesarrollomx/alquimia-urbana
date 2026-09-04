# Alquimia Urbana — *board técnico del grupo: el Gantt de trámites de los desarrollos*

Contexto para Claude Code. Lee este archivo completo antes de tocar nada.

## Qué es
Board grupal de **PG Arquitectos (técnico) · Aurum Arquitectos (diseño) · YoDesarrollo (desarrollo)**. Una sola pantalla: hero "¿dónde estamos?" (avance / el cuello / ritmo / dinero) + Gantt de **trámites** agrupado por etapa, con flechas de dependencia, línea HOY y ficha lateral por trámite (`index.html:339` `pintar()`, `index.html:416` `pintarGantt()`).

- **Solo trámites, estudios y entregables.** Las tareas viven en el board de tareas de YoDesarrollo/Aurum (dicho en el pie del propio board, `index.html:138`).
- **Quién lo usa:** los responsables de cada proyecto del grupo. Entra con la clave única del Sheet Registro o con liga mágica del Portero.
- **Dirección en vivo:** `https://yodesarrollomx.github.io/alquimia-urbana/` — **HTTP 200 verificado el 2026-09-04**.
- La casa vieja `https://alexpueblag.github.io/alquimia-urbana/` responde **200** pero ya solo es cascarón: `<meta refresh>` + `location.replace` al dominio nuevo (verificado 2026-09-04; commit `0b10299` "mudanza 1-sep").
- `https://tableros.yodesarrollo.mx/alquimia-urbana/` da **000 (no resuelve)** el 2026-09-04 — el DNS del dominio propio todavía no existe. ~~El commit `654f93c` (27-ago) apuntó portero.js a tableros.yodesarrollo.mx~~ **OBSOLETO desde 2026-09-01**: `0b10299` lo regresó a `yodesarrollomx.github.io` (`index.html:576`).

## Reglas INVIOLABLES
1. **El Sheet Registro manda.** Agregar un proyecto = agregar una fila en `PROYECTOS`. Nada de proyectos en código (`gas/Code.gs:22-27`). Si lo hardcodeas, el grupo pierde su forma de administrar.
2. **La clave se valida EN EL SERVIDOR** contra `CONFIG.clave` del Registro (`gas/Code.gs:139-141`). El HTML nunca trae la clave. Si la metes al front, la clave queda pública en un repo público.
3. **El payload es ALLOWLIST.** De CONFIG solo salen `CFG_PUBLICAS` (`gas/Code.gs:78`); de cada fila se borran `sheet_id`, `crear_carpetas`, `notas` (`gas/Code.gs:179-182`). Una key nueva del Sheet NO es pública hasta que alguien la agregue a propósito.
4. **Lo financiero de los proyectos JAMÁS se lee.** `liveResumen_` solo toca HITOS/TRAMITES/CONFIG; COSTOS/REPARTO no se abren (`gas/Code.gs:200`). Excepciones deliberadas documentadas en el encabezado del .gs: `live.precio_m2` del hito actual y `live.tramites` completo para el Gantt (`gas/Code.gs:33-42`).
5. **Se redactan los nombres privados** de cada proyecto (`redactarLive_`, `gas/Code.gs:296`): lo que el proyecto marcó privado sale como "un codesarrollador". Romperlo expone codesarrolladores.
6. **Solo URLs https** se pintan (`safeUrl`, `index.html:153`) — blindaje anti `javascript:`. No lo quites.
7. **El repo es ESPEJO del front, y `gas/` ni siquiera está en el repo** (ver Arquitectura). Nunca supongas que el archivo local es lo que corre.
8. **`noindex`** en el head (`index.html:6`). No lo borres: el board no es público.

## Archivos
- `index.html` (578 líneas) — todo el board: estilos propios del Gantt + gate de clave + `planear()` (infiere fechas por `depende_de`) + `cuello()` (la restricción tipo Goldratt/Hormozi) + drawer de ficha. Carga `portero.js` al final.
- `assets/estilo.css` (255 líneas) — piel compartida "Fintech Editorial" (oscuro cálido + oro, Instrument Serif + Manrope). Solo presentación; los datos siempre vienen del Sheet (así lo dice su propio encabezado).
- `fonts/Kefa-Regular.ttf` — resto de la v1 (2026-07-02, commits `2a9d28d`/`f26cadd`). No lo referencia el CSS actual (grep sin resultados) — **por confirmar** si se puede borrar.
- `gas/Code.gs` (811 líneas) + `gas/appsscript.json` + `gas/.clasp.json` — **NO están versionados**: `git ls-files` solo lista `assets/estilo.css`, `fonts/Kefa-Regular.ttf`, `index.html`, y `.git/info/exclude` trae `gas/` (decisión de la mudanza, ver [[rename-github-yodesarrollomx]] línea 84). Copia local de trabajo, nada más.

## Arquitectura de datos

```
Sheet "Alquimia Urbana - Registro"  (id 1peT5oM17danrCkbO5_Zzw5fNogh7cenMJ3KfiEOLT5M — memoria alquimia-urbana-plataforma)
  · CONFIG   → clave única + titulo/subtitulo/nota_uso/subcarpetas   (solo CFG_PUBLICAS viajan)
  · PROYECTOS→ 17 columnas HDR_PROY (gas/Code.gs:71): nombre, etapa, presupuesto,
               pagado, board_url, sheet_id, carpeta_drive_url, crear_carpetas, activo
        │
        ├─ fila con crear_carpetas=si y carpeta_drive_url vacío
        │     └→ ensureCarpetas_() crea las 7 subcarpetas en Drive y ESCRIBE la URL de vuelta
        │
        └─ fila con sheet_id (Sheet Maestro estilo Miramar)
              └→ liveResumen_() lee HITOS + TRAMITES + CONFIG de ESE sheet
                 → etapa actual, candados, conteos, urgentes, último cierre,
                   escalera y los trámites del Gantt (con edit_url a su fila)
                 → redactarLive_() antes de salir

  GAS (standalone "Alquimia Urbana - GAS")  ── caché 300 s (alq_grupo_v2) ──┐
  GET /exec?recurso=grupo&k=<clave>                                          │
  GET /exec?recurso=meta   (ping; responde {"ok":true,...} — probado 2026-09-04)
                                                                             ▼
                              index.html  ── fetch (timeout 15 s) ──→ pinta hero + Gantt
                              clave guardada en localStorage "au_clave_v1" (index.html:147)
                              &refrescar=1 vacía la caché (gas/Code.gs:143-145)

  portero.js (yodesarrollomx.github.io/potenciales-yod/portero.js, data-gate="suave")
     → inyecta "entrar con mi correo" + tema claro/oscuro; NO tapa el gate propio
  porteroOk_(k) (gas/Code.gs:87) lee DIRECTO el Sheet del Portero 1Ld2ytzw…
     hojas SESIONES + ACCESOS; el token debe traer board "AL" o "*"
```

**URL /exec en producción:** `https://script.google.com/macros/s/AKfycbwIkonz5lbmtdQQw33foXc_5DRDwoTiuaJgoM7LQILtRN1lx9uFUAhb-jgJhCnRzfPQ/exec` (`index.html:146`). Web App `ANYONE_ANONYMOUS`, ejecuta como el dueño (`gas/appsscript.json:12`) — la puerta real es la clave, no el acceso del deployment.

> ⚠ **EL REPO ES ESPEJO. Lo que corre es lo pegado en el editor de Apps Script.**
> `gas/Code.gs` es una copia local (además, sin versionar). Antes de tocar el backend: **pide el Code.gs vivo del editor** ([[backend-vivo-no-es-el-repo]]). Tras cambiarlo: pegar en el editor y publicar "Nueva versión" en Administrar implementaciones — la URL /exec no cambia (`gas/Code.gs:48-50`).
> El front SÍ es espejo fiel hoy: el `index.html` desplegado es **byte-idéntico** al local (curl + diff, 2026-09-04).

## Decisiones
- **2026-07-02 · Alejandro** — v1 en producción: board **funcional**, no imagen bonita; los responsables lo llenan; **UNA sola clave**; se entra igual que los otros boards; sin NEX; usar su Drive. (memoria [[alquimia-urbana-plataforma]])
- **2026-07-02 noche · Alejandro** — "muy disperso… solo trámites, flechas de vínculos + tiempo en la misma gráfica, clickeable y editable". Nace la v2: **el hub ES un Gantt de trámites**. Se elimina "Pendientes por responsable" porque eran tareas, y las tareas van en el board de tareas. (misma memoria)
- **2026-07-02 · Alejandro** — excepción deliberada: `live.tramites` y `edit_url` a la fila exacta del Sheet Maestro viajan al hub, pedidos por el dueño. El Sheet sigue protegido por permisos de Drive. (`gas/Code.gs:36-42`)
- **2026-07-03 · Alejandro** — Miramar migra a **exactamente 2 hojas** (PROYECTO con bloques `## …` + TRAMITES super-tabla con columna `tipo`), sin perder datos. Aquí quedó el adaptador `detectarFormatoAlq_`/`expandirVirtualAlq_` (`gas/Code.gs:497-570`): si el Sheet tiene hoja PROYECTO se lee v3, si no, lectura normal. Paridad verificada 0 celdas. (memoria)
- **2026-07-03 · "me parece bien" (Alejandro)** — skin institucional YoDesarrollo aprobado; dummy en `~/alquimia-dummy-primera-pantalla.html`. **Por confirmar** si ya se aplicó: el board hoy trae la piel Fintech Editorial oscura (`assets/estilo.css:1-12`), no el marino #0F1F3D del dummy.
- **2026-07-12 · sistema** — Alquimia se migra al Portero: además de la clave, un token de sesión vivo abre el board; `porteroOk_` exige board **"AL"** o `*` (GAS @26, memoria [[potenciales-yod-project]] líneas 344-349).
- **2026-08-02/03 · auditoría de contraste** — tokens de tema propios (`--inv`, `--tinta-fija`, `--cremaf`, `--sticky`, `--amar-tx`, `--rojo-tx`) y llave `[data-tema="claro"]` en `index.html`, porque portero voltea la paleta pero **no** `--texto`/`--surface2`. Claro 18→0 fallas. `?v=yod4` para romper caché (commits `1fc360f`, `26bb760`, `7e449c2`).
- **2026-09-01 · mudanza** — los tableros viven en `yodesarrollomx.github.io`; la puerta vieja reenvía (commit `0b10299`).
- **Regla heredada** — repos públicos con "seguridad suave": nada de datos reales en el HTML ([[yod-boards-seguridad]]). Este board cumple: el HTML no trae clave ni ids de Sheets.

## Pendientes
| Tema | Dueño | Qué evidencia lo cierra |
|---|---|---|
| Pegar la carpeta de Drive existente de Miramar en su fila de PROYECTOS | Alejandro | `carpeta_drive_url` lleno en la fila y el link "carpeta Drive" visible en el hero |
| Llenar Dunas Kino, La Cercada y Casa Alysa (hoy sin Sheet Maestro → salen con el cartel "aún no tiene Sheet Maestro conectado", `index.html:373`) | responsables de cada proyecto | los 3 chips con Gantt propio, no el cartel |
| Aplicar (o descartar) el skin YoD aprobado del dummy | Alejandro | captura del board real con el header marino, o un "ya no" por escrito |
| Fase F2: plantilla de Maestro generalizada, board genérico `?p=`, entregables a Drive desde la ficha | Alejandro (luz verde) | su OK explícito; hoy están congelados |
| Decidir "opción B": unificar a una sola clave (hoy hay Sheet de claves aparte `1jZOsov…`) | Alejandro | una sola clave viva y el Sheet de claves archivado |
| Archivar/ocultar las hojas `zzz_OLD_*` del Maestro de Miramar | Alejandro (cuando confíe) | las hojas ocultas y el board sin cambios |
| ¿Se borra `fonts/Kefa-Regular.ttf`? | Alejandro | confirmación de que ninguna página lo usa |

## Por confirmar (no lo afirmes sin preguntar)
- **¿En qué versión está desplegado el GAS hoy?** Las memorias dicen @14 (3-jul) y @26 (12-jul). Pregunta: *"En Administrar implementaciones de 'Alquimia Urbana - GAS', ¿qué número de versión tiene la implementación activa?"* — `?recurso=meta` solo confirma que responde, no la versión.
- **¿El `gas/Code.gs` local es igual al del editor?** No hay forma de comprobarlo desde aquí. Pregunta: *"¿Me pasas el Code.gs actual del editor de Apps Script?"* antes de cualquier cambio de backend.
- **¿Sigue viva la clave `AlquimiaYOD-2026`?** Aparece en la memoria [[yod-sistema-arquitectura-real]], pero vive en `CONFIG.clave` del Registro y pudo rotarse. No probé claves (no se hacen POST ni intentos de acceso desde aquí).
- **¿Se aplicó ya el skin YoD aprobado?** (ver Decisiones 2026-07-03).
- **Corrección a memoria:** [[potenciales-yod-project]] dice "~/alquimia-urbana y ~/real-miramar-board NO son clones git" (12-jul). **Ya no aplica al 2026-09-04**: `~/alquimia-urbana` sí es clon git de `https://github.com/yodesarrollomx/alquimia-urbana.git`, árbol limpio en `0b10299`.

## Qué NO hacer
- No hacer POST a ningún backend desde una sesión de documentación.
- No meter claves, ids de Sheets ni datos reales al HTML (repo público).
- No agregar frameworks ni build: el board es HTML+CSS+JS plano servido por GitHub Pages.
- No borrar las funciones inertes de migración del .gs (`migrarMiramar2Hojas_`, `swapMiramarA2Hojas_`, `construirV3EnSheet_`, `gas/Code.gs:642-700`) sin avisar: no tienen ruta en `doGet`, pero son el historial de cómo se migró Miramar.
