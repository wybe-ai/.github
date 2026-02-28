# Wybe — Arkitekturdokument

## 1. Hva er Wybe

Wybe er en AI-kollega du ansetter — ikke et verktøy du bruker. Svarer telefonen, følger opp leads, booker møter, sender epost, oppdaterer CRM. Proaktiv, multi-kanal, jobber 24/7.

Chat er bare én måte å snakke med Wybe. Det meste skjer autonomt: et lead kommer inn fra nettsiden → Wybe ringer tilbake innen 2 minutter. En kunde sender epost → Wybe sjekker kalender, lager svar, sender det. Noen ringer kontoret → Wybe tar telefonen, har en naturlig samtale, logger samtalen, lager oppfølgingsoppgaver.

Hjernen (`alive-core`) lever i skyen. Sover aldri. `wybe-os` er kroppen — kjører på en Mac Mini hos kunden, eller hostet i skyen.

---

## 2. Produktvisjon

### 5 kanaler

| Kanal | Retning | Beskrivelse |
|-------|---------|-------------|
| **Chat** (app.wybe.me) | Begge | Ansatt spør Wybe om å lage utkast til kontrakt |
| **Telefon** | Begge | Megler ringer Wybe fra bilen: "Visningen er ferdig, kan du lage kontrakten?" |
| **SMS** | Begge | Ansatt sender "påminn meg om å ringe Hansen kl 15" → Wybe bekrefter |
| **Kontor** | Begge | Walk-in: "God morgen! Du har to visninger i dag." |
| **Epost** | Begge | Kunde sender epost → Wybe svarer. Ansatt videresender: "Kan du oppsummere?" |

### Autonomi-modell

Hver handling kan konfigureres uavhengig:

| Nivå | Emoji | Oppførsel |
|------|-------|-----------|
| Autonomous | 🟢 | Wybe gjør det. Ingen godkjenning nødvendig. |
| Inform | 🟡 | Wybe gjør det, og varsler relevant person. |
| Draft | 🟠 | Wybe forbereder handlingen, mennesket godkjenner. |
| Disabled | 🔴 | Wybe kan ikke gjøre dette. |

14 handlingstyper: `answer_call`, `make_call`, `send_email`, `send_sms`, `book_meeting`, `update_crm`, `create_document`, `respond_to_lead`, `manage_calendar`, `draft_contract`, `create_listing`, `send_reminder`, `log_activity`, `escalate`

### 3 produkttiers

| Feature | Starter | Pro | Enterprise |
|---------|---------|-----|------------|
| Chat + hjerne | Ja | Ja | Ja |
| Minne & læring | Ja | Ja | Ja |
| Emosjonell intelligens | Ja | Ja | Ja |
| Wybe OS | Nei | Hostet i sky | Mac Mini on-site |
| Telefon (inn/ut/team) | Nei | Ja | Ja |
| SMS/Epost/Kalender | Nei | Ja | Ja |
| CRM-integrasjon | Nei | Nei | Ja |
| Autonomi-konfig | Basis | Per handling | Full kontroll |
| Kontortilstedeværelse | Nei | Nei | Ja |
| Teammedlemmer | 3 | 10 | Ubegrenset |
| Bevissthet | Delt | Dedikert | Dedikert + on-prem |

---

## 3. Repo-oversikt

### alive-core — Hjernen

| | |
|---|---|
| **Formål** | Kognitiv motor. 36 engines, bevissthet, minne, emosjoner, beslutninger. |
| **Tech stack** | TypeScript, Hono 4.7, Node.js 22, Drizzle ORM, Neon PostgreSQL + pgvector, Anthropic Claude SDK, node-llama-cpp, HuggingFace Transformers |
| **Deploy** | Fly.io (`alive-core`, region `ams`, 1 GB RAM, alltid på) |
| **Port** | 3210 |
| **Lokal sti** | `/Users/ivaraasen/alive-core/` |

### wybe-os — Kroppen

