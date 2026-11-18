# Clase 04: Funciones, arrow functions y callbacks

**Duración:** 1 a 2 horas

> Los bloques de desarrollo se leen en orden dentro de cada ejemplo. Las plantillas con `...` se completan en las actividades; las cheat sheets reúnen operaciones independientes. Los programas completos incluyen sus imports.

### Objetivos

- Declarar funciones con `function` y devolver valores con `return`.
- Usar parámetros por defecto y entender qué pasa con argumentos faltantes o sobrantes.
- Reemplazar los argumentos por nombre de Python con un objeto de opciones.
- Usar parámetros rest y spread, el equivalente de `*args`.
- Escribir arrow functions, el equivalente más cercano a `lambda`.
- Pasar funciones como argumentos (callbacks).
- Procesar arrays con `map`, `filter`, `find`, `some`, `every`, `reduce` y ordenar con un comparador.
- Entender el alcance de bloque y las closures.
- Documentar tipos con JSDoc, como hacíamos con los type hints.

---

## 1. Declarar funciones

En Python usábamos `def`. En JavaScript la forma básica es `function`:

```python
def sumar(a, b):
    return a + b
```

```javascript
function sumar(a, b) {
  return a + b;
}

const resultado = sumar(10, 5);
console.log(resultado);
```

Resultado:

```text
15
```

Como en Python, definir la función no ejecuta su cuerpo: `sumar(10, 5)` sí lo hace. Las llaves delimitan el cuerpo; la indentación es solo para leer mejor.

### Sin `return`: `undefined`

En Python, una función sin `return` devolvía `None`. En JavaScript devuelve `undefined`:

```javascript
function saludar(nombre) {
  console.log(`Hola ${nombre}`);
}

const valor = saludar("Ana");
console.log(valor);
```

Resultado:

```text
Hola Ana
undefined
```

`saludar()` muestra un texto, pero no lo devuelve. La diferencia entre imprimir y devolver es la misma que en Python.

> Cuidado con `return` en una línea sola. JavaScript inserta un punto y coma automático después de `return`, así que `return` seguido de un salto de línea devuelve `undefined`. Empieza el valor en la misma línea: `return {` o `return (`.

---

## 2. Parámetros

### Valores por defecto

```javascript
function saludar(nombre = "Usuario") {
  console.log(`Hola ${nombre}`);
}

saludar("Ana");
saludar();
```

Resultado:

```text
Hola Ana
Hola Usuario
```

El valor por defecto se usa cuando el argumento es `undefined`, ya sea porque se omitió o porque se pasó `undefined` explícitamente. Con `null` **no** se usa: `saludar(null)` muestra `Hola null`.

### Argumentos faltantes y sobrantes

En Python, llamar con menos o más argumentos lanzaba `TypeError`. En JavaScript no hay error:

```javascript
function crearUsuario(nombre, edad) {
  console.log(nombre, edad);
}

crearUsuario("Ana");
crearUsuario("Luis", 30, "México");
```

Resultado:

```text
Ana undefined
Luis 30
```

El parámetro faltante vale `undefined` y el argumento sobrante se ignora. Si un argumento es obligatorio, compruébalo tú; en la clase 05 veremos cómo lanzar un error.

| En Python | En JavaScript |
|---|---|
| `def f(a, b):` | `function f(a, b) { }` |
| Falta un argumento → `TypeError` | El parámetro vale `undefined` |
| Sobra un argumento → `TypeError` | Se ignora |
| Sin `return` → `None` | Sin `return` → `undefined` |
| `def f(x="defecto"):` | `function f(x = "defecto") { }` |

---

## 3. Argumentos por nombre: objeto de opciones

En Python podíamos llamar `crear_usuario(nombre="Ana", pais="México", edad=25)`. JavaScript **no** tiene argumentos por nombre. Si escribes `crearUsuario(nombre = "Ana")`, estás asignando una variable, no nombrando un argumento.

La convención es recibir un único objeto y desestructurarlo en los parámetros, con la desestructuración de la clase 03:

```javascript
function crearUsuario({ nombre, edad, pais = "México" }) {
  console.log(nombre, edad, pais);
}

crearUsuario({ nombre: "Ana", pais: "Chile", edad: 25 });
crearUsuario({ edad: 30, nombre: "Luis" });
```

Resultado:

