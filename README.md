# import serial
import time
import pyttsx3
import speech_recognition as sr

# Inizializzazione della voce di JARVIS
engine = pyttsx3.init()
engine.setProperty('rate', 160)

def parla(testo):
    print(f"JARVIS: {testo}")
    engine.say(testo)
    engine.runAndWait()

parla("Sistemi operativi attivi. Connessione all'esoscheletro.")

# Configurazione seriale (Arduino su COM6)
try:
    ser = serial.Serial('COM6', 9600)
    time.sleep(2)
    parla("Collegamento ad Arduino stabilito sulla porta COM 6.")
except:
    ser = None
    parla("Attenzione: Arduino non rilevato. Controlla la connessione.")

parla("Sistema vocale pronto. Dimmi 'chiuditi' o 'apriti'.")

# Funzione per inviare i comandi ad Arduino
def invia_ad_arduino(valore_p, valore_i, valore_m, valore_mi, valore_a):
    if ser:
        # Formato inviato: pollice,indice,medio,mignolo,altro\n
        stringa = f"{valore_p},{valore_i},{valore_m},{valore_mi},{valore_a}\n"
        ser.write(stringa.encode('utf-8'))
        print(f"Inviato ad Arduino -> {stringa.strip()}")

# Ciclo principale di ascolto vocale
recognizer = sr.Recognizer()
mic = sr.Microphone()

with mic as source:
    recognizer.adjust_for_ambient_noise(source, duration=1)

while True:
    try:
        with mic as source:
            print("\nJARVIS in ascolto... (di 'chiuditi' o 'apriti')")
            audio = recognizer.listen(source, timeout=5, phrase_time_limit=5)
        
        testo_ascoltato = recognizer.recognize_google(audio, language="it-IT").lower()
        print(f"Ho sentito: {testo_ascoltato}")
        
        if "chiuditi" in testo_ascoltato or "chiudi" in testo_ascoltato:
            parla("Eseguo la chiusura completa.")
            # Mandiamo 180 gradi a tutte e 5 le dita per chiudere la mano
            invia_ad_arduino(180, 180, 180, 180, 180)
            
        elif "apriti" in testo_ascoltato or "apri" in testo_ascoltato:
            parla("Apro la mano.")
            # Mandiamo 0 gradi a tutte e 5 le dita per aprire la mano
            invia_ad_arduino(0, 0, 0, 0, 0)
            
        elif "esci" in testo_ascoltato or "spegni" in testo_ascoltato:
            parla("Spegnimento dei sistemi. A presto, signore.")
            break
            
    except sr.WaitTimeoutError:
        # Se passa il tempo senza sentire nulla, ripete l'ascolto senza bloccarsi
        pass
    except Exception as e:
        # Gestisce eventuali errori di riconoscimento del microfono
        pass

if ser:
    ser.close()