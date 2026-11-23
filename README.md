# Z4firo

[**Descargar el APK**](https://github.com/ZafiroBotAss/ZafiroBot/releases/latest/download/Z4firo.apk) · [Todas las versiones](https://github.com/ZafiroBotAss/ZafiroBot/releases)

App Android nativa (Kotlin, Jetpack Compose, Material 3) de chat inspirada en Grok, impulsada por la **API oficial de Anthropic** (Messages API con streaming SSE).

## Características

- Chat en streaming con Claude. Modelo por defecto `claude-sonnet-5-5`, configurable en Ajustes.
- Historial de conversaciones guardado con Room: nuevo chat, renombrar y eliminar desde el cajón lateral.
- Adjuntos: imagen de la galería o de la cámara (base64), PDF (bloque `document`) y archivos de texto (en línea).
- Búsqueda web en vivo con la herramienta de servidor `web_search_20250305` (máx. 5 búsquedas), con interruptor en la barra inferior y chips de fuentes citadas.
- Voz: dictado con `SpeechRecognizer` (botón de micrófono) y lectura en voz alta con `TextToSpeech` en cada respuesta.
- Generación de imágenes con `/imagine <descripción>` o con el botón de imagen. Se muestra en línea y puedes guardarla.
- Botón de copiar en cada mensaje y botón de parar mientras responde.
- Tema oscuro por defecto. La personalidad (prompt del sistema) es editable.

## Requisitos

- Android Studio (o solo el SDK de línea de comandos).
- JDK 17.
- Android SDK con la plataforma 35 y build-tools.
- Variable de entorno `ANDROID_HOME` apuntando al SDK. No hace falta `local.properties`: el proyecto no lo necesita.
- Una API key de Anthropic (empieza por `sk-ant-`).

## Compilar

El wrapper de Gradle 8.10.2 ya está incluido, así que no hace falta instalar Gradle. La primera compilación descarga Gradle y las dependencias, por lo que necesita conexión.

Windows (desde la raíz del proyecto):

```bat
gradlew.bat assembleDebug
```

Linux / macOS:

```bash
chmod +x gradlew    # solo la primera vez
./gradlew assembleDebug
```

El APK de depuración queda en:

```
app/build/outputs/apk/debug/app-debug.apk
```

Para instalarlo en un dispositivo con depuración USB:

```bash
adb install app/build/outputs/apk/debug/app-debug.apk
```

## Configuración de la API key

La app no contiene ninguna clave. Al instalarla:

1. Abre el menú (☰) y entra en **Ajustes**.
2. Pega tu API key de Anthropic. Se guarda cifrada en el dispositivo con `EncryptedSharedPreferences` (AES-256).
3. Opcional: cambia el ID del modelo y el prompt del sistema, y pulsa **Guardar**.

Si la clave no es válida, la app muestra un aviso en un snackbar. Lo mismo ocurre con el límite de peticiones (429) y con errores de red.

## Uso

- Escribe tu mensaje y pulsa enviar. Mientras responde, el botón cambia a parar.
- Clip: adjuntar imagen (galería o cámara), PDF o archivo de texto.
- Micrófono: dictar el mensaje. El texto se añade al borrador.
- Altavoz bajo cada respuesta: leerla en voz alta.
- `/imagine un gato astronauta` (o el botón de imagen) genera una imagen. Con el icono de descarga eliges dónde guardarla.

## Servicio de terceros para imágenes

La generación de imágenes usa **Pollinations** (`https://image.pollinations.ai/prompt/...`). Es un servicio **independiente** de Anthropic, gratuito y sin clave. Su disponibilidad, calidad y condiciones dependen de ese servicio. Claude no genera imágenes. La implementación es intercambiable porque la app depende de la interfaz `ImageGenerator` (paquete `domain/imagegen`).

## Arquitectura

```
com.dremarc.grokbot
├── data/     Room, EncryptedSharedPreferences, OkHttp + SSE, Pollinations, adjuntos, voz
├── domain/   modelos, interfaces de repositorio, ImageGenerator
└── ui/       Compose: chat, ajustes, tema, navegación y ViewModels con StateFlow
```

Las dependencias se crean a mano en `AppContainer`. Las versiones están centralizadas en `gradle/libs.versions.toml`.

## Límites y notas

- Las peticiones van directamente del dispositivo a `api.anthropic.com`. La clave viaja en la cabecera `x-api-key`.
- Máximo 5 adjuntos por mensaje. PDF de hasta 20 MB, texto de hasta 512 KB. Las imágenes se reducen a 1568 px en el lado largo y se convierten a JPEG.
- Los adjuntos se envían solo en el turno en que se añaden. En turnos posteriores el historial conserva el texto del mensaje.
- La búsqueda web consume búsquedas de tu cuenta de Anthropic y puede tener coste.
