# Workflow „Onboarding"

## Ce face

Workflow-ul automatizează integrarea unui angajat nou în Slack:

1. trimite un mesaj de bun venit (reguli, canale importante, contacte);
2. cere într-un formular rolul/departamentul, data de început și preferința de laptop;
3. dacă departamentul este IT, anunță `#it-support` cu o cerere de acces;
4. prezintă persoana nouă în `#echipa`;
5. programează remindere la Ziua 1, 3, 7 și 30 pentru angajat și manager;
6. la final, trimite formularul de feedback de 30 de zile, iar rezultatele ajung privat la manager și HR.

Definiția declarativă se află în [`onboarding.workflow.json`](./onboarding.workflow.json). Este independentă de instrument și servește drept specificație. Slack nu poate importa acest fișier direct (vezi secțiunea de recreare).

## Variabile

| Variabilă | Descriere |
|---|---|
| `{{new_member}}` | Membrul nou |
| `{{manager}}` | Managerul direct |
| `{{buddy}}` | Colegul „buddy" |
| `{{department}}` | Departamentul (din formular) |
| `{{start_date}}` | Data de început (din formular) |

## Declanșatoare

| ID | Tip | Descriere |
|---|---|---|
| `trigger_member_joined` | Eveniment `member_joined_channel` pe `#bun-venit` | Pornește automat pentru membrul nou |
| `trigger_manual` | Manual: „Pornește onboarding" | Folosit de manager pentru un angajat nou numit |

## Pași

| # | ID | Tip | Destinatar / canal | Descriere |
|---|---|---|---|---|
| 1 | `welcome_dm` | send_dm | membrul nou | Bun venit, reguli, canale, contacte |
| 2 | `info_form` | send_form | membrul nou | Rol/departament, data de început, laptop |
| 3 | `check_department` | condition | - | Dacă departament = IT, merge la pasul 4, altfel la 5 |
| 4 | `notify_it_support` | send_channel_message | `#it-support` | Cerere de acces (doar IT) |
| 5 | `announce_in_team` | send_channel_message | `#echipa` | Prezentarea persoanei noi |
| 6 | `reminder_day_1` | create_reminder | membru + manager | Conturi și acces, întâlnire cu managerul, prezentare în echipă |
| 7 | `reminder_day_3` | create_reminder | membru + manager | Module de training, întâlnire cu buddy |
| 8 | `reminder_day_7` | create_reminder | membru + manager | Formular de feedback pentru prima săptămână |
| 9 | `reminder_day_30` | create_reminder | membru + manager | Evaluare cu managerul |
| 10 | `final_feedback_form` | send_form | membrul nou | Feedback la 30 de zile |
| 11 | `post_results` | send_dm | manager + HR (privat) | Rezultatele formularului |

### Condiții

| Condiție | Dacă adevărat | Dacă fals |
|---|---|---|
| `{{department}} == 'IT'` | `notify_it_support` | `announce_in_team` |

## Diagramă de flux

```
 [Trigger 1: membru intră în #bun-venit]   [Trigger 2: manual „Pornește onboarding"]
                    \                         /
                     v                       v
                   +---------------------------+
                   | 1. welcome_dm (DM)        |
                   +---------------------------+
                                |
                                v
                   +---------------------------+
                   | 2. info_form (formular)   |
                   +---------------------------+
                                |
                                v
                   +---------------------------+
                   | 3. Departament = IT ?     |
                   +---------------------------+
                      | DA                | NU
                      v                   |
        +---------------------------+     |
        | 4. Notificare #it-support |     |
        +---------------------------+     |
                      |                   |
                      v                   v
                   +---------------------------+
                   | 5. Anunț în #echipa       |
                   +---------------------------+
                                |
                                v
                   +---------------------------+
                   | 6. Reminder Ziua 1        |
                   +---------------------------+
                                |
                                v
                   +---------------------------+
                   | 7. Reminder Ziua 3        |
                   +---------------------------+
                                |
                                v
                   +---------------------------+
                   | 8. Reminder Ziua 7        |
                   +---------------------------+
                                |
                                v
                   +---------------------------+
                   | 9. Reminder Ziua 30       |
                   +---------------------------+
                                |
                                v
                   +---------------------------+
                   | 10. Formular feedback 30z |
                   +---------------------------+
                                |
                                v
                   +---------------------------+
                   | 11. Rezultate -> manager  |
                   |     și HR (privat)        |
                   +---------------------------+
```

