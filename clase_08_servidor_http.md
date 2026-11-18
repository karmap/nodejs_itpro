# Clase 08: Servidor HTTP con node:http

**Duración:** 2 horas

> Los bloques de desarrollo se leen en orden dentro de cada ejemplo. Las plantillas con `...` se completan en las actividades; las cheat sheets reúnen operaciones independientes. Los programas completos incluyen sus imports.

**Objetivo:** construir un servidor HTTP con el módulo incorporado `node:http`, probarlo desde el navegador, `curl` y un cliente con `fetch`, y entender qué trabajo resolverá Express en la clase 09.

### Objetivos

- Crear un servidor con `http.createServer()` y `server.listen()`.
- Leer el método, la ruta y los query parameters de una petición.
- Responder con códigos de estado, headers y JSON.
- Organizar las rutas con un helper `enviarJson()`.
- Entender por qué un solo proceso de Node.js atiende muchas peticiones y qué lo bloquea.
- Leer el body de un `POST` y validarlo a mano.
- Reconocer los problemas que motivan usar un framework.

---

## 1. ¿Qué vamos a construir?

En la clase 07 consumimos APIs:

```text
Nuestro programa
      ↓
fetch
      ↓
API externa
```

Ahora invertimos el flujo, igual que en el curso de Python (clase 10):

```text
Cliente
  │
  │ HTTP Request
  ▼
Nuestro servidor Node.js
  │
  │ HTTP Response
  ▼
Cliente
```

Lo probaremos desde:

```text
Navegador
curl
fetch
   │
   └────────────→ Servidor Node.js
                    │
                    └→ JSON
```

| En Python (`http.server`) | En Node.js (`node:http`) |
|---|---|
| `from http.server import ...` | `import http from "node:http";` |
| Clase `Handler(BaseHTTPRequestHandler)` | Una función `(req, res) => { ... }` |
| Un método por verbo: `do_GET()`, `do_POST()` | Una sola función para todos los métodos; se lee `req.method`. |
| `self.path` | `req.url` |
| `urlsplit(self.path).path` | `new URL(req.url, "http://localhost").pathname` |
| `self.send_response(200)` | `res.statusCode = 200` o `res.writeHead(200, ...)` |
| `self.send_header(...)` + `end_headers()` | `res.setHeader(...)` o los headers de `writeHead()` |
| `self.wfile.write(body)` | `res.end(body)` |
| `json.dumps(data).encode("utf-8")` | `JSON.stringify(datos)` |
| `server.serve_forever()` | `server.listen(8000, ...)` |

---

## 2. Primer servidor HTTP

Crea una carpeta `mini_api/` y dentro `server.js`:

```javascript
import http from "node:http";

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader("Content-Type", "text/plain; charset=utf-8");
  res.end("Hola desde Node.js");
});

server.listen(8000, "localhost", () => {
  console.log("Servidor corriendo en http://localhost:8000");
});
```

```bash
node server.js
```

```text
Servidor corriendo en http://localhost:8000
```

Qué hace cada parte:

| Código | Responsabilidad |
|---|---|
| `http.createServer(función)` | Crea el servidor. La función se ejecuta **una vez por cada petición**. |
| `req` | La petición: método, URL, headers y body. |
| `res` | La respuesta que vamos construyendo. |
| `res.statusCode = 200` | Código de estado. Si no lo cambias, es 200. |
| `res.setHeader(nombre, valor)` | Agrega una cabecera. |
| `res.end(texto)` | Envía el cuerpo y **termina** la respuesta. |
| `server.listen(puerto, host, función)` | Empieza a escuchar; la función se ejecuta cuando el servidor está listo. |

Toda petición debe terminar con `res.end()`. Si lo olvidas, el cliente se queda esperando hasta agotar su tiempo de espera.

`req` y `res` son nombres por convención: vienen de *request* y *response*. Los verás así en casi todo el código de Node.js, incluido Express.

El programa no termina después de `listen()`: el servidor mantiene vivo el proceso. Detenlo con `Ctrl + C` en la terminal. No necesitamos el `try` / `except KeyboardInterrupt` de Python.

Mientras desarrollas, puedes usar `node --watch server.js` (clase 06): Node.js reinicia el servidor cada vez que guardas el archivo.

---

## 3. `localhost` y puerto

```text
http://localhost:8000
        │        │
        │        └── puerto
        └─────────── tu propia computadora
```

