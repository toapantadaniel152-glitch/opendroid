# Mis notas del fork de OpenDroid

Notas personales sobre este fork. **Archivo nuevo, no upstream.**

> **Regla de este fork:** todo archivo que ya existía al clonar es UPSTREAM y no se
> edita nunca. Cualquier edición sobre un archivo upstream provoca conflictos en
> cada `git pull upstream main` futuro. Mi documentación vive solo en archivos
> nuevos: este, `PARCHES-PENDIENTES.md` y `DECISIONES.md`.

Última actualización: 2026-08-31

---

## Línea base

| | |
|---|---|
| Commit | `d5c32c438fd7a2014846f51e6fa7d0c72d870e21` (`d5c32c4`) |
| Fecha | 26 de agosto de 2026, 20:37:48 UTC |
| Autor | Yashab Alam |
| Mensaje | `docs: update contract address (CA) in README and website` |
| Versión de la app | 1.0.6 (`versionCode` 7) |

Al clonar, mi `main` era **idéntico** a `upstream/main` (divergencia 0 / 0).
Este fork desciende directamente de `yashab-cyber/opendroid`.

---

## Los tres remotos

| Remoto | URL | Para qué sirve |
|---|---|---|
| `origin` | `https://github.com/toapantadaniel152-glitch/opendroid.git` | Mi fork. Aquí subo mi trabajo. |
| `upstream` | `https://github.com/yashab-cyber/opendroid.git` | **Mi padre real.** Sincronización limpia: `d5c32c4` es ancestro suyo, así que traer sus cambios es siempre un fast-forward sin conflictos. |
| `jman` | `https://github.com/JMAN730/opendroid.git` | **Rama hermana**, separada de upstream el 13 de agosto de 2026 (antepasado común `94b1f6c`). **Cantera de parches vía `cherry-pick`. NO fusionar.** |

**Por qué `jman` no se fusiona:** tiene 39 commits que yo no tengo y le faltan 14 que
yo sí tengo. Además borró `ROADMAP.md`, reescribió el README apuntando a su propio
repositorio y modificó todo el sitio web. Un `merge` sería un conflicto garantizado.
Sus arreglos se traen de uno en uno. Ver `PARCHES-PENDIENTES.md` y `DECISIONES.md`.

---

## Versiones exactas requeridas

Cada dato leído de su archivo, no de memoria.

| Qué | Versión | De dónde salió |
|---|---|---|
| **JDK** | **21** (no más nuevo) | `gradle/gradle-daemon-jvm.properties:13` → `toolchainVersion=21` |
| ↳ confirmación | 21 | `app/build.gradle:99-100` → `sourceCompatibility/targetCompatibility JavaVersion.VERSION_21` |
| ↳ confirmación | 21 | `app/build.gradle:179` → `jvmToolchain(21)` |
| ↳ confirmación | 21 | `app/build.gradle:181` → `jvmTarget = ...JvmTarget.JVM_21` |
| ↳ confirmación (CI) | 21 | `.github/workflows/android-ci.yml:40` → `java-version: '21'` |
| **Android compileSdk** | **36** | `app/build.gradle:12` → `compileSdk 36` |
| **Android targetSdk** | **36** | `app/build.gradle:17` → `targetSdk 36` |
| **Android minSdk** | **26** (Android 8.0) | `app/build.gradle:16` → `minSdk 26` |
| **Gradle** | **9.7.0** | `gradle/wrapper/gradle-wrapper.properties:4` → `gradle-9.7.0-bin.zip` |
| **Kotlin** | **2.4.0** | `build.gradle:18` → `kotlin-gradle-plugin:2.4.0` |
| ↳ también | 2.4.0 | `build.gradle:23` y `build.gradle:24` (serialization, compose-compiler) |
| **Android Gradle Plugin** | **9.3.1** | `build.gradle:13` → `com.android.tools.build:gradle:9.3.1` |
| KSP | 2.3.10 | `build.gradle:25` |
| Hilt (inyección de dependencias) | 2.60.1 | `build.gradle:22` |
| Room (base de datos) | 2.8.4 | `build.gradle:14` |

**No hace falta instalar Gradle ni Kotlin por separado.** `./gradlew` (el wrapper)
descarga Gradle 9.7.0 solo, y el compilador de Kotlin llega como dependencia.
Solo hay que instalar el **JDK 21** y el **Android SDK 36**.

**Nota:** `gradle/libs.versions.toml` **no existe** en este proyecto. No hay ningún
archivo `.toml` en el repositorio ni referencia a `versionCatalogs`. Las versiones
están escritas directamente en los `build.gradle`.

**Las versiones van acopladas.** `build.gradle:8-9` explica que AGP 9.3.1 exige
Gradle >= 9.5.0, y por eso el wrapper está fijado a 9.7.0 con checksum. No cambiar
una sin las demás.

---

## Correcciones al README del original

Errores reales en archivos upstream. **No los corrijo allí** — quedan anotados aquí.

