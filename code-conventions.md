---
description: "Aletheia — development conventions, architecture rules and project commands"
trigger: always_on
---

# Aletheia — Project Conventions

Dessa regler definierar projektets tekniska konventioner, arkitektur och
standardiserade utvecklingssätt.

Nya implementationer ska följa dessa regler.

Om en ny funktion kräver att en arkitekturregel eller konvention bryts ska
regeln först ändras explicit. Regeln får inte kringgås lokalt i implementationen.

---

# 1. Project structure

Projektet är ett monorepo med tre npm-paket:

```text
/
├── package.json
├── backend/
│   ├── package.json
│   └── ...
└── frontend/
    ├── package.json
    └── ...
```

Root-paketet ansvarar endast för gemensamma utvecklingskommandon och
processhantering.

Backend och frontend är självständiga npm-paket med egna dependencies.

Dependencies ska vara installerade i samtliga tre paket innan
utvecklingsmiljön startas.

# 2. Standard commands

## Development

Från projektets root:

```bash
npm run dev
```

Detta startar:

- Backend på port 3000
- Frontend på port 5173

Rootens `npm run dev` ska användas som standard för lokal utveckling.

## Backend tests

```bash
cd backend && npm test
```

Backend använder Jest.

## Frontend tests

```bash
cd frontend && npm test
```

Frontend använder Vitest.

## Frontend build

```bash
cd frontend && npm run build
```

Build-output skrivs till:

```text
frontend/dist
```

Backend serverar den byggda frontend-applikationen statiskt.

# 3. Architecture rules

## Backend is authoritative

Backend är systemets enda auktoritativa källa för applikationens state.

Frontend skickar intents/actions.

Backend:

- tar emot eventet
- validerar request och aktuellt state
- genomför eventuell state-transition
- uppdaterar serverns state
- broadcastar det nya state enligt systemets event contract

Frontend får inte självständigt bestämma serverns state.

Business logic får inte dupliceras mellan frontend och backend.

Frontend får innehålla presentation, lokal UI-state och klientlogik, men inte
auktoritativ business logic.

# 4. State machine

Rum använder följande state machine:

```text
QUESTION_APPROVAL
        ↓
    ANSWERING
        ↓
    COUNTDOWN
        ↓
    REVEALED
        ↓
QUESTION_APPROVAL
```

Alla state-transitions ska gå genom den definierade transition-logiken:

`VALID_TRANSITIONS`

i:

`backend/RoomService.js`

En implementation får inte kringgå `VALID_TRANSITIONS` genom att mutera state
direkt.

Ogiltiga events ska resultera i ett `ERROR`-svar.

Ogiltiga events får inte kasta ett exception som lämnar socket-handlern och
avslutar eller förstör anslutningen.

Om en ny funktion kräver en ny state-transition ska:

- state machine uppdateras
- transitionen dokumenteras
- relevanta use cases uppdateras
- tester läggas till

# 5. Socket events

Socket event names använder:

`SCREAMING_SNAKE_CASE`

Exempel:

```text
JOIN_ROOM
START_ANSWERING
SUBMIT_ANSWER
ROOM_UPDATE
ERROR
```

Event-namn ska vara stabila API-kontrakt.

Ett befintligt event får inte byta namn eller payload-format utan att:

- berörda use cases uppdateras
- backend-dokumentationen uppdateras
- frontend uppdateras
- relevanta tester uppdateras

Servern ska normalt broadcasta `ROOM_UPDATE` när rummets auktoritativa state
har förändrats.

# 6. Backend conventions

Backend använder:

- Node.js
- Express 5
- Socket.IO
- CommonJS

Imports/exports ska använda:

```js
require(...)
module.exports = ...
```

ES module syntax ska inte introduceras i backend utan en explicit
arkitekturförändring.

# 7. Socket handlers

Socket handlers finns i:

`backend/socket/handlers.js`

Det ska finnas en `socket.on(...)`-handler per event.

Varje handler ska:

- validera input
- kontrollera relevant state
- anropa lämplig business logic/service
- returnera eller broadcasta resultat
- hantera exceptions

Handlers ska inte innehålla större mängder business logic.

Business logic hör hemma i services, framför allt `RoomService`.

