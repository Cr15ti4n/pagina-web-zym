# Guía de Deploy — Grupo Z&M

Instrucciones para publicar el sitio web en un hosting tradicional con cPanel/FTP.

---

## Requisitos previos

1. Un dominio registrado (ej. `grupozym.com`)
2. Un plan de hosting con cPanel (Hostinger, GoDaddy, Namecheap, etc.)
3. Acceso a cPanel o credenciales FTP
4. Un cliente FTP como FileZilla (opcional, cPanel tiene su propio administrador de archivos)

---

## Estructura del proyecto

El sitio es una página estática (HTML + CSS + JS en un solo archivo). No requiere base de datos, PHP ni ningún backend.

```
├── index.html              ← Archivo principal (todo el sitio)
├── dossier-iza-2024.pdf    ← Dossier IZA
├── dossier-zca-2024.pdf    ← Dossier Z&CA
├── dossier-zynkt-2024.pdf  ← Dossier ZYNKT
├── img/                    ← Todas las imágenes (WebP)
│   ├── aeropuerto-y-servicios-especiales/
│   ├── alila-mayakoba-hyatt/
│   ├── cinvestav/
│   ├── coopel/
│   ├── elektra/
│   ├── fairmont/
│   ├── ford/
│   ├── harmony/
│   ├── liverpool/
│   ├── office-depot/
│   ├── petco-plaza-jolie/
│   ├── radioshack/
│   ├── sabadell/
│   ├── scotiabank/
│   ├── smartfit/
│   ├── swarovski/
│   └── ...
└── img-transicion/         ← Imágenes del slider hero
```

---

## Archivos que NO se deben subir

Estas carpetas y archivos son solo de desarrollo y no deben subirse al servidor:

- `_img-originales-respaldo/`
- `_img-no-usadas/`
- `_archivos-desarrollo/`
- `_pdf-originales-respaldo/`
- `.git/`
- `.gitignore`
- `.DS_Store`
- `README.md`
- `DEPLOY.md`

---

## Opción 1: Subir por cPanel (Administrador de Archivos)

### Paso 1 — Acceder a cPanel

1. Ir a `tudominio.com/cpanel` o al enlace que te proporcionó tu proveedor de hosting
2. Iniciar sesión con usuario y contraseña

### Paso 2 — Abrir el Administrador de Archivos

1. En cPanel buscar **Administrador de archivos** (File Manager)
2. Navegar a la carpeta `public_html/` — esta es la raíz de tu sitio web

### Paso 3 — Limpiar public_html

1. Si hay archivos por defecto (como `cgi-bin`, `index.html` de ejemplo), eliminarlos
2. Dejar la carpeta vacía

### Paso 4 — Subir los archivos

1. Clic en **Cargar** (Upload) en la barra superior
2. Comprimir primero todos los archivos del proyecto en un `.zip` (sin incluir las carpetas de desarrollo)
3. Subir el archivo `.zip`
4. Una vez subido, clic derecho sobre el `.zip` → **Extraer** (Extract)
5. Verificar que `index.html` quede directamente en `public_html/` (no dentro de una subcarpeta)

### Paso 5 — Verificar

1. Abrir `tudominio.com` en el navegador
2. Confirmar que la página carga correctamente

---

## Opción 2: Subir por FTP (FileZilla)

### Paso 1 — Obtener credenciales FTP

En cPanel ir a **Cuentas FTP** y anotar:

- **Servidor (Host):** `ftp.tudominio.com` o la IP del servidor
- **Usuario:** tu usuario FTP
- **Contraseña:** tu contraseña FTP
- **Puerto:** `21` (FTP) o `990` (FTPS)

### Paso 2 — Conectar con FileZilla

1. Descargar FileZilla desde `https://filezilla-project.org`
2. Abrir FileZilla
3. En la barra superior ingresar: Host, Usuario, Contraseña y Puerto
4. Clic en **Conexión rápida**

