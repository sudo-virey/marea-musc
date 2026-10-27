<div align="center">

# Marea Music

**Tu biblioteca musical, a tu manera.**

Reproductor de música personal creado con **Flutter** para Android y iOS, con reproducción local, biblioteca offline, búsqueda de contenido y un gestor experimental de descargas en el dispositivo.

**Versión actual: 0.5.0+5 · Beta experimental**

</div>

> [!IMPORTANT]
> Marea Music es un proyecto independiente, no afiliado a YouTube, YouTube Music, Google ni a sus titulares de derechos. La búsqueda de YouTube muestra información pública mediante la YouTube Data API v3; **no concede derechos para descargar música**. Usa las funciones de descarga solo para contenido propio o cuya descarga esté expresamente permitida, y respeta las condiciones de cada servicio.

## Características

- **Reproductor local:** importa y escucha los archivos de audio disponibles en tu dispositivo, sin anuncios añadidos por Marea Music.
- **Biblioteca personal:** canciones, favoritos y listas de reproducción.
- **Reproducción en segundo plano:** integración con `just_audio`, `audio_session`, `audio_service` y `just_audio_background` para controles multimedia y pantalla bloqueada; requiere configuración nativa y pruebas en cada sistema.
- **Búsqueda de contenido:** consulta canciones y videos musicales mediante **YouTube Data API v3**; muestra títulos, canales y miniaturas.
- **Cola de descargas:** permite agregar resultados a una cola, mostrar el progreso y registrar pendientes.
- **Motor experimental:** integra [`ytdlp_flutter`](https://pub.dev/packages/ytdlp_flutter) para inspeccionar enlaces y gestionar descargas de fuentes autorizadas desde el teléfono, sin exigir una computadora.
- **Detección de duplicados:** por identificador de origen y hash SHA-256 de archivos idénticos. No es una huella acústica.
- **Sincronización opcional:** herramientas de escritorio y servidor Python para transferir una biblioteca personal desde Windows, macOS o Linux a un dispositivo móvil en la misma red.

## Estado del proyecto

| Componente | Estado |
| --- | --- |
| Interfaz y biblioteca Flutter | Implementadas; en pruebas |
| Búsqueda con YouTube Data API v3 | Probada manualmente en iPhone |
| Integración de `ytdlp_flutter` en iOS | Experimental; arranque y consulta probados en dispositivo |
| Descargas y cola en iOS | En pruebas; dependen de fuente y formato |
| Reproducción con pantalla bloqueada | Configurada para iOS y Android; funcionamiento sujeto a pruebas por dispositivo |
| Compilación Android | APK generado y ejecución de depuración confirmada; requieren pruebas funcionales en dispositivo real |
| Sincronización mediante PC | Opcional; prototipo |

No todos los proveedores ni formatos multimedia están soportados. La reproducción con pantalla bloqueada **no implica** que las descargas sigan ejecutándose cuando iOS suspende la aplicación.

## Tecnologías

| Área | Herramientas |
| --- | --- |
| Aplicación móvil | Flutter / Dart |
| Audio | `just_audio`, `just_audio_background`, `audio_service`, `audio_session` |
| Extracción experimental | `ytdlp_flutter` 0.1.0 |
| Búsqueda | YouTube Data API v3 |
| Preferencias y archivos | `shared_preferences`, `path_provider`, `file_picker` |
| Duplicados | `crypto` (SHA-256) |
| Sincronización opcional | Python, red local |

## Requisitos

- Flutter SDK compatible con el `pubspec.yaml` del repositorio (Dart >= 3.10).
- Android SDK y herramientas de compilación para Android.
- macOS, Xcode y CocoaPods para compilar iOS.
- Un dispositivo físico para validar el motor de descargas y el audio en segundo plano.
- Una clave personal de YouTube Data API v3 para activar la búsqueda.

Verifica tu instalación:

```bash
flutter doctor -v
flutter pub get
```

## Ejecutar en Android

Si clonaste el repositorio y no existe la carpeta `android/`, genera los archivos nativos con Flutter:

```bash
flutter create --platforms=android .
flutter pub get
```

El proyecto necesita configuración nativa adicional (no basta con ejecutar `flutter create`). En `android/app/src/main/AndroidManifest.xml`, dentro de `<manifest>` y antes de `<application>`, declara:

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.WAKE_LOCK" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_MEDIA_PLAYBACK" />
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
```

Dentro de `<application>`, utiliza como actividad principal `com.ryanheise.audioservice.AudioServiceActivity` (conservando los atributos y filtros de la actividad que creó Flutter) y registra estos componentes:

```xml
<service
    android:name="com.ryanheise.audioservice.AudioService"
    android:foregroundServiceType="mediaPlayback"
    android:exported="true">
    <intent-filter>
        <action android:name="android.media.browse.MediaBrowserService" />
    </intent-filter>
</service>
<receiver
    android:name="com.ryanheise.audioservice.MediaButtonReceiver"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MEDIA_BUTTON" />
    </intent-filter>
</receiver>
```

Para el empaquetado en `android/app/build.gradle.kts`, incorpora dentro de `android { ... }`:

```kotlin
packaging {
    jniLibs {
        useLegacyPackaging = true
        keepDebugSymbols.add("**/libpython.zip.so")
        keepDebugSymbols.add("**/libffmpeg.zip.so")
    }
}
```

**No agregues `android:extractNativeLibs="true"` directamente en `AndroidManifest.xml`:** con el Android Gradle Plugin de las pruebas actuales produce un error en `:app:packageDebug`. Revisa asimismo el `minSdk` exigido por la versión instalada de `ytdlp_flutter` (en las pruebas se utilizó 24). Las advertencias `llvm-strip` para los archivos ZIP camuflados como `.so` se abordan con `keepDebugSymbols`. Para detalle de estas integraciones consulta la documentación de [`just_audio_background`](https://pub.dev/packages/just_audio_background) y [`ytdlp_flutter`](https://pub.dev/packages/ytdlp_flutter).

```bash
flutter run
```

Para generar el APK:

```bash
flutter build apk --release
```

Ruta habitual del archivo resultante:

```text
build/app/outputs/flutter-apk/app-release.apk
```

Para generar APK por arquitectura:

```bash
flutter build apk --release --split-per-abi
```

> [!NOTE]
> El plugin `ytdlp_flutter` puede emitir una advertencia sobre el Kotlin Gradle Plugin y su futura migración a Built-in Kotlin. Una advertencia no implica por sí sola que la compilación haya fallado; revisa el resultado final de Gradle.

## Ejecutar en iOS

```bash
flutter pub get
cd ios
pod install
cd ..
open ios/Runner.xcworkspace
```

En Xcode, configura tu equipo de desarrollo en **Runner → Signing & Capabilities** y habilita **Background Modes → Audio, AirPlay, and Picture in Picture** para música local.

### Python integrado en iOS

`ytdlp_flutter` necesita que la biblioteca estándar de Python quede incluida en la aplicación. En **Runner → Build Phases**, al final del target, agrega una fase de tipo **Run Script** con este contenido:

```sh
set -e
sh "$PODS_ROOT/../.symlinks/plugins/ytdlp_flutter/ios/scripts/install_python_stdlib.sh"
```

Desmarca **Based on dependency analysis** para esa fase. Después ejecuta:

```bash
flutter run
```

No basta con un *hot reload* tras cambiar las fases nativas. El motor de Python para iOS sigue siendo experimental.

Para instalar una versión que no dependa de una sesión de depuración activa:

```bash
flutter devices
flutter run --release -d <ID_DE_TU_IPHONE>
```

Esta instalación **sigue sujeta a la firma de Apple**: con un Apple ID gratuito los perfiles de aprovisionamiento de desarrollo suelen caducar después de siete días, por lo que puede ser necesaria una reinstalación. Para distribución pública se requiere un método autorizado de firma y distribución.

## Configuración de búsqueda

1. Crea un proyecto en [Google Cloud Console](https://console.cloud.google.com/).
2. Habilita **YouTube Data API v3**.
3. Crea una **API Key** para consultar datos públicos.
4. Restringe la clave a **YouTube Data API v3** y configura las restricciones de aplicación adecuadas antes de distribuir.
5. Dentro de Marea Music, abre **Descargas → Configurar búsqueda**, introduce la clave y realiza una consulta.

**No subas tus claves a GitHub.** Evita incluirlas en ejemplos, capturas, archivos de configuración o commits. Una API Key incluida dentro de una app cliente no puede considerarse completamente secreta; aplica restricciones y cuotas. Si en el futuro el proyecto necesita controles más fuertes, considera un backend propio.

## Organización general

```text
marea_music/
├── lib/
│   ├── main.dart
│   ├── downloads_screen.dart
│   ├── mac_sync_screen.dart
│   ├── models/
│   └── services/
├── assets/
├── test/
├── desktop/            # Herramientas opcionales para escritorio
├── mac_sync/           # Servidor opcional de sincronización
├── tools/              # Configuración nativa
├── android/            # Proyecto nativo generado/configurado
├── ios/                # Proyecto nativo y ajustes de Xcode
└── pubspec.yaml
```

## Pruebas

```bash
flutter analyze
flutter test
```

Para las herramientas opcionales de Python, ejecuta sus pruebas por separado según las dependencias indicadas en `desktop/`.

Las pruebas automatizadas no sustituyen una verificación real en Android/iOS: prueba importación, descargas autorizadas, archivos repetidos, controles de reproducción, audio con el teléfono bloqueado y recuperación tras interrupciones.

## Almacenamiento de canciones: limitación actual

Por ahora, **los archivos de audio descargados por Marea Music se guardan en el almacenamiento gestionado por la aplicación**. Desinstalarla puede eliminar también las canciones y los metadatos asociados. **No desinstales la app si no tienes una copia de tu biblioteca.**

El almacenamiento persistente externo **no está implementado todavía**. Se contempla para una versión posterior:

- **Android:** exportación a la colección compartida `Music/Marea Music/` mediante MediaStore.
- **iOS:** exportación manual a una ubicación elegida con Archivos (por ejemplo, iCloud Drive), considerando los permisos y la disponibilidad offline.
- Migración segura y reindexación de canciones, sin borrar la copia original hasta comprobar la exportación.

La versión actual no promete conservar la música después de desinstalar ni sincronizar archivos automáticamente con iCloud.

## Problemas conocidos y soluciones

| Síntoma | Causa habitual / medida |
| --- | --- |
| `android/app/build.gradle` no existe | La carpeta `android/` puede faltar; ejecuta `flutter create --platforms=android .` y aplica la configuración nativa anterior. Los proyectos recientes utilizan `build.gradle.kts`. |
| Error en `:app:packageDebug` sobre `extractNativeLibs` | Elimina el atributo del manifiesto y usa `packaging.jniLibs.useLegacyPackaging = true` en Gradle Kotlin DSL. |
| `AudioServiceActivity` / `FlutterEngine` incorrecto | Declara la actividad y los servicios de `audio_service` como se documenta arriba. |
| `llvm-strip` no reconoce `libpython.zip.so` o `libffmpeg.zip.so` | Evita el stripping de esos artefactos mediante `keepDebugSymbols`. |
| Aviso Kotlin Gradle Plugin de `ytdlp_flutter` | Compatibilidad futura pendiente de la dependencia; no necesariamente bloquea el build actual. |
| `NO_STDLIB` en iOS | Revisa la fase `Install Python Stdlib` en Xcode y recompila completamente. |
| `SQLITE_BUSY` al iniciar | Se observó en registros del caché; pendiente de investigar si reaparece. No borres datos sin respaldarlos. |

## Limitaciones conocidas

- La compatibilidad de `ytdlp_flutter` depende de las implementaciones nativas, las actualizaciones de los sitios y sus mecanismos de extracción.
- iOS y Android pueden ofrecer comportamientos diferentes. El soporte de iOS es experimental.
- La compatibilidad con FFmpeg y los formatos descargables no es idéntica entre plataformas.
- No se garantiza la descarga en segundo plano ni la descarga de cualquier enlace.
- La biblioteca actualmente depende del almacenamiento de la aplicación: desinstalar puede eliminar las canciones.
- Los hashes detectan archivos binariamente idénticos, pero no diferentes codificaciones de la misma grabación.
- La sincronización local opcional requiere una red privada de confianza; su implementación HTTP actual no proporciona cifrado de transporte.

## Contribuir

Se agradecen reportes reproducibles de errores, propuestas y *pull requests*. Al reportar un problema, incluye:

- Plataforma y versión del sistema operativo.
- Salida de `flutter --version`.
- Pasos para reproducir y mensaje de error (sin claves, cookies ni tokens).
- Si es pertinente, versión del paquete `ytdlp_flutter`.

## Seguridad y privacidad

No publiques API Keys, tokens de sincronización, cookies, archivos de música privados ni registros que contengan credenciales. Antes de volver público el repositorio, revisa **todos los commits del historial**, además del estado actual del directorio.

## Licencias y atribuciones

El código original de Marea Music necesita un archivo `LICENSE` elegido por sus titulares antes de declarar una licencia de distribución. **No se debe asumir que el proyecto completo usa MIT solo porque alguna dependencia la utilice.** Revisa las licencias de Flutter, `yt-dlp`, `ytdlp_flutter` y de los componentes nativos incluidos, especialmente antes de distribuir binarios.

---

**Marea Music** es un proyecto de desarrollo público experimental (la licencia del código original debe definirse antes de declararlo formalmente de código abierto). No incluye ni distribuye un catálogo de música comercial.
# marea-musc
