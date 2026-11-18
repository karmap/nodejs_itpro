# Clase 09: Express, Zod y operaciones sobre productos

**Duración:** 2 horas

> Los bloques de desarrollo se leen en orden dentro de cada ejemplo. Las plantillas con `...` se completan en las actividades; las cheat sheets reúnen operaciones independientes. Los programas completos incluyen sus imports.

**Objetivo:** reconstruir la API manual de la clase 08 con Express, validar entradas con Zod y agregar parámetros, creación y eliminación de productos. PUT queda como práctica opcional.

### Objetivos

- Relacionar Express con FastAPI y Zod con Pydantic.
- Registrar rutas GET, POST y DELETE.
- Leer parámetros de ruta, de query y el cuerpo JSON.
- Validar datos con esquemas de Zod y responder 422 con los errores.
- Generar IDs sin repetirlos después de un DELETE.
- Centralizar las respuestas 404, 400 y 500 con middleware.
- Consumir la API desde un cliente con `fetch` y `readline`.
- Validar respuestas de IA con Zod.

---

## 1. De HTTP manual a Express

En la clase 08 elegimos rutas con `if`, leímos el cuerpo por partes y convertimos cada respuesta con `JSON.stringify()`. Conservaremos el cliente y reemplazaremos el servidor:

```text
Cliente → node:http → Express → función JavaScript → JSON → cliente
                         ↕
                   Zod / validación
```

En el curso de Python (clase 11) hicimos el mismo cambio con FastAPI. Las piezas se corresponden así:

| En Python | En JavaScript | Responsabilidad |
|---|---|---|
| FastAPI | Express | Registra rutas y coordina la API. |
| Uvicorn | `node:http` (dentro de Express) | Escucha HTTP y entrega cada petición a la aplicación. |
| Pydantic | Zod | Esquemas, conversión y validación de datos. |
| `/docs` automático | No incluido | Probamos con `curl` y con un cliente propio. |

Express es un framework pequeño. A diferencia de FastAPI, **no valida nada por su cuenta**: no lee los tipos de una función ni genera documentación. La validación será explícita con Zod, y la documentación interactiva queda fuera de esta clase.

| Servidor manual (clase 08) | Con Express |
|---|---|
| `if (req.method === "GET" && url.pathname === "/products")` | `app.get("/products", ...)` |
| `res.setHeader()` + `res.end(JSON.stringify(datos))` | `res.json(datos)` |
| `res.statusCode = 404` | `res.status(404).json(...)` |
| Leer el body con `for await` y `JSON.parse()` | `app.use(express.json())` y `req.body` |
| Comprobar cada campo con `if` | Esquema de Zod |

Express abstrae el trabajo manual que acabamos de practicar. Nosotros seguimos eligiendo qué hace cada ruta y qué código HTTP corresponde.

## 2. Preparar el proyecto

```text
products_api/
├── package.json
├── schemas.js
├── server.js
└── client.js
```

Desde una carpeta nueva `products_api/`:

```bash
npm init -y
npm pkg set type=module
npm pkg set scripts.dev="node --watch server.js"
npm pkg set scripts.start="node server.js"
npm install express@5 zod@4
```

`npm pkg set` modifica `package.json` sin abrirlo en el editor; puedes hacer los mismos cambios a mano, como en la clase 06. La parte relevante del archivo queda así:

```json
{
  "type": "module",
  "scripts": {
    "dev": "node --watch server.js",
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^5.3.0",
    "zod": "^4.6.7"
  }
}
```

La edición del curso se comprobó con Express 5.3.0 y Zod 4.6.7. El `package.json` de la raíz del repositorio fija esas versiones; también puedes ejecutar `npm install` desde la raíz si quieres reproducir ese entorno.

> Express 5 cambió varias cosas respecto a Express 4. En Internet encontrarás muchos ejemplos de la versión 4; los de esta clase usan la 5.

## 3. Primera aplicación

Crea `server.js`:

```javascript
import express from "express";

const PORT = Number(process.env.PORT ?? 8000);

const app = express();

app.get("/", (req, res) => {
  res.json({ mensaje: "Hola desde Express" });
});

app.listen(PORT, (error) => {
  if (error) {
    throw error;
  }
  console.log(`Products API en http://localhost:${PORT}`);
});
```

- `express()` crea la aplicación, equivalente a `app = FastAPI()`.
- `app.get(ruta, función)` registra la función para peticiones GET a esa ruta. En FastAPI usábamos el decorador `@app.get("/")`; aquí pasamos la función como **callback**, como vimos en la clase 04.
- Express llama a la función con dos objetos: `req` (la petición) y `res` (la respuesta). Son los mismos objetos de `node:http`, con métodos adicionales.
- `res.json(datos)` convierte los datos a JSON, añade la cabecera `Content-Type` y envía la respuesta. El código por defecto es 200.
- `app.listen()` inicia el servidor. Si el puerto está ocupado, el callback recibe el error y lo relanzamos para que el programa termine con un mensaje claro.
- `PORT` se lee del entorno (clase 06) y usa 8000 si no existe. `??` solo reemplaza `undefined` o `null`.

Ejecuta:

```bash
npm run dev
```

Abre `http://localhost:8000/`. `node --watch` reinicia el servidor al guardar cambios, como `fastapi dev`. Detén el servidor de la clase 08, o cualquier servidor de Python, antes de usar el mismo puerto; si no, verás `EADDRINUSE`.

