# Proyecto final: tres APIs integradas en la nube
## Guía de orientación y rúbrica

**Computación en la nube**

---

## Contexto

Durante el curso construimos, paso a paso, todas las piezas necesarias para desplegar una aplicación real en la nube: una API REST con arquitectura por capas, persistencia con Sequelize y PostgreSQL, contenedores con Docker, comunicación entre microservicios, máquinas virtuales, servicios administrados de base de datos, Azure Functions con Cosmos DB, y un pipeline de CI/CD con GitHub Actions.

El proyecto final no pide nada nuevo conceptualmente. Pide **ensamblar** todo lo anterior en un sistema coherente de tres APIs independientes que se comunican entre sí, más un módulo serverless de datos complementarios.

---

## Importante: este proyecto es sobre TU dominio, no sobre libros

En las guías del curso usamos un sistema de biblioteca (`libros`, `lectores`, `prestamos`) como ejemplo de referencia. Ese ejemplo existe para que entiendan el patrón — **no para que lo repliquen tal cual**.

Cada equipo debe construir el proyecto sobre **su propio dominio**: el mismo que ya usaron cuando levantaron sus tres APIs en `docker-compose` (por ejemplo `ventas` consultando a `productos` y `clientes`, o el dominio que hayan elegido desde el inicio del curso). La estructura es idéntica a la de biblioteca — solo cambian los nombres de las entidades y la lógica de negocio de la validación cruzada.

Esta guía explica cada paso usando biblioteca como ejemplo ilustrativo. Donde dice `api-libros`, ustedes leen el nombre de su primera API; donde dice `api-lectores`, su segunda API; donde dice `api-prestamos`, la API que integra a las otras dos.

| En esta guía (ejemplo) | En tu proyecto (tu dominio) |
|---|---|
| `api-libros` | Tu primera API independiente |
| `api-lectores` | Tu segunda API independiente |
| `api-prestamos` | Tu API que valida contra las otras dos antes de crear un registro |
| `isbn` | El identificador natural de tu primera entidad |
| `documento` | El identificador natural de tu segunda entidad |
| Reseñas de libros | El dato complementario que tenga sentido en tu dominio (ej: reseñas de productos, comentarios de servicio, calificaciones) |

Si tu dominio es `ventas`/`productos`/`clientes`, por ejemplo: `api-productos` y `api-clientes` van solas, y `api-ventas` valida que el producto y el cliente existen antes de registrar la venta — exactamente el mismo patrón que `api-prestamos` valida libro y lector.

---

## Arquitectura que debes construir

Usando biblioteca como ejemplo de la forma (adapta los nombres a tu dominio):

```
api-libros      (ya la tienes) → pipeline propio → ACR → ACI
                                                              ↕ PostgreSQL administrado
                                                                schema "libros"
api-lectores    (nueva)        → pipeline propio → ACR → ACI
                                                              ↕ PostgreSQL administrado
                                                                schema "lectores"
api-prestamos   (nueva)        → pipeline propio → ACR → ACI
     ↕ HTTP a api-libros y api-lectores                     ↕ PostgreSQL administrado
                                                                schema "prestamos"

Función de reseñas (Azure Functions + Cosmos DB)
     ↕ HTTP a api-libros (valida que el ISBN existe antes de guardar)
```

Las tres APIs y la Función quedan desplegadas de forma independiente, cada una con su propio pipeline, pero conectadas entre sí por HTTP y compartiendo un mismo servidor PostgreSQL administrado con schemas separados.

---

## Componente 1: tu primera API (ya la tienes)

Ya la construiste y le hiciste el pipeline en la guía de GitHub Actions con persistencia PostgreSQL administrada. Para el proyecto:

- Verifica que el pipeline sigue desplegando correctamente
- Confirma que la base de datos usa un schema propio (no `public`) para que puedas compartir el mismo servidor con las otras dos APIs
- Anota la URL pública de ACI — la necesitarás para tu tercera API y para la Función de datos complementarios

