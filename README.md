# globallife-ai

A front-end for a relocation platform: visa guidance and language learning in one dashboard,
built as a design and architecture exercise in Next.js 16 and React 19.

~7,100 lines of TypeScript · Next.js 16 · React 19 · NextAuth · Three.js · Tailwind · Zustand

> **Read this first.** This is a front-end build. Authentication is real — Google OAuth through
> NextAuth. The product surfaces are not backed by services yet: the support chatbot answers
> from a hardcoded response map, and the visa and lesson screens render designed states rather
> than live data. Details in [Status](#status). The name predates that decision and overstates
> what ships today.

---

## What's in it

**Three product areas**, each a route group under `/dashboard`:

| Area | Routes |
|---|---|
| Visa | `/visa` · `/visa/countries` · `/visa/documents` · `/visa/roadmap` · `/visa/chat` |
| Language | `/language` · `/language/lesson` · `/language/ielts` · `/language/progress` |
| Account | `/assistant` · `/stats` · `/settings` · `/support` · `/support/voice` |

Plus a marketing landing page and a three-step auth flow (`/auth/login`, `/auth/register`,
`/auth/onboarding`).

## Authentication

NextAuth with the Google provider. Two callbacks do the work worth noting:

```ts
async session({ session, token }) {
    if (session.user) {
        (session.user as Record<string, unknown>).id = token.sub;
    }
    return session;
}
```

NextAuth does not put the account id on the session by default, so `token.sub` is copied onto
`session.user.id` — without it the client has an email and no stable key to hang anything off.

The `redirect` callback keeps sign-in returns on-origin: relative URLs are resolved against
`baseUrl`, same-origin absolute URLs pass through, and anything else falls back to
`/dashboard`. That last branch is the point — it means a crafted `callbackUrl` cannot bounce a
signed-in user off to another host.

## 3D and motion

`three` driven directly rather than through a renderer abstraction, in four scene components:

- `Globe3D` — the landing hero
- `ParticleField` — ambient background points
- `SceneAuthentication` / `SceneBackground` — scene wrappers per context

`Scene3DWrapper` isolates the canvas so the WebGL context mounts once and the surrounding React
tree can re-render without tearing it down. Page transitions and micro-interactions are Framer
Motion; UI primitives are Radix, styled with Tailwind and merged through `tailwind-merge`.

## Internationalisation

Four locales — Turkish, English, German, Polish — in `src/lib/i18n.ts`. Translations are typed
as a nested object keyed by locale rather than flat string ids:

```ts
nav: {
    features: { tr: "Özellikler", en: "Features", de: "Funktionen", pl: "Funkcje" },
}
```

The trade-off is deliberate: TypeScript catches a missing translation at compile time, because
a locale key absent from the object is a type error rather than a silent fallback at runtime.
It does not scale to a translator workflow, which would want extracted message files.

## State

Zustand for cross-route client state. No server state library — there is no server state yet.

---

## Status

What is wired:

- Google OAuth sign-in, session handling, protected dashboard routes
- All screens, 3D scenes, transitions, responsive layouts
- Locale switching across four languages

What is not:

- **The support chatbot is scripted.** `AIChatBot.tsx` matches the first and last word of the
  input against a map of five canned answers and replies after a `setTimeout` that imitates
  typing latency. There is no model call.
- `@ai-sdk/openai` and `ai` are in `package.json` but not imported anywhere. They were added
  for the integration that has not happened.
- Visa roadmaps, document checklists, lessons and IELTS practice render static content. There
  is no backend and no database.

Wiring the chatbot to a real model is the next step; the SDK is already a dependency and the
only API route in the project is NextAuth's.

---

## Running it

```bash
git clone https://github.com/Tunaycel/globallife-ai
cd globallife-ai
npm install
```

`.env.local`:

```
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
NEXTAUTH_SECRET=      # openssl rand -base64 32
NEXTAUTH_URL=http://localhost:3000
```

Google credentials come from a Google Cloud OAuth client with
`http://localhost:3000/api/auth/callback/google` as an authorised redirect URI.

```bash
npm run dev     # http://localhost:3000
npm run build
```
