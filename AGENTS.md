# Project Standards

## 7. Technology Stack

### Core

| Area | Technology | Usage |
| --- | --- | --- |
| Framework | React + TypeScript | Application development |
| Build tool | Vite | Development server and production build |
| Routing | React Router | Client-side routing |
| Styling | Tailwind CSS | Styling, layout and responsiveness |
| UI components | shadcn/ui | Reusable UI components |
| UI primitives | Radix UI | Accessible UI primitives |
| Icons | Lucide React | Standard icon library |

### State and Data

| Area | Technology | Usage |
| --- | --- | --- |
| Local state | React state | Primary local state |
| Shared state | React Context | State genuinely shared between components |
| Forms | React Hook Form | Complex forms |
| Validation | Zod | Schema and data validation |
| API state | TanStack Query | API server state when caching/refetching/mutations are needed |
| API mocking | MSW | Mock API requests when appropriate |

### Data and Presentation

| Area | Technology | Usage |
| --- | --- | --- |
| Tables | TanStack Table | Complex or interactive tables |
| Charts | Recharts | Data visualization |
| Date handling | date-fns | Date manipulation |

Libraries listed as optional must not be introduced when the required
functionality can be implemented simply without them.

## 8. Animation

Tailwind CSS should be used for simple transitions and animations.

Motion may be introduced when more advanced animation or transition
behavior is required.

Motion should not be introduced solely for:

- simple hover effects
- simple focus effects
- simple CSS transitions

## 9. Development Conventions

### TypeScript

- TypeScript is the standard frontend language.
- TypeScript must use strict mode.
- Avoid `any` unless there is a concrete and documented reason.
- Prefer sound type design over unnecessary type assertions.

### React

- Use functional components and hooks.
- Prefer named exports.
- Keep components focused and reasonably small.
- Do not create abstractions merely to make the file structure appear
  sophisticated.

### External Communication

- Components must not communicate directly with external systems.
- Use the established service, hook, context or query boundary.
- Components should express application intentions rather than implementation
  details.

For example:

```ts
submitAnswer(answer)
```

is preferred over:

```ts
socket.emit("SUBMIT_ANSWER", answer)
```

inside a component.

### Styling

- Use Tailwind CSS for application styling.
- Prefer existing shadcn/ui and Radix UI primitives before creating
  duplicate implementations.

### Icons

Use Lucide React for standard icons.

Custom inline SVG should only be used when:

- the required visual does not exist in the standard icon library, or
- a specific custom SVG is required by the application.

### Accessibility

- Interactive elements must have appropriate accessible names and semantic
  behavior.
- User interfaces should use Swedish visible text unless the application
  explicitly targets another language.
- Swedish aria-label values should be used where appropriate.

## 10. Backend Architecture

Applications that include a backend should follow this conceptual
structure:

```text
Frontend
   ↓
HTTP / WebSocket
   ↓
Backend
   ↓
Database / Redis / External services
```

The backend is authoritative for server state.

Clients send intentions.

The server:

- validates the intention
- applies business rules
- changes authoritative state
- broadcasts the resulting state or event

The frontend must not independently decide authoritative server state.

## 11. Backend State Machines

When a backend uses a state machine, state transitions must be explicitly
defined.

For example:

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

- Valid transitions must be defined centrally, for example through a
  `VALID_TRANSITIONS` structure.
- Business logic must not bypass state validation through direct state
  mutation.
- Invalid transitions must produce a controlled error response.
- They must not crash the connection or process.

## 12. Backend Implementation Conventions

Where Node.js and Socket.IO are used:

- use CommonJS where required by the application
- use Express for HTTP APIs
- use Socket.IO for real-time communication
- use one socket handler per event
- wrap socket handlers in appropriate error handling
- never allow an invalid client request to crash the connection

Socket event names should use `SCREAMING_SNAKE_CASE`.

Errors should be returned through the application's defined error
contract rather than thrown beyond the socket handler.

Structured logging should be used consistently.

Example:

```js
const logger = createLogger("RoomService");

logger.info("Room updated", {
  roomId,
  state
});
```

## 13. Testing

Testing should verify both behavior and architectural boundaries.

The test levels are complementary:

```text
Unit
  ↓
Component
  ↓
Integration
  ↓
E2E
```

Tests should be placed at the lowest level that reliably verifies the
behavior.

### Unit Tests

- Use Vitest for frontend unit tests.
- Backend domain logic should have direct unit tests.
- Pure functions and business logic should be tested independently where
  practical.

### Component Tests

- Use Testing Library for React component behavior.
- Tests should verify observable behavior rather than implementation
  details.

### Integration Tests

- Integration tests should verify communication between meaningful system
  boundaries.
- For Socket.IO applications, use a real test server and real
  socket.io-client where practical.

### E2E Tests

- Use Playwright for central end-to-end user flows.
- E2E tests should correspond to important User Cases rather than attempt
  to duplicate every unit or component test.

### User Case Coverage

Every meaningful User Case variant should be covered by an automated test
where technically practical.

## 14. Frontend Testing Conventions

When Socket.IO is mocked in frontend tests:

- mock socket.io-client through the project's test mock mechanism
- use `vi.mock(...)`
- inject server messages through the mock socket
- test resulting observable UI behavior

Tests should not depend unnecessarily on private implementation details.

## 15. Code Quality

The standard tooling is:

