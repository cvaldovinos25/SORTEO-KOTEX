# Kotex ¡SORTEO! en Salcobrand: atrapa los productos (WebAR + NFC)

Experiencia web de realidad aumentada para celulares. La clienta toca el sticker NFC (o escanea el QR) de la góndola, se abre la página, apunta la cámara y juega **una sola vez**: de la caja Kotex en 3D salen **7 productos** que flotan a su alrededor y tiene que **encontrarlos moviendo el celular y atraparlos tocando la pantalla, lo más rápido posible**. Mientras menos tarde, más puntos (máximo 500) y **más probabilidad de ganar la PR Box de Kotex**. Al terminar recibe un **código único por jugada** con su **hora exacta**, registra nombre y celular, y todo queda en una planilla de Google Sheets lista para el sorteo.

```
kotex-ra/
├── index.html              Experiencia completa (cámara + caja 3D + juego + registro)
├── README.md               Este documento
└── apps-script/
    ├── Code.gs             Backend: guarda jugadas en Google Sheets y hace el sorteo ponderado
    └── appsscript.json     Manifiesto del script
```

Publicada en: https://cvaldovinos25.github.io/SORTEO-KOTEX/

## Cómo se juega

1. **Bienvenida.** Muestra el logo "Kotex ¡SORTEO!", las dos reglas (1 sola jugada; más puntaje, más probabilidad) y el botón **¡A jugar!**. Antes hay una animación de carga mientras se prepara la página.
2. **Permisos.** Al tocar "¡A jugar!" el celular pide **cámara** y **movimiento** (en iPhone aparece un aviso de permiso). Sin la cámara no se puede jugar.
3. **Colocar la caja.** La clienta apunta al espacio destinado y toca **"Colocar la caja aquí"**. La caja Kotex queda **fija en ese punto del espacio**.
4. **Cuenta regresiva** 3, 2, 1 y parte el cronómetro.
5. **Atrapar.** De la caja salen 7 productos que se reparten en los **360° alrededor** (también por detrás): CD "I LIKE IT", cámara instantánea, toallas Kotex Normal y Nocturna, sticker holográfico BONITA y ampolleta de camarín. Un texto fijo indica "Escanea tu alrededor para encontrar los objetos". Hay que girar el celular, **centrar el producto en la mira** y **tocar la pantalla**. Los productos se desplazan lentamente y el cronómetro arriba muestra el tiempo y cuántos lleva ("⏱ 0:12.4 · 3 de 7").
6. **Ayuda.** Si pasan 6 segundos sin atrapar nada, aparece una flecha que indica hacia dónde está el producto más cercano.
7. **Resultado.** Al atrapar los 7 (o al acabarse el tiempo) la caja se abre con confeti, el puntaje sube contando y aparece el código de la jugada. Luego registra nombre, celular y acepta las bases.

Equipos sin sensor de movimiento (por ejemplo un computador) usan un plan B: se arrastra el dedo para mirar alrededor.

## Puntaje

| Resultado | Puntos |
|---|---|
| Atrapa los 7 en **30 s o menos** | **500** (máximo) |
| Atrapa los 7 entre 30 s y 120 s | baja de forma pareja de 500 a 50 |
| Se acaba el tiempo (120 s) | hasta 50, en proporción a los productos atrapados |

Tocar donde no hay un producto no quita puntos. El tiempo no se guarda en la planilla: solo el puntaje que resulta de él.

## Reglas de participación (se muestran antes de jugar)

1. **Solo se puede jugar 1 vez.**
2. **Más puntaje, más probabilidad de ganar.** Cada punto es un boleto en el sorteo.

**Cómo se garantiza "1 sola vez":**
- En el celular: la jugada queda bloqueada desde que se coloca la caja. Si cierra o recarga la página a mitad de juego, la jugada se cierra y no se puede reiniciar. Al volver a abrir el link solo ve su resultado.
- En el servidor: 1 jugada por dispositivo y 1 participación válida por celular en toda la campaña (las repetidas quedan como `DUPLICADA` y no entran al sorteo; vale la primera que se registró).

