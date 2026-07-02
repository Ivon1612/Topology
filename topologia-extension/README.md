# Topología de Red — Tableau Dashboard Extension

Extensión propia (gratis, sin licencias de terceros) para visualizar topología
de red / árbol jerárquico dentro de un dashboard de Tableau, usando datos
reales del worksheet vía la Extensions API + D3.

## Contenido

```
topologia-extension/
├── index.html              <- la extensión (UI + lógica D3 + Extensions API)
├── manifest.trex            <- manifiesto que importas en Tableau
├── lib/
│   └── tableau.extensions.1.latest.js   <- librería oficial de Tableau (ya incluida)
└── README.md
```

## 1. Publicar index.html en una URL pública con HTTPS

Necesitas que `index.html` (con su carpeta `lib/`) sea accesible por una URL
HTTPS. La forma más simple y gratis: **GitHub Pages**.

```bash
# dentro de la carpeta topologia-extension/
git init
git add .
git commit -m "extension topologia inicial"
git branch -M main
git remote add origin https://github.com/TU-USUARIO/topologia-extension.git
git push -u origin main
```

Luego, en GitHub: Settings → Pages → Source: rama `main`, carpeta `/ (root)`.
Tu URL quedará algo como:

```
https://TU-USUARIO.github.io/topologia-extension/index.html
```

## 2. Editar manifest.trex

Abre `manifest.trex` y reemplaza:

```xml
<url>https://tu-usuario.github.io/topologia-extension/index.html</url>
```

con tu URL real de GitHub Pages del paso 1. También puedes ajustar
`<author>` con tu nombre/correo.

## 3. Agregar la extensión a la safe list en Tableau Cloud

Como tienes rol de admin:

1. Tableau Cloud → **Settings → Extensions**.
2. En "Network-enabled extensions", agrega tu URL (la misma del paso 1,
   o un wildcard como `https://TU-USUARIO.github.io/.*`).
3. Permite acceso a "Summary Data" (no necesitas Full Data Access para esto).

## 4. Agregar la extensión al dashboard

1. En Tableau Desktop o Web Authoring, arrastra el objeto **Extension** al dashboard.
2. Click en **"Access Local Extensions"**.
3. Selecciona el archivo `manifest.trex`.
4. Acepta el permiso de acceso a datos cuando te lo pida.

## 5. Ajustar los nombres de campo

Dentro de `index.html`, al inicio del `<script>`, hay un bloque `CONFIG`:

```javascript
const CONFIG = {
  campoNodo: "Nodo",
  campoNodoPadre: "NodoPadre"
};
```

Cambia `"Nodo"` y `"NodoPadre"` por los nombres reales de tus columnas
(el campo que identifica cada nodo, y el campo que indica su nodo padre;
vacío o null = nodo raíz).

## 6. Reemplazar el dibujo por tu D3 de topología existente

La función `dibujarArbol(filas)` en `index.html` es un árbol básico con
`d3.tree()` como punto de partida funcional. Si ya tienes tu propio código
D3/SVG para la topología (el que usabas en el iframe + URL Actions), puedes
pegarlo ahí mismo — `filas` ya te llega como un array de objetos
`{ id, parentId }` con los datos reales del worksheet.

Si tu topología es una **red general** (no estrictamente jerárquica, con
ciclos o múltiples padres), usa `d3-force` en vez de `d3.tree()`:

```javascript
const simulation = d3.forceSimulation(nodos)
  .force("link", d3.forceLink(enlaces).id(d => d.id))
  .force("charge", d3.forceManyBody())
  .force("center", d3.forceCenter(width / 2, height / 2));
```

## Probar localmente antes de subir a GitHub Pages (opcional)

```bash
cd topologia-extension
python3 -m http.server 8765
```

Y en tu `manifest.trex` (mientras pruebas) usa:
```xml
<url>http://localhost:8765/index.html</url>
```

Nota: para pruebas locales http (no https), Tableau Desktop sí lo permite;
Tableau Cloud requiere HTTPS siempre, así que el localhost solo te sirve
para iterar antes de publicar en GitHub Pages.
