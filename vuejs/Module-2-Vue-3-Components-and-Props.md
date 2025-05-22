# Module 2: Vue 3 Components and Props

## Core Problem

Building complex applications requires breaking UI into maintainable, reusable components with clear communication patterns while maintaining a declarative approach to DOM manipulation.

## Key Assumptions

- You understand basic Vue 3 concepts from Module 1
- You're familiar with JavaScript fundamentals and ES6+ syntax
- You want to build applications using component-based architecture
- You want to use Tailwind CSS v4 for styling components

## Essential Concepts

### 1. Single-File Components (SFC)

**Concise Explanation:**
Single-File Components (SFCs) are Vue's signature feature, combining HTML template, JavaScript logic, and CSS styles in a single `.vue` file. This encapsulation improves maintainability, provides IDE support, and enables more efficient build-time optimizations.

**Where to Use:**

- Medium to large-scale applications
- When building reusable UI elements
- When component logic and styling are tightly coupled
- For better organization in team environments

**Code Snippet:**

```html
<!-- Button.vue -->
<template>
  <button
    :class="[
      'rounded font-medium transition-colors focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2',
      variant === 'primary'
        ? 'bg-blue-600 text-white hover:bg-blue-700 focus-visible:outline-blue-600'
        : variant === 'secondary'
        ? 'bg-white text-gray-900 ring-1 ring-gray-300 hover:bg-gray-50'
        : 'bg-white text-gray-900 underline hover:text-gray-600',
    ]"
    :disabled="disabled"
    @click="$emit('click')"
  >
    <slot></slot>
  </button>
</template>

<script setup>
  defineProps({
    variant: {
      type: String,
      default: "primary",
      validator: (value) => ["primary", "secondary", "text"].includes(value),
    },
    disabled: {
      type: Boolean,
      default: false,
    },
  });

  defineEmits(["click"]);
</script>
```

**Real-World Example:**
A design system for a SaaS application would define Button, Card, Modal, and other UI components as SFCs. Each component would encapsulate its own logic, styling, and accessibility considerations while providing a consistent API for the rest of the application.

**Common Pitfalls:**

- Creating overly complex components with too many responsibilities
- Not using proper naming conventions for components
- Overusing global styles instead of scoping styles to components
- Using the wrong component organization strategy for the project size
- Keeping too much logic in the template instead of script section

**Frequently Asked Questions:**

1. **Q: What's the difference between `.vue` files and JavaScript components?**

   A: SFCs (`.vue` files) provide template compilation, scoped CSS, better tooling support, and hot module replacement. JavaScript components lack these features and require manual template compilation or render functions, making them more verbose and harder to maintain for complex UIs.

2. **Q: Should I always use Single-File Components?**

   A: For most applications, yes. SFCs provide the best developer experience and maintainability. The only exceptions might be tiny applications, quick prototypes, or environments where build tools can't be used (in which case you can use JavaScript components or template literals).

3. **Q: How are SFCs processed during the build process?**

   A: Vue's build tools (like Vite or Vue CLI) use loaders/plugins to transform SFCs into standard JavaScript modules. The template is compiled into render functions, styles are extracted (and scoped if specified), and the script section becomes the component's options/exports.

### 2. Component Registration (Global vs. Local)

**Concise Explanation:**
Vue components can be registered globally (available throughout the application) or locally (only where explicitly imported). Local registration is preferred for most use cases as it enables tree-shaking and makes component dependencies explicit.

**Where to Use:**

- Global registration: For truly ubiquitous components used throughout the app
- Local registration: For most components to maintain explicit dependencies
- Automatic registration: For convenience in smaller applications

**Code Snippet:**

```js
// Global Registration (main.js)
import { createApp } from "vue";
import BaseButton from "./components/BaseButton.vue";
import BaseInput from "./components/BaseInput.vue";
import App from "./App.vue";

const app = createApp(App);

// Register components globally (avoid for large applications)
app.component("BaseButton", BaseButton);
app.component("BaseInput", BaseInput);

app.mount("#app");
```

```html
<!-- Local Registration (preferred) -->
<template>
  <div>
    <ProductCard
      v-for="product in products"
      :key="product.id"
      :product="product"
    />
  </div>
</template>

<script setup>
  import { ref } from "vue";
  import ProductCard from "./components/ProductCard.vue";

  const products = ref([
    { id: 1, name: "Laptop", price: 999 },
    { id: 2, name: "Phone", price: 699 },
  ]);
</script>
```

**Real-World Example:**
An e-commerce application might globally register truly common components like BaseButton, BaseInput, and Icon, while locally registering more specific components like ProductCard, CartItem, and CheckoutForm in the pages where they're used.

**Common Pitfalls:**

- Over-using global registration which bloats your application
- Forgetting to import locally registered components
- Inconsistent naming conventions across registration types
- Not organizing components into logical directories
- Circular dependencies between components

**Frequently Asked Questions:**

1. **Q: Should I use PascalCase or kebab-case for component names?**

   A: Both work, but Vue recommends PascalCase for component imports and declarations (`ProductCard`), and kebab-case in templates (`<product-card>`). When using SFCs with `<script setup>`, the component name is automatically inferred from the filename.

2. **Q: Is there a performance difference between global and local registration?**

   A: Yes. Globally registered components are included in every bundle that imports the root application, even if they aren't used. Locally registered components are only included where imported, enabling tree-shaking to reduce bundle size.

3. **Q: How can I automatically register multiple components at once?**

   A: You can use dynamic imports with Webpack/Vite's require.context or import.meta features to automatically register components from a directory:

   ```js
   // Automatically register all components in the /components/base/ directory
   const baseComponents = import.meta.glob("./components/base/*.vue", {
     eager: true,
   });
   Object.entries(baseComponents).forEach(([path, component]) => {
     const componentName = path
       .split("/")
       .pop()
       .replace(/\.\w+$/, "");
     app.component(componentName, component.default);
   });
   ```

### 3. Component Lifecycle Hooks

**Concise Explanation:**
Lifecycle hooks are special methods that allow you to execute code at specific stages of a component's existence, from initialization to destruction. They provide control over setup, rendering, updates, and teardown of components.

**Where to Use:**

- `onMounted`: Initialize third-party libraries, fetch data, or access DOM
- `onUpdated`: React to component data changes after DOM updates
- `onUnmounted`: Clean up resources, event listeners, or subscriptions
- `onBeforeMount`/`onBeforeUpdate`: Perform prep work before DOM operations
- `onErrorCaptured`: Handle errors from descendant components

**Code Snippet:**

```html
<template>
  <div class="p-4 bg-white rounded shadow">
    <h2 class="text-lg font-semibold">{{ title }}</h2>
    <div ref="chartContainer" class="h-64 mt-4"></div>
  </div>
</template>

<script setup>
  import { ref, onMounted, onUpdated, onUnmounted, watch } from "vue";
  import ChartLibrary from "chart-library";

  const props = defineProps({
    title: String,
    data: Array,
  });

  const chartContainer = ref(null);
  let chart = null;

  // Called after component is mounted to DOM
  onMounted(() => {
    // Initialize third-party library
    chart = new ChartLibrary(chartContainer.value, {
      data: props.data,
      type: "line",
    });

    console.log("Chart component mounted");
  });

  // Watch for data changes and update chart
  watch(
    () => props.data,
    (newData) => {
      if (chart) {
        chart.updateData(newData);
      }
    },
    { deep: true }
  );

  // Called before component is removed from DOM
  onUnmounted(() => {
    // Clean up to prevent memory leaks
    if (chart) {
      chart.destroy();
      chart = null;
    }

    console.log("Chart component unmounted");
  });
</script>
```

**Real-World Example:**
A dashboard widget uses lifecycle hooks to initialize a chart library on mount, update it when data changes, and clean up resources when the widget is removed from the dashboard.

**Common Pitfalls:**

- Not cleaning up resources in `onUnmounted`, causing memory leaks
- Performing expensive operations in frequently triggered hooks
- Using the wrong hook for a specific use case
- Directly manipulating the DOM outside of the appropriate lifecycle hooks
- Not handling errors in asynchronous operations within hooks

**Frequently Asked Questions:**

1. **Q: What's the difference between onMounted and onBeforeMount?**

   A: `onBeforeMount` runs right before the component is mounted to the DOM, while `onMounted` runs after the DOM has been created and the component has been mounted. If you need to access the rendered DOM, use `onMounted`.

2. **Q: How do lifecycle hooks work with the Composition API vs. Options API?**

   A: In the Options API, lifecycle hooks are declared as methods (e.g., `mounted()`, `updated()`). In the Composition API, they are imported functions prefixed with "on" (e.g., `onMounted()`, `onUpdated()`). The Composition API hooks must be called within `setup()` or `<script setup>`.

3. **Q: When should I use watch vs. onUpdated?**

   A: Use `watch` to react to specific reactive data changes with fine-grained control. Use `onUpdated` when you need to access the updated DOM after any reactive data changes. `watch` is more efficient as it targets specific changes, while `onUpdated` runs after every update.

### 4. Props Definition and Validation

**Concise Explanation:**
Props are custom attributes for passing data from parent to child components. Vue allows defining props with type validation, default values, and custom validators to ensure components receive the correct data format.

**Where to Use:**

- Passing data down the component tree
- Configuring reusable components
- Creating component libraries with clear interfaces
- Enforcing data requirements for components

**Code Snippet:**