### 1. El README dice "Android SDK 35"; el build exige 36

`README.md:200` dice:

```
- **Android SDK 35** (Android 15)
```

Pero `app/build.gradle:12` y `app/build.gradle:17` exigen `compileSdk 36` y
`targetSdk 36`. **Hay que instalar el SDK 36.**

Comprobado el 2026-08-31: el error está en **las 52 ramas** de `upstream` y `jman`
(algunas antiguas dicen incluso "SDK 34"). Ninguna dice 36. Nadie lo ha corregido.

### 2. El README contiene un token de api.star-history.com

`README.md:249-251` incrusta un `sealed_token=_Y78t8Ar-D4NN...` en tres URLs de
`api.star-history.com`. Es una credencial de un servicio de terceros publicada en un
README público. No es una credencial de GitHub y no afecta al build.

El fork de JMAN730 lo eliminó en el commit `94a262b`, describiéndolo literalmente
como *"leaked star-history token"*.

### 3. AGENTS.md referencia archivos que no existen

`AGENTS.md` (que **sí es upstream**: Yashab fusionó un PR que lo añadió) apunta a:

- `CONTEXT.md` en la raíz → **no existe**
- `docs/adr/` → **no existe**
- el rastreador de incidencias `JMAN730/opendroid`, que no es este repositorio

---

## Mi equipo y mi teléfono

**Teléfono:** Samsung Galaxy S21+, Android 15 (API 35).
`minSdk` es 26 (Android 8.0), así que **la app corre sin problema**. Android 15 está
muy por encima del mínimo y por debajo del `targetSdk` 36.

Como Android 15 es 14+, me afectan de lleno las restricciones de servicios en primer
plano que arregla `faa87a2`. Ver `PARCHES-PENDIENTES.md`.

**PC (estado al 2026-08-31, nada instalado todavía):**

| Requisito | Estado |
|---|---|
| JDK 21 | Falta. Sin `java` en PATH, `JAVA_HOME` vacío |
| Android SDK 36 | Falta. `ANDROID_HOME` y `ANDROID_SDK_ROOT` vacíos |
| Android Studio | No instalado |
| Gradle | No hace falta (lo trae el wrapper) |
| Kotlin | No hace falta (dependencia de Gradle) |
| Git | 2.55.0.windows.4 |
| `local.properties` | No existe. Android Studio lo genera al abrir el proyecto |

---

## Comandos que voy a necesitar

### Sincronizar con upstream (yashab-cyber)

```bash
# 1. Traer los cambios de upstream sin tocar mi rama todavía
git fetch upstream

# 2. Ver qué hay de nuevo antes de aplicarlo
git log --oneline main..upstream/main

# 3. Aplicarlo con una fusión normal
git checkout main
git merge upstream/main

# 4. Subirlo a mi fork
git push origin main
```

**Por qué una fusión normal y no `--ff-only`.** Mientras mi `main` era idéntico a
`upstream/main`, `git merge --ff-only upstream/main` funcionaba y era lo más seguro.
En cuanto hice mi primer commit propio (estas notas), `main` dejó de ser un espejo
exacto: ahora tengo un commit que upstream no tiene, así que **ya no existe un
fast-forward posible** y `--ff-only` aborta siempre.

Eso no es un problema. La fusión será limpia porque mis archivos (`MIS-NOTAS.md`,
`PARCHES-PENDIENTES.md`, `DECISIONES.md`) **no existen en upstream**, así que no hay
nada que reconciliar. Ese es justo el motivo de la regla de no editar archivos
upstream: los archivos nuevos nunca chocan.

Si alguna vez quiero comprobar que sigo sin haber tocado nada de upstream:

```bash
# Debe listar SOLO mis tres archivos nuevos
git diff --name-status upstream/main main
```

### Traer un parche suelto de jman (cherry-pick)

```bash
# 1. Actualizar la referencia de jman
git fetch jman

# 2. Inspeccionar el parche ANTES de traerlo
git show <hash>              # el diff completo
git show --stat <hash>       # solo qué archivos toca

# 3. Trabajar siempre en una rama aparte, nunca directo sobre main
git checkout -b parche/<nombre-descriptivo> main

# 4. Traer el commit
git cherry-pick <hash>

# 5. Si hay conflictos: editar los archivos marcados, luego
git add <archivos-resueltos>
git cherry-pick --continue

#    O abortar y dejarlo todo como estaba
git cherry-pick --abort

# 6. Compilar y probar ANTES de fusionar a main
./gradlew assembleDebug
```

### Comprobar el estado de los tres remotos

```bash
git remote -v
git fetch --all
git rev-list --left-right --count main...upstream/main   # izq=míos, der=suyos
git rev-list --left-right --count main...jman/main
```

### Compilar e instalar

```bash
./gradlew assembleDebug
# El APK sale en: app/build/outputs/apk/debug/app-debug.apk
```