| | |
|---|---|
| **Formål** | Executor-service. Mottar intents fra alive-core via WebSocket, utfører dem. |
| **Tech stack** | TypeScript, Node.js, ws, Twilio, Nodemailer, Google APIs |
| **Executors** | call, sms, notify, email, calendar, crm, shell |
| **Deploy** | Mac Mini on-site (Enterprise) eller Fly.io (Pro) |
| **Lokal sti** | `/Users/ivaraasen/wybe-os/` |

### wybe-app — Kundeappen

| | |
|---|---|
| **Formål** | Kundevendt SaaS-app. Onboarding, chat, dashboard, admin, soul editor. |
| **Tech stack** | Next.js 14 (App Router), TypeScript, Tailwind CSS, Supabase, Anthropic Claude SDK, dnd-kit |
| **Deploy** | Vercel |
| **Lokal sti** | `/Users/ivaraasen/wybe-app/` |

### wybe-hq — Internt dashboard

| | |
|---|---|
| **Formål** | Intern "command center" for teamet. Chat med Wybe som CTO, sprint, agenter, økonomi, strategi. |
| **Tech stack** | Next.js 14 (App Router), TypeScript, Tailwind CSS, Supabase |
| **Deploy** | TBD (Vercel-kompatibel) |
| **Lokal sti** | `/Users/ivaraasen/wybe-hq/` |

### webpage — Landing page

| | |
|---|---|
| **Formål** | Markedsside for wybe.me |
| **Tech stack** | Vanilla HTML, Tailwind CSS, PostCSS |
| **Deploy** | Vercel |
| **Lokal sti** | `/Users/ivaraasen/webpage/` |

---

## 4. 3-lags arkitektur

```
┌─────────────────────────────────────────────────────┐
│  alive-core (Hjernen)                               │
│  Cloud — Fly.io — alltid på                         │
│  36 engines, bevissthet, minne, emosjoner           │
│  Tar ALLE beslutninger, kontrollerer alt            │
└────────────────────────┬────────────────────────────┘
                         │ /ws/wybe-os (WebSocket)
                         │ Intents ↓  Feedback ↑
┌────────────────────────┴────────────────────────────┐
│  Wybe OS (Kroppen)                                  │
│  Mac Mini on-site (Enterprise)                      │
│  — eller hostet på Fly.io (Pro)                     │
│  Utfører intents, TTS/STT, kamera, verktøy-APIer   │
│  Bestemmer aldri — utfører kun det hjernen sier     │
└────────────────────────┬────────────────────────────┘
                         │ APIer + hardware I/O
┌────────────────────────┴────────────────────────────┐
│  Verktøy & kanaler                                  │
│  Twilio (telefon/SMS), Gmail/Outlook, Kalender      │
│  CRM, webhooks, høyttaler, kamera, display          │
└─────────────────────────────────────────────────────┘
```

**Dashboards** (wybe-app og wybe-hq) kobler til alive-core via REST/SSE for chat, konfigurasjon, og orkestrering. De er observatører og konfiguratorer — ikke en del av den kognitive loopen.

---

## 5. Dataflyt

```
Bruker (telefon/SMS/epost/chat/lead)
  │
  ▼
alive-core mottar event
  │ POST /api/events/{type} eller /api/v1/messages
  ▼
Kognitiv loop prosesserer:
  TextInput → Perception → Attention → Binder → Arbiter
  + parallelt: EmotionInference, ToM, Memory, Strategy...
  │
  ▼
Arbiter beslutter handling
  │ Sjekker autonomi-nivå (autonomous/inform/draft/disabled)
  ▼
┌─ autonomous/inform ──→ wybe-os (WebSocket execute-intent)
│                           │
│                           ▼
│                        Executor (Twilio/Gmail/Calendar/CRM)
│                           │
│                           ▼
│                        Feedback → alive-core
│
└─ draft ──→ Supabase colleague_activity (pending_approval)
               │
               ▼
            wybe-app viser i dashboard
               │ Admin godkjenner
               ▼
            POST /api/wybe-os/execute → wybe-os
```

