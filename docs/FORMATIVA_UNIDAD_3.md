# Formativa Unidad 3: integración segura Android–IoT

## Alcance

Esta entrega reutiliza el prototipo de cama y comedero y añade un modo Android **Panel ESP32**. La app muestra los rasguños, el estado del servo y permite solicitar la entrega desde el Moto. Cama publica las lecturas por MQTT/TLS; el gateway Comedero valida su HMAC, maneja el servo y entrega el estado a Android por Wi-Fi.

```text
Botón/vibración → ESP32 Cama → MQTT/TLS + HMAC → ESP32 Gateway → Servo
                                                   ↕ Wi-Fi TCP + AES-GCM
                                              Android Panel
```

## Protección del enlace Android–gateway

- El gateway emite un nonce aleatorio para cada sesión. Android demuestra que conoce el secreto con HMAC-SHA-256 y verifica una prueba equivalente del gateway antes de marcarse conectado.
- Ambos derivan una clave AES-256 de sesión. Cada trama lleva IV aleatorio y etiqueta GCM; la dirección, el nonce y el número secuencial forman parte de los datos autenticados. Se rechazan alteraciones, mensajes repetidos y secuencias fuera de orden.
- El secreto del Panel es distinto del HMAC de MQTT. Se configura localmente en `secrets.h`, se ingresa en la pantalla de conexión y no se guarda en preferencias ni se imprime en los logs. Usa al menos 32 caracteres aleatorios.
- MQTT entre las placas conserva TLS y HMAC para autenticidad e integridad. Estas decisiones aplican controles de autenticación, confidencialidad e integridad asociados a ISO/IEC 27000; no constituyen certificación ni una auditoría formal.

Las tramas TCP son líneas delimitadas, con payload binario codificado en Base64. El puerto local de Wokwi se reenvía por ADB al teléfono. La consola UART a 115200 baudios entrega diagnósticos de firmware; no transmite el secreto.

## Preparación

1. En VS Code abre cada carpeta por separado: `formativa-u3/wokwi/cama` y `formativa-u3/wokwi/comedero`.
2. En cada carpeta copia `secrets.example.h` a `secrets.h`. Completa los mismos datos HiveMQ y `HMAC_SECRET_VALUE` que ya usan los dos nodos. No subas `secrets.h`.
3. En `comedero/secrets.h`, reemplaza `PANEL_HMAC_SECRET_VALUE` por una cadena aleatoria nueva de 32 bytes como mínimo. Puedes generarla en PowerShell:

   ```powershell
   [Convert]::ToHexString([Security.Cryptography.RandomNumberGenerator]::GetBytes(32))
   ```

   Copia ese valor únicamente al campo secreto del modo ESP32 en la app. No lo uses como contraseña de Firebase o HiveMQ.
4. Inicia las dos simulaciones con **Wokwi: Start Simulation** y espera MQTT/TLS en Cama y Comedero. Comedero escucha el puerto TCP 5050.
5. Conecta el teléfono al PC por USB con depuración autorizada y ejecuta:

   ```powershell
   adb reverse tcp:5050 tcp:5050
   ```

6. Instala/abre la app, inicia sesión, elige destino **ESP32**, escribe `127.0.0.1` y el secreto del gateway. Conecta el Panel. Wokwi debe registrar autenticación y canal AES-GCM activo; la app debe reflejar `CONECTADO`.
7. Pulsa el botón de Cama y confirma que la lectura llega a Android. Con tres pulsaciones dentro de cinco segundos, la app debe mostrar `3/3`; pulsa **Dispensar comida** y comprueba apertura, cierre del servo y confirmación. Al perder el Panel, el gateway cancela la solicitud y deja el servo cerrado.

Para retirar el reenvío USB: `adb reverse --remove tcp:5050`. Si el teléfono y el gateway comparten una red enrutable real, escribe la IP del gateway en lugar de `127.0.0.1`.

## Relación con la guía y verificación pendiente

| Criterio de la guía | Implementación | Evidencia a recopilar |
|---|---|---|
| 3.1.1 Envío protegido | HMAC de autenticación y AES-GCM por trama en Android↔gateway; MQTT/TLS y HMAC entre nodos. | Captura del Panel conectado, telemetría recibida y entrega; logs sin secretos. |
| 3.1.2 Comunicación serial segura | El firmware conserva consola UART de diagnóstico; el canal de control de la app usa TCP Wi-Fi cifrado. | Aclarar con la pauta si “serial” exige UART físico. Esta versión no presenta TCP como UART ni afirma haber demostrado un enlace USB/UART desde Android. |
| 3.1.3 Instrucciones inalámbricas | Cama reporta rasguños por MQTT/TLS; Android puede enviar dispensación al gateway por Wi-Fi seguro. | Capturas de lecturas 1/3, 2/3 y 3/3; movimiento y cierre del servo. |
| 3.1.4 Android conectado al IoT | Nuevo modo Panel ESP32 con secreto, autenticación mutua, estados y control. | Captura de `CONECTADO` y ejecución desde el teléfono. |

La compilación de código no reemplaza la demostración en Wokwi ni en hardware. Para presentar, registra fecha, modelo Android, resultado por paso y evidencia visual. La pauta de apreciación citada en el documento no venía adjunta, por lo que esta tabla sigue los criterios escritos de la guía.

## Pruebas manuales de aceptación

- [ ] Clave incorrecta: el Panel no autentica; el gateway sigue con el servo cerrado.
- [ ] Trama alterada o repetida: no modifica lecturas ni activa el actuador.
- [ ] Tres pulsaciones antes de cinco segundos: aparece `3/3` y la orden manual habilita la entrega.
- [ ] Menos de tres o expiración de la ventana: queda `0/3` y no se permite dispensar.
- [ ] La telemetría indica `DISPENSANDO` y vuelve a `CERRADO` al terminar.
- [ ] Pérdida del Panel: se cancela de forma segura y se cierra el servo.
- [ ] Verificar la misma conexión con el segundo teléfono disponible y guardar capturas/logs sin credenciales.