```text
Ana 25 Chile
Luis 30 México
```

- El orden de las propiedades no importa, como con los argumentos por nombre de Python.
- Cada propiedad puede tener su valor por defecto.
- Las propiedades que no se envían valen `undefined`.

Si todas las opciones son opcionales, agrega `= {}` para que la función se pueda llamar sin argumentos:

```javascript
function formatearPrecio(precio, { moneda = "MXN", decimales = 2 } = {}) {
  return `$${precio.toFixed(decimales)} ${moneda}`;
}

console.log(formatearPrecio(450));
console.log(formatearPrecio(450, { decimales: 0 }));
console.log(formatearPrecio(19.5, { moneda: "USD" }));
```

Resultado:

```text
$450.00 MXN
$450 MXN
$19.50 USD
```

Sin `= {}`, `formatearPrecio(450)` intentaría desestructurar `undefined` y lanzaría `TypeError`.

---

## 4. Rest y spread

### Número variable de argumentos

`...numeros` en los parámetros reúne los argumentos restantes en un **array**, como `*numeros` reunía una tupla en Python:

```python
def sumar(*numeros):
    return sum(numeros)
```

```javascript
function sumar(...numeros) {
  let total = 0;
  for (const numero of numeros) {
    total += numero;
  }
  return total;
}

console.log(sumar(10, 20, 30, 40));
console.log(sumar());
```

Resultado:

```text
100
0
```

El parámetro rest debe ser el último: `function registrar(nivel, ...mensajes)`.

### Desempacar un array al llamar

Al llamar una función, `...` hace lo contrario: reparte los elementos de un array como argumentos separados, como `*lista` en Python:

```javascript
const precios = [15000, 450, 900, 4200];

console.log(Math.max(...precios));
console.log(Math.min(...precios));
console.log(sumar(...precios));
```

Resultado:

```text
15000
450
20550
```

`Math.max(precios)` sin `...` devolvería `NaN`, porque recibiría un único argumento que es un array.

---

## 5. Function expressions y arrow functions

En JavaScript una función es un valor: se puede guardar en una variable.

```javascript
const duplicar = function (numero) {
  return numero * 2;
};
```

Esto se llama *function expression*. La forma moderna y más usada es la **arrow function**:

```javascript
const duplicar = (numero) => {
  return numero * 2;
};
```

Si el cuerpo es una sola expresión, se pueden quitar las llaves y el `return`. A eso se le llama **return implícito**:

```javascript
const duplicar = (numero) => numero * 2;
const sumar = (a, b) => a + b;
const saludar = () => "Hola";

console.log(duplicar(10));
console.log(sumar(3, 4));
console.log(saludar());
```

Resultado:

```text
20
7
Hola
```

Comparación con `lambda`:

```python
duplicar = lambda numero: numero * 2
```

```javascript
const duplicar = (numero) => numero * 2;
```

A diferencia de `lambda`, una arrow function puede tener varias líneas si usa llaves; en ese caso, el `return` vuelve a ser obligatorio:

```javascript
const clasificarPrecio = (precio) => {
  if (precio < 1000) {
    return "económico";
  }
  return "premium";
};

console.log(clasificarPrecio(450));
```

### Devolver un objeto

Las llaves después de `=>` se interpretan como el cuerpo de la función, no como un objeto. Para devolver un objeto con return implícito, envuélvelo en paréntesis:

```javascript
const crearProducto = (nombre, precio) => ({ nombre, precio });

console.log(crearProducto("Mouse", 450));
```

Resultado:

```text
{ nombre: 'Mouse', precio: 450 }
```

Sin paréntesis, `(nombre, precio) => { nombre, precio }` devolvería `undefined`.

| `lambda` en Python | Arrow function en JavaScript |
|---|---|
| Una sola expresión | Una expresión o un bloque con llaves |
| `lambda x: x * 2` | `(x) => x * 2` |
| `lambda: "Hola"` | `() => "Hola"` |
| Se recomienda solo para funciones pequeñas | Se usa en todas partes, sobre todo en callbacks |

¿Cuándo usar cada forma? En este curso usaremos `function` para las funciones principales de un programa y arrow functions para funciones pequeñas que se pasan como argumento. Ambas son correctas.

---

## 6. Funciones como valores y callbacks

Una función se puede pasar como argumento a otra función. La función que se pasa se llama **callback**: la otra función decide cuándo llamarla.

