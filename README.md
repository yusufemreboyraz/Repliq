# Repliq

A quote-guessing quiz game built with Next.js. Players are shown a quote (with
optional image and voice recording) and have to guess who said it, who it was
said to, or which title it's from. Content comes from two categories stored in
the database: movie quotes and video game quotes.

Note: the project's GitHub description says "film and series quiz game," but
the actual data model (`FilmQuote` and `GameQuote` in `prisma/schema.prisma`)
is movies and video games, not TV series. This README describes what the code
actually does.

## How the game works

- **Singleplayer**: pick a difficulty (easy, medium, hard) and play through
  randomly generated questions. Difficulty controls how much media is shown
  alongside the quote:
  - Easy: quote text, image, and audio all shown.
  - Medium: quote text plus one randomly chosen media type (image or audio),
    if available.
  - Hard: quote text only.
- **Multiplayer**: create a room (gets a 6-character code) or join one with a
  code, then play the same quiz synchronously with other players. Room state,
  players, scores, and per-question answers are synced in real time through
  Firebase Realtime Database.
- **Question types**: for each quote, a question can ask the player to
  identify the character who said it, the recipient the quote was said to,
  or the movie/game title. Incorrect multiple-choice options are pulled from
  other distinct values in the same table.
- Game results are recorded to a `GameHistory` table (score, mode, timestamp)
  and can be viewed from the singleplayer/multiplayer history views.
- An admin section (`/admin`) exists for creating, editing, and managing
  quote entries and user requests.

## Tech stack

- **Framework**: Next.js 15 (App Router), React 19, TypeScript
- **Styling/UI**: Tailwind CSS v4, Radix UI primitives, shadcn-style
  components (`components/ui`), `lucide-react` icons
- **Database**: MySQL via Prisma ORM (`@prisma/client`, `prisma`)
- **Auth**: NextAuth (Credentials provider, bcrypt-hashed passwords), backed
  by the Prisma adapter
- **Realtime/multiplayer state**: Firebase Realtime Database
- **File/media storage**: Google Cloud Storage (`@google-cloud/storage`),
  used for quote images/voice recordings (see `lib/gcs.ts`)
- **Forms/validation**: React Hook Form + Zod

This is a web application, not a mobile app. There is no React Native/Expo
code in the repository.

## Project structure

```
app/                    Next.js App Router pages and API routes
  admin/                Admin UI for managing quotes, users, requests
  api/                  REST-style API routes (auth, questions, users, etc.)
  home/                 Main authenticated landing page (game mode selector)
  login/, register/     Auth pages
  singleplayer/play/    Singleplayer game screen
  multiplayer/          Lobby and in-game screens, keyed by room code
components/             Shared React components, including components/ui
                        (Radix-based UI primitives)
contexts/               React context providers (e.g. AuthContext)
hooks/                  Custom hooks
lib/                    Server/client utilities: Prisma client, NextAuth
                        config, Firebase client, GCS client, difficulty logic
prisma/                 Prisma schema and migrations (MySQL)
src/server/utils/       Quiz question generation logic (quizGenerator.ts)
src/pages/api/          One legacy Pages Router API route (test-question)
types/                  Shared TypeScript type declarations
```

The app mixes the App Router (`app/`) with a single leftover Pages Router API
route under `src/pages/api/`, which appears to be a test/dev endpoint rather
than something used by the UI.

## Setup

### Prerequisites

- Node.js
- A MySQL database
- A Firebase project with Realtime Database enabled
- A Google Cloud Storage bucket (for quote images/audio), if you intend to
  use media features

### Environment variables

The code reads the following environment variables (there is no committed
`.env.example`, so this list is derived from actual usage in `lib/`):

- `DATABASE_URL` — MySQL connection string for Prisma
- `NEXTAUTH_SECRET` — secret used by NextAuth
- `NEXT_PUBLIC_FIREBASE_API_KEY`
- `NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN`
- `NEXT_PUBLIC_FIREBASE_PROJECT_ID`
- `NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET`
- `NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID`
- `NEXT_PUBLIC_FIREBASE_APP_ID`
- `GCS_BUCKET_NAME`
- Either `GOOGLE_APPLICATION_CREDENTIALS` (path to a service account JSON), or
  `GCS_PROJECT_ID` + `GCS_CLIENT_EMAIL` + `GCS_PRIVATE_KEY` for GCS access

Create a `.env` file in the project root with these values before running
the app.

### Install dependencies

```bash
npm install
```

### Set up the database

Run Prisma migrations against your MySQL database:

```bash
npx prisma migrate deploy
npx prisma generate
```

(`npx prisma migrate dev` also works for local development and will keep
migrations in sync as the schema changes.)

The database starts empty — quote data (`FilmQuote` / `GameQuote` rows) needs
to be added manually or through the admin UI at `/admin/create`.

## Running the app

Start the development server (uses Turbopack):

```bash
npm run dev
```

The app runs at `http://localhost:3000`.

Other scripts defined in `package.json`:

```bash
npm run build   # runs `prisma generate` then `next build`
npm run start   # starts the production server (after build)
npm run lint    # runs next lint
```

## Known rough edges

- `quizGenerator.ts` contains verbose debug `console.log`/`console.dir` calls
  left in from development.
- Several files contain Turkish-language comments alongside English code and
  UI text.
- There is no automated test suite in this repository.
