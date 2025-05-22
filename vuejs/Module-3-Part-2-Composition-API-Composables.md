# Module 3: Composition API - Part 2: Composables

## Core Problem

As applications grow in complexity, managing duplicated logic across components becomes challenging. Composables solve this by enabling extraction of stateful logic into reusable functions, promoting code reuse while maintaining reactivity.

## Key Assumptions

- You understand Vue 3 Composition API basics (ref, reactive, computed, watch)
- You're familiar with modern JavaScript (ES6+) patterns
- You're using `<script setup>` syntax in your components
- You're aiming to create maintainable, scalable code

## Essential Concepts

### 1. Creating Reusable Composables

**Concise Explanation:**
Composables are functions that leverage Vue's Composition API to encapsulate and reuse stateful logic. They follow the "use" prefix convention (e.g., `useCounter`) and can contain refs, computed properties, watchers, lifecycle hooks, and methods that components can import and use.

**Where to Use:**

- When logic needs to be shared across multiple components
- For complex state management that doesn't belong in a single component
- When implementing reusable behaviors like data fetching, form handling, or utilities
- To separate concerns in complex components

**Code Snippet:**

```js
// useCounter.js - A simple counter composable
import { ref, computed } from "vue";

export function useCounter(initialValue = 0, options = {}) {
  // Destructure options with defaults
  const { min = null, max = null } = options;

  // State
  const count = ref(initialValue);

  // Computed properties
  const isMin = computed(() => min !== null && count.value <= min);
  const isMax = computed(() => max !== null && count.value >= max);

  // Actions
  const increment = (amount = 1) => {
    if (max !== null && count.value + amount > max) {
      count.value = max;
    } else {
      count.value += amount;
    }
  };

  const decrement = (amount = 1) => {
    if (min !== null && count.value - amount < min) {
      count.value = min;
    } else {
      count.value -= amount;
    }
  };

  const reset = () => {
    count.value = initialValue;
  };

  // Return everything the composable exposes
  return {
    count,
    isMin,
    isMax,
    increment,
    decrement,
    reset,
  };
}
```

```html
<!-- CounterComponent.vue - Using the composable -->
<template>
  <div class="p-4 bg-white rounded shadow">
    <h2 class="text-xl font-semibold mb-4">Counter: {{ count }}</h2>

    <div class="flex space-x-2">
      <button
        @click="decrement()"
        :disabled="isMin"
        class="px-4 py-2 bg-red-500 text-white rounded disabled:opacity-50"
      >
        Decrease
      </button>

      <button @click="reset" class="px-4 py-2 bg-gray-500 text-white rounded">
        Reset
      </button>

      <button
        @click="increment()"
        :disabled="isMax"
        class="px-4 py-2 bg-green-500 text-white rounded disabled:opacity-50"
      >
        Increase
      </button>
    </div>

    <div class="mt-4 flex space-x-2">
      <button
        @click="decrement(5)"
        :disabled="isMin"
        class="px-4 py-2 bg-red-300 text-white rounded disabled:opacity-50"
      >
        -5
      </button>

      <button
        @click="increment(5)"
        :disabled="isMax"
        class="px-4 py-2 bg-green-300 text-white rounded disabled:opacity-50"
      >
        +5
      </button>
    </div>
  </div>
</template>

<script setup>
  import { useCounter } from "@/composables/useCounter";

  // Use the counter composable with options
  const { count, isMin, isMax, increment, decrement, reset } = useCounter(0, {
    min: 0,
    max: 10,
  });
</script>
```

**More Complex Example - Async Data Fetching:**

```js
// useAsyncData.js - A composable for data fetching with loading, error handling and refetching
import { ref, computed, watch, unref } from "vue";
import { isRef } from "vue";

export function useAsyncData(asyncFn, options = {}) {
  // Destructure options with defaults
  const {
    immediate = true,
    initialData = null,
    resetOnExecute = false,
    onSuccess = null,
    onError = null,
  } = options;

  // State
  const data = ref(initialData);
  const error = ref(null);
  const loading = ref(false);
  const loaded = ref(false);

  // Execute the async function
  const execute = async (...args) => {
    loading.value = true;
    error.value = null;

    if (resetOnExecute) {
      data.value = initialData;
    }

    try {
      const result = await asyncFn(...args);
      data.value = result;
      loaded.value = true;

      if (onSuccess) {
        onSuccess(result);
      }

      return result;
    } catch (err) {
      error.value = err;

      if (onError) {
        onError(err);
      }

      return Promise.reject(err);
    } finally {
      loading.value = false;
    }
  };

  // Watch for reactive params and re-execute
  if (immediate) {
    execute();
  }

  return {
    data,
    error,
    loading,
    loaded,
    execute,
  };
}
```

```html
<!-- UserProfile.vue - Using the async data composable -->
<template>
  <div class="p-6 bg-white rounded shadow">
    <h2 class="text-xl font-semibold mb-4">User Profile</h2>

    <!-- Loading state -->
    <div v-if="loading" class="py-8 flex justify-center">
      <div
        class="animate-spin rounded-full h-10 w-10 border-4 border-blue-500 border-t-transparent"
      ></div>
    </div>

    <!-- Error state -->
    <div v-else-if="error" class="p-4 bg-red-100 text-red-800 rounded">
      <p>{{ error.message || "Failed to load user data" }}</p>
      <button
        @click="execute"
        class="mt-2 px-4 py-2 bg-red-600 text-white rounded"
      >
        Retry
      </button>
    </div>

    <!-- Success state -->
    <div v-else-if="data" class="space-y-4">
      <div class="flex items-center space-x-4">
        <img
          :src="data.avatar"
          :alt="data.name"
          class="w-16 h-16 rounded-full object-cover"
        />

        <div>
          <h3 class="text-lg font-medium">{{ data.name }}</h3>
          <p class="text-gray-600">{{ data.email }}</p>
        </div>
      </div>

      <div class="pt-4 border-t">
        <h4 class="font-medium mb-2">User Stats</h4>
        <div class="grid grid-cols-3 gap-4">
          <div class="p-3 bg-blue-50 rounded text-center">
            <div class="text-xl font-bold text-blue-600">
              {{ data.stats.posts }}
            </div>
            <div class="text-sm text-gray-600">Posts</div>
          </div>
          <div class="p-3 bg-green-50 rounded text-center">
            <div class="text-xl font-bold text-green-600">
              {{ data.stats.followers }}
            </div>
            <div class="text-sm text-gray-600">Followers</div>
          </div>
          <div class="p-3 bg-purple-50 rounded text-center">
            <div class="text-xl font-bold text-purple-600">
              {{ data.stats.following }}
            </div>
            <div class="text-sm text-gray-600">Following</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
  import { useAsyncData } from "@/composables/useAsyncData";

  // Simulated API function
  const fetchUserProfile = async (userId) => {
    // Simulate API delay
    await new Promise((resolve) => setTimeout(resolve, 1500));

    // Simulate random error (20% chance)
    if (Math.random() < 0.2) {
      throw new Error("Failed to fetch user profile");
    }

    // Mock data
    return {
      id: userId,
      name: "Jane Doe",
      email: "jane.doe@example.com",
      avatar: "https://i.pravatar.cc/300?u=jane",
      stats: {
        posts: 42,
        followers: 1024,
        following: 128,
      },
    };
  };

  const { data, error, loading, execute } = useAsyncData(
    () => fetchUserProfile(1),
    {
      onSuccess: (result) => {
        console.log("Successfully loaded user:", result.name);
      },
      onError: (err) => {
        console.error("Error loading user:", err.message);
      },
    }
  );
</script>
```

**Real-World Example:**
In a content management system, you might create a `usePermissions` composable that manages user permissions throughout the application. It would handle checking permissions, caching results, and refreshing when the user role changes, keeping permission logic consistent across components.

```js
// usePermissions.js
import { ref, computed } from "vue";
import { useUserStore } from "@/stores/user";

export function usePermissions() {
  const userStore = useUserStore();
  const permissionsCache = ref(new Map());

  // Clear cache when user or roles change
  watch(
    () => userStore.user?.id,
    () => {
      permissionsCache.value.clear();
    }
  );

  // Check if user has permission
  const hasPermission = async (permission) => {
    if (!userStore.isLoggedIn) return false;

    // Check cache first
    if (permissionsCache.value.has(permission)) {
      return permissionsCache.value.get(permission);
    }

    try {
      // In real app, this would check against backend
      const result = await checkPermissionApi(userStore.user.id, permission);
      permissionsCache.value.set(permission, result);
      return result;
    } catch (error) {
      console.error("Permission check failed:", error);
      return false;
    }
  };

  // Get all permissions for current user (admin utility)
  const getAllPermissions = async () => {
    if (!userStore.isLoggedIn) return [];

    try {
      const permissions = await fetchUserPermissionsApi(userStore.user.id);

      // Update cache
      permissions.forEach((p) => {
        permissionsCache.value.set(p.name, true);
      });

      return permissions;
    } catch (error) {
      console.error("Failed to fetch permissions:", error);
      return [];
    }
  };

  // Computed helpers for common permissions
  const canCreateContent = computed(() => {
    return (
      userStore.user?.role === "admin" ||
      userStore.user?.role === "editor" ||
      permissionsCache.value.get("content:create")
    );
  });

  const canPublishContent = computed(() => {
    return (
      userStore.user?.role === "admin" ||
      permissionsCache.value.get("content:publish")
    );
  });

  return {
    hasPermission,
    getAllPermissions,
    canCreateContent,
    canPublishContent,
  };
}
```

**Common Pitfalls:**

- Returning non-reactive values when they should be reactive
- Over-engineering composables to handle too many concerns
- Not providing options for customization
- Creating side effects without cleanup
- Direct mutation of passed-in arguments

**Frequently Asked Questions:**

1. **Q: How are composables different from mixins?**

   A: Composables are more explicit and flexible than mixins. While mixins have implicit property merging that can cause naming conflicts, composables explicitly return what they expose, creating a clear contract. Composables also benefit from TypeScript support and can be combined more easily without namespace collisions.

2. **Q: Can composables use lifecycle hooks?**

   A: Yes, composables can use lifecycle hooks (`onMounted`, `onUnmounted`, etc.), which will be bound to the component that calls the composable. This allows composables to register and clean up side effects, just like components.

3. **Q: Should I make everything a composable?**

   A: No. Create composables for stateful, reusable logic that might be used across multiple components. For simple utility functions that don't use reactivity, plain JavaScript functions are more appropriate. If the logic is only used in one component and isn't complex, it might not need extraction.

### 2. Composable Organization Patterns

**Concise Explanation:**
Structuring composables effectively is crucial for maintainability. This involves organizing related functionality, establishing naming conventions, and defining clear interfaces. Well-organized composables enhance discoverability, simplify maintenance, and encourage reuse.

**Where to Use:**

- Medium to large applications with multiple composables
- When multiple developers work on the same codebase
- When building a design system or internal framework
- When composables need to interact with each other

**Code Snippet:**

**Directory Structure Example:**

```
src/
├── composables/
│   ├── index.js                 # Re-export all composables
│   ├── core/                    # Core functionality
│   │   ├── useStorage.js        # Local/session storage
│   │   ├── useFetch.js          # Network requests
│   │   └── useEventBus.js       # Event communication
│   ├── ui/                      # UI-related composables
│   │   ├── useBreakpoints.js    # Responsive design
│   │   ├── useScroll.js         # Scroll behavior
│   │   └── useIntersection.js   # Intersection observer
│   ├── form/                    # Form handling
│   │   ├── useForm.js           # Form state
│   │   ├── useField.js          # Field validation
│   │   └── useSubmit.js         # Form submission
│   └── auth/                    # Authentication
│       ├── useAuth.js           # Main auth composable
│       ├── usePermissions.js    # Permission checks
│       └── useProfile.js        # User profile
└── components/
    └── ...
```

**Barrel Export Pattern:**

```js
// composables/index.js - Barrel export pattern
// Re-export all composables for easier imports

// Core
export * from "./core/useStorage";
export * from "./core/useFetch";
export * from "./core/useEventBus";

// UI
export * from "./ui/useBreakpoints";
export * from "./ui/useScroll";
export * from "./ui/useIntersection";

// Form
export * from "./form/useForm";
export * from "./form/useField";
export * from "./form/useSubmit";

// Auth
export * from "./auth/useAuth";
export * from "./auth/usePermissions";
export * from "./auth/useProfile";
```

**Core Utilities Example:**

```js
// composables/core/useStorage.js
import { ref, watch } from "vue";

/**
 * Composable for persistent storage with reactivity
 * @param {string} key - Storage key
 * @param {any} initialValue - Default value if key doesn't exist
 * @param {Object} options - Configuration options
 * @param {boolean} options.session - Use sessionStorage instead of localStorage
 * @returns {Object} Storage utilities
 */
export function useStorage(key, initialValue = null, options = {}) {
  // Options
  const { session = false } = options;
  const storage = session ? sessionStorage : localStorage;

  // Get stored value or use initial
  const readStoredValue = () => {
    try {
      const item = storage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(`Error reading ${key} from storage:`, error);
      return initialValue;
    }
  };

  // Create reactive reference
  const storedValue = ref(readStoredValue());

  // Write to storage when value changes
  watch(
    storedValue,
    (newValue) => {
      try {
        if (newValue === null || newValue === undefined) {
          storage.removeItem(key);
        } else {
          storage.setItem(key, JSON.stringify(newValue));
        }
      } catch (error) {
        console.error(`Error writing ${key} to storage:`, error);
      }
    },
    { deep: true }
  );

  // Remove from storage
  const removeItem = () => {
    try {
      storage.removeItem(key);
      storedValue.value = initialValue;
    } catch (error) {
      console.error(`Error removing ${key} from storage:`, error);
    }
  };

  return {
    value: storedValue,
    removeItem,
  };
}
```

```js
// composables/ui/useBreakpoints.js
import { ref, computed, onMounted, onUnmounted } from "vue";

/**
 * Composable for responsive breakpoints
 * @param {Object} breakpoints - Custom breakpoints (optional)
 * @returns {Object} Reactive breakpoint utilities
 */
export function useBreakpoints(breakpoints = {}) {
  // Default breakpoints (Tailwind-like)
  const defaultBreakpoints = {
    sm: 640,
    md: 768,
    lg: 1024,
    xl: 1280,
    "2xl": 1536,
  };

  // Merge defaults with custom breakpoints
  const screens = { ...defaultBreakpoints, ...breakpoints };

  // Current window width
  const windowWidth = ref(
    typeof window !== "undefined" ? window.innerWidth : 0
  );

  // Update width on resize
  const updateWidth = () => {
    windowWidth.value = window.innerWidth;
  };

  // Setup resize listener
  onMounted(() => {
    window.addEventListener("resize", updateWidth);
    updateWidth();
  });

  // Cleanup
  onUnmounted(() => {
    window.removeEventListener("resize", updateWidth);
  });

  // Create computed properties for each breakpoint
  const isGreaterThan = {};
  const isSmallerThan = {};

  Object.entries(screens).forEach(([name, width]) => {
    isGreaterThan[name] = computed(() => windowWidth.value >= width);
    isSmallerThan[name] = computed(() => windowWidth.value < width);
  });

  // Current active breakpoint
  const current = computed(() => {
    const names = Object.keys(screens).sort((a, b) => screens[a] - screens[b]);

    return (
      names.find((name) => isSmallerThan[name].value) || names[names.length - 1]
    );
  });

  return {
    windowWidth,
    current,
    isGreaterThan,
    isSmallerThan,
    screens,
  };
}
```

