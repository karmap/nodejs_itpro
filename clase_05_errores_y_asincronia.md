# Clase 05: Errores y asincronía

**Duración:** 2 horas

> Los bloques de desarrollo se leen en orden dentro de cada ejemplo. Las plantillas con `...` se completan en las actividades; las cheat sheets reúnen operaciones independientes. Los programas completos incluyen sus imports.

### Objetivos

- Manejar errores con `try`, `catch` y `finally`.
- Lanzar errores con `throw` y crear tipos de error propios.
- Distinguir errores con `instanceof` y conservar su causa.
- Reconocer qué operaciones no lanzan errores en JavaScript y deben validarse.
- Entender el event loop y por qué Node no se detiene a esperar.
- Usar Promises con `async` y `await`.
- Ejecutar tareas en secuencia o en paralelo con `Promise.all()` y `Promise.allSettled()`.
- Leer datos del usuario con `node:readline/promises`.

---

## 1. `try` / `catch` / `finally`

En Python escribíamos:

```python
try:
    resultado = dividir(10, 0)
except ValueError as error:
    print("Error:", error)
finally:
    print("Operación terminada")
```

En JavaScript la estructura es la misma, con otras palabras y llaves:

```javascript
function dividir(a, b) {
  if (b === 0) {
    throw new Error("No se puede dividir entre cero");
  }
  return a / b;
}

try {
  console.log(dividir(10, 2));
  console.log(dividir(10, 0));
  console.log("Esta línea no se ejecuta");
} catch (error) {
  console.log("Error:", error.message);
} finally {
  console.log("Operación terminada");
}
```

Salida:

```text
5
Error: No se puede dividir entre cero
Operación terminada
```

| En Python | En JavaScript |
|---|---|
| `try:` | `try { ... }` |
| `except ValueError as error:` | `catch (error) { ... }` |
| `else:` | No existe; escribe ese código al final del `try`. |
| `finally:` | `finally { ... }` |
| `raise ValueError("...")` | `throw new Error("...")` |
| `str(error)` | `error.message` |
| `type(error).__name__` | `error.name` |

`throw` detiene la función, igual que `raise`, y busca el `catch` más cercano. Si ninguno lo captura, el programa termina y Node muestra el error con su traza.

`new Error("...")` crea un objeto de error. Técnicamente puedes lanzar cualquier valor (`throw "falló"`), pero no lo hagas: un texto no tiene `message`, `name` ni traza. Lanza siempre objetos `Error` o de una clase derivada.

---

## 2. Un solo `catch`: distinguir con `instanceof`

En Python podíamos poner varios `except`, uno por tipo de error. En JavaScript un `try` tiene **un solo** `catch`, que recibe cualquier error. Para tratar cada tipo de forma diferente usamos `instanceof`:

```javascript
try {
  JSON.parse("{precio: 10}");
} catch (error) {
  if (error instanceof SyntaxError) {
    console.log("El texto no es JSON válido");
  } else {
    throw error;
  }
}
```

`error instanceof SyntaxError` comprueba si el error es de ese tipo o de uno derivado de él. Volver a lanzar con `throw error` lo que no sabemos manejar equivale a no capturarlo en Python: evitamos ocultar errores inesperados.

### Errores incorporados frecuentes

| Error | Cuándo aparece |
|---|---|
| `SyntaxError` | `JSON.parse()` recibe texto que no es JSON válido. |
| `TypeError` | Usas un valor de un tipo incompatible, como leer una propiedad de `undefined` o llamar algo que no es función. |
| `RangeError` | Un número está fuera del rango permitido, como `toFixed(200)`. |
| `ReferenceError` | Usas una variable que no existe. |

```javascript
try {
  const producto = undefined;
  console.log(producto.nombre);
} catch (error) {
  console.log(error.name, "-", error.message);
}
```

Salida:

```text
TypeError - Cannot read properties of undefined (reading 'nombre')
```

Los mensajes de los errores incorporados están en inglés. Los mensajes de nuestros errores pueden estar en español.

---

## 3. Lo que en JavaScript no lanza errores

