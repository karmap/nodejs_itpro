# Clase 02: Arrays y Set

**Duración:** 1 a 2 horas

> Los bloques de desarrollo se leen en orden dentro de cada ejemplo. Las plantillas con `...` se completan en las actividades; las cheat sheets reúnen operaciones independientes. Los programas completos incluyen sus imports.

### Objetivos

- Traducir listas, tuplas y conjuntos de Python a `Array` y `Set`.
- Acceder, agregar, modificar y eliminar elementos de un array.
- Copiar, unir y recortar arrays sin modificar el original.
- Ordenar correctamente números y textos.
- Desestructurar arrays, como hacíamos con el desempaquetado de tuplas.
- Eliminar duplicados y comparar grupos de usuarios con `Set`.

---

## 1. De listas a arrays

En el curso de Python dedicamos tres clases a listas, tuplas y conjuntos. En JavaScript, el **array** cumple el papel de la lista y, con algunas precauciones, el de la tupla. El conjunto se llama `Set`.

| En Python | En JavaScript |
|---|---|
| `list` | `Array` |
| `tuple` | `Array` (sin una versión inmutable propia) |
| `set` | `Set` |

```javascript
const productos = ["Laptop", "Mouse", "Teclado"];
const precios = [15000, 450, 900];
const mezcla = ["Mouse", 450, true, null];
const vacio = [];

console.log(productos);
console.log(productos.length);
```

```text
[ 'Laptop', 'Mouse', 'Teclado' ]
3
```

`length` es una propiedad del array, sin paréntesis, como en los textos de la clase 01. `console.log()` muestra los textos de un array con comillas simples; es solo su forma de representarlos.

---

## 2. Acceder a elementos

Los índices empiezan en `0`, como en Python:

```javascript
const productos = ["Laptop", "Mouse", "Teclado", "Monitor"];

console.log(productos[0]);       // Laptop
console.log(productos[2]);       // Teclado
console.log(productos.at(-1));   // Monitor
console.log(productos.at(-2));   // Teclado
```

Los corchetes no aceptan índices negativos: `productos[-1]` devuelve `undefined`. Para contar desde el final, usa `at()`.

### Índices fuera de rango

En Python, `productos[99]` lanzaba `IndexError`. En JavaScript devuelve `undefined` y el programa continúa:

```javascript
const productos = ["Laptop", "Mouse"];

const producto = productos[99];
console.log(producto); // undefined

if (producto === undefined) {
  console.log("No existe un producto en esa posición");
}
```

Esta diferencia se repetirá a lo largo del curso: muchas operaciones que en Python lanzaban una excepción en JavaScript devuelven `undefined`. Comprueba los resultados antes de usarlos.

### Buscar elementos

```javascript
const productos = ["Laptop", "Mouse", "Teclado"];

console.log(productos.includes("Mouse"));  // true
console.log(productos.indexOf("Teclado")); // 2
console.log(productos.indexOf("Webcam"));  // -1
```

| En Python | En JavaScript |
|---|---|
| `"Mouse" in productos` | `productos.includes("Mouse")` |
| `productos.index("Teclado")` | `productos.indexOf("Teclado")` |
| `index()` lanza `ValueError` si no existe | `indexOf()` devuelve `-1` |

---

## 3. Modificar, agregar y eliminar

Asignar a un índice reemplaza el elemento:

```javascript
const productos = ["Laptop", "Mouse", "Teclado"];
productos[1] = "Mouse inalámbrico";
console.log(productos);
```

```text
[ 'Laptop', 'Mouse inalámbrico', 'Teclado' ]
```

`productos` es una constante y aun así cambiamos su contenido. `const` impide **reasignar la variable** (`productos = [...]`), no modificar el array al que apunta. Lo retomaremos en la sección 8.

### Agregar y quitar en los extremos

