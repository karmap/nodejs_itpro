# Clase 03: Objetos, Map y JSON

**Duración:** 1 a 2 horas

> Los bloques de desarrollo se leen en orden dentro de cada ejemplo. Las plantillas con `...` se completan en las actividades; las cheat sheets reúnen operaciones independientes. Los programas completos incluyen sus imports.

### Objetivos

- Usar objetos de JavaScript donde en Python usábamos diccionarios.
- Leer valores con punto y con corchetes, y manejar claves ausentes con `?.` y `??`.
- Agregar, modificar, eliminar y recorrer propiedades.
- Trabajar con objetos anidados y arrays de objetos.
- Combinar y desestructurar objetos con spread y desestructuración.
- Entender que los objetos se comparan y se copian por referencia.
- Saber cuándo conviene un `Map`.
- Convertir entre objetos y texto JSON con `JSON.stringify()` y `JSON.parse()`.

---

## 1. Del diccionario al objeto

En el curso de Python (clase 04) guardábamos pares clave y valor en un diccionario:

```python
usuario = {
    "nombre": "Ana",
    "edad": 25,
    "pais": "México"
}
```

En JavaScript la estructura equivalente más usada es el **objeto**:

```javascript
const usuario = {
  nombre: "Ana",
  edad: 25,
  pais: "México",
};

console.log(usuario);
```

Resultado:

```text
{ nombre: 'Ana', edad: 25, pais: 'México' }
```

Cada par se llama **propiedad**: `nombre` es la clave y `"Ana"` su valor. Las claves se escriben sin comillas cuando son nombres válidos de variable; si tienen espacios, guiones o empiezan con un número, necesitan comillas:

```javascript
const configuracion = {
  "vida nocturna": true,
  "max-dias": 14,
};
```

La coma después de la última propiedad es opcional. La usamos porque facilita agregar líneas sin olvidar comas.

| En Python | En JavaScript |
|---|---|
| `dict` | Objeto (`{}`) |
| `{"nombre": "Ana"}` | `{ nombre: "Ana" }` |
| Claves de cualquier tipo inmutable | Claves de texto (los números se convierten a texto) |
| `len(usuario)` | `Object.keys(usuario).length` |

> `console.log()` muestra el texto con comillas simples. Es solo la forma de presentarlo en la terminal; el código del curso usa comillas dobles.

---

## 2. Leer valores

Hay dos formas de acceder a una propiedad:

```javascript
console.log(usuario.nombre);
console.log(usuario["nombre"]);
```

El **punto** es la forma habitual. Los **corchetes** se necesitan cuando la clave está en una variable o no es un nombre válido:

```javascript
const campo = "edad";

console.log(usuario[campo]);
console.log(configuracion["vida nocturna"]);
```

`usuario.campo` buscaría una propiedad llamada literalmente `"campo"`, no el valor de la variable.

### Clave ausente: `undefined`, no `KeyError`

En Python, `usuario["telefono"]` lanzaba `KeyError`. En JavaScript no hay error:

```javascript
console.log(usuario.telefono);
```

Resultado:

```text
undefined
```

Esto es cómodo, pero también peligroso: un error de escritura como `usuario.nombr` no detiene el programa, solo produce `undefined`.

### Comprobar si existe una clave

```javascript
console.log("nombre" in usuario);
console.log(Object.hasOwn(usuario, "telefono"));
```

Resultado:

```text
true
false
```

`in` funciona como en Python con diccionarios. `Object.hasOwn()` comprueba solo las propiedades propias del objeto; es la opción recomendada cuando quieres saber si **tú** definiste esa clave.

### Valor por defecto: `??`

En Python usábamos `usuario.get("telefono", "Sin teléfono")`. En JavaScript combinamos el acceso con el operador `??` de la clase 01:

```javascript
const telefono = usuario.telefono ?? "Sin teléfono";

console.log(telefono);
```