**Sorteo ponderado:** una participante con 400 puntos tiene 400 boletos y otra con 100 tiene 100, por lo que la primera tiene 4 veces más probabilidad. Todas las que registran sus datos participan (mínimo 1 boleto), pero el puntaje inclina el resultado.

## Modo prueba (solo uso interno)

Permite jugar infinitas veces desde tu celular para probar.

- **Activar:** abrir una vez `https://cvaldovinos25.github.io/SORTEO-KOTEX/?tester=CLAVE`, donde `CLAVE` es el valor de `TESTER_KEY` en `CONFIG` (por defecto `salco-prueba-2026`; **cámbiala antes de publicar**). Aparece la etiqueta amarilla "MODO PRUEBA" y el celular lo recuerda.
- **Salir:** abrir `?tester=off`.
- Cada carga cuenta como un dispositivo nuevo. Las jugadas se guardan en la planilla con el dispositivo `TEST-xxxx`, para filtrarlas o borrarlas antes del lanzamiento. Si se registra siempre el mismo celular, desde la segunda vez queda `DUPLICADA`.
- No imprimas el QR del modo prueba para la góndola.

## Cómo se conecta con Excel

GitHub Pages solo sirve archivos estáticos, así que no puede escribir un `.xlsx` directamente. La solución es una **planilla de Google Sheets** que recibe cada jugada en tiempo real por una URL de Apps Script. Desde ahí se descarga como Excel: *Archivo > Descargar > Microsoft Excel (.xlsx)*.

Columnas de la hoja `Jugadas`: Código, Fecha y hora (Chile), Timestamp UTC, Hora del celular, Tienda, Góndola, Dispositivo, Nombre, Teléfono, Opt-in, Estado, Registrado en, User agent, Campaña y **Puntaje** (columna O).

Estados: `SIN_REGISTRO` (jugó pero no dejó datos), `VALIDA` (participa en el sorteo), `DUPLICADA` (celular ya registrado; no entra al sorteo).

## Puesta en marcha

### 1. Planilla y backend
1. Crea una planilla en Google Sheets (por ejemplo "Kotex RA Sorteo").
2. *Extensiones > Apps Script*. Pega el contenido de `apps-script/Code.gs`.
3. En *Configuración del proyecto* activa "Mostrar archivo de manifiesto" y reemplaza `appsscript.json` por el de este repo.
4. Arriba elige la función `setup` y toca **Ejecutar**. Google pedirá permisos: *Revisar permisos* > tu cuenta > si aparece "Google no verificó esta aplicación", toca *Configuración avanzada* > *Ir a (nombre del proyecto) (no seguro)* > *Permitir*. Es normal: el script es tuyo. Crea las hojas `Jugadas`, `Sorteo` y `Config`.
5. *Implementar > Nueva implementación*. En el engranaje elige tipo **Aplicación web**, y configura: *Ejecutar como* = **Yo**, *Quién tiene acceso* = **Cualquier persona**. Toca *Implementar* y copia la **URL de la aplicación web** (termina en `/exec`). Esa es la URL que va en `ENDPOINT`.

> Cada vez que cambies `Code.gs` debes crear una **nueva versión** de la implementación para que se apliquen los cambios. Esta versión del juego **no requiere cambios en `Code.gs`**: el puntaje máximo sigue siendo 500 (`SCORE_MAX`).

6. **Verifica la conexión** de dos formas:
   - En la planilla, menú **Kotex > Probar conexión**: escribe una fila de prueba, la borra y avisa.
   - Abre la URL `/exec` en el navegador: debe responder `{"ok":true,"service":"kotex-ra","sheet":"Jugadas","rows":0}`.

### 2. Frontend en GitHub Pages
1. Sube `index.html` al repo (reemplaza el anterior).
2. En el bloque `CONFIG` de `index.html` están las dos URL que hay que mantener:
   - `ENDPOINT:"https://script.google.com/macros/s/XXXX/exec",` (conexión con Google Sheets). Con `ENDPOINT` vacío corre en **modo demo**: guarda solo en el celular, sirve para probar el juego pero no escribe en la planilla, y se ve una etiqueta "MODO DEMO".
   - `BASES_URL:"https://.../bases.pdf",` (enlace a las bases del concurso, que se abre desde "Bases del concurso").
   > Si reemplazas `index.html` por una versión nueva, revisa que estas dos URL sigan puestas.