### Paso 3 — Subir archivos

1. En el panel izquierdo (local) navegar a la carpeta del proyecto en tu Mac: `~/Desktop/1. PAGINA WEB ZYM/`
2. En el panel derecho (servidor) navegar a `/public_html/`
3. Seleccionar los archivos y carpetas a subir (excluir los de desarrollo)
4. Arrastrar del panel izquierdo al derecho
5. Esperar a que se complete la transferencia

---

## Opción 3: Subir por Terminal (SCP/rsync)

```bash
# Usando rsync (recomendado, solo sube archivos nuevos o modificados)
rsync -avz --progress \
  --exclude='.git' \
  --exclude='.gitignore' \
  --exclude='.DS_Store' \
  --exclude='README.md' \
  --exclude='DEPLOY.md' \
  --exclude='_img-originales-respaldo' \
  --exclude='_img-no-usadas' \
  --exclude='_archivos-desarrollo' \
  --exclude='_pdf-originales-respaldo' \
  ~/Desktop/"1. PAGINA WEB ZYM"/ usuario@tudominio.com:/home/usuario/public_html/
```

---

## Configuración del dominio

Si el dominio aún no apunta al hosting:

1. Ir al panel de tu registrador de dominios (donde compraste el dominio)
2. Cambiar los **nameservers** (DNS) a los que te proporcionó tu hosting, por ejemplo:
   - `ns1.hosting.com`
   - `ns2.hosting.com`
3. Esperar la propagación DNS (puede tomar de 15 minutos a 48 horas)

---

## Configuración SSL (HTTPS)

Para que el sitio sea seguro con HTTPS:

1. En cPanel buscar **SSL/TLS** o **Let's Encrypt**
2. Generar un certificado gratuito con **Let's Encrypt** o **AutoSSL**
3. Verificar que el certificado se aplique al dominio
4. Activar la redirección de HTTP a HTTPS en cPanel o agregando un `.htaccess`:

```apache
# Archivo: .htaccess (crear en public_html/)
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
```

---

## Optimización del servidor

Agregar este archivo `.htaccess` en `public_html/` para mejorar el rendimiento:

```apache
# Compresión GZIP
<IfModule mod_deflate.c>
  AddOutputFilterByType DEFLATE text/html text/css application/javascript image/svg+xml
</IfModule>

# Caché del navegador
<IfModule mod_expires.c>
  ExpiresActive On
  ExpiresByType image/webp "access plus 1 year"
  ExpiresByType image/svg+xml "access plus 1 year"
  ExpiresByType text/css "access plus 1 month"
  ExpiresByType application/javascript "access plus 1 month"
  ExpiresByType application/pdf "access plus 1 month"
  ExpiresByType text/html "access plus 1 hour"
</IfModule>

# Redirección HTTP → HTTPS
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]

# Seguridad básica
Header set X-Content-Type-Options "nosniff"
Header set X-Frame-Options "SAMEORIGIN"
Header set X-XSS-Protection "1; mode=block"
```

---

## Actualizar el sitio

Cada vez que hagas cambios en los archivos locales:

```bash
# 1. Subir cambios a GitHub (respaldo)
cd ~/Desktop/"1. PAGINA WEB ZYM"
git add -A
git commit -m "descripción del cambio"
git push origin main

# 2. Subir al servidor (usando rsync)
rsync -avz --progress \
  --exclude='.git' \
  --exclude='.gitignore' \
  --exclude='.DS_Store' \
  --exclude='README.md' \
  --exclude='DEPLOY.md' \
  --exclude='_img-originales-respaldo' \
  --exclude='_img-no-usadas' \
  --exclude='_archivos-desarrollo' \
  --exclude='_pdf-originales-respaldo' \
  ~/Desktop/"1. PAGINA WEB ZYM"/ usuario@tudominio.com:/home/usuario/public_html/
```

---

## Contacto

Grupo Z&M — cristian.hernandez@zym.com.mx
