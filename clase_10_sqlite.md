# Clase 10: Persistencia con SQLite y `node:sqlite`

**Duración:** 2 horas

> Los bloques de desarrollo se leen en orden dentro de cada ejemplo. Las plantillas con `...` se completan en las actividades; las cheat sheets reúnen operaciones independientes. Los programas completos incluyen sus imports.

**Objetivo:** conservar los productos de nuestra API después de reiniciar, usando SQLite con el módulo `node:sqlite` incorporado en Node. Esta clase es obligatoria antes del proyecto final.

### Objetivos

- Abrir una base SQLite desde Node sin instalar paquetes.
- Escribir las sentencias SQL mínimas: crear una tabla, insertar, consultar, filtrar y eliminar.
- Usar parámetros `?` para evitar inyección SQL.
- Separar el acceso a datos en un módulo `database.js`.
- Comprobar que los registros y las eliminaciones sobreviven a un reinicio.
- Guardar estructuras de IA validadas como texto JSON.

---

## 1. De datos en memoria a persistencia

En la clase 09, reiniciar el servidor restablecía la lista y el contador. Ahora los datos se guardan en un archivo:

```text
Cliente / fetch
        ↓
Express
        ↓
Esquema de entrada / Zod
        ↓
database.js
        ↓
node:sqlite
        ↓
SQLite / products.db
```

En el curso de Python (clase 12) usamos SQLModel, un ORM que traducía objetos a SQL. Aquí tomaremos otro camino:

| En Python | En JavaScript | Responsabilidad |
|---|---|---|
| SQLModel + SQLAlchemy | `node:sqlite` | Abrir la base y ejecutar consultas. |
| Modelos con `table=True` | `CREATE TABLE` en SQL | Describir las columnas. |
| Pydantic | Zod | Validar datos de entrada antes de guardarlos. |
| SQLite | SQLite | Motor que guarda los registros en un archivo. |

`node:sqlite` viene incluido en Node 24: no hay nada que instalar. No es un ORM, así que escribiremos un poco de **SQL**, el lenguaje de consultas de las bases de datos relacionales. Solo necesitamos cinco sentencias, y conocerlas te servirá con cualquier base de datos o ORM que uses después.

Recordatorio: una **tabla** agrupa registros del mismo tipo. Cada fila es un producto y cada columna es un campo, como `nombre` o `precio`. La **clave primaria** identifica una fila de forma única.

> En Node 24, `node:sqlite` tiene estabilidad 1.2 (*release candidate*): ya no muestra advertencias, pero su API todavía puede recibir ajustes menores. Si usas otra versión de Node, revisa su documentación.

## 2. Preparar el proyecto

Reutiliza `products_api/`, su `package.json` y el cliente de la clase 09. Conservaremos `/`, `/health`, los filtros, `descripcion` y los mismos errores 400, 404 y 422. PUT sigue siendo opcional.

```text
products_api/
├── package.json
├── schemas.js
├── database.js      ← nuevo
├── server.js
├── client.js
└── products.db      ← se crea al iniciar el servidor
```

La base inicia vacía: los productos de la antigua lista no se importan automáticamente. Créalos con POST para practicar la persistencia.

Agrega la base de datos a `.gitignore`, junto a `node_modules/` y `.env`:

```text
node_modules/
.env
*.db
```

## 3. Primeros pasos con `node:sqlite`

Antes de tocar la API, practica en un archivo independiente, `sqlite_demo.js`. Usa la base especial `":memory:"`, que vive en memoria y desaparece al terminar el programa:

```javascript
import { DatabaseSync } from "node:sqlite";

const db = new DatabaseSync(":memory:");

db.exec(`
  CREATE TABLE products (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT NOT NULL,
    precio REAL NOT NULL,
    stock INTEGER NOT NULL
  )