```html
<!-- UserProfile.vue -->
<template>
  <div class="p-4 bg-white rounded shadow">
    <div class="flex items-center gap-4">
      <img
        :src="avatarUrl || defaultAvatarUrl"
        :alt="`${user.name}'s avatar`"
        class="w-16 h-16 rounded-full object-cover"
      />
      <div>
        <h2 class="text-xl font-bold">{{ user.name }}</h2>
        <p class="text-gray-600">{{ formattedRole }}</p>
      </div>
    </div>
    <div v-if="showStats" class="mt-4 pt-4 border-t border-gray-200">
      <ul class="grid grid-cols-3 gap-2 text-center">
        <li v-for="(value, key) in user.stats" :key="key">
          <div class="font-semibold">{{ value }}</div>
          <div class="text-sm text-gray-500">{{ key }}</div>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
  import { computed } from "vue";

  // Props definition with validation
  const props = defineProps({
    // Basic type checking
    user: {
      type: Object,
      required: true,
    },

    // With default value
    showStats: {
      type: Boolean,
      default: true,
    },

    // With custom validator
    role: {
      type: String,
      default: "user",
      validator: (value) => {
        return ["user", "admin", "moderator"].includes(value);
      },
    },

    // Optional prop with default
    avatarUrl: {
      type: String,
      default: null,
    },
  });

  // Computed property using props
  const formattedRole = computed(() => {
    const roles = {
      user: "Member",
      admin: "Administrator",
      moderator: "Community Moderator",
    };
    return roles[props.role] || "Member";
  });

  // Default avatar as fallback
  const defaultAvatarUrl =
    "https://ui-avatars.com/api/?name=" + encodeURIComponent(props.user.name);
</script>
```

**Real-World Example:**
A design system contains a Button component that accepts props for variant, size, icon, and loading state. Each prop has validation to ensure the component is used correctly, and the props are used to dynamically generate the right classes and accessibility attributes.

**Common Pitfalls:**

- Not validating props properly, leading to unexpected behavior
- Mutating props directly instead of emitting events for changes
- Using excessively complex prop validation that's hard to understand
- Not providing default values for optional props
- Using props for data that should be stored in component state

**Frequently Asked Questions:**

1. **Q: Can I use complex types like Arrays and Objects in prop validation?**

   A: Yes. Vue supports validating props as Objects, Arrays, Functions, etc. For complex objects, you can use PropType from Vue to provide more specific type information, especially useful with TypeScript:

   ```ts
   import { PropType } from "vue";

   interface User {
     id: number;
     name: string;
     email: string;
   }

   defineProps({
     user: {
       type: Object as PropType<User>,
       required: true,
     },
   });
   ```

2. **Q: How do I handle props that might be different types?**

   A: You can specify multiple possible types as an array:

   ```js
   defineProps({
     id: {
       type: [String, Number],
       required: true,
     },
   });
   ```

3. **Q: Should I modify props directly if I need to change them?**

   A: No. Props flow down from parent to child, and modifying them directly breaks the one-way data flow. Instead, either:

   - Emit an event to the parent suggesting a change (`this.$emit('update:propName', newValue)`)
   - Use the prop to initialize local state that you can modify
   - Use a computed property to transform the prop value

### 5. Dynamic and Static Props

**Concise Explanation:**
Props can be passed statically (literal values) or dynamically (bound to JavaScript expressions). Static props use normal HTML attributes, while dynamic props use the v-bind directive (:shorthand) to evaluate JavaScript expressions for their values.

**Where to Use:**

- Static props: When values don't change (strings, numbers, booleans)
- Dynamic props: When values come from component state or computed properties
- Mixed approach: For components with both fixed and variable configuration

**Code Snippet:**

```html
<template>
  <div class="p-4">
    <!-- Parent component -->
    <h1 class="text-2xl font-bold mb-4">Product Page</h1>

    <!-- Static props (literal values) -->
    <ProductCard
      title="Premium Headphones"
      price="149.99"
      currency="USD"
      category="electronics"
    />

    <!-- Dynamic props (bound to variables) -->
    <ProductCard
      :title="product.title"
      :price="product.price"
      :currency="currency"
      :category="product.category"
      :is-on-sale="product.onSale"
      :discount-percentage="calculateDiscount(product)"
      @add-to-cart="addToCart(product)"
    />

    <!-- Mixed static and dynamic props -->
    <button
      variant="primary"
      :disabled="!product.inStock"
      @click="addToCart(product)"
    >
      {{ product.inStock ? "Add to Cart" : "Out of Stock" }}
    </button>
  </div>
</template>

<script setup>
  import { ref, computed } from "vue";
  import ProductCard from "./components/ProductCard.vue";
  import Button from "./components/Button.vue";

  // Component state
  const currency = ref("USD");
  const product = ref({
    id: 1,
    title: "Wireless Earbuds",
    price: 89.99,
    category: "electronics",
    onSale: true,
    inStock: true,
  });

  // Computed values for dynamic props
  const calculateDiscount = (product) => {
    return product.onSale ? 15 : 0;
  };

  // Event handler
  const addToCart = (product) => {
    console.log(`Added ${product.title} to cart`);
  };
</script>
```

**Real-World Example:**
An e-commerce product listings page passes dynamic props to product card components, such as product details from an API, whether items are in stock (boolean), current user's preferred currency, and personalized discount rates.

**Common Pitfalls:**

- Forgetting the colon (:) for dynamic props
- Using dynamic binding for static values unnecessarily
- Not correctly parsing/formatting values when passing between components
- Trying to pass non-serializable objects like functions without proper binding
- Not accounting for prop type coercion in edge cases

**Frequently Asked Questions:**

1. **Q: What's the difference between `title="name"` and `:title="name"`?**

   A: `title="name"` passes the literal string "name" as a prop. `:title="name"` (or `v-bind:title="name"`) evaluates the JavaScript expression `name` and passes its value as a prop.

2. **Q: How do I pass boolean props correctly?**

   A: For `true`, you can use shorthand notation: `<Component prop />` which is equivalent to `:prop="true"`. For `false`, you must use the v-bind syntax: `:prop="false"`. Without the colon, all props are passed as strings.

3. **Q: Can I use dynamic prop names (computed keys)?**

   A: Yes, Vue 3 supports dynamic prop binding using square brackets:

   ```html
   <template>
     <Component :[propName]="value" />
   </template>

   <script setup>
     import { ref } from "vue";
     const propName = ref("title");
     const value = ref("Hello World");
   </script>
   ```

### 6. Component Events and Custom Events

**Concise Explanation:**
Events allow child components to communicate with their parents through the emit mechanism. Custom events use the `defineEmits()` method to declare events a component can trigger, enabling a clear contract for component communication.

**Where to Use:**

- Updating parent state from child components
- Notifying parent of user interactions (clicks, input, submissions)
- Complex form handling and validation
- Creating two-way binding patterns
- Communicating across multiple component levels

**Code Snippet:**

```html
<!-- SubmitButton.vue (Child Component) -->
<template>
  <button
    class="px-4 py-2 rounded font-medium bg-blue-600 text-white hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed"
    :disabled="loading || disabled"
    @click="handleClick"
  >
    <div v-if="loading" class="flex items-center">
      <svg
        class="animate-spin -ml-1 mr-2 h-4 w-4 text-white"
        fill="none"
        viewBox="0 0 24 24"
      >
        <circle
          class="opacity-25"
          cx="12"
          cy="12"
          r="10"
          stroke="currentColor"
          stroke-width="4"
        ></circle>
        <path
          class="opacity-75"
          fill="currentColor"
          d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
        ></path>
      </svg>
      Processing...
    </div>
    <slot v-else>Submit</slot>
  </button>
</template>

<script setup>
  // Define props
  const props = defineProps({
    loading: {
      type: Boolean,
      default: false,
    },
    disabled: {
      type: Boolean,
      default: false,
    },
  });

  // Define emitted events
  const emit = defineEmits(["click", "success", "error"]);

  // Handle click event with custom logic
  const handleClick = (event) => {
    // Emit the base click event
    emit("click", event);

    // Simulate an API call
    if (!props.loading && !props.disabled) {
      setTimeout(() => {
        const success = Math.random() > 0.3;
        if (success) {
          emit("success", { message: "Operation completed" });
        } else {
          emit("error", { message: "Operation failed" });
        }
      }, 1000);
    }
  };
</script>
```

```html
<!-- Form.vue (Parent Component) -->
<template>
  <form class="space-y-4" @submit.prevent="submitForm">
    <div>
      <label for="email" class="block text-sm font-medium text-gray-700"
        >Email</label
      >
      <input
        id="email"
        v-model="email"
        type="email"
        class="mt-1 block w-full rounded border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500"
      />
    </div>

    <div class="flex justify-end">
      <SubmitButton
        :loading="isSubmitting"
        :disabled="!isValid"
        @click="handleSubmitClick"
        @success="handleSuccess"
        @error="handleError"
      >
        Create Account
      </SubmitButton>
    </div>

    <div v-if="message" :class="['p-3 rounded', messageClass]">
      {{ message }}
    </div>
  </form>
</template>

<script setup>
  import { ref, computed } from "vue";
  import SubmitButton from "./SubmitButton.vue";

  const email = ref("");
  const isSubmitting = ref(false);
  const message = ref("");
  const messageType = ref("");

  // Computed properties
  const isValid = computed(() => {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email.value);
  });

  const messageClass = computed(() => {
    return messageType.value === "success"
      ? "bg-green-100 text-green-800"
      : "bg-red-100 text-red-800";
  });

  // Event handlers
  const handleSubmitClick = () => {
    console.log("Submit button clicked");
    isSubmitting.value = true;
  };

  const handleSuccess = (data) => {
    isSubmitting.value = false;
    messageType.value = "success";
    message.value = data.message || "Form submitted successfully";
  };

  const handleError = (error) => {
    isSubmitting.value = false;
    messageType.value = "error";
    message.value = error.message || "An error occurred";
  };

  const submitForm = () => {
    // This would be called on native form submit
    // but we're preventing default and using the button's logic instead
  };
