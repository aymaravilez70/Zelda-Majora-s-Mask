# The Legend of Zelda: Majora's Mask — WebAssembly Port

Versión web ejecutable directamente desde el navegador de **The Legend of Zelda: Majora's Mask**, basada en el motor decompilado **2 Ship 2 Harkinian (2S2H)** y la arquitectura clean-room de **mm-cleanroom**, compilada a **WebAssembly / WebGL2** con **Emscripten**.

No requiere emulador N64 ni descarga de ROM por parte del visitante.

---

## 🎮 Características

* **Ejecución directa en el navegador:** Renderizado acelerado por hardware con WebGL2.
* **Sin streaming:** Todo el código y audio se ejecutan localmente en el cliente mediante WebAssembly y AudioContext.
* **Guardado local persistente:** Partidas y ajustes guardados en el navegador a través de IndexedDB (`IDBFS` en `/saves`).
* **Controles versátiles:**
  * **Teclado:**
    * **Stick analógico:** `W`, `A`, `S`, `D`
    * **Botón A:** `X`
    * **Botón B:** `C`
    * **Botón Z:** `Z`
    * **Botón Start:** `Espacio`
    * **Botones C:** Flechas direccionales (`▲`, `◀`, `▶`, `▼`)
    * **Botón L:** `E`
    * **Botón R:** `R`
    * **D-Pad:** `T`, `F`, `G`, `H`
    * **Menú del port:** `Esc`
  * **Mandos / Gamepads:** Soporte nativo para mandos USB/Bluetooth mediante la Gamepad API de HTML5.
  * **Controles táctiles:** Stick analógico virtual y botones en pantalla para dispositivos móviles y tablets (añadir `?touch=1` para forzar en cualquier pantalla).
* **Assets Clean-Room:** Todos los recursos gráficos (13.542 texturas) y de audio (682 muestras VADPCM) han sido regenerados procedimentalmente o tipografiados con fuentes abiertas (OFL) para cumplir con el estándar clean-room (0% coincidencia de tramos de código propietario con datos minoristas).

---

## 🚀 Despliegue en Vercel

Este repositorio está preparado para desplegarse automáticamente en [Vercel](https://vercel.com):

1. Importa este repositorio desde tu panel de Vercel.
2. Selecciona **Other** como Framework Preset (la configuración por defecto sirve `index.html` en la raíz).
3. Haz clic en **Deploy**.
4. ¡Listo! Tu enlace de Vercel estará disponible y jugable en segundos.

El archivo `vercel.json` incluido optimiza el almacenamiento en caché de los archivos WebAssembly (`2ship.wasm`) y paquetes de recursos (`*.o2r`), así como los tipos MIME adecuados.

---

## 📜 Licencias y Atribución

* **2 Ship 2 Harkinian:** Licencia Creative Commons Zero v1.0 Universal (CC0).
* **libultraship:** Licencia MIT (Copyright (c) 2022 kenix3).
* **Fuentes tipográficas:**
  * Marcellus (SIL Open Font License)
  * Montserrat (SIL Open Font License)
  * Noto Sans JP (SIL Open Font License)

*Aviso: Este es un proyecto de investigación y desarrollo sin fines de lucro realizado por fans. No está afiliado ni respaldado por Nintendo. The Legend of Zelda y Majora's Mask son marcas registradas de Nintendo.*
