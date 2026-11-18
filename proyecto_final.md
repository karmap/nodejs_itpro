# Proyecto final: Generador de tarjetas de estudio con historial

Construye un cliente de consola que solicite mazos de tarjetas de estudio a una API y permita consultar los mazos guardados sin volver a llamar a la IA.

**Tecnologías:** Node.js, fetch, readline, Express, Zod, OpenRouter y SQLite con `node:sqlite`.

La estructura es paralela al proyecto final del curso de Python. El dominio cambia a propósito: no se trata de traducir línea por línea una solución anterior, sino de aplicar en Node.js las mismas decisiones de diseño.

## 1. Funcionamiento

El cliente debe ofrecer estas opciones:

- **Crear un mazo:** pedir tema, número de tarjetas y nivel; enviar la consulta y mostrar el mazo guardado con su ID.
- **Consultar el historial:** mostrar ID, tema, número de tarjetas y nivel de cada registro; indicar si está vacío.
- **Ver un mazo:** pedir un ID entero y mostrar su contenido completo.
- **Eliminar un mazo:** pedir un ID entero y mostrar la confirmación de eliminación.
- **Salir.**

El menú se repite después de cada operación o error recuperable. Los mensajes y resultados se presentan en español.

La API valida la entrada, consulta OpenRouter, valida el resultado, lo guarda en SQLite y responde al cliente. Cada creación exitosa genera un registro nuevo. El cliente accede al historial mediante la API; listar, consultar y eliminar registros no realiza llamadas a OpenRouter.

## 2. Datos de entrada

| Campo | Tipo | Regla |
|---|---|---|
| `topic` | Texto | Obligatorio, de 1 a 100 caracteres después de retirar espacios exteriores. |
| `count` | Entero | Obligatorio, entre 3 y 10. Es el número máximo de tarjetas solicitadas. |
| `level` | Texto | Obligatorio: `básico`, `intermedio` o `avanzado`. |

La API rechaza campos adicionales. `count` debe llegar como número JSON, no como texto. El cliente debe manejar entradas numéricas inválidas y opciones desconocidas sin terminar el programa.

## 3. Resultado de IA

Solicita tarjetas de pregunta y respuesta acordes con los tres datos de entrada. OpenRouter debe devolver un objeto JSON con:

| Campo | Tipo | Regla |
|---|---|---|
| `summary` | Texto | Resumen del tema, no vacío ni de solo espacios. |
| `cards` | Array de objetos | Entre 1 y `count` tarjetas. Cada una tiene exactamente `question` y `answer`, ambos textos no vacíos ni de solo espacios. |
| `study_minutes` | Entero | Tiempo de estudio sugerido, entre 5 y 180. |

Valida el resultado y rechaza campos adicionales, tanto en el objeto principal como en cada tarjeta, antes de guardarlo. Una consulta fallida o un resultado inválido no crea registros. Las tarjetas son material de repaso generado por IA: pueden contener errores y no sustituyen fuentes verificadas.

## 4. API

| Método y ruta | Respuesta exitosa |
|---|---|
| `GET /health` | 200: `{"status": "ok"}`. |
| `POST /decks` | 201: mazo completo con ID, después de guardarlo. |
| `GET /history` | 200: array con `id`, `topic`, `count` y `level`; `[]` si está vacío. |
| `GET /history/:id` | 200: mazo completo. |
| `DELETE /history/:id` | 200: `{"mensaje": "Mazo eliminado"}`, después de confirmar la eliminación. |

`POST /decks` recibe los tres campos de entrada. POST y GET por ID devuelven esta estructura; el ID real lo genera SQLite:

```json
{
  "id": 7,
  "topic": "Fotosíntesis",
  "count": 3,
  "level": "básico",
  "result": {
    "summary": "Proceso con el que las plantas transforman luz en energía química.",
    "cards": [
      {
        "question": "¿Qué gas absorben las plantas durante la fotosíntesis?",
        "answer": "Dióxido de carbono."
      },
      {
        "question": "¿En qué organelo ocurre la fotosíntesis?",
        "answer": "En el cloroplasto."
      },
      {
        "question": "¿Qué gas liberan las plantas como resultado?",
        "answer": "Oxígeno."
      }
    ],
    "study_minutes": 15
  }
}
```

