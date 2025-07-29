---
title: "How to Update to Nuxt 4: Complete Upgrade Guide"
date: 2025-07-27 20:50:14 +0530
---
Nuxt 4 is here! 🎉 After a year of real-world testing, the stable release of Nuxt 4 has landed—bringing enhanced performance, streamlined developer experience, and powerful new features. This comprehensive guide will walk you through everything you need to know about upgrading to Nuxt 4.

## What's Included?
Nuxt 4 includes all the features you've been testing with `compatibilityVersion: 4`:

- **🗂️ New Directory Structure** - code goes in `app/` for cleaner organization and better IDE performance
- **🔄 Improved Data Fetching** - smarter `useAsyncData` and `useFetch` with better caching and cleanup
- **🏷️ Consistent Component Names** - Vue DevTools and `<KeepAlive>` now see the same names as Nuxt's auto-imports
- **📄 Enhanced Head Management** - dropping deprecated features from Unhead v2 with better performance and tag optimization
- improvements to type 'environment' handling (for server, client, and shared code)
## Getting Started: Upgrading to Nuxt 4

Now that Nuxt 4 is officially released, here's how to upgrade your project:
### Step 1: Update Nuxt (Recommended)

The recommended way to upgrade to Nuxt 4 is using the upgrade command with deduplication:
```bash
# This will deduplicate your lockfile and ensure you get updates from dependencies
npx nuxt upgrade --dedupe

# use `npx nuxt upgrade` for other options
```

>If you prefer a gradual migration approach, you can still use the `compatibilityVersion` option in your existing Nuxt 3.12+ project:

## Major Breaking Changes and Migration Steps

Let's explore the most significant changes you'll encounter when upgrading to Nuxt 4.

### 1. New Directory Structure (Significant Impact)

**What Changed:** Nuxt 4 introduces a new default `srcDir` directory (`app/`) that separates your frontend code from configuration and server files.

**New Structure:**
```
.output/
.nuxt/
app/             # New default srcDir
  assets/
  components/
  composables/
  layouts/
  middleware/
  pages/
  plugins/
  utils/
  app.config.ts
  app.vue
  router.options.ts
content/
layers/
modules/
node_modules/
public/
shared/         # New shared directory
server/
  api/
  middleware/
  plugins/
  routes/
  utils/
nuxt.config.ts
```

**Migration Steps:**

1. Create a new `app/` directory in your project root
2. Move these folders into `app/`:
    - `assets/`
    - `components/`
    - `composables/`
    - `layouts/`
    - `middleware/`
    - `pages/`
    - `plugins/`
    - `utils/`
    - `app.vue`
    - `error.vue`
    - `app.config.ts`
3. Keep these at the root level:
    - `nuxt.config.ts`
    - `content/`
    - `layers/`
    - `modules/`
    - `public/`
    - `server/`

**Automated Migration:**
```bash
npx codemod@latest nuxt/4/file-structure
```

**Why This Change?**

- **Performance**: Keeps your code separate from `node_modules/` and `.git/`, making file watchers faster (especially on Windows and Linux)
- **IDE Support**: Better context about whether you're working with client or server code
- **Organization**: Cleaner separation between application code and configuration

### 2. Enhanced TypeScript Configuration (Minimal Impact)

**What Changed:** Nuxt now creates separate TypeScript projects for your app code, server code, shared/ folder, and builder code. This should mean better autocompletion, more accurate type inference and fewer confusing errors when you're working in different contexts.

The separate configurations are:

- `.nuxt/tsconfig.app.json` - App code (components, composables)
- `.nuxt/tsconfig.server.json` - Server-side code
- `.nuxt/tsconfig.node.json` - Build-time code
- `.nuxt/tsconfig.shared.json` - Shared code

**Migration Steps (Optional but Recommended):**

1. **Update root tsconfig.json to use project references:**

```json
{
  "files": [],
  "references": [
    { "path": "./.nuxt/tsconfig.app.json" },
    { "path": "./.nuxt/tsconfig.server.json" },
    { "path": "./.nuxt/tsconfig.shared.json" },
    { "path": "./.nuxt/tsconfig.node.json" }
  ]
}
```

2. **Update type checking scripts:**

```json
{
  "scripts": {
    "typecheck": "nuxt prepare && vue-tsc -b --noEmit"
  }
}
```

## Automated Migration with Codemods

Nuxt 4 provides automated migration tools to handle many changes. These are particularly useful for large codebases:

#### Run All Migrations
```bash
npx codemod@latest nuxt/4/migration-recipe
yarn dlx codemod@latest nuxt/4/migration-recipe
pnpm dlx codemod@latest nuxt/4/migration-recipe  
bun x codemod@latest nuxt/4/migration-recipe
```

#### Individual Migrations
```bash
# File structure migration
npx codemod@latest nuxt/4/file-structure

# Data fetching updates  
npx codemod@latest nuxt/4/default-data-error-value
npx codemod@latest nuxt/4/shallow-function-reactivity

# Component updates
npx codemod@latest nuxt/4/deprecated-dedupe-value
```

## Benefits of Upgrading

After completing the migration, you'll enjoy:

- **Better Performance**: Improved file watching, faster CLI, and build times
- **Enhanced Developer Experience**: Better IDE support, type safety, and autocompletion
- **Improved Memory Management**: Automatic cleanup of unused data
- **Faster Development**: Socket-based communication and native file watching
- **Better Separation of Concerns**: Clear boundaries between app, server, and build code
- **Modern Templates**: Updated UI templates with improved accessibility

## Conclusion

Nuxt 4.0 is here and ready for production! After a year of real-world testing, this stability-focused major release introduces thoughtful breaking changes to improve development experience. While there are breaking changes to consider, the automated migration tools and well-documented upgrade path make the transition manageable.

Most projects should upgrade smoothly, and the performance improvements, better TypeScript support, and enhanced developer experience make the upgrade worthwhile. Nuxt 3 will continue to receive maintenance updates until the end of January 2026, so you have time to plan your migration.

Start by running `npx nuxt upgrade --dedupe`, use the automated codemods, and test thoroughly. The investment in upgrading will pay dividends in improved performance and developer productivity.

Remember to check the [official documentation](https://nuxt.com/docs/4.x/getting-started/upgrade) for the latest updates and changes.

Happy upgrading with Nuxt 4! 🚀

<p style="text-align: center;">fin.</p>