3. *Settings > Pages > Deploy from a branch > main / root*.
4. Prueba abriendo `https://cvaldovinos25.github.io/SORTEO-KOTEX/?t=SALCO-001&g=G12&debug=1` desde un celular. El aviso `DEBUG` indica si está conectada a Google Sheets o en modo demo. Quita `&debug=1` en los stickers definitivos.
5. En modo demo, para volver a probar abre la página con `?reset=1`. Esa opción se desactiva sola cuando `ENDPOINT` tiene la URL real.
6. Después de subir cambios, GitHub Pages tarda 1 o 2 minutos. Para ver la versión nueva y no la guardada en el celular, abre el link en una ventana privada.

### 3. Stickers NFC y QR
- Usa etiquetas **NTAG213/215** y una app como *NFC Tools* para grabar una URL (registro NDEF) **distinta por tienda y góndola**:
  `https://cvaldovinos25.github.io/SORTEO-KOTEX/?t=SALCO-001&g=G12`
  `t` = código de tienda, `g` = código de góndola. Quedan guardados en cada jugada.
- Imprime un **QR con la misma URL** junto al sticker como respaldo para celulares sin NFC.
- iPhone (XS o superior) lee NFC sin app; en Android debe estar activado.

## Si no se guarda en la planilla (diagnóstico)
Abre la página con `?debug=1`. Mostrará algo como `GET ok (3 jugadas) | POST ok`. Si no:

| Mensaje | Causa y solución |
|---|---|
| `ENDPOINT vacío` | No pegaste la URL en `index.html` o no subiste el archivo nuevo a GitHub |
| `bloqueado` | La implementación no es pública o la URL está mal. En Apps Script: Implementar > Administrar implementaciones > lápiz > Quién tiene acceso = **Cualquier persona** (no "Cualquier persona con cuenta de Google"), Versión = **Nueva versión**. La URL debe terminar en `/exec` (no `/dev`) |
| `respuesta no válida` | La URL no es la de la aplicación web (abre la URL en el navegador: debe mostrar `{"ok":true,"service":"kotex-ra",...}`) |
| `servidor: ... no_sheet` o `no está vinculado` | El script no está dentro de la planilla. Pega el ID de la planilla (el texto largo de su URL, entre `/d/` y `/edit`) en `SPREADSHEET_ID` al inicio de `Code.gs` y crea una nueva versión. Las hojas se crean solas |
| `servidor: bad_score` | `SCORE_MAX` en Code.gs no coincide con el puntaje máximo del juego (500) |

## Cómo se hace el sorteo
1. (Opcional) En la hoja `Config`, celdas `B1` y `B2`, indica desde y hasta qué fecha y hora se consideran jugadas.
2. Menú **Kotex > Sortear ganadoras (por puntaje)**, ingresa cuántas ganadoras y suplentes.
3. El resultado queda en la hoja `Sorteo` con fecha del sorteo, puesto, código, nombre, celular, tienda, hora de la jugada y puntaje. Solo participan filas con estado `VALIDA`.
4. Antes del sorteo, filtra o borra las filas de prueba (dispositivo `TEST-...`).

## Ajustes (bloque `CONFIG` de index.html)
| Parámetro | Efecto |
|---|---|
| `ENDPOINT` | URL de Apps Script (termina en `/exec`) |
| `BASES_URL` | Enlace a las bases del concurso |
| `TESTER_KEY` | Clave del modo prueba |
| `MODE` | `'catch'` (atrapar productos, por defecto) o `'ring'` (juego anterior del aro de luz) |
| `ROUNDS`, `PER_ROUND` | En el modo `catch` solo definen el puntaje máximo: la suma de `PER_ROUND` es el máximo del juego (500) y `ROUNDS` debe ser igual a `CATCH.COUNT`. Si cambias el máximo, actualiza `SCORE_MAX` en `Code.gs` |
| `CAMERA_FOV` | Campo de visión vertical de la cámara virtual (grados, 60 por defecto). Si la caja se "desliza" respecto del fondo al girar el celular, ajústalo entre 55 y 70 |
| `MODEL_URL` | Opcional: URL de un modelo `.glb` real de la caja. Reemplaza al modelo incluido |
| `COLORS`, `DRAW_TEXT` | Paleta y texto final |