```javascript
const carrito = ["pan", "leche"];

carrito.push("café");      // al final
carrito.unshift("huevos"); // al inicio
console.log(carrito);      // [ 'huevos', 'pan', 'leche', 'café' ]

const ultimo = carrito.pop();    // quita el último
const primero = carrito.shift(); // quita el primero
console.log(ultimo, primero);    // café huevos
console.log(carrito);            // [ 'pan', 'leche' ]
```

### Insertar y eliminar en una posición con `splice()`

`splice(inicio, cantidadAEliminar, ...nuevos)` modifica el array: elimina elementos desde `inicio` e inserta los nuevos en esa posición. Devuelve un array con los elementos eliminados.

```javascript
const carrito = ["pan", "leche", "café"];

carrito.splice(1, 0, "té");  // inserta "té" en la posición 1
console.log(carrito);        // [ 'pan', 'té', 'leche', 'café' ]

const eliminados = carrito.splice(2, 1); // elimina 1 elemento en la posición 2
console.log(eliminados);     // [ 'leche' ]
console.log(carrito);        // [ 'pan', 'té', 'café' ]
```

Para eliminar por valor, primero busca la posición. Comprueba que no sea `-1`: `splice(-1, 1)` eliminaría el último elemento.

```javascript
const carrito = ["pan", "té", "café"];
const posicion = carrito.indexOf("té");

if (posicion !== -1) {
  carrito.splice(posicion, 1);
}
console.log(carrito); // [ 'pan', 'café' ]
```

| En Python | En JavaScript |
|---|---|
| `lista.append(x)` | `lista.push(x)` |
| `lista.insert(0, x)` | `lista.unshift(x)` |
| `lista.insert(i, x)` | `lista.splice(i, 0, x)` |
| `lista.pop()` | `lista.pop()` |
| `lista.pop(0)` | `lista.shift()` |
| `del lista[i]` | `lista.splice(i, 1)` |
| `lista.remove(x)` | `indexOf()` + `splice()` |
| `lista.clear()` | `lista.length = 0` |

---

## 4. Recortar, unir y copiar

### `slice()` en lugar de slicing

`slice(inicio, fin)` devuelve un array nuevo desde `inicio` hasta antes de `fin`, como `lista[inicio:fin]`. No modifica el original. No existe el tercer valor de paso de Python.

```javascript
const numeros = [10, 20, 30, 40, 50];

console.log(numeros.slice(1, 3)); // [ 20, 30 ]
console.log(numeros.slice(2));    // [ 30, 40, 50 ]
console.log(numeros.slice(-2));   // [ 40, 50 ]
console.log(numeros);             // [ 10, 20, 30, 40, 50 ]
```

No confundas `slice()` (copia una parte) con `splice()` (modifica el array).

### Unir con spread

En Python, `a + b` unía listas. En JavaScript, `+` convierte los arrays en texto. Para unirlos usamos el **operador spread** `...`, que expande los elementos de un array dentro de otro:

```javascript
const frutas = ["manzana", "pera"];
const verduras = ["zanahoria"];

const mercado = [...frutas, ...verduras, "pan"];
console.log(mercado);            // [ 'manzana', 'pera', 'zanahoria', 'pan' ]
console.log(frutas.concat(verduras)); // [ 'manzana', 'pera', 'zanahoria' ]
console.log(frutas + verduras);  // manzana,perazanahoria
```

`concat()` es otra forma de unir. El spread también expande un array como argumentos: `Math.max(...[3, 8, 1])` devuelve `8`, como `max(*lista)` en Python.

### Referencias y copias

Asignar un array a otra variable **no lo copia**: ambas variables apuntan al mismo array, igual que en Python.

```javascript
const original = ["pan", "leche"];
const mismo = original;
const copia = [...original];

mismo.push("café");

console.log(original); // [ 'pan', 'leche', 'café' ]
console.log(copia);    // [ 'pan', 'leche' ]
```

`[...original]` crea una **copia superficial**, como `lista.copy()`. Si el array contiene otros arrays, los internos se comparten. `structuredClone()` crea una **copia profunda**, como `copy.deepcopy()`:

```javascript
const pedidos = [["pan", 2], ["leche", 1]];

const superficial = [...pedidos];
const profunda = structuredClone(pedidos);

superficial[0][1] = 5;

console.log(pedidos[0]);  // [ 'pan', 5 ]
console.log(profunda[0]); // [ 'pan', 2 ]
```

---

## 5. Recorrer arrays

`for...of` recorre los elementos, como el `for` de Python:

```javascript
const productos = ["Laptop", "Mouse", "Teclado"];

for (const producto of productos) {
  console.log(producto.toUpperCase());
}
```

Para obtener también la posición, `entries()` produce pares `[indice, elemento]`, como `enumerate()`:

```python
for i, producto in enumerate(productos):
    print(i, producto)
```

```javascript
const productos = ["Laptop", "Mouse", "Teclado"];

for (const [indice, producto] of productos.entries()) {
  console.log(`${indice + 1}. ${producto}`);
}
```

```text
1. Laptop
2. Mouse
3. Teclado
```

`const [indice, producto]` desestructura cada par; lo explicamos en la sección 9. El `for` clásico de la clase 01 también funciona con `productos[i]` y `productos.length`, pero `for...of` es más claro cuando no necesitas manipular el índice.

### Mostrar un array como texto

`join()` une los elementos con un separador, como `", ".join(lista)` en Python; aquí el separador va como argumento:

```javascript
const productos = ["Laptop", "Mouse", "Teclado"];
console.log(productos.join(", ")); // Laptop, Mouse, Teclado
```

---

## Actividad 1: Lista de compras

Crea `compras.js` a partir de:

```javascript
const compras = ["pan", "leche", "huevos", "café", "azúcar"];
```

1. Muestra cuántos elementos hay, el primero y el último (usa `at()`).
2. Agrega `"queso"` al final y `"tortillas"` al inicio.
3. Inserta `"té"` después de `"leche"` usando `indexOf()` y `splice()`.
4. Elimina `"azúcar"` comprobando antes que exista.
5. Intenta eliminar `"jamón"` sin modificar el array por error.
6. Crea `primeros` con los tres primeros elementos usando `slice()`.
7. Recorre la lista con `entries()` y muestra cada elemento numerado desde 1.

Salida esperada del paso 7:

```text
1. tortillas
2. pan
3. leche
4. té
5. huevos
6. café
7. queso
```

---

## 6. Avance: `map()` y `filter()`

En Python transformábamos y filtrábamos listas con *comprehensions*. En JavaScript usamos métodos que reciben una función. La sintaxis `(n) => n * 2` es una **arrow function**: una función pequeña que recibe `n` y devuelve `n * 2`, parecida a `lambda n: n * 2`. La estudiaremos a fondo en la clase 04.

```python
dobles = [n * 2 for n in numeros]
pares = [n for n in numeros if n % 2 == 0]
```

```javascript
const numeros = [1, 2, 3, 4, 5, 6];

const dobles = numeros.map((n) => n * 2);
const pares = numeros.filter((n) => n % 2 === 0);

console.log(dobles); // [ 2, 4, 6, 8, 10, 12 ]
console.log(pares);  // [ 2, 4, 6 ]
```

- `map()` devuelve un array nuevo con el resultado de aplicar la función a cada elemento.
- `filter()` devuelve un array nuevo solo con los elementos para los que la función devuelve un valor verdadero.

Ninguno modifica el original. Para contar elementos que cumplen una condición (como `lista.count(x)`), filtra y consulta `length`:

```javascript
const temperaturas = [22, 27, 25, 27];
console.log(temperaturas.filter((t) => t === 27).length); // 2
```

---

## 7. Ordenar

### `sort()` ordena como texto

En Python, `sorted([10, 9, 1, 25])` devolvía `[1, 9, 10, 25]`. En JavaScript, `sort()` sin argumentos convierte los elementos a texto y los compara carácter por carácter:

```javascript
const numeros = [10, 9, 1, 25];
numeros.sort();
console.log(numeros); // [ 1, 10, 25, 9 ]
```

