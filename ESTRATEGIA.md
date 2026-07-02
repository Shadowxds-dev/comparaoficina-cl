# Estrategia: web de afiliados de Mercado Libre — Nicho "Oficina / Home Office" (Chile)

Basado en el vídeo analizado ("La Nueva Inteligencia Artificial Para Ganar Dinero"): una web de comparación de
productos que no vende nada propio, solo recomienda y enlaza a un marketplace con enlaces de afiliado,
monetizando por comisión. Adaptado a Chile usando el **Programa de Afiliados y Creadores de Mercado Libre** en
lugar de Amazon Asociados, y con arquitectura orientada a **tráfico orgánico (SEO)** en vez del generador
no-code del vídeo.

## 0. ¿Necesitas algo de Mercado Libre para que yo construya la web? No.

Construir el sitio (catálogo, fichas de producto, comparador, newsletter, SEO técnico) **no requiere ninguna
credencial ni aprobación de Mercado Libre** — es código que corre de forma independiente, con enlaces de
ejemplo listos para sustituir. Lo único que **no puedo hacer por ti** es lo que depende de tu identidad
personal:

- **Postular al Programa de Afiliados y Creadores de Mercado Libre** — pide RUT, cuenta personal de Mercado
  Libre y de Mercado Pago (para recibir los pagos). Es un registro atado a tu identidad/datos bancarios, así
  que debes hacerlo tú en [mercadolibre.cl/l/afiliados](https://www.mercadolibre.cl/l/afiliados).
- **Generar los enlaces de afiliado reales** una vez aprobado — se generan desde tu panel de afiliados sobre
  cada publicación real. Cuando los tengas, puedes pegármelos (o pegarlos tú mismo en `src/data/products.ts`,
  campo `affiliateUrl` de cada producto) y yo los integro.
- **Comprar el dominio, conectar el hosting y dar de alta Search Console** — puedo dejar todo listo para
  desplegar (`npm run build`), pero conectar tus cuentas de Vercel/GitHub, pagar el dominio y verificar la
  propiedad en Google Search Console son acciones sobre tus propias cuentas.

Todo lo demás — código, contenido, estructura, SEO técnico, textos — lo hago yo.

## 1. Por qué un clúster de nicho "oficina" y no un solo tipo de producto

Comparado con un comparador genérico tipo Solotodo (que cubre todo el catálogo de electro/hogar/tecnología a
escala), un sitio de nicho da más autoridad temática en Google y permite un comparador con specs realmente
relevantes. Pero limitarse a un solo producto (solo sillas) también se queda corto en inventario y ángulos de
contenido. El punto medio es un **clúster de intención de compra**: todo lo que alguien compra para armar un
puesto de trabajo — silla, escritorio, monitor y accesorios — sigue siendo un nicho coherente para SEO, pero
multiplica el catálogo y las oportunidades de contenido.

| Criterio | Sillas | Escritorios | Monitores | Accesorios |
|---|---|---|---|---|
| Comisión Mercado Libre | Mobiliario/hogar-decoración: **12%-24%** (venta directa) | Igual categoría que sillas | Electrónica/computación: **5%-10%** (más bajo, pero con tickets altos en gama premium) | Depende del tipo (electrónica o menaje/hogar) |
| Ticket medio | $55.000 – $400.000 CLP | $65.000 – $430.000 CLP | $80.000 – $400.000 CLP | $15.000 – $120.000 CLP |
| Intención de compra | Muy alta (dolor de espalda, teletrabajo) | Alta (setups nuevos, mudanzas) | Alta (upgrade de setup, gaming) | Media-alta (compra de impulso complementaria) |
| Rol en el sitio | Producto ancla, mayor tráfico de búsqueda | Ticket alto, buena comisión | Volumen de búsqueda muy alto | Ticket bajo pero facilita venta cruzada ("completa tu setup") |

**Importante:** las comisiones exactas por categoría cambian con el tiempo y Mercado Libre las revisa
periódicamente. Verifica el % vigente para cada categoría en tu panel de afiliados antes de proyectar ingresos.

## 2. Por qué se migró de React/Vite (SPA) a Astro (SSG) — el motivo técnico del SEO

La primera versión del sitio era una SPA (Single Page Application) 100% renderizada en el navegador. Funciona
bien como demo, pero es un techo real para el tráfico orgánico:

- El HTML inicial que recibe Google (y cualquier otro crawler) está prácticamente vacío; todo el contenido lo
  pinta JavaScript después. Google puede indexar esto, pero con una segunda pasada de renderizado que puede
  tardar días o semanas — mientras tanto, la página no aparece en resultados.
- No hay una URL/HTML distinto por categoría o producto — un buscador solo llega a ver "una" página.
- No hay meta tags por página (todas comparten el mismo `<title>`), ni datos estructurados, ni sitemap.

La solución fue migrar a **Astro** (SSG — Site Static Generation): cada página se genera como HTML real en el
momento del build, con su propio `<title>`, meta description y datos estructurados ya presentes en el HTML que
recibe el navegador o el crawler, sin depender de que se ejecute JavaScript primero. El comparador y el
formulario de newsletter siguen siendo interactivos (React "islands" hidratadas en el cliente), pero el
contenido en sí — textos, precios, specs, enlaces — está en el HTML desde el primer byte.

## 3. Lo que ya tienes construido

En `oficina-comparador/` (Astro + React + Tailwind) hay **40 páginas estáticas** generadas en el build
(`npm run build`), con **34 productos** en 4 categorías:

- **Home** (`/`) — categorías + productos más valorados + newsletter.
- **4 páginas de categoría** (`/sillas/`, `/escritorios/`, `/monitores/`, `/accesorios/`) — cada una con texto
  introductorio único (no genérico, para evitar contenido duplicado) y 3 preguntas frecuentes por categoría,
  pensadas para keywords de "mejor X para Y" y para aparecer en snippets de Google.
- **34 fichas de producto** (`/producto/:slug/`) generadas con `getStaticPaths()` — cada una con su propio
  título, meta description, ficha técnica, pros/contras, breadcrumbs y 3 productos relacionados (enlazado
  interno dentro de la misma categoría).
- **Comparador** (`/comparador/`) — isla interactiva de React: elige el tipo de producto y compara 2-4 modelos
  lado a lado. Acepta `?type=&add=` en la URL para llegar pre-seleccionado desde una ficha de producto.
- **Newsletter** — isla interactiva, lista para conectar a un proveedor real.
- Modelo de datos genérico en `src/data/products.ts`: cada producto tiene `type`, `tier` y un array `specs`
  flexible — añadir un producto nuevo se propaga automáticamente a home, categoría, ficha y comparador.

### SEO técnico ya resuelto

- **Meta tags únicos por página** (`title`, `description`, `canonical`, Open Graph, Twitter Card) vía
  `src/layouts/Layout.astro`.
- **Datos estructurados (JSON-LD)**: `Organization`+`WebSite` en todas las páginas, `Product` con
  `AggregateRating` y `Offer` en cada ficha, `BreadcrumbList` en categorías y fichas, `ItemList` en categorías,
  `FAQPage` en categorías (con preguntas reales, candidatas a aparecer en "la gente también pregunta").
- **Sitemap XML automático** (`@astrojs/sitemap`, se regenera en cada build) y **`robots.txt`** apuntando a él.
- **Enlazado interno**: breadcrumbs, "otras opciones en [categoría]" en cada ficha, y navegación por categoría
  en el header/footer — ayuda a que Google descubra y relacione todas las páginas entre sí.
- **Enlaces de afiliado marcados `rel="sponsored nofollow"`** — buena práctica exigida por Google para enlaces
  monetizados, evita que Google interprete el sitio como una granja de enlaces pagados.
- **Contenido único por categoría** (intro + FAQ escritos a mano, no plantilla repetida) para evitar el
  problema de "contenido fino/duplicado" que penaliza a muchos sitios de afiliados.

Para lanzarlo en local: `cd oficina-comparador && npm run dev`. Para generar el sitio estático:
`npm run build` (output en `dist/`).

## 4. Cómo pasar de demo a web real (pasos concretos)

1. **Postula al Programa de Afiliados y Creadores de Mercado Libre** (ver sección 0). El requisito de 10.000
   seguidores en Instagram/TikTok **no es obligatorio para postular con un sitio web** — aplica solo a un bono
   extra por visualizaciones. Confirma condiciones vigentes al postular (programa lanzado en 2025, puede variar).
2. Sustituye cada `affiliateUrl` de `src/data/products.ts` por tu enlace real generado desde el panel de
   afiliados.
3. Verifica/actualiza precio, rating y specs de cada producto contra la publicación real en Mercado Libre.
4. El producto recomendado debe ser nuevo y del vendedor con buena reputación (**nivel verde**) — requisito
   del programa para que la venta cuente como válida.
5. Sustituye los placeholders `ProductImage` por imágenes reales solo si el programa/vendedor lo permite.
6. Compra un dominio `.cl`, p. ej. `comparaoficina.cl`, y actualiza `SITE_URL` en `src/consts.ts` y `site` en
   `astro.config.mjs` (ahora mismo apuntan a un dominio de ejemplo).
7. Despliega en Vercel/Netlify (gratis) conectando tu repo — build command `npm run build`, output `dist/`.
8. Da de alta el dominio en **Google Search Console** y envía el sitemap (`/sitemap-index.xml`) — esto es lo
   que hace que Google empiece a rastrear e indexar activamente, en vez de esperar a encontrarlo solo.
9. Añade política de privacidad + aviso de cookies (ya está el disclaimer de afiliado en el footer).
10. Los pagos requieren acumular al menos **$4.500 CLP** provenientes de **3 transacciones válidas**.

## 5. Prompts para ampliar la web con una IA no-code (tipo la del vídeo)

Si en algún momento prefieres iterar visualmente con una herramienta no-code (Lovable, Bolt, v0, Databutton...)
en vez de pedirme cambios de código directamente, estos prompts siguen sirviendo como referencia adaptados al
nicho multi-categoría — aunque ya no aplican a la arquitectura Astro actual (esas herramientas generan SPAs).

**Prompt — Base de datos de producto genérica:**
> Crea una tabla de productos con estos campos comunes: marca, modelo, tipo de producto (silla/escritorio/
> monitor/accesorio), gama (económica/media/premium/gaming), precio en CLP, rating, nº de reseñas, perfil de
> usuario ideal, 3 ventajas, 2 inconvenientes, lista de características, y una lista flexible de
> especificaciones técnicas (etiqueta + valor) que varíe según el tipo de producto.

**Prompt — Ingesta rápida de productos con Claude:**
> Abre la publicación de Mercado Libre del producto que quieras añadir, copia título, precio, specs y reseñas
> destacadas, pégaselo a Claude y pídele: "Genera un JSON con estos campos: [pega tu lista de campos] a partir
> de este texto de publicación de Mercado Libre." Luego pega ese JSON en `src/data/products.ts`.

## 6. Lo que el SEO técnico NO resuelve por sí solo

Tener la arquitectura correcta es condición necesaria pero no suficiente. Para tráfico orgánico real, esto
sigue dependiendo de trabajo continuo (no es algo que se resuelva una sola vez):

1. **Contenido regular** — artículos de blog/comparativas que capturen búsquedas informacionales ("mejor silla
   para dolor lumbar", "escritorio elevable vale la pena") y enlacen a las páginas de categoría/producto.
2. **Backlinks** — Google pondera cuántos otros sitios enlazan al tuyo. Sin ningún enlace externo, incluso con
   el SEO técnico perfecto, es difícil competir por keywords de volumen medio-alto en los primeros meses.
3. **Antigüedad y consistencia** — los sitios nuevos suelen tardar 3-6 meses en ganar tracción en Google
   aunque todo esté técnicamente bien hecho ("sandbox" de dominios nuevos).
4. **Search Console activo** — revisar qué keywords ya te traen impresiones (aunque no clics) para orientar
   qué contenido nuevo escribir, y detectar errores de indexación.

## 7. Plan de tráfico para Chile (sin pagar ads)

1. **SEO de contenido** — artículos por categoría y comparativas cruzadas: "mejor silla ergonómica para
   teletrabajo 2026", "escritorio elevable vs fijo: ¿vale la pena en Chile?", "monitor para setup gamer bajo
   $250.000". El clúster de oficina da muchos más ángulos de artículo que un solo producto.
2. **Redes sociales** — Reels/TikTok/Instagram mostrando "setup completo" (silla + escritorio + monitor) con
   el precio total, cerrando con "arma tu setup ideal en el link de la bio" hacia el comparador.
3. **Newsletter** — ya integrada. Úsala para avisar de bajadas de precio reales y anunciar combos ("setup
   completo por menos de $400.000").
4. **WhatsApp / grupos de ofertas** — muy usado en Chile para compartir descuentos.

## 8. Próximos pasos sugeridos (por orden de impacto)

1. Postular al Programa de Afiliados y Creadores de Mercado Libre y sustituir los enlaces placeholder.
2. Comprar dominio, desplegar, y verificar propiedad en Google Search Console + enviar el sitemap.
3. Publicar 5–8 artículos SEO, al menos uno por categoría más uno de "setup completo".
4. Conectar la newsletter a un proveedor real (Brevo, MailerLite tienen plan gratuito).
5. Añadir sección "combo de la semana" (silla + escritorio + monitor con buen precio/calidad conjunto) —
   aprovecha tener las 4 categorías para vender el pack, no solo productos sueltos.
