# Node.js aplicado: de Python a JavaScript

Curso introductorio de Node.js en español para quienes ya completaron el curso **Python aplicado: APIs, IA y bases de datos**. No vuelve a enseñar a programar: parte de lo que ya sabes en Python y muestra, tema por tema, cómo se escribe lo mismo en JavaScript, qué cambia y qué errores son comunes al hacer el cambio.

La progresión es paralela a la del curso de Python. Repasa las estructuras de datos y las funciones, estudia la asincronía, que es lo verdaderamente nuevo, y después reconstruye los mismos programas con Node: el analizador de reseñas con IA, el servidor HTTP manual, la Products API y su persistencia en SQLite.

Los materiales están pensados para demostraciones en vivo y práctica en clases de 1 a 2 horas. Cada clase incluye tablas de equivalencias con Python, actividades, un ejercicio final, una sección de errores comunes y una cheat sheet.

Las explicaciones introducen la sintaxis antes de usarla en programas completos. Los fragmentos que dependen de código previo lo indican; las plantillas con `// ...` forman parte de los ejercicios. Ejecuta los programas desde la carpeta indicada y sigue el orden del índice.

## Requisitos

- Haber completado el curso de Python aplicado, o tener un nivel equivalente: estructuras de datos, funciones, excepciones, JSON, HTTP, consumo y creación de APIs.
- Node.js 24 LTS o superior. La edición se comprobó con Node.js 24.15.0 y npm 11.12.1. La clase 01 explica cómo instalarlo con un gestor de versiones; el archivo `.nvmrc` de la raíz fija la versión del curso.
- Un editor (recomendamos VS Code) y una terminal.
- Acceso a Internet y una API key de OpenRouter para las actividades de IA. La clave se configura fuera del código y del repositorio.

Comprueba la instalación en una terminal nueva:

```bash
node --version
```

```bash
npm --version
```

Las clases 01 a 05 solo usan Node.js. Los paquetes de npm se introducen en la clase 06; el `package.json` de la raíz fija las versiones comprobadas de Express y Zod. Para instalarlas desde la raíz del curso:

```bash
npm install
```

Crea tus archivos de práctica dentro de la carpeta del curso, por ejemplo en una subcarpeta `practicas/`. El `package.json` de la raíz contiene `"type": "module"`, que hace que Node.js trate cada `.js` como un módulo ES; la clase 06 explica este archivo. Desde la clase 06, cada proyecto tiene su propio `package.json`.

No es necesario instalar todo desde la primera clase: cada documento indica las dependencias que utiliza. La clave de OpenRouter se configura como variable de entorno en la clase 06.

## Contenido

Las clases siguen el patrón `clase_NN_tema.md`, con numeración continua del 00 al 10. El proyecto final tiene su propio documento al final del índice. Entre paréntesis aparece la clase equivalente del curso de Python.

### 0. [De JavaScript en el navegador a Node.js](clase_00_historia_de_javascript.md)

Línea de tiempo de JavaScript desde 1995: ECMAScript, V8, el nacimiento de Node.js y su event loop, npm, ES2015, `async` / `await` y el ecosistema actual. Explica de dónde vienen las rarezas del lenguaje y cómo distinguir el código antiguo del moderno. No requiere instalar nada.

### 1. [Node.js en tu computadora y de Python a JavaScript](clase_01_de_python_a_javascript.md)

Parte 1: instalación de Node.js con un gestor de versiones (fnm o nvm) en macOS, Windows y Linux, ambiente local (terminal, VS Code, carpeta del curso, `.nvmrc`), REPL, `--watch`, depurador y solución de problemas de instalación. Parte 2: variables, tipos, conversiones, comparaciones, valores truthy y falsy, condiciones y ciclos. (Base de todo el curso de Python.)

### 2. [Arrays y Set](clase_02_arrays_y_set.md)

Listas, tuplas y conjuntos de Python expresados con `Array` y `Set`: acceso, modificación, copias, ordenamiento, desestructuración y operaciones entre conjuntos. (Clases 01 a 03.)

