# impresora

Soporte experimental para imprimir directo en una Phomemo M02 desde una pagina web estatica.

## Probar en iPhone con Bluefy

1. Abre la pagina publicada por HTTPS en Bluefy.
2. Enciende la Phomemo M02 y dejala cerca del iPhone.
3. Toca `Conectar impresora`.
4. Elige la M02 en la lista Bluetooth.
5. Escribe exactamente el texto que quieres probar.
6. Para prueba segura, toca `TEST — RASTER 1 VEZ` una sola vez.
7. Espera a que el panel diga `END JOB`.
8. Si imprime mas de una vez, toca `Copiar diagnóstico` y pega aqui el resultado completo.

En la version `v26-single-band-raster`, la impresion normal queda bloqueada. La M02 ignoro el texto nativo, asi que el test vuelve a raster, pero limitado a una sola banda de 24 lineas como maximo para evitar el salto a una segunda banda.

El log del test seguro deberia mostrar:

- `WRITE MODE: withoutResponse preferred for Bluefy/M02`;
- `APP VERSION: v26-single-band-raster`;
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
