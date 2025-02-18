---
setup: |
    import PackageManagerTabs from '~/components/tabs/PackageManagerTabs.astro'
layout: ~/layouts/MainLayout.astro
title: Guía de migración
description: Cómo migrar tu proyecto de Astro a la última versión.
i18nReady: true
---

Esta guía te ayudará a migrar desde alguna versión vieja a la última versión de Astro.

Puedes actualizar a la última versión de Astro en tu proyecto utilizando tu gestor de paquetes. Si estás utilizando integraciones de Astro, deberías considerar actualizarlas también a la última versión disponible.
<PackageManagerTabs>
  <Fragment slot="npm">
  ```shell
  # actualiza la dependencia astro:
  npm upgrade astro
  # o, para actualizar todas las dependencias:
  npm upgrade
  ```
  </Fragment>
  <Fragment slot="pnpm">
  ```shell
  # actualiza la dependencia astro:
  pnpm upgrade astro
  # o, para actualizar todas las dependencias:
  pnpm upgrade
  ```
  </Fragment>
  <Fragment slot="yarn">
  ```shell
  # actualiza la dependencia astro:
  yarn upgrade astro
  # o, para actualizar todas las dependencias:
  yarn upgrade
  ```
  </Fragment>
</PackageManagerTabs>


Lee la siguiente guía para ver los aspectos más destacados e instrucciones sobre cómo manejar los cambios no retrocompatibles.

## Astro 1.0

Astro v1.0 introduce algunos cambios que requieren tu atención si migras desde versiones v0.x y v1.0-beta. Sigue leyendo para más detalles.

### Actualización: Vite 3