`);

const insertar = db.prepare(
  "INSERT INTO products (nombre, precio, stock) VALUES (?, ?, ?)",
);
console.log(insertar.run("Laptop", 15000, 5));
insertar.run("Mouse", 450, 12);
insertar.run("Audifonos", 1200, 0);

const todos = db.prepare("SELECT * FROM products").all();
console.log(todos);

const buscar = db.prepare("SELECT * FROM products WHERE id = ?");
console.log(buscar.get(2));
console.log(buscar.get(99));

const disponibles = db
  .prepare("SELECT nombre, precio FROM products WHERE stock > 0 ORDER BY precio DESC")
  .all();
console.log(JSON.stringify(disponibles));

console.log(db.prepare("DELETE FROM products WHERE id = ?").run(3));
console.log(db.prepare("DELETE FROM products WHERE id = ?").run(3));

db.close();
```

Resultado (las filas aparecen en varias líneas en tu terminal; aquí se muestran compactas):

```text
{ changes: 1, lastInsertRowid: 1 }
[
  [Object: null prototype] { id: 1, nombre: 'Laptop', precio: 15000, stock: 5 },
  [Object: null prototype] { id: 2, nombre: 'Mouse', precio: 450, stock: 12 },
  [Object: null prototype] { id: 3, nombre: 'Audifonos', precio: 1200, stock: 0 }
]
[Object: null prototype] { id: 2, nombre: 'Mouse', precio: 450, stock: 12 }
undefined
[{"nombre":"Laptop","precio":15000},{"nombre":"Mouse","precio":450}]
{ changes: 1, lastInsertRowid: 3 }
{ changes: 0, lastInsertRowid: 3 }
```

| Operación | Efecto |
|---|---|
| `new DatabaseSync(ruta)` | Abre el archivo de base de datos; lo crea si no existe. |
| `db.exec(sql)` | Ejecuta SQL sin parámetros ni resultados, como crear una tabla. |
| `db.prepare(sql)` | Prepara una sentencia que puede ejecutarse varias veces. |
| `sentencia.run(...valores)` | Ejecuta INSERT, UPDATE o DELETE. Devuelve `{ changes, lastInsertRowid }`. |
| `sentencia.get(...valores)` | Devuelve la primera fila como objeto o `undefined`. |
| `sentencia.all(...valores)` | Devuelve todas las filas en un array, vacío si no hay ninguna. |
| `db.close()` | Cierra la base. |

- `changes` indica cuántas filas cambiaron. El segundo DELETE devuelve `changes: 0` porque la fila ya no existía: así sabremos cuándo responder 404.
- `lastInsertRowid` es el ID que SQLite generó en el último INSERT.
- `get()` devuelve `undefined` si no hay resultados, como `session.get()` devolvía `None`.
- Las filas son objetos con **prototipo nulo**: no heredan métodos como `toString()`. Por eso `console.log()` muestra `[Object: null prototype]`. Para nosotros se comportan como objetos normales: puedes leer `fila.nombre`, y `JSON.stringify()` y `res.json()` funcionan igual.

## 4. SQL mínimo

| Sentencia | Para qué sirve | Equivalente en SQLModel |
|---|---|---|
| `CREATE TABLE IF NOT EXISTS products (...)` | Crea la tabla si todavía no existe. | `SQLModel.metadata.create_all(engine)` |
| `INSERT INTO products (col1, col2) VALUES (?, ?)` | Agrega una fila. | `session.add()` + `commit()` |
| `SELECT * FROM products` | Obtiene todas las filas y columnas. | `session.exec(select(Product)).all()` |
| `SELECT * FROM products WHERE id = ?` | Obtiene las filas que cumplen la condición. | `session.get(Product, id)` |
| `... WHERE precio >= ? AND stock > 0` | Combina condiciones. | `.where(...)` encadenado |
| `... ORDER BY id` | Ordena el resultado. | `.order_by(Product.id)` |
| `DELETE FROM products WHERE id = ?` | Elimina filas. | `session.delete()` + `commit()` |
| `UPDATE products SET nombre = ? WHERE id = ?` | Modifica filas. | Cambiar atributos + `commit()` |

Las palabras de SQL se escriben en mayúsculas por convención; SQLite no distingue mayúsculas en ellas. Los nombres de tablas y columnas son los que tú elegiste.

