# Taller 2 — PaaS con Azure App Service
## Guía técnica: desarrollar y desplegar una app Node.js en la nube

**Computación en la nube**

---

## ¿Qué vas a hacer?

Vas a construir una aplicación web con **Node.js** y desplegarla en **Azure App Service** usando la interfaz de despliegue de Kudu. A diferencia de un archivo HTML estático, esta app tiene un servidor propio escrito en JavaScript. Al subirla, Azure detecta automáticamente que es un proyecto Node.js, instala las dependencias y la pone en marcha. Podrás ver ese proceso completo en los logs de construcción.

Al finalizar, tu app estará disponible en:

```
https://[nombre-de-tu-app].azurewebsites.net
```

---

## Requisitos previos

- Cuenta activa en **Azure for Students** → [portal.azure.com](https://portal.azure.com)
- Un editor de texto (VS Code recomendado)
- Navegador web actualizado (Chrome o Edge)

---

## Parte 1 — Crear la aplicación Node.js

### 1.1 Estructura del proyecto

Crea una carpeta llamada `mi-app-node`. Dentro vas a crear tres archivos:

```
mi-app-node/
├── package.json
├── server.js
└── index.html
```

### 1.2 Archivo `package.json`

Este archivo le dice a Azure que el proyecto es Node.js. Oryx, el sistema de build de Azure, lo detecta automáticamente y sabe que debe ejecutar `npm install` antes de iniciar la app.

Crea `package.json` con este contenido:

```json
{
  "name": "mi-app-azure",
  "version": "1.0.0",
  "description": "App web desplegada en Azure App Service con Node.js",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "engines": {
    "node": ">=18.0.0"
  }
}
```

> **¿Por qué es importante este archivo?**  
> Sin `package.json`, Azure no puede identificar que es un proyecto Node.js y el build falla. Con él, Oryx detecta la plataforma, instala dependencias con `npm install` y luego inicia la app con el comando definido en `"start"`.

### 1.3 Archivo `server.js`

Este es el servidor web. Usa el módulo `http` nativo de Node.js — no necesita instalar ninguna dependencia externa.

```javascript
const http = require('http');
const fs   = require('fs');
const path = require('path');

// Azure define el puerto a través de la variable de entorno PORT.
// Si no está definida (por ejemplo al probar en local), usa el 3000.
const PORT = process.env.PORT || 3000;

const server = http.createServer((req, res) => {
  const filePath = path.join(__dirname, 'index.html');

  fs.readFile(filePath, (err, content) => {
    if (err) {
      res.writeHead(500);
      res.end('Error al cargar la página');
      return;
    }
    res.writeHead(200, { 'Content-Type': 'text/html; charset=utf-8' });
    res.end(content);
  });
});

server.listen(PORT, () => {
  console.log(`Servidor corriendo en el puerto ${PORT}`);
});
```

> **¿Por qué `process.env.PORT`?**  
> Azure App Service asigna el puerto de forma dinámica a través de la variable de entorno `PORT`. Si el servidor escucha en un puerto fijo (como `3000`), Azure no podrá redirigir el tráfico hacia él y la app no responderá. Usar `process.env.PORT || 3000` garantiza que funcione tanto en Azure como en tu equipo local.

### 1.4 Archivo `index.html`

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Mi app Node.js en Azure</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: 'Segoe UI', Arial, sans-serif;
      background: linear-gradient(135deg, #0078d4, #1F4E79);
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
    }

    .card {
      background: #fff;
      border-radius: 12px;
      padding: 48px 40px;
      max-width: 560px;
      width: 90%;
      box-shadow: 0 8px 32px rgba(0,0,0,0.18);
      text-align: center;
    }

    .badge {
      display: inline-block;
      background: #e6f1fb;
      color: #0078d4;
      font-size: 13px;
      font-weight: 600;
      padding: 4px 14px;
      border-radius: 20px;
      margin-bottom: 20px;
    }

    h1 {
      font-size: 26px;
      color: #1F4E79;
      margin-bottom: 12px;
    }

    p {
      font-size: 15px;
      color: #555;
      line-height: 1.7;
      margin-bottom: 24px;
    }

    .info-grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 12px;
      margin-bottom: 28px;
    }

    .info-item {
      background: #f5f9ff;
      border-radius: 8px;
      padding: 14px 10px;
    }

    .info-label {
      font-size: 11px;
      color: #888;
      text-transform: uppercase;
      letter-spacing: 0.06em;
      margin-bottom: 4px;
    }

    .info-value {
      font-size: 14px;
      font-weight: 600;
      color: #1F4E79;
    }

    .status {
      display: inline-flex;
      align-items: center;
      gap: 7px;
      background: #eaf3de;
      color: #3b6d11;
      font-size: 13px;
      font-weight: 600;
      padding: 8px 18px;
      border-radius: 20px;
      margin-bottom: 20px;
    }

    .dot {
      width: 8px;
      height: 8px;
      border-radius: 50%;
      background: #3b6d11;
    }

    footer {
      font-size: 12px;
      color: #aaa;
      border-top: 1px solid #eee;
      padding-top: 16px;
      margin-top: 8px;
    }
  </style>
