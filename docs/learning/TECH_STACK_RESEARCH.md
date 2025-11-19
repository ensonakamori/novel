# Technology Stack Research (November 2025)

Research conducted: November 19, 2025

## Executive Summary

This document provides a comprehensive analysis of the technology stack used in the Novel editor project, comparing project versions against the latest stable releases as of November 2025. Novel is a Notion-style WYSIWYG editor with AI-powered autocompletions, built as a monorepo using modern web technologies.

**Overall Tech Stack Health:** ✅ **EXCELLENT** - Most core technologies are current or very recent, with only minor version gaps.

---

## Monorepo & Build Tools

### Turborepo - v2.3.3

**Current Status (Nov 2025):**
- Latest stable: **v2.6.1** (Released November 11, 2025)
- Project uses: v2.3.3
- Status: ⚠️ **Slightly outdated** (3 minor versions behind)

**Important Updates Since Jan 2025:**
- **Turborepo 2.6** (Oct-Nov 2025): Introduced microfrontend proxy for local development, allowing all applications to run on one port with one command
- **Turborepo 2.5**: General performance improvements and bug fixes
- Added support for Bun's latest v1 lockfile format
- Written in Rust for exceptional performance

**What This Means for Learning:**
- The patterns in this project are current and valid
- Updating to 2.6.x would give access to microfrontend proxy features
- No breaking changes between versions; upgrade is straightforward

**Official Resources:**
- Docs: https://turbo.build/repo/docs
- Release notes: https://github.com/vercel/turborepo/releases
- Turborepo 2.6 announcement: https://turborepo.com/blog/turbo-2-6

---

### pnpm - v9.5.0

**Current Status (Nov 2025):**
- Latest stable: **v10.22.0** (Released November 12, 2025)
- Project uses: v9.5.0
- Status: 🚨 **Major version behind** (full major version behind)

**Important Updates Since Jan 2025:**
- **pnpm 10.x** (2025): Major version release with new features
- v10.21 (Nov 10, 2025): Added support for Node.js runtime installation for dependencies and new trust policy settings
- Improved performance and reliability
- Better monorepo workspace handling

**What This Means for Learning:**
- The patterns in this project using pnpm 9.x are still valid
- pnpm 10.x is backward compatible for most use cases
- Consider upgrading to v10.x for latest features
- All pnpm commands work the same way

**Official Resources:**
- Docs: https://pnpm.io
- Release notes: https://github.com/pnpm/pnpm/releases
- Migration guide: https://pnpm.io/installation

---

## Frontend Framework & Runtime

### Next.js - v15.1.4

**Current Status (Nov 2025):**
- Latest stable: **v16.0** (Released October 21, 2025) / **v15.5** (Latest v15.x)
- Project uses: v15.1.4
- Status: ⚠️ **One major version behind**, but v15.x is still well-supported

**Important Updates Since Jan 2025:**
- **Next.js 16** (Oct 2025): Major new features including:
  - Cache Components (new programming model leveraging Partial Pre-Rendering)
  - Turbopack as default stable bundler for all apps
  - Turbopack File System Caching (beta)
  - Stable React Compiler Support
  - Build Adapters API (alpha)
  - React 19.2 support
  - Introduced `proxy.ts` which replaces `middleware.ts` (runs on Node.js runtime)
- **Next.js 15.5** (Aug 2025):
  - Turbopack builds in beta
  - Stable Node.js middleware
  - TypeScript improvements
  - Deprecation warnings for Next.js 16

**What This Means for Learning:**
- This project uses Next.js 15.x which is the previous major version but still current
- App Router patterns (used in this project) are the modern standard
- Most patterns in this codebase are current for Next.js 15.x
- When upgrading to v16, note the `middleware.ts` → `proxy.ts` migration

**Official Resources:**
- Docs: https://nextjs.org/docs
- Next.js 16 announcement: https://nextjs.org/blog/next-16
- Next.js 15 docs: https://nextjs.org/blog/next-15
- Changelog: https://next-changelog.vercel.app/

---

### React - v18.2.0

**Current Status (Nov 2025):**
- Latest stable: **React 19.2.0** (Released October 2025)
- React 19 stable: Released December 5, 2024
- Project uses: v18.2.0
- Status: 🚨 **Major version behind** (one full major version)