**Composable Factory Example:**

```js
// composables/form/useField.js - Factory pattern for form fields
import { ref, computed, watch } from "vue";

/**
 * Creates validation rules for form fields
 */
export const rules = {
  required:
    (message = "This field is required") =>
    (value) => {
      return (value !== null && value !== undefined && value !== "") || message;
    },

  email:
    (message = "Please enter a valid email") =>
    (value) => {
      if (!value) return true; // Skip if empty (use with required)

      const regex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
      return regex.test(value) || message;
    },

  minLength:
    (length, message = `Must be at least ${length} characters`) =>
    (value) => {
      if (!value) return true; // Skip if empty

      return value.length >= length || message;
    },

  maxLength:
    (length, message = `Cannot exceed ${length} characters`) =>
    (value) => {
      if (!value) return true; // Skip if empty

      return value.length <= length || message;
    },

  pattern:
    (regex, message = "Invalid format") =>
    (value) => {
      if (!value) return true; // Skip if empty

      return regex.test(value) || message;
    },
};

/**
 * Factory function to create a field validation composable
 * @param {Object} options - Field options
 * @returns {Object} Field composable
 */
export function createFieldValidation(options = {}) {
  return function useField(name, value, fieldRules = []) {
    // Combine global options with field-specific options
    const mergedOptions = { ...options, ...(arguments[3] || {}) };
    const { validateOnChange = true, validateOnBlur = true } = mergedOptions;

    // Field state
    const fieldValue = ref(value);
    const isDirty = ref(false);
    const isTouched = ref(false);
    const errors = ref([]);

    // Validate the field
    const validate = () => {
      errors.value = [];

      // Run each validation rule
      fieldRules.forEach((rule) => {
        const result = rule(fieldValue.value);

        if (result !== true) {
          errors.value.push(result);
        }
      });

      return errors.value.length === 0;
    };

    // Input event handlers
    const onInput = (e) => {
      fieldValue.value = e.target?.value ?? e;
      isDirty.value = true;

      if (validateOnChange) {
        validate();
      }
    };

    const onBlur = () => {
      isTouched.value = true;

      if (validateOnBlur) {
        validate();
      }
    };

    const reset = () => {
      fieldValue.value = value;
      isDirty.value = false;
      isTouched.value = false;
      errors.value = [];
    };

    // Computed properties
    const isValid = computed(() => errors.value.length === 0);
    const errorMessage = computed(() => errors.value[0]);

    // Expose field API
    return {
      name,
      value: fieldValue,
      isDirty,
      isTouched,
      errors,
      isValid,
      errorMessage,
      validate,
      onInput,
      onBlur,
      reset,
    };
  };
}

// Create the default useField composable
export const useField = createFieldValidation();
```

```html
<!-- Using the field composable with factory pattern -->
<template>
  <div class="p-6 bg-white rounded shadow">
    <h2 class="text-xl font-semibold mb-4">Registration Form</h2>

    <form @submit.prevent="submitForm" class="space-y-4">
      <!-- Email field -->
      <div>
        <label class="block mb-1 text-sm font-medium text-gray-700">
          Email Address
        </label>
        <input
          type="email"
          :value="email.value"
          @input="email.onInput"
          @blur="email.onBlur"
          class="w-full px-3 py-2 border rounded"
          :class="{
            'border-red-500': email.isDirty && !email.isValid,
            'border-green-500': email.isDirty && email.isValid,
          }"
        />
        <p
          v-if="email.isDirty && !email.isValid"
          class="mt-1 text-sm text-red-600"
        >
          {{ email.errorMessage }}
        </p>
      </div>

      <!-- Password field -->
      <div>
        <label class="block mb-1 text-sm font-medium text-gray-700">
          Password
        </label>
        <input
          type="password"
          :value="password.value"
          @input="password.onInput"
          @blur="password.onBlur"
          class="w-full px-3 py-2 border rounded"
          :class="{
            'border-red-500': password.isDirty && !password.isValid,
            'border-green-500': password.isDirty && password.isValid,
          }"
        />
        <p
          v-if="password.isDirty && !password.isValid"
          class="mt-1 text-sm text-red-600"
        >
          {{ password.errorMessage }}
        </p>
      </div>

      <!-- Confirm Password field -->
      <div>
        <label class="block mb-1 text-sm font-medium text-gray-700">
          Confirm Password
        </label>
        <input
          type="password"
          :value="confirmPassword.value"
          @input="confirmPassword.onInput"
          @blur="confirmPassword.onBlur"
          class="w-full px-3 py-2 border rounded"
          :class="{
            'border-red-500':
              confirmPassword.isDirty && !confirmPassword.isValid,
            'border-green-500':
              confirmPassword.isDirty && confirmPassword.isValid,
          }"
        />
        <p
          v-if="confirmPassword.isDirty && !confirmPassword.isValid"
          class="mt-1 text-sm text-red-600"
        >
          {{ confirmPassword.errorMessage }}
        </p>
      </div>

      <!-- Submit button -->
      <button
        type="submit"
        :disabled="!isFormValid"
        class="w-full px-4 py-2 bg-blue-600 text-white rounded disabled:opacity-50"
      >
        Register
      </button>
    </form>
  </div>
</template>

<script setup>
  import { computed } from "vue";
  import { useField, rules } from "@/composables/form/useField";

  // Create strict field validation that validates on every change
  const strictField = createFieldValidation({
    validateOnChange: true,
    validateOnBlur: true,
  });

  // Field validations
  const email = useField("email", "", [rules.required(), rules.email()]);

  const password = strictField("password", "", [
    rules.required(),
    rules.minLength(8),
    rules.pattern(
      /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).+$/,
      "Password must include uppercase, lowercase, and numbers"
    ),
  ]);

  const confirmPassword = strictField("confirmPassword", "", [
    rules.required(),
    (value) => value === password.value.value || "Passwords must match",
  ]);

  // Computed form validity
  const isFormValid = computed(() => {
    return (
      email.isValid.value &&
      password.isValid.value &&
      confirmPassword.isValid.value
    );
  });

  // Form submission
  const submitForm = () => {
    // Validate all fields
    const isValid = [
      email.validate(),
      password.validate(),
      confirmPassword.validate(),
    ].every(Boolean);

    if (isValid) {
      console.log("Form submitted with:", {
        email: email.value.value,
        password: password.value.value,
      });
    }
  };
</script>
```

**Real-World Example:**
In an enterprise dashboard application, composables are organized by domain (users, reports, notifications) with cross-cutting concerns separated into core utilities. Each domain's folder contains specialized composables that build on core utilities, following consistent patterns for API interaction and state management.

**Common Pitfalls:**

- Inconsistent naming conventions leading to confusion
- Circular dependencies between composables
- Excessive fragmentation making composition difficult
- Monolithic composables that try to do too much
- Poor documentation of parameters and return values

**Frequently Asked Questions:**

1. **Q: Should I group composables by feature or by function type?**

   A: In most cases, grouping by feature or domain is more maintainable as your app grows. This keeps related functionality together, following the principle that things that change together should stay together. For very large applications, a hybrid approach may work best: organize by domain first, then by function type within each domain.

2. **Q: How do I handle dependencies between composables?**

   A: For explicit dependencies, pass one composable's return values to another as arguments. For implicit dependencies, use a layered approach where foundational composables are imported by higher-level ones. Avoid circular dependencies by keeping a clear hierarchy of composables.

3. **Q: How detailed should my composable documentation be?**

   A: At minimum, document:

   - Purpose of the composable
   - Input parameters with types and defaults
   - Return values and their meanings
   - Side effects (if any)
   - Example usage

   Consider using JSDoc for structured documentation that integrates with IDE tooling.

### 3. Sharing State Between Components

**Concise Explanation:**
Composables provide an elegant way to share state and logic between components without the complexity of a full state management solution. They can create a central source of truth that multiple components can access, while maintaining reactivity and encapsulation.

**Where to Use:**

- When multiple components need access to the same state
- For real-time features like notifications or chat
- When sharing state between distant components in the component tree
- As a lightweight alternative to Pinia/Vuex for simple state sharing

**Code Snippet:**

**Simple State Sharing:**

```js
// composables/useSharedState.js
import { ref, readonly } from "vue";

// State is created outside the function to be shared
// between all instances that import this composable
const count = ref(0);
const users = ref([]);

export function useSharedState() {
  // Methods to update state
  const increment = () => {
    count.value++;
  };

  const decrement = () => {
    count.value--;
  };

  const addUser = (user) => {
    users.value.push(user);
  };

  const removeUser = (userId) => {
    const index = users.value.findIndex((u) => u.id === userId);
    if (index !== -1) {
      users.value.splice(index, 1);
    }
  };

  // Return readonly state but mutable methods
  return {
    // Use readonly to prevent direct mutation from components
    count: readonly(count),
    users: readonly(users),

    // Methods to safely mutate state
    increment,
    decrement,
    addUser,
    removeUser,
  };
}
```

**Service Pattern with Event Bus:**

```js
// composables/useNotifications.js
import { ref, readonly } from "vue";

// Private variables
const NOTIFICATION_TIMEOUT = 5000;
let nextId = 0;

// Shared state
const notifications = ref([]);

export function useNotifications() {
  // Add a notification
  const notify = (message, options = {}) => {
    const {
      type = "info",
      timeout = NOTIFICATION_TIMEOUT,
      title = "",
      closable = true,
    } = options;

    // Create new notification
    const id = nextId++;
    const notification = {
      id,
      type,
      message,
      title,
      closable,
      createdAt: new Date(),
    };

    // Add to list
    notifications.value.push(notification);

    // Auto remove after timeout
    if (timeout > 0) {
      setTimeout(() => {
        removeNotification(id);
      }, timeout);
    }

    return id;
  };

  // Shorthand methods
  const success = (message, options = {}) =>
    notify(message, { type: "success", ...options });
  const error = (message, options = {}) =>
    notify(message, { type: "error", ...options });
  const warn = (message, options = {}) =>
    notify(message, { type: "warning", ...options });
  const info = (message, options = {}) =>
    notify(message, { type: "info", ...options });

  // Remove a notification
  const removeNotification = (id) => {
    const index = notifications.value.findIndex((n) => n.id === id);
    if (index !== -1) {
      notifications.value.splice(index, 1);
    }
  };

  // Clear all notifications
  const clearNotifications = () => {
    notifications.value = [];
  };

  return {
    // State (readonly)
    notifications: readonly(notifications),

    // Methods
    notify,
    success,
    error,
    warn,
    info,
    removeNotification,
    clearNotifications,
  };
}
```

```html
<!-- NotificationsProvider.vue -->
<template>
  <!-- Notification container -->
  <div class="fixed top-4 right-4 z-50 space-y-2 max-w-sm">
    <transition-group name="notification">
      <div
        v-for="notification in notifications"
        :key="notification.id"
        :class="[
          'p-4 rounded shadow-lg border-l-4 flex items-start',
          notificationClass(notification.type),
        ]"
      >
        <!-- Icon -->
        <div class="mr-3">
          <svg
            v-if="notification.type === 'success'"
            class="h-5 w-5 text-green-500"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M5 13l4 4L19 7"
            />
          </svg>
          <svg
            v-else-if="notification.type === 'error'"
            class="h-5 w-5 text-red-500"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M6 18L18 6M6 6l12 12"
            />
          </svg>
          <svg
            v-else-if="notification.type === 'warning'"
            class="h-5 w-5 text-yellow-500"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z"
            />
          </svg>
          <svg
            v-else
            class="h-5 w-5 text-blue-500"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
            />
          </svg>
        </div>

        <!-- Content -->
        <div class="flex-1">
          <h3 v-if="notification.title" class="font-medium">
            {{ notification.title }}
          </h3>
          <p :class="{ 'mt-1': notification.title }">
            {{ notification.message }}
          </p>
        </div>

        <!-- Close button -->
        <button
          v-if="notification.closable"
          @click="removeNotification(notification.id)"
          class="ml-3 text-gray-400 hover:text-gray-900"
        >
          <svg
            class="h-4 w-4"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M6 18L18 6M6 6l12 12"
            />
          </svg>
        </button>
      </div>
    </transition-group>
  </div>

  <!-- Pass any children through -->
  <slot></slot>
</template>

<script setup>
  import { useNotifications } from "@/composables/useNotifications";

  const { notifications, removeNotification } = useNotifications();

  // Determine CSS classes based on notification type
  const notificationClass = (type) => {
    switch (type) {
      case "success":
        return "bg-green-50 border-green-500 text-green-800";
      case "error":
        return "bg-red-50 border-red-500 text-red-800";
      case "warning":
        return "bg-yellow-50 border-yellow-500 text-yellow-800";
      default:
        return "bg-blue-50 border-blue-500 text-blue-800";
    }
  };
</script>

<style scoped>
  .notification-enter-active,
  .notification-leave-active {
    transition: all 0.3s ease;
  }
  .notification-enter-from {
    opacity: 0;
    transform: translateX(30px);
  }
  .notification-leave-to {
    opacity: 0;
    transform: translateY(-30px);
  }
</style>
```

```html
<!-- Usage in a component -->
<template>
  <div class="p-6 bg-white rounded shadow">
    <h2 class="text-xl font-semibold mb-4">Notifications Demo</h2>

    <div class="space-y-2">
      <button
        @click="showSuccessNotification"
        class="px-4 py-2 bg-green-600 text-white rounded"
      >
        Success Notification
      </button>

      <button
        @click="showErrorNotification"
        class="px-4 py-2 bg-red-600 text-white rounded"
      >
        Error Notification
      </button>

      <button
        @click="showWarningNotification"
        class="px-4 py-2 bg-yellow-600 text-white rounded"
      >
        Warning Notification
      </button>

      <button
        @click="showInfoNotification"
        class="px-4 py-2 bg-blue-600 text-white rounded"
      >
        Info Notification
      </button>
    </div>
  </div>
</template>

<script setup>
  import { useNotifications } from "@/composables/useNotifications";

  // Use the shared notifications service
  const { success, error, warn, info } = useNotifications();

  // Show different notification types
  const showSuccessNotification = () => {
    success("Operation completed successfully!", {
      title: "Success",
    });
  };

  const showErrorNotification = () => {
    error("An error occurred during the operation.", {
      title: "Error",
      timeout: 10000, // Longer timeout for errors
    });
  };

  const showWarningNotification = () => {
    warn("This action cannot be undone.", {
      title: "Warning",
    });
  };

  const showInfoNotification = () => {
    info("Your session will expire in 10 minutes.");
  };
</script>
```

