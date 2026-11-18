# Clase 01: De Python a JavaScript con Node.js

**Duración:** 1 a 2 horas

> Los bloques de desarrollo se leen en orden dentro de cada ejemplo. Las plantillas con `...` se completan en las actividades; las cheat sheets reúnen operaciones independientes. Los programas completos incluyen sus imports.

### Objetivos

- Entender qué es JavaScript y qué papel cumple Node.js.
- Instalar Node.js y ejecutar programas desde la terminal.
- Traducir a JavaScript lo que ya sabes de Python: variables, tipos, operadores, condiciones y ciclos.
- Reconocer las diferencias que más errores causan al venir de Python: `===`, `null` y `undefined`, conversiones que no lanzan errores y valores *truthy*.
- Construir un simulador de ahorro mes a mes.

---

## 1. JavaScript y Node.js

En el curso de Python escribíamos un archivo `.py` y lo ejecutaba el intérprete de Python:

```text
programa.py
     ↓
python (intérprete)
     ↓
resultado en la terminal
```

JavaScript nació para ejecutarse dentro del navegador. **Node.js** toma el motor de JavaScript de Chrome, llamado **V8**, y lo ejecuta fuera del navegador, con acceso a archivos, red y terminal:

```text
programa.js
     ↓
node (V8 + biblioteca de Node.js)
     ↓
resultado en la terminal
```

| Concepto | En Python | En JavaScript |
|---|---|---|
| Lenguaje | Python | JavaScript |
| Programa que ejecuta el código | Intérprete `python` | Runtime `node` |
| Extensión de archivo | `.py` | `.js` |
| Ejecutar un archivo | `python programa.py` | `node programa.js` |
| Consola interactiva | `python` | `node` |
| Instalar paquetes | `pip` | `npm` (clase 06) |

Un **runtime** es el entorno que ejecuta el código y le da acceso al sistema. En este curso no usaremos el navegador: todo se ejecuta con Node.js en la terminal, como hacíamos con Python.

---

## 2. Instalar Node.js

