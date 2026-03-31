# Prototype App Conventions

> Applies to `try-claude/prototype/` — the React + Vite + ShadCN v4 prototype app.

## Stack

- **React 19** with Vite 7 (dev server + build)
- **ShadCN v4** (`base-nova` style, Base UI primitives, not Radix)
- **Tailwind CSS v4** via `@tailwindcss/vite` plugin — no `tailwind.config.js`
- **TypeScript** with `@/*` path alias mapped to `./src/*`
- **Lucide React** for icons
- **cmdk** for command palette components
- **CSS variables** for design tokens (oklch color space) defined in `src/index.css`

## File Organization

```
src/
├── components/
│   ├── ui/           # ShadCN-generated primitives (button, card, dialog, etc.)
│   ├── layout/       # App shell components (app-sidebar, app-header)
│   ├── shared/       # Reusable domain components (badges, states, scores)
│   └── {feature}/    # Feature-specific components (e.g., suggestions/)
├── pages/            # Page-level components
├── hooks/            # Custom React hooks
├── lib/              # Utilities (utils.ts with cn() helper)
├── types/            # Shared TypeScript type definitions
├── data/             # Mock data files (mock-*.ts)
└── assets/           # Static assets (SVGs, images)
```

## Conventions

- **Do not manually edit** files in `src/components/ui/` — use `npx shadcn@latest add <component>`
- **Style composition**: Use `cn()` from `@/lib/utils` (combines `clsx` + `tailwind-merge`)
- **Path aliases**: Always import with `@/` prefix, never relative paths crossing directory boundaries
- **Component naming**: PascalCase for components, kebab-case for file names
- **No RSC**: `"rsc": false` in components.json — this is a client-side SPA
- **Font**: Geist Variable via `@fontsource-variable/geist`
