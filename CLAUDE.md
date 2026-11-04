# CLAUDE.md — print-post-electron

Contexto del proyecto para asistente de IA.

---

## Qué es este proyecto

App Electron que actúa como **servicio local de impresión térmica**. Corre un servidor Express en puerto `3010` y expone endpoints HTTP. Sistemas externos (ej: PanaderiApp, restaurantes) hacen `POST /print` para imprimir tickets, comandas, recibos, etc.

- **No es una app de usuario final** — vive en la bandeja del sistema.
- La UI (`pages/index.html`) es solo para pruebas y configuración.
- Clientes externos consumen la API HTTP.

---

## Estructura de archivos

```
main.js              — Proceso principal Electron. Ventana, tray, server, handlers IPC de config.
preload.js           — Bridge IPC: window.api (getConfig/setConfig/getPrinters/getUsbDevices).
server.js            — Servidor Express + templates de impresión ESC/POS.
printer-adapter.js   — Adapters de conexión (Network, LibUsb, WindowsDriver).
pages/index.html     — UI: botones de prueba + modal de configuración.
forge.config.js      — Config de empaquetado con electron-forge.
assets/              — Iconos e imágenes.
docs/                — Estado (PROJECT), backlog (ROADMAP), CHANGELOG, daily, sessions/. Ver docs/README.md.
TODO.md              — Tareas de la sesión activa.
README.md            — Referencia de la API HTTP para clientes.
```

---

## Stack

| Capa | Tecnología |
|------|-----------|
| Desktop | Electron 33 |
| Servidor HTTP | Express 4 |
| UI | Bootstrap 5.3.2, JavaScript vanilla |
| Impresión ESC/POS | escpos@3.0.0-alpha.6 |
| Driver red | escpos-network@3.0.0-alpha.5 |
| Driver USB | escpos-usb@3.0.0-alpha.4 |
| Build | electron-forge 7.6.0 |

---

## Flujo actual de impresión

```
POST http://127.0.0.1:3010/print?ip=<IP>&port=<PUERTO>
  Body: { template: "recibo", ...data }
    ↓
server.js → switch(template) → printRecipe/printTicket/...
    ↓
getDevice(req.query) → escpos.USB() | escpos.Network(ip, port)
    ↓
printWithDevice(device, fn) → device.open() → printer commands → cut() → close()
```

**Config**: persistida en disco (`userData/config.json`). Query params `?ip=&port=` la sobrescriben solo para esa petición.

---

## Templates de impresión disponibles

| Template | Función | Descripción |
|----------|---------|-------------|
| `recibo` | `printRecipe()` | Recibo de restaurante |
| `ticket` | `printTicket()` | Ticket de orden |
| `comanda` | `printComanda()` | Comanda de cocina |
| `ticket_comanda` | combo | Ticket + comanda (dos impresiones) |
| `cierre_caja` | `printCloseBox()` | Cierre de caja con desglose |
| `inventario` | `printInventory()` | Lista de stock |
| `recibo_tienda` | `printStoreReceipt()` | Recibo farmacia/retail |

**Cajón de dinero** (RestaurantesWeb ISSUE-114): se abre solo si la petición lo pide (`open_drawer: true` en el body de `/print`, o `POST /drawer` sin imprimir) **y** `cashDrawer.enabled` está activo. Sin el parámetro no se abre (antes se abría con cada impresión). Adaptadores: `print(cmds, { openDrawer })` y `openDrawer()`.

---

## Versión actual: v1.2.0 — Multi-conexión

Estado en `docs/PROJECT.md`, backlog en `docs/ROADMAP.md`, historial en `docs/CHANGELOG.md`.

Modos de conexión:
1. **USB + Driver Windows** — `electron-pos-printer`, usa impresoras instaladas en Windows, sin Zadig
2. **USB + LibUSB** — `escpos-usb`, requiere Zadig/WinUSB
3. **Red TCP/IP** — `escpos-network`

### Estructura de config (schema v1.2)

```json
{
  "connectionType": "network",
  "usbMode": "windows",
  "network": { "ip": "", "port": 9100 },
  "usb": { "printerName": "", "deviceId": "" },
  "cashDrawer": { "enabled": false, "pin": 2 }
}
```

Config persiste en disco via IPC. Ruta: `app.getPath('userData')/config.json`.

### Regla crítica: NO tocar

Las funciones de template (`printRecipe`, `printTicket`, `printComanda`, `printCloseBox`, `printInventory`, `printStoreReceipt`) **no se modifican**. El adapter debe ser transparente para ellas.

---

## Compatibilidad hacia atrás

Si un cliente externo envía `?ip=...&port=...` en la request:
- Esos valores sobrescriben config de disco **solo para esa petición**
- No persisten
- El flujo de red existente no se rompe

---

## Notas de desarrollo

- `electron-reload` está comentado en `main.js` — descomentar para hot reload en desarrollo
- `mainWindow.webContents.openDevTools()` está comentado — descomentar para debuggear renderer
- CORS configurado con `origin: '*'` — aceptable para servicio local
- App usa `app.requestSingleInstanceLock()` — solo corre una instancia
- Al cerrar ventana: se oculta (no cierra). Solo cierra desde menú de tray.
- Codepage de impresión: `CP850` (soporta caracteres latinos/españoles)

---

## Comandos útiles

```bash
npm start          # Iniciar en desarrollo
npm run make       # Build instalador Windows (Squirrel)
npm run package    # Solo empaquetar sin instalador
```