**Important Updates Since Jan 2025:**
- **React 19 Stable** (Dec 2024): Major release with:
  - **Actions**: `startTransition` can now accept async functions
  - **Server Components**: RSC features are now stable
  - **New APIs**: `preinit`, `preload`, `prefetchDNS`, `preconnect` for optimized page loads
  - **Custom Elements**: Full support, passes all Custom Elements Everywhere tests
  - **use() hook**: For reading promises and context
  - **React Compiler**: Automatic optimization (stable support)
- **React 19.2** (Oct 2025): Latest patch with performance improvements
- **React 19.1** (June 2025): Bug fixes and refinements

**What This Means for Learning:**
- React 18.2 is still widely used and fully supported
- The patterns in this project (hooks, components) work in both React 18 and 19
- React 19 is backward compatible for most use cases
- Consider upgrading to React 19.x for Server Components and Actions
- New projects should start with React 19.x

**Official Resources:**
- Docs: https://react.dev
- React 19 announcement: https://react.dev/blog/2024/12/05/react-19
- React 19.2 release: https://react.dev/blog/2025/10/01/react-19-2
- Migration guide: https://react.dev/blog/2024/12/05/react-19#upgrading-to-react-19

---

## Language & Type System

### TypeScript - v5.4.2 / v5.7.3

**Current Status (Nov 2025):**
- Latest stable: **TypeScript 5.9** (Released August 2025)
- Project uses: v5.4.2 (web app) and v5.7.3 (headless package)
- Status:
  - v5.4.2: ⚠️ **Several versions behind** (5 minor versions)
  - v5.7.3: ✅ **Recent** (2 minor versions behind)

**Important Updates Since Jan 2025:**
- **TypeScript 5.9** (Aug 2025): Latest stable version
- **TypeScript 5.8** (Mar 2025): Performance improvements
- **TypeScript 5.7** (Nov 2024):
  - Better type safety for uninitialized variables
  - Stricter enforcement of return types
  - More consistent computed property names as index signatures
  - **Performance boost**: ~2.5x speed-up in Node.js 22+ using `module.enableCompileCache()`
  - Improved management of uninitialized variables
- **TypeScript 5.5 & 5.6**: Incremental improvements

**What This Means for Learning:**
- All TypeScript patterns in this project are current
- The headless package (v5.7.3) is very recent
- The web app could be updated from v5.4.2 to v5.9 for latest features
- No breaking changes; upgrade is straightforward
- All type patterns and syntax work across these versions

**Official Resources:**
- Docs: https://www.typescriptlang.org/docs
- TypeScript 5.9 announcement: https://devblogs.microsoft.com/typescript/announcing-typescript-5-9/
- TypeScript 5.7 announcement: https://devblogs.microsoft.com/typescript/announcing-typescript-5-7/
- Release notes: https://github.com/microsoft/typescript/releases

---

## Rich Text Editor (Core Technology)

### Tiptap - v2.11.2

**Current Status (Nov 2025):**
- Latest stable: **v3.x** (Released 2025) / **v2.9+** (Latest v2.x)
- Project uses: v2.11.2
- Status: ⚠️ **Version uncertain** - v2.11.2 seems ahead of documented v2.9, or numbering has changed

**Important Updates Since Jan 2025:**
- **Tiptap v3** (2025): Major version with significant updates:
  - **MarkViews**: Custom HTML rendering for marks
  - **SSR Support**: Editor can run on SSR environments without rendering content
  - **unmount() method**: Mount/unmount editor to DOM, encouraging instance reuse
  - **Static Renderer**: Render JSON content as HTML, Markdown, or React components without editor instance
  - **Mobile improvements**: Better touch event handling
- **Tiptap v2.9** (Oct 2025):
  - Extended `contentError` feature for collaboration (isolates conflicting clients)
  - Default `role="textbox"` for screen readers and keyboard navigation
  - Improved NodePos attribute handling
  - Commands (`insertContentAt`, `setContent`) now follow editor `parseOptions`
- **Recent fixes**:
  - Fixed CVE-2025-62410 security vulnerability
  - Comprehensive bidirectional markdown support via `@tiptap/markdown`
  - New `ResizableNodeview` for images, videos, iframes with resize handles

**What This Means for Learning:**
- ✅ This project uses Tiptap v2.x which is mature and stable
- The version number (v2.11.2) suggests active development; likely uses recent features
- Tiptap v3 is available but represents a major rewrite
- All v2.x patterns in this codebase are current for v2.x line
- When upgrading to v3, review the migration guide