</script>
```

**Real-World Example:**
A payment form component emits events during different stages of the payment process: `form-validate` when client-side validation completes, `payment-processing` when the payment is being processed, and `payment-complete` or `payment-error` based on the transaction result.

**Common Pitfalls:**

- Not properly declaring emitted events with `defineEmits()`
- Emitting events with inconsistent payload structures
- Forgetting to listen for events in the parent component
- Overusing events for data that should be passed as props
- Creating circular communication patterns between components

**Frequently Asked Questions:**

1. **Q: What is the difference between native DOM events and component events?**

   A: Native DOM events (like click, input, submit) are automatically passed through components with `v-on`, while custom component events must be explicitly emitted using `emit()`. Custom events also automatically use kebab-case (e.g., `myEvent` becomes `my-event`).

2. **Q: How do I implement the equivalent of v-model in my custom component?**

   A: For Vue 3, you can implement v-model in a custom component by:

   ```html
   <!-- CustomInput.vue -->
   <template>
     <input
       :value="modelValue"
       @input="$emit('update:modelValue', $event.target.value)"
     />
   </template>

   <script setup>
     defineProps(["modelValue"]);
     defineEmits(["update:modelValue"]);
   </script>

   <!-- Usage in parent -->
   <CustomInput v-model="searchText" />
   ```

   For multiple v-models, you can specify the name:

   ```html
   <MyComponent v-model:name="name" v-model:email="email" />
   ```

3. **Q: Can I validate event payloads like I validate props?**

   A: Yes, Vue 3 allows validating event payloads by using an object syntax with `defineEmits()`:

   ```js
   const emit = defineEmits({
     // No validation
     click: null,

     // Validate submit event payload
     submit: (payload) => {
       if (!payload.email) {
         console.warn("Missing email in submit event payload");
         return false;
       }
       return true;
     },
   });
   ```

### 7. Interpolation and Directives

**Concise Explanation:**
Vue uses double curly braces `{{ }}` for text interpolation (displaying data) and directives (special attributes prefixed with `v-`) to apply special reactive behavior to the DOM. These are the core mechanisms for connecting your component's data to the rendered output.

**Where to Use:**

- Text interpolation: For rendering dynamic text content
- Attribute binding: For dynamic HTML attributes
- Directives: For DOM manipulations based on component state
- Expressions: For simple operations within templates

**Code Snippet:**

```html
<template>
  <div class="p-4 bg-white shadow rounded">
    <!-- Text Interpolation -->
    <h1 class="text-2xl font-bold text-gray-900">{{ product.name }}</h1>
    <p class="mt-2 text-gray-600">{{ product.description }}</p>

    <!-- Expression in interpolation -->
    <p class="mt-2 text-lg font-semibold">
      {{ formatCurrency(product.price) }}
      <span v-if="product.discountPercentage" class="text-red-600 ml-2">
        (Save {{ product.discountPercentage }}%)
      </span>
    </p>

    <!-- Attribute binding with v-bind directive or : shorthand -->
    <img
      :src="product.imageUrl"
      :alt="product.name"
      class="w-full h-48 object-cover rounded mt-4"
    />

    <!-- Conditional rendering -->
    <div v-if="product.inStock" class="text-green-600 mt-2">In Stock</div>
    <div v-else class="text-red-600 mt-2">Out of Stock</div>

    <!-- Event handling with v-on directive or @ shorthand -->
    <button
      @click="addToCart"
      :disabled="!product.inStock"
      class="mt-4 px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 disabled:opacity-50"
    >
      Add to Cart
    </button>

    <!-- Custom directives -->
    <p v-highlight class="mt-4">{{ product.promotionMessage }}</p>
  </div>
</template>

<script setup>
  import { ref } from "vue";

  // Custom directive
  const vHighlight = {
    mounted: (el) => {
      el.style.backgroundColor = "#fef3c7";
      el.style.padding = "0.5rem";
      el.style.borderRadius = "0.25rem";
    },
  };

  // Component data
  const product = ref({
    name: "Wireless Headphones",
    description:
      "Premium noise-cancelling wireless headphones with 20-hour battery life.",
    price: 249.99,
    discountPercentage: 15,
    imageUrl: "https://example.com/headphones.jpg",
    inStock: true,
    promotionMessage: "Limited time offer: Free carrying case with purchase!",
  });

  // Methods
  const formatCurrency = (value) => {
    return new Intl.NumberFormat("en-US", {
      style: "currency",
      currency: "USD",
    }).format(value);
  };

  const addToCart = () => {
    alert(`${product.value.name} added to cart!`);
  };
</script>
```

**Real-World Example:**
A product detail page uses interpolation to display dynamic product information, directives for conditional rendering of availability and promotions, and event handlers for user interactions like adding to cart or changing product variants.

**Common Pitfalls:**

- Putting complex logic in templates instead of computed properties
- Not understanding directive shorthand syntax (`:` for `v-bind`, `@` for `v-on`)
- Using `v-if` and `v-for` on the same element (Vue 3 no longer supports this)
- Forgetting that interpolation only works with text content, not attributes
- Overusing directives for simple conditional classes instead of computed props

**Frequently Asked Questions:**

1. **Q: What expressions can I use in template interpolation?**

   A: You can use any valid JavaScript expression that evaluates to a value, including:

   - Simple variables: `{{ name }}`
   - Simple operations: `{{ price * quantity }}`
   - Ternary expressions: `{{ isActive ? 'Yes' : 'No' }}`
   - Method calls: `{{ formatDate(createdAt) }}`

   You cannot use statements (if, for, etc.) or declare variables within interpolation.

2. **Q: How do I escape HTML in interpolations to prevent XSS?**

   A: Vue automatically escapes HTML in interpolations. If you need to render HTML, use the `v-html` directive, but be careful with user-provided content:

   ```html
   <div v-html="htmlContent"></div>
   ```

   Never use `v-html` with untrusted content, as it can lead to XSS vulnerabilities.

3. **Q: Can I create my own custom directives?**

   A: Yes, Vue allows creating custom directives for specialized DOM manipulations. Register them globally in your main.js:

   ```js
   // Global registration
   app.directive("focus", {
     mounted: (el) => el.focus(),
   });
   ```

   Or locally in a component:

   ```js
   // Local registration in <script setup>
   const vFocus = {
     mounted: (el) => el.focus(),
   };
   ```

   Then use it in your template:

   ```html
   <input v-focus />
   ```

### 8. Conditional Rendering (v-if, v-show)

**Concise Explanation:**
Vue provides two main directives for conditional rendering: `v-if` (which actually adds/removes elements from the DOM) and `v-show` (which toggles the CSS `display` property). The choice between them depends on the frequency of state changes and initial rendering needs.

**Where to Use:**

- `v-if`: When condition changes infrequently or for conditional initialization
- `v-show`: When toggling visibility frequently after initial render
- `v-else`/`v-else-if`: For multi-branch conditional display
- Inside complex rendering logic for UI elements

**Code Snippet:**

```html
<template>
  <div class="p-4 bg-white rounded shadow">
    <h1 class="text-2xl font-semibold mb-4">User Dashboard</h1>

    <!-- Loading state with v-if -->
    <div v-if="loading" class="py-12 flex justify-center">
      <div
        class="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600"
      ></div>
    </div>

    <!-- Error state with v-else-if -->
    <div v-else-if="error" class="p-4 bg-red-100 text-red-800 rounded">
      <p>{{ error }}</p>
      <button
        @click="fetchData"
        class="mt-2 px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700"
      >
        Retry
      </button>
    </div>

    <!-- Empty state with v-else -->
    <div v-else-if="!data.length" class="py-8 text-center text-gray-500">
      <p>No data available</p>
      <button
        @click="openCreateModal"
        class="mt-2 px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
      >
        Create New Item
      </button>
    </div>

    <!-- Success state with v-else -->
    <div v-else>
      <div
        v-for="item in data"
        :key="item.id"
        class="p-4 border-b border-gray-200 last:border-b-0"
      >
        <h3 class="font-medium">{{ item.title }}</h3>
        <p class="text-gray-600">{{ item.description }}</p>
      </div>
    </div>

    <!-- Toggleable panel with v-show (frequent toggling) -->
    <div class="mt-6">
      <button
        @click="showFilters = !showFilters"
        class="flex items-center px-4 py-2 bg-gray-200 text-gray-800 rounded hover:bg-gray-300"
      >
        <span>{{ showFilters ? "Hide" : "Show" }} Filters</span>
        <svg
          :class="[
            'w-4 h-4 ml-2 transition-transform',
            showFilters ? 'rotate-180' : '',
          ]"
          fill="none"
          viewBox="0 0 24 24"
          stroke="currentColor"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M19 9l-7 7-7-7"
          />
        </svg>
      </button>

      <div v-show="showFilters" class="mt-2 p-4 bg-gray-100 rounded">
        <div class="grid grid-cols-2 gap-4">
          <!-- Filter controls would go here -->
          <input
            type="text"
            placeholder="Search..."
            class="px-3 py-2 border border-gray-300 rounded"
          />
          <select class="px-3 py-2 border border-gray-300 rounded">
            <option>All Categories</option>
            <option>Category 1</option>
            <option>Category 2</option>
          </select>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
  import { ref, onMounted } from "vue";

  // Component state
  const loading = ref(true);
  const error = ref(null);
  const data = ref([]);
  const showFilters = ref(false);

  // Methods
  const fetchData = async () => {
    loading.value = true;
    error.value = null;

    try {
      // Simulate API call
      await new Promise((resolve) => setTimeout(resolve, 1500));

      // Mock data
      data.value = [
        {
          id: 1,
          title: "First Item",
          description: "Description for the first item",
        },
        {
          id: 2,
          title: "Second Item",
          description: "Description for the second item",
        },
        {
          id: 3,
          title: "Third Item",
          description: "Description for the third item",
        },
      ];
    } catch (err) {
      error.value = "Failed to load data. Please try again.";
    } finally {
      loading.value = false;
    }
  };

  const openCreateModal = () => {
    alert("Open create modal");
  };

  // Lifecycle hook to fetch data on mount
  onMounted(fetchData);