Tipos de columna que usaremos:

| SQLite | JavaScript |
|---|---|
| `INTEGER` | `number` entero |
| `REAL` | `number` decimal |
| `TEXT` | `string` |
| `NULL` | `null` |

`NOT NULL` impide guardar una fila sin ese valor. `INTEGER PRIMARY KEY AUTOINCREMENT` hace que SQLite genere el ID y que **nunca reutilice** el de una fila eliminada, igual que nuestro contador de la clase 09. Sin `AUTOINCREMENT`, SQLite podría volver a usar el ID más alto después de eliminarlo; ese era el caso de SQLModel en el curso de Python.

SQLite es flexible con los tipos: en ciertos casos acepta guardar texto en una columna `INTEGER`. Por eso seguimos validando con Zod **antes** de insertar.

## 5. Parámetros `?` e inyección SQL

Los valores siempre se entregan como argumentos de `run()`, `get()` o `all()`, en el lugar de cada `?`. **Nunca** los insertes en el SQL con template literals. Compruébalo con `inyeccion.js`:

```javascript
import { DatabaseSync } from "node:sqlite";

const db = new DatabaseSync(":memory:");
db.exec("CREATE TABLE products (id INTEGER PRIMARY KEY, nombre TEXT)");
db.exec("INSERT INTO products (nombre) VALUES ('Laptop'), ('Mouse'), ('Monitor')");

const nombre = "x' OR '1'='1";

// MAL: el texto del usuario se mezcla con el SQL.
const peligrosa = db.prepare(`SELECT * FROM products WHERE nombre = '${nombre}'`);
console.log("Interpolando:", peligrosa.all().length);

// BIEN: el valor viaja aparte, en el lugar de ?.
const segura = db.prepare("SELECT * FROM products WHERE nombre = ?");
console.log("Con parámetro:", segura.all(nombre).length);

db.close();
```

Resultado:

```text
Interpolando: 3
Con parámetro: 0
```

Al interpolar, el texto `x' OR '1'='1` cierra la comilla y añade una condición siempre verdadera: la consulta devuelve todos los productos. Con un DELETE, podría borrar la tabla completa. Esto se llama **inyección SQL**. Con `?`, SQLite trata el valor como dato, nunca como parte de la sentencia: busca un producto con ese nombre literal y no encuentra ninguno.

SQLModel generaba los parámetros por nosotros. Sin ORM, esta regla es responsabilidad nuestra.

## 6. Una API síncrona

`DatabaseSync` es **síncrona**: `run()`, `get()` y `all()` devuelven el resultado directamente, sin `await`. En la clase 05 vimos que bloquear el event loop detiene todas las peticiones. ¿Por qué es aceptable aquí?

- SQLite es un archivo local, no un servidor en la red: no hay espera de red.
- Las consultas de esta clase tardan microsegundos con pocos miles de filas.
- El código es más simple y no mantiene operaciones a medias entre peticiones.

Para consultas largas o tablas muy grandes, existen bibliotecas asíncronas y servidores de bases de datos. Para nuestra API y el proyecto final, la versión síncrona es suficiente. En cambio, la consulta a OpenRouter sí es una operación de red y sigue siendo asíncrona.

## 7. `database.js`: archivo y tabla

Concentraremos todo el SQL en un módulo. Crea `database.js`:

```javascript
import path from "node:path";
import { DatabaseSync } from "node:sqlite";

const dbPath = path.join(import.meta.dirname, "products.db");
const db = new DatabaseSync(dbPath);

db.exec(`
  CREATE TABLE IF NOT EXISTS products (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT NOT NULL,
    precio REAL NOT NULL,
    stock INTEGER NOT NULL,
    descripcion TEXT
  )
`);
```