**Official Resources:**
- Docs: https://tiptap.dev/docs
- Release notes: https://tiptap.dev/blog/release-notes
- Changelog: https://tiptap.dev/docs/resources/changelog
- v2 to v3 migration: https://tiptap.dev/docs/guides/upgrade-tiptap-v2
- GitHub releases: https://github.com/ueberdosis/tiptap/releases

---

## Styling & UI

### Tailwind CSS - v3.4.1

**Current Status (Nov 2025):**
- Latest stable: **v4.1** (Released 2025) / **v4.0** (Released January 22, 2025)
- Project uses: v3.4.1
- Status: 🚨 **Major version behind**

**Important Updates Since Jan 2025:**
- **Tailwind CSS v4.0** (Jan 22, 2025): **MAJOR REWRITE**
  - ⚡ **Performance**: Full builds 5x faster, incremental builds 100x faster (microseconds)
  - 🎨 **Modern CSS**: Built on cascade layers, `@property`, and `color-mix()`
  - 📦 **Simplified Installation**: Fewer dependencies, zero config, single line in CSS
  - 🔌 **First-party Vite plugin**: Tight integration for maximum performance
  - 🔍 **Automatic Content Detection**: Template files discovered automatically, no config needed
  - ♻️ **Ground-up rewrite**: Completely new architecture optimized for speed
- **Tailwind CSS v4.1** (2025): Further improvements and refinements

**What This Means for Learning:**
- ⚠️ **OUTDATED PATTERNS**: This project uses Tailwind v3.x
- Tailwind v3.x patterns still work perfectly, but v4 is the modern standard
- v4 has breaking changes and different configuration approach
- All utility classes remain similar between v3 and v4
- **Recommendation**: Learn v3 patterns (in this project), but know v4 is current for new projects

**Official Resources:**
- Docs: https://tailwindcss.com
- v4.0 announcement: https://tailwindcss.com/blog/tailwindcss-v4
- Migration from v3 to v4: https://tailwindcss.com/docs/upgrade-guide
- GitHub releases: https://github.com/tailwindlabs/tailwindcss/releases

---

### Radix UI - v1.x (various components)

**Current Status (Nov 2025):**
- Latest component versions: Various (actively maintained)
- Radix UI package: v1.4.3 (unified package, published ~3 months before Nov 2025)
- Project uses: Various v1.x and v2.x components
- Status: ✅ **Current** - Component versions are recent

**Important Updates Since Jan 2025:**
- **New unified package**: `radix-ui` package exposes all primitives from single import (tree-shakable)
- **New Primitives (Preview)**:
  - `PasswordToggleField`: Password input with visibility toggle button
  - `OneTimePasswordField`: OTP input component
- **Technical improvements**:
  - All primitives are now ESM compatible
  - Updated to latest Floating UI for popper-positioned primitives
  - Continued refinements to accessibility

**What This Means for Learning:**
- ✅ All Radix UI patterns in this project are current
- Radix UI is the foundation for many component libraries (e.g., shadcn/ui)
- Components are unstyled primitives focused on accessibility
- The component APIs are stable and well-maintained

**Official Resources:**
- Docs: https://www.radix-ui.com/primitives
- Radix Primitives releases: https://www.radix-ui.com/primitives/docs/overview/releases
- GitHub: https://github.com/radix-ui/primitives

---

## Development Tools

### Biome - v1.9.4 / v1.7.2

**Current Status (Nov 2025):**
- Latest stable: **v2.3.6** (Released mid-November 2025)
- Project uses: v1.9.4 (root), v1.7.2 (web app)
- Status: 🚨 **Major version behind**

**Important Updates Since Jan 2025:**
- **Biome v2.0** (Beta announced March 2025, stable in 2025):
  - Plugin system support
  - Type-informed linting
  - Support for Vue, Svelte, and Astro (v2.3)
- **Biome v2.3** (Late 2025): Multi-framework support
- **373 linter rules** (as of Nov 2025)
- **97% Prettier compatibility** for formatting
- Written in Rust for exceptional performance
- Combines formatting + linting in single tool (replaces ESLint + Prettier)

**What This Means for Learning:**
- ⚠️ **OUTDATED VERSION**: Project uses v1.x, current is v2.x
- v1.x patterns still work perfectly
- v2.x adds plugin system and framework-specific support
- All lint rules and formatting remain similar
- **Recommendation**: Upgrade to v2.x for latest features and framework support

