<h1 align="center">
    <a href="https://crawlee.dev">
        <picture>
          <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/apify/crawlee-python/master/website/static/img/crawlee-dark.svg?sanitize=true">
          <img alt="Crawlee" src="https://raw.githubusercontent.com/apify/crawlee-python/master/website/static/img/crawlee-light.svg?sanitize=true" width="500">
        </picture>
    </a>
    <br>
    <small>Una biblioteca de web scraping y automatización de navegadores</small>
</h1>

<p align=center>
    <a href="https://trendshift.io/repositories/11169" target="_blank"><img src="https://trendshift.io/api/badge/repositories/11169" alt="apify%2Fcrawlee-python | Trendshift" style="width: 250px; height: 55px;" width="250" height="55"/></a>
</p>

<p align=center>
    <a href="https://badge.fury.io/py/crawlee" rel="nofollow">
        <img src="https://badge.fury.io/py/crawlee.svg" alt="Versión de PyPI" style="max-width: 100%;">
    </a>
    <a href="https://pypi.org/project/crawlee/" rel="nofollow">
        <img src="https://img.shields.io/pypi/dm/crawlee" alt="PyPI - Descargas" style="max-width: 100%;">
    </a>
    <a href="https://pypi.org/project/crawlee/" rel="nofollow">
        <img src="https://img.shields.io/pypi/pyversions/crawlee" alt="PyPI - Versión de Python" style="max-width: 100%;">
    </a>
    <a href="https://discord.gg/jyEM2PRvMU" rel="nofollow">
        <img src="https://img.shields.io/discord/801163717915574323?label=discord" alt="Chatea en Discord" style="max-width: 100%;">
    </a>
</p>

Crawlee cubre tu rastreo y scraping de principio a fin y **te ayuda a construir scrapers fiables. Rápido.**

Tus crawlers parecerán casi humanos y pasarán desapercibidos para las protecciones de bots modernas incluso con la configuración predeterminada. Crawlee te da las herramientas para rastrear la web en busca de enlaces, extraer datos y almacenarlos de forma persistente en formatos legibles por máquina, sin tener que preocuparte por los detalles técnicos. Y gracias a las ricas opciones de configuración, puedes ajustar casi cualquier aspecto de Crawlee para adaptarlo a las necesidades de tu proyecto si la configuración predeterminada no es suficiente.

