# Welcome to the Novel Editor Learning Path 📚

> **Last Updated:** November 19, 2025
> **Tech Stack Research:** [View Current Tech Stack Analysis](./TECH_STACK_RESEARCH.md)
> **Documentation Plan:** [View Execution Plan](./EXECUTION_PLAN.md)

Welcome! This learning path will help you understand, contribute to, and integrate the **Novel** editor - a Notion-style WYSIWYG editor with AI-powered autocompletions built on Tiptap, React, and Next.js.

---

## 🎯 Quick Links

- **[Getting Started](./GETTING_STARTED.md)** - Set up your development environment
- **[Architecture Overview](./ARCHITECTURE_OVERVIEW.md)** - Understand the system design
- **[How-To Guide](./HOW_TO_GUIDE.md)** - Step-by-step guides for common tasks
- **[API Documentation](./API_DOCUMENTATION.md)** - Complete API reference
- **[FAQ](./FAQ.md)** - Answers to common questions

**External Resources:**
- 🌐 [Novel Homepage](https://novel.sh)
- 📦 [npm Package](https://www.npmjs.com/package/novel)
- 💻 [GitHub Repository](https://github.com/steven-tey/novel)
- 🎨 [Live Demo](https://novel.sh)

---

## 👥 Learning Paths by Role

### 🆕 New Developer Joining the Project

**You're a mid-level React/TypeScript developer ready to contribute to Novel.**

**Start Here:**
1. ✅ [Tech Stack Research](./TECH_STACK_RESEARCH.md) - Know what's current (Nov 2025)
2. ✅ [Getting Started](./GETTING_STARTED.md) - Clone, install, run
3. ✅ [Project Structure](./PROJECT_STRUCTURE.md) - Navigate the codebase
4. ✅ [Architecture Overview](./ARCHITECTURE_OVERVIEW.md) - Understand the design
5. ✅ [Tech Stack Guide](./TECH_STACK_GUIDE.md) - Learn the technologies
6. ✅ [Code Tours](./CODE_TOURS.md) - Follow key features through the code
7. ✅ [Development Workflow](./DEVELOPMENT_WORKFLOW.md) - Daily development process
8. ✅ [First Contributions](./FIRST_CONTRIBUTIONS.md) - Make your first PR

**Estimated Time:** 8-12 hours of reading + hands-on practice

---

### 🛠️ Contributor / Maintainer

**You want to extend Novel, fix bugs, or add features.**

**Your Path:**
1. ✅ [Getting Started](./GETTING_STARTED.md) - Set up environment
2. ✅ [Architecture Overview](./ARCHITECTURE_OVERVIEW.md) - System design
3. ✅ [Editor Core Guide](./EDITOR_CORE_GUIDE.md) - Deep dive into Tiptap
4. ✅ [Extension Development](./EXTENSION_DEVELOPMENT.md) - Create custom extensions
5. ✅ [State Management](./STATE_MANAGEMENT.md) - Jotai patterns
6. ✅ [Patterns & Conventions](./PATTERNS_AND_CONVENTIONS.md) - Code standards
7. ✅ [Testing Guide](./TESTING_GUIDE.md) - Test your changes
8. ✅ [Debugging Guide](./DEBUGGING_GUIDE.md) - Troubleshoot issues
9. ✅ [How-To Guide](./HOW_TO_GUIDE.md) - Common implementation tasks
10. ✅ [Development Workflow](./DEVELOPMENT_WORKFLOW.md) - Git, commits, publishing

**Estimated Time:** 12-16 hours

---

### 🔌 Integrator / Library User

**You want to use Novel in your own application.**

**Your Path:**
1. ✅ [Tech Stack Guide](./TECH_STACK_GUIDE.md) - Understand the technologies
2. ✅ [Integration Guide](./INTEGRATION_GUIDE.md) - **START HERE** - Install & configure
3. ✅ [API Documentation](./API_DOCUMENTATION.md) - Component & extension API
4. ✅ [Frontend Architecture](./FRONTEND_ARCHITECTURE.md) - Styling & theming
5. ✅ [AI Features Guide](./AI_FEATURES_GUIDE.md) - Optional AI integration
6. ✅ [Extension Development](./EXTENSION_DEVELOPMENT.md) - Custom extensions
7. ✅ [How-To Guide](./HOW_TO_GUIDE.md) - Common customizations
8. ✅ [Debugging Guide](./DEBUGGING_GUIDE.md) - Troubleshooting

**Estimated Time:** 4-8 hours

---

## 📖 Documentation Index

### 🚀 Getting Started & Setup

| Document | Description | Audience |
|----------|-------------|----------|
| [**Getting Started**](./GETTING_STARTED.md) | Installation, setup, first run | Everyone |
| [**Project Structure**](./PROJECT_STRUCTURE.md) | Codebase organization | New devs, contributors |
| [**Development Workflow**](./DEVELOPMENT_WORKFLOW.md) | Daily development process | Contributors |

---

### 🏗️ Architecture & Design

| Document | Description | Audience |
|----------|-------------|----------|
| [**Architecture Overview**](./ARCHITECTURE_OVERVIEW.md) | System design, component hierarchy | Everyone |
| [**Tech Stack Guide**](./TECH_STACK_GUIDE.md) | Technology explanations & comparisons | New devs, integrators |
| [**Data Flow Guide**](./DATA_FLOW_GUIDE.md) | Request lifecycle, state updates | Contributors |
| [**Frontend Architecture**](./FRONTEND_ARCHITECTURE.md) | Next.js app, components, styling | New devs, integrators |
| [**Editor Core Guide**](./EDITOR_CORE_GUIDE.md) | Tiptap deep dive, extensions | Contributors |
| [**State Management**](./STATE_MANAGEMENT.md) | Jotai patterns and architecture | Contributors |

---

### 🔧 Implementation & Development

| Document | Description | Audience |
|----------|-------------|----------|
| [**Integration Guide**](./INTEGRATION_GUIDE.md) | Using Novel in your app | Integrators |
| [**Extension Development**](./EXTENSION_DEVELOPMENT.md) | Creating custom extensions | Contributors, integrators |
| [**AI Features Guide**](./AI_FEATURES_GUIDE.md) | AI integration & customization | Integrators, contributors |
| [**Patterns & Conventions**](./PATTERNS_AND_CONVENTIONS.md) | Code standards, best practices | Contributors |
| [**How-To Guide**](./HOW_TO_GUIDE.md) | Step-by-step task guides | Everyone |
| [**Code Tours**](./CODE_TOURS.md) | Guided walkthroughs | New devs, contributors |

---

### 📋 Reference & Quality

| Document | Description | Audience |
|----------|-------------|----------|
| [**API Documentation**](./API_DOCUMENTATION.md) | Complete API reference | Everyone |
| [**Testing Guide**](./TESTING_GUIDE.md) | Testing strategies | Contributors |
| [**Debugging Guide**](./DEBUGGING_GUIDE.md) | Troubleshooting & common issues | Everyone |
| [**Tech Stack Research**](./TECH_STACK_RESEARCH.md) | Technology versions (Nov 2025) | Everyone |
| [**FAQ**](./FAQ.md) | Common questions & answers | Everyone |

---

### 🎓 Learning & Practice

| Document | Description | Audience |
|----------|-------------|----------|
| [**Exercises**](./EXERCISES.md) | Hands-on practice tasks | New devs, contributors |
| [**First Contributions**](./FIRST_CONTRIBUTIONS.md) | Making your first PR | New contributors |

---

## 🔬 Technology Stack Overview

Novel is built with modern web technologies. Here's what you'll be working with:

### Core Editor
- **Tiptap v2.11.2** - Rich text editor framework (✅ Current for v2.x)
- **ProseMirror** - Underlying editor engine (via Tiptap)

### Frontend Framework
- **Next.js v15.1.4** - React framework with App Router (⚠️ v16 available)
- **React v18.2.0** - UI library (🚨 v19 available)
- **TypeScript v5.7.3** - Type safety (✅ Recent)

### Styling & UI
- **Tailwind CSS v3.4.1** - Utility-first CSS (⚠️ v4 available - major rewrite)
- **Radix UI** - Headless UI components (✅ Current)
- **shadcn/ui** - Pre-built components (✅ Current)

### State Management
- **Jotai v2.11.0** - Atomic state management (✅ Very recent)

### AI & ML
- **Vercel AI SDK v3.0.12** - AI integration (🚨 v5 available)
- **OpenAI API v4.28.4** - Language models (✅ Current)

### Build & Tools
- **Turborepo v2.3.3** - Monorepo build system (⚠️ v2.6.1 available)
- **pnpm v9.5.0** - Package manager (🚨 v10.22.0 available)
- **Biome v1.9.4** - Linter/formatter (🚨 v2.3.6 available)
- **tsup v8.3.5** - TypeScript bundler (✅ Current)

**Legend:**
- ✅ **Current** - Up-to-date with latest stable
- ⚠️ **Slightly behind** - Minor versions behind, non-breaking
- 🚨 **Major version behind** - Consider upgrading

📊 **[View Full Tech Stack Analysis](./TECH_STACK_RESEARCH.md)** for version details, breaking changes, and upgrade paths.

---

## 🧠 Key Concepts to Understand

Before diving deep, familiarize yourself with these core concepts:

### 1. **Headless Component Pattern**
Novel provides the editor logic and structure but leaves styling to you.

💡 **Think of it as:** A car engine (Novel) that you can put in any car body (your app's design).

---

### 2. **Extension-Based Architecture**
Everything in the editor is an extension - headings, lists, images, AI completion, etc.

🧠 **Mental Model:** Like browser extensions or VS Code plugins, but for a text editor.

**Learn more:** [Editor Core Guide](./EDITOR_CORE_GUIDE.md)

---

### 3. **Monorepo Structure**
Novel uses Turborepo + pnpm to manage multiple packages in one repository:
- `/packages/headless` - The reusable editor library (npm: "novel")
- `/apps/web` - Next.js demo and reference implementation

🌉 **Compared to:** Nx, Lerna, or Yarn Workspaces monorepos.

**Learn more:** [Project Structure](./PROJECT_STRUCTURE.md)

---

### 4. **Atomic State Management (Jotai)**
Instead of one big state tree, Novel uses small, independent "atoms" of state.

💡 **Think of it as:** Recoil or individual useState hooks, but with global access.

**Learn more:** [State Management](./STATE_MANAGEMENT.md)

---

### 5. **Tiptap = React + ProseMirror**
Tiptap is a wrapper around ProseMirror (the powerful editor core) that makes it React-friendly.

🌉 **Compared to:** Draft.js, Slate, Quill - but more extensible.

**Learn more:** [Editor Core Guide](./EDITOR_CORE_GUIDE.md)

---

## 🎯 Common Use Cases

### I want to...

**...use Novel in my Next.js app**
→ [Integration Guide](./INTEGRATION_GUIDE.md)

**...add a custom slash command**
→ [How-To Guide](./HOW_TO_GUIDE.md#how-to-add-a-new-slash-command)

**...create a custom block type (node)**
→ [Extension Development](./EXTENSION_DEVELOPMENT.md#creating-custom-nodes)

**...customize the bubble menu**
→ [How-To Guide](./HOW_TO_GUIDE.md#how-to-customize-the-bubble-menu)

**...add AI features to my editor**
→ [AI Features Guide](./AI_FEATURES_GUIDE.md)

**...change the editor's theme/colors**
→ [Frontend Architecture](./FRONTEND_ARCHITECTURE.md#theming)

**...understand how slash commands work**
→ [Code Tours](./CODE_TOURS.md#tour-1-follow-a-keystroke)

**...fix a bug in the editor**
→ [Debugging Guide](./DEBUGGING_GUIDE.md)

**...contribute to the project**
→ [First Contributions](./FIRST_CONTRIBUTIONS.md)

**...understand the codebase quickly**
→ [Code Tours](./CODE_TOURS.md)

---

## ⚠️ Important Notes for Learners

### Version Awareness (November 2025)

This project uses some technologies that are **one major version behind** the latest:

1. **React 18** (latest: React 19)
   - All patterns in this project work perfectly
   - React 19 adds Server Actions and new hooks
   - Migration is non-breaking for most use cases

2. **Tailwind CSS v3** (latest: v4)
   - v4 is a complete rewrite with breaking changes
   - All v3 patterns here are still valid
   - New projects should consider v4

3. **Vercel AI SDK v3** (latest: v5)
   - v5 has breaking changes and improved type safety
   - If integrating Novel, consider using AI SDK v5 in your app

4. **pnpm v9** (latest: v10)
   - v10 has new features but is backward compatible
   - All pnpm commands work the same

**📖 Full details:** [Tech Stack Research](./TECH_STACK_RESEARCH.md)

---

### Documentation Accuracy Markers

Throughout these docs, you'll see these markers:

- ✅ **CURRENT (Nov 2025)** - Pattern/approach is up-to-date
- ⚠️ **OUTDATED PATTERN** - Works but newer approaches exist
- 🚨 **DEPRECATED** - Should not be used in new code
- 🆕 **NEW IN 2025** - Feature/pattern introduced recently
- 🔍 **NEEDS VERIFICATION** - Requires deeper investigation
- ❓ **ASSUMPTION** - Based on inference, not direct observation
- 🚧 **TODO** - Placeholder needing investigation

These markers ensure you know what's fact vs. inference and what's current vs. legacy.

---

## 🤝 Getting Help

### Within Documentation
- Check the [FAQ](./FAQ.md) first
- Search for your question in the index above
- Follow code links to see implementations

### External Resources
- **GitHub Issues:** [Report bugs or ask questions](https://github.com/steven-tey/novel/issues)
- **Tiptap Docs:** [Official Tiptap documentation](https://tiptap.dev/docs)
- **Next.js Docs:** [Official Next.js documentation](https://nextjs.org/docs)
- **Jotai Docs:** [Official Jotai documentation](https://jotai.org)

### Community
- **GitHub Discussions:** Ask questions and share ideas
- **Twitter/X:** Follow [@steventey](https://twitter.com/steventey) for updates

---

## 🗺️ Recommended Learning Order

### Week 1: Foundations
**Goal:** Understand the project structure and run it locally

1. [Getting Started](./GETTING_STARTED.md) - 1 hour
2. [Tech Stack Research](./TECH_STACK_RESEARCH.md) - 1 hour (skim)
3. [Project Structure](./PROJECT_STRUCTURE.md) - 2 hours
4. [Architecture Overview](./ARCHITECTURE_OVERVIEW.md) - 2 hours
5. [Tech Stack Guide](./TECH_STACK_GUIDE.md) - 3 hours
6. Run `pnpm dev` and explore the demo app - 2 hours

**Total:** ~11 hours

---

### Week 2: Deep Dive
**Goal:** Understand how the editor works internally

1. [Editor Core Guide](./EDITOR_CORE_GUIDE.md) - 3 hours
2. [Data Flow Guide](./DATA_FLOW_GUIDE.md) - 2 hours
3. [Code Tours](./CODE_TOURS.md) - 3 hours (follow along in IDE)
4. [State Management](./STATE_MANAGEMENT.md) - 2 hours
5. [Frontend Architecture](./FRONTEND_ARCHITECTURE.md) - 2 hours

**Total:** ~12 hours

---

### Week 3: Hands-On
**Goal:** Make changes and contribute

1. [Development Workflow](./DEVELOPMENT_WORKFLOW.md) - 2 hours
2. [Patterns & Conventions](./PATTERNS_AND_CONVENTIONS.md) - 2 hours
3. [How-To Guide](./HOW_TO_GUIDE.md) - 3 hours (try examples)
4. [Exercises](./EXERCISES.md) - 4 hours (complete beginner exercises)
5. [Extension Development](./EXTENSION_DEVELOPMENT.md) - 3 hours
6. [First Contributions](./FIRST_CONTRIBUTIONS.md) - 2 hours

**Total:** ~16 hours

---

### Week 4+: Specialization
**Choose based on your goals:**

**For Contributors:**
- [Testing Guide](./TESTING_GUIDE.md)
- [Debugging Guide](./DEBUGGING_GUIDE.md)
- [Exercises](./EXERCISES.md) - Advanced section
- Start contributing!

**For Integrators:**
- [Integration Guide](./INTEGRATION_GUIDE.md)
- [API Documentation](./API_DOCUMENTATION.md)
- [AI Features Guide](./AI_FEATURES_GUIDE.md)
- Build your integration!

---

## 📊 Progress Tracking

Use this checklist to track your learning:

### Foundations ✅
- [ ] Repository cloned and running locally
- [ ] Understand monorepo structure
- [ ] Familiar with tech stack
- [ ] Can navigate codebase

### Core Concepts ✅
- [ ] Understand Tiptap architecture
- [ ] Know how extensions work
- [ ] Understand state management (Jotai)
- [ ] Followed at least one code tour

### Practical Skills ✅
- [ ] Can add a slash command
- [ ] Can create a custom extension
- [ ] Can customize editor styling
- [ ] Can debug common issues

### Contribution Ready ✅
- [ ] Understand Git workflow
- [ ] Know code conventions
- [ ] Can run tests
- [ ] Made first contribution

---

## 🚀 What's Next?

**Ready to start?**
1. **New to the project?** → [Getting Started](./GETTING_STARTED.md)
2. **Want to integrate?** → [Integration Guide](./INTEGRATION_GUIDE.md)
3. **Want to contribute?** → [First Contributions](./FIRST_CONTRIBUTIONS.md)
4. **Need something specific?** → [How-To Guide](./HOW_TO_GUIDE.md)

---

## 📝 About This Documentation

**Created:** November 19, 2025
**Last Updated:** November 19, 2025
**Documentation Version:** 1.0
**Covers Project Version:** Novel v1.0.0 (packages/headless)

**Methodology:**
- Tech stack researched as of November 2025
- All code examples verified against actual codebase
- Uncertainty clearly marked with indicators
- Links to current official documentation

**Contributing to Docs:**
- Found an error? Open an issue
- Want to add content? Submit a PR
- Unclear explanation? Let us know

---

## 🎓 Learning Philosophy

This documentation is designed with these principles:

1. **Progressive Disclosure:** Start simple, add complexity gradually
2. **Learning by Doing:** Exercises and code tours for hands-on practice
3. **Honest Uncertainty:** Clear markers for what's known vs. inferred
4. **Current Information:** Tech stack research ensures accuracy (Nov 2025)
5. **Multiple Entry Points:** Different paths for different roles
6. **Real Code Examples:** Every concept links to actual implementation

**Happy learning! 🎉**

---

**[⬆️ Back to Top](#welcome-to-the-novel-editor-learning-path-)**
