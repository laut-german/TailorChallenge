# Tailor Fullstack Challenge

## The Challenge

Your task is to build a restaurants app where users can discover restaurants, manage favourites, leave comments, and create table reservations.

Reservations are based on time-slot availability. Each restaurant has a limited number of seats available per time slot, and every reservation reduces the available capacity for that specific restaurant, date, and time.

We are interested in a pragmatic, well-structured solution. Prioritise clear domain modelling, good API contracts, a usable frontend flow, and realistic tests over unnecessary complexity.

## Backend

Build a REST API in Node.js using Express, Fastify, NestJS, or a similar framework.

TypeScript is expected. If you decide not to use it, explain why in the README.

Persistence is optional. You may keep the data in memory, load it from JSON files, use a local database, or use a hosted free tier such as Supabase (https://supabase.com/), Neon (https://neon.com/), or MongoDB Atlas (https://www.mongodb.com/products/platform/atlas-database).

Whichever option you choose, keep the data model clear and intentional. If you use a database, define the schema and manage migrations with an ORM or migration tool of your choice, such as Prisma, Drizzle, TypeORM, or similar.

If you use in-memory or JSON-based storage, keep data access isolated behind repositories or services instead of coupling the business logic directly to arrays or files. We are more interested in clean boundaries and maintainable code than in a specific persistence technology.


### Required Features

1. Restaurants

- Return the provided `restaurants.json` payload, an adapted version of it, or your own mock dataset.
- Implement CRUD endpoints for restaurants.
- Each restaurant should include enough information to render the list and detail views, such as name, description, address, image, rating, capacity, and cuisine type.
- Each restaurant should define reservation settings, including service windows, slot interval, and default capacity per slot.
- The provided seed already includes reservation-related fields such as `capacity` and `reservationSettings`.

2. Users and Authentication

- Create a `User` model with username, password, and favourite restaurants.
- Implement login/authentication for the API.
- Protect user-specific endpoints such as favourites and reservations.
- JWT authentication is recommended, but a simpler token-based approach is acceptable if clearly documented.

3. Favourites

- Users should be able to add and remove restaurants from their favourites list.
- Users should be able to retrieve their favourite restaurants.

4. Comments

- Implement CRUD endpoints for restaurant comments.
- Comments should belong to a restaurant and a user.
- The frontend should be able to create, edit, and delete comments.

5. Availability

- Users should be able to check available reservation slots for a restaurant on a specific date.
- Availability should be generated from the restaurant reservation settings.
- The default service windows are lunch from `13:00` to `15:00` and dinner from `20:00` to `23:00`, split into 30-minute slots.
- Service window start times are inclusive and end times are exclusive. For example, `13:00` to `15:00` with 30-minute slots generates `13:00`, `13:30`, `14:00`, and `14:30`.
- Each slot has a capacity. Existing reservations or seeded booked slots reduce the available seats.
- If a future date has no existing bookings, the API should still generate the available slots from the restaurant settings.
- Some slots in the seed data should already be partially booked or fully booked so the UI can display unavailable times.
- Do not pre-create every future date in the dataset. Future dates should be generated on demand from `reservationSettings`.
- The frontend should ask the user for a date and party size before displaying time slots.
- The user should only be able to select a time slot that can fit the requested party size.

The intended frontend flow is:

```txt
Restaurant detail
-> user selects date
-> user selects party size
-> frontend requests availability
-> user selects an available time slot
-> frontend creates the reservation
```

Availability should be calculated from:

```txt
restaurant.reservationSettings.bookedSlots
+ confirmed reservations created by users
- cancelled reservations
```

You may store new reservations separately instead of mutating the original seed data.

Example response:

```json
{
  "restaurantId": 1,
  "date": "2026-07-10",
  "slots": [
    {
      "time": "13:00",
      "capacity": 8,
      "reservedSeats": 8,
      "availableSeats": 0,
      "available": false
    },
    {
      "time": "13:30",
      "capacity": 8,
      "reservedSeats": 3,
      "availableSeats": 5,
      "available": true
    }
  ]
}
```

6. Reservations

- Users should be able to create a reservation for a restaurant.
- A reservation should include restaurant, user, date, time, party size, and status.
- Store `date` and `time` as separate fields. For example, `date: "2026-07-10"` and `time: "13:30"`.
- Use these reservation statuses:

```ts
type ReservationStatus = "confirmed" | "cancelled";
```

- Creating a reservation should immediately consume capacity for the selected restaurant, date, and time.
- Users should be able to list their own reservations.
- Users should be able to cancel their own reservations.
- Cancelling a reservation should make those seats available again.

Example create reservation request:

```json
{
  "restaurantId": 1,
  "date": "2026-07-10",
  "time": "13:30",
  "partySize": 4
}
```

Example reservation response:

```json
{
  "id": "res_123",
  "restaurantId": 1,
  "userId": 1,
  "date": "2026-07-10",
  "time": "13:30",
  "partySize": 4,
  "status": "confirmed"
}
```

The backend must recalculate availability when creating a reservation. The frontend availability response is useful for the UI, but it must not be trusted as the source of truth.

### Business Rules

Implement at least the following rules:

- A reservation cannot be created in the past.
- A reservation time must match one of the restaurant's generated slots.
- Party size must be greater than zero.
- Party size cannot exceed the available seats for the selected slot.
- Creating a reservation reduces the available seats for the same restaurant, date, and time.
- Cancelling a reservation releases its seats back to the selected slot.
- A cancelled reservation cannot be cancelled again.
- Availability must be scoped by restaurant, date, and time.
- Availability must be recalculated server-side before accepting a reservation.

### Suggested API Endpoints

You may adjust the exact shape if your README explains the decisions.

```txt
POST   /auth/login

GET    /restaurants
GET    /restaurants/:id
POST   /restaurants
PATCH  /restaurants/:id
DELETE /restaurants/:id

GET    /me/favourites
POST   /me/favourites/:restaurantId
DELETE /me/favourites/:restaurantId

GET    /restaurants/:restaurantId/comments
POST   /restaurants/:restaurantId/comments
PATCH  /comments/:commentId
DELETE /comments/:commentId

GET    /restaurants/:restaurantId/availability?date=YYYY-MM-DD&partySize=4

POST   /reservations
GET    /me/reservations
GET    /reservations/:reservationId
PATCH  /reservations/:reservationId/cancel
```

## Frontend

Create a Next.js app that consumes your API.

### Required Features

- Display an initial list of restaurants.
- Allow the user to view restaurant details.
- Allow the user to create, edit, and delete restaurants.
- Allow the user to create, edit, and delete comments.
- Allow the user to log in.
- Allow the user to manage favourite restaurants.
- Allow the user to select a restaurant, date, and party size to check available reservation times.
- Display available and unavailable time slots clearly.
- Allow the user to create a reservation for an available slot.
- Display the user's reservations and their current status.
- Allow the user to cancel a reservation.
- Display loading states while API requests are in progress.
- Display empty states and error states where appropriate.
- Use a state management or server-state library where it makes sense.
- Make it responsive and visually appealing.

## Figma Design

Use this Figma file as a visual reference:

https://www.figma.com/file/LuwjRZZb3ms0MeAmu7gZch/Tailor-Prueba-t%C3%A9cnica-Junior?type=design&node-id=2%3A15&mode=design&t=mYbsPNUJscBcb7yN-1

The Figma design is only a small visual guide. You may reuse its style, layout ideas, typography, spacing, colours, and restaurant screens, but you are expected to adapt it and add whatever UI is missing to support the required functionality, especially date selection, party size, availability slots, reservation creation, and reservation management.

Pixel-perfect implementation is not required. A coherent, usable, responsive interface is more important than matching every detail of the provided design.

## Project Structure

You may organise the solution in any of the following ways:

- A monorepo containing both backend and frontend apps.
- Two separate repositories, one for the API and one for the frontend.
- A single repository with clearly separated folders, for example `api/` and `web/`.

Monorepo tooling such as Bun workspaces, npm/pnpm/yarn workspaces, Turborepo, Nx, or similar tools is accepted. Use whatever setup helps you keep the solution simple, easy to run, and easy to review.

If you use multiple apps or repositories, document clearly how to install dependencies, run each app, run tests, and verify the main flow locally.

## AI Usage

Using AI tools to complete the challenge is accepted and encouraged. We care about how you use them, how you validate the output, and whether you can explain the final solution.

If you use AI, include a short section in the README explaining:

- Which AI tools or agents you used.
- What parts of the solution they helped with.
- What you reviewed, changed, or rejected.
- Any limitations or risks you identified in the generated output.

You may also include supporting evidence such as an `AGENTS.md`, custom agent instructions, skills, prompts, or notes used during the implementation. This is not required to be perfect, but it should make your workflow transparent.

We will review the code structure, the AI usage notes, and your ability to explain the implementation during the technical discussion. Do not submit code you cannot reason about.

## Deliverables

- Push the code to a public GitHub repository.
- Include a `README.md` explaining how to run the backend and frontend apps.
- Include any required environment variables or setup steps.
- Include sample credentials or seed data needed to test the app.
- Include a short explanation of your technical decisions and trade-offs.
- If AI tools were used, include the AI usage notes described above.

## Bonus Points

- Deploy the app.
- Use JWT authentication.
- Use Tailwind CSS.
- Add a persistence layer such as SQLite, PostgreSQL, MongoDB, or similar.
- Add OpenAPI/Swagger documentation.
- Add Docker or Docker Compose.
- Write realistic unit tests for backend business rules.
- Write integration tests for API endpoints.
- Write end-to-end tests for the main frontend flow.
- Add basic accessibility improvements.
- Handle concurrent reservation attempts for the same slot.

## Non-Goals

You do not need to implement:

- Deposits or checkout flows.
- Webhooks.
- Email notifications.
- Real-time updates.
- A complex admin panel.
- Perfect visual parity with the Figma file.

## What We Will Evaluate

- Correctness of the main user flows.
- API design and request validation.
- Error handling.
- Type safety.
- Clear separation of concerns.
- How reservation availability and state are modelled.
- Frontend usability and responsiveness.
- Loading, empty, and error states.
- Test quality and relevance.
- README quality and developer experience.
