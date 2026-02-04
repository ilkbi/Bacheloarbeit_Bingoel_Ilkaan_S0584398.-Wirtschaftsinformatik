
import sqlite3
import random

import json
from datetime import datetime

import os
API_KEY=""
from openai import OpenAI
import re
import time
import Levenshtein
from datetime import datetime
import openai


hint = r"""
"""

def RequestToGpt(task, hint, content, history,model="gpt-3.5-turbo",protectFromDetectors=False
):
        i = 0
        if history is not None:
            messages = history
        else: messages =[]
        messages.append({
        "role": "user",
        "content": "###Task/Aufgabe = "+task+"###Hint/Hinweis = "+hint+"###Input = "+content})
        client = OpenAI(api_key=API_KEY)
        response = client.chat.completions.create(
                messages=messages,
                model = model,
                logprobs = False
                )
        assistant_response = response.choices[0].message.content
        return (assistant_response)

class GptAdapter:
    def send2gpt(self, text, history):
        para = RequestToGpt(
            f"""{text}""",
            f"""""",
            f"""""",
            history,
            model="gpt-4o",
        )
        return para


def init_db(conn):
    cur = conn.cursor()
    cur.execute("""
        CREATE TABLE IF NOT EXISTS customers (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            first_name TEXT,
            last_name TEXT,
            email TEXT,
            city TEXT,
            postal_code TEXT,
            house_year INTEGER,
            living_area_m2 INTEGER,
            current_heating TEXT,
            insulation_level TEXT,
            has_pv INTEGER,
            pain_point TEXT,
            created_at TEXT
        )
    """)
    cur.execute("""
        CREATE TABLE IF NOT EXISTS emails (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            customer_id INTEGER,
            subject TEXT,
            body TEXT,
            prompt TEXT,
            raw_response TEXT,
            created_at TEXT
        )
    """)
    conn.commit()


def count_customers(conn):
    cur = conn.cursor()
    cur.execute("SELECT COUNT(*) FROM customers")
    return int(cur.fetchone()[0])


def seed_customers(conn, n, seed=42):
    rng = random.Random(seed)

    first_names = ["Anna","Ben","Clara","David","Elena","Felix","Greta","Henrik","Isabel","Jonas","Klara","Leon","Mia","Nina","Oskar","Paula","Rafael","Sofia","Tom","Ute"]
    last_names  = ["Müller","Schmidt","Schneider","Fischer","Weber","Meyer","Wagner","Becker","Hoffmann","Schulz","Koch","Richter","Klein","Wolf","Neumann","Schwarz"]
    cities = [("München","80331"),("Hamburg","20095"),("Berlin","10115"),("Köln","50667"),("Frankfurt am Main","60311"),
              ("Stuttgart","70173"),("Düsseldorf","40213"),("Leipzig","04109"),("Dresden","01067"),("Nürnberg","90402")]
    heating_types = ["Gastherme","Ölheizung","Pelletheizung","Nachtspeicherheizung","Fernwärme"]
    insulation_levels = ["schwach","mittel","gut"]
    pain_points = [
        "steigende Energiekosten",
        "Heizung ist alt und wartungsintensiv",
        "Wunsch nach weniger CO₂ und mehr Unabhängigkeit",
        "Planung einer Sanierung in den nächsten 12 Monaten",
        "Unsicherheit, ob sich ein Umstieg finanziell lohnt",
    ]

    cur = conn.cursor()
    now = datetime.utcnow().isoformat(timespec="seconds") + "Z"

    for _ in range(n):
        fn = rng.choice(first_names)
        ln = rng.choice(last_names)
        city, plz = rng.choice(cities)
        house_year = rng.randint(1955, 2015)
        living_area = rng.randint(100, 220)
        heating = rng.choice(heating_types)
        insulation = rng.choices(insulation_levels, weights=[0.35, 0.45, 0.20], k=1)[0]
        has_pv = 1 if rng.random() < 0.25 else 0
        pain = rng.choice(pain_points)

        suffix = rng.randint(1000, 9999)
        email = (fn.lower() + "." + ln.lower() + "." + str(suffix) + "@example.com").replace("ü","ue").replace("ö","oe").replace("ä","ae").replace("ß","ss")

        cur.execute("""
            INSERT INTO customers (
                first_name,last_name,email,city,postal_code,
                house_year,living_area_m2,current_heating,insulation_level,has_pv,
                pain_point,created_at
            ) VALUES (?,?,?,?,?,?,?,?,?,?,?,?)
        """, (fn, ln, email, city, plz, house_year, living_area, heating, insulation, has_pv, pain, now))

    conn.commit()