---

## Componente 2: tu segunda API (nueva)

Construir siguiendo exactamente el mismo patrón de la primera:

- La entidad principal con un identificador natural que el cliente proporciona (como el `isbn` en libros, o el `documento` en lectores)
- Arquitectura por capas: `config`, `models`, `controllers`, `routes`, `app.js`, `index.js`
- Los 4 métodos HTTP con Sequelize
- Dockerfile idéntico en estructura al de tu primera API
- Su propio workflow de GitHub Actions, desplegando a su propio contenedor ACI
- Conectada al mismo servidor PostgreSQL administrado, pero en su propio schema

> Recuerda: cada schema en PostgreSQL necesita su propio `CREATE SCHEMA` y su propio `GRANT` — el mismo fix que aplicamos para el schema `public` aplica a cualquier schema nuevo.

---

## Componente 3: tu tercera API — la que integra a las otras dos (la más compleja)

Esta API no solo persiste datos — **valida contra las otras dos antes de guardar**.

### Qué debe hacer (ejemplo con biblioteca)

Un préstamo relaciona un libro (`isbn`) con un lector (`documento`). Antes de crear un préstamo, `api-prestamos` debe:

1. Llamar por HTTP a `api-libros` para confirmar que el libro existe
2. Llamar por HTTP a `api-lectores` para confirmar que el lector existe
3. Si ambos existen, crear el registro en su propio schema
4. Si alguno no existe, responder con un error claro indicando cuál falló

En tu dominio, esta misma lógica aplica con tus propias entidades: una venta valida que el producto y el cliente existen; una reserva valida que el vuelo y el pasajero existen; una matrícula valida que el curso y el estudiante existen. El patrón es idéntico — solo cambia qué se está validando.

### Cómo hacer las llamadas HTTP entre servicios

Cada servicio corre en su propio contenedor ACI con su propia URL pública. Usa esas URLs como variables de entorno:

```javascript
// Ejemplo con biblioteca — adapta los nombres a tu dominio
const respuestaLibro = await fetch(`${process.env.API_LIBROS_URL}/libros/${isbn}`);
if (respuestaLibro.status === 404) {
  return res.status(400).json({ error: `El libro con ISBN ${isbn} no existe.` });
}

const respuestaLector = await fetch(`${process.env.API_LECTORES_URL}/lectores/${documento}`);
if (respuestaLector.status === 404) {
  return res.status(400).json({ error: `El lector con documento ${documento} no existe.` });
}

// Ambos existen, crear el registro
const nuevoPrestamo = await Prestamo.create({ isbn, documento, fechaPrestamo: new Date() });
```

Las URLs se configuran como variables de entorno en el despliegue de ACI, igual que hiciste con `DB_HOST`.

### El resto de la API

- Su propio schema en PostgreSQL
- Su propio Dockerfile y workflow de GitHub Actions
- Al menos estos endpoints: crear registro (con la validación cruzada), listar todos, consultar los registros relacionados con una de las dos entidades

---

## Componente 4: Función de datos complementarios (Azure Functions + Cosmos DB)

Reutiliza la guía de Functions + Cosmos DB que ya construimos, adaptada a un dato complementario de tu dominio — algo con estructura variable que no encajaría bien en una tabla relacional fija (reseñas, comentarios, calificaciones, notas, etc.). Con un ajuste importante: **antes de guardar, la función debe validar que la entidad relacionada existe llamando a tu primera API.**

```javascript
// Ejemplo con biblioteca — adapta a tu dominio
module.exports = async function (context, req) {
    const resena = req.body;

    const respuestaLibro = await fetch(`${process.env.API_LIBROS_URL}/libros/${resena.isbn}`);
    if (respuestaLibro.status === 404) {
        context.res = { status: 400, body: `El libro con ISBN ${resena.isbn} no existe.` };
        return;
    }

    context.bindings.outputDocument = { ...resena, id: `${resena.isbn}-${Date.now()}` };
    context.res = { status: 201, body: context.bindings.outputDocument };
};
```

