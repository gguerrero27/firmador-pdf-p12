Firma PDF visible y digital

Proyecto para firmar PDFs con certificado .p12, mostrar firma visible (QR + texto) en una ubicación escogida por el usuario y generar el PDF firmado listo para descarga. Backend en Node.js/Express, frontend con PDF.js y un canvas para capturar la posición exacta de firma.

Librerías principales (y versión sugerida)

Node / NPM — Node.js 18+ (ej. 18.20.8 que usas).

express — Servidor HTTP y endpoints.

multer — Manejo de multipart/form-data y subida de archivos.

pdf-lib — Manipulación y edición de PDF (embed de imágenes, texto, rectángulos).

node-signpdf — Firma digital (inserta PKCS#7 / CMS en PDF). Depende de placeholder prep.

node-forge — Leer y parsear .p12, extraer certificado (CN), etc.

qrcode (npm qrcode) — Generar PNG/SVG Data URL del QR para embed en PDF.

pdfjs-dist (a.k.a. PDF.js) — Render en frontend para mostrar PDF en <canvas> y permitir selección por clic (versión usada: 3.11.174 en tu front).

(opcional) canvas — Si necesitas manipular imágenes en Node (p. ej. crear imágenes compuestas), pero en tu flujo embedes PNG generado por qrcode en pdf-lib, así que canvas no es obligatorio.

(dev/ops): nodemon para desarrollo, .gitignore, pm2 o systemd para producción si corres siempre.

Instalación ejemplo:

npm install express multer pdf-lib node-signpdf node-forge qrcode pdfjs-dist
# dev
npm install -D nodemon

Estructura de archivos (sugerida)
/project
  /public
    index.html
    app.js              # frontend (PDF.js viewer + selección)
    styles.css
  /utils
    signPdf.js          # lógica de añadir visible + placeholder + firmar (.p12)
    signImage.js (opt)  # si generas imágenes complejas (opcional)
  server.js             # express + multer + endpoint /sign
  package.json
  .gitignore
  README.md

Flujo completo (end-to-end) — paso a paso
1) Frontend — carga y renderizado del PDF

Usuario selecciona el PDF (<input type="file">).

PDF.js (pdfjsLib.getDocument) carga el archivo en memoria y renderiza la página actual en un <canvas> con un viewport a escala (ej. scale = 1.0 o configurable).

El canvas tiene ancho/alto internos (canvas.width/height) que representan la resolución real del PDF renderizado; el CSS puede escalar la vista en pantalla.
— Clave: para obtener coordenadas PDF exactas hay que convertir el clic desde display coordinates (px) a internal canvas coordinates usando scaleX = canvas.width / rect.width.

2) Frontend — selección de posición

Cuando el usuario hace clic en el canvas:

clickX_screen = e.clientX - rect.left

clickY_screen = e.clientY - rect.top

realX = clickX_screen * (canvas.width / rect.width)

realY_from_bottom = canvas.height - (clickY_screen * (canvas.height / rect.height))
(PDF usa origen abajo-izquierda; por eso invertimos Y)

Se muestra un marcador visual (div#marker) en la posición de la pantalla para feedback.

Se guarda { page, x: realX, y: realY_from_bottom } y se puede permitir varios marcadores (array) si se solicitan múltiples firmas.

3) Envío al servidor

Se crea FormData con:

pdf (archivo)

cert (.p12)

password (string)

x, y, page (coordenadas y página)

Se fetch('/sign', { method: 'POST', body: form }).

4) Server (Express + Multer)

multer guarda temporalmente PDF y .p12 en uploads/.

Se parsean x, y, page desde req.body y rutas de archivos desde req.files.

Llamada a signPdfVisibleAndDigital({ pdfPath, certPath, password, x, y, page, options }).

5) signPdfVisibleAndDigital (utils/signPdf.js)

Función core que:

Lee pdfBytes y p12 buffer.

Extrae nombre del firmante (CN) con node-forge.

Genera QR con contenido (ej. bloque FirmaEC) usando qrcode.toDataURL(...).

Convierte DataURL a Buffer/Uint8Array y embedPng en pdf-lib.

Carga el PDF con PDFDocument.load(pdfBytes).

Selecciona la página pages[page-1].

Dibuja:

un rectángulo de fondo blanco (opcional, para legibilidad),

la imagen del QR en (x + padding, y + padding) (recuerda que pdf-lib usa origen abajo-izquierda),

texto (firmado por, fecha, validar con...) con embedFont(StandardFonts.Helvetica) y drawText.

dimensiones de la imagen y texto definidas (p. ej. QR 50×50 px).

NOTA: coord {x,y} que recibe la función debe estar en las unidades internas del PDF (puntos) — por eso el frontend convierte.

serializa PDF a buffer (pdfDoc.save({ useObjectStreams: false })).

Inserta placeholder para node-signpdf con plainAddPlaceholder({ pdfBuffer, reason, name, location }).

Firma con SignPdf().sign(pdfWithPlaceholder, p12Buffer, { passphrase: password }).

Devuelve Buffer firmado al servidor.

6) Server responde al cliente

res.setHeader('Content-Type','application/pdf')

res.setHeader('Content-Disposition','attachment; filename="original-signed.pdf"')

res.send(signedPdfBuffer)

Limpieza: fs.unlinkSync de archivos temporales.

Puntos críticos / Trucos

Coordenadas Y: PDF usa origen en esquina inferior izquierda; el canvas usa esquina superior izquierda -> debes invertir Y.

Escalado: siempre convertir screen px -> canvas internal px con scale = canvas.width / rect.width.

Placeholder: node-signpdf necesita un placeholder PKCS#7 vacío. plainAddPlaceholder te lo añade con el tamaño correcto.

PDF corrupto: usar useObjectStreams: false en pdf-lib.save() evita problemas de compatibilidad con node-signpdf.

QR embedding: qrcode.toDataURL() devuelve DataURL (base64). Convertir a Buffer.from(base64, 'base64') antes de embedPng.

Tamaños: ajusta qrWidth/qrHeight según prefieras; ofrece controles en frontend para cambiar tamaño visible.

Cross-platform: extraer os info es opcional y sensible — evita exponer datos innecesarios en QR público.

Seguridad y producción

No almacenes certificados .p12 ni PDFs sensibles sin cifrado en disco por mucho tiempo. Usa TTL y limpia inmediatamente.

Rate-limit y protecciones contra uploads maliciosos (size limit en multer).

Validaciones: validar page, x, y antes de usar.

HTTPS obligatorio en producción (no enviar certificados por HTTP claro).

Logs: no registrar contraseñas en texto plano.

Permisos: carpeta uploads/ con permisos seguros y borrado post-proceso.

Ejemplo de comandos útiles

Instalar dependencias:

npm init -y
npm install express multer pdf-lib node-signpdf node-forge qrcode pdfjs-dist
npm install -D nodemon


Entrar:

node server.js
# o en dev
npx nodemon server.js