```javascript
function aplicarATodos(numeros, operacion) {
  const resultado = [];
  for (const numero of numeros) {
    resultado.push(operacion(numero));
  }
  return resultado;
}

console.log(aplicarATodos([1, 2, 3], (n) => n * 10));
console.log(aplicarATodos([1, 2, 3], duplicar));
```

Resultado:

```text
[ 10, 20, 30 ]
[ 2, 4, 6 ]
```

Fíjate en que pasamos `duplicar`, sin paréntesis. `duplicar()` la ejecutaría y pasaría su resultado.

Ya usaste esta idea en Python con `sort(key=lambda producto: producto[1])`: `sort()` recibía una función y la llamaba durante la ordenación.

### Ordenar con un comparador

En la clase 02 vimos que `sort()` necesita un **comparador** para ordenar números. El comparador recibe dos elementos `a` y `b` y devuelve:

- un número negativo si `a` va antes que `b`,
- un número positivo si `a` va después,
- `0` si da igual.

Con productos, comparamos una propiedad:

```javascript
const productos = [
  { nombre: "Café", precio: 8 },
  { nombre: "Pan", precio: 9 },
  { nombre: "Leche", precio: 6 },
];

const porPrecio = productos.toSorted((a, b) => a.precio - b.precio);
const porPrecioDesc = productos.toSorted((a, b) => b.precio - a.precio);
const porNombre = productos.toSorted((a, b) => a.nombre.localeCompare(b.nombre));

console.log(porPrecio.map((p) => p.nombre));
console.log(porPrecioDesc.map((p) => p.nombre));
console.log(porNombre.map((p) => p.nombre));
```

Resultado:

```text
[ 'Leche', 'Café', 'Pan' ]
[ 'Pan', 'Café', 'Leche' ]
[ 'Café', 'Leche', 'Pan' ]
```

| En Python | En JavaScript |
|---|---|
| `sorted(productos, key=lambda p: p["precio"])` | `productos.toSorted((a, b) => a.precio - b.precio)` |
| `reverse=True` | Invertir la resta: `b.precio - a.precio` |
| `productos.sort(key=...)` | `productos.sort((a, b) => ...)` (modifica el original) |

Python pide una función que calcule una **clave** por elemento; JavaScript pide una función que **compare dos** elementos.

---

## Actividad 1: Funciones y callbacks

Crea `utilidades.js` y completa:

```javascript
function calcularIva(precio, { tasa = 0.16 } = {}) {
  // Completa durante la actividad.
}

const aplicarDescuento = (precio, porcentaje) => {
  // Completa durante la actividad.
};

function transformarPrecios(precios, operacion) {
  // Completa durante la actividad.
}
```

1. `calcularIva(1000)` debe devolver `160`; `calcularIva(1000, { tasa: 0.08 })` debe devolver `80`.
2. `aplicarDescuento(1000, 15)` debe devolver `850`. Reescríbela después con return implícito.
3. `transformarPrecios([100, 200], (p) => p * 2)` debe devolver `[200, 400]` sin modificar el array original.
4. Llama `transformarPrecios()` pasando una arrow function que aplique un 10 % de descuento con `aplicarDescuento()`.
5. Llama `aplicarDescuento(1000)` sin porcentaje. Observa el resultado (`NaN`) y agrega un valor por defecto de `0`.

---

## 7. Métodos de arrays con callbacks

Los arrays tienen métodos que reciben callbacks. Reemplazan a las comprehensions y a varias funciones de Python. Usaremos estos productos:

```javascript
const productos = [
  { nombre: "Laptop", precio: 15000, stock: 5 },
  { nombre: "Mouse", precio: 450, stock: 12 },
  { nombre: "Monitor", precio: 4200, stock: 3 },
  { nombre: "Audifonos", precio: 1200, stock: 0 },
];
```

### `map`: transformar cada elemento

```javascript
const nombres = productos.map((producto) => producto.nombre.toUpperCase());

console.log(nombres);
```

Resultado:

```text
[ 'LAPTOP', 'MOUSE', 'MONITOR', 'AUDIFONOS' ]
```

Equivale a `[p["nombre"].upper() for p in productos]`. Devuelve un array nuevo del mismo tamaño.

### `filter`: quedarse con algunos

```javascript
const disponibles = productos.filter((producto) => producto.stock > 0);

console.log(disponibles.length);
```

