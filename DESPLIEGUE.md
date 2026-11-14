# Desplegar OpenInbox en Cloudflare (Workers Builds)

Este repositorio ya está listo: el proyecto (Worker + app web + migraciones D1) está en la raíz y
Cloudflare puede compilarlo y desplegarlo en cada push a `main`.

## 1. Antes de empezar

- Cuenta de Cloudflare **gratis**.
- Un **dominio con los nameservers en Cloudflare** (necesario para Email Routing: recibir correo).
- Una cuenta de envío gratis (**Resend**, Brevo o Mailjet). Solo hace falta para **enviar**;
  recibir funciona sin proveedor.
- No necesitas configurar secretos ni variables: el asistente de setup genera la clave de cifrado.

## 2. Conectar el repositorio (una sola vez)

1. Entra en <https://dash.cloudflare.com/> → **Workers & Pages** → **Create**.
2. Pestaña **Workers** → **Import a repository** (Git) → autoriza la GitHub App de Cloudflare si
   te lo pide y elige el repo `zebhastyanangelo-svg/Gmail_menugran`, rama `main`.
3. Rellena:
   - **Project name**: `openinbox` (o el que quieras; define el nombre del Worker).
   - **Build command**: `npm run build`
   - **Deploy command**: `npx wrangler deploy`
   - **Root directory**: `/` (déjalo vacío si ya es la raíz).
   - Opcional (Settings → Build → Variables): `NODE_VERSION=24`.
4. Pulsa **Deploy**. La primera build tarda unos minutos. Wrangler **crea la base de datos D1
   `openinbox` automáticamente** y el Worker aplica sus migraciones al arrancar.
5. A partir de aquí, cada `git push` a `main` vuelve a compilar y desplegar solo.

> Si el primer deploy se queja de que falta `database_id`: crea la base en
> **Storage & Databases → D1 → Create** con el nombre `openinbox`, copia su UUID y pégalo en
> `wrangler.jsonc` (`"database_id": "..."`). Después vuelve a desplegar.

## 3. Terminar la configuración en la app

1. Abre la URL del Worker, `https://openinbox.TU-SUBDOMINIO.workers.dev`.
   Si no aparece, actívala en **Settings → Domains & Routes → workers.dev**.
2. La página de setup pide un **API token de Cloudflare**. Pulsa **Create token**: Cloudflare abre
   la pantalla con los permisos ya marcados; solo pulsa **Continue to summary → Create Token**,
   copia el token y pégalo.
3. Se crea tu **recovery key** y se muestra **una sola vez**. Guárdala en un gestor de contraseñas:
   es la única forma de recuperar la cuenta de propietario.
4. Crea tu cuenta de administrador y conecta tu dominio (el setup configura Email Routing, SPF,
   DKIM y DMARC por ti).
5. En **Admin → Integrations**, conecta Resend (o Brevo/Mailjet) con su API key para poder enviar.

## 4. Comprobar que funciona

- Envía un correo desde otra cuenta a tu dirección nueva.
- Míralo en la bandeja; si no llega, revisa **Admin → Domains → Check health** y
  **Admin → Mail flow**.
- Responde desde OpenInbox para probar el envío.

## 5. Actualizaciones

- Cambios propios: push a `main` → Cloudflare redespliega.
- Nueva versión del proyecto original: **Actions → Sync upstream → Run workflow** en GitHub, luego
  Cloudflare redespliega y el Worker aplica las migraciones nuevas.

## Límites del plan gratis

| Qué | Límite |
| --- | --- |
| Peticiones al Worker | 100.000 / día por cuenta |
| Almacenamiento D1 | 500 MB (≈50.000–100.000 mensajes) |
| Tamaño de correo entrante | 25 MB |
| Envío | Resend 100/día, Brevo 300/día, Mailjet 200/día |

Los adjuntos grandes solo se conservan con R2 activado (requiere método de pago en Cloudflare).

## Errores típicos

| Síntoma | Solución |
| --- | --- |
| La build falla por la versión de Node | Añade la variable `NODE_VERSION=24` en la build |
| No hay URL `workers.dev` | Settings → Domains & Routes → activar `workers.dev` |
| No llega correo | El dominio debe estar en la misma cuenta Cloudflare y con Email Routing activo (lo activa el setup) |
| "A repository with that name already exists" al usar el botón Deploy | No uses el botón además de Workers Builds: crearía una segunda instancia |
| No puedo enviar | Falta conectar un proveedor en Admin → Integrations, o la API key es incorrecta |