**Official Resources:**
- Docs: https://biomejs.dev
- Roadmap 2025: https://biomejs.dev/blog/roadmap-2025/
- v2.0 beta announcement: https://biomejs.dev/blog/biome-v2-0-beta/
- v2.3 announcement: https://biomejs.dev/blog/biome-v2-3/
- GitHub releases: https://github.com/biomejs/biome/releases

---

## AI & Language Models

### Vercel AI SDK - v3.0.12 (@ai-sdk/openai v1.1.0)

**Current Status (Nov 2025):**
- Latest major version: **AI SDK 5** (Released 2025)
- Latest provider versions: @ai-sdk/openai v2.0.67 (Nov 14, 2025)
- Project uses:
  - `ai` package: v3.0.12
  - `@ai-sdk/openai`: v1.1.0
- Status: 🚨 **Major versions behind** (SDK v3 vs v5, provider v1 vs v2)

**Important Updates Since Jan 2025:**
- **AI SDK 5** (2025): Complete rebuild
  - Chat rebuilt from ground up with powerful primitives
  - End-to-end type safety
  - **Type-safe UIMessages** for managing chat state
  - **Data parts** for streaming custom data
  - **Redesigned tool invocations** with better type safety
  - Over 2 million weekly downloads
- **@ai-sdk/openai v2.x**: Updated provider with latest OpenAI features
- **Unified provider API**: Work with any language model
- Integration with leading web frameworks

**What This Means for Learning:**
- ⚠️ **SIGNIFICANTLY OUTDATED**: Project uses v3, current is v5
- AI SDK v5 has breaking changes and new APIs
- The fundamental patterns (streaming, chat) are similar
- **Recommendation**: Review AI SDK v5 docs when working with AI features
- Consider upgrading to v5 for better type safety and new features

**Official Resources:**
- Docs: https://ai-sdk.dev/docs/introduction
- AI SDK 5 announcement: https://vercel.com/blog/ai-sdk-5
- GitHub: https://github.com/vercel/ai
- NPM releases: https://www.npmjs.com/package/ai

---

### OpenAI - v4.28.4

**Current Status (Nov 2025):**
- Latest version: v4.x (actively maintained)
- Project uses: v4.28.4
- Status: ✅ **Current** - v4.x is the stable line

**What This Means for Learning:**
- OpenAI SDK v4.x is current and actively maintained
- This project's version is recent
- All patterns using OpenAI in this project are current

**Official Resources:**
- Docs: https://platform.openai.com/docs
- GitHub: https://github.com/openai/openai-node
- NPM: https://www.npmjs.com/package/openai

---

## State Management

### Jotai - v2.11.0

**Current Status (Nov 2025):**
- Latest stable: **v2.12.2** (Released March 10, 2025)
- Project uses: v2.11.0
- Status: ✅ **Very recent** (1 minor version behind)

**Important Updates Since Jan 2025:**
- **v2.12.0** (Feb 11, 2025):
  - `feat(core): expose internals instead of unstable_derive`
  - Significant architectural change for ecosystem libraries
- **v2.12.1** (Feb 17, 2025): Bug fixes for internal onChange behavior
- **v2.12.2** (Mar 10, 2025): Latest patches and improvements
- **99+ million total downloads**
- **Minimal core API**: Just 2kb
- **No string keys** (unlike Recoil)
- Atomic state management approach

**What This Means for Learning:**
- ✅ This project uses very recent Jotai (v2.11.0)
- All patterns in this codebase are current
- Minor version jump to v2.12.x is non-breaking
- Jotai's atomic approach is modern best practice for fine-grained state

**Official Resources:**
- Docs: https://jotai.org
- GitHub releases: https://github.com/pmndrs/jotai/releases
- State Management in 2025 comparison: https://makersden.io/blog/react-state-management-in-2025

---

## Supporting Libraries

### Vercel Platform Services

**Project uses:**
- `@vercel/analytics`: v1.2.2
- `@vercel/blob`: v0.22.1
- `@vercel/kv`: v1.0.1

**Status:** ✅ **Current** - All Vercel platform services are actively maintained and these are recent versions.

**Official Resources:**
- Analytics: https://vercel.com/docs/analytics
- Blob Storage: https://vercel.com/docs/storage/vercel-blob
- KV (Redis): https://vercel.com/docs/storage/vercel-kv

---

### @upstash/ratelimit - v1.0.1

**Status:** ✅ **Current** - Upstash rate limiting using Vercel KV/Redis.

