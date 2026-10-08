# Fase 1 — Arquitectura técnicamente viable

Revisado: 7 de octubre de 2026.

## Decisión

La arquitectura más viable para llegar a **abrir un enlace y jugar sin que cada visitante aporte una ROM** es partir del port de navegador existente `mm-cleanroom`: motor 2 Ship 2 Harkinian compilado a WebAssembly con Emscripten y renderizado con WebGL2. Tiene una build web real, controles de teclado/gamepad/táctiles y guardado del navegador. No es un emulador N64 genérico.

Hay una condición importante: el proyecto regenera recursos visuales y de audio; su autor señala que el resultado difiere de los datos comerciales. La página dice que la build arranca y la prueba publicada llegó al juego en Clock Town, pero está marcada como prueba ligera, no como una versión completamente verificada. Su código declara licencia CC0 para 2Ship, MIT para libultraship y OFL para fuentes; esa licencia del código **no** resuelve por sí misma los derechos de marcas, hechos, assets generados o distribución del juego. La afirmación clean-room es del autor y no constituye una revisión legal independiente.

Para este experimento recomiendo esa ruta como base de investigación, conservando la ROM exclusivamente local. Antes de publicar, habría que revisar el origen/licencia del port y todos los datos generados. Si se exige fidelidad visual y sonora al cartucho, la alternativa técnica es un emulador N64 WebAssembly, pero para que un enlace público lo ejecute sin pasos manuales habría que entregar los datos del juego al visitante. No se debe subir la ROM ni assets extraídos protegidos; sin una vía de distribución autorizada, esas condiciones no se pueden satisfacer a la vez.

## Comparación

| Ruta | Qué ejecuta | ROM del visitante | Calidad/fidelidad | Viabilidad aquí |
|---|---|---:|---|---|
| Emulador N64 en WASM (p. ej., Mupen64Plus-Next/EmulatorJS o N64Wasm) | ROM retail | Sí, o se redistribuye el juego | La más fiel al cartucho; compatibilidad/performance depende del core | Buena para una prueba local con ROM propia; conflicto con “abrir enlace y jugar” si no se puede distribuir el juego |
| 2Ship2Harkinian + `mm-cleanroom` | Port decompilado, compilado WASM/WebGL2 | No para jugar a la build publicada | Juego jugable, recursos regenerados, no idéntico a retail | Mejor punto de partida web existente; rendimiento y juego completo aún requieren validación |
| Compilar el 2Ship nativo directo a WASM | Port decompilado | Normalmente se requieren datos de juego | Potencialmente alta | No es un `emcc` automático: requiere adaptar plataforma, render, audio, entrada, filesystem y guardado. `mm-cleanroom` ya aporta una adaptación web concreta |
| Emulador escrito desde cero o `n64js` | ROM retail | Sí | Depende de completar core/render/plugins | No recomendado: rehace trabajo complejo ya disponible |

`mm-cleanroom` requiere ROM propia **estadounidense para reconstruir** el proyecto según su README. La copia disponible en este workspace está etiquetada como Europa multiidioma (`.n64`), así que no doy por hecho que sirva para ese pipeline. La página publicada sí se puede jugar sin aportar ROM. No se debe cambiar de región ni descargar otra ROM para esta fase.

## WASM, Emscripten y gráficos

- WASM es el formato de ejecución del código compilado; Emscripten adapta C/C++ y genera el glue JavaScript que conecta el motor con el navegador.
- Emscripten soporta WebGL mediante un subconjunto compatible de OpenGL ES 2/3. WebGL2 es la elección inicial: lo usa la build existente y ofrece una ruta de compatibilidad más madura que crear un backend WebGPU nuevo.
- WebGPU no acelera automáticamente WASM; solo tendría sentido si el renderer se adaptase y mediciones reales lo justificaran.
- Audio debe inicializarse tras interacción del usuario por las restricciones de autoplay del navegador. La build pública indica que el sonido empieza después del primer toque/tecla.
- No daría por obligatorio poner todo en Worker ni activar pthreads: puede exigir `SharedArrayBuffer` y cabeceras COOP/COEP. Primero mediría el port monohilo; añadir hilos aumenta los requisitos de despliegue y compatibilidad.

## Tamaños medidos y carga

Medí las respuestas públicas actuales de la build de referencia el 7 de octubre de 2026, leyendo bytes sin guardarlos en el workspace:

| Archivo | Tamaño servido |
|---|---:|
| `2ship.wasm` | 22.941.752 bytes (21,88 MiB) |
| `mm.o2r` | 37.953.594 bytes (36,20 MiB) |
| `2ship.js` | 230.512 bytes (0,22 MiB) |
| **Subtotal de estos tres** | **61.125.858 bytes (58,30 MiB)** |

A esto se suma HTML, CSS, scripts de controles y posiblemente otros recursos. Por tanto, la primera visita de esa arquitectura no es instantánea: la transferencia conocida es de al menos ~58,3 MiB más recursos menores. La página simple es rápida de mostrar, pero el motor/paquete de datos debe descargarse antes de que el runtime pueda arrancar. No encontré evidencia de que el juego pueda iniciar antes de cargar `mm.o2r`; el paquete principal parece una unidad de datos, así que el streaming de red por sí solo no prueba carga progresiva por zonas.

