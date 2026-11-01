# Controla

Punto de venta para negocios pequeños, con **módulos que se activan según el giro** del negocio.

### 🔗 En producción: **[controlaec.web.app](https://controlaec.web.app)**

> **Sobre este repositorio:** es una vitrina técnica. Documenta las decisiones de arquitectura. El código fuente es privado por ser un producto comercial en operación.

---

## Qué resuelve

Un POS genérico obliga a toda tienda a usar la misma pantalla. Pero una tienda de barrio, un taller de reparación de celulares y un local que vende fiado necesitan **cosas distintas**, y pagar por funciones que no usan es la razón más común por la que un comerciante abandona el software y vuelve al cuaderno.

Controla parte de un núcleo común —ventas, inventario, caja— y **activa módulos según el negocio**.

---

## Arquitectura por módulos

```
          ┌─────────── Núcleo común ───────────┐
          │  Ventas · Inventario · Caja        │
          │  Cierre · Gastos · Dashboard       │
          └────────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
   Reparaciones        Fiados          Impresión térmica
   (órdenes de       (crédito a        (tickets en
    servicio)         clientes)         impresora POS)
```

Cada negocio ve solo los módulos que activó. El mismo despliegue sirve a todos, con **aislamiento multi-tenant por negocio**.

### Módulos

| Módulo | Para quién | Qué hace |
|---|---|---|
| **Ventas e inventario** | Todos | Catálogo, stock, códigos de barras |
| **Caja y cierre** | Todos | Apertura, movimientos y cierre de caja cuadrado |
| **Gastos** | Todos | Registro de egresos para que el cierre refleje la realidad |
| **Dashboard** | Todos | Ventas, márgenes y tendencias |
| **Reparaciones** | Talleres | Órdenes de servicio con estado y seguimiento |
| **Fiados** | Tiendas de barrio | Crédito a clientes, saldo y abonos |
| **Impresión térmica** | Quien tenga impresora POS | Tickets en impresora térmica directa |
| **Facturación** | Quien emita | Comprobantes electrónicos |

---

## Decisiones técnicas

### Impresión térmica desde el navegador

Imprimir en una impresora POS desde una app web es la parte que más suele fallar. Se resolvió con un módulo dedicado (`impresion-repo`, `impresor-ticket`, `ticket-eventos`) que separa **qué se imprime** de **cómo se imprime**, de modo que el formato del ticket es independiente del dispositivo.

### Códigos de barras

Generación con **JsBarcode** para que el comerciante pueda etiquetar su propio inventario sin comprar software aparte.

### Caché en memoria

`cache-memoria.ts` evita releer catálogos completos en cada pantalla — importante cuando el POS corre en un teléfono de gama baja con conexión intermitente, que es el caso real.

### PWA instalable

El comerciante instala la app en su teléfono o tablet (`instalar-app.ts`). No hay que publicar en ninguna tienda de aplicaciones ni pagar por ello.

---

## Stack

- **Next.js** (App Router) + **TypeScript**
- **Firebase** — Firestore, Auth, Hosting
- **JsBarcode** para códigos de barras
- **Lucide** para iconografía
- PWA instalable

---

## Multi-tenant

Cada negocio tiene sus propios productos, ventas, caja y clientes. El aislamiento se aplica en cada consulta, no solo en la interfaz — un negocio no puede alcanzar los datos de otro ni accediendo por URL directa.

---

*Construido por [José David Villavicencio Canelos](https://github.com/josevillanelos95-crypto)*
