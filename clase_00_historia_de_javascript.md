# Clase 00: De JavaScript en el navegador a Node.js

**Duración:** 1 hora

> Esta clase no requiere instalar nada: las actividades se hacen en la consola del navegador. La instalación de Node.js y del ambiente local es el tema de la clase 01.

### Objetivos

- Conocer de dónde viene JavaScript y por qué tiene algunas rarezas que veremos en el curso.
- Distinguir JavaScript, ECMAScript, V8 y Node.js.
- Entender qué problema resolvió Node.js y por qué su modelo es asíncrono.
- Reconocer el JavaScript "antiguo" y el moderno cuando lo encuentres en Internet.
- Ubicar el estado actual del ecosistema: versiones LTS, npm, TypeScript y otros runtimes.

---

## 1. Por qué empezar con historia

En el curso de Python casi todo lo que aprendimos tenía una lógica clara: Python se diseñó con calma y su filosofía ("debería haber una forma obvia de hacerlo") se nota en el lenguaje. JavaScript tuvo un origen muy distinto: se diseñó en días, para una sola tarea pequeña, y terminó ejecutándose en miles de millones de dispositivos sin poder romper la compatibilidad con el código antiguo.

Eso explica muchas cosas que veremos en las próximas clases:

| Lo que verás | De dónde viene |
|---|---|
| `==` y `===` conviven | `==` es de 1995; `===` se agregó en 1999 para corregirlo sin romper la Web. |
| `var`, `let` y `const` | `var` es original; `let` y `const` llegaron en 2015. |
| `typeof null === "object"` | Un error de la primera implementación que nunca se pudo corregir. |
| Callbacks, promesas y `async` / `await` | Tres generaciones de soluciones al mismo problema: la asincronía. |
| `require()` e `import` | Node.js inventó su sistema de módulos antes de que el lenguaje tuviera uno. |

Regla del lenguaje que conviene recordar: **en JavaScript casi nada se elimina**. Las partes problemáticas se reemplazan por otras nuevas y las viejas se quedan. Por eso el curso indica qué forma usar y cuál solo reconocer.

---

## 2. Cuatro nombres que se confunden

| Nombre | Qué es | Equivalente aproximado en Python |
|---|---|---|
| **JavaScript** | El lenguaje que escribimos. | El lenguaje Python. |
| **ECMAScript** | La especificación oficial del lenguaje (estándar ECMA-262). Define la sintaxis y los objetos básicos, como `Array` o `Promise`. | La referencia del lenguaje y los PEP. |
| **Motor** (V8, SpiderMonkey, JavaScriptCore) | El programa que lee y ejecuta el código JavaScript. | El intérprete CPython. |
| **Runtime** (navegador, Node.js, Deno, Bun) | El motor más todo lo que le permite hablar con el mundo: archivos, red, pantalla, temporizadores. | CPython más su biblioteca estándar. |

El lenguaje por sí solo no sabe leer archivos ni abrir conexiones de red. El **runtime** decide qué puede hacer el código:

```text
                JavaScript (ECMAScript)
                         │
          ┌──────────────┴──────────────┐
          ▼                             ▼
     Navegador                       Node.js
   V8 + DOM, fetch,              V8 + fs, http, path,
   ventanas, clics               process, fetch, terminal
```

Por eso `document.querySelector()` existe en el navegador pero no en Node.js, y `fs.readFile()` existe en Node.js pero no en el navegador. Ambos ejecutan el mismo lenguaje.

---

## 3. Línea de tiempo