- `import.meta.dirname` es la carpeta de este módulo (clase 06), como `Path(__file__).parent` en Python. Así `products.db` queda junto a `database.js`, aunque la terminal esté en otra carpeta.
- El módulo se ejecuta una sola vez, la primera vez que alguien lo importa. La conexión `db` se comparte entre todas las peticiones; no la abrimos en cada ruta.
- `CREATE TABLE IF NOT EXISTS` crea la tabla la primera vez y no hace nada en los reinicios: conserva los registros. No actualiza columnas de una tabla existente. Si cambias el esquema, prueba con una base nueva o usa otro nombre de archivo; no borres una base con datos que quieras guardar.
- `descripcion` no tiene `NOT NULL`: acepta `null`.
- `db` no se exporta. El resto del programa solo usará las funciones del módulo.

### Confirmación de los cambios

En SQLModel necesitábamos `session.commit()` para confirmar las escrituras. Con `node:sqlite`, cada `run()` se confirma **inmediatamente**: cuando la función devuelve, el cambio ya está en el archivo. No hay sesión ni `commit()` que olvidar.

Si alguna vez necesitas que varias sentencias se confirmen juntas o ninguna, se agrupan en una transacción con `db.exec("BEGIN")`, `db.exec("COMMIT")` y `db.exec("ROLLBACK")`. En esta clase cada operación es una sola sentencia, así que no las necesitamos.

## 8. Funciones de acceso

Añade estas funciones al final de `database.js`.

### Crear

```javascript
export function crearProducto({ nombre, precio, stock, descripcion }) {
  const resultado = db
    .prepare(
      "INSERT INTO products (nombre, precio, stock, descripcion) VALUES (?, ?, ?, ?)",
    )
    .run(nombre, precio, stock, descripcion);
  return obtenerProducto(Number(resultado.lastInsertRowid));
}
```

La función recibe los datos ya validados por Zod y los desestructura en el parámetro (clase 04). Después del INSERT, consulta la fila con su ID nuevo y la devuelve completa, como `session.refresh()`. `Number()` asegura un número: según la configuración, `lastInsertRowid` también puede ser un `BigInt`, un tipo para enteros muy grandes que `res.json()` no sabe convertir.

`descripcion` debe ser texto o `null`; `node:sqlite` no acepta `undefined` como valor. El `.default(null)` del esquema garantiza que nunca llegue `undefined`.

### Listar con filtros

```javascript
export function listarProductos({ min_price: minPrice, in_stock: inStock }) {
  const condiciones = [];
  const parametros = [];

  if (minPrice !== undefined) {
    condiciones.push("precio >= ?");
    parametros.push(minPrice);
  }
  if (inStock) {
    condiciones.push("stock > 0");
  }

  let sql = "SELECT * FROM products";
  if (condiciones.length > 0) {
    sql += ` WHERE ${condiciones.join(" AND ")}`;
  }
  sql += " ORDER BY id";

  return db.prepare(sql).all(...parametros);
}
```

La función recibe directamente `filtros.data` de la ruta y renombra sus claves al desestructurar (clase 03). Construimos la consulta por partes:

| Filtros | SQL resultante | Parámetros |
|---|---|---|
| Ninguno | `SELECT * FROM products ORDER BY id` | `[]` |
| `min_price=1000` | `SELECT * FROM products WHERE precio >= ? ORDER BY id` | `[1000]` |
| Ambos | `... WHERE precio >= ? AND stock > 0 ORDER BY id` | `[1000]` |

Aquí sí usamos un template literal, pero solo une fragmentos **que escribimos nosotros**. Los valores del usuario siguen viajando en `parametros`, y `...parametros` los entrega a `all()` como argumentos separados (spread, clase 04). La base de datos hace el filtrado; ya no recorremos un array global.

`ORDER BY id` fija el orden. Sin él, SQL no garantiza ningún orden específico.

### Obtener y eliminar

```javascript
export function obtenerProducto(id) {
  return db.prepare("SELECT * FROM products WHERE id = ?").get(id);
}

export function eliminarProducto(id) {
  const resultado = db.prepare("DELETE FROM products WHERE id = ?").run(id);
  return resultado.changes > 0;
}
```

`obtenerProducto()` devuelve el objeto o `undefined`. `eliminarProducto()` devuelve `true` si borró una fila y `false` si el ID no existía; la ruta decidirá entre 200 y 404. Así la capa de datos no conoce HTTP y el servidor no conoce SQL.

