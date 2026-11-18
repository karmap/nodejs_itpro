# Clase 06: Módulos, archivos y npm

**Duración:** 2 horas

> Los bloques de desarrollo se leen en orden dentro de cada ejemplo. Las plantillas con `...` se completan en las actividades; las cheat sheets reúnen operaciones independientes. Los programas completos incluyen sus imports.

### Objetivos

- Dividir un programa en módulos con `export` e `import`.
- Usar los módulos incorporados de Node con el prefijo `node:`.
- Reconocer el equivalente de `if __name__ == "__main__"`.
- Leer y escribir archivos de texto y JSON con `node:fs/promises`.
- Construir rutas con `node:path` e `import.meta.dirname`.
- Crear un proyecto con `package.json` e instalar paquetes con npm.
- Leer variables de entorno y cargar un archivo `.env` sin librerías.
- Reiniciar un programa automáticamente con `node --watch`.

---

## 1. Módulos en JavaScript

En el curso de Python (clase 07), cada archivo `.py` era un módulo y podíamos importar sus funciones:

```python
# products.py
def calcular_valor(productos):
    ...

# main.py
from products import calcular_valor
```

En JavaScript cada archivo `.js` también es un módulo, con una diferencia importante: **nada es público por defecto**. Una función solo se puede importar si el módulo la marca con `export`.

Usaremos los **módulos ES** (ESM), el formato estándar de JavaScript, el mismo que usan los navegadores. Desde la clase 05 ya escribimos imports de este tipo, como `import { setTimeout as esperar } from "node:timers/promises";`.

---

## 2. Exportar e importar

Crea una carpeta `modulos/` con dos archivos.

### `calculos.js`

```javascript
export const IVA = 0.16;

export function calcularValor(productos) {
  return productos.reduce((total, p) => total + p.precio * p.stock, 0);
}

export function precioConIva(precio) {
  return precio * (1 + IVA);
}
```

### `main.js`

```javascript
import { calcularValor, precioConIva, IVA } from "./calculos.js";

const productos = [
  { nombre: "Laptop", precio: 15000, stock: 5 },
  { nombre: "Mouse", precio: 450, stock: 12 }
];

console.log("Valor:", calcularValor(productos));
console.log("Mouse con IVA:", precioConIva(450));
console.log("Tasa:", IVA);
```

Desde `modulos/`:

```bash
node main.js
```

Salida:

```text
Valor: 80400
Mouse con IVA: 522
Tasa: 0.16
```

- `export` delante de una función, `const` o `class` la hace importable. Lo que no se exporta queda privado dentro del módulo.
- `import { a, b } from "./archivo.js"` importa esos nombres. Las llaves no crean un objeto: indican qué exportaciones queremos.
- `"./calculos.js"` es una ruta relativa al archivo que importa, no a la terminal.

| En Python | En JavaScript |
|---|---|
| Todo lo definido en el módulo se puede importar | Solo lo marcado con `export` |
| `from products import calcular_valor` | `import { calcularValor } from "./products.js";` |
| `from products import calcular_valor as valor` | `import { calcularValor as valor } from "./products.js";` |
| `import products` y `products.calcular_valor()` | `import * as products from "./products.js";` |
| `import json` | `import ... from "node:..."` (sección 4) |

### Rutas con extensión

En los módulos ES, las rutas relativas deben incluir la extensión. Si escribes:

```javascript
import { calcularValor } from "./calculos";
```

Node responde:

```text
Error [ERR_MODULE_NOT_FOUND]: Cannot find module '.../modulos/calculos' imported from .../modulos/sin_ext.js
Did you mean to import "./calculos.js"?
```

En Python escribíamos `from products import ...` sin `.py`. En Node, escribe siempre `./products.js`. Las rutas empiezan con `./` (misma carpeta) o `../` (carpeta superior); un nombre sin ruta, como `"zod"`, indica un paquete instalado.

---

## 3. Export default e `import * as`

Un módulo puede tener, además, **una** exportación por defecto. Se importa sin llaves y con el nombre que elijas:

### `formato.js`

```javascript
export default function formatearPrecio(precio) {
  return `$${precio.toFixed(2)}`;
}
```

```javascript
import formatearPrecio from "./formato.js";

console.log(formatearPrecio(522));
```

`import * as nombre` reúne todas las exportaciones nombradas en un objeto:

```javascript
import * as calculos from "./calculos.js";

console.log(calculos.calcularValor([{ precio: 10, stock: 3 }]));
console.log(calculos.IVA);
```

Así importamos `readline` en la clase 05: `import * as readline from "node:readline/promises";`.

| Forma | Exportar | Importar |
|---|---|---|
| Nombrada | `export function f() {}` | `import { f } from "./m.js";` |
| Por defecto | `export default function f() {}` | `import cualquierNombre from "./m.js";` |
| Todo el módulo | Exportaciones nombradas | `import * as m from "./m.js";` |

En este curso preferimos las exportaciones nombradas: el nombre importado coincide con el exportado y el editor puede sugerirlo. Algunas bibliotecas, como Express en la clase 09, usan export default.

---

## 4. Módulos incorporados: `node:`

Python incluía una biblioteca estándar (`json`, `pathlib`, `os`, `time`). Node incluye módulos incorporados que se importan con el prefijo `node:`:

| Módulo | Uso | Equivalente en Python |
|---|---|---|
| `node:fs/promises` | Leer y escribir archivos | `open()`, `pathlib.Path.read_text()` |
| `node:path` | Construir y analizar rutas | `pathlib`, `os.path` |
| `node:process` | Entrada, salida, entorno, salida del proceso | `sys`, `os.environ` |
| `node:readline/promises` | Leer líneas de la terminal | `input()` |
| `node:timers/promises` | Esperas con Promises | `time.sleep()` |
| `node:http` | Servidor HTTP (clase 08) | `http.server` |
| `node:sqlite` | Base de datos SQLite (clase 10) | `sqlite3` |

El prefijo `node:` es opcional en muchos módulos, pero lo usaremos siempre: deja claro que no es un paquete instalado con npm.

`JSON`, `console`, `Math`, `setTimeout`, `fetch` y `process` son **globales**: están disponibles sin import.

---

## 5. Un módulo se ejecuta una vez

Al importar un módulo, Node ejecuta su código de nivel superior una sola vez, antes que el código del módulo que lo importa. Si otro archivo vuelve a importarlo, reutiliza el resultado. Igual ocurría en Python.

### `import.meta.main`: el equivalente de `__name__`

En Python usábamos:

```python
if __name__ == "__main__":
    main()
```

para ejecutar `main()` solo cuando el archivo se ejecutaba directamente. En Node 24, `import.meta.main` es `true` en el archivo que pasamos a `node` y `false` en los módulos importados.

Agrega al final de `calculos.js`:

```javascript
console.log("calculos.js - import.meta.main:", import.meta.main);
```

y al final de `main.js`:

```javascript
console.log("main.js - import.meta.main:", import.meta.main);
```

Ejecuta los dos archivos:

```bash
node main.js
node calculos.js
```

Salida de `node main.js`:

```text
calculos.js - import.meta.main: false
Valor: 80400
Mouse con IVA: 522
Tasa: 0.16
main.js - import.meta.main: true
```

Salida de `node calculos.js`:

```text
calculos.js - import.meta.main: true
```

La primera línea aparece antes que las de `main.js`: los imports se resuelven y ejecutan primero. Retira esos dos `console.log` después de la prueba.

El patrón que usaremos:

```javascript
async function main() {
  // Código del programa.
}

if (import.meta.main) {
  await main();
}
```

`import.meta` es un objeto con información del módulo actual. `import.meta.main` existe desde Node 24.2; en versiones anteriores es `undefined`.

### CommonJS: el formato antiguo

En tutoriales y proyectos existentes verás otro formato de módulos, **CommonJS**:

```javascript
const fs = require("node:fs");

function calcularValor(productos) {
  // ...
}

module.exports = { calcularValor };
```

`require()` y `module.exports` cumplen el papel de `import` y `export`. Basta con reconocerlos: en este curso usamos módulos ES. No mezcles ambos formatos en el mismo archivo.

---

## Actividad 1: Dividir el procesador de productos

Toma el procesador de productos de la clase 04 y divídelo:

```text
procesador/
├── products.js
├── reports.js
└── main.js
```

1. `products.js` exporta `productosDisponibles()`, `calcularValor()` y `buscarProducto()`.
2. `reports.js` exporta `mostrarProductos()` y puede importar de `products.js` lo que necesite.
3. `main.js` define los datos, importa las funciones y ejecuta `main()` solo si `import.meta.main` es `true`.
4. Comprueba que `node products.js` no imprima nada.
5. Quita la extensión `.js` de un import y lee el mensaje de error.

