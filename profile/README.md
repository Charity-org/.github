# Noor Elrahma Charity Platform

> A multilingual charity platform that connects public project discovery, donor accounts, donations and payment processing, favorites, dynamic content, and internal administration through a shared backend.

**Production website:** https://nouralrahma.com/en

Noor Elrahma is a multi-application platform rather than a single charity website.

The organization is built around three connected products:

- A **public Next.js application** used by visitors and donors.
- A **protected React administration dashboard** used by staff to manage projects and public content.
- A **NestJS/PostgreSQL backend** that owns authentication, projects, donations, payments, favorites, managed pages, contact requests, storage, and shared business data.

A fourth repository contains cross-project engineering documentation and workflow guidance.

The system is designed so public presentation and internal administration can evolve independently while still sharing one authoritative backend and database.

---

## Table of Contents

- [What Noor Elrahma Is](#what-noor-elrahma-is)
- [Platform Goals](#platform-goals)
- [Who Uses the Platform](#who-uses-the-platform)
- [System Architecture](#system-architecture)
- [Repository Map](#repository-map)
- [Public Web Platform](#public-web-platform)
  - [Public Application Responsibilities](#public-application-responsibilities)
  - [Public Route Model](#public-route-model)
  - [Authentication Experience](#authentication-experience)
  - [Project Discovery](#project-discovery)
  - [Donation Experience](#donation-experience)
  - [Favorites](#favorites)
  - [Profile and Authenticated Experience](#profile-and-authenticated-experience)
  - [News and Organizational Content](#news-and-organizational-content)
  - [Contact Flow](#contact-flow)
  - [Internationalization](#internationalization)
  - [SEO and Discoverability](#seo-and-discoverability)
  - [Public Frontend Technology](#public-frontend-technology)
  - [Public Frontend Quality Workflow](#public-frontend-quality-workflow)
- [Administration Dashboard](#administration-dashboard)
  - [Dashboard Responsibilities](#dashboard-responsibilities)
  - [Protected Route Structure](#protected-route-structure)
  - [Dashboard Overview](#dashboard-overview)
  - [Project Administration](#project-administration)
  - [Donator Views](#donator-views)
  - [About Us Management](#about-us-management)
  - [Homepage Management](#homepage-management)
  - [News Management](#news-management)
  - [Team Management](#team-management)
  - [Footer Management](#footer-management)
  - [Contact Request Management](#contact-request-management)
  - [Bilingual Content Editing](#bilingual-content-editing)
  - [Media Workflows](#media-workflows)
  - [Dashboard Technology](#dashboard-technology)
  - [Dashboard Engineering Workflow](#dashboard-engineering-workflow)
- [Backend API](#backend-api)
  - [Backend Responsibilities](#backend-responsibilities)
  - [NestJS Module Boundaries](#nestjs-module-boundaries)
  - [Application Bootstrap and HTTP Layer](#application-bootstrap-and-http-layer)
  - [Database and Drizzle ORM](#database-and-drizzle-orm)
  - [Authentication and Sessions](#authentication-and-sessions)
  - [Roles and Account Model](#roles-and-account-model)
  - [Projects Domain](#projects-domain)
  - [Managed Pages Domain](#managed-pages-domain)
  - [Donations Domain](#donations-domain)
  - [Payments Domain](#payments-domain)
  - [Payment Provider Routing](#payment-provider-routing)
  - [Payment Completion and Project Progress](#payment-completion-and-project-progress)
  - [Favorites Domain](#favorites-domain)
  - [Contact Domain](#contact-domain)
  - [Storage](#storage)
  - [Health and Operational Endpoints](#health-and-operational-endpoints)
  - [Validation](#validation)
  - [HTTP Security](#http-security)
  - [CORS and Client Integration](#cors-and-client-integration)
  - [Localization at the API Layer](#localization-at-the-api-layer)
  - [Backend Technology](#backend-technology)
  - [Database Migrations and Seeders](#database-migrations-and-seeders)
  - [Backend Testing](#backend-testing)
- [Core Data Model](#core-data-model)
  - [Users and Sessions](#users-and-sessions)
  - [Projects and Project Images](#projects-and-project-images)
  - [Donations](#donations)
  - [Payments](#payments)
  - [Managed Pages and Page Images](#managed-pages-and-page-images)
  - [Relationship Model](#relationship-model)
- [How the Applications Work Together](#how-the-applications-work-together)
- [Important Product Flows](#important-product-flows)
  - [Visitor-to-Project Flow](#visitor-to-project-flow)
  - [Registration and Session Flow](#registration-and-session-flow)
  - [Donation and Payment Flow](#donation-and-payment-flow)
  - [Project Publishing Flow](#project-publishing-flow)
  - [Content Publishing Flow](#content-publishing-flow)
  - [Favorites Flow](#favorites-flow)
  - [Contact Request Flow](#contact-request-flow)
- [State and Source-of-Truth Rules](#state-and-source-of-truth-rules)
- [Security Boundaries](#security-boundaries)
- [Payment Safety](#payment-safety)
- [Localization and Bilingual Content](#localization-and-bilingual-content)
- [Media and Storage Strategy](#media-and-storage-strategy)
- [Forms and Validation](#forms-and-validation)
- [Error, Loading, and User Feedback](#error-loading-and-user-feedback)
- [SEO, Metadata, and Public Discoverability](#seo-metadata-and-public-discoverability)
- [Engineering Conventions](#engineering-conventions)
- [Documentation Strategy](#documentation-strategy)
- [Development and Verification](#development-and-verification)
- [Deployment Model](#deployment-model)
- [Privacy and Sensitive Data](#privacy-and-sensitive-data)
- [Project Status](#project-status)
- [Repository Guide for Reviewers](#repository-guide-for-reviewers)

---

# What Noor Elrahma Is

Noor Elrahma is a digital platform for presenting humanitarian work and supporting the operational flow around charity projects and donations.

The public side gives visitors a way to discover projects, read organization content, access news, create an account, save favorite projects, and move through donation-oriented flows.

The internal side gives staff a separate administration environment for managing the content and project data that appears publicly.

The backend connects both applications to the same authoritative data model and provides the security-sensitive pieces that should not live in a browser:

- authentication,
- user/session persistence,
- project persistence,
- donation records,
- payment initiation,
- webhook-based payment updates,
- favorite relationships,
- managed page content,
- contact requests,
- file/media storage,
- validation,
- and shared API behavior.

The result is closer to a small publishing, donor-management, and fundraising platform than a static organization website.

---

# Platform Goals

The platform has several product goals that affect its architecture.

## Make projects easy to discover

Visitors should be able to understand:

- what projects exist,
- what each project is about,
- where a project is associated geographically,
- its current status,
- its target/goal,
- and the amount raised where that information is exposed by the experience.

## Support a real donor journey

The platform is built around more than a “Donate” button.

The backend has separate models for:

- donations,
- payments,
- provider transactions,
- payment status,
- donation status,
- currency,
- payment method/provider,
- and project progress.

That separation allows the system to represent a donation before, during, and after payment processing instead of assuming that starting checkout is equal to receiving money.

## Support authenticated users

The platform includes:

- sign-up,
- sign-in,
- email/account verification flows,
- forgot-password,
- reset-password,
- sessions,
- user accounts,
- profile,
- favorites,
- and user-linked donations.

## Support multilingual audiences

The application is localized at the route level and the database contains explicit Arabic/English fields for project and managed-page content.

Localization is therefore not limited to button labels.

## Give staff a proper content-management workspace

Public charity content changes over time.

The administration dashboard covers:

- projects,
- homepage content,
- About content,
- news,
- team information,
- footer content,
- donor/donation-related views,
- and contact requests.

## Keep sensitive rules on the backend

The frontend applications can validate input and control navigation, but they are not the final authority for:

- user roles,
- donation state,
- payment completion,
- project fundraising totals,
- session validity,
- protected content mutations,
- or persistent relationships.

---

# Who Uses the Platform

The system can be understood through three primary audiences.

## Public visitors

Visitors can browse the public website without using the internal dashboard.

Typical visitor goals include:

- learning about Noor Elrahma,
- discovering humanitarian projects,
- reading project details,
- viewing news and organization content,
- contacting the organization,
- and deciding whether to create an account or donate.

## Registered users / donors

Authenticated users have a more persistent experience.

Depending on the flow, this includes:

- profile-related access,
- favorite projects,
- donation-related actions,
- session-aware navigation,
- and account recovery/verification.

## Internal staff / administrators

Staff use the separate dashboard.

They work with content and operational data through protected routes instead of editing public pages directly in source code.

---

# System Architecture

At a high level:

```text
                         ┌─────────────────────────────┐
                         │      Noor Elrahma           │
                         │       Platform              │
                         └──────────────┬──────────────┘
                                        │
             ┌──────────────────────────┼──────────────────────────┐
             │                          │                          │
             ▼                          ▼                          ▼

┌───────────────────────┐   ┌────────────────────────┐   ┌─────────────────────────┐
│ Public Web Platform   │   │ Admin Dashboard        │   │ Backend API             │
│ Next.js 16            │   │ React + Vite           │   │ NestJS 11               │
│ React 19              │   │ TanStack               │   │ PostgreSQL + Drizzle    │
│ multilingual          │   │ protected CMS          │   │ business/data authority │
└───────────┬───────────┘   └────────────┬───────────┘   └────────────┬────────────┘
            │                            │                            │
            └────────────────────────────┼────────────────────────────┘
                                         │
                                         ▼
                             ┌────────────────────────┐
                             │      PostgreSQL        │
                             │ users / projects /     │
                             │ donations / payments / │
                             │ pages / favorites      │
                             └────────────────────────┘
```

The public app and admin dashboard are two different user experiences over the same domain.

They do not synchronize directly with each other.

Instead:

```text
Admin changes content
       ↓
Backend validates and persists
       ↓
Public app requests current content
       ↓
Visitor sees updated state
```

---

# Repository Map

| Repository | Responsibility | Main Stack |
| --- | --- | --- |
| `Charity-org/noor_elrahma_platform` | Public multilingual website and donor-facing experience | Next.js 16, React 19, TypeScript |
| `Charity-org/noor_elrahma_dahsbourd` | Protected administration and content-management UI | React 19, Vite, TanStack, TypeScript |
| `Charity-org/noor-rahma-back` | Shared backend API, persistence, auth, donations, payments, content | NestJS 11, PostgreSQL, Drizzle |
| `Charity-org/documentations` | Shared workflow and engineering documentation | Markdown / repository guidance |

The repository names are kept as they currently exist in the organization, including the existing `dahsbourd` spelling.

---

# Public Web Platform

Repository:

`Charity-org/noor_elrahma_platform`

The public application is the visitor-facing part of the system and is deployed as the public Noor Elrahma website.

Production:

**https://nouralrahma.com/en**

It uses Next.js App Router and organizes user-facing pages under a locale segment.

---

## Public Application Responsibilities

The public application is responsible for:

- rendering localized public content,
- project discovery,
- project-detail experiences,
- donation UI,
- authentication screens,
- account recovery/verification,
- profile-facing UI,
- favorite-project flows,
- news,
- About content,
- contact UI,
- SEO-oriented metadata/routes,
- loading/error handling,
- client/server interaction with the backend,
- and public navigation.

It should not be treated as the final payment, auth, or persistence authority.

---

## Public Route Model

The application uses a locale-first App Router structure.

At the top level:

```text
src/app/
├── [locale]/
├── actions.ts
├── global-error.tsx
├── globals.css
├── robots.ts
└── sitemap.ts
```

Inside `[locale]`, the app separates authentication and public-site concerns:

```text
src/app/[locale]/
├── (auth)/
└── (site)/
```

This is useful because the auth experience can have its own layout and lifecycle without being mixed into the primary public-site composition.

---

## Authentication Experience

The localized auth group includes dedicated routes for:

```text
forgot-password/
reset-password/
sign-in/
sign-up/
verify/
```

This means account access is not implemented as one generic modal or a single catch-all screen.

The public app can present each part of the authentication lifecycle clearly while the backend/Better Auth integration remains responsible for authoritative session/account behavior.

### Typical sign-up flow

```text
User opens localized sign-up page
        ↓
Form values collected
        ↓
Client validation
        ↓
Authentication request
        ↓
Backend / Better Auth processes account
        ↓
Verification flow when required
        ↓
Session-aware public experience
```

### Password-recovery flow

```text
Forgot password
      ↓
Recovery request
      ↓
Reset flow
      ↓
New credential processed
      ↓
User returns to authenticated experience
```

---

## Project Discovery

Projects are a core public content domain.

The backend project model supports fields such as:

- English name
- Arabic name
- English content
- Arabic content
- English description
- Arabic description
- video URL
- image
- country
- fundraising goal
- amount raised
- donation count/value field
- start date
- status
- hero assignment
- hero order
- timestamps

Projects also have a separate project-image relation so one project can have multiple images.

This gives the public frontend enough structure for project cards, project details, hero/project promotion, galleries, fundraising context, and localized presentation.

### Conceptual discovery flow

```text
Homepage / Projects
      ↓
Project collection
      ↓
Filter/search/navigation where applicable
      ↓
Project details
      ↓
Read bilingual project content
      ↓
Favorite or donate
```

---

## Donation Experience

Donations are represented independently from payments.

This is an important design decision.

A donation record includes:

- amount,
- currency,
- status,
- transaction ID,
- user relationship,
- project relationship,
- IP address,
- country,
- method,
- creation time.

The status model includes:

```text
PENDING
COMPLETED
FAILED
```

A donation can therefore exist before payment completion.

The public frontend initiates the user journey, but the backend is responsible for:

- creating/reading the donation record,
- selecting the provider path,
- creating payment state,
- processing provider callbacks,
- updating the donation,
- and updating the associated project’s raised amount after successful payment.

---

## Favorites

Favorites create a relationship between authenticated users and projects.

This makes favorites domain data rather than a browser-only convenience.

A typical flow is:

```text
Authenticated user
      ↓
Views a project
      ↓
Favorites / unfavorites project
      ↓
Backend persists relationship
      ↓
Favorites page reflects current state
```

This persistence means favorites can survive navigation and sessions instead of being tied only to local browser state.

---

## Profile and Authenticated Experience

The public site contains a dedicated `profile` route under the localized site group.

Profile behavior is connected to authenticated session state rather than existing as a completely separate application.

The user/account model stored by the backend includes:

- ID
- name
- email
- email verification state
- image
- country
- activation state
- role
- created/updated timestamps

---

## News and Organizational Content

The public route structure includes:

- About Us
- News
- Home
- Projects
- Contact Us

The backend also contains a managed-pages domain that supports multiple content types.

Managed page types include:

- HERO
- IMPACT_PROJECT
- NEWS
- ABOUT_US
- WHO_WE_ARE
- FOOTER
- MIDDLEPAGE
- WORKER_SAY
- WHY_DONATE
- RECENT_PROJECT

That structure gives the platform a CMS-like content model instead of requiring every public content change to be hardcoded in Next.js.

---

## Contact Flow

The public app has a dedicated Contact Us route.

The backend contains its own `contact-us` module and schema, allowing contact requests to be treated as persistent operational data.

The dashboard then has its own Contact Us management route so staff can work with submitted requests separately from the public form.

---

## Internationalization

Localization is part of the route architecture.

The public application uses:

- `[locale]` route segmentation
- `next-intl`
- bilingual backend fields
- an API language interceptor
- Arabic/English content in projects and managed pages

This means localization exists across:

```text
URL
 ↓
frontend route
 ↓
frontend messages/content
 ↓
API language handling
 ↓
database bilingual fields
```

It is not limited to changing a few navigation labels.

---

## SEO and Discoverability

The Next.js app contains explicit:

- `robots.ts`
- `sitemap.ts`
- App Router page structure
- localized routes

These are useful foundations for public discoverability because charity projects and organization content need to be indexable and shareable.

The system also includes project- and content-driven pages that can participate in a structured SEO strategy.

---

## Public Frontend Technology

### Framework

- Next.js 16.1.1
- React 19.2.3
- React DOM 19.2.3
- TypeScript 5
- App Router
- React Compiler

### Localization

- next-intl

### Authentication

- Better Auth client-side integration

### Forms and validation

- React Hook Form
- Zod
- `@hookform/resolvers`

### UI

- Tailwind CSS 4
- Radix UI primitives
- Lucide React
- Sonner
- class-variance-authority
- clsx
- tailwind-merge
- next-themes

### Interaction

- Motion
- Lenis
- Embla Carousel
- Embla Autoplay
- dotted-map

### Data

- Axios
- TanStack Table

### Payment-related frontend package

The public frontend includes `@paypal/react-paypal-js` in its dependencies.

The authoritative backend payment implementation currently contains Paymob and Polar provider flows. Provider-specific frontend packages should therefore be understood as client integration capability, while server-side payment records and completion state remain backend-owned.

---

## Public Frontend Quality Workflow

The public app exposes scripts for:

```bash
npm run dev
npm run build
npm run build:turbo
npm run start
npm run lint
npm run format
npm run format:check
npm run type-check
```

It also uses:

- Husky
- lint-staged
- Commitlint
- Prettier
- ESLint

This keeps formatting, linting, type checking, and commit workflow explicit instead of relying only on editor settings.

---

# Administration Dashboard

Repository:

`Charity-org/noor_elrahma_dahsbourd`

The administration dashboard is a separate React/Vite application for internal management.

The separation is intentional.

Public users should not be loading a large staff-oriented application, and internal content workflows should not be embedded into visitor-facing routes.

---

## Dashboard Responsibilities

The dashboard is responsible for:

- protected internal navigation,
- loading current backend-managed content,
- presenting project/content tables and editors,
- validating staff input,
- sending mutations to the backend,
- managing bilingual content,
- coordinating media updates,
- presenting donor-related data,
- showing dashboard summaries,
- and keeping server state synchronized after changes.

The dashboard is an authoring/operations interface.

The backend remains the persistence and authorization boundary.

---

## Protected Route Structure

The dashboard uses TanStack Router with a protected route group.

At a high level:

```text
src/routes/
├── __root.tsx
├── index.tsx
└── _protected/
    ├── route.tsx
    └── dashboard/
```

Inside the protected dashboard area, the current route set includes:

```text
about-us.tsx
contact-us.tsx
donators.tsx
footer.tsx
home.tsx
index.tsx
news.tsx
projects/
team.tsx
```

This route structure maps closely to the content domains presented on the public website.

---

## Dashboard Overview

The dashboard index/home areas provide the internal landing point for staff.

A dashboard overview is useful for:

- orienting administrators,
- surfacing important platform data,
- providing quick navigation,
- and presenting high-level operational context before staff enter specific editors.

The React dashboard stack includes Recharts, so data visualization support is available for dashboard reporting/summary experiences where used.

---

## Project Administration

Projects have a dedicated nested route group rather than being reduced to one generic content form.

This is important because a project can contain:

- bilingual names,
- bilingual descriptions,
- bilingual long content,
- main media,
- video,
- gallery images,
- country,
- goal,
- raised amount,
- status,
- hero placement,
- and ordering metadata.

### Typical project editing flow

```text
Projects route
      ↓
Project list
      ↓
Select project
      ↓
Load current backend record
      ↓
Edit bilingual text and media
      ↓
Validate form
      ↓
Submit mutation
      ↓
Backend persists project
      ↓
Refresh current project/list data
      ↓
Public site receives current state
```

Project administration is therefore one of the clearest examples of how the dashboard and public application are connected through the backend.

---

## Donator Views

The dashboard includes a `donators.tsx` route.

This provides an internal place to work with donor/donation-oriented information separately from public donation UX.

The backend’s user, donation, and payment records provide the data relationships needed for this area.

---

## About Us Management

About Us has a dedicated protected route.

The backend managed-pages model includes types such as:

- ABOUT_US
- WHO_WE_ARE
- WORKER_SAY
- WHY_DONATE

This allows richer About-related sections to be represented as managed content rather than a single hardcoded text block.

---

## Homepage Management

Home also has a dedicated protected route.

The managed-page model includes home-oriented section types such as:

- HERO
- IMPACT_PROJECT
- RECENT_PROJECT
- MIDDLEPAGE

Projects can also be assigned to hero placement with an explicit order.

That combination supports a homepage that is composed from backend-managed content and selected project data.

---

## News Management

News has a dedicated dashboard route and is also represented in the managed-page type model.

This allows the public news experience and internal publishing workflow to remain separate.

---

## Team Management

Team content has its own dashboard route.

The managed-page schema also includes author/author-role fields, which can support people/testimonial/content presentation patterns where those fields are used.

---

## Footer Management

Footer content is not treated as immutable source code.

The dashboard has a dedicated footer route and the backend managed-page type includes `FOOTER`.

This allows business/contact/footer information to be updated through administration workflows.

---

## Contact Request Management

The dashboard has its own Contact Us route.

The normal flow is:

```text
Visitor submits public contact form
        ↓
Backend validates/persists request
        ↓
Staff opens protected contact area
        ↓
Request is reviewed operationally
```

This keeps incoming messages out of frontend-only state.

---

## Bilingual Content Editing

Projects and managed pages both contain explicit Arabic and English fields.

Examples include:

```text
name / nameAr
content / contentAr
description / descriptionAr
slug / slugAr
title / titleAr
video_title / video_titleAr
```

This means the dashboard has to treat bilingual editing as a data-model requirement, not just an interface translation feature.

---

## Media Workflows

The project and page schemas support:

- main images,
- project images,
- page images,
- videos,
- video titles.

The backend contains a storage module, and the package set includes Vercel Blob.

The storage implementation also contains disk-storage support.

This gives the backend an abstraction point for media/file handling rather than coupling all upload logic directly to project controllers.

---

## Dashboard Technology

### Core

- React 19.2
- React DOM 19.2
- TypeScript 5.9
- Vite 7.2
- React Compiler

### Routing and server state

- TanStack Router
- TanStack Query
- TanStack Table
- TanStack Router Devtools
- TanStack Query Devtools

### Authentication

- Better Auth

### Forms and validation

- React Hook Form
- Zod
- `@hookform/resolvers`

### UI

- Tailwind CSS 4
- Radix UI
- Tabler Icons
- Lucide React
- Sonner
- Vaul
- React Day Picker
- next-themes

### Interaction

- dnd-kit
- sortable/modifier utilities

### Data visualization

- Recharts

### Networking and utilities

- Axios
- date-fns
- class-variance-authority
- clsx
- tailwind-merge

---

## Dashboard Engineering Workflow

Dashboard scripts include:

```bash
npm run dev
npm run build
npm run lint
npm run preview
npm run format
npm run format:check
npm run type-check
```

The build command runs:

```text
tsc -b
   ↓
vite build
```

The project also uses:

- Husky
- lint-staged
- Commitlint
- Prettier
- ESLint
- TanStack Query ESLint tooling

---

# Backend API

Repository:

`Charity-org/noor-rahma-back`

Default branch:

`dev`

The backend is the shared authority for both web applications.

It is implemented with NestJS and PostgreSQL/Drizzle.

---

## Backend Responsibilities

The backend owns:

- configuration,
- authentication,
- users,
- sessions/accounts,
- project data,
- project galleries,
- donations,
- payment records,
- payment provider integration,
- webhook handling,
- favorites,
- dynamic/managed pages,
- contact requests,
- storage,
- health endpoints,
- language interception,
- validation,
- security middleware,
- database migrations,
- seed/restore tooling,
- and testing.

---

## NestJS Module Boundaries

The current top-level backend source includes:

```text
src/
├── auth/
├── common/
├── contact-us/
├── database/
├── donations/
├── favorites/
├── health/
├── pages/
├── payments/
├── projects/
├── users/
├── app.controller.ts
├── app.module.ts
└── main.ts
```

The root NestJS module imports:

- DatabaseModule
- UsersModule
- PagesModule
- ProjectsModule
- DonationsModule
- FavoritesModule
- HealthModule
- ContactUsModule
- AuthModule
- StorageModule
- PaymentsModule

This makes the system modular at the application level rather than placing all controllers/services into one folder.

---

## Application Bootstrap and HTTP Layer

The application bootstrap configures several cross-cutting concerns.

### Global API prefix

The backend uses:

```text
/api
```

as a global prefix.

### Static uploads

The Nest/Express application exposes an uploads path for static assets where disk storage is used.

### Logging

A global logging interceptor is configured.

### Global validation

A NestJS `ValidationPipe` is configured with:

- transformation enabled,
- whitelist enabled,
- non-whitelisted properties forbidden,
- implicit conversion enabled.

### Query parsing

The Express query parser is configured through `qs` with explicit depth, array, and parameter limits.

### Better Auth mounting

Better Auth is mounted before the normal JSON/body parser for `/api/auth/*` requests.

This is an implementation detail that matters because authentication frameworks may need direct access to the request body/headers before general parsing changes the request shape.

---

## Database and Drizzle ORM

The backend uses PostgreSQL with:

- Drizzle ORM
- Drizzle Kit
- `pg`
- `postgres`

The database folder includes:

- connection/module code,
- Drizzle setup,
- schema-related repair tooling,
- donation debugging utilities,
- page-table restore tooling,
- and seeders.

The package scripts expose:

```bash
npm run drizzle:generate
npm run migrate
npm run seed
npm run seed:demo
npm run seed:restore
```

This means schema evolution and data preparation are represented as explicit development/operations commands.

---

## Authentication and Sessions

Authentication uses Better Auth with PostgreSQL-backed user/session/account data.

The schema includes separate tables for:

- user,
- session,
- account,
- verification.

### User

A user record includes:

- ID
- name
- email
- email verification state
- image
- country
- activation state
- role
- creation/update timestamps

### Session

Sessions include:

- ID
- expiration
- token
- user relationship
- IP address
- user agent
- timestamps

### Account

Accounts include provider/account relationships and can store:

- provider ID
- access token
- refresh token
- ID token
- token-expiration metadata
- scope
- password
- user relationship

### Verification

Verification records contain:

- identifier
- verification value
- expiry
- timestamps

This supports real account/session persistence rather than frontend-only authentication flags.

---

## Roles and Account Model

The backend schema defines these roles:

```text
ADMIN
GUEST
USER
PROMOTER
```

The role field is stored with the user record.

The exact permission rules belong to backend/domain behavior, but the schema establishes that user role is persistent domain data rather than an arbitrary frontend label.

---

## Projects Domain

Projects have their own module and database schema.

A project includes:

- ID
- English name
- Arabic name
- English content
- Arabic content
- English description
- Arabic description
- video URL
- main image
- country
- goal
- amount raised
- donation-related count/value field
- start date
- status
- hero assignment flag
- hero order
- created/updated timestamps

Projects can also have many project-image records.

Project images reference the project with cascade deletion.

This gives project data enough structure for:

- listing,
- detail pages,
- bilingual content,
- galleries,
- fundraising progress,
- homepage selection,
- and status-driven presentation.

---

## Managed Pages Domain

The `pages` module acts as a flexible managed-content layer.

Page categories include:

```text
HOME
ABOUT_US
NEWS
ALL
```

Managed section/content types include:

```text
HERO
IMPACT_PROJECT
NEWS
ABOUT_US
WHO_WE_ARE
FOOTER
MIDDLEPAGE
WORKER_SAY
WHY_DONATE
RECENT_PROJECT
```

Each managed page/section can store fields such as:

- slug / Arabic slug
- title / Arabic title
- description / Arabic description
- content / Arabic content
- image
- video title / Arabic video title
- video
- linked project
- page category
- section type
- author
- author role
- published state
- phone
- email
- address
- link
- timestamps

Managed pages can also have multiple page images.

This gives the dashboard a real content-management backend rather than forcing all public-site sections into hardcoded constants.

---

## Donations Domain

Donations are separate persistent records.

A donation stores:

- generated UUID
- amount
- currency
- status
- provider transaction reference
- optional user
- optional project
- IP address
- country
- method
- creation timestamp

Status options include:

```text
PENDING
COMPLETED
FAILED
```

A donation belongs to a user/project when those relationships exist and can have payment records.

---

## Payments Domain

Payment records are separate from donations.

A payment stores:

- UUID
- donation relationship
- provider
- provider transaction ID
- checkout URL
- amount
- currency
- status
- metadata
- created/updated timestamps

Supported schema provider values are:

```text
POLAR
PAYMOB
```

Payment status supports:

```text
PENDING
COMPLETED
FAILED
REFUNDED
```

The separation between `donations` and `payments` is important.

One donation represents the fundraising intent/domain record.

A payment represents the provider transaction used to fulfill that donation.

---

## Payment Provider Routing

The backend payment service routes providers based on donation currency.

Current implementation:

### Polar

Used for:

```text
USD
EUR
```

The service creates a Polar checkout and stores:

- provider = POLAR
- provider transaction ID
- checkout URL
- amount/currency
- pending status
- provider metadata

### Paymob

Used for:

```text
EGP
OMR
SAR
AED
```

The Paymob sequence performs:

```text
Authenticate with Paymob
        ↓
Create Paymob order
        ↓
Create payment key
        ↓
Build iframe checkout URL
        ↓
Create local payment record
        ↓
Return checkout information
```

Unsupported currencies are rejected rather than silently sent to an arbitrary provider.

---

## Payment Completion and Project Progress

The backend handles provider completion through webhook-oriented logic.

### Paymob

The implementation:

- validates HMAC,
- resolves the Paymob order/provider transaction,
- finds the local payment,
- marks it completed or failed,
- updates the linked donation,
- and applies project progress updates after successful payment.

### Polar

The implementation uses Polar webhook validation and provider event data to locate/update a payment.

### Successful-payment flow

Conceptually:

```text
Provider webhook
      ↓
Validate provider callback
      ↓
Find local payment
      ↓
Update payment status
      ↓
Update donation status
      ↓
If completed:
    update project raised amount
      ↓
If raised >= goal:
    mark project completed
```

This is a much safer model than incrementing project totals when the user merely clicks “Pay”.

---

## Favorites Domain

Favorites have their own module and schema.

The user schema explicitly relates users to favorites.

Projects and authenticated users can therefore be connected through persisted favorite records.

This supports a real saved-project experience across sessions.

---

## Contact Domain

Contact Us is represented as its own backend module.

This separation is useful because incoming public requests have a different lifecycle from page content, donations, or user accounts.

---

## Storage

The backend includes a dedicated storage module under `common/storage`.

The current source contains:

- storage service abstraction,
- disk storage service,
- storage module.

The dependency set also includes `@vercel/blob`.

This indicates a design where media storage can remain behind a backend boundary rather than letting frontend code directly own privileged file-storage credentials.

---

## Health and Operational Endpoints

The backend has a dedicated `health` module.

Health endpoints are useful for:

- deployment verification,
- service availability checks,
- uptime tooling,
- infrastructure diagnostics,
- and distinguishing frontend errors from backend unavailability.

---

## Validation

Global NestJS validation is configured with:

```text
transform: true
whitelist: true
forbidNonWhitelisted: true
enableImplicitConversion: true
```

That means backend DTO validation is not just advisory.

Unexpected fields can be rejected rather than silently accepted into domain operations.

Frontend Zod validation improves the user experience, but backend validation remains required because public clients cannot be trusted.

---

## HTTP Security

The backend uses Helmet.

The current configuration includes CSP directives around:

- same-origin resources,
- Google account/OAuth endpoints,
- Google-hosted images,
- frame/connect behavior.

`crossOriginEmbedderPolicy` is disabled where required for relevant OAuth behavior.

The package also includes NestJS Throttler for rate-limiting/throttling capability.

---

## CORS and Client Integration

CORS is configured with credentials support and explicit allowed origins for:

- local public frontend,
- deployed public frontend,
- local dashboard development ports.

Allowed methods include:

```text
GET
POST
PUT
PATCH
DELETE
OPTIONS
```

Allowed headers include auth, language, content, origin, and XSRF-oriented headers.

Cookie-related response headers are exposed for authentication/session flows.

---

## Localization at the API Layer

The root module registers a `LangInterceptor`.

The client can send language-related headers such as:

```text
x-lang
lang
```

This complements the public app’s locale route and the bilingual database fields.

The platform therefore handles language across several layers rather than treating it as a pure CSS/UI concern.

---

## Backend Technology

### Runtime / framework

- Node.js
- NestJS 11
- TypeScript 5.7
- RxJS

### Database

- PostgreSQL
- Drizzle ORM
- Drizzle Kit
- `pg`
- `postgres`

### Authentication

- Better Auth
- `@node-rs/argon2`

### Validation

- class-validator
- class-transformer

### Security / configuration

- Helmet
- NestJS Config
- NestJS Throttler

### Payments

- Paymob SDK for Egypt
- Polar SDK

### Storage / communication

- Vercel Blob
- Nodemailer

### Utilities

- ExcelJS
- geoip-lite

### Testing

- Jest
- Supertest
- NestJS testing utilities
- ts-jest

---

## Database Migrations and Seeders

The backend package exposes explicit commands for:

```bash
npm run drizzle:generate
npm run migrate
npm run seed
npm run seed:demo
npm run seed:restore
```

The database directory also contains specialized utilities such as:

- donation debugging,
- schema repair,
- pages-table restoration.

This is useful for a content-heavy platform because seed/restore behavior matters during migrations, staging setup, and operational recovery.

---

## Backend Testing

Backend scripts include:

```bash
npm run test
npm run test:watch
npm run test:cov
npm run test:debug
npm run test:e2e
```

Testing uses:

- Jest
- Supertest
- NestJS test utilities
- ts-jest

The test configuration collects TypeScript/JavaScript source coverage into a dedicated coverage directory.

---

# Core Data Model

The main data model can be summarized as follows.

```text
User
 ├─ Sessions
 ├─ Accounts
 ├─ Favorites ───────────────┐
 └─ Donations                │
        │                    │
        ├──────────────┐     │
        ▼              │     │
     Payment           │     │
                       │     │
                       ▼     ▼
                     Project
                       │
                       └─ Project Images

Managed Page
 ├─ optional Project link
 └─ Page Images
```

---

## Users and Sessions

A user can have:

- many sessions,
- many provider/auth accounts,
- many favorites,
- many donations.

Sessions include device/request context such as IP address and user agent.

---

## Projects and Project Images

A project can have:

- multiple images,
- multiple donations.

Project-image records reference project IDs with cascade deletion.

---

## Donations

A donation can belong to:

- one user,
- one project.

A donation can have:

- many payment records.

This is useful if a payment flow is retried or provider-level records need to be represented separately from the underlying donation intent.

---

## Payments

Each payment belongs to one donation.

Payment provider state is intentionally independent of project content.

---

## Managed Pages and Page Images

A managed page/section can:

- link to a project,
- have multiple images,
- contain bilingual content,
- be marked published/unpublished,
- belong to a public page category and section type.

---

## Relationship Model

The relational model makes several product behaviors possible:

### Donor history

```text
User
 ↓
Donations
 ↓
Payments
```

### Project fundraising

```text
Project
 ↓
Donations
 ↓
Successful Payments
 ↓
Raised progress
```

### Favorite projects

```text
User
 ↓
Favorites
 ↓
Project
```

### Managed homepage content

```text
Managed Page Section
 ↓
optional Project link
 ↓
public rendering
```

---

# How the Applications Work Together

Noor Elrahma follows a shared-backend model.

## Public read path

```text
Visitor opens website
       ↓
Next.js localized route
       ↓
Public app requests current content/projects
       ↓
NestJS API
       ↓
PostgreSQL / Drizzle
       ↓
Localized public UI
```

## Admin write path

```text
Authorized staff opens dashboard
       ↓
Protected route
       ↓
Editor/form
       ↓
Client validation
       ↓
NestJS API
       ↓
Backend validation/auth
       ↓
PostgreSQL update
       ↓
Dashboard refresh
       ↓
Public app receives updated data
```

## Donation path

```text
Public donation UI
       ↓
Donation record
       ↓
Payment initiation
       ↓
Provider checkout
       ↓
Provider webhook
       ↓
Payment + donation update
       ↓
Project raised total update
       ↓
Updated project visible publicly
```

---

# Important Product Flows

## Visitor-to-Project Flow

```text
Homepage
   ↓
Project discovery
   ↓
Project card/list
   ↓
Project detail
   ↓
Read localized description/content
   ↓
Favorite and/or donate
```

---

## Registration and Session Flow

```text
Localized sign-up
   ↓
Client form validation
   ↓
Better Auth request
   ↓
Backend auth handling
   ↓
User/account record
   ↓
Verification when required
   ↓
Session
   ↓
Profile / favorites / donor experience
```

---

## Donation and Payment Flow

```text
Project / donation entry point
      ↓
Amount + currency
      ↓
Donation record created
      ↓
Backend chooses provider by currency
      ├─ USD/EUR → Polar
      └─ EGP/OMR/SAR/AED → Paymob
      ↓
Local payment record = PENDING
      ↓
Checkout URL returned
      ↓
Donor completes provider flow
      ↓
Provider callback/webhook
      ↓
Signature/HMAC validation
      ↓
Payment status updated
      ↓
Donation status updated
      ↓
If successful:
   project raised amount increases
      ↓
If goal reached:
   project status can become completed
```

This is one of the platform’s most important cross-application flows.

---

## Project Publishing Flow

```text
Staff opens project editor
       ↓
Loads current project
       ↓
Edits bilingual content/media
       ↓
Validates input
       ↓
Backend persists project
       ↓
Project list/details refresh
       ↓
Public project pages use new data
```

---

## Content Publishing Flow

```text
Staff opens Home / About / News / Footer editor
       ↓
Managed page/section loaded
       ↓
Content changed
       ↓
Backend persists bilingual section
       ↓
Publication state respected
       ↓
Public Next.js app reads current content
```

---

## Favorites Flow

```text
Authenticated user
      ↓
Project
      ↓
Favorite action
      ↓
Backend favorite relation
      ↓
Profile/Favorites page
```

---

## Contact Request Flow

```text
Visitor
  ↓
Contact Us form
  ↓
Frontend validation
  ↓
Backend contact module
  ↓
Persistent request
  ↓
Admin Contact Us route
```

---

# State and Source-of-Truth Rules

A central engineering rule is deciding which application owns which kind of state.

## Backend-owned state

Examples:

- users,
- sessions,
- roles,
- email verification state,
- projects,
- project fundraising progress,
- donations,
- payments,
- favorite relationships,
- managed content,
- contact requests,
- published state.

## Public frontend state

Examples:

- current route,
- local form values,
- open/closed UI,
- loading state,
- interaction state,
- transient filters,
- current locale presentation.

## Admin frontend state

Examples:

- current editor values,
- table state,
- active route,
- local upload progress,
- dialog state,
- mutation loading state,
- current query cache.

The public website and dashboard should never become independent databases.

---

# Security Boundaries

Security is layered.

```text
Frontend route/UI state
        ↓
Session-aware client behavior
        ↓
Authenticated HTTP request
        ↓
Backend auth/session validation
        ↓
Backend role/domain checks
        ↓
Database mutation
```

Client-side route protection is useful but is not sufficient.

A user should not gain protected mutation capability merely by constructing a request outside the dashboard.

---

# Payment Safety

Payment state deserves stricter rules than normal content editing.

## Do not trust redirects as payment proof

A browser returning from a provider is not by itself sufficient evidence that payment completed.

The backend provider webhook flow updates the authoritative payment/donation state.

## Keep provider records separate

The `payments` table keeps:

- provider,
- provider transaction ID,
- checkout URL,
- status,
- metadata.

This helps preserve provider-specific state without polluting the donation/project model.

## Validate callbacks

Paymob callback handling includes HMAC verification.

Polar handling uses provider webhook validation.

## Update fundraising only after success

The backend increments project raised totals after a payment reaches completed state.

---

# Localization and Bilingual Content

Noor Elrahma uses a multi-layer localization model.

## URL layer

```text
/[locale]/...
```

## Frontend translation layer

`next-intl`

## Database layer

Bilingual fields such as:

```text
name / nameAr
description / descriptionAr
content / contentAr
title / titleAr
slug / slugAr
```

## API layer

Language interceptor plus accepted language headers.

This design is important because translated navigation is not enough when projects and organization content are managed dynamically.

---

# Media and Storage Strategy

Projects and pages can contain multiple media fields.

The backend provides a storage abstraction under `common/storage`.

The codebase contains disk-storage support and has Vercel Blob available in backend dependencies.

Keeping storage behind a backend service provides a place to centralize:

- file naming,
- upload handling,
- storage provider choice,
- cleanup,
- and privileged credentials.

---

# Forms and Validation

Both frontend applications use:

- React Hook Form
- Zod

The backend uses:

- class-validator
- class-transformer
- global ValidationPipe

The normal validation model is:

```text
User/staff input
      ↓
Frontend schema validation
      ↓
Request
      ↓
Backend DTO validation
      ↓
Domain logic
      ↓
Database
```

Frontend validation exists for good UX.

Backend validation exists for trust and data integrity.

---

# Error, Loading, and User Feedback

The public Next.js app contains:

- global error handling,
- site-level error handling,
- loading UI.

The dashboard stack includes:

- server-state libraries,
- toast/notification tooling,
- query devtools.

For a donation and content platform, explicit error states matter because:

- a failed payment should not look completed,
- a failed project update should not disappear silently,
- an unavailable backend should not look like “no projects”,
- a failed auth request should not create ambiguous session state.

---

# SEO, Metadata, and Public Discoverability

Public project/content platforms benefit from strong indexability.

The Next.js app contains:

- localized App Router pages,
- `robots.ts`,
- `sitemap.ts`,
- dynamic project/content routes.

A well-maintained production deployment can use these foundations for:

- project discovery through search,
- correct indexing,
- localized URLs,
- cleaner shareable links,
- and explicit crawler behavior.

---

# Engineering Conventions

Several engineering principles are visible across the repositories.

## Separate public and admin products

The public website and dashboard have different audiences and should not be one large application.

## Keep business data authoritative on the backend

Projects, donations, payment state, favorites, roles, and managed pages live in PostgreSQL through backend contracts.

## Separate donation from payment

Fundraising intent/domain state is not the same as payment-provider state.

## Model multilingual content explicitly

Arabic/English data is part of the schema rather than being hidden inside ad hoc JSON in components.

## Keep content dynamic

Homepage/About/News/Footer-related sections can be managed without rebuilding hardcoded pages for every content change.

## Use modular backend domains

NestJS modules keep auth, users, projects, donations, payments, favorites, pages, contact, and infrastructure separate.

## Make migration/seed operations explicit

Drizzle migration and seed/restore scripts are available as project commands.

## Validate at both edges

Clients validate for UX; server validates for correctness/security.

---

# Documentation Strategy

Repository:

`Charity-org/documentations`

The documentation repository is intended for shared engineering guidance that applies across multiple Noor Elrahma applications.

Examples include:

- Git/GitHub workflow
- branch conventions
- commit conventions
- pull-request expectations
- Husky/local-hook references
- cross-repository coordination
- handoff guidance

Application-specific technical truth should remain inside each application repository.

For example:

### Public app owns

- Next.js setup
- localization
- public routes
- public environment variables
- SEO behavior
- build commands

### Dashboard owns

- protected routes
- dashboard architecture
- content-editor behavior
- TanStack setup
- dashboard build commands

### Backend owns

- NestJS modules
- database schema
- migrations
- seeders
- auth implementation
- payment implementation
- backend environment/configuration
- tests

This reduces documentation drift.

---

# Development and Verification

Each product has its own development workflow.

## Public website

```bash
npm run dev
npm run lint
npm run type-check
npm run format:check
npm run build
```

## Dashboard

```bash
npm run dev
npm run lint
npm run type-check
npm run format:check
npm run build
```

## Backend

```bash
npm run start:dev
npm run lint
npm run test
npm run test:e2e
npm run build
```

Database changes can use:

```bash
npm run drizzle:generate
npm run migrate
```

Seed operations include:

```bash
npm run seed
npm run seed:demo
npm run seed:restore
```

---

# Deployment Model

The repositories are independently deployable because they are independent applications.

At a conceptual level:

```text
Public Next.js deployment
        │
        │ HTTPS/API
        ▼
NestJS backend deployment
        │
        ▼
PostgreSQL

Admin React/Vite deployment
        │
        └────── HTTPS/API ──────► same backend
```

The backend CORS configuration explicitly recognizes local public/admin development origins and the deployed public frontend origin.

Production environment variables, database credentials, payment secrets, auth secrets, and provider keys belong outside source control.

---

# Privacy and Sensitive Data

A charity platform can process sensitive or trust-critical information such as:

- account details,
- email addresses,
- sessions,
- IP addresses,
- user-agent data,
- donation records,
- payment transaction references,
- contact requests,
- provider metadata.

Important boundaries therefore include:

- do not expose backend credentials in frontend bundles,
- do not treat frontend state as payment proof,
- do not expose private provider metadata unnecessarily,
- keep protected admin mutations authenticated,
- validate user input on the server,
- keep secrets environment-driven,
- avoid putting donor-sensitive details into public repository documentation.

This organization profile intentionally explains the architecture without publishing deployment secrets or real donor information.

---

# Project Status

Noor Elrahma is a deployed multi-application charity platform.

The public site is available at:

**https://nouralrahma.com/en**

The organization contains:

- a public-facing Next.js product,
- an internal React/Vite administration product,
- a NestJS/PostgreSQL backend,
- and shared engineering documentation.

Individual repositories can continue to receive:

- content changes,
- project-management improvements,
- payment-provider changes,
- authentication improvements,
- localization work,
- performance work,
- accessibility fixes,
- security hardening,
- and maintenance updates.

The organization README should describe the full platform, while contributor CVs or portfolios should separately describe each person’s verified contribution.

---

# Repository Guide for Reviewers

If you are reviewing Noor Elrahma from an engineering perspective, this order gives the clearest picture.

## 1. Public platform

`Charity-org/noor_elrahma_platform`

Review this repository first to understand:

- the public user journey,
- localized routing,
- auth screens,
- project discovery,
- donations UI,
- favorites,
- profile,
- news,
- About and Contact experiences,
- Next.js structure,
- SEO foundations.

## 2. Administration dashboard

`Charity-org/noor_elrahma_dahsbourd`

Review this next to understand:

- staff workflows,
- protected routing,
- project management,
- bilingual editors,
- donor-related views,
- About/Home/News/Team/Footer management,
- forms,
- TanStack architecture,
- server-state synchronization.

## 3. Backend

`Charity-org/noor-rahma-back`

This is where the core domain model becomes visible.

Pay attention to:

- NestJS modules,
- Better Auth integration,
- PostgreSQL/Drizzle schemas,
- users/sessions,
- project relations,
- donation/payment separation,
- Paymob/Polar payment routing,
- webhook completion logic,
- project fundraising updates,
- managed pages,
- favorites,
- storage,
- validation,
- database migrations and seeders.

## 4. Documentation

`Charity-org/documentations`

Use this for:

- team workflow,
- repository collaboration rules,
- shared engineering references,
- handoff conventions.

---

# In Short

Noor Elrahma is best understood as four connected layers:

```text
PUBLIC EXPERIENCE
Next.js / React
projects + content + auth + donations + favorites
        │
        ▼
SHARED BACKEND
NestJS
auth + domain rules + payments + storage + API
        │
        ▼
DATA
PostgreSQL / Drizzle
users + sessions + projects + donations + payments + pages + favorites
        ▲
        │
INTERNAL OPERATIONS
React / Vite dashboard
projects + donors + content + publishing
```

The technical value of the platform is not only the number of pages it contains.

The more important engineering work is the coordination between:

- public and internal applications,
- bilingual content,
- authenticated users,
- project fundraising state,
- donation records,
- provider-specific payment transactions,
- webhook-confirmed payment completion,
- dynamic content management,
- persistent favorites,
- and a shared backend that keeps these clients consistent.

That is the architecture this organization represents.
