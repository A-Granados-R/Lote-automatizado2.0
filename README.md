# Lote-automatizado2.0

## Mejora y debug para el sistema de automatización 

La versión 1 tenía un problema. La IA de OpenAI alucinaba las respuestas y las inventaba. Las instrucciones en su prompt eran claras "contestar preguntas del inventario unicamente usando el tool de Google Sheets" cosa la cual hacía caso omiso en el 50% de las ejecuciones, provocando respuestas erroneas a los clientes. Ahora el problema a solucionar era, como modificar el workflow de manera que el agente mantuviera su habilidad de agendar citas, agregar clientes al CRM, pero quitarle creatividad de respuesta en las preguntas sobre los autos. He aquí la solución.

## Tabla de contenido 

## Funciones principales

- Chatbot inteligente con un LLM (OpenAI) entrenado para actuar como asesor de ventas.
- Integración con el número de Whatsapp del lote via UltraMsg.
- Consulta en tiempo real del inventario (vía Google Sheets).
- Registro automático de leads en HubSpot desde la conversación.
- Envío de recordatorios y eventos al calendario de Google.

## Tecnologías utilizadas

- [n8n (cloud)](https://n8n.io/)
- [OpenAI](https://platform.openai.com/)
- [UltraMSG](https://ultramsg.com/es/)
- [Telegram Bot API](https://core.telegram.org/bots)
- [HubSpot Developer Portal](https://developers.hubspot.com/)
- [Google Cloud Console](https://console.cloud.google.com/)
- Google Sheets / Docs / Calendar

## Arquitectura

### Main/Parent Workflow 

<img width="1128" height="651" alt="image" src="https://github.com/user-attachments/assets/75ede963-3e01-4f56-8091-9ea296275a7e" />

Inicia con un Webhook de UltraMSG al momento en que un mensaje es recibido al número del lote. Éste es procesado por el primer agente de IA con un prompt que le da las instrucciones de; 
- Responder a mensajes de clientes.
- Agendar visitas al lote.
- En cuanto el cliente pregunte por información de los vehículos, acudir directamente al sub-workflow. Nunca contestar él.
- Pedirle sus datos al cliente y agregarlos al CRM (Hubspot).

### IA/LLM
Utilizo "gpt-4o" como el modelo para el agente, modificandole la temperatura a 0.7 para reducir la creatividad en sus respuestas y evitar que invente respuestas, asimismo reduje el Top P a 0.3 para reducir su amplitud de respuesta y apegarse solo al prompt. 

### Send Msg
Utilicé la acción de "HTTP Request" para que el flujo mande el mensaje directo a la conversación con el cliente, poniendo como METHOD-POST, y como URL el API de mi instancia de UltraMSG. Agregué 2 parametros personalizados, 1 para el numero al cual el flujo debe mandar el mensaje y otro para el body del mensaje de texto. 
- El Numero al que debe mandar lo puse como un texto JSON el cual extrae el numero del cliente directo del Webhook inicial
- El body de igual manera lo puse como Expresion JSON, el cual es el output de nuestro agente.

### Calendario
Tiene como acción unicamente "crear evento" directo a la cuenta Google del lote, con valores de "start", "end", attendees y descripcion puestos para ser definidos automaticamente por la IA

### CRM
Utilizando Hubspot como el sistema para el lote, el tool en el flujo sirve para crear o editar contactos dentro del mismo, haciendo que la IA agregue nombre, correo, numero, preferencias y descripcion del cliente.

## Sub-Workflow

<img width="1062" height="557" alt="image" src="https://github.com/user-attachments/assets/dcf20893-af82-4399-97d6-42871f980e8b" />

### Funcionamiento
El flow inicia al momento en que el parent le solicita, es decir, cada que se pida por información de inventario. La razón de este sub-flujo fue para resolver el problema de que el LLM se invente respuestas, aquí, el prompt del agente es simple; "Tu unica tarea es utilizar el tool de Google Sheets y solo responder con eso, no tiene mas conocimiento ni otra capacidad", utilizo el mismo gpt-4o pero ahora con una temperatura de 0.3 y un top p de 0.3 para reducir aún más su amplitud de respuesta. 

El HTTP request está configurado de la misma manera que el parent. 