def get_customers(conn, limit):
    cur = conn.cursor()
    cur.execute("SELECT * FROM customers ORDER BY id LIMIT ?", (limit,))
    return cur.fetchall()


def save_email(conn, customer_id, subject, body, prompt, raw_response):
    cur = conn.cursor()
    now = datetime.utcnow().isoformat(timespec="seconds") + "Z"
    cur.execute("""
        INSERT INTO emails (customer_id, subject, body, prompt, raw_response, created_at)
        VALUES (?,?,?,?,?,?)
    """, (customer_id, subject, body, prompt, raw_response, now))
    conn.commit()


# -----------------------------
# Prompt + Parsing
# -----------------------------
def build_prompt(c):
    pv_text = "Ja" if c["has_pv"] == 1 else "Nein"

    prompt = f"""
Schreibe eine personalisierte Kaltakquise-E-Mail (Deutsch) an einen Hausbesitzer (Einfamilienhaus),
der eine Luft/Wasser-Wärmepumpe kaufen soll. Ton: seriös, freundlich, nicht aufdringlich.

Kundendaten (synthetisch):
- Name: {c["first_name"]} {c["last_name"]}
- Ort: {c["postal_code"]} {c["city"]}
- Baujahr Haus: {c["house_year"]}
- Wohnfläche: {c["living_area_m2"]} m²
- Aktuelle Heizung: {c["current_heating"]}
- Dämmung (grob): {c["insulation_level"]}
- PV vorhanden: {pv_text}
- Schmerzpunkt: {c["pain_point"]}

Anforderungen:
- Body max. 160–190 Wörter.
- Keine erfundenen Zahlen zu Förderung/Ersparnis. Nur allgemein: Förderprogramme sind möglich und müssen individuell geprüft werden.
- Keine Druck-/Angst-Formulierungen.
- CTA: kurze Antwort mit Ja/Nein oder Terminvorschlag für 15-Minuten-Eignungscheck (Telefon/Video).
- Opt-out am Ende: kurze Antwort genügt, dann keine weitere Kontaktaufnahme.

Gib ausschließlich gültiges JSON zurück:
{{
  "subject": "...",
  "body": "Zeile1\\n\\nZeile2..."
}}
""".strip()

    return prompt


def parse_email_json(raw):
    raw = (raw or "").strip()
    start = raw.find("{")
    end = raw.rfind("}")
    if start == -1 or end == -1 or end <= start:
        raise ValueError("Keine JSON-Struktur gefunden.")
    data = json.loads(raw[start:end+1])
    return data["subject"].strip(), data["body"].strip()


def call_gpt(prompt_text):
    adapter = GptAdapter()
    para = adapter.send2gpt(prompt_text, history=[])



    return para


def main():
    DB_PATH = "customers_heatpump.db"
    SYNTH_CUSTOMERS = 30
    EMAILS_TO_GENERATE = 10

    conn = sqlite3.connect(DB_PATH)
    conn.row_factory = sqlite3.Row

    init_db(conn)

    if count_customers(conn) == 0:
        seed_customers(conn, SYNTH_CUSTOMERS, seed=42)

    customers = get_customers(conn, EMAILS_TO_GENERATE)

    results = []
    for c in customers:
        prompt = build_prompt(c)

        raw = call_gpt(prompt)
        try:
            subject, body = parse_email_json(raw)
        except Exception as e:
            subject = "PARSING_FEHLER"
            body = "Konnte JSON nicht parsen. Rohantwort gespeichert."
            print("Parsing-Fehler bei Kunde", c["id"], ":", str(e))

        save_email(conn, c["id"], subject, body, prompt, raw)

        print("=" * 80)
        print("Kunde:", c["first_name"], c["last_name"], "<" + c["email"] + ">")
        print("Betreff:", subject)
        print()
        print(body)
        print("=" * 80)
        print()

        results.append({"customer_id": c["id"], "subject": subject, "body": body})

    conn.close()
    return results


if __name__ == "__main__":
    results = main()