### `database.js` completo

```javascript
import path from "node:path";
import { DatabaseSync } from "node:sqlite";

const dbPath = path.join(import.meta.dirname, "products.db");
const db = new DatabaseSync(dbPath);

db.exec(`
  CREATE TABLE IF NOT EXISTS products (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT NOT NULL,
    precio REAL NOT NULL,
    stock INTEGER NOT NULL,
    descripcion TEXT
  )
`);

export function crearProducto({ nombre, precio, stock, descripcion }) {
  const resultado = db
    .prepare(
      "INSERT INTO products (nombre, precio, stock, descripcion) VALUES (?, ?, ?, ?)",
    )
    .run(nombre, precio, stock, descripcion);
  return obtenerProducto(Number(resultado.lastInsertRowid));
}

export function listarProductos({ min_price: minPrice, in_stock: inStock }) {
  const condiciones = [];
  const parametros = [];

  if (minPrice !== undefined) {
    condiciones.push("precio >= ?");
    parametros.push(minPrice);
  }
  if (inStock) {
    condiciones.push("stock > 0");
  }

  let sql = "SELECT * FROM products";
  if (condiciones.length > 0) {
    sql += ` WHERE ${condiciones.join(" AND ")}`;
  }
  sql += " ORDER BY id";

  return db.prepare(sql).all(...parametros);
}

export function obtenerProducto(id) {
  return db.prepare("SELECT * FROM products WHERE id = ?").get(id);
}

export function eliminarProducto(id) {
  const resultado = db.prepare("DELETE FROM products WHERE id = ?").run(id);
  return resultado.changes > 0;
}
```

## 9. Código completo de `server.js`

`schemas.js` no cambia. Reemplaza `server.js` con esta versión. No conserves la lista global, el contador ni rutas duplicadas.

```javascript
import express from "express";
import * as z from "zod";
import {
  crearProducto,
  eliminarProducto,
  listarProductos,
  obtenerProducto,
} from "./database.js";
import { ProductFilters, ProductId, ProductInput } from "./schemas.js";

const PORT = Number(process.env.PORT ?? 8000);

const app = express();
app.use(express.json());

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
  res.json(listarProductos(filtros.data));
});

app.get("/products/:id", (req, res) => {
  const id = ProductId.safeParse(req.params.id);
  if (!id.success) {
    return responderError422(res, id.error);
  }

  const producto = obtenerProducto(id.data);
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
  res.status(201).json(crearProducto(entrada.data));
});

app.delete("/products/:id", (req, res) => {
  const id = ProductId.safeParse(req.params.id);
  if (!id.success) {
    return responderError422(res, id.error);
  }

  if (!eliminarProducto(id.data)) {
    return responderNoEncontrado(res);
  }
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

Las rutas conservan la misma semántica de la clase 09: validar primero, consultar después. Una entrada inválida responde 422 **antes** de llegar a la base de datos, así que nunca se guarda. Si una operación de la base falla de forma inesperada, el error llega al middleware final, se registra y el cliente recibe 500: nunca anunciamos éxito.

Inicia el servidor con `npm run dev` y envía:

```bash
curl -X POST http://localhost:8000/products \
  -H "Content-Type: application/json" \
  -d '{"nombre": "Laptop", "precio": 15000, "stock": 5, "descripcion": "Equipo de trabajo"}'