</script>
```

**Real-World Example:**
A content management dashboard displays different UI states based on conditions: a loading spinner during API calls, error messages when requests fail, empty state prompts when no content exists, and the actual content when available. Filters that are frequently toggled use v-show for better performance.

**Common Pitfalls:**

- Using `v-show` for elements that have expensive initial rendering
- Using `v-if` for elements that toggle frequently, causing performance issues
- Forgetting that `v-if` affects component initialization and lifecycle hooks
- Creating overly complex conditions in templates instead of computed properties
- Using `v-if` and `v-for` on the same element (not supported in Vue 3)

**Frequently Asked Questions:**

1. **Q: When should I use v-if vs. v-show?**

   A: Use `v-if` when:

   - The condition rarely changes
   - You want to completely avoid rendering until needed
   - The initial render cost is high
   - You need to conditionally render multiple elements

   Use `v-show` when:

   - You toggle visibility frequently
   - The initial render cost is acceptable
   - The element needs to be pre-initialized (e.g., third-party plugins)

2. **Q: Can I use v-if and v-for together?**

   A: In Vue 3, you cannot use `v-if` and `v-for` on the same element. Instead, use a wrapper element or computed property:

   ```html
   <!-- Using a wrapper <template> element (preferred) -->
   <template v-for="item in items" :key="item.id">
     <div v-if="item.isVisible">{{ item.name }}</div>
   </template>

   <!-- Or using a computed property -->
   <div v-for="item in visibleItems" :key="item.id">{{ item.name }}</div>

   <script setup>
     const visibleItems = computed(() =>
       items.value.filter((item) => item.isVisible)
     );
   </script>
   ```

3. **Q: Does v-if work with multiple elements?**

   A: Yes, you can use `v-if` on a `<template>` element to conditionally render multiple elements without introducing an extra wrapper in the DOM:

   ```html
   <template v-if="isLoggedIn">
     <h1>Welcome back</h1>
     <p>Your profile is complete.</p>
   </template>
   <template v-else>
     <h1>Hello guest</h1>
     <p>Please sign in to continue.</p>
   </template>
   ```

### 9. List Rendering (v-for)

**Concise Explanation:**
The `v-for` directive renders a list of elements based on an array or object. It requires a special syntax with `in` or `of` and should always have a `:key` attribute to help Vue efficiently track and update list items.

**Where to Use:**

- Displaying dynamic lists of data (products, users, messages, etc.)
- Creating repeated UI elements with variations
- Rendering table rows and form fields from data structures
- Iterating through both arrays and objects

**Code Snippet:**

```html
<template>
  <div class="p-4 bg-white rounded shadow">
    <h1 class="text-2xl font-semibold mb-4">Task Manager</h1>

    <!-- Add new task form -->
    <form @submit.prevent="addTask" class="mb-6 flex">
      <input
        v-model="newTask"
        placeholder="Add a new task..."
        class="flex-1 px-4 py-2 border border-gray-300 rounded-l focus:outline-none focus:ring-2 focus:ring-blue-500"
      />
      <button
        type="submit"
        class="px-4 py-2 bg-blue-600 text-white rounded-r hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500"
      >
        Add
      </button>
    </form>

    <!-- Task filters -->
    <div class="mb-4 flex space-x-2">
      <button
        v-for="filter in filters"
        :key="filter.value"
        @click="currentFilter = filter.value"
        :class="[
          'px-3 py-1 rounded',
          currentFilter === filter.value
            ? 'bg-blue-600 text-white'
            : 'bg-gray-200 text-gray-700 hover:bg-gray-300',
        ]"
      >
        {{ filter.label }}
      </button>
    </div>

    <!-- Task list with v-for -->
    <div v-if="filteredTasks.length" class="space-y-2">
      <div
        v-for="(task, index) in filteredTasks"
        :key="task.id"
        class="p-4 border border-gray-200 rounded flex items-center"
      >
        <!-- Checkbox with v-model -->
        <input
          type="checkbox"
          v-model="task.completed"
          class="h-5 w-5 text-blue-600"
        />

        <!-- Task content -->
        <div
          :class="[
            'ml-3 flex-1',
            task.completed ? 'line-through text-gray-500' : 'text-gray-900',
          ]"
        >
          {{ task.text }}
          <span v-if="task.priority" class="ml-2 text-sm">
            (Priority: {{ task.priority }})
          </span>
        </div>

        <!-- Task actions -->
        <div class="flex space-x-2">
          <button
            @click="editTask(index)"
            class="p-1 text-gray-600 hover:text-blue-600"
          >
            Edit
          </button>
          <button
            @click="deleteTask(task.id)"
            class="p-1 text-gray-600 hover:text-red-600"
          >
            Delete
          </button>
        </div>
      </div>
    </div>

    <!-- Empty state -->
    <div v-else class="py-8 text-center text-gray-500">No tasks found.</div>

    <!-- Task statistics using v-for with objects -->
    <div class="mt-6 p-3 bg-gray-100 rounded">
      <h3 class="font-medium mb-2">Statistics</h3>
      <ul class="grid grid-cols-2 gap-2">
        <li
          v-for="(value, key) in statistics"
          :key="key"
          class="flex justify-between"
        >
          <span class="text-gray-600">{{ formatStatName(key) }}:</span>
          <span class="font-medium">{{ value }}</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
  import { ref, computed } from "vue";

  // Component state
  const newTask = ref("");
  const tasks = ref([
    {
      id: 1,
      text: "Complete Vue.js tutorial",
      completed: true,
      priority: "High",
    },
    {
      id: 2,
      text: "Build a task manager app",
      completed: false,
      priority: "Medium",
    },
    { id: 3, text: "Deploy to production", completed: false, priority: "Low" },
  ]);
  const currentFilter = ref("all");
  const nextId = ref(4);

  // Filter options
  const filters = [
    { label: "All", value: "all" },
    { label: "Completed", value: "completed" },
    { label: "Active", value: "active" },
  ];

  // Computed properties
  const filteredTasks = computed(() => {
    switch (currentFilter.value) {
      case "completed":
        return tasks.value.filter((task) => task.completed);
      case "active":
        return tasks.value.filter((task) => !task.completed);
      default:
        return tasks.value;
    }
  });

  const statistics = computed(() => {
    const completed = tasks.value.filter((task) => task.completed).length;
    const active = tasks.value.length - completed;

    return {
      total: tasks.value.length,
      completed,
      active,
      completionRate: tasks.value.length
        ? Math.round((completed / tasks.value.length) * 100) + "%"
        : "0%",
    };
  });

  // Methods
  const addTask = () => {
    if (newTask.value.trim()) {
      tasks.value.push({
        id: nextId.value++,
        text: newTask.value.trim(),
        completed: false,
        priority: "Medium",
      });
      newTask.value = "";
    }
  };

  const editTask = (index) => {
    const task = filteredTasks.value[index];
    const newText = prompt("Edit task:", task.text);
    if (newText !== null && newText.trim()) {
      task.text = newText.trim();
    }
  };

  const deleteTask = (id) => {
    const index = tasks.value.findIndex((task) => task.id === id);
    if (index !== -1) {
      tasks.value.splice(index, 1);
    }
  };

  const formatStatName = (name) => {
    return name
      .replace(/([A-Z])/g, " $1")
      .replace(/^./, (str) => str.toUpperCase())
      .replace(/Rate$/, " Rate");
  };
