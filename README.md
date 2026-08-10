# impresora

Soporte experimental para imprimir directo en una Phomemo M02 desde una pagina web estatica.

## Probar en iPhone con Bluefy

1. Abre la pagina publicada por HTTPS en Bluefy.
2. Enciende la Phomemo M02 y dejala cerca del iPhone.
3. Toca `Conectar impresora`.
4. Elige la M02 en la lista Bluetooth.
5. Escribe exactamente el texto que quieres probar.
6. Para prueba segura, toca `TEST — SOLO TEXTO` una sola vez.
7. Espera a que el panel diga `END JOB`.
8. Si imprime mas de una vez, toca `Copiar diagnóstico` y pega aqui el resultado completo.

En la version `v22-text-only-test`, el test seguro imprime solo el texto escrito en el cuadro, sin marcos, barras ni contenido extra. La impresion normal sigue disponible, pero limitada temporalmente a 120 lineas para proteger el rollo.

El log del test seguro deberia mostrar:

- `WRITE MODE: withoutResponse preferred for Bluefy/M02`;
- `FLOW CONTROL: FF03 credit gating enabled`;
- `FLOW CREDIT +... from FF03`;
- writes `withoutResponse` de hasta 180 bytes;
- `TEXT TEST: ... chars`;
- segmentos de hasta 24 lineas;
- `SEGMENT .../...`;
- `PRINTER READY from FF01 1A0F0C`;
- `FLOW CREDIT +7 from FF01 1A0F0C ready-refill`;
- ningun `BLE write` despues de `END JOB`.

El log de impresion normal puede mostrar:

- `SAFETY CROP: ... lines -> 120 lines`, si el contenido era mas alto;
- segmentos de 24 lineas;
- recargas por `FF01 1A0F0C ready-refill`.

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