- `localhost` significa "este mismo equipo". Al escuchar en `"localhost"`, solo los programas de tu computadora pueden conectarse.
- El **puerto** identifica qué programa recibe la conexión. Usamos 8000, igual que en el curso de Python.

Si omites `"localhost"` en `listen(8000)`, Node.js acepta conexiones desde cualquier red a la que esté conectado tu equipo. Para practicar, limítalo a `localhost`.

Solo un programa puede escuchar en un puerto. Si el servidor de Python de la clase 10 sigue activo, o ejecutas `node server.js` dos veces, verás:

```text
Error: listen EADDRINUSE: address already in use ::1:8000
```

`EADDRINUSE` significa "dirección en uso". Detén el otro servidor con `Ctrl + C`. Si quieres un mensaje más claro, maneja el evento `error` del servidor antes de `listen()`:

```javascript
server.on("error", (error) => {
  if (error.code === "EADDRINUSE") {
    console.log("El puerto 8000 está ocupado. Detén el otro servidor.");
  } else {
    throw error;
  }
});
```

`server.on("error", función)` registra una función que Node.js llama cuando ocurre ese evento. Los objetos de Node.js que emiten eventos, como el servidor, usan este patrón.

---

## 4. Probar desde el navegador

Con el servidor activo, abre:

```text
http://localhost:8000
```

Verás:

```text
Hola desde Node.js
```

Prueba también `http://localhost:8000/cualquier/ruta`. El resultado es el mismo: nuestra función todavía no mira la ruta. Lo arreglaremos en la sección 8.

El navegador es un cliente HTTP: envía un `GET` y muestra el cuerpo de la respuesta.

---

## 5. Código de estado

El código de estado se puede asignar con `res.statusCode` o con `res.writeHead()`, que además recibe los headers en un objeto:

```javascript
res.writeHead(404, { "Content-Type": "text/plain; charset=utf-8" });
res.end("No encontrado");
```

`writeHead()` envía la línea de estado y las cabeceras de una vez; después ya no puedes cambiarlas. Por eso se llama antes de `res.end()`.

| Código | Significado | Cuándo lo usaremos |
|---|---|---|
| 200 | OK | Consulta exitosa. |
| 201 | Created | Se creó un registro con `POST`. |
| 400 | Bad Request | El body no es JSON o un parámetro no tiene formato válido. |
| 404 | Not Found | La ruta o el recurso no existe. |
| 422 | Unprocessable Content | JSON válido con campos incorrectos, como respondía FastAPI. |
| 500 | Internal Server Error | Falló algo inesperado en el servidor. |

---

## 6. Headers y JSON

Para responder JSON necesitamos dos cosas:

1. La cabecera `Content-Type: application/json; charset=utf-8`, para que el cliente sepa que el cuerpo es JSON codificado en UTF-8.
2. Convertir el objeto en texto con `JSON.stringify()`.

```javascript
import http from "node:http";

const server = http.createServer((req, res) => {
  const datos = { mensaje: "Mi primera API con Node.js" };

  res.writeHead(200, { "Content-Type": "application/json; charset=utf-8" });
  res.end(JSON.stringify(datos));
});

server.listen(8000, "localhost", () => {
  console.log("Servidor corriendo en http://localhost:8000");
});
```

```text
objeto
  ↓
JSON.stringify()
  ↓
texto JSON
  ↓
res.end()
  ↓
bytes UTF-8 por la red
```

### ¿Dónde quedó el `.encode()` de Python?

En Python, `wfile.write()` exigía bytes: `json.dumps(data).encode("utf-8")`. En Node.js, `res.end()` acepta texto y lo codifica en UTF-8 por defecto. También acepta un `Buffer`, que es el tipo de Node.js para bytes; lo usaremos al leer el body de un `POST`.

No tienes que escribir la cabecera `Content-Length`: Node.js indica el tamaño del cuerpo por su cuenta (con `Content-Length` o con `Transfer-Encoding: chunked`, según cómo construyas la respuesta).

`res.end()` solo acepta texto o bytes. Si pasas un objeto, como `res.end(datos)`, Node.js lanza un `TypeError`. Siempre usa `JSON.stringify()`.

---

## 7. Probar con `curl`

`curl` es un cliente HTTP de terminal, como en el curso de Python:

```bash
curl http://localhost:8000
```

```text
{"mensaje":"Mi primera API con Node.js"}
```

