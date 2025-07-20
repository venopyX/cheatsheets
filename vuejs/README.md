# Complete Vue.js Cheatsheet Outline

## **Part I: Foundations**

### 1. **Introduction to Vue.js**
   - What is Vue.js
   - Progressive Framework Philosophy
   - Virtual DOM
   - Component-Based Architecture
   - Vue.js Ecosystem
   - Framework Comparisons (React, Angular, Svelte)
   - When to Choose Vue.js
   - Vue 2 vs Vue 3 Differences

### 2. **Development Environment Setup**
   - Node.js and Package Managers (npm, yarn, pnpm)
   - Code Editors and Extensions
     - VS Code with Volar
     - WebStorm
     - Vim/Neovim plugins
   - Browser DevTools
   - Vue DevTools Extension
   - Project Creation Methods
     - Vite + Vue (recommended)
     - Vue CLI
     - Nuxt.js
     - Manual Setup
   - Project Structure and Organization
   - Configuration Files
   - Environment Variables
   - Hot Module Replacement (HMR)

### 3. **Vue Application Instance**
   - `createApp()` API
   - Application Configuration
   - Multiple App Instances
   - App-level Properties and Methods
   - Global Components Registration
   - Global Directives Registration
   - Plugin Installation
   - Mounting and Unmounting
   - Error Handling

## **Part II: Template System**

### 4. **Template Syntax**
   - Text Interpolation (Mustache Syntax)
   - Raw HTML (`v-html`)
   - Attribute Binding (`v-bind`)
   - JavaScript Expressions
   - Template Comments
   - Template Literals
   - Template Compilation

### 5. **Directives**
   - **Built-in Directives**
     - `v-text`
     - `v-html` 
     - `v-show`
     - `v-if`, `v-else-if`, `v-else`
     - `v-for`
     - `v-on` (Event Handling)
     - `v-bind` (Attribute Binding)
     - `v-model` (Two-way Binding)
     - `v-slot`
     - `v-pre`
     - `v-once`
     - `v-memo`
     - `v-cloak`
   - **Directive Arguments**
     - Static Arguments
     - Dynamic Arguments
   - **Directive Modifiers**
     - Event Modifiers
     - Key Modifiers
     - System Modifier Keys
     - Mouse Button Modifiers
     - Form Input Modifiers
   - **Custom Directives**
     - Directive Hooks
     - Directive Arguments and Modifiers
     - Global vs Local Directives

## **Part III: Reactivity System**

### 6. **Vue 3 Reactivity Fundamentals**
   - Proxy-based Reactivity
   - `ref()` for Primitives
   - `reactive()` for Objects
   - `shallowRef()` and `shallowReactive()`
   - `readonly()` and `shallowReadonly()`
   - `toRef()` and `toRefs()`
   - `unref()` and `isRef()`
   - `markRaw()`
   - Reactivity Transform (Experimental)

### 7. **Computed Properties**
   - Basic Computed Properties
   - Computed with Getter and Setter
   - Computed Caching
   - Computed vs Methods
   - Computed vs Watchers
   - Debugging Computed Properties

### 8. **Watchers**
   - `watch()` API
   - `watchEffect()`
   - Watching Single Source
   - Watching Multiple Sources
   - Deep Watching
   - Immediate Execution
   - Callback Flush Timing
   - Stopping Watchers
   - Effect Cleanup
   - Watcher Debugging

## **Part IV: Component System**

### 9. **Component Basics**
   - Creating Components
   - Component Registration (Global vs Local)
   - Component Naming Conventions
   - Single File Components (SFC)
   - Component Instance Properties
   - Component Lifecycle

### 10. **Component Communication**
   - **Props**
     - Prop Types and Validation
     - Default Values
     - Required Props
     - Boolean Props
     - Object and Array Props
     - Prop Casing
   - **Events**
     - Emitting Events
     - Event Arguments
     - Event Validation
     - Native Event Binding
   - **Slots**
     - Basic Slots
     - Named Slots
     - Scoped Slots
     - Slot Props
     - Dynamic Slot Names
     - Conditional Slots
   - **Provide/Inject**
     - Basic Usage
     - Injection Keys
     - Reactivity with Provide/Inject
     - Working with Symbol Keys

### 11. **Advanced Component Features**
   - Dynamic Components (`component` tag)
   - Async Components
     - Basic Async Components  
     - Advanced Async Component Options
     - Loading and Error States
   - Component `v-model`
     - Basic `v-model`
     - Multiple `v-model` bindings
     - Custom `v-model` modifiers
   - Attributes Inheritance
     - `$attrs` object
     - Disabling Attribute Inheritance
     - Multiple Root Nodes

### 12. **Component Lifecycle**
   - **Options API Lifecycle**
     - `beforeCreate`
     - `created`
     - `beforeMount`
     - `mounted`
     - `beforeUpdate`
     - `updated`
     - `beforeUnmount`
     - `unmounted`
     - `errorCaptured`
     - `renderTracked`
     - `renderTriggered`
   - **Composition API Lifecycle**
     - `onBeforeMount`
     - `onMounted`
     - `onBeforeUpdate`
     - `onUpdated`
     - `onBeforeUnmount`
     - `onUnmounted`
     - `onErrorCaptured`
     - `onRenderTracked`
     - `onRenderTriggered`

## **Part V: Composition API**

### 13. **Composition API Fundamentals**
   - `setup()` Function
   - `setup()` vs Options API
   - Composition API Benefits
   - Logic Reuse and Organization

### 14. **Composition API Features**
   - Reactive References
   - Lifecycle Hooks
   - Template Refs
   - `defineProps()` and `defineEmits()`
   - `defineExpose()`
   - `useSlots()` and `useAttrs()`
   - `useCssVars()`

### 15. **Composables**
   - What are Composables
   - Creating Custom Composables
   - Composable Conventions
   - State Management with Composables
   - Popular Composable Libraries (VueUse)
   - Testing Composables

## **Part VI: Built-in Components**

### 16. **Special Elements**
   - `<component>`
   - `<template>`
   - `<slot>`

### 17. **Built-in Components**
   - `<Transition>`
     - CSS Transitions
     - CSS Animations
     - Custom Transition Classes
     - JavaScript Hooks
     - Transition Modes
   - `<TransitionGroup>`
     - List Transitions
     - Move Transitions
     - Staggered List Transitions
   - `<KeepAlive>`
     - Caching Components
     - Include/Exclude Props
     - Max Cached Instances
   - `<Teleport>`
     - Rendering to Different DOM Locations
     - Conditional Teleporting
     - Multiple Teleports
   - `<Suspense>`
     - Async Component Loading
     - Loading States
     - Error Boundaries

## **Part VII: Forms and User Input**

### 18. **Form Handling**
   - `v-model` Fundamentals
   - Form Input Bindings
     - Text Inputs
     - Multiline Text (textarea)
     - Checkboxes
     - Radio Buttons
     - Select Options
   - `v-model` Modifiers
     - `.lazy`
     - `.number`
     - `.trim`
   - Custom Input Components
   - Form Validation Strategies

### 19. **Event Handling**
   - Event Listeners (`v-on`)
   - Inline Handlers vs Method Handlers
   - Event Objects
   - Multiple Event Listeners
   - Event Modifiers
     - `.stop`
     - `.prevent`
     - `.capture`
     - `.self`
     - `.once`
     - `.passive`
   - Key Modifiers
   - Mouse Button Modifiers
   - Custom Events

## **Part VIII: Styling**

### 20. **Class and Style Bindings**
   - Dynamic Class Binding
     - Object Syntax
     - Array Syntax
     - Conditional Classes
   - Inline Style Binding
     - Object Syntax
     - Array Syntax
     - CSS Properties
   - CSS Variables
   - Scoped CSS

### 21. **CSS Features in SFC**
   - `<style>` Blocks
   - Scoped CSS (`scoped` attribute)
   - CSS Modules
   - CSS-in-JS
   - `v-bind()` in CSS
   - Global Styles
   - CSS Preprocessors (Sass, Less, Stylus)

## **Part IX: Routing**

### 22. **Vue Router Fundamentals**
   - Installation and Setup
   - Route Configuration
   - Router Instance
   - Route Matching
   - Dynamic Route Matching
   - Route Parameters
   - Query Parameters
   - Hash Mode vs History Mode

### 23. **Navigation**
   - Programmatic Navigation
   - `<router-link>`
   - `<router-view>`
   - Named Routes
   - Named Views
   - Route Props
   - Active Link Classes

### 24. **Advanced Routing**
   - Nested Routes
   - Route Guards
     - Global Guards
     - Per-Route Guards  
     - In-Component Guards
   - Route Meta Fields
   - Lazy Loading Routes
   - Navigation Guards Execution Order
   - Route Transitions
   - Scroll Behavior

## **Part X: State Management**

### 25. **Local State Management**
   - Component State
   - Props Down, Events Up
   - State Lifting
   - Provide/Inject for State

### 26. **Pinia (Recommended)**
   - Installation and Setup
   - Defining Stores
   - State
   - Getters
   - Actions
   - Store Composition
   - Plugins
   - DevTools Integration
   - Testing Stores

### 27. **Vuex (Legacy but Still Used)**
   - Core Concepts
   - State
   - Getters
   - Mutations
   - Actions
   - Modules
   - Namespacing
   - Helper Functions
   - DevTools Integration

## **Part XI: HTTP and API Integration**

### 28. **HTTP Clients**
   - Fetch API
   - Axios
   - Other HTTP Libraries
   - Request and Response Interceptors

### 29. **API Integration Patterns**
   - RESTful APIs
   - GraphQL with Vue
   - WebSocket Integration
   - Server-Sent Events (SSE)
   - Authentication and Authorization
   - Error Handling
   - Loading States
   - Caching Strategies

## **Part XII: Testing**

### 30. **Unit Testing**
   - Testing Philosophy
   - Vue Test Utils
   - Testing Components
   - Testing Props and Events
   - Testing Slots
   - Mocking Dependencies
   - Testing Composables
   - Coverage Reports

### 31. **Integration Testing**
   - Component Integration Tests
   - Testing with Vue Router
   - Testing with State Management
   - API Mocking

### 32. **End-to-End Testing**
   - Cypress
   - Playwright
   - Testing User Workflows
   - Visual Regression Testing

## **Part XIII: Performance Optimization**

### 33. **Bundle Optimization**
   - Code Splitting
   - Lazy Loading
   - Tree Shaking
   - Bundle Analysis
   - Dynamic Imports

### 34. **Runtime Performance**
   - Virtual Scrolling
   - `v-memo` Directive
   - `v-once` for Static Content
   - Avoiding Reactivity Overhead
   - Component Optimization
   - Memory Management

### 35. **Build Optimization**
   - Vite Configuration
   - Asset Optimization
   - Preloading Strategies
   - Service Workers
   - Progressive Web Apps (PWA)

## **Part XIV: Advanced Concepts**

### 36. **Custom Renderers**
   - Render Functions
   - JSX in Vue
   - Custom Render Targets
   - Virtual Node (VNode) API

### 37. **Plugins**
   - Creating Plugins
   - Plugin Installation
   - Popular Vue Plugins
   - Plugin Best Practices

### 38. **Mixins (Legacy)**
   - Global Mixins
   - Local Mixins
   - Mixin Merging Strategies
   - Why Composition API is Preferred

### 39. **TypeScript Integration**
   - Vue 3 + TypeScript Setup
   - Typing Components
   - Typing Props and Events
   - Typing Composables
   - Generic Components
   - TypeScript with Options API

## **Part XV: Server-Side Rendering**

### 40. **SSR Fundamentals**
   - What is SSR
   - Benefits and Trade-offs
   - Hydration
   - Universal Code Considerations

### 41. **Nuxt.js**
   - Nuxt 3 Features
   - File-based Routing
   - Auto-imports
   - Server API Routes
   - Deployment Options

### 42. **Vite SSR**
   - Vite SSR Setup
   - Build Configuration
   - Production Deployment

## **Part XVI: Mobile Development**

### 43. **Mobile Web Apps**
   - Responsive Design
   - Touch Events
   - Mobile Performance
   - PWA Features

### 44. **Native Mobile with Vue**
   - NativeScript Vue
   - Capacitor + Vue
   - Quasar Framework

## **Part XVII: Desktop Development**

### 45. **Desktop Applications**
   - Electron + Vue
   - Tauri + Vue
   - Building and Distribution

## **Part XVIII: Deployment and DevOps**

### 46. **Build and Deployment**
   - Production Builds
   - Environment Configuration
   - Static Site Generation (SSG)
   - Deployment Platforms
     - Netlify
     - Vercel
     - GitHub Pages
     - AWS
     - Docker Deployment

### 47. **CI/CD**
   - GitHub Actions
   - GitLab CI
   - Testing in CI/CD
   - Automated Deployment

## **Part XIX: Debugging and Development Tools**

### 48. **Debugging Techniques**
   - Vue DevTools
   - Browser DevTools
   - Debugging Reactivity
   - Performance Profiling
   - Error Tracking

### 49. **Development Tools**
   - ESLint Configuration
   - Prettier Setup
   - Husky and Lint-staged
   - VS Code Extensions
   - Development Workflow

## **Part XX: Best Practices and Patterns**

### 50. **Code Organization**
   - File and Folder Structure
   - Component Organization
   - Composable Organization
   - Naming Conventions

### 51. **Performance Best Practices**
   - Component Design Patterns
   - State Management Patterns
   - Memory Management
   - Bundle Size Optimization

### 52. **Security Considerations**
   - XSS Prevention
   - CSRF Protection
   - Content Security Policy
   - Authentication Best Practices

### 53. **Accessibility (a11y)**
   - Semantic HTML
   - ARIA Attributes
   - Keyboard Navigation
   - Screen Reader Support
   - Focus Management

## **Part XXI: Migration and Interoperability**

### 54. **Vue 2 to Vue 3 Migration**
   - Breaking Changes
   - Migration Build
   - Step-by-step Migration Guide
   - Common Migration Issues

### 55. **Integration with Other Frameworks**
   - Vue in Existing Applications
   - Micro-frontends with Vue
   - Web Components with Vue

## **Part XXII: Real-World Applications**

### 56. **Project Architecture**
   - Large-scale Application Structure
   - Monorepo vs Multi-repo
   - Component Libraries
   - Design Systems

### 57. **Case Studies**
   - E-commerce Application
   - Dashboard Application  
   - Real-time Chat Application
   - Content Management System

## **Part XXIII: Interview and Career**

### 58. **Interview Preparation**
   - Common Vue.js Questions
   - Coding Challenges
   - System Design Questions
   - Behavioral Questions

### 59. **Building Portfolio Projects**
   - Project Ideas
   - Showcasing Skills
   - GitHub Best Practices
   - Resume Tips

## **Part XXIV: Future and Ecosystem**

### 60. **Vue.js Roadmap**
   - Upcoming Features
   - Experimental Features
   - Community Contributions

### 61. **Ecosystem Libraries**
   - UI Component Libraries
     - Vuetify
     - Quasar
     - Element Plus
     - Ant Design Vue
     - PrimeVue
   - Utility Libraries
     - VueUse
     - Vue Demi
   - Animation Libraries
   - Form Libraries
   - Data Visualization Libraries

---

# Complete Vue.js Cheatsheet

## **Part I: Foundations**

### 1. **Introduction to Vue.js**

#### What is Vue.js
Vue.js is a progressive JavaScript framework for building user interfaces. It focuses on the view layer and is designed to be incrementally adoptable.

**Real-world usage:** Build interactive web applications, single-page applications (SPAs), and even integrate into existing projects gradually.

```javascript
// Basic Vue app
import { createApp } from 'vue'

const app = createApp({
  data() {
    return {
      message: 'Hello Vue!'
    }
  }
})

app.mount('#app')
```

#### Progressive Framework Philosophy
Vue can be adopted incrementally - start with just a few components and gradually expand to full SPA architecture.

**Real-world usage:** Add Vue to legacy jQuery projects, migrate React components one by one, or build new apps from scratch.

```html
<!-- Drop-in usage in existing HTML -->
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
<div id="app">{{ message }}</div>
<script>
  Vue.createApp({
    data: () => ({ message: 'Vue added to existing site!' })
  }).mount('#app')
</script>
```

#### Virtual DOM
Vue uses a virtual representation of the DOM in memory, comparing changes and updating only what's necessary for optimal performance.

**Real-world usage:** Enables smooth UI updates in data-heavy applications like dashboards, real-time feeds, and large lists.

```vue
<template>
  <!-- Vue efficiently updates only changed items -->
  <div v-for="item in items" :key="item.id">
    {{ item.name }} - {{ item.price }}
  </div>
</template>

<script setup>
import { ref } from 'vue'
const items = ref([
  { id: 1, name: 'Laptop', price: 999 },
  { id: 2, name: 'Phone', price: 599 }
])

// Only the changed item re-renders
setTimeout(() => {
  items.value[0].price = 899
}, 2000)
</script>
```

#### Component-Based Architecture
Applications are built as a tree of reusable, self-contained components that manage their own state and logic.

**Real-world usage:** Create reusable UI elements like buttons, modals, forms that can be used across different pages and projects.

```vue
<!-- ProductCard.vue -->
<template>
  <div class="card">
    <h3>{{ product.name }}</h3>
    <p>${{ product.price }}</p>
    <button @click="addToCart">Add to Cart</button>
  </div>
</template>

<script setup>
defineProps(['product'])
defineEmits(['add-to-cart'])

const addToCart = () => {
  emit('add-to-cart', product)
}
</script>

<!-- App.vue -->
<template>
  <ProductCard 
    v-for="product in products" 
    :key="product.id"
    :product="product"
    @add-to-cart="handleAddToCart"
  />
</template>
```

#### Vue.js Ecosystem
Rich ecosystem including routing, state management, build tools, and UI libraries.

**Real-world usage:** Complete application development with Vue Router (routing), Pinia (state), Vite (build tool), and UI frameworks.

```javascript
// Modern Vue 3 stack
import { createApp } from 'vue'
import { createRouter, createWebHistory } from 'vue-router'
import { createPinia } from 'pinia'
import App from './App.vue'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    { path: '/', component: Home },
    { path: '/about', component: About }
  ]
})

const pinia = createPinia()

createApp(App)
  .use(router)
  .use(pinia)
  .mount('#app')
```

#### Framework Comparisons

**Vue vs React:**
- Vue: Template-based, built-in state management, less boilerplate
- React: JSX, larger ecosystem, more job opportunities

**Vue vs Angular:**
- Vue: Gentler learning curve, smaller bundle size, more flexible
- Angular: Full framework, TypeScript-first, enterprise-focused

**Vue vs Svelte:**
- Vue: Mature ecosystem, better tooling, larger community
- Svelte: Compile-time optimization, smaller runtime, newer approach

**Real-world usage:** Choose Vue for rapid development, team productivity, and when you need a balance between simplicity and power.

#### When to Choose Vue.js

**Choose Vue when:**
- Building SPAs or progressive enhancement
- Team prefers template syntax over JSX
- Need gentle learning curve for new developers
- Want built-in solutions (routing, state management)
- Migrating from jQuery or other legacy frameworks

**Real-world scenarios:**
```vue
<!-- E-commerce product pages -->
<template>
  <div class="product-page">
    <ProductGallery :images="product.images" />
    <ProductInfo :product="product" />
    <ReviewSection :reviews="reviews" />
  </div>
</template>

<!-- Admin dashboards -->
<template>
  <div class="dashboard">
    <Sidebar :nav-items="navItems" />
    <DataTable :data="analytics" />
    <ChartContainer :chart-data="metrics" />
  </div>
</template>
```

#### Vue 2 vs Vue 3 Differences

**Key Vue 3 improvements:**
- Composition API for better logic reuse
- Multiple root elements in templates
- Better TypeScript support
- Smaller bundle size and better performance
- Teleport and Suspense built-in components

```vue
<!-- Vue 2 Options API -->
<template>
  <div>
    <p>{{ count }}</p>
    <button @click="increment">+</button>
  </div>
</template>

<script>
export default {
  data() {
    return { count: 0 }
  },
  methods: {
    increment() {
      this.count++
    }
  }
}
</script>

<!-- Vue 3 Composition API -->
<template>
  <p>{{ count }}</p>
  <button @click="increment">+</button>
</template>

<script setup>
import { ref } from 'vue'
const count = ref(0)
const increment = () => count.value++
</script>
```

**Real-world migration considerations:**
- Vue 2: Still widely used, stable, extensive plugin ecosystem
- Vue 3: Modern apps, better performance, future-proof development
- Migration: Use Vue 3 migration build for gradual transition

---

### 2. **Development Environment Setup**

#### Node.js and Package Managers
Node.js runtime environment with package managers for dependency management and project tooling.

**Real-world usage:** Essential foundation for Vue development, build processes, and dependency management in modern web development.

```bash
# Check Node.js version (16+ recommended)
node --version
npm --version

# Package manager options
npm install package-name
yarn add package-name
pnpm add package-name

# Global Vue CLI (legacy)
npm install -g @vue/cli
```

#### Code Editors and Extensions

**VS Code with Volar (Recommended)**
Primary Vue.js extension providing syntax highlighting, IntelliSense, and debugging support.

**Real-world usage:** Essential for productive Vue development with autocomplete, error detection, and component intelligence.

```json
// .vscode/extensions.json
{
  "recommendations": [
    "Vue.volar",
    "Vue.vscode-typescript-vue-plugin",
    "bradlc.vscode-tailwindcss",
    "esbenp.prettier-vscode",
    "dbaeumer.vscode-eslint"
  ]
}

// .vscode/settings.json
{
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode"
}
```

**WebStorm**
Full-featured IDE with built-in Vue support, debugging, and refactoring tools.

**Vim/Neovim plugins**
```vim
" Essential Vim plugins for Vue development
Plug 'neovim/nvim-lspconfig'
Plug 'johnsoncodehk/volar'
Plug 'posva/vim-vue'
Plug 'pangloss/vim-javascript'
```

#### Browser DevTools
Essential debugging tools built into modern browsers for inspecting and debugging Vue applications.

**Real-world usage:** Debug component state, inspect DOM changes, monitor network requests, and profile performance.

```javascript
// Access Vue instance in browser console
$vm0 // Selected component instance
$vm0.$data // Component data
$vm0.$props // Component props
$vm0.$emit('event-name') // Trigger events
```

#### Vue DevTools Extension
Browser extension providing Vue-specific debugging capabilities with component tree inspection and state management.

**Real-world usage:** Debug component hierarchy, inspect reactive state, time-travel debugging, and performance profiling.

```vue
<!-- Component will show in Vue DevTools -->
<template>
  <div class="user-profile">
    <h2>{{ user.name }}</h2>
    <p>{{ user.email }}</p>
  </div>
</template>

<script setup>
import { ref } from 'vue'
// This reactive state is visible in DevTools
const user = ref({
  name: 'John Doe',
  email: 'john@example.com'
})
</script>
```

#### Project Creation Methods

**Vite + Vue (Recommended)**
Modern, fast build tool optimized for Vue development with hot module replacement and optimized production builds.

**Real-world usage:** Fastest development experience for modern Vue applications with instant server start and lightning-fast HMR.

```bash
# Create new Vue 3 project with Vite
npm create vue@latest my-web-app

# Interactive project setup
✔ Project name: my-web-app
✔ Add TypeScript? Yes
✔ Add JSX Support? No  
✔ Add Vue Router for Single Page Application development? Yes
✔ Add Pinia for state management? Yes
✔ Add Vitest for Unit Testing? No
✔ Add an End-to-End Testing Solution? No
✔ Add ESLint for code quality? Yes
✔ Add Prettier for code formatting? Yes

# Navigate and install
cd my-web-app
npm install
npm run format
npm run dev
```

**Vue CLI (Legacy)**
Traditional scaffolding tool with webpack-based build system.

```bash
# Global installation
npm install -g @vue/cli

# Create project
vue create my-project
vue add router
vue add vuex
```

**Nuxt.js**
Full-stack Vue framework with server-side rendering, file-based routing, and auto-imports.

```bash
# Create Nuxt 3 project
npx nuxi@latest init my-nuxt-app
cd my-nuxt-app
npm install
npm run dev
```

**Manual Setup**
Custom configuration for specific requirements or learning purposes.

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
  <title>Vue App</title>
</head>
<body>
  <div id="app"></div>
  <script type="module" src="/src/main.js"></script>
</body>
</html>
```

```javascript
// src/main.js
import { createApp } from 'vue'
import App from './App.vue'

createApp(App).mount('#app')
```

#### Project Structure and Organization
Standard Vue project architecture for maintainable and scalable applications.

**Real-world usage:** Organized structure improves team collaboration, code maintainability, and feature development.

```
my-web-app/
├── public/
│   └── favicon.ico
├── src/
│   ├── assets/           # Static assets (images, fonts, styles)
│   │   ├── base.css
│   │   ├── logo.svg
│   │   └── main.css
│   ├── components/       # Reusable components
│   │   ├── ui/          # Generic UI components
│   │   │   ├── Button.vue
│   │   │   └── Modal.vue
│   │   └── HelloWorld.vue
│   ├── composables/      # Reusable composition functions
│   │   └── useCounter.js
│   ├── router/          # Vue Router configuration
│   │   └── index.ts
│   ├── stores/          # Pinia state management
│   │   └── counter.ts
│   ├── views/           # Page-level components
│   │   ├── HomeView.vue
│   │   └── AboutView.vue
│   ├── utils/           # Utility functions
│   │   └── api.js
│   ├── App.vue          # Root component
│   └── main.ts          # Application entry point
├── index.html           # HTML template
├── package.json         # Dependencies and scripts
└── vite.config.ts       # Build configuration
```

#### Configuration Files
Essential configuration files for development environment setup and build optimization.

**Real-world usage:** Customize build process, configure development server, set up linting rules, and manage dependencies.

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
      '@components': resolve(__dirname, 'src/components'),
      '@views': resolve(__dirname, 'src/views')
    }
  },
  server: {
    port: 3000,
    open: true
  }
})
```

```json
// package.json scripts
{
  "scripts": {
    "dev": "vite",
    "build": "vue-tsc && vite build", 
    "preview": "vite preview",
    "format": "prettier --write src/",
    "lint": "eslint . --fix"
  }
}
```

```typescript
// eslint.config.ts
export default {
  extends: [
    '@vue/eslint-config-typescript',
    '@vue/eslint-config-prettier'
  ],
  rules: {
    'vue/multi-word-component-names': 'off',
    '@typescript-eslint/no-unused-vars': 'warn'
  }
}
```

#### Environment Variables
Configuration values for different deployment environments (development, staging, production).

**Real-world usage:** Manage API endpoints, feature flags, and sensitive configuration across different environments.

```bash
# .env (default for all environments)
VITE_APP_TITLE=My Vue App
VITE_API_BASE_URL=https://api.example.com

# .env.local (local development, git-ignored)
VITE_API_BASE_URL=http://localhost:3001
VITE_DEBUG_MODE=true

# .env.production (production builds)
VITE_API_BASE_URL=https://prod-api.example.com
VITE_ANALYTICS_ID=GA-XXXXXXXXX
```

```typescript
// Using environment variables
const config = {
  apiBaseUrl: import.meta.env.VITE_API_BASE_URL,
  appTitle: import.meta.env.VITE_APP_TITLE,
  isDev: import.meta.env.DEV,
  isProd: import.meta.env.PROD
}

// In components
<template>
  <h1>{{ appTitle }}</h1>
</template>

<script setup>
const appTitle = import.meta.env.VITE_APP_TITLE
</script>
```

#### Hot Module Replacement (HMR)
Development feature that updates modules in the browser without full page reload, preserving application state.

**Real-world usage:** Dramatically speeds up development by maintaining component state while updating code, especially useful for forms and complex interactions.

```vue
<!-- Changes to this component update instantly -->
<template>
  <div class="counter">
    <p>Count: {{ count }}</p>
    <button @click="increment">+</button>
    <button @click="decrement">-</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// State is preserved during HMR updates
const count = ref(0)

const increment = () => count.value++
const decrement = () => count.value--

// HMR API for advanced cases
if (import.meta.hot) {
  import.meta.hot.accept()
  import.meta.hot.dispose(() => {
    console.log('Component updated via HMR')
  })
}
</script>
```

```javascript
// vite.config.ts HMR configuration
export default defineConfig({
  server: {
    hmr: {
      overlay: true, // Show error overlay
      port: 24678    // Custom HMR port
    }
  }
})
```

---

### 3. **Vue Application Instance**

#### `createApp()` API
Creates a new Vue application instance that serves as the root of your Vue application with its own scope and configuration.

**Real-world usage:** Initialize Vue applications, configure global settings, and mount components to DOM elements.

```javascript
// Basic app creation
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)
app.mount('#app')

// With root component options
const app = createApp({
  data() {
    return {
      message: 'Hello Vue!'
    }
  },
  template: '<h1>{{ message }}</h1>'
})

// Multiple configurations before mounting
const app = createApp(App)
  .use(router)
  .use(store)
  .mount('#app')
```

#### Application Configuration
Global configuration options that affect the entire Vue application instance.

**Real-world usage:** Set up error handling, performance settings, and global behaviors for production applications.

```javascript
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)

// Global configuration
app.config.globalProperties.$http = axios
app.config.globalProperties.$filters = {
  currency: (value) => `$${value.toFixed(2)}`
}

// Performance and debugging
app.config.performance = true
app.config.devtools = true

// Error handling
app.config.errorHandler = (err, instance, info) => {
  console.error('Global error:', err)
  console.log('Component instance:', instance)
  console.log('Error info:', info)
}

// Warning handler
app.config.warnHandler = (msg, instance, trace) => {
  console.warn(`Warning: ${msg}`)
}

app.mount('#app')
```

#### Multiple App Instances
Create separate Vue applications with isolated configurations and state.

**Real-world usage:** Integrate Vue into existing applications, create micro-frontends, or separate admin panels from main applications.

```javascript
// Main application
const mainApp = createApp({
  data() {
    return { title: 'Main App' }
  },
  template: '<h1>{{ title }}</h1>'
})
mainApp.mount('#main-app')

// Admin panel with different configuration
const adminApp = createApp({
  data() {
    return { adminTitle: 'Admin Dashboard' }
  },
  template: '<h2>{{ adminTitle }}</h2>'
})
adminApp.use(adminRouter)
adminApp.use(adminStore)
adminApp.mount('#admin-panel')

// Widget for existing site
const widgetApp = createApp({
  data() {
    return { 
      products: [],
      loading: false 
    }
  },
  template: `
    <div v-if="loading">Loading...</div>
    <div v-else>
      <div v-for="product in products" :key="product.id">
        {{ product.name }}
      </div>
    </div>
  `
})
widgetApp.mount('#product-widget')
```

#### App-level Properties and Methods
Global properties and methods available to all components within the application instance.

**Real-world usage:** Share common utilities, API clients, and helper functions across all components without imports.

```javascript
const app = createApp(App)

// Global properties accessible in all components
app.config.globalProperties.$api = {
  get: (url) => fetch(url).then(r => r.json()),
  post: (url, data) => fetch(url, {
    method: 'POST',
    body: JSON.stringify(data)
  })
}

app.config.globalProperties.$utils = {
  formatDate: (date) => new Date(date).toLocaleDateString(),
  truncate: (text, length) => text.length > length 
    ? text.substring(0, length) + '...' 
    : text
}

app.config.globalProperties.$constants = {
  API_BASE_URL: 'https://api.example.com',
  MAX_FILE_SIZE: 5 * 1024 * 1024 // 5MB
}

app.mount('#app')
```

```vue
<!-- Using global properties in components -->
<template>
  <div>
    <p>{{ $utils.formatDate(user.createdAt) }}</p>
    <p>{{ $utils.truncate(user.bio, 50) }}</p>
    <button @click="loadUserData">Load Data</button>
  </div>
</template>

<script setup>
import { ref, getCurrentInstance } from 'vue'

const { proxy } = getCurrentInstance()
const user = ref({})

const loadUserData = async () => {
  const userData = await proxy.$api.get(`${proxy.$constants.API_BASE_URL}/user/1`)
  user.value = userData
}
</script>
```

#### Global Components Registration
Register components globally to use them anywhere in the application without importing.

**Real-world usage:** Common UI components like buttons, modals, and icons that are used frequently across the application.

```javascript
import { createApp } from 'vue'
import App from './App.vue'

// Import common components
import BaseButton from './components/BaseButton.vue'
import BaseModal from './components/BaseModal.vue'
import BaseIcon from './components/BaseIcon.vue'
import LoadingSpinner from './components/LoadingSpinner.vue'

const app = createApp(App)

// Register components globally
app.component('BaseButton', BaseButton)
app.component('BaseModal', BaseModal)
app.component('BaseIcon', BaseIcon)
app.component('LoadingSpinner', LoadingSpinner)

// Bulk registration
const globalComponents = {
  BaseButton,
  BaseModal,
  BaseIcon,
  LoadingSpinner
}

Object.entries(globalComponents).forEach(([name, component]) => {
  app.component(name, component)
})

app.mount('#app')
```

```vue
<!-- Use global components without importing -->
<template>
  <div>
    <BaseButton @click="showModal = true">
      <BaseIcon name="plus" />
      Add Item
    </BaseButton>
    
    <BaseModal v-model="showModal" title="Add New Item">
      <form @submit.prevent="handleSubmit">
        <input v-model="itemName" placeholder="Item name" />
        <BaseButton type="submit" :disabled="loading">
          <LoadingSpinner v-if="loading" />
          {{ loading ? 'Adding...' : 'Add Item' }}
        </BaseButton>
      </form>
    </BaseModal>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const showModal = ref(false)
const itemName = ref('')
const loading = ref(false)

const handleSubmit = async () => {
  loading.value = true
  // Submit logic
  loading.value = false
  showModal.value = false
}
</script>
```

#### Global Directives Registration
Register custom directives globally for reusable DOM manipulation across the application.

**Real-world usage:** Common behaviors like focus management, click-outside detection, and element animations.

```javascript
const app = createApp(App)

// Focus directive
app.directive('focus', {
  mounted(el) {
    el.focus()
  }
})

// Click outside directive
app.directive('click-outside', {
  beforeMount(el, binding) {
    el.clickOutsideEvent = (event) => {
      if (!(el === event.target || el.contains(event.target))) {
        binding.value(event)
      }
    }
    document.addEventListener('click', el.clickOutsideEvent)
  },
  unmounted(el) {
    document.removeEventListener('click', el.clickOutsideEvent)
  }
})

// Tooltip directive
app.directive('tooltip', {
  beforeMount(el, binding) {
    el.setAttribute('title', binding.value)
    el.style.position = 'relative'
  },
  updated(el, binding) {
    el.setAttribute('title', binding.value)
  }
})

// Permission directive
app.directive('permission', {
  beforeMount(el, binding) {
    const { value } = binding
    const userPermissions = getCurrentUser().permissions
    
    if (!userPermissions.includes(value)) {
      el.style.display = 'none'
    }
  }
})

app.mount('#app')
```