## 4. Middleware

Un **middleware** es una función que Express ejecuta para cada petición antes de llegar, o en lugar de llegar, a una ruta. Se registra con `app.use()` y se ejecuta **en el orden en que lo registras**.

Añade esta línea justo después de `const app = express();`:

```javascript
app.use(express.json());
```

`express.json()` lee el cuerpo de las peticiones con `Content-Type: application/json`, aplica `JSON.parse()` y deja el resultado en `req.body`. Es el trabajo que hicimos a mano con `for await` en la clase 08.

```text
petición → express.json() → ruta que coincide → respuesta
```

Si la petición no trae JSON, `req.body` queda como `undefined`. Si el JSON está mal formado, `express.json()` produce un error que trataremos en la sección 10.

## 5. GET y parámetros

Añade los datos debajo de `app.use(express.json());`:

```javascript
const productos = [
  { id: 1, nombre: "Laptop", precio: 15000, stock: 5, descripcion: null },
  { id: 2, nombre: "Mouse", precio: 450, stock: 12, descripcion: null },
  { id: 3, nombre: "Monitor", precio: 4200, stock: 3, descripcion: null },
];
```

En Express, los parámetros de ruta se declaran con `:` y llegan en `req.params`. Los de query llegan en `req.query`:

| En FastAPI | En Express |
|---|---|
| `@app.get("/products/{product_id}")` | `app.get("/products/:id", ...)` |
| `product_id: int` | `req.params.id` (texto) |
| `min_price: float | None = None` | `req.query.min_price` (texto o `undefined`) |

Prueba esta versión provisional para observar qué recibe Express:

```javascript
app.get("/products", (req, res) => {
  console.log(req.query);
  res.json(productos);
});

app.get("/products/:id", (req, res) => {
  console.log(req.params);
  const producto = productos.find((p) => p.id === req.params.id);
  if (producto === undefined) {
    return res.status(404).json({ detail: "Producto no encontrado" });
  }
  res.json(producto);
});
```

Visita `/products?min_price=1000&in_stock=true` y `/products/1`. En la terminal del servidor verás:

```text
[Object: null prototype] { min_price: '1000', in_stock: 'true' }
[Object: null prototype] { id: '1' }
```

Todo llega como **texto**. Por eso `/products/1` responde 404: `"1" === 1` es `false`. FastAPI convertía los valores gracias a las anotaciones; Express no lo hace.

Podríamos usar `Number(req.params.id)`, pero `Number("abc")` devuelve `NaN` sin error (clase 01) y respondería 404 en lugar de indicar que el ID no es válido. Necesitamos validar, como hacía FastAPI con 422. Esa es la tarea de Zod.

`res.status(404)` fija el código y devuelve el mismo `res`, por eso podemos encadenar `.json()`. Usamos `return` para salir de la función después de responder: una petición solo puede tener una respuesta. Usamos la clave `detail` para que los errores tengan la misma forma que en FastAPI.

> Express usa la **primera** ruta registrada que coincide. Si escribes una nueva versión de `/products`, reemplaza la anterior; no la añadas debajo.

## 6. Zod: esquemas y validación

Zod describe la forma esperada de los datos con un **esquema**. Crea `schemas.js`:

```javascript
import * as z from "zod";

z.config(z.locales.es());

export const ProductInput = z.strictObject({
  nombre: z.string().trim().min(1),
  precio: z.number().min(0),
  stock: z.int().min(0),
  descripcion: z.string().trim().nullable().default(null),
});
```

`z.config(z.locales.es())` pone los mensajes de error en español; se ejecuta una sola vez al importar el módulo.

| Pydantic (Python) | Zod (JavaScript) |
|---|---|
| `class ProductInput(BaseModel)` | `z.strictObject({ ... })` / `z.object({ ... })` |
| `extra="forbid"` | `z.strictObject()` rechaza claves desconocidas, incluido `id`. |
| `str_strip_whitespace=True` + `Field(min_length=1)` | `z.string().trim().min(1)` |
| `float = Field(ge=0, allow_inf_nan=False)` | `z.number().min(0)` (ya rechaza `Infinity` y `NaN`). |
| `int = Field(ge=0)` | `z.int().min(0)` |
| `str | None = None` | `z.string().nullable().default(null)` |
| `Literal["a", "b"]` | `z.enum(["a", "b"])` |
| `StrictBool` | `z.boolean()` |

