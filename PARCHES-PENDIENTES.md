# Parches pendientes de jman

Arreglos que existen en `JMAN730/opendroid` y **no** en `yashab-cyber/opendroid`.
**Archivo nuevo, no upstream.**

Comprobado el 2026-08-31 con `git cherry upstream/main jman/main`: **30 commits sin
equivalente en upstream**. Ninguno fue aplicado ni reescrito allí. Upstream sí fusionó
PRs de JMAN730 hasta el 9 de agosto de 2026 (`68aa38c`, `fd46ccd`), pero dejó de
hacerlo. Todo lo de abajo es trabajo posterior a esa ruptura.

Estos parches **no se fusionan**, se traen de uno en uno con `cherry-pick`.
Comandos en `MIS-NOTAS.md`.

**Estado de todos: `[no aplicado]`.** Ninguno traído todavía.

---

## Prioridad crítica

### `faa87a2` — Bucle de crasheo del servicio en primer plano tras reiniciar (Android 14+)

- **Estado:** `[no aplicado]`
- **Fecha:** 2026-08-11 · **Autor:** JMAN730
- **Asunto:** `fix: stop OpenDroidService FGS crash-loop after reboot on Android 14+`
- **Toca:** `BootReceiver.kt`, `ForegroundServiceStartPolicy.kt` (nuevo),
  `OpenDroidService.kt`, `ForegroundServiceStartPolicyTest.kt` (nuevo) — 4 archivos,
  +200 / -38

**Qué arregla.** En Android 14+ el sistema prohíbe arrancar un servicio en primer
plano de tipo `microphone` desde el arranque del teléfono. `OpenDroidService` lo
intentaba igualmente, lanzaba excepción, el sistema lo reiniciaba, volvía a fallar:
**bucle infinito de crasheo**. El parche omite el auto-arranque por `BOOT_COMPLETED`
cuando ningún tipo de servicio es legal, llama a `startForeground` antes de
inicializar los motores, y usa `START_NOT_STICKY` cuando el arranque es rechazado.

**Por qué me importa.** Mi Samsung Galaxy S21+ tiene **Android 15**, que es 14+, así
que me afecta de lleno. Mi caso de uso depende de que el servicio **sobreviva a un
reinicio del teléfono**: es justo el escenario que este bug rompe. Sin este parche,
el asistente no arranca solo tras reiniciar, y encima deja el sistema en bucle.

**Ojo:** este commit es el primero de una serie de cinco que se corrigen entre sí.
Traer solo `faa87a2` deja un comportamiento a medias — ver los cuatro siguientes.

---

## Prioridad alta — continuación de `faa87a2`

Los cuatro corrigen o completan el anterior. Aplicar **en orden cronológico**.

### `273b87f` — Revisión de PR sobre el arranque del servicio

- **Estado:** `[no aplicado]` · **Fecha:** 2026-08-12
- **Asunto:** `fix: address PR #187 review feedback on FGS startup`
- **Qué arregla.** Dos cosas. `onCreate` mantenía `enginesInitialized` en `true`
  aunque la construcción de motores fallara; ahora limpia y llama a `stopSelf`.
  Y corrige la premisa del parche anterior: `specialUse` **no** está prohibido en
  `BOOT_COMPLETED` en Android 14/15, solo `microphone` lo está.
- **Por qué me importa.** Sin él, `faa87a2` bloquea el auto-arranque más de lo
  necesario en mi teléfono.

### `d3b7c36` — Permitir auto-arranque en Android 14+ con micrófono concedido

- **Estado:** `[no aplicado]` · **Fecha:** 2026-08-12
- **Asunto:** `fix: allow boot auto-start on Android 14+ when mic granted`
- **Qué arregla.** El respaldo a `specialUse` nunca está restringido en el arranque,
  así que siempre hay un tipo válido alcanzable. Condicionar el arranque a
  "micrófono no concedido" anulaba ese respaldo por completo.