```vue
<!-- Using global directives -->
<template>
  <div>
    <!-- Auto-focus input -->
    <input v-focus v-model="searchQuery" placeholder="Search..." />
    
    <!-- Dropdown with click-outside -->
    <div class="dropdown" v-click-outside="closeDropdown">
      <button @click="showDropdown = !showDropdown">Options</button>
      <ul v-show="showDropdown">
        <li>Option 1</li>
        <li>Option 2</li>
      </ul>
    </div>
    
    <!-- Tooltip on hover -->
    <button v-tooltip="'This button saves your changes'">
      Save
    </button>
    
    <!-- Permission-based visibility -->
    <button v-permission="'admin'" @click="deleteUser">
      Delete User
    </button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const searchQuery = ref('')
const showDropdown = ref(false)

const closeDropdown = () => {
  showDropdown.value = false
}
</script>
```

#### Plugin Installation
Install and configure plugins that extend Vue's functionality at the application level.

**Real-world usage:** Add routing, state management, UI libraries, and third-party integrations to Vue applications.

```javascript
import { createApp } from 'vue'
import { createRouter, createWebHistory } from 'vue-router'
import { createPinia } from 'pinia'
import App from './App.vue'

// Router plugin
const router = createRouter({
  history: createWebHistory(),
  routes: [
    { path: '/', component: () => import('./views/Home.vue') },
    { path: '/about', component: () => import('./views/About.vue') }
  ]
})

// State management plugin
const pinia = createPinia()

// Custom plugin
const myPlugin = {
  install(app, options) {
    app.config.globalProperties.$translate = (key) => {
      return options.messages[key] || key
    }
    
    app.provide('theme', options.theme)
  }
}

// UI library plugin
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'

const app = createApp(App)

// Install plugins
app.use(router)
app.use(pinia)
app.use(ElementPlus)
app.use(myPlugin, {
  messages: {
    hello: 'Hello World',
    goodbye: 'Goodbye'
  },
  theme: 'dark'
})

app.mount('#app')
```

```javascript
// Creating a custom plugin
// plugins/api.js
export default {
  install(app, options) {
    const api = {
      baseURL: options.baseURL,
      async request(method, url, data) {
        const response = await fetch(`${this.baseURL}${url}`, {
          method,
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${options.token}`
          },
          body: data ? JSON.stringify(data) : undefined
        })
        return response.json()
      }
    }
    
    app.config.globalProperties.$api = api
    app.provide('api', api)
  }
}
```

#### Mounting and Unmounting
Control when and where Vue applications are attached to and removed from the DOM.

**Real-world usage:** Dynamic application loading, SPA route transitions, and cleanup for memory management.

```javascript
import { createApp } from 'vue'
import App from './App.vue'

// Basic mounting
const app = createApp(App)
app.mount('#app')

// Conditional mounting
const mountApp = () => {
  const app = createApp(App)
  return app.mount('#app')
}

// Unmounting for cleanup
let appInstance = null

const initApp = () => {
  if (appInstance) {
    appInstance.unmount()
  }
  
  const app = createApp(App)
  appInstance = app.mount('#app')
  
  return appInstance
}

// Multiple mount points
const createWidget = (selector, props) => {
  const app = createApp({
    ...App,
    props: Object.keys(props),
    setup() {
      return props
    }
  })
  
  return app.mount(selector)
}

// Dynamic mounting with error handling
const safeMountApp = async () => {
  try {
    const app = createApp(App)
    
    // Setup error boundary
    app.config.errorHandler = (err) => {
      console.error('App error:', err)
      // Unmount and show error UI
      app.unmount()
      document.getElementById('app').innerHTML = '<div>Application Error</div>'
    }
    
    app.mount('#app')
  } catch (error) {
    console.error('Failed to mount app:', error)
  }
}
```

```vue
<!-- Component-level mounting/unmounting -->
<template>
  <div>
    <button @click="toggleWidget">
      {{ widgetMounted ? 'Hide' : 'Show' }} Widget
    </button>
    <div ref="widgetContainer"></div>
  </div>
</template>

<script setup>
import { ref, nextTick } from 'vue'
import { createApp } from 'vue'
import WidgetComponent from './Widget.vue'

const widgetContainer = ref(null)
const widgetMounted = ref(false)
let widgetApp = null

const toggleWidget = async () => {
  if (widgetMounted.value) {
    // Unmount widget
    widgetApp.unmount()
    widgetApp = null
    widgetMounted.value = false
  } else {
    // Mount widget
    await nextTick()
    widgetApp = createApp(WidgetComponent)
    widgetApp.mount(widgetContainer.value)
    widgetMounted.value = true
  }
}
</script>
```

#### Error Handling
Configure global error handling for unhandled exceptions and component errors in Vue applications.

**Real-world usage:** Capture and log errors in production, show user-friendly error messages, and prevent application crashes.

```javascript
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)

// Global error handler
app.config.errorHandler = (err, instance, info) => {
  // Log error details
  console.error('Vue Error:', {
    error: err,
    component: instance?.$options.name || 'Unknown',
    errorInfo: info,
    timestamp: new Date().toISOString()
  })
  
  // Send to error reporting service
  if (import.meta.env.PROD) {
    errorReportingService.captureException(err, {
      tags: {
        vue_error_info: info,
        component: instance?.$options.name
      }
    })
  }
  
  // Show user notification
  showErrorNotification('An unexpected error occurred')
}

// Unhandled promise rejections
window.addEventListener('unhandledrejection', (event) => {
  console.error('Unhandled promise rejection:', event.reason)
  event.preventDefault()
})

app.mount('#app')
```

```vue
<!-- Error boundary component -->
<template>
  <div>
    <div v-if="error" class="error-boundary">
      <h2>Something went wrong</h2>
      <p>{{ error.message }}</p>
      <button @click="retry">Try Again</button>
    </div>
    <slot v-else />
  </div>
</template>

<script setup>
import { ref, onErrorCaptured } from 'vue'

const error = ref(null)

// Capture errors from child components
onErrorCaptured((err, instance, info) => {
  error.value = err
  console.error('Error captured:', err, info)
  
  // Return false to prevent error from propagating
  return false
})

const retry = () => {
  error.value = null
  // Force re-render of child components
}
</script>

<!-- Using error boundary -->
<template>
  <ErrorBoundary>
    <SomeComponent />
    <AnotherComponent />
  </ErrorBoundary>
</template>
```

```javascript
// Advanced error handling setup
const setupErrorHandling = (app) => {
  // Development vs Production error handling
  if (import.meta.env.DEV) {
    app.config.errorHandler = (err, instance, info) => {
      console.group('🚨 Vue Error')
      console.error('Error:', err)
      console.log('Component:', instance)
      console.log('Error Info:', info)
      console.log('Stack Trace:', err.stack)
      console.groupEnd()
    }
  } else {
    app.config.errorHandler = (err, instance, info) => {
      // Production error handling
      const errorDetails = {
        message: err.message,
        stack: err.stack,
        component: instance?.$options.__file || 'Unknown',
        errorInfo: info,
        url: window.location.href,
        userAgent: navigator.userAgent,
        timestamp: Date.now()
      }
      
      // Send to monitoring service
      fetch('/api/errors', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(errorDetails)
      })
    }
  }
}

const app = createApp(App)
setupErrorHandling(app)
app.mount('#app')
```

---

## **Part II: Template System**

### 4. **Template Syntax**

#### Text Interpolation (Mustache Syntax)
Double curly braces syntax for displaying reactive data and computed values in templates.

**Real-world usage:** Display user names, product prices, status messages, and any dynamic content that needs to update when data changes.

```vue
<template>
  <div>
    <!-- Basic interpolation -->
    <h1>{{ title }}</h1>
    <p>Hello, {{ user.name }}!</p>
    
    <!-- Numbers and calculations -->
    <p>Price: ${{ product.price }}</p>
    <p>Total: ${{ product.price * quantity }}</p>
    
    <!-- Boolean values -->
    <p>Status: {{ isOnline ? 'Online' : 'Offline' }}</p>
    
    <!-- Arrays and objects -->
    <p>Items count: {{ items.length }}</p>
    <p>First item: {{ items[0]?.name }}</p>
    
    <!-- Method calls -->
    <p>Formatted date: {{ formatDate(createdAt) }}</p>
    
    <!-- Computed properties -->
    <p>{{ fullName }}</p>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const title = ref('My Vue App')
const user = ref({ name: 'John Doe', email: 'john@example.com' })
const product = ref({ name: 'Laptop', price: 999.99 })
const quantity = ref(2)
const isOnline = ref(true)
const items = ref([
  { name: 'Item 1' },
  { name: 'Item 2' }
])
const createdAt = ref(new Date())

const fullName = computed(() => `${user.value.name} (${user.value.email})`)

const formatDate = (date) => {
  return new Intl.DateTimeFormat('en-US').format(date)
}
</script>
```

#### Raw HTML (`v-html`)
Renders HTML content directly into the DOM, bypassing Vue's template compilation.

**Real-world usage:** Display rich text content from CMS, render formatted blog posts, show HTML emails, or display user-generated content (with proper sanitization).

```vue
<template>
  <div>
    <!-- Basic HTML rendering -->
    <div v-html="htmlContent"></div>
    
    <!-- Blog post content -->
    <article v-html="blogPost.content"></article>
    
    <!-- Rich text editor output -->
    <div class="editor-output" v-html="editorContent"></div>
    
    <!-- Conditional HTML -->
    <div v-html="showFormatted ? formattedText : plainText"></div>
    
    <!-- HTML from API response -->
    <div v-html="emailTemplate"></div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import DOMPurify from 'dompurify' // For security

const htmlContent = ref('<p><strong>Bold text</strong> and <em>italic text</em></p>')

const blogPost = ref({
  title: 'My Blog Post',
  content: `
    <h2>Introduction</h2>
    <p>This is a <strong>rich text</strong> blog post with:</p>
    <ul>
      <li>Lists</li>
      <li><a href="#section1">Links</a></li>
      <li><code>Code snippets</code></li>
    </ul>
  `
})

const editorContent = ref('<p>Content from WYSIWYG editor</p>')
const showFormatted = ref(true)
const plainText = ref('Plain text content')
const formattedText = ref('<span style="color: blue;">Formatted content</span>')

// Sanitized HTML for security
const emailTemplate = computed(() => {
  const unsafeHtml = '<script>alert("XSS")</script><p>Safe content</p>'
  return DOMPurify.sanitize(unsafeHtml)
})
</script>

<style scoped>
.editor-output {
  border: 1px solid #ddd;
  padding: 1rem;
  border-radius: 4px;
}
</style>
```

#### Attribute Binding (`v-bind`)
Dynamically bind HTML attributes to reactive data, enabling dynamic styling, URLs, and element properties.

**Real-world usage:** Dynamic CSS classes, image sources, form field attributes, accessibility properties, and conditional element states.

```vue
<template>
  <div>
    <!-- Basic attribute binding -->
    <img :src="imageUrl" :alt="imageAlt" />
    
    <!-- Class binding -->
    <div :class="containerClass">Content</div>
    <button :class="{ active: isActive, disabled: isDisabled }">
      Button
    </button>
    
    <!-- Style binding -->
    <div :style="{ color: textColor, fontSize: fontSize + 'px' }">
      Styled text
    </div>
    
    <!-- Multiple attributes -->
    <input 
      :type="inputType"
      :value="inputValue"
      :placeholder="inputPlaceholder"
      :disabled="isFormDisabled"
      :required="isRequired"
    />
    
    <!-- Dynamic link -->
    <a :href="profileUrl" :target="linkTarget">View Profile</a>
    
    <!-- Conditional attributes -->
    <button 
      :aria-expanded="isMenuOpen"
      :aria-label="buttonLabel"
      :data-testid="testId"
    >
      Menu
    </button>
    
    <!-- Binding objects -->
    <div v-bind="dynamicAttrs">Dynamic element</div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const imageUrl = ref('/images/profile.jpg')
const imageAlt = ref('User profile picture')

const isActive = ref(true)
const isDisabled = ref(false)
const containerClass = ref('container large')

const textColor = ref('#3498db')
const fontSize = ref(16)

const inputType = ref('email')
const inputValue = ref('')
const inputPlaceholder = ref('Enter your email')
const isFormDisabled = ref(false)
const isRequired = ref(true)

const userId = ref(123)
const profileUrl = computed(() => `/profile/${userId.value}`)
const linkTarget = ref('_blank')

const isMenuOpen = ref(false)
const buttonLabel = computed(() => isMenuOpen.value ? 'Close menu' : 'Open menu')
const testId = ref('main-menu-button')

// Dynamic attributes object
const dynamicAttrs = ref({
  id: 'dynamic-element',
  'data-version': '1.0',
  class: 'dynamic-class',
  style: 'background-color: #f0f0f0;'
})
</script>
```

#### JavaScript Expressions
Use JavaScript expressions within template interpolations and directives for dynamic calculations and transformations.

**Real-world usage:** Data formatting, conditional logic, mathematical calculations, string manipulation, and array/object operations directly in templates.

```vue
<template>
  <div>
    <!-- Mathematical expressions -->
    <p>Total: ${{ price * quantity }}</p>
    <p>Tax: ${{ (price * quantity * 0.08).toFixed(2) }}</p>
    <p>Discount: {{ discount > 0 ? `${discount}% off` : 'No discount' }}</p>
    
    <!-- String manipulation -->
    <p>{{ message.toUpperCase() }}</p>
    <p>{{ user.name.split(' ')[0] }}</p>
    <p>{{ description.length > 50 ? description.slice(0, 50) + '...' : description }}</p>
    
    <!-- Array operations -->
    <p>Items: {{ items.length }}</p>
    <p>First item: {{ items[0]?.name || 'No items' }}</p>
    <p>Categories: {{ items.map(item => item.category).join(', ') }}</p>
    
    <!-- Object access -->
    <p>{{ user.address?.city || 'City not provided' }}</p>
    <p>{{ Object.keys(settings).length }} settings</p>
    
    <!-- Date formatting -->
    <p>{{ new Date(createdAt).toLocaleDateString() }}</p>
    <p>{{ new Date().getFullYear() }}</p>
    
    <!-- Conditional expressions -->
    <span :class="{ 'text-green': score >= 80, 'text-red': score < 60 }">
      Score: {{ score }}
    </span>
    
    <!-- Complex expressions -->
    <div :style="{ 
      transform: `rotate(${rotation}deg)`,
      opacity: isVisible ? 1 : 0.5,
      backgroundColor: `hsl(${hue}, 70%, 50%)`
    }">
      Styled element
    </div>
    
    <!-- Function calls -->
    <p>{{ formatCurrency(amount, 'USD') }}</p>
    <p>{{ calculateAge(birthDate) }} years old</p>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const price = ref(29.99)
const quantity = ref(3)
const discount = ref(15)

const message = ref('Hello World')
const user = ref({
  name: 'John Doe Smith',
  address: { city: 'New York', country: 'USA' }
})
const description = ref('This is a very long description that needs to be truncated when it exceeds the character limit')

const items = ref([
  { name: 'Laptop', category: 'Electronics' },
  { name: 'Book', category: 'Education' },
  { name: 'Shirt', category: 'Clothing' }
])

const settings = ref({
  theme: 'dark',
  notifications: true,
  autoSave: false
})

const createdAt = ref('2023-07-20T10:30:00Z')
const score = ref(85)
const rotation = ref(45)
const isVisible = ref(true)
const hue = ref(200)
const amount = ref(1234.56)
const birthDate = ref('1990-05-15')

// Helper functions
const formatCurrency = (value, currency) => {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: currency
  }).format(value)
}

const calculateAge = (birthDate) => {
  const today = new Date()
  const birth = new Date(birthDate)
  let age = today.getFullYear() - birth.getFullYear()
  const monthDiff = today.getMonth() - birth.getMonth()
  
  if (monthDiff < 0 || (monthDiff === 0 && today.getDate() < birth.getDate())) {
    age--
  }
  
  return age
}
</script>
```

#### Template Comments
HTML and Vue-specific comments for documentation and conditional compilation.

**Real-world usage:** Document complex template logic, leave TODO notes for team members, temporarily disable code sections, and provide context for future maintenance.

```vue
<template>
  <div>
    <!-- Standard HTML comments -->
    <!-- This section displays user information -->
    <div class="user-section">
      <h2>{{ user.name }}</h2>
      <!-- TODO: Add user avatar -->
      <p>{{ user.email }}</p>
    </div>
    
    <!-- Multi-line comments for complex sections -->
    <!--
      Product listing component
      - Displays filterable list of products
      - Includes pagination
      - Supports sorting by price/name
    -->
    <div class="product-list">
      <ProductCard 
        v-for="product in products" 
        :key="product.id"
        :product="product"
      />
    </div>
    
    <!-- Conditional comments for debugging -->
    <!-- DEBUG: Remove this section before production -->
    <div v-if="isDevelopment" class="debug-info">
      <pre>{{ JSON.stringify(debugData, null, 2) }}</pre>
    </div>
    
    <!-- Comments for disabled features -->
    <!--
    <FeatureComponent 
      v-if="featureFlag.enableNewFeature"
      :config="featureConfig"
    />
    Feature temporarily disabled - see ticket #123
    -->
    
    <!-- Documentation for complex directives -->
    <!-- 
      v-permission directive checks user permissions
      Available permissions: 'read', 'write', 'admin'
    -->
    <button v-permission="'admin'" @click="deleteUser">
      Delete User
    </button>
    
    <!-- Comments for maintenance notes -->
    <!-- 
      MAINTENANCE NOTE: 
      This component will be refactored in v2.0
      Contact @john.doe for migration questions
    -->
    <LegacyComponent :data="legacyData" />
  </div>
</template>

<script setup>
import { ref } from 'vue'

const user = ref({
  name: 'John Doe',
  email: 'john@example.com'
})

const products = ref([
  { id: 1, name: 'Product 1', price: 99 },
  { id: 2, name: 'Product 2', price: 149 }
])

const isDevelopment = ref(import.meta.env.DEV)
const debugData = ref({
  timestamp: new Date().toISOString(),
  userAgent: navigator.userAgent,
  viewport: {
    width: window.innerWidth,
    height: window.innerHeight
  }
})

const legacyData = ref({ legacy: true })
</script>

<!-- Style comments -->
<style scoped>
/* Main container styling */
.user-section {
  padding: 1rem;
  border: 1px solid #ddd;
}

/* 
  Product list grid layout
  TODO: Make responsive for mobile devices
*/
.product-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 1rem;
}

/* Development-only styles */
.debug-info {
  background: #f0f0f0;
  padding: 1rem;
  margin: 1rem 0;
  border-radius: 4px;
}
</style>
```

#### Template Literals
ES6 template literal syntax for dynamic string construction within Vue templates.

**Real-world usage:** Build dynamic URLs, create formatted messages, construct CSS custom properties, and generate complex string content.

```vue
<template>
  <div>
    <!-- Dynamic URLs -->
    <a :href="`/user/${user.id}/profile`">View Profile</a>
    <img :src="`${baseUrl}/images/${user.avatar}`" :alt="`${user.name} avatar`" />
    
    <!-- Dynamic CSS custom properties -->
    <div :style="`--primary-color: ${themeColor}; --font-size: ${fontSize}px`">
      Themed content
    </div>
    
    <!-- Complex class names -->
    <div :class="`card card--${cardType} ${isActive ? 'card--active' : ''}`">
      Card content
    </div>
    
    <!-- Dynamic messages -->
    <p>{{ `Welcome back, ${user.name}! You have ${notifications.length} new notifications.` }}</p>
    
    <!-- API endpoints -->
    <button @click="fetchData(`/api/users/${user.id}/posts?page=${currentPage}`)">
      Load Posts
    </button>
    
    <!-- Formatted text with variables -->
    <span :title="`Last updated: ${formatDate(lastUpdated)} by ${updatedBy}`">
      {{ `Version ${version}.${build}` }}
    </span>
    
    <!-- Dynamic attributes -->
    <input 
      :id="`input-${fieldId}`"
      :name="`user_${fieldName}`"
      :placeholder="`Enter your ${fieldLabel.toLowerCase()}`"
    />
    
    <!-- CSS Grid template areas -->
    <div :style="`grid-template-areas: 
      '${headerArea} ${headerArea}'
      '${sidebarArea} ${contentArea}'
      '${footerArea} ${footerArea}'`"
    >
      Grid layout
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const user = ref({
  id: 123,
  name: 'Alice Johnson',
  avatar: 'alice.jpg'
})

const baseUrl = ref('https://cdn.example.com')
const themeColor = ref('#3498db')
const fontSize = ref(16)

const cardType = ref('primary')
const isActive = ref(true)

const notifications = ref([
  { id: 1, message: 'New message' },
  { id: 2, message: 'Update available' }
])

const currentPage = ref(1)
const lastUpdated = ref(new Date())
const updatedBy = ref('John Doe')
const version = ref(2)
const build = ref(15)

const fieldId = ref('email')
const fieldName = ref('email')
const fieldLabel = ref('Email Address')

// Grid areas
const headerArea = ref('header')
const sidebarArea = ref('sidebar')
const contentArea = ref('content')
const footerArea = ref('footer')

const formatDate = (date) => {
  return new Intl.DateTimeFormat('en-US', {
    year: 'numeric',
    month: 'short',
    day: 'numeric'
  }).format(date)
}

const fetchData = async (url) => {
  console.log(`Fetching data from: ${url}`)
  // API call logic
}
</script>
```

#### Template Compilation
Understanding how Vue templates are compiled into render functions for optimization and debugging.

**Real-world usage:** Optimize bundle size, debug template issues, create dynamic components, and understand performance implications of template choices.

```vue
<!-- This template... -->
<template>
  <div class="container">
    <h1>{{ title }}</h1>
    <p v-if="showDescription">{{ description }}</p>
    <button @click="handleClick">{{ buttonText }}</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const title = ref('My App')
const description = ref('App description')
const showDescription = ref(true)
const buttonText = ref('Click me')

const handleClick = () => {
  console.log('Button clicked')
}
</script>

<!-- ...compiles to something like this render function: -->
<script>
// Compiled output (simplified)
import { createElementVNode, toDisplayString, withDirectives, vShow } from 'vue'

export function render(_ctx, _cache) {
  return createElementVNode("div", { class: "container" }, [
    createElementVNode("h1", null, toDisplayString(_ctx.title), 1),
    _ctx.showDescription 
      ? createElementVNode("p", { key: 0 }, toDisplayString(_ctx.description), 1)
      : createCommentVNode("", true),
    createElementVNode("button", {
      onClick: _ctx.handleClick
    }, toDisplayString(_ctx.buttonText), 9, ["onClick"])
  ])
}
</script>
```

```vue
<!-- Template compilation optimization examples -->
<template>
  <div>
    <!-- Static hoisting - these don't change -->
    <div class="header">
      <h1>Static Title</h1>
      <nav>
        <a href="/home">Home</a>
        <a href="/about">About</a>
      </nav>
    </div>
    
    <!-- Dynamic content -->
    <div class="content">
      <p>{{ dynamicText }}</p>
      <button @click="updateText">Update</button>
    </div>
    
    <!-- Patch flag optimizations -->
    <div :class="dynamicClass">
      Dynamic class only
    </div>
    
    <div :style="{ color: textColor }">
      Dynamic style only
    </div>
    
    <!-- v-memo for expensive renders -->
    <div v-memo="[user.id, user.name]">
      <ExpensiveComponent :user="user" />
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const dynamicText = ref('Hello')
const dynamicClass = ref('active')
const textColor = ref('blue')
const user = ref({ id: 1, name: 'John' })

const updateText = () => {
  dynamicText.value = 'Updated!'
}
</script>
```

```javascript
// Build-time template compilation configuration
// vite.config.js
export default {
  define: {
    __VUE_OPTIONS_API__: false, // Disable Options API for smaller bundle
    __VUE_PROD_DEVTOOLS__: false // Disable devtools in production
  },
  build: {
    rollupOptions: {
      external: ['vue'], // Externalize Vue for CDN usage
      output: {
        globals: {
          vue: 'Vue'
        }
      }
    }
  }
}

// Runtime template compilation (not recommended for production)
import { createApp, compile } from 'vue'

const template = `
  <div>
    <h1>{{ title }}</h1>
    <p>{{ message }}</p>
  </div>
`

const renderFunction = compile(template)

const app = createApp({
  data() {
    return {
      title: 'Runtime Template',
      message: 'Compiled at runtime'
    }
  },
  render: renderFunction
})
```

---

### 5. **Directives**

#### Built-in Directives

**`v-text`**
Sets the text content of an element, equivalent to using mustache syntax but prevents layout shift during loading.

**Real-world usage:** Prevent flashing of uncompiled templates, set text content that might contain HTML entities, and ensure text-only content.

```vue
<template>
  <div>
    <!-- Equivalent to {{ message }} but safer -->
    <p v-text="message"></p>
    
    <!-- Prevents HTML injection -->
    <span v-text="userInput"></span>
    
    <!-- Loading states -->
    <h1 v-text="title || 'Loading...'"></h1>
    
    <!-- Computed values -->
    <div v-text="fullName"></div>
    
    <!-- Number formatting -->
    <span v-text="formatPrice(product.price)"></span>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const message = ref('Hello Vue!')
const userInput = ref('<script>alert("XSS")</script>Safe text')
const title = ref('')
const firstName = ref('John')
const lastName = ref('Doe')
const product = ref({ price: 29.99 })

const fullName = computed(() => `${firstName.value} ${lastName.value}`)

const formatPrice = (price) => `$${price.toFixed(2)}`

// Simulate loading
setTimeout(() => {
  title.value = 'Page Loaded'
}, 1000)
</script>
```

**`v-html`**
Sets the innerHTML of an element, rendering raw HTML content.

**Real-world usage:** Display rich content from CMS, render formatted blog posts, show HTML emails (with sanitization).

```vue
<template>
  <div>
    <!-- Rich text content -->
    <div v-html="articleContent"></div>
    
    <!-- Email templates -->
    <div v-html="emailTemplate"></div>
    
    <!-- Sanitized user content -->
    <div v-html="sanitizedUserContent"></div>
    
    <!-- Conditional HTML -->
    <div v-html="showFormatted ? richContent : plainContent"></div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import DOMPurify from 'dompurify'

const articleContent = ref(`
  <h2>Article Title</h2>
  <p>This is <strong>rich text</strong> with <a href="#link">links</a></p>
  <ul><li>List item 1</li><li>List item 2</li></ul>
`)

const emailTemplate = ref(`
  <div style="background: #f5f5f5; padding: 20px;">
    <h1 style="color: #333;">Welcome!</h1>
    <p>Thank you for joining our platform.</p>
  </div>
`)

const userContent = ref('<script>alert("xss")</script><p>User content</p>')
const sanitizedUserContent = computed(() => DOMPurify.sanitize(userContent.value))

const showFormatted = ref(true)
const richContent = ref('<em>Formatted content</em>')
const plainContent = ref('Plain content')
</script>
```

**`v-show`**
Toggles the CSS display property of an element based on a condition.

**Real-world usage:** Show/hide UI elements that toggle frequently, modal overlays, dropdown menus, and conditional content that needs to maintain state.

```vue
<template>
  <div>
    <!-- Basic show/hide -->
    <div v-show="isVisible">Visible content</div>
    
    <!-- Modal overlay -->
    <div v-show="showModal" class="modal-overlay">
      <div class="modal">
        <h2>Modal Title</h2>
        <button @click="showModal = false">Close</button>
      </div>
    </div>
    
    <!-- Dropdown menu -->
    <div class="dropdown">
      <button @click="showDropdown = !showDropdown">Menu</button>
      <ul v-show="showDropdown" class="dropdown-menu">
        <li>Option 1</li>
        <li>Option 2</li>
        <li>Option 3</li>
      </ul>
    </div>
    
    <!-- Loading spinner -->
    <div v-show="isLoading" class="spinner">Loading...</div>
    
    <!-- Error message -->
    <div v-show="errorMessage" class="error">{{ errorMessage }}</div>
    
    <!-- Advanced visibility -->
    <div v-show="user.role === 'admin' && hasPermission">Admin Panel</div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const isVisible = ref(true)
const showModal = ref(false)
const showDropdown = ref(false)
const isLoading = ref(false)
const errorMessage = ref('')
const user = ref({ role: 'admin' })
const hasPermission = ref(true)
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
}

.modal {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: white;
  padding: 2rem;
  border-radius: 8px;
}

.dropdown {
  position: relative;
}

.dropdown-menu {
  position: absolute;
  top: 100%;
  left: 0;
  background: white;
  border: 1px solid #ddd;
  border-radius: 4px;
  list-style: none;
  padding: 0;
  margin: 0;
}
</style>
```

**`v-if`, `v-else-if`, `v-else`**
Conditionally renders elements in the DOM based on expressions.

**Real-world usage:** Authentication states, feature flags, progressive disclosure, error states, and complex conditional rendering.

```vue
<template>
  <div>
    <!-- Authentication states -->
    <div v-if="user.isAuthenticated">
      <h2>Welcome, {{ user.name }}!</h2>
      <button @click="logout">Logout</button>
    </div>
    <div v-else>
      <h2>Please log in</h2>
      <LoginForm @login="handleLogin" />
    </div>
    
    <!-- User roles -->
    <div v-if="user.role === 'admin'">
      <AdminPanel />
    </div>
    <div v-else-if="user.role === 'moderator'">
      <ModeratorPanel />
    </div>
    <div v-else-if="user.role === 'user'">
      <UserPanel />
    </div>
    <div v-else>
      <GuestPanel />
    </div>
    
    <!-- Loading and error states -->
    <div v-if="loading">
      <LoadingSpinner />
    </div>
    <div v-else-if="error">
      <ErrorMessage :error="error" @retry="fetchData" />
    </div>
    <div v-else-if="data.length === 0">
      <EmptyState message="No data available" />
    </div>
    <div v-else>
      <DataTable :data="data" />
    </div>
    
    <!-- Complex conditions -->
    <button 
      v-if="user.isAuthenticated && user.hasPermission('write') && !isReadOnly"
      @click="createPost"
    >
      Create Post
    </button>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'

const user = reactive({
  isAuthenticated: false,
  name: '',
  role: 'guest',
  hasPermission: (permission) => user.permissions?.includes(permission)
})

const loading = ref(false)
const error = ref(null)
const data = ref([])
const isReadOnly = ref(false)

const handleLogin = (userData) => {
  user.isAuthenticated = true
  user.name = userData.name
  user.role = userData.role
  user.permissions = userData.permissions
}

const logout = () => {
  user.isAuthenticated = false
  user.name = ''
  user.role = 'guest'
}

const fetchData = async () => {
  loading.value = true
  error.value = null
  
  try {
    // Simulate API call
    const response = await fetch('/api/data')
    data.value = await response.json()
  } catch (err) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}

const createPost = () => {
  console.log('Creating post...')
}
</script>
```

**`v-for`**
Renders lists of elements by iterating over arrays, objects, strings, or numbers.

**Real-world usage:** Display product lists, user tables, navigation menus, form options, and any repeated content.

```vue
<template>
  <div>
    <!-- Array iteration -->
    <ul>
      <li v-for="item in items" :key="item.id">
        {{ item.name }} - ${{ item.price }}
      </li>
    </ul>
    
    <!-- Array with index -->
    <div v-for="(product, index) in products" :key="product.id">
      <span>{{ index + 1 }}. {{ product.name }}</span>
    </div>
    
    <!-- Object iteration -->
    <dl>
      <template v-for="(value, key) in userProfile" :key="key">
        <dt>{{ key }}:</dt>
        <dd>{{ value }}</dd>
      </template>
    </dl>
    
    <!-- Object with index -->
    <div v-for="(value, key, index) in settings" :key="key">
      {{ index }}. {{ key }}: {{ value }}
    </div>
    
    <!-- Nested loops -->
    <div v-for="category in categories" :key="category.id">
      <h3>{{ category.name }}</h3>
      <ul>
        <li v-for="product in category.products" :key="product.id">
          {{ product.name }}
        </li>
      </ul>
    </div>
    
    <!-- Number iteration -->
    <span v-for="n in 10" :key="n">{{ n }}</span>
    
    <!-- String iteration -->
    <span v-for="letter in 'HELLO'" :key="letter">{{ letter }}</span>
    
    <!-- Template wrapper -->
    <template v-for="user in users" :key="user.id">
      <div class="user-card">
        <h4>{{ user.name }}</h4>
        <p>{{ user.email }}</p>
      </div>
      <hr v-if="user.id !== users[users.length - 1].id">
    </template>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'

const items = ref([
  { id: 1, name: 'Laptop', price: 999 },
  { id: 2, name: 'Phone', price: 599 },
  { id: 3, name: 'Tablet', price: 399 }
])

const products = ref([
  { id: 1, name: 'Product A' },
  { id: 2, name: 'Product B' }
])

const userProfile = reactive({
  name: 'John Doe',
  email: 'john@example.com',
  age: 30,
  city: 'New York'
})

const settings = reactive({
  theme: 'dark',
  notifications: true,
  autoSave: false
})

const categories = ref([
  {
    id: 1,
    name: 'Electronics',
    products: [
      { id: 101, name: 'Smartphone' },
      { id: 102, name: 'Laptop' }
    ]
  },
  {
    id: 2,
    name: 'Books',
    products: [
      { id: 201, name: 'Vue.js Guide' },
      { id: 202, name: 'JavaScript Basics' }
    ]
  }
])

const users = ref([
  { id: 1, name: 'Alice', email: 'alice@example.com' },
  { id: 2, name: 'Bob', email: 'bob@example.com' }
])
</script>
```

**`v-on` (Event Handling)**
Attaches event listeners to elements to handle user interactions.

**Real-world usage:** Handle clicks, form submissions, keyboard input, mouse events, and custom component events.

```vue
<template>
  <div>
    <!-- Basic click events -->
    <button @click="handleClick">Click me</button>
    <button @click="count++">Count: {{ count }}</button>
    
    <!-- Event with parameters -->
    <button @click="greet('Hello')">Greet</button>
    <button @click="addItem('New Item')">Add Item</button>
    
    <!-- Multiple event listeners -->
    <button 
      @click="handleClick"
      @mouseenter="handleHover"
      @mouseleave="handleLeave"
    >
      Hover and Click
    </button>
    
    <!-- Form events -->
    <form @submit.prevent="handleSubmit">
      <input 
        v-model="formData.name"
        @input="handleInput"
        @focus="handleFocus"
        @blur="handleBlur"
      />
      <button type="submit">Submit</button>
    </form>
    
    <!-- Keyboard events -->
    <input 
      @keyup.enter="search"
      @keyup.esc="clearSearch"
      @keydown.ctrl.s.prevent="saveData"
      v-model="searchQuery"
      placeholder="Search..."
    />
    
    <!-- Mouse events -->
    <div 
      @mousedown="handleMouseDown"
      @mouseup="handleMouseUp"
      @mousemove="handleMouseMove"
      @wheel="handleWheel"
      class="interactive-area"
    >
      Interactive Area
    </div>
    
    <!-- Custom component events -->
    <Modal 
      @close="showModal = false"
      @confirm="handleConfirm"
      :show="showModal"
    />
    
    <!-- Dynamic event names -->
    <button @[eventName]="handleDynamicEvent">
      Dynamic Event
    </button>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'

const count = ref(0)
const searchQuery = ref('')
const showModal = ref(false)
const eventName = ref('click')

const formData = reactive({
  name: '',
  email: ''
})

const handleClick = (event) => {
  console.log('Button clicked:', event.target)
}

const greet = (message) => {
  alert(message)
}

const addItem = (item) => {
  console.log('Adding item:', item)
}

const handleHover = () => {
  console.log('Mouse entered')
}

const handleLeave = () => {
  console.log('Mouse left')
}

const handleSubmit = () => {
  console.log('Form submitted:', formData)
}

const handleInput = (event) => {
  console.log('Input changed:', event.target.value)
}

const handleFocus = () => {
  console.log('Input focused')
}

const handleBlur = () => {
  console.log('Input blurred')
}

const search = () => {
  console.log('Searching for:', searchQuery.value)
}

const clearSearch = () => {
  searchQuery.value = ''
}

const saveData = () => {
  console.log('Saving data...')
}

const handleMouseDown = (event) => {
  console.log('Mouse down at:', event.clientX, event.clientY)
}

const handleMouseUp = () => {
  console.log('Mouse up')
}

const handleMouseMove = (event) => {
  // console.log('Mouse move:', event.clientX, event.clientY)
}

const handleWheel = (event) => {
  console.log('Wheel delta:', event.deltaY)
}

const handleConfirm = (data) => {
  console.log('Modal confirmed:', data)
  showModal.value = false
}

const handleDynamicEvent = () => {
  console.log('Dynamic event triggered')
}
</script>

<style scoped>
.interactive-area {
  width: 200px;
  height: 100px;
  border: 2px solid #ccc;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
}
</style>
```