Las funciones de envío y consulta (por identificador principal, y por id de documento) de la guía anterior se mantienen; solo se agrega la validación cruzada en la función de envío.

---

## Entregables

| # | Entregable |
|---|---|
| 1 | Repositorio(s) con las tres APIs de tu dominio, cada una con su Dockerfile y workflow de GitHub Actions |
| 2 | Las tres APIs desplegadas y respondiendo en sus URLs de ACI |
| 3 | Un solo servidor PostgreSQL administrado con los tres schemas correspondientes a tus tres entidades |
| 4 | La Función de datos complementarios desplegada, conectada a Cosmos DB y validando contra tu primera API |
| 5 | Colección de Postman con todos los endpoints de las tres APIs y las funciones |
| 6 | Guía técnica en `.md` documentando el despliegue completo, al estilo de las guías del curso |
| 7 | Demo en vivo: un cambio de código en cualquiera de las APIs, push, y ver el pipeline redesplegar automáticamente |

---

## Rúbrica de evaluación (50 puntos)

| Criterio | Qué se evalúa | Puntos |
|---|---|---|
| **1. Segunda API funcional** | Arquitectura por capas, persistencia real en PostgreSQL, los 4 métodos HTTP funcionando | 8 |
| **2. Tercera API y validación cruzada** | Crea registros solo si las dos entidades relacionadas existen; responde con errores claros cuando no; persiste en su propio schema | 12 |
| **3. Tres pipelines de CI/CD** | Cada API tiene su workflow de GitHub Actions funcionando: build, push a ACR, despliegue a ACI | 10 |
| **4. Función de datos complementarios con validación** | La función valida la entidad relacionada contra tu primera API antes de guardar en Cosmos DB; los triggers de envío y consulta funcionan | 10 |
| **5. Documentación y colección de Postman** | Guía técnica completa y reproducible; colección de Postman organizada con todos los endpoints | 5 |
| **6. Demo en vivo** | El grupo hace un cambio de código real durante la presentación y muestra el pipeline redesplegando automáticamente | 5 |
| **Total** | | **50** |

### Bono: cliente que consume las APIs (+5 puntos adicionales)

Construir una interfaz simple (puede ser una página HTML con JavaScript, una app de consola, o cualquier cliente que prefieran) que consuma las tres APIs de tu dominio para mostrar en pantalla el flujo completo: listar las dos entidades base y registrar un nuevo elemento en la API que las integra. No necesita ser una aplicación con diseño elaborado — el objetivo es demostrar que las APIs son consumibles por un cliente real, más allá de Postman.

### Bono: Timer trigger adicional (+3 puntos adicionales)

Agregar una función con **Timer trigger** que corra periódicamente y haga algo útil sobre los datos del proyecto — por ejemplo, un resumen o ranking calculado a partir de la información guardada en Cosmos DB o en las APIs, similar al `ResumenSemanal` que vimos en la guía de Functions. No es necesario que se ejecute en un horario real de producción; puede configurarse con una expresión NCRONTAB corta (cada 2 minutos) para demostrarlo durante la presentación.

---

## Recordatorios importantes

- **Elimina o detén los recursos de Azure** al finalizar cada sesión de trabajo para no agotar los créditos de Azure for Students
- Cada API es un servicio **independiente**: no deben compartir código entre sí, solo comunicarse por HTTP
- El servidor PostgreSQL administrado es compartido, pero cada schema pertenece a una sola API — ninguna API debe tocar el schema de otra directamente
- La comunicación entre servicios siempre es por HTTP a través de sus URLs públicas, nunca accediendo directamente a la base de datos de otro servicio
- El dominio es libre, pero la estructura (arquitectura por capas, esquemas separados, validación cruzada, pipeline por API) es obligatoria para todos los equipos

---

*Computación en la nube · Proyecto final*