</head>
<body>
  <div class="card">
    <div class="badge">☁ Azure App Service — PaaS · Node.js</div>

    <h1>¡Mi primera app Node.js en la nube!</h1>

    <p>
      Esta página es servida por un servidor <strong>Node.js</strong>
      corriendo en <strong>Azure App Service</strong>.
      Azure detectó el runtime, instaló las dependencias y lo puso en marcha
      automáticamente.
    </p>

    <div class="info-grid">
      <div class="info-item">
        <div class="info-label">Modelo de servicio</div>
        <div class="info-value">PaaS</div>
      </div>
      <div class="info-item">
        <div class="info-label">Runtime</div>
        <div class="info-value">Node.js</div>
      </div>
      <div class="info-item">
        <div class="info-label">SO gestionado por</div>
        <div class="info-value">Microsoft Azure</div>
      </div>
      <div class="info-item">
        <div class="info-label">Servidor escrito por</div>
        <div class="info-value">El estudiante</div>
      </div>
    </div>

    <div class="status">
      <span class="dot"></span>
      App en línea y funcionando
    </div>

    <p id="timestamp" style="font-size:13px; color:#888; margin-bottom:0;"></p>

    <footer>
      Computación en la nube · Taller 2 — PaaS con Azure App Service
    </footer>
  </div>

  <script>
    const now = new Date();
    document.getElementById('timestamp').textContent =
      'Cargada el ' +
      now.toLocaleDateString('es-CO', {
        day: '2-digit', month: 'long', year: 'numeric'
      }) +
      ' a las ' +
      now.toLocaleTimeString('es-CO', {
        hour: '2-digit', minute: '2-digit'
      });
  </script>
</body>
</html>
```

### 1.5 Probar en local (opcional pero recomendado)

Si tienes Node.js instalado en tu equipo, puedes probar la app antes de subirla:

```bash
cd mi-app-node
node server.js
```

Abre el navegador en `http://localhost:3000`. Debes ver la misma página que aparecerá en Azure. Para detener el servidor presiona `Ctrl+C`.

> Si no tienes Node.js instalado localmente, no es problema — puedes desplegar directamente a Azure y verificarlo ahí.

### 1.6 Empaquetar en ZIP

Los tres archivos deben quedar en la **raíz** del ZIP, sin carpetas intermedias.

**En Windows:**
1. Entra a la carpeta `mi-app-node/`
2. Selecciona los tres archivos: `package.json`, `server.js` e `index.html`
3. Clic derecho → **Comprimir en archivo ZIP**
4. Nómbralo `app.zip`

**En Mac / Linux:**
```bash
cd mi-app-node
zip app.zip package.json server.js index.html
```

> ⚠️ **Verificación:** abre el `app.zip`. Al abrirlo debes ver los tres archivos directamente — `package.json`, `server.js` e `index.html` — sin ninguna carpeta intermedia. Si ves una carpeta `mi-app-node/` dentro del ZIP, el build de Azure fallará porque no encontrará el `package.json` en la raíz.

---

## Parte 2 — Crear el recurso en Azure

### 2.1 Acceder al portal

