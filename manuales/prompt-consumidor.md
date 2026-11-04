# Prompt para sistemas que consumen print-post-electron

> Copiar el bloque de abajo en el asistente de IA (o pasarlo al dev) del sistema que imprime: RestaurantesWeb frontend, PanaderiApp, retail, etc.

---

Vas a integrar este sistema con **print-post-electron**, un servicio local de impresión térmica (ESC/POS) que corre en la PC de la caja. Expone una API HTTP en `http://<host>:3010` (por defecto `http://127.0.0.1:3010`). No tiene autenticación: el header `Authorization` se ignora. CORS abierto.

## Endpoints

| Método | Ruta | Uso |
|---|---|---|
| `GET` | `/` | Healthcheck → `{"success":1,"message":"Servicio activo"}` |
| `GET` | `/test` | Imprime una página de prueba |
| `POST` | `/print` | Imprime un comprobante (`Content-Type: application/json`) |
| `POST` | `/drawer` | Abre el cajón de dinero sin imprimir |

### Conexión a la impresora
- La impresora (red, USB con driver de Windows o LibUSB) se configura **en el propio servicio**. El consumidor no necesita mandar nada.
- Opcional, por compatibilidad: `?ip=192.168.1.50&port=9100` en la query fuerza una impresora de red **solo para esa petición**. Si `port` está vacío, se usa 9100. Si `ip` está vacío, se usa la config del servicio.

### Respuestas
- Éxito: `200 {"success":1,"message":"Ticket impreso correctamente"}`
- Error de impresora o de datos: `500 {"success":0,"message":"Error al imprimir: <detalle>"}`
- `/drawer`: `200 {"success":1,"opened":true|false}`. `opened:false` = el cajón está deshabilitado en la config del servicio (no es error).
- ⚠️ **Si falta un campo obligatorio, el servicio responde 200 igual y NO imprime nada.** Validar en el cliente antes de enviar.

## Body de `POST /print`

`template` decide qué se imprime. Campos comunes:

| Campo | Tipo | Notas |
|---|---|---|
| `template` | string | `recibo`, `ticket`, `comanda`, `ticket_comanda`, `recibo_tienda`, `cierre_caja`, `inventario`. Si falta o no existe → recibo + comanda. |
| `open_drawer` | bool | `true` abre el cajón junto con la impresión, si está habilitado en el servicio. Sin el campo no se abre. Mandarlo solo al cobrar en efectivo o combinado. |
| `font_size` | number/string | `0` normal, `1` o `2` más alto. Aplica a recibo, comanda y recibo_tienda. |
| `employee` | string | Imprime `Atendido por: <nombre>` al pie (recibo, comanda, recibo_tienda). |

### Reglas de tipos (importante)
- En `recibo`, `comanda` y `ticket_comanda`, **`details[].total` debe ser un NÚMERO** (`12.5`, no `"12.50"`). Un string hace fallar la impresión con 500.
- `payment_type` es **texto que se imprime tal cual**: `"Pago: <valor>"`. No mandar códigos internos (`1`, `2`, `3`); mandar el nombre legible. Si no se manda, la línea de pago no aparece.
- Texto en codepage CP850: acentos y ñ funcionan; emojis no.

### `recibo` — Recibo de venta
Obligatorios: `sale_number`, `details`.
```json
{
  "template": "recibo",
  "company_name": "Mi Restaurante",
  "sale_number": "001",
  "sale_type": "Mesa",
  "table_number": 5,
  "customer": "Juan Pérez",
  "payment_type": "Efectivo",
  "discount": 0,
  "font_size": 0,
  "employee": "Carlos",
  "open_drawer": true,
  "details": [
    { "product": "Pollo económico", "quantity": 1, "total": 12 },
    { "product": "Hamburguesa completa", "quantity": 2, "total": 24 }
  ]
}
```
El total impreso = suma de `details[].total` − `discount`.

### `ticket` — Ticket de turno (sin items)
Obligatorios: `sale_number`, `sale_type`.
```json
{ "template": "ticket", "company_name": "Mi Restaurante", "sale_number": "001", "sale_type": "Mesa", "table_number": 5 }
```