</script>
```

**Real-World Example:**
An e-commerce admin panel displays product inventory with v-for, showing product details, stock levels, and pricing. The same page includes sales statistics rendered as a list of key-value pairs and a dynamic form for product attributes based on the selected category.

**Common Pitfalls:**

- Forgetting to provide a unique `:key` attribute
- Using non-unique keys (like array indices) when items can move or be reordered
- Directly modifying the array with index assignments instead of array methods
- Inefficient filtering/sorting in the template instead of using computed properties
- Creating unnecessarily nested v-for loops that hurt performance

**Frequently Asked Questions:**

1. **Q: Why do I need to use :key with v-for?**

   A: Keys help Vue identify each node in the virtual DOM to reuse and reorder existing elements efficiently. Without keys, Vue might recreate DOM elements unnecessarily when the list changes, causing performance issues and potential state loss. Always use unique, stable identifiers (like IDs) as keys.

2. **Q: How do I access both index and value in v-for?**

   A: Vue's v-for allows you to access both the item and its index:

   ```html
   <!-- For arrays -->
   <div v-for="(item, index) in items" :key="item.id">
     {{ index }}: {{ item.name }}
   </div>

   <!-- For objects -->
   <div v-for="(value, key, index) in object" :key="key">
     {{ index }}. {{ key }}: {{ value }}
   </div>
   ```

3. **Q: How can I update arrays or objects reactively in Vue 3?**

   A: Vue 3's reactivity system based on Proxies handles most array and object mutations automatically. Use these methods for updating:

   For arrays:

   ```js
   // These methods trigger reactive updates
   tasks.value.push(newTask);
   tasks.value.pop();
   tasks.value.shift();
   tasks.value.unshift(newTask);
   tasks.value.splice(index, 1);
   tasks.value.sort((a, b) => a.name.localeCompare(b.name));
   tasks.value.reverse();

   // Replace entire array
   tasks.value = tasks.value.filter((task) => !task.completed);

   // Avoid direct index assignment for new items
   // Instead of: tasks.value[index] = newTask
   // Use: tasks.value.splice(index, 1, newTask)
   ```

   For objects:

   ```js
   // Adding or updating properties
   user.value.name = "New name"; // Reactive in Vue 3

   // For multiple properties, consider replacing the object
   user.value = { ...user.value, name: "New name", role: "admin" };
   ```

### 10. Event Handling (v-on)

**Concise Explanation:**
Vue's `v-on` directive (or `@` shorthand) attaches event listeners to elements. It can invoke methods, inline expressions, or use modifiers to enhance behavior such as preventing default actions or stopping event propagation.

**Where to Use:**

- Handling user interactions (clicks, input, hover, etc.)
- Form submissions and validation
- Custom component events
- Complex event handling with modifiers

**Code Snippet:**

```html
<template>
  <div class="p-4 bg-white rounded shadow">
    <h1 class="text-2xl font-semibold mb-4">Event Handling Examples</h1>

    <!-- Basic click event -->
    <button
      @click="counter++"
      class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 mr-2"
    >
      Count: {{ counter }}
    </button>

    <!-- Method with parameter -->
    <button
      @click="greet('World')"
      class="px-4 py-2 bg-green-600 text-white rounded hover:bg-green-700 mr-2"
    >
      Greet
    </button>

    <!-- Event modifiers: stop, prevent -->
    <div @click="alertOuter" class="mt-4 p-4 bg-gray-100 rounded">
      Outer div (click me)

      <div @click.stop="alertInner" class="mt-2 p-4 bg-blue-100 rounded">
        Inner div (click stops propagation)
      </div>

      <!-- Using multiple modifiers -->
      <a
        href="https://vuejs.org"
        @click.prevent.stop="handleLinkClick"
        class="mt-2 block text-blue-600 hover:underline"
      >
        Vue.js (click prevented & stopped)
      </a>
    </div>

    <!-- Key modifiers -->
    <div class="mt-4">
      <label class="block text-gray-700 mb-2">Press Enter to submit:</label>
      <input
        type="text"
        v-model="message"
        @keyup.enter="submitMessage"
        class="w-full px-4 py-2 border border-gray-300 rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
      />

      <!-- Multiple key modifiers -->
      <p class="mt-2 text-gray-600">
        Keyboard shortcut: Press <kbd>Ctrl</kbd>+<kbd>S</kbd> to save
      </p>
      <textarea
        v-model="notes"
        @keydown.ctrl.s.prevent="saveNotes"
        class="w-full mt-1 px-4 py-2 border border-gray-300 rounded h-24 focus:outline-none focus:ring-2 focus:ring-blue-500"
        placeholder="Type something and press Ctrl+S to save..."
      ></textarea>
    </div>

    <!-- Mouse modifiers -->
    <div class="mt-4 grid grid-cols-2 gap-4">
      <button
        @click.right.prevent="showContextMenu"
        class="px-4 py-2 bg-purple-600 text-white rounded hover:bg-purple-700"
      >
        Right-click me
      </button>

      <button
        @click.middle.prevent="handleMiddleClick"
        class="px-4 py-2 bg-yellow-600 text-white rounded hover:bg-yellow-700"
      >
        Middle-click me
      </button>
    </div>

    <!-- Event object access -->
    <div class="mt-4">
      <label class="block text-gray-700 mb-2">Mouse position:</label>
      <div
        @mousemove="updateCoordinates"
        class="w-full h-32 bg-gray-200 rounded flex items-center justify-center"
      >
        Coordinates: {{ coordinates.x }}, {{ coordinates.y }}
      </div>
    </div>

    <!-- Custom component event -->
    <CustomButton class="mt-4" @custom-click="handleCustomClick">
      Trigger custom event
    </CustomButton>
  </div>
</template>

<script setup>
  import { ref } from "vue";
  import CustomButton from "./CustomButton.vue";

  // Component state
  const counter = ref(0);
  const message = ref("");
  const notes = ref("");
  const coordinates = ref({ x: 0, y: 0 });

  // Methods for event handlers
  const greet = (name) => {
    alert(`Hello, ${name}!`);
  };

  const alertOuter = () => {
    alert("Outer div clicked (event bubbles up)");
  };

  const alertInner = () => {
    alert("Inner div clicked (propagation stopped)");
  };

  const handleLinkClick = () => {
    alert("Link click prevented, navigation canceled");
  };

  const submitMessage = () => {
    alert(`Submitted: ${message.value}`);
    message.value = "";
  };

  const saveNotes = () => {
    alert(`Notes saved: ${notes.value}`);
  };

  const showContextMenu = () => {
    alert("Custom context menu would show here");
  };

  const handleMiddleClick = () => {
    alert("Middle mouse button clicked");
  };

  const updateCoordinates = (event) => {
    // Get coordinates relative to the element
    const rect = event.currentTarget.getBoundingClientRect();
    coordinates.value = {
      x: Math.round(event.clientX - rect.left),
      y: Math.round(event.clientY - rect.top),
    };
  };

  const handleCustomClick = (payload) => {
    alert(`Custom event received with payload: ${JSON.stringify(payload)}`);
  };
</script>
```

```html
<!-- CustomButton.vue -->
<template>
  <button
    @click="emitCustomClick"
    class="px-4 py-2 bg-indigo-600 text-white rounded hover:bg-indigo-700"
  >
    <slot></slot>
  </button>
</template>

<script setup>
  const emit = defineEmits(["custom-click"]);

  const emitCustomClick = () => {
    emit("custom-click", {
      time: new Date().toISOString(),
      source: "CustomButton",
    });
  };