Exceptions får inte krascha socket-anslutningen.

Fel ska kommuniceras genom projektets definierade error contract:

```js
{
  type: 'ERROR',
  message
}
```

# 8. Logging

Backend använder projektets strukturerade logger.

I filer som behöver logging ska logger skapas på filnivå:

```js
const logger = createLogger('Context')
```

Logging ska använda strukturerade metadata:

```js
logger.info(msg, { meta })
```

Använd inte godtyckliga `console.log` för applikationslogging.

Loggar ska beskriva relevanta systemhändelser utan att exponera känsliga
eller onödiga användardata.

# 9. Backend testing

Backend-tester ska skilja mellan unit tests och integration tests.

## Unit tests

Unit tests anropar `RoomService`-metoder direkt.

De ska användas för att verifiera:

- business logic
- state transitions
- validation
- edge cases
- error conditions

## Socket integration tests

Socket integration tests använder:

`backend/test-helpers/socket.js`

Integrationstester ska använda:

- riktig server
- slumpmässig port
- riktig socket.io-client

Socket-flöden ska därför inte reduceras till enbart mockade unit tests.

# 10. Frontend conventions

Frontend använder ES modules.

Komponenter använder:

`.jsx`

Hooks och utilities använder:

`.js`

Exports ska vara named exports.

Undvik default exports.

# 11. Frontend structure

Frontend organiseras efter ansvar:

```text
components/
views/
hooks/
context/
utils/
test/
```

Tester ska normalt ligga tillsammans med källfilen.

Exempel:

```text
QuestionView.jsx
QuestionView.test.jsx
```

En komponent ska inte placeras i en annan kategori enbart för att det är
bekvämt.

Filens placering ska återspegla dess huvudsakliga ansvar.

# 12. Frontend state

Server state ska hanteras genom:

`RoomContext`

Komponenter ska konsumera server state genom:

`useRoom()`

Komponenter ska inte skapa egna parallella kopior av serverns state.

`useState` får användas för lokal UI-state och temporär input.

Exempel på lämplig lokal state:

- inputfält
- öppnad/stängd dialog
- lokal UI-toggle
- tillfälligt formulärvärde

Exempel på state som inte ska dupliceras lokalt:

- aktuell room state
- deltagare
- serverns fråga
- serverns countdown
- serverns aktuella fas

# 13. Socket communication in frontend

För varje socket event som frontend skickar ska den relevanta
socket-kommunikationen kapslas i en tydlig funktion.

Exempel:

```js
const emitStartAnswering = useCallback(() => {
  socket.emit('START_ANSWERING')
}, [socket])
```

Komponenter ska inte sprida råa `socket.emit(...)`-anrop över hela
applikationen.

Socket events ska exponeras genom `RoomContext` eller relevant hook.

# 14. Styling

Projektet använder Tailwind CSS.

Tailwind utility classes ska normalt skrivas direkt i JSX.

Conditional classes ska hanteras med template literals eller projektets
etablerade utility för detta.

Exempel:

```jsx
className={`base-class ${active ? 'active-class' : 'inactive-class'}`}
```

En separat CSS-klass ska inte skapas när motsvarande Tailwind utilities
räcker.

# 15. Icons

Ikoner ska implementeras som inline SVG.

Ingen extern icon library ska introduceras utan explicit arkitekturbeslut.

SVG ska vara semantiskt och tillgängligt när ikonen har funktionell betydelse.

Dekorativa ikoner ska inte få en falsk accessible label.

# 16. Accessibility and language

All användarvänd text i frontend ska vara på svenska.

Detta gäller bland annat:

- knappar
- rubriker
- felmeddelanden
- hjälptexter
- tomtillstånd
- loading states
- aria-labels

Interaktiva element ska ha korrekt semantik och relevanta
accessibility-attribut.

`aria-label` ska användas när ett interaktivt element saknar en synlig
textetikett.

# 17. Session persistence

Klientsessionen persisteras genom `localStorage`.

Följande keys används:

- `userId`
- `roomId`
- `recentRooms`

Nya keys ska inte introduceras för state som redan representeras av
`RoomContext` utan ett konkret behov.

`localStorage` ska betraktas som klientens persistenslager, inte som
auktoritativ server state.

# 18. Frontend testing

