# Architecture

## Översikt

Frontend-applikationerna använder:

- React
- TypeScript
- Vite
- React Router
- Tailwind CSS
- Context API vid behov
- services för extern kommunikation

Frontend byggs som en statisk applikation och distribueras som Docker image.

## Projektstruktur

```text
src/
├── components/
├── features/
├── pages/
├── layouts/
├── hooks/
├── services/
├── data/
├── types/
└── utils/
```

### components/

Återanvändbara UI-komponenter.

Exempel:

```text
components/
├── Button.tsx
├── Dialog.tsx
├── SearchInput.tsx
└── ProductCard.tsx
```

### features/

Funktions- eller domänspecifik kod.

```text
features/
├── products/
│   ├── components/
│   ├── hooks/
│   ├── services/
│   └── types/
└── rooms/
    ├── components/
    ├── hooks/
    ├── services/
    └── types/
```

Feature-specifik kod ska placeras nära den feature den tillhör.

### pages/

Sidor kopplade till routes.

```text
pages/
├── HomePage.tsx
├── ProductPage.tsx
└── RoomPage.tsx
```

### layouts/

Gemensam sidstruktur.

```text
layouts/
├── AppLayout.tsx
└── AuthLayout.tsx
```

### hooks/

Återanvändbara custom hooks.

Hooks som endast används av en feature placeras helst i featuren.

### services/

Kommunikation med externa system.

Exempel:

```text
services/
├── apiClient.ts
├── productService.ts
└── socketService.ts
```

Services ansvarar för kommunikation och eventuell normalisering vid systemgränsen.

### data/

Statisk data, mockdata och datarelaterad konfiguration.

### types/

Delade TypeScript-typer.

Feature-specifika typer placeras helst i respektive feature.

### utils/

Generella rena funktioner.

Exempel:

```text
utils/
├── formatDate.ts
├── formatNumber.ts
└── sortByName.ts
```

Feature-specifik logik ska inte placeras i utils/.

## Dataflöde

Standardflöde för extern data:

```text
API / WebSocket / mockdata
          ↓
       service
          ↓
   state / context
          ↓
      component
          ↓
          UI
```

Komponenter ansvarar för presentation och användarinteraktion.

Services ansvarar för extern kommunikation.

Context eller annan state-hantering ansvarar för delad state.

## Lokal UI-state

Lokal UI-state stannar i komponenten när den inte behöver delas.

```text
Component
├── search
├── selectedTab
├── isDialogOpen
└── expanded
```

Exempel:

```ts
const [search, setSearch] = useState('');
const [isDialogOpen, setIsDialogOpen] = useState(false);
```

## Delad state

Context används när flera komponenter behöver samma state.

Exempel:

```text
RoomProvider
├── room
├── users
├── connectionState
└── actions
```

Komponenter använder Context via en hook:

```ts
const {
  room,
  submitAnswer,
} = useRoom();
```

## Server state

När applikationen har backend ska server state skiljas från lokal UI-state.

```text
Server state
├── products
├── room
├── users
└── session

UI state
├── search
├── selectedTab
├── isDialogOpen
└── expanded
```

TanStack Query används när API-baserad server state kräver caching, refetching eller mutationshantering.

## Realtidskommunikation

Applikationer med Socket.IO använder servern som källa till sanning för realtidsstate.

Standardflöde:

```text
User interaction
      ↓
Component
      ↓
Context / hook
      ↓
Socket event
      ↓
Backend
      ↓
State update
      ↓
Broadcast
      ↓
Frontend state
      ↓
UI
```

Komponenten skickar en intention.

```ts
submitAnswer(answer)
```

Inte ett direkt socket-event.

```ts
socket.emit('SUBMIT_ANSWER', ...)
```

## Routing

React Router används för client-side routing.

Routes definieras centralt eller i respektive route-struktur beroende på applikationens storlek.

Exempel:

```tsx
<Routes>
  <Route path="/" element={<HomePage />} />
  <Route path="/products/:id" element={<ProductPage />} />
</Routes>
```

## Feature-struktur

När en applikation växer används feature-baserad organisering.

```text
features/
└── products/
    ├── components/
    │   ├── ProductList.tsx
    │   └── ProductDetail.tsx
    ├── services/
    │   └── productService.ts
    ├── hooks/
    │   └── useProducts.ts
    └── types/
        └── product.ts
```

Gemensamma komponenter flyttas till `components/` först när de faktiskt används av flera features.

## Mockdata

Frontend-only-applikationer kan använda mockdata i `data/`.

```text
data/
├── products.json
└── users.json
```

Mockdata ska kunna ersättas med API-kommunikation utan att UI-komponenterna behöver känna till datakällan.

```text
Component
    ↓
data/service interface
    ↓
mockdata OR API
```

## Runtime och build

Vite används för frontend-build.

Build-specifik konfiguration anges via `VITE_*`.

Exempel:

```text
VITE_API_URL
VITE_MOCK_DATA_FILE
```

Vite-konfigurationen byggs in i frontend-applikationen.

Runtime configuration introduceras först när samma Docker image behöver konfigureras olika efter deployment.

## Docker

Frontend byggs med multi-stage Docker build.

```text
Stage 1
Node
  ↓
npm ci
  ↓
vite build
  ↓
dist/

Stage 2
nginx
  ↓
copy dist/
  ↓
serve frontend
```

Build dependencies och Node runtime behövs inte i den färdiga nginx-imagen.

Docker image är det primära deployment-artefaktet.

## SPA deployment

Nginx ska hantera client-side routes genom fallback till `index.html`.

Exempel:

```text
/
├── index.html
├── assets/
└── ...
```

En route som:

```text
/products/123
```

ska fortfarande returnera frontendens `index.html` så att React Router kan hantera routen.

## Backend

När backend introduceras hålls frontend och backend separerade.

```text
Frontend
   ↓
HTTP / WebSocket
   ↓
Backend
   ↓
Database / Redis / external services
```

Frontend ska inte ha direkt åtkomst till backendens databas.

## Testing architecture

Tester delas efter nivå:

```text
Unit
  ↓
Component
  ↓
Integration
  ↓
E2E
```

- Vitest används för unit- och komponenttester.
- Testing Library används för React-komponenter.
- Playwright används för end-to-end-flöden.

## Arkitekturprinciper

- UI-komponenter ska inte innehålla extern kommunikation.
- Feature-specifik kod hålls nära featuren.
- Delad kod placeras centralt först när den faktiskt delas.
- Lokal state föredras framför global state.
- Context används för avgränsad delad state.
- Services kapslar extern kommunikation.
- Rena funktioner används för ren datalogik.
- Abstraktioner införs vid konkret behov.