Cada método devuelve un esquema nuevo con una regla más, por eso se encadenan. `.trim()` retira espacios exteriores antes de comprobar la longitud: un nombre de solo espacios queda inválido. `.nullable()` acepta `null` y `.default(null)` usa ese valor si el campo se omite.

`z.object()` también existe: acepta claves desconocidas pero **las elimina** del resultado. `z.strictObject()` las rechaza con un error. Para la entrada de la API usamos la versión estricta, como `extra="forbid"`.

### `safeParse()` y `parse()`

Prueba el esquema en un archivo aparte, `probar_schemas.js`, junto a `schemas.js`:

```javascript
import * as z from "zod";
import { ProductInput } from "./schemas.js";

const valido = ProductInput.safeParse({
  nombre: "  Webcam ",
  precio: 800,
  stock: 6,
});
console.log(valido);

const invalido = ProductInput.safeParse({
  nombre: "  ",
  precio: -1,
  stock: 1.5,
  id: 9,
});
console.log(invalido.success);
console.log(z.flattenError(invalido.error));
```

Resultado:

```text
{
  success: true,
  data: { nombre: 'Webcam', precio: 800, stock: 6, descripcion: null }
}
false
{
  formErrors: [ 'Llave desconocida: "id"' ],
  fieldErrors: {
    nombre: [
      'Demasiado pequeño: se esperaba que texto tuviera >=1 caracteres'
    ],
    precio: [ 'Demasiado pequeño: se esperaba que número fuera >=0' ],
    stock: [ 'Entrada inválida: se esperaba int, recibido número' ]
  }
}
```

- `safeParse(datos)` **no lanza** excepciones. Devuelve `{ success: true, data }` o `{ success: false, error }`. `data` contiene los datos ya limpios: sin espacios exteriores y con `descripcion: null`.
- `parse(datos)` devuelve los datos válidos o **lanza** un `ZodError`, que es un `Error` y se captura con `try`/`catch` (clase 05).
- `z.flattenError(error)` agrupa los mensajes: `fieldErrors` por campo y `formErrors` para problemas del objeto completo, como una clave desconocida.

En las rutas usaremos `safeParse()`, porque un error de validación es un resultado esperado, no una falla del servidor.

> **Diferencia con Pydantic:** Pydantic convertía `"800"` en número. Zod, por defecto, **no convierte**: `precio: "800"` es inválido. En el cuerpo JSON esperamos números reales. Para la query, donde todo llega como texto, pediremos la conversión de forma explícita.

## 7. Validar query y ruta

Añade a `schemas.js`:

```javascript
export const ProductFilters = z.object({
  min_price: z.coerce.number().min(0).optional(),
  in_stock: z.stringbool().default(false),
});

export const ProductId = z.coerce.number().int().positive();
```

- `z.coerce.number()` aplica `Number()` antes de validar. `"1000"` se convierte en `1000`; `"abc"` se convierte en `NaN` y falla.
- `.optional()` permite que `min_price` no venga en la URL; su valor queda `undefined`, como `None` en FastAPI.
- `z.stringbool()` interpreta textos como `"true"`, `"false"`, `"1"`, `"0"`, `"yes"` o `"no"`. Otro valor es inválido. Por defecto, `in_stock` es `false`.
- `ProductId` exige un entero positivo: `"abc"`, `"1.5"`, `"0"` y `""` fallan.

Para los filtros usamos `z.object()` y no la versión estricta: un parámetro de query desconocido se ignora, como hacía FastAPI.

Actualiza el import en `server.js` y reemplaza las dos rutas provisionales:

```javascript
import express from "express";
import * as z from "zod";
import { ProductFilters, ProductId } from "./schemas.js";
```

```javascript
function responderError422(res, error) {
  return res.status(422).json({ detail: z.flattenError(error) });
}

function responderNoEncontrado(res) {
  return res.status(404).json({ detail: "Producto no encontrado" });
}

app.get("/products", (req, res) => {
  const filtros = ProductFilters.safeParse(req.query);
  if (!filtros.success) {
    return responderError422(res, filtros.error);
  }

  const { min_price, in_stock } = filtros.data;
  let resultado = productos;
  if (min_price !== undefined) {
    resultado = resultado.filter((p) => p.precio >= min_price);
  }
  if (in_stock) {
    resultado = resultado.filter((p) => p.stock > 0);
  }
  res.json(resultado);
});

app.get("/products/:id", (req, res) => {
  const id = ProductId.safeParse(req.params.id);
  if (!id.success) {
    return responderError422(res, id.error);
  }

  const producto = productos.find((p) => p.id === id.data);
  if (producto === undefined) {
    return responderNoEncontrado(res);
  }
  res.json(producto);
});
```

Las dos funciones auxiliares evitan repetir la forma de las respuestas de error. `filter()` crea arrays nuevos (clase 04): filtrar no modifica la lista original.

Prueba en el navegador:

```text
/products
/products?min_price=1000
/products?in_stock=true
/products?min_price=1000&in_stock=true
/products?min_price=abc
/products/1
/products/999
/products/abc
```

`/products/1` responde 200, `/products/999` responde 404 y `/products/abc` responde 422:

```json
{"detail":{"formErrors":["Entrada inválida: se esperaba número, recibido NaN"],"fieldErrors":{}}}
```

Devolver `{ error: "..." }` con `res.json()` por sí solo no cambia el código HTTP; sería 200. Usa siempre `res.status()` para los errores.

## Actividad 1: Filtros y errores

1. Agrega un producto con stock cero a la lista inicial.
2. Comprueba los filtros por separado y combinados.
3. Verifica que el filtrado no cambie la lista original.
4. Prueba un ID existente, uno ausente, `abc` y `1.5`.
5. Prueba `in_stock=quizas` y observa el mensaje de Zod.

## 8. POST y generación de IDs

Actualiza el import de `schemas.js` para incluir `ProductInput`. Añade el contador junto a los datos iniciales:

```javascript
let siguienteId = 4;
```

En Python usamos `count(start=4)` y `next(ids)`. Aquí basta una variable con `let`: cada creación usa el valor actual y lo incrementa. Eliminar productos no reduce el contador. No usamos `productos.length + 1`, que podría repetir un ID después de un DELETE.

```javascript
app.post("/products", (req, res) => {
  const entrada = ProductInput.safeParse(req.body);
  if (!entrada.success) {
    return responderError422(res, entrada.error);
  }

  const nuevoProducto = { id: siguienteId, ...entrada.data };
  siguienteId += 1;
  productos.push(nuevoProducto);
  res.status(201).json(nuevoProducto);
});
```

`...entrada.data` copia los campos validados en el objeto nuevo con el spread de la clase 03; en FastAPI escribíamos `{"id": ..., **product.model_dump()}`. El código 201 indica que se creó un registro.

Sin `/docs`, enviamos el POST con `curl` (la clase 08 de este curso y la clase 10 de Python lo usaron):

```bash
curl -X POST http://localhost:8000/products \
  -H "Content-Type: application/json" \
  -d '{"nombre": "Webcam", "precio": 800, "stock": 6, "descripcion": "Webcam Full HD"}'
```

Respuesta:

```json
{"id":4,"nombre":"Webcam","precio":800,"stock":6,"descripcion":"Webcam Full HD"}
```

> En Windows PowerShell usa `curl.exe` en lugar de `curl`, o prueba desde Git Bash. Las comillas del JSON se escriben distinto en cada terminal; si se complica, usa el cliente de la sección 12.

Prueba también cuerpos inválidos. Todos deben responder 422 sin crear productos:

```bash
curl -X POST http://localhost:8000/products -H "Content-Type: application/json" -d '{}'
curl -X POST http://localhost:8000/products -H "Content-Type: application/json" -d '{"nombre": "  ", "precio": -1, "stock": 1.5}'
curl -X POST http://localhost:8000/products -H "Content-Type: application/json" -d '{"nombre": "X", "precio": "abc", "stock": 1}'
curl -X POST http://localhost:8000/products -H "Content-Type: application/json" -d '{"nombre": "X", "precio": 10, "stock": 1, "id": 99}'
```

Si olvidas `-H "Content-Type: application/json"`, `express.json()` no lee el cuerpo, `req.body` queda `undefined` y Zod responde 422 con `se esperaba objeto, recibido indefinido`.

Este ejemplo mantiene datos y contador en memoria dentro de un único proceso. Al reiniciar, o cuando `--watch` recarga el archivo, se restauran los datos iniciales. En la clase 10 persistiremos los productos y dejaremos los IDs a SQLite.

## 9. DELETE

```javascript
app.delete("/products/:id", (req, res) => {
  const id = ProductId.safeParse(req.params.id);
  if (!id.success) {
    return responderError422(res, id.error);
  }

  const indice = productos.findIndex((p) => p.id === id.data);
  if (indice === -1) {
    return responderNoEncontrado(res);
  }
  productos.splice(indice, 1);
  res.json({ mensaje: "Producto eliminado" });
});
```

`findIndex()` devuelve `-1` si no encuentra el elemento (clase 04) y `splice(indice, 1)` elimina un elemento en esa posición (clase 02). Una eliminación exitosa devuelve 200 con un mensaje JSON; si el ID ya no existe, 404. No uses 204 si vas a devolver un cuerpo.

```bash
curl -X DELETE http://localhost:8000/products/2
```

## 10. Rutas no encontradas y errores

¿Qué pasa con una ruta que no existe, como `/nada`? Express responde 404 con una página HTML. Y si el cuerpo JSON está mal formado, responde 400, también en HTML. Nuestro cliente espera JSON, así que registraremos dos middleware **al final**, después de todas las rutas:

```javascript
app.use((req, res) => {
  res.status(404).json({ detail: "Ruta no encontrada" });
});

app.use((error, req, res, next) => {
  if (error.type === "entity.parse.failed") {
    return res.status(400).json({ detail: "El cuerpo no es JSON válido" });
  }
  console.error(error);
  res.status(500).json({ detail: "Error interno del servidor" });
});
```

- El primero no tiene ruta: se ejecuta para cualquier petición que **ninguna ruta anterior** respondió. Por eso va al final.
- El segundo es un **middleware de error**. Express lo reconoce porque recibe **cuatro** parámetros; aunque no usemos `next`, debe estar en la lista.
- `express.json()` marca el JSON inválido con `error.type === "entity.parse.failed"`. Respondemos 400, porque el cliente envió un cuerpo que no se puede leer. Es distinto de 422: con 400 ni siquiera hay datos que validar.
- Cualquier otro error es una falla nuestra. La registramos con `console.error()` para depurar y respondemos 500 sin revelar detalles internos al cliente.

En Express 5, si una función `async` lanza un error o una promesa es rechazada, el error llega solo a este middleware. En Express 4 había que hacerlo manualmente; verás ese código en ejemplos antiguos.

Prueba:

```bash
curl http://localhost:8000/nada
curl -X POST http://localhost:8000/products -H "Content-Type: application/json" -d '{mal json'
```

Para observar el 500, añade temporalmente esta ruta **antes** de los middleware finales, visita `/boom` y después elimínala:

```javascript
app.get("/boom", async () => {
  throw new Error("Falla de prueba");
});
```

## 11. Código completo

Reemplaza el contenido de los archivos por estas versiones consolidadas; no las añadas debajo de las rutas anteriores.

### `schemas.js`

```javascript
import * as z from "zod";

z.config(z.locales.es());

export const ProductInput = z.strictObject({
  nombre: z.string().trim().min(1),
  precio: z.number().min(0),
  stock: z.int().min(0),
  descripcion: z.string().trim().nullable().default(null),
});

export const ProductFilters = z.object({
  min_price: z.coerce.number().min(0).optional(),
  in_stock: z.stringbool().default(false),
});

export const ProductId = z.coerce.number().int().positive();
```

### `server.js`

```javascript
import express from "express";
import * as z from "zod";
import { ProductFilters, ProductId, ProductInput } from "./schemas.js";

const PORT = Number(process.env.PORT ?? 8000);

const app = express();
app.use(express.json());

const productos = [
  { id: 1, nombre: "Laptop", precio: 15000, stock: 5, descripcion: null },
  { id: 2, nombre: "Mouse", precio: 450, stock: 12, descripcion: null },
  { id: 3, nombre: "Monitor", precio: 4200, stock: 3, descripcion: null },
];
let siguienteId = 4;

function responderError422(res, error) {
  return res.status(422).json({ detail: z.flattenError(error) });
}

function responderNoEncontrado(res) {
  return res.status(404).json({ detail: "Producto no encontrado" });
}

app.get("/", (req, res) => {
  res.json({ mensaje: "Products API" });
});

app.get("/health", (req, res) => {
  res.json({ status: "ok" });
});

app.get("/products", (req, res) => {
  const filtros = ProductFilters.safeParse(req.query);
  if (!filtros.success) {
    return responderError422(res, filtros.error);
  }

  const { min_price, in_stock } = filtros.data;
  let resultado = productos;
  if (min_price !== undefined) {
    resultado = resultado.filter((p) => p.precio >= min_price);
  }
  if (in_stock) {
    resultado = resultado.filter((p) => p.stock > 0);
  }
  res.json(resultado);
});

app.get("/products/:id", (req, res) => {
  const id = ProductId.safeParse(req.params.id);
  if (!id.success) {
    return responderError422(res, id.error);
  }

  const producto = productos.find((p) => p.id === id.data);
  if (producto === undefined) {
    return responderNoEncontrado(res);
  }
  res.json(producto);
});

app.post("/products", (req, res) => {
  const entrada = ProductInput.safeParse(req.body);
  if (!entrada.success) {
    return responderError422(res, entrada.error);
  }

  const nuevoProducto = { id: siguienteId, ...entrada.data };
  siguienteId += 1;
  productos.push(nuevoProducto);
  res.status(201).json(nuevoProducto);
});

app.delete("/products/:id", (req, res) => {
  const id = ProductId.safeParse(req.params.id);
  if (!id.success) {
    return responderError422(res, id.error);
  }

  const indice = productos.findIndex((p) => p.id === id.data);
  if (indice === -1) {
    return responderNoEncontrado(res);
  }
  productos.splice(indice, 1);
  res.json({ mensaje: "Producto eliminado" });
});

app.use((req, res) => {
  res.status(404).json({ detail: "Ruta no encontrada" });
});

app.use((error, req, res, next) => {
  if (error.type === "entity.parse.failed") {
    return res.status(400).json({ detail: "El cuerpo no es JSON válido" });
  }
  console.error(error);
  res.status(500).json({ detail: "Error interno del servidor" });
});

app.listen(PORT, (error) => {
  if (error) {
    throw error;
  }
  console.log(`Products API en http://localhost:${PORT}`);
});
```

Observa el orden: middleware de JSON, datos, rutas, middleware de 404 y middleware de error. Si mueves el de 404 antes de las rutas, todas las peticiones responderán 404.

## 12. Cliente con `fetch`

Crea `client.js`. El cliente solo conoce HTTP, sin importar qué biblioteca usa el servidor. `pedirJson()` es la misma idea de la clase 07, adaptada a una URL base; `readline` viene de la clase 05:

```javascript
import { stdin as input, stdout as output } from "node:process";
import * as readline from "node:readline/promises";