**Global Store Pattern:**

```js
// composables/useUserStore.js - A lightweight store pattern
import { reactive, computed, readonly } from "vue";
import { useStorage } from "./useStorage";

// Create persistent state with useStorage
const { value: persistedUser } = useStorage("user", null);

// Global state
const state = reactive({
  user: persistedUser.value,
  isLoggingIn: false,
  error: null,
});

// Actions/mutations
const actions = {
  async login(email, password) {
    state.isLoggingIn = true;
    state.error = null;

    try {
      // Simulated API request
      const response = await fetch("/api/login", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ email, password }),
      });

      if (!response.ok) {
        throw new Error("Invalid credentials");
      }

      const user = await response.json();
      state.user = user;
      persistedUser.value = user;
      return user;
    } catch (err) {
      state.error = err.message;
      throw err;
    } finally {
      state.isLoggingIn = false;
    }
  },

  logout() {
    state.user = null;
    persistedUser.value = null;
  },

  updateProfile(profileData) {
    state.user = { ...state.user, ...profileData };
    persistedUser.value = state.user;
  },
};

// Getters
const getters = {
  isLoggedIn: computed(() => !!state.user),
  username: computed(() => state.user?.username || "Guest"),
  userRole: computed(() => state.user?.role || "visitor"),
  isAdmin: computed(() => state.user?.role === "admin"),
};

// Create the composable
export function useUserStore() {
  return {
    // State (readonly to prevent direct mutation)
    state: readonly(state),

    // Getters
    ...getters,

    // Actions
    ...actions,
  };
}
```

**Real-World Example:**
In a collaborative editing application, a `useDocumentCollaboration` composable manages shared document state, changes from multiple users, and synchronization with a backend. Components across the application can access the same document instance, apply changes, and see updates from other users in real-time.

**Common Pitfalls:**

- Creating too many isolated state instances when shared state is needed
- Not protecting shared state from direct mutation by components
- Memory leaks from shared state that persists after components unmount
- Race conditions when multiple components update shared state concurrently
- Tight coupling between components through shared state

**Frequently Asked Questions:**

1. **Q: When should I use composables for state sharing vs. Pinia/Vuex?**

   A: Use composables for simpler state sharing needs:

   - When the state is limited to a specific feature
   - When you don't need time-travel debugging or complex state inspection
   - When you prefer a more lightweight solution

   Use Pinia/Vuex when:

   - You need centralized state management for the entire application
   - Your state has complex interactions between different modules
   - You need advanced features like time-travel debugging
   - Your team is familiar with the Flux/Redux pattern

2. **Q: How do I prevent direct mutation of shared state?**

   A: Use `readonly()` when returning state from your composable. This creates a read-only proxy that throws an error when components try to modify it directly. To allow controlled mutations, provide methods that modify the internal state.

   ```js
   // Inside the composable
   const state = reactive({ count: 0 });

   return {
     state: readonly(state), // Read-only state
     increment: () => state.count++, // Method for controlled mutation
   };
   ```

3. **Q: How can I share state only between specific components?**

   A: Use Vue's provide/inject mechanism in combination with composables:

   ```js
   // Parent component
   import { provide, reactive } from "vue";

   // Create state in the parent
   const sharedState = reactive({
     /* ... */
   });

   // Provide it to children
   provide("sharedState", sharedState);

   // In child components
   import { inject } from "vue";

   // Inject and use the shared state
   const sharedState = inject("sharedState");
   ```

   This limits state sharing to the provided component and its descendants.

### 4. Error Handling in Composables

**Concise Explanation:**
Effective error handling in composables involves capturing, processing, and communicating errors in a consistent way. Well-designed error handling makes applications more robust, improves user experience, and simplifies debugging.

**Where to Use:**

- In composables that interact with external systems (APIs, storage, etc.)
- When performing operations that might fail (parsing, calculations, validations)
- For user-facing features requiring specific error messages
- In critical application paths that need graceful degradation

**Code Snippet:**

**Basic Error Handling Pattern:**

```js
// composables/useApiRequest.js
import { ref, shallowRef } from "vue";

export function useApiRequest(url, options = {}) {
  const data = ref(null);
  const error = ref(null);
  const loading = ref(false);

  // Use shallowRef for the response object to avoid deep reactivity overhead
  const response = shallowRef(null);

  const execute = async (body = null, customOptions = {}) => {
    // Reset state
    loading.value = true;
    error.value = null;
    data.value = null;

    try {
      // Merge options
      const fetchOptions = {
        ...options,
        ...customOptions,
        headers: {
          "Content-Type": "application/json",
          ...options.headers,
          ...customOptions.headers,
        },
      };

      // Add body if provided
      if (body) {
        fetchOptions.body = JSON.stringify(body);
      }

      // Make the request
      const res = await fetch(url, fetchOptions);
      response.value = res;

      // Handle HTTP error status
      if (!res.ok) {
        let errorData;
        try {
          // Try to parse error response as JSON
          errorData = await res.json();
        } catch (e) {
          // If not JSON, use status text
          errorData = { message: res.statusText };
        }

        // Create a custom error with additional data
        const apiError = new Error(errorData.message || "API request failed");
        apiError.status = res.status;
        apiError.data = errorData;
        throw apiError;
      }

      // Parse successful response
      const responseData = await res.json();
      data.value = responseData;

      return { data: responseData, response: res };
    } catch (err) {
      error.value = err;
      // Re-throw for caller to handle if needed
      throw err;
    } finally {
      loading.value = false;
    }
  };

  return {
    data,
    error,
    loading,
    response,
    execute,
  };
}
```

**Error Classification Pattern:**

```js
// errors/apiErrors.js - Defining custom error types
export class ApiError extends Error {
  constructor(message, status, data = null) {
    super(message || "API request failed");
    this.name = "ApiError";
    this.status = status;
    this.data = data;
  }

  isAuthError() {
    return this.status === 401 || this.status === 403;
  }

  isNetworkError() {
    return this.status === 0 || this.message.includes("network");
  }

  isServerError() {
    return this.status >= 500;
  }

  isClientError() {
    return this.status >= 400 && this.status < 500;
  }
}

export class AuthError extends ApiError {
  constructor(message, status = 401, data = null) {
    super(message || "Authentication failed", status, data);
    this.name = "AuthError";
  }
}

export class NetworkError extends ApiError {
  constructor(message) {
    super(message || "Network error", 0);
    this.name = "NetworkError";
  }
}

export class ValidationError extends ApiError {
  constructor(message, fieldErrors = {}) {
    super(message || "Validation failed", 422, { fieldErrors });
    this.name = "ValidationError";
    this.fieldErrors = fieldErrors;
  }

  getFieldError(field) {
    return this.fieldErrors[field] || null;
  }
}
```

```js
// composables/useApi.js - Using custom error types
import { ref, shallowRef } from "vue";
import {
  ApiError,
  AuthError,
  NetworkError,
  ValidationError,
} from "@/errors/apiErrors";

export function useApi(baseUrl = "/api") {
  const defaultOptions = {
    headers: {
      "Content-Type": "application/json",
    },
  };

  // Create a request composable with error classification
  const createRequest = (endpoint, method = "GET", options = {}) => {
    const url = `${baseUrl}${endpoint}`;
    const requestOptions = { ...defaultOptions, ...options, method };

    const data = ref(null);
    const error = ref(null);
    const loading = ref(false);
    const response = shallowRef(null);

    const execute = async (body = null, customOptions = {}) => {
      loading.value = true;
      error.value = null;

      try {
        // Merge options
        const fetchOptions = {
          ...requestOptions,
          ...customOptions,
          headers: {
            ...requestOptions.headers,
            ...customOptions.headers,
          },
        };

        // Add body if provided
        if (
          body &&
          (method === "POST" || method === "PUT" || method === "PATCH")
        ) {
          fetchOptions.body = JSON.stringify(body);
        }

        // Handle network errors
        try {
          response.value = await fetch(url, fetchOptions);
        } catch (networkErr) {
          throw new NetworkError(networkErr.message);
        }

        const res = response.value;

        // Handle HTTP error status
        if (!res.ok) {
          let errorData = {};

          // Try to parse error response
          try {
            errorData = await res.json();
          } catch (e) {
            // Not JSON, use status text
            errorData = { message: res.statusText };
          }

          // Classify error by status code
          if (res.status === 401 || res.status === 403) {
            throw new AuthError(errorData.message, res.status, errorData);
          } else if (res.status === 422) {
            throw new ValidationError(
              errorData.message,
              errorData.errors || {}
            );
          } else {
            throw new ApiError(errorData.message, res.status, errorData);
          }
        }

        // Parse JSON response or return text for non-JSON
        let responseData;
        const contentType = res.headers.get("content-type");

        if (contentType && contentType.includes("application/json")) {
          responseData = await res.json();
        } else {
          responseData = await res.text();
        }

        data.value = responseData;
        return responseData;
      } catch (err) {
        error.value = err;
        throw err;
      } finally {
        loading.value = false;
      }
    };

    return {
      data,
      error,
      loading,
      response,
      execute,
    };
  };

  // Shorthand methods for common HTTP verbs
  return {
    get: (endpoint, options) => createRequest(endpoint, "GET", options),
    post: (endpoint, options) => createRequest(endpoint, "POST", options),
    put: (endpoint, options) => createRequest(endpoint, "PUT", options),
    patch: (endpoint, options) => createRequest(endpoint, "PATCH", options),
    delete: (endpoint, options) => createRequest(endpoint, "DELETE", options),
  };
}
```

**Form Validation Error Handling:**

```js
// composables/useForm.js - Form handling with validation errors
import { reactive, computed, ref } from "vue";
import { ValidationError } from "@/errors/apiErrors";

export function useForm(initialValues = {}, options = {}) {
  // Form state
  const values = reactive({ ...initialValues });
  const errors = reactive({});
  const touched = reactive({});

  // Form meta state
  const isSubmitting = ref(false);
  const isValidating = ref(false);
  const submitCount = ref(0);
  const submitError = ref(null);

  // Extract options
  const { validate: validateFn, onSubmit: onSubmitFn } = options;

  // Mark all fields as touched
  const touchAll = () => {
    Object.keys(values).forEach((key) => {
      touched[key] = true;
    });
  };

  // Reset form
  const reset = () => {
    Object.keys(values).forEach((key) => {
      values[key] = initialValues[key];
      delete errors[key];
      delete touched[key];
    });
    submitError.value = null;
  };

  // Run validation
  const validate = async () => {
    if (!validateFn) return true;

    isValidating.value = true;
    errors.value = {};

    try {
      await validateFn(values);
      return true;
    } catch (err) {
      if (err instanceof ValidationError) {
        // Process validation errors from API
        Object.entries(err.fieldErrors).forEach(([field, message]) => {
          errors[field] = message;
        });
      } else if (typeof err === "object") {
        // Process validation errors from local validation
        Object.entries(err).forEach(([field, message]) => {
          errors[field] = message;
        });
      }
      return false;
    } finally {
      isValidating.value = false;
    }
  };

  // Handle submission
  const handleSubmit = async (e) => {
    if (e) e.preventDefault();

    submitCount.value++;
    touchAll();

    // Validate before submitting
    const isValid = await validate();
    if (!isValid) return;

    isSubmitting.value = true;
    submitError.value = null;

    try {
      if (onSubmitFn) {
        await onSubmitFn(values);
      }
    } catch (err) {
      submitError.value = err;

      if (err instanceof ValidationError) {
        // Process validation errors from API
        Object.entries(err.fieldErrors).forEach(([field, message]) => {
          errors[field] = message;
        });
      }

      throw err;
    } finally {
      isSubmitting.value = false;
    }
  };

  // Computed properties
  const isValid = computed(() => Object.keys(errors).length === 0);
  const isDirty = computed(() => {
    return Object.keys(initialValues).some((key) => {
      return values[key] !== initialValues[key];
    });
  });

  return {
    // State
    values,
    errors,
    touched,
    isSubmitting,
    isValidating,
    submitCount,
    submitError,

    // Computed
    isValid,
    isDirty,

    // Methods
    handleSubmit,
    validate,
    reset,
    touchAll,
  };
}
```

```html
<!-- Using the form composable with error handling -->
<template>
  <div class="p-6 bg-white rounded shadow">
    <h2 class="text-xl font-semibold mb-4">User Profile</h2>

    <!-- Form with error handling -->
    <form @submit="handleSubmit" class="space-y-4">
      <!-- Name field -->
      <div>
        <label class="block mb-1 text-sm font-medium text-gray-700">
          Full Name
        </label>
        <input
          v-model="values.name"
          type="text"
          @blur="touched.name = true"
          class="w-full px-3 py-2 border rounded"
          :class="{ 'border-red-500': touched.name && errors.name }"
        />
        <p v-if="touched.name && errors.name" class="mt-1 text-sm text-red-600">
          {{ errors.name }}
        </p>
      </div>

      <!-- Email field -->
      <div>
        <label class="block mb-1 text-sm font-medium text-gray-700">
          Email Address
        </label>
        <input
          v-model="values.email"
          type="email"
          @blur="touched.email = true"
          class="w-full px-3 py-2 border rounded"
          :class="{ 'border-red-500': touched.email && errors.email }"
        />
        <p
          v-if="touched.email && errors.email"
          class="mt-1 text-sm text-red-600"
        >
          {{ errors.email }}
        </p>
      </div>

      <!-- Role field -->
      <div>
        <label class="block mb-1 text-sm font-medium text-gray-700">
          Role
        </label>
        <select
          v-model="values.role"
          @blur="touched.role = true"
          class="w-full px-3 py-2 border rounded"
          :class="{ 'border-red-500': touched.role && errors.role }"
        >
          <option value="">Select a role</option>
          <option value="user">User</option>
          <option value="editor">Editor</option>
          <option value="admin">Administrator</option>
        </select>
        <p v-if="touched.role && errors.role" class="mt-1 text-sm text-red-600">
          {{ errors.role }}
        </p>
      </div>

      <!-- Global form error -->
      <div v-if="submitError" class="p-3 bg-red-100 text-red-800 rounded">
        {{ submitError.message || "An error occurred while submitting the form"
        }}
      </div>

      <!-- Submit button -->
      <button
        type="submit"
        :disabled="isSubmitting"
        class="w-full px-4 py-2 bg-blue-600 text-white rounded disabled:opacity-50"
      >
        <span v-if="isSubmitting">
          <span
            class="animate-spin inline-block h-4 w-4 border-2 border-white border-t-transparent rounded-full mr-1"
          ></span>
          Saving...
        </span>
        <span v-else>Save Profile</span>
      </button>
    </form>
  </div>
</template>

<script setup>
  import { useForm } from "@/composables/useForm";
  import { useApi } from "@/composables/useApi";
  import { ValidationError } from "@/errors/apiErrors";

  // Create API client
  const api = useApi();
  const updateProfile = api.put("/users/profile");

  // Form validation function
  const validateProfile = (values) => {
    const errors = {};

    if (!values.name) {
      errors.name = "Name is required";
    }

    if (!values.email) {
      errors.email = "Email is required";
    } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(values.email)) {
      errors.email = "Invalid email format";
    }

    if (!values.role) {
      errors.role = "Please select a role";
    }

    if (Object.keys(errors).length) {
      throw errors;
    }
  };

  // Submit handler
  const handleProfileSubmit = async (values) => {
    try {
      await updateProfile.execute(values);
    } catch (error) {
      if (error instanceof ValidationError) {
        // ValidationError is already handled by useForm
        throw error;
      } else {
        // Rethrow as validation error to display in form
        throw new ValidationError("Failed to update profile: " + error.message);
      }
    }
  };

  // Use form composable
  const { values, errors, touched, isSubmitting, submitError, handleSubmit } =
    useForm(
      // Initial values
      {
        name: "John Doe",
        email: "john.doe@example.com",
        role: "user",
      },
      // Options
      {
        validate: validateProfile,
        onSubmit: handleProfileSubmit,
      }
    );
</script>
```

