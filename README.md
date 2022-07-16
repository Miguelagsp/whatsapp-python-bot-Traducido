# WhatsApp Bot
Demo Bot para api https://chat-api.com/en/swagger.html

# Oportunidades:
- La salida de la lista de comandos
- La salida del ID del chat actual
- La salida de la hora del servidor actual del bot que se está ejecutando
- La salida de su nombre
- Envío de archivos de diferentes formatos (PDF, jpg, doc, mp3, etc.)
- Envío de mensajes de voz pregrabados
- Envío de coordenadas geográficas (ubicaciones)
- Establecimiento de una conferencia (grupo)

Atención: Para que el bot sea totalmente funcional, por favor, mantenga siempre su teléfono en línea. Su teléfono no debe ser utilizado para el WhatsApp Web al mismo tiempo.

# Cómo empezar
Al principio, vamos a enlazar el WhatsApp con nuestro script de una vez para poder comprobar el funcionamiento del código mientras escribimos uno. Para ello, cambiaremos a nuestra cuenta y obtendremos allí un código QR. Luego abrimos WhatsApp en el móvil, vamos a Ajustes → WhatsApp Web → Escanear el código QR.

En este punto, se debe proporcionar la URL del WebHook para que el servidor active nuestro script para los mensajes entrantes. La URL del WebHook es un enlace al que se enviarán datos JSON con información sobre los mensajes o notificaciones entrantes mediante el método POST. Así que necesitamos un servidor para que el bot se ejecute y este servidor aceptará y procesará esta información. Mientras escribíamos este artículo, desplegamos el servidor utilizando el microframework FLASK. El servidor FLASK nos permite responder cómodamente a las peticiones entrantes y procesarlas.
> pip install flask

Luego clona el repositorio para ti.
A continuación, vaya a la **wabot.py** y cambie las variables APIUrl y token por las suyas de su gabinete personal
https://app.chat-api.com/instance/

Constructor de la clase, el predeterminado que aceptará JSON, que contendrá información sobre los mensajes entrantes (será recibido por WebHook y reenviado a la clase). Para ver cómo será el JSON recibido, puedes entrar en la sección de pruebas, muy fácil de usar, disponible en tu gabinete. También puedes probar las peticiones de WebHook en ella. https://app.chat-api.com/testing

Puedes ver la estructura JSON en la sección de pruebas del elemento "Check WebHook". Es necesario para ejecutar la comprobación y enviar mensajes a su chat de WhatsApp. Verá el JSON enviado al WebHook en la pantalla.

Copiamos este JSON y ejecutamos nuestro servidor FLASK local con la ayuda de un depurador en el editor de código o a través del
> flask run

Para emular la petición al servidor, necesitamos enviar una petición POST de JSON, que hemos copiado en el paso anterior. La solicitud se envía a la dirección localhost donde se ejecuta flask. Así, podemos emular las acciones de WebHook y probar la funcionalidad del bot.

# Funciones
## enviar_petición
Se utiliza para enviar solicitudes a la API del sitio
```python
 def send_requests(self, method, data):
        url = f"{self.APIUrl}{method}?token={self.token}"
        headers = {'Content-type': 'application/json'}
        answer = requests.post(url, data=json.dumps(data), headers=headers)
        return answer.json()
```
- **method** determina que se llame al método de la API de chat.
- **data** contiene los datos necesarios para el envío.


## enviar_mensaje
Se utiliza para enviar mensajes al chat de WhatsApp
```python
 def send_message(self, chatID, text):
        data = {"chatID" : chatID,
                "body" : text}  
        answer = self.send_requests('sendMessage', data)
        return answer
```
- ChatID – ID del chat al que se debe enviar el mensaje
- Text – Texto del mensaje

## mostrar_chat_id
Sirve para responder al comando "chatId". Envía el id de usuario al chat
```python
def show_chat_id(self,chatID):
        return self.send_message(chatID, f"Chat ID : {chatID}")
```
- ChatID – ID del chat al que se debe enviar el mensaje