Resultado:

```text
3
```

Equivale a `[p for p in productos if p["stock"] > 0]`. El callback devuelve `true` para conservar el elemento.

### `find` y `findIndex`: buscar uno

```javascript
const monitor = productos.find((producto) => producto.nombre === "Monitor");
const teclado = productos.find((producto) => producto.nombre === "Teclado");
const posicion = productos.findIndex((producto) => producto.nombre === "Mouse");

console.log(monitor);
console.log(teclado);
console.log(posicion);
```

Resultado:

```text
{ nombre: 'Monitor', precio: 4200, stock: 3 }
undefined
1
```

`find()` devuelve el **primer** elemento que cumple la condición o `undefined`. `findIndex()` devuelve su posición o `-1`. Es el equivalente de `next((p for p in productos if ...), None)`.

### `some` y `every`: preguntas de sí o no

```javascript
const hayAgotados = productos.some((producto) => producto.stock === 0);
const todosConPrecio = productos.every((producto) => producto.precio > 0);

console.log(hayAgotados);
console.log(todosConPrecio);
```

Resultado:

```text
true
true
```

Equivalen a `any(...)` y `all(...)`.

### `reduce`: acumular un resultado

`reduce()` recorre el array y va actualizando un acumulador. Recibe el callback y el valor inicial:

```javascript
const valorTotal = productos.reduce(
  (total, producto) => total + producto.precio * producto.stock,
  0
);

console.log(valorTotal);
```

Resultado:

```text
93000
```

En cada paso, `total` es lo acumulado hasta ahora y `producto` el elemento actual; lo que devuelve el callback se convierte en el nuevo `total`. Equivale a `sum(p["precio"] * p["stock"] for p in productos)`.

Siempre pasa el valor inicial (`0`). Sin él, `reduce()` usa el primer elemento como acumulador y falla con un array vacío.

Si `reduce()` se vuelve difícil de leer, un `for...of` con una variable `let total = 0` es igual de válido.

### `forEach`: ejecutar algo por elemento

```javascript
productos.forEach((producto, indice) => {
  console.log(`${indice + 1}. ${producto.nombre}`);
});
```

`forEach()` no devuelve nada; sirve para efectos como imprimir. El segundo parámetro de los callbacks de estos métodos es el índice, como `enumerate()`. En la mayoría de los casos, `for...of` es más claro y además permite `break`.

### Encadenar

Como `map()` y `filter()` devuelven arrays, se pueden encadenar:

```javascript
const resumen = productos
  .filter((producto) => producto.stock > 0)
  .toSorted((a, b) => b.precio - a.precio)
  .map((producto) => `${producto.nombre}: $${producto.precio}`);

console.log(resumen);
```

Resultado:

```text
[ 'Laptop: $15000', 'Monitor: $4200', 'Mouse: $450' ]
```

Se lee de arriba abajo: filtrar disponibles, ordenar de mayor a menor precio y convertir a texto. `toSorted()` devuelve una copia; `filter()` y `map()` tampoco modifican el original.

| En Python | En JavaScript |
|---|---|
| `[f(x) for x in lista]` | `lista.map((x) => f(x))` |
| `[x for x in lista if cond]` | `lista.filter((x) => cond)` |
| `next((x for x in lista if cond), None)` | `lista.find((x) => cond)` |
| `any(cond for x in lista)` | `lista.some((x) => cond)` |
| `all(cond for x in lista)` | `lista.every((x) => cond)` |
| `sum(x["precio"] for x in lista)` | `lista.reduce((t, x) => t + x.precio, 0)` |
| `for i, x in enumerate(lista):` | `lista.forEach((x, i) => { })` o `for (const [i, x] of lista.entries())` |

---

## 8. Alcance y closures

### Alcance de bloque

`let` y `const` solo existen dentro del bloque `{ }` donde se declaran. Eso incluye los bloques de `if` y `for`, algo que no ocurre en Python:

```javascript
function revisarStock(stock) {
  if (stock > 0) {
    const mensaje = "Disponible";
    console.log(mensaje);
  }
  // console.log(mensaje); // ReferenceError: mensaje is not defined
}

revisarStock(5);
```

En Python, una variable creada dentro de un `if` sigue existiendo después del bloque. En JavaScript no. Si la necesitas después, declárala antes con `let`.