**Retry Pattern:**

```js
// composables/useRetry.js - Retry pattern for error handling
import { ref } from "vue";

export function useRetry(asyncFn, options = {}) {
  // Options with defaults
  const {
    maxRetries = 3,
    initialDelay = 1000,
    maxDelay = 10000,
    backoffFactor = 2,
    retryableErrors = [],
    onRetry = null,
  } = options;

  // State
  const retryCount = ref(0);
  const isRetrying = ref(false);

  // Check if an error should be retried
  const shouldRetry = (error) => {
    // Don't retry if we've hit the max
    if (retryCount.value >= maxRetries) {
      return false;
    }

    // If retryableErrors is empty, retry all errors
    if (retryableErrors.length === 0) {
      return true;
    }

    // Check if error is in the list of retryable errors
    return retryableErrors.some((errorType) => {
      return (
        error instanceof errorType ||
        error.name === errorType ||
        error.status === errorType
      );
    });
  };

  // Calculate delay with exponential backoff
  const getRetryDelay = () => {
    const delay = initialDelay * Math.pow(backoffFactor, retryCount.value);
    return Math.min(delay, maxDelay);
  };

  // Execute function with retry
  const execute = async (...args) => {
    // Reset state
    retryCount.value = 0;
    isRetrying.value = false;

    // Try to execute
    try {
      return await asyncFn(...args);
    } catch (error) {
      if (shouldRetry(error)) {
        return await retry(error, ...args);
      }
      throw error;
    }
  };

  // Retry logic
  const retry = async (error, ...args) => {
    isRetrying.value = true;

    try {
      // Increment retry count
      retryCount.value++;

      // Calculate delay
      const delay = getRetryDelay();

      // Notify of retry if callback provided
      if (onRetry) {
        onRetry(retryCount.value, delay, error);
      }

      // Wait before retrying
      await new Promise((resolve) => setTimeout(resolve, delay));

      // Try again
      try {
        return await asyncFn(...args);
      } catch (retryError) {
        if (shouldRetry(retryError)) {
          return await retry(retryError, ...args);
        }
        throw retryError;
      }
    } finally {
      isRetrying.value = false;
    }
  };

  return {
    execute,
    retryCount,
    isRetrying,
  };
}
```

```html
<!-- Using the retry pattern -->
<template>
  <div class="p-6 bg-white rounded shadow">
    <h2 class="text-xl font-semibold mb-4">Retry Example</h2>
    <!-- Using the retry pattern -->
    <template>
      <div class="p-6 bg-white rounded shadow">
        <h2 class="text-xl font-semibold mb-4">File Upload with Retry</h2>

        <!-- File input -->
        <div class="mb-4">
          <label class="block mb-1 text-sm font-medium text-gray-700">
            Select a file to upload
          </label>
          <input
            type="file"
            @change="onFileSelected"
            class="w-full text-sm text-gray-500
               file:mr-4 file:py-2 file:px-4
               file:rounded file:border-0
               file:text-sm file:font-medium
               file:bg-blue-50 file:text-blue-700
               hover:file:bg-blue-100"
          />
        </div>

        <!-- Upload button -->
        <button
          @click="uploadFile"
          :disabled="!selectedFile || isUploading"
          class="px-4 py-2 bg-blue-600 text-white rounded disabled:opacity-50"
        >
          <template v-if="isUploading">
            <span v-if="isRetrying" class="flex items-center">
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
              Retrying... (Attempt {{ retryCount }}/3)
            </span>
            <span v-else class="flex items-center">
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
              Uploading...
            </span>
          </template>
          <span v-else>Upload File</span>
        </button>

        <!-- Upload status -->
        <div class="mt-4">
          <div
            v-if="uploadStatus.success"
            class="p-3 bg-green-100 text-green-800 rounded"
          >
            File uploaded successfully!
          </div>

          <div
            v-if="uploadStatus.error"
            class="p-3 bg-red-100 text-red-800 rounded"
          >
            <p class="font-medium">Upload failed:</p>
            <p>{{ uploadStatus.error }}</p>
            <button
              v-if="!isRetrying && retryCount < 3"
              @click="uploadFile"
              class="mt-2 px-3 py-1 bg-red-600 text-white text-sm rounded"
            >
              Try Again
            </button>
          </div>

          <div
            v-if="retryCount > 0 && uploadStatus.success"
            class="mt-2 text-sm text-gray-600"
          >
            Succeeded after {{ retryCount }} {{ retryCount === 1 ? "retry" :
            "retries" }}.
          </div>
        </div>
      </div>
    </template>

    <script setup>
      import { ref, reactive } from "vue";
      import { useRetry } from "@/composables/useRetry";
      import { NetworkError } from "@/errors/apiErrors";

      // State
      const selectedFile = ref(null);
      const isUploading = ref(false);
      const uploadStatus = reactive({
        success: false,
        error: null,
      });

      // Handle file selection
      const onFileSelected = (event) => {
        selectedFile.value = event.target.files[0] || null;

        // Reset status when new file is selected
        uploadStatus.success = false;
        uploadStatus.error = null;
      };

      // Simulated file upload function with random failures
      const uploadFileToServer = async (file) => {
        if (!file) throw new Error("No file selected");

        // Simulate upload API call
        await new Promise((resolve) => setTimeout(resolve, 1500));

        // Randomly fail sometimes to demonstrate retry
        if (Math.random() < 0.7) {
          throw new NetworkError(
            "Network connection interrupted during upload"
          );
        }

        // Return success
        return {
          name: file.name,
          size: file.size,
          url: URL.createObjectURL(file),
        };
      };

      // Create a retry wrapper for the upload
      const {
        execute: executeUpload,
        retryCount,
        isRetrying,
      } = useRetry(uploadFileToServer, {
        maxRetries: 3,
        initialDelay: 1000,
        backoffFactor: 1.5,
        retryableErrors: [NetworkError],
        onRetry: (attempt, delay, error) => {
          console.log(
            `Retrying upload (${attempt}/3) after ${delay}ms due to: ${error.message}`
          );
        },
      });

      // Handle file upload with retry
      const uploadFile = async () => {
        if (!selectedFile.value) return;

        isUploading.value = true;
        uploadStatus.success = false;
        uploadStatus.error = null;

        try {
          const result = await executeUpload(selectedFile.value);
          uploadStatus.success = true;
          console.log("Upload successful:", result);
        } catch (error) {
          uploadStatus.error = error.message;
          console.error("Upload failed:", error);
        } finally {
          isUploading.value = false;
        }
      };
    </script>
  </div>
</template>
```

**Real-World Example:**
In a financial application, error handling in composables is critical for transaction processing. A `usePaymentProcessor` composable would implement specialized error types for different payment failures (insufficient funds, expired card, fraud detection), with specific handling strategies for each. Retry logic would be applied selectively based on error type and transaction importance.

**Common Pitfalls:**

- Not differentiating between different types of errors
- Silently swallowing errors instead of propagating or handling them
- Overly generic error messages that don't help users resolve issues
- Retrying operations that should not be retried (e.g., failed validations)
- Not cleaning up resources when errors occur

**Frequently Asked Questions:**

1. **Q: Should I throw errors from composables or return them?**

   A: This depends on your use case:

   - **Throw errors** when the calling component needs to know about failures to proceed (e.g., API requests, critical operations).
   - **Return errors** as part of the state when the component should handle the error as just another state (e.g., validation errors, non-critical operations).

   A common pattern is to do both: expose an `error` ref in the returned state and also throw errors for immediate handling:

   ```js
   const { data, error, loading, execute } = useDataFetcher()

   try {
     await execute()
   } catch (e) {
     // Immediate error handling (e.g., show a notification)
   }

   // In the template, also handle the error state
   <div v-if="error">Error: {{ error.message }}</div>
   ```

2. **Q: How do I handle errors in async composables used in setup?**

   A: Since `setup` or `<script setup>` can't be async, you have three options:

   - Use a combination of lifecycle hooks and try/catch:

     ```js
     onMounted(async () => {
       try {
         await someAsyncOperation();
       } catch (error) {
         handleError(error);
       }
     });
     ```

   - Use the `error` state from the composable and reactive logic:

     ```js
     const { data, error, loading, execute } = useAsyncData();

     // Start the async operation
     execute();

     // Use watchers to react to the error state
     watch(error, (newError) => {
       if (newError) {
         handleError(newError);
       }
     });
     ```

   - For Vue 3.3+, use the Suspense component to handle async setup functions.

3. **Q: When should I use retry logic in my composables?**

   A: Use retry logic for operations that might fail due to temporary issues:

   - Network requests that might fail due to connectivity issues
   - Rate-limited API calls that might succeed after a delay
   - Operations that depend on external resources that might be temporarily unavailable

   Avoid retrying:

   - Operations that failed due to invalid input (validation errors)
   - Authentication failures (wrong credentials)
   - Operations that might cause side effects if repeated

### 5. Testing Composables

**Concise Explanation:**
Testing composables involves verifying their behavior, reactivity, and error handling without the context of a component. Good composable tests ensure functionality remains consistent as the application evolves and validate the contract between composables and components.

**Where to Use:**

- Unit testing core composable logic
- Testing reactivity behavior
- Ensuring error handling works correctly
- Validating composable integrations
- Creating test mocks of composables for component testing

**Code Snippet:**

**Basic Composable Test - Counter:**

```js
// composables/useCounter.js
import { ref, computed } from "vue";

export function useCounter(initialValue = 0, options = {}) {
  const { min = null, max = null } = options;

  const count = ref(initialValue);

  const isMin = computed(() => min !== null && count.value <= min);
  const isMax = computed(() => max !== null && count.value >= max);

  const increment = (amount = 1) => {
    if (max !== null && count.value + amount > max) {
      count.value = max;
    } else {
      count.value += amount;
    }
  };

  const decrement = (amount = 1) => {
    if (min !== null && count.value - amount < min) {
      count.value = min;
    } else {
      count.value -= amount;
    }
  };

  const reset = () => {
    count.value = initialValue;
  };

  return {
    count,
    isMin,
    isMax,
    increment,
    decrement,
    reset,
  };
}
```

```js
// tests/composables/useCounter.test.js
import { describe, it, expect, beforeEach } from "vitest";
import { useCounter } from "@/composables/useCounter";

describe("useCounter", () => {
  // Test basic functionality
  it("should initialize with the default value", () => {
    const { count } = useCounter();
    expect(count.value).toBe(0);
  });

  it("should initialize with the provided value", () => {
    const { count } = useCounter(10);
    expect(count.value).toBe(10);
  });

  // Test counter methods
  it("should increment the counter", () => {
    const { count, increment } = useCounter(5);
    increment();
    expect(count.value).toBe(6);
  });

  it("should increment by the specified amount", () => {
    const { count, increment } = useCounter(5);
    increment(3);
    expect(count.value).toBe(8);
  });

  it("should decrement the counter", () => {
    const { count, decrement } = useCounter(5);
    decrement();
    expect(count.value).toBe(4);
  });

  it("should decrement by the specified amount", () => {
    const { count, decrement } = useCounter(10);
    decrement(3);
    expect(count.value).toBe(7);
  });

  it("should reset the counter to the initial value", () => {
    const { count, increment, reset } = useCounter(5);
    increment(10);
    expect(count.value).toBe(15);
    reset();
    expect(count.value).toBe(5);
  });

  // Test min/max constraints
  describe("with min/max constraints", () => {
    it("should not go below the minimum value", () => {
      const { count, decrement, isMin } = useCounter(5, { min: 3 });

      decrement();
      expect(count.value).toBe(4);
      expect(isMin.value).toBe(false);

      decrement();
      expect(count.value).toBe(3);
      expect(isMin.value).toBe(true);

      decrement();
      expect(count.value).toBe(3); // Should not decrement further
      expect(isMin.value).toBe(true);
    });

    it("should not go above the maximum value", () => {
      const { count, increment, isMax } = useCounter(5, { max: 7 });

      increment();
      expect(count.value).toBe(6);
      expect(isMax.value).toBe(false);

      increment();
      expect(count.value).toBe(7);
      expect(isMax.value).toBe(true);

      increment();
      expect(count.value).toBe(7); // Should not increment further
      expect(isMax.value).toBe(true);
    });

    it("should handle large increments with max constraint", () => {
      const { count } = useCounter(5, { max: 10 });

      increment(20);
      expect(count.value).toBe(10); // Should cap at max
    });

    it("should handle large decrements with min constraint", () => {
      const { count } = useCounter(15, { min: 10 });

      decrement(20);
      expect(count.value).toBe(10); // Should cap at min
    });
  });
});
```

**Testing Async Composables:**

