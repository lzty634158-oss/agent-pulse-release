# Agent Pulse

Agent Pulse sincroniza los estados de trabajo de Claude Code y Codex con un Dashboard local de Windows, una ventana flotante y una luz física tricolor ESP32, para que puedas conocer el progreso de tus sesiones de programación con IA sin tener que mirar continuamente el terminal.

Idioma: [English](README.md) | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Français](README.fr.md) | [Deutsch](README.de.md) | Español

## Significado de los estados

| Color | Significado habitual |
|---|---|
| Verde | Sesión inactiva, tarea terminada o resultado disponible para revisar |
| Amarillo | Respondiendo, invocando una herramienta, continuando el procesamiento tras finalizar una herramienta o se necesita información adicional |
| Rojo | Solicitud de permisos, fallo de herramienta, bloqueo o se requiere intervención humana |

Los estados se guardan y muestran por directorio de proyecto; varios proyectos del mismo equipo pueden aparecer simultáneamente en el Dashboard.

## Instalación

### Recomendado: instalador de Windows

Descarga y ejecuta `AgentPulseSetup-<版本>.exe`. Los usuarios normales no necesitan instalar por su cuenta Node.js, npm, Python, BLE Bridge, PyInstaller ni herramientas de Arduino.

Enlaces oficiales de descarga:

- [GitHub Releases](https://github.com/lzty634158-oss/agent-pulse-release/releases)
- [Gitee Releases](https://gitee.com/lzty634158/agent-pulse-release/releases)

El instalador, para el usuario actual de Windows:

- instala Agent Pulse daemon, el entorno de ejecución de Node integrado, BLE Bridge y la ventana flotante;
- combina de forma segura los hooks de Claude Code y los hooks de Codex, sin sobrescribir otros hooks existentes;
- configura el inicio automático tras iniciar sesión;
- inicia Agent Pulse y abre el Dashboard al finalizar.

La ubicación de instalación predeterminada normalmente es:

```text
%LOCALAPPDATA%\AgentPulse
```

Después de instalar, reinicia Claude Code/Codex o abre una nueva sesión para que los hooks se recarguen.

### Instalación desde código fuente/línea de comandos

Esta es la ruta para desarrolladores, no un paso necesario para usuarios del instalador de Windows. Consulta el [apéndice para desarrolladores](#开发者附录) al final del documento.

## Uso diario

### Dashboard

Abre:

```text
http://127.0.0.1:7900
```

También puedes abrirlo desde **Open Dashboard** en el menú Inicio. El Dashboard es el punto de entrada para las operaciones diarias y permite ver:

- proyectos actuales, colores de luz y eventos en tiempo real;
- estado de conexión BLE y batería del dispositivo (cuando el dispositivo lo admite);
- mostrar/ocultar la ventana flotante;
- actualizaciones de programa y firmware;
- acceso a la configuración.

El Dashboard solo escucha en la dirección de loopback local y no se expone a la red local.

### Página de configuración

Haz clic en «Configuración» en el Dashboard para abrir la página de configuración. Su dirección predeterminada es:

```text
http://127.0.0.1:4321/?lang=zh
```

Puedes ajustar las notificaciones, el tiempo de detección de bloqueo, los efectos de color/parpadeo/respiración de cada tipo de evento, el brillo, el sonido y más. `7900` es el Dashboard y `4321` es la página de configuración independiente; tienen finalidades distintas.

### Integración con Claude Code y Codex

El instalador combina los hooks globales de Agent Pulse en:

```text
%USERPROFILE%\.claude\settings.json
%USERPROFILE%\.codex\hooks.json
```

Detecta eventos como el inicio de sesión, envío del usuario, antes y después de llamadas a herramientas, solicitudes de permisos, detenciones y fallos, y actualiza el Dashboard, la ventana flotante y la luz física. Se conservarán tus otros hooks y ajustes.

Validación: abre una nueva sesión de Claude Code o Codex, envía una solicitud y desencadena una llamada a herramienta o una solicitud de permisos; observa los eventos en tiempo real y el color de estado en el Dashboard.

> Codex Offline Sandbox puede bloquear la red de loopback local; Agent Pulse continúa sincronizando el estado mediante la supervisión de archivos de estado locales, sin depender de ese canal de red.

#### Confianza y configuración de los hooks de Codex

Codex debe permitir la ejecución de hooks de comandos externos para que Agent Pulse pueda recibir los eventos de Codex. En la primera instalación o cuando Codex muestre una confirmación de seguridad para hooks, selecciona **confiar/permitir los hooks de Agent Pulse**; si los rechazas o no les das confianza, Codex no ejecutará estos comandos y el Dashboard y la luz física no cambiarán con el estado de Codex.

Pasos de configuración:

1. Abre el Dashboard de Agent Pulse y haz clic en «Configuración».
2. En la página de configuración, haz clic en «Instalar hooks de Codex».
3. Confirma que `%USERPROFILE%\.codex\hooks.json` contiene los hooks de Agent Pulse; la instalación conservará los demás hooks existentes.
4. Reinicia Codex o abre una nueva sesión.
5. Cuando Codex muestre una confirmación para confiar en los hooks o ejecutarlos, selecciona confiar o permitir.
6. Envía una solicitud y desencadena una llamada a herramienta para confirmar que los eventos en tiempo real del Dashboard muestran el estado de Codex.

Si el estado no se actualiza, primero confirma el estado de confianza de los hooks de Codex; después, vuelve a instalar los hooks correspondientes desde la página de configuración y reinicia Codex o abre una nueva sesión. Las instalaciones repetidas no acumulan hooks de Agent Pulse; si has usado una versión antigua y detectas una lentitud notable, vuelve a instalar los hooks una vez para completar la migración de limpieza.

## Luz de estado de hardware

El dispositivo físico actual HW v2 / ESP32-C3-next utiliza **tres LED independientes rojo, amarillo y verde**; no tiene LED azul. Los estados de la luz física son:

- **Luz verde**: tarea completada, sesión inactiva o resultado disponible para revisar.
- **Luz amarilla**: respondiendo, invocando una herramienta o procesando.
- **Luz roja**: solicitud de permisos, fallo de herramienta, bloqueo o se requiere intervención humana.

Los efectos de luz fija, parpadeo y respiración se pueden ajustar libremente en la página de configuración.

> El icono BLE del Dashboard puede mostrarse en azul; solo indica que el equipo está buscando o conectándose por Bluetooth, **no que el dispositivo se ilumine en azul**.

### Encendido, apagado y botón

- **Encendido**: con el dispositivo apagado, mantén pulsado el botón unos 2 segundos; el dispositivo bloqueará la alimentación y se iniciará.
- **Respuesta de encendido**: el dispositivo muestra Rojo → Amarillo → Verde en secuencia, luego entra en el estado predeterminado de parpadeo verde y difusión BLE; si el sonido está activado, reproduce un tono de inicio.
- **Apagado**: después de encenderlo, vuelve a mantener pulsado unos 2 segundos; el dispositivo apaga las luces y desactiva la retención de alimentación; si el sonido está activado, reproduce un tono de apagado.
- **Pulsación corta**: muestra la batería durante unos 2 segundos; si aún no hay conexión BLE, también reinicia/despierta la difusión.
- **Durante una actualización**: las operaciones del botón se ignoran durante la transferencia OTA para evitar interrumpir accidentalmente la actualización.

### Luz física y respuesta de identificación

- **Difundiendo y esperando conexión**: la luz verde respira.
- **BLE conectado**: la luz verde queda fija; a continuación se restaura/recibe el estado actual del agente.
- **Conexión desconectada**: el dispositivo vuelve a difundir y la luz verde vuelve a respirar.
- **Sin conexión después de unos 60 segundos**: deja de difundir y la luz roja parpadea; una pulsación corta puede iniciar de nuevo la difusión.
- **Identificar dispositivo**: al ejecutar la identificación desde el Dashboard, el dispositivo muestra rápidamente Rojo → Amarillo → Verde → Apagado, repite el ciclo varias veces y luego recupera el estado anterior.
- **Animación de conexión**: el dispositivo indica el proceso de conexión en orden rojo, amarillo, verde; cuando se completa la conexión, el host envía el estado de trabajo actual.

### Batería, carga y sonido

Una pulsación corta del dispositivo muestra la batería durante unos 2 segundos:

| Estimación de voltaje | Efecto de luz |
|---|---|
| Aprox. ≥ 4.0 V | Se encienden rojo, amarillo y verde |
| Aprox. 3.7–4.0 V | Se encienden rojo y amarillo |
| Aprox. < 3.7 V | Se enciende la luz roja |

Cuando está conectado y el dispositivo lo admite, el Dashboard y la ventana flotante muestran el voltaje estimado, el porcentaje y el estado de carga. Los valores sirven para una valoración diaria y no deben utilizarse como medidor de batería de precisión.

El interruptor «Sonido» de la página de configuración controla los tonos del zumbador; está desactivado de forma predeterminada. Los ajustes de brillo tricolor y sonido se guardan en el dispositivo y se conservan tras cortar la alimentación.

### Métodos de conexión

#### Conexión BLE (recomendada)

1. Alimenta el dispositivo; si no ha estado conectado durante mucho tiempo, haz una pulsación corta para que vuelva a difundir.
2. Abre el Dashboard y espera a que el estado BLE cambie de buscando/conectando a conectado.
3. Tras conectarse correctamente, Agent Pulse sincronizará automáticamente el estado actual con la luz física.

En general, el icono BLE del Dashboard indica: azul para buscando/conectando, verde para interacción válida del dispositivo recibida recientemente, gris para no conectado y rojo para error de conexión. El azul solo es el estado de un icono de software, no de un LED físico.

Si no se encuentra el dispositivo, confirma que esté alimentado y difundiendo, que Bluetooth de Windows esté disponible, que el dispositivo esté lo bastante cerca y evita ejecutar varias instancias de Agent Pulse/BLE Bridge al mismo tiempo.

#### Conexión USB, diagnóstico y recuperación

USB se puede usar para controlar la luz por cable, leer información/batería del dispositivo, diagnóstico, recuperación y actualizaciones de firmware de dispositivos compatibles. Usa un cable de datos en lugar de un cable solo de carga y confirma en el Administrador de dispositivos que aparezca un puerto COM.

La versión actual filtra los dispositivos candidatos por la identificación del fabricante de los puertos serie USB. Si hay varios ESP32 u otros dispositivos serie USB comunes conectados, especifica explícitamente el puerto de destino en la línea de comandos, por ejemplo:

```powershell
agent-traffic-light-monitor device list
agent-traffic-light-monitor device push --port COM3
```

No utilices dispositivos serie no relacionados como destino del control de luz de Agent Pulse. Las versiones futuras enviarán primero una solicitud `deviceInfo` a los puertos candidatos y solo se conectarán automáticamente después de recibir una respuesta de dispositivo válida.

Si el dispositivo no tiene ningún puerto serie, revisa el cable, los controladores y si el firmware ha habilitado USB CDC. USB es el método de recuperación preferido después de una primera grabación, una migración de particiones o un fallo de OTA.

## Ventana flotante

Haz clic en «Abrir luz flotante» o «Cerrar luz flotante» en el Dashboard. La ventana flotante muestra el color de estado actual, el nombre del proyecto, el estado BLE y, cuando está disponible, la batería del dispositivo.

La administra el daemon de la versión instalada; aunque no pueda iniciarse la ventana flotante, el Dashboard y la sincronización de estado pueden seguir funcionando.

## Actualización de programa

En el área **Actualización de programa de AgentPulse** del Dashboard:

1. Haz clic en «Buscar actualizaciones de programa».
2. Cuando se encuentre una nueva versión, haz clic en «Confirmar instalación».
3. Agent Pulse descarga y valida el manifiesto de actualización firmado, el nombre del paquete de instalación, el tamaño y SHA-256.
4. Cuando la validación se completa, el Explorador de Windows se abre y selecciona el paquete de instalación validado.
5. Haz doble clic manualmente en ese EXE y completa la instalación en el asistente visible de Inno Setup.

Agent Pulse no ejecuta el paquete de instalación de forma silenciosa ni completa el asistente de instalación por ti. Durante una instalación superpuesta, el asistente de Inno cierra el daemon de Agent Pulse, la ventana flotante y BLE Bridge del directorio de instalación actual para liberar archivos en uso; no cierra de forma generalizada programas no relacionados.

Los paquetes de instalación descargados se almacenan en caché de forma predeterminada en:

```text
%LOCALAPPDATA%\AgentPulse\updates\desktop\
```

## Actualización de firmware

Capacidades de hardware:

- **Firmware actualizable ESP32-C3-next**: la información del dispositivo debe informar la siguiente identificación de hardware y capacidad OTA para poder utilizar OTA por BLE/USB desde el Dashboard:

  ```text
  agentpulse-esp32c3-next
  ```

Antes de actualizar, confirma la información del dispositivo y el firmware de destino. OTA solo acepta **imágenes de aplicación** de Arduino `.ino.bin`; no cargues bootloader, tabla de particiones, `merged.bin` ni otros archivos completos de primera grabación.

### Limitaciones importantes

La OTA actual sigue siendo una función de laboratorio: el firmware aún no implementa validación de firma de imagen, Secure Boot, Flash Encryption, confirmación de estado correcto ni reversión automática. No cortes la alimentación, desconectes el cable, desactives Bluetooth ni cierres el daemon durante la actualización; si falla, prioriza la recuperación por USB.

***Se recomienda conectar la alimentación durante la actualización para evitar que una pérdida repentina de energía provoque un fallo de actualización que afecte al uso.***

La disposición de particiones OTA de dispositivos antiguos no se puede migrar mediante una OTA de aplicación BLE/USB normal. Para migrar la disposición de particiones o realizar la primera grabación, se deben grabar por completo bootloader, tabla de particiones, OTA boot selector y factory app mediante el modo de descarga USB/bootloader.

## Datos y privacidad

Los datos de ejecución predeterminados se guardan localmente:

```text
%LOCALAPPDATA%\AgentPulse\
  config.json
  projects\<projectId>\status.json
  projects\<projectId>\events.jsonl
  daemon\
  updates\
```

De forma predeterminada, Agent Pulse no carga código, prompts, salida de terminal ni archivos de proyecto. El antiguo `.agent-pulse` de la raíz del proyecto solo se usa para compatibilidad/migración; las versiones nuevas ya no escriben datos de ejecución nuevos en directorios de proyecto.

## Preguntas frecuentes

### No se puede abrir el Dashboard

Confirma que accedes a `http://127.0.0.1:7900`, y no al puerto `4321` de la página de configuración. En la versión instalada puedes intentar reiniciar Agent Pulse desde el menú Inicio; los desarrolladores pueden comprobarlo desde la línea de comandos:

```powershell
agent-traffic-light-monitor daemon status
agent-traffic-light-monitor daemon logs
```

No ejecutes simultáneamente el daemon desde código fuente y la versión instalada; competirán por `7900`, `47801`, `7950` y el dispositivo BLE.

### El estado de Claude Code/Codex no cambia

1. Abre una nueva sesión de Claude Code/Codex.
2. Vuelve a instalar los hooks correspondientes en la página de configuración.
3. Confirma que `%USERPROFILE%\.claude\settings.json` o `%USERPROFILE%\.codex\hooks.json` aún contienen la configuración de Agent Pulse.
4. Los usuarios de Claude Code pueden ejecutar:

   ```powershell
   agent-traffic-light-monitor doctor
   ```

### BLE no puede conectarse

Comprueba la alimentación del dispositivo, Bluetooth de Windows, la distancia y el estado del Dashboard; no inicies manualmente un BLE Bridge adicional para evitar ocupar `47801`.

### No se encuentra el dispositivo USB

Usa un cable de datos, revisa «Puertos (COM y LPT)» en el Administrador de dispositivos y, si es necesario, selecciona un puerto COM explícito. Si no hay puerto COM, revisa el firmware USB CDC y los controladores.

### Las notificaciones son demasiado frecuentes

Desactiva los avisos de finalización/error/bloqueo en la página de configuración o ajusta el tiempo de detección de «bloqueado».

## Notas

- Este documento está dirigido a usuarios del instalador de Windows. Antes de usarlo en producción, valida los hooks de Claude/Codex, BLE, la ventana flotante y los flujos de actualización en el equipo y hardware de destino.
- Actualmente, varios dispositivos Agent Pulse no deben distinguirse automáticamente solo mediante el mismo nombre BLE; los futuros escenarios con varios dispositivos deben usar una vinculación `deviceId` única, y RSSI solo es adecuado como base de ordenación durante el primer descubrimiento.
- La actualización de programa y la OTA de firmware son procesos distintos: la actualización de programa instala el EXE de Windows; la OTA de firmware solo escribe la imagen de aplicación en dispositivos compatibles.