En el curso de Python muchos problemas producían una excepción inmediata. En JavaScript varios de ellos devuelven un valor especial y el programa sigue:

| Operación | Python | JavaScript |
|---|---|---|
| Posición inexistente | `IndexError` | `[1, 2][5]` devuelve `undefined` |
| Clave inexistente | `KeyError` | `{}.precio` devuelve `undefined` |
| Convertir texto inválido | `int("abc")` lanza `ValueError` | `Number("abc")` devuelve `NaN` |
| Mezclar tipos | `"10" * 2` repite el texto | `"10" * 2` devuelve `20` |
| Faltan argumentos | `TypeError` | El parámetro vale `undefined` |

El error aparece después, lejos de su causa: por ejemplo, un `undefined` llega a otra función y allí provoca un `TypeError`. Por eso en JavaScript **validamos explícitamente** los datos que vienen del usuario, de archivos o de otras APIs, y lanzamos nuestro propio error con un mensaje claro.

---

## 4. Errores propios

En Python podíamos lanzar `ValueError` para datos inválidos. En JavaScript es habitual crear una clase de error con un nombre propio:

```javascript
class ValidationError extends Error {
  constructor(message, options) {
    super(message, options);
    this.name = "ValidationError";
  }
}
```

Solo necesitamos esta forma mínima de `class`:

- `class ValidationError extends Error` crea un tipo nuevo que hereda de `Error`, como `class ProductInput(BaseModel)` heredaba de Pydantic.
- `constructor` es el equivalente de `__init__`: se ejecuta al usar `new ValidationError(...)`.
- `super(message, options)` llama al constructor de `Error`, como `super().__init__()`.
- `this` es el equivalente de `self`. Asignamos `this.name` para que el error se muestre con su propio nombre.

Uso:

```javascript
class ValidationError extends Error {
  constructor(message, options) {
    super(message, options);
    this.name = "ValidationError";
  }
}

function leerPrecio(texto) {
  const precio = Number(texto);
  if (texto.trim() === "" || Number.isNaN(precio)) {
    throw new ValidationError(`Precio inválido: "${texto}"`);
  }
  if (precio < 0) {
    throw new ValidationError("El precio no puede ser negativo");
  }
  return precio;
}

for (const entrada of ["450", "abc", "-5", ""]) {
  try {
    console.log("Precio:", leerPrecio(entrada));
  } catch (error) {
    if (error instanceof ValidationError) {
      console.log("Dato rechazado:", error.message);
    } else {
      throw error;
    }
  }
}
```

Salida:

```text
Precio: 450
Dato rechazado: Precio inválido: "abc"
Dato rechazado: El precio no puede ser negativo
Dato rechazado: Precio inválido: ""
```

`Number("")` devuelve `0`, no `NaN`. Por eso comprobamos también el texto vacío.

### Conservar la causa

En Python escribíamos `raise ... from error` para conservar el error original. En JavaScript se pasa como opción `cause`:

```javascript
function cargarConfiguracion(texto) {
  try {
    return JSON.parse(texto);
  } catch (error) {
    throw new Error("La configuración no es JSON válido", { cause: error });
  }
}

try {
  cargarConfiguracion("{puerto: 8000}");
} catch (error) {
  console.log(error.message);
  console.log("Causa:", error.cause.name);
}
```

Salida:

```text
La configuración no es JSON válido
Causa: SyntaxError
```

El mensaje nuevo explica el problema en términos del programa; `error.cause` guarda el detalle técnico para depurar. Nuestra clase `ValidationError` recibe `options` y lo entrega a `super()`, así que también acepta `{ cause: error }`.

---

## Actividad 1: Convertir cantidades

Crea `cantidades.js` con la clase `ValidationError` y esta función:

```javascript
function convertirCantidad(texto) {
  // Completa esta función durante la actividad.
}
```

Debe devolver un número entero no negativo o lanzar `ValidationError` con un mensaje distinto para cada caso:

| Entrada | Resultado |
|---|---|
| `"3"` | `3` |
| `"  7 "` | `7` |
| `""` o `"   "` | Error: escribe una cantidad. |
| `"abc"` | Error: no es un número. |
| `"2.5"` | Error: debe ser un número entero. |
| `"-1"` | Error: no puede ser negativa. |