> 👉 **Consulta la documentación completa, guías y ejemplos en el [sitio web del proyecto Crawlee](https://crawlee.dev/python/)** 👈

También tenemos una implementación de Crawlee en TypeScript, que puedes explorar y utilizar para tus proyectos. Visita nuestro repositorio de GitHub para más información [Crawlee para JS/TS en GitHub](https://github.com/apify/crawlee).

## Instalación

Te recomendamos visitar el [tutorial de Introducción](https://crawlee.dev/python/docs/introduction) en la documentación de Crawlee para más información.

Crawlee está disponible como el paquete [`crawlee`](https://pypi.org/project/crawlee/) en PyPI. Este paquete incluye la funcionalidad principal, mientras que las características adicionales están disponibles como extras opcionales para mantener las dependencias y el tamaño del paquete al mínimo.

Para instalar Crawlee with all features, ejecuta el siguiente comando:

```sh
python -m pip install 'crawlee[all]'
```

Luego, instala las dependencias de [Playwright](https://playwright.dev/):

```sh
playwright install
```

Verifica que Crawlee se ha instalado correctamente:

```sh
python -c 'import crawlee; print(crawlee.__version__)'
```

Para instrucciones de instalación detalladas, consulta la página de documentación [Configuración](https://crawlee.dev/python/docs/introduction/setting-up).

### Con Crawlee CLI

La forma más rápida de empezar con Crawlee es usando la CLI de Crawlee y seleccionando una de las plantillas preparadas. Primero, asegúrate de tener [uv](https://pypi.org/project/uv/) instalado:

```sh
uv --help
```

Si [uv](https://pypi.org/project/uv/) no está instalado, sigue la [guía de instalación oficial](https://docs.astral.sh/uv/getting-started/installation/).

Luego, ejecuta la CLI y elige entre las plantillas disponibles:

```sh
uvx 'crawlee[cli]' create my-crawler
```

Si ya tienes `crawlee` instalado, puedes iniciarlo ejecutando:

```sh
crawlee create my-crawler
```

## Ejemplos

Aquí tienes algunos ejemplos prácticos para ayudarte a empezar con diferentes tipos de crawlers en Crawlee. Cada ejemplo demuestra cómo configurar y ejecutar un crawler para casos de uso específicos, ya sea que necesites manejar páginas HTML simples o interactuar con sitios con mucho JavaScript. La ejecución de un crawler creará un directorio `storage/` en tu directorio de trabajo actual.

### BeautifulSoupCrawler

El [`BeautifulSoupCrawler`](https://crawlee.dev/python/api/class/BeautifulSoupCrawler) descarga páginas web usando una biblioteca HTTP y proporciona contenido analizado en HTML al usuario. Por defecto, utiliza [`HttpxHttpClient`](https://crawlee.dev/python/api/class/HttpxHttpClient) para la comunicación HTTP y [BeautifulSoup](https://pypi.org/project/beautifulsoup4/) para analizar HTML. Es ideal para proyectos que requieren una extracción eficiente de datos de contenido HTML. Este crawler tiene un rendimiento muy bueno ya que no utiliza un navegador. Sin embargo, si necesitas ejecutar JavaScript del lado del cliente para obtener tu contenido, esto no será suficiente y necesitarás usar [`PlaywrightCrawler`](https://crawlee.dev/python/api/class/PlaywrightCrawler). Además, si quieres usar este crawler, asegúrate de instalar `crawlee` con el extra `beautifulsoup`.

```python
import asyncio

from crawlee.crawlers import BeautifulSoupCrawler, BeautifulSoupCrawlingContext


async def main() -> None:
    crawler = BeautifulSoupCrawler(
        # Limita el rastreo a un máximo de solicitudes. Elimínalo o auméntalo para rastrear todos los enlaces.
        max_requests_per_crawl=10,
    )

    # Define el manejador de solicitudes predeterminado, que se llamará para cada solicitud.
    @crawler.router.default_handler
    async def request_handler(context: BeautifulSoupCrawlingContext) -> None:
        context.log.info(f'Procesando {context.request.url} ...')

        # Extrae datos de la página.
        data = {
            'url': context.request.url,
            'title': context.soup.title.string if context.soup.title else None,
        }

        # Envía los datos extraídos al conjunto de datos predeterminado.
        await context.push_data(data)

        # Encola todos los enlaces encontrados en la página.
        await context.enqueue_links()

    # Ejecuta el crawler con la lista inicial de URLs.
    await crawler.run(['https://crawlee.dev'])


if __name__ == '__main__':
    asyncio.run(main())
```

### PlaywrightCrawler

El [`PlaywrightCrawler`](https://crawlee.dev/python/api/class/PlaywrightCrawler) utiliza un navegador sin cabeza para descargar páginas web y proporciona una API para la extracción de datos. Está construido sobre [Playwright](https://playwright.dev/), una biblioteca de automatización diseñada para gestionar navegadores sin cabeza. Sobresale en la recuperación de páginas web que dependen de JavaScript del lado del cliente para la generación de contenido, o tareas que requieren interacción con contenido impulsado por JavaScript. Para escenarios donde la ejecución de JavaScript es innecesaria o se requiere un mayor rendimiento, considera usar el [`BeautifulSoupCrawler`](https://crawlee.dev/python/api/class/BeautifulSoupCrawler). Además, si quieres usar este crawler, asegúrate de instalar `crawlee` con el extra `playwright`.

```python
import asyncio

from crawlee.crawlers import PlaywrightCrawler, PlaywrightCrawlingContext


async def main() -> None:
    crawler = PlaywrightCrawler(
        # Limita el rastreo a un máximo de solicitudes. Elimínalo o auméntalo para rastrear todos los enlaces.
        max_requests_per_crawl=10,
    )

    # Define el manejador de solicitudes predeterminado, que se llamará para cada solicitud.
    @crawler.router.default_handler
    async def request_handler(context: PlaywrightCrawlingContext) -> None:
        context.log.info(f'Procesando {context.request.url} ...')

        # Extrae datos de la página.
        data = {
            'url': context.request.url,
            'title': await context.page.title(),
        }

        # Envía los datos extraídos al conjunto de datos predeterminado.
        await context.push_data(data)

        # Encola todos los enlaces encontrados en la página.
        await context.enqueue_links()

    # Ejecuta el crawler con la lista inicial de solicitudes.
    await crawler.run(['https://crawlee.dev'])


if __name__ == '__main__':
    asyncio.run(main())
```

### Más ejemplos

Explora nuestra página de [Ejemplos](https://crawlee.dev/python/docs/examples) en la documentación de Crawlee para una amplia gama de casos de uso y demostraciones adicionales.

## Características

¿Por qué Crawlee es la opción preferida para el web scraping y el rastreo?

### ¿Por qué usar Crawlee en lugar de una biblioteca HTTP cualquiera con un analizador de HTML?

- Interfaz unificada para rastreo **HTTP y de navegador sin cabeza**.
- **Rastreo paralelo** automático basado en los recursos del sistema disponibles.
- Escrito en Python con **sugerencias de tipo** - mejora la DX (autocompletado del IDE) y reduce los errores (comprobación de tipo estática).
- **Reintentos automáticos** en caso de errores o cuando te bloquean.
- **Rotación de proxy** y gestión de sesiones integradas.
- **Enrutamiento de solicitudes** configurable - dirige las URLs a los manejadores apropiados.
- **Cola persistente** para las URLs a rastrear.
- **Almacenamiento** conectable tanto de datos tabulares como de archivos.
- **Manejo de errores** robusto.

### ¿Por qué usar Crawlee en lugar de Scrapy?

- **Basado en Asyncio** – Aprovechando la biblioteca estándar [Asyncio](https://docs.python.org/3/library/asyncio.html), Crawlee ofrece un mejor rendimiento y una compatibilidad perfecta con otras bibliotecas asíncronas modernas.
- **Sugerencias de tipo** – Proyecto más nuevo construido con Python moderno y cobertura completa de sugerencias de tipo para una mejor experiencia de desarrollador.
- **Integración simple** – Los crawlers de Crawlee son scripts de Python regulares, que no requieren ningún ejecutor de lanzador adicional. Esta flexibilidad permite integrar un crawler directamente en otras aplicaciones.
- **Persistencia de estado** – Admite la persistencia del estado durante las interrupciones, ahorrando tiempo y costos al evitar la necesidad de reiniciar las canalizaciones de scraping desde cero después de un problema.
- **Almacenamiento de datos organizado** – Permite guardar múltiples tipos de resultados en una sola ejecución de scraping. Ofrece varias opciones de almacenamiento (ver [conjuntos de datos](https://crawlee.dev/python/api/class/Dataset) y [almacenes de clave-valor](https://crawlee.dev/python/api/class/KeyValueStore)).

## Ejecución en la plataforma Apify

Crawlee es de código abierto y se ejecuta en cualquier lugar, but since it's developed by [Apify](https://apify.com), it's easy to set up on the Apify platform and run in the cloud. Visit the [Apify SDK website](https://docs.apify.com/sdk/python/) to learn more about deploying Crawlee to the Apify platform.

## Soporte

Si encuentras algún error o problema con Crawlee, por favor [envía un issue en GitHub](https://github.com/apify/crawlee-python/issues). Para preguntas, puedes preguntar en [Stack Overflow](https://stackoverflow.com/questions/tagged/apify), en las Discusiones de GitHub o puedes unirte a nuestro [servidor de Discord](https://discord.com/invite/jyEM2PRvMU).

## Contribuciones

Tus contribuciones de código son bienvenidas, ¡y serás elogiado por la eternidad! Si tienes alguna idea de mejora, envía un issue o crea una pull request. Para las directrices de contribución y el código de conducta, consulta [CONTRIBUTING.md](https://github.com/apify/crawlee-python/blob/master/CONTRIBUTING.md).

## Licencia

Este proyecto está licenciado bajo la Licencia Apache 2.0 - consulta el archivo [LICENSE](https://github.com/apify/crawlee-python/blob/master/LICENSE) para más detalles.