const BASE_URL = "http://localhost:8000";

class HttpError extends Error {
  constructor(status, cuerpo) {
    super(`La API respondió con el código ${status}`);
    this.name = "HttpError";
    this.status = status;
    this.cuerpo = cuerpo;
  }
}

async function pedirJson(ruta, opciones = {}) {
  const response = await fetch(`${BASE_URL}${ruta}`, {
    ...opciones,
    signal: AbortSignal.timeout(10_000),
  });
  const datos = await response.json();
  if (!response.ok) {
    throw new HttpError(response.status, datos);
  }
  return datos;
}

function enviarJson(ruta, metodo, datos) {
  return pedirJson(ruta, {
    method: metodo,
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(datos),
  });
}

function leerNumero(texto) {
  const limpio = texto.trim();
  return limpio === "" ? NaN : Number(limpio);
}

async function main() {
  const rl = readline.createInterface({ input, output });

  try {
    const salud = await pedirJson("/health");
    console.log("Estado:", salud.status);
    console.log("Productos:", await pedirJson("/products"));

    const nombre = (await rl.question("Nombre: ")).trim();
    const precio = leerNumero(await rl.question("Precio: "));
    const stock = leerNumero(await rl.question("Stock: "));
    const descripcion =
      (await rl.question("Descripción (opcional): ")).trim() || null;

    if (!Number.isFinite(precio) || !Number.isFinite(stock)) {
      console.log("Precio y stock deben ser números válidos");
      return;
    }

    const producto = { nombre, precio, stock, descripcion };
    const creado = await enviarJson("/products", "POST", producto);
    console.log("Creado:", creado);
    console.log("Lista actualizada:", await pedirJson("/products"));
  } catch (error) {
    if (error instanceof HttpError) {
      console.log("La API rechazó la petición:", error.status);
      console.log(JSON.stringify(error.cuerpo, null, 2));
    } else if (error.name === "TimeoutError") {
      console.log("La API tardó demasiado en responder");
    } else if (error instanceof SyntaxError) {
      console.log("La API no devolvió JSON válido");
    } else if (error instanceof TypeError) {
      console.log("No fue posible conectar con la API:", error.message);
    } else {
      throw error;
    }
  } finally {
    rl.close();
  }
}

if (import.meta.main) {
  await main();
}
```

Con el servidor activo en una terminal, ejecuta `node client.js` en otra.

- `fetch` no lanza errores con 4xx o 5xx (clase 07); `pedirJson()` revisa `response.ok` y lanza `HttpError`, como `raise_for_status()`.
- `leerNumero()` existe porque `Number("")` devuelve `0`: una respuesta vacía se convertiría en precio cero. Con `NaN`, `Number.isFinite()` detecta el problema. En Python, `float("")` lanzaba `ValueError`.
- `texto.trim() || null` devuelve el texto si queda contenido o `null` si queda vacío; `||` devuelve uno de sus operandos, igual que `or` en Python.
- El cliente comprueba los números para explicar errores locales, pero el servidor sigue siendo responsable de validar cualquier petición: prueba un stock `1.5` y verás el 422 con el mensaje de Zod.
- `finally` cierra `readline` para que el programa termine aunque ocurra un error.

## Actividad 2: Cliente POST y DELETE

1. Crea un producto y conserva el ID de la respuesta.
2. Consulta ese ID con `pedirJson()`.
3. Elimínalo con `pedirJson(ruta, { method: "DELETE" })` y consulta de nuevo para observar el `HttpError` con 404.
4. Elimina un producto inicial que no sea el último y crea dos productos más. Comprueba que todos los IDs existentes sean distintos.
5. Detén el servidor y ejecuta el cliente: debe mostrar el mensaje de conexión, no un error sin manejar.
6. Reinicia el servidor y observa que los cambios en memoria desaparecen.

Usa siempre `AbortSignal.timeout()`, revisa `response.ok` y maneja los errores de red.

## 13. CRUD y PUT opcional

| Operación | Método | Estado en esta clase |
|---|---|---|
| Crear | POST | Implementado |
| Leer | GET | Implementado |
| Actualizar | PUT o PATCH | PUT como bonus |
| Eliminar | DELETE | Implementado |

Añade este endpoint al `server.js` completo, antes de los middleware finales, si sobra tiempo:

```javascript
app.put("/products/:id", (req, res) => {
  const id = ProductId.safeParse(req.params.id);
  if (!id.success) {
    return responderError422(res, id.error);
  }
  const entrada = ProductInput.safeParse(req.body);
  if (!entrada.success) {
    return responderError422(res, entrada.error);
  }

  const indice = productos.findIndex((p) => p.id === id.data);
  if (indice === -1) {
    return responderNoEncontrado(res);
  }
  const actualizado = { id: id.data, ...entrada.data };
  productos[indice] = actualizado;
  res.json(actualizado);
});
```

PUT reemplaza los campos del producto conservando su ID. Como `descripcion` tiene un valor por defecto, omitirla en PUT la reemplaza por `null`. PATCH serviría para cambios parciales y queda fuera de esta clase.

## 14. Validar respuestas de IA con Zod

En la clase 07 comprobamos claves y tipos manualmente con `typeof` y `includes()`. Zod permite expresar ese contrato como un esquema, igual que hicimos con Pydantic en Python. Crea `review_schema.js`:

```javascript
import * as z from "zod";

