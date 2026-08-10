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
8. Para prueba segura, toca `TEST — RASTER 1 VEZ` una sola vez.
9. Espera a que el panel diga `END JOB`.
10. Si imprime mas de una vez, toca `Copiar diagnóstico` y pega aqui el resultado completo.

En la version `v28-with-response-probe`, la impresion normal queda bloqueada. `FF82` no imprimio nada, asi que el test vuelve a `FF02` pero usando escritura BLE con respuesta y una pausa mas larga entre paquetes.

El log del test seguro deberia mostrar:

- `APP VERSION: v28-with-response-probe`;
- `WRITE CHANNEL SELECTED: ...`;
- `WRITE MODE SELECTED: withResponse`;
- `SINGLE BAND RASTER TEST: ... chars`;
- `SEGMENT 1/1`;
- hasta 7 `BLE write`;
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