```js
// tests/composables/useAsyncData.test.js
import { describe, it, expect, vi, beforeEach } from "vitest";
import { nextTick } from "vue";
import { useAsyncData } from "@/composables/useAsyncData";

describe("useAsyncData", () => {
  // Mock API function
  const successResult = { id: 1, name: "Test" };
  const errorMessage = "API Error";

  let mockApiSuccess;
  let mockApiError;

  beforeEach(() => {
    // Reset mocks before each test
    mockApiSuccess = vi.fn().mockResolvedValue(successResult);
    mockApiError = vi.fn().mockRejectedValue(new Error(errorMessage));
  });

  // Test initial state
  it("should initialize with the correct state", () => {
    const { data, error, loading, loaded } = useAsyncData(mockApiSuccess, {
      immediate: false,
    });

    expect(data.value).toBe(null);
    expect(error.value).toBe(null);
    expect(loading.value).toBe(false);
    expect(loaded.value).toBe(false);
  });

  it("should initialize with the provided initial data", () => {
    const initialData = { test: true };
    const { data } = useAsyncData(mockApiSuccess, {
      immediate: false,
      initialData,
    });

    expect(data.value).toEqual(initialData);
  });

  // Test successful execution
  it("should load data when executed", async () => {
    const { data, error, loading, loaded, execute } = useAsyncData(
      mockApiSuccess,
      { immediate: false }
    );

    // Start execution
    const promise = execute();

    // Check loading state
    expect(loading.value).toBe(true);
    expect(loaded.value).toBe(false);

    // Wait for execution to complete
    await promise;

    // Check final state
    expect(loading.value).toBe(false);
    expect(error.value).toBe(null);
    expect(data.value).toEqual(successResult);
    expect(loaded.value).toBe(true);
    expect(mockApiSuccess).toHaveBeenCalledTimes(1);
  });

  // Test immediate execution
  it("should execute immediately when immediate option is true", async () => {
    const { loading } = useAsyncData(mockApiSuccess, { immediate: true });

    // Should be loading right away
    expect(loading.value).toBe(true);

    // Wait for async execution
    await nextTick();
    await nextTick();

    expect(loading.value).toBe(false);
    expect(mockApiSuccess).toHaveBeenCalledTimes(1);
  });

  // Test error handling
  it("should handle errors", async () => {
    const { data, error, loading, loaded, execute } = useAsyncData(
      mockApiError,
      { immediate: false }
    );

    // Start execution
    const promise = execute();

    // Check loading state
    expect(loading.value).toBe(true);

    // Wait for rejection and catch it
    try {
      await promise;
      // Should not reach here
      expect(true).toBe(false);
    } catch (err) {
      // Error should be thrown and also set in state
      expect(err.message).toBe(errorMessage);
    }

    // Check final state
    expect(loading.value).toBe(false);
    expect(error.value).toBeInstanceOf(Error);
    expect(error.value.message).toBe(errorMessage);
    expect(data.value).toBe(null); // Data should not be set
    expect(loaded.value).toBe(false);
  });

  // Test resetOnExecute option
  it("should reset data when resetOnExecute is true", async () => {
    const initialData = { initial: true };
    const { data, execute } = useAsyncData(mockApiSuccess, {
      immediate: false,
      initialData,
      resetOnExecute: true,
    });

    // Initial data should be set
    expect(data.value).toEqual(initialData);

    // Start execution
    const promise = execute();

    // Data should be reset to initialData during loading
    expect(data.value).toEqual(initialData);

    // Wait for execution to complete
    await promise;

    // Data should now be the result
    expect(data.value).toEqual(successResult);
  });

  // Test callbacks
  it("should call onSuccess callback on successful execution", async () => {
    const onSuccess = vi.fn();
    const { execute } = useAsyncData(mockApiSuccess, {
      immediate: false,
      onSuccess,
    });

    await execute();

    expect(onSuccess).toHaveBeenCalledWith(successResult);
  });

  it("should call onError callback on failed execution", async () => {
    const onError = vi.fn();
    const { execute } = useAsyncData(mockApiError, {
      immediate: false,
      onError,
    });

    try {
      await execute();
    } catch (err) {
      // Ignore the error
    }

    expect(onError).toHaveBeenCalled();
    expect(onError.mock.calls[0][0]).toBeInstanceOf(Error);
    expect(onError.mock.calls[0][0].message).toBe(errorMessage);
  });

  // Test execute with parameters
  it("should pass parameters to the async function", async () => {
    const mockApiWithParams = vi.fn().mockResolvedValue(successResult);
    const { execute } = useAsyncData(mockApiWithParams, { immediate: false });

    await execute("param1", 123);

    expect(mockApiWithParams).toHaveBeenCalledWith("param1", 123);
  });
});
```

**Testing Composables with Local Storage:**

```js
// tests/composables/useStorage.test.js
import { describe, it, expect, vi, beforeEach, afterEach } from "vitest";
import { useStorage } from "@/composables/useStorage";

describe("useStorage", () => {
  // Mock localStorage
  let localStorageMock;
  let getItemSpy;
  let setItemSpy;
  let removeItemSpy;

  beforeEach(() => {
    // Create localStorage mock
    getItemSpy = vi.spyOn(Storage.prototype, "getItem");
    setItemSpy = vi.spyOn(Storage.prototype, "setItem");
    removeItemSpy = vi.spyOn(Storage.prototype, "removeItem");

    // Clear all mocks before each test
    vi.clearAllMocks();
  });

  // Test localStorage functionality
  describe("localStorage", () => {
    it("should get initial value from localStorage", () => {
      const mockValue = JSON.stringify({ test: true });
      getItemSpy.mockReturnValueOnce(mockValue);

      const { value } = useStorage("testKey");

      expect(getItemSpy).toHaveBeenCalledWith("testKey");
      expect(value.value).toEqual({ test: true });
    });

    it("should use initial value if localStorage is empty", () => {
      const initialValue = { initial: true };
      getItemSpy.mockReturnValueOnce(null);

      const { value } = useStorage("testKey", initialValue);

      expect(getItemSpy).toHaveBeenCalledWith("testKey");
      expect(value.value).toEqual(initialValue);
    });

    it("should save value to localStorage when value changes", async () => {
      const { value } = useStorage("testKey", { initial: true });

      // Update the value
      value.value = { updated: true };

      // Wait for the watch effect to run
      await vi.runAllTimers();

      expect(setItemSpy).toHaveBeenCalledWith(
        "testKey",
        JSON.stringify({ updated: true })
      );
    });

    it("should remove item from localStorage when value is null", async () => {
      const { value } = useStorage("testKey", { initial: true });

      // Set value to null
      value.value = null;

      // Wait for the watch effect to run
      await vi.runAllTimers();

      expect(removeItemSpy).toHaveBeenCalledWith("testKey");
    });

    it("should handle errors when reading from localStorage", () => {
      // Mock localStorage.getItem to throw an error
      getItemSpy.mockImplementationOnce(() => {
        throw new Error("getItem error");
      });

      const initialValue = { initial: true };
      const { value } = useStorage("testKey", initialValue);

      // Should use initial value when localStorage throws an error
      expect(value.value).toEqual(initialValue);
    });

    it("should handle errors when writing to localStorage", async () => {
      // Mock localStorage.setItem to throw an error
      const consoleErrorSpy = vi
        .spyOn(console, "error")
        .mockImplementation(() => {});
      setItemSpy.mockImplementationOnce(() => {
        throw new Error("setItem error");
      });

      const { value } = useStorage("testKey", { initial: true });

      // Update the value
      value.value = { updated: true };

      // Wait for the watch effect to run
      await vi.runAllTimers();

      // Should log the error
      expect(consoleErrorSpy).toHaveBeenCalled();
      expect(consoleErrorSpy.mock.calls[0][0]).toContain(
        "Error writing testKey to storage"
      );

      // Restore console.error
      consoleErrorSpy.mockRestore();
    });
  });

  // Test sessionStorage functionality
  describe("sessionStorage", () => {
    let sessionStorageGetItemSpy;
    let sessionStorageSetItemSpy;

    beforeEach(() => {
      // Create sessionStorage spies
      sessionStorageGetItemSpy = vi.spyOn(sessionStorage, "getItem");
      sessionStorageSetItemSpy = vi.spyOn(sessionStorage, "setItem");

      // Clear all mocks
      vi.clearAllMocks();
    });

    it("should use sessionStorage when session option is true", () => {
      const mockValue = JSON.stringify({ test: true });
      sessionStorageGetItemSpy.mockReturnValueOnce(mockValue);

      const { value } = useStorage("testKey", null, { session: true });

      expect(sessionStorageGetItemSpy).toHaveBeenCalledWith("testKey");
      expect(value.value).toEqual({ test: true });
      expect(getItemSpy).not.toHaveBeenCalled(); // localStorage should not be used
    });

    it("should save to sessionStorage when session option is true", async () => {
      const { value } = useStorage(
        "testKey",
        { initial: true },
        { session: true }
      );

      // Update the value
      value.value = { updated: true };

      // Wait for the watch effect to run
      await vi.runAllTimers();

      expect(sessionStorageSetItemSpy).toHaveBeenCalledWith(
        "testKey",
        JSON.stringify({ updated: true })
      );
      expect(setItemSpy).not.toHaveBeenCalled(); // localStorage should not be used
    });
  });

  // Test removeItem method
  it("should provide a method to remove the item from storage", () => {
    const { removeItem } = useStorage("testKey", { initial: true });

    removeItem();

    expect(removeItemSpy).toHaveBeenCalledWith("testKey");
  });
});
```

**Testing Composables with Component Integration:**

```js
// tests/components/CounterComponent.test.js
import { describe, it, expect, vi } from "vitest";
import { mount } from "@vue/test-utils";
import CounterComponent from "@/components/CounterComponent.vue";
import { useCounter } from "@/composables/useCounter";

// Mock the useCounter composable
vi.mock("@/composables/useCounter", () => {
  // Create mock implementations
  const mockCount = ref(0);
  const mockIsMin = ref(false);
  const mockIsMax = ref(false);
  const mockIncrement = vi.fn(() => {
    mockCount.value++;
  });
  const mockDecrement = vi.fn(() => {
    mockCount.value--;
  });
  const mockReset = vi.fn(() => {
    mockCount.value = 0;
  });

  return {
    useCounter: vi.fn(() => ({
      count: mockCount,
      isMin: mockIsMin,
      isMax: mockIsMax,
      increment: mockIncrement,
      decrement: mockDecrement,
      reset: mockReset,
    })),
  };
});

describe("CounterComponent", () => {
  it("should render the counter value", () => {
    const wrapper = mount(CounterComponent);

    // Check if counter value is displayed
    expect(wrapper.find("h2").text()).toContain("Counter: 0");
  });

  it("should call increment when increase button is clicked", async () => {
    const wrapper = mount(CounterComponent);

    // Find and click the increase button
    await wrapper.find("button:nth-child(3)").trigger("click");

    // Check if increment was called
    expect(useCounter().increment).toHaveBeenCalled();
  });

  it("should call decrement when decrease button is clicked", async () => {
    const wrapper = mount(CounterComponent);

    // Find and click the decrease button
    await wrapper.find("button:nth-child(1)").trigger("click");

    // Check if decrement was called
    expect(useCounter().decrement).toHaveBeenCalled();
  });

  it("should call reset when reset button is clicked", async () => {
    const wrapper = mount(CounterComponent);

    // Find and click the reset button
    await wrapper.find("button:nth-child(2)").trigger("click");

    // Check if reset was called
    expect(useCounter().reset).toHaveBeenCalled();
  });

  it("should disable the decrease button when at minimum", () => {
    // Set isMin to true before mounting
    useCounter().isMin.value = true;

    const wrapper = mount(CounterComponent);

    // Check if decrease button is disabled
    expect(
      wrapper.find("button:nth-child(1)").attributes("disabled")
    ).toBeDefined();
  });

  it("should disable the increase button when at maximum", () => {
    // Set isMax to true before mounting
    useCounter().isMax.value = true;

    const wrapper = mount(CounterComponent);

    // Check if increase button is disabled
    expect(
      wrapper.find("button:nth-child(3)").attributes("disabled")
    ).toBeDefined();
  });
});
```

**Real-World Example:**
In an e-commerce application, composable tests for a `useShoppingCart` would verify that:

1. Items can be added, modified, and removed correctly
2. Price calculations, including tax and shipping, are accurate
3. Cart persists between sessions
4. Integration with payment process is validated
5. Error handling works as expected for out-of-stock scenarios

**Common Pitfalls:**

- Not mocking external dependencies (localStorage, fetch, etc.)
- Incomplete test coverage for error paths
- Not testing reactivity behavior
- Failing to test integration with components
- Testing implementation details instead of behavior

**Frequently Asked Questions:**

1. **Q: How do I test composables that use Vue lifecycle hooks?**

   A: For composables using lifecycle hooks, you need to test them while mounted in a component or by manually calling the lifecycle functions:

   ```js
   import { mount } from "@vue/test-utils";

   // Option 1: Create a wrapper component that uses the composable
   const TestComponent = {
     setup() {
       const result = useMyComposable();
       return { ...result };
     },
     template: "<div>Test</div>",
   };

   const wrapper = mount(TestComponent);

   // Option 2: Manually call lifecycle functions (for specific tests)
   import { onMounted, onUnmounted } from "vue";

   // Mock lifecycle hooks
   vi.mock("vue", async () => {
     const actual = await vi.importActual("vue");
     return {
       ...actual,
       onMounted: vi.fn((fn) => fn()),
       onUnmounted: vi.fn(),
     };
   });

   // Now you can test the composable directly
   const result = useMyComposable();

   // Verify that onMounted was called
   expect(onMounted).toHaveBeenCalled();
   ```

2. **Q: How do I test composables that use asynchronous operations?**

   A: Use async/await with Vitest's support for Promises:

   ```js
   it("should handle async operations", async () => {
     // For composables that return a promise
     const { data, execute } = useAsyncData(mockApi);

     // Initiate async operation
     const promise = execute();

     // Test initial loading state
     expect(loading.value).toBe(true);

     // Wait for the operation to complete
     await promise;

     // Test final state
     expect(loading.value).toBe(false);
     expect(data.value).toEqual(expectedResult);
   });
   ```

   For timing-based operations, use Vitest's timer mocks:

   ```js
   vi.useFakeTimers();

   const { result } = useDebounce(callback, 1000);

   // Fast-forward time
   vi.advanceTimersByTime(1000);

   // Check if callback was called
   expect(callback).toHaveBeenCalled();
   ```

3. **Q: How should I structure my composable tests?**

   A: Follow these principles:

   - Group tests logically by feature or behavior
   - Test both success and error paths
   - Test edge cases and boundaries
   - Isolate tests from external dependencies
   - Test integration with components when relevant

   A good structure:

   ```js
   describe("useFeature", () => {
     // Initial state
     describe("initial state", () => {
       it("should initialize with default values", () => {});
       it("should initialize with provided values", () => {});
     });

     // Core functionality
     describe("core functionality", () => {
       it("should perform the main operation", () => {});
       it("should handle different input types", () => {});
     });

     // Edge cases
     describe("edge cases", () => {
       it("should handle empty input", () => {});
       it("should handle maximum values", () => {});
     });

     // Error handling
     describe("error handling", () => {
       it("should catch and process errors", () => {});
       it("should recover from specific errors", () => {});
     });
   });
   ```

### 6. Ecosystem Composables

**Concise Explanation:**
Ecosystem composables are ready-made, reusable composables from the Vue community that solve common problems. They range from utility composables for everyday tasks to specialized ones for specific domains, saving development time and ensuring best practices.

**Where to Use:**

- Common UI patterns (modals, tooltips, etc.)
- Browser API interactions (storage, network, geolocation)
- Animation and transitions
- Form handling and validation
- State management
- Data fetching and caching

**Code Snippet:**

**VueUse Examples:**