</script>
```

**Real-World Example:**
A drag-and-drop file uploader uses event handling to manage the entire interaction: handling dragenter/dragleave events to show visual feedback, preventing default browser behavior with prevent modifier, and processing the files on drop. The component emits custom events like 'file-added' and 'upload-complete'.

**Common Pitfalls:**

- Not using event modifiers when needed, causing unexpected behavior
- Overusing inline expressions for complex logic instead of methods
- Forgetting to prevent default behavior for form submissions
- Not cleaning up event listeners for window/document events
- Passing incorrect arguments to event handlers

**Frequently Asked Questions:**

1. **Q: What are the most useful event modifiers in Vue?**

   A: The most commonly used event modifiers are:

   - `.stop`: Stops event propagation (equivalent to `event.stopPropagation()`)
   - `.prevent`: Prevents default behavior (equivalent to `event.preventDefault()`)
   - `.capture`: Uses capture mode for the event listener
   - `.once`: Trigger handler only once, then remove listener
   - `.self`: Only trigger if event.target is the element itself

   Key modifiers:

   - `.enter`, `.tab`, `.delete`, `.esc`, `.space`, `.up`, `.down`, `.left`, `.right`
   - System modifiers: `.ctrl`, `.alt`, `.shift`, `.meta`

   Mouse modifiers:

   - `.left`, `.right`, `.middle`

2. **Q: How do I access the original DOM event in a method?**

   A: Vue automatically passes the native DOM event to the method when no arguments are specified:

   ```html
   <button @click="handleClick">Click me</button>

   <script setup>
     const handleClick = (event) => {
       console.log(event); // Native DOM event
     };
   </script>
   ```

   If you need to pass custom arguments and still access the event:

   ```html
   <button @click="handleClick('hello', $event)">Click me</button>

   <script setup>
     const handleClick = (message, event) => {
       console.log(message); // 'hello'
       console.log(event); // Native DOM event
     };
   </script>
   ```

3. **Q: How do I listen for events on the window or document?**

   A: You can use lifecycle hooks to add and remove global event listeners:

   ```js
   import { onMounted, onUnmounted } from "vue";

   // Setup event listeners
   onMounted(() => {
     window.addEventListener("resize", handleResize);
     document.addEventListener("keydown", handleKeyDown);
   });

   // Clean up event listeners
   onUnmounted(() => {
     window.removeEventListener("resize", handleResize);
     document.removeEventListener("keydown", handleKeyDown);
   });

   const handleResize = () => {
     // Handle window resize
   };

   const handleKeyDown = (event) => {
     // Handle key presses
   };
   ```

### 11. Form Input Bindings (v-model)

**Concise Explanation:**
`v-model` creates two-way data binding between form inputs and component state. It automatically handles the appropriate event for each input type and updates the component state when the input changes, simplifying form handling in Vue applications.

**Where to Use:**

- Form inputs (text fields, checkboxes, radio buttons, etc.)
- Custom form components
- Interactive elements requiring two-way binding
- Multi-step forms and wizards

**Code Snippet:**

```html
<template>
  <div class="p-4 bg-white rounded shadow">
    <h1 class="text-2xl font-semibold mb-6">User Registration Form</h1>

    <form @submit.prevent="submitForm" class="space-y-6">
      <!-- Text input with v-model -->
      <div>
        <label for="name" class="block text-sm font-medium text-gray-700 mb-1">
          Full Name
        </label>
        <input
          id="name"
          v-model.trim="form.name"
          type="text"
          class="w-full px-4 py-2 border border-gray-300 rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
          :class="{ 'border-red-500': errors.name }"
        />
        <p v-if="errors.name" class="mt-1 text-sm text-red-600">
          {{ errors.name }}
        </p>
      </div>

      <!-- Email input with modifiers -->
      <div>
        <label for="email" class="block text-sm font-medium text-gray-700 mb-1">
          Email Address
        </label>
        <input
          id="email"
          v-model.trim.lazy="form.email"
          type="email"
          class="w-full px-4 py-2 border border-gray-300 rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
          :class="{ 'border-red-500': errors.email }"
        />
        <p class="mt-1 text-sm text-gray-500">
          Using .lazy modifier: updates on change event instead of input
        </p>
        <p v-if="errors.email" class="mt-1 text-sm text-red-600">
          {{ errors.email }}
        </p>
      </div>

      <!-- Number input with modifiers -->
      <div>
        <label for="age" class="block text-sm font-medium text-gray-700 mb-1">
          Age
        </label>
        <input
          id="age"
          v-model.number="form.age"
          type="number"
          class="w-full px-4 py-2 border border-gray-300 rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
          :class="{ 'border-red-500': errors.age }"
        />
        <p class="mt-1 text-sm text-gray-500">
          Using .number modifier: automatically converts to number type
        </p>
        <p v-if="errors.age" class="mt-1 text-sm text-red-600">
          {{ errors.age }}
        </p>
      </div>

      <!-- Textarea -->
      <div>
        <label for="bio" class="block text-sm font-medium text-gray-700 mb-1">
          Short Bio
        </label>
        <textarea
          id="bio"
          v-model.trim="form.bio"
          rows="3"
          class="w-full px-4 py-2 border border-gray-300 rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
        ></textarea>
        <p class="mt-1 text-sm text-gray-500">
          {{ 200 - form.bio.length }} characters remaining
        </p>
      </div>

      <!-- Checkbox -->
      <div>
        <div class="flex items-center">
          <input
            id="newsletter"
            v-model="form.newsletter"
            type="checkbox"
            class="h-4 w-4 text-blue-600 focus:ring-blue-500 border-gray-300 rounded"
          />
          <label for="newsletter" class="ml-2 block text-sm text-gray-700">
            Subscribe to newsletter
          </label>
        </div>
      </div>

      <!-- Checkbox group -->
      <div>
        <span class="block text-sm font-medium text-gray-700 mb-2">
          Interests (choose at least one)
        </span>
        <div class="space-y-2">
          <div
            v-for="interest in interests"
            :key="interest.value"
            class="flex items-center"
          >
            <input
              :id="interest.value"
              v-model="form.interests"
              :value="interest.value"
              type="checkbox"
              class="h-4 w-4 text-blue-600 focus:ring-blue-500 border-gray-300 rounded"
            />
            <label
              :for="interest.value"
              class="ml-2 block text-sm text-gray-700"
            >
              {{ interest.label }}
            </label>
          </div>
        </div>
        <p v-if="errors.interests" class="mt-1 text-sm text-red-600">
          {{ errors.interests }}
        </p>
      </div>

      <!-- Radio buttons -->
      <div>
        <span class="block text-sm font-medium text-gray-700 mb-2">
          Account Type
        </span>
        <div class="space-y-2">
          <div
            v-for="type in accountTypes"
            :key="type.value"
            class="flex items-center"
          >
            <input
              :id="type.value"
              v-model="form.accountType"
              :value="type.value"
              type="radio"
              class="h-4 w-4 text-blue-600 focus:ring-blue-500 border-gray-300"
            />
            <label :for="type.value" class="ml-2 block text-sm text-gray-700">
              {{ type.label }}
            </label>
          </div>
        </div>
        <p v-if="errors.accountType" class="mt-1 text-sm text-red-600">
          {{ errors.accountType }}
        </p>
      </div>

      <!-- Select dropdown -->
      <div>
        <label
          for="country"
          class="block text-sm font-medium text-gray-700 mb-1"
        >
          Country
        </label>
        <select
          id="country"
          v-model="form.country"
          class="w-full px-4 py-2 border border-gray-300 rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
        >
          <option value="" disabled>Select a country</option>
          <option
            v-for="country in countries"
            :key="country.code"
            :value="country.code"
          >
            {{ country.name }}
          </option>
        </select>
        <p v-if="errors.country" class="mt-1 text-sm text-red-600">
          {{ errors.country }}
        </p>
      </div>

      <!-- Multiple select -->
      <div>
        <label
          for="languages"
          class="block text-sm font-medium text-gray-700 mb-1"
        >
          Programming Languages
        </label>
        <select
          id="languages"
          v-model="form.languages"
          multiple
          class="w-full px-4 py-2 border border-gray-300 rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
        >
          <option
            v-for="lang in programmingLanguages"
            :key="lang"
            :value="lang"
          >
            {{ lang }}
          </option>
        </select>
        <p class="mt-1 text-sm text-gray-500">
          Hold Ctrl/Cmd key to select multiple options
        </p>
      </div>

      <!-- Custom component with v-model -->
      <div>
        <label class="block text-sm font-medium text-gray-700 mb-1">
          Experience Level
        </label>
        <RangeSlider
          v-model="form.experienceLevel"
          :min="0"
          :max="10"
          :step="1"
        />
        <p class="mt-1 text-sm text-gray-500">
          Experience level: {{ form.experienceLevel }}
        </p>
      </div>

      <!-- Toggle switch custom component -->
      <div>
        <ToggleSwitch v-model="form.darkMode" label="Enable Dark Mode" />
      </div>

      <!-- Form submission -->
      <div class="pt-4">
        <button
          type="submit"
          :disabled="isSubmitting"
          class="w-full px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed"
        >
          <span v-if="isSubmitting">
            <svg
              class="animate-spin -ml-1 mr-2 h-4 w-4 text-white inline-block"
              fill="none"
              viewBox="0 0 24 24"
            >
              <circle
                class="opacity-25"
                cx="12"
                cy="12"
                r="10"
                stroke="currentColor"
                stroke-width="4"
              ></circle>
              <path
                class="opacity-75"
                fill="currentColor"
                d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
              ></path>
            </svg>
            Submitting...
          </span>
          <span v-else>Register</span>
        </button>
      </div>
    </form>

    <!-- Preview of form data -->
    <div v-if="showPreview" class="mt-8 p-4 bg-gray-100 rounded">
      <h2 class="text-lg font-semibold mb-2">Form Data Preview</h2>
      <pre class="text-sm">{{ JSON.stringify(form, null, 2) }}</pre>
    </div>
  </div>
</template>

<script setup>
  import { ref, reactive, computed } from "vue";
  import RangeSlider from "./RangeSlider.vue";
  import ToggleSwitch from "./ToggleSwitch.vue";

  // Form data with reactive object
  const form = reactive({
    name: "",
    email: "",
    age: null,
    bio: "",
    newsletter: false,
    interests: [],
    accountType: "",
    country: "",
    languages: [],
    experienceLevel: 5,
    darkMode: false,
  });

  // Form submission state
  const isSubmitting = ref(false);
  const showPreview = ref(false);
  const errors = reactive({});

  // Options for form inputs
  const interests = [
    { value: "technology", label: "Technology" },
    { value: "design", label: "Design" },
    { value: "business", label: "Business" },
    { value: "marketing", label: "Marketing" },
  ];

  const accountTypes = [
    { value: "personal", label: "Personal" },
    { value: "business", label: "Business" },
    { value: "education", label: "Education" },
  ];

  const countries = [
    { code: "us", name: "United States" },
    { code: "ca", name: "Canada" },
    { code: "uk", name: "United Kingdom" },
    { code: "au", name: "Australia" },
    { code: "jp", name: "Japan" },
  ];

  const programmingLanguages = [
    "JavaScript",
    "TypeScript",
    "Python",
    "Java",
    "C#",
    "Go",
    "Rust",
  ];

  // Form validation
  const validateForm = () => {
    errors.name = !form.name ? "Name is required" : "";
    errors.email = !form.email
      ? "Email is required"
      : !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(form.email)
      ? "Email format is invalid"
      : "";
    errors.age =
      form.age === null
        ? "Age is required"
        : form.age < 18
        ? "You must be at least 18 years old"
        : "";
    errors.interests =
      form.interests.length === 0 ? "Select at least one interest" : "";
    errors.accountType = !form.accountType ? "Select an account type" : "";
    errors.country = !form.country ? "Select a country" : "";

    // Check if any errors exist
    return !Object.values(errors).some((error) => error);
  };

  // Form submission
  const submitForm = async () => {
    if (!validateForm()) {
      // Scroll to first error
      const firstError = document.querySelector(".text-red-600");
      if (firstError) {
        firstError.scrollIntoView({ behavior: "smooth", block: "center" });
      }
      return;
    }

    isSubmitting.value = true;

    try {
      // Simulate API call
      await new Promise((resolve) => setTimeout(resolve, 1500));

      // Success handling
      showPreview.value = true;
      alert("Form submitted successfully!");

      // Reset form in real application
      // Object.keys(form).forEach(key => form[key] = '')
    } catch (error) {
      alert("An error occurred. Please try again.");
    } finally {
      isSubmitting.value = false;
    }
  };
</script>
```

```html
<!-- RangeSlider.vue -->
<template>
  <div>
    <input
      type="range"
      :min="min"
      :max="max"
      :step="step"
      :value="modelValue"
      @input="updateValue"
      class="w-full h-2 bg-gray-200 rounded appearance-none cursor-pointer"
    />
    <div class="flex justify-between text-xs text-gray-600 mt-1">
      <span>{{ min }}</span>
      <span>{{ max }}</span>
    </div>
  </div>
