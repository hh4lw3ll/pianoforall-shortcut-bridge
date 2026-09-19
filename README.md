# PianoForAll: abrir libros desde Notion

Puente estático para convertir un enlace HTTPS aceptado por Notion en una ejecución de Apple Shortcuts que abre el libro elegido en Apple Books. El sitio no usa servidor, dependencias, analítica ni credenciales.

## URL y uso

Página publicada: <https://hh4lw3ll.github.io/pianoforall-shortcut-bridge/>

Para abrir un libro con el Shortcut general:

<https://hh4lw3ll.github.io/pianoforall-shortcut-bridge/?book=Bk1-Pt1>

El parámetro `book` acepta el patrón `BkN-PtN`, por ejemplo `Bk1-Pt2` o `Bk2-Pt1`. El valor se codifica como texto de entrada del Shortcut. Si falta el parámetro, el puente conserva el comportamiento previo y ejecuta `PianoForAll — Bk1-Pt1`. Un formato distinto muestra un error y no ejecuta ningún Shortcut.

## Por qué hace falta HTTPS

Las pruebas reales en Notion mostraron que un enlace normal, la acción `Open page or URL` de un Button y un enlace creado con la fórmula `link()` no ejecutaban directamente el esquema `shortcuts://`. La URL web HTTPS sí fue aceptada por el botón. GitHub Pages sirve esta página como HTTPS; el JavaScript del puente solicita después a macOS abrir Shortcuts.

Esto no elimina los avisos o confirmaciones que Safari/macOS puedan presentar. Que la página esté publicada no demuestra por sí solo la ejecución local: la prueba final requiere abrir el enlace en Safari en el Mac y comprobar el libro en Books.

## Shortcut fijo validado como fallback

Nombre exacto: `PianoForAll — Bk1-Pt1`.

Acciones observadas y confirmadas en el flujo funcional:

1. `Find Books` con filtro `Title is Bk1-Pt1`.
2. Límite de resultados: `1`.
3. `Open Specific Book [Book]`.

El esquema que el puente fijo ejecuta es:

`shortcuts://run-shortcut?name=PianoForAll%20%E2%80%94%20Bk1-Pt1`

El usuario validó el recorrido completo: Notion → botón `📖 Open in Books` → GitHub Pages → Shortcut fijo → Apple Books → `Bk1-Pt1`. Mantener este atajo como fallback hasta probar el general en el Mac.

## Crear el Shortcut general en macOS

Shortcuts debe recibir texto como `Shortcut Input`. Duplicar el atajo fijo o crear uno nuevo llamado exactamente `PianoForAll — Open Book`; no renombrar ni borrar el atajo fijo.

Configurar sus acciones así:

1. `Find Books`, filtro `Title is Shortcut Input` (la variable de entrada de texto recibida por el atajo).
2. `Limit`: activar el límite y poner `1`.
3. `Open Specific Book` usando el resultado `Book` de Find Books.

Guardar el atajo. Antes de conectarlo a Notion, probar en Shortcuts pasando `Bk1-Pt1` como entrada y confirmar que Books abre ese libro. Si la búsqueda no encuentra una coincidencia, revisar que el título de Books sea idéntico.

Cuando esté creado, probar desde Safari esta URL:

`https://hh4lw3ll.github.io/pianoforall-shortcut-bridge/?book=Bk1-Pt1`

El esquema resultante esperado es equivalente a:

`shortcuts://run-shortcut?name=PianoForAll%20%E2%80%94%20Open%20Book&input=text&text=Bk1-Pt1`

Apple documenta la ejecución de Shortcuts mediante `shortcuts://run-shortcut` y la entrega de una cadena codificada con `input=text&text=…`: [Run a shortcut using a URL scheme on Mac](https://support.apple.com/en-au/guide/shortcuts-mac/apd624386f42/mac).

## Configurar el botón en Notion

En la lección, dentro de `STUDY MATERIAL`, editar el botón `📖 Open in Books`, añadir acción `Open` y elegir la URL HTTPS. Para `Bk1-Pt1` usar:

`https://hh4lw3ll.github.io/pianoforall-shortcut-bridge/?book=Bk1-Pt1`

La URL anterior, sin `?book=`, sigue siendo el fallback al Shortcut fijo. En la prueba inicial, Notion aceptó esa URL HTTPS y el usuario confirmó que funcionó hasta Books. El botón puede requerir una URL distinta por lección, ya que esta implementación no deriva automáticamente el libro de una propiedad de Notion.

## Añadir futuros libros

Con `PianoForAll — Open Book` ya probado, para cada lección se puede reutilizar el mismo puente cambiando sólo el parámetro. Ejemplos:

- `.../?book=Bk1-Pt2`
- `.../?book=Bk2-Pt1`

El texto debe coincidir exactamente con el título que Shortcuts encuentra en Books. No se necesita otro repositorio, página o Shortcut por libro.

## Código y límites de prueba

`index.html` valida el identificador, construye el enlace de Shortcuts mediante `URLSearchParams` y muestra un enlace manual de respaldo. Sin parámetro usa el Shortcut fijo. El cambio se guardó en GitHub con el commit `2b152b1`.

Se verificó en el editor de GitHub el contenido guardado y el patrón del código. La ejecución del JavaScript en Pages, su despliegue tras el commit y la ruta al Shortcut general todavía requieren comprobación real en el Mac; el Shortcut general aún no está creado. No describir la generalización como validada hasta completar esa prueba extremo a extremo. La URL scheme es conforme con la guía de Apple enlazada arriba.