La ROM local mide **33.554.432 bytes (32 MiB)** sin comprimir. Ese número no equivale al paquete web ni a los assets exportados. La build clean-room actual ocupa al menos esos 58,3 MiB de payload conocido. Para una futura build con recursos retail no se puede dar un peso serio hasta generar e inspeccionar el paquete; no se puede afirmar que sea más pequeño.

## Caché y partidas

- **HTTP cache:** WASM y paquetes con nombres versionados pueden servirse con `Cache-Control: public, max-age=31536000, immutable`; HTML debe revalidarse para poder actualizar el cargador. Brotli/Gzip deben comprobarse en los encabezados del hosting, no suponerse. Los datos ya comprimidos quizá ahorren poco.
- **Service Worker + Cache API:** cachear los archivos versionados de la app shell y runtime; hacer precache del paquete de juego si el espacio y la cuota son aceptables. Actualizar por versión, sin borrar la versión en uso hasta que la nueva se valide. El Service Worker requiere HTTPS (salvo localhost).
- **Segunda visita:** puede evitar la descarga repetida si el navegador conserva la caché; almacenamiento del navegador puede ser eliminado o desalojado. No elimina la transferencia inicial.
- **Saves:** usar IndexedDB para partida normal y configuración; confirmar guardado y manejar cuota/errores. `mm-cleanroom` declara que persiste saves en IndexedDB. Save states pueden ser grandes e incompatibles entre versiones; no los prometería hasta verificar que el runtime expone una API estable.

## Rendimiento y compatibilidad

La evidencia disponible es limitada: el sitio clean-room declara prueba headless en Edge hasta Clock Town y su página pública etiqueta la build como ligeramente probada. No hay benchmarks publicados de FPS, uso de CPU/GPU, memoria, tiempo en teléfonos ni compatibilidad comparativa en Chrome/Firefox. WASM + WebGL2 puede ser fluido en equipos modernos, pero no afirmaría 60 FPS ni soporte móvil general sin medir este título y estos paquetes en dispositivos reales.

Plan de medición para una fase posterior: arranque frío/caliente, bytes transferidos, tiempo hasta título y juego controlable, FPS sostenidos, memoria, audio, gamepad, persistencia del save y pruebas en Chrome/Edge/Firefox desktop y al menos un teléfono representativo.

## Arquitectura objetivo si se continúa

```text
majora-mask-web/
  index.html
  src/                         interfaz y controles web
  public/
    service-worker.js
    wasm/                      JS glue + WASM
    data/                      paquete de datos de la build
  private/                     archivos locales, no versionar ni desplegar
    rom/
    extracted/
  docs/
    brief-original.txt
    fase-1-arquitectura.md
```

Frontend estático servido por HTTPS/CDN; canvas con WebGL2; WASM/Emscripten; teclado y Gamepad API; controles táctiles opcionales; audio después de gesto; IndexedDB para saves; Cache API/Service Worker para runtime y datos versionados. El contenido privado queda fuera de `public/` y está cubierto por el `.gitignore` de esta carpeta.

## Resultado de Fase 1

La ruta de navegador existe y no hay que inventar un emulador vacío. Para cumplir mejor “enlace → jugar” sin distribuir la ROM, la opción viable es estudiar/adaptar la build WASM/WebGL2 de `mm-cleanroom`, aceptando recursos no retail y verificando derechos antes de cualquier publicación. La ruta de emulación con ROM intacta sirve para jugar localmente con tu copia, pero no resuelve el enlace público sin trasladar al visitante el archivo del juego.

No se inicia instalación de toolchain ni compilación en esta fase. Para la siguiente fase, antes de tocar el runtime, habría que escoger entre fidelidad clean-room y emulación de tu ROM local; esa decisión cambia toda la base técnica.

## Fuentes

- [Build y controles de Majora's Mask en Decomp Games](https://decompgames.com/games/majoras-mask/)
- [README de `mm-cleanroom`](https://github.com/andrewnakas/mm-cleanroom/blob/main/README.md) — build Emscripten/WASM/WebGL2, archivos requeridos para reconstrucción y recursos regenerados.
- [Estado y pruebas declaradas de `mm-cleanroom`](https://github.com/andrewnakas/mm-cleanroom/blob/main/STATUS.md)
- [2 Ship 2 Harkinian y guía de ROM compatible](https://github.com/2ship2harkinian/2ship2harkinian)
- [Emscripten: soporte OpenGL/WebGL](https://emscripten.org/docs/porting/multimedia_and_graphics/OpenGL-support.html)
- [Emscripten: pthreads y requisitos COOP/COEP](https://emscripten.org/docs/porting/pthreads.html)
- [EmulatorJS: núcleos soportados](https://emulatorjs.org/docs4devs/cores/) y [Mupen64Plus-Next: formatos, saves y estados](https://docs.libretro.com/library/mupen64plus/)
- [MDN: Cache API y posible desalojo de caché](https://developer.mozilla.org/en-US/docs/Web/API/Cache)
