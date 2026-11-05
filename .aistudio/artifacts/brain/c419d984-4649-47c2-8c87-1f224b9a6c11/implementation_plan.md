# Plan Maestro de Auditoría Técnica y Sincronización OpenWiki

Plan de auditoría técnica exhaustiva para contrastar la documentación de `openwiki/` contra la implementación real del código fuente en los módulos `:app`, `:terminal-view`, `:terminal-emulator` y `:termux-shared`, corrigiendo desviaciones, expandiendo subsistemas omitidos y sincronizando claims granulares e índices de metadatos.

## Decisiones Críticas y Preferencias Confirmadas

> [!IMPORTANT]
> Se han incorporado las directivas confirmadas en la fase de clarificación interactiva para guiar el alcance y rigor de la auditoría.

- **Alcance Confirmado**: Auditoría exhaustiva completa abarcando todos los módulos (`:app`, `:terminal-emulator`, `:terminal-view`, `:termux-shared`), conceptos y flujos de trabajo.
- **Granularidad de Claims**: Detalle granular obligatorio en `.claims/` especificando nombres de métodos, firmas de API y rangos exactos de líneas de código fuente.
- **Regla Condicional de Cero Modificaciones**: Si alguna sección resulta estar ya 100% sincronizada, se preserva intacta. Se intervienen únicamente los archivos con brechas o discrepancias empíricamente verificadas.

---

## 1. Diagnóstico de Brechas y Hallazgos Iniciales

Tras la inspección preliminar de código fuente e índices documentales, se han detectado las siguientes discrepancias concretas que impiden la fidelidad matemática:

1. **`openwiki/workflows/package-management-and-store.md` (Desviación en IDs de Categorías y Aliases)**:
   - **Discrepancia en Categorías**: La tabla y documentación refieren IDs en inglés (`"system"`, `"tools"`, `"dev"`, `"google"`, `"ia"`), mientras que la implementación real en `TermuxStoreActivity.java` (líneas 165-202) utiliza IDs en español: `"sistema"`, `"herramientas"`, `"desarrollo"`, `"google"`, `"ia"`.
   - **Omisión de Aliases de Zsh**: No documenta los aliases Zsh recientemente estandarizados (`alias x="exit"`, `alias X="exit"`, `alias c="clear"`, `alias C="clear"`) inyectados en la instalación de Zsh y en el comando de actualización del catálogo.
   - **Claims Desactualizados (`.claims/workflows/package-management-and-store.json`)**: Solo cuenta con 3 claims genéricos de alto nivel sin firmas de métodos (`initPackageList`, `selectCategoryChip`, `filterPackages`, `executeShellCommandInTerminal`, `cleanupLegacyZshScripts`, `StorePackageAdapter.onBindViewHolder`).

2. **`openwiki/concepts/rendering-and-view.md` y `.claims/concepts/rendering-and-view.json` (Claims Obsoletos)**:
   - Si bien el archivo Markdown documenta la integración de Gboard IME y Bracketed Paste, el archivo de claims `.claims/concepts/rendering-and-view.json` data de la versión inicial (`2026-10-26`), careciendo por completo de claims que certifiquen:
     - `mActiveWordBuffer` y prevención de duplicación en `getTextBeforeCursor`.
     - Manejo de `commitCorrection` ignorando re-emisión redundante.
     - DECSET 2004 Bracketed Paste Mode en `TerminalEmulator.paste()` para bloques multilínea.
     - Pipeline de rich content `commitContent` (`resolveOrSaveClipboardContent`, `mapToTermuxStoragePath`, `saveContentStreamToFile`).

3. **`openwiki/workflows/session-lifecycle.md` (Cobertura Parcial de UI y Controladores)**:
   - El documento actual se limita a la relación abstracta entre `TermuxService` y `TerminalSession` (65 líneas).
   - Omite los componentes de interfaz que gobiernan las sesiones: `TermuxDrawerLayout`, `TermuxSessionsListViewController` (gestión del drawer, cambio de sesión activa, creación de sesiones normales y failsafe, renombrado y terminación interactiva con botón de cierre).
   - El archivo de claims correspondiente (`.claims/workflows/session-lifecycle.json`) carece de claims sobre la vinculación de interfaz y controladores de sesión.

4. **Sincronización de Manifiesto y Metadatos**:
   - `.page-manifest.json` y `.last-update.json` deben actualizarse con las nuevas versiones de página y marcas temporales tras sincronizar los claims y contenidos.

---

