# impresora

Soporte experimental para imprimir directo en una Phomemo M02 desde una pagina web estatica.

## Probar en iPhone con Bluefy

1. Abre la pagina publicada por HTTPS en Bluefy.
2. Enciende la Phomemo M02 y dejala cerca del iPhone.
3. Toca `Conectar impresora`.
4. Elige la M02 en la lista Bluetooth.
5. Escribe exactamente el texto que quieres probar.
6. Para prueba segura, toca `TEST — TEXTO 1 VEZ` una sola vez.
7. Espera a que el panel diga `END JOB`.
8. Si imprime mas de una vez, toca `Copiar diagnóstico` y pega aqui el resultado completo.

En la version `v24-native-text-probe`, la impresion normal queda bloqueada otra vez porque la M02 imprimio 7 copias fisicas aunque la pagina envio un solo trabajo. El test seguro ahora manda texto nativo en un paquete corto, sin raster, sin bandas y sin reutilizar el valor `7` como permiso para seguir enviando.

El log del test seguro deberia mostrar:

- `WRITE MODE: withoutResponse preferred for Bluefy/M02`;
- `NATIVE TEXT TEST: ... chars / ... bytes`;
- `BLE raw write 1/1`, si el texto cabe en una escritura;
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
