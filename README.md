# Horas Extras de Enfermería — V3 conector (desarrollo)

Esta entrega adapta la pantalla para GitHub Pages y agrega un Worker de Cloudflare como intermediario. **No es una versión lista para producción.** Hereda limitaciones funcionales del motor V2 y requiere pruebas de seguridad y funcionamiento.

## Archivos
- `index.html`: pantalla para GitHub Pages. Requiere pegar el Client ID OAuth real localmente.
- `worker.js`: código para Cloudflare Worker.
- `Code.gs`: Apps Script con endpoint `doPost` protegido por secreto compartido.
- `Config.gs`: plantilla; conservá el Client ID real que ya está en tu proyecto, no lo reemplaces por el placeholder.

## Diseño de seguridad
1. GitHub Pages aloja el frontend público; no se guardan datos personales ni secretos allí.
2. Google Identity Services entrega un ID token al navegador.
3. Cloudflare Worker valida el token con Google y verifica audiencia, emisor, vencimiento y correo verificado.
4. Worker envía email verificado, acción y argumentos a Apps Script junto con un secreto compartido.
5. Apps Script compara el secreto con `Script Properties` antes de ejecutar una acción permitida y verifica el email contra `Personal`/`Administradores`.

El secreto compartido debe ser aleatorio, largo (al menos 32 bytes) y nunca incluirse en GitHub ni en el HTML. La URL del Worker y el Client ID no son secretos.

## Configuración que falta
- `GOOGLE_CLIENT_ID` en Cloudflare Worker Variables (texto).
- `APPS_SCRIPT_URL` en Worker Variables: URL `/exec` de la implementación de Apps Script.
- En Apps Script, ejecutar una vez `initializeApiSecret()` y copiar el valor que aparece en el registro de ejecución. Pegarlo como `APP_SHARED_SECRET` en Cloudflare Worker > Settings > Variables and Secrets > Secrets. El valor queda guardado en Script Properties automáticamente; no pegarlo en GitHub ni compartirlo por chat.
- En `index.html`, sustituir `PEGAR_CLIENT_ID_AQUI` por el Client ID real.
- En `worker.js`, cambiar `https://gacel86.github.io` si el origen real de GitHub Pages difiere.

## Precaución sobre Apps Script ContentService
Apps Script puede responder con una redirección a `script.googleusercontent.com`. El Worker maneja una redirección GET para recuperar la respuesta. Debe verificarse en el despliegue real; si Google cambia el comportamiento, hay que adaptar el transporte.

## Limitaciones funcionales heredadas de V2
- Interfaz MVP: no tiene calendario completo, filtros por periodo ni edición integral.
- Estadísticas y CSV son acumulados, no por quincena/mes.
- Disponibilidad avanzada no está implementada.
- El correo de lista de espera es un aviso, no un enlace de aceptación autenticado.
- Digest de 10 minutos se procesa mediante trigger cada 5 minutos, por lo que no es exacto.
- El motor requiere pruebas y correcciones de concurrencia, ofertas de espera, desbloqueo 75%, cancelaciones y turnos que cruzan medianoche.
- No cargar datos reales ni invitar al personal hasta completar pruebas.

## Importante
No subas `Code.gs`, `Config.gs`, archivos de datos, Client ID si querés mantenerlo privado, ni secretos a un repositorio público. Para GitHub Pages solo debe publicarse `index.html` (con Client ID público) y recursos estáticos que no contengan secretos. El Worker se edita desde Cloudflare.