```

Respuesta:

```json
{"id":1,"nombre":"Laptop","precio":15000,"stock":5,"descripcion":"Equipo de trabajo"}
```

Prueba `{}`, precio `"abc"`, valores negativos y un `id` enviado en el body. Todos deben devolver 422 y no guardar registros. Prueba también los filtros de la clase 09.

## Actividad: Comprobar persistencia

1. Crea Laptop, Mouse y un producto con stock cero desde el cliente o con `curl`.
2. Lista los productos con y sin filtros.
3. Elimina el producto con stock cero y consulta su ID para observar 404.
4. Crea otro producto y comprueba que su ID es **mayor** que el eliminado: `AUTOINCREMENT` no lo reutiliza.
5. Detén el servidor con `Ctrl+C` e inícialo de nuevo.
6. Lista los productos: deben seguir ahí, y el eliminado debe seguir ausente.
7. Ejecuta `node client.js`: el cliente de la clase 09 funciona sin cambios, porque la interfaz HTTP es la misma.

> `node --watch` reinicia el servidor en cada cambio. Antes, eso borraba los productos creados; ahora deben conservarse.

## 10. PUT opcional: actualización persistente

Añade esta función a `database.js`:

```javascript
export function actualizarProducto(id, { nombre, precio, stock, descripcion }) {
  const resultado = db
    .prepare(
      "UPDATE products SET nombre = ?, precio = ?, stock = ?, descripcion = ? WHERE id = ?",
    )
    .run(nombre, precio, stock, descripcion, id);
  if (resultado.changes === 0) {
    return undefined;
  }
  return obtenerProducto(id);
}
```

Agrega `actualizarProducto` al import de `server.js` y esta ruta antes de los middleware finales:

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

  const actualizado = actualizarProducto(id.data, entrada.data);
  if (actualizado === undefined) {
    return responderNoEncontrado(res);
  }
  res.json(actualizado);
});
```

Los `?` se reemplazan en orden: los cuatro campos de `SET` y al final el `id` del `WHERE`. Un UPDATE **sin** `WHERE` modificaría todas las filas; revisa siempre esa parte. Si ninguna fila coincide, `changes` es 0 y respondemos 404. PUT sustituye todos los campos modificables y conserva el ID; `descripcion` se vuelve `null` si se omite. Reinicia el servidor y comprueba que el cambio persiste.

## 11. Guardar estructuras de IA como JSON

El proyecto final necesita persistir un array de tarjetas. Una columna de SQLite no guarda arrays ni objetos. Para evitar relaciones entre tablas en este curso, guardaremos el resultado completo, ya validado, como **texto JSON** en una columna `TEXT`, igual que en Python.

Esta demostración es un archivo independiente, `json_storage.js`, con su propia tabla y archivo de base de datos:

```javascript
import path from "node:path";
import { DatabaseSync } from "node:sqlite";

const db = new DatabaseSync(path.join(import.meta.dirname, "json_demo.db"));

db.exec(`
  CREATE TABLE IF NOT EXISTS saved_decks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    topic TEXT NOT NULL,
    count INTEGER NOT NULL,
    level TEXT NOT NULL,
    result_json TEXT NOT NULL
  )
`);

// Resultado fijo para practicar el almacenamiento; la API real lo validará
// con validarMazo() antes de guardarlo.
const resultado = {
  summary: "Ideas básicas de la fotosíntesis",
  cards: [
    { question: "¿Qué produce la fotosíntesis?", answer: "Glucosa y oxígeno" },
    { question: "¿Dónde ocurre?", answer: "En los cloroplastos" },
  ],
  study_minutes: 15,
};

const info = db
  .prepare(
    "INSERT INTO saved_decks (topic, count, level, result_json) VALUES (?, ?, ?, ?)",
  )
  .run("Fotosíntesis", 3, "básico", JSON.stringify(resultado));
const deckId = Number(info.lastInsertRowid);
console.log("ID guardado:", deckId);

const guardado = db.prepare("SELECT * FROM saved_decks WHERE id = ?").get(deckId);
if (guardado === undefined) {
  throw new Error("No se encontró el mazo guardado");
}

const reconstruido = JSON.parse(guardado.result_json);
console.log(typeof guardado.result_json, typeof reconstruido);
for (const tarjeta of reconstruido.cards) {
  console.log(`${tarjeta.question} → ${tarjeta.answer}`);
}

db.close();
```

Resultado de la primera ejecución:

```text
ID guardado: 1
string object
¿Qué produce la fotosíntesis? → Glucosa y oxígeno
¿Dónde ocurre? → En los cloroplastos
```

