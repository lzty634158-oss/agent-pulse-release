# Agent Pulse

Agent Pulse sincroniza los estados de trabajo de Claude Code y Codex con un Dashboard local de Windows, una ventana flotante y una luz física tricolor ESP32, para que puedas conocer el progreso de tus sesiones de programación con IA sin tener que mirar continuamente el terminal.

Idioma: [English](README.md) | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Français](README.fr.md) | [Deutsch](README.de.md) | Español

## Versión actual

La versión actual del código fuente es `0.4.1` (2026-08-14) y el firmware ESP32 se mantiene en `0.1.21+22`. Una sola luz conserva el comportamiento sencillo y sigue la tarea más reciente. Con varias luces, cada una puede seguir de forma independiente la tarea más reciente, un proyecto o un agente como Claude Code, Codex, WorkBuddy o CodeBuddy, y utilizar su propio perfil de iluminación.

Esta versión también mantiene visibles todos los dispositivos BLE y USB conectados con su estado de conexión y batería, evita procesos BLE Bridge duplicados y muestra en la ventana flotante el agente y el proyecto de la tarea más reciente. Bluetooth conserva la conexión por proximidad y la conexión emparejada por Windows. Las actualizaciones usan primero Gitee y cambian automáticamente a GitHub si falla. Consulta el [historial de cambios](CHANGELOG.md) para obtener todos los detalles.

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

#### Conexión BLE por proximidad (recomendada)

1. Desconecta el cable de datos USB de la luz y enciéndela. Si ha dejado de anunciarse, pulsa brevemente el botón para reanudar la difusión.
2. Abre el Dashboard. Sin un dispositivo vinculado, Agent Pulse busca continuamente hasta vincular automáticamente, vincular manualmente o detener la búsqueda por decisión del usuario.
3. Acerca la luz deseada al adaptador Bluetooth de este equipo. La lista muestra en tiempo real el nombre, MAC/identificador, RSSI y número de muestras.
4. La vinculación automática requiere al menos 3 muestras, un RSSI de `-45 dBm` o superior y una ventaja mínima de `8 dB` sobre el segundo candidato. Si las diferencias de recepción impiden cumplirlo, selecciona la fila correcta y pulsa «Vincular».

Después de vincular, la búsqueda se detiene y el identificador se guarda en la configuración local. Los siguientes inicios solo conectan con esa luz y no cambian de dispositivo según la señal. Para sustituirla, pulsa «Olvidar dispositivo» y repite la vinculación por proximidad o manual.

El icono BLE es azul durante búsqueda/conexión, verde tras una comunicación válida reciente, gris sin conexión y rojo en caso de error. Tras conectar solo se sincroniza el estado válido actual; no se reproducen eventos de luz históricos caducados. Windows suele mostrar la MAC Bluetooth. Por la privacidad de CoreBluetooth, macOS puede mostrar un UUID asignado por el sistema; permite una vinculación local estable, pero no es la MAC del hardware.

Si no aparece ningún dispositivo, comprueba la alimentación y difusión, el Bluetooth del sistema, que USB esté desconectado y que no haya otra instancia de Agent Pulse/BLE Bridge. En macOS también debes aprobar el permiso Bluetooth la primera vez.

#### Conexión serie USB, selección y recuperación

1. Usa un cable USB con transferencia de datos. Un cable solo de carga no crea un puerto serie.
2. Abre el panel «Puerto serie USB» del Dashboard. Windows usa `COM*`; macOS normalmente usa `/dev/cu.usbmodem*` o `/dev/cu.usbserial*`.
3. La selección automática predeterminada solo conecta con el dispositivo AgentPulse sin controlador `VID:PID 303A:1001`. No abre automáticamente adaptadores CH340, CP210x, FTDI u otros.
4. Si hay varios dispositivos `303A:1001`, o necesitas otro adaptador compatible, revisa el puerto y VID/PID en la lista y selecciona manualmente el destino.

La selección manual se guarda localmente. Si desaparece el puerto elegido, Agent Pulse permanece sin conexión y no abre silenciosamente otro puerto. Puedes volver a selección automática. La lista marca puertos predeterminados, seleccionados, conectados y ausentes; la batería y el estado de carga también aparecen en la cabecera cuando el firmware los admite.

USB tiene prioridad sobre BLE: al conectar USB se pausan la búsqueda y conexión BLE; al desconectarlo se reanudan la búsqueda o reconexión con el dispositivo vinculado. Si no aparece ningún puerto, revisa el cable, los controladores y el firmware USB CDC. USB es el método de recuperación preferido para la primera grabación, migración de particiones o tras un fallo OTA.

## Ventana flotante

Haz clic en «Abrir luz flotante» o «Cerrar luz flotante» en el Dashboard. La ventana flotante muestra el color de estado actual, el nombre del proyecto, el estado BLE y, cuando está disponible, la batería del dispositivo.

![Ventana flotante de escritorio (luz amarilla = en curso)](docs/screenshots/floating-window.png)

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
- Los entornos con varios dispositivos usan un identificador único vinculado de forma persistente. RSSI solo participa en la primera selección por proximidad y nunca cambia una luz ya vinculada. En Windows suele ser una MAC y en macOS puede ser un UUID de CoreBluetooth.
- La actualización de programa y la OTA de firmware son procesos distintos: la actualización de programa instala el EXE de Windows; la OTA de firmware solo escribe la imagen de aplicación en dispositivos compatibles.
