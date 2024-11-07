# Aplicación simple de LLM con LangChain
## Visión general
En este tutorial, te guiaremos a través de la construcción de una aplicación simple de LLM con
LangChain que traduce texto del inglés a otro idioma. Esta aplicación demostrará cómo usar las
características de LangChain como:
- Uso de modelos de lenguaje
- Uso de Plantillas de Prompts y Parsers de Salida
- Uso del Lenguaje de Expresiones LangChain (LCEL) para encadenar componentes
- Depuración y seguimiento de tu aplicación con LangSmith
- Desplegar tu aplicación con LangServe
## Configuración
### Jupyter Notebook
Este tutorial usa Jupyter notebooks, una excelente herramienta para aprender sistemas LLM ya que
permite depuración interactiva cuando las cosas no salen bien. Para ejecutar este tutorial en
Jupyter, instala Jupyter con:
```bash
pip install notebook
```
O, si usas Anaconda:
```bash
conda install notebook
```
### Instalación
Instala LangChain usando `pip` o `conda`:
- **Pip**:
```bash
pip install langchain
```
- **Conda**:
Sigue las instrucciones de instalación [aquí](https://langchain.com/docs/installation/).
### LangSmith
LangSmith ayuda a rastrear y depurar tus aplicaciones LangChain, permitiéndote inspeccionar lo
que está ocurriendo dentro de tus cadenas. Después de registrarte, configura las variables de
entorno para el rastreo:
```bash
export LANGCHAIN_TRACING_V2="true"
export LANGCHAIN_API_KEY="tu_api_key"
```
O, en Jupyter:
```python
import getpass
import os
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```
## Usando Modelos de Lenguaje
LangChain soporta muchos modelos de lenguaje, incluyendo OpenAI, Anthropic, Azure, Google,
Cohere, entre otros. Para usar los modelos GPT de OpenAI:
1. Instala el paquete necesario:
```bash
pip install -qU langchain-openai
```
2. Configura tu clave de API de OpenAI:
```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = "llave_api_openai"
```
3. Inicializa el modelo y realiza una traducción:
```python
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage, SystemMessage
model = ChatOpenAI(model="gpt-4")
messages = [
SystemMessage(content="Translate the following from English into Italian"),
HumanMessage(content="hi!"),
]
response = model.invoke(messages)
print(response)
```
## Parsers de Salida
Para extraer solo la cadena de respuesta del modelo, usa un `OutputParser`. Por ejemplo, el
`StrOutputParser` extrae la respuesta en cadena:
```python
from langchain_core.output_parsers import StrOutputParser
parser = StrOutputParser()
result = model.invoke(messages)
parsed_response = parser.invoke(result)
print(parsed_response)
```
## Plantillas de Prompts
Para facilitar la construcción de prompts, puedes usar `PromptTemplates` para formatear la entrada
del usuario. Aquí te mostramos cómo crear una plantilla de prompt que traduzca texto a un idioma
especificado:
```python
from langchain_core.prompts import ChatPromptTemplate
system_template = "Translate the following into {language}:"
prompt_template = ChatPromptTemplate.from_messages([
('system', system_template),
('user', '{text}')
])
result = prompt_template.invoke({"language": "italian", "text": "hi"})
print(result.to_messages())
```
## Encadenando Componentes con LCEL
Ahora, encadena el modelo, la plantilla de prompt y el parser de salida usando LangChain
Expression Language (LCEL):
```python
chain = prompt_template | model | parser
response = chain.invoke({"language": "italian", "text": "hi"})
print(response) # Output: 'ciao'
```
## Desplegando con LangServe
### Instalación
Para desplegar tu aplicación, instala `LangServe`:
```bash
pip install "langserve[all]"
```
### Crea un Servidor FastAPI
Crea un archivo `serve.py` para servir tu aplicación LangChain:
```python
from fastapi import FastAPI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_openai import ChatOpenAI
from langserve import add_routes
# Define la plantilla de prompt, modelo y parser
system_template = "Translate the following into {language}:"
prompt_template = ChatPromptTemplate.from_messages([
('system', system_template),
('user', '{text}')
])
model = ChatOpenAI()
parser = StrOutputParser()
# Crea la cadena
chain = prompt_template | model | parser
# Aplicación FastAPI
app = FastAPI(
title="LangChain Server",
version="1.0",
description="Una aplicación simple de API usando las interfaces Runnable de LangChain"
)
# Añade las rutas
add_routes(app, chain, path="/chain")
if __name__ == "__main__":
import uvicorn
uvicorn.run(app, host="localhost", port=8000)
```
Ejecuta tu servidor con:
```bash
python serve.py
```
Tu aplicación estará disponible en `http://localhost:8000/chain`.
### Playground
LangServe proporciona una interfaz simple para interactuar con tu aplicación. Accede a ella en:
```
http://localhost:8000/chain/playground/
```
Puedes probar la traducción pasando los datos `{"language": "italian", "text": "hi"}`.
### Cliente
Para interactuar con el servidor de manera programática, usa el cliente `RemoteRunnable`:
```python
from langserve import RemoteRunnable
remote_chain = RemoteRunnable("http://localhost:8000/chain/")
response = remote_chain.invoke({"language": "italian", "text": "hi"})
print(response) # Output: 'ciao'
```
### pruebas de funcionalidad:
![image](https://github.com/user-attachments/assets/c96919ea-e7f9-41b5-9b2c-9c8ba164c099)

![image](https://github.com/user-attachments/assets/2ddb854d-7d8c-4fba-8c17-d2b5520c6181)

![image](https://github.com/user-attachments/assets/2da306f7-c514-48d4-a61e-26c01efca3e7)

![image](https://github.com/user-attachments/assets/ae8d99a5-0925-4eb3-9422-8167d411576d)

![image](https://github.com/user-attachments/assets/0b9c0fd5-b103-41ce-b299-858034b495c7)