# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev       # Start dev server with Turbopack (http://localhost:3000)
npm run build     # Production build with Turbopack
npm run start     # Start production server
npm run lint      # Run ESLint
```

There are no tests in this project.

## Environment Variables

Create a `.env.local` file at the root with:
- `GITHUB_USERNAME` / `GITHUB_TOKEN` (read:user scope) — used by `app/api/github-stats/route.js` for live GitHub GraphQL stats
- `CONTACT_EMAIL_USER` / `CONTACT_EMAIL_APP_PASSWORD` — Gmail account + app password used by `app/api/contact/route.js` (nodemailer) to send contact form emails
- `CONTACT_TO_EMAIL` (optional) — inbox that receives contact form submissions; falls back to `CONTACT_EMAIL_USER` if unset

## Architecture

**Next.js 15 App Router** portfolio site for Sadekur Rahman. All pages live in `app/` and use `.jsx`. Path alias `@` maps to the project root (configured in `jsconfig.json`).

### Pages
- `/` — Home: hero section with photo, social links, and animated stat counters; includes mouse-tracking `HeroGlow` background effect
- `/services` — Grid of offered services with hover animations; service data defined inline as a `services` array
- `/resume` — Tabbed view of experience, education, skills, and about info (data defined inline as constants at the top of the file)
- `/work` — Swiper-based project slider (project data defined inline as a `projects` array)
- `/contact` — Contact form (name, email, phone, service selector, message) alongside a contact info list; submits to `/api/contact`
- `/blog` — Grid of blog post previews (post data defined inline as a `posts` array); posts are not yet linked to individual detail pages
- `/api/github-stats` — Server-side API route that fetches live GitHub GraphQL data; cached 1 hour via `next: { revalidate: 3600 }`
- `/api/contact` — POST endpoint that sends contact form submissions as email via `nodemailer` (Gmail transport); validates required fields and returns JSON `{ error }` / `{ success }`

### Page Transition System
Route changes trigger an animated page transition defined in `app/layout.jsx`:

- **`PageTransection`** (`components/PageTransection.jsx`) — wraps all page content. Uses Framer Motion `AnimatePresence mode="wait"` keyed on `usePathname()`, so each navigation re-mounts the animation. Internally uses a **`FrozenRouter`** pattern (via `LayoutRouterContext`) to freeze the exiting route's subtree so it renders correctly during its exit animation. Animation timings come from `lib/variants.js` (`pageVariants`).

- **`StairEffect`** (`components/StairEffect.jsx`) — a stair-step block animation component that exists in the codebase but is **not currently mounted in the layout**. It can be added back to `app/layout.jsx` if the stair animation is desired.

Child elements that want stagger-in animation can use `childVariants` from `lib/variants.js` as Framer Motion variants.

### Hire Me Modal
`components/HireMeModal.jsx` is a Radix `Dialog` (not a route) rendered inside `Header.jsx`'s desktop nav as the "Hire Me" trigger button. It contains a service-inquiry form (name, email, phone, service selector via `CustomSelect`, message) styled to match the site's dark theme; the form has no submit handler wired up yet (`onSubmit` just calls `preventDefault`).

### Custom Select
Both the Hire Me Modal and the `/contact` page use `components/ui/CustomSelect.jsx` for the service dropdown — a hand-rolled replacement for the Radix `Select` primitive (removed from the codebase; `@radix-ui/react-select` is no longer a dependency). Use `CustomSelect`, not Radix, for any new dropdown UI.

### Component Structure
- `components/` — Layout and feature components (Header, Nav, MobileNav, NameTicker, Photo, PixelPhoto, Social, Status, HeroGlow, StairEffect, PageTransection, WorkSliderButton, HireMeModal)
- `components/ui/` — shadcn/ui primitives (button, input, tabs, tooltip, scroll-area, sheet, textarea) plus `CustomSelect.jsx` and `Stairs.jsx`
- `lib/utils.js` — `cn()` utility (clsx + tailwind-merge)
- `lib/variants.js` — shared Framer Motion variants: `pageVariants` (page enter/exit) and `childVariants` (staggered child elements)

### Styling
- **Tailwind CSS v3** with custom config in `tailwind.config.js` (v3 `@tailwind` directives in `globals.css`, processed via `postcss.config.js`'s `tailwindcss` plugin).
- Key custom tokens: `primary` = `#0d0d1a` (dark bg), `accent` = CSS var (violet-purple `#8b5cf6`, defined in `globals.css`), `popover.DEFAULT` = `#8b5cf6` with hover `#7c3aed`
- Font: JetBrains Mono loaded via `next/font/google`, applied as `font-primary` class via CSS variable `--font-jetbrains-mono`
- shadcn/ui components use Radix UI primitives; `components.json` configures the shadcn setup
- Two ESLint config files coexist: `.eslintrc.json` (legacy format) and `eslint.config.mjs` — `.eslintrc.json` holds the active custom rules

### Data Pattern
Content data (projects, experience, education, skills, services) is defined as plain JS objects/arrays at the top of each page file — there is no external CMS or data layer. To update portfolio content, edit the constants directly in the relevant page file.

The CV file (`CV of Sadekur Rahman.pdf`) lives at the repo root and is intended to be served as a downloadable asset from the home page's "Download CV" button.

### `Status` Component
Fetches `/api/github-stats` on mount and feeds live `totalRepos` and `totalCommits` into `react-countup` animated counters. Falls back to hardcoded values if the API fails.