`result` debe ser un objeto JSON, no una cadena con JSON escapado. El cliente muestra el resumen, las tarjetas numeradas y el tiempo sugerido de forma legible.

Los errores de la API usan un objeto JSON con la clave `detail`, como en las clases 09 y 10. Una ruta inexistente también responde con JSON.

## 5. Persistencia y errores

Guarda en `decks.db`, junto a `server.js`, una tabla con `id`, `topic`, `count`, `level` y `result_json`. El último campo contiene el resultado validado como texto JSON. Usa consultas parametrizadas; no construyas SQL concatenando datos del usuario. Los registros y las eliminaciones deben conservarse después de reiniciar la API.

| Situación | Código HTTP |
|---|---|
| Body que no es JSON válido | 400. |
| Entrada inválida o ID de ruta no numérico | 422. |
| ID entero inexistente | 404. |
| Error de conexión o HTTP de OpenRouter, contenido ausente o resultado inválido | 502, sin guardar un registro. |
| Tiempo de espera agotado al consultar OpenRouter | 504, sin guardar un registro. |
| Error inesperado del servidor | 500, con un mensaje genérico y sin exponer detalles internos. |

El cliente muestra mensajes comprensibles ante errores de la API, conexión, espera o respuesta y regresa al menú. Nunca anuncia éxito si falló el guardado o la eliminación.

Configura `OPENROUTER_API_KEY` únicamente en el entorno del servidor, por ejemplo en un archivo `.env` cargado con `--env-file`. Usa un tiempo de espera en todas las peticiones: 30 segundos para OpenRouter, 60 para crear un mazo desde el cliente y 10 para consultar o eliminar registros.

## 6. Archivos y ejecución

Organiza el proyecto en `client.js`, `server.js`, `schemas.js`, `ai.js` y `database.js`. Incluye `package.json` con `"type": "module"`, las dependencias fijadas y estos scripts:

| Script | Comando |
|---|---|
| `dev` | `node --watch --env-file-if-exists=.env server.js` |
| `start` | `node --env-file-if-exists=.env server.js` |
| `client` | `node client.js` |

Incluye también `.gitignore`, `.env.example` sin una clave real e instrucciones breves de ejecución. Excluye de Git `node_modules/`, la base de datos y los archivos con claves.

Instala las dependencias desde la carpeta del proyecto:

```bash
npm install
```

Copia `.env.example` como `.env`, escribe tu clave e inicia el servidor:

```bash
npm run dev
```

En otra terminal, desde la misma carpeta:

```bash
npm run client
```

Usa `http://localhost:8000` como URL base. Express no genera una página `/docs`: prueba las rutas con el cliente y con `curl`, por ejemplo:

```bash
curl -i -X POST http://localhost:8000/decks -H "Content-Type: application/json" -d '{"topic": "Fotosíntesis", "count": 3, "level": "básico"}'
```

## 7. Criterios de entrega

- Crear, listar, consultar por ID y eliminar mazos desde la consola.
- Recuperar un mazo después de reiniciar la API y comprobar que una eliminación también persiste.
- Consultar el historial cuando OpenRouter no esté disponible.
- Rechazar entradas y resultados inválidos sin crear registros adicionales, incluido un resultado con más tarjetas que `count`.
- Manejar IDs inexistentes, JSON inválido y errores de conexión sin cerrar el cliente.

Entrega el código completo, `package.json`, `package-lock.json` y las instrucciones para ejecutar y comprobar estas operaciones.

### Práctica opcional: modo repaso

Agrega al menú una opción **Repasar un mazo**: pide un ID, muestra cada pregunta, espera a que el usuario presione Enter y después muestra la respuesta. Al terminar, muestra cuántas tarjetas se repasaron. Esta opción usa `GET /history/:id` y no llama a OpenRouter.

---

[Índice del curso](README.md)