## tiempo
Sirve para responder al comando "time". Envía la hora actual del servidor al chat.
```python
def time(self, chatID):
        t = datetime.datetime.now()
        time = t.strftime('%d:%m:%Y')
        return self.send_message(chatID, time)
```
- ChatID – ID del chat al que se debe enviar el mensaje

## archivo 
Sirve para responder al comando "file". Envía al archivo de chat, que se encuentra en el servidor en el formato especificado.
 ```python
def file(self, chatID, format):
        availableFiles = {'doc' : 'document.doc',
                        'gif' : 'gifka.gif',
                        'jpg' : 'jpgfile.jpg',
                        'png' : 'pngfile.png',
                        'pdf' : 'presentation.pdf',
                        'mp4' : 'video.mp4',
                        'mp3' : 'mp3file.mp3'}
        if format in availableFiles.keys():
            data = {
                        'chatId' : chatID,
                        'body': f'https://domain.com/Python/{availableFiles[format]}',                      
                        'filename' : availableFiles[format],
                        'caption' : f'Get your file {availableFiles[format]}'
                    }
            return self.send_requests('sendFile', data)
```
- chatID – ID del chat al que se debe enviar el mensaje
- format – es el formato del archivo a enviar. Todos los archivos que se envían se almacenan en el lado del servidor.

Qué son los *data*
- ChatID – ID del chat al que se debe enviar el mensaje
- Body – enlace directo al archivo que se va a enviar
- Filename – el nombre del archivo
- Caption – el texto que se enviará junto con el archivo

Generar solicitud con el **send_requests**  y el **“sendFile”** como parámetro; presentar nuestro *data* a ella.

## ptt
Se utiliza para enviar un mensaje de voz a una sala de chat.
```python
def ptt(self, chatID):        
            data = {
            "audio" : 'https://domain.com/Python/ptt.ogg',
            "chatId" : chatID }
            return self.send_requests('sendAudio', data)
```
- ChatID – ID del chat al que se debe enviar el mensaje

Qué es una *data*
- “audio”  – un enlace directo a un archivo en formato ogg
-  "chatID" – ID del chat al que se debe enviar el mensaje

Generar solicitud con el **send_requests**  y el **“sendAudio”** como parámetro; presentar nuestro *data* con ella.

## geo
Se utiliza para enviar la geolocalización a una sala de chat
def geo(self, chatID):
```python
        data = {
                "lat" : '51.51916',
                "lng" : '-0.139214',
                "address" :'Your address',
                "chatId" : chatID
        }
        answer = self.send_requests('sendLocation', data)
        return answer
```
- ChatID – ID del chat al que se debe enviar el mensaje

Qué es una *data*
- "сhatID" – ID del chat al que se debe enviar el mensaje
- “lat” – coordenadas predefinidas
- “lng” – coordenadas predefinidas
- “address” – esta es su dirección o cualquier cadena que necesite.

Generar solicitud con el **send_requests**  y el **“sendLocation”** como parámetro; presentar nuestro *data* con ella.

## grupo
Se utiliza para crear un grupo de bots y usuarios
```python
def group(self, author):
        phone = author.replace('@c.us', '')
        data = {
            "groupName"  :  'Group with the bot Python',
             "phones"  :  phone,
             "messageText"  :  'It is your group. Enjoy'
        }
        answer = self.send_requests('group', data)
        return answer
```
- author –  el cuerpo JSON enviado por el WebHook con información sobre el remitente del mensaje.

Que es una *data*
- “groupName” – el nombre de la conferencia cuando se crea
- “phones” – todos los números de teléfono de los participantes necesarios; también puede enviar una matriz de varios números de teléfono
- “messageText” – El primer mensaje de la conferencia

Generar solicitud con el **send_requests**  y el **“group”** como parámetro; presentar nuestro *data* con ella.

