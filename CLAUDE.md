# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Development Commands

### Local Development
- `pnpm dev` - Start development server
- `pnpm build` - Build for production
- `pnpm lint` - Run ESLint
- `pnpm email:dev` - Start email template development server on port 3001

### Database Operations
- `pnpm db:generate [MIGRATION_NAME]` - Generate new database migration after schema changes
- `pnpm db:migrate:dev` - Apply migrations to local D1 database
- `pnpm d1:cache:clean` - Clear D1 cache tables (tags and revalidations)

### Cloudflare Operations
- `pnpm cf-typegen` - Generate TypeScript types from wrangler.jsonc (run after changes to wrangler.jsonc)
- `pnpm opennext:build` - Build using OpenNext for Cloudflare Workers
- `pnpm deploy` - Deploy to Cloudflare Workers
- `pnpm preview` - Preview deployment locally

### KV Operations
- `pnpm list:kv` - List KV namespace keys
- `pnpm delete:kv` - Delete KV namespace keys from kv.log file

### Bundle Analysis
- `pnpm build:analyze` - Build with bundle analyzer

## Architecture Overview

### Core Stack
- **Frontend**: Next.js 15 App Router, React Server Components, TypeScript
- **UI**: Tailwind CSS, Shadcn UI (Radix UI), Lucide Icons
- **Backend**: Cloudflare Workers with OpenNext
- **Database**: Drizzle ORM + Cloudflare D1 (SQLite)
- **Storage**: Cloudflare KV (sessions, cache)
- **Authentication**: Lucia Auth with KV sessions
- **Email**: React Email + Resend/Brevo
- **Payments**: Stripe integration
- **State**: Zustand for client state

### Key Directories
- `src/app/` - Next.js App Router with route groups:
  - `(auth)/` - Authentication flows
  - `(dashboard)/` - Main application features
  - `(settings)/` - User/team settings
  - `(admin)/` - Admin dashboard
- `src/components/` - React components (ui/ contains Shadcn components)
- `src/db/` - Database schema and migrations
- `src/utils/` - Core utilities including auth, email, team management
- `src/actions/` - Server actions for data mutations
- `src/schemas/` - Zod validation schemas
- `src/react-email/` - Email templates

### Authentication System
- Session-based auth using Lucia Auth
- KV-based session storage with 30-day expiration
- Multi-factor authentication support (WebAuthn/Passkeys)
- Google OAuth integration
- Rate limiting on auth endpoints
- Team-based multi-tenancy with role permissions

### Database Schema
- CUID2-based IDs for all entities
- Common columns: createdAt, updatedAt, updateCounter
- Core tables: users, teams, teamMemberships, teamRoles, credits, transactions
- Team permissions stored as JSON in teamRoles table

## Development Guidelines

### Code Style
- Use functional components with TypeScript
- Server Components by default, 'use client' only when needed
- Add `import "server-only"` to server-only files (except page.tsx)
- Use named object parameters for functions with >1 parameter
- Use `pnpm` for package management

### Database Changes
- Never manually create SQL migrations
- Modify `src/db/schema.ts` then run `pnpm db:generate [MIGRATION_NAME]`
- Apply with `pnpm db:migrate:dev` for local development

### Cloudflare Integration
- Access bindings via `getCloudflareContext()` from `@opennextjs/cloudflare`
- Run `pnpm cf-typegen` after changes to wrangler.jsonc
- Use existing KV namespace, don't create new ones

### Form Handling
- Use Zod schemas for validation (stored in `src/schemas/`)
- Server actions with `zsa` and `zsa-react`
- Client forms with React Hook Form + Zod resolvers

### Security
- Rate limiting for API endpoints
- Input validation and sanitization
- Anti-disposable email protection
- Session management with secure cookies
- Team-based authorization with permission checks

## Configuration Files

### Key Files to Customize
- `src/constants.ts` - Site configuration, credit packages, limits
- `wrangler.jsonc` - Cloudflare Workers configuration
- `src/components/footer.tsx` - Footer links and branding
- `src/app/layout.tsx` - Site metadata
- `src/app/globals.css` - Color palette and design tokens

### Environment Setup
- Copy `.dev.vars.example` to `.dev.vars` (Cloudflare secrets)
- Copy `.env.example` to `.env` (Next.js environment variables)
- Configure D1 database and KV namespace in wrangler.jsonc

## Multi-tenancy System

### Team Structure
- Teams have members with system roles (admin, user) and custom team roles
- Team permissions stored as JSON in teamRoles table
- Team-scoped resources and billing
- Team invitations with email flow

### Credit System
- Credit-based billing with monthly refresh
- Three credit packages: 500 ($5), 1200 ($10), 3000 ($20)
- Transaction history and usage tracking
- Stripe payment integration

## Important Notes

- Always use existing utilities in `src/utils/` before creating new ones
- Check `package.json` for existing packages before adding new dependencies
- Global types go in `custom-env.d.ts`, not `cloudflare-env.d.ts`
- Email templates use React Email in `src/react-email/`
- Admin features are in `src/app/(admin)/` with role-based access