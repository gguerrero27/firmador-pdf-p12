📄 Firma PDF Visible y Digital

Aplicación completa para firmar documentos PDF usando certificados .p12, agregando una firma visible (QR + texto) en la posición seleccionada por el usuario y generando un PDF firmado digitalmente (PKCS#7) listo para descarga.

Incluye un visor con PDF.js para navegar entre páginas y elegir exactamente dónde colocar la firma con un solo clic.

🚀 Características principales

✔️ Carga de PDF y certificado .p12.

✔️ Dibujo de firma visible (QR + texto) en la coordenada seleccionada.

✔️ Firma digital real en formato CMS / PKCS#7.

✔️ Visualizador completo del PDF con paginación.

✔️ Conversión precisa de click → coordenadas PDF.

✔️ Compatible con múltiples páginas.

✔️ Descarga automática del PDF firmado.

🧩 Tecnologías y librerías usadas
Backend (Node.js 18+)
Librería	Función
express	Servidor HTTP + endpoint /sign.
multer	Manejo de multipart/form-data y subida de archivos.
pdf-lib	Inserción de QR, texto y gráficos dentro del PDF.
node-signpdf	Generación de la firma digital PKCS#7 en PDF.
node-forge	Lectura y parseo del .p12 (certificado y clave).
qrcode	Generación de QR embebible como PNG base64.
Frontend
Librería	Función
pdfjs-dist (PDF.js)	Renderizado del PDF en <canvas> para seleccionar coordenadas.
DevTools

nodemon (opcional) — Recarga automática en desarrollo.

.gitignore — Excluye certificados y archivos temporales.

📁 Estructura del proyecto
/project
  /public
    index.html
    app.js
    styles.css
  /utils
    signPdf.js        # Lógica de firma visible + digital
  /uploads            # Archivos temporales (gitignore)
  server.js           # Express + Multer + endpoint /sign
  package.json
  .gitignore
  README.md

🔧 Instalación
npm install


Si necesitas instalarlas manualmente:

npm install express multer pdf-lib node-signpdf node-forge qrcode pdfjs-dist
npm install -D nodemon

▶️ Ejecución del proyecto
Modo normal
node server.js

Modo desarrollo
npx nodemon server.js


Servidor disponible en:

http://localhost:3000

📝 Uso

Cargar un PDF.

Cargar un certificado .p12.

Escribir la contraseña del .p12.

Navegar entre páginas del PDF.

Hacer clic donde se desea ubicar la firma visible.

Pulsar Firmar y descargar.

Se empieza la descarga del PDF firmado.

🔍 Explicación técnica del flujo
1. Frontend

Se carga el PDF usando pdfjsLib.getDocument.

Se renderiza la página actual en <canvas>.

Al hacer clic:

Se convierte la posición del cursor desde coordenadas de pantalla → coordenadas reales del PDF.

Se muestra un marcador.

Se envían al backend: x, y, page.

2. Backend

multer recibe PDF + .p12.

Se extrae certificado y clave usando node-forge.

Se genera un QR con qrcode.toDataURL.

Con pdf-lib:

Se inserta el QR.

Se dibuja texto (firmante, fecha, etc.).

Se genera un PDF intermedio.

node-signpdf inserta la firma digital real PKCS#7.

El servidor retorna el PDF final.

🔐 Seguridad

❗ El servidor borra automáticamente los archivos PDF y .p12 temporales.

Se recomienda usar HTTPS en producción.

No almacenar certificados de usuarios en disco.

Establecer límite de tamaño en uploads.

🧪 Pendientes / Mejoras opcionales

Permitir múltiples firmas visibles.

Ajuste de tamaño del recuadro de firma.

Soporte para varios firmantes.

Configuración avanzada del QR (color, logo, etc.).

Vista miniatura de todas las páginas.

📫 Autor