- **Por qué me importa.** Es la diferencia entre que el asistente arranque solo tras
  reiniciar o no arranque nunca.

### `aebd2e0` — Silenciar la palabra de activación bajo el respaldo specialUse

- **Estado:** `[no aplicado]` · **Fecha:** 2026-08-12
- **Asunto:** `fix: suppress wake word when foreground start used specialUse fallback`
- **Qué arregla.** Arrancar el detector de palabra de activación contra un servicio
  promovido a `specialUse` (que no tiene permiso de micrófono) provoca un bucle de
  `onError` **cada segundo**.
- **Por qué me importa.** Batería y logs. Y evita que el servicio grabe sin un
  servicio en primer plano tipado como micrófono.

### `f2ef6f5` — Re-promover el servicio al tipo micrófono

- **Estado:** `[no aplicado]` · **Fecha:** 2026-08-13
- **Asunto:** `fix: re-promote the boot foreground service to the microphone type`
- **Qué arregla.** El más importante de los cuatro. Cuando el servicio arranca en
  `specialUse` y silencia la palabra de activación, esa supresión era **permanente**:
  abrir la app o conceder `RECORD_AUDIO` llegaba a un servicio ya en marcha que nunca
  reintentaba la promoción. Ahora `onStartCommand` reevalúa en cada arranque y vuelve
  a llamar a `startForeground` con el tipo micrófono cuando ya es legal. Añade además
  retroceso exponencial (1s → 30s) al `WakeWordDetector`.
- **Por qué me importa.** Sin él, tras cada reinicio del teléfono **la palabra de
  activación queda muerta hasta que reinicio la app a mano**. Eso rompe el caso de uso
  entero.

---

## Prioridad media — ventana de contexto del modelo en dispositivo

Relevante solo si uso modelos locales (LiteRT / Gemma) en vez de una API en la nube.

### `ef73e02` — Ventana de contexto configurable hasta 32K

- **Estado:** `[no aplicado]` · **Fecha:** 2026-08-11
- **Asunto:** `feat: let users raise the on-device context window up to 32K`
- **Qué arregla.** Los modelos locales corrían con la ventana fijada en su entrada de
  catálogo (4096 en las builds de Gemma LiteRT), así que una petición de varios pasos
  como *"abre WhatsApp y resume el último mensaje"* fallaba con "esta conversación es
  demasiado larga" mucho antes del límite real del modelo. Añade un selector en
  Ajustes, por modelo.
- **Por qué me importa.** Si uso el modelo en dispositivo, las peticiones de varios
  pasos —que son el punto de un asistente— fallan sin esto.

### `0837f9b` — Reutilizar el motor de respaldo en vez de reintentar

- **Estado:** `[no aplicado]` · **Fecha:** 2026-08-11
- **Asunto:** `fix: reuse fallback engine instead of retrying at failing context window`
- **Qué arregla.** Cerraba el motor de respaldo que funcionaba, reintentaba con el
  tamaño que ya se sabía fallido, y recargaba el modelo **en cada llamada**.
- **Por qué me importa.** Recargar el modelo en cada petición es lentísimo en un móvil.

### `ffa3d86` — No retirar el motor por rechazo de presupuesto de prompt

- **Estado:** `[no aplicado]` · **Fecha:** 2026-08-11
- **Asunto:** `fix: keep fallback engine cached on prompt-budget rejection`
- **Qué arregla.** Una `IllegalStateException` genérica hacía que el catch-all cerrara
  un motor sano cuando lo único que falló fue el presupuesto del prompt. Introduce
  `PromptBudgetExceededException`.

### `accc2bf` — Eximir cancelaciones de la retirada del motor en streaming

- **Estado:** `[no aplicado]` · **Fecha:** 2026-08-12
- **Asunto:** `fix: exempt budget rejections and cancellation from streaming engine retirement`

### `a491553` — Aplicar el modelo seleccionado en `generate()` y relanzar cancelaciones