**`v-bind` (Attribute Binding)**
Dynamically binds HTML attributes and properties to reactive data.

**Real-world usage:** Dynamic styling, conditional attributes, data binding for forms, accessibility attributes, and responsive layouts.

```vue
<template>
  <div>
    <!-- Basic attribute binding -->
    <img :src="imageUrl" :alt="imageAlt" :width="imageWidth" />
    
    <!-- Class binding -->
    <div :class="{ active: isActive, disabled: isDisabled }">Status</div>
    <div :class="[baseClass, typeClass, { error: hasError }]">Multiple classes</div>
    
    <!-- Style binding -->
    <div :style="{ color: textColor, fontSize: fontSize + 'px' }">Styled text</div>
    <div :style="[baseStyles, conditionalStyles]">Multiple styles</div>
    
    <!-- Form attributes -->
    <input 
      :type="inputType"
      :placeholder="inputPlaceholder"
      :disabled="isFormDisabled"
      :required="isFieldRequired"
      :maxlength="maxLength"
    />
    
    <!-- Accessibility attributes -->
    <button 
      :aria-label="buttonLabel"
      :aria-expanded="isMenuOpen"
      :aria-disabled="isDisabled"
      :tabindex="tabIndex"
    >
      Accessible Button
    </button>
    
    <!-- Data attributes -->
    <div 
      :data-id="userId"
      :data-role="userRole"
      :data-testid="`user-${userId}`"
    >
      User element
    </div>
    
    <!-- Binding entire objects -->
    <div v-bind="elementAttributes">Dynamic attributes</div>
    
    <!-- Conditional attributes -->
    <a 
      :href="link.external ? link.url : null"
      :target="link.external ? '_blank' : null"
      :rel="link.external ? 'noopener noreferrer' : null"
    >
      {{ link.text }}
    </a>
  </div>
</template>

<script setup>
import { ref, reactive, computed } from 'vue'

const imageUrl = ref('/images/placeholder.jpg')
const imageAlt = ref('Placeholder image')
const imageWidth = ref(300)

const isActive = ref(true)
const isDisabled = ref(false)
const hasError = ref(false)

const baseClass = ref('card')
const typeClass = ref('primary')

const textColor = ref('#333')
const fontSize = ref(16)

const baseStyles = reactive({
  padding: '1rem',
  border: '1px solid #ddd'
})

const conditionalStyles = computed(() => ({
  backgroundColor: isActive.value ? '#f0f8ff' : '#f5f5f5'
}))

const inputType = ref('email')
const inputPlaceholder = ref('Enter your email')
const isFormDisabled = ref(false)
const isFieldRequired = ref(true)
const maxLength = ref(50)

const buttonLabel = ref('Open navigation menu')
const isMenuOpen = ref(false)
const tabIndex = ref(0)

const userId = ref(123)
const userRole = ref('admin')

const elementAttributes = reactive({
  id: 'dynamic-element',
  class: 'dynamic-class',
  'data-version': '1.0',
  style: 'margin: 1rem;'
})

const link = reactive({
  text: 'External Link',
  url: 'https://example.com',
  external: true
})
</script>
```

**`v-model` (Two-way Binding)**
Creates two-way data binding between form inputs and component data.

**Real-world usage:** Form handling, user input collection, search functionality, settings panels, and real-time data synchronization.

```vue
<template>
  <div>
    <!-- Text inputs -->
    <input v-model="username" placeholder="Username" />
    <input v-model.trim="email" type="email" placeholder="Email" />
    <input v-model.number="age" type="number" placeholder="Age" />
    
    <!-- Textarea -->
    <textarea v-model="message" placeholder="Your message"></textarea>
    
    <!-- Checkboxes -->
    <label>
      <input type="checkbox" v-model="agreed" />
      I agree to the terms
    </label>
    
    <!-- Multiple checkboxes -->
    <div>
      <label v-for="hobby in hobbies" :key="hobby">
        <input type="checkbox" :value="hobby" v-model="selectedHobbies" />
        {{ hobby }}
      </label>
    </div>
    
    <!-- Radio buttons -->
    <div>
      <label v-for="size in sizes" :key="size">
        <input type="radio" :value="size" v-model="selectedSize" />
        {{ size }}
      </label>
    </div>
    
    <!-- Select dropdown -->
    <select v-model="selectedCountry">
      <option value="">Choose country</option>
      <option v-for="country in countries" :key="country.code" :value="country.code">
        {{ country.name }}
      </option>
    </select>
    
    <!-- Multiple select -->
    <select v-model="selectedSkills" multiple>
      <option v-for="skill in skills" :key="skill" :value="skill">
        {{ skill }}
      </option>
    </select>
    
    <!-- Custom components -->
    <CustomInput v-model="customValue" placeholder="Custom input" />
    <DatePicker v-model="selectedDate" />
    <ColorPicker v-model="selectedColor" />
    
    <!-- Modifiers -->
    <input v-model.lazy="lazyValue" placeholder="Updates on blur" />
    <input v-model.number="numericValue" placeholder="Auto-converts to number" />
    <input v-model.trim="trimmedValue" placeholder="Auto-trims whitespace" />
    
    <!-- Display values -->
    <div class="output">
      <p>Username: {{ username }}</p>
      <p>Email: {{ email }}</p>
      <p>Age: {{ age }}</p>
      <p>Message: {{ message }}</p>
      <p>Agreed: {{ agreed }}</p>
      <p>Hobbies: {{ selectedHobbies.join(', ') }}</p>
      <p>Size: {{ selectedSize }}</p>
      <p>Country: {{ selectedCountry }}</p>
      <p>Skills: {{ selectedSkills.join(', ') }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const username = ref('')
const email = ref('')
const age = ref(null)
const message = ref('')
const agreed = ref(false)

const hobbies = ['Reading', 'Gaming', 'Sports', 'Music']
const selectedHobbies = ref([])

const sizes = ['Small', 'Medium', 'Large']
const selectedSize = ref('')

const countries = [
  { code: 'us', name: 'United States' },
  { code: 'ca', name: 'Canada' },
  { code: 'uk', name: 'United Kingdom' }
]
const selectedCountry = ref('')

const skills = ['JavaScript', 'Vue.js', 'React', 'Node.js', 'Python']
const selectedSkills = ref([])

const customValue = ref('')
const selectedDate = ref('')
const selectedColor = ref('#ff0000')

const lazyValue = ref('')
const numericValue = ref(0)
const trimmedValue = ref('')
</script>

<style scoped>
.output {
  margin-top: 2rem;
  padding: 1rem;
  background: #f5f5f5;
  border-radius: 4px;
}

label {
  display: block;
  margin: 0.5rem 0;
}

input, textarea, select {
  margin: 0.5rem 0;
  padding: 0.5rem;
  border: 1px solid #ddd;
  border-radius: 4px;
}

textarea {
  width: 100%;
  height: 100px;
}
</style>
```

**`v-slot`**
Defines named slots and scoped slots for flexible component content distribution.

**Real-world usage:** Layout components, card templates, modal containers, table customization, and reusable UI patterns.

```vue
<!-- Parent component using slots -->
<template>
  <div>
    <!-- Basic slot usage -->
    <Card>
      <h3>Card Title</h3>
      <p>Card content goes here</p>
    </Card>
    
    <!-- Named slots -->
    <Modal>
      <template #header>
        <h2>Modal Title</h2>
      </template>
      
      <template #default>
        <p>Main modal content</p>
      </template>
      
      <template #footer>
        <button @click="closeModal">Cancel</button>
        <button @click="saveModal">Save</button>
      </template>
    </Modal>
    
    <!-- Scoped slots -->
    <UserList>
      <template #user="{ user, index }">
        <div class="user-card">
          <img :src="user.avatar" :alt="user.name" />
          <h4>{{ index + 1 }}. {{ user.name }}</h4>
          <p>{{ user.email }}</p>
          <span :class="user.isOnline ? 'online' : 'offline'">
            {{ user.isOnline ? 'Online' : 'Offline' }}
          </span>
        </div>
      </template>
    </UserList>
    
    <!-- Table with custom columns -->
    <DataTable :data="tableData">
      <template #column-name="{ item }">
        <strong>{{ item.name }}</strong>
      </template>
      
      <template #column-status="{ item }">
        <span :class="`status-${item.status}`">
          {{ item.status.toUpperCase() }}
        </span>
      </template>
      
      <template #column-actions="{ item, index }">
        <button @click="editItem(item, index)">Edit</button>
        <button @click="deleteItem(item, index)">Delete</button>
      </template>
    </DataTable>
    
    <!-- Conditional slots -->
    <Article>
      <template #title v-if="showTitle">
        <h1>{{ article.title }}</h1>
      </template>
      
      <template #content>
        <div v-html="article.content"></div>
      </template>
      
      <template #sidebar v-if="article.tags.length">
        <div class="tags">
          <span v-for="tag in article.tags" :key="tag" class="tag">
            {{ tag }}
          </span>
        </div>
      </template>
    </Article>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const showTitle = ref(true)

const tableData = ref([
  { id: 1, name: 'John Doe', status: 'active' },
  { id: 2, name: 'Jane Smith', status: 'inactive' }
])

const article = ref({
  title: 'Vue.js Slots Guide',
  content: '<p>Content about Vue slots...</p>',
  tags: ['vue', 'javascript', 'frontend']
})

const closeModal = () => {
  console.log('Modal closed')
}

const saveModal = () => {
  console.log('Modal saved')
}

const editItem = (item, index) => {
  console.log('Editing:', item, 'at index:', index)
}

const deleteItem = (item, index) => {
  console.log('Deleting:', item, 'at index:', index)
}
</script>

<!-- Example slot-receiving components -->

<!-- Card.vue -->
<template>
  <div class="card">
    <slot></slot>
  </div>
</template>

<!-- Modal.vue -->
<template>
  <div class="modal-overlay">
    <div class="modal">
      <header class="modal-header">
        <slot name="header"></slot>
      </header>
      
      <main class="modal-body">
        <slot></slot>
      </main>
      
      <footer class="modal-footer">
        <slot name="footer"></slot>
      </footer>
    </div>
  </div>
</template>

<!-- UserList.vue -->
<template>
  <div class="user-list">
    <div v-for="(user, index) in users" :key="user.id">
      <slot name="user" :user="user" :index="index"></slot>
    </div>
  </div>
</template>

<script setup>
defineProps(['users'])
</script>
```

**`v-pre`**
Skips compilation for an element and its children, useful for displaying raw Vue syntax or improving performance.

**Real-world usage:** Display code examples, show raw template syntax, optimize static content rendering, and documentation sites.

```vue
<template>
  <div>
    <!-- Display raw Vue syntax -->
    <pre v-pre>
      {{ message }}
      <span v-if="condition">Conditional content</span>
    </pre>
    
    <!-- Code examples in documentation -->
    <div class="code-example">
      <h3>Template Example:</h3>
      <code v-pre>
        &lt;div v-for="item in items" :key="item.id"&gt;
          {{ item.name }}
        &lt;/div&gt;
      </code>
    </div>
    
    <!-- Large static content -->
    <div v-pre class="static-content">
      <h1>Static Header</h1>
      <p>This large static content block won't be compiled by Vue</p>
      <ul>
        <li>Item 1</li>
        <li>Item 2</li>
        <li>Item 3</li>
      </ul>
    </div>
    
    <!-- Mustache syntax examples -->
    <table class="syntax-table">
      <tr>
        <td>Interpolation:</td>
        <td v-pre>{{ message }}</td>
      </tr>
      <tr>
        <td>Attribute binding:</td>
        <td v-pre>:src="imageUrl"</td>
      </tr>
      <tr>
        <td>Event handling:</td>
        <td v-pre>@click="handleClick"</td>
      </tr>
    </table>
  </div>
</template>

<script setup>
// These won't affect the v-pre content
const message = ref('This will be processed')
const condition = ref(true)
</script>

<style scoped>
.code-example {
  background: #f5f5f5;
  padding: 1rem;
  border-radius: 4px;
  margin: 1rem 0;
}

.static-content {
  border: 2px solid #ddd;
  padding: 1rem;
  margin: 1rem 0;
}

.syntax-table {
  border-collapse: collapse;
  width: 100%;
}

.syntax-table td {
  border: 1px solid #ddd;
  padding: 0.5rem;
}
</style>
```

**`v-once`**
Renders element or component only once, caching the result for performance optimization.

**Real-world usage:** Static content that never changes, expensive computed content, one-time interpolations, and performance optimization for large lists.

```vue
<template>
  <div>
    <!-- Render expensive component once -->
    <ExpensiveChart v-once :data="chartData" />
    
    <!-- One-time string interpolation -->
    <h1 v-once>{{ title }}</h1>
    
    <!-- Static list that won't change -->
    <ul v-once>
      <li v-for="item in staticItems" :key="item.id">
        {{ item.name }}
      </li>
    </ul>
    
    <!-- Child component rendered once -->
    <div v-once>
      <UserProfile :user="currentUser" />
      <UserStats :stats="userStats" />
    </div>
    
    <!-- One-time expensive computation -->
    <div v-once>
      {{ expensiveCalculation() }}
    </div>
    
    <!-- Comparison: with and without v-once -->
    <div>
      <h3>Without v-once (updates with counter):</h3>
      <p>Current time: {{ new Date().toLocaleString() }}</p>
      <p>Counter: {{ counter }}</p>
      
      <h3>With v-once (frozen at first render):</h3>
      <p v-once>Current time: {{ new Date().toLocaleString() }}</p>
      <p v-once>Counter: {{ counter }}</p>
      
      <button @click="counter++">Increment Counter</button>
    </div>
    
    <!-- Performance optimization for large static lists -->
    <div class="user-directory">
      <h2>Employee Directory (Static)</h2>
      <div v-once class="employee-grid">
        <div v-for="employee in employees" :key="employee.id" class="employee-card">
          <img :src="employee.avatar" :alt="employee.name" />
          <h4>{{ employee.name }}</h4>
          <p>{{ employee.department }}</p>
          <p>{{ employee.email }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const title = ref('Application Title')
const counter = ref(0)

const chartData = ref([
  { label: 'Q1', value: 100 },
  { label: 'Q2', value: 150 },
  { label: 'Q3', value: 200 },
  { label: 'Q4', value: 175 }
])

const staticItems = ref([
  { id: 1, name: 'Static Item 1' },
  { id: 2, name: 'Static Item 2' },
  { id: 3, name: 'Static Item 3' }
])

const currentUser = ref({
  name: 'John Doe',
  email: 'john@example.com',
  avatar: '/avatars/john.jpg'
})

const userStats = ref({
  posts: 45,
  followers: 1230,
  following: 890
})

const employees = ref(
  Array.from({ length: 100 }, (_, i) => ({
    id: i + 1,
    name: `Employee ${i + 1}`,
    department: ['Engineering', 'Marketing', 'Sales', 'HR'][i % 4],
    email: `employee${i + 1}@company.com`,
    avatar: `/avatars/employee${i + 1}.jpg`
  }))
)

const expensiveCalculation = () => {
  console.log('Expensive calculation running...')
  // Simulate expensive operation
  let result = 0
  for (let i = 0; i < 1000000; i++) {
    result += Math.random()
  }
  return result.toFixed(2)
}

// Simulate data that might change
setTimeout(() => {
  title.value = 'Updated Title' // This won't update the v-once element
}, 3000)
</script>

<style scoped>
.employee-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 1rem;
}

.employee-card {
  border: 1px solid #ddd;
  border-radius: 4px;
  padding: 1rem;
  text-align: center;
}

.employee-card img {
  width: 50px;
  height: 50px;
  border-radius: 50%;
}
</style>
```

**`v-memo`**
Memoizes a sub-tree of the template based on a dependency array, re-rendering only when dependencies change.

**Real-world usage:** Optimize large lists with complex items, expensive component renders, and performance-critical sections.

```vue
<template>
  <div>
    <!-- Basic v-memo usage -->
    <div v-memo="[user.name, user.email]">
      <UserCard :user="user" />
      <UserStats :stats="user.stats" />
    </div>
    
    <!-- Large list optimization -->
    <div class="product-list">
      <div 
        v-for="product in products" 
        :key="product.id"
        v-memo="[product.name, product.price, product.inStock]"
        class="product-item"
      >
        <img :src="product.image" :alt="product.name" />
        <h3>{{ product.name }}</h3>
        <p class="price">${{ product.price }}</p>
        <p class="stock" :class="{ 'out-of-stock': !product.inStock }">
          {{ product.inStock ? 'In Stock' : 'Out of Stock' }}
        </p>
        <button 
          :disabled="!product.inStock"
          @click="addToCart(product)"
        >
          Add to Cart
        </button>
      </div>
    </div>
    
    <!-- Complex component with memo -->
    <div 
      v-for="report in reports" 
      :key="report.id"
      v-memo="[report.data, report.config, report.lastUpdated]"
    >
      <ExpensiveChart 
        :data="report.data"
        :config="report.config"
        :title="report.title"
      />
    </div>
    
    <!-- Conditional memoization -->
    <div 
      v-memo="optimizeRender ? [criticalData] : []"
      class="dashboard-widget"
    >
      <DashboardWidget 
        :data="criticalData"
        :settings="widgetSettings"
        :theme="currentTheme"
      />
    </div>
    
    <!-- Performance comparison -->
    <div class="performance-demo">
      <h3>Without v-memo (always re-renders):</h3>
      <div class="item-list">
        <div v-for="item in performanceItems" :key="item.id">
          <ExpensiveItem :item="item" :counter="globalCounter" />
        </div>
      </div>
      
      <h3>With v-memo (only re-renders when item changes):</h3>
      <div class="item-list">
        <div 
          v-for="item in performanceItems" 
          :key="item.id"
          v-memo="[item.value, item.status]"
        >
          <ExpensiveItem :item="item" :counter="globalCounter" />
        </div>
      </div>
      
      <button @click="globalCounter++">
        Increment Global Counter: {{ globalCounter }}
      </button>
      <button @click="updateRandomItem">
        Update Random Item
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'

const user = reactive({
  name: 'John Doe',
  email: 'john@example.com',
  stats: {
    posts: 25,
    followers: 150,
    following: 89
  }
})

const products = ref([
  {
    id: 1,
    name: 'Laptop Pro',
    price: 1299,
    inStock: true,
    image: '/images/laptop.jpg'
  },
  {
    id: 2,
    name: 'Wireless Mouse',
    price: 29,
    inStock: false,
    image: '/images/mouse.jpg'
  },
  {
    id: 3,
    name: 'Mechanical Keyboard',
    price: 89,
    inStock: true,
    image: '/images/keyboard.jpg'
  }
])

const reports = ref([
  {
    id: 1,
    title: 'Sales Report',
    data: [100, 200, 150, 300],
    config: { type: 'line', color: 'blue' },
    lastUpdated: Date.now()
  },
  {
    id: 2,
    title: 'User Analytics',
    data: [50, 75, 100, 125],
    config: { type: 'bar', color: 'green' },
    lastUpdated: Date.now()
  }
])

const optimizeRender = ref(true)
const criticalData = ref({ value: 100, timestamp: Date.now() })
const widgetSettings = ref({ theme: 'dark', refresh: 30 })
const currentTheme = ref('dark')

const globalCounter = ref(0)
const performanceItems = ref(
  Array.from({ length: 1000 }, (_, i) => ({
    id: i,
    value: Math.random() * 100,
    status: ['active', 'inactive', 'pending'][i % 3]
  }))
)

const addToCart = (product) => {
  console.log('Added to cart:', product.name)
}

const updateRandomItem = () => {
  const randomIndex = Math.floor(Math.random() * performanceItems.value.length)
  performanceItems.value[randomIndex].value = Math.random() * 100
  performanceItems.value[randomIndex].status = 
    ['active', 'inactive', 'pending'][Math.floor(Math.random() * 3)]
}

// Simulate data updates
setInterval(() => {
  criticalData.value = {
    value: Math.random() * 100,
    timestamp: Date.now()
  }
}, 5000)
</script>

<style scoped>
.product-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 1rem;
  margin: 2rem 0;
}

.product-item {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 1rem;
  text-align: center;
}

.product-item img {
  width: 100%;
  height: 150px;
  object-fit: cover;
  border-radius: 4px;
}

.price {
  font-size: 1.2em;
  font-weight: bold;
  color: #007bff;
}

.out-of-stock {
  color: #dc3545;
}

.performance-demo {
  margin: 2rem 0;
  padding: 1rem;
  border: 2px solid #007bff;
  border-radius: 8px;
}

.item-list {
  max-height: 200px;
  overflow-y: auto;
  border: 1px solid #ddd;
  margin: 1rem 0;
}
</style>
```

**`v-cloak`**
Hides elements until Vue compilation is complete, preventing flash of uncompiled templates.

**Real-world usage:** Prevent FOUC (Flash of Unstyled Content), hide mustache syntax during loading, and improve perceived performance.

```vue
<template>
  <div>
    <!-- Hide content until Vue loads -->
    <div v-cloak class="app-content">
      <h1>{{ appTitle }}</h1>
      <p>{{ description }}</p>
      
      <div v-for="item in items" :key="item.id">
        {{ item.name }} - {{ item.price }}
      </div>
    </div>
    
    <!-- Loading state while Vue initializes -->
    <div class="loading-fallback">
      <p>Loading application...</p>
      <div class="spinner"></div>
    </div>
    
    <!-- Complex template that needs hiding -->
    <div v-cloak class="dashboard">
      <header class="header">
        <h1>{{ user.name }}'s Dashboard</h1>
        <nav>
          <a v-for="link in navigation" :key="link.path" :href="link.path">
            {{ link.title }}
          </a>
        </nav>
      </header>
      
      <main class="main-content">
        <div class="widgets">
          <div v-for="widget in widgets" :key="widget.id" class="widget">
            <h3>{{ widget.title }}</h3>
            <p>{{ widget.content }}</p>
          </div>
        </div>
      </main>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const appTitle = ref('My Vue Application')
const description = ref('Welcome to our amazing application!')

const items = ref([
  { id: 1, name: 'Product A', price: 99 },
  { id: 2, name: 'Product B', price: 149 },
  { id: 3, name: 'Product C', price: 199 }
])

const user = ref({
  name: 'John Doe',
  email: 'john@example.com'
})

const navigation = ref([
  { path: '/dashboard', title: 'Dashboard' },
  { path: '/profile', title: 'Profile' },
  { path: '/settings', title: 'Settings' }
])

const widgets = ref([
  { id: 1, title: 'Sales', content: 'Total sales this month: $12,345' },
  { id: 2, title: 'Users', content: 'Active users: 1,234' },
  { id: 3, title: 'Revenue', content: 'Monthly revenue: $45,678' }
])
</script>

<style>
/* Hide elements with v-cloak until Vue loads */
[v-cloak] {
  display: none !important;
}

/* Show loading fallback by default */
.loading-fallback {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 200px;
}

/* Hide loading fallback when Vue loads */
.app-content:not([v-cloak]) ~ .loading-fallback {
  display: none;
}

.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid #f3f3f3;
  border-top: 4px solid #007bff;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.dashboard {
  min-height: 100vh;
}

.header {
  background: #007bff;
  color: white;
  padding: 1rem;
}

.header nav a {
  color: white;
  text-decoration: none;
  margin-right: 1rem;
}

.widgets {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 1rem;
  padding: 2rem;
}

.widget {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 1rem;
  background: white;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}
</style>
```

---

#### Directive Arguments

**Static Arguments**
Fixed argument names passed to directives to specify which property or event to bind to.

**Real-world usage:** Bind specific HTML attributes, handle particular events, and target specific element properties.

```vue
<template>
  <div>
    <!-- v-bind with static arguments -->
    <img v-bind:src="imageUrl" v-bind:alt="imageAlt" />
    
    <!-- v-on with static arguments -->
    <button v-on:click="handleClick">Click</button>
    <input v-on:keyup="handleKeyup" />
    
    <!-- Custom directive with static arguments -->
    <div v-tooltip:top="tooltipMessage">Hover me</div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const imageUrl = ref('/photo.jpg')
const imageAlt = ref('Profile photo')
const tooltipMessage = ref('This is a tooltip')

const handleClick = () => console.log('Clicked')
const handleKeyup = (e) => console.log('Key:', e.key)
</script>
```

**Dynamic Arguments**
Variable argument names determined at runtime, allowing flexible directive usage based on reactive data.

**Real-world usage:** Dynamic event handling, conditional attribute binding, and runtime directive configuration.

```vue
<template>
  <div>
    <!-- Dynamic event binding -->
    <button v-on:[eventName]="handleEvent">
      Event: {{ eventName }}
    </button>
    
    <!-- Dynamic attribute binding -->
    <input v-bind:[attributeName]="attributeValue" />
    
    <!-- Dynamic directive arguments -->
    <div v-tooltip:[tooltipPosition]="tooltipText">
      Dynamic tooltip
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const eventName = ref('click')
const attributeName = ref('placeholder')
const attributeValue = ref('Dynamic placeholder')
const tooltipPosition = ref('bottom')
const tooltipText = ref('Dynamic tooltip')

const handleEvent = () => console.log(`${eventName.value} triggered`)

// Update dynamically
setTimeout(() => {
  eventName.value = 'dblclick'
  tooltipPosition.value = 'right'
}, 3000)
</script>
```

#### Directive Modifiers

**Event Modifiers**
Chainable modifiers that alter event behavior without additional JavaScript logic.

**Real-world usage:** Form validation, preventing default behaviors, event bubbling control.

```vue
<template>
  <div>
    <!-- Prevent default -->
    <form @submit.prevent="handleSubmit">
      <button type="submit">Submit</button>
    </form>
    
    <!-- Stop propagation -->
    <div @click="parentClick">
      Parent
      <button @click.stop="childClick">Child</button>
    </div>
    
    <!-- Self modifier -->
    <div @click.self="handleSelf">
      <p>Click container only</p>
    </div>
    
    <!-- Once modifier -->
    <button @click.once="handleOnce">Click once</button>
  </div>
</template>

<script setup>
const handleSubmit = () => console.log('Form submitted')
const parentClick = () => console.log('Parent clicked')
const childClick = () => console.log('Child clicked')
const handleSelf = () => console.log('Container clicked')
const handleOnce = () => console.log('Will only log once')
</script>
```

**Key Modifiers**
Handle specific keyboard events with semantic key names.

**Real-world usage:** Keyboard shortcuts, form navigation, accessibility features.

```vue
<template>
  <div>
    <!-- Basic key modifiers -->
    <input 
      @keyup.enter="search"
      @keyup.esc="clear"
      v-model="query"
      placeholder="Enter to search, Esc to clear"
    />
    
    <!-- Arrow navigation -->
    <input 
      @keydown.arrow-down="selectNext"
      @keydown.arrow-up="selectPrevious"
      placeholder="Use arrow keys"
    />
    
    <!-- Custom shortcuts -->
    <textarea 
      @keydown.ctrl.s.prevent="save"
      @keydown.ctrl.enter="submit"
      v-model="content"
    ></textarea>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const query = ref('')
const content = ref('')

const search = () => console.log('Searching:', query.value)
const clear = () => query.value = ''
const selectNext = () => console.log('Next item')
const selectPrevious = () => console.log('Previous item')
const save = () => console.log('Saved')
const submit = () => console.log('Submitted')
</script>
```

**System Modifier Keys**
Handle combinations with system keys (Ctrl, Alt, Shift, Meta) for advanced shortcuts.

**Real-world usage:** Application shortcuts, power user features, desktop-like interactions.

```vue
<template>
  <div>
    <!-- Ctrl combinations -->
    <div 
      @keydown.ctrl.s.prevent="saveDoc"
      @keydown.ctrl.z.prevent="undo"
      tabindex="0"
    >
      Document area (Ctrl+S, Ctrl+Z)
    </div>
    
    <!-- Alt combinations -->
    <div 
      @keydown.alt.f.prevent="showMenu"
      tabindex="0"
    >
      Alt+F for menu
    </div>
    
    <!-- Multi-selection -->
    <div 
      v-for="item in items" 
      :key="item.id"
      @click.ctrl="toggleSelect(item)"
      @click.exact="selectOnly(item)"
    >
      {{ item.name }}
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const items = ref([
  { id: 1, name: 'Item 1' },
  { id: 2, name: 'Item 2' }
])

const saveDoc = () => console.log('Document saved')
const undo = () => console.log('Undo')
const showMenu = () => console.log('Menu shown')
const toggleSelect = (item) => console.log('Toggle:', item.name)
const selectOnly = (item) => console.log('Select only:', item.name)
</script>
```

**Mouse Button Modifiers**
Handle specific mouse button events for enhanced interactions.

**Real-world usage:** Context menus, middle-click actions, advanced selection behaviors.

```vue
<template>
  <div>
    <!-- Mouse button detection -->
    <div 
      @click.left="leftClick"
      @click.right.prevent="rightClick"
      @click.middle="middleClick"
    >
      Try different mouse buttons
    </div>
    
    <!-- Link behaviors -->
    <a 
      href="https://example.com"
      @click.left.exact="normalNav"
      @click.middle.prevent="newTab"
      @click.ctrl.left.prevent="backgroundTab"
    >
      Smart link
    </a>
  </div>
</template>

<script setup>
const leftClick = () => console.log('Left click')
const rightClick = () => console.log('Right click')
const middleClick = () => console.log('Middle click')
const normalNav = (e) => { e.preventDefault(); console.log('Normal navigation') }
const newTab = () => console.log('Open in new tab')
const backgroundTab = () => console.log('Open in background')
</script>
```

**Form Input Modifiers**
Modify v-model behavior for different input scenarios.

**Real-world usage:** Form validation, data formatting, performance optimization.

```vue
<template>
  <div>
    <!-- Lazy modifier -->
    <input v-model.lazy="lazyValue" placeholder="Updates on blur" />
    <span>{{ lazyValue }}</span>
    
    <!-- Number modifier -->
    <input v-model.number="age" type="number" />
    <span>{{ age }} ({{ typeof age }})</span>
    
    <!-- Trim modifier -->
    <input v-model.trim="username" placeholder="Removes whitespace" />
    <span>"{{ username }}"</span>
    
    <!-- Combined modifiers -->
    <input v-model.lazy.number.trim="price" placeholder="Price" />
    <span>{{ price }}</span>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const lazyValue = ref('')
const age = ref(0)
const username = ref('')
const price = ref(0)
</script>
```

#### Custom Directives

**Directive Hooks**
Lifecycle hooks that control when and how custom directives execute.

**Real-world usage:** DOM manipulation, third-party library integration, custom animations.

```vue
<template>
  <div>
    <input v-focus placeholder="Auto-focused" />
    <div v-click-outside="close">Click outside to close</div>
    <div v-tooltip="'Hover tooltip'">Hover me</div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const showPanel = ref(true)

const close = () => showPanel.value = false

// Focus directive
const vFocus = {
  mounted(el) {
    el.focus()
  }
}

// Click outside directive
const vClickOutside = {
  beforeMount(el, binding) {
    el.clickOutsideEvent = (event) => {
      if (!(el === event.target || el.contains(event.target))) {
        binding.value()
      }
    }
    document.addEventListener('click', el.clickOutsideEvent)
  },
  unmounted(el) {
    document.removeEventListener('click', el.clickOutsideEvent)
  }
}

// Tooltip directive
const vTooltip = {
  mounted(el, binding) {
    const tooltip = document.createElement('div')
    tooltip.textContent = binding.value
    tooltip.style.cssText = 'position:absolute;background:black;color:white;padding:4px;display:none'
    document.body.appendChild(tooltip)
    el._tooltip = tooltip
    
    el.addEventListener('mouseenter', () => tooltip.style.display = 'block')
    el.addEventListener('mouseleave', () => tooltip.style.display = 'none')
  },
  unmounted(el) {
    if (el._tooltip) document.body.removeChild(el._tooltip)
  }
}
</script>
```

**Directive Arguments and Modifiers**
Pass arguments and modifiers to custom directives for flexible behavior.

**Real-world usage:** Position-aware components, configurable animations, context-specific behaviors.

```vue
<template>
  <div>
    <!-- Tooltip with position -->
    <button v-tooltip:top="'Top tooltip'">Top</button>
    <button v-tooltip:bottom="'Bottom tooltip'">Bottom</button>
    
    <!-- Animation with modifiers -->
    <div v-animate.fade.slow="visible">Fade slow</div>
    <div v-animate.slide.fast="visible">Slide fast</div>
    
    <!-- Color with argument -->
    <div v-color:background="'red'">Red background</div>
    <div v-color:text="'blue'">Blue text</div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const visible = ref(true)

// Enhanced tooltip with position
const vTooltip = {
  mounted(el, binding) {
    const position = binding.arg || 'top'
    // Implementation based on position
    console.log('Tooltip position:', position)
  }
}

// Animation with modifiers
const vAnimate = {
  mounted(el, binding) {
    const type = binding.modifiers.fade ? 'fade' : 'slide'
    const speed = binding.modifiers.slow ? '1s' : binding.modifiers.fast ? '0.2s' : '0.5s'
    
    el.style.transition = `all ${speed}`
    if (type === 'fade') {
      el.style.opacity = binding.value ? '1' : '0'
    }
  }
}

// Color directive with argument
const vColor = {
  mounted(el, binding) {
    const property = binding.arg === 'background' ? 'backgroundColor' : 'color'
    el.style[property] = binding.value
  }
}
</script>
```

**Global vs Local Directives**
Register directives globally for app-wide use or locally within components.

**Real-world usage:** Common utilities as global, component-specific behaviors as local.

```javascript
// Global registration (main.js)
import { createApp } from 'vue'

const app = createApp(App)

app.directive('focus', {
  mounted(el) { el.focus() }
})

app.directive('tooltip', {
  mounted(el, binding) {
    // Global tooltip implementation
  }
})

app.mount('#app')
```

```vue
<!-- Local directives in component -->
<template>
  <div>
    <!-- Global directive -->
    <input v-focus />
    
    <!-- Local directive -->
    <div v-highlight="color">Highlighted</div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const color = ref('yellow')

// Local directive - only available in this component
const vHighlight = {
  mounted(el, binding) {
    el.style.backgroundColor = binding.value
  },
  updated(el, binding) {
    el.style.backgroundColor = binding.value
  }
}
</script>
```

---

