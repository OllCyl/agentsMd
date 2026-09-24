# Frontend Technology Stack

Gemensam teknikstack för frontend-baserade demoapplikationer.

## Core

| Område | Teknik | Användning |
|---|---|---|
| Framework | React + TypeScript | Applikationsutveckling |
| Build tool | Vite | Development server och production build |
| Routing | React Router | Klientbaserad routing |
| Styling | Tailwind CSS | Styling, layout och responsivitet |
| UI components | shadcn/ui | Återanvändbara UI-komponenter |
| UI primitives | Radix UI | Tillgängliga primitives som grund för shadcn/ui |
| Icons | Lucide React | Standardbibliotek för ikoner |

## State och data

| Område | Teknik | Användning |
|---|---|---|
| Local state | React state | Primär state-hantering |
| Shared state | React Context | State som faktiskt behöver delas mellan komponenter |
| Forms | React Hook Form | Komplexare formulär |
| Validation | Zod | Schema- och datavalidering |
| API state | TanStack Query | Introduceras när applikationen får backend/API |
| API mocking | MSW | Mockade API-anrop vid behov |

## Data och presentation

| Område | Teknik | Användning |
|---|---|---|
| Tables | TanStack Table | Komplexa eller interaktiva tabeller |
| Charts | Recharts | Datavisualiseringar |
| Date handling | date-fns | Datumhantering |

Bibliotek under "vid behov" ska inte inkluderas i en applikation om funktionaliteten kan implementeras enkelt utan dem.

## Animation

Tailwind CSS används för enklare transitions och animationer.

Motion kan användas när mer avancerade animationer eller transitions behövs.

Motion ska inte introduceras enbart för enkla hover-, focus- eller transition-effekter.

## Testing

| Område | Teknik | Användning |
|---|---|---|
| Unit/component tests | Vitest | Test runner |
| Component testing | Testing Library | Testning av React-komponenter |
| E2E | Playwright | End-to-end-testning av centrala användarflöden |

## Code quality

| Område | Teknik | Användning |
|---|---|---|
| Linting | ESLint | Statisk kodanalys |
| Formatting | Prettier | Automatisk kodformatering |
| Type checking | TypeScript | Statisk typkontroll |

TypeScript ska köras med `strict` aktiverat.

## Deployment

Frontend-applikationerna paketeras som Docker images.

Production image byggs med en multi-stage Docker build:

1. Node.js används i build stage för installation av dependencies och Vite build.
2. Den färdiga `dist/`-katalogen kopieras till en minimal nginx-runtime image.
3. Node.js och build dependencies ska inte finnas i production image.

### Runtime

- nginx-unprivileged
- SPA fallback för klientrouting
- HTTP-port enligt deploymentmiljö
- Statisk frontend serveras från nginx

### Deployment principle

Docker image är den primära deploymentartefakten.

Samma image ska kunna användas mellan miljöer där konfigurationen inte kräver en ny frontend-build.