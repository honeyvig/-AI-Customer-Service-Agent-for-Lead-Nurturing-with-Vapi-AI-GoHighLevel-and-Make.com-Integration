# -AI-Customer-Service-Agent-for-Lead-Nurturing-with-Vapi-AI-GoHighLevel-and-Make.com-Integration
AI Customer Service Agent for Lead Nurturing with Vapi AI, GoHighLevel, and Make.com Integration

This script outlines the process for creating an AI-powered customer service agent that can be integrated with Vapi AI, GoHighLevel (GHL), and Make.com to handle lead follow-ups, appointment scheduling, and automated workflows. The core tasks involve building the AI agent, managing dynamic conversation pathways, and integrating with Google Calendar, SMS, WhatsApp, and email systems.

To integrate these technologies, Python can be used for automation, API interactions, and integrating workflows.

Below is the Python code outline for implementing this system:
1. Install Dependencies

First, you need to install the necessary libraries for interacting with the APIs.

pip install requests Flask twilio google-auth google-api-python-client

    requests: To interact with external APIs (Vapi AI, GoHighLevel, Make.com).
    Flask: A lightweight web framework for handling webhook requests.
    twilio: For SMS and WhatsApp messaging.
    google-auth and google-api-python-client: For Google Calendar API integration.

2. Define Helper Functions

We'll need functions to interact with Vapi AI, GoHighLevel (GHL), Google Calendar, and Make.com.
a. Vapi AI Interaction (AI Agent)

import requests

VAPI_AI_API_URL = "https://api.vapi.ai/agent"  # replace with the actual VAPI AI endpoint
VAPI_AI_API_KEY = "YOUR_VAPI_API_KEY"

def send_message_to_vapi_ai(user_message):
    """Send user message to Vapi AI and get the response."""
    headers = {
        'Authorization': f'Bearer {VAPI_AI_API_KEY}',
        'Content-Type': 'application/json'
    }
    
    payload = {
        'message': user_message,
        'user_id': 'unique_user_id'  # Use a unique user ID per customer
    }

    response = requests.post(VAPI_AI_API_URL, json=payload, headers=headers)
    return response.json()

b. Google Calendar Integration

from google.auth.transport.requests import Request
from google.oauth2.credentials import Credentials
from googleapiclient.discovery import build

def get_google_calendar_service(credentials_file='token.json'):
    """Authenticate and get Google Calendar API service."""
    creds = None
    if os.path.exists(credentials_file):
        creds = Credentials.from_authorized_user_file(credentials_file, ['https://www.googleapis.com/auth/calendar'])
    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(Request())
        else:
            # Assume Google OAuth2 flow here to get new credentials
            pass
    return build('calendar', 'v3', credentials=creds)

def create_google_calendar_event(event_details):
    """Create a Google Calendar event."""
    service = get_google_calendar_service()
    
    event = {
        'summary': event_details['title'],
        'location': event_details['location'],
        'description': event_details['description'],
        'start': {
            'dateTime': event_details['start_time'],
            'timeZone': 'America/Los_Angeles',
        },
        'end': {
            'dateTime': event_details['end_time'],
            'timeZone': 'America/Los_Angeles',
        },
        'attendees': [
            {'email': event_details['attendee_email']},
        ],
        'reminders': {
            'useDefault': True,
        },
    }

    created_event = service.events().insert(calendarId='primary', body=event).execute()
    return created_event

c. GoHighLevel Integration

GHL_API_URL = "https://api.gohighlevel.com/v1"  # Replace with the actual API endpoint
GHL_API_KEY = "YOUR_GHL_API_KEY"

def create_appointment_in_ghl(customer_data):
    """Create an appointment in GoHighLevel CRM."""
    headers = {
        'Authorization': f'Bearer {GHL_API_KEY}',
        'Content-Type': 'application/json'
    }
    
    payload = {
        'name': customer_data['name'],
        'phone': customer_data['phone'],
        'email': customer_data['email'],
        'date': customer_data['appointment_date'],
        'notes': customer_data['notes'],
    }
    
    response = requests.post(f'{GHL_API_URL}/appointments', json=payload, headers=headers)
    return response.json()

d. SMS and WhatsApp Integration (via Twilio)

from twilio.rest import Client

TWILIO_ACCOUNT_SID = 'YOUR_TWILIO_ACCOUNT_SID'
TWILIO_AUTH_TOKEN = 'YOUR_TWILIO_AUTH_TOKEN'
TWILIO_PHONE_NUMBER = 'YOUR_TWILIO_PHONE_NUMBER'

def send_sms_or_whatsapp(to, message, via_whatsapp=False):
    """Send SMS or WhatsApp message using Twilio."""
    client = Client(TWILIO_ACCOUNT_SID, TWILIO_AUTH_TOKEN)

    if via_whatsapp:
        to = f'whatsapp:{to}'
    message = client.messages.create(
        body=message,
        from_=f'whatsapp:{TWILIO_PHONE_NUMBER}' if via_whatsapp else TWILIO_PHONE_NUMBER,
        to=to
    )
    return message.sid

3. Automating Workflow and User Interaction

You will use Flask to create an API that listens for inbound or outbound lead triggers and responds based on the interaction with the AI agent.

from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/webhook', methods=['POST'])
def webhook():
    """Endpoint to receive lead information and trigger the AI agent conversation."""
    data = request.json
    user_message = data.get('message', '')
    
    # Interact with Vapi AI
    ai_response = send_message_to_vapi_ai(user_message)
    response_message = ai_response.get('response_text', 'I didn\'t understand that.')

    # Depending on the conversation, trigger an appointment creation or send messages
    if 'schedule appointment' in response_message.lower():
        # Assuming AI response triggers appointment scheduling
        customer_data = {
            'name': data['name'],
            'phone': data['phone'],
            'email': data['email'],
            'appointment_date': '2024-05-10T15:00:00',  # Example date
            'notes': 'Follow-up appointment'
        }

        # Create the appointment in GHL
        ghl_response = create_appointment_in_ghl(customer_data)

        # Send confirmation via SMS/WhatsApp
        send_sms_or_whatsapp(data['phone'], 'Your appointment has been scheduled.')
        return jsonify({'message': 'Appointment scheduled successfully!'}), 200
    
    return jsonify({'message': response_message}), 200

if __name__ == '__main__':
    app.run(debug=True)

4. Workflow Logic

    Trigger the workflow: When a lead is received (via email, form submission, or database entry), it triggers the Flask API.
    AI Conversation: The system passes the user’s message to the AI model, which determines the response. If needed, the AI suggests scheduling an appointment or responding via email, SMS, or WhatsApp.
    Integration with Google Calendar & GHL: If the conversation results in an appointment, the system integrates with Google Calendar to schedule the event and with GoHighLevel to track the lead.
    Notifications: Sends confirmation of actions (e.g., appointment confirmation) to users via SMS or WhatsApp using Twilio.

5. Testing & Deployment

    You can test this solution using tools like Postman or cURL to simulate user interactions and verify if the integrations work as expected.
    For deployment, consider using Heroku, AWS Lambda, or Google Cloud Functions to host the Flask application.

Final Thoughts:

This solution provides an intelligent AI customer service agent capable of nurturing leads, managing appointments, and automating workflows with seamless integration into platforms like Vapi AI, GoHighLevel, and Make.com. By leveraging these technologies, we can offer an innovative AI-driven service for your marketing agency’s clients to improve lead management and follow-up processes.