---

## 6. Archivos con `node:fs/promises`

Leer y escribir archivos es una operación lenta, así que en Node se hace con Promises y `await`:

```javascript
import { mkdir, readFile, writeFile, appendFile } from "node:fs/promises";
import path from "node:path";

const carpeta = path.join(import.meta.dirname, "data");
const archivo = path.join(carpeta, "notas.txt");

await mkdir(carpeta, { recursive: true });
await writeFile(archivo, "Primera línea\n", "utf8");
await appendFile(archivo, "Segunda línea\n", "utf8");

const texto = await readFile(archivo, "utf8");
console.log(texto);

const lineas = texto.trim().split("\n");
console.log(lineas.length, "líneas");
```

Salida:

```text
Primera línea
Segunda línea

2 líneas
```

| En Python | En Node |
|---|---|
| `open(ruta, encoding="utf-8").read()` | `await readFile(ruta, "utf8")` |
| `open(ruta, "w", encoding="utf-8").write(texto)` | `await writeFile(ruta, texto, "utf8")` |
| `open(ruta, "a", encoding="utf-8").write(texto)` | `await appendFile(ruta, texto, "utf8")` |
| `Path(carpeta).mkdir(parents=True, exist_ok=True)` | `await mkdir(carpeta, { recursive: true })` |
| `with open(...)` cierra el archivo | Estas funciones abren y cierran el archivo solas |

- Pasa siempre `"utf8"`. Sin codificación, `readFile()` devuelve un `Buffer` (bytes), parecido a leer en modo `"rb"`.
- `writeFile()` reemplaza el contenido, como el modo `"w"`. Crea el archivo si no existe, pero no crea carpetas.
- `mkdir()` con `recursive: true` crea las carpetas intermedias y no falla si ya existen.

Existen versiones síncronas en `node:fs`, como `readFileSync()`. Bloquean el hilo mientras leen; en este curso usaremos las asíncronas.

### Rutas con `node:path` e `import.meta.dirname`

Una ruta relativa como `"data/productos.json"` se interpreta desde la carpeta de la **terminal**, no desde la del archivo. Igual que en Python con `open()`. Para ubicar archivos junto al código usamos `import.meta.dirname`:

```javascript
import path from "node:path";

console.log(import.meta.dirname);
console.log(import.meta.filename);
console.log(process.cwd());
console.log(path.join(import.meta.dirname, "data", "productos.json"));
console.log(path.basename("data/productos.json"));
console.log(path.extname("data/productos.json"));
```

Salida al ejecutar `node modulos/rutas.js` desde la carpeta superior (las rutas dependen de tu equipo):

```text
/Users/ana/curso/modulos
/Users/ana/curso/modulos/rutas.js
/Users/ana/curso
/Users/ana/curso/modulos/data/productos.json
productos.json
.json
```

| En Python | En Node |
|---|---|
| `Path(__file__).resolve()` | `import.meta.filename` |
| `Path(__file__).resolve().parent` | `import.meta.dirname` |
| `Path.cwd()` | `process.cwd()` |
| `carpeta / "data" / "productos.json"` | `path.join(carpeta, "data", "productos.json")` |
| `Path(ruta).name` | `path.basename(ruta)` |
| `Path(ruta).suffix` | `path.extname(ruta)` |
| `Path(ruta).parent` | `path.dirname(ruta)` |

