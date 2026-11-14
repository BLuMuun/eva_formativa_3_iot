# Comedero IoT — Evaluación Formativa Unidad 3

Integración de la aplicación Android de Comedero IoT con los nodos ESP32 simulados de Cama y Comedero. La Cama publica lecturas por MQTT/TLS con HMAC; el gateway Comedero mueve el servo y presenta telemetría a la aplicación Panel mediante TCP/5050 protegido con autenticación HMAC mutua y AES-GCM.

## Contenido

- `app/`: aplicación Android en Kotlin y Jetpack Compose.
- `formativa-u3/wokwi/cama/`: sketch y diagrama de la Cama.
- `formativa-u3/wokwi/comedero/`: sketch, diagrama y servo del gateway.
- `docs/FORMATIVA_UNIDAD_3.md`: alcance, preparación y lista de aceptación.
- `docs/INSTALACION.md` y `docs/FIREBASE_LOGIN.md`: configuración local de Android y Firebase.

## Preparación

1. Abre la carpeta raíz en Android Studio. Instala el SDK indicado por Gradle y agrega localmente `app/google-services.json` desde Firebase Console.
2. En cada carpeta Wokwi copia `secrets.example.h` como `secrets.h`; completa las credenciales de HiveMQ y claves aleatorias locales. Nunca publiques esos archivos.
3. Inicia Cama y Comedero en Wokwi. Para un teléfono conectado por USB, ejecuta `adb reverse tcp:5050 tcp:5050`.
4. Inicia sesión en la app, elige ESP32, conecta a `127.0.0.1` y usa la clave local del gateway.

## Estado de verificación

La lista manual de criterios y los puntos que faltan comprobar están en `docs/FORMATIVA_UNIDAD_3.md`. La carpeta contiene firmware preparado para Wokwi; la simulación no sustituye una prueba con placas físicas. La entrega formal de sketches también debe cargarse en AAI según las instrucciones de la asignatura.

No se incluye `google-services.json`, `local.properties` ni ningún `secrets.h`; el `.gitignore` los excluye.