---

## 6. Hosting & infrastruktur

| Komponent | Plattform | Detaljer |
|-----------|-----------|----------|
| **alive-core** | Fly.io | App: `alive-core`, region: `ams`, 1 GB RAM, 1 shared CPU, auto-stop OFF, min 1 maskin |
| **wybe-app** | Vercel | Next.js, auto-deploy fra main |
| **wybe-hq** | TBD | Vercel-kompatibel Next.js |
| **wybe-os** (Pro) | Fly.io | Hostet instans per kunde |
| **wybe-os** (Enterprise) | Mac Mini | On-site hos kunden |
| **webpage** | Vercel | Statisk side |
| **Supabase** | Supabase Cloud | Auth, konfig, aktivitet, tool_connections |
| **Neon** | Neon Cloud | PostgreSQL + pgvector for minne, samtaler, agenter |

---

## 7. Databaser

### Supabase (auth, konfig, aktivitet)

| Tabell | Formål |
|--------|--------|
| `companies` | Firmaregister — navn, bransje, vertical, lifecycle_stage, onboarding_data, kontaktinfo |
| `profiles` | Brukerprofiler — extends auth.users, rolle, company_id, is_company_admin, is_wybe_admin |
| `onboarding` | Onboarding-status per bruker — steg, status, oppgaver |
| `colleague_configs` | ColleagueProfile + WybeOverrides per firma (JSONB) |
| `tool_connections` | Integrasjoner per firma — type, provider, krypterte credentials, status |
| `wybe_os_instances` | Mac Mini/hostet instanser — instance_id, auth_token, type, status, manifest |
| `colleague_activity` | Audit trail — action_type, summary, autonomy_level, godkjenning |
| `conversations` | Chat/samtale-records |
| `chat_messages` | Meldinger i samtaler |
| `memory_bits` | Wybes minne — kategori, scope (shared/personal), tekst, confidence |
| `agent_configs` | Kompilert agent-markdown per firma |
| `config_snapshots` | Admin-genererte config-snapshots |
| `deploy_jobs` | Deploy-jobb-tracking |
| `setup_sessions` | Token-gated kundeoppsettsøkter |
| `waitlist` | Venteliste-påmeldinger |
| `admin_audit_log` | Admin audit trail |

### Neon (PostgreSQL + pgvector)

| Tabell | Formål |
|--------|--------|
| `users` | Brukerkontoer, tier, kryptert API-nøkkel |
| `conversations` | Samtalesøkter med kanal |
| `messages` | Meldingshistorikk med kognitive metadata og emotion_shift |
| `cognitive_states` | Per-bruker selvtilstand (valence, arousal, confidence, energy, social, curiosity) |
| `memories` | Vektorminne — type (episodic/semantic/procedural/person), embedding vector(384), scope |
| `relationships` | Per-person kognitiv + relasjonsmodell (12 dimensjoner + ToM) |
| `consciousness_snapshots` | Periodiske tilstandslagringer for kontinuitet over restarter |
| `scheduled_jobs` | Planlagte oppgaver med cron |
| `usage_records` | API-brukssporing |
| `agent_registry` | Agent-teamregister + ToM-modell (accuracy, speed, reliability) |
| `sprint_items` | Sprint kanban (tittel, status, prioritet, assignee, PR-url) |
| `finance_records` | Utgiftssporing (NOK) |
| `finance_budget` | Månedlig budsjett |
| `strategic_goals` | OKR-stil strategiske mål |

---

## 8. API-endepunkter

### alive-core (port 3210)

**Autentisering:** Alle endepunkter (unntatt health og Twilio-webhooks) krever `x-api-key` header.