`??` devuelve el valor de la derecha solo si el de la izquierda es `null` o `undefined`. No uses `||` para valores por defecto si `0` o `""` son valores válidos:

```javascript
const producto = { nombre: "Audifonos", stock: 0 };

console.log(producto.stock || 10);
console.log(producto.stock ?? 10);
```

Resultado:

```text
10
0
```

`||` reemplazó el `0` porque es falsy; `??` lo conservó.

### Encadenamiento opcional: `?.`

Si una propiedad intermedia puede faltar, `?.` detiene el acceso y devuelve `undefined` en lugar de lanzar un error:

```javascript
const cliente = { nombre: "Luis" };

console.log(cliente.direccion?.ciudad);
console.log(cliente.direccion?.ciudad ?? "Ciudad desconocida");
```

Resultado:

```text
undefined
Ciudad desconocida
```

Sin `?.`, `cliente.direccion.ciudad` lanzaría `TypeError: Cannot read properties of undefined`, porque se intenta leer `ciudad` de `undefined`.

| En Python | En JavaScript |
|---|---|
| `usuario["nombre"]` | `usuario.nombre` o `usuario["nombre"]` |
| `usuario["x"]` → `KeyError` | `usuario.x` → `undefined` |
| `"x" in usuario` | `"x" in usuario` u `Object.hasOwn(usuario, "x")` |
| `usuario.get("x", "defecto")` | `usuario.x ?? "defecto"` |

---

## 3. Agregar, modificar y eliminar

```javascript
const usuario = {
  nombre: "Ana",
  edad: 25,
};

usuario.pais = "México";
usuario.edad = 26;
delete usuario.nombre;

console.log(usuario);
```

Resultado:

```text
{ edad: 26, pais: 'México' }
```

- Asignar a una clave nueva la crea; asignar a una existente la reemplaza.
- `delete objeto.clave` elimina la propiedad. Si no existe, no hay error.
- `const` impide reasignar la variable `usuario`, pero no impide modificar sus propiedades, igual que en la clase 02 con los arrays.

En Python, `pop()` eliminaba la clave y devolvía su valor. En JavaScript se hace en dos pasos:

```javascript
const edad = usuario.edad;
delete usuario.edad;

console.log(edad);
console.log(usuario);
```

---

## 4. Recorrer un objeto

`Object.keys()`, `Object.values()` y `Object.entries()` devuelven **arrays**, así que podemos usar todo lo de la clase 02:

```javascript
const producto = {
  nombre: "Laptop",
  precio: 15000,
  stock: 5,
};

console.log(Object.keys(producto));
console.log(Object.values(producto));
console.log(Object.entries(producto));
console.log(Object.keys(producto).length);
```

Resultado:

```text
[ 'nombre', 'precio', 'stock' ]
[ 'Laptop', 15000, 5 ]
[ [ 'nombre', 'Laptop' ], [ 'precio', 15000 ], [ 'stock', 5 ] ]
3
```

`Object.entries()` devuelve pares `[clave, valor]`. Con `for...of` y la desestructuración de arrays de la clase 02 obtenemos el equivalente de `items()`:

```javascript
for (const [clave, valor] of Object.entries(producto)) {
  console.log(clave, valor);
}
```

```python
for clave, valor in producto.items():
    print(clave, valor)
```

Las propiedades se recorren en el orden en que se agregaron, como en los diccionarios de Python. La excepción son las claves que parecen números enteros, como `"1"` o `"20"`: JavaScript las coloca primero y en orden numérico.

> Existe también `for...in`, que recorre claves. Evítalo por ahora: también puede recorrer propiedades heredadas y en arrays devuelve índices como texto. `Object.keys()` y `Object.entries()` son más predecibles.

| En Python | En JavaScript |
|---|---|
| `usuario.keys()` | `Object.keys(usuario)` |
| `usuario.values()` | `Object.values(usuario)` |
| `usuario.items()` | `Object.entries(usuario)` |
| `for k, v in usuario.items():` | `for (const [k, v] of Object.entries(usuario))` |

