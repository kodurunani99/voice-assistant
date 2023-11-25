import speech_recognition as sr
import pyttsx3
import datetime
import webbrowser
import os
import smtplib
import requests
from googletrans import Translator
import calendar
import wikipedia
import pyjokes
import time

def speak(text):
    engine = pyttsx3.init()
    voices = engine.getProperty('voices')
    engine.setProperty('voice', voices[1].id)
    engine.say(text)
    engine.runAndWait()

def listen():
    recognizer = sr.Recognizer()

    with sr.Microphone() as source:
        print("Listening...")
        recognizer.adjust_for_ambient_noise(source)
        audio = recognizer.listen(source)

    try:
        print("Recognizing...")
        query = recognizer.recognize_google(audio)
        print(f"User: {query}")
        return query.lower()
    except sr.UnknownValueError:
        print("Sorry, I did not hear your request. Please try again.")
        return ""
    except sr.RequestError as e:
        print(f"Error connecting to Google API: {e}")
        return ""

def get_wikipedia_info(query):
    query = query.replace("search Wikipedia for", "").strip()
    result = wikipedia.summary(query, sentences=1)
    speak(result)

def tell_joke():
    joke = pyjokes.get_joke()
    speak(joke)

def location_based_greeting(query):
    if "hello" in query and "in" in query:
        location = query.split("in ")[1]
        speak(f"Hello from {location}! How can I assist you today?")

def set_reminder(query):
    if "set a reminder" in query:
        speak("Sure, what would you like me to remind you about?")
        reminder_text = listen()
        speak("When should I remind you? For example, say 'in 5 minutes' or 'at 3 PM'")
        time_input = listen()

        try:
            time_value = int(time_input.split(" ")[1])
            time_unit = time_input.split(" ")[2].lower()

            if "minute" in time_unit:
                time.sleep(time_value * 60)
            elif "hour" in time_unit:
                time.sleep(time_value * 3600)

            speak(f"Reminder: {reminder_text}")
        except ValueError:
            speak("Sorry, I couldn't understand the time. Please try again.")

def get_weather(city):
    # Replace 'your_openweathermap_api_key' with a valid OpenWeatherMap API key
    api_key = 'your_openweathermap_api_key'
    base_url = f'http://api.openweathermap.org/data/2.5/weather?q={city}&appid={api_key}'
    response = requests.get(base_url)
    data = response.json()

    if data["cod"] != "404":
        main_data = data["main"]
        temperature = main_data["temp"]
        humidity = main_data["humidity"]
        weather_description = data["weather"][0]["description"]

        return f"The weather in {city} is {weather_description} with a temperature of {temperature}°C and {humidity}% humidity."
    else:
        return "City not found. Please check the city name and try again."

def send_email(receiver, subject, body):
    # Replace 'your_email' and 'your_password' with your email credentials
    sender_email = 'your_email@gmail.com'
    sender_password = 'your_password'

    message = f"Subject: {subject}\n\n{body}"

    try:
        with smtplib.SMTP('smtp.gmail.com', 587) as server:
            server.starttls()
            server.login(sender_email, sender_password)
            server.sendmail(sender_email, receiver, message)
        print("Email sent successfully.")
    except Exception as e:
        print(f"Error sending email: {str(e)}")
# ... (previous code)

def get_calendar_events():
    # Simulated calendar events
    events = ["Meeting at 10 AM", "Lunch with a friend at 1 PM"]
    return "Your upcoming events are:\n" + "\n".join(events)

def perform_math_calculation(expression):
    try:
        result = eval(expression)
        return f"The result of {expression} is: {result}"
    except Exception as e:
        return f"Error in calculating {expression}: {str(e)}"

# ... (remaining code)


def open_application(application):
    # You can customize this function to open specific applications based on your needs
    if "browser" in application:
        webbrowser.open("https://www.google.com")
    elif "music player" in application:
        # Open your preferred music player or replace with the appropriate command
        os.system("start spotify")
    elif "text editor" in application:
        # Open your preferred text editor or replace with the appropriate command
        os.system("start notepad.exe")
    else:
        print("Application not recognized.")

# ... (remaining code)

def assistant(query):
    if "hello" in query:
        speak("Hello! How can I assist you today?")
        location_based_greeting(query)
    elif "what is the time" in query:
        current_time = datetime.datetime.now().strftime("%H:%M:%S")
        speak(f"The current time is {current_time}")
    elif "open Google" in query:
        webbrowser.open("https://www.google.com")
    elif "play music" in query:
        music_folder = "Music"
        music_files = [f for f in os.listdir(music_folder) if f.endswith(".mp3")]
        if music_files:
            os.startfile(os.path.join(music_folder, music_files[0]))
        else:
            speak("No music files found in the specified folder.")
    elif "search" in query:
        search_query = query.replace("search", "").strip()
        webbrowser.open(f"https://www.google.com/search?q={search_query}")
    elif "weather" in query:
        city = query.split("weather in ")[1]
        speak(get_weather(city))
    elif "send email" in query:
        speak("To whom would you like to send an email?")
        receiver = listen()
        speak("What should be the subject of the email?")
        subject = listen()
        speak("What should be the body of the email?")
        body = listen()
        send_email(receiver, subject, body)
        speak("Email sent successfully.")
    elif "calendar events" in query:
        speak(get_calendar_events())
    elif "calculate" in query:
        expression = query.split("calculate ")[1]
        speak(perform_math_calculation(expression))
    elif "tell me a joke" in query:
        tell_joke()
    elif "search Wikipedia for" in query:
        get_wikipedia_info(query)
    elif "set a reminder" in query:
        set_reminder(query)
    elif "exit" in query or "quit" in query:
        speak("Goodbye! Have a great day.")
        exit()
    else:
        speak("I'm sorry, I didn't understand that.")

# ... (remaining code)

if __name__ == "__main__":
    speak("Welcome to your custom voice assistant!")

    while True:
        user_query = listen()
        if user_query:
            assistant(user_query)