Con `-i` incluye la línea de estado y las cabeceras:

```bash
curl -i http://localhost:8000
```

```text
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Date: ...
Connection: keep-alive
Keep-Alive: timeout=5
Transfer-Encoding: chunked

{"mensaje":"Mi primera API con Node.js"}
```

En Windows PowerShell usa `curl.exe` para no llamar al alias `curl` de PowerShell, que funciona distinto.

---

## 8. Método, ruta y query parameters

`req.url` contiene solo la ruta y la query, por ejemplo `/products?min_price=1000`. No incluye el protocolo ni el servidor. Para separar sus partes usamos el objeto `URL` de la clase 07, con una base cualquiera:

```javascript
const url = new URL(req.url, "http://localhost");

console.log(req.method);                      // "GET"
console.log(url.pathname);                    // "/products"
console.log(url.searchParams.get("min_price")); // "1000"
```

- `req.method` es el método en mayúsculas: `"GET"`, `"POST"`, `"DELETE"`...
- `url.pathname` es la ruta sin la query. Así `/health?x=1` sigue siendo `/health`.
- `url.searchParams.get(nombre)` devuelve el valor como **texto**, o `null` si no se envió.

### Una función para todos los métodos

En Python, `do_GET()` solo recibía peticiones `GET`. En Node.js, la misma función recibe **todos** los métodos. Si solo comparas la ruta, un `DELETE /products` respondería la lista de productos como si fuera un `GET`. Por eso combinamos método y ruta en una sola cadena:

```javascript
const ruta = `${req.method} ${url.pathname}`;
```

Y elegimos qué hacer con `switch`, que compara con `===`:

```javascript
switch (ruta) {
  case "GET /":
    // ...
    break;
  case "GET /health":
    // ...
    break;
  default:
    // Ninguna ruta coincidió: 404.
}
```

`break` termina el `switch`. Sin él, la ejecución continúa en el siguiente `case`. `default` funciona como el `else` final.

---

## 9. Helper `enviarJson()`

Cada respuesta JSON repite las mismas tres líneas. Igual que `enviar_json()` en Python, creamos una función:

```javascript
function enviarJson(res, status, datos) {
  res.writeHead(status, { "Content-Type": "application/json; charset=utf-8" });
  res.end(JSON.stringify(datos));
}
```

Recibe `res` como parámetro porque no es una variable global: cada petición tiene su propia respuesta.

### Rutas `/`, `/health`, `/products` y 404

Reemplaza `server.js`:

```javascript
import http from "node:http";

const productos = [
  { id: 1, nombre: "Laptop", precio: 15000, stock: 5 },
  { id: 2, nombre: "Mouse", precio: 450, stock: 12 },
  { id: 3, nombre: "Monitor", precio: 4200, stock: 3 },
];

function enviarJson(res, status, datos) {
  res.writeHead(status, { "Content-Type": "application/json; charset=utf-8" });
  res.end(JSON.stringify(datos));
}

const server = http.createServer((req, res) => {
  const url = new URL(req.url, "http://localhost");
  const ruta = `${req.method} ${url.pathname}`;
  console.log(ruta);

  switch (ruta) {
    case "GET /":
      enviarJson(res, 200, { mensaje: "Mi primera API con Node.js" });
      break;
    case "GET /health":
      enviarJson(res, 200, { status: "ok" });
      break;
    case "GET /products":
      enviarJson(res, 200, productos);
      break;
    default:
      enviarJson(res, 404, { error: "Ruta no encontrada" });
  }
});

server.listen(8000, "localhost", () => {
  console.log("Servidor corriendo en http://localhost:8000");
});
```

Prueba:

```bash
curl -i http://localhost:8000/health
curl -i http://localhost:8000/products
curl -i http://localhost:8000/no-existe
curl -i -X DELETE http://localhost:8000/products
```

Las dos últimas responden 404 con `{"error":"Ruta no encontrada"}`. `-X DELETE` cambia el método de la petición. En la terminal del servidor verás una línea por petición, como `GET /health`, gracias al `console.log(ruta)`.

Una API más precisa respondería 405 (*Method Not Allowed*) cuando la ruta existe pero el método no. Para este curso, 404 es suficiente.

---

## Actividad 1: Cambiar respuestas y probar errores