#### Kjerne
| Metode | Sti | Beskrivelse |
|--------|-----|-------------|
| GET | `/api/health` | Helsesjekk |
| GET | `/api/state` | Full bevissthetstilstand |
| POST | `/api/v1/messages` | Chat — Anthropic-kompatibel proxy med kognitiv berikelse. Støtter `stream: true` |
| POST | `/api/mind/think` | Direkte tenk-kall |
| POST | `/api/mind/tom` | Theory of Mind-inferens |
| POST | `/api/config` | Push ColleagueProfile + WybeOverrides |
| GET | `/api/config` | Les gjeldende konfig |

#### Events (system-inputs)
| Metode | Sti | Beskrivelse |
|--------|-----|-------------|
| POST | `/api/events/lead` | Nytt lead |
| POST | `/api/events/email` | Innkommende epost |
| POST | `/api/events/call` | Samtalehendelse |
| POST | `/api/events/sms` | Innkommende SMS |
| POST | `/api/events/calendar` | Kalenderhendelse |
| POST | `/api/events/schedule` | Planlagt oppgave-trigger |

#### Minne
| Metode | Sti | Beskrivelse |
|--------|-----|-------------|
| POST | `/api/memory/sync` | Synk memory bits fra wybe-app |
| POST/GET | `/api/memory/search` | Semantisk søk |

#### Orkestrering
| Metode | Sti | Beskrivelse |
|--------|-----|-------------|
| GET/POST/DELETE | `/api/orchestration/agents` | Agent-register CRUD |
| GET/POST/PATCH | `/api/orchestration/sprint` | Sprint-items CRUD + auto-assign |
| GET/POST | `/api/orchestration/finances` | Budsjett + utgifter |
| GET/POST/PATCH/DELETE | `/api/orchestration/goals` | Strategiske mål CRUD |
| GET | `/api/orchestration/context` | Full operasjonell kontekst |

#### Wybe OS
| Metode | Sti | Beskrivelse |
|--------|-----|-------------|
| GET | `/api/wybe-os/status` | Tilkoblede instanser |
| POST | `/api/wybe-os/execute` | Utfør godkjent draft-handling |

#### Twilio Webhooks (ingen API-key — Twilio-signaturvalidering)
| Metode | Sti | Beskrivelse |
|--------|-----|-------------|
| POST | `/api/twilio/voice/inbound` | Innkommende samtale → TwiML |
| POST | `/api/twilio/voice/gather` | Talegjenkjenningsresultat |
| POST | `/api/twilio/sms/inbound` | Innkommende SMS → auto-svar |

#### WebSockets
| Sti | Beskrivelse |
|-----|-------------|
| `/ws` | Observability dashboard |
| `/ws/wybe-os` | wybe-os-tilkobling (autentisert) |

### wybe-app (Next.js API routes)

| Sti | Beskrivelse |
|-----|-------------|
| `POST /api/chat` | Stream AI-respons (ruter: alive → gateway → Claude) |
| `GET /api/activity` | Kollegaaktivitet |
| `GET/PUT /api/colleague-config` | Les/skriv ColleagueProfile |
| `GET/POST /api/tool-connections` | Verktøyintegrasjoner |
| `POST /api/compile-agent` | Kompiler memory bits til agent-markdown |
| `GET /api/dashboard/company-summary` | Dashboard-statistikk |
| `POST /api/waitlist` | Venteliste-påmelding |
| `POST /api/setup/[token]/chat` | Kundeoppsettshat (token-gated) |
| `* /api/admin/*` | Admin-endepunkter (krever is_wybe_admin) |

---

## 9. Konfigurasjon

### ColleagueProfile (settes av firmadmin i wybe-app)