Recorre esas entradas con `for...of`, captura solo `ValidationError` y muestra el resultado o el mensaje. Usa `Number.isNaN()` y `Number.isInteger()`.

---

## 5. Un hilo y el event loop

Hasta ahora todo el código se ejecutó en orden, línea por línea. En el curso de Python, `requests.get()` y `time.sleep()` detenían el programa hasta terminar.

Node funciona diferente. Ejecuta nuestro código JavaScript en **un solo hilo** y no se detiene a esperar operaciones lentas, como temporizadores, red o archivos. Las registra, sigue con el resto del código y ejecuta la continuación cuando la operación termina. El mecanismo que reparte ese trabajo se llama **event loop**:

```text
Tu código se ejecuta completo
        ↓
Operaciones lentas quedan pendientes (red, archivos, temporizadores)
        ↓
Event loop: ¿terminó alguna?
        ↓
Ejecuta su continuación
        ↓
Repite mientras queden operaciones pendientes
```

`setTimeout(funcion, ms)` registra una función para ejecutarla después de, al menos, `ms` milisegundos:

```javascript
console.log("1. Inicio");

setTimeout(() => {
  console.log("3. Terminó el temporizador");
}, 1000);

console.log("2. Fin del programa principal");
```

Salida:

```text
1. Inicio
2. Fin del programa principal
3. Terminó el temporizador
```

El programa no espera un segundo en la línea de `setTimeout`. La función se ejecuta después, cuando el código principal ya terminó. El proceso de Node termina cuando no quedan operaciones pendientes.

Incluso con `0` milisegundos, la función espera a que termine el código actual:

```javascript
console.log("A");
setTimeout(() => console.log("C"), 0);
console.log("B");
```

Imprime `A`, `B` y `C`.

### No bloquees el hilo

Como hay un solo hilo, un cálculo síncrono largo impide que se ejecute cualquier otra cosa:

```javascript
const inicio = Date.now();

setTimeout(() => {
  console.log(`Temporizador de 100 ms ejecutado a los ${Date.now() - inicio} ms`);
}, 100);

// Trabajo síncrono que ocupa el único hilo durante 2 segundos.
while (Date.now() - inicio < 2000) {
  // No hace nada útil: solo mantiene ocupado el hilo.
}

console.log("Terminó el ciclo");
```

Salida aproximada:

```text
Terminó el ciclo
Temporizador de 100 ms ejecutado a los 2001 ms
```

El temporizador estaba listo a los 100 ms, pero tuvo que esperar al ciclo. En la clase 08 veremos que un servidor bloqueado así deja de responder a todos sus clientes.

---

## 6. De callbacks a Promises

Un **callback** es una función que entregamos para que otro código la llame después, como la función que recibe `setTimeout`. Muchas APIs antiguas de Node usaban callbacks con la forma `(error, resultado) => { ... }`. Cuando una operación dependía de otra, los callbacks quedaban anidados y el código se volvía difícil de leer y de manejar con errores.

Una **Promise** es un objeto que representa un resultado futuro. Tiene tres estados:

```text
pending    → la operación sigue en curso
fulfilled  → terminó con un valor
rejected   → terminó con un error
```

Una Promise empieza `pending` y termina una sola vez, cumplida o rechazada.

Puedes crear una Promise con `new Promise()`. La función que recibe tiene dos parámetros: `resolve` la cumple con un valor y `reject` la rechaza con un error:

```javascript
function esperarMs(ms) {
  return new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}

function dividirDespues(a, b) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (b === 0) {
        reject(new Error("División entre cero"));
      } else {
        resolve(a / b);
      }
    }, 500);
  });
}
```

Este código convierte una API de callbacks en una Promise. En la práctica casi nunca lo escribirás: `fetch()`, `node:fs/promises` y otras APIs modernas ya devuelven Promises. Lo importante es saber **consumirlas**.

### `.then()` y `.catch()`

La forma original de consumir una Promise es registrar funciones:

```javascript
dividirDespues(10, 4)
  .then((resultado) => console.log("Resultado:", resultado))
  .catch((error) => console.log("Error:", error.message));
```

`.then()` recibe el valor cuando la Promise se cumple y `.catch()` recibe el error cuando se rechaza. Lo verás en documentación y código existente. En este curso usaremos `async` y `await`, que se leen como código secuencial.

---

## 7. `async` y `await`

`await` espera a que una Promise termine y devuelve su valor. Si la Promise se rechaza, `await` **lanza** el error, que podemos capturar con `try` / `catch`:

```javascript
async function main() {
  await esperarMs(500);
  console.log(await dividirDespues(10, 4));
  try {
    await dividirDespues(1, 0);
  } catch (error) {
    console.log("Rechazada:", error.message);
  }
}

main();
```

Salida (requiere las dos funciones de la sección anterior):

```text
2.5
Rechazada: División entre cero
```

Reglas:

- `await` solo se puede usar dentro de una función declarada con `async` o en el nivel superior de un módulo.
- Una función `async` **siempre devuelve una Promise**, aunque escribas `return 5`. Quien la llama debe usar `await` para obtener el valor.
- `await` pausa solo esa función. El event loop sigue atendiendo otras tareas mientras tanto.

| En Python (curso) | En Node |
|---|---|
| `time.sleep(1)` bloquea todo el programa | `await esperar(1000)` pausa solo esa función |
| `requests.get(url)` devuelve la respuesta | `await fetch(url)` devuelve la respuesta (clase 07) |
| `open()` y `read()` | `await readFile()` (clase 06) |

Si conoces `asyncio` de Python, la idea es la misma. La diferencia es que en Node casi todo el acceso a red y archivos es asíncrono, así que usaremos `await` constantemente.

### `await` en el nivel superior

En los módulos ES, el formato que usamos en el curso, `await` también funciona fuera de cualquier función. Node incluye una versión de `setTimeout` que devuelve una Promise en `node:timers/promises`:

```javascript
import { setTimeout as esperar } from "node:timers/promises";

console.log("Esperando...");
await esperar(1000);
console.log("Pasó un segundo");
```

`import { setTimeout as esperar }` importa la función y le cambia el nombre para no confundirla con el `setTimeout` de callbacks. En la clase 06 estudiaremos los imports en detalle; el prefijo `node:` indica un módulo incorporado, como la biblioteca estándar de Python. `esperarMs()` de la sección anterior hacía lo mismo; desde ahora usaremos `esperar()`.

### Olvidar `await`

Es el error más común al empezar:

```javascript
import { setTimeout as esperar } from "node:timers/promises";

async function obtenerPrecio(nombre) {
  await esperar(500);
  return { nombre, precio: 450 };
}

const sinAwait = obtenerPrecio("Mouse");
console.log(sinAwait);

const conAwait = await obtenerPrecio("Mouse");
console.log(conAwait);
```

Salida:

```text
Promise { <pending> }
{ nombre: 'Mouse', precio: 450 }
```

Sin `await` obtienes la Promise, no el producto. `sinAwait.precio` sería `undefined`, sin ningún error que te avise. Si ves `Promise { <pending> }` o un `undefined` inesperado, busca un `await` faltante.

### Errores con `await`

```javascript
import { setTimeout as esperar } from "node:timers/promises";

async function buscarProducto(id) {
  await esperar(300);
  if (id !== 1) {
    throw new Error(`Producto ${id} no encontrado`);
  }
  return { id: 1, nombre: "Laptop" };
}

try {
  const producto = await buscarProducto(1);
  console.log("Encontrado:", producto.nombre);
  await buscarProducto(99);
} catch (error) {
  console.log("Error:", error.message);
}
```

Salida:

```text
Encontrado: Laptop
Error: Producto 99 no encontrado
```

`throw` dentro de una función `async` rechaza su Promise; `await` lo convierte de nuevo en un error que `catch` captura. Sin `await`, el `try` terminaría antes de que ocurriera el error y no lo capturaría.

### Rechazos sin manejar