## 2. Plan de Intervención por Fases

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                 FLUJO DE AUDITORÍA Y SINCRONIZACIÓN OPENWIKI                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  [FASE 1: AUDITORÍA DE CÓDIGO Y MATRIZ DE COBERTURA]                        │
│    ├── Inspección de clases en :app, :terminal-view, :terminal-emulator    │
│    └── Validación cruzada de firmas de métodos y constantes                │
│                                                                             │
│  [FASE 2: CORRECCIÓN Y ENRIQUECIMIENTO DOCUMENTAL]                          │
│    ├── package-management-and-store.md (IDs reales, aliases Zsh, flujos)   │
│    ├── session-lifecycle.md (TermuxDrawerLayout, SessionsListController)  │
│    └── rendering-and-view.md (Revisión de consistencia IME / Clipboard)    │
│                                                                             │
│  [FASE 3: GENERACIÓN Y ACTUALIZACIÓN DE CLAIMS GRANULARES]                  │
│    ├── .claims/workflows/package-management-and-store.json (APIs + líneas) │
│    ├── .claims/workflows/session-lifecycle.json (APIs + líneas)            │
│    └── .claims/concepts/rendering-and-view.json (Claims Gboard & Paste)    │
│                                                                             │
│  [FASE 4: SINCRONIZACIÓN DE ÍNDICES Y METADATOS]                            │
│    ├── quickstart.md y archivos index.md de directorio                      │
│    ├── .page-manifest.json (actualización de hashes y fuentes)              │
│    └── .last-update.json (actualización de timestamp y estado)              │
│                                                                             │
│  [FASE 5: QUALITY GATE Y VERIFICACIÓN]                                      │
│    ├── compile_applet (compilación limpia del proyecto Android)             │
│    ├── gradle :app:testDebugUnitTest (suite de pruebas JVM)                 │
│    └── codegraph sync --no-color (sincronización del grafo estructural)    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Especificación Detallada de Cambios

### A. Documentación de Flujos (`openwiki/workflows/`)

1. **`package-management-and-store.md`**:
   - Corregir los IDs de categoría en la tabla canónica:
     - `"sistema"` (Termux Storage, Backup/Restore, Mirrors, Termux-API)
     - `"herramientas"` (Zsh, Oh My Zsh con aliases x/c, Lazygit, Yazi, Neovim, Tmux, Htop)
     - `"desarrollo"` (Git, GitHub CLI, Node.js, Python, Golang, Rust)
     - `"google"` (Gemini CLI, Cloud SDK, Angular CLI, Antigravity)
     - `"ia"` (OpenWiki CLI, CodeGraph, Engram)
   - Incorporar subsección técnica detallando la configuración de aliases Zsh inyectados en `.zshrc`.
   - Incluir tabla de firmas de métodos de `TermuxStoreActivity` y `StorePackageAdapter`.

2. **`session-lifecycle.md`**:
   - Ampliar el flujo para cubrir la gestión de sesiones en la interfaz gráfica:
     - `TermuxDrawerLayout`: aperturas/cierres del drawer lateral de sesiones.
     - `TermuxSessionsListViewController`: adaptador de lista, binding de `item_terminal_sessions_list.xml`, selección de sesión actual (`setCurrentSession`), creación de nueva sesión estándar vs sesión failsafe (`TermuxActivity.addNewSession(boolean isFailSafe, String sessionName)`), renombrado de sesión mediante diálogo y finalización (`session.finish()`).
   - Diagrama de secuencia actualizado que incluya el `TermuxSessionsListViewController` y la interacción con `TermuxService`.

### B. Base de Claims Granulares (`openwiki/.claims/`)

1. **`.claims/workflows/package-management-and-store.json`**:
   - Claim de inicialización del catálogo y listeners de búsqueda/chips (`TermuxStoreActivity.onCreate`, `initPackageList`, `selectCategoryChip`).
   - Claim de filtrado en tiempo real (`TermuxStoreActivity.filterPackages`).
   - Claim de verificación multi-ruta de binarios (`StorePackageAdapter.onBindViewHolder`).
   - Claim de inyección de comandos Zsh y aliases (`TermuxStoreActivity.initPackageList`).
   - Claim de despacho atómico sin scripts temporales (`TermuxStoreActivity.executeShellCommandInTerminal`, `TermuxActivity.executeCommandInCurrentSession`).

2. **`.claims/concepts/rendering-and-view.json`**:
   - Claim de buffer activo de palabras (`TerminalView.mActiveWordBuffer`, `getTextBeforeCursor`, `appendToBuffer`).
   - Claim de no re-emisión en `commitCorrection`.
   - Claim de protección de comandos multilínea mediante `TerminalEmulator.paste()` y DECSET 2004.
   - Claim de resolución de rich content URI (`TermuxTerminalViewClient.resolveOrSaveClipboardContent`, `mapToTermuxStoragePath`, `saveContentStreamToFile`).

3. **`.claims/workflows/session-lifecycle.json`**:
   - Claim de control de sesiones en cajón lateral (`TermuxSessionsListViewController`).
   - Claim de creación de sesión normal y failsafe (`TermuxActivity.addNewSession`).
   - Claim de puente con `TermuxService` y `TermuxShellManager`.

### C. Manifiesto y Metadatos (`openwiki/`)

- **`.page-manifest.json`**: Registrar las nuevas versiones de página calculadas con hashes representativos y actualizar referencias en `pages`.
- **`.last-update.json`**: Actualizar `updatedAt` al timestamp ISO actual, certificando la auditoría técnica completa.
- **`quickstart.md`**: Asegurar que las referencias reflejen con precisión las adiciones en `session-lifecycle.md` y `package-management-and-store.md`.

---

## 4. Quality Gate de Integridad

Al completar las modificaciones documentales y de claims:
1. **Compilación Limpia (`compile_applet`)**: Verificar que el proyecto Android compile sin errores.
2. **Pruebas Unitarias JVM (`run_command`)**: Ejecutar `gradle :app:testDebugUnitTest` para garantizar cero regresiones en los tests existentes.
3. **Sincronización Estructural (`run_command`)**: Ejecutar `codegraph sync --no-color` para actualizar el grafo relacional.
4. **Reporte Final**: Certificación detallada con lista de archivos modificados, discrepancias resueltas y trazabilidad matemática con el código.