Abre [portal.azure.com](https://portal.azure.com) e inicia sesión con tu cuenta Azure for Students.

### 2.2 Crear un nuevo Web App

1. En la barra de búsqueda superior escribe **`Web App`**
2. Selecciona **App Services** en los resultados
3. Haz clic en **+ Crear** → **Aplicación web**

### 2.3 Configurar el recurso

| Campo | Valor |
|---|---|
| **Suscripción** | Azure for Students |
| **Grupo de recursos** | Crear nuevo → `rg-taller-paas` |
| **Nombre** | Un nombre único, por ejemplo `app-[tu-apellido]-node` |
| **Publicar** | Código |
| **Pila de runtime** | Node.js 20 LTS |
| **Sistema operativo** | Linux |
| **Región** | East US |
| **Plan de Linux** | Crear nuevo → `plan-taller-paas` |
| **Plan de precios** | **Free F1** |

> ⚠️ Verifica que el plan sea **Free F1** antes de crear el recurso para no generar costos en tu cuenta de Students.

### 2.4 Crear el recurso

1. Haz clic en **Revisar y crear**
2. Confirma que el costo estimado sea **$0/mes**
3. Haz clic en **Crear** y espera entre 1 y 3 minutos

### 2.5 Ir al recurso

Cuando aparezca **"La implementación se completó"**, haz clic en **Ir al recurso**. Anota la URL pública que aparece en la sección **Información general**:

```
https://[nombre-de-tu-app].azurewebsites.net
```

---

## Parte 3 — Desplegar con el nuevo Kudu

### 3.1 Abrir Kudu

1. En el menú lateral de tu App Service, ve a **"Herramientas de desarrollo"**
2. Haz clic en **"Herramientas avanzadas"**
3. Haz clic en **"Ir →"**

Se abre Kudu en una nueva pestaña. Si aparece la interfaz clásica, haz clic en **"Go to new UI"** en el banner superior.

### 3.2 Subir el ZIP

1. En el menú lateral de Kudu, haz clic en **"Deployments"** (con el badge PREVIEW)
2. Haz clic en **"+ New Deployment"**
3. Arrastra tu `app.zip` a la zona de carga o usa el botón para buscarlo
4. Haz clic en **"Deploy"**

### 3.3 Observar el proceso de build

Aquí es donde ocurre la diferencia respecto a un HTML estático. Verás tres etapas en la pantalla:

**Uploading → Building → Deploying**

Durante la etapa **Building**, el sistema Oryx de Azure analiza el ZIP y ejecuta el build automáticamente. En el panel de **Deployment Logs** puedes seguir el proceso en tiempo real. Verás líneas como estas:

```
Detecting platforms...
Detected following platforms: nodejs
Installing Node.js version: 20.x
Running npm install...
Build completed successfully.
```

Esto confirma que Azure identificó el `package.json`, detectó Node.js como plataforma, instaló las dependencias y ejecutó el comando de inicio definido en `"scripts": { "start": "node server.js" }`.

> Si el log muestra **"Build completed successfully"** y la etapa **Deploying** pasa a verde, el despliegue fue exitoso.

### 3.4 Verificar la app

1. Abre la URL pública en una nueva pestaña:
   ```
   https://[nombre-de-tu-app].azurewebsites.net
   ```
2. Presiona **Ctrl+F5** para forzar recarga
3. Debe aparecer la tarjeta con el mensaje **"¡Mi primera app Node.js en la nube!"**

> La primera carga puede tardar entre 10 y 30 segundos en el plan Free porque el servidor se "despierta" tras inactividad. Las siguientes cargas son inmediatas.

---

## Parte 4 — Personalizar la app *(opcional)*

### 4.1 Cambiar el título en `index.html`

```html
<h1>App de [tu nombre] — Node.js en Azure</h1>
```

### 4.2 Agregar una ruta en `server.js`

Puedes extender el servidor para responder de forma diferente según la ruta:

```javascript
const server = http.createServer((req, res) => {

  if (req.url === '/estado') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({
      estado: 'activo',
      plataforma: 'Azure App Service',
      modelo: 'PaaS',
      timestamp: new Date().toISOString()
    }));
    return;
  }

  // Ruta por defecto: sirve index.html
  const filePath = path.join(__dirname, 'index.html');
  fs.readFile(filePath, (err, content) => {
    if (err) { res.writeHead(500); res.end('Error'); return; }
    res.writeHead(200, { 'Content-Type': 'text/html; charset=utf-8' });
    res.end(content);
  });
});
```

Después de agregar esto, empaqueta nuevamente en ZIP, súbelo en Kudu y abre:

```
https://[nombre-de-tu-app].azurewebsites.net/estado
```

Verás la respuesta JSON del servidor directamente en el navegador.

### 4.3 Cambiar los colores del fondo en `index.html`

```css
/* Verde */
background: linear-gradient(135deg, #1a7a4a, #0d4a2d);

/* Naranja */
background: linear-gradient(135deg, #e67e22, #a93226);
```

Guarda, empaqueta en ZIP y vuelve a subir en Kudu.

---

## Solución de problemas frecuentes

| Problema | Causa | Solución |
|---|---|---|
| Build falla con "Couldn't detect nodejs" | El `package.json` no está en la raíz del ZIP — está dentro de una subcarpeta | Recrea el ZIP seleccionando los tres archivos directamente, no la carpeta |
| La app responde con error 503 | El servidor no escucha en `process.env.PORT` | Verifica que `server.js` use `process.env.PORT || 3000` y no un puerto fijo |
| La URL carga muy lento la primera vez | El plan Free F1 apaga la app tras inactividad | Normal en Free — espera hasta 30 segundos en la primera carga |
| El log muestra "Build completed" pero la app no abre | El servidor tardó en iniciar | Espera 1 minuto y recarga con Ctrl+F5 |
| Error al crear el grupo de recursos | El nombre ya existe en la suscripción | Usa un nombre diferente, por ejemplo `rg-taller-paas-2` |

---

## Referencia rápida — Azure CLI (opcional)

Desde Cloud Shell (`>_` en la barra superior del portal):

```bash
# Ver el estado de tu app
az webapp show \
  --name [nombre-de-tu-app] \
  --resource-group rg-taller-paas \
  --query state

# Ver los logs en tiempo real
az webapp log tail \
  --name [nombre-de-tu-app] \
  --resource-group rg-taller-paas

# Desplegar ZIP desde Cloud Shell
az webapp deploy \
  --resource-group rg-taller-paas \
  --name [nombre-de-tu-app] \
  --src-path app.zip \
  --type zip
```

---

*Computación en la nube · Taller 2 — PaaS con Azure App Service*
