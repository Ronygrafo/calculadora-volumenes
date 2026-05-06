# Calculadora de Volúmenes — Design Spec

**Fecha:** 2026-04-30  
**Proyecto:** Calculadora de Volúmenes para arriendo de bodegas personales (B2C)  
**Contexto:** Feature a medida embebida en sitio Wix existente del cliente

---

## Contexto del negocio

El cliente opera un negocio B2C de arriendo de bodegas personales. El sitio web corporativo está construido en Wix. Se necesita una sección interactiva donde el usuario pueda ingresar las dimensiones de sus objetos, calcular el volumen total que ocupa, y recibir una recomendación del tamaño de bodega que necesita arrendar.

---

## Arquitectura

Un único archivo `index.html` autocontenido (HTML + CSS + JS inline, sin dependencias externas) embebido en Wix mediante un **HTML Component (iframe)**. Toda la lógica y presentación corre en el browser del usuario. No hay backend.

```
index.html
  ├── <style>   — estilos de la calculadora
  ├── <body>    — UI dividida en 3 zonas
  └── <script>  — lógica y persistencia
```

---

## Funcionalidad

### Formulario de entrada
- Campos: **Nombre del item** (opcional), **Largo**, **Ancho**, **Alto** (los tres en centímetros)
- Botón "Agregar item"
- Si el nombre no se completa, se asigna automáticamente "Item 1", "Item 2", etc. (contador incremental que nunca retrocede; eliminar un item no reutiliza su número)
- Validación: Largo, Ancho y Alto deben ser números positivos mayores a 0; error inline si no se cumple

### Lista de items
- Muestra cada item con: nombre, dimensiones en cm, y volumen individual calculado en m³
- Botón "×" por fila para eliminar el item
- Estado vacío: mensaje "Agrega tu primer item para comenzar" cuando no hay items

### Cálculo de volumen
- Fórmula por item: `Largo (cm) × Ancho (cm) × Alto (cm) ÷ 1.000.000 = m³`
- Volumen total: suma de todos los items en lista
- El total se recalcula en tiempo real al agregar o eliminar cualquier item

### Resultado y recomendación
- La zona de resultado solo es visible cuando hay al menos 1 item
- Muestra el **volumen total en m³** (2 decimales)
- Recomienda automáticamente el tamaño de bodega según tabla placeholder:

| Volumen total | Bodega recomendada |
|---|---|
| Hasta 3 m³ | Bodega Pequeña (3 m³) |
| Más de 3 m³ hasta 8 m³ | Bodega Mediana (8 m³) |
| Más de 8 m³ | Bodega Grande (15 m³) |

> **Nota:** Los tamaños y nombres de bodegas son placeholders. Deben ser reemplazados con los valores reales del cliente antes del go-live.

---

## Persistencia

- Se usa `sessionStorage` para guardar el estado de la lista de items
- Al recargar la página, si existen datos en `sessionStorage`, la lista se restaura automáticamente
- Al cerrar el tab o el browser, los datos se limpian solos (comportamiento nativo de sessionStorage)
- No hay backend ni cookies involucrados

---

## Estilo visual

- **Tipografía:** sans-serif del sistema (sin dependencias externas)
- **Paleta:** fondo blanco/gris claro, acento en azul oscuro (variable CSS fácil de reemplazar con el color del cliente)
- **Inputs:** grandes y claros, aptos para interacción móvil
- **Lista:** filas alternadas para legibilidad
- **Zona resultado:** fondo de color suave, tipografía más grande, visualmente destacada
- **Responsive:** funciona correctamente en mobile, tablet y desktop

---

## Validaciones

| Campo | Regla |
|---|---|
| Nombre | Opcional; fallback automático "Item N" |
| Largo / Ancho / Alto | Requeridos, número > 0, no texto ni negativo |
| Formulario vacío | No permite agregar sin medidas válidas |

Errores mostrados inline debajo del campo correspondiente, sin alerts del browser.

---

## Organización del script

El bloque `<script>` se estructura en secciones claramente delimitadas con comentarios. Las constantes de lógica de negocio se declaran primero, en scope global, para que sean fáciles de encontrar y modificar sin tocar el resto del código:

```js
// ─── CONFIGURACIÓN DEL NEGOCIO ────────────────────────────────────────────
const STORAGE_UNITS = [
  { name: 'Bodega Pequeña', maxVolume: 3 },
  { name: 'Bodega Mediana', maxVolume: 8 },
  { name: 'Bodega Grande',  maxVolume: Infinity },
];
const SESSION_KEY = 'calculadora_items';

// ─── ESTADO ───────────────────────────────────────────────────────────────
// ─── LÓGICA DE CÁLCULO ────────────────────────────────────────────────────
// ─── PERSISTENCIA (sessionStorage) ────────────────────────────────────────
// ─── RENDERIZADO ──────────────────────────────────────────────────────────
// ─── EVENTOS ──────────────────────────────────────────────────────────────
// ─── INIT ─────────────────────────────────────────────────────────────────
```

Cualquier cambio en los tamaños o nombres de bodegas se hace únicamente en `STORAGE_UNITS`, sin tocar la lógica de cálculo ni el renderizado.

---

## Fuera del alcance (MVP)

- Animaciones complejas
- Modo oscuro
- Multilenguaje (solo español)
- Persistencia entre sesiones (localStorage o backend)
- CTA a cotización o catálogo de bodegas
- Edición de items ya agregados (solo eliminar)

---

## Integración en Wix

1. Desarrollar y testear `index.html` localmente en el browser
2. En el editor Wix, agregar un componente **"HTML iFrame"** en la sección deseada
3. Pegar el contenido del archivo `index.html` en el editor del componente
4. Ajustar la altura del iframe en Wix según el contenido renderizado

> El archivo se desarrolla y versiona fuera de Wix. Para actualizaciones, se edita el archivo y se pega nuevamente en el editor Wix.