Si una Promise se rechaza y nadie la espera con `await` ni registra `.catch()`, Node termina el proceso con el error:

```javascript
async function fallar() {
  throw new Error("Algo salió mal");
}

fallar();
console.log("Esta línea sí se imprime");
```

La línea se imprime, pero inmediatamente después Node muestra `Error: Algo salió mal` y termina con un código distinto de cero. Espera siempre las Promises que puedan fallar.

---

## Actividad 2: Predecir el orden

Antes de ejecutar este programa, escribe en papel el orden en que aparecerán los números. Después ejecútalo y explica cada posición:

```javascript
import { setTimeout as esperar } from "node:timers/promises";

console.log("1");

setTimeout(() => console.log("2"), 0);

async function tarea() {
  console.log("3");
  await esperar(100);
  console.log("4");
}

tarea();
console.log("5");
```

Pistas: una función `async` se ejecuta de inmediato hasta su primer `await`; el temporizador de `0` ms espera a que termine el código actual.

---

## 8. Secuencial o en paralelo

Cuando varias tareas asíncronas son independientes, no es necesario esperar cada una antes de iniciar la siguiente.

```javascript
import { setTimeout as esperar } from "node:timers/promises";

async function consultarStock(nombre, ms) {
  await esperar(ms);
  return `${nombre}: disponible`;
}

console.time("Secuencial");
for (const nombre of ["Laptop", "Mouse", "Monitor"]) {
  console.log(await consultarStock(nombre, 1000));
}
console.timeEnd("Secuencial");

console.time("Paralelo");
const resultados = await Promise.all([
  consultarStock("Laptop", 1000),
  consultarStock("Mouse", 1000),
  consultarStock("Monitor", 1000)
]);
console.log(resultados);
console.timeEnd("Paralelo");
```

Salida aproximada:

```text
Laptop: disponible
Mouse: disponible
Monitor: disponible
Secuencial: 3.009s
[ 'Laptop: disponible', 'Mouse: disponible', 'Monitor: disponible' ]
Paralelo: 1.002s
```

`console.time(etiqueta)` y `console.timeEnd(etiqueta)` miden el tiempo entre ambas llamadas.

- **Secuencial:** `for...of` con `await` espera cada consulta antes de iniciar la siguiente. Tarda la suma de los tiempos.
- **Paralelo:** las tres llamadas se inician de inmediato y `Promise.all()` espera a que todas terminen. Tarda aproximadamente lo que la más lenta. Devuelve los resultados en el mismo orden del array, aunque terminen en otro orden.

```text
Secuencial:  [Laptop──][Mouse───][Monitor─]   ≈ 3 s
Paralelo:    [Laptop──]
             [Mouse───]                        ≈ 1 s
             [Monitor─]
```

Con `map()` podemos crear el array de Promises a partir de datos:

```javascript
const nombres = ["Laptop", "Mouse", "Monitor"];
const resultados = await Promise.all(
  nombres.map((nombre) => consultarStock(nombre, 1000))
);
```

Usa la versión secuencial cuando una tarea necesita el resultado de la anterior, o cuando un servicio limita cuántas peticiones aceptas enviar a la vez. En la clase 07 analizaremos reseñas con IA en secuencia por esa razón.

### `Promise.all()` frente a `Promise.allSettled()`

`Promise.all()` se rechaza en cuanto **una** Promise falla, y perdemos los resultados de las demás. `Promise.allSettled()` espera a todas y describe cada resultado:

```javascript
import { setTimeout as esperar } from "node:timers/promises";

async function consultarStock(nombre) {
  await esperar(500);
  if (nombre === "Audifonos") {
    throw new Error("Audifonos sin stock");
  }
  return `${nombre}: disponible`;
}

const nombres = ["Laptop", "Audifonos", "Mouse"];

try {
  await Promise.all(nombres.map((nombre) => consultarStock(nombre)));
} catch (error) {
  console.log("Promise.all falló:", error.message);
}

const resultados = await Promise.allSettled(
  nombres.map((nombre) => consultarStock(nombre))
);
for (const resultado of resultados) {
  if (resultado.status === "fulfilled") {
    console.log("OK:", resultado.value);
  } else {
    console.log("Falló:", resultado.reason.message);
  }
}
```