- **Estado:** `[no aplicado]` · **Fecha:** 2026-08-12
- **Asunto:** `fix: apply Hybrid-selected model to generate() and rethrow cancellation`
- **Qué arregla.** `HybridOnDeviceProvider.generate` resolvía el modelo correcto y
  luego lo descartaba al delegar, así que una ventana de contexto elegida en Ajustes
  **no tenía ningún efecto** en los planes de varios pasos.

### `70f8bef` — Revertir la selección si la persistencia no está lista

- **Estado:** `[no aplicado]` · **Fecha:** 2026-08-11
- **Asunto:** `fix: roll back context-window selection on non-Ready persistence state`
- **Qué arregla.** La UI podía informar de un tamaño que en realidad no se guardó.

---

## Prioridad media — seguridad

### `d538c7c` — Endurecer la ejecución de acciones MCP

- **Estado:** `[no aplicado]` · **Fecha:** 2026-08-09
- **Asunto:** `security: harden MCP action execution`
- **Toca:** `ActionSequenceExecutor.kt`, `AliasResolver.kt`, `McpServer.kt`,
  `PersistentTerminalManager.kt`, tres archivos de test, y
  `docs/security_architecture.md`
- **Qué arregla.** Protege contra el despacho de acciones no autorizadas vía MCP.
- **Por qué me importa.** MCP puede ejecutar acciones reales en el teléfono. Si acabo
  usando esa función, esto es la diferencia entre un asistente y una puerta trasera.
- **Ojo:** toca `docs/security_architecture.md`, que **es un archivo upstream**. Un
  cherry-pick lo modificaría, y eso rompe mi regla de no tocar upstream. Decidir al
  traerlo: o excluir ese archivo del parche, o aceptar el conflicto futuro.

---

## Prioridad baja

### `5901760` — Refactor grande de proveedores, APIs de UI y helpers

- **Estado:** `[no aplicado]` · **Fecha:** 2026-08-11
- **Asunto:** `refactor: desloppify providers, UI APIs, shared helpers (#183)`
- **Tamaño:** 49 archivos, +940 / -1050
- **Qué hace.** Unifica los proveedores compatibles con OpenAI sobre una base común,
  elimina modelos muertos y helpers duplicados, migra APIs de Compose obsoletas y
  añade logs a bloques `catch` que fallaban en silencio.
- **Por qué me importa (poco, por ahora).** No arregla ningún fallo que me afecte, y
  es enorme. Es el que más conflictos va a dar. Dejarlo para el final o no traerlo.

---

## Atajo: el commit consolidado

### `9e08baf` — Todo lo anterior en un solo commit

- **Estado:** `[no aplicado]` · **Fecha:** 2026-08-14
- **Asunto:** `fix: Android 14+ service startup, context-window controls, MCP hardening`
- **Tamaño:** 69 archivos, +2074 / -1215

Su propio mensaje dice: *"Consolidates work from the JMAN730/opendroid fork"*. Agrupa
el arranque del servicio en Android 14+, los controles de ventana de contexto, el
endurecimiento de MCP y el refactor, con cobertura de tests. Fue preparado como PR
hacia upstream (PR #192), que upstream **no** fusionó.

**Ventaja:** un solo `cherry-pick` en vez de doce.
**Desventaja:** todo o nada. Incluye el refactor grande `5901760` y toca dos archivos
de documentación upstream (`docs/qa/v1.0.4-test-plan.md` es nuevo, pero
`docs/security_architecture.md` es upstream y sería modificado).

**Decisión pendiente.** Mi inclinación: traer primero los cinco parches del servicio
en primer plano uno a uno (que es lo que necesito de verdad), y evaluar el
consolidado más adelante.

---

## Registro de aplicación

Cuando aplique uno, anotarlo aquí: fecha, hash original, hash nuevo tras el
cherry-pick, y si hubo conflictos.

| Fecha | Hash original | Hash nuevo | Conflictos | Notas |
|---|---|---|---|---|
| — | — | — | — | *(ninguno aplicado todavía)* |
