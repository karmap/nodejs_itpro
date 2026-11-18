# Clase 01: Node.js en tu computadora y de Python a JavaScript

**Duración:** 2 horas

> Los bloques de desarrollo se leen en orden dentro de cada ejemplo. Las plantillas con `...` se completan en las actividades; las cheat sheets reúnen operaciones independientes. Los programas completos incluyen sus imports.

La clase tiene dos partes. La **parte 1** (unos 50 minutos) deja lista tu computadora: Node.js instalado con un gestor de versiones, editor, terminal y la carpeta del curso. Nadie debería pasar a la parte 2 sin completar la Actividad 1. La **parte 2** traduce a JavaScript la sintaxis básica de Python. Si el tiempo no alcanza, el ejercicio final se puede dejar como tarea.

### Objetivos

- Instalar Node.js con un gestor de versiones y elegir la versión correcta.
- Preparar un ambiente local: terminal, editor, carpeta del curso y versión fijada con `.nvmrc`.
- Ejecutar programas desde la terminal, reiniciarlos al guardar con `--watch` y depurarlos con VS Code.
- Resolver los problemas de instalación más frecuentes.
- Traducir a JavaScript lo que ya sabes de Python: variables, tipos, operadores, condiciones y ciclos.
- Reconocer las diferencias que más errores causan al venir de Python: `===`, `null` y `undefined`, conversiones que no lanzan errores y valores *truthy*.
- Construir un simulador de ahorro mes a mes.

---

# Parte 1: Node.js en tu computadora

## 1. JavaScript y Node.js

En el curso de Python escribíamos un archivo `.py` y lo ejecutaba el intérprete de Python. Con Node.js el flujo es el mismo:

```text
programa.py                         programa.js
     ↓                                   ↓
python (intérprete)                 node (V8 + biblioteca de Node.js)
     ↓                                   ↓
resultado en la terminal            resultado en la terminal
```

Como vimos en la clase 00, **Node.js** toma **V8**, el motor de JavaScript de Chrome, y lo ejecuta fuera del navegador, con acceso a archivos, red y terminal. Un **runtime** es ese entorno que ejecuta el código y le da acceso al sistema.

| Concepto | En Python | En JavaScript |
|---|---|---|
| Lenguaje | Python | JavaScript |
| Programa que ejecuta el código | Intérprete `python` | Runtime `node` |
| Extensión de archivo | `.py` | `.js` |
| Ejecutar un archivo | `python programa.py` | `node programa.js` |
| Consola interactiva | `python` | `node` |
| Instalar paquetes | `pip` | `npm` (clase 06) |

En este curso no usaremos el navegador: todo se ejecuta con Node.js en la terminal, como hacíamos con Python.

---

## 2. Qué versión instalar

Node.js publica versiones numeradas (22, 24, 26...). Lo que importa es si la versión es **LTS**:

| Tipo | Significado | ¿Para el curso? |
|---|---|---|
| **LTS** (*Long Term Support*) | Versión estable con correcciones durante unos 30 meses. | Sí |
| **Current** | La más nueva, para probar novedades. Su soporte es corto. | No |