Descarga la versión **LTS** (soporte a largo plazo) desde [nodejs.org](https://nodejs.org). Este curso se comprobó con Node.js 24. Si prefieres administrar varias versiones, puedes usar `nvm` (macOS/Linux) o `fnm` (macOS, Linux y Windows).

Comprueba la instalación:

```bash
node --version
npm --version
```

Resultado aproximado:

```text
v24.15.0
11.12.1
```

`npm` se instala junto con Node.js. Lo usaremos a partir de la clase 06.

---

## 3. Consola interactiva y primer programa

Así como `python` abría una consola interactiva, `node` abre el **REPL** (*Read-Eval-Print Loop*):

```bash
node
```

```text
> 2 + 3
5
> "hola".toUpperCase()
'HOLA'
> .exit
```

Para salir escribe `.exit` o presiona `Ctrl + C` dos veces. El REPL sirve para probar expresiones pequeñas; los programas se escriben en archivos.

Trabaja dentro de la carpeta del curso, por ejemplo en `practicas/`: el `package.json` de la raíz configura Node.js como espera el curso (lo explicaremos en la clase 06). Crea `hola.js`:

```javascript
console.log("Hola desde Node.js");
console.log("Total:", 10 + 5);
```

Ejecútalo desde la carpeta donde está el archivo:

```bash
node hola.js
```

```text
Hola desde Node.js
Total: 15
```

`console.log()` equivale a `print()`: muestra sus argumentos separados por un espacio y termina con un salto de línea. `console.log` es la función `log` del objeto `console`; el punto accede a ella, como `texto.upper()` en Python.

---

## 4. Sintaxis básica: llaves, punto y coma y comentarios

En Python la indentación define los bloques. En JavaScript los bloques se delimitan con **llaves** `{ }`; la indentación solo facilita la lectura:

```python
if precio > 1000:
    print("Producto caro")
    print("Revisa el presupuesto")
```

```javascript
const precio = 1500;

if (precio > 1000) {
  console.log("Producto caro");
  console.log("Revisa el presupuesto");
}
```

Diferencias visibles:

- La condición va entre paréntesis: `if (precio > 1000)`.
- No hay dos puntos al final de la línea.
- Cada instrucción termina con `;`. JavaScript puede insertar algunos automáticamente, pero en este curso los escribiremos siempre.
- Usaremos 2 espacios de indentación, la convención más común en JavaScript.

Comentarios:

```javascript
// Comentario de una línea, como # en Python.

/*
  Comentario de varias líneas.
  Útil para explicaciones largas.
*/
```

---

## 5. Variables: `const` y `let`

En Python bastaba con asignar: `precio = 450`. En JavaScript declaramos la variable la primera vez:

```javascript
const nombre = "Mouse";
let stock = 12;

stock = stock - 1;
console.log(nombre, stock);
```

| Palabra | Uso |
|---|---|
| `const` | La variable no se puede reasignar. Úsala por defecto. |
| `let` | La variable se puede reasignar, por ejemplo un contador o un acumulador. |
| `var` | Forma antigua. La verás en código viejo; no la usaremos. |

Reasignar una constante provoca un error:

```javascript
const iva = 0.16;
iva = 0.08;
```

```text
TypeError: Assignment to constant variable.
```

Declara una variable una sola vez. Las siguientes asignaciones a una variable `let` no repiten la palabra `let`.

### Nombres

En Python usábamos `snake_case`; en JavaScript la convención es `camelCase`:

```python
precio_total = 300
```

```javascript
const precioTotal = 300;
```

Mantendremos los nombres en español, como en el curso de Python: `productos`, `precioTotal`, `calcularDescuento`.

---

## 6. Números

Python tiene `int` y `float`. JavaScript tiene un único tipo para ambos casos: `number`.

```javascript
console.log(10 + 5);      // 15
console.log(7 / 2);       // 3.5
console.log(2 ** 10);     // 1024
console.log(10 % 3);      // 1
console.log(0.1 + 0.2);   // 0.30000000000000004
```

| Operación | En Python | En JavaScript |
|---|---|---|
| División | `7 / 2` → `3.5` | `7 / 2` → `3.5` |
| División entera | `7 // 2` → `3` | `Math.floor(7 / 2)` → `3` |
| Potencia | `2 ** 10` | `2 ** 10` |
| Residuo | `10 % 3` | `10 % 3` |
| ¿Es entero? | `isinstance(x, int)` | `Number.isInteger(x)` |
| Redondear a 2 decimales para mostrar | `f"{x:.2f}"` | `x.toFixed(2)` |
| División entre cero | `ZeroDivisionError` | `Infinity`, sin error |

`Math` reúne funciones matemáticas: `Math.floor()` redondea hacia abajo, `Math.trunc()` elimina los decimales, `Math.round()` redondea al entero más cercano, `Math.max()` y `Math.min()` devuelven el mayor y el menor de sus argumentos.

```javascript
const precio = 15000 * 1.16;

console.log(precio);             // 17400
console.log(Math.floor(9.99));   // 9
console.log(Math.trunc(-3.5));   // -3
console.log(Math.max(3, 9, 1));  // 9
console.log(Number.isInteger(5.0)); // true

const promedio = 22.857142857;
console.log(promedio.toFixed(2)); // "22.86"
```

`toFixed(2)` devuelve **texto**, no un número. Úsalo para mostrar resultados, no para seguir calculando.

---

## 7. Texto y template literals

Las cadenas se escriben con comillas dobles, simples o *backticks* (`` ` ``). Usaremos comillas dobles, salvo en los template literals.

Un **template literal** usa backticks e inserta expresiones con `${...}`. Es el equivalente de las f-strings:

```python
nombre = "Laptop"
precio = 15000
print(f"{nombre} cuesta ${precio}")
```

```javascript
const nombre = "Laptop";
const precio = 15000;
console.log(`${nombre} cuesta $${precio}`);
```

```text
Laptop cuesta $15000
```

En `$${precio}`, el primer `$` es texto normal y `${precio}` inserta la variable. Los template literals también pueden ocupar varias líneas.

### Métodos de texto

```javascript
const producto = "  Laptop Pro  ";

console.log(producto.length);               // 14
console.log(producto.trim());               // "Laptop Pro"
console.log(producto.trim().toUpperCase()); // "LAPTOP PRO"
console.log("Laptop".slice(0, 3));          // "Lap"
console.log("Laptop".slice(-3));            // "top"
console.log("Laptop".includes("top"));      // true
console.log("café,pan,leche".split(","));   // [ 'café', 'pan', 'leche' ]
console.log("a-b-c".replaceAll("-", "/"));  // "a/b/c"
console.log("=".repeat(10));                // "=========="
console.log("7".padStart(3, "0"));          // "007"
```

| En Python | En JavaScript |
|---|---|
| `len(texto)` | `texto.length` (propiedad, sin paréntesis) |
| `texto.upper()` / `texto.lower()` | `texto.toUpperCase()` / `texto.toLowerCase()` |
| `texto.strip()` | `texto.trim()` |
| `"top" in texto` | `texto.includes("top")` |
| `texto.startswith("La")` | `texto.startsWith("La")` |
| `texto[0:3]` | `texto.slice(0, 3)` |
| `texto[-1]` | `texto.at(-1)` |
| `texto.split(",")` | `texto.split(",")` |
| `texto.replace("-", "/")` | `texto.replaceAll("-", "/")` |
| `"=" * 10` | `"=".repeat(10)` |

`replace()` también existe en JavaScript, pero con un texto como patrón solo reemplaza la **primera** aparición. Para reemplazar todas, usa `replaceAll()`.

---

## 8. Booleanos, `null` y `undefined`

Los booleanos se escriben en minúsculas: `true` y `false`.

Python tiene un solo valor para la ausencia de resultado: `None`. JavaScript tiene dos:

| Valor | Significado habitual |
|---|---|
| `undefined` | Algo que no recibió valor: una variable sin asignar, un argumento omitido, una propiedad inexistente. |
| `null` | Ausencia de valor elegida explícitamente por el programador. |

```javascript
let descuento;
console.log(descuento); // undefined

let cupon = null;
console.log(cupon);     // null
```

Cuando quieras expresar "todavía no hay resultado", como hacíamos con `None`, asigna `null`. En la clase 03 veremos que leer una clave inexistente también produce `undefined`.

### `typeof`

`typeof` devuelve el tipo de un valor como texto, parecido a `type()`:

```javascript
console.log(typeof 450);        // "number"
console.log(typeof "Mouse");    // "string"
console.log(typeof true);       // "boolean"
console.log(typeof undefined);  // "undefined"
console.log(typeof null);       // "object"
```

`typeof null` devuelve `"object"` por un error histórico del lenguaje que se conservó por compatibilidad. Para preguntar por `null`, compara directamente: `valor === null`.

---

## 9. Conversiones de tipo

En Python, `int("abc")` lanzaba `ValueError`. En JavaScript, `Number("abc")` **no lanza un error**: devuelve `NaN` (*Not a Number*), un valor especial de tipo `number`.

```javascript
console.log(Number("12"));       // 12
console.log(Number("12.5"));     // 12.5
console.log(Number("abc"));      // NaN
console.log(Number(""));         // 0
console.log(parseInt("12px"));   // 12
console.log(parseFloat("3.5kg")); // 3.5
console.log(String(42));         // "42"
```

| En Python | En JavaScript |
|---|---|
| `int("12")` | `Number("12")` o `parseInt("12")` |
| `float("12.5")` | `Number("12.5")` o `parseFloat("12.5")` |
| `str(42)` | `String(42)` |
| `int("abc")` lanza `ValueError` | `Number("abc")` devuelve `NaN` |

`Number()` exige que todo el texto sea un número; `parseInt()` y `parseFloat()` leen desde el inicio y se detienen en el primer carácter inválido. Observa también que `Number("")` devuelve `0`: un texto vacío no se considera inválido.

Para comprobar si una conversión falló, usa `Number.isNaN()`. `NaN` no es igual ni siquiera a sí mismo, así que `valor === NaN` siempre es `false`:

```javascript
const cantidad = Number("doce");

if (Number.isNaN(cantidad)) {
  console.log("La cantidad no es un número válido");
}
```

### Conversiones automáticas

Python no permitía sumar texto y números (`"5" + 3` lanzaba `TypeError`). JavaScript convierte automáticamente, y el resultado depende del operador:

```javascript
console.log("5" + 3);   // "53"  (+ con texto concatena)
console.log("5" - 3);   // 2     (- convierte a número)
console.log("5" * "2"); // 10
```

Convierte tú mismo los valores antes de operar. Así el programa hace lo que lees y no lo que el lenguaje adivina.

---

## Actividad 1: Tipos y conversiones

Crea `tipos.js` y completa las líneas marcadas:

```javascript
const nombre = "  teclado mecánico  ";
const precioTexto = "900";
const stockTexto = "ocho";

// 1. Muestra el nombre sin espacios exteriores y en mayúsculas.
// 2. Convierte precioTexto a número y muestra el precio con IVA (16 %) con 2 decimales.
// 3. Convierte stockTexto a número y muestra un mensaje si la conversión falló.
// 4. Muestra con typeof el tipo del precio convertido y el de precioConIva.toFixed(2).
// 5. Muestra: "TECLADO MECÁNICO cuesta $1044.00 con IVA" usando un template literal.
```

---

## 10. Comparaciones y operadores lógicos

### `===` en lugar de `==`

JavaScript tiene dos operadores de igualdad. `==` convierte los tipos antes de comparar y produce resultados sorprendentes; `===` compara valor **y** tipo, como esperarías viniendo de Python:

```javascript
console.log("5" == 5);   // true
console.log("5" === 5);  // false
console.log(0 == "");    // true
console.log(0 === "");   // false
```

Usa siempre `===` y `!==`. En este curso no usaremos `==`.

| En Python | En JavaScript |
|---|---|
| `==` / `!=` | `===` / `!==` |
| `and` | `&&` |
| `or` | `\|\|` |
| `not` | `!` |
| `x is None` | `x === null` (o `x === undefined`) |
| `a < b < c` | `a < b && b < c` |

En Python, `1 < x < 10` encadena comparaciones. En JavaScript esa expresión compara un booleano con `10`; escríbela como `1 < x && x < 10`.

### Valor por defecto con `??`

`??` devuelve el valor de la derecha cuando el de la izquierda es `null` o `undefined`:

```javascript
const cupon = null;
const codigo = cupon ?? "SIN-CUPON";
console.log(codigo); // "SIN-CUPON"
```

`||` hace algo parecido, pero también reemplaza `0` y `""`:

```javascript
const stock = 0;
console.log(stock || 10); // 10  (¡perdimos el 0!)
console.log(stock ?? 10); // 0
```

Usa `??` cuando `0` o el texto vacío sean valores válidos.

---

## 11. Valores *truthy* y *falsy*

Como en Python, una condición acepta cualquier valor y lo interpreta como verdadero o falso. La lista de valores **falsy** en JavaScript es:

```text
false   0   ""   null   undefined   NaN
```

Todo lo demás es *truthy*. Aquí aparecen diferencias importantes con Python:

| Valor | En Python | En JavaScript |
|---|---|---|
| Lista o array vacío `[]` | Falso | **Verdadero** |
| Diccionario u objeto vacío `{}` | Falso | **Verdadero** |
| Texto `"0"` | Verdadero | Verdadero |
| Texto `""` | Falso | Falso |
| `None` / `null` | Falso | Falso |

```javascript
const carrito = [];

if (carrito) {
  console.log("Esto se imprime aunque el carrito esté vacío");
}

if (carrito.length === 0) {
  console.log("El carrito está vacío");
}
```

Para saber si un array está vacío, compara su `length`. Veremos los arrays en la clase 02.

---

## 12. Condiciones

```javascript
const temperatura = 27;

if (temperatura >= 30) {
  console.log("Calor extremo");
} else if (temperatura >= 25) {
  console.log("Calor");
} else {
  console.log("Templado");
}
```

`elif` se escribe `else if`.

### Operador ternario

El ternario equivale a la expresión condicional de Python:

```python
estado = "disponible" if stock > 0 else "agotado"
```

```javascript
const stock = 0;
const estado = stock > 0 ? "disponible" : "agotado";
console.log(estado); // "agotado"
```

Se lee: *condición* `?` *valor si es verdadera* `:` *valor si es falsa*. Úsalo para elegir un valor; para ejecutar varias instrucciones, usa `if`.

---

## 13. Ciclos

### `for` clásico

JavaScript no tiene `range()`. El `for` clásico tiene tres partes separadas por `;`: inicialización, condición y actualización:

```python
for i in range(1, 6):
    print(i)
```

```javascript
for (let i = 1; i <= 5; i++) {
  console.log(i);
}
```

`i++` suma 1 a `i`; equivale a `i += 1`. El ciclo empieza con `i = 1`, se repite mientras `i <= 5` y después de cada vuelta ejecuta `i++`.

### `for...of`

`for...of` recorre los elementos de algo iterable, como el `for` de Python. Con un texto, recorre sus caracteres:

```javascript
for (const letra of "Node") {
  console.log(letra);
}
```

En la clase 02 lo usaremos con arrays. Existe también `for...in`, pero recorre **claves**, no valores; no lo uses para recorrer elementos.

### `while`, `break` y `continue`

```javascript
let saldo = 100;
let meses = 0;

while (saldo < 200) {
  saldo = saldo * 1.1;
  meses++;
}

console.log(`Meses necesarios: ${meses}`); // Meses necesarios: 8
```

`break` termina el ciclo y `continue` pasa a la siguiente vuelta, igual que en Python:

```javascript
for (let i = 1; i <= 10; i++) {
  if (i % 2 === 0) {
    continue;
  }
  if (i > 7) {
    break;
  }
  console.log(i); // 1, 3, 5, 7
}
```

### Entrada del usuario

En Python usábamos `input()`. En Node.js leer de la terminal requiere asincronía, que veremos en la clase 05. Mientras tanto, nuestros programas trabajarán con datos escritos en el código.

---

## Actividad 2: Tabla de descuentos

Crea `descuentos.js`. Para cantidades de 1 a 10 unidades de un producto de `$450`:

1. Calcula el subtotal.
2. Aplica 10 % de descuento a partir de 5 unidades y 20 % a partir de 8.
3. Omite con `continue` la cantidad 3, que no se vende por promoción.
4. Muestra cada línea con `toFixed(2)`.

Salida esperada:

```text
1 unidad(es): $450.00
2 unidad(es): $900.00
4 unidad(es): $1800.00
5 unidad(es): $2025.00
6 unidad(es): $2430.00
7 unidad(es): $2835.00
8 unidad(es): $2880.00
9 unidad(es): $3240.00
10 unidad(es): $3600.00
```

---

## Errores comunes viniendo de Python

1. **Olvidar declarar la variable.** Escribe `const` o `let` la primera vez. Reasignar con `let` no repite la palabra.
2. **Usar `==`.** `"5" == 5` es `true`. Usa `===`.
3. **Esperar un error al convertir.** `Number("abc")` devuelve `NaN` y `Number("")` devuelve `0`; comprueba el resultado con `Number.isNaN()`.
4. **Sumar texto y números.** `"5" + 3` es `"53"`. Convierte antes de operar.
5. **Comprobar un array vacío con `if (lista)`.** `[]` es *truthy*; usa `lista.length === 0`.
6. **Usar `||` para valores por defecto con números.** `0 || 10` es `10`; usa `??`.
7. **Encadenar comparaciones.** `1 < x < 10` no funciona como en Python; usa `1 < x && x < 10`.
8. **Escribir `True`, `False` o `None`.** En JavaScript son `true`, `false` y `null`.

---

## Cheat sheet

| Tarea | En Python | En JavaScript |
|---|---|---|
| Ejecutar | `python programa.py` | `node programa.js` |
| Mostrar | `print("a", 1)` | `console.log("a", 1);` |
| Comentario | `# texto` | `// texto` |
| Variable | `x = 1` | `let x = 1;` / `const x = 1;` |
| Texto con variables | `f"Total: {total}"` | `` `Total: ${total}` `` |
| Longitud de texto | `len(texto)` | `texto.length` |
| Ausencia de valor | `None` | `null` / `undefined` |
| Tipo | `type(x)` | `typeof x` |
| Texto a número | `int(x)` / `float(x)` | `Number(x)` |
| ¿Conversión inválida? | `except ValueError` | `Number.isNaN(n)` |
| Igualdad | `==` | `===` |
| Lógicos | `and`, `or`, `not` | `&&`, `\|\|`, `!` |
| Valor por defecto | `x if x is not None else 0` | `x ?? 0` |
| Condición | `if / elif / else` | `if / else if / else` |
| Condicional en una línea | `a if c else b` | `c ? a : b` |
| Contar de 0 a 4 | `for i in range(5):` | `for (let i = 0; i < 5; i++) {` |
| Recorrer elementos | `for x in datos:` | `for (const x of datos) {` |
| Dos decimales | `f"{x:.2f}"` | `x.toFixed(2)` |
| División entera | `a // b` | `Math.floor(a / b)` |

---

## Ejercicio final: Simulador de ahorro

Crea `ahorro.js` con estos datos:

```javascript
const saldoInicial = 1000;
const ahorroMensual = 500;
const tasaMensual = 0.01;
const meta = 6000;
const meses = 12;
```

Cada mes, primero se calcula el interés sobre el saldo actual (`saldo * tasaMensual`) y después se suman ese interés y el ahorro mensual.

### Requisitos

1. Usa `let` solo para los valores que cambian: el saldo, el total de intereses y el mes en que se alcanzó la meta.
2. Inicializa el mes de la meta con `null`, porque al principio todavía no existe.
3. Recorre los meses con un `for` clásico.
4. Muestra cada mes con dos dígitos (`padStart`) y el saldo con `toFixed(2)`.
5. Marca con `<- meta alcanzada` solo el **primer** mes en que el saldo llega a la meta.
6. Al final muestra el total aportado, los intereses ganados y el saldo final.
7. Si se alcanzó la meta, indica en qué mes; si no, cuánto falta.
8. Usa un ternario para clasificar el ahorro: `excelente` si el saldo final es al menos el 120 % de la meta, `bueno` si alcanza la meta e `insuficiente` en otro caso.

### Salida esperada

```text
SIMULADOR DE AHORRO
==============================
Mes 01: $1510.00
Mes 02: $2025.10
Mes 03: $2545.35
Mes 04: $3070.80
Mes 05: $3601.51
Mes 06: $4137.53
Mes 07: $4678.90
Mes 08: $5225.69
Mes 09: $5777.95
Mes 10: $6335.73 <- meta alcanzada
Mes 11: $6899.09
Mes 12: $7468.08
==============================
Total aportado: $7000.00
Intereses ganados: $468.08
Saldo final: $7468.08
Meta de $6000 alcanzada en el mes 10.
Nivel de ahorro: excelente
```

### Práctica opcional

1. Cambia `meta` a `9000` y comprueba que el programa muestre cuánto falta y el nivel `insuficiente`.
2. Agrega una constante `retiroMes = 6` que reste `$800` en ese mes. Comprueba que el retiro se aplique una sola vez.
3. Usa un `while` para calcular cuántos meses se necesitan para llegar a `$10000` sin límite de 12 meses.

---

[Índice del curso](README.md)
