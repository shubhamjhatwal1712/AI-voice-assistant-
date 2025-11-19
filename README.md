#libraries
import os
import time
import pyautogui
import pyttsx3
import requests
import speech_recognition as sr
import datetime
import cv2
from requests import get
import wikipedia
import webbrowser
import pywhatkit
import smtplib
import sys
import operator
import pyjokes
import instaloader
from PyQt5 import QtCore, QtGui, QtWidgets
from PyQt5.QtCore import QTimer, QTime, QDate, Qt, QThread
from PyQt5.QtGui import QMovie
from PyQt5.QtWidgets import QMainWindow, QApplication
from jarvis import Ui_JARVISUI

engine = pyttsx3.init('sapi5')
voices = engine.getProperty('voices')
engine.setProperty('voices', voices[0].id)

# TEXT TO SPEECH
def speak(audio):
    engine.say(audio)
    print(audio)
    engine.runAndWait()

class MainThread(QThread):
    def __init__(self):
        super(MainThread, self).__init__()

    def run(self):
        self.TaskExecution()

    # TO CONVERT VOICE INTO TEXT
    def takecommand(self):
        r = sr.Recognizer()
        with sr.Microphone() as source:
            print("listening...")
            r.pause_threshold = 1
            audio = r.listen(source, timeout=3, phrase_time_limit=5)

            try:
                print("Recognizing...")
                query = r.recognize_google(audio, language='en-in')
                print(f"user said: {query}")

            except Exception as e:
                speak("Say that again please...")
                return "none"
            return query

    # TO WISH
    def wish(self):
        hour = int(datetime.datetime.now().hour)
        if hour >= 0 and hour <= 12:
            speak("Hello! A very good morning to you,sir, it's")
        elif hour >= 12 and hour < 18:
            speak("Hello! A very good afternoon to you,sir, it's")
        else:
            speak("Hello! A very good evening to you,sir, it's")
        tt = time.strftime("%I:%M %p")
        speak(tt)
        speak("I’m Jarvis sir, your intelligent companion. Please let me know what I can do for you.")

    # TO SEND EMAIL
    def sendEmail(self, to, content):
        server = smtplib.SMTP('smtp.gmail.com', 587)
        server.ehlo()
        server.starttls()
        server.login('your username', 'your password')
        server.sendmail('prakartijbd@gmail.com', to, content)
        server.close()

    def TaskExecution(self):
        self.wish()
        while True:
            query = self.takecommand().lower()

            if "open notepad" in query:
                npath = "C:\\Windows\\notepad.exe"
                os.startfile(npath)

            elif "open command prompt" in query:
                os.system("start cmd")

            elif "open camera" in query:
                cap = cv2.VideoCapture(0)
                while True:
                    ret, img = cap.read()
                    cv2.imshow('webcam', img)
                    if cv2.waitKey(50) == 27:
                        break
                cap.release()
                cv2.destroyAllWindows()

            elif "ip address" in query:
                ip = get('https://api.ipify.org').text
                speak(f"Your IP address is {ip}")

            # Corrected Wikipedia Code
            elif "wikipedia" in query:
                speak("Searching Wikipedia...")
                query = query.replace("wikipedia", "").strip()  # Remove 'wikipedia' and extra spaces
                try:
                    results = wikipedia.summary(query, sentences=6)
                    speak("According to Wikipedia")
                    speak(results)
                    print(results)
                except wikipedia.exceptions.DisambiguationError as e:
                    speak("There are multiple results for your search. Please be more specific.")
                    print(f"Disambiguation Error: {e.options}")
                except wikipedia.exceptions.PageError:
                    speak("Sorry, I couldn't find any page with that name.")
                except wikipedia.exceptions.WikipediaException as e:
                    speak("An error occurred while accessing Wikipedia.")
                    print(f"Wikipedia error: {e}")

            elif "open youtube" in query:
                webbrowser.open("www.youtube.com")

            elif "open gmail" in query:
                webbrowser.open("www.gmail.com")

            elif "open google" in query:
                speak("What should I search on Google?")
                cm = self.takecommand().lower()
                webbrowser.open(f"{cm}")

            elif "send message" in query:
                pywhatkit.sendwhatmsg("+918310559122", "This is Jarvis Pari", 22, 13)

            elif "play song on youtube" in query:
                speak("What should I play?")
                cy = self.takecommand().lower()
                pywhatkit.playonyt(f"{cy}")

            elif "send email to pari" in query:
                try:
                    speak("What should I type in the email?")
                    content = self.takecommand().lower()
                    to = "prakartijbd@gmail.com"
                    self.sendEmail(to, content)
                    speak("Email has been sent to Pari.")
                except Exception as e:
                    speak("Sorry sir, unable to send the Email")

            elif "you can sleep" in query:
                speak("Thanks for using me, sir. Have a great day.")
                sys.exit()

            elif "close notepad" in query:
                speak("Closing Notepad")
                os.system("taskkill /f /im notepad.exe")

            elif "tell me a joke" in query:
                joke = pyjokes.get_joke()
                speak(joke)

            elif "shut down the system" in query:
                os.system("shutdown /s /t 5")

            elif "restart the system" in query:
                os.system("shutdown /r /t 5")

            elif "sleep the system" in query:
                os.system("rundll32.exe powrprof.dll,SetSuspendState Sleep")

            elif "switch the window" in query:
                pyautogui.keyDown("alt")
                pyautogui.press("tab")
                time.sleep(1)
                pyautogui.keyUp("alt")

            elif "instagram profile" in query:
                speak("Please enter the username correctly")
                name = input("Enter username here: ")
                webbrowser.open(f"www.instagram.com/{name}")
                speak(f"Here is the profile of the user {name}.")
                time.sleep(5)
                speak("Would you like to download the profile picture of this account?")
                condition = self.takecommand().lower()
                if "yes" in condition:
                    mod = instaloader.Instaloader()
                    mod.download_profile(name, profile_pic_only=True)
                    speak("The profile picture is saved in the main folder.")
                else:
                    pass

            elif "take screenshot" in query:
                speak("Please tell me the name for this screenshot file.")
                name = self.takecommand().lower()
                speak("Hold the screen for a few seconds, I am taking the screenshot.")
                time.sleep(3)
                img = pyautogui.screenshot()
                img.save(f"{name}.png")
                speak("Screenshot is saved in the main folder.")


            elif "do some calculations" in query or "can you calculate" in query:
                r = sr.Recognizer()
                with sr.Microphone() as source:
                    speak("Tell me what you want me to calculate , example: 3 plus 3")
                    print("listening....")
                    r.adjust_for_ambient_noise(source)
                    audio = r.listen(source)
                my_string=r.recognize_google(audio)
                print(my_string)
                def get_operator_fn(op):
                    return{
                        '+': operator.add, #plus
                        '-': operator.sub, #minus
                        '*': operator.mul, #multiplied by
                        'divided' :operator.__truediv__,#divided
                        }[op]
                def eval_binary_expr(op1, oper, op2):
                    op1,op2 = int(op1), int(op2)
                    return get_operator_fn(oper)(op1, op2)
                speak("your result is")
                speak(eval_binary_expr(*(my_string.split())))

            elif "where am I" in query or "where are we" in query:
                speak("Let me check.")
                try:
                    ipAdd = requests.get('https://api.ipify.org').text
                    url = f'https://get.geojs.io/v1/ip/geo/{ipAdd}.json'
                    geo_requests = requests.get(url)
                    geo_data = geo_requests.json()

                    city = geo_data['city']
                    state = geo_data['region']
                    country = geo_data['country']

                    speak(f"Sir, I think we are in {city}, {state}, {country}.")
                except Exception as e:
                    speak("Sorry sir, I couldn't find the location due to a network issue.")