```typescript
{
  displayName: string           // "Karoline" — persona-navn
  personality: string           // Fritekst personlighetsbeskrivelse
  tone: "warm" | "professional" | "casual"
  language: "no" | "en" | "auto"
  responsibilities: ActionType[] // Hvilke handlinger denne kollegaen håndterer
  workingHours: {
    schedule: "always" | "business" | "custom"
    timezone: string
    custom?: { days: number[]; start: string; end: string }
  }
  escalation: {
    defaultPerson: string
    rules: EscalationRule[]     // situasjon → handling → mål → kanal
  }
  autonomy: Record<ActionType, AutonomyLevel>  // per handling
  companyInfo: string           // Fritekst firmakunnskap
  faqs: { question, answer }[]
  scripts: { name, trigger, talkingPoints[] }[]
  teamMembers: { name, role, phone?, email? }[]
}
```

### WybeOverrides (settes av Wybe-admin)

```typescript
{
  tier: "starter" | "pro" | "enterprise"
  capabilities: ActionType[]    // Hva denne tier tillater
  models: { primary, fast }
  safety: {
    maxTokensPerRequest: number
    maxCallDurationSeconds: number
    maxOutboundCallsPerDay: number
    maxEmailsPerDay: number
    maxMonthlyCostNok: number
    blockedActions: ActionType[]
    requireApprovalOverride: ActionType[]
    contentFilter: "standard" | "strict"
    allowedLanguages: string[]
  }
  features: Record<string, boolean>
  billing: { monthlyPriceNok: number }
}
```

### Autonomi-oppløsning (prioritet)

```
1. blockedActions?          → disabled (Wybe-admin veto)
2. Ikke i capabilities?     → disabled (tier-begrensning)
3. requireApprovalOverride? → draft (Wybe-admin overstyring)
4. profile.autonomy[action] → firma-konfigurert nivå
5. Ingen konfig             → disabled (default)
```

---

## 10. Agent-teamet

| Agent | Modell | Rolle | Eier (kode) |
|-------|--------|-------|-------------|
| **Wybe** | alive-core/cognitive-loop | CTO & orkestrator. Sprint-planlegging, code review, oppgave-routing, minne. | Alt |
| **Erling** | Claude Opus 4.6 | Backend. Engines, signal-bus, API, minne, TypeScript. | `alive-core/src/` |
| **Nora** | Claude Opus 4.6 | Voice, NLP, norsk, prompts, TwiML, expression. | Engines, executors |
| **Saga** | GPT-5.3 Codex | Frontend. Next.js, React, Tailwind, Supabase, UI/UX. | `wybe-app/` |
| **Astrid** | Claude Sonnet 4.6 | Typer, schemas, migrasjoner, OpenAPI, arkitektur, kontrakter. | types.ts, schema.ts |
| **Ragnar** | Claude Sonnet 4.6 | DevOps. Docker, CI/CD, deploy, testing, monitoring, executors. | `wybe-os/`, Dockerfile |

Hvert agent har et ToM-system (Theory of Mind) som tracker `accuracy`, `speed`, `reliability`, `strengths`, `weaknesses` basert på de siste 20 oppgaveresultatene. Stuck-detection trigger etter 2 timer i `working`-status.

Oppgave-routing: `autoAssignTask()` matcher oppgavens capabilities mot agentens og velger den med best reliability + lavest nåværende belastning.

---

## 11. Miljøvariabler

### alive-core
| Variabel | Påkrevd | Beskrivelse |
|----------|---------|-------------|
| `ANTHROPIC_API_KEY` | Ja | Claude API-nøkkel |
| `DATABASE_URL` | Ja (prod) | Neon PostgreSQL connection string |
| `ALIVE_API_KEY` | Ja (prod) | API-nøkkel for alle endepunkter |
| `SUPABASE_URL` | Anbefalt | Supabase prosjekt-URL |
| `SUPABASE_SERVICE_KEY` | Anbefalt | Supabase service role-nøkkel |
| `TWILIO_AUTH_TOKEN` | For Twilio | Validerer Twilio webhook-signaturer |
| `TWILIO_WEBHOOK_BASE_URL` | For Twilio | Offentlig base-URL for Twilio |
| `WYBE_OS_AUTH_TOKEN` | Fallback | Auth-token hvis Supabase ikke er konfigurert |
| `LOCAL_MODEL_PATH` | Valgfri | Sti til GGUF-modell for brainstem |
| `PORT` | Valgfri | Server-port (default: 3210) |