## Recrearea în Slack Workflow Builder (UI)

Slack nu permite importul JSON pentru workflow-uri, deci fluxul se construiește manual pe baza fișierului JSON.

1. În Slack, deschide **Tools** (sau **Automations**) > **Workflow Builder** și apasă **New workflow** (Nou).
2. Denumește-l **Onboarding**.
3. **Declanșator 1:** alege *When a person joins a channel* (Când o persoană intră pe un canal) și selectează `#bun-venit`. Variabila „Person who joined" corespunde lui `new_member`.
4. **Declanșator 2 (manual):** Slack permite un singur declanșator per workflow. Creează un al doilea workflow, **Pornește onboarding**, cu declanșatorul *From a link in Slack* (sau *Slash command*), cu un pas inițial *Open a form* (Angajat nou, Manager, Buddy), apoi repetă pașii de mai jos sau, unde planul o permite, apelează primul workflow.
5. **Pas 1:** *Send a message* > către *Person who joined* și adaugă textul din `welcome_dm`. Inserează variabilele din meniul *Insert a variable*.
6. **Pas 2:** *Send a form* către *Person who joined*, cu câmpurile Rol/departament (listă), Data de început (dată) și Preferință laptop (listă). Răspunsurile devin variabile.
7. **Pas 3 (condiție):** dacă planul Slack nu are ramificări, adaugă un pas *Send a message* către `#it-support` care include mereu cererea (sau folosește un pas de tip *Custom / Workflow step* dacă este disponibil), iar cei care nu sunt din IT o ignoră. Pe planurile cu logică condițională, adaugă *Add a condition* cu `Departament = IT`.
8. **Pas 4:** *Send a message* în `#it-support` cu textul din `notify_it_support`.
9. **Pas 5:** *Send a message* în `#echipa` cu textul din `announce_in_team`.
10. **Pași 6-9 (remindere):** Workflow Builder nu are pași „la N zile după data X". Alternative: (a) un workflow separat pentru fiecare reminder, pornit programat (*On a schedule*), sau (b) folosește `/remind @persoană "mesaj" în 3 zile` și `/remind` pentru manager, la finalul workflow-ului. Textele sunt cele din `reminder_day_*`.
11. **Pas 10:** pentru feedback-ul de 30 de zile, creează un workflow programat care trimite *Send a form* către angajat cu câmpurile din `final_feedback_form`.
12. **Pas 11:** adaugă *Send a message* către manager și HR (persoane sau un grup privat) cu răspunsurile formularului. Alternativ, *Add to a spreadsheet* pentru arhivă.
13. Verifică textele, apasă **Finish Up** și **Publish**. Testează cu un cont de test.

Notă: denumirile butoanelor diferă ușor între planuri și versiuni ale Slack. Funcțiile de ramificare și de programare depind de planul folosit.

## Sfat: personalizare

- Numele canalelor (`#bun-venit`, `#echipa`, `#it-support`, `#general`, `#anunturi`, `#ajutor-it`) și grupul HR se schimbă direct în JSON, la câmpurile `channel`, `to` și în textele mesajelor.
- Ajustează mesajele și zilele (`offset_days`) după procesele companiei.
- Păstrează variabilele `{{...}}` neschimbate, ca fluxul să rămână consecvent.
- Dacă modifici JSON-ul, actualizează și acest README și rulează `python3 -m json.tool workflows/onboarding/onboarding.workflow.json` pentru validare.