1. Cambia el mensaje de `/` y comprueba el cambio en el navegador. Si usas `node --watch`, no necesitas reiniciar.
2. Agrega un cuarto producto a la lista.
3. Con `curl -i`, comprueba el código y el `Content-Type` de `/health` y de una ruta inexistente.
4. Prueba `http://localhost:8000/health?x=1`. ¿Por qué sigue respondiendo `/health`?
5. Envía `curl -i -X POST http://localhost:8000/health`. Explica por qué responde 404.

---

## 10. Query parameters: filtrar productos

Agreguemos `/products?min_price=1000`. Los valores de `searchParams` siempre son texto, así que hay que convertirlos con `Number()` (clase 01) y comprobar el resultado. Recuerda que `Number("abc")` devuelve `NaN` sin lanzar error, y que `Number("")` devuelve `0`.

```javascript
function listarProductos(res, url) {
  const minPriceTexto = url.searchParams.get("min_price");
  if (minPriceTexto === null) {
    enviarJson(res, 200, productos);
    return;
  }

  const minPrice = Number(minPriceTexto);
  if (minPriceTexto.trim() === "" || Number.isNaN(minPrice)) {
    enviarJson(res, 400, { error: "min_price debe ser un número" });
    return;
  }
  enviarJson(res, 200, productos.filter((p) => p.precio >= minPrice));
}
```

Y en el `switch`:

```javascript
case "GET /products":
  listarProductos(res, url);
  break;
```

Prueba:

```text
/products
/products?min_price=1000
/products?min_price=abc
/products?min_price=
```

Las dos primeras responden 200; las dos últimas, 400. Sin la comprobación de cadena vacía, `?min_price=` filtraría con `0` y parecería una consulta correcta.

`filter()` (clase 04) crea un array nuevo: la lista original no cambia.

`return` después de responder termina la función. Así evitamos llamar dos veces a `enviarJson()`, que provocaría el error `ERR_HTTP_HEADERS_SENT` (las cabeceras ya se enviaron).

---

## 11. Probar desde un programa con `fetch`

El servidor responde a cualquier cliente HTTP. Con el servidor activo, crea en otra carpeta o en la misma `client.js`:

```javascript
const BASE_URL = "http://localhost:8000";

try {
  const response = await fetch(`${BASE_URL}/products`, {
    signal: AbortSignal.timeout(10_000),
  });
  if (!response.ok) {
    throw new Error(`Error HTTP ${response.status}`);
  }

  const productos = await response.json();

  for (const producto of productos) {
    console.log(producto.nombre, producto.precio);
  }
} catch (error) {
  if (error instanceof TypeError) {
    console.log("No fue posible conectar con la API");
  } else {
    console.log("Error:", error.message);
  }
}
```

Ejecuta el cliente en **otra terminal**, porque la primera está ocupada por el servidor:

```bash
node client.js
```

```text
Laptop 15000
Mouse 450
Monitor 4200
```

Si detienes el servidor y ejecutas de nuevo el cliente, verás `No fue posible conectar con la API`.

```text
NAVEGADOR ─┐
CURL ──────┼──→ Servidor node:http
FETCH ─────┘
```

Antes, `fetch` hablaba con un servidor externo. Ahora habla con el nuestro.

---

## 12. Un proceso, muchas peticiones: el event loop

En la clase 05 vimos que Node.js ejecuta tu código en un solo hilo con un event loop. En un servidor eso significa:

- Mientras una petición **espera** (un temporizador, un archivo, una consulta a otra API), el event loop atiende otras peticiones.
- Mientras una petición **ejecuta código síncrono largo**, nadie más puede ser atendido.

Compruébalo con este servidor de demostración, `bloqueo.js`:

```javascript
import http from "node:http";
import { setTimeout as esperar } from "node:timers/promises";

function enviarJson(res, status, datos) {
  res.writeHead(status, { "Content-Type": "application/json; charset=utf-8" });
  res.end(JSON.stringify(datos));
}

const server = http.createServer(async (req, res) => {
  const url = new URL(req.url, "http://localhost");

  if (url.pathname === "/espera") {
    await esperar(5000); // Espera sin bloquear: el event loop sigue libre.
    enviarJson(res, 200, { mensaje: "Terminé de esperar" });
  } else if (url.pathname === "/bloqueo") {
    const fin = Date.now() + 5000;
    while (Date.now() < fin) {
      // Trabajo síncrono: nadie más puede ser atendido.
    }
    enviarJson(res, 200, { mensaje: "Terminé de bloquear" });
  } else {
    enviarJson(res, 200, { status: "ok" });
  }
});

server.listen(8000, "localhost", () => {
  console.log("Servidor corriendo en http://localhost:8000");
});
```