## **Part III: Reactivity System**

### 6. **Vue 3 Reactivity Fundamentals**

#### Proxy-based Reactivity
Vue 3 uses JavaScript Proxy to intercept and track property access, enabling automatic dependency tracking and updates.

**Real-world usage:** Automatic UI updates when data changes, computed property invalidation, and watcher triggering without manual observation setup.

```vue
<template>
  <div>
    <p>Count: {{ count }}</p>
    <p>User: {{ user.name }} ({{ user.age }})</p>
    <button @click="increment">Increment</button>
    <button @click="updateUser">Update User</button>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'

// Proxy-wrapped reactive data
const count = ref(0)
const user = reactive({ name: 'John', age: 30 })

const increment = () => count.value++ // Triggers reactivity
const updateUser = () => {
  user.name = 'Jane' // Triggers reactivity
  user.age = 25
}
</script>
```

#### `ref()` for Primitives
Creates reactive references for primitive values (string, number, boolean) and single objects.

**Real-world usage:** Form inputs, toggles, counters, loading states, and any single value that needs reactivity.

```vue
<template>
  <div>
    <input v-model="message" />
    <p>{{ message }}</p>
    
    <button @click="toggle">{{ isVisible ? 'Hide' : 'Show' }}</button>
    <div v-if="isVisible">Toggleable content</div>
    
    <p>Count: {{ count }}</p>
    <button @click="count++">+</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// Primitive refs
const message = ref('Hello')
const isVisible = ref(true)
const count = ref(0)

// Object ref
const user = ref({ name: 'John', email: 'john@example.com' })

const toggle = () => isVisible.value = !isVisible.value

// Access with .value in script
console.log(message.value) // "Hello"
user.value.name = 'Jane' // Triggers reactivity
</script>
```

#### `reactive()` for Objects
Creates reactive proxy for objects and arrays, allowing direct property access without `.value`.

**Real-world usage:** Form data, application state, user profiles, settings objects, and complex nested data structures.

```vue
<template>
  <div>
    <form @submit.prevent="submitForm">
      <input v-model="form.name" placeholder="Name" />
      <input v-model="form.email" placeholder="Email" />
      <button type="submit">Submit</button>
    </form>
    
    <ul>
      <li v-for="item in items" :key="item.id">
        {{ item.name }} - {{ item.price }}
      </li>
    </ul>
    <button @click="addItem">Add Item</button>
  </div>
</template>

<script setup>
import { reactive } from 'vue'

// Reactive object
const form = reactive({
  name: '',
  email: '',
  preferences: {
    theme: 'dark',
    notifications: true
  }
})

// Reactive array
const items = reactive([
  { id: 1, name: 'Item 1', price: 10 },
  { id: 2, name: 'Item 2', price: 20 }
])

const submitForm = () => {
  console.log('Form data:', form)
}

const addItem = () => {
  items.push({
    id: items.length + 1,
    name: `Item ${items.length + 1}`,
    price: Math.floor(Math.random() * 100)
  })
}

// Direct access without .value
form.name = 'John' // Triggers reactivity
items[0].price = 15 // Triggers reactivity
</script>
```

#### `shallowRef()` and `shallowReactive()`
Create shallow reactive references that only track the root level, not nested properties.

**Real-world usage:** Performance optimization for large objects, external library integration, and when you control updates manually.

```vue
<template>
  <div>
    <!-- shallowRef example -->
    <p>Shallow count: {{ shallowCount }}</p>
    <p>Deep object: {{ deepObject.nested.value }}</p>
    <button @click="updateShallow">Update Shallow</button>
    <button @click="updateDeep">Update Deep (won't react)</button>
    
    <!-- shallowReactive example -->
    <p>User: {{ shallowUser.name }}</p>
    <p>Nested: {{ shallowUser.profile.bio }}</p>
    <button @click="updateUser">Update User</button>
    <button @click="updateProfile">Update Profile (won't react)</button>
  </div>
</template>

<script setup>
import { shallowRef, shallowReactive, triggerRef } from 'vue'

// Shallow ref - only root level is reactive
const shallowCount = shallowRef(0)
const deepObject = shallowRef({
  nested: { value: 'initial' }
})

// Shallow reactive - only first level properties are reactive
const shallowUser = shallowReactive({
  name: 'John',
  age: 30,
  profile: {
    bio: 'Developer',
    avatar: 'avatar.jpg'
  }
})

const updateShallow = () => {
  shallowCount.value++ // Triggers reactivity
}

const updateDeep = () => {
  deepObject.value.nested.value = 'updated' // Won't trigger reactivity
  triggerRef(deepObject) // Manual trigger needed
}

const updateUser = () => {
  shallowUser.name = 'Jane' // Triggers reactivity
}

const updateProfile = () => {
  shallowUser.profile.bio = 'Designer' // Won't trigger reactivity
  // Need to replace entire object
  shallowUser.profile = { ...shallowUser.profile, bio: 'Designer' }
}
</script>
```

#### `readonly()` and `shallowReadonly()`
Create immutable versions of reactive objects to prevent accidental mutations.

**Real-world usage:** Prop validation, shared state protection, API response data, and enforcing data flow patterns.

```vue
<template>
  <div>
    <p>Original: {{ original.count }}</p>
    <p>Readonly: {{ readonlyData.count }}</p>
    <p>Shallow readonly: {{ shallowReadonlyData.user.name }}</p>
    
    <button @click="updateOriginal">Update Original</button>
    <button @click="tryUpdateReadonly">Try Update Readonly</button>
  </div>
</template>

<script setup>
import { reactive, readonly, shallowReadonly } from 'vue'

const original = reactive({
  count: 0,
  user: { name: 'John' }
})

// Deep readonly - all levels are immutable
const readonlyData = readonly(original)

// Shallow readonly - only first level is immutable
const shallowReadonlyData = shallowReadonly({
  count: 0,
  user: { name: 'John' }
})

const updateOriginal = () => {
  original.count++ // Works and updates readonly version too
}

const tryUpdateReadonly = () => {
  try {
    readonlyData.count++ // Will warn in development
  } catch (error) {
    console.warn('Cannot update readonly data')
  }
  
  // This works because shallow readonly only protects first level
  shallowReadonlyData.user.name = 'Jane'
}
</script>
```

#### `toRef()` and `toRefs()`
Create reactive references from reactive object properties while maintaining reactivity connection.

**Real-world usage:** Destructuring reactive objects, passing specific properties to composables, and maintaining reactivity in component props.

```vue
<template>
  <div>
    <!-- Using individual refs -->
    <p>Name: {{ name }}</p>
    <p>Age: {{ age }}</p>
    <p>Email: {{ email }}</p>
    
    <!-- Using specific property ref -->
    <p>Count: {{ countRef }}</p>
    
    <button @click="updateUser">Update User</button>
    <button @click="updateCount">Update Count</button>
  </div>
</template>

<script setup>
import { reactive, toRef, toRefs } from 'vue'

const user = reactive({
  name: 'John',
  age: 30,
  email: 'john@example.com'
})

const state = reactive({
  count: 0,
  items: []
})

// Destructure with toRefs - maintains reactivity
const { name, age, email } = toRefs(user)

// Single property ref with toRef
const countRef = toRef(state, 'count')

const updateUser = () => {
  user.name = 'Jane' // Updates name ref automatically
  user.age = 25     // Updates age ref automatically
}

const updateCount = () => {
  countRef.value++ // Updates state.count
  // OR
  state.count++ // Also updates countRef
}

// Useful for composables
function useUserData() {
  return {
    ...toRefs(user), // Spread individual reactive refs
    updateName: (newName) => user.name = newName
  }
}
</script>
```

#### `unref()` and `isRef()`
Utility functions to work with refs and check ref types safely.

**Real-world usage:** Composable functions, type checking, unwrapping values, and handling mixed ref/non-ref parameters.

```vue
<template>
  <div>
    <p>Value 1: {{ value1 }}</p>
    <p>Value 2: {{ value2 }}</p>
    <p>Unwrapped: {{ unwrappedValue }}</p>
    <button @click="processValues">Process Values</button>
  </div>
</template>

<script setup>
import { ref, unref, isRef, computed } from 'vue'

const value1 = ref(10)
const value2 = 20 // Not a ref
const unwrappedValue = ref(0)

// Function that accepts refs or regular values
function processValue(val) {
  // Check if it's a ref
  if (isRef(val)) {
    console.log('This is a ref:', val.value)
    return val.value * 2
  } else {
    console.log('This is a regular value:', val)
    return val * 2
  }
}

// Using unref - always gets the value
function safeProcessValue(val) {
  const actualValue = unref(val) // Works with refs and regular values
  return actualValue * 3
}

const processValues = () => {
  // Both work the same way
  console.log('Processed ref:', processValue(value1))
  console.log('Processed value:', processValue(value2))
  
  // unref simplifies handling
  unwrappedValue.value = safeProcessValue(value1) + safeProcessValue(value2)
}

// Composable example
function useFlexibleCounter(initialValue) {
  // Accept ref or regular value
  const count = isRef(initialValue) ? initialValue : ref(initialValue)
  
  const increment = () => count.value++
  const decrement = () => count.value--
  
  return { count, increment, decrement }
}

// Usage
const { count: counter1 } = useFlexibleCounter(ref(5))
const { count: counter2 } = useFlexibleCounter(10)
</script>
```

#### `markRaw()`
Mark an object as non-reactive to prevent Vue from making it reactive.

**Real-world usage:** Third-party library instances, large datasets, performance optimization, and objects that shouldn't trigger updates.

```vue
<template>
  <div>
    <p>Regular object updates: {{ regularObj.count }}</p>
    <p>Raw object count: {{ rawObj.count }}</p>
    <p>Chart instance: {{ chartInstance ? 'Loaded' : 'Loading' }}</p>
    
    <button @click="updateObjects">Update Objects</button>
    <button @click="initChart">Initialize Chart</button>
  </div>
</template>

<script setup>
import { reactive, markRaw, ref } from 'vue'

// Regular reactive object
const regularObj = reactive({ count: 0 })

// Non-reactive object with markRaw
const rawObj = reactive({
  count: 0,
  // Large data that shouldn't be reactive
  heavyData: markRaw({
    largeArray: new Array(10000).fill(0),
    computedValues: {}
  })
})

const chartInstance = ref(null)

const updateObjects = () => {
  regularObj.count++ // Triggers reactivity
  rawObj.count++     // Triggers reactivity
  
  // This won't trigger reactivity (good for performance)
  rawObj.heavyData.largeArray[0] = Math.random()
}

const initChart = () => {
  // Third-party library instance marked as raw
  const chart = markRaw({
    // Imagine this is a Chart.js or D3 instance
    render: () => console.log('Rendering chart'),
    destroy: () => console.log('Destroying chart'),
    data: [1, 2, 3, 4, 5]
  })
  
  chartInstance.value = chart
}

// Performance optimization for large static data
const staticConfig = markRaw({
  apiEndpoints: {
    users: '/api/users',
    posts: '/api/posts'
  },
  constants: {
    MAX_FILE_SIZE: 5000000,
    ALLOWED_TYPES: ['jpg', 'png', 'gif']
  }
})
</script>
```

#### Reactivity Transform (Experimental)
Compile-time feature that allows using reactive variables without `.value` syntax.

**Real-world usage:** Cleaner syntax in development, reduced boilerplate, but experimental and may change.

```vue
<template>
  <div>
    <!-- Direct usage without .value -->
    <p>Count: {{ count }}</p>
    <p>Message: {{ message }}</p>
    <button @click="increment">Increment</button>
  </div>
</template>

<script setup>
// Note: This is experimental and requires special build configuration
import { $ref, $computed } from 'vue/macros'

// Using $ref instead of ref
let count = $ref(0)
let message = $ref('Hello')

// Using $computed instead of computed
const doubled = $computed(() => count * 2)

// Direct assignment without .value
const increment = () => {
  count++ // No need for count.value++
  message = `Count is ${count}` // No need for message.value =
}

// Compiler transforms this to:
// const count = ref(0)
// const message = ref('Hello')
// const increment = () => {
//   count.value++
//   message.value = `Count is ${count.value}`
// }
</script>
```

```javascript
// vite.config.js - Required configuration
export default {
  plugins: [
    vue({
      reactivityTransform: true
    })
  ]
}
```

---

### 7. **Computed Properties**

#### Basic Computed Properties
Reactive properties that automatically recalculate when their dependencies change, returning cached results for better performance.

**Real-world usage:** Data formatting, filtered lists, calculated totals, derived state, and any value that depends on other reactive data.

```vue
<template>
  <div>
    <input v-model="firstName" placeholder="First name" />
    <input v-model="lastName" placeholder="Last name" />
    <p>Full name: {{ fullName }}</p>
    
    <ul>
      <li v-for="item in expensiveItems" :key="item.id">
        {{ item.name }} - ${{ item.price }}
      </li>
    </ul>
    <p>Total: ${{ totalPrice }}</p>
    <p>Average: ${{ averagePrice }}</p>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const firstName = ref('John')
const lastName = ref('Doe')

// Basic computed property
const fullName = computed(() => {
  return `${firstName.value} ${lastName.value}`
})

const items = ref([
  { id: 1, name: 'Laptop', price: 1000 },
  { id: 2, name: 'Phone', price: 500 },
  { id: 3, name: 'Tablet', price: 300 }
])

// Computed filter
const expensiveItems = computed(() => {
  return items.value.filter(item => item.price > 400)
})

// Computed calculations
const totalPrice = computed(() => {
  return items.value.reduce((sum, item) => sum + item.price, 0)
})

const averagePrice = computed(() => {
  return items.value.length ? totalPrice.value / items.value.length : 0
})
</script>
```

#### Computed with Getter and Setter
Create computed properties that can both read and write values, useful for two-way data binding with transformations.

**Real-world usage:** Form inputs with formatting, unit conversions, data synchronization, and complex v-model implementations.

```vue
<template>
  <div>
    <!-- Two-way computed binding -->
    <input v-model="fullName" placeholder="Full name" />
    <p>First: {{ firstName }}</p>
    <p>Last: {{ lastName }}</p>
    
    <!-- Temperature converter -->
    <label>
      Celsius: <input v-model.number="celsius" type="number" />
    </label>
    <label>
      Fahrenheit: <input v-model.number="fahrenheit" type="number" />
    </label>
    
    <!-- Formatted price -->
    <input v-model="formattedPrice" placeholder="$0.00" />
    <p>Raw price: {{ price }}</p>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const firstName = ref('John')
const lastName = ref('Doe')

// Computed with getter and setter
const fullName = computed({
  get() {
    return `${firstName.value} ${lastName.value}`
  },
  set(newValue) {
    const parts = newValue.split(' ')
    firstName.value = parts[0] || ''
    lastName.value = parts.slice(1).join(' ') || ''
  }
})

const temperature = ref(0)

const celsius = computed({
  get() {
    return temperature.value
  },
  set(value) {
    temperature.value = value
  }
})

const fahrenheit = computed({
  get() {
    return (temperature.value * 9/5) + 32
  },
  set(value) {
    temperature.value = (value - 32) * 5/9
  }
})

const price = ref(0)

const formattedPrice = computed({
  get() {
    return `$${price.value.toFixed(2)}`
  },
  set(value) {
    const numericValue = parseFloat(value.replace(/[$,]/g, ''))
    price.value = isNaN(numericValue) ? 0 : numericValue
  }
})
</script>
```

#### Computed Caching
Computed properties cache their results and only recalculate when reactive dependencies change, unlike methods that run every time.

**Real-world usage:** Expensive calculations, API data processing, complex filtering, and performance optimization.

```vue
<template>
  <div>
    <p>Computed result: {{ expensiveComputed }}</p>
    <p>Method result: {{ expensiveMethod() }}</p>
    <p>Calls - Computed: {{ computedCalls }}, Method: {{ methodCalls }}</p>
    
    <button @click="triggerRerender">Trigger Re-render</button>
    <button @click="updateDependency">Update Dependency</button>
    
    <!-- Multiple usage shows caching benefit -->
    <div>
      <span>{{ filteredUsers.length }} users</span>
      <ul>
        <li v-for="user in filteredUsers" :key="user.id">
          {{ user.name }}
        </li>
      </ul>
      <p>Active users: {{ filteredUsers.length }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const dependency = ref(1)
const computedCalls = ref(0)
const methodCalls = ref(0)
const rerenderTrigger = ref(0)

// Cached computed property
const expensiveComputed = computed(() => {
  computedCalls.value++
  console.log('Computed recalculated')
  
  // Simulate expensive operation
  let result = 0
  for (let i = 0; i < 1000000; i++) {
    result += dependency.value
  }
  return result
})

// Method runs every time
const expensiveMethod = () => {
  methodCalls.value++
  console.log('Method called')
  
  // Same expensive operation
  let result = 0
  for (let i = 0; i < 1000000; i++) {
    result += dependency.value
  }
  return result
}

const users = ref([
  { id: 1, name: 'John', active: true },
  { id: 2, name: 'Jane', active: false },
  { id: 3, name: 'Bob', active: true }
])

// Cached filtering - only recalculates when users change
const filteredUsers = computed(() => {
  console.log('Filtering users')
  return users.value.filter(user => user.active)
})

const triggerRerender = () => {
  rerenderTrigger.value++
}

const updateDependency = () => {
  dependency.value++
}
</script>
```

#### Computed vs Methods
Understanding when to use computed properties versus methods for optimal performance and reactivity.

**Real-world usage:** Choose computed for derived state, choose methods for actions and event handlers.

```vue
<template>
  <div>
    <!-- Computed: cached, reactive, declarative -->
    <p>Filtered products (computed): {{ filteredProducts.length }}</p>
    <p>Total value (computed): ${{ totalValue }}</p>
    
    <!-- Methods: not cached, imperative -->
    <button @click="addProduct">Add Product</button>
    <button @click="clearProducts">Clear All</button>
    
    <!-- When to use each -->
    <p>Current time (method): {{ getCurrentTime() }}</p>
    <p>Formatted date (computed): {{ formattedDate }}</p>
    
    <input v-model="searchTerm" placeholder="Search products" />
    <div v-for="product in searchResults" :key="product.id">
      {{ product.name }} - ${{ product.price }}
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const products = ref([
  { id: 1, name: 'Laptop', price: 1000, category: 'electronics' },
  { id: 2, name: 'Book', price: 20, category: 'education' }
])

const searchTerm = ref('')
const selectedCategory = ref('electronics')
const currentDate = ref(new Date())

// ✅ Use computed for: derived state, caching, reactivity
const filteredProducts = computed(() => {
  return products.value.filter(p => p.category === selectedCategory.value)
})

const totalValue = computed(() => {
  return filteredProducts.value.reduce((sum, p) => sum + p.price, 0)
})

const searchResults = computed(() => {
  if (!searchTerm.value) return products.value
  return products.value.filter(p => 
    p.name.toLowerCase().includes(searchTerm.value.toLowerCase())
  )
})

const formattedDate = computed(() => {
  return currentDate.value.toLocaleDateString()
})

// ✅ Use methods for: actions, events, side effects
const addProduct = () => {
  const newProduct = {
    id: Date.now(),
    name: `Product ${products.value.length + 1}`,
    price: Math.floor(Math.random() * 1000),
    category: 'electronics'
  }
  products.value.push(newProduct)
}

const clearProducts = () => {
  products.value = []
}

// ✅ Use methods for: non-reactive, always fresh data
const getCurrentTime = () => {
  return new Date().toLocaleTimeString()
}

// ❌ Don't use computed for: side effects, async operations
// const badComputed = computed(() => {
//   fetch('/api/data') // ❌ Side effect
//   return someValue
// })
</script>
```

#### Computed vs Watchers
Choose between computed properties and watchers based on whether you need derived values or side effects.

**Real-world usage:** Use computed for transformations, use watchers for API calls, logging, and external system updates.

```vue
<template>
  <div>
    <input v-model="searchQuery" placeholder="Search users" />
    <p>Search results: {{ searchResults.length }} found</p>
    
    <input v-model.number="userId" type="number" placeholder="User ID" />
    <div v-if="userProfile">
      <h3>{{ userProfile.name }}</h3>
      <p>{{ userProfile.email }}</p>
    </div>
    
    <p>Total price: ${{ totalPrice }}</p>
    <p>Savings needed: ${{ savingsNeeded }}</p>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const searchQuery = ref('')
const users = ref([
  { id: 1, name: 'John', email: 'john@example.com' },
  { id: 2, name: 'Jane', email: 'jane@example.com' }
])

// ✅ Computed: for deriving/transforming data
const searchResults = computed(() => {
  if (!searchQuery.value) return users.value
  return users.value.filter(user => 
    user.name.toLowerCase().includes(searchQuery.value.toLowerCase())
  )
})

const cartItems = ref([
  { price: 100 }, { price: 50 }
])
const budget = ref(200)

const totalPrice = computed(() => {
  return cartItems.value.reduce((sum, item) => sum + item.price, 0)
})

const savingsNeeded = computed(() => {
  const needed = totalPrice.value - budget.value
  return needed > 0 ? needed : 0
})

// ✅ Watcher: for side effects and async operations
const userId = ref(1)
const userProfile = ref(null)

watch(userId, async (newId) => {
  if (newId) {
    try {
      // Side effect: API call
      userProfile.value = await fetchUserProfile(newId)
    } catch (error) {
      console.error('Failed to fetch user:', error)
      userProfile.value = null
    }
  }
}, { immediate: true })

// ✅ Watcher: for logging, analytics
watch(searchQuery, (newQuery, oldQuery) => {
  if (newQuery) {
    console.log(`User searched for: ${newQuery}`)
    // Analytics tracking
    trackSearchEvent(newQuery)
  }
})

// ✅ Watcher: for complex side effects
watch(totalPrice, (newTotal, oldTotal) => {
  if (newTotal > budget.value) {
    alert('Budget exceeded!')
  }
  
  // Save to localStorage
  localStorage.setItem('cartTotal', newTotal.toString())
})

// Mock functions
async function fetchUserProfile(id) {
  // Simulate API call
  await new Promise(resolve => setTimeout(resolve, 500))
  return users.value.find(user => user.id === id)
}

function trackSearchEvent(query) {
  // Analytics tracking
  console.log('Analytics: search performed', query)
}
</script>
```

#### Debugging Computed Properties
Techniques and tools for debugging computed property issues and performance problems.

**Real-world usage:** Troubleshoot infinite loops, identify performance bottlenecks, and understand dependency tracking.

```vue
<template>
  <div>
    <input v-model="input" placeholder="Type something" />
    <p>Processed: {{ processedInput }}</p>
    <p>Expensive result: {{ expensiveComputed }}</p>
    
    <div>
      Debug info:
      <pre>{{ debugInfo }}</pre>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watchEffect } from 'vue'

const input = ref('')
const computedCallCount = ref(0)
const lastComputedTime = ref(null)

// Debugging computed dependencies
const processedInput = computed(() => {
  console.log('🔄 processedInput computed called')
  console.log('Dependencies:', { input: input.value })
  
  return input.value.toUpperCase().trim()
})

// Debugging expensive computations
const expensiveComputed = computed(() => {
  const startTime = performance.now()
  computedCallCount.value++
  
  console.log('🔄 expensiveComputed started')
  
  // Simulate expensive operation
  let result = 0
  for (let i = 0; i < 1000000; i++) {
    result += input.value.length
  }
  
  const endTime = performance.now()
  lastComputedTime.value = endTime - startTime
  
  console.log(`⏱️ expensiveComputed took ${lastComputedTime.value}ms`)
  
  return result
})

// Debug info computed
const debugInfo = computed(() => {
  return {
    inputLength: input.value.length,
    computedCalls: computedCallCount.value,
    lastExecutionTime: lastComputedTime.value,
    timestamp: new Date().toISOString()
  }
})

// Watch for debugging infinite loops
watchEffect(() => {
  console.log('👀 Watching input changes:', input.value)
})

// Debug computed that might cause issues
const problematicComputed = computed(() => {
  // ❌ This would cause infinite loop in some cases
  // someReactiveValue.value++
  
  // ✅ Proper computed - only reads values
  return input.value.split('').reverse().join('')
})

// Development-only debugging
if (import.meta.env.DEV) {
  // Log all computed recalculations
  watchEffect(() => {
    console.group('🐛 Computed Debug')
    console.log('Input:', input.value)
    console.log('Processed:', processedInput.value)
    console.log('Expensive result:', expensiveComputed.value)
    console.groupEnd()
  })
}
</script>
```

---

### 8. **Watchers**

#### `watch()` API
Explicitly watch reactive sources and execute callbacks when they change, ideal for side effects and async operations.

**Real-world usage:** API calls when data changes, form validation, localStorage updates, and external library synchronization.

```vue
<template>
  <div>
    <input v-model="searchTerm" placeholder="Search..." />
    <input v-model.number="userId" type="number" placeholder="User ID" />
    
    <div v-if="loading">Loading...</div>
    <div v-else-if="searchResults.length">
      <div v-for="result in searchResults" :key="result.id">
        {{ result.name }}
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const searchTerm = ref('')
const userId = ref(1)
const searchResults = ref([])
const loading = ref(false)

// Basic watch syntax
watch(searchTerm, (newValue, oldValue) => {
  console.log(`Search changed from "${oldValue}" to "${newValue}"`)
  
  if (newValue) {
    performSearch(newValue)
  } else {
    searchResults.value = []
  }
})

// Watch with options
watch(userId, async (newId) => {
  if (newId) {
    loading.value = true
    try {
      const user = await fetchUser(newId)
      console.log('User fetched:', user)
    } catch (error) {
      console.error('Failed to fetch user:', error)
    } finally {
      loading.value = false
    }
  }
}, {
  immediate: true // Run immediately with current value
})

async function performSearch(term) {
  loading.value = true
  // Simulate API call
  setTimeout(() => {
    searchResults.value = [
      { id: 1, name: `Result for ${term}` }
    ]
    loading.value = false
  }, 500)
}

async function fetchUser(id) {
  // Simulate API call
  return new Promise(resolve => {
    setTimeout(() => resolve({ id, name: `User ${id}` }), 300)
  })
}
</script>
```

#### `watchEffect()`
Automatically tracks reactive dependencies and runs when any of them change, no need to specify what to watch.

**Real-world usage:** Complex side effects with multiple dependencies, automatic dependency tracking, and reactive cleanup.

```vue
<template>
  <div>
    <input v-model="title" placeholder="Document title" />
    <textarea v-model="content" placeholder="Content"></textarea>
    
    <p>Auto-saved: {{ lastSaved || 'Never' }}</p>
    <p>Word count: {{ wordCount }}</p>
  </div>
</template>

<script setup>
import { ref, watchEffect } from 'vue'

const title = ref('')
const content = ref('')
const lastSaved = ref(null)
const wordCount = ref(0)

// Automatically tracks title and content
watchEffect(() => {
  // Auto-save document when title or content changes
  if (title.value || content.value) {
    console.log('Auto-saving document...')
    saveDocument({
      title: title.value,
      content: content.value,
      timestamp: new Date()
    })
    lastSaved.value = new Date().toLocaleTimeString()
  }
})

// Multiple dependencies tracked automatically
watchEffect(() => {
  // Update word count
  wordCount.value = content.value.split(/\s+/).filter(word => word.length > 0).length
  
  // Update document title in browser
  document.title = title.value || 'Untitled Document'
  
  // Log analytics
  if (content.value.length > 0) {
    console.log('User typing activity', {
      titleLength: title.value.length,
      contentLength: content.value.length,
      wordCount: wordCount.value
    })
  }
})

function saveDocument(doc) {
  localStorage.setItem('document', JSON.stringify(doc))
}
</script>
```

#### Watching Single Source
Watch individual reactive references, computed properties, or getter functions.

**Real-world usage:** Form field validation, user preference changes, state transitions, and specific property monitoring.

```vue
<template>
  <div>
    <input v-model="email" placeholder="Email" />
    <span v-if="emailError" class="error">{{ emailError }}</span>
    
    <select v-model="theme">
      <option>light</option>
      <option>dark</option>
    </select>
    
    <p>Status: {{ userStatus }}</p>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const email = ref('')
const emailError = ref('')
const theme = ref('light')
const isOnline = ref(true)
const lastActivity = ref(new Date())

const userStatus = computed(() => {
  return isOnline.value ? 'Online' : 'Offline'
})

// Watch ref
watch(email, (newEmail) => {
  validateEmail(newEmail)
})

// Watch computed
watch(userStatus, (newStatus) => {
  console.log('User status changed to:', newStatus)
  updateUserActivity(newStatus)
})

// Watch getter function
watch(
  () => theme.value,
  (newTheme) => {
    document.body.className = `theme-${newTheme}`
    localStorage.setItem('theme', newTheme)
  }
)

// Watch object property
const user = ref({ name: 'John', settings: { notifications: true } })

watch(
  () => user.value.settings.notifications,
  (enabled) => {
    console.log('Notifications:', enabled ? 'enabled' : 'disabled')
  }
)

function validateEmail(email) {
  if (!email) {
    emailError.value = ''
  } else if (!/\S+@\S+\.\S+/.test(email)) {
    emailError.value = 'Invalid email format'
  } else {
    emailError.value = ''
  }
}

function updateUserActivity(status) {
  lastActivity.value = new Date()
}
</script>
```

#### Watching Multiple Sources
Watch arrays of reactive sources simultaneously, useful for coordinating multiple state changes.

**Real-world usage:** Form validation with multiple fields, coordinated animations, multi-factor conditions, and batch operations.

```vue
<template>
  <div>
    <input v-model="firstName" placeholder="First name" />
    <input v-model="lastName" placeholder="Last name" />
    <input v-model="email" placeholder="Email" />
    
    <p>Form valid: {{ isFormValid }}</p>
    <p>Completion: {{ completionPercentage }}%</p>
    
    <div>Coordinates: {{ position.x }}, {{ position.y }}</div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const firstName = ref('')
const lastName = ref('')
const email = ref('')
const position = ref({ x: 0, y: 0 })

const isFormValid = ref(false)
const completionPercentage = ref(0)

// Watch multiple sources with array
watch(
  [firstName, lastName, email],
  ([newFirst, newLast, newEmail], [oldFirst, oldLast, oldEmail]) => {
    console.log('Form fields changed:', {
      firstName: { old: oldFirst, new: newFirst },
      lastName: { old: oldLast, new: newLast },
      email: { old: oldEmail, new: newEmail }
    })
    
    validateForm(newFirst, newLast, newEmail)
    updateProgress(newFirst, newLast, newEmail)
  }
)

// Watch multiple computed/getter functions
watch(
  [
    () => position.value.x,
    () => position.value.y,
    () => firstName.value.length
  ],
  ([x, y, nameLength]) => {
    console.log('Coordinates and name length:', { x, y, nameLength })
    
    // Coordinate-based logic
    if (x > 100 && y > 100) {
      console.log('Moved to significant position')
    }
  }
)

// Watch reactive objects
const userPrefs = ref({ theme: 'light', lang: 'en' })
const appState = ref({ loading: false, error: null })

watch(
  [userPrefs, appState],
  ([newPrefs, newState], [oldPrefs, oldState]) => {
    // Save preferences when either changes
    localStorage.setItem('userPrefs', JSON.stringify(newPrefs))
    localStorage.setItem('appState', JSON.stringify(newState))
    
    console.log('State updated:', { preferences: newPrefs, state: newState })
  },
  { deep: true }
)

function validateForm(first, last, email) {
  const hasFirstName = first.length > 0
  const hasLastName = last.length > 0
  const hasValidEmail = /\S+@\S+\.\S+/.test(email)
  
  isFormValid.value = hasFirstName && hasLastName && hasValidEmail
}

function updateProgress(first, last, email) {
  let progress = 0
  if (first.length > 0) progress += 33
  if (last.length > 0) progress += 33
  if (/\S+@\S+\.\S+/.test(email)) progress += 34
  
  completionPercentage.value = progress
}

// Simulate position updates
setInterval(() => {
  position.value.x += Math.random() * 10 - 5
  position.value.y += Math.random() * 10 - 5
}, 2000)
</script>
```

#### Deep Watching
Watch nested properties in objects and arrays for changes at any level.

**Real-world usage:** Complex form objects, nested settings, shopping carts, and hierarchical data structures.

```vue
<template>
  <div>
    <h3>User Profile</h3>
    <input v-model="user.name" placeholder="Name" />
    <input v-model="user.email" placeholder="Email" />
    
    <h4>Address</h4>
    <input v-model="user.address.street" placeholder="Street" />
    <input v-model="user.address.city" placeholder="City" />
    
    <h4>Shopping Cart</h4>
    <div v-for="(item, index) in cart.items" :key="index">
      <input v-model="item.name" placeholder="Item name" />
      <input v-model.number="item.quantity" type="number" />
      <input v-model.number="item.price" type="number" step="0.01" />
    </div>
    <button @click="addItem">Add Item</button>
    
    <p>Cart Total: ${{ cart.total }}</p>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const user = ref({
  name: 'John',
  email: 'john@example.com',
  address: {
    street: '123 Main St',
    city: 'Anytown',
    country: 'USA'
  },
  preferences: {
    theme: 'light',
    notifications: {
      email: true,
      push: false
    }
  }
})

const cart = ref({
  items: [
    { name: 'Laptop', quantity: 1, price: 999.99 }
  ],
  total: 0
})

// Deep watch object - detects nested changes
watch(
  user,
  (newUser, oldUser) => {
    console.log('User profile changed (deep)')
    
    // Auto-save profile
    saveUserProfile(newUser)
    
    // Validate profile completeness
    validateProfile(newUser)
  },
  { deep: true }
)

// Deep watch array with objects
watch(
  () => cart.value.items,
  (newItems) => {
    console.log('Cart items changed')
    
    // Recalculate total
    cart.value.total = newItems.reduce((sum, item) => {
      return sum + (item.quantity * item.price)
    }, 0)
    
    // Save cart to localStorage
    localStorage.setItem('cart', JSON.stringify(cart.value))
  },
  { deep: true }
)

// Watch specific nested property
watch(
  () => user.value.preferences.theme,
  (newTheme) => {
    console.log('Theme changed to:', newTheme)
    document.body.className = `theme-${newTheme}`
  }
)

// Deep watch with immediate
watch(
  () => user.value.address,
  (newAddress) => {
    console.log('Address updated:', newAddress)
    
    // Validate address format
    if (newAddress.street && newAddress.city) {
      console.log('Address is complete')
    }
    
    // Update shipping options based on location
    updateShippingOptions(newAddress)
  },
  { deep: true, immediate: true }
)

function saveUserProfile(profile) {
  localStorage.setItem('userProfile', JSON.stringify(profile))
}

function validateProfile(profile) {
  const isComplete = profile.name && profile.email && 
                    profile.address.street && profile.address.city
  console.log('Profile complete:', isComplete)
}

function updateShippingOptions(address) {
  console.log('Updating shipping for:', address.city)
}

function addItem() {
  cart.value.items.push({
    name: '',
    quantity: 1,
    price: 0
  })
}
</script>
```

#### Immediate Execution
Execute watcher callback immediately with the current value, useful for initialization logic.

**Real-world usage:** Initial data loading, setting up external libraries, and ensuring consistent state on component mount.