---

## Actividad 1: Perfil de usuario

Crea `perfil.js`:

```javascript
const perfil = {
  nombre: "Carla",
  correo: "carla@example.com",
  edad: 31,
};

// Completa durante la actividad.
```

1. Muestra el nombre usando punto y la edad usando corchetes con una variable `campo`.
2. Muestra el teléfono o `"Sin teléfono"` si no existe.
3. Agrega `ciudad: "Monterrey"` y cambia la edad a `32`.
4. Elimina el correo.
5. Recorre el perfil con `Object.entries()` y muestra cada línea como `clave: valor`.
6. Muestra cuántas propiedades tiene el perfil.

Ejecuta con `node perfil.js`.

---

## 5. Objetos anidados y arrays de objetos

Un objeto puede contener otros objetos y arrays:

```javascript
const pedido = {
  id: 101,
  cliente: {
    nombre: "Ana",
    direccion: {
      ciudad: "Guadalajara",
      cp: "44100",
    },
  },
  productos: ["Laptop", "Mouse"],
};

console.log(pedido.cliente.direccion.ciudad);
console.log(pedido.productos[1]);
console.log(pedido.productos.length);
```

Resultado:

```text
Guadalajara
Mouse
2
```

La estructura más común en las APIs es un **array de objetos**, el equivalente de `list[dict]` en Python:

```javascript
const productos = [
  { nombre: "Laptop", precio: 15000, stock: 5 },
  { nombre: "Mouse", precio: 450, stock: 12 },
  { nombre: "Teclado", precio: 900, stock: 8 },
];

for (const producto of productos) {
  console.log(`${producto.nombre} - $${producto.precio}`);
}
```

Resultado:

```text
Laptop - $15000
Mouse - $450
Teclado - $900
```

En el template literal, el primer `$` es texto normal y `${...}` inserta el valor. Por eso `$${producto.precio}` produce `$15000`.

> `console.log()` resume los objetos muy anidados con `[Object]`. Si quieres ver todo, usa `console.dir(objeto, { depth: null })` o `JSON.stringify(objeto, null, 2)` (sección 10).

---

## 6. Atajos al crear objetos

### Propiedades abreviadas

Si una variable tiene el mismo nombre que la clave, basta escribirla una vez:

```javascript
const nombre = "Webcam";
const precio = 800;

const producto = { nombre, precio, stock: 6 };

console.log(producto);
```

Resultado:

```text
{ nombre: 'Webcam', precio: 800, stock: 6 }
```

`{ nombre, precio }` equivale a `{ nombre: nombre, precio: precio }`. Lo usaremos mucho al construir respuestas JSON.

### Claves calculadas

Cuando el nombre de la clave está en una variable, se escribe entre corchetes:

```javascript
const campo = "descuento";
const ajustes = { [campo]: 10 };

console.log(ajustes);
```

Resultado:

```text
{ descuento: 10 }
```

---

## 7. Spread y desestructuración

### Combinar objetos

En el curso de Python usamos `{**reseña, **analisis}` para combinar diccionarios. En JavaScript, `...` dentro de un objeto copia sus propiedades:

```javascript
const reseña = { id: 3, comentario: "Me cobraron dos veces" };
const analisis = { sentimiento: "negativo", urgente: true };

const registro = { ...reseña, ...analisis };

console.log(registro);
```

Resultado:

```text
{
  id: 3,
  comentario: 'Me cobraron dos veces',
  sentimiento: 'negativo',
  urgente: true
}
```

Si una clave se repite, gana la que aparece después. Eso sirve para crear una versión modificada sin tocar el original:

```javascript
const original = { nombre: "Mouse", precio: 500, stock: 12 };
const actualizado = { ...original, precio: 450 };

console.log(original.precio);
console.log(actualizado.precio);
```

Resultado:

```text
500
450
```