`"10"` va antes que `"9"` porque el carácter `"1"` va antes que `"9"`. Para ordenar números, entrega una **función de comparación**:

```javascript
const numeros = [10, 9, 1, 25];

numeros.sort((a, b) => a - b);
console.log(numeros); // [ 1, 9, 10, 25 ]

numeros.sort((a, b) => b - a);
console.log(numeros); // [ 25, 10, 9, 1 ]
```

La función recibe dos elementos y devuelve un número: negativo si `a` va antes, positivo si `b` va antes y `0` si da igual. `a - b` produce orden ascendente y `b - a` descendente, como `reverse=True`.

### Ordenar textos

`sort()` sin argumentos compara códigos de caracteres: las mayúsculas van antes que las minúsculas y las letras acentuadas quedan al final. `localeCompare()` compara como lo haría un diccionario:

```javascript
const nombres = ["Óscar", "ana", "Zoe", "Ángel"];

console.log(nombres.toSorted());
console.log(nombres.toSorted((a, b) => a.localeCompare(b)));
```

```text
[ 'Zoe', 'ana', 'Ángel', 'Óscar' ]
[ 'ana', 'Ángel', 'Óscar', 'Zoe' ]
```

### Ordenar sin modificar

`sort()` modifica el array, como `lista.sort()`. `toSorted()` devuelve un array nuevo y conserva el original, como `sorted()`:

```javascript
const precios = [900, 450, 15000];

const ordenados = precios.toSorted((a, b) => a - b);

console.log(ordenados); // [ 450, 900, 15000 ]
console.log(precios);   // [ 900, 450, 15000 ]
```

Lo mismo ocurre con `reverse()`, que invierte el array original, y `toReversed()`, que devuelve uno nuevo.

| En Python | En JavaScript |
|---|---|
| `lista.sort()` | `lista.sort((a, b) => a - b)` |
| `lista.sort(reverse=True)` | `lista.sort((a, b) => b - a)` |
| `sorted(lista)` | `lista.toSorted((a, b) => a - b)` |
| `lista.reverse()` | `lista.reverse()` |
| `list(reversed(lista))` | `lista.toReversed()` |
| `max(lista)` | `Math.max(...lista)` |

---

## 8. ¿Y las tuplas?

JavaScript no tiene tuplas. Usamos arrays, y para impedir cambios podemos congelarlos con `Object.freeze()`:

```javascript
const coordenadas = Object.freeze([19.43, -99.13]);

try {
  coordenadas[0] = 0;
} catch (error) {
  console.log(error.message);
}

console.log(coordenadas); // [ 19.43, -99.13 ]
```

```text
Cannot assign to read only property '0' of object '[object Array]'
[ 19.43, -99.13 ]
```

`try`/`catch` captura el error, como `try`/`except`; lo veremos en la clase 05.

| Declaración | ¿Se puede reasignar la variable? | ¿Se puede cambiar el contenido? |
|---|---|---|
| `let lista = [1, 2]` | Sí | Sí |
| `const lista = [1, 2]` | No | Sí |
| `const lista = Object.freeze([1, 2])` | No | No |

Dentro de la carpeta del curso, modificar un array congelado lanza `TypeError`: el `package.json` de la raíz declara los archivos como módulos ES (clase 06), que siempre usan el **modo estricto** de JavaScript. En el REPL, o en un archivo suelto fuera de esa carpeta y sin `import`, el cambio se **ignora en silencio** sin error. `Object.freeze()` solo congela el primer nivel: si contiene arrays internos, estos se pueden modificar.

Para "modificar" un array congelado, crea uno nuevo: `const nuevas = [...coordenadas, 2240];`.

---

## 9. Desestructuración de arrays

La desestructuración extrae elementos en variables, como el desempaquetado de Python:

```python
a, b, c = (10, 20, 30)
primero, *resto = (10, 20, 30)
```

