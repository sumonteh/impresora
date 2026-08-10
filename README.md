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
8. Elige `Copias` entre 1 y 5.
9. Toca `Imprimir`.
10. Espera a que termine. Si imprime mas de lo elegido, toca `Copiar diagnóstico` y pega aqui el resultado completo.

En la version `v32-final-copies`, la impresion normal queda habilitada usando el flujo validado en la M02: cada copia se empaqueta como un raster corto, se envia en una sola llamada GATT con respuesta, y la siguiente copia espera el aviso `READY` de la impresora. El selector de copias esta limitado a 5 por envio para proteger el rollo.

El log del test seguro deberia mostrar:

- `APP VERSION: v32-final-copies`;
- `WRITE CHANNEL SELECTED: ...`;
- `WRITE MODE SELECTED: withResponse`;
- `COPY RUN: ... copia(s)`;
- `BOTTOM GAP: +24 blank lines`;
- `ready-credit`, despues de que la impresora avise que esta lista;
- `SEGMENT 1/1`;
- `BLE write 1/1`;
- ningun `BLE write` despues de `END JOB`.

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