Salida:

```text
Promise.all falló: Audifonos sin stock
OK: Laptop: disponible
Falló: Audifonos sin stock
OK: Mouse: disponible
```

Cada elemento de `allSettled()` es un objeto con `status`: `"fulfilled"` incluye `value` y `"rejected"` incluye `reason`, el error.

| Necesitas | Usa |
|---|---|
| Todos los resultados o ninguno | `Promise.all()` |
| Cada resultado, aunque algunos fallen | `Promise.allSettled()` |

---

## Actividad 3: Medir tiempos

1. Crea `consultarPrecio(nombre)`, que espere 800 ms con `esperar()` y devuelva el precio del producto desde un array con los datos de la clase 04.
2. Consulta Laptop, Mouse, Teclado y Monitor en secuencia y mide el tiempo.
3. Repite con `Promise.all()` y compara.
4. Haz que `consultarPrecio()` lance un error si el producto no existe. Agrega `"Tablet"` a la lista y compara el comportamiento de `Promise.all()` y `Promise.allSettled()`.

---

## 9. Entrada del usuario con `readline`

Python tenía `input()`. Node no tiene una función síncrona equivalente: leer la terminal también es una operación que espera. El módulo `node:readline/promises` ofrece `question()`, que devuelve una Promise con el texto escrito:

```javascript
import * as readline from "node:readline/promises";
import { stdin as input, stdout as output } from "node:process";

const rl = readline.createInterface({ input, output });

const nombre = await rl.question("¿Cómo te llamas? ");
const edadTexto = await rl.question("¿Cuántos años tienes? ");
const edad = Number(edadTexto);

if (edadTexto.trim() === "" || Number.isNaN(edad)) {
  console.log("La edad debe ser un número");
} else {
  console.log(`Hola, ${nombre.trim()}. El próximo año tendrás ${edad + 1}.`);
}

rl.close();
```

- `import * as readline` reúne todas las funciones del módulo bajo el nombre `readline`.
- `stdin` y `stdout` son la entrada y la salida de la terminal. Los renombramos como `input` y `output`, los nombres que espera `createInterface()`.
- `createInterface()` crea la interfaz de lectura.
- `await rl.question(texto)` muestra el texto y espera una línea, como `input(texto)`. Devuelve siempre una cadena, sin el salto de línea final.
- `rl.close()` libera la terminal. Si lo olvidas, el programa no termina: la interfaz abierta cuenta como una operación pendiente.

| En Python | En Node |
|---|---|
| `nombre = input("Nombre: ")` | `const nombre = await rl.question("Nombre: ");` |
| `int(input("Edad: "))` | `Number(await rl.question("Edad: "))` y validar `NaN` |
| No necesita cierre | `rl.close()` al terminar |

### Menú en bucle

```javascript
import * as readline from "node:readline/promises";
import { stdin as input, stdout as output } from "node:process";

const rl = readline.createInterface({ input, output });
const productos = ["Laptop", "Mouse"];

while (true) {
  console.log("\n1. Ver productos");
  console.log("2. Agregar producto");
  console.log("3. Salir");
  const opcion = (await rl.question("Opción: ")).trim();

  if (opcion === "1") {
    console.log(productos.join(", "));
  } else if (opcion === "2") {
    const nombre = (await rl.question("Nombre: ")).trim();
    if (nombre) {
      productos.push(nombre);
      console.log("Producto agregado");
    } else {
      console.log("El nombre no puede estar vacío");
    }
  } else if (opcion === "3") {
    break;
  } else {
    console.log("Opción no válida");
  }
}

rl.close();
console.log("Hasta luego");
```

`(await rl.question("Opción: ")).trim()` necesita los paréntesis: primero esperamos el texto y después llamamos a `trim()`. Sin ellos, `trim()` se aplicaría a la Promise.

Para interrumpir un programa que no responde, usa `Ctrl+C`.

---

## Errores comunes viniendo de Python

