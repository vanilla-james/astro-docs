---
title: Despliega tu proyecto de Astro en GitHub pages
description: Cómo desplegar tu proyecto de Astro usando GitHub Pages.
layout: ~/layouts/DeployGuideLayout.astro
i18nReady: true
---

Puedes usar [GitHub pages](https://pages.github.com/) para desplegar tu proyecto de Astro directamente desde un repositorio en [GitHub.com](https://github.com/).

## Cómo desplegar

Puedes desplegar un proyecto de Astro en GitHub pages usando [GitHub Actions](https://github.com/features/actions) para construir y desplegar tu proyecto automáticamente. Para hacer esto, tu código fuente debe estar alojado en GitHub.

Astro mantiene la action oficial `withastro/action` para desplegar tu proyecto con muy poca configuración. Sigue las instrucciones a continuación para desplegar tu proyecto de Astro en GitHub pages y consulta el [README](https://github.com/withastro/action) si necesitas más información.

1. Configura las opciones [`site`](/es/reference/configuration-reference/#site) y, si es necesario, [`base`](/es/reference/configuration-reference/#base) en `astro.config.mjs`.

    ```js title="astro.config.mjs" ins={4-5}
    import { defineConfig } from 'astro/config'

    export default defineConfig({
      site: 'https://astronaut.github.io',
      base: '/mi-repo',
    })
    ```
    - `site` debe ser `https://<TU_NOMBRE_DE_USUARIO>.github.io` o `https://mi-dominio.com`
    - `base` debe ser el nombre de tu repositorio comenzando con una barra diagonal, por ejemplo `/mi-repo`. Esto es para que Astro entienda que el *root* de tu sitio web es `/mi-repo`, en lugar de `/` que es el valor por defecto.

    :::note
      No necesitas configuar el parametro `base` si:

    - Tu repositorio se llama `<TU_NOMBRE_DE_USUARIO>.github.io`.
    - Estás usando un dominio personalizado.
    :::

    :::caution
        Si previamente no configuraste un valor para `base`, y acabas de configurarlo para desplegar en GitHub, recuerda actualizar tus enlaces internos para que incluyan la `base`

    ```astro
    <a href="/mi-repo/nosotros">Nosotros</a>
    ```
    :::

2. Crea un nuevo archivo en tu proyecto en `.github/workflows/deploy.yml`, dentro del mismo copia y pega el siguiente código YAML.

    ```yaml title="deploy.yml"
    name: Github Pages Astro CI

    on:
      # Activa el workflow cada vez que hay un push a la rama `main`
      # ¿Usas un nombre diferente a main? Reemplaza `main` con el nombre de tu rama
      push:
        branches: [ main ]
      # Te permite ejecutar este workflow manualmente desde la pestaña Actions en GitHub.
      workflow_dispatch:
      
    # Permite clonar el repositorio y desplegar el proyecto
    permissions:
      contents: read
      pages: write
      id-token: write

    jobs:
      build:
        runs-on: ubuntu-latest
        steps:
          - name: Checkout your repository using git
            uses: actions/checkout@v2          
          - name: Install, build, and upload your site
            uses: withastro/action@v0

      deploy:
        needs: build
        runs-on: ubuntu-latest
        environment:
          name: github-pages
          url: ${{ steps.deployment.outputs.page_url }}
        steps:
          - name: Deploy to GitHub Pages
            id: deployment
            uses: actions/deploy-pages@v1
    ```
    
    :::caution
    La [action oficial de Astro](https://github.com/withastro/action) busca un archivo lockfile para detectar tu gestor de paquetes (`npm`, `yarn` o `pnpm`). Debes tener el archivo `package-lock.json`, `yarn.lock` o `pnpm-lock.yaml`, generado automáticamente por tu gestor de paquetes, en tu repositorio.
    :::

3. En GitHub, ve a la pestaña **Settings** de tu repositorio y busca la sección **Pages**.

4. Elige **GitHub Actions** como el **Source** de tu proyecto y presiona **Save**.  

5. Has *commit* del nuevo archivo workflow y has *push* a GitHub. 

¡Tu proyecto debería estar publicado! Cuando haya cambios en tu repositorio, GitHub Action los implementará automáticamente por ti.

:::tip[configurar un dominio personalizado]
Opcionalmente, puedes configurar un dominio personalizado agregando el siguiente archivo `./public/CNAME` a tu proyecto:

```txt title="public/CNAME"
sub.midominio.com
```

Esto desplegará tu proyecto en el dominio personalizado en lugar de `user.github.io`. No olvides también [configurar el DNS en el proveedor de dominio](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-a-subdomain).
:::