```text
1995 ── JavaScript nace en Netscape
1997 ── Primer estándar: ECMAScript 1
1999 ── ES3: try/catch, expresiones regulares, ===
2005 ── "Ajax": páginas que se actualizan sin recargar
2006 ── jQuery
2008 ── Chrome y el motor V8
2009 ── ES5 (JSON, map/filter) ─── Node.js
2010 ── npm y Express
2012 ── TypeScript
2015 ── ES2015 / ES6: let, const, =>, clases, promesas, módulos
       └─ Node.js se reunifica y empieza el modelo LTS
2017 ── async / await
2019 ── Módulos ES (import/export) sin flags en Node.js
2020 ── ?. y ??   ─── Deno 1.0
2022 ── fetch y node:test llegan a Node.js
2023 ── Bun 1.0
2024 ── node:sqlite, ejecución de TypeScript sin compilar
2025 ── Node.js 24 LTS (la versión de este curso)
2026 ── Node.js 26 LTS; Node.js pasa a una versión mayor por año
```

Las siguientes secciones recorren esa línea por etapas.

---

## 4. 1995–2004: un lenguaje para "pegar" cosas en la página

**1995.** Netscape, la empresa del navegador más usado de la época, quería un lenguaje sencillo para agregar interactividad a las páginas: validar un formulario, cambiar una imagen al pasar el mouse. Le encargó el trabajo a **Brendan Eich**, quien escribió el primer prototipo en unos diez días.

El lenguaje se llamó primero *Mocha*, luego *LiveScript* y finalmente **JavaScript**. El nombre fue una decisión de marketing: Java estaba de moda y Netscape tenía un acuerdo con Sun Microsystems. **JavaScript y Java no tienen relación**; se parecen tanto como "carro" y "carpeta".

**1996.** Microsoft creó su propia versión, *JScript*, para Internet Explorer 3. Dos implementaciones distintas de un lenguaje sin especificación era una receta para el caos, así que Netscape llevó el lenguaje a la organización de estándares **Ecma International**.

**1997.** Se publicó **ECMAScript 1**. Desde entonces "ECMAScript" es el nombre del estándar y "JavaScript" el del lenguaje que todos usamos.

**1999.** **ES3** agregó `try` / `catch`, expresiones regulares y `===`. Fue la versión de referencia durante diez años: el intento de una versión 4 muy ambiciosa se abandonó en 2008 por falta de acuerdo.

En esta etapa JavaScript tenía mala fama: se usaba para ventanas emergentes y efectos, cada navegador se comportaba distinto y los programadores "serios" lo evitaban.

---

## 5. 2004–2008: la Web se vuelve aplicación

**2004–2005.** Gmail y Google Maps mostraron que una página podía comportarse como un programa de escritorio: pedía datos al servidor en segundo plano y actualizaba solo una parte de la pantalla, sin recargar. En 2005 la técnica recibió un nombre, **Ajax**, y JavaScript pasó de adorno a pieza central.

Esa forma de trabajar ya era **asíncrona**: el navegador pide datos, sigue respondiendo a clics y, cuando llegan los datos, ejecuta una función. Ese patrón de "pide y avísame cuando termine" es la base de Node.js.

**2006.** **jQuery** simplificó el trabajo con el navegador y ocultó las diferencias entre Internet Explorer, Firefox y Safari. Durante años "saber JavaScript" significó, en la práctica, saber jQuery. Lo verás en tutoriales antiguos como `$(".boton").click(...)`.

**2008.** Google lanzó **Chrome** con un motor nuevo, **V8**, que compilaba JavaScript a código máquina mientras se ejecutaba (*JIT*, *just-in-time*). JavaScript pasó de ser lento a ser uno de los lenguajes dinámicos más rápidos, y los demás navegadores tuvieron que ponerse al día. V8 además era de código abierto y se podía usar fuera de Chrome.

---

## 6. 2009–2014: JavaScript sale del navegador

**2009.** Ocurren dos cosas el mismo año:

- **ES5** agregó modo estricto, `JSON.parse` / `JSON.stringify` y métodos de arrays como `forEach`, `map` y `filter`, que usaremos en las clases 02 a 04.
- **Ryan Dahl** presentó **Node.js**: tomó V8, le agregó acceso a archivos, red y procesos, y lo convirtió en un programa que se ejecuta desde la terminal.