La función que recibe `createServer()` puede ser `async`, porque usamos `await` dentro.

1. Abre `http://localhost:8000/espera` en una pestaña y, antes de que pasen cinco segundos, `http://localhost:8000/` en otra. La segunda responde de inmediato.
2. Repite con `/bloqueo`. Ahora `/` tarda hasta que termina el ciclo `while`.

```text
/espera   → await → el event loop atiende otras peticiones
/bloqueo  → while → todo el servidor espera
```

Regla práctica: en un servidor, usa las versiones asíncronas de las operaciones lentas (`fs/promises`, `fetch`) y evita ciclos largos de cálculo dentro de una petición.

---

## 13. ¿Qué pasa con POST?

En Python vimos el `POST` solo de forma conceptual. Aquí lo implementaremos para sentir todo el trabajo manual.

### Leer el body

El body no llega de una vez: llega en partes (*chunks*) de bytes. `req` es un **stream** que podemos recorrer con `for await...of`, la versión asíncrona de `for...of`:

```javascript
async function leerJson(req) {
  const partes = [];
  for await (const parte of req) {
    partes.push(parte);
  }
  const texto = Buffer.concat(partes).toString("utf8");
  return JSON.parse(texto);
}
```

- Cada `parte` es un `Buffer` (bytes).
- `Buffer.concat(partes)` une los bytes y `.toString("utf8")` los convierte en texto. Unimos antes de decodificar para no cortar un carácter como `á`, que ocupa dos bytes, entre dos partes.
- `JSON.parse()` lanza `SyntaxError` si el texto no es JSON. Un body vacío también falla, porque `""` no es JSON válido.

Esta versión no limita el tamaño del body. Un servidor real debe rechazar cuerpos demasiado grandes; Express lo hace por nosotros (100 KB por defecto).

### Validar a mano

`JSON.parse()` solo garantiza que el texto es JSON. Igual que con la IA en la clase 07, falta comprobar campos, tipos y reglas:

```javascript
function validarProducto(datos) {
  const errores = [];
  if (typeof datos !== "object" || datos === null || Array.isArray(datos)) {
    return ["El cuerpo debe ser un objeto JSON"];
  }
  if (typeof datos.nombre !== "string" || datos.nombre.trim() === "") {
    errores.push("nombre debe ser texto no vacío");
  }
  if (!Number.isFinite(datos.precio) || datos.precio < 0) {
    errores.push("precio debe ser un número mayor o igual a 0");
  }
  if (!Number.isInteger(datos.stock) || datos.stock < 0) {
    errores.push("stock debe ser un entero mayor o igual a 0");
  }
  return errores;
}
```

`Number.isFinite()` devuelve `false` para texto como `"800"`, para `NaN` y para `Infinity`. `Number.isInteger(1.5)` es `false`. La función devuelve un array de mensajes; si está vacío, los datos son válidos.

### Crear el producto

Añade junto a los datos iniciales:

```javascript
let siguienteId = 4;
```

Usamos `let` porque el valor cambia. Como en el curso de Python, no calculamos el ID con `productos.length + 1`: después de eliminar un producto podría repetirse un ID.

```javascript
async function crearProducto(req, res) {
  let datos;
  try {
    datos = await leerJson(req);
  } catch {
    enviarJson(res, 400, { error: "El cuerpo debe ser JSON válido" });
    return;
  }

  const errores = validarProducto(datos);
  if (errores.length > 0) {
    enviarJson(res, 422, { error: "Datos inválidos", detalles: errores });
    return;
  }

  const nuevoProducto = {
    id: siguienteId,
    nombre: datos.nombre.trim(),
    precio: datos.precio,
    stock: datos.stock,
  };
  siguienteId += 1;
  productos.push(nuevoProducto);
  enviarJson(res, 201, nuevoProducto);
}
```

- `catch` sin `(error)` es válido cuando no necesitas el objeto del error.
- Construimos `nuevoProducto` campo por campo: así un `id` o cualquier campo extra enviado por el cliente no entra en la lista.
- 400 indica que el body no es JSON; 422, que es JSON pero con datos incorrectos. Express y Zod usarán la misma distinción en la clase 09.

En el `switch`, la ruta usa `await` porque leer el body es asíncrono. Por eso la función de `createServer()` debe ser `async`:

```javascript
case "POST /products":
  await crearProducto(req, res);
  break;
```

Si una función `async` del servidor lanza un error que nadie captura, el rechazo no manejado termina el proceso (clase 05) y el servidor se cae. Por eso `crearProducto()` captura el error de `JSON.parse()` y responde 400.

---

## 14. Servidor completo

Reemplaza `server.js` por esta versión consolidada:

```javascript
import http from "node:http";

const productos = [
  { id: 1, nombre: "Laptop", precio: 15000, stock: 5 },
  { id: 2, nombre: "Mouse", precio: 450, stock: 12 },
  { id: 3, nombre: "Monitor", precio: 4200, stock: 3 },
];
let siguienteId = 4;

function enviarJson(res, status, datos) {
  res.writeHead(status, { "Content-Type": "application/json; charset=utf-8" });
  res.end(JSON.stringify(datos));
}

async function leerJson(req) {
  const partes = [];
  for await (const parte of req) {
    partes.push(parte);
  }
  const texto = Buffer.concat(partes).toString("utf8");
  return JSON.parse(texto);
}

function validarProducto(datos) {
  const errores = [];
  if (typeof datos !== "object" || datos === null || Array.isArray(datos)) {
    return ["El cuerpo debe ser un objeto JSON"];
  }
  if (typeof datos.nombre !== "string" || datos.nombre.trim() === "") {
    errores.push("nombre debe ser texto no vacío");
  }
  if (!Number.isFinite(datos.precio) || datos.precio < 0) {
    errores.push("precio debe ser un número mayor o igual a 0");
  }
  if (!Number.isInteger(datos.stock) || datos.stock < 0) {
    errores.push("stock debe ser un entero mayor o igual a 0");
  }
  return errores;
}

function listarProductos(res, url) {
  const minPriceTexto = url.searchParams.get("min_price");
  if (minPriceTexto === null) {
    enviarJson(res, 200, productos);
    return;
  }

  const minPrice = Number(minPriceTexto);
  if (minPriceTexto.trim() === "" || Number.isNaN(minPrice)) {
    enviarJson(res, 400, { error: "min_price debe ser un número" });
    return;
  }
  enviarJson(res, 200, productos.filter((p) => p.precio >= minPrice));
}

async function crearProducto(req, res) {
  let datos;
  try {
    datos = await leerJson(req);
  } catch {
    enviarJson(res, 400, { error: "El cuerpo debe ser JSON válido" });
    return;
  }

  const errores = validarProducto(datos);
  if (errores.length > 0) {
    enviarJson(res, 422, { error: "Datos inválidos", detalles: errores });
    return;
  }

  const nuevoProducto = {
    id: siguienteId,
    nombre: datos.nombre.trim(),
    precio: datos.precio,
    stock: datos.stock,
  };
  siguienteId += 1;
  productos.push(nuevoProducto);
  enviarJson(res, 201, nuevoProducto);
}

const server = http.createServer(async (req, res) => {
  const url = new URL(req.url, "http://localhost");
  const ruta = `${req.method} ${url.pathname}`;
  console.log(ruta);

  switch (ruta) {
    case "GET /":
      enviarJson(res, 200, { mensaje: "Mi primera API con Node.js" });
      break;
    case "GET /health":
      enviarJson(res, 200, { status: "ok" });
      break;
    case "GET /products":
      listarProductos(res, url);
      break;
    case "POST /products":
      await crearProducto(req, res);
      break;
    default:
      enviarJson(res, 404, { error: "Ruta no encontrada" });
  }
});

server.listen(8000, "localhost", () => {
  console.log("Servidor corriendo en http://localhost:8000");
});
```

Prueba el `POST` con `curl`. En `-d` va el body y en `-H` una cabecera:

```bash
curl -i -X POST http://localhost:8000/products -H "Content-Type: application/json" -d '{"nombre": "Webcam", "precio": 800, "stock": 6}'
```

```text
HTTP/1.1 201 Created
Content-Type: application/json; charset=utf-8
...

{"id":4,"nombre":"Webcam","precio":800,"stock":6}
```

Prueba también los errores:

```bash
curl -i -X POST http://localhost:8000/products -H "Content-Type: application/json" -d '{malo'
curl -i -X POST http://localhost:8000/products -H "Content-Type: application/json" -d '{"nombre": "", "precio": "800", "stock": 1.5}'
```