```html
<template>
  <div class="p-6 bg-white rounded shadow">
    <h2 class="text-xl font-semibold mb-4">VueUse Examples</h2>

    <!-- Mouse position tracker -->
    <div class="mb-6 p-4 bg-gray-50 rounded">
      <h3 class="text-lg font-medium mb-2">Mouse Position</h3>
      <div
        class="h-40 border rounded relative cursor-none"
        @mousemove="handleMouseMove"
        @mouseleave="isMouseInside = false"
        @mouseenter="isMouseInside = true"
      >
        <div
          v-if="isMouseInside"
          class="absolute w-6 h-6 bg-blue-500 rounded-full opacity-50"
          :style="mouseStyle"
        ></div>
        <div class="absolute bottom-2 right-2 text-sm text-gray-500">
          x: {{ Math.round(x) }}, y: {{ Math.round(y) }}
        </div>
      </div>
    </div>

    <!-- Device motion detection -->
    <div class="mb-6 p-4 bg-gray-50 rounded">
      <h3 class="text-lg font-medium mb-2">Device Orientation</h3>
      <div v-if="deviceSupported" class="text-center">
        <div
          class="inline-block w-32 h-32 border-8 border-gray-300 rounded-lg shadow-inner"
          :style="deviceStyle"
        ></div>
        <p class="mt-2 text-sm text-gray-500">
          Tilt your device or use DevTools to simulate
        </p>
      </div>
      <div v-else class="text-center text-gray-500">
        Device orientation not supported or permission denied
      </div>
    </div>

    <!-- Idle timer -->
    <div class="mb-6 p-4 bg-gray-50 rounded">
      <h3 class="text-lg font-medium mb-2">Idle Timer</h3>
      <div
        :class="[
          'px-4 py-2 rounded text-white text-center',
          isIdle ? 'bg-red-500' : 'bg-green-500',
        ]"
      >
        <span v-if="isIdle"
          >You've been idle for {{ Math.floor(idleTime / 1000) }} seconds</span
        >
        <span v-else>Active</span>
      </div>
      <p class="mt-2 text-center text-sm text-gray-500">
        Idle timeout: {{ idleTimeout / 1000 }} seconds
      </p>
    </div>

    <!-- Dark mode toggle -->
    <div class="mb-6 p-4 bg-gray-50 rounded">
      <h3 class="text-lg font-medium mb-2">Dark Mode Toggle</h3>
      <div class="flex items-center justify-between">
        <span>Current mode: {{ isDark ? "Dark" : "Light" }}</span>
        <button
          @click="toggleDark()"
          class="px-4 py-2 rounded"
          :class="
            isDark ? 'bg-gray-800 text-white' : 'bg-yellow-400 text-gray-900'
          "
        >
          {{ isDark ? "Switch to Light" : "Switch to Dark" }}
        </button>
      </div>
    </div>

    <!-- Local storage -->
    <div class="mb-6 p-4 bg-gray-50 rounded">
      <h3 class="text-lg font-medium mb-2">Persistent Storage</h3>
      <div class="space-y-2">
        <input
          v-model="name"
          placeholder="Enter your name"
          class="w-full px-3 py-2 border rounded"
        />
        <div class="flex justify-between text-sm text-gray-600">
          <span>Name is saved automatically to localStorage</span>
          <button @click="name = ''" class="text-red-600 hover:underline">
            Clear
          </button>
        </div>
      </div>
    </div>

    <!-- Clipboard -->
    <div class="mb-6 p-4 bg-gray-50 rounded">
      <h3 class="text-lg font-medium mb-2">Clipboard</h3>
      <div class="flex space-x-2">
        <input
          v-model="clipboardText"
          placeholder="Text to copy"
          class="flex-1 px-3 py-2 border rounded"
        />
        <button
          @click="copy(clipboardText)"
          class="px-4 py-2 bg-blue-600 text-white rounded disabled:opacity-50"
          :disabled="!clipboardText"
        >
          {{ copied ? "Copied!" : "Copy" }}
        </button>
      </div>
      <div v-if="copied" class="mt-2 text-sm text-green-600">
        Text copied to clipboard!
      </div>
    </div>
  </div>
</template>

<script setup>
  import { ref, computed } from "vue";
  import { useMouse } from "@vueuse/core";
  import { useDeviceOrientation } from "@vueuse/core";
  import { useIdle } from "@vueuse/core";
  import { useDark, useToggle } from "@vueuse/core";
  import { useStorage } from "@vueuse/core";
  import { useClipboard } from "@vueuse/core";

  // Mouse position tracking
  const { x, y } = useMouse();
  const isMouseInside = ref(false);
  const mouseStyle = computed(() => ({
    transform: `translate(${x.value}px, ${y.value}px) translate(-50%, -50%)`,
  }));

  const handleMouseMove = (event) => {
    // The useMouse composable already handles this
  };

  // Device orientation
  const {
    isSupported: deviceSupported,
    alpha,
    beta,
    gamma,
  } = useDeviceOrientation();
  const deviceStyle = computed(() => ({
    transform: deviceSupported
      ? `rotateX(${beta.value || 0}deg) rotateY(${
          gamma.value || 0
        }deg) rotateZ(${alpha.value || 0}deg)`
      : "none",
  }));

  // Idle timer
  const idleTimeout = 5000; // 5 seconds
  const { idle: isIdle, lastActive: lastActiveTime } = useIdle(idleTimeout);
  const idleTime = computed(() => Date.now() - lastActiveTime.value);

  // Dark mode
  const isDark = useDark();
  const toggleDark = useToggle(isDark);

  // Persistent storage
  const name = useStorage("user-name", "");

  // Clipboard
  const { copy, copied, text: clipboardText } = useClipboard({ legacy: true });
</script>
```

**Form Validation with VeeValidate:**

```html
<template>
  <div class="p-6 bg-white rounded shadow">
    <h2 class="text-xl font-semibold mb-4">Contact Form</h2>

    <form @submit="onSubmit" class="space-y-4">
      <!-- Name Field -->
      <div>
        <label class="block mb-1 text-sm font-medium text-gray-700"
          >Full Name</label
        >
        <input
          v-model="name"
          type="text"
          name="name"
          class="w-full px-3 py-2 border rounded"
          :class="{ 'border-red-500': errors.name }"
        />
        <p v-if="errors.name" class="mt-1 text-sm text-red-600">
          {{ errors.name }}
        </p>
      </div>

      <!-- Email Field -->
      <div>
        <label class="block mb-1 text-sm font-medium text-gray-700"
          >Email</label
        >
        <input
          v-model="email"
          type="email"
          name="email"
          class="w-full px-3 py-2 border rounded"
          :class="{ 'border-red-500': errors.email }"
        />
        <p v-if="errors.email" class="mt-1 text-sm text-red-600">
          {{ errors.email }}
        </p>
      </div>

      <!-- Message Field -->
      <div>
        <label class="block mb-1 text-sm font-medium text-gray-700"
          >Message</label
        >
        <textarea
          v-model="message"
          name="message"
          rows="4"
          class="w-full px-3 py-2 border rounded"
          :class="{ 'border-red-500': errors.message }"
        ></textarea>
        <p v-if="errors.message" class="mt-1 text-sm text-red-600">
          {{ errors.message }}
        </p>
      </div>

      <!-- Agreement Checkbox -->
      <div>
        <label class="flex items-center">
          <input
            v-model="agreement"
            type="checkbox"
            name="agreement"
            class="h-4 w-4 text-blue-600 rounded"
          />
          <span class="ml-2 text-sm text-gray-700">
            I agree to the terms and conditions
          </span>
        </label>
        <p v-if="errors.agreement" class="mt-1 text-sm text-red-600">
          {{ errors.agreement }}
        </p>
      </div>

      <!-- Submit Button -->
      <button
        type="submit"
        :disabled="isSubmitting || !meta.valid"
        class="w-full px-4 py-2 bg-blue-600 text-white rounded disabled:opacity-50"
      >
        <span v-if="isSubmitting">
          <span
            class="animate-spin inline-block h-4 w-4 border-2 border-white border-t-transparent rounded-full mr-1"
          ></span>
          Submitting...
        </span>
        <span v-else>Submit</span>
      </button>

      <!-- Form Status -->
      <div
        v-if="submitStatus"
        :class="[
          'p-3 rounded',
          submitStatus.type === 'success'
            ? 'bg-green-100 text-green-800'
            : 'bg-red-100 text-red-800',
        ]"
      >
        {{ submitStatus.message }}
      </div>
    </form>
  </div>
</template>

<script setup>
  import { ref, reactive } from "vue";
  import { useForm, useField } from "vee-validate";
  import * as yup from "yup";

  // Define validation schema
  const schema = yup.object({
    name: yup.string().required("Name is required").min(2, "Name is too short"),
    email: yup
      .string()
      .required("Email is required")
      .email("Email is not valid"),
    message: yup
      .string()
      .required("Message is required")
      .min(10, "Message is too short"),
    agreement: yup.boolean().oneOf([true], "You must agree to the terms"),
  });

  // Use VeeValidate's useForm
  const { handleSubmit, errors, meta, isSubmitting } = useForm({
    validationSchema: schema,
    initialValues: {
      name: "",
      email: "",
      message: "",
      agreement: false,
    },
  });

  // Use fields
  const { value: name } = useField("name");
  const { value: email } = useField("email");
  const { value: message } = useField("message");
  const { value: agreement } = useField("agreement");

  // Form submission status
  const submitStatus = ref(null);

  // Submit handler
  const onSubmit = handleSubmit(async (values) => {
    try {
      // Reset status
      submitStatus.value = null;

      // Simulate API call
      await new Promise((resolve) => setTimeout(resolve, 1500));

      // Show success message
      submitStatus.value = {
        type: "success",
        message: "Form submitted successfully! We will get back to you soon.",
      };

      // In a real app, you would submit to an API here
      console.log("Form submitted:", values);
    } catch (error) {
      // Show error message
      submitStatus.value = {
        type: "error",
        message: `Failed to submit form: ${error.message || "Unknown error"}`,
      };
    }
  });
</script>
```

**Data Fetching with Vue Query:**

```html
<template>
  <div class="p-6 bg-white rounded shadow">
    <h2 class="text-xl font-semibold mb-4">User Directory</h2>

    <!-- Search and filters -->
    <div class="mb-4 flex space-x-2">
      <input
        v-model="searchQuery"
        placeholder="Search users..."
        class="flex-1 px-3 py-2 border rounded"
      />
      <button
        @click="refetch"
        :disabled="isRefetching"
        class="px-4 py-2 bg-blue-600 text-white rounded disabled:opacity-50"
      >
        <span v-if="isRefetching">
          <span
            class="animate-spin inline-block h-4 w-4 border-2 border-white border-t-transparent rounded-full mr-1"
          ></span>
          Refreshing...
        </span>
        <span v-else>Refresh</span>
      </button>
    </div>

    <!-- Loading state -->
    <div v-if="isLoading" class="py-12 flex justify-center">
      <div
        class="animate-spin h-12 w-12 border-4 border-blue-500 border-t-transparent rounded-full"
      ></div>
    </div>

    <!-- Error state -->
    <div v-else-if="isError" class="p-4 bg-red-100 text-red-800 rounded">
      <p class="font-medium">Error loading users:</p>
      <p>{{ error.message }}</p>
      <button
        @click="refetch"
        class="mt-2 px-3 py-1 bg-red-600 text-white text-sm rounded"
      >
        Try Again
      </button>
    </div>

    <!-- Empty state -->
    <div
      v-else-if="!filteredUsers.length"
      class="p-12 text-center text-gray-500"
    >
      <p class="text-lg">No users found</p>
      <p class="text-sm">Try adjusting your search or filters</p>
    </div>

    <!-- User list -->
    <div v-else class="space-y-4">
      <div
        v-for="user in filteredUsers"
        :key="user.id"
        class="p-4 border rounded hover:bg-gray-50"
      >
        <div class="flex items-center space-x-4">
          <img
            :src="user.avatar"
            :alt="user.name"
            class="w-12 h-12 rounded-full object-cover"
          />
          <div>
            <h3 class="font-medium">{{ user.name }}</h3>
            <p class="text-sm text-gray-600">{{ user.email }}</p>
          </div>
          <div class="ml-auto">
            <button
              @click="loadUserDetails(user.id)"
              class="px-3 py-1 bg-gray-200 text-gray-800 rounded hover:bg-gray-300"
            >
              Details
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Pagination -->
    <div v-if="data?.total > 0" class="mt-4 flex justify-between items-center">
      <span class="text-sm text-gray-600">
        Showing {{ filteredUsers.length }} of {{ data.total }} users
      </span>
      <div class="space-x-2">
        <button
          @click="page = Math.max(1, page - 1)"
          :disabled="page === 1"
          class="px-3 py-1 bg-gray-200 text-gray-800 rounded disabled:opacity-50"
        >
          Previous
        </button>
        <button
          @click="page = page + 1"
          :disabled="page * pageSize >= data.total"
          class="px-3 py-1 bg-gray-200 text-gray-800 rounded disabled:opacity-50"
        >
          Next
        </button>
      </div>
    </div>

    <!-- User details modal -->
    <div
      v-if="userDetailsOpen"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50"
    >
      <div class="bg-white rounded shadow-lg max-w-md w-full p-6">
        <div class="flex justify-between items-center mb-4">
          <h3 class="text-lg font-semibold">User Details</h3>
          <button
            @click="userDetailsOpen = false"
            class="text-gray-400 hover:text-gray-600"
          >
            <svg
              class="w-6 h-6"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M6 18L18 6M6 6l12 12"
              />
            </svg>
          </button>
        </div>

        <!-- User details content -->
        <div v-if="userQuery.isLoading" class="py-8 flex justify-center">
          <div
            class="animate-spin h-8 w-8 border-4 border-blue-500 border-t-transparent rounded-full"
          ></div>
        </div>

        <div
          v-else-if="userQuery.isError"
          class="p-4 bg-red-100 text-red-800 rounded"
        >
          <p>Failed to load user details: {{ userQuery.error.message }}</p>
        </div>

        <div v-else-if="userQuery.data" class="space-y-4">
          <div class="flex items-center space-x-4">
            <img
              :src="userQuery.data.avatar"
              :alt="userQuery.data.name"
              class="w-20 h-20 rounded-full object-cover"
            />
            <div>
              <h4 class="text-xl font-medium">{{ userQuery.data.name }}</h4>
              <p class="text-gray-600">{{ userQuery.data.email }}</p>
              <p class="text-sm text-gray-500">
                {{ userQuery.data.role || "User" }}
              </p>
            </div>
          </div>

          <div class="border-t pt-4">
            <h5 class="font-medium mb-2">Contact Information</h5>
            <p>
              <span class="text-gray-600">Phone:</span>
              {{ userQuery.data.phone || "N/A" }}
            </p>
            <p>
              <span class="text-gray-600">Location:</span>
              {{ userQuery.data.location || "N/A" }}
            </p>
          </div>

          <div class="border-t pt-4">
            <h5 class="font-medium mb-2">Bio</h5>
            <p class="text-gray-700">
              {{ userQuery.data.bio || "No bio available" }}
            </p>
          </div>
        </div>

        <div class="mt-6 flex justify-end">
          <button
            @click="userDetailsOpen = false"
            class="px-4 py-2 bg-gray-200 text-gray-800 rounded hover:bg-gray-300"
          >
            Close
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
  import { ref, computed, watch } from "vue";
  import { useQuery } from "@tanstack/vue-query";

  // Pagination state
  const page = ref(1);
  const pageSize = 10;
  const searchQuery = ref("");

  // User details modal
  const userDetailsOpen = ref(false);
  const selectedUserId = ref(null);

  // Fetch users with pagination
  const fetchUsers = async ({ queryKey }) => {
    const [_, currentPage] = queryKey;

    // Simulate API call
    await new Promise((resolve) => setTimeout(resolve, 1000));

    // Mock response
    return {
      users: Array.from({ length: pageSize }, (_, i) => ({
        id: (currentPage - 1) * pageSize + i + 1,
        name: `User ${(currentPage - 1) * pageSize + i + 1}`,
        email: `user${(currentPage - 1) * pageSize + i + 1}@example.com`,
        avatar: `https://i.pravatar.cc/150?u=user${
          (currentPage - 1) * pageSize + i + 1
        }`,
      })),
      total: 100,
      page: currentPage,
      pageSize,
    };
  };

  // Fetch single user
  const fetchUserById = async ({ queryKey }) => {
    const [_, userId] = queryKey;

    if (!userId) return null;

    // Simulate API call
    await new Promise((resolve) => setTimeout(resolve, 800));

    // Simulate error for some IDs
    if (userId === 4) {
      throw new Error("User not found");
    }

    // Mock response
    return {
      id: userId,
      name: `User ${userId}`,
      email: `user${userId}@example.com`,
      avatar: `https://i.pravatar.cc/150?u=user${userId}`,
      phone: `+1 (555) ${100 + userId}-${1000 + userId}`,
      location: ["New York", "San Francisco", "Seattle", "Austin", "Chicago"][
        userId % 5
      ],
      role: ["User", "Admin", "Editor", "Viewer"][userId % 4],
      bio: "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nullam euismod, nisl eget aliquam ultricies, nunc nisl aliquet nunc, quis aliquam nisl nunc eu nisl.",
    };
  };

  // Use Vue Query for data fetching
  const { isLoading, isError, error, data, refetch, isRefetching } = useQuery(
    ["users", page],
    fetchUsers,
    {
      keepPreviousData: true,
      staleTime: 5 * 60 * 1000, // 5 minutes
    }
  );

  // Filter users based on search query
  const filteredUsers = computed(() => {
    if (!data?.users) return [];

    if (!searchQuery.value) return data.users;

    const query = searchQuery.value.toLowerCase();
    return data.users.filter(
      (user) =>
        user.name.toLowerCase().includes(query) ||
        user.email.toLowerCase().includes(query)
    );
  });

  // User details query
  const userQuery = useQuery(["user", selectedUserId], fetchUserById, {
    enabled: !!selectedUserId,
    staleTime: 10 * 60 * 1000, // 10 minutes
  });

  // Load user details
  const loadUserDetails = (userId) => {
    selectedUserId.value = userId;
    userDetailsOpen.value = true;
  };

  // Reset page when search query changes
  watch(searchQuery, () => {
    page.value = 1;
  });