### El problema que resolvió Node.js

Los servidores web de la época solían atender cada conexión con un hilo o proceso propio. Mientras ese hilo esperaba a la base de datos o al disco, no hacía nada más, pero seguía ocupando memoria. Con miles de usuarios conectados, el servidor se quedaba sin recursos aunque casi todos los hilos estuvieran esperando.

Dahl notó que JavaScript ya estaba acostumbrado a otra cosa: en el navegador nunca se bloquea esperando; se registra una función y se continúa. Node.js aplica la misma idea al servidor:

```text
Modelo tradicional (un hilo por petición)     Node.js (un hilo + event loop)

petición 1 → hilo 1: espera BD...             petición 1 → pide la BD, sigue
petición 2 → hilo 2: espera disco...          petición 2 → pide el disco, sigue
petición 3 → hilo 3: espera red...            petición 3 → pide la red, sigue
                                              ...cuando cada respuesta llega,
                                                 el event loop ejecuta su función
```

Un solo hilo de JavaScript atiende muchas peticiones porque **nunca se queda esperando**. Este es el **event loop**, que estudiaremos con detalle en la clase 05. Es la diferencia más importante con el Python que escribimos en el curso anterior, donde `requests.get()` detenía el programa hasta recibir la respuesta.

**2010.** **Isaac Schlueter** publicó **npm**, el gestor de paquetes de Node.js, y apareció **Express**, el framework web minimalista que usaremos en la clase 09. npm creció hasta convertirse en el registro de paquetes más grande del mundo.

**2011–2014.** Node.js llegó a Windows y empresas como LinkedIn, PayPal y Netflix lo adoptaron. JavaScript se volvió un lenguaje para el servidor, la línea de comandos e incluso el escritorio: editores como VS Code están construidos con **Electron**, que combina Chrome y Node.js.

En el navegador surgieron frameworks como AngularJS (2010), React (2013) y Vue (2014), y en 2012 Microsoft presentó **TypeScript**, JavaScript con tipos estáticos que se compila a JavaScript normal.

---

## 7. 2015: ES2015, el JavaScript moderno

**ES2015**, también llamado **ES6**, fue la actualización más grande de la historia del lenguaje. Casi todo lo que hace que JavaScript se sienta moderno llegó aquí:

| Novedad de ES2015 | Clase del curso |
|---|---|
| `let` y `const` | 01 |
| Template literals `` `Hola ${nombre}` `` | 01 |
| Desestructuración, `Set` y `Map` | 02 y 03 |
| Arrow functions `(x) => x * 2` y parámetros por defecto | 04 |
| `Promise` | 05 |
| Módulos con `import` / `export` | 06 |
| `class` | — |

Desde entonces el comité **TC39** publica una versión **cada año** (ES2016, ES2017...) con cambios más pequeños. Ya no hay que esperar diez años por una mejora.

El mismo año Node.js resolvió una crisis interna. En 2014 parte de la comunidad, inconforme con el ritmo del proyecto, había creado una copia llamada **io.js**. En 2015 ambos proyectos se unieron bajo la **Node.js Foundation** (hoy **OpenJS Foundation**) y nació el modelo de versiones **LTS** (*Long Term Support*): versiones con soporte garantizado durante años, pensadas para producción.

### El mismo código, antes y después

Este fragmento sigue el estilo de 2012: `var`, funciones anónimas, concatenación de texto y callbacks anidados.

```javascript
var fs = require("fs");

fs.readFile("precios.json", "utf8", function (error, texto) {
  if (error) {
    console.log("Error: " + error.message);
    return;
  }
  var precios = JSON.parse(texto);
  var caros = [];
  for (var i = 0; i < precios.length; i++) {
    if (precios[i] > 1000) {
      caros.push(precios[i]);
    }
  }
  console.log("Productos caros: " + caros.length);
});
```