1. **Olvidar `await`.** Obtienes `Promise { <pending> }` o `undefined` en lugar del dato, sin ningún error.
2. **Escribir varios `catch`.** Solo hay uno por `try`; distingue los tipos con `instanceof` y vuelve a lanzar lo que no manejas.
3. **Esperar una excepción que no llega.** `Number("abc")`, `arr[99]` u `obj.campoInexistente` no lanzan; valida y lanza tu propio error.
4. **Lanzar textos.** `throw "error"` funciona, pero pierde `message`, `name` y la traza. Usa `throw new Error("...")`.
5. **Creer que `setTimeout` pausa el programa.** Registra una función para después; el código siguiente se ejecuta de inmediato. Para pausar una función `async`, usa `await esperar(ms)`.
6. **`await` dentro de un `for...of` cuando las tareas son independientes.** Funciona, pero tarda la suma de los tiempos; considera `Promise.all()`.
7. **No cerrar `readline`.** El programa queda abierto esperando. Llama a `rl.close()`.
8. **Bloquear el hilo.** Un ciclo síncrono largo detiene temporizadores, peticiones y entrada del usuario.

---

## Cheat sheet

| Tarea | Python | JavaScript |
|---|---|---|
| Capturar | `try: ... except E as e:` | `try { ... } catch (error) { ... }` |
| Distinguir tipos | Varios `except` | `if (error instanceof SyntaxError)` |
| Siempre ejecutar | `finally:` | `finally { ... }` |
| Lanzar | `raise ValueError("...")` | `throw new Error("...")` |
| Mensaje | `str(error)` | `error.message` |
| Causa | `raise Nuevo() from error` | `new Error("...", { cause: error })` |
| Error propio | `class MiError(Exception)` | `class MiError extends Error` |
| Pausar | `time.sleep(1)` | `await esperar(1000)` |
| Función asíncrona | `async def f():` | `async function f() { ... }` |
| Esperar varias | `asyncio.gather(...)` | `await Promise.all([...])` |
| Leer de la terminal | `input("...")` | `await rl.question("...")` |

```javascript
import * as readline from "node:readline/promises";
import { stdin as input, stdout as output } from "node:process";
import { setTimeout as esperar } from "node:timers/promises";

class ValidationError extends Error {
  constructor(message, options) {
    super(message, options);
    this.name = "ValidationError";
  }
}

async function tarea(valor) {
  await esperar(500);
  if (valor < 0) {
    throw new ValidationError("Valor negativo");
  }
  return valor * 2;
}

try {
  console.log(await tarea(5));
  console.log(await Promise.all([tarea(1), tarea(2)]));
  const resultados = await Promise.allSettled([tarea(1), tarea(-1)]);
  console.log(resultados.map((r) => r.status));
} catch (error) {
  if (error instanceof ValidationError) {
    console.log(error.message);
  } else {
    throw error;
  }
}

const rl = readline.createInterface({ input, output });
const texto = await rl.question("Escribe algo: ");
console.log(texto.trim());
rl.close();
```

---

## Ejercicio final: Simulador de pedidos

Crea `pedidos.js`. El programa simula un servicio lento que procesa pedidos, compara el procesamiento secuencial con el paralelo y ofrece un menú.

Datos:

```javascript
const productos = [
  { nombre: "Laptop", precio: 15000, stock: 5 },
  { nombre: "Mouse", precio: 450, stock: 12 },
  { nombre: "Teclado", precio: 900, stock: 8 },
  { nombre: "Monitor", precio: 4200, stock: 3 },
  { nombre: "Audifonos", precio: 1200, stock: 0 },
  { nombre: "Webcam", precio: 800, stock: 6 }
];

const pedidos = [
  { id: 1, producto: "Laptop", cantidad: 2 },
  { id: 2, producto: "Audifonos", cantidad: 1 },
  { id: 3, producto: "mouse", cantidad: 3 },
  { id: 4, producto: "Tablet", cantidad: 1 },
  { id: 5, producto: "Webcam", cantidad: 0 },
  { id: 6, producto: "Monitor", cantidad: 5 }
];
```