Cada ejecución agrega un registro nuevo y el ID aumenta. `JSON.stringify()` convierte el objeto en texto para guardarlo; `JSON.parse()` reconstruye el objeto al leerlo (clase 03). A diferencia de `json.dumps()`, `JSON.stringify()` conserva los acentos sin necesidad de `ensure_ascii=False`.

La tabla guarda la consulta y el resultado juntos: `topic`, `count` y `level` en sus propias columnas, y el resultado en `result_json`. El listado del historial del proyecto puede leer solo las primeras columnas.

### Conectar generación, validación y guardado

La demostración anterior usa datos fijos. Al integrar OpenRouter, la ruta POST del proyecto sigue este orden:

```text
entrada → DeckInput (422) → OpenRouter (504 / 502) → validarMazo (502) → INSERT → 201
```

El siguiente fragmento va dentro de un handler `async`, después de validar la entrada con `DeckInput` (clase 09) y guardar sus datos en `datos`. `pedirContenidoIA()` representa la función de tu `ai.js` que consulta OpenRouter y devuelve el texto de `choices[0].message.content`:

```javascript
let contenido;
try {
  contenido = await pedirContenidoIA(datos);
} catch (error) {
  if (error.name === "TimeoutError") {
    return res.status(504).json({ detail: "La IA tardó demasiado en responder" });
  }
  return res.status(502).json({ detail: "No fue posible obtener una respuesta de la IA" });
}

let resultado;
try {
  resultado = validarMazo(contenido, datos.count);
} catch {
  return res.status(502).json({ detail: "La IA devolvió un mazo inválido" });
}

// En el proyecto, esta sentencia pertenece a una función de database.js.
const info = db
  .prepare(
    "INSERT INTO saved_decks (topic, count, level, result_json) VALUES (?, ?, ?, ?)",
  )
  .run(datos.topic, datos.count, datos.level, JSON.stringify(resultado));
```

- `return` dentro de cada `catch` termina el handler **antes** del INSERT: una consulta fallida o un resultado inválido no crea registros.
- `AbortSignal.timeout(30_000)` en el `fetch` de `pedirContenidoIA()` produce un error con `name === "TimeoutError"`, que corresponde a 504. A diferencia del `timeout` de `requests`, este límite cubre toda la operación, incluida la lectura del cuerpo.
- Cualquier otro error al consultar es 502: red caída (`TypeError`), código HTTP de error (tu `HttpError` de la clase 07) o una respuesta sin `choices` o sin contenido. Para lo último, `pedirContenidoIA()` debe comprobar que el contenido sea texto y lanzar un error si no lo es; el encadenamiento opcional `datos.choices?.[0]?.message?.content` evita un `TypeError` al recorrer una estructura incompleta.
- `catch` sin paréntesis es válido cuando no necesitas el error. `validarMazo()` puede lanzar `SyntaxError`, `ZodError` o `RangeError`; los tres significan lo mismo para el cliente.
- El JSON guardado proviene del objeto **validado** por Zod, no directamente del texto del modelo: así se guardan los textos sin espacios exteriores y nunca campos inesperados.

En `/history/:id`, decodifica `result_json` con `JSON.parse()` y devuelve el resultado como objeto JSON; no expongas una cadena con JSON escapado como si fuera el objeto original.

La API key sigue únicamente en el entorno del servidor. No la guardes en ninguna tabla, respuesta o archivo del proyecto. Termina la consulta a OpenRouter y la validación antes de escribir en la base.

---

## Errores comunes viniendo de Python