```text
400 → {"error":"El cuerpo debe ser JSON válido"}
422 → {"error":"Datos inválidos","detalles":["nombre debe ser texto no vacío","precio debe ser un número mayor o igual a 0","stock debe ser un entero mayor o igual a 0"]}
```

En Windows PowerShell las comillas simples de `-d` se interpretan distinto. Si tienes problemas, prueba el `POST` desde un programa con `fetch`:

```javascript
const response = await fetch("http://localhost:8000/products", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ nombre: "Webcam", precio: 800, stock: 6 }),
  signal: AbortSignal.timeout(10_000),
});

console.log(response.status);
console.log(await response.json());
```

```text
201
{ id: 4, nombre: 'Webcam', precio: 800, stock: 6 }
```

Reinicia el servidor y consulta `/products`: el producto creado desapareció. Los datos viven en memoria; en la clase 10 los guardaremos en SQLite.

---

## 15. ¿Qué está haciendo nuestro servidor?

```text
Recibir request
      ↓
Separar método, ruta y query
      ↓
Elegir la función según "MÉTODO /ruta"
      ↓
Leer y convertir el body (POST)
      ↓
Validar los datos
      ↓
Crear la respuesta
      ↓
JSON.stringify()
      ↓
Enviar status y headers (writeHead)
      ↓
Enviar body (end)
```

---

## 16. ¿Qué problema aparece?

Con cuatro rutas ya escribimos a mano:

```text
routing por método y ruta
conversión de query parameters
lectura del body por partes
JSON inválido → 400
validación de cada campo → 422
status codes y headers
JSON.stringify en cada respuesta
```

Y faltan cosas: rutas con parámetros como `/products/3`, límite de tamaño del body, `DELETE`, `PUT`, manejo central de errores inesperados. Con 20 o 50 rutas, el `switch` y las validaciones manuales se vuelven difíciles de mantener.

---

## 17. Puente a Express y Zod

Es el mismo salto que dimos de `http.server` a FastAPI en Python.

Con nuestro servidor manual:

```javascript
switch (ruta) {
  case "GET /products":
    listarProductos(res, url);
    break;
  // ...
}
```

Con Express veremos:

```javascript
app.get("/products", (req, res) => {
  res.json(productos);
});
```

En lugar de `leerJson()`, usaremos `express.json()`. En lugar de `validarProducto()`, un esquema de Zod, el equivalente de un modelo Pydantic:

```text
routing          → Express
body JSON        → express.json()
JSON en respuesta → res.json()
validación       → Zod
status codes     → los seguimos eligiendo nosotros
```

Una diferencia con FastAPI: Express no genera documentación `/docs` automáticamente. Seguiremos probando con `curl` y con un cliente `fetch`.

---

## Ejercicio final: Mini Products API

Crea `mini_api/server.js` con `node:http` y estas rutas:

```text
GET /
GET /health
GET /products
GET /users
GET /info
```

### `/`

```json
{
  "mensaje": "Bienvenido a mi API"
}
```

### `/health`

```json
{
  "status": "ok"
}
```

### `/products`

Debe devolver al menos 5 productos con `id`, `nombre`, `precio` y `stock`. Puedes usar los datos del curso: Laptop, Mouse, Teclado, Monitor, Audifonos y Webcam. Debe aceptar el filtro opcional `?min_price=` de la sección 10 y responder 400 si no es un número.

### `/users`

Debe devolver al menos 5 usuarios con `id` y `nombre`.

### `/info`

```json
{
  "nombre": "Products API",
  "version": "1.0",
  "lenguaje": "JavaScript (Node.js)"
}
```

Cualquier otra combinación de método y ruta debe responder `404` con:

```json
{
  "error": "Ruta no encontrada"
}
```

Requisitos:

1. Usa un helper `enviarJson(res, status, datos)`.
2. Envía `Content-Type: application/json; charset=utf-8` en todas las respuestas.
3. Usa `new URL(req.url, "http://localhost")` para ignorar la query al elegir la ruta.
4. Escucha en `localhost`, puerto 8000.
5. Comprueba cada ruta con el navegador, `curl -i` y `fetch`.

---

## Cliente del ejercicio

Crea `mini_api/client.js`. Debe consultar `/products` y mostrar:

```text
PRODUCTOS

1 - Laptop - $15000
2 - Mouse - $450
3 - Teclado - $900
...
```

Además:

