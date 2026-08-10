# impresora

Soporte experimental para imprimir directo en una Phomemo M02 desde una pagina web estatica.

## Probar en iPhone con Bluefy

1. Abre la pagina publicada por HTTPS en Bluefy.
2. Enciende la Phomemo M02 y dejala cerca del iPhone.
3. Toca `Conectar impresora`.
4. Elige la M02 en la lista Bluetooth.
5. Escribe exactamente el texto que quieres probar.
6. Deja el canal `FF00 / FF02`.
7. Deja el modo `Con respuesta BLE`.
8. Para prueba segura, toca `TEST — 1 ENVÍO` una sola vez.
9. Espera a que el panel diga `END JOB`.
10. Si imprime mas de una vez, toca `Copiar diagnóstico` y pega aqui el resultado completo.

En la version `v31-ready-credit-gap`, la impresion normal queda bloqueada. La M02 ya imprime una sola vez cuando el raster corto se envia en una sola llamada GATT. Esta version aumenta el margen inferior y usa el aviso `READY` de la impresora para habilitar la siguiente prueba sin agotar los creditos iniciales.

El log del test seguro deberia mostrar:

- `APP VERSION: v31-ready-credit-gap`;
- `WRITE CHANNEL SELECTED: ...`;
- `WRITE MODE SELECTED: withResponse`;
- `SINGLE BAND RASTER TEST: ... chars`;
- `BOTTOM GAP: +24 blank lines`;
- `ready-credit`, despues de que la impresora avise que esta lista;
- `SEGMENT 1/1`;
- `BLE write 1/1`;
- ningun `BLE write` despues de `END JOB`.

La impresion normal queda bloqueada temporalmente hasta resolver la repeticion fisica de 7 copias.

## Diagnostico M02

El panel `Diagnóstico M02` muestra:

- estado BLE;
- nombre del dispositivo;
- servicios encontrados;
- characteristics encontradas;
- numero de trabajo;
- pulsaciones de imprimir;
- BLE writes;
- bytes enviados;
- chunk size;
- pausa entre chunks;
- duracion;
- log con bytes del payload.

## Protocolo

Ver `docs/M02_PROTOCOL.md`.