### `comanda` — Comanda de cocina
Obligatorios: `sale_number`, `details`. No lleva precios ni método de pago.
```json
{
  "template": "comanda",
  "sale_number": "001", "sale_type": "Mesa", "table_number": 5,
  "customer": "Juan Pérez", "observations": "Sin aceitunas",
  "font_size": 0, "employee": "Carlos",
  "details": [ { "product": "Pollo económico", "quantity": 1, "total": 12 } ]
}
```

### `ticket_comanda`
Imprime `ticket` y después `comanda` con el mismo body. Obligatorios: `sale_number`, `sale_type`, `details`. El cajón se abre una sola vez.

### `recibo_tienda` — Retail / farmacia
Obligatorios: `sale_number`, `details` (con `unit_price`).
```json
{
  "template": "recibo_tienda",
  "company_name": "Farmacia Central",
  "sale_number": "001",
  "customer": "Juan Pérez",
  "payment_type": "Efectivo",
  "discount": 1.00,
  "tax": 1.08,
  "employee": "Ana",
  "details": [
    { "product": "Paracetamol 500mg", "quantity": 2, "unit_price": 5.00, "total": 10.00 }
  ]
}
```

### `cierre_caja`
Obligatorios: `user`, `date`. Montos numéricos. `real_amount` opcional: si llega, imprime "Esperado" y "Real".
```json
{
  "template": "cierre_caja",
  "user": "Admin", "date": "15/05/2026",
  "opening_amount": 500, "income_amount": 1200, "expenses_amount": 150,
  "closed_amount": 1550, "real_amount": 1550,
  "missing_amount": 0, "surplus_amount": 0, "qr_amount": 200, "total_sales_amount": 1200,
  "products": [ { "name": "Pollo", "opening": 20, "closed": 10 } ],
  "sales":    [ { "product": "Pollo", "quantity": "10", "total": "120.00" } ],
  "money":    [ { "amount": "Bs. 200", "quantity": "3" } ]
}
```
`products`, `sales` y `money` son opcionales: si faltan, esa sección no se imprime. En `money` el servicio calcula el subtotal de cada corte (`amount × quantity`) y `TOTAL CORTES`; `amount` acepta `200`, `"200.00"` o `"Bs. 200"`.

### `inventario`
Obligatorio: `products`.
```json
{ "template": "inventario", "user": "Admin", "branch": "Sucursal Central", "products": [ { "name": "Paracetamol", "stock": 120 } ] }
```

## Cajón de dinero
- Cobro con impresión → `open_drawer: true` en el body de `/print`, solo en una impresora (la principal).
- Cobro sin impresión → `POST /drawer`.
- No mandarlo en pagos solo QR/tarjeta.
- Versiones viejas del servicio (o printpos-app en Android) no tienen `/drawer` y responden error: ignorarlo.

## Checklist de integración
1. `GET /` responde antes de habilitar la impresión térmica.
2. Todos los obligatorios de cada template están presentes (si no, no imprime y no avisa).
3. `details[].total` es número.
4. `payment_type` va como texto legible.
5. `open_drawer` solo en efectivo o combinado.

---

## Nota específica RestaurantesWeb (frontend)

Implementado 2026-11-03: `paymentLabel()` en `frontend/src/utils/printing.js`, usado por `thermalBody()`. La venta guarda `payment_type` numérico y se envía como texto:

| Venta | Enviar |
|---|---|
| `paid_out` + `payment_type 1` | `"Efectivo"` |
| `paid_out` + `payment_type 2` | `"QR/Tarjeta"` |
| `paid_out` + `payment_type 3` | `"Efectivo 20.00 / QR 15.00"` (`amount_received_cash` / `amount_received_qr`) |
| sin cobrar, `on_credit` | `"Crédito"` |
| sin cobrar | `null` (no se imprime la línea) |

Solo lo usan `recibo` y la impresión sin template (recibo + comanda); la comanda lo ignora.