# Procesamiento de mensajes entrantes
```python
def processing(self):
        if self.dict_messages != []:
            for message in self.dict_messages:
                text = message['body'].split()
                if not message['fromMe']:
                    id  = message['chatId']
                    if text[0].lower() == 'hi':
                        return self.welcome(id)
                    elif text[0].lower() == 'time':
                        return self.time(id)
                    elif text[0].lower() == 'chatid':
                        return self.show_chat_id(id)
                    elif text[0].lower() == 'me':
                        return self.me(id, message['senderName'])
                    elif text[0].lower() == 'file':
                        return self.file(id, text[1])
                    elif text[0].lower() == 'ptt':
                        return self.ptt(id)
                    elif text[0].lower() == 'geo':
                        return self.geo(id)
                    elif text[0].lower() == 'group':
                        return self.group(message['author'])
                    else:
                        return self.welcome(id, True)
                else: return 'NoCommand'
```

Ahora necesitamos organizar la lógica del bot para que pueda reaccionar a los comandos e interactuar con el usuario también.
Llamaremos a esta función cada vez que recibamos datos en nuestro WebHook

Recuerdas el atributo de nuestro bot dict_messages que creamos al principio. Tiene los diccionarios de los mensajes que hemos aceptado. La prueba filtra los datos que no contienen ningún mensaje. El WebHook puede recibir una petición sin mensaje.
```python
 if self.dict_messages != []:
```

Podemos recibir varios mensajes en una sola petición, y nuestro bot debe procesarlos todos. Para ello, vamos a recorrer todos los diccionarios que tienen una lista dict_messages.

```python
para el mensaje en self.dict_messages:
                text = message['body'].split()
```
Declaramos una variable texto -que será una lista de palabras contenidas en nuestro mensaje- después de entrar en el bucle. Para ello, acudimos al diccionario de mensajes mediante la función ['body'] para obtener el texto del mensaje entrante y simplemente llamar a la función split(), que nos permite dividir el texto en palabras.

A continuación, comprobamos si el mensaje entrante no procede de nosotros mismos consultando el 'fromMe' que contiene valores Verdadero o Falso y valida de quién era el mensaje. Si no se realiza esta prueba, el bot puede convertirse en una recursión infinita.

```python
 if not message['fromMe']:
```

Ahora obtenemos el identificador del chat desde el mismo diccionario de mensajes utilizando la función ['ChatID'] clave. Nos dirigimos al primer elemento de la lista de palabras, lo ponemos en minúsculas para que el bot pueda reaccionar a los mensajes escritos con MAYÚSCULAS o MiXeD cAsE y luego lo comparamos con los comandos que necesitamos. A continuación, sólo hay que analizar qué comando vino y llamar a las funciones correspondientes 
```python
if text[0].lower() == 'hi':
                        return self.welcome(id)
                    elif text[0].lower() == 'time':
                        return self.time(id)
                    elif text[0].lower() == 'chatid':
                        return self.show_chat_id(id)
                    elif text[0].lower() == 'me':
                        return self.me(id, message['senderName'])
                    elif text[0].lower() == 'file':
                        return self.file(id, text[1])
                    elif text[0].lower() == 'ptt':
                        return self.ptt(id)
                    elif text[0].lower() == 'geo':
                        return self.geo(id)
                    elif text[0].lower() == 'group':
                        return self.group(message['author'])
                    else:
                        return self.welcome(id, True)
                else: return 'NoCommand'
```

# Frasco 
Para procesar las peticiones entrantes a nuestro servidor utilizamos esta función 
```python
@app.route('/', methods=['POST'])
def home():
    if request.method == 'POST':
        bot = WABot(request.json)
        return bot.processing()
```

Escribiremos la ruta app.route('/', methods = ['POST']) para ello. Este decorador significa que nuestra función de inicio será llamada cada vez que nuestro servidor FLASK sea accedido a través de una petición de correo por la ruta principal.

Estamos comprobando que se ha accedido al servidor a través del método POST. Ahora creamos una instancia de nuestro bot y le enviamos datos JSON. Así que ahora sólo llamamos al método bot.processing() que gestiona las peticiones de nuestro objeto.


# Traducido por Miguelagsp
