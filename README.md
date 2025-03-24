# Verbatim

import streamlit as st
import speech_recognition as sr
import pyttsx3
import numpy as np
import sounddevice as sd
import noisereduce as nr
import wave
import threading

def noise_reduce(audio_data, sample_rate):
    #Apply noise reduction to improve speech recognition accuracy.
    reduced_audio = nr.reduce_noise(y=audio_data, sr=sample_rate)
    return reduced_audio

def recognize_speech(stop_event):
    recognizer = sr.Recognizer()
    with sr.Microphone() as source:
        st.write("Listening...")
        while not stop_event.is_set():
            audio = recognizer.listen(source)
            try:
                text = recognizer.recognize_google(audio, language='en-NG')
                st.session_state['transcription'] += text + " "
            except sr.UnknownValueError:
                st.session_state['transcription'] += "[Unrecognized Speech] "
            except sr.RequestError:
                st.session_state['transcription'] += "[Error: Connection Issue] "

def text_to_speech(text):
    engine = pyttsx3.init()
    engine.say(text)
    engine.runAndWait()

def start_recording():
    if 'stop_event' in st.session_state and st.session_state['stop_event'].is_set():
        st.session_state['stop_event'].clear()
    else:
        st.session_state['stop_event'] = threading.Event()
        st.session_state['transcription'] = ""
        threading.Thread(target=recognize_speech, args=(st.session_state['stop_event'],)).start()

def stop_recording():
    if 'stop_event' in st.session_state:
        st.session_state['stop_event'].set()

def main():
    st.set_page_config(page_title="Verbatim - Speech to Text", layout="wide")
    
    st.sidebar.title("Menu")
    st.sidebar.button("User Profile")
    st.sidebar.button("Settings")
    st.sidebar.button("Session History")
    
    st.title("Verbatim")
    st.write("Speech To Text Program Also Supports Nigerian English")
    
    col1, col2 = st.columns(2)
    
    with col1:
        if st.button("Start Transcription"):
            start_recording()
        if st.button("Stop Transcription"):
            stop_recording()
    
    with col2:
        if st.button("Read Aloud"):
            text_to_speech(st.session_state.get('transcription', ''))
    
    st.session_state['transcription'] = st.text_area("Transcribed Text:", st.session_state.get('transcription', ''), height=300)
    st.download_button("Download Transcript", st.session_state.get('transcription', ''), "transcription.txt")
    
if __name__ == "__main__":
    main()
import streamlit as st
import speech_recognition as sr
import pyttsx3
import time
import pyaudio

def recognize_speech():
    recognizer = sr.Recognizer()
    with sr.Microphone() as source:
        recognizer.adjust_for_ambient_noise(source)
        st.session_state['status'] = "Listening..."
        try:
            audio = recognizer.listen(source, timeout=5)
            text = recognizer.recognize_google(audio, language='en-NG')
            
            if 'transcription' not in st.session_state:
                st.session_state['transcription'] = ""
            
            st.session_state['transcription'] += text + " "
            st.session_state['updated'] = True
        
        except sr.UnknownValueError:
            st.session_state['transcription'] += "[Unrecognized Speech] "
        except sr.RequestError:
            st.session_state['transcription'] += "[Error: Connection Issue] "
        except Exception as e:
            st.session_state['transcription'] += f"[Error: {str(e)}] "

def text_to_speech():
    engine = pyttsx3.init()
    engine.say(st.session_state.get('transcription', ''))
    engine.runAndWait()

def start_recording():
    st.session_state['recording'] = True
    st.session_state['transcription'] = ""
    st.session_state['status'] = "Recording..."

def stop_recording():
    st.session_state['recording'] = False
    st.session_state['status'] = "Stopped"

def main():
    st.set_page_config(page_title="Verbatim - Speech to Text", layout="wide")
    
    st.sidebar.title("Menu")
    st.sidebar.button("User Profile")
    st.sidebar.button("Settings")
    st.sidebar.button("Session History")
    
    st.title("Verbatim")
    st.write("Speech To Text Program Also Supports Nigerian English")
    
    col1, col2 = st.columns(2)
    
    with col1:
        if st.button("Record & Transcribe"):
            start_recording()
        if st.button("Stop Transcription"):
            stop_recording()
    
    with col2:
        if st.button("Read Aloud"):
            text_to_speech()
    
    st.text_area("Transcribed Text:", st.session_state.get('transcription', ''), height=300, key="transcription")
    st.download_button("Download Transcript", st.session_state.get('transcription', ''), "transcription.txt")
    
    st.write(f"Status: {st.session_state.get('status', 'Idle')}")
    
    if st.session_state.get('recording', False):
        recognize_speech()
        time.sleep(0.1)
        st.rerun()

if __name__ == "__main__":
    if 'transcription' not in st.session_state:
        st.session_state['transcription'] = ""
    if 'recording' not in st.session_state:
        st.session_state['recording'] = False
    if 'status' not in st.session_state:
        st.session_state['status'] = "Idle"
    main()