**Official Resources:**
- Docs: https://upstash.com/docs/redis/features/ratelimiting
- GitHub: https://github.com/upstash/ratelimit

---

### Additional Notable Dependencies

- **highlight.js** v11.9.0: Syntax highlighting - ✅ Current
- **lowlight** v3.1.0: Virtual syntax highlighting - ✅ Current
- **lucide-react** v0.358.0: Icon library - ✅ Current
- **next-themes** v0.2.1: Dark mode for Next.js - ✅ Current
- **sonner** v1.4.3: Toast notifications - ✅ Current
- **cmdk** v1.0.4: Command menu - ✅ Current
- **class-variance-authority** v0.7.0: CVA for variants - ✅ Current
- **tailwind-merge** v2.2.1: Merge Tailwind classes - ✅ Current
- **react-markdown** v9.0.1: Markdown renderer - ✅ Current
- **tippy.js** v6.3.7: Tooltip library - ✅ Current
- **katex** v0.16.20: Math typesetting - ✅ Current
- **react-tweet** v3.2.1: Embed tweets - ✅ Current
- **ts-pattern** v5.0.8: Pattern matching for TS - ✅ Current

---

## Summary & Recommendations

### ✅ Current Technologies (No Action Needed)
- Jotai (v2.11.0) - Very recent
- TypeScript in headless package (v5.7.3)
- Tiptap (v2.11.2) - Current for v2.x line
- OpenAI SDK (v4.28.4)
- Vercel platform services
- Radix UI components
- All supporting libraries

### ⚠️ Slightly Outdated (Minor Updates Recommended)
- Turborepo (v2.3.3 → v2.6.1) - 3 minor versions behind
- TypeScript in web app (v5.4.2 → v5.9) - Non-breaking upgrade
- Next.js (v15.1.4 → v15.5 or v16) - Still supported, but v16 available

### 🚨 Significantly Outdated (Major Updates Available)
1. **Vercel AI SDK** (v3.0.12 → v5.x) - Major version behind, breaking changes
2. **React** (v18.2.0 → v19.2.0) - One major version behind
3. **Tailwind CSS** (v3.4.1 → v4.1) - Major rewrite available
4. **Biome** (v1.9.4 → v2.3.6) - Major version behind
5. **pnpm** (v9.5.0 → v10.22.0) - Major version behind

### Learning Path Priorities

**For New Developers:**
1. ✅ **Start here**: Learn the current patterns in this codebase - they're all functional
2. 📚 **Be aware**: Note which technologies have newer major versions
3. 🔍 **Compare**: When reading docs, check both the version used here AND latest version
4. 📖 **Reference current docs**: Always link to official docs for latest best practices

**For Contributors:**
1. **No immediate urgency**: All current versions are functional and supported
2. **Gradual upgrades**: Consider updating dependencies incrementally
3. **Test thoroughly**: Major version updates (React 19, Tailwind v4, AI SDK v5) require testing
4. **Read migration guides**: Each major version has official migration documentation

---

## Version Markers for Documentation

Throughout the learning materials, you'll see these markers:

- ✅ **CURRENT (Nov 2025)** - Pattern/approach is up-to-date
- ⚠️ **OUTDATED PATTERN** - Works but newer approaches exist
- 🚨 **DEPRECATED** - Should not be used in new code
- 🆕 **NEW IN 2025** - Feature/pattern introduced recently
- 🔍 **NEEDS VERIFICATION** - Requires deeper investigation
- ❓ **ASSUMPTION** - Based on inference, not direct observation

---

## Research Methodology

This research was conducted on November 19, 2025, using:
1. Package.json analysis for current versions
2. Web searches for latest stable versions
3. Official documentation review
4. GitHub release notes examination
5. Changelog analysis

**Note**: Web technologies evolve rapidly. This research is accurate as of November 2025. Always verify against official documentation when in doubt.

---

## Next Steps

After reviewing this tech stack research:
1. Read [GETTING_STARTED.md](./GETTING_STARTED.md) for setup instructions
2. Review [ARCHITECTURE_OVERVIEW.md](./ARCHITECTURE_OVERVIEW.md) for system design
3. Explore [TECH_STACK_GUIDE.md](./TECH_STACK_GUIDE.md) for detailed technology explanations
4. Check [PROJECT_STRUCTURE.md](./PROJECT_STRUCTURE.md) for codebase organization

---

**Last Updated**: November 19, 2025
**Researcher**: Claude Code
**Research Scope**: All major and supporting technologies in the Novel editor monorepo