Frontend använder Vitest.

Socket.IO-klienten mockas genom:

`__mocks__/`

och:

`vi.mock(...)`

Servermeddelanden ska injiceras genom:

`mockSocket.receive(...)`

Tester ska verifiera observerbart frontend-beteende snarare än implementation
av interna React-detaljer när det är möjligt.

# 19. Documentation

Följande dokument ska hållas synkroniserade med implementationen:

- `README.md`
- `backend/README.md`
- `frontend/README.md`

Detta gäller särskilt:

- Socket.IO event lists
- payloads
- commands
- project structure
- state machine
- relevanta arkitekturregler

Dokumentation får inte medvetet lämnas i ett tillstånd där den beskriver ett
annat API eller beteende än implementationen.

Användarflöden dokumenteras enligt reglerna för user-case-driven development.

# 20. Separation of concerns

Ansvar ska hållas separerade:

```text
Frontend
  ├── UI
  ├── presentation
  ├── lokal UI-state
  └── server communication

Context / Hooks
  ├── client state
  └── socket communication

Socket handlers
  ├── transport
  ├── validation
  └── error handling

Services
  ├── business logic
  ├── state transitions
  └── domain rules

Tests
  └── verification of behaviour
```

Business logic ska inte flyttas till en komponent bara för att implementationen
blir kortare.

Socket handlers ska inte bli ett andra service-lager.

Services ska inte känna till React eller frontendens implementation.

# 21. Dependency rules

Nya dependencies ska endast introduceras när befintliga projektverktyg inte
rimligen kan lösa problemet.

Innan en dependency läggs till ska följande övervägas:

- Kan funktionaliteten implementeras med befintliga dependencies?
- Kan den implementeras med plattformens standard-API?
- Introducerar dependencyen betydande komplexitet?
- Påverkar den bundle size, säkerhet eller underhåll?
- Behöver konventionerna eller dokumentationen uppdateras?

En dependency får inte introduceras enbart för att spara några få rader kod.

# 22. Error handling

Fel ska hanteras explicit.

Användaren ska få ett definierat felmeddelande när ett operation misslyckas
på grund av exempelvis:

- ogiltigt state
- ogiltig input
- saknad resurs
- otillåten operation
- serverfel

Exceptions ska inte användas som normalt kontrollflöde.

Fel som passerar systemgränser ska följa definierade error contracts.

# 23. Security

Klienten ska betraktas som opålitlig.

Servern ska validera:

- inkommande payloads
- användaridentitet där relevant
- behörighet
- aktuellt state
- tillåtna transitions

Klienten får aldrig betraktas som en säkerhetsgräns.

Information ska inte skickas till klienten enbart för att den "kan döljas"
i UI.

Servern ansvarar för att endast exponera data som klienten får tillgång till.

# 24. Changes to architecture

Om en funktion inte kan implementeras utan att bryta mot dessa regler ska
reglerna inte kringgås.

Gör i stället en explicit arkitekturändring.

En arkitekturändring ska:

- beskriva problemet
- beskriva varför nuvarande arkitektur inte räcker
- beskriva den nya lösningen
- uppdatera relevanta dokument
- uppdatera tester
- uppdatera berörda use cases

Kod som kräver en permanent "speciallösning" för att undvika en
arkitekturändring ska betraktas som en signal om att arkitekturen behöver
omprövas.

# 25. Definition of Done

En implementation är färdig när:

- [ ] Projektets arkitekturregler följs.
- [ ] Befintliga konventioner följs.
- [ ] Ingen business logic har flyttats till frontend.
- [ ] State transitions går genom `VALID_TRANSITIONS`.
- [ ] Socket events följer event-kontraktet.
- [ ] Fel hanteras utan att socket-anslutningen kraschar.
- [ ] Backend-tester är uppdaterade.
- [ ] Frontend-tester är uppdaterade.
- [ ] Relevanta integrationstester finns.
- [ ] Dokumentationen är uppdaterad.
- [ ] Berörda use cases är uppdaterade enligt user-case-regeln.
- [ ] Nya dependencies är motiverade.
- [ ] Inga nya parallella state-källor har introducerats.
- [ ] Ingen arkitekturregel har kringgåtts genom en lokal speciallösning.
