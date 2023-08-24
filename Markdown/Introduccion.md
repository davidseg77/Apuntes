# Introducción a Markdown


## 1. Sintaxis básica

Estos son los elementos descritos en el documento de diseño original de John Gruber. Todas las aplicaciones de Markdown admiten estos elementos.


| Element |	Markdown Syntax |
|---------|-----------------|
| Heading |	# H1, ## H2, ### H3 |
| Bold | **bold text** |
| Italic | *italicized text* |
| Blockquote | > blockquote |
| Ordered List | 1. First item, 2. Second item, 3. Third item |
| Unordered List | - First item, - Second item, - Third item |
| Code | `code` |
| Horizontal Rule |	--- |
| Link | [title](https://www.example.com) |
| Image | ![alt text](image.jpg) |


### 1.1 Imagénes

**Images**

1. Open the file containing the Linux mascot.
2. Marvel at its beauty.

```
    ![Tux, the Linux mascot](/assets/images/tux.png)
``` 

3. Close the file.


### 1.2 Enlaces

Para crear un enlace, incluya el texto del enlace entre paréntesis (por ejemplo, [Duck Duck Go]) y luego sígalo inmediatamente con la URL entre paréntesis (por ejemplo, (https://duckduckgo.com)).

``` 
My favorite search engine is [Duck Duck Go](https://duckduckgo.com).
``` 

La salida renderizada se ve así:

Mi motor de búsqueda favorito es Duck Duck Go


### 1.3 URL y direcciones de correo electrónico

Para convertir rápidamente una URL o dirección de correo electrónico en un enlace, enciérrelo entre corchetes angulares.

``` 
<https://www.markdownguide.org>
<fake@example.com>
``` 

La salida renderizada se ve así:

https://www.markdownguide.org
fake@example.com


### 1.4 Imágenes

Para agregar una imagen, agregue un signo de exclamación ( !), seguido de texto alternativo entre paréntesis y la ruta o URL al recurso de imagen entre paréntesis. Opcionalmente, puede agregar un título entre comillas después de la ruta o URL. Yo, por ejemplo, voy a agregar una de mi compañero Frodo:

``` 
![Frodo!](https://github.com/davidseg77/Apuntes/blob/main/Markdown/frodo.jpg "Frodo")
``` 

La imagen se mostrará de tal manera:

![Frodo!](https://github.com/davidseg77/Apuntes/blob/main/Markdown/frodo.jpg "Frodo")







