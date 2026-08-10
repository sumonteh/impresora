# impresora

Soporte experimental para imprimir directo en una Phomemo M02 desde una pagina web estatica.

## Probar en iPhone con Bluefy

1. Abre la pagina publicada por HTTPS en Bluefy.
2. Enciende la Phomemo M02 y dejala cerca del iPhone.
3. Toca `Conectar impresora`.
4. Elige la M02 en la lista Bluetooth.
5. Escribe exactamente el texto que quieres probar.
6. Deja el canal `FF00 / FF02` para la primera prueba, o cambia a `FF80 / FF82` para probar el canal alternativo.
7. Para prueba segura, toca `TEST — RASTER 1 VEZ` una sola vez.
8. Espera a que el panel diga `END JOB`.
9. Si imprime mas de una vez, toca `Copiar diagnóstico` y pega aqui el resultado completo.

En la version `v27-write-channel-probe`, la impresion normal queda bloqueada. La M02 ignoro el texto nativo, asi que el test usa raster de una sola banda. Ahora tambien permite elegir el canal BLE de escritura para comparar `FF02` contra canales alternativos como `FF82`.

El log del test seguro deberia mostrar:

- `WRITE MODE: withoutResponse preferred for Bluefy/M02`;
- `APP VERSION: v27-write-channel-probe`;
- `WRITE CHANNEL SELECTED: ...`;
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