Como en Python, es preferible pasar información con parámetros y devolver resultados con `return`.

### Closures

Una función recuerda las variables del lugar donde se creó, aunque ese lugar ya haya terminado de ejecutarse. A esa combinación se le llama **closure**:

```javascript
function crearContador(inicio = 1) {
  let actual = inicio;
  return () => {
    const valor = actual;
    actual += 1;
    return valor;
  };
}

const siguienteId = crearContador(4);

console.log(siguienteId());
console.log(siguienteId());
console.log(siguienteId());
```

Resultado:

```text
4
5
6
```

`crearContador()` terminó, pero la arrow function que devolvió conserva acceso a `actual`. Cada llamada a `crearContador()` crea un contador independiente. Es el mismo comportamiento que `ids = count(start=4)` y `next(ids)` en la clase 11 del curso de Python; en la clase 09 de este curso generaremos IDs de forma parecida.

### Hoisting

Las funciones declaradas con `function` se pueden llamar antes de la línea donde aparecen, porque JavaScript las registra al cargar el archivo. Esto se llama *hoisting*:

```javascript
console.log(cuadrado(4));

function cuadrado(n) {
  return n * n;
}
```

Las funciones guardadas en `const`, incluidas las arrow functions, **no** se pueden usar antes de su declaración: lanzan `ReferenceError`. Para evitar confusiones, declara las funciones antes de usarlas o deja la llamada principal al final del archivo.

---

## 9. JSDoc: documentar tipos

En el curso de Python (clase 07) usamos type hints: `def calcular_valor(productos: list[dict]) -> float:`. JavaScript no tiene anotaciones de tipos en su sintaxis, pero los editores entienden comentarios **JSDoc**:

```javascript
// @ts-check

/**
 * Calcula el valor del inventario.
 * @param {{ precio: number, stock: number }[]} productos
 * @returns {number}
 */
function calcularValor(productos) {
  return productos.reduce((total, p) => total + p.precio * p.stock, 0);
}

console.log(calcularValor([{ precio: 10, stock: 3 }]));
```

- `/** ... */` es un comentario de documentación que se coloca justo antes de la función.
- `@param {tipo} nombre` y `@returns {tipo}` describen la entrada y la salida.
- `// @ts-check` en la primera línea pide a editores como VS Code que revisen el archivo y subrayen errores, por ejemplo `calcularValor("hola")`.

Como los type hints de Python, JSDoc **no valida nada al ejecutar**: Node ignora los comentarios. Es una ayuda para el editor y para quien lee el código. La validación real de datos externos la haremos con Zod en la clase 09.

> TypeScript es una extensión de JavaScript con tipos en la sintaxis. Queda fuera de este curso; JSDoc es un buen paso previo.

---

## Errores comunes viniendo de Python

1. **Esperar `TypeError` por argumentos faltantes.** El parámetro vale `undefined` y el error aparece más tarde, por ejemplo como `NaN`.
2. **Intentar argumentos por nombre.** `f(nombre = "Ana")` asigna una variable. Usa un objeto: `f({ nombre: "Ana" })`.
3. **Olvidar `return` en una arrow function con llaves.** `(x) => { x * 2 }` devuelve `undefined`. Quita las llaves o escribe `return`.
4. **Devolver un objeto sin paréntesis.** `() => { a: 1 }` no devuelve un objeto; usa `() => ({ a: 1 })`.
5. **Llamar la función al pasarla como callback.** `lista.map(duplicar())` ejecuta `duplicar` antes; pasa `duplicar` sin paréntesis.
6. **Ordenar como en Python.** `sort()` no recibe `key=`; recibe un comparador `(a, b) => a.precio - b.precio`.
7. **Olvidar el valor inicial de `reduce()`.** Escribe siempre `reduce(callback, 0)` o el valor que corresponda.
8. **Usar una variable fuera del bloque donde se declaró.** `let` y `const` no existen fuera de sus llaves.

---

## Cheat sheet