| Error | Qué ocurre | Solución |
|---|---|---|
| Interpolar valores en el SQL con `${}` | Inyección SQL y errores con comillas en los textos. | Usar `?` y pasar los valores a `run()`, `get()` o `all()`. |
| Buscar un `commit()` | No existe: cada `run()` se confirma solo. | Agrupar con `BEGIN`/`COMMIT` solo si varias sentencias deben ir juntas. |
| Usar `await` con `DatabaseSync` | No hace falta: los métodos devuelven el resultado directamente. | Llamar `get()`, `all()` y `run()` sin `await`. |
| Comparar el resultado de `get()` con `null` | Sin resultados devuelve `undefined`, no `null`. | `if (producto === undefined)`. |
| Pasar `undefined` como valor | `node:sqlite` lanza un error al enlazar el parámetro. | Usar `null` para valores ausentes, como hace `.default(null)`. |
| Guardar un array u objeto directamente | Una columna no guarda estructuras. | `JSON.stringify()` al guardar y `JSON.parse()` al leer. |
| Olvidar `WHERE` en un DELETE o UPDATE | Afecta a **todas** las filas. | Revisar el `WHERE` y comprobar `changes`. |
| Crear la base en la carpeta de la terminal | Aparecen varios `products.db` según desde dónde ejecutes. | `path.join(import.meta.dirname, "products.db")`. |

---

## Ejercicio final: Products API persistente

Usa los endpoints de esta clase, conserva las funciones del cliente y verifica filtros, descripción, validación, eliminación y reinicio.

1. La base se crea junto a `database.js` y está excluida de Git.
2. Las rutas no contienen SQL; `database.js` no contiene objetos `req` ni `res`.
3. Todas las consultas usan `?` para los valores.
4. Los errores 400, 404 y 422 se comportan igual que en la clase 09.
5. Los productos creados y las eliminaciones se conservan después de reiniciar.

PUT es opcional; las operaciones de historial del proyecto final solo requieren GET, POST y DELETE.

### Práctica opcional

Agrega `GET /stats` que devuelva la cantidad de productos y el valor total del inventario calculado por SQLite:

```sql
SELECT COUNT(*) AS total, COALESCE(SUM(precio * stock), 0) AS valor FROM products
```

`COUNT(*)` cuenta filas, `SUM()` suma una expresión y `COALESCE()` devuelve 0 cuando la tabla está vacía, porque `SUM()` devolvería `NULL`. Escribe la consulta en una función de `database.js` y usa `get()`, ya que devuelve una sola fila.

---

## Cheat sheet

```javascript
// Fragmentos de consulta: requieren importar DatabaseSync y crear la tabla.
import { DatabaseSync } from "node:sqlite";

const db = new DatabaseSync("products.db");

const info = db
  .prepare("INSERT INTO products (nombre, precio, stock) VALUES (?, ?, ?)")
  .run("Mouse", 450, 3);
const id = Number(info.lastInsertRowid);

const producto = db.prepare("SELECT * FROM products WHERE id = ?").get(id);
const productos = db.prepare("SELECT * FROM products ORDER BY id").all();
const { changes } = db.prepare("DELETE FROM products WHERE id = ?").run(id);
```

| Tarea | Python / SQLModel | JavaScript / `node:sqlite` |
|---|---|---|
| Abrir la base | `create_engine("sqlite:///products.db")` | `new DatabaseSync("products.db")` |
| Crear tablas | `SQLModel.metadata.create_all(engine)` | `db.exec("CREATE TABLE IF NOT EXISTS ...")` |
| Insertar | `session.add(p)` + `commit()` | `db.prepare("INSERT ...").run(...valores)` |
| ID generado | `session.refresh(p)` → `p.id` | `Number(info.lastInsertRowid)` |
| Buscar por ID | `session.get(Product, id)` → objeto o `None` | `.get(id)` → objeto o `undefined` |
| Listar | `session.exec(select(Product)).all()` | `.all()` → array |
| Filtrar | `.where(Product.precio >= x)` | `"... WHERE precio >= ?"` + parámetro |
| Eliminar | `session.delete(p)` + `commit()` | `.run(id)` → `changes` |
| Ruta del archivo | `Path(__file__).resolve().with_name(...)` | `path.join(import.meta.dirname, ...)` |
| Guardar estructura | `json.dumps(datos, ensure_ascii=False)` | `JSON.stringify(datos)` |
| Leer estructura | `json.loads(texto)` | `JSON.parse(texto)` |

Siguiente paso: [Proyecto final: Generador de tarjetas de estudio con historial](proyecto_final.md), integrando la consola, Express, validación de entrada y salida de IA con Zod, y un historial persistente.

---

[Índice del curso](README.md)