```vue
<template>
  <div>
    <select v-model="selectedLanguage">
      <option v-for="lang in languages" :key="lang.code" :value="lang.code">
        {{ lang.name }}
      </option>
    </select>
    
    <div>{{ t('welcome') }}</div>
    <div>{{ t('current_user') }}: {{ username }}</div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const selectedLanguage = ref('en')
const username = ref('John')
const currentTranslations = ref({})

const languages = [
  { code: 'en', name: 'English' },
  { code: 'es', name: 'Español' },
  { code: 'fr', name: 'Français' }
]

// Immediate execution - runs on mount and on changes
watch(
  selectedLanguage,
  async (newLang) => {
    console.log('Loading translations for:', newLang)
    
    try {
      // Load translations immediately
      currentTranslations.value = await loadTranslations(newLang)
      
      // Update page language
      document.documentElement.lang = newLang
      
      // Save user preference
      localStorage.setItem('preferredLanguage', newLang)
      
      console.log('Translations loaded for:', newLang)
    } catch (error) {
      console.error('Failed to load translations:', error)
    }
  },
  { immediate: true } // Runs immediately with current value
)

// Multiple watchers with immediate
watch(
  username,
  (newUsername) => {
    console.log('Username changed to:', newUsername)
    
    // Update user session
    updateUserSession(newUsername)
    
    // Load user preferences
    loadUserPreferences(newUsername)
  },
  { immediate: true }
)

// Immediate with complex logic
const apiEndpoint = ref('/api/v1')
const authToken = ref('')

watch(
  [apiEndpoint, authToken],
  ([endpoint, token]) => {
    console.log('API configuration changed')
    
    // Initialize API client
    if (endpoint && token) {
      initializeApiClient(endpoint, token)
    }
    
    // Set up authentication headers
    setupAuthHeaders(token)
  },
  { immediate: true }
)

function t(key) {
  return currentTranslations.value[key] || key
}

async function loadTranslations(lang) {
  // Simulate API call
  const translations = {
    en: { welcome: 'Welcome', current_user: 'Current User' },
    es: { welcome: 'Bienvenido', current_user: 'Usuario Actual' },
    fr: { welcome: 'Bienvenue', current_user: 'Utilisateur Actuel' }
  }
  
  return new Promise(resolve => {
    setTimeout(() => resolve(translations[lang] || translations.en), 300)
  })
}

function updateUserSession(username) {
  console.log('Updating session for:', username)
}

function loadUserPreferences(username) {
  console.log('Loading preferences for:', username)
}

function initializeApiClient(endpoint, token) {
  console.log('Initializing API client:', { endpoint, token })
}

function setupAuthHeaders(token) {
  console.log('Setting up auth headers with token:', token)
}
</script>
```

#### Callback Flush Timing
Control when watcher callbacks execute in relation to component updates and DOM changes.

**Real-world usage:** DOM manipulation after updates, performance optimization, and coordinating with external libraries.

```vue
<template>
  <div>
    <input v-model="message" placeholder="Type to see flush timing" />
    <div ref="output">{{ message }}</div>
    
    <button @click="count++">Count: {{ count }}</button>
    <div ref="countDisplay">{{ count }}</div>
    
    <div ref="scrollContainer" style="height: 200px; overflow-y: auto;">
      <div v-for="n in items" :key="n" style="height: 50px;">
        Item {{ n }}
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, nextTick } from 'vue'

const message = ref('')
const count = ref(0)
const items = ref(Array.from({ length: 20 }, (_, i) => i + 1))
const output = ref(null)
const countDisplay = ref(null)
const scrollContainer = ref(null)

// Default flush: 'pre' - before component updates
watch(
  message,
  (newMessage) => {
    console.log('PRE flush - before DOM update')
    console.log('DOM content:', output.value?.textContent)
    // DOM still shows old value
  }
)

// Post flush - after DOM updates
watch(
  message,
  (newMessage) => {
    console.log('POST flush - after DOM update')
    console.log('DOM content:', output.value?.textContent)
    // DOM shows new value
    
    // Safe to do DOM measurements
    if (output.value) {
      const rect = output.value.getBoundingClientRect()
      console.log('Element dimensions:', rect.width, rect.height)
    }
  },
  { flush: 'post' }
)

// Sync flush - synchronous, before any updates
watch(
  count,
  (newCount) => {
    console.log('SYNC flush - immediate execution')
    console.log('Count display DOM:', countDisplay.value?.textContent)
    // Runs immediately, before any updates
    
    // Use for urgent updates or preventing race conditions
    if (newCount > 10) {
      console.log('Count is getting high!')
    }
  },
  { flush: 'sync' }
)

// Post flush for DOM manipulation
watch(
  items,
  async (newItems) => {
    console.log('Items changed, updating scroll')
    
    // Wait for DOM update
    await nextTick()
    
    // Now safe to manipulate DOM
    if (scrollContainer.value) {
      scrollContainer.value.scrollTop = scrollContainer.value.scrollHeight
    }
  },
  { flush: 'post' }
)

// Practical example: Auto-resize textarea
const textareaContent = ref('')
const textarea = ref(null)

watch(
  textareaContent,
  () => {
    // Use post flush to ensure DOM is updated
    nextTick(() => {
      if (textarea.value) {
        textarea.value.style.height = 'auto'
        textarea.value.style.height = textarea.value.scrollHeight + 'px'
      }
    })
  },
  { flush: 'post' }
)

// Performance monitoring with different flush timings
watch(
  message,
  (newMessage) => {
    const start = performance.now()
    
    // Expensive operation
    for (let i = 0; i < 100000; i++) {
      Math.random()
    }
    
    const end = performance.now()
    console.log(`Operation took ${end - start}ms (flush: pre)`)
  }
)

watch(
  message,
  (newMessage) => {
    const start = performance.now()
    
    // Same expensive operation
    for (let i = 0; i < 100000; i++) {
      Math.random()
    }
    
    const end = performance.now()
    console.log(`Operation took ${end - start}ms (flush: post)`)
  },
  { flush: 'post' }
)
</script>
```

#### Stopping Watchers
Manually stop watchers when they're no longer needed to prevent memory leaks and unnecessary executions.

**Real-world usage:** Conditional watching, component cleanup, dynamic watcher management, and performance optimization.

```vue
<template>
  <div>
    <button @click="startWatching" :disabled="isWatching">Start Watching</button>
    <button @click="stopWatching" :disabled="!isWatching">Stop Watching</button>
    
    <input v-model="watchedValue" placeholder="This will be watched" />
    <p>Watch status: {{ isWatching ? 'Active' : 'Stopped' }}</p>
    
    <div>
      <h4>Conditional Watchers</h4>
      <input type="checkbox" v-model="enableLocationWatch" /> Watch Location
      <input type="checkbox" v-model="enablePerformanceWatch" /> Watch Performance
    </div>
    
    <button @click="cleanup">Cleanup All Watchers</button>
  </div>
</template>

<script setup>
import { ref, watch, watchEffect, onUnmounted } from 'vue'

const watchedValue = ref('')
const isWatching = ref(false)
const enableLocationWatch = ref(false)
const enablePerformanceWatch = ref(false)

// Store watcher stop functions
let currentWatcher = null
let locationWatcher = null
let performanceWatcher = null
const watchers = []

const startWatching = () => {
  if (currentWatcher) return
  
  currentWatcher = watch(
    watchedValue,
    (newValue, oldValue) => {
      console.log(`Value changed: ${oldValue} -> ${newValue}`)
    }
  )
  
  isWatching.value = true
  console.log('Watcher started')
}

const stopWatching = () => {
  if (currentWatcher) {
    currentWatcher() // Call the stop function
    currentWatcher = null
    isWatching.value = false
    console.log('Watcher stopped')
  }
}

// Conditional watchers
watch(
  enableLocationWatch,
  (enabled) => {
    if (enabled && !locationWatcher) {
      locationWatcher = watchEffect(() => {
        // Simulate location watching
        console.log('Watching location...')
        
        if (navigator.geolocation) {
          navigator.geolocation.getCurrentPosition(
            (position) => {
              console.log('Location updated:', position.coords)
            },
            (error) => {
              console.error('Location error:', error)
            }
          )
        }
      })
      
      console.log('Location watcher started')
    } else if (!enabled && locationWatcher) {
      locationWatcher()
      locationWatcher = null
      console.log('Location watcher stopped')
    }
  }
)

// Performance monitoring watcher
watch(
  enablePerformanceWatch,
  (enabled) => {
    if (enabled && !performanceWatcher) {
      performanceWatcher = watchEffect(() => {
        // Monitor performance metrics
        const start = performance.now()
        
        // Simulate work
        setTimeout(() => {
          const end = performance.now()
          console.log(`Performance check: ${end - start}ms`)
        }, 100)
      })
      
      console.log('Performance watcher started')
    } else if (!enabled && performanceWatcher) {
      performanceWatcher()
      performanceWatcher = null
      console.log('Performance watcher stopped')
    }
  }
)

// Automatically stop watchers based on conditions
const autoStopWatcher = watch(
  watchedValue,
  (newValue) => {
    console.log('Auto-stop watcher checking:', newValue)
    
    // Stop watching after specific condition
    if (newValue.includes('stop')) {
      console.log('Auto-stopping watcher due to "stop" keyword')
      autoStopWatcher() // Stop this watcher
    }
  }
)

// Store all watchers for batch cleanup
watchers.push(autoStopWatcher)

// Timed watcher that stops itself
const timedWatcher = watch(
  watchedValue,
  (newValue) => {
    console.log('Timed watcher:', newValue)
  }
)

// Stop timed watcher after 10 seconds
setTimeout(() => {
  if (timedWatcher) {
    timedWatcher()
    console.log('Timed watcher auto-stopped after 10 seconds')
  }
}, 10000)

const cleanup = () => {
  // Stop all active watchers
  if (currentWatcher) {
    currentWatcher()
    currentWatcher = null
    isWatching.value = false
  }
  
  if (locationWatcher) {
    locationWatcher()
    locationWatcher = null
    enableLocationWatch.value = false
  }
  
  if (performanceWatcher) {
    performanceWatcher()
    performanceWatcher = null
    enablePerformanceWatch.value = false
  }
  
  // Stop stored watchers
  watchers.forEach(stop => stop())
  watchers.length = 0
  
  console.log('All watchers cleaned up')
}

// Automatic cleanup on component unmount
onUnmounted(() => {
  cleanup()
  console.log('Component unmounted - watchers cleaned up')
})
</script>
```

#### Effect Cleanup
Clean up side effects in watchers to prevent memory leaks and resource accumulation.

**Real-world usage:** Event listeners, timers, subscriptions, WebSocket connections, and external library cleanup.

```vue
<template>
  <div>
    <input v-model="searchTerm" placeholder="Search with debounce" />
    <div>Search results: {{ searchResults.length }}</div>
    
    <input type="checkbox" v-model="enableWebSocket" /> Enable WebSocket
    <div v-if="wsStatus">WebSocket: {{ wsStatus }}</div>
    
    <button @click="startPolling">Start Polling</button>
    <button @click="stopPolling">Stop Polling</button>
    <div>Poll count: {{ pollCount }}</div>
  </div>
</template>

<script setup>
import { ref, watch, onUnmounted } from 'vue'

const searchTerm = ref('')
const searchResults = ref([])
const enableWebSocket = ref(false)
const wsStatus = ref('')
const pollCount = ref(0)
const isPolling = ref(false)

// Debounced search with cleanup
watch(
  searchTerm,
  (newTerm) => {
    console.log('Setting up search for:', newTerm)
    
    // Cleanup function will clear this timeout
    const timeoutId = setTimeout(() => {
      performSearch(newTerm)
    }, 500)
    
    // Return cleanup function
    return () => {
      console.log('Cleaning up search timeout')
      clearTimeout(timeoutId)
    }
  }
)

// WebSocket management with cleanup
watch(
  enableWebSocket,
  (enabled) => {
    if (enabled) {
      console.log('Connecting to WebSocket...')
      
      const ws = new WebSocket('wss://echo.websocket.org')
      
      ws.onopen = () => {
        wsStatus.value = 'Connected'
        console.log('WebSocket connected')
      }
      
      ws.onmessage = (event) => {
        console.log('WebSocket message:', event.data)
      }
      
      ws.onerror = (error) => {
        wsStatus.value = 'Error'
        console.error('WebSocket error:', error)
      }
      
      ws.onclose = () => {
        wsStatus.value = 'Disconnected'
        console.log('WebSocket disconnected')
      }
      
      // Return cleanup function
      return () => {
        console.log('Cleaning up WebSocket connection')
        if (ws.readyState === WebSocket.OPEN || ws.readyState === WebSocket.CONNECTING) {
          ws.close()
        }
        wsStatus.value = ''
      }
    }
  }
)

// Event listener cleanup
const mousePosition = ref({ x: 0, y: 0 })
const trackMouse = ref(false)

watch(
  trackMouse,
  (enabled) => {
    if (enabled) {
      console.log('Starting mouse tracking')
      
      const handleMouseMove = (event) => {
        mousePosition.value = { x: event.clientX, y: event.clientY }
      }
      
      document.addEventListener('mousemove', handleMouseMove)
      
      // Cleanup function
      return () => {
        console.log('Cleaning up mouse tracking')
        document.removeEventListener('mousemove', handleMouseMove)
      }
    }
  }
)

// Interval cleanup
let pollInterval = null

const startPolling = () => {
  if (isPolling.value) return
  
  isPolling.value = true
  pollInterval = setInterval(() => {
    pollCount.value++
    console.log('Polling...', pollCount.value)
  }, 1000)
}

const stopPolling = () => {
  if (pollInterval) {
    clearInterval(pollInterval)
    pollInterval = null
    isPolling.value = false
    console.log('Polling stopped')
  }
}

// Multiple cleanup resources
watch(
  () => [searchTerm.value, enableWebSocket.value],
  () => {
    console.log('Multiple resource setup')
    
    // Set up multiple resources
    const resources = []
    
    // Resource 1: API subscription
    const subscription = subscribeToAPI()
    resources.push(() => {
      console.log('Cleaning up API subscription')
      subscription.unsubscribe()
    })
    
    // Resource 2: Cache
    const cache = new Map()
    resources.push(() => {
      console.log('Cleaning up cache')
      cache.clear()
    })
    
    // Return cleanup function that cleans all resources
    return () => {
      resources.forEach(cleanup => cleanup())
    }
  }
)

function performSearch(term) {
  console.log('Searching for:', term)
  // Simulate search results
  searchResults.value = term ? [
    { id: 1, title: `Result for ${term}` }
  ] : []
}

function subscribeToAPI() {
  console.log('Subscribing to API')
  return {
    unsubscribe: () => console.log('API subscription cleaned up')
  }
}

// Component cleanup
onUnmounted(() => {
  stopPolling()
  console.log('Component unmounted - all resources cleaned up')
})
</script>
```

#### Watcher Debugging
Techniques for debugging watcher behavior, performance issues, and unexpected triggers.

**Real-world usage:** Troubleshoot infinite loops, identify performance bottlenecks, and understand dependency tracking.

```vue
<template>
  <div>
    <input v-model="debugValue" placeholder="Debug input" />
    <input v-model.number="counter" type="number" />
    
    <div>
      <h4>Debug Info</h4>
      <pre>{{ debugInfo }}</pre>
    </div>
    
    <button @click="triggerComplexUpdate">Trigger Complex Update</button>
    <button @click="toggleDebugMode">Debug Mode: {{ debugMode ? 'ON' : 'OFF' }}</button>
  </div>
</template>

<script setup>
import { ref, watch, watchEffect, computed } from 'vue'

const debugValue = ref('')
const counter = ref(0)
const debugMode = ref(false)
const watcherCalls = ref({})
const debugInfo = ref({})

// Debug wrapper for watchers
function debugWatch(source, callback, options = {}) {
  const watcherId = `watcher_${Math.random().toString(36).substr(2, 9)}`
  watcherCalls.value[watcherId] = 0
  
  return watch(
    source,
    (...args) => {
      watcherCalls.value[watcherId]++
      
      if (debugMode.value) {
        console.group(`🔍 ${watcherId}`)
        console.log('Call count:', watcherCalls.value[watcherId])
        console.log('Arguments:', args)
        console.log('Timestamp:', new Date().toISOString())
        console.trace('Call stack')
        console.groupEnd()
      }
      
      const startTime = performance.now()
      const result = callback(...args)
      const endTime = performance.now()
      
      if (debugMode.value) {
        console.log(`⏱️ ${watcherId} execution time: ${endTime - startTime}ms`)
      }
      
      return result
    },
    options
  )
}

// Debug problematic watcher
debugWatch(
  debugValue,
  (newValue, oldValue) => {
    console.log('Debug value watcher triggered')
    
    // Simulate potential issue
    if (newValue.includes('infinite')) {
      console.warn('Potential infinite loop detected!')
      // Don't modify reactive state here!
    }
    
    updateDebugInfo('debugValue', { old: oldValue, new: newValue })
  }
)

// Performance monitoring watcher
debugWatch(
  counter,
  (newCounter) => {
    console.log('Counter watcher - expensive operation starting')
    
    // Simulate expensive operation
    const start = performance.now()
    for (let i = 0; i < 1000000; i++) {
      Math.random()
    }
    const end = performance.now()
    
    updateDebugInfo('performance', {
      counter: newCounter,
      executionTime: end - start,
      timestamp: new Date().toISOString()
    })
  }
)

// Watch effect with debugging
const watchEffectCalls = ref(0)

watchEffect(() => {
  watchEffectCalls.value++
  
  if (debugMode.value) {
    console.log('🔄 WatchEffect called:', watchEffectCalls.value)
    console.log('Dependencies:', {
      debugValue: debugValue.value,
      counter: counter.value,
      debugMode: debugMode.value
    })
  }
  
  // Track all reactive dependencies
  updateDebugInfo('watchEffect', {
    calls: watchEffectCalls.value,
    dependencies: {
      debugValue: debugValue.value,
      counter: counter.value
    }
  })
})

// Detect infinite loop patterns
let lastCallTime = 0
let rapidCallCount = 0

watch(
  debugValue,
  () => {
    const now = Date.now()
    
    if (now - lastCallTime < 10) { // Less than 10ms between calls
      rapidCallCount++
      
      if (rapidCallCount > 5) {
        console.error('🚨 Possible infinite loop detected!')
        console.error('Rapid successive calls detected')
        
        // Prevent infinite loop
        return
      }
    } else {
      rapidCallCount = 0
    }
    
    lastCallTime = now
  }
)

// Memory usage monitoring
watch(
  [debugValue, counter],
  () => {
    if (debugMode.value && performance.memory) {
      updateDebugInfo('memory', {
        used: Math.round(performance.memory.usedJSHeapSize / 1024 / 1024) + ' MB',
        total: Math.round(performance.memory.totalJSHeapSize / 1024 / 1024) + ' MB',
        limit: Math.round(performance.memory.jsHeapSizeLimit / 1024 / 1024) + ' MB'
      })
    }
  }
)

function updateDebugInfo(key, value) {
  debugInfo.value = {
    ...debugInfo.value,
    [key]: value,
    lastUpdate: new Date().toISOString()
  }
}

const triggerComplexUpdate = () => {
  // Trigger multiple watchers simultaneously
  debugValue.value = `complex_${Date.now()}`
  counter.value += 10
  
  console.log('🎯 Complex update triggered')
}

const toggleDebugMode = () => {
  debugMode.value = !debugMode.value
  console.log('Debug mode:', debugMode.value ? 'enabled' : 'disabled')
}

// Development-only debugging
if (import.meta.env.DEV) {
  // Log all watcher activity
  const originalWatch = watch
  window.watcherStats = watcherCalls
  
  console.log('🛠️ Watcher debugging enabled')
  console.log('Access watcher stats via window.watcherStats')
}
</script>
```

---

## **Part IV: Component System**

### 9. **Component Basics**

#### Creating Components
Define reusable Vue components using Composition API or Options API for modular application architecture.

**Real-world usage:** UI element reusability, feature encapsulation, code organization, and maintainable application structure.

```vue
<!-- UserCard.vue - Basic component -->
<template>
  <div class="user-card">
    <img :src="user.avatar" :alt="user.name" />
    <h3>{{ user.name }}</h3>
    <p>{{ user.email }}</p>
    <button @click="$emit('contact', user.id)">Contact</button>
  </div>
</template>

<script setup>
// Composition API approach
defineProps({
  user: {
    type: Object,
    required: true
  }
})

defineEmits(['contact'])
</script>

<!-- Options API approach -->
<script>
export default {
  name: 'UserCard',
  props: {
    user: {
      type: Object,
      required: true
    }
  },
  emits: ['contact'],
  methods: {
    handleContact() {
      this.$emit('contact', this.user.id)
    }
  }
}
</script>
```

```vue
<!-- ProductList.vue - Component with state -->
<template>
  <div class="product-list">
    <div class="filters">
      <input v-model="searchTerm" placeholder="Search products..." />
      <select v-model="selectedCategory">
        <option value="">All Categories</option>
        <option v-for="cat in categories" :key="cat" :value="cat">
          {{ cat }}
        </option>
      </select>
    </div>
    
    <div class="products">
      <ProductCard 
        v-for="product in filteredProducts" 
        :key="product.id"
        :product="product"
        @add-to-cart="handleAddToCart"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import ProductCard from './ProductCard.vue'

const props = defineProps({
  products: Array,
  categories: Array
})

const emit = defineEmits(['add-to-cart'])

const searchTerm = ref('')
const selectedCategory = ref('')

const filteredProducts = computed(() => {
  return props.products.filter(product => {
    const matchesSearch = product.name.toLowerCase()
      .includes(searchTerm.value.toLowerCase())
    const matchesCategory = !selectedCategory.value || 
      product.category === selectedCategory.value
    
    return matchesSearch && matchesCategory
  })
})

const handleAddToCart = (productId) => {
  emit('add-to-cart', productId)
}
</script>
```

#### Component Registration (Global vs Local)
Register components globally for app-wide access or locally within specific components for better performance and encapsulation.

**Real-world usage:** Common UI components globally, feature-specific components locally, and optimized bundle splitting.

```javascript
// main.js - Global registration
import { createApp } from 'vue'
import App from './App.vue'

// Import components
import BaseButton from './components/BaseButton.vue'
import BaseModal from './components/BaseModal.vue'
import LoadingSpinner from './components/LoadingSpinner.vue'

const app = createApp(App)

// Global registration - available everywhere
app.component('BaseButton', BaseButton)
app.component('BaseModal', BaseModal)
app.component('LoadingSpinner', LoadingSpinner)

// Bulk global registration
const globalComponents = {
  BaseButton,
  BaseModal,
  LoadingSpinner
}

Object.entries(globalComponents).forEach(([name, component]) => {
  app.component(name, component)
})

app.mount('#app')
```

```vue
<!-- Local registration example -->
<template>
  <div>
    <!-- Global components - no import needed -->
    <BaseButton @click="showModal = true">Open Modal</BaseButton>
    <LoadingSpinner v-if="loading" />
    
    <!-- Local components -->
    <UserProfile :user="currentUser" />
    <UserSettings v-if="showSettings" @save="handleSave" />
    
    <!-- Conditional component loading -->
    <AdminPanel v-if="isAdmin" />
  </div>
</template>

<script setup>
import { ref } from 'vue'
// Local imports - only available in this component
import UserProfile from './components/UserProfile.vue'
import UserSettings from './components/UserSettings.vue'

// Conditional import
import AdminPanel from './components/AdminPanel.vue'

const showModal = ref(false)
const loading = ref(false)
const showSettings = ref(false)
const currentUser = ref({ name: 'John', role: 'admin' })

const isAdmin = computed(() => currentUser.value.role === 'admin')

const handleSave = (settings) => {
  console.log('Settings saved:', settings)
}
</script>
```

```javascript
// Plugin-based registration
// plugins/components.js
export default {
  install(app) {
    // Register common components
    const components = import.meta.glob('../components/Base*.vue', { eager: true })
    
    Object.entries(components).forEach(([path, component]) => {
      const componentName = path.split('/').pop().replace('.vue', '')
      app.component(componentName, component.default)
    })
    
    // Register with prefix
    app.component('AppHeader', () => import('../components/AppHeader.vue'))
    app.component('AppFooter', () => import('../components/AppFooter.vue'))
  }
}

// main.js
import componentsPlugin from './plugins/components'
app.use(componentsPlugin)
```

#### Component Naming Conventions
Follow Vue's naming conventions for consistent, readable, and maintainable component organization.

**Real-world usage:** Team collaboration, IDE support, build tool optimization, and avoiding naming conflicts.

```vue
<!-- ✅ Good naming conventions -->

<!-- PascalCase for component names -->
<!-- UserProfileCard.vue -->
<template>
  <div class="user-profile-card">
    <BaseAvatar :src="user.avatar" :alt="user.name" />
    <div class="user-info">
      <UserName :name="user.name" />
      <UserEmail :email="user.email" />
    </div>
  </div>
</template>

<!-- MultiWord component names (recommended) -->
<!-- ProductDetailView.vue -->
<template>
  <div class="product-detail-view">
    <ProductImage :src="product.image" />
    <ProductInfo :product="product" />
    <ProductActions @add-to-cart="handleAddToCart" />
  </div>
</template>

<!-- Base component prefix -->
<!-- BaseButton.vue -->
<template>
  <button :class="buttonClasses" @click="$emit('click')">
    <slot></slot>
  </button>
</template>

<script setup>
const props = defineProps({
  variant: {
    type: String,
    default: 'primary',
    validator: (value) => ['primary', 'secondary', 'danger'].includes(value)
  },
  size: {
    type: String,
    default: 'medium'
  }
})

const buttonClasses = computed(() => [
  'base-button',
  `base-button--${props.variant}`,
  `base-button--${props.size}`
])
</script>
```

```vue
<!-- ❌ Avoid these naming patterns -->

<!-- Single word names -->
<!-- Card.vue - too generic -->
<!-- Button.vue - conflicts with HTML elements -->

<!-- Non-descriptive names -->
<!-- Component1.vue -->
<!-- Temp.vue -->
<!-- Test.vue -->

<!-- Inconsistent naming -->
<!-- user_profile.vue - use kebab-case for files -->
<!-- UserProfile.vue - but kebab-case in templates -->

<!-- ✅ Better alternatives -->
<!-- UserCard.vue -->
<!-- ActionButton.vue -->
<!-- UserProfileComponent.vue -->
```

```javascript
// Naming in different contexts

// File naming (kebab-case)
// user-profile-card.vue
// product-detail-view.vue
// base-button.vue

// Component registration (PascalCase)
app.component('UserProfileCard', UserProfileCard)
app.component('ProductDetailView', ProductDetailView)

// Template usage (kebab-case)
// <user-profile-card />
// <product-detail-view />

// Import naming (PascalCase)
import UserProfileCard from './UserProfileCard.vue'
import ProductDetailView from './ProductDetailView.vue'
```

#### Single File Components (SFC)
Organize template, script, and styles in a single .vue file for component encapsulation and developer experience.

**Real-world usage:** Component organization, style scoping, build tool integration, and hot module replacement.

```vue
<!-- TodoItem.vue - Complete SFC example -->
<template>
  <div class="todo-item" :class="{ completed: todo.completed }">
    <input 
      type="checkbox" 
      :checked="todo.completed"
      @change="$emit('toggle', todo.id)"
    />
    
    <span 
      v-if="!editing" 
      class="todo-text"
      @dblclick="startEdit"
    >
      {{ todo.text }}
    </span>
    
    <input 
      v-else
      ref="editInput"
      v-model="editText"
      class="edit-input"
      @blur="finishEdit"
      @keyup.enter="finishEdit"
      @keyup.esc="cancelEdit"
    />
    
    <div class="actions">
      <button @click="startEdit" v-if="!editing">Edit</button>
      <button @click="$emit('delete', todo.id)">Delete</button>
    </div>
  </div>
</template>

<script setup>
import { ref, nextTick } from 'vue'

const props = defineProps({
  todo: {
    type: Object,
    required: true
  }
})

const emit = defineEmits(['toggle', 'delete', 'edit'])

const editing = ref(false)
const editText = ref('')
const editInput = ref(null)

const startEdit = async () => {
  editing.value = true
  editText.value = props.todo.text
  
  await nextTick()
  editInput.value?.focus()
}

const finishEdit = () => {
  if (editText.value.trim()) {
    emit('edit', props.todo.id, editText.value.trim())
  }
  editing.value = false
}

const cancelEdit = () => {
  editing.value = false
  editText.value = props.todo.text
}
</script>

<style scoped>
.todo-item {
  display: flex;
  align-items: center;
  padding: 12px;
  border: 1px solid #e0e0e0;
  border-radius: 4px;
  margin-bottom: 8px;
  transition: all 0.2s ease;
}

.todo-item:hover {
  background-color: #f5f5f5;
}

.todo-item.completed {
  opacity: 0.6;
}

.todo-item.completed .todo-text {
  text-decoration: line-through;
}

.todo-text {
  flex: 1;
  margin: 0 12px;
  cursor: pointer;
}

.edit-input {
  flex: 1;
  margin: 0 12px;
  padding: 4px 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.actions {
  display: flex;
  gap: 8px;
}

.actions button {
  padding: 4px 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background: white;
  cursor: pointer;
}

.actions button:hover {
  background-color: #f0f0f0;
}
</style>
```

```vue
<!-- WeatherCard.vue - SFC with different script syntaxes -->
<template>
  <div class="weather-card">
    <div class="weather-header">
      <h3>{{ location }}</h3>
      <span class="temperature">{{ temperature }}°{{ unit }}</span>
    </div>
    
    <div class="weather-details">
      <div class="detail">
        <span>Humidity</span>
        <span>{{ humidity }}%</span>
      </div>
      <div class="detail">
        <span>Wind</span>
        <span>{{ windSpeed }} km/h</span>
      </div>
    </div>
    
    <button @click="refresh" :disabled="loading">
      {{ loading ? 'Loading...' : 'Refresh' }}
    </button>
  </div>
</template>

<script>
// Options API in SFC
export default {
  name: 'WeatherCard',
  props: {
    location: String,
    apiKey: String
  },
  data() {
    return {
      temperature: 0,
      humidity: 0,
      windSpeed: 0,
      unit: 'C',
      loading: false
    }
  },
  async mounted() {
    await this.fetchWeather()
  },
  methods: {
    async fetchWeather() {
      this.loading = true
      try {
        // Simulate API call
        const data = await this.getWeatherData()
        this.temperature = data.temperature
        this.humidity = data.humidity
        this.windSpeed = data.windSpeed
      } catch (error) {
        console.error('Weather fetch failed:', error)
      } finally {
        this.loading = false
      }
    },
    async getWeatherData() {
      // Mock weather API
      return new Promise(resolve => {
        setTimeout(() => {
          resolve({
            temperature: Math.round(Math.random() * 30 + 10),
            humidity: Math.round(Math.random() * 40 + 40),
            windSpeed: Math.round(Math.random() * 20 + 5)
          })
        }, 1000)
      })
    },
    refresh() {
      this.fetchWeather()
    }
  }
}
</script>

<style lang="scss" scoped>
.weather-card {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 24px;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  
  .weather-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 16px;
    
    h3 {
      margin: 0;
      font-size: 1.2em;
    }
    
    .temperature {
      font-size: 2em;
      font-weight: bold;
    }
  }
  
  .weather-details {
    display: flex;
    justify-content: space-between;
    margin-bottom: 16px;
    
    .detail {
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 4px;
    }
  }
  
  button {
    width: 100%;
    padding: 8px;
    border: none;
    border-radius: 6px;
    background: rgba(255, 255, 255, 0.2);
    color: white;
    cursor: pointer;
    transition: background 0.2s;
    
    &:hover:not(:disabled) {
      background: rgba(255, 255, 255, 0.3);
    }
    
    &:disabled {
      opacity: 0.6;
      cursor: not-allowed;
    }
  }
}
</style>
```

#### Component Instance Properties
Access component instance properties and methods for component introspection and dynamic behavior.

**Real-world usage:** Parent-child communication, component refs, dynamic component handling, and testing utilities.

```vue
<!-- ParentComponent.vue -->
<template>
  <div>
    <ChildComponent 
      ref="childRef"
      :data="childData"
      @custom-event="handleChildEvent"
    />
    
    <button @click="accessChild">Access Child</button>
    <button @click="logInstanceInfo">Log Instance Info</button>
    
    <ComponentList 
      v-for="(config, index) in components"
      :key="index"
      ref="componentRefs"
      :config="config"
    />
  </div>
</template>

<script setup>
import { ref, getCurrentInstance, nextTick, onMounted } from 'vue'
import ChildComponent from './ChildComponent.vue'
import ComponentList from './ComponentList.vue'

const childRef = ref(null)
const componentRefs = ref([])
const childData = ref({ message: 'Hello Child' })

const components = ref([
  { id: 1, type: 'user', data: {} },
  { id: 2, type: 'product', data: {} }
])

// Get current component instance
const instance = getCurrentInstance()

const accessChild = async () => {
  await nextTick()
  
  if (childRef.value) {
    // Access child component methods
    childRef.value.childMethod()
    
    // Access child component data
    console.log('Child data:', childRef.value.publicData)
    
    // Trigger child component events
    childRef.value.$emit('programmatic-event')
  }
}

const logInstanceInfo = () => {
  if (instance) {
    console.log('Component instance:', instance)
    console.log('Props:', instance.props)
    console.log('Emit:', instance.emit)
    console.log('Slots:', instance.slots)
    console.log('Parent:', instance.parent)
    console.log('Root:', instance.root)
  }
}

const handleChildEvent = (data) => {
  console.log('Child event received:', data)
}

onMounted(() => {
  // Access all component refs
  componentRefs.value.forEach((componentRef, index) => {
    if (componentRef) {
      console.log(`Component ${index}:`, componentRef)
    }
  })
})
</script>
```

```vue
<!-- ChildComponent.vue -->
<template>
  <div class="child-component">
    <h3>Child Component</h3>
    <p>{{ data.message }}</p>
    <p>Internal state: {{ internalState }}</p>
    
    <button @click="emitEvent">Emit Event</button>
    <button @click="updateInternal">Update Internal</button>
  </div>
</template>

<script setup>
import { ref, getCurrentInstance } from 'vue'

const props = defineProps({
  data: Object
})

const emit = defineEmits(['custom-event', 'programmatic-event'])

const internalState = ref('initial')
const publicData = ref({ exposed: true })

// Get instance for internal use
const instance = getCurrentInstance()

// Public methods (can be called via ref)
const childMethod = () => {
  console.log('Child method called')
  internalState.value = 'method-called'
}

const updateInternal = () => {
  internalState.value = `updated-${Date.now()}`
}

const emitEvent = () => {
  emit('custom-event', {
    timestamp: new Date(),
    state: internalState.value
  })
}

// Expose public API
defineExpose({
  childMethod,
  publicData,
  // Expose specific reactive refs
  internalState: readonly(internalState)
})

// Component instance utilities
if (instance) {
  console.log('Child instance created:', instance.uid)
  console.log('Child props:', instance.props)
}
</script>
```

#### Component Lifecycle
Understand component lifecycle hooks for proper initialization, cleanup, and state management.

**Real-world usage:** API calls, event listeners, timers, subscriptions, and resource management.

