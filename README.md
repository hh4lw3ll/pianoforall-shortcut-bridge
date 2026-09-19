# PianoForAll: abrir libros desde Notion

Puente estático para convertir un enlace HTTPS aceptado por Notion en una ejecución de Apple Shortcuts que abre el libro elegido en Apple Books. El sitio no usa servidor, dependencias, analítica ni credenciales.

## URL y uso

Página publicada: <https://hh4lw3ll.github.io/pianoforall-shortcut-bridge/>

Para abrir un libro con el Shortcut general:

<https://hh4lw3ll.github.io/pianoforall-shortcut-bridge/?book=Bk1-Pt1>

El parámetro `book` acepta el patrón `BkN-PtN`, por ejemplo `Bk1-Pt2` o `Bk2-Pt1`. El valor se codifica como texto de entrada del Shortcut. Si falta el parámetro, el puente conserva el comportamiento previo y ejecuta `PianoForAll — Bk1-Pt1`. Un formato distinto muestra un error y no ejecuta ningún Shortcut.

## Por qué hace falta HTTPS

Las pruebas reales en Notion mostraron que un enlace normal, la acción `Open page or URL` de un Button y un enlace creado con la fórmula `link()` no ejecutaban directamente el esquema `shortcuts://`. La URL web HTTPS sí fue aceptada por el botón. GitHub Pages sirve esta página como HTTPS; el JavaScript del puente solicita después a macOS abrir Shortcuts.

Safari o macOS pueden mostrar una confirmación al abrir Shortcuts. El recorrido parametrizado se probó en Safari en el Mac y el usuario confirmó que Books mostró el libro solicitado.

## Shortcut fijo validado como fallback

Nombre exacto: `PianoForAll — Bk1-Pt1`.

Acciones observadas y confirmadas en el flujo funcional:

1. `Find Books` con filtro `Title is Bk1-Pt1`.
2. Límite de resultados: `1`.
3. `Open Specific Book [Book]`.

El esquema que el puente fijo ejecuta es:

`shortcuts://run-shortcut?name=PianoForAll%20%E2%80%94%20Bk1-Pt1`

El usuario validó el recorrido fijo completo: Notion → botón `📖 Open in Books` → GitHub Pages → Shortcut fijo → Apple Books → `Bk1-Pt1`. El atajo fijo se conserva como fallback; la URL de Pages sin parámetro sigue llamándolo.

## Shortcut general en macOS

Se duplicó el atajo fijo y se renombró la copia `PianoForAll — Open Book`. El atajo original se conservó intacto. El general recibe texto como `Shortcut Input` y ejecuta estas acciones:

1. `Find Books`, filtro `Title is Shortcut Input` (la variable de entrada de texto recibida por el atajo).
2. `Limit`: activar el límite y poner `1`.
3. `Open Specific Book` usando el resultado `Book` de `Find Books`.

La prueba parametrizada en Safari confirmó que esta URL abre `Bk1-Pt1` en Apple Books:

`https://hh4lw3ll.github.io/pianoforall-shortcut-bridge/?book=Bk1-Pt1`

El esquema que genera el puente es:

`shortcuts://run-shortcut?name=PianoForAll%20%E2%80%94%20Open%20Book&input=text&text=Bk1-Pt1`

Apple documenta la ejecución de Shortcuts mediante `shortcuts://run-shortcut` y la entrega de una cadena codificada con `input=text&text=…`: [Run a shortcut using a URL scheme on Mac](https://support.apple.com/en-au/guide/shortcuts-mac/apd624386f42/mac).

## Configurar el botón en Notion

En la lección, dentro de `STUDY MATERIAL`, editar el botón `📖 Open in Books`, añadir acción `Open` y elegir la URL HTTPS. Para `Bk1-Pt1` usar:

`https://hh4lw3ll.github.io/pianoforall-shortcut-bridge/?book=Bk1-Pt1`

La URL anterior, sin `?book=`, sigue siendo el fallback al Shortcut fijo. El botón `📖 Open in Books` ya se probó en Notion con la URL sin parámetro. Para usar el atajo general desde Notion, configurar la acción `Open` con la URL parametrizada correspondiente a esa lección. Cada botón requiere su valor `book`; esta implementación no lo deriva automáticamente de una propiedad de Notion.

## Añadir futuros libros

Para cada lección se reutiliza el mismo puente y Shortcut, cambiando sólo el parámetro. Ejemplos:

- `.../?book=Bk1-Pt2`
- `.../?book=Bk2-Pt1`

El texto debe coincidir exactamente con el título que Shortcuts encuentra en Books. No se necesita otro repositorio, página o Shortcut por libro.

## Código y límites de prueba

`index.html` valida el identificador, codifica el nombre del Shortcut con `%20` y el libro con `encodeURIComponent`, y muestra un enlace manual de respaldo. Sin parámetro usa el Shortcut fijo. La prueba inicial mostró que Shortcuts interpretaba literalmente los `+` que `URLSearchParams` usaba en el nombre; se corrigió a `%20` en el commit `5378003` y el despliegue de GitHub Pages terminó correctamente.

Pruebas confirmadas: flujo fijo desde Notion hasta Books; URL parametrizada desde Safari hasta Books; formato de URL esperado y filtro `Shortcut Input` configurados. La ruta parametrizada desde el Button de Notion todavía requiere poner en ese Button la URL con `?book=` y probarla desde la lección. La guía de Apple enlazada arriba documenta `shortcuts://run-shortcut` y el texto de entrada.
