# Mis notas del fork de OpenDroid

Notas personales sobre este fork. **Archivo nuevo, no upstream.**

> **Regla 1 — nada de tocar upstream.** Todo archivo que ya existía al clonar es
> UPSTREAM y no se edita nunca. Cualquier edición sobre un archivo upstream provoca
> conflictos en cada `git pull upstream main` futuro. Mi documentación vive solo en
> archivos nuevos: este, `PARCHES-PENDIENTES.md` y `DECISIONES.md`.
>
> **Regla 2 — las compilaciones las lanzo YO.** Siempre desde mi propia terminal,
> nunca desde Claude Code. Motivo y detalle en
> [Reglas de trabajo en este equipo](#reglas-de-trabajo-en-este-equipo).

Última actualización: 2026-09-02

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

**PC:** Windows 11 Home, x64, Intel i5-1135G7.

**Entorno instalado el 2026-09-01, verificado en mi propia terminal:**

| Componente | Versión | Dónde está |
|---|---|---|
| JDK 21 (Eclipse Temurin) | `21.0.12.1` | `C:\Program Files\Eclipse Adoptium\jdk-21.0.12.101-hotspot\` |
| Android Studio | `2026.1.3.7` | `C:\Program Files\Android\Android Studio` |
| Android SDK — platform | `platforms/android-36` (2.0.0) | ver ruta del SDK abajo |
| Android SDK — build-tools | `build-tools/36.1.0` | idem |
| Android SDK — platform-tools | `37.0.1` (`adb` 1.0.41) | idem |
| Android SDK — cmdline-tools | `latest` (rev 23) | idem |
| Gradle | No instalado a propósito | Lo descarga `./gradlew` (9.7.0) |
| Kotlin | No instalado a propósito | Llega como dependencia de Gradle |
| Git | `2.55.0.windows.4` | |

**Variables de entorno:**

| Variable | Valor | Ámbito |
|---|---|---|
| `JAVA_HOME` | `C:\Program Files\Eclipse Adoptium\jdk-21.0.12.101-hotspot\` | Máquina |
| `ANDROID_HOME` | `C:\Users\Usuario\Android\Sdk` | Usuario |
| `PATH` | + `...\Android\Sdk\platform-tools` y `...\Android\Sdk\cmdline-tools\latest\bin` | Usuario |

### ⚠️ El SDK NO está en la ruta por defecto

```
C:\Users\Usuario\Android\Sdk                  ← LA MÍA (real, verificada)
C:\Users\Usuario\AppData\Local\Android\Sdk    ← la que asumen todos los tutoriales
```

**Si Android Studio o cualquier herramienta me pide la ruta del SDK, es la primera.**
Cuando siga un tutorial que dé por hecho la segunda, tengo que traducir la ruta.

**Por qué no está en la estándar.** El primer intento de instalación se hizo desde
Claude Code, y sus escrituras a `AppData` no llegan al disco real (ver la sección
siguiente). Los archivos se quedaron en una caché privada y la ruta estándar quedó
vacía, aunque todas las verificaciones parecían correctas. Al detectarlo, el SDK se
movió a `C:\Users\Usuario\Android\Sdk`, **fuera de `AppData`**, donde el problema no
puede repetirse y no hacen falta permisos de administrador.

No es un apaño: el SDK de Android es reubicable. Lo confirma el propio `adb`, que
reporta `Installed as C:\Users\Usuario\Android\Sdk\platform-tools\adb.exe`.
Ver `DECISIONES.md`.

**Mi `AppData` es normal.** Comprobado en el registro: `User Shell Folders` da
`Local AppData = C:\Users\Usuario\AppData\Local`, sin junctions ni redirecciones.
El problema es de Claude Code, no de este equipo.

### Otras notas del entorno

- **Android Studio trae su propio Java, y es el 25**, no el 21. Por eso el JDK 21 se
  instaló aparte: el proyecto exige 21 y no más nuevo. `gradle-daemon-jvm.properties`
  hace que Gradle elija el 21 aunque Studio use el 25 para sí mismo.
- **`sdkmanager` está obsoleto.** Google lo sustituye por la Android CLI (`android sdk`)
  y cambió los identificadores de `platforms;android-36` a `platforms/android-36`, con
  barra. Casi toda la documentación de internet usa la sintaxis antigua.
- **`local.properties`** no hace falta: con `ANDROID_HOME` definido, Gradle encuentra el
  SDK. Si Android Studio lo crea igualmente, no importa — está en `.gitignore`.
- Las variables de entorno solo se leen al **abrir** una terminal. Tras cambiarlas hay
  que abrir una ventana nueva; las que ya estaban abiertas conservan los valores viejos.

---

## Reglas de trabajo en este equipo

Tres peculiaridades de esta máquina que ya costaron tiempo una vez. Están aquí para no
volver a descubrirlas por las malas.

### Regla: las compilaciones las lanzo YO, desde mi terminal

**Nunca compilar desde Claude Code.** Siempre desde mi propia PowerShell.

**Por qué.** Gradle deja un proceso residente vivo entre compilaciones (el *daemon*).
Si Claude Code arranca uno, ese daemon hereda su vista alterada del disco. Cuando yo
compile después desde mi terminal, mi cliente puede **reutilizar ese daemon**, que ve
un sistema de archivos distinto al mío. El resultado son fallos incoherentes y muy
difíciles de diagnosticar: exactamente el tipo de problema que costó cuatro rondas de
diagnósticos equivocados el 2026-09-01.

Claude Code sí puede leer el proyecto, buscar en el código, editar archivos y ayudarme
a interpretar los errores del build. Lo que no hace es **lanzarlo**.

Si alguna vez sospecho que hay un daemon con vista rara:

```bash
cd C:\Users\Usuario\Desktop\opendroid
```
```bash
.\gradlew --stop
```

### Las escrituras de Claude Code a AppData no llegan al disco

Claude Code corre como aplicación empaquetada de Windows (`Claude_pzs8sxrjxfjjc`), y
Windows le redirige `AppData\Local` a una caché privada del paquete:

```
C:\Users\Usuario\AppData\Local\Packages\Claude_pzs8sxrjxfjjc\LocalCache\Local\
```

Lo que Claude Code escribe "en" `AppData\Local` aterriza ahí. **Y lo peor: al releerlo
también lee la caché, así que sus verificaciones dan verde mientras mi disco sigue
vacío.** Eso fue exactamente lo que pasó con el SDK.

**No está redirigido** (verificado): el registro, el Escritorio, la raíz del perfil
(`C:\Users\Usuario\...`) y `TEMP`. Ahí sus escrituras sí llegan.

**Consecuencia práctica:** si Claude Code tiene que instalar algo, que sea **fuera de
`AppData`**, o vía `winget` con elevación UAC (que escribe en el sistema real). Y la
verificación final la hago **yo en mi terminal**, no vale la suya.

### ⚠️ OneDrive: no aceptar la copia del Escritorio

OneDrive está instalado y corriendo (carpeta `C:\Users\Usuario\OneDrive`), pero la
**copia de seguridad de carpetas conocidas está desactivada**. Verificado: el registro
da `Desktop = C:\Users\Usuario\Desktop`, ruta local, sin junction.

Por eso el proyecto en `C:\Users\Usuario\Desktop\opendroid` **no se sincroniza**.

**Si OneDrive ofrece alguna vez "hacer copia de seguridad de tus carpetas" y el
Escritorio está incluido: decir que NO,** o mover el proyecto antes. Si el Escritorio
pasara a sincronizarse, `Desktop` apuntaría a `C:\Users\Usuario\OneDrive\Escritorio` y
el proyecto entraría con él. Una compilación de Android genera decenas de miles de
archivos en `app\build`, y OneDrive intentaría subirlos todos: compilaciones lentísimas,
archivos bloqueados a media escritura y conflictos de sincronización.

(La carpeta de OneDrive contiene accesos directos e instaladores con pinta de
escritorio antiguo. Es cosa aparte; el proyecto no está dentro.)

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

**La compilación la lanzo YO desde mi terminal** (Regla 2). Nunca Claude Code.

```bash
cd C:\Users\Usuario\Desktop\opendroid
```
```bash
.\gradlew assembleDebug
```

El APK sale en `app\build\outputs\apk\debug\app-debug.apk`.

Instalar en el S21+ (con el teléfono conectado y autorizado):

```bash
adb install -r app\build\outputs\apk\debug\app-debug.apk
```

`-r` reinstala conservando los datos si ya hubiera una versión.

Ver los logs de la app en vivo (el equivalente a las DevTools):

```bash
adb logcat -s OpenDroid:V AndroidRuntime:E
```

`Ctrl+C` para salir. Aquí es donde aparecerá el bucle de crasheo del servicio en
primer plano si se manifiesta tras un reinicio del teléfono — ver `faa87a2` en
`PARCHES-PENDIENTES.md`.

Si `adb` se comporta de forma extraña (dispositivo que no aparece, estados
incoherentes), puede haber un servidor viejo colgado:

```bash
adb kill-server
```

Arranca uno nuevo solo con volver a ejecutar `adb devices`.

---

## Registro: primera compilación e instalación

**2026-09-02 — funcionó a la primera.**

| | |
|---|---|
| Resultado | `BUILD SUCCESSFUL` |
| Duración | **7 min 36 s** (42 tareas, 42 ejecutadas) |
| APK | `app\build\outputs\apk\debug\app-debug.apk`, **69,7 MB** |
| Paquete | `com.opendroid.aiagent` v1.0.6 (`versionCode` 7) |
| Compilado contra | `compileSdk` 36 / `targetSdk` 36 |
| Firma | Clave de depuración (`apkSigningVersion=2`) — no sirve para publicar |
| Instalación | `Success`, `firstInstallTime=2026-09-02 13:10:10` |

**Las siguientes compilaciones son mucho más rápidas.** Gradle guardó la caché de
configuración (`Configuration cache entry stored`) y las dependencias quedaron en
`C:\Users\Usuario\.gradle`. Espero **1-2 minutos**, no 7 y medio.

### Avisos normales que NO son errores

La compilación imprime muchas advertencias de deprecación de Compose, del tipo:

```
w: Icons.Filled.KeyboardArrowRight is deprecated.
   Use the AutoMirrored version at Icons.AutoMirrored.Filled.KeyboardArrowRight
```

Son recomendaciones de accesibilidad (iconos que se voltean en idiomas de derecha a
izquierda). **No las arreglo**: están en archivos upstream. El refactor `5901760` de
jman incluye esa migración, con prioridad baja en `PARCHES-PENDIENTES.md`.

Lo único que importa es la última línea: `BUILD SUCCESSFUL` o `BUILD FAILED`.

### Mi teléfono, datos verificados con adb

| Dato | Valor |
|---|---|
| Modelo | `SM-G996U1` — Galaxy S21+ 5G (nombre interno `t2q`) |
| Serie (`adb devices`) | `RFCR91PWY7A` |
| Android | 15 — **API 35** |
| Parche de seguridad | 2026-01-01 |
| Arquitectura | `arm64-v8a` |

API 35 contra `minSdk 26`: compatible de sobra. Y **API 35 es Android 14+**, así que
las restricciones de servicios en primer plano que arregla `faa87a2` me afectan de
lleno — ya no es una suposición, está medido.

Como el proyecto no usa código nativo ni filtros de ABI, el APK es universal y no hay
nada que configurar por arquitectura.