```vue
<!-- LifecycleDemo.vue -->
<template>
  <div class="lifecycle-demo">
    <h3>Lifecycle Demo</h3>
    <p>Counter: {{ counter }}</p>
    <p>API Data: {{ apiData?.title || 'Loading...' }}</p>
    
    <button @click="counter++">Increment</button>
    <button @click="forceUpdate">Force Update</button>
    
    <ChildWithLifecycle v-if="showChild" :key="childKey" />
    <button @click="toggleChild">{{ showChild ? 'Hide' : 'Show' }} Child</button>
    <button @click="remountChild">Remount Child</button>
  </div>
</template>

<script setup>
import { 
  ref, 
  onBeforeMount, 
  onMounted, 
  onBeforeUpdate, 
  onUpdated,
  onBeforeUnmount, 
  onUnmounted,
  nextTick
} from 'vue'

const counter = ref(0)
const apiData = ref(null)
const showChild = ref(true)
const childKey = ref(0)

// Before component is mounted
onBeforeMount(() => {
  console.log('🔄 onBeforeMount: Component about to be mounted')
  console.log('DOM not yet available')
})

// After component is mounted
onMounted(async () => {
  console.log('✅ onMounted: Component mounted to DOM')
  
  // Safe to access DOM elements
  const element = document.querySelector('.lifecycle-demo')
  console.log('Component element:', element)
  
  // Set up event listeners
  window.addEventListener('resize', handleResize)
  
  // Start timers
  startTimer()
  
  // Load initial data
  await loadApiData()
  
  // Set up subscriptions
  subscribeToUpdates()
})

// Before component updates
onBeforeUpdate(() => {
  console.log('🔄 onBeforeUpdate: Component about to update')
  console.log('Current counter:', counter.value)
})

// After component updates
onUpdated(async () => {
  console.log('✅ onUpdated: Component updated')
  
  // Wait for DOM update
  await nextTick()
  
  // DOM is now updated
  console.log('DOM updated, counter:', counter.value)
})

// Before component unmounts
onBeforeUnmount(() => {
  console.log('🔄 onBeforeUnmount: Component about to unmount')
  
  // Cleanup event listeners
  window.removeEventListener('resize', handleResize)
  
  // Clear timers
  clearInterval(timer)
  
  // Unsubscribe
  if (subscription) {
    subscription.unsubscribe()
  }
})

// After component unmounts
onUnmounted(() => {
  console.log('✅ onUnmounted: Component unmounted')
  console.log('Cleanup completed')
})

// Component logic
let timer = null
let subscription = null

const startTimer = () => {
  timer = setInterval(() => {
    console.log('Timer tick:', new Date().toLocaleTimeString())
  }, 5000)
}

const handleResize = () => {
  console.log('Window resized:', window.innerWidth, window.innerHeight)
}

const loadApiData = async () => {
  try {
    // Simulate API call
    const response = await fetch('https://jsonplaceholder.typicode.com/posts/1')
    apiData.value = await response.json()
    console.log('API data loaded:', apiData.value)
  } catch (error) {
    console.error('Failed to load API data:', error)
  }
}

const subscribeToUpdates = () => {
  subscription = {
    unsubscribe: () => console.log('Subscription cleaned up')
  }
}

const forceUpdate = async () => {
  // Trigger update manually
  const oldValue = counter.value
  counter.value = oldValue + 0 // Same value, but triggers reactivity
  
  await nextTick()
  console.log('Force update completed')
}

const toggleChild = () => {
  showChild.value = !showChild.value
}

const remountChild = () => {
  childKey.value++
}
</script>
```

```vue
<!-- ChildWithLifecycle.vue -->
<template>
  <div class="child-lifecycle">
    <h4>Child Component</h4>
    <p>Mounted at: {{ mountTime }}</p>
    <p>Updates: {{ updateCount }}</p>
  </div>
</template>

<script setup>
import { 
  ref, 
  onBeforeMount, 
  onMounted, 
  onBeforeUpdate, 
  onUpdated,
  onBeforeUnmount, 
  onUnmounted 
} from 'vue'

const mountTime = ref('')
const updateCount = ref(0)

onBeforeMount(() => {
  console.log('👶 Child onBeforeMount')
})

onMounted(() => {
  console.log('👶 Child onMounted')
  mountTime.value = new Date().toLocaleTimeString()
})

onBeforeUpdate(() => {
  console.log('👶 Child onBeforeUpdate')
})

onUpdated(() => {
  console.log('👶 Child onUpdated')
  updateCount.value++
})

onBeforeUnmount(() => {
  console.log('👶 Child onBeforeUnmount')
})

onUnmounted(() => {
  console.log('👶 Child onUnmounted')
})
</script>
```

---

### 10. **Component Communication**

#### Props

**Prop Types and Validation**
Define prop types with validation to ensure component receives correct data and provide better development experience.

**Real-world usage:** API response validation, form data validation, component library type safety, and preventing runtime errors.

```vue
<!-- UserProfile.vue -->
<template>
  <div class="user-profile">
    <img :src="user.avatar" :alt="user.name" />
    <h2>{{ user.name }}</h2>
    <p>Age: {{ user.age }}</p>
    <span :class="`status-${status}`">{{ status }}</span>
    <div v-if="permissions.length">
      Permissions: {{ permissions.join(', ') }}
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  // String with validation
  status: {
    type: String,
    required: true,
    validator: (value) => ['active', 'inactive', 'pending'].includes(value)
  },
  
  // Object with shape validation
  user: {
    type: Object,
    required: true,
    validator: (user) => {
      return user && 
             typeof user.name === 'string' && 
             typeof user.age === 'number' && 
             user.age > 0
    }
  },
  
  // Array with content validation
  permissions: {
    type: Array,
    default: () => [],
    validator: (perms) => {
      return perms.every(perm => typeof perm === 'string')
    }
  },
  
  // Multiple types
  id: {
    type: [String, Number],
    required: true
  },
  
  // Custom constructor validation
  createdAt: {
    type: Date,
    validator: (date) => date instanceof Date && !isNaN(date)
  }
})
</script>
```

**Default Values**
Provide fallback values for optional props to ensure components work without explicit prop values.

**Real-world usage:** Configuration defaults, theme settings, optional features, and backwards compatibility.

```vue
<!-- ProductCard.vue -->
<template>
  <div class="product-card" :class="`variant-${variant}`">
    <img :src="product.image || defaultImage" :alt="product.name" />
    <h3>{{ product.name }}</h3>
    <p class="price">${{ product.price }}</p>
    
    <div class="actions">
      <button :class="`btn-${buttonStyle}`" @click="$emit('add-to-cart')">
        {{ addToCartText }}
      </button>
      <button v-if="showWishlist" @click="$emit('add-to-wishlist')">
        ♥
      </button>
    </div>
    
    <div v-if="showRating" class="rating">
      ⭐ {{ rating }}
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  product: {
    type: Object,
    required: true
  },
  
  // Simple default
  variant: {
    type: String,
    default: 'standard'
  },
  
  // Function default for objects/arrays
  config: {
    type: Object,
    default: () => ({
      showRating: true,
      showWishlist: false,
      buttonStyle: 'primary'
    })
  },
  
  // Factory function default
  metadata: {
    type: Object,
    default: () => ({
      createdAt: new Date(),
      tags: []
    })
  },
  
  // Computed default based on other props
  addToCartText: {
    type: String,
    default: 'Add to Cart'
  },
  
  // Default from environment or constants
  defaultImage: {
    type: String,
    default: '/images/placeholder.jpg'
  },
  
  // Conditional defaults
  rating: {
    type: Number,
    default: 0
  },
  
  showRating: {
    type: Boolean,
    default: true
  },
  
  showWishlist: {
    type: Boolean,
    default: false
  },
  
  buttonStyle: {
    type: String,
    default: 'primary'
  }
})
</script>
```

**Required Props**
Mark essential props as required to enforce proper component usage and catch missing data early.

**Real-world usage:** Core component data, user information, API endpoints, and critical configuration.

```vue
<!-- DataTable.vue -->
<template>
  <div class="data-table">
    <table>
      <thead>
        <tr>
          <th v-for="column in columns" :key="column.key">
            {{ column.label }}
          </th>
          <th v-if="actions.length">Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="row in data" :key="row[idField]">
          <td v-for="column in columns" :key="column.key">
            {{ row[column.key] }}
          </td>
          <td v-if="actions.length">
            <button 
              v-for="action in actions"
              :key="action.name"
              @click="$emit(action.event, row)"
            >
              {{ action.label }}
            </button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
const props = defineProps({
  // Required array data
  data: {
    type: Array,
    required: true
  },
  
  // Required column configuration
  columns: {
    type: Array,
    required: true,
    validator: (cols) => {
      return cols.every(col => col.key && col.label)
    }
  },
  
  // Required unique identifier field
  idField: {
    type: String,
    required: true
  },
  
  // Optional with validation
  actions: {
    type: Array,
    default: () => [],
    validator: (actions) => {
      return actions.every(action => 
        action.name && action.label && action.event
      )
    }
  },
  
  // Required for accessibility
  tableCaption: {
    type: String,
    required: true
  }
})

// Validate required props have valid data
if (props.data.length > 0 && !props.data[0][props.idField]) {
  console.warn(`ID field "${props.idField}" not found in data`)
}
</script>
```

**Boolean Props**
Handle boolean props correctly for toggles, features, and conditional rendering.

**Real-world usage:** Feature toggles, loading states, modal visibility, and component behavior switches.

```vue
<!-- Modal.vue -->
<template>
  <Teleport to="body">
    <div v-if="visible" class="modal-overlay" @click.self="handleOverlayClick">
      <div class="modal" :class="modalClasses">
        <header v-if="!hideHeader" class="modal-header">
          <h3>{{ title }}</h3>
          <button v-if="closable" @click="$emit('close')">&times;</button>
        </header>
        
        <main class="modal-body">
          <slot></slot>
        </main>
        
        <footer v-if="!hideFooter" class="modal-footer">
          <slot name="footer">
            <button v-if="!hideCancel" @click="$emit('close')">
              Cancel
            </button>
            <button v-if="!hideConfirm" @click="$emit('confirm')">
              {{ confirmText }}
            </button>
          </slot>
        </footer>
      </div>
    </div>
  </Teleport>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  // Boolean props - no default needed, false by default
  visible: Boolean,
  closable: Boolean,
  persistent: Boolean,        // Can't close by clicking overlay
  loading: Boolean,
  hideHeader: Boolean,
  hideFooter: Boolean,
  hideCancel: Boolean,
  hideConfirm: Boolean,
  
  // Boolean with explicit default
  fullscreen: {
    type: Boolean,
    default: false
  },
  
  // String props with defaults
  title: {
    type: String,
    default: 'Modal'
  },
  
  confirmText: {
    type: String,
    default: 'OK'
  },
  
  size: {
    type: String,
    default: 'medium',
    validator: (value) => ['small', 'medium', 'large', 'fullscreen'].includes(value)
  }
})

const emit = defineEmits(['close', 'confirm'])

const modalClasses = computed(() => ({
  'modal--fullscreen': props.fullscreen,
  'modal--loading': props.loading,
  [`modal--${props.size}`]: true
}))

const handleOverlayClick = () => {
  if (!props.persistent) {
    emit('close')
  }
}
</script>

<!-- Usage examples -->
<!-- <Modal visible /> - short form for :visible="true" -->
<!-- <Modal :visible="showModal" closable persistent /> -->
<!-- <Modal :visible="true" :hide-footer="true" /> -->
```

**Object and Array Props**
Handle complex data structures with proper validation and reactivity considerations.

**Real-world usage:** Configuration objects, data lists, form schemas, and nested component data.

```vue
<!-- FormBuilder.vue -->
<template>
  <form @submit.prevent="handleSubmit" class="form-builder">
    <div 
      v-for="field in formSchema.fields" 
      :key="field.name"
      class="form-field"
    >
      <label :for="field.name">{{ field.label }}</label>
      
      <input 
        v-if="field.type === 'text'"
        :id="field.name"
        v-model="formData[field.name]"
        :required="field.required"
        :placeholder="field.placeholder"
      />
      
      <select 
        v-else-if="field.type === 'select'"
        :id="field.name"
        v-model="formData[field.name]"
      >
        <option 
          v-for="option in field.options" 
          :key="option.value" 
          :value="option.value"
        >
          {{ option.label }}
        </option>
      </select>
    </div>
    
    <div class="form-actions">
      <button 
        v-for="action in actions"
        :key="action.type"
        :type="action.type"
        :class="action.class"
      >
        {{ action.label }}
      </button>
    </div>
  </form>
</template>

<script setup>
import { ref, reactive, watch } from 'vue'

const props = defineProps({
  // Complex object with nested validation
  formSchema: {
    type: Object,
    required: true,
    validator: (schema) => {
      return schema.fields && Array.isArray(schema.fields) &&
             schema.fields.every(field => 
               field.name && field.type && field.label
             )
    }
  },
  
  // Array of objects with validation
  actions: {
    type: Array,
    default: () => [
      { type: 'submit', label: 'Submit', class: 'btn-primary' },
      { type: 'button', label: 'Cancel', class: 'btn-secondary' }
    ],
    validator: (actions) => {
      return actions.every(action => 
        action.type && action.label
      )
    }
  },
  
  // Initial form data
  initialData: {
    type: Object,
    default: () => ({})
  },
  
  // Validation rules
  validationRules: {
    type: Object,
    default: () => ({}),
    validator: (rules) => {
      return Object.values(rules).every(rule => 
        typeof rule === 'function' || Array.isArray(rule)
      )
    }
  },
  
  // Nested configuration
  config: {
    type: Object,
    default: () => ({
      showLabels: true,
      inline: false,
      validateOnChange: true
    })
  }
})

const emit = defineEmits(['submit', 'change', 'action'])

// Initialize form data reactively
const formData = reactive({})

// Watch for schema changes to rebuild form data
watch(
  () => props.formSchema.fields,
  (fields) => {
    fields.forEach(field => {
      if (!(field.name in formData)) {
        formData[field.name] = props.initialData[field.name] || field.default || ''
      }
    })
  },
  { immediate: true, deep: true }
)

const handleSubmit = () => {
  emit('submit', { ...formData })
}
</script>
```

**Prop Casing**
Follow Vue's casing conventions for props in templates and JavaScript.

**Real-world usage:** Consistent naming, HTML attribute mapping, and team coding standards.

```vue
<!-- Component definition with camelCase -->
<script setup>
const props = defineProps({
  // camelCase in JavaScript
  userName: String,
  isLoggedIn: Boolean,
  userProfile: Object,
  maxFileSize: Number,
  allowedFileTypes: Array,
  onUserClick: Function
})
</script>

<!-- Usage in template with kebab-case -->
<template>
  <div>
    <!-- kebab-case in templates -->
    <UserCard 
      :user-name="currentUser.name"
      :is-logged-in="authenticated"
      :user-profile="profile"
      :max-file-size="5000000"
      :allowed-file-types="['jpg', 'png']"
      @user-click="handleClick"
    />
    
    <!-- camelCase also works in templates -->
    <UserCard 
      :userName="currentUser.name"
      :isLoggedIn="authenticated"
    />
  </div>
</template>
```

#### Events

**Emitting Events**
Communicate from child to parent components using custom events for data flow and user interactions.

**Real-world usage:** Form submissions, user actions, data updates, and component state changes.

```vue
<!-- SearchBox.vue -->
<template>
  <div class="search-box">
    <input 
      v-model="query"
      @input="handleInput"
      @keyup.enter="handleSearch"
      @focus="handleFocus"
      @blur="handleBlur"
      placeholder="Search..."
    />
    <button @click="handleSearch">Search</button>
    <button v-if="query" @click="handleClear">Clear</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const emit = defineEmits([
  'search',
  'input-change',
  'clear',
  'focus',
  'blur'
])

const query = ref('')

const handleInput = () => {
  emit('input-change', {
    query: query.value,
    timestamp: new Date()
  })
}

const handleSearch = () => {
  if (query.value.trim()) {
    emit('search', {
      query: query.value.trim(),
      filters: {},
      timestamp: new Date()
    })
  }
}

const handleClear = () => {
  query.value = ''
  emit('clear')
  emit('input-change', { query: '', timestamp: new Date() })
}

const handleFocus = () => {
  emit('focus')
}

const handleBlur = () => {
  emit('blur')
}
</script>

<!-- Parent usage -->
<template>
  <SearchBox 
    @search="performSearch"
    @input-change="handleInputChange"
    @clear="handleClear"
    @focus="handleSearchFocus"
  />
</template>
```

**Event Arguments**
Pass data through events to communicate complex information between components.

**Real-world usage:** Form data, user selections, error information, and action contexts.

```vue
<!-- DataGrid.vue -->
<template>
  <div class="data-grid">
    <table>
      <thead>
        <tr>
          <th v-for="column in columns" :key="column.key">
            <button @click="handleSort(column)">
              {{ column.label }}
              <span v-if="sortBy === column.key">
                {{ sortOrder === 'asc' ? '↑' : '↓' }}
              </span>
            </button>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr 
          v-for="(row, index) in data" 
          :key="row.id"
          @click="handleRowClick(row, index)"
        >
          <td v-for="column in columns" :key="column.key">
            {{ row[column.key] }}
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  data: Array,
  columns: Array
})

const emit = defineEmits([
  'sort',
  'row-click',
  'row-select',
  'error'
])

const sortBy = ref('')
const sortOrder = ref('asc')

const handleSort = (column) => {
  const newOrder = sortBy.value === column.key && sortOrder.value === 'asc' 
    ? 'desc' 
    : 'asc'
  
  sortBy.value = column.key
  sortOrder.value = newOrder
  
  // Emit with detailed sort information
  emit('sort', {
    column: column.key,
    order: newOrder,
    columnConfig: column,
    timestamp: new Date(),
    previousSort: {
      column: sortBy.value,
      order: sortOrder.value
    }
  })
}

const handleRowClick = (row, index) => {
  // Emit with row data and context
  emit('row-click', {
    row,
    index,
    id: row.id,
    event: 'click',
    metadata: {
      timestamp: new Date(),
      position: { index },
      context: 'table-row'
    }
  })
  
  // Also emit selection event
  emit('row-select', {
    selectedRow: row,
    selectedIndex: index,
    allSelected: [row.id],
    selectionMode: 'single'
  })
}

// Error handling with detailed context
const handleError = (error, context) => {
  emit('error', {
    error: error.message,
    code: error.code,
    context,
    timestamp: new Date(),
    component: 'DataGrid',
    recovery: {
      possible: true,
      suggestions: ['Refresh data', 'Check network']
    }
  })
}
</script>
```

**Event Validation**
Validate emitted events to ensure proper data structure and catch issues during development.

**Real-world usage:** API integration validation, form data validation, and debugging event flow.

```vue
<!-- FormValidator.vue -->
<template>
  <form @submit.prevent="handleSubmit">
    <input v-model="formData.email" type="email" required />
    <input v-model="formData.password" type="password" required />
    <button type="submit">Submit</button>
  </form>
</template>

<script setup>
import { reactive } from 'vue'

// Define events with validation
const emit = defineEmits({
  // Simple validation
  submit: (payload) => {
    return payload && typeof payload === 'object'
  },
  
  // Detailed validation
  'form-change': (data) => {
    return data &&
           typeof data.field === 'string' &&
           data.hasOwnProperty('value') &&
           typeof data.timestamp === 'object'
  },
  
  // Complex validation
  'validation-error': (error) => {
    const requiredProps = ['field', 'message', 'type']
    return error &&
           requiredProps.every(prop => error.hasOwnProperty(prop)) &&
           ['required', 'format', 'length'].includes(error.type)
  },
  
  // Array validation
  'multi-select': (items) => {
    return Array.isArray(items) &&
           items.every(item => 
             item && 
             typeof item.id !== 'undefined' &&
             typeof item.value !== 'undefined'
           )
  }
})

const formData = reactive({
  email: '',
  password: ''
})

const handleSubmit = () => {
  const payload = {
    email: formData.email,
    password: formData.password,
    timestamp: new Date(),
    formId: 'login-form'
  }
  
  // This will pass validation
  emit('submit', payload)
  
  // This would fail validation in development
  // emit('submit', 'invalid-payload')
}

// Emit with validation during form changes
const handleFieldChange = (field, value) => {
  const changeData = {
    field,
    value,
    timestamp: new Date(),
    formState: { ...formData }
  }
  
  emit('form-change', changeData)
}

// Emit validation errors
const emitValidationError = (field, message, type) => {
  emit('validation-error', {
    field,
    message,
    type,
    timestamp: new Date(),
    severity: 'error'
  })
}
</script>
```

**Native Event Binding**
Bind native DOM events directly to component root elements or specific child elements.

**Real-world usage:** Focus management, keyboard shortcuts, mouse interactions, and accessibility features.

```vue
<!-- InteractiveCard.vue -->
<template>
  <div 
    class="interactive-card"
    tabindex="0"
    @click="handleCardClick"
    @keyup.enter="handleEnterKey"
    @keyup.space="handleSpaceKey"
    @focus="handleFocus"
    @blur="handleBlur"
  >
    <header @click.stop="handleHeaderClick">
      <h3>{{ title }}</h3>
      <button @click.stop="$emit('close')">×</button>
    </header>
    
    <main @scroll="handleScroll">
      <slot></slot>
    </main>
    
    <footer>
      <!-- Forward native events to custom button -->
      <button 
        @click="handleAction"
        @mouseenter="handleButtonHover"
        @mouseleave="handleButtonLeave"
        @focus="handleButtonFocus"
      >
        Action
      </button>
    </footer>
  </div>
</template>

<script setup>
const props = defineProps({
  title: String
})

const emit = defineEmits([
  'click',
  'action',
  'close',
  'focus',
  'blur',
  'header-click',
  'scroll'
])

const handleCardClick = (event) => {
  emit('click', {
    nativeEvent: event,
    target: event.target,
    coordinates: {
      x: event.clientX,
      y: event.clientY
    },
    timestamp: new Date()
  })
}

const handleEnterKey = (event) => {
  emit('action', { trigger: 'keyboard', key: 'enter' })
}

const handleSpaceKey = (event) => {
  event.preventDefault()
  emit('action', { trigger: 'keyboard', key: 'space' })
}

const handleFocus = () => {
  emit('focus', { source: 'card' })
}

const handleBlur = () => {
  emit('blur', { source: 'card' })
}

const handleHeaderClick = (event) => {
  emit('header-click', {
    nativeEvent: event,
    section: 'header'
  })
}

const handleScroll = (event) => {
  emit('scroll', {
    scrollTop: event.target.scrollTop,
    scrollLeft: event.target.scrollLeft
  })
}

const handleAction = () => {
  emit('action', { trigger: 'click' })
}

const handleButtonHover = () => {
  console.log('Button hover')
}

const handleButtonLeave = () => {
  console.log('Button leave')
}

const handleButtonFocus = () => {
  emit('focus', { source: 'button' })
}
</script>

<!-- Parent component using native events -->
<template>
  <InteractiveCard
    title="My Card"
    @click="handleCardClick"
    @action="handleAction"
    @focus="handleCardFocus"
    @scroll="handleCardScroll"
  >
    Card content here
  </InteractiveCard>
</template>
```

#### Slots

**Basic Slots**
Provide content placeholders in components for flexible, reusable component composition.

**Real-world usage:** Layout components, card containers, modal content, and customizable UI elements.

```vue
<!-- Card.vue -->
<template>
  <div class="card" :class="cardClasses">
    <slot></slot>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  variant: {
    type: String,
    default: 'default'
  }
})

const cardClasses = computed(() => [
  'card',
  `card--${props.variant}`
])
</script>

<!-- Panel.vue -->
<template>
  <div class="panel">
    <header class="panel-header">
      <slot name="header">
        <h3>Default Title</h3>
      </slot>
    </header>
    
    <main class="panel-body">
      <slot>
        <p>Default content</p>
      </slot>
    </main>
    
    <footer class="panel-footer">
      <slot name="footer">
        <button>OK</button>
      </slot>
    </footer>
  </div>
</template>

<!-- Usage -->
<template>
  <Card>
    <h2>Welcome</h2>
    <p>This content goes inside the card.</p>
  </Card>
  
  <Panel>
    <template #header>
      <h2>Custom Header</h2>
    </template>
    
    <p>Custom panel content</p>
    
    <template #footer>
      <button>Save</button>
      <button>Cancel</button>
    </template>
  </Panel>
</template>
```

**Named Slots**
Define multiple content areas within components for complex layouts and structured content.

**Real-world usage:** Page layouts, dashboard widgets, form sections, and multi-area components.

```vue
<!-- AppLayout.vue -->
<template>
  <div class="app-layout">
    <header class="app-header">
      <slot name="header">
        <h1>Default App Header</h1>
      </slot>
    </header>
    
    <nav class="app-navigation" v-if="$slots.navigation">
      <slot name="navigation"></slot>
    </nav>
    
    <aside class="app-sidebar" v-if="$slots.sidebar">
      <slot name="sidebar"></slot>
    </aside>
    
    <main class="app-main">
      <slot></slot>
    </main>
    
    <footer class="app-footer" v-if="$slots.footer">
      <slot name="footer"></slot>
    </footer>
  </div>
</template>

<!-- Dashboard.vue -->
<template>
  <div class="dashboard">
    <div class="dashboard-header">
      <slot name="title">
        <h2>Dashboard</h2>
      </slot>
      
      <div class="dashboard-actions">
        <slot name="actions">
          <button>Default Action</button>
        </slot>
      </div>
    </div>
    
    <div class="dashboard-content">
      <div class="dashboard-stats">
        <slot name="stats"></slot>
      </div>
      
      <div class="dashboard-charts">
        <slot name="charts"></slot>
      </div>
      
      <div class="dashboard-tables">
        <slot name="tables"></slot>
      </div>
    </div>
  </div>
</template>

<!-- Usage -->
<template>
  <AppLayout>
    <template #header>
      <h1>My Application</h1>
      <nav>Navigation items</nav>
    </template>
    
    <template #navigation>
      <ul>
        <li>Home</li>
        <li>Dashboard</li>
        <li>Settings</li>
      </ul>
    </template>
    
    <template #sidebar>
      <div>Sidebar content</div>
    </template>
    
    <!-- Default slot -->
    <Dashboard>
      <template #title>
        <h2>Sales Dashboard</h2>
      </template>
      
      <template #actions>
        <button>Export</button>
        <button>Refresh</button>
      </template>
      
      <template #stats>
        <div>Revenue: $10,000</div>
        <div>Orders: 150</div>
      </template>
      
      <template #charts>
        <div>Chart components here</div>
      </template>
    </Dashboard>
    
    <template #footer>
      <p>&copy; 2024 My Company</p>
    </template>
  </AppLayout>
</template>
```

**Scoped Slots**
Pass data from parent to child component content, enabling dynamic content generation with component data.

**Real-world usage:** Data tables, list rendering, form builders, and any component that needs to expose internal data.

```vue
<!-- DataList.vue -->
<template>
  <div class="data-list">
    <div 
      v-for="(item, index) in items" 
      :key="item.id || index"
      class="data-item"
    >
      <slot 
        :item="item" 
        :index="index"
        :isFirst="index === 0"
        :isLast="index === items.length - 1"
        :isEven="index % 2 === 0"
        :select="() => selectItem(item)"
        :remove="() => removeItem(index)"
      >
        <!-- Default content if no slot provided -->
        <div>{{ item.name || item.title || 'Item' }}</div>
      </slot>
    </div>
    
    <div v-if="!items.length" class="empty-state">
      <slot name="empty" :reload="loadItems">
        <p>No items found</p>
      </slot>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  items: Array
})

const emit = defineEmits(['select', 'remove', 'reload'])

const selectItem = (item) => {
  emit('select', item)
}

const removeItem = (index) => {
  emit('remove', index)
}

const loadItems = () => {
  emit('reload')
}
</script>

<!-- UserTable.vue -->
<template>
  <table class="user-table">
    <thead>
      <tr>
        <th v-for="column in columns" :key="column.key">
          {{ column.label }}
        </th>
        <th>Actions</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="user in users" :key="user.id">
        <td v-for="column in columns" :key="column.key">
          <slot 
            :name="`column-${column.key}`"
            :user="user"
            :value="user[column.key]"
            :column="column"
            :formatValue="(val) => formatValue(val, column.type)"
          >
            {{ formatValue(user[column.key], column.type) }}
          </slot>
        </td>
        <td>
          <slot 
            name="actions"
            :user="user"
            :edit="() => editUser(user)"
            :delete="() => deleteUser(user)"
            :view="() => viewUser(user)"
          >
            <button @click="editUser(user)">Edit</button>
            <button @click="deleteUser(user)">Delete</button>
          </slot>
        </td>
      </tr>
    </tbody>
  </table>
</template>

<script setup>
const props = defineProps({
  users: Array,
  columns: Array
})

const emit = defineEmits(['edit', 'delete', 'view'])

const formatValue = (value, type) => {
  switch (type) {
    case 'date':
      return new Date(value).toLocaleDateString()
    case 'currency':
      return `$${value.toFixed(2)}`
    default:
      return value
  }
}

const editUser = (user) => emit('edit', user)
const deleteUser = (user) => emit('delete', user)
const viewUser = (user) => emit('view', user)
</script>

<!-- Usage -->
<template>
  <DataList :items="products">
    <template #default="{ item, index, isEven, select, remove }">
      <div :class="{ 'even-row': isEven }" class="product-item">
        <h3>{{ item.name }}</h3>
        <p>Price: ${{ item.price }}</p>
        <p>Stock: {{ item.stock }}</p>
        <button @click="select">Select</button>
        <button @click="remove">Remove</button>
      </div>
    </template>
    
    <template #empty="{ reload }">
      <div class="empty-products">
        <h3>No products available</h3>
        <button @click="reload">Load Products</button>
      </div>
    </template>
  </DataList>
  
  <UserTable :users="userList" :columns="tableColumns">
    <template #column-avatar="{ user }">
      <img :src="user.avatar" :alt="user.name" class="avatar" />
    </template>
    
    <template #column-status="{ user, value }">
      <span :class="`status-${value}`">
        {{ value.toUpperCase() }}
      </span>
    </template>
    
    <template #actions="{ user, edit, delete }">
      <button @click="edit" class="btn-primary">Edit</button>
      <button @click="delete" class="btn-danger">Delete</button>
      <button @click="sendMessage(user)">Message</button>
    </template>
  </UserTable>
</template>
```

**Slot Props**
Pass data to slot content for flexible, data-driven content rendering with component state access.

**Real-world usage:** Complex data rendering, customizable component interfaces, and exposing internal component state.

```vue
<!-- AsyncDataLoader.vue -->
<template>
  <div class="async-data-loader">
    <slot 
      v-if="loading"
      name="loading"
      :progress="progress"
      :message="loadingMessage"
    >
      <div>Loading... {{ progress }}%</div>
    </slot>
    
    <slot 
      v-else-if="error"
      name="error"
      :error="error"
      :retry="fetchData"
      :reset="resetState"
    >
      <div class="error">
        Error: {{ error.message }}
        <button @click="fetchData">Retry</button>
      </div>
    </slot>
    
    <slot 
      v-else
      name="default"
      :data="data"
      :refresh="fetchData"
      :loading="loading"
      :meta="meta"
    >
      <pre>{{ JSON.stringify(data, null, 2) }}</pre>
    </slot>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  url: String,
  params: Object
})

const data = ref(null)
const loading = ref(false)
const error = ref(null)
const progress = ref(0)
const loadingMessage = ref('')
const meta = ref({})

const fetchData = async () => {
  loading.value = true
  error.value = null
  progress.value = 0
  loadingMessage.value = 'Initializing...'
  
  try {
    // Simulate progress updates
    for (let i = 0; i <= 100; i += 20) {
      progress.value = i
      loadingMessage.value = `Loading... ${i}%`
      await new Promise(resolve => setTimeout(resolve, 200))
    }
    
    // Simulate API call
    const response = await fetch(props.url, {
      method: 'GET',
      headers: { 'Content-Type': 'application/json' }
    })
    
    data.value = await response.json()
    meta.value = {
      timestamp: new Date(),
      status: response.status,
      headers: Object.fromEntries(response.headers)
    }
  } catch (err) {
    error.value = err
  } finally {
    loading.value = false
  }
}

const resetState = () => {
  data.value = null
  error.value = null
  progress.value = 0
}

defineExpose({ fetchData, resetState })
</script>

<!-- Usage with slot props -->
<template>
  <AsyncDataLoader url="/api/users">
    <template #loading="{ progress, message }">
      <div class="custom-loader">
        <div class="progress-bar">
          <div 
            class="progress-fill" 
            :style="{ width: progress + '%' }"
          ></div>
        </div>
        <p>{{ message }}</p>
      </div>
    </template>
    
    <template #error="{ error, retry, reset }">
      <div class="error-state">
        <h3>Oops! Something went wrong</h3>
        <p>{{ error.message }}</p>
        <div class="error-actions">
          <button @click="retry">Try Again</button>
          <button @click="reset">Reset</button>
        </div>
      </div>
    </template>
    
    <template #default="{ data, refresh, meta }">
      <div class="user-list">
        <div class="list-header">
          <h2>Users ({{ data.length }})</h2>
          <button @click="refresh">Refresh</button>
          <small>Last updated: {{ meta.timestamp?.toLocaleString() }}</small>
        </div>
        
        <div class="user-grid">
          <div v-for="user in data" :key="user.id" class="user-card">
            <h3>{{ user.name }}</h3>
            <p>{{ user.email }}</p>
          </div>
        </div>
      </div>
    </template>
  </AsyncDataLoader>
</template>
```

**Dynamic Slot Names**
Use dynamic slot names for flexible content distribution based on runtime conditions.

**Real-world usage:** Dynamic forms, conditional layouts, tab systems, and configurable interfaces.

```vue
<!-- DynamicForm.vue -->
<template>
  <form class="dynamic-form">
    <div 
      v-for="field in fields" 
      :key="field.name"
      class="form-field"
    >
      <label>{{ field.label }}</label>
      
      <!-- Dynamic slot names based on field type -->
      <slot 
        :name="`field-${field.type}`"
        :field="field"
        :value="formData[field.name]"
        :update="(value) => updateField(field.name, value)"
        :validate="() => validateField(field)"
      >
        <!-- Default field rendering -->
        <input 
          :type="field.type"
          :value="formData[field.name]"
          @input="updateField(field.name, $event.target.value)"
        />
      </slot>
      
      <!-- Dynamic error slot -->
      <slot 
        v-if="errors[field.name]"
        :name="`error-${field.name}`"
        :error="errors[field.name]"
        :field="field"
      >
        <span class="error">{{ errors[field.name] }}</span>
      </slot>
    </div>
    
    <!-- Dynamic action slots -->
    <div class="form-actions">
      <slot 
        v-for="action in actions"
        :key="action"
        :name="`action-${action}`"
        :submit="handleSubmit"
        :reset="resetForm"
        :validate="validateForm"
      >
        <button :type="action">{{ action }}</button>
      </slot>
    </div>
  </form>
</template>

<script setup>
import { ref, reactive } from 'vue'

const props = defineProps({
  fields: Array,
  actions: {
    type: Array,
    default: () => ['submit', 'reset']
  }
})

const formData = reactive({})
const errors = reactive({})

const updateField = (name, value) => {
  formData[name] = value
  if (errors[name]) {
    delete errors[name]
  }
}

const validateField = (field) => {
  // Validation logic
  return true
}

const validateForm = () => {
  return props.fields.every(field => validateField(field))
}

const handleSubmit = () => {
  if (validateForm()) {
    console.log('Form submitted:', formData)
  }
}

const resetForm = () => {
  Object.keys(formData).forEach(key => {
    formData[key] = ''
  })
  Object.keys(errors).forEach(key => {
    delete errors[key]
  })
}
</script>

<!-- Usage with dynamic slots -->
<template>
  <DynamicForm :fields="formFields" :actions="['submit', 'save', 'cancel']">
    <!-- Custom field renderers -->
    <template #field-email="{ field, value, update }">
      <input 
        type="email"
        :value="value"
        @input="update($event.target.value)"
        placeholder="Enter email address"
      />
    </template>
    
    <template #field-select="{ field, value, update }">
      <select :value="value" @change="update($event.target.value)">
        <option v-for="option in field.options" :key="option.value" :value="option.value">
          {{ option.label }}
        </option>
      </select>
    </template>
    
    <template #field-textarea="{ field, value, update }">
      <textarea 
        :value="value"
        @input="update($event.target.value)"
        :rows="field.rows || 3"
      ></textarea>
    </template>
    
    <!-- Custom error displays -->
    <template #error-email="{ error }">
      <div class="email-error">
        📧 {{ error }}
      </div>
    </template>
    
    <!-- Custom action buttons -->
    <template #action-submit="{ submit, validate }">
      <button type="submit" @click="submit" class="btn-primary">
        Submit Form
      </button>
    </template>
    
    <template #action-save="{ validate }">
      <button type="button" @click="saveAsDraft" class="btn-secondary">
        Save Draft
      </button>
    </template>
  </DynamicForm>
</template>
```

