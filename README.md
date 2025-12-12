📄 Firma PDF visible y digital — Descripción técnica para GitHub

Aplicación completa para firmar PDFs con certificados .p12, añadir una firma visible (QR + texto) en la posición seleccionada sobre el documento y generar un PDF firmado digitalmente (PKCS#7) listo para descarga.
Incluye frontend con visor PDF.js y backend Node.js/Express para procesar la firma.

🔧 Tecnologías y librerías usadas
Backend (Node.js 18+)

express → Servidor HTTP y endpoint /sign.

multer → Recepción de archivos PDF y .p12 como multipart/form-data.

pdf-lib → Edición del PDF: agregar QR, texto y elementos visibles.

node-signpdf → Firma digital CMS/PKCS#7 dentro del PDF.

node-forge → Lectura del certificado .p12, extracción de claves y CN.

qrcode → Generación de QR en base64 para insertar en el PDF.

Frontend

pdfjs-dist (PDF.js) → Render del PDF dentro de <canvas> para permitir seleccionar la posición exacta donde irá la firma.

JavaScript Vanilla → Cálculo de coordenadas reales (canvas interno vs. pantalla), control de páginas, envío del formulario.

DevTools

nodemon (opcional) → Recarga automática en desarrollo.

.gitignore → Exclusión de /uploads, certificados y artefactos.

🏗️ Arquitectura y flujo del proceso
1. Carga y visualización del PDF (Frontend)

PDF.js carga el documento y lo dibuja en un <canvas>.

Se normaliza el clic del usuario convirtiendo coordenadas pantalla → PDF (canvas interno).

Se guarda { page, x, y } para enviarlo al backend.

2. Envío al backend

Se envían:

archivo PDF

archivo .p12

contraseña del certificado

coordenadas x, y

número de página

Usando fetch() + FormData.

3. Procesamiento en el servidor

multer recibe y almacena temporalmente los archivos.

pdf-lib abre el PDF base.

Se genera QR con qrcode.

Se inserta QR y texto de firma visible en la página seleccionada.

Se añade placeholder PKCS#7 con node-signpdf.

Se firma usando el certificado .p12 + contraseña.

4. Respuesta al usuario

El servidor devuelve el PDF firmado listo para descarga.

El frontend genera un Blob y fuerza la descarga automática.
