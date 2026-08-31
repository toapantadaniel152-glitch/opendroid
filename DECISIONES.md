# Decisiones

Registro de decisiones de este fork. **Archivo nuevo, no upstream.**

**Este archivo es congelado.** Cada entrada se escribe una vez y no se reescribe
nunca. Si una decisión cambia, se añade una entrada nueva al principio que revoca la
anterior y explica por qué; la vieja se queda tal cual, como registro histórico.

Formato de cada entrada: **fecha · qué se decidió · qué se descartó · por qué**.
Orden: la más reciente arriba.

---

## 2026-08-31 — `upstream` apunta a yashab-cyber, no a JMAN730

### Qué se decidió

El remoto `upstream` queda apuntando a **`https://github.com/yashab-cyber/opendroid.git`**.

`JMAN730/opendroid` se conserva como un tercer remoto llamado **`jman`**, de consulta,
del que se traen parches sueltos con `cherry-pick`. Nunca se fusiona, y nunca se
renombra a `upstream`.

### Qué se descartó

**Repuntar `upstream` a `JMAN730/opendroid`.**

Se consideró en serio porque ese fork tiene 30 commits de arreglos reales que
yashab-cyber no ha incorporado, incluido el bucle de crasheo del servicio en primer
plano en Android 14+ (`faa87a2`), que afecta directamente a mi teléfono.

### Por qué

**1. yashab-cyber es el padre real, con divergencia cero.**

Medido el 2026-08-31 tras `git fetch` de ambos:

```
main vs upstream/main   →   0 commits míos,  0 suyos
main vs jman/main       →  14 commits míos, 39 suyos
```

Mi commit base `d5c32c4` **es ancestro de `upstream/main`** y **no lo es de
`jman/main`**. El antepasado común con jman es `94b1f6c`, del 13 de agosto de 2026:
ahí se separaron las ramas. jman no es un upstream más nuevo, es una **rama hermana**.

**2. Apuntar a jman convertiría cada actualización en una fusión conflictiva.**

Con yashab-cyber, `git merge --ff-only upstream/main` es siempre un avance limpio, sin
resolución manual. Con jman sería una fusión de **39 commits con conflictos
garantizados**: borró `ROADMAP.md`, reescribió el README apuntando a su propio
repositorio y modificó el sitio web entero. Y encima yo perdería o tendría que
reconciliar mis 14 commits que él no tiene.

**3. yashab-cyber tiene el producto más avanzado y las etiquetas oficiales.**

Último commit de upstream: 2026-08-26. De jman: 2026-08-15, once días antes.
Upstream contiene tres releases que jman **no tiene**: v1.0.5 (Screen Understanding,
Personal Growth Memory), v1.0.6 (Habit & Routine Detection, Telegram Control), y el
grafo de conocimiento personal. Las etiquetas `v1.0.0`–`v1.0.6` solo existen ahí.

**4. La palabra "upstream" debe significar "de dónde vengo".**

Cambiarla destruiría la única referencia fiable de mi línea base.

### Contexto: la divergencia de política entre los dos repositorios

No es solo una separación técnica, y conviene dejarlo escrito porque explica por qué
jman dejó de sincronizar con upstream y por qué upstream dejó de fusionar sus PRs.

Upstream **sí** fusionó PRs de JMAN730 hasta el 9 de agosto de 2026 (`68aa38c`,
`fd46ccd`). Después dejó de hacerlo, y el trabajo posterior de jman —incluido el
arreglo del bucle de crasheo— se quedó sin incorporar. jman llegó a preparar un PR
consolidado hacia upstream (`9e08baf`, PR #192, 2026-08-14) que tampoco fue fusionado.

En paralelo, los rumbos se separaron explícitamente:

- **Upstream** dedicó sus dos últimos commits (`2f6aee9`, `d5c32c4`) a añadir y
  actualizar un *contract address* de un token cripto en el README y en el sitio web.
- **jman** dedicó sus tres últimos commits a lo contrario: repuntar el README y la web
  a su propio repositorio (`94a262b`, `53042c9`) y añadir
  **`fe8f787 — "docs: add no-affiliation notice disclaiming any crypto token"`**,
  un aviso que desmiente cualquier vínculo con un token cripto.

Ese mismo commit `94a262b` eliminó además un `sealed_token` de `api.star-history.com`
incrustado en tres URLs del README, describiéndolo como *"leaked star-history token"*.
Ese token **sigue presente** en el README de upstream, y por tanto en mi copia
(`README.md:249-251`). Anotado en `MIS-NOTAS.md`.

**Qué significa esto para la decisión.** No cambia la conclusión —sigo descendiendo de
yashab-cyber y sincronizar con él sigue siendo lo limpio— pero sí explica por qué los
dos repositorios no van a reconverger solos, y por qué `PARCHES-PENDIENTES.md` tiene
que existir: los arreglos técnicos de jman **no van a llegar a upstream por sí solos**.
Si los quiero, tengo que traerlos yo.

### Cómo revisar esta decisión más adelante

Se replantea si ocurre alguna de estas:

- Upstream deja de publicar durante varios meses mientras jman sigue activo.
- Acabo trayendo tantos parches de jman que mi rama se parece más a jman que a
  upstream.
- Upstream toma un rumbo que no quiero seguir (por ejemplo, más contenido cripto en
  el código y no solo en la documentación).

En ese caso: entrada nueva arriba, esta se queda.
