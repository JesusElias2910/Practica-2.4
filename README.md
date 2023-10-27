# Práctica-2.4

# 2.4 PicoW + OLED + Boton + CHATGTP
Depto de Sistemas y Computación  
Ing. En Sistemas Computacionales  
SISTEMAS PROGRAMABLES 23a  
Autor: Jesús Elías Martínez  
Fecha de entrega:   26/10/2023  
Objetivo: Desarrollar una interfaz de chat simple utilizando el microcontrolador Raspberry Pi Pico, una pantalla OLED, un botón, y el modelo de lenguaje GPT-3 (CHATGPT) para interactuar y obtener respuestas a preguntas o comandos ingresados por el usuario.

**Conexión física**   
![](Imagenes/fis.jpg)  

**Código de thonny**   
Lenguaje: Phyton
```
#Elías Martínez Jesús    20211768

#Importación de librerías
import json
import network
import time
import urequests
import ssd1306

#Configuración física
i2c = machine.I2C(0, sda=machine.Pin(0), scl=machine.Pin(1), freq=400000)
Pan_Oled = ssd1306.SSD1306_I2C(128, 64, i2c)
btn = machine.Pin(16, machine.Pin.IN, machine.Pin.PULL_UP)

#Función para la conexión
def GPT(ssid, password, endpoint, api_key, model, prompt, max_tokens):
    """
        Description: This is a function to hit chat gpt api and get
            a response.
        
        Parameters:
        
        ssid[str]: The name of your internet connection
        password[str]: Password for your internet connection
        endpoint[str]: API enpoint
        api_key[str]: API key for access
        model[str]: AI model (see openAI documentation)
        prompt[str]: Input to the model
        max_tokens[int]: The maximum number of tokens to
            generate in the completion.
        
        Returns: Simply prints the response
    """
    #Conexión a internet
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
    wlan.connect(ssid, password)
    
    #Tiempo de espera para conexión
    max_wait = 10
    while max_wait > 0:
      if wlan.status() < 0 or wlan.status() >= 3:
        break
      max_wait -= 1
      print('Esperando a conectar...')
      time.sleep(1)
    #Selectiva en caso de error
    if wlan.status() != 3:
       print(wlan.status())
       raise RuntimeError('Conexión de red fallida')
    else:
      print('Conectado con éxito')
      print(wlan.status())
      status = wlan.ifconfig()
    
    ##Formato
    headers = {'Content-Type': 'application/json',
               "Authorization": "Bearer " + api_key}
    data = {"model": model,
            "prompt": prompt,
            "max_tokens": max_tokens}
    
    print("Attempting to send Prompt")
    r = urequests.post("https://api.openai.com/v1/{}".format(endpoint),
                       json=data,
                       headers=headers)
    
    if r.status_code >= 300 or r.status_code < 200:
        print("There was an error with your request \n" +
              "Response Status: " + str(r.text))
    else:
        print("Success")
        response_data = json.loads(r.text)
        completion = response_data["choices"][0]["text"]
        print(completion)
        
        #Limpiar pantalla oled
        Pan_Oled.fill(0)

        #Separar el texto
        completion_text = completion
        max_chars_per_line = 15
        lines = [completion_text[i:i+max_chars_per_line] for i in range(0, len(completion_text), max_chars_per_line)]

        #Ciclo para mostrar texto
        for i, line in enumerate(lines):
            Pan_Oled.text(line, 0, i * 10)  #inicar con espacio

        Pan_Oled.show()

    r.close()

#Función para envío
def btn_callback(p):
    #Selectiva para envío
    if not btn.value():
        print("Botón presionado. Envío prompt...")
        Pan_Oled.fill(0)
        Pan_Oled.text("Envio", 0, 0)
        Pan_Oled.show()
        
        
        #Llmada a la función enciando parámetros
        GPT("Jesail",
                 "",
                 "completions",
                 "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",
                 "text-davinci-003",
                  "Give me a song name",
                 20)

#Al presionar el botón hacer la interrupción y ejecutar las funciones
btn.irq(trigger=machine.Pin.IRQ_FALLING, handler=btn_callback)

while True:
    #Principal
    pass
```

**Resultados**  
_Envío de solicutd a CHAT GPT_  
![](Imagenes/envio.jpg)  

_Resultado CHAT GPT_  
![](Imagenes/resu.jpg)   