### Desestructurar objetos

La desestructuración de arrays de la clase 02 usaba posiciones. Con objetos se usan los **nombres** de las propiedades:

```javascript
const producto = { nombre: "Laptop", precio: 15000, stock: 5 };

const { nombre, precio } = producto;

console.log(nombre);
console.log(precio);
```

El orden no importa: `const { precio, nombre } = producto;` produce lo mismo. También puedes renombrar, dar valores por defecto y reunir el resto:

```javascript
const producto = { nombre: "Laptop", precio: 15000, stock: 5 };

const { nombre: titulo, descripcion = "Sin descripción", ...resto } = producto;

console.log(titulo);
console.log(descripcion);
console.log(resto);
```

Resultado:

```text
Laptop
Sin descripción
{ precio: 15000, stock: 5 }
```

- `nombre: titulo` lee `nombre` y lo guarda en la variable `titulo`.
- `descripcion = "Sin descripción"` se usa solo si la propiedad es `undefined`.
- `...resto` reúne las propiedades que no se nombraron.

La desestructuración también funciona en `for...of`:

```javascript
const productos = [
  { nombre: "Laptop", precio: 15000 },
  { nombre: "Mouse", precio: 450 },
];

for (const { nombre, precio } of productos) {
  console.log(nombre, precio);
}
```

En la clase 04 la usaremos en los parámetros de las funciones para simular los argumentos por nombre de Python.

---

## 8. Comparación por referencia y copias

En Python, `==` compara el contenido de dos diccionarios. En JavaScript, `===` con objetos o arrays compara si son **el mismo objeto en memoria**:

```javascript
const a = { nombre: "Mouse" };
const b = { nombre: "Mouse" };
const c = a;

console.log(a === b);
console.log(a === c);
```

Resultado:

```text
false
true
```

`a` y `b` tienen el mismo contenido, pero son objetos distintos. `c` no es una copia: es otro nombre para el mismo objeto. Por eso modificar `c` modifica `a`:

```javascript
c.nombre = "Teclado";

console.log(a.nombre);
```

Resultado:

```text
Teclado
```

Es el mismo comportamiento que tenía Python al asignar una lista o diccionario a otra variable; la diferencia es que en JavaScript ni siquiera `===` compara contenidos.

### Copia superficial y copia profunda

`{ ...objeto }` crea un objeto nuevo, pero los objetos anidados se siguen compartiendo:

```javascript
const original = {
  nombre: "Laptop",
  specs: { ram: 16 },
};

const superficial = { ...original };
const profunda = structuredClone(original);

superficial.specs.ram = 32;

console.log(original.specs.ram);
console.log(profunda.specs.ram);
```

Resultado:

```text
32
16
```

| En Python | En JavaScript |
|---|---|
| `usuario.copy()` | `{ ...usuario }` |
| `copy.deepcopy(usuario)` | `structuredClone(usuario)` |
| `a == b` compara contenido | `a === b` compara identidad |

Para comparar contenidos tienes que comparar propiedad por propiedad. Para objetos sencillos de datos, comparar `JSON.stringify(a) === JSON.stringify(b)` funciona si las claves están en el mismo orden; no lo uses como regla general.

---

## Actividad 2: Copias seguras

```javascript
const inventario = {
  laptop: { precio: 15000, stock: 5 },
  mouse: { precio: 450, stock: 12 },
};

// Completa durante la actividad.
```

1. Crea `copia` con spread y cambia `copia.mouse.stock` a `0`. Muestra el stock del mouse en `inventario` y explica el resultado.
2. Restaura el stock original a `12`.
3. Crea `respaldo` con `structuredClone()`, cambia `respaldo.mouse.stock` a `0` y comprueba que `inventario` no cambió.
4. Crea `oferta` a partir de `inventario.laptop` con spread, cambiando solo el precio a `13500`.
5. Muestra `inventario.laptop === oferta` y explica por qué es `false`.