startExecution = MainThread()

class Main(QMainWindow):
    def __init__(self):
        super(Main, self).__init__()
        self.ui = Ui_JARVISUI()
        self.ui.setupUi(self)
        self.ui.pushButton.clicked.connect(self.startTask)
        self.ui.pushButton_2.clicked.connect(self.close)

    def startTask(self):
        self.ui.movie = QMovie("../../../Downloads/7LP8.gif")
        self.ui.label.setMovie(self.ui.movie)
        self.ui.movie.start()
        self.ui.movie = QMovie("../../../Downloads/Jarvis_Loading_Screen.gif")
        self.ui.label_2.setMovie(self.ui.movie)
        self.ui.movie.start()
        timer = QTimer(self)
        timer.timeout.connect(self.showTime)
        timer.start(1000)
        startExecution.start()

    def showTime(self):
        current_time = QTime.currentTime()
        current_date = QDate.currentDate()
        label_time = current_time.toString('hh:mm:ss')
        label_date = current_date.toString(Qt.ISODate)
        self.ui.textBrowser.setText(label_date)
        self.ui.textBrowser_2.setText(label_time)

app = QApplication(sys.argv)
Ui_JARVISUI = Main()
Ui_JARVISUI.show()
sys.exit(app.exec_())
main.py
Displaying main.py. 
