# Projecto RAG ➡️ Datos + LLM + Base de datos

# Índice

1. [Introducción](#introducción)  
2. [Instalaciones](#instalaciones)   
3. [Nuestros sistemas RAG](#nuestros-sistemas-rag)
4. [GUI RAG de pdf](#gui-rag-de-pdf)
5. [RAG Mongo Atlas](#rag-mongo-atlas)

## Introducción
Este proyecto tiene como objetivo crear un sistema RAG. Un sistema RAG está formado por datos (en nuestro caso los sacamos de dos formas: 
pdf y web) que, posteriormente, procesaremos, un modelo de lenguaje que nos sirva para podr responder a nuestras preguntas en base a esos datos y, finalmente,
un lugar en donde guardarlo (Chroma o Mongo DB)

---
## Instalaciones
Las instalaciones principales son:
  - Ollama. Un sistema que nos permite correr modelos de lenguaje como llama3.2, que es el que utilizaremos.
  - Open-webui. Un gestor gráfico con el que será más fácil instalar los modelos y gestionarlos, además de poder utilizarlos visualmente.

Primero de todo, crearemos una web en donde estarán estos dos sistemas dockerizados.
```console
docker network create ollama_network
```

### Ollama
Utilizaremos el siguiente comando:
```console
docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama --net=ollama_network ollama/ollama
```
### Open-webui
Uitlizaremos este comando:
```console
docker run -d -p 3000:8080 -e OLLAMA_BASE_URL=http://ollama:11434 -v open-webui:/app/backend/data --name open-webui --net=ollama_network --restart always ghcr.io/open-webui/open-webui:main
```
---
## Nuestros sistemas RAG

Hemos creado dos sitemas RAG, uno que leerá datos a partir de pdfs y otro que los leerá desde una página web (web scrapping). 
El funcionamiento es prácticamente el mismo: 
  > Recogida de los datos
  >> Procesado de los datos. Los datos se dividen varias veces y se transforman en vectores numéricos.
  >>> Guardado de vectores
  >>>> Preparamos el LLM con los vectores y un template
  >>>>> Preguntamos lo que queramos ⁉️

### ¿Cómo le damos un contexto?

  #### VS_PDF
  En el caso del pdf, puedes descargar uno o varios pdfs y posicionarlos en la carpeta **notebooks* del repositorio. El nombre del pdf deberás editarlo en el código para
  que el RAG escoja los que quieres.

  ```python
    # Cargar documentos desde los PDFs

    pdf_paths = ['naturaleza-yo.pdf'] # Aquí debes poner todos los pdfs que quieras
    documents = []
    
    for pdf_path in pdf_paths:
        loader = PyPDFLoader(pdf_path)
        documents.extend(loader.load())
  ```

  #### VS_web_scrapping
  En el caso de darle un contexto desde web, deberás darle un enlace web al que acceder.
  
  ```python
    # URL de la página web que deseas procesar
    url = "https://www.bbc.com/news/articles/c047r4kreg3o"  # Cambia esta URL por la que quieras
    
    response = requests.get(url)
    soup = BeautifulSoup(response.content, "html.parser")
  ```
> [!NOTE]
> Ten en cuenta que el LLM está configurado en inglés para este caso, por lo que la página debe ser también en inglés.

### ¿Cómo le preguntamos?
Para preguntar necesitas editar la variable 'pregunta' que hay al final del código. Aunque, con un simple cambio se podría configurar para
que lo puedas cambiar directamente desde el intérprete.

```python
# Ejemplo de uso
if __name__ == "__main__":
    
    # Realizar una consulta
    pregunta = "Sabes ya qué preguntar?" # Esta es tu pregunta
    print(f'Pregunta: {pregunta}')

    # Ejecutar la consulta
    print("Respuesta:")
    print(chain.invoke(pregunta))
```
---

## GUI RAG de pdf
Hemos mejorado el código RAG lector de pdfs para poder utilizarlo desde una interfaz gráfica.
El funcionamiento es el mismo:

> Subes todos los pdf. 📁
>> Se procesan los archivos.
>>> Haces tus preguntas.
>>>> Tienes tus respuestas. 🔤
---

## RAG Mongo Atlas
Como hemos dicho antes, los vectores se deben guardar en algún sitio. En los códigos de 📂GUI y 📂notebooks, esos vectores se guardan a través de la base de datos
vectorial Chroma en cada carpeta que se ejecuta el código.

Sin embargo, hemos mejorado un poco más el código y se ha implementado el guardado de vectoers en un cluster Mongo Atlas. De esta manera, no ocupa espacio en local
y lo podemos tener en un base de datos no relacional, incluso con redundancia de datos.