---

## 9. `Map`

Los objetos son la herramienta habitual para datos con campos conocidos, como un producto. Para colecciones donde las claves se agregan y consultan dinámicamente, JavaScript tiene `Map`:

```javascript
const ventas = new Map();

ventas.set("Laptop", 3);
ventas.set("Mouse", 10);
ventas.set("Laptop", ventas.get("Laptop") + 1);

console.log(ventas.get("Laptop"));
console.log(ventas.has("Monitor"));
console.log(ventas.size);

for (const [producto, cantidad] of ventas) {
  console.log(producto, cantidad);
}
```

Resultado:

```text
4
false
2
Laptop 4
Mouse 10
```

`Map` se parece más a un diccionario de Python que un objeto:

- Acepta claves de cualquier tipo, incluidos números y objetos, sin convertirlas a texto.
- Tiene `size`, `has()`, `get()`, `set()`, `delete()` y `clear()`.
- Se recorre directamente con `for...of` y siempre conserva el orden de inserción.
- `get()` de una clave ausente devuelve `undefined`, igual que un objeto.

```javascript
const objeto = { 1: "uno" };
const mapa = new Map([[1, "uno"]]);

console.log(Object.keys(objeto));
console.log(mapa.has(1));
console.log(mapa.has("1"));
```

Resultado:

```text
[ '1' ]
true
false
```

Convertir entre ambos:

```javascript
const precios = { laptop: 15000, mouse: 450 };

const mapaPrecios = new Map(Object.entries(precios));
const deVuelta = Object.fromEntries(mapaPrecios);

console.log(mapaPrecios);
console.log(deVuelta);
```

Resultado:

```text
Map(2) { 'laptop' => 15000, 'mouse' => 450 }
{ laptop: 15000, mouse: 450 }
```

Regla práctica: usa objetos para registros con forma fija y para todo lo que vaya a JSON; usa `Map` para conteos, cachés o índices con claves que no son texto. `JSON.stringify()` no sabe convertir un `Map`: conviértelo antes con `Object.fromEntries()`.

---

## 10. JSON

JSON significa *JavaScript Object Notation*: su sintaxis viene de los objetos de JavaScript. Aun así, JSON es **texto** con reglas más estrictas:

| Objeto JavaScript | Texto JSON |
|---|---|
| Claves sin comillas si son nombres válidos | Claves siempre entre comillas dobles |
| Comillas simples o dobles | Solo comillas dobles |
| Comas finales permitidas | Comas finales prohibidas |
| `undefined`, funciones, `Map` | No existen |

### Objeto → texto JSON

`JSON.stringify()` equivale a `json.dumps()`:

```javascript
const producto = {
  nombre: "Café",
  precio: 8,
  etiquetas: ["bebida", "caliente"],
};

const texto = JSON.stringify(producto);
const bonito = JSON.stringify(producto, null, 2);

console.log(texto);
console.log(bonito);
console.log(typeof texto);
```

Resultado:

```text
{"nombre":"Café","precio":8,"etiquetas":["bebida","caliente"]}
{
  "nombre": "Café",
  "precio": 8,
  "etiquetas": [
    "bebida",
    "caliente"
  ]
}
string
```

El tercer argumento es la indentación, como `indent=2`. El segundo, `null`, es un filtro opcional que no usaremos. No hace falta un equivalente de `ensure_ascii=False`: los acentos se conservan tal cual.

### Texto JSON → objeto

`JSON.parse()` equivale a `json.loads()`:

```javascript
const recibido = '{"id": 1, "nombre": "Ana", "activo": true, "telefono": null}';

const usuario = JSON.parse(recibido);

console.log(usuario.nombre);
console.log(usuario.activo);
console.log(usuario.telefono);
```

Resultado:

```text
Ana
true
null
```

Las comillas simples exteriores delimitan la cadena de JavaScript; dentro, el JSON usa comillas dobles. Los valores se convierten así:

| JSON | Python | JavaScript |
|---|---|---|
| `{}` | `dict` | objeto |
| `[]` | `list` | array |
| `"texto"` | `str` | string |
| `1`, `2.5` | `int`, `float` | number |
| `true` / `false` | `True` / `False` | `true` / `false` |
| `null` | `None` | `null` |

### Lo que JSON no conserva

```javascript
const datos = {
  nombre: "Mouse",
  descuento: undefined,
  creado: new Date("2026-01-15T10:00:00Z"),
  precio: NaN,
};

console.log(JSON.stringify(datos));
```

Resultado:

```text
{"nombre":"Mouse","creado":"2026-01-15T10:00:00.000Z","precio":null}
```

- Las propiedades con `undefined` desaparecen. Usa `null` si necesitas que la clave exista en el JSON.
- Las fechas se convierten en texto; al hacer `JSON.parse()` recibirás una cadena, no una fecha.
- `NaN` e `Infinity` se convierten en `null`.

### JSON inválido

Si el texto no es JSON válido, `JSON.parse()` lanza `SyntaxError`, como `json.loads()` lanzaba `json.JSONDecodeError`. La sintaxis completa de `try` / `catch` llega en la clase 05; por ahora basta este patrón:

```javascript
const textoInvalido = "{nombre: 'Ana'}";

try {
  JSON.parse(textoInvalido);
} catch (error) {
  console.log("No es JSON válido:", error.name);
}
```

Resultado:

```text
No es JSON válido: SyntaxError
```

El texto parece un objeto de JavaScript, pero no es JSON: la clave no tiene comillas dobles y el valor usa comillas simples.

Leer y guardar archivos `.json` requiere los módulos de archivos de Node; lo haremos en la clase 06.

---

## Errores comunes viniendo de Python

1. **Esperar `KeyError`.** `producto.precoi` devuelve `undefined` en silencio. Revisa los nombres y valida los datos que vienen de fuera.
2. **Usar `||` como `dict.get()`.** `stock || 10` reemplaza un `0` legítimo. Usa `??`.
3. **Comparar objetos con `===`.** `{ a: 1 } === { a: 1 }` es `false`. Compara propiedades.
4. **Creer que `const` congela el objeto.** Solo impide reasignar la variable; las propiedades se pueden cambiar.
5. **Escribir `objeto.variable` en vez de `objeto[variable]`.** El punto usa el nombre literal de la propiedad.
6. **Copiar con spread un objeto anidado y modificar la copia.** Los objetos internos se comparten; usa `structuredClone()`.
7. **Usar `len()` o `.length` con un objeto.** Los objetos no tienen `length`; usa `Object.keys(objeto).length`.
8. **Escribir JSON con comillas simples o claves sin comillas.** Es válido como objeto de JavaScript, pero `JSON.parse()` lo rechaza.

---

## Cheat sheet

| Tarea | Python | JavaScript |
|---|---|---|
| Crear | `p = {"nombre": "Mouse"}` | `const p = { nombre: "Mouse" };` |
| Leer | `p["nombre"]` | `p.nombre` / `p["nombre"]` |
| Clave en variable | `p[campo]` | `p[campo]` |
| Valor por defecto | `p.get("x", 0)` | `p.x ?? 0` |
| Acceso seguro anidado | `p.get("a", {}).get("b")` | `p.a?.b` |
| Existe | `"x" in p` | `Object.hasOwn(p, "x")` |
| Agregar / modificar | `p["x"] = 1` | `p.x = 1;` |
| Eliminar | `del p["x"]` | `delete p.x;` |
| Tamaño | `len(p)` | `Object.keys(p).length` |
| Claves / valores / pares | `keys()` / `values()` / `items()` | `Object.keys(p)` / `Object.values(p)` / `Object.entries(p)` |
| Recorrer | `for k, v in p.items():` | `for (const [k, v] of Object.entries(p))` |
| Combinar | `{**a, **b}` | `{ ...a, ...b }` |
| Desempaquetar | `nombre = p["nombre"]` | `const { nombre } = p;` |
| Copia superficial | `p.copy()` | `{ ...p }` |
| Copia profunda | `copy.deepcopy(p)` | `structuredClone(p)` |
| Objeto → JSON | `json.dumps(p, indent=2)` | `JSON.stringify(p, null, 2)` |
| JSON → objeto | `json.loads(texto)` | `JSON.parse(texto)` |
| Mapa con claves de cualquier tipo | `dict` | `new Map()` |

