# impresora

Soporte experimental para imprimir directo en una Phomemo M02 desde una pagina web estatica.

## Probar en iPhone con Bluefy

1. Abre la pagina publicada por HTTPS en Bluefy.
2. Enciende la Phomemo M02 y dejala cerca del iPhone.
3. Toca `Conectar impresora`.
4. Elige la M02 en la lista Bluetooth.
5. Toca `TEST M02 — UNA IMPRESIÓN` una sola vez.
6. Espera a que el panel diga `END JOB`.
7. Si imprime mas de una vez, toca `Copiar diagnóstico` y pega aqui el resultado completo.

En la version `v15-bandas-seguras`, el log deberia mostrar:

- `WRITE MODE: withResponse preferred when available`;
- writes `withResponse` si Bluefy lo permite;
- raster dividido en varios bloques pequenos de hasta 24 lineas;
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