Este es el estilo que usaremos en el curso:

```javascript
import { readFile } from "node:fs/promises";

try {
  const precios = JSON.parse(await readFile("precios.json", "utf8"));
  const caros = precios.filter((precio) => precio > 1000);
  console.log(`Productos caros: ${caros.length}`);
} catch (error) {
  console.log(`Error: ${error.message}`);
}
```

No necesitas entender cada línea todavía. Fíjate en que el segundo se lee de arriba abajo, como un programa de Python, aunque sigue siendo asíncrono.

---

## 8. 2016–2023: madurez

**2017.** **ES2017** agregó `async` / `await`, y Node.js 8 lo incorporó en su versión LTS. Antes, el código asíncrono terminaba en el *callback hell*: funciones dentro de funciones dentro de funciones. Con `await` el código asíncrono se escribe casi como el síncrono.

**2018.** Ryan Dahl dio una charla titulada *10 cosas de las que me arrepiento de Node.js*: el sistema de módulos propio, `node_modules`, la falta de seguridad por defecto. Con esas ideas creó **Deno**, que publicó su versión 1.0 en 2020.

**2019.** Node.js habilitó los **módulos ES** (`import` / `export`) sin opciones experimentales. Durante años convivieron con el sistema original de Node.js, **CommonJS** (`require()`), y todavía conviven: en código de Node.js verás tanto `require()` como `import`. En este curso usaremos módulos ES.

**2020.** **ES2020** agregó `?.` (*optional chaining*) y `??` (*nullish coalescing*), que usaremos desde las clases 01 y 03.

**2022–2023.** Node.js empezó a incorporar lo que antes requería paquetes externos: `fetch` (el equivalente de `requests`, clase 07), un ejecutor de pruebas (`node:test`) y el modo `--watch`, que reinicia el programa al guardar. En 2023 apareció **Bun**, otro runtime enfocado en velocidad y compatible con la mayor parte del ecosistema de Node.js.

---

## 9. 2024–hoy: Node.js como lo usaremos

La tendencia de los últimos años es clara: **Node.js trae incorporado cada vez más de lo que antes se instalaba con npm**. Para este curso eso significa menos dependencias:

| Necesidad | Antes | Hoy en Node.js |
|---|---|---|
| Peticiones HTTP | Paquete `axios` o `node-fetch` | `fetch` incorporado |
| Variables de entorno desde `.env` | Paquete `dotenv` | Opción `--env-file` |
| Reiniciar al guardar | Paquete `nodemon` | Opción `--watch` |
| Pruebas | Paquetes `jest` o `mocha` | `node:test` |
| Base de datos SQLite | Paquetes `sqlite3` o `better-sqlite3` | `node:sqlite` (clase 10) |
| Ejecutar TypeScript | Compilar antes con `tsc` | Node.js quita los tipos y ejecuta el archivo `.ts` |

`node:sqlite` todavía no está marcado como estable en la documentación oficial, pero es suficiente para aprender y para proyectos pequeños. TypeScript queda fuera de este curso; en la clase 04 veremos cómo documentar tipos con JSDoc, que es un buen primer paso.

### Versiones de Node.js

Hasta 2026 Node.js publicaba dos versiones mayores al año. Las **pares** (20, 22, 24, 26) se convertían en **LTS** en octubre; las impares eran de prueba y duraban poco. Node.js 26, que pasó a LTS en octubre de 2026, es la última con ese esquema: a partir de Node.js 27 habrá **una versión mayor por año**, y todas llegarán a LTS.

| Tipo de versión | Para qué sirve |
|---|---|
| **LTS** | Proyectos reales y cursos. Recibe correcciones durante unos 30 meses. |
| **Current** | Probar las novedades. Puede cambiar o quedarse sin soporte pronto. |

Este curso se comprobó con **Node.js 24 LTS**. En la clase 01 lo instalaremos con un gestor de versiones, que permite tener varias versiones a la vez y cambiar entre ellas, como `pyenv` en Python.

