---
description: "Användarflödesdriven utveckling — nya och ändrade funktioner specificeras genom dokumenterade användarfall"
trigger: always_on
---

# Användarflödesdriven utveckling

Nya eller ändrade funktioner utvecklas utifrån dokumenterade användarfall (UC)
och systeminvarianter, inte direkt utifrån befintlig implementation.

Kod beskriver hur systemet är implementerat.
Use cases beskriver vilket observerbart beteende systemet ska ha.

---

# Källor

Följande dokument används i prioritetsordning:

1. `Development instructions/user_cases_actual.md`
   — dokumenterar aktuellt och verifierat beteende.

2. `Development instructions/User_Cases.md`
   — ursprunglig funktionell specifikation.

3. `Development instructions/Architecture.md`
   — systemets state machine, event contracts och arkitektoniska regler.

Om källorna motsäger varandra ska konflikten identifieras innan implementation
påbörjas.

`user_cases_actual.md` är den primära källan för hur systemet faktiskt beter sig.

Detta innebär dock inte att befintlig implementation automatiskt är korrekt.
Om implementationen avviker från det dokumenterade beteendet ska avvikelsen
klassificeras innan ändringar görs.

---

# Grundprincip

Ingen ny funktion eller beteendeförändring implementeras innan det berörda
användarflödet är identifierat och dokumenterat.

Use cases ska beskriva observerbart beteende och användarens perspektiv.
De ska inte i onödan låsas till intern implementation.

Intern implementation får ändras utan att ett UC behöver ändras, så länge
det observerbara beteendet och definierade tekniska kontrakt förblir
oförändrade.

---

# Systeminvarianter

Systeminvarianter är regler som alltid måste vara sanna, oavsett vilket
användarflöde som körs.

Exempel:

- En användare får inte tillhöra flera aktiva rum samtidigt.
- Ett avslutat rum får inte återgå till ett aktivt state.
- Servern är auktoritativ för applikationens state.
- Klienten får inte självständigt besluta om serverns state-transition.
- En användare får endast utföra events som är giltiga i aktuellt state.
- Privata data får inte exponeras genom publika events.

Systeminvarianter dokumenteras i `Architecture.md` och gäller för samtliga UC.

Ett UC får inte introducera ett beteende som bryter mot en systeminvariant utan
att invarianten först ändras som en explicit arkitekturell förändring.

---

# Process vid ny eller ändrad funktion

## 1. Identifiera berörda use cases

Läs relevanta delar av:

- `Development instructions/user_cases_actual.md`
- `Development instructions/User_Cases.md`
- `Development instructions/Architecture.md`

Identifiera:

- berörda Happy Paths
- berörda varianter
- relevanta states
- relevanta events
- berörda systeminvarianter
- befintliga tester

Ändra inte kod innan detta är gjort.

---

## 2. Identifiera konflikter

Om dokumentation och implementation skiljer sig åt ska avvikelsen klassificeras
som en av följande:

- **Bug** — implementationen följer inte avsett beteende.
- **Avsiktlig beteendeförändring** — systemets beteende ska ändras.
- **Odokumenterat befintligt beteende** — implementationen gör något som inte
  tidigare varit dokumenterat.

En konflikt får inte lösas genom att tyst ändra dokumentationen för att
matcha befintlig kod.

Om det är oklart vilket beteende som är avsett ska frågan lyftas innan kod
skrivs.

---

## 3. Skriv eller uppdatera UC före kod

Nya funktioner ska ha ett nytt UC.

Ändringar av befintligt beteende ska uppdatera det berörda UC:t.

UC ska beskriva:

- användarens handling
- frontendens observerbara state/view
- request eller socket-event
- serverns validering
- relevant state-transition
- response/broadcast
- förväntat resultat
- felhantering
- timeout/disconnect när relevant

Tekniska detaljer ska endast inkluderas när de utgör en del av ett kontrakt
som andra delar av systemet måste förhålla sig till.

---

## 4. Implementera mot UC

Implementationens observerbara beteende ska uppfylla UC.

Event, payloads, states och state-transitions ska följa de kontrakt som
definieras i `Architecture.md`.

Implementation får inte introducera ett nytt observerbart beteende som inte
är dokumenterat i ett UC, om inte beteendet är rent internt och saknar
konsekvens för systemets kontrakt eller användarflöde.

---

## 5. Testa UC

Varje Happy Path och varje relevant variant ska vara verifierbar genom
automatiserade tester där det är tekniskt möjligt.

Testnivå väljs efter vad som ska verifieras:

- **Backend unit tests** — individuell logik och services.
- **Backend integration tests** — state, handlers och socket-flöden.
- **Frontend tests** — komponenter, state och användarinteraktioner.
- **End-to-end tests** — kompletta användarflöden där det är motiverat.

Varje UC-variant ska kunna spåras till ett eller flera tester.

Ett UC behöver alltså inte motsvara exakt ett test.

---

## 6. Uppdatera actual-dokumentationen

När implementation och tester är färdiga ska:

`Development instructions/user_cases_actual.md`

uppdateras i samma ändring.

Dokumentet ska beskriva det beteende som faktiskt är implementerat och
verifierat.

Dokumentation och implementation får inte lämnas i ett medvetet inkonsekvent
tillstånd.

---

# Format för användarfall

```markdown
## Användarfall N: <Titel>

**Syfte**

<Kort beskrivning av vad användaren ska kunna uppnå.>

**Förutsättningar**

- <relevant state>
- <relevant användarroll>
- <eventuella andra krav>

**Huvudflöde (Happy Path)**

1. Användaren <handling>.
2. Client skickar `<EVENT_NAME>` med `{ ... }`.
3. Servern validerar requesten och aktuellt state.
4. Servern genomför `<STATE_TRANSITION>`.
5. Servern skickar/broadcastar `<EVENT_NAME>`.
6. Client uppdaterar till `<VIEW/STATE>`.
7. Användaren ser <förväntat resultat>.

**Varianter**

- **UCN.1** <edge case> — <förväntat beteende>.
- **UCN.2** <felstate> — <förväntat beteende>.
- **UCN.3** <timeout/disconnect> — <förväntat beteende>.

**Kontrakt**

- Events: `<EVENT>`
- States: `<STATE>`
- Payload: `{ ... }`
- Errors: `<ERROR>`