```javascript
function sumar(a, b) {
  return a + b;
}

function saludar(nombre = "Usuario") {
  return `Hola ${nombre}`;
}

function crearUsuario({ nombre, edad, pais = "México" }) {
  return { nombre, edad, pais };
}

function sumarVarios(...numeros) {
  return numeros.reduce((total, n) => total + n, 0);
}

const duplicar = (x) => x * 2;
const crearProducto = (nombre, precio) => ({ nombre, precio });

const productos = [
  { nombre: "Café", precio: 8, stock: 3 },
  { nombre: "Pan", precio: 9, stock: 0 },
];

productos.map((p) => p.nombre);
productos.filter((p) => p.stock > 0);
productos.find((p) => p.nombre === "Pan");
productos.some((p) => p.stock === 0);
productos.every((p) => p.precio > 0);
productos.reduce((total, p) => total + p.precio * p.stock, 0);
productos.toSorted((a, b) => b.precio - a.precio);
Math.max(...productos.map((p) => p.precio));
```

| Python | JavaScript |
|---|---|
| `def f(x):` | `function f(x) { }` |
| `lambda x: x * 2` | `(x) => x * 2` |
| `f(nombre="Ana")` | `f({ nombre: "Ana" })` |
| `def f(*args):` | `function f(...args) { }` |
| `f(*lista)` | `f(...lista)` |
| `None` al no haber `return` | `undefined` |
| `sorted(l, key=lambda p: p["precio"])` | `l.toSorted((a, b) => a.precio - b.precio)` |
| `def f(x: int) -> int:` | JSDoc `@param {number} x` / `@returns {number}` |

---

## Ejercicio final: Procesador de productos

Es el ejercicio de la clase 05 del curso de Python, ahora con métodos de arrays. Crea `procesador.js`:

```javascript
const productos = [
  { nombre: "Laptop", precio: 15000, stock: 5 },
  { nombre: "Mouse", precio: 450, stock: 12 },
  { nombre: "Teclado", precio: 900, stock: 8 },
  { nombre: "Monitor", precio: 4200, stock: 3 },
  { nombre: "Audifonos", precio: 1200, stock: 0 },
  { nombre: "Webcam", precio: 800, stock: 6 },
];

function productosDisponibles(productos) {
  // Completa durante la actividad.
}

function calcularValor(productos) {
  // Completa durante la actividad.
}

function buscarProducto(productos, nombre) {
  // Completa durante la actividad.
}

function mostrarProductos(productos) {
  // Completa durante la actividad.
}
```

### Requisitos

1. `productosDisponibles()` debe devolver un array nuevo solo con productos cuyo `stock` sea mayor a `0`. Usa `filter()`.
2. `calcularValor()` debe calcular el valor total del inventario (`precio × stock`) con `reduce()`.
3. `buscarProducto()` debe buscar sin importar mayúsculas o minúsculas con `find()` y `toLowerCase()`. Debe devolver `undefined` si no existe.

```javascript
buscarProducto(productos, "LAPTOP");
buscarProducto(productos, "laptop");
```

4. `mostrarProductos()` debe usar `toUpperCase()` y producir:

```text
LAPTOP - $15000 - Stock: 5
MOUSE - $450 - Stock: 12
```

5. Ordena los productos de **mayor a menor precio** con `toSorted()` y un comparador, sin modificar el array original.
6. Agrega comentarios JSDoc a las cuatro funciones y `// @ts-check` al inicio del archivo.

---

## Salida esperada aproximada

```text
PRODUCTOS DISPONIBLES

LAPTOP - $15000 - Stock: 5
MOUSE - $450 - Stock: 12
TECLADO - $900 - Stock: 8
MONITOR - $4200 - Stock: 3
WEBCAM - $800 - Stock: 6

Valor total del inventario:
$105000

Búsqueda: LAPTOP

Producto encontrado:
Laptop - $15000

ORDENADOS POR PRECIO

LAPTOP - $15000
MONITOR - $4200
AUDIFONOS - $1200
TECLADO - $900
WEBCAM - $800
MOUSE - $450
```

### Práctica opcional

Agrega un objeto de opciones:

```javascript
function mostrarProductos(productos, { mostrarSinStock = false } = {}) {
  // Completa durante la actividad.
}
```

Si `mostrarSinStock` es `false`, no debe mostrar productos sin stock. Si es `true`, debe mostrarlos todos. Llámala de las dos formas:

```javascript
mostrarProductos(productos);
mostrarProductos(productos, { mostrarSinStock: true });
```

Después crea `crearContador()` y úsalo para asignar un `id` a cada producto con `map()`, empezando en `1`, sin modificar los objetos originales.

---

[Índice del curso](README.md)