```javascript
const [a, b, c] = [10, 20, 30];
console.log(a, b, c); // 10 20 30

const [primero, ...resto] = [10, 20, 30];
console.log(primero, resto); // 10 [ 20, 30 ]
```

Diferencias con Python:

- Si hay menos elementos que variables, las sobrantes valen `undefined` en lugar de lanzar `ValueError`.
- Puedes dar valores por defecto y saltar posiciones con comas.

```javascript
const [x, y, z = 0] = [5, 8];
console.log(x, y, z); // 5 8 0

const [, segundo] = ["Laptop", "Mouse", "Teclado"];
console.log(segundo); // Mouse
```

### Intercambiar valores

```python
a, b = b, a
```

```javascript
let izquierda = "Laptop";
let derecha = "Mouse";

[izquierda, derecha] = [derecha, izquierda];
console.log(izquierda, derecha); // Mouse Laptop
```

Las variables se declaran con `let` porque cambian.

---

## Actividad 2: Ordenar y desestructurar

Crea `ventas.js` con:

```javascript
const ventas = [1200, 450, 9800, 300, 4200, 450, 15000];
```

1. Obtén las ventas ordenadas de mayor a menor con `toSorted()` y comprueba que `ventas` no cambió.
2. Desestructura las tres ventas más altas en `oro`, `plata` y `bronce`, y el resto en `otras`.
3. Calcula con `filter()` cuántas ventas superan `1000`.
4. Crea con `map()` un array con las ventas más 16 % de IVA.
5. Obtén la venta máxima y la mínima con `Math.max()` y `Math.min()`.
6. Explica en un comentario por qué `ventas.sort()` sin argumentos no daría el resultado correcto.

Resultados esperados de los pasos 2, 3 y 5:

```text
Oro: 15000, plata: 9800, bronce: 4200
Otras: 1200, 450, 450, 300
Ventas mayores a 1000: 4
Máxima: 15000, mínima: 300
```

---

## 10. `Set`

Un `Set` guarda valores **únicos**, como un `set` de Python. Se crea con `new Set()`, opcionalmente a partir de un array:

```javascript
const etiquetas = new Set(["oferta", "nuevo", "oferta"]);

console.log(etiquetas);      // Set(2) { 'oferta', 'nuevo' }
console.log(etiquetas.size); // 2

etiquetas.add("envío gratis");
console.log(etiquetas.has("nuevo"));    // true

console.log(etiquetas.delete("nuevo"));  // true
console.log(etiquetas.delete("agotado")); // false

for (const etiqueta of etiquetas) {
  console.log(etiqueta);
}
```

`new` crea un objeto a partir de una clase, como llamar a `set()` en Python; lo volveremos a ver en la clase 05.

| En Python | En JavaScript |
|---|---|
| `set()` | `new Set()` |
| `set(lista)` | `new Set(lista)` |
| `len(conjunto)` | `conjunto.size` |
| `conjunto.add(x)` | `conjunto.add(x)` |
| `x in conjunto` | `conjunto.has(x)` |
| `conjunto.discard(x)` | `conjunto.delete(x)` (devuelve `true` o `false`) |
| `conjunto.remove(x)` | No existe: `delete()` nunca lanza error |
| `list(conjunto)` | `[...conjunto]` |

Dos diferencias con Python:

- No hay sintaxis literal: `{}` crea un objeto (clase 03), no un conjunto.
- Un `Set` conserva el **orden de inserción**. Al recorrerlo o mostrarlo, los elementos aparecen en el orden en que se agregaron.

### Eliminar duplicados

```javascript
const numeros = [3, 1, 3, 2, 1];

const unicos = [...new Set(numeros)];
console.log(unicos); // [ 3, 1, 2 ]
```

Convertimos a `Set` para quitar duplicados y lo expandimos de nuevo a un array. A diferencia de Python, se conserva el orden de la primera aparición.

### Operaciones entre conjuntos

