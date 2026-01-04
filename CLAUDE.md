# CLAUDE.md - AI Assistant Guide for Fluidd

This document provides essential context for AI assistants working on the Fluidd codebase.

## Project Overview

**Fluidd** is a free and open-source web interface for [Klipper](https://www.klipper3d.org/) 3D printer firmware. It communicates with printers through [Moonraker](https://github.com/Arksine/moonraker), Klipper's API server.

- **Version**: 1.36.2
- **License**: GPL-3.0
- **Repository**: https://github.com/fluidd-core/fluidd

## Technology Stack

| Category | Technology |
|----------|------------|
| Framework | Vue 2.7 with TypeScript |
| State Management | Vuex 3.6 (27 modules) |
| UI Framework | Vuetify 2.7 (Material Design) |
| Build Tool | Vite 6.4 |
| Testing | Vitest 3.2 with JSDOM |
| Linting | ESLint 9 with neostandard |
| Package Manager | npm (Node 22 or 24) |
| Styling | SCSS with Vuetify utilities |

### Key Dependencies
- **Vue Class Component** + **Vue Property Decorator** for class-based components
- **Axios** for HTTP requests, custom WebSocket plugin for real-time updates
- **ECharts** for data visualization
- **Monaco Editor** for config file editing
- **Vue i18n** for internationalization (15+ languages)

## Project Structure

```
src/
├── api/                    # HTTP and WebSocket API clients
│   ├── httpClientActions.ts
│   └── socketActions.ts
├── components/             # Vue components (~230 total)
│   ├── common/             # Reusable components
│   ├── layout/             # Header, nav, drawers
│   ├── settings/           # Settings panels
│   ├── ui/                 # UI elements
│   └── widgets/            # Dashboard widgets
├── directives/             # Vue directives
├── locales/                # i18n translations (YAML)
├── monaco/                 # Monaco editor config
├── plugins/                # Vue plugins (httpClient, socketClient, vuetify, i18n)
├── router/                 # Vue Router config
├── scss/                   # Global styles
├── store/                  # Vuex store modules (27 modules)
├── types/                  # TypeScript definitions
├── util/                   # Utility functions
├── views/                  # Page-level components (15 routes)
├── workers/                # Web workers
├── App.vue                 # Root component
├── globals.ts              # Global constants and MDI icons
├── init.ts                 # App initialization
└── main.ts                 # Entry point

public/                     # Static assets
server/                     # Nginx configuration
tests/unit/                 # Test setup and utilities
docs/                       # Jekyll documentation site
tools/                      # Build tools (svgo, theme converter)
```

## Development Commands

```bash
# Install dependencies
npm install

# Start development server (with HMR)
npm run dev

# Build for production
npm run build

# Run linting
npm run lint

# Run unit tests
npm run test:unit

# Type checking
npm run type-check

# Check for circular dependencies
npm run circular-check

# Install git hooks (run after clone)
npm run bootstrap
```

## Code Conventions

### Component Pattern
Components use Vue Class Component with TypeScript decorators:

```typescript
@Component({
  components: { ChildComponent }
})
export default class MyComponent extends Vue {
  @Prop({ required: true }) readonly data!: DataType
  @Prop({ default: false }) readonly disabled!: boolean

  localState = ''

  @Watch('data')
  onDataChange(newVal: DataType) { /* ... */ }

  get computedValue() { return this.data.value }

  handleClick() { /* ... */ }
}
```

### Vuex Store Structure
Each store module follows this pattern:
```
store/<module>/
├── index.ts      # Module registration
├── state.ts      # State definition with defaultState()
├── mutations.ts  # MutationTree
├── actions.ts    # ActionTree (async operations)
├── getters.ts    # GetterTree
└── types.ts      # TypeScript interfaces
```

### Styling
- Use scoped SCSS: `<style scoped lang="scss">`
- Global variables in `src/scss/variables.scss`
- Prefer Vuetify utility classes for layout

### Naming Conventions
- **Components**: PascalCase (`MyComponent.vue`)
- **Directories**: kebab-case or camelCase for store modules
- **Variables/Functions**: camelCase
- **Constants**: UPPER_SNAKE_CASE (see `globals.ts`)
- **Types/Interfaces**: PascalCase

## Linting Rules

ESLint configuration (key settings):
- Vue 2 recommended rules via `eslint-plugin-vue`
- TypeScript recommended via `typescript-eslint`
- `neostandard` for code style
- `camelcase`: off (allows snake_case for API compatibility)
- `@typescript-eslint/no-explicit-any`: off (but discouraged)
- `no-console`/`no-debugger`: warn in production only

**Pre-commit hook** runs:
1. ESLint with `--max-warnings 0` on staged files
2. SVG optimization via SVGO on `.vue`, `.svg` files and `globals.ts`

## Testing

- **Framework**: Vitest (Jest-compatible API)
- **Environment**: JSDOM
- **Test location**: Colocated with source in `__tests__/` directories
- **Pattern**: `*.spec.ts`

Example:
```typescript
describe('myFunction', () => {
  it.each([
    ['input1', 'expected1'],
    ['input2', 'expected2'],
  ])('should transform %s to %s', (input, expected) => {
    expect(myFunction(input)).toBe(expected)
  })
})
```

## Git Workflow

### Commit Messages
Follow [Conventional Commits](https://www.conventionalcommits.org/):
```
<type>(<scope>): <subject>

<body>

Signed-off-by: Your Name <email@example.com>
```

Types: `feat`, `fix`, `refactor`, `docs`, `style`, `chore`, `test`, `perf`

### Branch Strategy
- `master`: stable releases
- `develop`: development branch
- Feature branches should be rebased and squashed before merge

### CI Pipeline
On push/PR to develop/master:
1. `npm run lint` (no warnings allowed)
2. `npm run test:unit`
3. `npm run circular-check`
4. `npm run build`

## Important Files

| File | Purpose |
|------|---------|
| `src/globals.ts` | MDI icons and global constants |
| `src/init.ts` | Application initialization flow |
| `src/store/index.ts` | Vuex store with all modules |
| `src/plugins/socketClient.ts` | WebSocket connection to Moonraker |
| `src/api/socketActions.ts` | Socket action handlers |
| `public/config.json` | Host configuration for API endpoints |
| `vite.config.ts` | Build configuration |
| `vitest.config.ts` | Test configuration |

## Store Modules

The application uses 27 Vuex modules:
`afc`, `analysis`, `announcements`, `auth`, `charts`, `config`, `console`, `database`, `files`, `gcodePreview`, `history`, `jobQueue`, `layout`, `macros`, `mesh`, `mmu`, `notifications`, `power`, `printer`, `sensors`, `server`, `socket`, `spoolman`, `timelapse`, `version`, `wait`, `webcams`

## Common Patterns

### API Calls
```typescript
// HTTP via httpClientActions
await this.$http.get(...)

// WebSocket via socketActions
SocketActions.machineReboot()
```

### Accessing Store
```typescript
// In components
this.$store.state.printer.klippy.state
this.$store.getters['printer/getHeaterByName']('extruder')
this.$store.dispatch('files/uploadFile', payload)
this.$store.commit('config/setUiSettings', settings)
```

### i18n
```vue
<template>
  {{ $t('app.general.label.cancel') }}
</template>
```

Translation files are in `src/locales/` using YAML format.

## Deployment

- **Docker**: Alpine Nginx serving static files from `/dist`
- **Docker Unprivileged**: Available as `fluidd-unprivileged` image
- **Platforms**: amd64, arm/v6, arm/v7, arm64/v8, ppc64le, s390x
- **Registry**: `ghcr.io/fluidd-core/fluidd`

## Quick Reference

```bash
# Start developing
npm run dev

# Before committing
npm run lint
npm run test:unit

# Create release
npm run release:patch  # or :minor, :major
```