### 3. [Objetos, Map y JSON](clase_03_objetos_map_y_json.md)

Diccionarios como objetos, acceso seguro con `?.` y `??`, spread, desestructuración, comparación por referencia, `Map` y `JSON.stringify` / `JSON.parse`. (Clases 04 y 08.)

### 4. [Funciones, arrow functions y callbacks](clase_04_funciones_y_callbacks.md)

Parámetros, objetos de opciones, arrow functions como equivalente de `lambda`, `map`, `filter`, `reduce`, closures y anotaciones con JSDoc. (Clases 05 y 07.)

### 5. [Errores y asincronía](clase_05_errores_y_asincronia.md)

`try` / `catch`, errores propios, event loop, promesas, `async` / `await`, `Promise.all` y entrada de usuario con `readline`. (Clase 06 y un tema nuevo.)

### 6. [Módulos, archivos y npm](clase_06_modulos_archivos_y_npm.md)

Módulos ES, `node:fs/promises`, `node:path`, `package.json`, npm, variables de entorno y un inventario persistente en varios módulos. (Clases 06, 07 y 08.)

### 7. [HTTP, APIs e IA con fetch](clase_07_http_apis_ia_fetch.md)

`fetch` como equivalente de `requests`, timeouts, errores HTTP, OpenRouter y el analizador de reseñas con IA. (Clase 09.)

### 8. [Servidor HTTP con node:http](clase_08_servidor_http.md)

Servidor sin frameworks, rutas, códigos de estado, respuestas JSON, lectura del body y pruebas con navegador, `curl` y `fetch`. (Clase 10.)

### 9. [Express y Zod](clase_09_express_y_zod.md)

Products API con Express, validación de entrada y de respuestas de IA con Zod, errores 404 y 422, DELETE y PUT como bonus. (Clase 11.)

### 10. [Persistencia con SQLite](clase_10_sqlite.md)

Base de datos con el módulo incorporado `node:sqlite`, consultas parametrizadas y almacenamiento de resultados de IA como JSON. (Clase 12.)

Esta clase es obligatoria antes del proyecto final.

### 11. [Proyecto final: Generador de tarjetas de estudio con historial](proyecto_final.md)

Cliente de consola que crea y consulta mazos de tarjetas de estudio mediante una API con Express, OpenRouter, Zod y SQLite. Incluye datos de entrada y salida, rutas, errores y criterios de entrega.

## Equivalencias principales

| Python | Node.js |
|---|---|
| `python archivo.py` | `node archivo.js` |
| `print()` | `console.log()` |
| `None` | `null` y `undefined` |
| `list`, `tuple`, `set`, `dict` | `Array`, `Array` / `Object.freeze`, `Set`, objeto o `Map` |
| List comprehension | `map()` y `filter()` |
| `lambda` | Arrow function `(x) => ...` |
| `try` / `except` / `raise` | `try` / `catch` / `throw` |
| `input()` | `readline/promises` con `await` |
| `pyenv` + `.python-version` | `fnm` / `nvm` + `.nvmrc` |
| `venv` + `pip` + `requirements.txt` | `node_modules` + `npm` + `package.json` |
| `os.getenv()` | `process.env` |
| `pathlib` y `open()` | `node:path` y `node:fs/promises` |
| `requests` | `fetch` incorporado |
| `http.server` | `node:http` |
| FastAPI | Express |
| Pydantic | Zod |
| SQLModel + SQLite | `node:sqlite` |

## Progresión del curso

```text
Historia: de JavaScript a Node.js
  ↓
Instalación y ambiente local
  ↓
Python que ya conoces
  ↓
Sintaxis de JavaScript
  ↓
Arrays, objetos y funciones
  ↓
Errores y asincronía
  ↓
Módulos, archivos y npm
  ↓
fetch / APIs de IA
  ↓
Servidor HTTP con node:http
  ↓
Express + Zod
  ↓
SQLite
  ↓
Proyecto final
```