`path.join()` usa el separador correcto en cada sistema operativo (`/` en macOS y Linux, `\` en Windows). No construyas rutas concatenando textos.

---

## 7. Errores de archivos y JSON

En Python capturábamos `FileNotFoundError`. En Node, los errores del sistema son objetos `Error` con una propiedad `code`:

```javascript
import { readFile } from "node:fs/promises";

try {
  const texto = await readFile("no_existe.json", "utf8");
  console.log(JSON.parse(texto));
} catch (error) {
  if (error.code === "ENOENT") {
    console.log("El archivo no existe:", error.path);
  } else if (error instanceof SyntaxError) {
    console.log("El archivo no contiene JSON válido");
  } else {
    throw error;
  }
}
```

Salida:

```text
El archivo no existe: no_existe.json
```

| En Python | Código en Node |
|---|---|
| `FileNotFoundError` | `"ENOENT"` (no existe el archivo o la carpeta) |
| `PermissionError` | `"EACCES"` o `"EPERM"` |
| `IsADirectoryError` | `"EISDIR"` |
| `json.JSONDecodeError` | `SyntaxError` de `JSON.parse()` |

### Leer y guardar JSON

No hay un equivalente de `json.load(archivo)`: leemos el texto y lo convertimos con `JSON.parse()`, que vimos en la clase 03.

```javascript
import { mkdir, readFile, writeFile } from "node:fs/promises";
import path from "node:path";

const archivo = path.join(import.meta.dirname, "data", "productos.json");

const productos = [
  { nombre: "Laptop", precio: 15000, stock: 5 },
  { nombre: "Audífonos", precio: 1200, stock: 0 }
];

await mkdir(path.dirname(archivo), { recursive: true });
await writeFile(archivo, JSON.stringify(productos, null, 2), "utf8");

const cargados = JSON.parse(await readFile(archivo, "utf8"));
console.log(cargados[1].nombre);
console.log(Array.isArray(cargados));
```

Salida:

```text
Audífonos
true
```

| En Python | En Node |
|---|---|
| `json.load(archivo)` | `JSON.parse(await readFile(ruta, "utf8"))` |
| `json.dump(datos, archivo, indent=4, ensure_ascii=False)` | `await writeFile(ruta, JSON.stringify(datos, null, 2), "utf8")` |

`JSON.stringify()` conserva los acentos; no necesita una opción como `ensure_ascii=False`.

Como en el curso de Python, las funciones de almacenamiento pueden dejar propagar estos errores. El programa principal los captura: así una lectura fallida no se confunde con un inventario vacío y no se sobrescribe el archivo por accidente.

---

## 8. npm y `package.json`

En el curso de Python (clase 08) usamos `venv` para aislar dependencias, `pip` para instalarlas y `requirements.txt` para fijar versiones. Node reúne esas tareas en **npm**, que se instala junto con Node, y en el archivo `package.json`.

| En Python | En Node |
|---|---|
| `python -m venv .venv` | `npm init -y` (crea `package.json`) |
| Activar el entorno | No es necesario |
| `.venv/` | `node_modules/` |
| `python -m pip install requests` | `npm install zod` |
| `requirements.txt` | `dependencies` en `package.json` |
| Versiones exactas fijadas | `package-lock.json` |
| `python -m pip install -r requirements.txt` | `npm install` |
| `python -m pip uninstall paquete` | `npm uninstall paquete` |
| `python -m pip list` | `npm ls` |

### Crear el proyecto

Crea una carpeta `inventario/` y, desde ella:

```bash
npm init -y
```

`-y` acepta las respuestas por defecto. npm crea `package.json`:

```json
{
  "name": "inventario",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "type": "commonjs"
}
```

npm 11 escribe `"type": "commonjs"`. Cámbialo a `"module"` para usar `import` y `export`:

```bash
npm pkg set type=module
```

También puedes editar el archivo a mano. `"type": "module"` indica que los archivos `.js` del proyecto son módulos ES. La raíz del repositorio del curso ya incluye un `package.json` con esta opción.

### Instalar un paquete

Un **paquete** es una biblioteca publicada en el registro de npm, como los paquetes de PyPI. Instalaremos Zod, que usaremos en la clase 09 para validar datos como hacíamos con Pydantic:

```bash
npm install zod@4
```

`@4` pide la versión mayor 4. npm descarga el paquete y:

1. Lo guarda en `node_modules/`, dentro del proyecto.
2. Lo agrega a `package.json`:

```json
"dependencies": {
  "zod": "^4.6.7"
}
```

3. Registra la versión exacta instalada, y las de sus dependencias, en `package-lock.json`.

Comprueba que se puede importar. Crea `prueba_zod.js`:

```javascript
import * as z from "zod";

const Nombre = z.string().trim().min(1);

console.log(Nombre.safeParse("  Mouse  ").data);
console.log(Nombre.safeParse("   ").success);
```

```bash
node prueba_zod.js
```

Salida:

```text
Mouse
false
```

Un nombre sin ruta, como `"zod"`, se busca en `node_modules/`. En la clase 09 veremos Zod en detalle.

No hay que activar nada: Node busca `node_modules/` en la carpeta del archivo y, si no lo encuentra, en las carpetas superiores. Cada proyecto tiene sus propias dependencias, como con un entorno virtual.

### Versiones y `package-lock.json`

`^4.6.7` significa "4.6.7 o una versión posterior compatible dentro de 4.x". npm sigue el versionado semántico `MAYOR.MENOR.PARCHE`: un cambio de versión mayor puede romper el código existente.

| Escritura | Acepta |
|---|---|
| `"4.6.7"` | Exactamente 4.6.7 |
| `"~4.6.7"` | Parches: 4.6.x |
| `"^4.6.7"` | Menores y parches: 4.x.x |

`package-lock.json` fija la versión exacta que se instaló. Súbelo a Git: así todos instalan las mismas versiones. Si quieres que `package.json` guarde la versión exacta, usa `npm install zod@4.6.7 --save-exact`.

### Reinstalar y quitar

`node_modules/` puede ocupar mucho espacio y se reconstruye desde `package.json` y `package-lock.json`. **No lo subas a Git**, como no subíamos `.venv/`. Al clonar un proyecto:

```bash
npm install
```

`npm ci` hace una instalación limpia que respeta exactamente `package-lock.json`; es la opción habitual en servidores de integración continua.

Para quitar un paquete:

```bash
npm uninstall zod
```

No lo quites ahora: lo usaremos en la clase 09.

### Dependencias de desarrollo y `npx`

`npm install -D paquete` agrega el paquete a `devDependencies`: herramientas que solo se usan al desarrollar, como formateadores o pruebas. `npx paquete` ejecuta el comando de un paquete sin agregarlo al proyecto.

### Scripts

La sección `scripts` de `package.json` guarda comandos con nombre:

```json
"scripts": {
  "start": "node main.js",
  "dev": "node --watch main.js"
}
```

```bash
npm start
npm run dev
node --run dev
```

- `npm start` es un atajo para el script `start`.
- `npm run nombre` ejecuta cualquier script.
- `node --run nombre` también lo ejecuta, más rápido y sin mostrar el encabezado de npm.

Los scripts documentan cómo se ejecuta el proyecto. Cualquier persona puede clonar el repositorio, ejecutar `npm install` y después `npm start`.

### `.gitignore`

```text
node_modules/
.env
.env.*
!.env.example
*.db
```

`!.env.example` vuelve a incluir ese archivo, que sí se sube porque no contiene claves (sección 9). `*.db` excluye las bases SQLite de la clase 10.

---

## Actividad 2: Preparar un proyecto

1. Crea la carpeta `inventario/`, ejecuta `npm init -y` y cambia el tipo a `module`.
2. Instala `zod@4` y localiza el paquete en `package.json`, `package-lock.json` y `node_modules/`.
3. Ejecuta `prueba_zod.js`.
4. Agrega los scripts `start` y `dev`, aunque `main.js` todavía no exista, y crea el `.gitignore`.
5. Borra la carpeta `node_modules/` desde el explorador de archivos, ejecuta `npm install` y comprueba que `prueba_zod.js` vuelve a funcionar.

---

## 9. Variables de entorno

En Python leíamos la clave con `os.getenv("OPENROUTER_API_KEY")`. En Node, el entorno está en `process.env`, un objeto global:

```javascript
const apiKey = process.env.OPENROUTER_API_KEY;

if (!apiKey) {
  console.log("Configura OPENROUTER_API_KEY antes de ejecutar el programa");
  process.exit(1);
}

console.log("Clave configurada con", apiKey.length, "caracteres");
```

- Una variable que no existe vale `undefined`, como `None` con `os.getenv()`.
- Los valores son siempre texto. `process.env.PORT` es `"8000"`, no `8000`; conviértelo con `Number()` si lo necesitas.
- `process.exit(1)` termina el programa con un código de error, como `sys.exit(1)`.
- Nunca imprimas la clave. Este programa solo muestra su longitud para comprobar que se cargó.

Puedes configurar la variable en la terminal, igual que en el curso de Python:

macOS/Linux:

```bash
export OPENROUTER_API_KEY="TU_CLAVE"
```

Windows PowerShell:

```powershell
$env:OPENROUTER_API_KEY = "TU_CLAVE"
```

Windows CMD:

```bat
set OPENROUTER_API_KEY=TU_CLAVE
```

### Archivo `.env` con `--env-file`

Escribir la clave en cada terminal nueva es incómodo. Node puede cargar un archivo `.env` **sin instalar ninguna librería**. Crea `.env` junto al programa:

```text
OPENROUTER_API_KEY=TU_CLAVE
```

y ejecuta:

```bash
node --env-file=.env clave.js
```

Si `.env` no existe, `--env-file` termina con un error. `--env-file-if-exists=.env` continúa sin él y deja que el programa decida qué hacer:

```bash
node --env-file-if-exists=.env clave.js
```

Si la misma variable está en la terminal y en el archivo, gana el valor de la terminal.

Reglas para claves:

- `.env` va en `.gitignore`. Comprueba con `git status` que no aparezca antes de cualquier commit.
- Sube un `.env.example` con los nombres de las variables y valores ficticios, para documentar qué necesita el proyecto:

```text
OPENROUTER_API_KEY=pon-aqui-tu-clave
```

- La opción puede ir en un script para no repetirla:

```json
"scripts": {
  "start": "node --env-file-if-exists=.env main.js"
}
```

---

## 10. `node --watch`

`node --watch main.js` ejecuta el programa y lo reinicia cada vez que guardas un archivo que importa. Es el equivalente del modo de recarga de `fastapi dev`. Lo usaremos con los servidores de las clases 08 a 10. Detenlo con `Ctrl+C`.

```bash
node --watch --env-file-if-exists=.env main.js
```

No es útil con programas interactivos de `readline`: cada reinicio interrumpe lo que estabas escribiendo.

---

## Errores comunes viniendo de Python

1. **Omitir la extensión en los imports.** `"./products"` falla; escribe `"./products.js"`.
2. **Olvidar `export`.** En Python todo era importable; en JavaScript, importar algo no exportado produce un `SyntaxError` que indica que el módulo no proporciona ese nombre.
3. **Dejar `"type": "commonjs"`.** `npm init -y` lo escribe así; con ese valor, un `import` en un archivo `.js` falla con `SyntaxError: Cannot use import statement outside a module`. Cámbialo a `"module"`.
4. **Subir `node_modules/` o `.env` a Git.** Agrégalos a `.gitignore` antes del primer commit.
5. **Leer sin `"utf8"`.** `readFile(ruta)` devuelve un `Buffer`, no un texto.
6. **Rutas relativas a la terminal.** `"data/productos.json"` depende de la carpeta desde donde ejecutas `node`; usa `path.join(import.meta.dirname, ...)`.
7. **Buscar `FileNotFoundError`.** En Node comprueba `error.code === "ENOENT"`.
8. **Tratar `process.env` como números o booleanos.** Todos los valores son texto o `undefined`.

---

## Cheat sheet

| Tarea | Python | Node |
|---|---|---|
| Exportar | Automático | `export function f() {}` |
| Importar nombres | `from m import f` | `import { f } from "./m.js";` |
| Importar todo | `import m` | `import * as m from "./m.js";` |
| Ejecutar solo directo | `if __name__ == "__main__":` | `if (import.meta.main) { ... }` |
| Carpeta del archivo | `Path(__file__).resolve().parent` | `import.meta.dirname` |
| Unir rutas | `carpeta / "data"` | `path.join(carpeta, "data")` |
| Leer texto | `Path(r).read_text(encoding="utf-8")` | `await readFile(r, "utf8")` |
| Escribir texto | `Path(r).write_text(t, encoding="utf-8")` | `await writeFile(r, t, "utf8")` |
| Archivo inexistente | `except FileNotFoundError` | `error.code === "ENOENT"` |
| Variable de entorno | `os.getenv("CLAVE")` | `process.env.CLAVE` |
| Crear proyecto | `python -m venv .venv` | `npm init -y` y `"type": "module"` |
| Instalar paquete | `pip install requests` | `npm install zod` |
| Instalar dependencias | `pip install -r requirements.txt` | `npm install` |
| Ejecutar script | — | `npm run dev` o `node --run dev` |

```bash
npm init -y
npm pkg set type=module
npm install zod@4
npm install
npm uninstall zod
npm ls
npm start
npm run dev
node --run dev
node --watch main.js
node --env-file=.env main.js
node --env-file-if-exists=.env main.js
```

---

## Ejercicio final: Inventario persistente

Es la versión en Node del ejercicio de la clase 08 de Python. Trabaja en el proyecto `inventario/` de la actividad 2. Crea `data/productos.json` con:

```json
[
  { "nombre": "Laptop", "precio": 15000, "stock": 5 },
  { "nombre": "Mouse", "precio": 450, "stock": 12 },
  { "nombre": "Teclado", "precio": 900, "stock": 8 },
  { "nombre": "Monitor", "precio": 4200, "stock": 3 },
  { "nombre": "Audifonos", "precio": 1200, "stock": 0 },
  { "nombre": "Webcam", "precio": 800, "stock": 6 }
]
```

Estructura:

```text
inventario/
├── package.json
├── package-lock.json
├── .gitignore
├── main.js
├── products.js
├── storage.js
└── data/
    └── productos.json
```

### `storage.js`

```javascript
import { readFile, writeFile } from "node:fs/promises";

export async function cargarProductos(archivo) {
  const texto = await readFile(archivo, "utf8");
  return JSON.parse(texto);
}

export async function guardarProductos(archivo, productos) {
  const texto = JSON.stringify(productos, null, 2);
  await writeFile(archivo, texto + "\n", "utf8");
}
```

Las funciones dejan propagar los errores de lectura y de JSON; `main.js` los captura.

### `products.js`

Completa y exporta:

```javascript
export function buscarProducto(productos, nombre) {
  // Completa esta función durante la actividad.
}

export function productosDisponibles(productos) {
  // Completa esta función durante la actividad.
}

export function actualizarStock(productos, nombre, cantidad) {
  // Completa esta función durante la actividad.
}
```

- `buscarProducto()` ignora espacios exteriores y mayúsculas; devuelve el producto o `undefined`.
- `productosDisponibles()` devuelve un array nuevo con los productos de stock mayor que cero.
- `actualizarStock()` lanza `RangeError` si `cantidad` no es un entero no negativo. Si el producto no existe, devuelve `false`; si existe, reemplaza su stock y devuelve `true`. `cantidad` es el nuevo total, no un incremento.

### `main.js`

Importa las funciones de `products.js` y `storage.js`, define una función `async function main()` y llámala solo si `import.meta.main` es `true`.

El programa debe:

1. Construir la ruta del archivo con `path.join(import.meta.dirname, "data", "productos.json")`.
2. Cargar los productos.
3. Mostrar los productos disponibles.
4. Pedir el nombre de un producto con `readline` y buscarlo. Si no existe, mostrar `Producto no encontrado` y terminar.
5. Pedir el nuevo stock y convertirlo con `Number()`. Trata el texto vacío como inválido: `Number("")` devuelve `0`.
6. Actualizar el stock y guardar el archivo **solo** si la actualización devuelve `true`.
7. Capturar `ENOENT`, `SyntaxError` y `RangeError` con mensajes comprensibles, y volver a lanzar cualquier otro error.
8. Cerrar la interfaz de `readline` en un bloque `finally`.

Agrega a `package.json`:

```json
"scripts": {
  "start": "node main.js",
  "dev": "node --watch main.js"
}
```

y ejecuta con `npm start`.

### Salida esperada aproximada

```text
PRODUCTOS DISPONIBLES
Laptop - $15000 - Stock: 5
Mouse - $450 - Stock: 12
Teclado - $900 - Stock: 8
Monitor - $4200 - Stock: 3
Webcam - $800 - Stock: 6

Producto a buscar: mouse
Encontrado: Mouse (stock: 12)
Nuevo stock: 20
Inventario guardado
```

Comprueba también:

- Un stock `abc`, `-3`, `2.5` o vacío muestra un mensaje y no modifica el archivo.
- Un producto inexistente no modifica el archivo.
- El programa funciona igual si lo ejecutas desde la carpeta superior con `node inventario/main.js`.
- Si renombras `data/productos.json`, aparece un mensaje de archivo inexistente en lugar de la traza del error.
- Si el archivo contiene JSON inválido, el programa lo indica y no lo sobrescribe.

### Práctica opcional

Agrega un menú en bucle, como el de la clase 05, con opciones para ver productos, actualizar stock y salir. Guarda después de cada actualización correcta.

---

Siguiente paso: en la clase 07 usaremos `fetch` para consumir APIs, incluida la API de IA de OpenRouter con la clave configurada en el entorno.

---

[Índice del curso](README.md)