### El ecosistema actual

```text
                         JavaScript / TypeScript
                                  │
     ┌───────────────┬────────────┼─────────────┬──────────────────┐
     ▼               ▼            ▼             ▼                  ▼
 Navegador        Node.js       Deno           Bun         Electron (VS Code,
 React, Vue,      servidores,   runtime        runtime     Slack, Discord)
 Svelte...        CLIs, APIs    seguro         rápido
                     │
                     ▼
                    npm (millones de paquetes)
```

Node.js sigue siendo el runtime más usado fuera del navegador y el que tiene el ecosistema más grande. Lo que aprendas en este curso se aplica casi sin cambios en Deno y Bun.

---

## 10. Python y JavaScript, lado a lado

| | Python | JavaScript / Node.js |
|---|---|---|
| Nace | 1991, Guido van Rossum | 1995, Brendan Eich (Node.js: 2009, Ryan Dahl) |
| Diseñado para | Lenguaje de propósito general, legible | Scripts pequeños dentro del navegador |
| Gran ruptura | Python 2 → 3 (2008), incompatible | ES5 → ES2015, compatible: lo viejo sigue funcionando |
| Quién decide el lenguaje | PEP y Steering Council | Comité TC39 de Ecma |
| Implementación principal | CPython | V8 (Chrome, Node.js, Deno) |
| Gestor de paquetes | pip y PyPI | npm y su registro |
| Modelo para esperar E/S | Bloqueante por defecto; `asyncio` opcional | Asíncrono por defecto: event loop |
| Dónde se ejecuta | Terminal, servidores, ciencia de datos | Navegador, servidores, CLIs, escritorio |

Python decidió romper la compatibilidad para limpiar el lenguaje. JavaScript no puede hacerlo: si una versión nueva rompiera el código viejo, se romperían millones de páginas web. Por eso aprenderás a elegir la forma moderna de cada cosa y a reconocer la antigua.

---

## Actividad 1: JavaScript ya está en tu computadora

Antes de instalar Node.js, comprueba que ya tienes un runtime de JavaScript: el navegador.

1. Abre Chrome, Edge, Firefox o Safari.
2. Abre las herramientas de desarrollo con `F12` o `Cmd + Opción + I` en macOS (en Safari, primero activa *Mostrar funciones para desarrolladores web* en Ajustes → Avanzado).
3. Entra a la pestaña **Console** y escribe una línea a la vez:

```javascript
2 + 3
"hola".toUpperCase()
typeof null
"5" == 5
"5" === 5
navigator.userAgent
document.title
```

4. Responde:
   - ¿Cuál de los resultados es el error histórico del que habla la sección 1?
   - ¿Qué diferencia hay entre `==` y `===` según lo que viste?
   - `navigator` y `document` pertenecen al navegador, no al lenguaje. ¿Crees que existirán en Node.js? Lo comprobaremos en la clase 01.

---

## Actividad 2: Ordena la historia

Sin mirar la línea de tiempo, ordena estos eventos del más antiguo al más reciente. Después compruébalo con la sección 3.

```text
A. Aparece async / await
B. Se publica el primer estándar ECMAScript
C. Ryan Dahl presenta Node.js
D. Google lanza Chrome con V8
E. Llega ES2015 con let, const y arrow functions
F. Netscape crea JavaScript
G. Node.js incorpora fetch
H. Aparece npm
```

Para cada pareja de eventos consecutivos, explica en una línea por qué el primero hizo posible el segundo. Ejemplo: *"D antes de C: Node.js se construyó sobre V8."*

---

## Actividad 3: ¿De qué época es este código?

Clasifica cada fragmento como **antiguo** (antes de 2015) o **moderno**. Indica qué palabra o símbolo te dio la pista y, para los antiguos, qué usarías hoy.