</template>

<script setup>
  const props = defineProps({
    modelValue: {
      type: Number,
      required: true,
    },
    min: {
      type: Number,
      default: 0,
    },
    max: {
      type: Number,
      default: 100,
    },
    step: {
      type: Number,
      default: 1,
    },
  });

  const emit = defineEmits(["update:modelValue"]);

  const updateValue = (event) => {
    emit("update:modelValue", Number(event.target.value));
  };
</script>
```

```html
<!-- ToggleSwitch.vue -->
<template>
  <div class="flex items-center">
    <button
      type="button"
      :class="[
        'relative inline-flex h-6 w-11 flex-shrink-0 cursor-pointer rounded-full border-2 border-transparent transition-colors duration-200 ease-in-out focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2',
        modelValue ? 'bg-blue-600' : 'bg-gray-200',
      ]"
      @click="updateValue(!modelValue)"
      role="switch"
      :aria-checked="modelValue"
    >
      <span
        :class="[
          'pointer-events-none relative inline-block h-5 w-5 transform rounded-full bg-white shadow ring-0 transition duration-200 ease-in-out',
          modelValue ? 'translate-x-5' : 'translate-x-0',
        ]"
      >
        <span
          :class="[
            'absolute inset-0 flex h-full w-full items-center justify-center transition-opacity',
            modelValue
              ? 'opacity-0 duration-100 ease-out'
              : 'opacity-100 duration-200 ease-in',
          ]"
          aria-hidden="true"
        >
          <svg class="h-3 w-3 text-gray-400" fill="none" viewBox="0 0 12 12">
            <path
              d="M4 8l2-2m0 0l2-2M6 6L4 4m2 2l2 2"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
            />
          </svg>
        </span>
        <span
          :class="[
            'absolute inset-0 flex h-full w-full items-center justify-center transition-opacity',
            modelValue
              ? 'opacity-100 duration-200 ease-in'
              : 'opacity-0 duration-100 ease-out',
          ]"
          aria-hidden="true"
        >
          <svg
            class="h-3 w-3 text-blue-600"
            fill="currentColor"
            viewBox="0 0 12 12"
          >
            <path
              d="M3.707 5.293a1 1 0 00-1.414 1.414l1.414-1.414zM5 8l-.707.707a1 1 0 001.414 0L5 8zm4.707-3.293a1 1 0 00-1.414-1.414l1.414 1.414zm-7.414 2l2 2 1.414-1.414-2-2-1.414 1.414zm3.414 2l4-4-1.414-1.414-4 4 1.414 1.414z"
            />
          </svg>
        </span>
      </span>
    </button>
    <span class="ml-3 text-sm" @click="updateValue(!modelValue)">
      {{ label }}
    </span>
  </div>
</template>

<script setup>
  const props = defineProps({
    modelValue: {
      type: Boolean,
      required: true,
    },
    label: {
      type: String,
      default: "",
    },
  });

  const emit = defineEmits(["update:modelValue"]);

  const updateValue = (value) => {
    emit("update:modelValue", value);
  };
</script>
```

**Real-World Example:**
A customer onboarding flow uses v-model across multiple form components to collect user information, preferences, and settings. The form includes validation, conditional fields that appear based on previous inputs, and custom form components like date pickers and autocomplete fields that implement v-model internally.

**Common Pitfalls:**

- Not using the correct modifiers (`.number`, `.trim`, `.lazy`) for specific use cases
- Mutating props in custom components instead of using emit for v-model
- Forgetting to validate form input and provide user feedback
- Not handling different input types appropriately (checkboxes vs text fields)
- Over-complicating custom components that implement v-model

**Frequently Asked Questions:**

1. **Q: How do I implement v-model on a custom component?**

   A: For Vue 3, implement v-model by accepting a `modelValue` prop and emitting an `update:modelValue` event:

   ```html
   <!-- Custom component -->
   <template>
     <input
       :value="modelValue"
       @input="$emit('update:modelValue', $event.target.value)"
     />
   </template>

   <script setup>
     defineProps(["modelValue"]);
     defineEmits(["update:modelValue"]);
   </script>

   <!-- Usage -->
   <CustomInput v-model="searchText" />
   ```

   For multiple v-model bindings on a single component:

   ```html
   <!-- Component -->
   <template>
     <input
       :value="firstName"
       @input="$emit('update:firstName', $event.target.value)"
     />
     <input
       :value="lastName"
       @input="$emit('update:lastName', $event.target.value)"
     />
   </template>

   <script setup>
     defineProps(["firstName", "lastName"]);
     defineEmits(["update:firstName", "update:lastName"]);
   </script>

   <!-- Usage -->
   <UserNameInput v-model:firstName="firstName" v-model:lastName="lastName" />
   ```

2. **Q: What are the modifiers available for v-model and when should I use them?**

   A: Vue provides three modifiers for v-model:

   - `.trim`: Automatically removes whitespace from the beginning and end of input
   - `.number`: Converts the input to a number (useful for number inputs)
   - `.lazy`: Only updates the value when the input loses focus or Enter is pressed, not on every keypress

   Use `.trim` for text inputs where whitespace is irrelevant, `.number` for numeric inputs, and `.lazy` for performance optimization or when you only want updates after the user completes typing.

3. **Q: How can I validate form inputs with v-model?**

   A: Validation can be implemented in several ways:

   - Real-time validation with watchers:

     ```js
     watch(
       () => form.email,
       (newValue) => {
         errors.email = !newValue
           ? "Email is required"
           : !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(newValue)
           ? "Invalid email format"
           : "";
       }
     );
     ```

   - Computed properties for validation status:

     ```js
     const isEmailValid = computed(() => {
       return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(form.email);
     });
     ```

   - Form-level validation on submit:
     ```js
     const validateForm = () => {
       // Check all fields and set error messages
       return !Object.values(errors).some((error) => error);
     };
     ```

### 12. Slots and Named Slots

**Concise Explanation:**
Slots are Vue's content distribution mechanism, allowing you to pass template content from a parent component to a child component. Default slots handle simple content insertion, while named slots enable more complex layouts with multiple insertion points.

**Where to Use:**

- Creating layout components (Card, Modal, Page)
- Building reusable UI components with flexible content
- Implementing compound components (Tabs, Accordions)
- Creating components with consistent styling but variable content

**Code Snippet:**

```html
<!-- Card.vue - Component with default and named slots -->
<template>
  <div class="bg-white rounded shadow overflow-hidden">
    <!-- Header with named slot -->
    <div v-if="$slots.header" class="p-4 bg-gray-50 border-b border-gray-200">
      <slot name="header">
        <!-- Fallback content if no header is provided -->
        <h3 class="text-lg font-medium text-gray-900">Card Title</h3>
      </slot>
    </div>

    <!-- Default slot for main content -->
    <div class="p-4">
      <slot>
        <!-- Fallback content -->
        <p class="text-gray-500">No content provided</p>
      </slot>
    </div>

    <!-- Footer with named slot -->
    <div v-if="$slots.footer" class="p-4 bg-gray-50 border-t border-gray-200">
      <slot name="footer"></slot>
    </div>

    <!-- Scoped slot example -->
    <div v-if="$slots.actions" class="px-4 py-3 bg-gray-100 flex justify-end">
      <slot
        name="actions"
        :save="save"
        :cancel="cancel"
        :isPending="isPending"
      ></slot>
    </div>
  </div>
</template>

<script setup>
  import { ref } from "vue";

  // State for scoped slot
  const isPending = ref(false);

  // Methods to be exposed via scoped slots
  const save = async () => {
    isPending.value = true;
    try {
      await new Promise((resolve) => setTimeout(resolve, 1000));
      alert("Saved successfully!");
    } finally {
      isPending.value = false;
    }
  };

  const cancel = () => {
    alert("Operation cancelled");
  };
</script>
```

```html
<!-- Usage of the Card component with slots -->
<template>
  <div class="p-4">
    <h1 class="text-2xl font-bold mb-4">Slot Examples</h1>

    <!-- Basic usage with default slot -->
    <Card class="mb-4">
      <p>This is the card content using the default slot.</p>
    </Card>

    <!-- Using named slots -->
    <Card class="mb-4">
      <!-- Header slot -->
      <template #header>
        <div class="flex items-center justify-between">
          <h3 class="text-lg font-semibold">User Profile</h3>
          <span class="text-sm bg-green-100 text-green-800 px-2 py-1 rounded"
            >Active</span
          >
        </div>
      </template>

      <!-- Default slot -->
      <div class="space-y-3">
        <div class="flex items-center space-x-3">
          <img
            src="https://ui-avatars.com/api/?name=John+Doe"
            alt="User avatar"
            class="w-12 h-12 rounded-full"
          />
          <div>
            <h4 class="font-medium">John Doe</h4>
            <p class="text-gray-600">john.doe@example.com</p>
          </div>
        </div>
        <p>
          Frontend Developer with 5 years of experience in Vue.js and React.
        </p>
      </div>

      <!-- Footer slot -->
      <template #footer>
        <p class="text-sm text-gray-500">Member since January 2022</p>
      </template>

      <!-- Scoped slot example -->
      <template #actions="{ save, cancel, isPending }">
        <div class="space-x-2">
          <button
            @click="cancel"
            class="px-4 py-2 border border-gray-300 rounded text-gray-700 hover:bg-gray-50"
          >
            Cancel
          </button>
          <button
            @click="save"
            :disabled="isPending"
            class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 disabled:opacity-50"
          >
            <span v-if="isPending">Saving...</span>
            <span v-else>Save</span>
          </button>
        </div>
      </template>
    </Card>

    <!-- Dynamic slot names -->
    <Card class="mb-4">
      <template #[dynamicSlotName]>
        <p>
          This content is rendered in a dynamically named slot:
          <strong>{{ dynamicSlotName }}</strong>
        </p>
      </template>

      <p>Default content is still here.</p>
    </Card>

    <!-- Tabs example using slots -->
    <Tabs>
      <Tab title="Profile">
        <div class="p-4">
          <h3 class="text-lg font-medium mb-2">User Profile</h3>
          <p>This is the profile tab content.</p>
        </div>
      </Tab>
      <Tab title="Settings">
        <div class="p-4">
          <h3 class="text-lg font-medium mb-2">User Settings</h3>
          <p>This is the settings tab content.</p>
        </div>
      </Tab>
      <Tab title="Notifications">
        <div class="p-4">
          <h3 class="text-lg font-medium mb-2">Notifications</h3>
          <p>This is the notifications tab content.</p>
        </div>
      </Tab>
    </Tabs>
  </div>