const ReviewAnalysis = z.object({
  sentimiento: z.enum(["positivo", "neutral", "negativo", "mixto"]),
  categoria: z.enum(["producto", "envio", "soporte", "cobro", "otro"]),
  urgente: z.boolean(),
  resumen: z.string().trim().min(1),
});

const contenido = `
{"sentimiento":"negativo","categoria":"cobro",
 "urgente":true,"resumen":"Cobro duplicado"}
`;

try {
  const analisis = ReviewAnalysis.parse(JSON.parse(contenido));
  console.log(analisis);
} catch (error) {
  if (error instanceof SyntaxError) {
    console.log("La IA no devolvió JSON válido:", error.message);
  } else if (error instanceof z.ZodError) {
    console.log("La IA devolvió datos inválidos:");
    console.log(z.prettifyError(error));
  } else {
    throw error;
  }
}
```

- Zod no tiene un equivalente a `model_validate_json()`: primero `JSON.parse()` convierte el texto y después `parse()` valida. Cada paso tiene su propio error: `SyntaxError` y `ZodError`.
- `z.boolean()` exige un booleano real, como `StrictBool`: rechaza la cadena `"true"`.
- `z.object()` elimina claves adicionales del resultado. Si la IA añade un campo inesperado, no llega a nuestros datos.
- `z.prettifyError()` produce un texto legible con cada problema y su campo.

Cambia `"urgente":true` por `"urgente":"true"` y luego borra la llave de cierre `}` del JSON para observar cada error. Este archivo no llama a `z.config(z.locales.es())`, así que los mensajes de Zod aparecen en inglés:

```text
La IA devolvió datos inválidos:
✖ Invalid input: expected boolean, received string
  → at urgente
```

En una API que consulte IA, una entrada inválida del usuario corresponde a 422. Si el servicio externo devuelve JSON inválido o un esquema incorrecto, respondemos 502; si agota la espera, 504. `502` indica una respuesta inválida del servicio externo; `504` indica que ese servicio no respondió a tiempo.

### Esquemas del proyecto final

Estas reglas preparan el proyecto final. Guarda estos esquemas en el `schemas.js` de ese proyecto, no en `products_api/`:

```javascript
import * as z from "zod";

z.config(z.locales.es());

export const DeckInput = z.strictObject({
  topic: z.string().trim().min(1).max(100),
  count: z.int().min(3).max(10),
  level: z.enum(["básico", "intermedio", "avanzado"]),
});

const Card = z.strictObject({
  question: z.string().trim().min(1),
  answer: z.string().trim().min(1),
});

export const DeckResult = z.strictObject({
  summary: z.string().trim().min(1),
  cards: z.array(Card).min(1).max(10),
  study_minutes: z.int().min(5).max(180),
});