| Area | Technology |
| --- | --- |
| Linting | ESLint |
| Formatting | Prettier |
| Type checking | TypeScript |
| Unit/component tests | Vitest |
| Component testing | Testing Library |
| E2E | Playwright |

TypeScript must run with `strict: true`.

Code should be formatted consistently and pass the project's lint and
type-checking requirements.

## 16. Configuration and Environment Variables

- Vite environment variables using the `VITE_*` prefix are public.
- They must never contain secrets.
- Secrets must not be bundled into the frontend application.
- Runtime configuration may be introduced when the same Docker image needs
  to operate in multiple environments without rebuilding the frontend.
- Runtime configuration should be explicit and documented.

## 17. Deployment

Frontend applications are packaged as Docker images.

The production image should use a multi-stage Docker build:

- Node.js is used in the build stage.
- Dependencies are installed and the Vite production build is created.
- The resulting `dist/` directory is copied into a minimal nginx
  runtime image.
- Node.js and build dependencies must not exist in the production image.

### Runtime

The standard frontend runtime is `nginx-unprivileged`.

The runtime must provide:

- SPA fallback for client-side routing
- correct static asset handling
- the HTTP port required by the deployment environment

Static frontend assets are served by nginx.

### Deployment Principle

The Docker image is the primary frontend deployment artifact.

The same image should be usable across environments when configuration
does not require a new frontend build.

## 18. Mock Data

Mock data should be isolated behind the same conceptual boundaries as
real data.

Components should not need to know whether data originates from:

- static mock data
- an API
- a WebSocket
- another external service

When practical, the transition from mocked data to real data should occur
behind the service/data boundary.

## 19. Documentation

Documentation must remain consistent with the implementation.

Relevant documentation includes:

- User Cases
- Architecture
- Technology Stack
- Development Conventions
- README files
- API documentation
- Socket event documentation
- deployment documentation

User Case documents are written in markdown and stored in the
`User Cases/` directory at the repository root.

When an external contract changes, update all documentation that defines
or references that contract.

Documentation must not be changed merely to hide an implementation
problem.

## 20. Change Management

When making a change, consider all affected layers:

```text
User Case
    ↓
Architecture
    ↓
Technology
    ↓
Convention
    ↓
Implementation
    ↓
Tests
    ↓
Documentation
```

Not every change affects every layer.

Only update a higher-level document when the corresponding requirement or
architectural decision has actually changed.

Do not duplicate detailed implementation rules across documents without a
reason.

## 21. Definition of Done

A feature or change is complete when:

- the relevant User Cases are defined or updated
- architectural constraints are respected
- the approved technology stack is used
- implementation follows the project conventions
- relevant automated tests exist
- existing tests pass
- TypeScript type checking passes
- linting passes
- formatting is consistent
- relevant documentation is updated
- externally visible contracts are synchronized
- verified behavior is reflected in `user_cases_actual.md`

A feature is not considered complete merely because the UI appears to
work manually.

## 22. Rule Precedence

When rules appear to conflict, use the following conceptual hierarchy:

```text
User Case
    ↓
Architecture
    ↓
Technology Stack
    ↓
Conventions
    ↓
Implementation
```

However, these layers answer different questions and should not normally
conflict.

If a genuine conflict is discovered:

- identify the conflicting rules
- determine which layer actually owns the decision
- resolve the conflict explicitly
- update the affected documentation
- do not silently override the higher-level rule in code

Application-specific requirements may extend the common standards, but
such deviations should be explicit.

## 23. Git Workflow

Git history should be small, coherent and traceable.

### Commits

Changes should be committed in small, logical units.

Each commit should:

- represent one coherent change
- have a clear and meaningful purpose
- avoid unrelated modifications
- be small enough to review and understand independently
- leave the application in a valid state whenever practical

Do not combine unrelated features, refactorings, formatting changes or
documentation changes into a single commit merely for convenience.

Prefer several focused commits over one large commit.

Commits should make it possible to understand what changed and why
without reconstructing the entire development session.

Do not create commits merely to record intermediate agent activity.

### Branches

Before making changes, the agent must determine whether the user wants
the work performed on the current branch or in a separate branch.

If this has not already been specified, ask the user:

> Ska förändringarna göras i den aktuella branchen eller i en separat branch?

Do not create, switch to, rename or delete branches without the user's
explicit instruction.

Once the branch decision has been made, keep all related changes within
that branch unless the user instructs otherwise.

### Scope

A single task may result in multiple commits when the changes represent
distinct logical steps.

For example:

```text
feat: add room approval flow
test: add room approval scenarios
docs: update room user cases
```

Do not artificially split a single inseparable change merely to increase
the number of commits.

### Before Committing

Before creating a commit:

- Inspect the changes.
- Verify that unrelated changes are not included.
- Run the relevant tests and validation.
- Confirm that the commit represents the intended change.
- Use a clear commit message describing the change.

The agent must not commit unrelated user changes that were already present
in the working tree.

### User Changes

Never discard, overwrite or silently modify uncommitted user changes.

If existing changes make the requested work ambiguous or unsafe, stop and
ask the user before proceeding.

## 24. Guiding Principle

The purpose of these standards is not to maximize the number of rules.

The purpose is to make demo applications structurally predictable,
technically coherent and easy to reason about.

Prefer:

- explicit > implicit
- simple > clever
- observable behavior > implementation assumptions
- small abstractions > premature abstractions
- shared conventions > individual preferences

The architecture should be strong enough to provide boundaries without
becoming an excuse for unnecessary complexity.