Crea:

```javascript
class ValidationError extends Error {
  // Completa esta clase durante la actividad.
}

class StockError extends Error {
  // Completa esta clase durante la actividad.
}

async function consultarProducto(nombre) {
  // Completa esta función durante la actividad.
}

async function procesarPedido(pedido) {
  // Completa esta función durante la actividad.
}

async function procesarSecuencial(lista) {
  // Completa esta función durante la actividad.
}

async function procesarParalelo(lista) {
  // Completa esta función durante la actividad.
}
```

### Requisitos

1. `ValidationError` y `StockError` derivan de `Error` y asignan su propio `name`.
2. `consultarProducto()` espera 500 ms con `esperar()`, busca el producto sin distinguir mayúsculas de minúsculas y lo devuelve. Si no existe, lanza `ValidationError`.
3. `procesarPedido()`:
   - Lanza `ValidationError` si `cantidad` no es un entero positivo, antes de consultar el producto.
   - Consulta el producto y espera otros 500 ms para simular el registro.
   - Lanza `StockError` si el stock es menor que la cantidad.
   - Devuelve `{ id, producto, cantidad, total }`, con el nombre escrito como en el inventario y `total = precio × cantidad`.
   - No modifica el stock del inventario.
4. `procesarSecuencial()` procesa los pedidos uno por uno con `for...of` y `await`. Captura cada error dentro del ciclo para que un pedido rechazado no detenga los demás.
5. `procesarParalelo()` inicia todos los pedidos a la vez con `Promise.allSettled()` y muestra los resultados en el orden original.
6. Ambas funciones muestran el tiempo transcurrido en segundos con un decimal. Puedes usar `performance.now()`, que devuelve milisegundos, y `toFixed(1)`.
7. Al mostrar un error, distingue `ValidationError` y `StockError` con `instanceof`. Vuelve a lanzar cualquier otro error.
8. Menú con `node:readline/promises`:
   - `1` procesa los pedidos en secuencia.
   - `2` los procesa en paralelo.
   - `3` pide un nombre y muestra el producto o el mensaje de error.
   - `4` sale y cierra la interfaz.
   - Cualquier otra opción muestra `Opción no válida` y repite el menú.

### Salida esperada aproximada

```text
1. Procesar pedidos uno por uno
2. Procesar pedidos en paralelo
3. Consultar un producto
4. Salir
Opción: 1

PROCESAMIENTO SECUENCIAL
Pedido 1: Laptop x2 = $30000
Pedido 2 rechazado (stock): Solo hay 0 unidades de Audifonos
Pedido 3: Mouse x3 = $1350
Pedido 4 rechazado (datos): No existe el producto "Tablet"
Pedido 5 rechazado (datos): La cantidad debe ser un entero positivo
Pedido 6 rechazado (stock): Solo hay 3 unidades de Monitor
Tiempo: 4.5 s

...
Opción: 2

PROCESAMIENTO EN PARALELO
Pedido 1: Laptop x2 = $30000
Pedido 2 rechazado (stock): Solo hay 0 unidades de Audifonos
Pedido 3: Mouse x3 = $1350
Pedido 4 rechazado (datos): No existe el producto "Tablet"
Pedido 5 rechazado (datos): La cantidad debe ser un entero positivo
Pedido 6 rechazado (stock): Solo hay 3 unidades de Monitor
Tiempo: 1.0 s

...
Opción: 3
Producto: webcam
Webcam - $800 - Stock: 6
```

Explica por qué el procesamiento secuencial tarda 4.5 segundos y no 6: no todos los pedidos llegan a las dos esperas.

### Práctica opcional

Agrega la opción `5. Agregar pedido`, que pida producto y cantidad. Convierte la cantidad con la función `convertirCantidad()` de la actividad 1 y muestra el error si es inválida, sin terminar el programa. Los IDs nuevos deben continuar desde el último existente.

---

Siguiente paso: en la clase 06 organizaremos el código en módulos, leeremos y guardaremos archivos con `node:fs/promises` y prepararemos proyectos con npm.

---

[Índice del curso](README.md)