---

## Ejercicio final: Sistema de inventario

Es el mismo ejercicio de la clase 04 del curso de Python, ahora con un objeto cuyas claves son los nombres de los productos. Crea `inventario.js` con estos datos:

```javascript
const productos = {
  laptop: { precio: 15000, stock: 5 },
  mouse: { precio: 500, stock: 12 },
  teclado: { precio: 900, stock: 8 },
  monitor: { precio: 4200, stock: 3 },
  audifonos: { precio: 1200, stock: 0 },
};
```

Crea un programa que:

1. Muestre cuántos productos diferentes existen.
2. Muestre el precio y stock de `laptop`.
3. Cambie el precio de `mouse` a `450`.
4. Agregue un nuevo producto:

```javascript
productos.webcam = { precio: 800, stock: 6 };
```

5. Recorra todos los productos con `Object.entries()` y desestructuración, y muestre su nombre, precio y stock.
6. Muestre únicamente los productos con stock mayor a `0`.
7. Calcule el valor total del inventario (`precio × stock`).
8. Encuentre el producto con el precio más alto **sin escribir manualmente cuál es**.
9. Elimine `audifonos` con `delete`, guardando antes su información en una variable `eliminado`.
10. Compruebe si `monitor` continúa en el inventario con `Object.hasOwn()`.
11. Muestre el inventario final como texto JSON con indentación de 2 espacios.

### Salida esperada aproximada

```text
Productos diferentes: 5

Laptop
Precio: 15000
Stock: 5

Inventario:

laptop - $15000 - stock: 5
mouse - $450 - stock: 12
teclado - $900 - stock: 8
monitor - $4200 - stock: 3
audifonos - $1200 - stock: 0
webcam - $800 - stock: 6

Productos disponibles:
laptop
mouse
teclado
monitor
webcam

Valor total del inventario: $105000

Producto más caro: laptop
Precio: $15000

Eliminado: audifonos - $1200
¿Existe monitor?: true

Inventario final en JSON:
{
  "laptop": {
    "precio": 15000,
    "stock": 5
  },
  "mouse": {
    "precio": 450,
    "stock": 12
  },
  "teclado": {
    "precio": 900,
    "stock": 8
  },
  "monitor": {
    "precio": 4200,
    "stock": 3
  },
  "webcam": {
    "precio": 800,
    "stock": 6
  }
}
```

> Nota: el valor total se calcula después de cambiar el precio del mouse y agregar la webcam, pero antes de eliminar audífonos.

Para el punto 8, recorre los productos guardando el nombre y el precio del más caro encontrado hasta el momento. Empieza con `let masCaro = null;` y reemplázalo cuando el precio actual sea mayor.

### Práctica opcional

1. Convierte el inventario en un array de objetos con la forma `{ nombre, precio, stock }`. Usa `Object.entries()`, desestructuración y propiedades abreviadas.
2. Cuenta cuántos productos hay en cada rango de precio (`"económico"` menor a 1000, `"medio"` hasta 5000, `"premium"` el resto) usando un `Map`.
3. Convierte el `Map` a objeto con `Object.fromEntries()` y muéstralo con `JSON.stringify()`.
4. Toma el texto JSON final, conviértelo de nuevo con `JSON.parse()` y comprueba que `webcam.precio` sea `800`.

---

[Índice del curso](README.md)