- usar `AbortSignal.timeout(10_000)`;
- comprobar `response.ok` y lanzar un error si es `false`;
- mostrar `No fue posible conectar con la API.` si el servidor no está activo (`TypeError`).

---

## Práctica opcional

1. Haz que `client.js` consulte primero `/health`. Solo si recibe `{"status": "ok"}`, consulta `/products`.
2. Agrega `POST /products` con la lectura del body y la validación de la sección 13. Pruébalo con `curl` y con `fetch`, incluidos un JSON mal formado y campos inválidos.
3. Agrega un filtro `?in_stock=true` que devuelva solo productos con `stock` mayor a 0. Combínalo con `min_price`.

---

## Errores comunes viniendo de Python

1. **Olvidar `res.end()`.** No hay un retorno que termine la respuesta: sin `end()`, el cliente se queda esperando.
2. **Responder dos veces.** Llamar a `enviarJson()` dos veces en la misma petición lanza `ERR_HTTP_HEADERS_SENT`. Usa `return` después de responder en una rama.
3. **No comprobar el método.** No existe `do_GET()`: la misma función recibe `GET`, `POST` y `DELETE`. Compara `req.method` junto con la ruta.
4. **Comparar `req.url` con la ruta.** `req.url` incluye la query: `"/health?x=1" === "/health"` es `false`. Usa `url.pathname`.
5. **Pasar un objeto a `res.end()`.** Solo acepta texto o bytes. Convierte con `JSON.stringify()`.
6. **Tratar los query parameters como números.** `searchParams.get()` devuelve texto o `null`; `Number("")` es `0` y `Number("abc")` es `NaN`, sin excepción.
7. **Bloquear el event loop.** Un ciclo largo o una lectura síncrona dentro de una petición detiene a todos los clientes.
8. **Dejar escapar errores en funciones `async`.** Un error no capturado dentro del handler termina el proceso y el servidor deja de responder.

---

## Cheat sheet

| Tarea | Python (`http.server`) | Node.js (`node:http`) |
|---|---|---|
| Importar | `from http.server import ...` | `import http from "node:http";` |
| Crear servidor | `HTTPServer(("localhost", 8000), Handler)` | `http.createServer((req, res) => { ... })` |
| Escuchar | `server.serve_forever()` | `server.listen(8000, "localhost", () => { ... })` |
| Método | `do_GET()`, `do_POST()` | `req.method` |
| Ruta | `urlsplit(self.path).path` | `new URL(req.url, "http://localhost").pathname` |
| Query | `parse_qs(urlsplit(self.path).query)` | `url.searchParams.get("min_price")` |
| Status | `self.send_response(404)` | `res.statusCode = 404` |
| Header | `self.send_header("Content-Type", ...)` | `res.setHeader("Content-Type", ...)` |
| Status + headers | `send_response()` + `send_header()` + `end_headers()` | `res.writeHead(404, { "Content-Type": ... })` |
| Body | `self.wfile.write(body)` | `res.end(texto)` |
| Datos → JSON | `json.dumps(data).encode("utf-8")` | `JSON.stringify(datos)` |
| Leer body | `self.rfile.read(length)` | `for await (const parte of req)` + `Buffer.concat()` |
| Bytes → texto | `body.decode("utf-8")` | `buffer.toString("utf8")` |

```javascript
function enviarJson(res, status, datos) {
  res.writeHead(status, { "Content-Type": "application/json; charset=utf-8" });
  res.end(JSON.stringify(datos));
}
```

```bash
curl http://localhost:8000/products
curl -i http://localhost:8000/health
curl -i -X POST http://localhost:8000/products -H "Content-Type: application/json" -d '{"nombre": "Webcam", "precio": 800, "stock": 6}'
```

---

## Flujo completo

```text
Cliente (navegador, curl, fetch)
        ↓
HTTP Request
        ↓
http.createServer((req, res) => ...)
        ↓
new URL(req.url) → pathname, searchParams
        ↓
switch ("MÉTODO /ruta")
        ↓
función de la ruta
        ↓
enviarJson(res, status, datos)
        ↓
HTTP Response
        ↓
Cliente
```

---

## Siguiente clase

En la clase 09 reconstruiremos esta API con **Express** y validaremos los datos con **Zod**. Agregaremos `GET /products/:id`, `DELETE` y errores 422 con el mismo formato que usaba FastAPI.

---

[Índice del curso](README.md)