### wybe-app
| Variabel | Påkrevd | Beskrivelse |
|----------|---------|-------------|
| `NEXT_PUBLIC_SUPABASE_URL` | Ja | Supabase prosjekt-URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Ja | Supabase anon-nøkkel |
| `SUPABASE_SERVICE_ROLE_KEY` | Ja | Supabase service role-nøkkel |
| `ANTHROPIC_API_KEY` | Ja | Claude API-nøkkel (fallback for direkte kall) |
| `ALIVE_API_URL` | Valgfri | URL til alive-core (aktiverer kognitivt ruting) |
| `ALIVE_API_KEY` | Valgfri | API-nøkkel for alive-core |

### wybe-hq
| Variabel | Påkrevd | Beskrivelse |
|----------|---------|-------------|
| `NEXT_PUBLIC_SUPABASE_URL` | Ja | Supabase prosjekt-URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Ja | Supabase anon-nøkkel |
| `ALIVE_CORE_URL` | Nei | URL til alive-core (default: localhost:3210) |
| `ALIVE_API_KEY` | Nei (prod ja) | API-nøkkel for alive-core |

### wybe-os
| Variabel | Påkrevd | Beskrivelse |
|----------|---------|-------------|
| `ALIVE_WS_URL` | Ja | WebSocket URL til alive-core |
| `INSTANCE_ID` | Ja | Unik instans-ID |
| `AUTH_TOKEN` | Ja | Autentiseringstoken |
| `TWILIO_ACCOUNT_SID` | For telefon | Twilio konto-SID |
| `TWILIO_AUTH_TOKEN` | For telefon | Twilio auth token |
| `TWILIO_PHONE_NUMBER` | For telefon | Twilio telefonnummer |
| `EMAIL_USER` | For epost | Gmail-bruker |
| `EMAIL_PASS` | For epost | Gmail app-passord |

---

## 12. Lokal utvikling

### Forutsetninger
- Node.js 22+
- npm
- Git
- GitHub CLI (`gh`)

### Oppsett

```bash
# 1. Klone alle repos
gh repo clone wybe-ai/alive-core
gh repo clone wybe-ai/wybe-os
gh repo clone wybe-ai/wybe-app
gh repo clone wybe-ai/wybe-hq
gh repo clone wybe-ai/webpage

# 2. Installer avhengigheter
cd alive-core && npm install && cd ..
cd wybe-os && npm install && cd ..
cd wybe-app && npm install && cd ..
cd wybe-hq && npm install && cd ..
cd webpage && npm install && cd ..

# 3. Konfigurer miljøvariabler
# Kopier .env.example til .env i hvert repo og fyll inn verdier
cp alive-core/.env.example alive-core/.env
cp wybe-os/.env.example wybe-os/.env
# wybe-app: .env.local
# wybe-hq: .env.local

# 4. Start alive-core (hjernen)
cd alive-core && npm run dev
# Kjører på http://localhost:3210

# 5. Start wybe-os (kroppen) — i ny terminal
cd wybe-os && npm run dev
# Kobler til alive-core via WebSocket

# 6. Start wybe-app (kundeappen) — i ny terminal
cd wybe-app && npm run dev
# Kjører på http://localhost:3000

# 7. Start wybe-hq (internt dashboard) — i ny terminal
cd wybe-hq && npm run dev
# Kjører på http://localhost:3001
```

### Rekkefølge
Start `alive-core` først (hjernen), deretter `wybe-os` (kobler seg til hjernen), og til slutt dashboardene.

---

## 13. Onboarding-flow for kunder

Onboarding eies av `wybe-app` og er en 8-stegs samtalebasert flow:

| Steg | ID | Hva skjer |
|------|----|-----------|
| 0 | `hello` | Viser "hei." i stor tekst. Auto-transition etter 2s. |
| 1 | `name` | "Jeg er Wybe. Hva heter du?" — Gjetter navn fra epost hvis mulig. |
| 2 | `greeting` | "Hyggelig å møte deg, [Fornavn]." — Auto-transition etter 2.4s. |
| 3 | `company` | "Hvor jobber du?" — Fritekst for firmanavn. |
| 4 | `vertical` | "Hva gjør [Firma]?" — 13 bransjetiles (Eiendom, Helse, Tannlege, Regnskap, Advokat, etc.) |
| 5 | `responsibilities` | "Hva skal Wybe hjelpe med?" — Checkbox-grid med handlingstyper, forhåndsvalgt basert på bransje. |
| 6 | `phone` | "La oss snakke — legg inn nummeret ditt." — Norsk telefonnummer. |
| 7 | `choice` | "Du velger." — To knapper: "Ring Wybe" eller "Vent på at Wybe ringer". |

Ved valg lagres alt til Supabase: `companies`, `profiles`, `colleague_configs`, `onboarding`. Første bruker per firma settes som `is_company_admin`.

---

## 14. Nåværende status

| Komponent | Status |
|-----------|--------|
| alive-core | ✅ Deployed på Fly.io. 36 engines, kognitiv loop, bevissthet, minne, streaming API. |
| wybe-app | ✅ Deployed på Vercel. Next.js, Supabase auth, admin dashboard, chat UI, onboarding. |
| alive-core ↔ wybe-app | ✅ Wired. Chat ruter gjennom alive-core med kognitiv berikelse. |
| wybe-os | ✅ Kjører. WebSocket-tilkobling til alive-core, executors for call/sms/email/calendar/crm/shell. |
| wybe-hq | ✅ Kjører. Internt command center med chat (tool-use), sprint, agenter, strategi. |
| ColleagueProfile config | 🔨 Ikke bygget (Phase 1). Typer eksisterer, men ingen UI for konfigurasjon. |
| Voice/Phone | 🔨 Ikke bygget (Phase 3). Twilio SDK er integrert, webhooks finnes, men pipeline ikke komplett. |
| Autonome handlinger | 🔨 Ikke bygget (Phase 4). Intent-dispatcher og autonomi-sjekk eksisterer, men end-to-end flow mangler. |
| Fysisk tilstedeværelse | 🔨 Delvis (Phase 5). HAL-typer i alive-core, Mac Mini-hardware finnes. |

---

## 15. Kjente issues

1. **ColleagueProfile er ikke bygget** — Typer eksisterer i koden, men det finnes ingen UI for firmaadmins til å konfigurere sin Wybe-instans. Uten dette vet ingen instans hvordan den skal oppføre seg.

2. **Voice pipeline mangler** — Twilio webhooks og SDK er satt opp, men end-to-end voice (innkommende samtale → tale-til-tekst → alive-core → tekst-til-tale → svar) er ikke ferdig.

3. **Autonome handlinger er ikke wired** — Intent-dispatcher og autonomi-nivåsjekk eksisterer, men hele flyten fra beslutning → wybe-os → utførelse → feedback er ikke komplett for alle handlingstyper.

4. **wybe-os mangler reconnect-robusthet** — Eksponentiell backoff finnes, men ingen persistent queue for intents som går tapt ved disconnect.

5. **Memory sync er enveis** — wybe-app synker bits til alive-core, men endringer i alive-core (via MemoryWrite-engine) synkes ikke tilbake.

6. **Ingen multi-tenant isolasjon i alive-core** — Kjører én kognitiv loop for alle kunder. For skalering trengs enten per-kunde instanser eller tenant-isolasjon i loopen.

7. **Mangler end-to-end testing** — Individuelle komponenter har tester (Vitest i alive-core), men ingen integrasjonstester som dekker hele flyten fra event → beslutning → utførelse → feedback.

---

*Sist oppdatert: Mars 2025*
