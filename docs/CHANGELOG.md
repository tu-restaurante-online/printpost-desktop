# Changelog

## v1.2.0 — 2026-11-03

### Agregado
- Multi-conexión: Red TCP/IP, USB con driver Windows (`electron-pos-printer`, sin Zadig) y USB con LibUSB. Abstracción en `printer-adapter.js`.
- Config persistente en `userData/config.json` vía IPC (`config:get`, `config:set`, `printers:list`, `usb:list`).
- Modal de configuración (tipo de conexión, impresora Windows, dispositivo USB, cajón) + badge de modo activo.
- Selección de dispositivo LibUSB por `vid:pid`.
- Campo `employee` opcional en las impresiones.
- `cierre_caja`: imprime Esperado y Real cuando llega `real_amount`.
- `POST /drawer`: abre el cajón sin imprimir.

### Mejorado
- Cajón de dinero se abre solo si la petición trae `open_drawer: true` (antes se abría en cada impresión) — RestaurantesWeb ISSUE-114.
- `forge.config.js`: claves inválidas eliminadas y archivos de desarrollo/docs excluidos del paquete.

### Compatibilidad
- `?ip=...&port=...` en la query sigue funcionando y sobrescribe la config de disco solo para esa petición.

## v1.1.0

### Agregado
- Template `recibo_tienda` (farmacia/retail).
- Template `inventario`.
- Template `cierre_caja` con lista de ventas y arqueo de caja.

### Corregido
- Impresión secuencial (ticket + comanda).
- Generación del ejecutable.

### Mejorado
- UI migrada de Tailwind a Bootstrap.
- Refactor de `server.js`.