**Conditional Slots**
Conditionally render slots based on component state or props for adaptive interfaces.

**Real-world usage:** Permission-based UI, feature flags, responsive layouts, and progressive disclosure.

```vue
<!-- ConditionalCard.vue -->
<template>
  <div class="conditional-card" :class="cardClasses">
    <!-- Header slot - conditional on showHeader prop -->
    <header v-if="showHeader && $slots.header" class="card-header">
      <slot name="header" :title="title" :subtitle="subtitle"></slot>
    </header>
    
    <!-- Navigation - only for authenticated users -->
    <nav v-if="isAuthenticated && $slots.navigation" class="card-nav">
      <slot name="navigation" :user="currentUser"></slot>
    </nav>
    
    <!-- Main content -->
    <main class="card-content">
      <slot></slot>
    </main>
    
    <!-- Actions - conditional on permissions -->
    <footer v-if="hasActions" class="card-footer">
      <!-- Admin actions -->
      <div v-if="canEdit && $slots['admin-actions']" class="admin-actions">
        <slot name="admin-actions" :edit="edit" :delete="remove"></slot>
      </div>
      
      <!-- User actions -->
      <div v-if="$slots['user-actions']" class="user-actions">
        <slot name="user-actions" :save="save" :share="share"></slot>
      </div>
      
      <!-- Default actions -->
      <div v-if="!$slots['admin-actions'] && !$slots['user-actions']" class="default-actions">
        <slot name="actions" :close="close"></slot>
      </div>
    </footer>
    
    <!-- Debug info - only in development -->
    <aside v-if="isDevelopment && showDebug && $slots.debug" class="debug-panel">
      <slot name="debug" :props="$props" :slots="$slots" :attrs="$attrs"></slot>
    </aside>
  </div>
</template>

<script setup>
import { computed, useSlots } from 'vue'

const props = defineProps({
  title: String,
  subtitle: String,
  showHeader: { type: Boolean, default: true },
  isAuthenticated: Boolean,
  currentUser: Object,
  permissions: Array,
  showDebug: Boolean
})

const slots = useSlots()

const cardClasses = computed(() => ({
  'card--with-header': props.showHeader && slots.header,
  'card--authenticated': props.isAuthenticated,
  'card--debug': props.showDebug
}))

const canEdit = computed(() => {
  return props.permissions?.includes('edit') || props.currentUser?.role === 'admin'
})

const hasActions = computed(() => {
  return slots['admin-actions'] || slots['user-actions'] || slots.actions
})

const isDevelopment = computed(() => {
  return import.meta.env.DEV
})

// Action methods
const edit = () => console.log('Edit action')
const remove = () => console.log('Delete action')
const save = () => console.log('Save action')
const share = () => console.log('Share action')
const close = () => console.log('Close action')
</script>

<!-- Usage with conditional slots -->
<template>
  <ConditionalCard
    title="User Profile"
    subtitle="Manage your account"
    :is-authenticated="user.isLoggedIn"
    :current-user="user"
    :permissions="user.permissions"
    :show-debug="debugMode"
  >
    <!-- Always shown if showHeader is true -->
    <template #header="{ title, subtitle }">
      <h1>{{ title }}</h1>
      <p>{{ subtitle }}</p>
    </template>
    
    <!-- Only shown for authenticated users -->
    <template #navigation="{ user }">
      <ul>
        <li><a href="/profile">Profile</a></li>
        <li><a href="/settings">Settings</a></li>
        <li v-if="user.role === 'admin'"><a href="/admin">Admin</a></li>
      </ul>
    </template>
    
    <!-- Main content -->
    <div>
      <p>Welcome, {{ user.name }}!</p>
    </div>
    
    <!-- Only shown for admin users -->
    <template #admin-actions="{ edit, delete }">
      <button @click="edit" class="btn-primary">Edit User</button>
      <button @click="delete" class="btn-danger">Delete User</button>
    </template>
    
    <!-- Shown for regular authenticated users -->
    <template #user-actions="{ save, share }" v-if="!isAdmin">
      <button @click="save">Save Changes</button>
      <button @click="share">Share Profile</button>
    </template>
    
    <!-- Debug info - only in development -->
    <template #debug="{ props, slots, attrs }">
      <div class="debug-info">
        <h4>Debug Information</h4>
        <pre>Props: {{ JSON.stringify(props, null, 2) }}</pre>
        <pre>Available slots: {{ Object.keys(slots) }}</pre>
      </div>
    </template>
  </ConditionalCard>
</template>
```

#### Provide/Inject

**Basic Usage**
Share data between ancestor and descendant components without prop drilling.

**Real-world usage:** Theme systems, user authentication, global settings, and plugin data sharing.

```vue
<!-- App.vue - Provider -->
<template>
  <div id="app">
    <Header />
    <MainContent />
    <Footer />
  </div>
</template>

<script setup>
import { provide, ref, reactive } from 'vue'

// Provide theme data
const theme = reactive({
  mode: 'light',
  primaryColor: '#007bff',
  fontSize: '16px'
})

const toggleTheme = () => {
  theme.mode = theme.mode === 'light' ? 'dark' : 'light'
}

// Provide user data
const user = ref({
  id: 1,
  name: 'John Doe',
  email: 'john@example.com',
  role: 'admin'
})

// Provide API configuration
const apiConfig = reactive({
  baseURL: 'https://api.example.com',
  timeout: 5000,
  headers: {
    'Content-Type': 'application/json'
  }
})

// Provide multiple values
provide('theme', { theme, toggleTheme })
provide('user', user)
provide('apiConfig', apiConfig)

// Provide utility functions
provide('utils', {
  formatDate: (date) => new Date(date).toLocaleDateString(),
  formatCurrency: (amount) => `$${amount.toFixed(2)}`,
  validateEmail: (email) => /\S+@\S+\.\S+/.test(email)
})
</script>

<!-- Header.vue - Consumer -->
<template>
  <header :class="`header header--${theme.mode}`">
    <h1 :style="{ color: theme.primaryColor }">My App</h1>
    <nav>
      <UserMenu />
    </nav>
    <button @click="toggleTheme">
      {{ theme.mode === 'light' ? '🌙' : '☀️' }}
    </button>
  </header>
</template>

<script setup>
import { inject } from 'vue'

// Inject theme
const { theme, toggleTheme } = inject('theme')

// Inject user for navigation
const user = inject('user')
</script>

<!-- UserMenu.vue - Deep consumer -->
<template>
  <div class="user-menu">
    <button @click="showMenu = !showMenu">
      {{ user.name }} ▼
    </button>
    
    <ul v-if="showMenu" class="menu-dropdown">
      <li>Profile</li>
      <li>Settings</li>
      <li v-if="user.role === 'admin'">Admin Panel</li>
      <li @click="logout">Logout</li>
    </ul>
  </div>
</template>

<script setup>
import { ref, inject } from 'vue'

const showMenu = ref(false)

// Inject user data (available even in deeply nested components)
const user = inject('user')

// Inject utilities
const { formatDate } = inject('utils')

const logout = () => {
  // Logout logic
  console.log('User logged out')
}
</script>
```

**Injection Keys**
Use string or Symbol keys for type-safe injection and avoiding naming conflicts.

**Real-world usage:** Large applications, plugin systems, library development, and preventing key collisions.

```vue
<!-- keys.js - Injection key definitions -->
<script>
export const THEME_KEY = 'theme'
export const USER_KEY = 'user'
export const API_KEY = Symbol('api')
export const ROUTER_KEY = Symbol('router')
export const STORE_KEY = Symbol('store')

// Type-safe keys with TypeScript
export const NOTIFICATION_KEY = Symbol('notifications')
export const MODAL_KEY = Symbol('modal')
</script>

<!-- ProviderComponent.vue -->
<template>
  <div class="app-provider">
    <slot></slot>
  </div>
</template>

<script setup>
import { provide, reactive } from 'vue'
import { THEME_KEY, USER_KEY, API_KEY, NOTIFICATION_KEY } from './keys'

// Provide with string keys
provide(THEME_KEY, {
  mode: 'light',
  colors: {
    primary: '#007bff',
    secondary: '#6c757d'
  }
})

provide(USER_KEY, {
  id: 1,
  name: 'John Doe',
  permissions: ['read', 'write']
})

// Provide with Symbol keys (more secure)
provide(API_KEY, {
  baseURL: process.env.VUE_APP_API_URL,
  key: process.env.VUE_APP_API_KEY,
  methods: {
    get: (endpoint) => fetch(`${baseURL}/${endpoint}`),
    post: (endpoint, data) => fetch(`${baseURL}/${endpoint}`, {
      method: 'POST',
      body: JSON.stringify(data)
    })
  }
})

// Notification system
const notifications = reactive([])

const addNotification = (message, type = 'info') => {
  notifications.push({
    id: Date.now(),
    message,
    type,
    timestamp: new Date()
  })
}

const removeNotification = (id) => {
  const index = notifications.findIndex(n => n.id === id)
  if (index > -1) {
    notifications.splice(index, 1)
  }
}

provide(NOTIFICATION_KEY, {
  notifications,
  addNotification,
  removeNotification
})
</script>

<!-- ConsumerComponent.vue -->
<template>
  <div :class="`component component--${theme.mode}`">
    <h2>{{ user.name }}'s Dashboard</h2>
    
    <div class="notifications">
      <div 
        v-for="notification in notifications.notifications"
        :key="notification.id"
        :class="`notification notification--${notification.type}`"
      >
        {{ notification.message }}
        <button @click="notifications.removeNotification(notification.id)">
          ×
        </button>
      </div>
    </div>
    
    <button @click="showSuccess">Show Success</button>
    <button @click="loadData">Load Data</button>
  </div>
</template>

<script setup>
import { inject } from 'vue'
import { THEME_KEY, USER_KEY, API_KEY, NOTIFICATION_KEY } from './keys'

// Inject with default values
const theme = inject(THEME_KEY, {
  mode: 'light',
  colors: { primary: '#000' }
})

const user = inject(USER_KEY, {
  name: 'Guest',
  permissions: []
})

// Inject with Symbol keys
const api = inject(API_KEY)
const notifications = inject(NOTIFICATION_KEY)

const showSuccess = () => {
  notifications.addNotification('Operation successful!', 'success')
}

const loadData = async () => {
  try {
    notifications.addNotification('Loading data...', 'info')
    const data = await api.methods.get('users')
    notifications.addNotification('Data loaded successfully', 'success')
  } catch (error) {
    notifications.addNotification('Failed to load data', 'error')
  }
}
</script>
```

**Reactivity with Provide/Inject**
Maintain reactivity when sharing data through provide/inject for live updates.

**Real-world usage:** Global state management, real-time data updates, shared reactive configurations, and live user preferences.

```vue
<!-- StateProvider.vue -->
<template>
  <div class="state-provider">
    <slot></slot>
  </div>
</template>

<script setup>
import { provide, ref, reactive, computed, readonly } from 'vue'

// Reactive state management
const appState = reactive({
  loading: false,
  error: null,
  user: {
    isAuthenticated: false,
    profile: null
  },
  settings: {
    theme: 'light',
    language: 'en',
    notifications: true
  },
  cart: {
    items: [],
    total: 0
  }
})

// Computed values
const cartItemCount = computed(() => appState.cart.items.length)
const isLoggedIn = computed(() => appState.user.isAuthenticated)

// Actions
const actions = {
  // User actions
  login: async (credentials) => {
    appState.loading = true
    try {
      // Simulate API call
      const user = await authenticate(credentials)
      appState.user.isAuthenticated = true
      appState.user.profile = user
    } catch (error) {
      appState.error = error.message
    } finally {
      appState.loading = false
    }
  },
  
  logout: () => {
    appState.user.isAuthenticated = false
    appState.user.profile = null
    appState.cart.items = []
    appState.cart.total = 0
  },
  
  // Settings actions
  updateSettings: (newSettings) => {
    Object.assign(appState.settings, newSettings)
  },
  
  toggleTheme: () => {
    appState.settings.theme = appState.settings.theme === 'light' ? 'dark' : 'light'
  },
  
  // Cart actions
  addToCart: (product) => {
    const existingItem = appState.cart.items.find(item => item.id === product.id)
    if (existingItem) {
      existingItem.quantity++
    } else {
      appState.cart.items.push({ ...product, quantity: 1 })
    }
    updateCartTotal()
  },
  
  removeFromCart: (productId) => {
    const index = appState.cart.items.findIndex(item => item.id === productId)
    if (index > -1) {
      appState.cart.items.splice(index, 1)
      updateCartTotal()
    }
  },
  
  clearCart: () => {
    appState.cart.items = []
    appState.cart.total = 0
  }
}

const updateCartTotal = () => {
  appState.cart.total = appState.cart.items.reduce(
    (sum, item) => sum + (item.price * item.quantity), 0
  )
}

// Provide reactive state (read-only) and actions
provide('appState', readonly(appState))
provide('appActions', actions)
provide('cartItemCount', cartItemCount)
provide('isLoggedIn', isLoggedIn)

// Helper functions
const authenticate = async (credentials) => {
  // Simulate API call
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve({
        id: 1,
        name: 'John Doe',
        email: credentials.email
      })
    }, 1000)
  })
}
</script>

<!-- ShoppingCart.vue - Consumer with reactive updates -->
<template>
  <div class="shopping-cart">
    <h3>Shopping Cart ({{ cartItemCount }})</h3>
    
    <div v-if="appState.cart.items.length === 0">
      Your cart is empty
    </div>
    
    <div v-else>
      <div 
        v-for="item in appState.cart.items"
        :key="item.id"
        class="cart-item"
      >
        <span>{{ item.name }}</span>
        <span>{{ item.quantity }} × ${{ item.price }}</span>
        <button @click="appActions.removeFromCart(item.id)">Remove</button>
      </div>
      
      <div class="cart-total">
        Total: ${{ appState.cart.total.toFixed(2) }}
      </div>
      
      <button @click="appActions.clearCart">Clear Cart</button>
    </div>
  </div>
</template>

<script setup>
import { inject } from 'vue'

// Inject reactive state and actions
const appState = inject('appState')
const appActions = inject('appActions')
const cartItemCount = inject('cartItemCount')
</script>

<!-- UserProfile.vue - Another consumer -->
<template>
  <div class="user-profile" :class="`theme--${appState.settings.theme}`">
    <div v-if="!isLoggedIn">
      <form @submit.prevent="handleLogin">
        <input v-model="credentials.email" type="email" placeholder="Email" />
        <input v-model="credentials.password" type="password" placeholder="Password" />
        <button type="submit" :disabled="appState.loading">
          {{ appState.loading ? 'Logging in...' : 'Login' }}
        </button>
      </form>
    </div>
    
    <div v-else>
      <h2>Welcome, {{ appState.user.profile.name }}!</h2>
      <button @click="appActions.logout">Logout</button>
      <button @click="appActions.toggleTheme">
        Switch to {{ appState.settings.theme === 'light' ? 'dark' : 'light' }} mode
      </button>
    </div>
    
    <div v-if="appState.error" class="error">
      {{ appState.error }}
    </div>
  </div>
</template>

<script setup>
import { ref, inject } from 'vue'

const appState = inject('appState')
const appActions = inject('appActions')
const isLoggedIn = inject('isLoggedIn')

const credentials = ref({
  email: '',
  password: ''
})

const handleLogin = () => {
  appActions.login(credentials.value)
}
</script>
```

**Working with Symbol Keys**
Use Symbol keys for private or plugin-specific injection to avoid naming conflicts.

**Real-world usage:** Plugin development, library isolation, private APIs, and preventing accidental overwrites.

```vue
<!-- plugin-system.js -->
<script>
// Plugin-specific symbols
export const PLUGIN_REGISTRY = Symbol('pluginRegistry')
export const EVENT_BUS = Symbol('eventBus')
export const CACHE_MANAGER = Symbol('cacheManager')

// Create plugin system
export class PluginSystem {
  constructor() {
    this.plugins = new Map()
    this.eventBus = new EventTarget()
    this.cache = new Map()
  }
  
  register(name, plugin) {
    this.plugins.set(name, plugin)
    this.eventBus.dispatchEvent(new CustomEvent('plugin:registered', {
      detail: { name, plugin }
    }))
  }
  
  get(name) {
    return this.plugins.get(name)
  }
  
  emit(event, data) {
    this.eventBus.dispatchEvent(new CustomEvent(event, { detail: data }))
  }
  
  on(event, handler) {
    this.eventBus.addEventListener(event, handler)
  }
  
  setCache(key, value, ttl = 300000) { // 5 minutes default
    this.cache.set(key, {
      value,
      expires: Date.now() + ttl
    })
  }
  
  getCache(key) {
    const item = this.cache.get(key)
    if (item && item.expires > Date.now()) {
      return item.value
    }
    this.cache.delete(key)
    return null
  }
}
</script>

<!-- App.vue - Plugin provider -->
<template>
  <div id="app">
    <PluginDemo />
    <AnotherComponent />
  </div>
</template>

<script setup>
import { provide } from 'vue'
import { 
  PluginSystem, 
  PLUGIN_REGISTRY, 
  EVENT_BUS, 
  CACHE_MANAGER 
} from './plugin-system'

// Create plugin system instance
const pluginSystem = new PluginSystem()

// Register some plugins
pluginSystem.register('analytics', {
  track: (event, data) => {
    console.log('Analytics:', event, data)
  },
  pageView: (path) => {
    console.log('Page view:', path)
  }
})

pluginSystem.register('notifications', {
  show: (message, type = 'info') => {
    console.log(`Notification [${type}]:`, message)
  },
  hide: (id) => {
    console.log('Hide notification:', id)
  }
})

// Provide using Symbol keys
provide(PLUGIN_REGISTRY, pluginSystem)
provide(EVENT_BUS, {
  emit: pluginSystem.emit.bind(pluginSystem),
  on: pluginSystem.on.bind(pluginSystem)
})
provide(CACHE_MANAGER, {
  set: pluginSystem.setCache.bind(pluginSystem),
  get: pluginSystem.getCache.bind(pluginSystem)
})

// Initialize plugins
pluginSystem.on('plugin:registered', (event) => {
  console.log('Plugin registered:', event.detail.name)
})
</script>

<!-- PluginDemo.vue - Consumer -->
<template>
  <div class="plugin-demo">
    <h3>Plugin System Demo</h3>
    <button @click="trackEvent">Track Event</button>
    <button @click="showNotification">Show Notification</button>
    <button @click="cacheData">Cache Data</button>
    <button @click="loadCachedData">Load Cached Data</button>
    
    <div v-if="cachedValue">
      Cached value: {{ cachedValue }}
    </div>
  </div>
</template>

<script setup>
import { ref, inject, onMounted } from 'vue'
import { PLUGIN_REGISTRY, EVENT_BUS, CACHE_MANAGER } from './plugin-system'

// Inject using Symbol keys
const pluginRegistry = inject(PLUGIN_REGISTRY)
const eventBus = inject(EVENT_BUS)
const cacheManager = inject(CACHE_MANAGER)

const cachedValue = ref(null)

// Use plugins
const trackEvent = () => {
  const analytics = pluginRegistry.get('analytics')
  analytics?.track('button_click', { button: 'track_event' })
}

const showNotification = () => {
  const notifications = pluginRegistry.get('notifications')
  notifications?.show('Hello from plugin!', 'success')
}

const cacheData = () => {
  const data = { timestamp: new Date().toISOString(), random: Math.random() }
  cacheManager.set('demo-data', data, 10000) // 10 seconds
  cachedValue.value = data
}

const loadCachedData = () => {
  const data = cacheManager.get('demo-data')
  cachedValue.value = data
}

// Listen to plugin events
onMounted(() => {
  eventBus.on('plugin:registered', (event) => {
    console.log('New plugin available:', event.detail.name)
  })
  
  // Custom events
  eventBus.on('data:updated', (event) => {
    console.log('Data updated:', event.detail)
  })
})
</script>

<!-- AnotherComponent.vue - Another consumer -->
<template>
  <div class="another-component">
    <p>This component also has access to the same plugin system</p>
    <button @click="emitCustomEvent">Emit Custom Event</button>
  </div>
</template>

<script setup>
import { inject } from 'vue'
import { EVENT_BUS } from './plugin-system'

const eventBus = inject(EVENT_BUS)

const emitCustomEvent = () => {
  eventBus.emit('data:updated', {
    source: 'AnotherComponent',
    timestamp: new Date()
  })
}
</script>
```

---

### 12. **Component Lifecycle**

#### Options API Lifecycle

**`beforeCreate`**
Called before component instance is created, before data observation and event setup.

**Real-world usage:** Plugin initialization, global event listeners setup, and early configuration.

```vue
<script>
export default {
  beforeCreate() {
    console.log('beforeCreate: Component instance not yet created')
    // this.$data is undefined
    // this.$el is undefined
    // Props are not available yet
    
    // Initialize plugins or global settings
    this.$plugin = new SomePlugin()
  },
  
  data() {
    return {
      message: 'Hello'
    }
  }
}
</script>
```

**`created`**
Called after component instance is created, data observation is set up, but before mounting.

**Real-world usage:** API calls, data initialization, and setting up watchers.

```vue
<script>
export default {
  data() {
    return {
      users: [],
      loading: false
    }
  },
  
  created() {
    console.log('created: Instance created, data available')
    // this.$data is available
    // this.$el is still undefined
    
    // Fetch initial data
    this.fetchUsers()
    
    // Set up watchers
    this.$watch('searchTerm', this.handleSearch)
  },
  
  methods: {
    async fetchUsers() {
      this.loading = true
      this.users = await api.getUsers()
      this.loading = false
    }
  }
}
</script>
```

**`beforeMount`**
Called before the component is mounted to the DOM.

**Real-world usage:** Last-minute setup before DOM creation, preparing external libraries.

```vue
<script>
export default {
  beforeMount() {
    console.log('beforeMount: About to mount to DOM')
    // this.$el is still undefined
    
    // Prepare external libraries
    this.initializeChart()
    
    // Set loading states
    this.ismounting = true
  }
}
</script>
```

**`mounted`**
Called after component is mounted to the DOM.

**Real-world usage:** DOM manipulation, third-party library initialization, focus management.

```vue
<template>
  <div>
    <input ref="searchInput" />
    <div ref="chartContainer"></div>
  </div>
</template>

<script>
export default {
  mounted() {
    console.log('mounted: Component mounted to DOM')
    // this.$el is available
    // DOM elements are accessible
    
    // Focus input
    this.$refs.searchInput.focus()
    
    // Initialize chart library
    this.chart = new Chart(this.$refs.chartContainer, {
      type: 'line',
      data: this.chartData
    })
    
    // Add event listeners
    window.addEventListener('resize', this.handleResize)
  }
}
</script>
```

**`beforeUpdate`**
Called before component re-renders due to reactive data changes.

**Real-world usage:** Capturing DOM state before updates, preparing for changes.

```vue
<script>
export default {
  beforeUpdate() {
    console.log('beforeUpdate: About to update')
    
    // Capture scroll position before update
    this.scrollPosition = this.$el.scrollTop
    
    // Store current selection
    this.selectedElements = this.getSelectedElements()
  }
}
</script>
```

**`updated`**
Called after component re-renders and DOM is updated.

**Real-world usage:** DOM manipulation after updates, third-party library updates.

```vue
<script>
export default {
  updated() {
    console.log('updated: Component updated')
    
    // Restore scroll position
    this.$el.scrollTop = this.scrollPosition
    
    // Update chart with new data
    if (this.chart) {
      this.chart.update()
    }
    
    // Re-apply focus or selection
    this.restoreSelection()
  }
}
</script>
```

**`beforeUnmount`**
Called before component is unmounted and destroyed.

**Real-world usage:** Cleanup, removing event listeners, canceling timers.

```vue
<script>
export default {
  beforeUnmount() {
    console.log('beforeUnmount: About to unmount')
    
    // Cancel ongoing requests
    this.cancelPendingRequests()
    
    // Save state to localStorage
    localStorage.setItem('componentState', JSON.stringify(this.state))
  }
}
</script>
```

**`unmounted`**
Called after component is unmounted and destroyed.

**Real-world usage:** Final cleanup, removing global event listeners, destroying libraries.

```vue
<script>
export default {
  unmounted() {
    console.log('unmounted: Component destroyed')
    
    // Remove global event listeners
    window.removeEventListener('resize', this.handleResize)
    
    // Destroy chart instance
    if (this.chart) {
      this.chart.destroy()
    }
    
    // Clear intervals/timeouts
    clearInterval(this.timer)
  }
}
</script>
```

**`errorCaptured`**
Called when an error is captured from a descendant component.

**Real-world usage:** Error boundaries, logging errors, graceful error handling.

```vue
<script>
export default {
  errorCaptured(error, instance, info) {
    console.log('errorCaptured:', error, info)
    
    // Log error to service
    this.logError(error, {
      component: instance.$options.name,
      errorInfo: info
    })
    
    // Show user-friendly error message
    this.showErrorMessage('Something went wrong')
    
    // Return false to stop error propagation
    return false
  }
}
</script>
```

**`renderTracked`**
Called when a reactive dependency is tracked during render (development only).

**Real-world usage:** Debugging reactivity, performance optimization, dependency tracking.

```vue
<script>
export default {
  renderTracked(event) {
    console.log('renderTracked:', event)
    // event.target - the reactive object
    // event.type - 'get'
    // event.key - the property key
  }
}
</script>
```

**`renderTriggered`**
Called when a reactive dependency triggers a re-render (development only).

**Real-world usage:** Debugging unnecessary re-renders, performance optimization.

```vue
<script>
export default {
  renderTriggered(event) {
    console.log('renderTriggered:', event)
    // event.target - the reactive object
    // event.type - 'set' | 'add' | 'delete' | 'clear'
    // event.key - the property key
    // event.oldValue - previous value
    // event.newValue - new value
  }
}
</script>
```

#### Composition API Lifecycle

**`onBeforeMount`**
Composition API equivalent of `beforeMount`.

**Real-world usage:** Setup before DOM creation, external library preparation.

```vue
<script setup>
import { onBeforeMount, ref } from 'vue'

const isReady = ref(false)

onBeforeMount(() => {
  console.log('onBeforeMount: Preparing to mount')
  
  // Initialize state
  isReady.value = false
  
  // Prepare external resources
  prepareChart()
})

const prepareChart = () => {
  // Chart preparation logic
}
</script>
```

**`onMounted`**
Composition API equivalent of `mounted`.

**Real-world usage:** DOM access, library initialization, API calls requiring DOM.

```vue
<template>
  <div>
    <input ref="inputRef" />
    <canvas ref="canvasRef"></canvas>
  </div>
</template>

<script setup>
import { onMounted, ref } from 'vue'

const inputRef = ref(null)
const canvasRef = ref(null)

onMounted(() => {
  console.log('onMounted: Component mounted')
  
  // Focus input
  inputRef.value.focus()
  
  // Initialize canvas
  const ctx = canvasRef.value.getContext('2d')
  initializeCanvas(ctx)
  
  // Set up global listeners
  window.addEventListener('keydown', handleKeydown)
})

const handleKeydown = (event) => {
  if (event.key === 'Escape') {
    inputRef.value.blur()
  }
}
</script>
```

**`onBeforeUpdate`**
Called before component updates due to reactive changes.

**Real-world usage:** Capturing state before updates, preparing for DOM changes.

```vue
<script setup>
import { onBeforeUpdate, ref } from 'vue'

const scrollPosition = ref(0)

onBeforeUpdate(() => {
  console.log('onBeforeUpdate: About to update')
  
  // Capture current state
  if (document.querySelector('.scrollable')) {
    scrollPosition.value = document.querySelector('.scrollable').scrollTop
  }
})
</script>
```

**`onUpdated`**
Called after component updates and DOM is patched.

**Real-world usage:** DOM manipulation after updates, external library synchronization.

```vue
<script setup>
import { onUpdated, nextTick } from 'vue'

onUpdated(async () => {
  console.log('onUpdated: Component updated')
  
  // Wait for DOM to be fully updated
  await nextTick()
  
  // Restore scroll position
  const scrollable = document.querySelector('.scrollable')
  if (scrollable) {
    scrollable.scrollTop = scrollPosition.value
  }
  
  // Update external library
  updateChart()
})
</script>
```

**`onBeforeUnmount`**
Called before component is unmounted.

**Real-world usage:** Save state, cancel operations, prepare for cleanup.

```vue
<script setup>
import { onBeforeUnmount, ref } from 'vue'

const componentState = ref({})

onBeforeUnmount(() => {
  console.log('onBeforeUnmount: Preparing to unmount')
  
  // Save state
  localStorage.setItem('appState', JSON.stringify(componentState.value))
  
  // Cancel pending operations
  if (pendingRequest.value) {
    pendingRequest.value.abort()
  }
})
</script>
```

**`onUnmounted`**
Called after component is unmounted.

**Real-world usage:** Final cleanup, remove listeners, destroy instances.

```vue
<script setup>
import { onMounted, onUnmounted, ref } from 'vue'

const timer = ref(null)
const chartInstance = ref(null)

onMounted(() => {
  // Set up timer
  timer.value = setInterval(() => {
    console.log('Timer tick')
  }, 1000)
  
  // Create chart
  chartInstance.value = new Chart()
})

onUnmounted(() => {
  console.log('onUnmounted: Cleanup')
  
  // Clear timer
  if (timer.value) {
    clearInterval(timer.value)
  }
  
  // Destroy chart
  if (chartInstance.value) {
    chartInstance.value.destroy()
  }
  
  // Remove global listeners
  window.removeEventListener('resize', handleResize)
})
</script>
```

**`onErrorCaptured`**
Capture errors from descendant components.

**Real-world usage:** Error boundaries, error logging, graceful degradation.

```vue
<script setup>
import { onErrorCaptured, ref } from 'vue'

const hasError = ref(false)
const errorMessage = ref('')

onErrorCaptured((error, instance, info) => {
  console.log('onErrorCaptured:', error)
  
  // Set error state
  hasError.value = true
  errorMessage.value = error.message
  
  // Log to error service
  logError({
    error: error.message,
    stack: error.stack,
    component: instance?.$options.name,
    info
  })
  
  // Prevent error propagation
  return false
})
</script>

<template>
  <div v-if="hasError" class="error-boundary">
    <h3>Something went wrong</h3>
    <p>{{ errorMessage }}</p>
    <button @click="hasError = false">Try Again</button>
  </div>
  <slot v-else></slot>
</template>
```

**`onRenderTracked`**
Debug hook for tracking reactive dependencies during render.

**Real-world usage:** Development debugging, performance optimization, dependency analysis.

```vue
<script setup>
import { onRenderTracked, ref } from 'vue'

const count = ref(0)

onRenderTracked((event) => {
  console.log('onRenderTracked:', {
    type: event.type,
    key: event.key,
    target: event.target
  })
  
  // Track which dependencies are being accessed
  if (event.key === 'count') {
    console.log('Count dependency tracked')
  }
})
</script>
```

**`onRenderTriggered`**
Debug hook for tracking what triggers re-renders.

**Real-world usage:** Performance debugging, identifying unnecessary re-renders, optimization.

```vue
<script setup>
import { onRenderTriggered, ref } from 'vue'

const data = ref({ count: 0, name: 'test' })

onRenderTriggered((event) => {
  console.log('onRenderTriggered:', {
    type: event.type,
    key: event.key,
    oldValue: event.oldValue,
    newValue: event.newValue
  })
  
  // Identify what caused the re-render
  if (event.key === 'count') {
    console.log(`Count changed from ${event.oldValue} to ${event.newValue}`)
  }
})
</script>
```

---

## **Part V: Composition API**

### 13. **Composition API Fundamentals**

#### `setup()` Function
The entry point for Composition API, where reactive state and logic are defined and returned.

**Real-world usage:** Component initialization, state management, computed properties, and lifecycle hooks.

```vue
<template>
  <div>
    <h1>{{ title }}</h1>
    <p>Count: {{ count }}</p>
    <button @click="increment">Increment</button>
    <input v-model="searchTerm" placeholder="Search..." />
    <ul>
      <li v-for="item in filteredItems" :key="item.id">
        {{ item.name }}
      </li>
    </ul>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'

export default {
  props: {
    initialCount: { type: Number, default: 0 }
  },
  
  setup(props, { emit, attrs, slots, expose }) {
    // Reactive state
    const count = ref(props.initialCount)
    const title = ref('My Component')
    const searchTerm = ref('')
    const items = ref([])
    
    // Computed properties
    const filteredItems = computed(() => {
      return items.value.filter(item => 
        item.name.toLowerCase().includes(searchTerm.value.toLowerCase())
      )
    })
    
    // Methods
    const increment = () => {
      count.value++
      emit('count-changed', count.value)
    }
    
    const loadItems = async () => {
      items.value = await fetchItems()
    }
    
    // Lifecycle
    onMounted(() => {
      loadItems()
    })
    
    // Expose public methods
    expose({
      reset: () => count.value = 0
    })
    
    // Return reactive state and methods
    return {
      count,
      title,
      searchTerm,
      filteredItems,
      increment
    }
  }
}

const fetchItems = async () => {
  return [
    { id: 1, name: 'Apple' },
    { id: 2, name: 'Banana' }
  ]
}
</script>
```

```vue
<!-- setup() with async -->
<script>
import { ref, onMounted } from 'vue'

export default {
  async setup() {
    const data = ref(null)
    const loading = ref(true)
    
    // Async operation in setup
    try {
      const response = await fetch('/api/data')
      data.value = await response.json()
    } catch (error) {
      console.error('Failed to load data:', error)
    } finally {
      loading.value = false
    }
    
    return {
      data,
      loading
    }
  }
}
</script>
```

#### `setup()` vs Options API
Compare and contrast the two API styles for component development.

**Real-world usage:** Migration strategies, team preferences, code organization, and project requirements.

