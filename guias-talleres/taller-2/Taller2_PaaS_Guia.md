# Taller 2 — PaaS con Azure App Service
## Guía técnica: desarrollar y desplegar una app web en la nube

**Computación en la nube**

---

## ¿Qué vas a hacer?

Vas a construir una aplicación web sencilla y desplegarla en **Azure App Service**, un servicio PaaS de Microsoft. Al finalizar, tu app estará disponible públicamente en una URL del tipo:

```
https://[nombre-de-tu-app].azurewebsites.net
```

No vas a instalar ningún servidor web. No vas a configurar un sistema operativo. Solo código y el portal de Azure.

---

## Requisitos previos

- Cuenta activa en **Azure for Students** → [portal.azure.com](https://portal.azure.com)
- Un editor de texto (VS Code, Notepad++ o cualquier editor de tu preferencia)
- Navegador web actualizado (Chrome o Edge recomendados)

---

## Parte 1 — Crear la aplicación web

### 1.1 Estructura del proyecto

Crea una carpeta en tu equipo llamada `mi-app-azure`. Dentro, crea el siguiente archivo:

```
mi-app-azure/
└── index.html
```

### 1.2 Código de la aplicación

Abre `index.html` con tu editor y pega el siguiente código completo:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Mi app en Azure App Service</title>
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
      max-width: 540px;
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
    <div class="badge">☁ Azure App Service — PaaS</div>

    <h1>¡Mi primera app desplegada en la nube!</h1>

    <p>
      Esta página se ejecuta en <strong>Azure App Service</strong>.
      No configuré ningún servidor: Azure gestiona el sistema operativo,
      el runtime y la infraestructura. Solo subí el código.
    </p>

    <div class="info-grid">
      <div class="info-item">
        <div class="info-label">Modelo de servicio</div>
        <div class="info-value">PaaS</div>
      </div>
      <div class="info-item">
        <div class="info-label">Plataforma</div>
        <div class="info-value">Azure App Service</div>
      </div>
      <div class="info-item">
        <div class="info-label">SO gestionado por</div>
        <div class="info-value">Microsoft Azure</div>
      </div>
      <div class="info-item">
        <div class="info-label">Código gestionado por</div>
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

> **¿Qué hace este código?**  
> Es una página HTML estática con estilos CSS incorporados. Muestra una tarjeta con información del modelo PaaS y la fecha/hora de carga generada con JavaScript. No necesita servidor de aplicaciones ni base de datos.

### 1.3 Probar en local antes de subir

1. Navega a la carpeta `mi-app-azure/` en tu explorador de archivos
2. Haz doble clic sobre `index.html`
3. Verifica que la página cargue correctamente en el navegador antes de continuar

### 1.4 Empaquetar en ZIP

Azure App Service recibe la aplicación como un archivo `.zip`. El archivo `index.html` debe quedar en la **raíz** del ZIP, no dentro de una subcarpeta.

**En Windows:**
1. Entra a la carpeta `mi-app-azure/`
2. Selecciona el archivo `index.html`
3. Clic derecho → **Comprimir en archivo ZIP**
4. Nómbralo `app.zip`

**En Mac / Linux:**
```bash
cd mi-app-azure
zip app.zip index.html
```

> ⚠️ **Verificación rápida:** abre el `app.zip` que creaste. Al abrirlo debes ver `index.html` directamente, sin carpetas intermedias. Si ves una carpeta `mi-app-azure/` dentro del ZIP, el despliegue fallará — en ese caso recrea el ZIP seleccionando solo el archivo, no la carpeta.

---

## Parte 2 — Crear el recurso en Azure

### 2.1 Acceder al portal

1. Abre [portal.azure.com](https://portal.azure.com) e inicia sesión con tu cuenta Azure for Students
2. Verifica que en la esquina superior derecha aparezca tu correo institucional

### 2.2 Crear un nuevo Web App

1. En la barra de búsqueda superior escribe **`Web App`**
2. Selecciona **App Services** en los resultados
3. Haz clic en **+ Crear** → **Aplicación web**

### 2.3 Configurar el recurso

Completa los campos con exactamente estos valores:

| Campo | Valor |
|---|---|
| **Suscripción** | Azure for Students |
| **Grupo de recursos** | Crear nuevo → `rg-taller-paas` |
| **Nombre** | Un nombre único, por ejemplo `app-[tu-apellido]-paas` |
| **Publicar** | Código |
| **Pila de runtime** | ASP.NET V4.8 |
| **Sistema operativo** | **Windows** |
| **Región** | Brasil South o Canada Central (O según disponibilidad) |
| **Plan de Windows** | Dejar por defecto |
| **Plan de precios** | **Free F1** |

> ⚠️ **Puntos críticos antes de crear:**
> - El Sistema operativo debe ser **Windows** — con Linux no estará disponible el menú de Zip Push Deploy en Kudu.
> - El plan debe ser **Free F1** para no generar costo en tu cuenta de Students.

### 2.4 Crear el recurso

1. Haz clic en **Revisar y crear**
2. Revisa que esté Gratis
3. Haz clic en **Crear**
4. Espera entre 1 y 3 minutos mientras Azure aprovisiona el recurso

### 2.5 Ir al recurso

Cuando aparezca **"La implementación se completó"**:

1. Haz clic en **Ir al recurso**
2. En la sección **Información general** verás la URL pública de tu app:
   ```
   https://[nombre-de-tu-app].azurewebsites.net
   ```
3. Puedes abrirla — por ahora mostrará la página predeterminada de Azure, lo cual es normal. Aún no has subido tu código.

---

## Parte 3 — Desplegar la aplicación con Kudu

### 3.1 Abrir Kudu desde el portal

1. En el menú lateral izquierdo de tu App Service, busca la sección **"Herramientas de desarrollo"**
2. Haz clic en **"Herramientas avanzadas"**
3. Haz clic en el botón **"Ir →"**

Se abrirá una nueva pestaña con la interfaz de Kudu. La URL tiene el formato:
```
https://[nombre-de-tu-app].scm.azurewebsites.net
```

### 3.2 Ir a Zip Push Deploy

En la barra de menú superior de Kudu:

1. Haz clic en **"Tools"**
2. Selecciona **"Zip Push Deploy"**

Verás una pantalla con una zona gris punteada que dice **"Drop zip file here to deploy"**.

### 3.3 Subir el archivo ZIP

1. Arrastra el archivo `app.zip` desde tu explorador de archivos y suéltalo sobre la zona
2. Aparecerá una barra de progreso mientras se sube
3. Cuando termine verás la lista de archivos desplegados — `index.html` debe aparecer ahí

### 3.4 Verificar el despliegue

1. Abre una nueva pestaña y escribe la URL pública de tu app:
   ```
   https://[nombre-de-tu-app].azurewebsites.net
   ```
2. Presiona **Ctrl+F5** para forzar recarga sin caché
3. Debe aparecer la tarjeta azul con el mensaje **"¡Mi primera app desplegada en la nube!"**

> Si después de 2 minutos sigue mostrando la página predeterminada de Azure, verifica en Kudu que `index.html` aparezca en la lista de archivos y que no haya quedado dentro de una carpeta dentro del ZIP.

---

## Parte 4 — Personalizar la app *(opcional)*

Si terminas antes que el resto del grupo, modifica la app para personalizarla.

### 4.1 Cambiar el título

En `index.html`, encuentra:

```html
<h1>¡Mi primera app desplegada en la nube!</h1>
```

Cámbialo por algo tuyo, por ejemplo:

```html
<h1>App de [tu nombre] — desplegada en Azure</h1>
```

### 4.2 Cambiar el nombre en el footer

```html
<footer>
  Computación en la nube · Taller 2 — [Nombre del equipo]
</footer>
```

### 4.3 Cambiar los colores del fondo

Encuentra esta línea en el CSS:

```css
background: linear-gradient(135deg, #0078d4, #1F4E79);
```

Algunos ejemplos de variación:

```css
/* Verde */
background: linear-gradient(135deg, #1a7a4a, #0d4a2d);

/* Naranja */
background: linear-gradient(135deg, #e67e22, #a93226);

/* Gris oscuro */
background: linear-gradient(135deg, #2c3e50, #1a252f);
```

Guarda el archivo, genera el ZIP nuevamente y vuelve a subirlo en Kudu. El cambio se refleja en segundos.

---

## Entregables

| # | Entregable |
|---|---|
| 1 | URL pública funcional de tu app |

---

## Solución de problemas frecuentes

| Problema | Causa | Solución |
|---|---|---|
| El menú "Tools → Zip Push Deploy" no aparece en Kudu | El App Service fue creado con **Linux** | Recrea el recurso seleccionando **Windows** como sistema operativo |
| La URL sigue mostrando la página predeterminada de Azure | El `index.html` quedó dentro de una carpeta en el ZIP | Abre el ZIP y verifica que `index.html` esté en la raíz; recrea el ZIP y vuelve a subir |
| Error 403 al abrir la URL | El despliegue aún se está propagando | Espera 2 minutos y recarga con Ctrl+F5 |
| El plan Free F1 no aparece | La región seleccionada no lo soporta | Cambia la región a East US o West Europe |
| Error al crear el grupo de recursos | El nombre ya existe en la suscripción | Usa un nombre distinto, por ejemplo `rg-taller-paas-2` |

---

## Referencia rápida — Azure CLI (opcional)

Si en algún momento prefieres usar la Azure CLI desde Cloud Shell (`>_` en la barra superior del portal):

```bash
# Ver tus App Services activos
az webapp list --output table

# Desplegar ZIP directamente
az webapp deploy \
  --resource-group rg-taller-paas \
  --name [nombre-de-tu-app] \
  --src-path app.zip \
  --type zip
```

---

*Computación en la nube · Taller 2 — PaaS con Azure App Service*