```javascript
const a = new Set([1, 2, 3]);
const b = new Set([2, 3, 4]);

console.log(a.union(b));               // Set(4) { 1, 2, 3, 4 }
console.log(a.intersection(b));        // Set(2) { 2, 3 }
console.log(a.difference(b));          // Set(1) { 1 }
console.log(a.symmetricDifference(b)); // Set(2) { 1, 4 }
console.log(new Set([2, 3]).isSubsetOf(a)); // true
```

| En Python | En JavaScript |
|---|---|
| `a \| b` o `a.union(b)` | `a.union(b)` |
| `a & b` | `a.intersection(b)` |
| `a - b` | `a.difference(b)` |
| `a ^ b` | `a.symmetricDifference(b)` |
| `a <= b` | `a.isSubsetOf(b)` |

JavaScript no tiene operadores para conjuntos; usa los métodos. Todos devuelven un `Set` nuevo y requieren Node.js 22 o superior.

---

## Errores comunes viniendo de Python

1. **Esperar `IndexError`.** `lista[99]` devuelve `undefined`. Compruébalo antes de usarlo.
2. **Usar `lista[-1]`.** Devuelve `undefined`; usa `lista.at(-1)`.
3. **Ordenar números con `sort()` sin comparador.** `[10, 9, 1].sort()` da `[1, 10, 9]`. Usa `(a, b) => a - b`.
4. **Olvidar que `sort()` y `reverse()` modifican el array.** Usa `toSorted()` y `toReversed()` para conservar el original.
5. **Unir arrays con `+`.** Produce texto. Usa `[...a, ...b]`.
6. **Confundir `slice()` y `splice()`.** El primero copia; el segundo modifica.
7. **Creer que `const` congela el array.** Solo impide reasignar la variable; usa `Object.freeze()`.
8. **Comprobar un array vacío con `if (lista)`.** `[]` es *truthy*; usa `lista.length === 0`.
9. **Escribir `{}` para un conjunto vacío.** Crea un objeto; usa `new Set()`.

---

## Cheat sheet

```javascript
// Crear
const lista = ["a", "b", "c"];
const vacia = [];

// Acceder
lista[0];
lista.at(-1);
lista.length;

// Buscar
lista.includes("b");
lista.indexOf("b"); // -1 si no existe

// Agregar y quitar
lista.push("d");
lista.unshift("z");
lista.pop();
lista.shift();
lista.splice(1, 0, "x"); // insertar
lista.splice(1, 1);      // eliminar

// Recortar, unir y copiar
lista.slice(0, 2);
const unida = [...lista, ...["e", "f"]];
const copia = [...lista];
const profunda = structuredClone([[1, 2], [3, 4]]);

// Recorrer
for (const elemento of lista) {
  console.log(elemento);
}
for (const [indice, elemento] of lista.entries()) {
  console.log(indice, elemento);
}
lista.join(", ");

// Transformar y filtrar (clase 04)
[1, 2, 3].map((n) => n * 2);
[1, 2, 3].filter((n) => n > 1);

// Ordenar
const numeros = [10, 9, 1];
numeros.toSorted((a, b) => a - b);
numeros.sort((a, b) => b - a);
["b", "a"].toSorted((a, b) => a.localeCompare(b));

// Inmutable y desestructuración
const fijo = Object.freeze([1, 2]);
const [primero, ...resto] = [10, 20, 30];

// Set
const conjunto = new Set([1, 2, 2]);
conjunto.add(3);
conjunto.has(3);
conjunto.delete(3);
conjunto.size;
const sinDuplicados = [...new Set([1, 1, 2])];
new Set([1, 2]).union(new Set([2, 3]));
new Set([1, 2]).intersection(new Set([2, 3]));
new Set([1, 2]).difference(new Set([2, 3]));
new Set([1, 2]).symmetricDifference(new Set([2, 3]));
```

---

## Ejercicio final

### Parte A: Análisis de temperaturas

Crea `temperaturas.js` con un array congelado, como la tupla del curso de Python:

```javascript
const temperaturas = Object.freeze([
  22, 24, 21, 25, 27, 27, 23,
  20, 22, 24, 27, 26, 21, 27,
]);
```

La coma después del último elemento es válida en JavaScript y facilita agregar líneas.

Crea un programa que:

1. Muestre cuántas temperaturas hay, la primera y la última.
2. Cuente cuántas veces apareció `27` y la primera posición donde apareció.
3. Muestre la primera semana con `slice()` y `join()`.
4. Desestructure las tres primeras temperaturas en `dia1`, `dia2` y `dia3`, y el resto en `resto`.
5. Recorra el array con `entries()` y muestre el número de día de cada temperatura mayor a `24`.
6. Calcule el promedio con un `for...of` y un acumulador.
7. Obtenga la mínima y la máxima a partir de una copia ordenada con `toSorted()`.
8. Muestre los valores distintos ordenados usando `Set`.
9. Cree un array nuevo con una temperatura adicional de `28`, sin modificar el original.

#### Salida esperada

```text
Total de registros: 14
Primera temperatura: 22
Última temperatura: 27
Veces que apareció 27: 4
Primera posición de 27: 4

Primera semana:
22, 24, 21, 25, 27, 27, 23

Primeros tres días:
22 24 21
Días restantes: 11

Temperaturas mayores a 24:
Día 4: 25
Día 5: 27
Día 6: 27
Día 11: 27
Día 12: 26
Día 14: 27

Promedio: 24.00
Mínima: 20
Máxima: 27
Valores distintos: 20, 21, 22, 23, 24, 25, 26, 27

Nuevo registro:
Registros originales: 14
Registros nuevos: 15
Última temperatura nueva: 28
```

### Parte B: Comparación de usuarios

Crea `usuarios.js` con las listas del curso de Python:

```javascript
const appA = [
  "ana", "carlos", "maria", "pedro",
  "lucia", "carlos", "diego", "ana",
  "sofia", "fernando",
];

const appB = [
  "maria", "pedro", "jorge", "lucia",
  "sofia", "elena", "jorge", "raul",
  "pedro", "monica",
];
```

Crea un programa que:

1. Convierta ambas listas a `Set`.
2. Muestre cuántos usuarios únicos tiene cada aplicación.
3. Obtenga los usuarios de ambas aplicaciones, todos los usuarios, los que solo usan App A, los que solo usan App B y los que usan una aplicación pero no ambas.
4. Muestre cada resultado con su tamaño y sus nombres unidos con `join()`.
5. Agregue `"adrian"` a App A.
6. Intente eliminar `"roberto"` de App B y muestre el valor que devuelve `delete()`.

Para no repetir el código del paso 4, puedes guardar los resultados en un array de pares `[titulo, conjunto]` y recorrerlo con `for (const [titulo, conjunto] of resultados)`.

#### Salida esperada

Como `Set` conserva el orden de inserción, la salida es siempre la misma:

```text
Usuarios únicos App A: 8
Usuarios únicos App B: 8

Usuarios en ambas (4):
maria, pedro, lucia, sofia

Todos los usuarios (12):
ana, carlos, maria, pedro, lucia, diego, sofia, fernando, jorge, elena, raul, monica

Solo App A (4):
ana, carlos, diego, fernando

Solo App B (4):
jorge, elena, raul, monica

En una aplicación pero no en ambas (8):
ana, carlos, diego, fernando, jorge, elena, raul, monica

App A después de agregar a adrian: 9
¿Se eliminó a roberto de App B? false
¿adrian usa App B? false
```

### Práctica opcional

1. En la parte A, intenta hacer `temperaturas.push(28)` dentro de un `try`/`catch` y muestra el mensaje del error.
2. En la parte B, ordena alfabéticamente la lista de todos los usuarios con `localeCompare()` antes de mostrarla.
3. Crea un `Set` con los usuarios que aparecen repetidos dentro de `appA` (pista: recorre el array y usa un segundo `Set` con los ya vistos).

---

[Índice del curso](README.md)