```javascript
// Fragmento 1
var total = 0;
for (var i = 0; i < precios.length; i++) {
  total = total + precios[i];
}
```

```javascript
// Fragmento 2
const total = precios.reduce((suma, precio) => suma + precio, 0);
```

```javascript
// Fragmento 3
const respuesta = await fetch(url);
const datos = await respuesta.json();
```

```javascript
// Fragmento 4
$.ajax({
  url: "/api/productos",
  success: function (datos) {
    console.log("Recibidos: " + datos.length);
  },
});
```

```javascript
// Fragmento 5
const nombre = usuario?.perfil?.nombre ?? "Anónimo";
```

```javascript
// Fragmento 6
var express = require("express");
```

Cuando encuentres una respuesta en Internet con `var`, `require` o `function () {}` por todas partes, probablemente fue escrita antes de 2015. Sigue funcionando, pero en este curso escribiremos la versión moderna.

---

## Mitos y confusiones comunes

1. **"JavaScript es una versión de Java."** No tienen relación; el nombre fue una estrategia de marketing de 1995.
2. **"ECMAScript y JavaScript son lenguajes distintos."** ECMAScript es el estándar; JavaScript es su implementación. ES2015, ES2020, etc., son versiones del estándar.
3. **"Node.js es un lenguaje" o "es un framework".** Es un runtime: ejecuta JavaScript fuera del navegador. Express sí es un framework.
4. **"JavaScript es lento."** Lo era antes de 2008. Los motores modernos como V8 compilan a código máquina mientras el programa se ejecuta.
5. **"Node.js tiene un solo hilo, así que solo atiende a un usuario a la vez."** Tiene un solo hilo para tu código, pero no se bloquea mientras espera; delega la espera y atiende a otros mientras tanto.
6. **"Si funciona en el navegador, funciona en Node.js."** El lenguaje es el mismo, pero `document`, `window` o `alert` solo existen en el navegador, y `fs` o `process` solo en Node.js.
7. **"Hay que aprender `var` porque aparece en todos lados."** Basta con reconocerla. Usa `const` y `let`.

---

## Resumen

| Año | Evento | Por qué importa en el curso |
|---|---|---|
| 1995 | Nace JavaScript | Origen de rarezas como `==` y `typeof null`. |
| 1997 | ECMAScript 1 | El lenguaje tiene un estándar común. |
| 2008 | V8 | El motor que ejecuta tu código en Node.js. |
| 2009 | Node.js | JavaScript fuera del navegador, con event loop. |
| 2010 | npm y Express | Gestor de paquetes (clase 06) y framework web (clase 09). |
| 2015 | ES2015 | La base del JavaScript moderno que usaremos. |
| 2017 | `async` / `await` | Asincronía legible (clase 05). |
| 2019 | Módulos ES en Node.js | `import` / `export` (clase 06). |
| 2022 | `fetch` en Node.js | Consumir APIs sin paquetes (clase 07). |
| 2024 | `node:sqlite` | Base de datos sin paquetes (clase 10). |
| 2025 | Node.js 24 LTS | La versión con la que se comprobó el curso. |

---

## Para profundizar (opcional)

- [Introducción a JavaScript en MDN](https://developer.mozilla.org/es/docs/Web/JavaScript/Guide/Introduction): la referencia más usada del lenguaje.
- [Calendario de versiones de Node.js](https://nodejs.org/en/about/previous-releases): qué versiones están en LTS hoy.
- *JavaScript: The First 20 Years*, de Allen Wirfs-Brock y Brendan Eich (2020): la historia completa contada por sus protagonistas, disponible gratis en la ACM Digital Library.
- La charla original de Ryan Dahl presentando Node.js en JSConf EU 2009 y la de 2018, *10 Things I Regret About Node.js*, están disponibles en YouTube.

---

[Índice del curso](README.md) · [Clase 01 →](clase_01_de_python_a_javascript.md)