</template>

<script setup>
  import { ref } from "vue";
  import Card from "./Card.vue";
  import Tabs from "./Tabs.vue";
  import Tab from "./Tab.vue";

  const dynamicSlotName = ref("header");

  // Change the dynamic slot after 3 seconds
  setTimeout(() => {
    dynamicSlotName.value = "footer";
  }, 3000);
</script>
```

```html
<!-- Tabs.vue - Compound component using slots -->
<template>
  <div>
    <!-- Tab navigation -->
    <div class="border-b border-gray-200">
      <nav class="flex -mb-px">
        <button
          v-for="(tab, index) in tabs"
          :key="index"
          @click="activeTabIndex = index"
          :class="[
            'px-4 py-2 font-medium text-sm',
            activeTabIndex === index
              ? 'border-b-2 border-blue-500 text-blue-600'
              : 'text-gray-500 hover:text-gray-700 hover:border-gray-300',
          ]"
        >
          {{ tab.title }}
        </button>
      </nav>
    </div>

    <!-- Active tab content -->
    <div class="py-4">
      <component
        :is="tabs[activeTabIndex]?.component"
        v-if="tabs.length && activeTabIndex >= 0"
      />
    </div>
  </div>
</template>

<script setup>
  import { ref, provide, onMounted } from "vue";

  const tabs = ref([]);
  const activeTabIndex = ref(0);

  // Register a tab
  const registerTab = (tab) => {
    tabs.value.push(tab);
  };

  // Provide the registration function to child components
  provide("registerTab", registerTab);

  // Make the active tab index available to child components
  provide("activeTabIndex", activeTabIndex);
</script>
```

```html
<!-- Tab.vue - Child component using provide/inject with slots -->
<template>
  <div v-if="isActive">
    <slot></slot>
  </div>
</template>

<script setup>
  import { inject, computed, onMounted, h } from "vue";

  const props = defineProps({
    title: {
      type: String,
      required: true,
    },
  });

  // Inject functions from parent Tabs component
  const registerTab = inject("registerTab");
  const activeTabIndex = inject("activeTabIndex");

  // Generate a unique ID for this tab
  const id = Date.now().toString(36) + Math.random().toString(36).substring(2);

  // Determine if this tab is active
  const isActive = computed(() => {
    const index = tabs.value.findIndex((tab) => tab.id === id);
    return index === activeTabIndex.value;
  });

  // Register this tab with the parent component
  onMounted(() => {
    registerTab({
      id,
      title: props.title,
      component: h({
        setup() {
          return () => h("div", {}, slots.default?.());
        },
      }),
    });
  });
</script>
```

**Real-World Example:**
A reusable Modal component uses named slots for the header, body, and footer sections, allowing different parts of the application to customize the modal content while maintaining consistent styling and behavior. A scoped slot is used to provide modal actions (close, confirm) to the content.

**Common Pitfalls:**

- Not checking if a slot exists before rendering its container
- Overusing slots when props would be simpler
- Using slots for dynamic data that should be passed as props
- Not providing fallback content in slots
- Creating overly complex slot APIs that are hard to understand

**Frequently Asked Questions:**

1. **Q: What's the difference between named slots and scoped slots?**

   A: Named slots are for organizing multiple content distribution points in a component. Scoped slots allow the child component to pass data back to the parent template, enabling more customization based on the child's internal state. A slot can be both named and scoped.

2. **Q: How do I check if a slot has content before rendering it?**

   A: Use the `$slots` object to conditionally render slot containers:

   ```html
   <div v-if="$slots.header" class="card-header">
     <slot name="header"></slot>
   </div>
   ```

   In Vue 3, you can also use the `v-slot` API to check if a slot is provided:

   ```html
   <template v-if="$slots.header">
     <slot name="header"></slot>
   </template>
   ```

3. **Q: What are the different ways to define slot content in the parent component?**

   A: Vue 3 provides several syntaxes for defining slot content:

   Default slot:

   ```html
   <Card>Default slot content</Card>
   ```

   Named slots with shorthand:

   ```html
   <Card>
     <template #header>Header content</template>
     <template #footer>Footer content</template>
     Default slot content
   </Card>
   ```

   Named slots with full syntax:

   ```html
   <Card>
     <template v-slot:header>Header content</template>
     Default slot content
   </Card>
   ```

   Scoped slots:

   ```html
   <Card>
     <template #actions="{ save, cancel }">
       <button @click="save">Save</button>
       <button @click="cancel">Cancel</button>
     </template>
   </Card>
   ```

## Next Actions: Exercises and Micro-Projects

### 1. Component Architecture Exercise

Build a reusable Button component with props for variant, size, and loading state. Implement proper validation and support for both icon and text content. Use Tailwind v4 for styling.

### 2. Registration Form Project

Create a multi-step registration form using Vue components. Each step should be a separate component, with form state managed in the parent. Implement validation and smooth transitions between steps.

### 3. Data Table Component

Build a reusable data table component that accepts column definitions and row data as props. Implement features like sorting, filtering, and pagination. Use slots for custom cell rendering.

### 4. Modal System

Create a modal system with a base Modal component and specialized variants (AlertModal, ConfirmModal, FormModal). Use provide/inject for managing global modal state and slots for content customization.

### 5. Tab Interface

Build a tab interface using slots and dynamic components. The parent TabGroup should manage active tab state while child Tab components provide content via slots and titles via props.

### 6. Event Bus Alternative

Implement a component communication pattern using props, events, and provide/inject. Compare this approach to using global state management for different scenarios.

## Success Criteria

You've mastered Module 2 when you can:

1. **Create Well-Structured Components** - Design and implement reusable Vue components with clear APIs, proper validation, and appropriate defaults.

2. **Implement Component Communication** - Use props and events correctly to establish clear parent-child communication patterns.

3. **Use Lifecycle Hooks Effectively** - Apply appropriate lifecycle hooks to handle initialization, updates, and cleanup in components.

4. **Create Flexible Templates** - Build templates with dynamic content using directives, conditional rendering, and list processing.

5. **Master Form Handling** - Implement reactive forms with validation, different input types, and efficient state management.

6. **Design Component Composition** - Compose complex UIs from smaller, specialized components using slots and provide/inject.

## Troubleshooting Guide

### Component Registration Issues

- **Problem**: Component is not recognized in the template.
  **Solution**: Check that the component is properly imported and registered. If using local registration, ensure the component is registered in the current component. If using global registration, verify it's registered before the app is mounted.

- **Problem**: Component works in development but not in production.
  **Solution**: Check for case sensitivity issues in component names and file paths, as these can be masked in development but cause problems in production.

### Prop Validation Errors

- **Problem**: Getting warnings about prop types.
  **Solution**: Ensure you're passing the correct data types to props and that you're using dynamic binding (`:prop`) for non-string values.

- **Problem**: Component not receiving updated prop values.
  **Solution**: Check if you're mutating props directly (forbidden in Vue). Instead, emit events to request changes or make a local copy of the prop.

### Lifecycle Hook Issues

- **Problem**: DOM interaction in onMounted not working.
  **Solution**: Ensure you're using template refs correctly and accessing DOM elements after the component is mounted. Check for timing issues with async operations.

- **Problem**: Memory leaks from event listeners.
  **Solution**: Always remove event listeners, timers, and subscriptions in the onUnmounted hook.

### Template Syntax Problems

- **Problem**: v-for items not updating when array changes.
  **Solution**: Ensure you're using unique and stable keys for v-for elements. Use Vue's reactive array methods or replace the entire array reference.

- **Problem**: Conditional rendering not working as expected.
  **Solution**: Check the evaluation of your conditions. Use computed properties for complex conditions instead of inline expressions.

### Form Binding Issues

- **Problem**: Form inputs not updating component state.
  **Solution**: Verify v-model usage. For custom components, ensure you're implementing v-model correctly with the appropriate prop and event.

- **Problem**: Form validation timing issues.
  **Solution**: Decide between immediate (with watchers) or on-submit validation. For immediate validation, consider debouncing to avoid overwhelming the user.

### Slot Content Problems

- **Problem**: Slot content not rendering.
  **Solution**: Check if you're using the correct slot syntax and that slot names match between parent and child. Verify the component hierarchy and make sure slot content is wrapped in a template tag when needed.

- **Problem**: Scoped slot data not available in parent.
  **Solution**: Ensure you're properly binding scoped slot props in the parent template and that the child component is correctly passing the data to the slot.