```vue
<!-- Options API -->
<template>
  <div>
    <input v-model="searchQuery" />
    <p>{{ formattedResults }}</p>
    <button @click="performSearch">Search</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      searchQuery: '',
      results: [],
      loading: false
    }
  },
  
  computed: {
    formattedResults() {
      return `Found ${this.results.length} results`
    }
  },
  
  watch: {
    searchQuery(newQuery) {
      if (newQuery.length > 2) {
        this.debouncedSearch()
      }
    }
  },
  
  methods: {
    async performSearch() {
      this.loading = true
      this.results = await this.searchAPI(this.searchQuery)
      this.loading = false
    },
    
    searchAPI(query) {
      return fetch(`/api/search?q=${query}`).then(r => r.json())
    }
  },
  
  created() {
    this.debouncedSearch = this.debounce(this.performSearch, 300)
  }
}
</script>
```

```vue
<!-- Composition API equivalent -->
<template>
  <div>
    <input v-model="searchQuery" />
    <p>{{ formattedResults }}</p>
    <button @click="performSearch">Search</button>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import { debounce } from 'lodash-es'

const searchQuery = ref('')
const results = ref([])
const loading = ref(false)

const formattedResults = computed(() => {
  return `Found ${results.value.length} results`
})

const searchAPI = async (query) => {
  const response = await fetch(`/api/search?q=${query}`)
  return response.json()
}

const performSearch = async () => {
  loading.value = true
  results.value = await searchAPI(searchQuery.value)
  loading.value = false
}

const debouncedSearch = debounce(performSearch, 300)

watch(searchQuery, (newQuery) => {
  if (newQuery.length > 2) {
    debouncedSearch()
  }
})
</script>
```

```vue
<!-- Mixed API usage -->
<script>
import { ref, computed } from 'vue'

export default {
  // Options API
  props: {
    userId: Number
  },
  
  emits: ['user-updated'],
  
  // Composition API
  setup(props, { emit }) {
    const user = ref(null)
    const isEditing = ref(false)
    
    const userName = computed(() => {
      return user.value?.name || 'Unknown'
    })
    
    const toggleEdit = () => {
      isEditing.value = !isEditing.value
    }
    
    return {
      user,
      isEditing,
      userName,
      toggleEdit
    }
  },
  
  // Options API methods
  methods: {
    async saveUser() {
      await this.$api.updateUser(this.user)
      this.$emit('user-updated', this.user)
    }
  }
}
</script>
```

#### Composition API Benefits
Understand the advantages of Composition API over Options API.

**Real-world usage:** Better TypeScript support, logic reuse, tree-shaking, and code organization.

```vue
<!-- Better Logic Organization -->
<script setup>
import { ref, computed, watch, onMounted } from 'vue'

// User management logic grouped together
const user = ref(null)
const isLoggedIn = computed(() => !!user.value)
const userPermissions = computed(() => user.value?.permissions || [])

const loginUser = async (credentials) => {
  user.value = await authenticate(credentials)
}

const logoutUser = () => {
  user.value = null
}

// Search functionality grouped together
const searchQuery = ref('')
const searchResults = ref([])
const isSearching = ref(false)

const performSearch = async () => {
  if (!searchQuery.value.trim()) return
  
  isSearching.value = true
  searchResults.value = await searchAPI(searchQuery.value)
  isSearching.value = false
}

watch(searchQuery, debounce(performSearch, 300))

// Shopping cart logic grouped together
const cartItems = ref([])
const cartTotal = computed(() => {
  return cartItems.value.reduce((sum, item) => sum + item.price * item.quantity, 0)
})

const addToCart = (product) => {
  const existing = cartItems.value.find(item => item.id === product.id)
  if (existing) {
    existing.quantity++
  } else {
    cartItems.value.push({ ...product, quantity: 1 })
  }
}

const removeFromCart = (productId) => {
  const index = cartItems.value.findIndex(item => item.id === productId)
  if (index > -1) {
    cartItems.value.splice(index, 1)
  }
}

// Initialization
onMounted(() => {
  loadUserFromStorage()
  initializeCart()
})
</script>
```

```vue
<!-- Better TypeScript Support -->
<script setup lang="ts">
import { ref, computed, Ref } from 'vue'

interface User {
  id: number
  name: string
  email: string
  role: 'admin' | 'user'
}

interface Product {
  id: number
  name: string
  price: number
  category: string
}

// Type-safe reactive refs
const user: Ref<User | null> = ref(null)
const products: Ref<Product[]> = ref([])
const selectedCategory = ref<string>('')

// Type-safe computed
const isAdmin = computed((): boolean => {
  return user.value?.role === 'admin'
})

const filteredProducts = computed((): Product[] => {
  if (!selectedCategory.value) return products.value
  return products.value.filter(p => p.category === selectedCategory.value)
})

// Type-safe functions
const updateUser = (userData: Partial<User>): void => {
  if (user.value) {
    Object.assign(user.value, userData)
  }
}

const addProduct = (product: Product): void => {
  products.value.push(product)
}
</script>
```

#### Logic Reuse and Organization
Extract and reuse logic with composables for better code organization.

**Real-world usage:** Shared business logic, state management, utility functions, and cross-component functionality.

```javascript
// composables/useCounter.js
import { ref, computed } from 'vue'

export function useCounter(initialValue = 0) {
  const count = ref(initialValue)
  
  const increment = () => count.value++
  const decrement = () => count.value--
  const reset = () => count.value = initialValue
  
  const isZero = computed(() => count.value === 0)
  const isPositive = computed(() => count.value > 0)
  
  return {
    count,
    increment,
    decrement,
    reset,
    isZero,
    isPositive
  }
}
```

```javascript
// composables/useApi.js
import { ref, isRef } from 'vue'

export function useApi() {
  const loading = ref(false)
  const error = ref(null)
  
  const execute = async (apiCall) => {
    loading.value = true
    error.value = null
    
    try {
      const result = await apiCall()
      return result
    } catch (err) {
      error.value = err.message
      throw err
    } finally {
      loading.value = false
    }
  }
  
  return {
    loading,
    error,
    execute
  }
}

export function useFetch(url) {
  const data = ref(null)
  const { loading, error, execute } = useApi()
  
  const fetch = async () => {
    const response = await execute(() => 
      window.fetch(isRef(url) ? url.value : url).then(r => r.json())
    )
    data.value = response
    return response
  }
  
  return {
    data,
    loading,
    error,
    fetch
  }
}
```

```vue
<!-- Using composables -->
<template>
  <div>
    <!-- Counter functionality -->
    <div>
      <p>Count: {{ count }}</p>
      <button @click="increment">+</button>
      <button @click="decrement" :disabled="isZero">-</button>
      <button @click="reset">Reset</button>
    </div>
    
    <!-- API functionality -->
    <div>
      <button @click="loadUsers" :disabled="loading">
        {{ loading ? 'Loading...' : 'Load Users' }}
      </button>
      
      <div v-if="error" class="error">{{ error }}</div>
      
      <ul v-if="users">
        <li v-for="user in users" :key="user.id">
          {{ user.name }}
        </li>
      </ul>
    </div>
    
    <!-- Search functionality -->
    <div>
      <input v-model="searchQuery" placeholder="Search..." />
      <ul>
        <li v-for="result in searchResults" :key="result.id">
          {{ result.title }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useCounter } from './composables/useCounter'
import { useFetch } from './composables/useApi'
import { useSearch } from './composables/useSearch'

// Reuse counter logic
const { count, increment, decrement, reset, isZero } = useCounter(10)

// Reuse API logic
const { data: users, loading, error, fetch: loadUsers } = useFetch('/api/users')

// Reuse search logic
const searchQuery = ref('')
const { results: searchResults } = useSearch(searchQuery, '/api/search')
</script>
```

```javascript
// composables/useSearch.js
import { ref, watch, computed } from 'vue'
import { debounce } from 'lodash-es'

export function useSearch(query, endpoint, options = {}) {
  const results = ref([])
  const loading = ref(false)
  const error = ref(null)
  
  const { minLength = 2, debounceMs = 300 } = options
  
  const search = async (searchTerm) => {
    if (searchTerm.length < minLength) {
      results.value = []
      return
    }
    
    loading.value = true
    error.value = null
    
    try {
      const response = await fetch(`${endpoint}?q=${encodeURIComponent(searchTerm)}`)
      results.value = await response.json()
    } catch (err) {
      error.value = err.message
      results.value = []
    } finally {
      loading.value = false
    }
  }
  
  const debouncedSearch = debounce(search, debounceMs)
  
  watch(query, (newQuery) => {
    debouncedSearch(newQuery)
  }, { immediate: true })
  
  const hasResults = computed(() => results.value.length > 0)
  const isEmpty = computed(() => !loading.value && results.value.length === 0 && query.value.length >= minLength)
  
  return {
    results,
    loading,
    error,
    hasResults,
    isEmpty,
    search: debouncedSearch
  }
}
```

---

### 14. **Composition API Features**

#### Reactive References
Create and manage reactive state using `ref()`, `reactive()`, and related APIs.

**Real-world usage:** Component state, form data, UI toggles, and any data that needs reactivity.

```vue
<template>
  <div>
    <input v-model="username" />
    <p>{{ greeting }}</p>
    <button @click="toggleTheme">{{ theme }} mode</button>
    
    <form @submit.prevent="submitForm">
      <input v-model="form.email" type="email" />
      <input v-model="form.password" type="password" />
      <button type="submit">Submit</button>
    </form>
  </div>
</template>

<script setup>
import { ref, reactive, computed, readonly, toRefs } from 'vue'

// Simple reactive references
const username = ref('')
const theme = ref('light')
const isLoggedIn = ref(false)

// Reactive objects
const form = reactive({
  email: '',
  password: '',
  remember: false
})

// Computed values
const greeting = computed(() => {
  return username.value ? `Hello, ${username.value}!` : 'Hello, Guest!'
})

// Methods
const toggleTheme = () => {
  theme.value = theme.value === 'light' ? 'dark' : 'light'
}

const submitForm = () => {
  console.log('Form data:', form)
}

// Destructure reactive object
const { email, password } = toRefs(form)

// Read-only data
const config = readonly({
  apiUrl: 'https://api.example.com',
  version: '1.0.0'
})
</script>
```

#### Lifecycle Hooks
Use lifecycle hooks in Composition API for component lifecycle management.

**Real-world usage:** API calls, event listeners, timers, and resource cleanup.

```vue
<script setup>
import { 
  ref, 
  onMounted, 
  onUpdated, 
  onUnmounted, 
  onBeforeUnmount,
  watch 
} from 'vue'

const data = ref([])
const loading = ref(false)
const timer = ref(null)

// Mount hook - setup
onMounted(async () => {
  console.log('Component mounted')
  
  // Load initial data
  await loadData()
  
  // Set up polling
  timer.value = setInterval(loadData, 30000)
  
  // Add event listeners
  window.addEventListener('online', handleOnline)
  window.addEventListener('offline', handleOffline)
})

// Update hook - sync external libraries
onUpdated(() => {
  console.log('Component updated')
  // Update chart library, etc.
})

// Before unmount - save state
onBeforeUnmount(() => {
  console.log('About to unmount')
  // Save user data
  localStorage.setItem('userData', JSON.stringify(data.value))
})

// Unmount hook - cleanup
onUnmounted(() => {
  console.log('Component unmounted')
  
  // Clear timer
  if (timer.value) {
    clearInterval(timer.value)
  }
  
  // Remove listeners
  window.removeEventListener('online', handleOnline)
  window.removeEventListener('offline', handleOffline)
})

const loadData = async () => {
  loading.value = true
  try {
    const response = await fetch('/api/data')
    data.value = await response.json()
  } catch (error) {
    console.error('Failed to load data:', error)
  } finally {
    loading.value = false
  }
}

const handleOnline = () => {
  console.log('Back online')
  loadData()
}

const handleOffline = () => {
  console.log('Gone offline')
}
</script>
```

#### Template Refs
Access DOM elements and component instances in Composition API.

**Real-world usage:** Focus management, DOM manipulation, third-party library integration.

```vue
<template>
  <div>
    <input ref="inputRef" v-model="query" />
    <button @click="focusInput">Focus Input</button>
    
    <div ref="chartContainer" class="chart"></div>
    
    <ChildComponent ref="childRef" />
    <button @click="callChildMethod">Call Child Method</button>
    
    <!-- Dynamic refs -->
    <ul>
      <li v-for="(item, index) in items" :key="item.id" :ref="el => itemRefs[index] = el">
        {{ item.name }}
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick } from 'vue'

// Template refs
const inputRef = ref(null)
const chartContainer = ref(null)
const childRef = ref(null)

// Dynamic refs
const itemRefs = ref([])
const items = ref([
  { id: 1, name: 'Item 1' },
  { id: 2, name: 'Item 2' }
])

const query = ref('')

// Methods using refs
const focusInput = () => {
  inputRef.value?.focus()
}

const callChildMethod = () => {
  childRef.value?.someMethod()
}

const scrollToItem = (index) => {
  itemRefs.value[index]?.scrollIntoView()
}

// Setup on mount
onMounted(async () => {
  // Initialize chart
  if (chartContainer.value) {
    initChart(chartContainer.value)
  }
  
  // Wait for DOM update
  await nextTick()
  
  // Access dynamic refs
  console.log('Item elements:', itemRefs.value)
})

const initChart = (container) => {
  // Chart library initialization
  console.log('Initializing chart in:', container)
}
</script>
```

#### `defineProps()` and `defineEmits()`
Define component props and events with type safety and validation.

**Real-world usage:** Component interfaces, type checking, and parent-child communication.

```vue
<template>
  <div class="user-card">
    <img :src="user.avatar" :alt="user.name" />
    <h3>{{ user.name }}</h3>
    <p>{{ user.email }}</p>
    
    <div class="actions">
      <button @click="handleEdit" :disabled="!canEdit">Edit</button>
      <button @click="handleDelete" :disabled="!canDelete">Delete</button>
      <button @click="handleContact">Contact</button>
    </div>
    
    <div v-if="showDetails" class="details">
      <p>Role: {{ user.role }}</p>
      <p>Status: {{ status }}</p>
    </div>
  </div>
</template>

<script setup>
// Define props with validation
const props = defineProps({
  user: {
    type: Object,
    required: true,
    validator: (user) => user && user.id && user.name
  },
  canEdit: {
    type: Boolean,
    default: false
  },
  canDelete: {
    type: Boolean,
    default: false
  },
  status: {
    type: String,
    default: 'active',
    validator: (value) => ['active', 'inactive', 'pending'].includes(value)
  },
  showDetails: Boolean
})

// Define emits with validation
const emit = defineEmits({
  edit: (userId) => typeof userId === 'number',
  delete: (userId) => typeof userId === 'number',
  contact: (userEmail) => typeof userEmail === 'string' && userEmail.includes('@')
})

// Event handlers
const handleEdit = () => {
  emit('edit', props.user.id)
}

const handleDelete = () => {
  if (confirm('Are you sure?')) {
    emit('delete', props.user.id)
  }
}

const handleContact = () => {
  emit('contact', props.user.email)
}
</script>
```

```vue
<!-- TypeScript props and emits -->
<script setup lang="ts">
interface User {
  id: number
  name: string
  email: string
  role: 'admin' | 'user'
  avatar?: string
}

interface Props {
  user: User
  canEdit?: boolean
  canDelete?: boolean
  status?: 'active' | 'inactive' | 'pending'
  showDetails?: boolean
}

interface Emits {
  edit: [userId: number]
  delete: [userId: number]
  contact: [email: string]
}

const props = defineProps<Props>()
const emit = defineEmits<Emits>()

// Props are automatically typed
const handleEdit = () => {
  emit('edit', props.user.id) // TypeScript knows this is number
}
</script>
```

#### `defineExpose()`
Expose component methods and properties to parent components.

**Real-world usage:** Imperative APIs, form validation, component control from parent.

```vue
<!-- ChildComponent.vue -->
<template>
  <div class="modal" :class="{ open: isOpen }">
    <div class="modal-content">
      <h3>{{ title }}</h3>
      <slot></slot>
      <div class="modal-actions">
        <button @click="close">Close</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  title: String
})

const isOpen = ref(false)
const data = ref(null)

// Internal methods
const open = () => {
  isOpen.value = true
}

const close = () => {
  isOpen.value = false
}

const setData = (newData) => {
  data.value = newData
}

const getData = () => {
  return data.value
}

// Computed for internal use
const hasData = computed(() => !!data.value)

// Expose specific methods and properties to parent
defineExpose({
  open,
  close,
  setData,
  getData,
  isOpen: readonly(isOpen), // Read-only access
  hasData
})
</script>
```

```vue
<!-- ParentComponent.vue -->
<template>
  <div>
    <button @click="openModal">Open Modal</button>
    <button @click="setModalData">Set Data</button>
    <button @click="getModalData">Get Data</button>
    
    <ChildComponent ref="modalRef" title="My Modal">
      <p>Modal content here</p>
    </ChildComponent>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const modalRef = ref(null)

const openModal = () => {
  modalRef.value?.open()
}

const setModalData = () => {
  modalRef.value?.setData({ message: 'Hello from parent' })
}

const getModalData = () => {
  const data = modalRef.value?.getData()
  console.log('Modal data:', data)
}
</script>
```

#### `useSlots()` and `useAttrs()`
Access slots and attributes programmatically in Composition API.

**Real-world usage:** Dynamic slot rendering, attribute forwarding, and conditional layouts.

```vue
<template>
  <div class="wrapper" :class="wrapperClasses">
    <!-- Conditional slot rendering -->
    <header v-if="slots.header" class="header">
      <slot name="header"></slot>
    </header>
    
    <nav v-if="hasNavigation" class="navigation">
      <slot name="navigation"></slot>
    </nav>
    
    <main class="content" v-bind="contentAttrs">
      <slot></slot>
    </main>
    
    <aside v-if="slots.sidebar" class="sidebar">
      <slot name="sidebar"></slot>
    </aside>
    
    <footer v-if="slots.footer" class="footer">
      <slot name="footer"></slot>
    </footer>
    
    <!-- Debug info -->
    <div v-if="showDebug" class="debug">
      <p>Available slots: {{ availableSlots }}</p>
      <p>Forwarded attrs: {{ forwardedAttrs }}</p>
    </div>
  </div>
</template>

<script setup>
import { computed, useSlots, useAttrs } from 'vue'

const props = defineProps({
  layout: { type: String, default: 'default' },
  showDebug: Boolean
})

const slots = useSlots()
const attrs = useAttrs()

// Computed based on available slots
const hasNavigation = computed(() => {
  return !!slots.navigation
})

const availableSlots = computed(() => {
  return Object.keys(slots)
})

// Dynamic classes based on slots
const wrapperClasses = computed(() => ({
  [`layout-${props.layout}`]: true,
  'has-header': !!slots.header,
  'has-sidebar': !!slots.sidebar,
  'has-footer': !!slots.footer
}))

// Filter attributes for specific elements
const contentAttrs = computed(() => {
  const { class: className, style, id, ...rest } = attrs
  return {
    id: id ? `${id}-content` : undefined,
    ...rest
  }
})

const forwardedAttrs = computed(() => {
  return Object.keys(attrs)
})
</script>
```

#### `useCssVars()`
Reactive CSS custom properties for dynamic styling.

**Real-world usage:** Theme systems, dynamic colors, responsive design, and component styling.

```vue
<template>
  <div class="themed-component">
    <div class="header">
      <h2>{{ title }}</h2>
    </div>
    
    <div class="content">
      <p>This component uses reactive CSS variables</p>
      <button @click="randomizeTheme">Randomize Theme</button>
    </div>
    
    <div class="controls">
      <label>
        Primary Color:
        <input v-model="theme.primary" type="color" />
      </label>
      <label>
        Font Size:
        <input v-model.number="theme.fontSize" type="range" min="12" max="24" />
      </label>
      <label>
        Border Radius:
        <input v-model.number="theme.borderRadius" type="range" min="0" max="20" />
      </label>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, useCssVars } from 'vue'

const props = defineProps({
  title: { type: String, default: 'Themed Component' }
})

// Reactive theme object
const theme = reactive({
  primary: '#007bff',
  secondary: '#6c757d',
  fontSize: 16,
  borderRadius: 8,
  spacing: 16
})

// Bind reactive values to CSS custom properties
useCssVars(() => ({
  primary: theme.primary,
  secondary: theme.secondary,
  fontSize: theme.fontSize + 'px',
  borderRadius: theme.borderRadius + 'px',
  spacing: theme.spacing + 'px'
}))

const randomizeTheme = () => {
  const colors = ['#007bff', '#28a745', '#dc3545', '#ffc107', '#17a2b8']
  theme.primary = colors[Math.floor(Math.random() * colors.length)]
  theme.fontSize = Math.floor(Math.random() * 8) + 14
  theme.borderRadius = Math.floor(Math.random() * 16) + 4
}
</script>

<style scoped>
.themed-component {
  padding: var(--spacing);
  border: 2px solid var(--primary);
  border-radius: var(--borderRadius);
  font-size: var(--fontSize);
}

.header {
  background-color: var(--primary);
  color: white;
  padding: var(--spacing);
  margin: calc(var(--spacing) * -1) calc(var(--spacing) * -1) var(--spacing);
  border-radius: var(--borderRadius) var(--borderRadius) 0 0;
}

.content {
  margin: var(--spacing) 0;
}

.controls {
  display: flex;
  gap: var(--spacing);
  margin-top: var(--spacing);
}

.controls label {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

button {
  background-color: var(--primary);
  color: white;
  border: none;
  padding: 8px var(--spacing);
  border-radius: var(--borderRadius);
  cursor: pointer;
}
</style>
```

---

### 15. **Composables**

#### What are Composables
Composables are reusable functions that encapsulate and compose stateful logic using Vue's Composition API.

**Real-world usage:** Shared business logic, state management, API interactions, and cross-component functionality.

```javascript
// Basic composable structure
import { ref, computed } from 'vue'

export function useExample() {
  // Reactive state
  const state = ref(0)
  
  // Computed properties
  const doubled = computed(() => state.value * 2)
  
  // Methods
  const increment = () => state.value++
  
  // Return public API
  return {
    state,
    doubled,
    increment
  }
}
```

```vue
<!-- Using composables in components -->
<template>
  <div>
    <p>Counter: {{ count }}</p>
    <p>Doubled: {{ doubled }}</p>
    <button @click="increment">Increment</button>
  </div>
</template>

<script setup>
import { useCounter } from './composables/useCounter'

const { count, doubled, increment } = useCounter(0)
</script>
```

#### Creating Custom Composables
Build custom composables for specific business logic and reusable functionality.

**Real-world usage:** API calls, form handling, local storage, and complex state management.

```javascript
// composables/useApi.js
import { ref, isRef, unref } from 'vue'

export function useApi(baseURL = '') {
  const loading = ref(false)
  const error = ref(null)
  
  const request = async (url, options = {}) => {
    loading.value = true
    error.value = null
    
    try {
      const fullUrl = baseURL + (isRef(url) ? unref(url) : url)
      const response = await fetch(fullUrl, {
        headers: { 'Content-Type': 'application/json' },
        ...options
      })
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`)
      }
      
      return await response.json()
    } catch (err) {
      error.value = err.message
      throw err
    } finally {
      loading.value = false
    }
  }
  
  const get = (url) => request(url)
  const post = (url, data) => request(url, {
    method: 'POST',
    body: JSON.stringify(data)
  })
  
  return { loading, error, get, post, request }
}
```

```javascript
// composables/useLocalStorage.js
import { ref, watch } from 'vue'

export function useLocalStorage(key, defaultValue) {
  const storedValue = localStorage.getItem(key)
  const initialValue = storedValue ? JSON.parse(storedValue) : defaultValue
  
  const value = ref(initialValue)
  
  // Watch for changes and update localStorage
  watch(
    value,
    (newValue) => {
      localStorage.setItem(key, JSON.stringify(newValue))
    },
    { deep: true }
  )
  
  return value
}
```

```javascript
// composables/useForm.js
import { reactive, computed } from 'vue'

export function useForm(initialValues = {}, validationRules = {}) {
  const values = reactive({ ...initialValues })
  const errors = reactive({})
  const touched = reactive({})
  
  const isValid = computed(() => Object.keys(errors).length === 0)
  const isDirty = computed(() => Object.keys(touched).length > 0)
  
  const validate = (field, value) => {
    const rules = validationRules[field]
    if (!rules) return true
    
    for (const rule of rules) {
      const result = rule(value)
      if (result !== true) {
        errors[field] = result
        return false
      }
    }
    
    delete errors[field]
    return true
  }
  
  const setFieldValue = (field, value) => {
    values[field] = value
    touched[field] = true
    validate(field, value)
  }
  
  const reset = () => {
    Object.assign(values, initialValues)
    Object.keys(errors).forEach(key => delete errors[key])
    Object.keys(touched).forEach(key => delete touched[key])
  }
  
  return {
    values,
    errors,
    touched,
    isValid,
    isDirty,
    setFieldValue,
    validate,
    reset
  }
}
```

#### Composable Conventions
Follow best practices and naming conventions for maintainable composables.

**Real-world usage:** Team consistency, code readability, and composable discoverability.

```javascript
// ✅ Good composable conventions

// 1. Use "use" prefix
export function useCounter() { }
export function useApi() { }
export function useLocalStorage() { }

// 2. Accept reactive parameters
export function useSearch(query, options = {}) {
  const searchQuery = isRef(query) ? query : ref(query)
  // ... implementation
}

// 3. Return consistent object structure
export function useData() {
  return {
    // State
    data: ref(null),
    loading: ref(false),
    error: ref(null),
    
    // Actions
    fetch: () => {},
    refresh: () => {},
    
    // Computed
    isEmpty: computed(() => !data.value?.length)
  }
}

// 4. Handle cleanup properly
export function useEventListener(target, event, handler) {
  onMounted(() => {
    target.addEventListener(event, handler)
  })
  
  onUnmounted(() => {
    target.removeEventListener(event, handler)
  })
}

// 5. Make composables flexible
export function useFetch(url, options = {}) {
  const {
    immediate = true,
    method = 'GET',
    ...fetchOptions
  } = options
  
  const data = ref(null)
  const loading = ref(false)
  
  const execute = async () => {
    // Implementation
  }
  
  if (immediate) {
    execute()
  }
  
  return { data, loading, execute }
}
```

```javascript
// ❌ Avoid these patterns

// Don't use non-reactive parameters when reactivity is expected
export function useBadSearch(query) {
  // query won't react to changes
  const results = ref([])
  // ... 
}

// Don't forget cleanup
export function useBadEventListener(element, event, handler) {
  element.addEventListener(event, handler)
  // Missing cleanup - memory leak
}

// Don't return inconsistent APIs
export function useInconsistent() {
  // Sometimes returns object, sometimes array
  if (someCondition) {
    return { data, loading }
  }
  return [data, loading]
}
```

#### State Management with Composables
Use composables for application-wide state management and shared logic.

**Real-world usage:** Global state, user authentication, shopping carts, and application settings.

```javascript
// stores/useUserStore.js
import { ref, computed, readonly } from 'vue'

// Global state
const user = ref(null)
const isAuthenticated = computed(() => !!user.value)
const permissions = computed(() => user.value?.permissions || [])

export function useUserStore() {
  const login = async (credentials) => {
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(credentials)
      })
      
      if (response.ok) {
        user.value = await response.json()
        localStorage.setItem('user', JSON.stringify(user.value))
      }
    } catch (error) {
      console.error('Login failed:', error)
    }
  }
  
  const logout = () => {
    user.value = null
    localStorage.removeItem('user')
  }
  
  const hasPermission = (permission) => {
    return permissions.value.includes(permission)
  }
  
  // Initialize from localStorage
  const initUser = () => {
    const saved = localStorage.getItem('user')
    if (saved) {
      user.value = JSON.parse(saved)
    }
  }
  
  return {
    // Read-only state
    user: readonly(user),
    isAuthenticated,
    permissions,
    
    // Actions
    login,
    logout,
    hasPermission,
    initUser
  }
}
```

```javascript
// stores/useCartStore.js
import { ref, computed } from 'vue'

const items = ref([])

export function useCartStore() {
  const total = computed(() => {
    return items.value.reduce((sum, item) => sum + item.price * item.quantity, 0)
  })
  
  const itemCount = computed(() => {
    return items.value.reduce((sum, item) => sum + item.quantity, 0)
  })
  
  const addItem = (product, quantity = 1) => {
    const existing = items.value.find(item => item.id === product.id)
    
    if (existing) {
      existing.quantity += quantity
    } else {
      items.value.push({ ...product, quantity })
    }
  }
  
  const removeItem = (productId) => {
    const index = items.value.findIndex(item => item.id === productId)
    if (index > -1) {
      items.value.splice(index, 1)
    }
  }
  
  const updateQuantity = (productId, quantity) => {
    const item = items.value.find(item => item.id === productId)
    if (item) {
      item.quantity = quantity
    }
  }
  
  const clear = () => {
    items.value = []
  }
  
  return {
    items: readonly(items),
    total,
    itemCount,
    addItem,
    removeItem,
    updateQuantity,
    clear
  }
}
```

#### Popular Composable Libraries (VueUse)
Leverage existing composable libraries for common functionality.

**Real-world usage:** Utility functions, browser APIs, animations, and development productivity.

```vue
<template>
  <div>
    <!-- Mouse position -->
    <p>Mouse: {{ x }}, {{ y }}</p>
    
    <!-- Online status -->
    <p>Status: {{ isOnline ? 'Online' : 'Offline' }}</p>
    
    <!-- Local storage -->
    <input v-model="settings.theme" />
    <p>Theme: {{ settings.theme }}</p>
    
    <!-- Dark mode -->
    <button @click="toggleDark()">
      {{ isDark ? 'Light' : 'Dark' }} Mode
    </button>
    
    <!-- Window size -->
    <p>Window: {{ width }} x {{ height }}</p>
    
    <!-- Clipboard -->
    <button @click="copy('Hello World')">Copy Text</button>
    <p v-if="copied">Copied!</p>
  </div>
</template>

<script setup>
import {
  useMouse,
  useOnline,
  useLocalStorage,
  useDark,
  useToggle,
  useWindowSize,
  useClipboard
} from '@vueuse/core'

// Mouse position tracking
const { x, y } = useMouse()

// Online/offline status
const isOnline = useOnline()

// Reactive localStorage
const settings = useLocalStorage('app-settings', {
  theme: 'light',
  language: 'en'
})

// Dark mode
const isDark = useDark()
const toggleDark = useToggle(isDark)

// Window dimensions
const { width, height } = useWindowSize()

// Clipboard operations
const { copy, copied } = useClipboard()
</script>
```

```javascript
// Custom composables with VueUse
import { ref } from 'vue'
import { useEventListener, useThrottledRef } from '@vueuse/core'

export function useScrollSpy(targets) {
  const activeId = ref('')
  
  const observer = new IntersectionObserver((entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        activeId.value = entry.target.id
      }
    })
  })
  
  onMounted(() => {
    targets.forEach((target) => {
      const element = document.getElementById(target)
      if (element) observer.observe(element)
    })
  })
  
  onUnmounted(() => {
    observer.disconnect()
  })
  
  return { activeId }
}

export function useInfiniteScroll(fetchMore) {
  const loading = ref(false)
  
  useEventListener('scroll', useThrottledRef(async () => {
    const { scrollTop, scrollHeight, clientHeight } = document.documentElement
    
    if (scrollTop + clientHeight >= scrollHeight - 5 && !loading.value) {
      loading.value = true
      await fetchMore()
      loading.value = false
    }
  }, 200))
  
  return { loading }
}
```

#### Testing Composables
Write tests for composables to ensure reliability and maintainability.

**Real-world usage:** Unit testing, integration testing, and test-driven development.

```javascript
// composables/__tests__/useCounter.test.js
import { describe, it, expect } from 'vitest'
import { useCounter } from '../useCounter'

describe('useCounter', () => {
  it('should initialize with default value', () => {
    const { count } = useCounter()
    expect(count.value).toBe(0)
  })
  
  it('should initialize with custom value', () => {
    const { count } = useCounter(10)
    expect(count.value).toBe(10)
  })
  
  it('should increment count', () => {
    const { count, increment } = useCounter(5)
    increment()
    expect(count.value).toBe(6)
  })
  
  it('should decrement count', () => {
    const { count, decrement } = useCounter(5)
    decrement()
    expect(count.value).toBe(4)
  })
  
  it('should reset to initial value', () => {
    const { count, increment, reset } = useCounter(3)
    increment()
    increment()
    expect(count.value).toBe(5)
    
    reset()
    expect(count.value).toBe(3)
  })
})
```

```javascript
// composables/__tests__/useApi.test.js
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { useApi } from '../useApi'

// Mock fetch
global.fetch = vi.fn()

describe('useApi', () => {
  beforeEach(() => {
    fetch.mockClear()
  })
  
  it('should handle successful GET request', async () => {
    const mockData = { id: 1, name: 'Test' }
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => mockData
    })
    
    const { get, loading, error } = useApi()
    
    expect(loading.value).toBe(false)
    
    const result = await get('/users/1')
    
    expect(result).toEqual(mockData)
    expect(loading.value).toBe(false)
    expect(error.value).toBe(null)
    expect(fetch).toHaveBeenCalledWith('/users/1', {
      headers: { 'Content-Type': 'application/json' }
    })
  })
  
  it('should handle API errors', async () => {
    fetch.mockResolvedValueOnce({
      ok: false,
      status: 404,
      statusText: 'Not Found'
    })
    
    const { get, error } = useApi()
    
    await expect(get('/users/999')).rejects.toThrow('HTTP 404: Not Found')
    expect(error.value).toBe('HTTP 404: Not Found')
  })
  
  it('should handle network errors', async () => {
    const networkError = new Error('Network error')
    fetch.mockRejectedValueOnce(networkError)
    
    const { get, error } = useApi()
    
    await expect(get('/users')).rejects.toThrow('Network error')
    expect(error.value).toBe('Network error')
  })
})
```

```javascript
// composables/__tests__/useLocalStorage.test.js
import { describe, it, expect, beforeEach, vi } from 'vitest'
import { useLocalStorage } from '../useLocalStorage'

// Mock localStorage
const localStorageMock = {
  getItem: vi.fn(),
  setItem: vi.fn(),
  removeItem: vi.fn()
}
global.localStorage = localStorageMock

describe('useLocalStorage', () => {
  beforeEach(() => {
    localStorageMock.getItem.mockClear()
    localStorageMock.setItem.mockClear()
  })
  
  it('should return default value when no stored value', () => {
    localStorageMock.getItem.mockReturnValue(null)
    
    const value = useLocalStorage('test-key', 'default')
    
    expect(value.value).toBe('default')
    expect(localStorageMock.getItem).toHaveBeenCalledWith('test-key')
  })
  
  it('should return stored value when available', () => {
    localStorageMock.getItem.mockReturnValue('"stored-value"')
    
    const value = useLocalStorage('test-key', 'default')
    
    expect(value.value).toBe('stored-value')
  })
  
  it('should update localStorage when value changes', async () => {
    localStorageMock.getItem.mockReturnValue(null)
    
    const value = useLocalStorage('test-key', 'initial')
    value.value = 'updated'
    
    // Wait for watcher to trigger
    await new Promise(resolve => setTimeout(resolve, 0))
    
    expect(localStorageMock.setItem).toHaveBeenCalledWith(
      'test-key',
      '"updated"'
    )
  })
})
```

---