</script>
```

**Animation with Motion One:**

```html
<template>
  <div class="p-6 bg-white rounded shadow">
    <h2 class="text-xl font-semibold mb-4">Animation Examples</h2>

    <!-- Simple animation -->
    <div class="mb-8">
      <h3 class="text-lg font-medium mb-2">Simple Animation</h3>
      <div class="flex space-x-4 mb-4">
        <button
          @click="animateBox"
          class="px-4 py-2 bg-blue-600 text-white rounded"
        >
          Animate
        </button>
        <button
          @click="resetBox"
          class="px-4 py-2 bg-gray-200 text-gray-800 rounded"
        >
          Reset
        </button>
      </div>

      <div class="h-32 flex items-center">
        <div ref="box" class="w-20 h-20 bg-blue-500 rounded"></div>
      </div>
    </div>

    <!-- Animation sequence -->
    <div class="mb-8">
      <h3 class="text-lg font-medium mb-2">Animation Sequence</h3>
      <button
        @click="animateSequence"
        class="px-4 py-2 bg-purple-600 text-white rounded mb-4"
      >
        Play Sequence
      </button>

      <div class="h-32 flex items-center space-x-4">
        <div
          v-for="(_, index) in 5"
          :key="index"
          :ref="
            (el) => {
              if (el) sequenceBoxes[index] = el;
            }
          "
          class="w-10 h-10 bg-purple-500 rounded"
        ></div>
      </div>
    </div>

    <!-- Scroll animations -->
    <div>
      <h3 class="text-lg font-medium mb-2">Scroll Animations</h3>
      <p class="mb-4">Scroll down to see animations</p>

      <div class="space-y-8 h-80 overflow-auto p-4 border rounded">
        <div
          v-for="index in 5"
          :key="index"
          :ref="
            (el) => {
              if (el) scrollItems[index - 1] = el;
            }
          "
          class="h-32 w-full bg-gradient-to-r from-blue-500 to-purple-500 rounded flex items-center justify-center text-white text-xl font-bold"
        >
          Item {{ index }}
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
  import { ref, onMounted } from "vue";
  import { animate, stagger, inView } from "motion";

  // Element refs
  const box = ref(null);
  const sequenceBoxes = ref([]);
  const scrollItems = ref([]);

  // Simple animation
  const animateBox = () => {
    animate(
      box.value,
      {
        x: "100%",
        backgroundColor: "#F59E0B",
        rotate: 180,
        scale: 1.2,
      },
      {
        duration: 1,
        easing: "ease-in-out",
      }
    );
  };

  const resetBox = () => {
    animate(
      box.value,
      {
        x: 0,
        backgroundColor: "#3B82F6",
        rotate: 0,
        scale: 1,
      },
      {
        duration: 0.5,
      }
    );
  };

  // Animation sequence
  const animateSequence = () => {
    animate(
      sequenceBoxes.value,
      {
        y: [-20, 0],
        opacity: [0, 1],
        scale: [0.8, 1],
      },
      {
        duration: 0.5,
        delay: stagger(0.1),
        easing: "ease-out",
      }
    );
  };

  // Set up scroll animations
  onMounted(() => {
    // Reset sequence boxes for initial state
    sequenceBoxes.value.forEach((box) => {
      box.style.opacity = 1;
      box.style.transform = "none";
    });

    // Set up scroll animations
    scrollItems.value.forEach((item) => {
      inView(
        item,
        () => {
          animate(
            item,
            {
              opacity: [0, 1],
              y: [50, 0],
            },
            {
              duration: 0.6,
              easing: "ease-out",
            }
          );
        },
        { margin: "-20% 0px -20% 0px" }
      );
    });
  });
</script>
```

**State Management with Pinia:**

```html
<template>
  <div class="p-6 bg-white rounded shadow">
    <h2 class="text-xl font-semibold mb-4">Todo App with Pinia</h2>

    <!-- Add todo form -->
    <form @submit.prevent="addTodo" class="mb-6 flex">
      <input
        v-model="newTodo"
        placeholder="Add a new todo..."
        class="flex-1 px-4 py-2 border rounded-l focus:outline-none focus:ring-2 focus:ring-blue-500"
      />
      <button
        type="submit"
        class="px-4 py-2 bg-blue-600 text-white rounded-r"
        :disabled="!newTodo.trim()"
      >
        Add
      </button>
    </form>

    <!-- Filters -->
    <div class="mb-4 flex space-x-2">
      <button
        v-for="filter in filters"
        :key="filter.value"
        @click="todoStore.filter = filter.value"
        :class="[
          'px-3 py-1 rounded text-sm',
          todoStore.filter === filter.value
            ? 'bg-blue-600 text-white'
            : 'bg-gray-200 text-gray-800',
        ]"
      >
        {{ filter.label }}
      </button>
    </div>

    <!-- Todo List -->
    <div class="space-y-2">
      <div
        v-if="todoStore.filteredTodos.length === 0"
        class="p-4 text-center text-gray-500"
      >
        No todos to display
      </div>

      <div
        v-for="todo in todoStore.filteredTodos"
        :key="todo.id"
        :class="[
          'p-3 border rounded flex items-center',
          todo.completed ? 'bg-gray-50' : 'bg-white',
        ]"
      >
        <!-- Checkbox -->
        <input
          type="checkbox"
          :checked="todo.completed"
          @change="toggleTodo(todo.id)"
          class="h-5 w-5 text-blue-600 rounded"
        />

        <!-- Todo text -->
        <span
          :class="[
            'ml-3 flex-1',
            todo.completed ? 'line-through text-gray-500' : 'text-gray-900',
          ]"
        >
          {{ todo.text }}
        </span>

        <!-- Actions -->
        <button
          @click="deleteTodo(todo.id)"
          class="ml-2 text-red-500 hover:text-red-700"
        >
          <svg
            class="h-5 w-5"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"
            />
          </svg>
        </button>
      </div>
    </div>

    <!-- Stats -->
    <div class="mt-4 flex justify-between text-sm text-gray-600">
      <span>{{ todoStore.totalCount }} items</span>
      <span>{{ todoStore.completedCount }} completed</span>
      <button
        v-if="todoStore.completedCount > 0"
        @click="todoStore.clearCompleted()"
        class="text-blue-600 hover:underline"
      >
        Clear completed
      </button>
    </div>

    <!-- Persistence controls -->
    <div class="mt-6 border-t pt-4">
      <div class="flex justify-between">
        <span class="text-sm text-gray-600">
          Todos are saved to localStorage automatically
        </span>
        <button
          @click="todoStore.$reset()"
          class="text-red-600 hover:underline text-sm"
        >
          Reset to default
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
  import { ref } from "vue";
  import { useTodoStore } from "@/stores/todo";

  // Todo store from Pinia
  const todoStore = useTodoStore();

  // New todo input
  const newTodo = ref("");

  // Filter options
  const filters = [
    { label: "All", value: "all" },
    { label: "Active", value: "active" },
    { label: "Completed", value: "completed" },
  ];

  // Add a new todo
  const addTodo = () => {
    if (newTodo.value.trim()) {
      todoStore.addTodo(newTodo.value.trim());
      newTodo.value = "";
    }
  };

  // Toggle todo completion status
  const toggleTodo = (id) => {
    todoStore.toggleTodo(id);
  };

  // Delete a todo
  const deleteTodo = (id) => {
    todoStore.deleteTodo(id);
  };
</script>
```

```js
// stores/todo.js - Pinia store example
import { defineStore } from "pinia";
import { useStorage } from "@vueuse/core";

export const useTodoStore = defineStore("todo", () => {
  // State - using VueUse's useStorage for persistence
  const todos = useStorage("pinia-todos", [
    { id: 1, text: "Learn Vue 3", completed: true },
    { id: 2, text: "Master Composition API", completed: false },
    { id: 3, text: "Build a project with Pinia", completed: false },
  ]);

  const filter = ref("all");
  let nextId = computed(() => {
    return todos.value.length > 0
      ? Math.max(...todos.value.map((t) => t.id)) + 1
      : 1;
  });

  // Getters
  const filteredTodos = computed(() => {
    switch (filter.value) {
      case "completed":
        return todos.value.filter((todo) => todo.completed);
      case "active":
        return todos.value.filter((todo) => !todo.completed);
      default:
        return todos.value;
    }
  });

  const totalCount = computed(() => todos.value.length);

  const completedCount = computed(
    () => todos.value.filter((todo) => todo.completed).length
  );

  const activeCount = computed(
    () => todos.value.filter((todo) => !todo.completed).length
  );

  // Actions
  function addTodo(text) {
    todos.value.push({
      id: nextId.value,
      text,
      completed: false,
    });
  }

  function toggleTodo(id) {
    const todo = todos.value.find((todo) => todo.id === id);
    if (todo) {
      todo.completed = !todo.completed;
    }
  }

  function deleteTodo(id) {
    const index = todos.value.findIndex((todo) => todo.id === id);
    if (index !== -1) {
      todos.value.splice(index, 1);
    }
  }

  function clearCompleted() {
    todos.value = todos.value.filter((todo) => !todo.completed);
  }

  return {
    // State
    todos,
    filter,

    // Getters
    filteredTodos,
    totalCount,
    completedCount,
    activeCount,

    // Actions
    addTodo,
    toggleTodo,
    deleteTodo,
    clearCompleted,
  };
});
```

**Routing with Vue Router:**

```html
<template>
  <div class="p-6 bg-white rounded shadow">
    <h2 class="text-xl font-semibold mb-4">Vue Router Composable Example</h2>

    <!-- Navigation -->
    <nav class="mb-6 flex space-x-4 border-b pb-2">
      <RouterLink
        v-for="route in routes"
        :key="route.path"
        :to="route.path"
        class="px-4 py-2 rounded"
        :class="
          route.path === currentRoute.path
            ? 'bg-blue-600 text-white'
            : 'text-gray-700 hover:bg-gray-100'
        "
      >
        {{ route.name }}
      </RouterLink>
    </nav>

    <!-- Route info -->
    <div class="mb-6 p-4 bg-gray-50 rounded">
      <h3 class="text-lg font-medium mb-2">Current Route</h3>
      <div class="space-y-1 text-sm">
        <p><strong>Path:</strong> {{ currentRoute.path }}</p>
        <p><strong>Name:</strong> {{ currentRoute.name }}</p>
        <p>
          <strong>Params:</strong> {{ JSON.stringify(currentRoute.params) }}
        </p>
        <p><strong>Query:</strong> {{ JSON.stringify(currentRoute.query) }}</p>
      </div>
    </div>

    <!-- Dynamic navigation -->
    <div class="mb-6">
      <h3 class="text-lg font-medium mb-2">Dynamic Navigation</h3>
      <div class="space-y-3">
        <div>
          <label class="block mb-1 text-sm text-gray-700">User ID</label>
          <div class="flex">
            <input
              v-model="userId"
              type="number"
              min="1"
              class="flex-1 px-3 py-2 border rounded-l"
            />
            <button
              @click="navigateToUser"
              class="px-4 py-2 bg-blue-600 text-white rounded-r"
            >
              Go to User
            </button>
          </div>
        </div>

        <div>
          <label class="block mb-1 text-sm text-gray-700">Search Query</label>
          <div class="flex">
            <input
              v-model="searchQuery"
              class="flex-1 px-3 py-2 border rounded-l"
            />
            <button
              @click="searchProducts"
              class="px-4 py-2 bg-blue-600 text-white rounded-r"
            >
              Search
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Navigation history -->
    <div>
      <h3 class="text-lg font-medium mb-2">Navigation History</h3>
      <div class="flex space-x-2 mb-4">
        <button
          @click="goBack"
          class="px-4 py-2 bg-gray-200 text-gray-800 rounded"
          :disabled="!canGoBack"
        >
          Go Back
        </button>
        <button
          @click="goForward"
          class="px-4 py-2 bg-gray-200 text-gray-800 rounded"
          :disabled="!canGoForward"
        >
          Go Forward
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
  import { ref } from "vue";
  import { useRouter, useRoute } from "vue-router";

  // Get router and current route
  const router = useRouter();
  const currentRoute = useRoute();

  // Dynamic navigation state
  const userId = ref(1);
  const searchQuery = ref("");

  // Example routes for display
  const routes = [
    { path: "/", name: "Home" },
    { path: "/about", name: "About" },
    { path: "/products", name: "Products" },
    { path: "/contact", name: "Contact" },
  ];

  // Navigation methods
  const navigateToUser = () => {
    router.push({
      name: "UserProfile",
      params: { id: userId.value },
    });
  };

  const searchProducts = () => {
    router.push({
      path: "/products",
      query: { q: searchQuery.value },
    });
  };

  // History navigation
  const goBack = () => {
    router.back();
  };

  const goForward = () => {
    router.forward();
  };

  // Navigation state
  const canGoBack = ref(window.history.state?.back !== undefined);
  const canGoForward = ref(window.history.state?.forward !== undefined);

  // Listen for route changes
  router.afterEach(() => {
    canGoBack.value = window.history.state?.back !== undefined;
    canGoForward.value = window.history.state?.forward !== undefined;
  });