Astro v1.0 ha sido actualizado de Vite 2 a [Vite 3](https://vitejs.dev/). Nos hemos encargado de la mayoría de la actualización por ti; sin embargo, algún comportamiento sutil puede cambiar entre versiones. Puedes acudir a la [Guía de Migración de Vite](https://vitejs.dev/guide/migration.html#general-changes) oficial si tienes problemas.

### Deprecado: `Astro.canonicalURL`

Ahora puedes usar el nuevo helper [`Astro.url`](/es/reference/api-reference/#astrourl) para construir tu propia URL canónica desde la página/request URL actual.

```js del="Astro.canonicalURL" ins="new URL(Astro.url.pathname, Astro.site)"
// Antes:
const canonicalURL = Astro.canonicalURL;
// Después:
const canonicalURL = new URL(Astro.url.pathname, Astro.site);
```

### Cambio: Especificidad del CSS Local

La [especificidad](https://developer.mozilla.org/es/docs/Web/CSS/Specificity) ahora va a ser respetada en los estilos CSS locales. Este cambio causará que la mayoría de los estilos locales _respeten_ la precedencia sobre los estilos globales. Pero, este comportamiento no es garantizado de manera explícita.  

Técnicamente, esto se logra utilizando [la pseudo-clase `:where()`](https://developer.mozilla.org/es/docs/Web/CSS/:where) en vez de usar clases directamente en el output de CSS de Astro.

Usemos el siguiente bloque de estilo en un componente de Astro como ejemplo:

```astro
<style>
  div { color: red; } /* especificidad 0-0-1 */
</style>
```

Anteriormente, Astro transformaría este bloque en el siguiente CSS, que tiene una especificidad de `0-1-1` — una especificidad mayor al CSS fuente:

```css del=".astro-XXXXXX"
div.astro-XXXXXX { color: red; } /* especificidad 0-1-1 */
```

Ahora, Astro envuelve el selector de clase con `:where()`, manteniendo la especificidad:

```css ins=":where(.astro-XXXXXX)"
div:where(.astro-XXXXXX) { color: red; } /* especificidad 0-0-1 */
```
El anterior aumento en especificidad complicaba la combinación entre estilos locales en Astro con archivos CSS o librerías de estilo (por ejemplo Tailwind, CSS Modules, Styled Components, Stitches), Este cambio permite que los estilos locales en Astro funcionen en conjunto de manera consistente mientras se preservan los límites exclusivos que previenen que los estilos se filtren fuera del componente.

:::caution
Al actualizar, por favor realiza una inspección visual de tu proyecto para asegurarte que se apliquen los estilos esperados. De no ser así, puedes buscar el estilo local e incrementar la especificidad del selector manualmente para mantener el comportamiento anterior.
:::

### Deprecado: Componentes y JSX en Markdown

Astro ya no es compatible con componentes o expresiones JSX en páginas de Markdown por defecto. Para obtener soporte a largo plazo debes migrar a la integración [`@astrojs/mdx`](/es/guides/integrations-guide/mdx/).

Para hacer la migración más simple, puedes utilizar la nueva [legacy flag](/es/reference/configuration-reference/#legacyastroflavoredmarkdown) para volver a activar las características previas de Markdown.

### Convertir archivos `.md` a `.mdx`

Si MDX no te es familiar, te presentamos unos pasos que puedes seguir para convertir un archivo "Astro Flavored Markdown" a MDX. Mientras más aprendas sobre MDX, ¡siéntete libre de explorar otras maneras de escribir tus páginas!

1. Instala la integración [`@astrojs/mdx`](/es/guides/integrations-guide/mdx/).

2. Cambia la extensión de tus archivos `.md` a `.mdx`

3. Remueve cualquier propiedad `setup:` de tu frontmatter y escribe las importaciones debajo del frontmatter.

    ```mdx del={4-5} ins={10}
    // src/pages/posts/my-post.mdx
    ---
    layout: '../../layouts/BaseLayout.astro'
    setup: |
      import ReactCounter from '../../components/ReactCounter.jsx'
    title: 'Migrando a MDX'
    date: 2022-07-26
    tags: ["markdown", "mdx", "astro"]
    ---
    import ReactCounter from '../../components/ReactCounter.jsx'

    # {frontmatter.title}

    Este es mi componente de contador, funcionando en MDX:

    <ReactCounter client:load />
    ```

4. Actualiza el uso de `Astro.glob()` que actualmente retornen archivos `.md` para que ahora retornen tus archivos `.mdx`.

    :::caution
    El objeto retornado al importar archivos `.mdx` (inclusive usando Astro.glob) difiere del objeto retornado al importar archivos `.md`. Sin embargo, `frontmatter`, `file`, y `url` funcionan de manera idéntica.
    :::

5. Actualiza el uso del componente `<Content />` para usar la exportación por defecto cuando importas MDX:

    ```astro title="src/pages/index.astro" ins=".default"
    ---
    // Múltiples importaciones con Astro.glob
    const mdxPosts = await Astro.glob('./posts/*.mdx');
    ---

    {mdxPosts.map(Post => <Post.default />)}
    ```
    
    ```astro title="src/pages/index.astro" ins="default as"
    ---
    // Importa una sola página
    import { default as About } from './about.mdx';
    ---

    <About />    
    ```

:::tip
Mientras estés migrando a MDX, puede que quieras [activar la flag legacy](/es/reference/configuration-reference/#legacyastroflavoredmarkdown) e incluir ambos archivos **`.md` y `.mdx`**, así tu sitio continúa funcionando normalmente aun luego de haber convertido todos tus archivos. Puedes hacerlo de la siguiente manera:

```astro
---
const mdPosts = await Astro.glob('../pages/posts/*.md');
const mdxPosts = await Astro.glob('../pages/posts/*.mdx');
const allPosts = [...mdxPosts, ...mdPosts];
---
```
:::

### Componente `<Markdown />` Removido

El componente `<Markdown />` incluido en Astro ha sido movido a un paquete aparte. Para continuar usando este componente, debes instalar `@astrojs/markdown-component` y actualizar tus importaciones como corresponde. Para más detalles, lee [el README de `@astrojs/markdown`](https://github.com/withastro/astro/tree/main/packages/markdown/component).

:::tip
Astro ahora soporta [MDX](https://mdxjs.com/) a través de nuestra [Integración con MDX](https://github.com/withastro/astro/tree/main/packages/integrations/mdx). MDX te permite incluir ambos Markdown y componentes importados en el mismo archivo. MDX puede ser una buena alternativa al componente `<Markdown />` gracias a su gran comunidad y APIs estables.
:::

## Migrar a v1.0.0-beta

El 4 de Abril de 2022 publicamos Astro 1.0 Beta! 🎉

Si te encuentras en la v0.25 o anterior, asegúrate de haber leído y seguido la [Guía de Migración a v0.26](#migrar-a-v026), la cual contiene muchos cambios no retrocompatibles.

La versión `v1.0.0-beta.0` de Astro no contiene cambios no retrocompatibles. A continuación veremos unos cambios pequeños que fueron introducidos durante el período de beta.

### Cambio: RSS Feeds

Los feeds RSS ahora deben ser generados usando el paquete `@astrojs/rss`, tal como se describe en nuestra [guía de RSS](/es/guides/rss/).

## Migrar a v0.26
### Nueva API de Configuración

Nuestra API de Configuración ha sido rediseñada para resolver algunos puntos de confusión que se han generado durante el último año. La mayoría de las opciones de configuración han sido movidas o renombradas, por lo cual es posible que sea una actualización rápida para la mayoría de los usuarios. Unas pocas opciones han sido refactorizadas más profundamente, y pueden requerir algunos cambios adicionales:

- `.buildOptions.site` ha sido reemplazado por `.site` (tu dominio desplegado) y una nueva opción `.base` (tu subdirección desplegada).
- `.markdownOptions` ha sido reemplazado por `.markdown`, un objeto de configuración similar con pequeños cambios para simplificar la configuración del Markdown.
- `.sitemap` fue movido dentro de la integración [@astrojs/sitemap](https://www.npmjs.com/package/@astrojs/sitemap).

Si ejecutas Astro con configuración legacy, verás una advertencia con instrucciones sobre cómo actualizar. Lee nuestra [Referencia de configuración](/es/reference/configuration-reference/) para más información.

Lee [RFC0019](https://github.com/withastro/rfcs/blob/main/proposals/0019-config-finalization.md) para obtener más contexto sobre estos cambios.

### Nueva API de Markdown

Astro v0.26 publica una nueva API de Markdown para tu contenido. Esto incluye tres grandes cambios al usuario:
- Ahora puedes usar `import`/`import()` para contenido markdown directamente utilizando una importación ESM.
- Una renovada API `Astro.glob()`, para importaciones glob más simples (especialmente para Markdown).
- **CAMBIO NO RETROCOMPATIBLE:** `Astro.fetchContent()` ha sido eliminado y reemplazado por `Astro.glob()`
- **CAMBIO NO RETROCOMPATIBLE:** Se ha actualizado la interfaz de los objetos de Markdown.

```js del={2} ins={4}
// v0.25
let allPosts = Astro.fetchContent('./posts/*.md');
// v0.26+
let allPosts = await Astro.glob('./posts/*.md');
```

Al migrar, ten cuidado con la nueva interfaz de los objetos de Markdown. El frontmatter, por ejemplo, ha sido movido a la propiedad `.frontmatter`, entonces referencias tales como `post.title` deberían cambiarse por `post.frontmatter.title`.

Esto debería resolver muchos problemas para usuarios de Markdown, incluyendo algunas mejoras de performance para sitios grandes.

Lee [RFC0017](https://github.com/withastro/rfcs/blob/main/proposals/0017-markdown-content-redesign.md) para obtener más contexto sobre estos cambios.

### Nuevo comportamiento de Script por defecto

Las etiquetas `<script>` en los componentes Astro ahora son construidos, empaquetados y optimizados por defecto. Esto logra finalmente que nuestra sintaxis de componentes de Astro sea más consistente, a tono con el comportamiento optimizado por defecto que nuestras etiquetas `<style>` tienen hoy en día.

Esto incluye algunos cambios de los que deberías estar al tanto:

- **NO RETROCOMPATIBLE:** `<script hoist>` es el nuevo comportamiento por defecto de `<script>`. El atributo `hoist` fue eliminado. Para usar el nuevo comportamiento por defecto, asegúrate que no haya otros atributos en la etiqueta `<script>`. Por ejemplo, elimina `type="module"` si lo estabas utilizando.
- Nueva directiva `<script is:inline>`, para revertir una etiqueta `<script>` a su comportamiento previo (sin que Astro la construya, empaquete ni toque).
- Nueva directiva `<style is:inline>`, para indicar que una etiqueta de estilo se mantenga en línea en la plantilla de página (similar al comportamiento previo de `<script>`).
- Nueva directiva `<style is:global>` para reemplazar `<style global>` en una publicación futura.


```js del={2} ins={4}
// v0.25
<script hoist type="module">
// v0.26+
<script>
```

Lee cómo usar [scripts del lado del cliente](/es/core-concepts/astro-components/#scripts-del-lado-del-cliente) en Astro detalladamente.

Lee [RFC0016](https://github.com/withastro/rfcs/blob/main/proposals/0016-style-script-defaults.md) para obtener más contexto sobre estos cambios.

### Actualización de la API `Astro.request`


`Astro.request` ha sido cambiada desde un objeto personalizado nuestro a un objeto estándar `Request`. Esto es parte de un proyecto para usar más APIs estándar de la web, especialmente durante la utilización de SSR.

Esto incluye algunos cambios de los que deberías estar al tanto:

- Cambia `Astro.request` para convertirse en un objeto [Request](https://developer.mozilla.org/es/docs/Web/API/Request).
- Mueve `Astro.request.params` a `Astro.params`.
- Mueve `Astro.request.canonicalURL` a `Astro.canonicalURL`.

Lee [RFC0018](https://github.com/withastro/rfcs/blob/main/proposals/0018-astro-request.md) para obtener más contexto sobre estos cambios.


### Otros cambios

- Mejoras en la API `Astro.slots` para admitir pasar argumentos a slots basados en funciones. Esto permite componentes utilitarios de mayor ergonomía que pueden aceptar una función callback como hijo.
- Actualizaciones en el formato del texto de salida en la CLI, especialmente en reportes de errores.
- Actualizaciones en `@astrojs/compiler`, arreglando algunos bugs relacionados a utilización de RegExp en el frontmatter.

## Migrar a v0.25

### Integraciones de Astro

¡La configuración de `renderers` ha sido reemplazada por un nuevo y oficial sistema de integraciones! Esto desbloquea algunas características emocionantes para Astro. Puedes leer nuestra guía [Usando Integraciones](/es/guides/integrations-guide/) para más detalles sobre cómo usar este nuevo sistema.

Las Integraciones reemplazan nuestro concepto original de `renderers` y trae consigo cambios no retrocompatibles y nuevas configuraciones por defecto para usuarios existentes. Veremos estos cambios a continuación.

#### Removido: Soporte de Frameworks incorporado

Anteriormente, React, Preact, Svelte y Vue estaban incluidos con Astro por defecto. Desde la v0.25.0, Astro ya no trae ningún renderer incorporado. Si no tenías una configuración `renderers` definida en tu proyecto, ahora debes instalar tú mismo todos los frameworks que utilices.

Lee nuestra [guía paso a paso](/es/guides/integrations-guide/) para aprender a agregar una nueva integración a Astro para el o los frameworks que estés utilizando.
#### Deprecado: Renderers

:::note
Lee esta sección si ya tienes "renderers" personalizados definidos en tu archivo de configuración.
:::

El nuevo sistema de integraciones reemplaza al viejo sistema de `renderers`, incluyendo los paquetes `@astrojs/renderer-*` publicados en npm. De aquí en más, `@astrojs/renderer-react` se convierte en `@astrojs/react`, `@astrojs/renderer-vue` se convierte en `@astrojs/vue`, etcétera.

**Para migrar:** actualiza Astro a `v0.25.0` y luego ejecuta `astro dev` o `astro build` con tu configuración vieja con `"renderers"`. Verás inmediatamente una noticia indicándote exactamente los cambio que debes realizar en tu archivo `astro.config.mjs`, basándote en tu configuración actual. También puedes actualizar los paquetes tú mismo, siguiendo la tabla que veremos a continuación.

Para una guía más detallada, lee nuestra [guía paso a paso](/es/guides/integrations-guide/) para aprender cómo reemplazar renderers existentes con una nueva integración de Astro.

```shell add={3-4}
# Instala tus nuevas integraciones y frameworks:
# (Lee la guía detallada: https://docs.astro.build/es/guides/integrations-guide)
npm install @astrojs/lit lit
npm install @astrojs/react react react-dom
```

```js ins={3-4,8} del={7}
// Luego, actualiza tu archivo `astro.config.mjs`:
// (Lee la guía detallada: https://docs.astro.build/es/guides/integrations-guide)
import lit from '@astrojs/lit';
import react from '@astrojs/react';

export default {
  renderers: ['@astrojs/renderer-lit', '@astrojs/renderer-react'],
  integrations: [lit(), react()],
}
```


| Renderers Deprecados en npm | Integraciones v0.25+ en npm|
| --------------------------- | -------------------------- |
| @astrojs/renderer-react     | @astrojs/react             |
| @astrojs/renderer-preact    | @astrojs/preact            |
| @astrojs/renderer-solid     | @astrojs/solid-js          |
| @astrojs/renderer-vue       | @astrojs/vue               |
| @astrojs/renderer-svelte    | @astrojs/svelte            |

#### Manejando Dependencias Peer

:::note
Lee esta sección si no tienes instalada la v14 de Node **o** si usas un gestor de paquetes distinto a npm.
:::

Diferente a los viejos renderers, las integraciones ya no tienen los frameworks ("react", "svelte", "vue", etc.) como dependencias directas. En cambio ahora, deberías instalar los paquetes de framework tú mismo *además de* tus integraciones.

```shell ins="react react-dom"
# Ejemplo: Instala integraciones y frameworks en conjunto
npm install @astrojs/react react react-dom
```

Si ves una advertencia con el texto `"Cannot find package 'react'"` (o similar) cuando intentas ejecutar Astro, esto significa que debes instalar ese paquete en tu proyecto. Lee nuestra [nota sobre dependencias peer](/es/guides/troubleshooting/#cannot-find-package-x) en la guía de solución de problemas para más información.

Si usas `npm` y Node v16+ entonces es posible que esto sea manejado automáticamente por `npm`, ya que las últimas versiones de `npm` (v7+) instala las dependencias peer por ti automáticamente. En este caso, instalar un framework como "react" en tu proyecto es un paso opcional aunque recomendado.

### Actualización: Resaltado de Sintaxis

Amamos encontrar configuraciones por defecto sensatas y que "simplemente funcionan" out-of-the-box. Como parte de esto, decidimos que [Shiki](https://github.com/shikijs/shiki) sea nuestro nuevo resaltador de sintaxis por defecto. Viene pre-configurado con el tema `github-dark`, proveyendo resaltado de sintaxis con cero configuración en tus bloques de código sin clases extra de css, hojas de estilos, o JS del lado del cliente extraños.

Lee nuestros nuevos [docs de resaltado de sintaxis](/es/guides/markdown-content/#resaltado-de-sintaxis) para más detalles. **Si deseas mantener Prism como tu resaltador de sintaxis,** [establece la opción `syntaxHighlight` a `'prism'`](/es/guides/markdown-content/#configuración-de-prism) en la configuración del markdown de tu proyecto.

#### El componente `<Prism />` tiene un nuevo hogar

Como parte de nuestra misión de mantener el núcleo (core) de Astro lo más pequeño posible, hemos movido el componente incorporado `Prism` fuera de `astro/components` y dentro del paquete `@astrojs/prism`. Ahora puedes importar este componente desde `@astrojs/prism` de la siguiente manera:

```astro
---
import { Prism } from '@astrojs/prism';
---
```

¡Como el paquete `@astrojs/prism` todavía está empaquetado con el núcleo (core) de `astro`, no necesitas instalar nada nuevo ni añadir Prism como integración! Sin embargo, puedes notar que nuestro plan es extraer `@astrojs/prism` (y el resaltado de sintaxis de Prism en general) a un paquete separado e instalable en un futuro. Lee [la referencia de la API del componente `<Prism />`](/es/reference/api-reference/#prism-) para más información.

### Actualización del Parser de CSS

Nuestro parser interno de CSS ha sido actualizado y provee mejor soporte para uso de sintaxis avanzada de CSS, como container queries. Este cambio será invisible a la mayoría de los usuarios pero creemos que los usuarios más avanzados disfrutarán de esta nueva característica.
## Migrar a v0.24

:::note
La nueva estrategia de construcción está activada por defecto en 0.24. Si tienes algún problema puedes continuar usando la vieja estrategia de construcción usando la flag `--legacy-build`. Por favor [abre un issue](https://github.com/withastro/astro/issues/new/choose) para que podamos resolver cualquier problema causado por la nueva estrategia de construcción.
:::

0.24 introdujo una nueva estrategia de *static build* que cambia el comportamiento de algunas características nuevas. Este comportamiento estaba disponible en versiones anteriores de Astro por medio de una flag opcional: `--experimental-static-build`.

Para migrar a la transición, ten en cuenta los siguientes cambios que serán requeridos para este nuevo engine de construcción. Puedes hacer estos cambios en tu proyecto en cualquier momento para adelantar trabajo.

### Deprecado: `Astro.resolve()`

`Astro.resolve()` te permite obtener URLs resueltas a recursos que necesites referenciar en el navegador. Esto era más usado comúnmente dentro de etiquetas `<link>` y `<img>` para cargar archivos CSS e imágenes depende de cómo sea necesario. Desafortunadamente esto no va a funcionar debido a que ahora Astro construye los recursos en *tiempo de construcción* en vez de *tiempo de renderizado*. Es mejor que actualices las referencias a tus recursos de las siguientes opciones "a prueba del futuro":

#### Cómo Resolver Archivos CSS

**1. Importación ESM (Recomendadp)**

**Ejemplo:** `import './style.css';`
**Cuándo usar esto:** Si tu archivo CSS se encuentra dentro de la carpeta `src/` y quieres aprovechar características de construcción y optimización de CSS automáticas.

Usa una importación ESM para añadir CSS en la página. Astro detecta estas importaciones de CSS y luego construye, optimiza y añade el CSS a la página automáticamente. Esta es la forma más fácil de migrar desde `Astro.resolve()` mientras mantienes la construcción/empaquetado automático que provee Astro.

```astro
---
// Ejemplo: Astro va a incluir y optimizar este CSS por ti automáticamente
import './style.css';
---
<html><!-- Tu página aquí --></html>
```

Importar archivos CSS debe funcionar en cualquier lugar que las importaciones ESM sean admitidas, incluyendo:
- Archivos JavaScript
- Archivos TypeScript
- Frontmatter de componentes Astro
- Componentes no-Astro como React, Svelte y otros

Cuando se importa un archivo CSS usando ese método, cualquier declaración de `@import` también es resuelta y agregada en línea al archivo CSS importado. Todas las referencias de `url()` también son resueltas de forma relativa al archivo fuente, y cualquier recurso referenciado con `url()` va a ser incluido en la construcción final de tu sitio.


**2. Path URL Absoluto**

**Ejemplo:** `<link href="/style.css">`
**Cuándo usar esto:** Si tu archivo CSS se encuentra dentro de la carpeta `public/` y prefierees crear tu elemento HTML `link` por ti mismo.

Puedes referenciar cualquier archivo dentro de la carpeta `public/` usando path URL absoluto en la plantilla de tu componente. Esta es una buena opción si quieres mantener el control de la etiqueta `<link>`. Sin embargo, este enfoque saltea el procesado, empaquetado y optimizaciones de CSS provistas por Astro cuando usas el método `import` descripto anteriormente.

Recomendamos usar el enfoque de `import` sobre el de path URL absoluto ya que provee la mejor performance y mayores características de CSS por defecto.

#### Cómo Resolver Archivos JavaScript


**1. Path URL Absoluto**

**Ejemplo:** `<script src="/some-external-script.js" />`
**Cuándo usar esto:** Si tu archivo JavaScript se encuentra dentro de la carpeta `public/`.

Puedes referenciar cualquier archivo dentro de la carpeta `public/` usando path URL absoluto en la plantilla de tu componente. Esta es una buena opción si quieres mantener el control de la etiqueta `<script>`.

Nota que este enfoque saltea el procesado, empaquetado y optimizaciones de JavaScript provistas por Astro cuando usas el método `import` descrito aquí debajo. Sin embargo, puedes preferir esto para scripts externos que hayan sido publicados y _minificados_ de forma separada a Astro. Si tu script fue descargado de una fuente externa, entonces es probable que prefieras usar este método.

**2. Importación ESM vía `<script hoist>`**

**Ejemplo:** `<script hoist>import './some-external-script.js';</script>`
**Cuándo usar esto:** Si tu script externo se encuentra dentro de la carpeta `src/` _y_ soporta el tipo de módulo ESM.

Usa una importación ESM dentro de un elemento `<script hoist>` en la plantilla de tu componente Astro y Astro incluirá el archivo JavaScript en la construcción final. Astro detecta estas importaciones de JavaScript del lado del cliente y luego construye, optimiza y añade el JavaScript a la página automáticamente. Esta es la forma más simple de migrar desde `Astro.resolve()` mientras mantienes la construcción/empaquetado automático que provee Astro.

```astro
<script hoist>
  import './algun-script-externo.js';
</script>
```

Nota que Astro empaquetará este script externo con el resto de tu JavaScript del lado del cliente y lo cargará en el contexto de script `type=module`. Es probable que algunos archivos de JavaScript viejos no estén escritos para el contexto `module`, en cuyo caso estos archivos deberían ser actualizados para usar este método.

#### Cómo Resolver Imágenes & Otros Recursos

**1. Path URL Absoluto (Recomendado)**

**Ejemplo:** `<img src="/penguin.png">`
**Cuándo usar esto:** Si tu recurso se encuentra dentro de la carpeta `public/`.

Si colocas tus imágenes dentro de la carpeta `public/` puedes referenciarlas con un path URL absoluto directo en la plantilla de tu componente. Esta es la forma más simple de referenciar un recurso para ser utilizado y es lo recomendado para la mayoría de los usuarios que recién comienzan a usar Astro.

**2. Importación ESM**

**Ejemplo:** `import imgUrl from './penguin.png'`
**Cuándo usar esto:** Si tu recurso se encuentra dentro de la carpeta `src/` y quieres aprovechar características de optimización automáticas tales como hashing en nombres de archivos.

Esto funciona dentro de cualquier componente JavaScript o Astro y retorna una URL resuelta a la imagen final. Una vez que tienes la URL resuelta, puedes usarla en cualquier lugar de la plantilla de tu componente.

```astro
---
// Ejemplo: Astro incluirá esta imagen en el resultado final
import imgUrl from './penguin.png';
---
<img src={imgUrl} />
```

De manera similar a cómo Astro maneja CSS, la importación ESM le permite a Astro realizar algunas optimizaciones simples por ti automáticamente. Por ejemplo, cualquier recurso dentro de la carpeta `src/` que sea importado por medio de una importación ESM (ejemplo: `import imgUrl from './penguin.png'`) tendrá su nombre _hasheado_ automáticamente. Esto te permite guardar el archivo en cache en el servidor de manera más agresiva, mejorando la performance para el usuario. En un futuro, Astro agregará más optimizaciones de este estilo.

**Tip:** Si no te gustan las importaciones ESM estáticas, Astro también soporta importaciones ESM dinámicas. Solamente recomendamos esta opción si prefieres este tipo de sintaxis: `<img src={(await import('./penguin.png')).default} />`.

### Deprecado: Procesamiento de `<script>` por Defecto

Anteriormente, todos los elementos `<script>` del HTML generado eran leídos, procesados y empaquetados automáticamente. Este comportamiento ya no es así por defecto. Desde la versión 0.24, debes optar al procesamiento de elementos `<script>` por medio del atributo `hoist`. También es requerido `type="module"` para módulos hoisted.

```astro
<script>
  // ¡Va a ser renderizado en el HTML tal cual esté escrito!
  // Las importaciones ESM no serán resueltas de forma relativa al archivo.
</script>
<script type="module" hoist>
  // ¡Procesado! ¡Empaquetado! Las importaciones ESM funcionan, aun para paquetes npm.
</script>
```


## Migrar a v0.23

### Error: No se encuentra Sass

```
Preprocessor dependency "sass" not found. Did you install it?
```

En nuestra búsqueda por reducir el tamaño de instalación por npm, hemos movido [Sass](https://sass-lang.com/) a una dependencia externa opcional. Si usas Sass en tu proyecto, deberías asegurarte de ejectutar `npm install sass --save-dev` para instalarlo como dependencia.

### Deprecado: HTML sin escapar

En Astro v0.23+, el contenido HTML sin escapar en expresiones está deprecado.
En publicaciones futuras, el contenido dentro de expresiones tendrá strings escapadas para protegerlas contra inyección de HTML inesperadas.

```astro del={1} ins={2}
<h1>{title}</h1> <!-- <h1>Hello <strong>World</strong></h1> -->
<h1>{title}</h1> <!-- <h1>Hello &lt;strong&gt;World&lt;/strong&gt;</h1> -->
```

Para continuar inyectando HTML sin escapar, puedes usar `set:html`.

```astro del={1} ins={2}
<h1>{title}</h1>
<h1 set:html={title} />
```

Para evitar tener un elemento envolvente, `set:html` funciona en conjunto con `<Fragment>`.

```astro del={1} ins={2}
<h1>{title}!</h1>
<h1><Fragment set:html={title}>!</h1>
```

También puedes protegerte de inyecciones de HTML inesperadas con `set:text`.

```astro
<h1 set:text={title} /> <!-- <h1>Hello &lt;strong&gt;World&lt;/strong&gt;</h1> -->
```

## Migrar a v0.21

### Vite

Desde la v0.21, Astro es construido usando [Vite].
Como resultado de esto, las configuraciones escritas en `snowpack.config.mjs` deben ser movidas a `astro.config.mjs`.

```js
// @ts-check

/** @type {import('astro').AstroUserConfig} */
export default {
  renderers: [],
  vite: {
    plugins: [],
  },
};
```

Para aprender más sobre configuraciones de Vite, por favor visita su [guía de configuración](https://vitejs.dev/config/).

#### Plugins de Vite

En Astro v0.21+, los plugins de Vite pueden ser configurados dentro de `astro.config.mjs`.

```js ins={4-6}
import { imagetools } from 'vite-imagetools';

export default {
  vite: {
    plugins: [imagetools()],
  },
};
```

Para aprender más sobre plugins de Vite, por favor visita su [guía de plugins](https://vitejs.dev/guide/using-plugins.html).

#### Cambios en Vite sobre Renderers

En Astro v0.21+, los plugins deben usar `viteConfig()`.

```js del={8-9} ins={2,10-23}
// renderer-svelte/index.js
import { svelte } from '@sveltejs/vite-plugin-svelte';

export default {
  name: '@astrojs/renderer-svelte',
  client: './client.js',
  server: './server.js',
  snowpackPlugin: '@snowpack/plugin-svelte',
  snowpackPluginOptions: { compilerOptions: { hydratable: true } },
  viteConfig() {
    return {
      optimizeDeps: {
        include: ['@astrojs/renderer-svelte/client.js', 'svelte', 'svelte/internal'],
        exclude: ['@astrojs/renderer-svelte/server.js'],
      },
      plugins: [
        svelte({
          emitCss: true,
          compilerOptions: { hydratable: true },
        }),
      ],
    };
  },
}
```

Para aprender más sobre plugins de Vite, por favor visita su [guía de plugins](https://vitejs.dev/guide/using-plugins.html).

:::note
En publicaciones anteriores, los plugins eran configurados usando `snowpackPlugin` o `snowpackPluginOptions`.
:::


### Alias

En Astro v0.21+, los alias en importaciones pueden ser añadidos en `tsconfig.json` o `jsconfig.json`.

```json add={4-6}
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/components/*": ["src/components/*"]
    }
  }
}
```

_Estos aliases se integrarán automáticamente en [VSCode](https://code.visualstudio.com/docs/languages/jsconfig) y otros editores._

### Extensiones de Archivos en Importaciones

En Astro v0.21+, se debe referenciar a los archivos incluyendo su extensión, tal cual como están en el disco. En este ejemplo, `Div.tsx` debería ser referenciado como `Div.tsx`, y no como `Div.jsx`.

```js del={1} ins={2}
import Div from './Div.jsx' // Astro v0.20
import Div from './Div.tsx' // Astro v0.21
```

Este mismo cambio aplica a archivos que se compilan a css, como `Div.scss`:

```astro del={1} ins={2}
<link rel="stylesheet" href={Astro.resolve('./Div.css')}>
<link rel="stylesheet" href={Astro.resolve('./Div.scss')}>
```

### Removido: Componentes en el Frontmatter

Antes, era posible crear mini componentes de Astro dentro del Frontmatter de Astro, utilizando sintaxis JSX en vez de sintaxis de componentes de Astro. Esto siempre fue un estilo de hack pero en el nuevo compilador se volvió imposible de dar soporte. Esperamos re-introducir esta característica en una publicación futura de Astro usando una API diferente, que no sea JSX.

Para migrar a v0.21+, por favor convierte todos los componentes Astro en JSX (si es que tienes componentes de Astro creados dentro del frontmatter de otro componente) a componentes separados.


### Cambios en Estilos

#### Autoprefixer

Autoprefixer ya no es ejecutado por defecto. Para habilitarlo:

1. Instala la última versión (`npm i autoprefixer`)
2. Crea un archivo `postcss.config.cjs` en la raíz de tu proyecto con el siguiente contenido:
   ```js
   module.exports = {
     plugins: {
       autoprefixer: {},
     },
   };
   ```

#### Tailwind CSS

Asegúrate de tener PostCSS instalado. Esto era opcional en versiones anteriores pero ahora es un requerimiento:

1. Instala la última versión de postcss (`npm i -D postcss`)
2. Crea un archivo `postcss.config.cjs` en la raíz de tu proyecto con el siguiente contenido:
   ```js
   module.exports = {
     plugins: {
       tailwindcss: {},
     },
   };
   ```
   Para más información, lee la [documentación de Tailwind CSS](https://tailwindcss.com/docs/installation#add-tailwind-as-a-post-css-plugin)


### Problemas conocidos

#### Imports en la parte superior

En Astro v0.21+, se ha introducido un bug que obliga a que las importaciones tengan que estar en la parte superior del frontmatter.

```astro
---
import Componente from '../componentes/Componente.astro'
const dondeDeboImportarComponentes = "en la parte superior!"
---
```


[vite]: https://vitejs.dev