export function validarMazo(contenido, count) {
  const datos = JSON.parse(contenido);
  const resultado = DeckResult.parse(datos);
  if (resultado.cards.length > count) {
    throw new RangeError(`Se pidieron ${count} tarjetas como máximo`);
  }
  return resultado;
}
```

- `DeckInput` describe lo que envía el usuario: tema, cantidad de tarjetas y nivel. `.max(100)` limita la longitud del tema después de retirar espacios.
- `Card` describe cada tarjeta. Un esquema puede usarse dentro de otro: `z.array(Card)` exige un array cuyos elementos cumplan `Card`.
- En un array, `.min(1)` exige al menos un elemento. Como cada pregunta y respuesta usa `.trim().min(1)`, una tarjeta con texto vacío o de solo espacios también es inválida. En Python tuvimos que recorrer `must_do` a mano para esa regla.
- `DeckResult` usa `z.strictObject()` en todos sus niveles: el proyecto exige rechazar campos adicionales.
- El máximo de tarjetas depende de lo que pidió el usuario, no es fijo. Por eso `validarMazo()` recibe `count` y comprueba esa regla después del esquema.

`validarMazo()` puede lanzar tres errores: `SyntaxError` si el texto no es JSON, `ZodError` si no cumple el esquema y `RangeError` si sobran tarjetas. El servidor debe tratar **cualquiera** de ellos como un resultado inválido de la IA: responder 502 y terminar sin guardar. En la clase 10 veremos cómo encaja con la base de datos.

---

## Errores comunes viniendo de Python

| Error | Qué ocurre | Solución |
|---|---|---|
| Comparar `req.params.id` con un número | `"1" === 1` es `false`: siempre 404. | Validar y convertir con `ProductId.safeParse()`. |
| Esperar que Express valide los tipos | Express no lee anotaciones; acepta cualquier cuerpo. | Validar cada entrada con Zod. |
| Enviar `"800"` como precio | Pydantic lo convertía; Zod lo rechaza. | Enviar números en el JSON; usar `z.coerce` solo para query y ruta. |
| Olvidar `return` después de responder | El código sigue y Express intenta enviar una segunda respuesta (`ERR_HTTP_HEADERS_SENT`). | `return res.status(...).json(...)`. |
| Registrar el middleware de 404 antes de las rutas | Todas las peticiones responden 404. | Registrar 404 y errores al final. |
| Middleware de error con tres parámetros | Express lo trata como middleware normal y no recibe errores. | Declarar `(error, req, res, next)`. |
| Olvidar `Content-Type: application/json` | `req.body` queda `undefined` y la validación falla. | Añadir la cabecera en `curl` y en `fetch`. |
| Usar `productos.length + 1` como ID | Se repiten IDs después de un DELETE. | Contador propio, y en la clase 10, SQLite. |

---

## Ejercicio final: Products API

Construye los endpoints del código completo y un cliente que consulte `/health`, liste productos, cree uno, lo consulte y lo elimine. Conserva la descripción, valida los errores 400, 404 y 422 y comprueba los IDs después de DELETE.

Requisitos del cliente:

1. Mostrar el estado de `/health`.
2. Listar productos y mostrarlos como `NOMBRE - $precio - Stock: n`.
3. Crear un producto con datos leídos con `readline`.
4. Consultar el producto creado por su ID.
5. Eliminarlo y comprobar que una segunda consulta produce 404.
6. Mostrar mensajes comprensibles ante errores de la API, conexión o espera.

PUT es opcional. La persistencia se añade en la siguiente clase.

### Práctica opcional

Agrega una ruta `GET /products/search?q=texto` que devuelva los productos cuyo nombre contenga el texto, sin distinguir mayúsculas y minúsculas. Define su esquema de query con Zod y exige que `q` tenga al menos un carácter. Regístrala **antes** de `/products/:id`: si no, Express interpretará `search` como un ID.

---

## Cheat sheet

| Tarea | Python / FastAPI | JavaScript / Express + Zod |
|---|---|---|
| Ejecutar en desarrollo | `fastapi dev main.py` | `npm run dev` (`node --watch server.js`) |
| Crear la app | `app = FastAPI()` | `const app = express();` |
| Leer JSON del body | Automático | `app.use(express.json());` |
| Ruta GET | `@app.get("/products")` | `app.get("/products", (req, res) => { ... })` |
| Parámetro de ruta | `/products/{product_id}` | `/products/:id` → `req.params.id` |
| Query | `min_price: float | None = None` | `req.query.min_price` (texto) |
| Responder JSON | `return datos` | `res.json(datos)` |
| Crear | `@app.post(..., status_code=201)` | `res.status(201).json(nuevo)` |
| Error | `raise HTTPException(404, ...)` | `return res.status(404).json({ detail: "..." })` |
| Modelo de entrada | `class X(BaseModel)` + `extra="forbid"` | `z.strictObject({ ... })` |
| Texto sin espacios exteriores | `str_strip_whitespace=True` | `z.string().trim()` |
| Entero ≥ 0 | `int = Field(ge=0)` | `z.int().min(0)` |
| Opcional con `None` | `str | None = None` | `z.string().nullable().default(null)` |
| Valores permitidos | `Literal["a", "b"]` | `z.enum(["a", "b"])` |
| Validar sin excepción | — | `Schema.safeParse(datos)` |
| Validar con excepción | `Model.model_validate(datos)` | `Schema.parse(datos)` |
| Texto JSON a datos validados | `Model.model_validate_json(texto)` | `Schema.parse(JSON.parse(texto))` |
| Errores por campo | `error.errors()` | `z.flattenError(error)` |
| Convertir texto de query | Automático | `z.coerce.number()`, `z.stringbool()` |

Siguiente paso: sustituir la lista y el contador en memoria por registros e IDs de SQLite con el módulo `node:sqlite`.

---

[Índice del curso](README.md)
