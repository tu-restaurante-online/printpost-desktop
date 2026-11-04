# print-post-electron — estado del proyecto

**Versión:** 1.2.0 · **Fase:** producción (servicio local de impresión térmica para RestaurantesWeb, PanaderiApp y retail)

## Stack

Electron 33 · Express 4 (puerto 3010) · escpos 3.0.0-alpha · electron-pos-printer 1.4 · Bootstrap 5.3 · electron-forge 7.6 (instalador Squirrel)

## Módulos

| Módulo | Estado |
|---|---|
| API HTTP `/`, `/test`, `/print`, `/drawer` | ✔ |
| Templates: recibo, ticket, comanda, ticket_comanda, cierre_caja, inventario, recibo_tienda | ✔ |
| Conexión Red TCP/IP | ✔ |
| Conexión USB con driver Windows | ✔ |
| Conexión USB con LibUSB (Zadig) | ✔ |
| Config persistente (`userData/config.json`) + modal de configuración | ✔ |
| Cajón de dinero a pedido (`open_drawer` / `POST /drawer`) | ✔ |
| Build instalador Windows | ✔ build + arranque verificados; falta prueba con impresora real |

## Problemas conocidos

- Bug en `escpos-usb@3.0.0-alpha.4` (ver README → Problemas frecuentes).
- El modo LibUSB requiere driver WinUSB instalado con Zadig.

## Próximo objetivo

Instalar `print-pos-1.2.0 Setup.exe` en un equipo con impresora y probar los 3 modos de conexión y el cajón.