Este curso se comprobó con **Node.js 24 LTS** (24.15.0, con npm 11.12.1). Usaremos esa versión para que todo lo que veas en clase coincida con tu computadora. Puedes consultar qué versiones tienen soporte en el [calendario de versiones de Node.js](https://nodejs.org/en/about/previous-releases).

### ¿Por qué un gestor de versiones?

En Python, si un proyecto necesitaba Python 3.11 y otro 3.13, se podía usar `pyenv` para tener ambas. Node.js tiene herramientas equivalentes, y en JavaScript se usan todavía más porque cada proyecto suele fijar su versión de Node.js.

Un **gestor de versiones** instala Node.js dentro de tu carpeta de usuario y te permite:

- Tener varias versiones instaladas y cambiar entre ellas con un comando.
- Cambiar de versión automáticamente al entrar a la carpeta de un proyecto.
- Instalar paquetes globales sin `sudo` ni permisos de administrador.

Usaremos **fnm** (*Fast Node Manager*) porque funciona igual en macOS, Windows y Linux. Si ya usas **nvm**, también sirve: los comandos equivalentes aparecen en la sección 3.

---

## 3. Instalar Node.js

Antes de empezar, comprueba si ya tienes Node.js:

```bash
node --version
```

Si muestra `v24.x.x` o superior y no quieres usar un gestor de versiones, puedes saltar a la sección 4. Si muestra una versión anterior, o si no sabes cómo se instaló, sigue con el gestor de versiones: convivirá con la instalación anterior y tendrá prioridad.

### macOS

Con [Homebrew](https://brew.sh):

```bash
brew install fnm
```

Agrega esta línea al final de `~/.zshrc` para que la terminal cargue fnm al abrirse. La opción `--use-on-cd` cambia de versión automáticamente al entrar a una carpeta con un archivo `.nvmrc`:

```bash
eval "$(fnm env --use-on-cd --shell zsh)"
```

Puedes abrir el archivo con `open -e ~/.zshrc`. Guarda, **cierra la terminal y abre una nueva**.

### Windows

En PowerShell:

```powershell
winget install Schniz.fnm
```

Abre tu perfil de PowerShell (si no existe, se crea):

```powershell
if (-not (Test-Path $PROFILE)) { New-Item $PROFILE -Force }
notepad $PROFILE
```

Agrega esta línea, guarda, **cierra PowerShell y abre una ventana nueva**:

```powershell
fnm env --use-on-cd --shell powershell | Out-String | Invoke-Expression
```

Si al abrir PowerShell aparece un error sobre la ejecución de scripts deshabilitada, revisa la sección 8.

### Linux (y WSL en Windows)

```bash
curl -fsSL https://fnm.vercel.app/install | bash
```

El instalador agrega la configuración a `~/.bashrc` o `~/.zshrc`. Cierra la terminal y abre una nueva. Si quieres el cambio automático de versión, verifica que la línea agregada incluya `--use-on-cd`, como en la de macOS.

### Instalar Node.js 24

Con fnm ya configurado, el procedimiento es igual en los tres sistemas:

```bash
fnm --version
```

```bash
fnm install 24
```

```bash
fnm default 24
```

`fnm install 24` descarga la versión 24 más reciente; `fnm default 24` la deja como la versión que se usa al abrir una terminal. Otros comandos útiles:

| Tarea | fnm | nvm (si ya lo usas) |
|---|---|---|
| Instalar una versión | `fnm install 24` | `nvm install 24` |
| Usarla en esta terminal | `fnm use 24` | `nvm use 24` |
| Versión por defecto | `fnm default 24` | `nvm alias default 24` |
| Ver versiones instaladas | `fnm list` | `nvm ls` |
| Ver la versión activa | `fnm current` | `nvm current` |
| Instalar la versión de `.nvmrc` | `fnm install` | `nvm install` |
| Usar la versión de `.nvmrc` | `fnm use` | `nvm use` |

### Alternativa: instalador oficial

Si el gestor de versiones te da problemas y necesitas avanzar, descarga el instalador **LTS** desde [nodejs.org](https://nodejs.org/es/download) y sigue el asistente. Funciona bien para el curso; la desventaja es que solo tendrás una versión y para actualizarla tendrás que repetir la instalación.

No mezcles métodos sin saber cuál tiene prioridad: tener Node.js de Homebrew, del instalador y de fnm a la vez es la causa más común de "instalé la 24 pero me sale la 18". La sección 8 explica cómo detectarlo.

---

## 4. Comprobar la instalación

En una terminal **nueva**:

```bash
node --version
```

```bash
npm --version
```

Resultado aproximado:

```text
v24.15.0
11.12.1
```

`npm` se instala junto con Node.js; lo usaremos a partir de la clase 06.

Comprueba también **qué** `node` se está ejecutando. En macOS y Linux:

```bash
which node
```

En PowerShell:

```powershell
Get-Command node
```

Si usas fnm, la ruta debe contener `fnm`. Si aparece otra ruta, por ejemplo `/opt/homebrew/bin/node` o `C:\Program Files\nodejs\node.exe`, tienes una instalación anterior con prioridad: revisa la sección 8.

---

## 5. Ambiente local

Un ambiente local es todo lo que necesitas en tu computadora para escribir, ejecutar y depurar programas sin depender de nada externo. Para este curso son cuatro piezas: Node.js, una terminal, un editor y la carpeta del curso.

### Comparación con Python

Muchas piezas tienen un equivalente directo en lo que ya usabas:

| Necesidad | En Python | En Node.js |
|---|---|---|
| Administrar versiones del lenguaje | `pyenv` | `fnm` o `nvm` |
| Fijar la versión de un proyecto | `.python-version` | `.nvmrc` |
| Aislar las dependencias de un proyecto | `venv` (hay que activarlo) | Carpeta `node_modules` (automática, sin activar) |
| Instalar paquetes | `pip install` | `npm install` |
| Lista de dependencias | `requirements.txt` | `package.json` |
| Paquetes del sistema | `pip install` fuera de un venv | `npm install -g` |

La diferencia más importante: **en Node.js no hay que activar nada**. Cada proyecto guarda sus paquetes en su propia carpeta `node_modules`, y Node.js los busca ahí automáticamente. Lo veremos en la clase 06.

### Terminal

Usa la terminal que ya usabas en el curso de Python:

| Sistema | Terminal recomendada |
|---|---|
| macOS | Terminal o iTerm2 (shell `zsh`) |
| Windows | Windows Terminal con PowerShell |
| Linux | La terminal de tu distribución (shell `bash` o `zsh`) |

Los comandos de Node.js son los mismos en todos los sistemas. Los de la terminal varían un poco:

| Tarea | macOS / Linux | PowerShell |
|---|---|---|
| Carpeta actual | `pwd` | `pwd` |
| Listar archivos | `ls` | `ls` o `dir` |
| Entrar a una carpeta | `cd practicas` | `cd practicas` |
| Subir un nivel | `cd ..` | `cd ..` |
| Crear una carpeta | `mkdir practicas` | `mkdir practicas` |
| Limpiar la pantalla | `clear` | `cls` o `clear` |

### Editor

Recomendamos **Visual Studio Code**. Trae soporte para JavaScript y un depurador para Node.js sin instalar extensiones. Dos ajustes útiles:

- **Abrir la carpeta, no el archivo:** usa *Archivo → Abrir carpeta* y elige la carpeta del curso. Así el editor reconoce el `package.json` y la terminal integrada (`` Ctrl + ` ``) se abre en el lugar correcto.
- **Comando `code` en la terminal:** en macOS, abre la paleta de comandos (`Cmd + Shift + P`) y ejecuta *Shell Command: Install 'code' command in PATH*. En Windows y Linux el instalador lo agrega. Después, `code .` abre la carpeta actual en VS Code.

La extensión **Prettier** es opcional: formatea el código al guardar. No necesitas más extensiones para el curso.

### Carpeta del curso

Descarga o clona la carpeta del curso y trabaja siempre dentro de ella. Crea una subcarpeta `practicas/` para tus archivos:

```text
node_itpro/
├── .nvmrc            ← versión de Node.js del curso
├── package.json      ← configuración de Node.js y dependencias
├── README.md
├── clase_00_historia_de_javascript.md
├── clase_01_de_python_a_javascript.md
├── ...
└── practicas/        ← tus archivos
    └── clase01/
        ├── hola.js
        └── ambiente.js
```

Dos archivos de la raíz preparan el ambiente:

- **`.nvmrc`** contiene solo `24`. Con fnm configurado con `--use-on-cd`, al entrar a la carpeta la terminal cambia a Node.js 24 automáticamente. Si no, ejecuta `fnm use` (o `nvm use`) dentro de la carpeta.
- **`package.json`** contiene `"type": "module"`, que hace que Node.js trate cada `.js` como un módulo ES, la forma moderna de JavaScript. Lo explicaremos en la clase 06; por ahora basta con trabajar dentro de la carpeta.

Entra a la carpeta del curso y prepara la tuya:

```bash
cd node_itpro
```

```bash
fnm use
```

```bash
mkdir -p practicas/clase01
```

En PowerShell, `mkdir practicas/clase01` crea ambas carpetas sin la opción `-p`.

---

## 6. Consola interactiva y primer programa

Así como `python` abría una consola interactiva, `node` abre el **REPL** (*Read-Eval-Print Loop*):

```bash
node
```

```text
> 2 + 3
5
> "hola".toUpperCase()
'HOLA'
> typeof document
'undefined'
> .exit
```

Para salir escribe `.exit` o presiona `Ctrl + C` dos veces. `typeof document` responde la pregunta de la clase 00: `document` pertenece al navegador y no existe en Node.js. El REPL sirve para probar expresiones pequeñas; los programas se escriben en archivos.

Crea `practicas/clase01/hola.js`:

```javascript
console.log("Hola desde Node.js");
console.log("Total:", 10 + 5);
```

Ejecútalo desde la carpeta donde está el archivo:

```bash
cd practicas/clase01
```

```bash
node hola.js
```

```text
Hola desde Node.js
Total: 15
```

`console.log()` equivale a `print()`: muestra sus argumentos separados por un espacio y termina con un salto de línea. `console.log` es la función `log` del objeto `console`; el punto accede a ella, como `texto.upper()` en Python.

### Reiniciar al guardar con `--watch`

Mientras practicas, es cómodo que el programa se vuelva a ejecutar cada vez que guardas:

```bash
node --watch hola.js
```

Cambia el texto de `hola.js`, guarda y observa la terminal. Para detenerlo presiona `Ctrl + C`. Es útil con programas cortos como los de esta clase y, más adelante, con servidores.

---

## 7. Depurar con VS Code

En el curso de Python, cuando un programa no hacía lo esperado, agregábamos `print()` o usábamos el depurador. En Node.js `console.log()` cumple el papel de `print()`, y VS Code incluye un depurador:

1. Abre la paleta de comandos (`Cmd + Shift + P` o `Ctrl + Shift + P`) y ejecuta **JavaScript Debug Terminal**. Se abre una terminal especial.
2. En `hola.js`, haz clic a la izquierda del número de la línea 2. Aparece un punto rojo: un **breakpoint**.
3. En la terminal de depuración ejecuta `node hola.js`.
4. El programa se detiene en la línea 2. Desde el panel lateral puedes ver las variables y avanzar línea por línea con `F10`, o continuar con `F5`.

Cualquier programa que ejecutes con `node` en esa terminal se conecta al depurador automáticamente; no hay que configurar nada.

---

## Actividad 1: Verifica tu ambiente

Crea `practicas/clase01/ambiente.js` y copia el siguiente programa. Usa sintaxis que veremos en la parte 2; al final de la clase podrás explicar cada línea.

```javascript
console.log("Node.js:", process.version);
console.log("Ejecutable:", process.execPath);
console.log("Motor V8:", process.versions.v8);
console.log("Sistema:", process.platform, process.arch);
console.log("Carpeta actual:", process.cwd());

const versionMayor = Number(process.versions.node.split(".")[0]);

if (versionMayor >= 24) {
  console.log("Listo: tu versión de Node.js sirve para el curso.");
} else {
  console.log("Atención: instala Node.js 24 o superior.");
}
```

`process` es un objeto propio de Node.js con información del programa en ejecución, como `sys` y `os` en Python. Así como `document` solo existe en el navegador, `process` solo existe en Node.js.

Resultado aproximado en macOS (las rutas cambian en cada computadora):

```text
Node.js: v24.15.0
Ejecutable: /Users/ana/…/fnm_multishells/…/bin/node
Motor V8: 13.6.233.17-node.48
Sistema: darwin arm64
Carpeta actual: /Users/ana/node_itpro/practicas/clase01
Listo: tu versión de Node.js sirve para el curso.
```

Completa esta lista antes de continuar:

- [ ] `node --version` muestra `v24` o superior en una terminal nueva.
- [ ] `which node` o `Get-Command node` apunta a la instalación que esperas.
- [ ] La carpeta del curso está abierta en tu editor y la terminal integrada está dentro de ella.
- [ ] `node --watch ambiente.js` vuelve a ejecutar el programa al guardar.
- [ ] El depurador se detiene en un breakpoint de `ambiente.js`.

---

## 8. Problemas frecuentes de instalación

| Síntoma | Causa probable | Solución |
|---|---|---|
| `command not found: node` o `node no se reconoce como un comando` | La terminal se abrió antes de instalar, o falta la línea de fnm en `~/.zshrc` / `$PROFILE`. | Cierra **todas** las terminales y abre una nueva. Si persiste, revisa que la línea de configuración esté guardada. |
| `command not found: fnm` | fnm no se instaló o la terminal no se reinició. | Repite la instalación de tu sistema y abre una terminal nueva. |
| `node --version` muestra una versión vieja | Hay otra instalación con prioridad. | Revisa `which node` / `Get-Command node`. Desinstala la otra (`brew uninstall node` o desde *Agregar o quitar programas* en Windows) o ejecuta `fnm default 24`. |
| PowerShell: *la ejecución de scripts está deshabilitada en este sistema* | La política de ejecución de Windows bloquea el perfil. | Ejecuta `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` y abre PowerShell de nuevo. Afecta solo a tu usuario. |
| `fnm use` responde que la versión no está instalada | La carpeta tiene un `.nvmrc` con una versión que no tienes. | Ejecuta `fnm install` en esa carpeta. |
| `EACCES: permission denied` al usar `npm install -g` | Node.js se instaló en una carpeta del sistema. | No uses `sudo`. Instala Node.js con fnm: los paquetes globales quedan en tu carpeta de usuario. |
| VS Code muestra otra versión en su terminal integrada | VS Code se abrió antes de configurar fnm. | Cierra VS Code por completo (no solo la ventana) y ábrelo de nuevo. |

Si nada de esto resuelve el problema, comparte con el instructor la salida de `node --version`, `which node` (o `Get-Command node`) y el mensaje de error completo.

---

# Parte 2: De Python a JavaScript

## 9. Sintaxis básica: llaves, punto y coma y comentarios

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

## 10. Variables: `const` y `let`

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

## 11. Números

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

## 12. Texto y template literals

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

## 13. Booleanos, `null` y `undefined`

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

## 14. Conversiones de tipo

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

## Actividad 2: Tipos y conversiones

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

## 15. Comparaciones y operadores lógicos

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

## 16. Valores *truthy* y *falsy*

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

## 17. Condiciones

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

## 18. Ciclos

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

## Actividad 3: Tabla de descuentos

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

## Errores comunes

### En el ambiente

1. **No abrir una terminal nueva después de instalar.** La terminal lee su configuración al abrirse; las que ya estaban abiertas no ven a fnm ni a Node.js.
2. **Tener varias instalaciones de Node.js sin saberlo.** Si la versión no es la esperada, revisa `which node` / `Get-Command node` antes de reinstalar.
3. **Usar `sudo npm install -g`.** Si npm pide permisos de administrador, la instalación de Node.js es la que hay que corregir.
4. **Ejecutar desde la carpeta equivocada.** `node hola.js` busca el archivo en la carpeta actual; comprueba con `pwd` dónde estás. Trabaja dentro de la carpeta del curso para que se apliquen su `.nvmrc` y su `package.json`.
5. **Abrir un archivo suelto en el editor.** Abre la carpeta del curso completa para que la terminal integrada y el depurador trabajen en el lugar correcto.

### Viniendo de Python

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

### Ambiente

| Tarea | En Python | En Node.js |
|---|---|---|
| Ver la versión | `python --version` | `node --version` |
| Instalar una versión | `pyenv install 3.13` | `fnm install 24` |
| Versión por defecto | `pyenv global 3.13` | `fnm default 24` |
| Versión del proyecto | `.python-version` | `.nvmrc` + `fnm use` |
| Dónde está el ejecutable | `which python` | `which node` / `Get-Command node` |
| Consola interactiva | `python` | `node` (salir con `.exit`) |
| Reiniciar al guardar | — | `node --watch programa.js` |
| Depurar | Depurador de VS Code | *JavaScript Debug Terminal* en VS Code |

### Sintaxis

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