</script>
```

**Custom Composable with VueUse Integration:**

```js
// composables/useInfiniteScroll.js - Custom composable using VueUse
import { ref, computed, watch } from "vue";
import { useIntersectionObserver } from "@vueuse/core";
import { useScroll } from "@vueuse/core";

export function useInfiniteScroll(options = {}) {
  // Options with defaults
  const {
    initialPage = 1,
    pageSize = 10,
    threshold = 0.5,
    apiCallback,
    immediate = true,
  } = options;

  // State
  const items = ref([]);
  const page = ref(initialPage);
  const isLoading = ref(false);
  const error = ref(null);
  const hasMore = ref(true);

  // Sentinel element for intersection observer
  const sentinel = ref(null);

  // Computed
  const isEmpty = computed(() => items.value.length === 0);
  const total = computed(() => items.value.length);

  // Load data
  const loadMore = async () => {
    if (isLoading.value || !hasMore.value) return;

    isLoading.value = true;
    error.value = null;

    try {
      // If no callback provided, warn and return
      if (!apiCallback) {
        console.warn("No API callback provided to useInfiniteScroll");
        return;
      }

      // Call API
      const result = await apiCallback({
        page: page.value,
        pageSize,
      });

      // Process results
      const newItems = result.data || result.items || result;

      if (newItems && newItems.length > 0) {
        // Add new items to the list
        items.value = [...items.value, ...newItems];

        // Increment page
        page.value++;

        // Check if more items available
        if (result.hasMore !== undefined) {
          hasMore.value = result.hasMore;
        } else if (newItems.length < pageSize) {
          hasMore.value = false;
        }
      } else {
        hasMore.value = false;
      }
    } catch (err) {
      error.value = err;
      console.error("Error loading more items:", err);
    } finally {
      isLoading.value = false;
    }
  };

  // Reset everything
  const reset = () => {
    items.value = [];
    page.value = initialPage;
    isLoading.value = false;
    error.value = null;
    hasMore.value = true;

    // Load initial data if immediate is true
    if (immediate && apiCallback) {
      loadMore();
    }
  };

  // Set up intersection observer for infinite scroll
  const { stop: stopObserver } = useIntersectionObserver(
    sentinel,
    ([entry]) => {
      if (entry.isIntersecting && !isLoading.value && hasMore.value) {
        loadMore();
      }
    },
    { threshold }
  );

  // Cleanup function for the component
  const cleanup = () => {
    stopObserver();
  };

  // Load initial data if immediate is true
  if (immediate && apiCallback) {
    loadMore();
  }

  return {
    // State
    items,
    isLoading,
    error,
    hasMore,
    isEmpty,
    total,
    page,
    sentinel,

    // Methods
    loadMore,
    reset,
    cleanup,
  };
}
```

```html
<!-- Using the custom infinite scroll composable -->
<template>
  <div class="p-6 bg-white rounded shadow">
    <h2 class="text-xl font-semibold mb-4">Infinite Scroll Products</h2>

    <!-- Search -->
    <div class="mb-4">
      <div class="flex">
        <input
          v-model="searchQuery"
          placeholder="Search products..."
          class="flex-1 px-4 py-2 border rounded-l focus:outline-none focus:ring-2 focus:ring-blue-500"
          @keyup.enter="handleSearch"
        />
        <button
          @click="handleSearch"
          class="px-4 py-2 bg-blue-600 text-white rounded-r"
        >
          Search
        </button>
      </div>
    </div>

    <!-- Product grid -->
    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
      <div
        v-for="product in products.items"
        :key="product.id"
        class="border rounded overflow-hidden hover:shadow-md transition-shadow"
      >
        <img
          :src="product.image"
          :alt="product.name"
          class="w-full h-48 object-cover object-center"
        />
        <div class="p-4">
          <h3 class="font-medium text-gray-900">{{ product.name }}</h3>
          <p class="text-gray-600 text-sm mb-2">{{ product.category }}</p>
          <div class="flex justify-between items-center">
            <span class="font-bold">${{ product.price.toFixed(2) }}</span>
            <button class="px-3 py-1 bg-blue-600 text-white rounded text-sm">
              Add to Cart
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Loading state -->
    <div v-if="products.isLoading" class="py-8 flex justify-center">
      <div
        class="animate-spin h-10 w-10 border-4 border-blue-500 border-t-transparent rounded-full"
      ></div>
    </div>

    <!-- Error state -->
    <div
      v-else-if="products.error"
      class="p-4 bg-red-100 text-red-800 rounded my-4"
    >
      <p class="font-medium">Error loading products:</p>
      <p>{{ products.error.message }}</p>
      <button
        @click="products.loadMore"
        class="mt-2 px-3 py-1 bg-red-600 text-white text-sm rounded"
      >
        Try Again
      </button>
    </div>

    <!-- Empty state -->
    <div v-else-if="products.isEmpty" class="py-8 text-center text-gray-500">
      <p class="text-lg">No products found</p>
      <p class="text-sm">Try a different search query</p>
    </div>

    <!-- Sentinel element for infinite scroll -->
    <div ref="products.sentinel" class="h-4 w-full my-4"></div>

    <!-- End of results message -->
    <div
      v-if="!products.hasMore && !products.isEmpty"
      class="py-4 text-center text-gray-500"
    >
      No more products to load
    </div>
  </div>
</template>

<script setup>
  import { ref } from "vue";
  import { useInfiniteScroll } from "@/composables/useInfiniteScroll";

  // Search state
  const searchQuery = ref("");

  // Mock product API function
  const fetchProducts = async ({ page, pageSize }) => {
    // Simulate API delay
    await new Promise((resolve) => setTimeout(resolve, 1000));

    // Simulate server response
    const totalItems = 50;
    const startIndex = (page - 1) * pageSize;
    const hasMore = startIndex + pageSize < totalItems;

    // Generate mock products
    const items = Array.from(
      { length: Math.min(pageSize, totalItems - startIndex) },
      (_, i) => {
        const id = startIndex + i + 1;
        return {
          id,
          name: `Product ${id}`,
          category: ["Electronics", "Clothing", "Home", "Books", "Sports"][
            id % 5
          ],
          price: 10 + (id % 90),
          image: `https://picsum.photos/seed/product${id}/300/200`,
        };
      }
    );

    return {
      items,
      hasMore,
    };
  };

  // Use infinite scroll composable
  const products = useInfiniteScroll({
    apiCallback: fetchProducts,
    pageSize: 9,
    immediate: true,
  });

  // Handle search
  const handleSearch = () => {
    // Reset and reload with new search term
    products.reset();

    // In a real app, you would pass the search query to the API callback
    // and modify the fetchProducts function to use it
  };
</script>
```

**Real-World Example:**
In a modern media platform, ecosystem composables are layered together to create a complete feature. A team might combine:

1. VueUse's `useMediaControls` for video playback
2. TanStack Query for API data fetching with caching
3. Pinia for global state (user preferences, watch history)
4. A custom `useAnalytics` composable for tracking user engagement
5. VeeValidate for comment forms

This approach allows for modular, testable development while leveraging community solutions for common problems.

**Common Pitfalls:**

- Inadequate error handling when using third-party composables
- Version conflicts between different ecosystem libraries
- Over-reliance on ecosystem composables for simple tasks
- Not understanding the internal implementation of ecosystem composables
- Missing cleanup when composables set up side effects

**Frequently Asked Questions:**

1. **Q: How do I choose between writing my own composable vs. using an ecosystem one?**

   A: Consider these factors:

   - **Complexity**: For complex features with edge cases and maintainability concerns, ecosystem composables from established libraries are often better.
   - **Specificity**: For very application-specific logic, writing your own is usually preferable.
   - **Maintenance**: Ecosystem composables are maintained and improved by the community.
   - **Bundle size**: Evaluate if adding a dependency for a simple feature is worth the extra code.
   - **Learning curve**: Consider if your team already knows the ecosystem library.

   When in doubt, start with ecosystem composables for common problems and only build custom ones for specific needs.

2. **Q: How do I combine multiple ecosystem composables effectively?**

   A: Follow these best practices:

   - Create wrapper composables that integrate multiple ecosystem composables with your app's specific logic
   - Use composition to build higher-level abstractions from smaller building blocks
   - Handle cleanup properly by calling any cleanup functions in `onUnmounted`
   - Ensure consistent error handling across different composables
   - Document dependencies and integration points clearly

   Example:

   ```js
   // A wrapper composable that combines multiple ecosystem composables
   import { useStorage } from "@vueuse/core";
   import { useQuery } from "@tanstack/vue-query";

   export function useUserPreferences() {
     const storage = useStorage("user-prefs", { theme: "light" });

     // Use Vue Query for fetching remote preferences
     const { data, error, isLoading } = useQuery(
       ["user-preferences"],
       fetchUserPreferences,
       {
         onSuccess: (data) => {
           // Merge remote preferences with local ones
           Object.assign(storage.value, data);
         },
       }
     );

     return {
       preferences: storage,
       error,
       isLoading,
     };
   }
   ```

3. **Q: What are the most useful ecosystem composables to learn first?**

   A: Start with these foundational libraries:

   - **VueUse**: A collection of essential Vue Composition API utilities

     - `useStorage`: For persistent data
     - `useFetch`: For basic API requests
     - `useMediaQuery`: For responsive design
     - `useDark`: For dark mode

   - **Pinia**: For state management

     - Replacing Vuex with a more composition-friendly approach

   - **TanStack Query (Vue Query)**: For advanced data fetching

     - Caching, refetching, and invalidation
     - Pagination and infinite scrolling

   - **VeeValidate**: For form handling
     - Validation, error messages, and form state management

## Next Actions: Exercises and Micro-Projects

### 1. Custom Composable Project

Create a `useMediaLibrary` composable that manages a collection of media items (images, videos) with features like filtering, sorting, tagging, and searching. Implement persistence with localStorage and optimize for performance with pagination.

### 2. Ecosystem Integration Challenge

Build a dashboard that integrates at least three ecosystem composables to display real-time data, user preferences, and interactive charts. Handle error states gracefully and ensure a smooth user experience.

### 3. State Management Migration

Take an existing Vue 2 + Vuex application and migrate it to Vue 3 with Pinia using composables. Compare the code organization, readability, and performance before and after.

### 4. Testing Workshop

Write comprehensive tests for a set of composables, covering both unit tests for isolated functionality and integration tests for composables that work together. Implement test mocks for external dependencies and edge cases.

### 5. Form System

Build a complete form system using composables for field validation, form submission, error handling, and dynamic field generation. Support different field types and complex validation rules.

### 6. Real-time Collaboration

Implement a collaborative editing feature using composables to manage WebSocket connections, change tracking, and conflict resolution. Design for reactivity and performance.

## Success Criteria

You've mastered Module 3: Composition API - Composables when you can:

1. **Design Effective Composables** - Create well-structured, reusable composables that separate concerns and solve specific problems.

2. **Organize Related Functionality** - Group related composables logically and establish consistent patterns for your application.

3. **Share State Efficiently** - Implement state sharing between components that balances reactivity, performance, and maintainability.

4. **Handle Errors Gracefully** - Implement robust error handling in composables, including appropriate error propagation and recovery.

5. **Write Comprehensive Tests** - Create thorough test coverage for composables, including edge cases, asynchronous behavior, and side effects.

6. **Leverage the Ecosystem** - Evaluate, integrate, and extend existing ecosystem composables to solve common problems efficiently.

## Troubleshooting Guide

### Reactivity Issues

- **Problem**: Changes to state in composables not reflecting in components.
  **Solution**: Ensure you're returning refs and reactive objects directly, not unwrapping them with `.value` or destructuring. Use `toRefs` if you need to destructure reactive objects.

- **Problem**: Losing reactivity when sharing state between components.
  **Solution**: Use the singleton pattern or provide/inject for shared state. Make sure state changes are made through methods that modify the reactive source of truth.

### Organization Issues

- **Problem**: Composables growing too large and complex.
  **Solution**: Break them down into smaller, focused composables with clear responsibilities. Use composition to build higher-level features from simpler building blocks.

- **Problem**: Too many interdependencies between composables.
  **Solution**: Establish a clear hierarchy and dependency flow. Consider using dependency injection for shared services rather than direct imports.

### Error Handling Issues

- **Problem**: Errors in composables causing entire application crashes.
  **Solution**: Implement proper try/catch blocks for all async operations. Use error state in composables to handle and display errors gracefully in components.

- **Problem**: Difficulty tracing errors back to source in complex composable chains.
  **Solution**: Add context to errors as they propagate through composables. Consider creating custom error types for different failure scenarios.

### Testing Issues

- **Problem**: Difficulty testing composables with external dependencies.
  **Solution**: Design composables to accept dependencies as parameters when possible. Use mocking for tests and write integration tests alongside unit tests.

- **Problem**: Testing composables that use lifecycle hooks.
  **Solution**: Create wrapper components for testing or mock the Vue lifecycle hooks directly. Use component testing tools for integration tests.

### Ecosystem Integration Issues

- **Problem**: Version conflicts between ecosystem libraries.
  **Solution**: Maintain a clear dependency strategy. Create adapter composables that can be updated if underlying libraries change their APIs.

- **Problem**: Excessive bundle size from too many ecosystem imports.
  **Solution**: Use code splitting and dynamic imports for heavy features. Consider implementing simple functionality yourself for minimal features.
