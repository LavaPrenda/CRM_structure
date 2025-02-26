# 🤖 Bot de WhatsApp Business para LavaPrenda

![Banner](https://via.placeholder.com/800x200?text=Lavanderia+WhatsApp+Bot)

Un bot completo para la automatización de reservas, consultas y pagos para negocios de lavandería y renta de lavadoras. Optimiza tus operaciones y mejora la experiencia de tus clientes.

[![Python](https://img.shields.io/badge/Python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![Twilio](https://img.shields.io/badge/Twilio-API-red.svg)](https://www.twilio.com/)
[![Heroku](https://img.shields.io/badge/Heroku-Deploy-purple.svg)](https://heroku.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## 📑 Índice

- [Introducción](#introducción)
- [Funcionalidades](#funcionalidades)
- [Requisitos Previos](#requisitos-previos)
- [Instalación y Configuración](#instalación-y-configuración)
- [Guía Paso a Paso](#guía-paso-a-paso)
- [Deployment](#deployment)
- [Monitoreo y Optimización](#monitoreo-y-optimización)
- [Soporte y Contribuciones](#soporte-y-contribuciones)
- [Licencia](#licencia)

## 🌟 Introducción

Este proyecto proporciona una solución completa para implementar un bot de WhatsApp Business específicamente diseñado para negocios de lavandería y servicios de renta de lavadoras. El bot permite la automatización de procesos como consultas de precios, reservas de equipos y gestión de pagos, mejorando significativamente la experiencia del cliente y optimizando las operaciones del negocio.

## ⚙️ Funcionalidades

- **Respuestas automáticas a consultas frecuentes**
- **Sistema de reservas de lavadoras**
- **Procesamiento de pagos integrado con Stripe**
- **Base de datos para seguimiento de clientes y reservas**
- **Integración con IA (Dialogflow) para respuestas inteligentes**
- **Panel de métricas para analizar el rendimiento**

## 📋 Requisitos Previos

- Cuenta de WhatsApp Business
- Cuenta de Twilio
- Python 3.10 o superior
- Cuenta de Stripe (para procesamiento de pagos)
- Cuenta de Heroku (para despliegue)

## 🔧 Instalación y Configuración

### Dependencias de Python

```bash
pip install flask twilio python-dotenv requests sqlite3 stripe gunicorn
```

### Estructura del Proyecto

```
mi_bot/
├── app.py             # Aplicación principal
├── .env               # Variables de entorno
├── reservas.db        # Base de datos SQLite
├── Procfile           # Para deployment en Heroku
├── requirements.txt   # Dependencias
├── templates/         # Plantillas HTML
└── static/            # Archivos estáticos (CSS, JS)
```

## 📖 Guía Paso a Paso

### 1. Preparación Inicial

#### 1.1 Crear Cuenta de WhatsApp Business

1. Descarga la app WhatsApp Business desde Google Play o App Store
2. Regístrate con el número de teléfono de tu negocio
3. Completa tu perfil:
   - Nombre del negocio (Ej: "Lavandería Express")
   - Dirección y horarios
   - Descripción del servicio

#### 1.2 Registrarse en Twilio

1. Entra a [Twilio](https://www.twilio.com/) y crea una cuenta
2. Elige el plan "Pay-As-You-Go"
3. En la consola, localiza y guarda:
   - Account SID
   - Auth Token

#### 1.3 Activar Sandbox de WhatsApp

1. En Twilio Console, navega a WhatsApp > Sandbox
2. Escanea el código QR con tu WhatsApp Business
3. Verifica que aparezca "Your phone is connected"

### 2. Configuración del Entorno de Desarrollo

#### 2.1 Instalar Python

1. Descarga [Python 3.10+](https://www.python.org/downloads/)
2. Durante la instalación, marca "Add Python to PATH"
3. Verifica la instalación en la terminal:
   ```bash
   python --version
   ```

#### 2.2 Clonar Este Repositorio

```bash
git clone https://github.com/tu-usuario/bot-whatsapp-lavanderia.git
cd bot-whatsapp-lavanderia
```

#### 2.3 Configurar Variables de Entorno

Crea un archivo `.env` con las siguientes variables:

```
TWILIO_ACCOUNT_SID=tu_sid
TWILIO_AUTH_TOKEN=tu_token
STRIPE_API_KEY=tu_key_stripe
```

### 3. Desarrollo del Bot

#### 3.1 Código Base del Bot

El archivo `app.py` contiene el código principal:

```python
from flask import Flask, request
from twilio.twiml.messaging_v2 import MessagingResponse
from dotenv import load_dotenv
import os
import sqlite3

load_dotenv()
app = Flask(__name__)

# Crear tabla de reservas
def crear_tabla():
    conn = sqlite3.connect("reservas.db")
    conn.execute("""CREATE TABLE IF NOT EXISTS reservas (
                    id INTEGER PRIMARY KEY,
                    cliente TEXT,
                    modelo TEXT,
                    fecha DATE)""")
    conn.close()

crear_tabla()

@app.route("/webhook", methods=["POST"])
def bot():
    user_msg = request.form.get("Body", "").lower()
    resp = MessagingResponse()

    if "hola" in user_msg:
        respuesta = "¡Hola! ¿En qué puedo ayudarte? Opciones:\n1. Rentar lavadora\n2. Consultar precios"
    elif "1" in user_msg:
        respuesta = "Elige modelo:\nA) Básica ($10/día)\nB) Premium ($20/día)"
    elif "a" in user_msg:
        respuesta = "✅ ¡Reserva exitosa! Envía *pagar* para completar el proceso."
    else:
        respuesta = "Escribe *hola* para comenzar."

    resp.message(respuesta)
    return str(resp)

if __name__ == "__main__":
    app.run(debug=True)
```

#### 3.2 Probar Localmente con Ngrok

1. Descarga [Ngrok](https://ngrok.com/download)
2. Ejecuta Flask:
   ```bash
   python app.py
   ```
3. En otra terminal, inicia Ngrok:
   ```bash
   ./ngrok http 5000
   ```
4. Copia la URL HTTPS generada
5. En Twilio Sandbox, configura el webhook:
   ```
   https://tu-url-ngrok.io/webhook
   ```

### 4. Funcionalidades Avanzadas

#### 4.1 Integración de Pagos con Stripe

Agrega el siguiente código a `app.py`:

```python
import stripe
stripe.api_key = os.getenv("STRIPE_API_KEY")

# Dentro del flujo de reserva
if "pagar" in user_msg:
    checkout_session = stripe.checkout.Session.create(
        payment_method_types=["card"],
        line_items=[{
            "price_data": {
                "currency": "usd",
                "product_data": {"name": "Lavadora Básica"},
                "unit_amount": 1000,
            },
            "quantity": 1,
        }],
        mode="payment",
        success_url="https://tudominio.com/success",
    )
    respuesta = f"🔄 Paga aquí: {checkout_session.url}"
```

#### 4.2 Respuestas Inteligentes con Dialogflow

1. Crea un agente en [Dialogflow](https://dialogflow.cloud.google.com/)
2. Entrena con frases comunes de lavandería
3. Integra con este código:

```python
# Código de integración con Dialogflow
# (Ver documentación completa en la wiki del proyecto)
```

## 🚀 Deployment

### Desplegar en Heroku

1. Crea un archivo `Procfile`:
   ```
   web: gunicorn app:app
   ```

2. Crea `requirements.txt`:
   ```bash
   pip freeze > requirements.txt
   ```

3. Inicializa git y sube a Heroku:
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   heroku create tu-app-lavanderia
   git push heroku master
   ```

4. Configura las variables de entorno en Heroku:
   ```bash
   heroku config:set TWILIO_ACCOUNT_SID=tu_sid
   heroku config:set TWILIO_AUTH_TOKEN=tu_token
   heroku config:set STRIPE_API_KEY=tu_key_stripe
   ```

5. Configura el webhook en Twilio con tu URL de Heroku:
   ```
   https://tu-app-lavanderia.herokuapp.com/webhook
   ```

## 📊 Monitoreo y Optimización

### Configurar Métricas con Metabase

1. Instala [Metabase](https://www.metabase.com/)
2. Conecta tu base de datos SQLite
3. Crea un dashboard con:
   - Reservas por día
   - Tasa de conversión
   - Ingresos mensuales

## 🤝 Soporte y Contribuciones

- **Documentación completa**: [Wiki del Proyecto](https://github.com/tu-usuario/bot-whatsapp-lavanderia/wiki)
- **Soporte Técnico**: Únete a nuestro [Canal de Discord](https://discord.gg/tu-discord)
- **Contribuciones**: Las pull requests son bienvenidas. Para cambios importantes, abre un issue primero.

## 📄 Licencia

Este proyecto está bajo la Licencia MIT - ver el archivo [LICENSE](LICENSE) para más detalles.

---

Desarrollado con ❤️ para la comunidad de emprendedores de lavanderías.

![Footer](https://via.placeholder.com/800x100?text=WhatsApp+Bot+Lavanderias)