### Dificultad del modo atrapar (objeto `CATCH`, justo antes de la función `startCatchRound`)
| Parámetro | Efecto |
|---|---|
| `COUNT` | Cantidad de productos a atrapar (7). Debe coincidir con `ROUNDS` |
| `BEST` | Segundos o menos para obtener el puntaje máximo (30) |
| `WORST` | Segundos en que se acaba el tiempo (120) |
| `MIN` | Puntos al terminar justo sobre el límite (50) |
| `RAD` | Tolerancia de puntería en radianes (0,28, unos 16°). Menor = más difícil |
| `HINT_AFTER` | Segundos sin atrapar nada para que aparezca la flecha guía (6) |
| `DRIFT` | Velocidad con que se desplazan los productos (0,12) |

Los 30 y 120 segundos son una primera propuesta: si casi nadie baja de un minuto, sube `BEST`.

## Animación, sonido y modelos 3D
- **Caja:** se cierra al empezar. Al colocarla queda fija en el espacio (anclada con el giroscopio). Al terminar tiembla, se abre la tapa con destello, ampolletas encendidas y confeti. Respeta la opción "reducir movimiento" del celular.
- **Productos:** son modelos 3D hechos por código (CD, cámara, dos empaques de toallas, sticker, ampolleta) con iluminación de estudio. Al atrapar uno hay chispas, onda de impacto, vibración y un contador flotante.
- **Sonido:** se genera por código (sin archivos). Hay botón de silencio arriba a la izquierda que recuerda la elección. En iPhone funciona aunque el interruptor lateral esté en silencio. Se activa al tocar "¡A jugar!" (los celulares no permiten audio antes del primer toque).
- **Caja 3D:** se arma por código a partir del diseño real, con texturas rectificadas desde la foto de referencia (tapa con espejo, frontal con los dibujos, cajón con la cámara y el piso azul). Es una aproximación fiel, no un escaneo. Para máxima fidelidad pide a un artista 3D un `.glb` liviano (menos de 3 MB) y pon su URL en `MODEL_URL`.
- **Logos:** el logo "Kotex ¡SORTEO!" y el de Salcobrand van incrustados en `index.html`, así que no hace falta subir imágenes aparte.

## Limitaciones a tener en cuenta
- **El puntaje lo calcula el celular.** El servidor valida que esté entre 0 y 500, pero una persona técnica podría enviar un puntaje inflado. Para premios de alto valor conviene sumar verificación (reCAPTCHA, o registrar los instantes de cada toque y validarlos en el servidor) o un backend propio.
- La URL del backend es pública. Las cuotas de Google Apps Script alcanzan para un piloto; para campañas masivas conviene un backend dedicado.
- Quien borra los datos del navegador o usa otro celular obtiene un dispositivo nuevo; el control definitivo es 1 participación válida por número de celular.
- **Cómo es la RA:** la caja y los productos se anclan al espacio usando el giroscopio (3 grados de libertad: giro, no distancia). Si la clienta camina hacia la góndola, la caja no se agranda ni se achica, y con el paso de los minutos el sensor puede "derivar" un poco (en una jugada de dos minutos o menos casi no se nota). Para anclar a una superficie o afiche con seguimiento real se puede migrar a MindAR o 8th Wall sin cambiar el juego ni el backend.
- El juego necesita un celular con cámara y, idealmente, giroscopio. En equipos muy antiguos el 3D puede ir lento.
- Las bases legales deben reflejar el sorteo ponderado por puntaje, la regla de una sola jugada y que el puntaje depende del tiempo, y el consentimiento de datos personales debe revisarse con el área legal del cliente.
