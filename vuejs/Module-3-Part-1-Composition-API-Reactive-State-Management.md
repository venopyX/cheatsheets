# Module 3: Composition API - Part 1: Reactive State Management

## Core Problem

Managing state in complex applications becomes challenging with the Options API as components grow. The Composition API solves this by allowing developers to organize code by logical concern rather than by option type, leading to more maintainable and reusable code.

## Key Assumptions

- You have basic understanding of Vue 3 components
- You're familiar with JavaScript ES6+ features (destructuring, arrow functions)
- You understand reactive programming concepts at a basic level
- You want to use the Composition API with the `<script setup>` syntax

## Essential Concepts

### 1. Ref and Reactive

**Concise Explanation:**
Vue 3's Composition API provides two main ways to create reactive state: `ref` for primitive values (requiring `.value` to access) and `reactive` for objects (with direct property access). These functions wrap your data in reactive proxies that Vue can track for changes.

**Where to Use:**

- `ref`: For primitive values (strings, numbers, booleans) and when you need to pass reactive values between functions
- `reactive`: For objects and arrays when you want direct property access
- Both in combination: For complex state management where some values need to be passed separately

**Code Snippet:**

```html
<template>
  <div class="p-6 max-w-md mx-auto bg-white rounded-xl shadow-md">
    <!-- Using ref in template (no .value needed) -->
    <h1 class="text-xl font-medium text-gray-700">Count: {{ count }}</h1>

    <!-- Using reactive object properties in template -->
    <div class="mt-4 p-4 bg-gray-50 rounded">
      <h2 class="text-lg font-medium text-gray-600">User Info</h2>
      <p>Name: {{ user.name }}</p>
      <p>Email: {{ user.email }}</p>
      <p>Active: {{ user.isActive ? "Yes" : "No" }}</p>
    </div>

    <!-- Event handlers for updating state -->
    <div class="mt-4 flex space-x-2">
      <button
        @click="increment"
        class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
      >
        Increment
      </button>
      <button
        @click="toggleActive"
        class="px-4 py-2 bg-green-600 text-white rounded hover:bg-green-700"
      >
        Toggle Active
      </button>
      <button
        @click="resetUser"
        class="px-4 py-2 bg-gray-600 text-white rounded hover:bg-gray-700"
      >
        Reset User
      </button>
    </div>
  </div>
</template>

<script setup>
  import { ref, reactive } from "vue";

  // Primitive value with ref
  const count = ref(0);

  // Object with reactive
  const user = reactive({
    name: "John Doe",
    email: "john@example.com",
    isActive: true,
  });

  // Nested reactive object
  const state = reactive({
    posts: [
      { id: 1, title: "Vue 3 Guide" },
      { id: 2, title: "Composition API" },
    ],
    loading: false,
    error: null,
  });

  // Methods to update the state
  const increment = () => {
    // Need .value for refs in JavaScript code
    count.value++;
  };

  const toggleActive = () => {
    // Direct mutation for reactive objects
    user.isActive = !user.isActive;
  };

  const resetUser = () => {
    // Updating multiple properties on reactive object
    Object.assign(user, {
      name: "John Doe",
      email: "john@example.com",
      isActive: true,
    });

    // Alternative: individual assignments
    // user.name = 'John Doe'
    // user.email = 'john@example.com'
    // user.isActive = true
  };

  // Using the reactive state in functions
  const fetchPosts = async () => {
    state.loading = true;
    state.error = null;

    try {
      // Simulated API call
      const response = await fetch("https://api.example.com/posts");
      const data = await response.json();
      state.posts = data;
    } catch (error) {
      state.error = error.message || "Failed to fetch posts";
    } finally {
      state.loading = false;
    }
  };
</script>
```

**Real-World Example:**
In a dashboard application, you might use `ref` for user preferences that are accessed across multiple components (theme, language) and `reactive` for complex data structures like the current view's dataset, filters, and pagination state.

```html
<script setup>
  import { ref, reactive, onMounted } from "vue";
  import { fetchDashboardData } from "@/api/dashboard";

  // Simple values with ref
  const isLoading = ref(false);
  const selectedView = ref("overview");
  const refreshInterval = ref(30); // seconds

  // Complex state with reactive
  const dashboardState = reactive({
    metrics: {
      visits: 0,
      conversions: 0,
      revenue: 0,
    },
    recentOrders: [],
    topProducts: [],
    filters: {
      dateRange: "last7days",
      status: "all",
    },
    pagination: {
      page: 1,
      limit: 10,
      total: 0,
    },
  });

  // Load dashboard data
  onMounted(async () => {
    isLoading.value = true;
    try {
      const data = await fetchDashboardData(dashboardState.filters);
      // Update multiple properties at once
      Object.assign(dashboardState.metrics, data.metrics);
      dashboardState.recentOrders = data.recentOrders;
      dashboardState.topProducts = data.topProducts;
      dashboardState.pagination.total = data.totalOrders;
    } catch (error) {
      console.error("Failed to load dashboard:", error);
    } finally {
      isLoading.value = false;
    }
  });
</script>
```

**Common Pitfalls:**

- Forgetting to use `.value` when accessing or modifying refs in JavaScript code
- Destructuring reactive objects, which loses reactivity
- Not using the correct API for the right data type
- Trying to make existing, non-reactive objects reactive after creation
- Replacing entire reactive objects instead of modifying their properties

**Frequently Asked Questions:**

1. **Q: When should I use ref vs. reactive?**

   A: Use `ref` for primitive values (strings, numbers, booleans) or when you need to pass reactive values around as arguments. Use `reactive` for objects when you want direct property access without `.value`. As a rule of thumb: if it's a single value, use `ref`; if it's an object with properties, use `reactive`.

2. **Q: Can I use both ref and reactive in the same component?**

   A: Yes, you can and often should. It's common to use `ref` for simpler state and `reactive` for more complex objects. They can work together seamlessly in the same component.

3. **Q: Why do I lose reactivity when destructuring reactive objects?**

   A: The reactive proxy wraps the entire object; when you destructure it, you're extracting the raw values, which are no longer connected to the reactive system. To preserve reactivity, either:

   - Access properties directly: `user.name` instead of `const { name } = user`
   - Use `toRefs` to convert reactive properties to refs: `const { name } = toRefs(user)`
   - Use computed properties: `const name = computed(() => user.name)`

### 2. Computed Properties

**Concise Explanation:**
Computed properties are reactive values derived from other reactive state. They automatically recalculate when their dependencies change and cache their results for better performance. They're perfect for values that need to be derived from other state.

**Where to Use:**

- Formatting or transforming data for display
- Filtering or sorting lists
- Calculating values based on multiple reactive sources
- Form validation rules
- Any derived value that depends on other reactive state

**Code Snippet:**

```html
<template>
  <div class="p-6 max-w-md mx-auto bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800">Product Filter</h1>

    <!-- Search input -->
    <div class="mt-4">
      <input
        v-model="searchQuery"
        placeholder="Search products..."
        class="w-full px-4 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
      />
    </div>

    <!-- Category filter -->
    <div class="mt-4">
      <select
        v-model="selectedCategory"
        class="w-full px-4 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
      >
        <option value="">All Categories</option>
        <option
          v-for="category in uniqueCategories"
          :key="category"
          :value="category"
        >
          {{ category }}
        </option>
      </select>
    </div>

    <!-- Price range -->
    <div class="mt-4">
      <label class="block text-sm font-medium text-gray-700"
        >Max Price: ${{ maxPrice }}</label
      >
      <input
        v-model="maxPrice"
        type="range"
        min="0"
        max="1000"
        step="50"
        class="w-full"
      />
    </div>

    <!-- Display stats using computed properties -->
    <div class="mt-4 p-3 bg-blue-50 rounded">
      <p>
        Showing {{ filteredProducts.length }} of {{ products.length }} products
      </p>
      <p>Average price: ${{ averagePrice }}</p>
    </div>

    <!-- Product list -->
    <div v-if="filteredProducts.length" class="mt-4 space-y-3">
      <div
        v-for="product in filteredProducts"
        :key="product.id"
        class="p-3 border rounded hover:bg-gray-50"
      >
        <div class="flex justify-between">
          <h3 class="font-medium">{{ product.name }}</h3>
          <span :class="priceColorClass(product.price)"
            >${{ product.price }}</span
          >
        </div>
        <p class="text-sm text-gray-600">Category: {{ product.category }}</p>
      </div>
    </div>

    <div v-else class="mt-4 p-3 bg-gray-50 rounded text-center text-gray-500">
      No products match your filters.
    </div>
  </div>
</template>

<script setup>
  import { ref, reactive, computed } from "vue";

  // Input state with refs
  const searchQuery = ref("");
  const selectedCategory = ref("");
  const maxPrice = ref(500);

  // Product data with reactive
  const products = reactive([
    { id: 1, name: "Laptop", price: 899, category: "Electronics" },
    { id: 2, name: "Headphones", price: 249, category: "Electronics" },
    { id: 3, name: "Coffee Maker", price: 129, category: "Kitchen" },
    { id: 4, name: "Running Shoes", price: 89, category: "Sports" },
    { id: 5, name: "Monitor", price: 349, category: "Electronics" },
    { id: 6, name: "Blender", price: 79, category: "Kitchen" },
  ]);

  // Simple computed property
  const uniqueCategories = computed(() => {
    return [...new Set(products.map((product) => product.category))].sort();
  });

  // Computed property with complex logic
  const filteredProducts = computed(() => {
    return (
      products
        .filter((product) => {
          // Filter by search query
          if (
            searchQuery.value &&
            !product.name
              .toLowerCase()
              .includes(searchQuery.value.toLowerCase())
          ) {
            return false;
          }

          // Filter by category
          if (
            selectedCategory.value &&
            product.category !== selectedCategory.value
          ) {
            return false;
          }

          // Filter by price
          if (product.price > maxPrice.value) {
            return false;
          }

          return true;
        })
        // Sort by price
        .sort((a, b) => a.price - b.price)
    );
  });

  // Computed property for a calculated value
  const averagePrice = computed(() => {
    if (filteredProducts.value.length === 0) return 0;

    const total = filteredProducts.value.reduce(
      (sum, product) => sum + product.price,
      0
    );
    return (total / filteredProducts.value.length).toFixed(2);
  });

  // Function that uses computed values
  const priceColorClass = (price) => {
    if (price < 100) return "text-green-600";
    if (price < 300) return "text-blue-600";
    return "text-red-600";
  };
</script>
```

**Real-World Example:**
In an e-commerce checkout flow, computed properties can manage the shopping cart summary, calculating subtotals, tax, shipping cost, and final total. They would automatically update when items are added, removed, or quantities change.

```html
<template>
  <div class="p-6 max-w-lg mx-auto bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-bold mb-4">Checkout Summary</h1>

    <!-- Cart items -->
    <div class="border-b pb-4">
      <div
        v-for="item in cart.items"
        :key="item.id"
        class="flex justify-between py-2"
      >
        <div>
          <span class="font-medium">{{ item.name }}</span>
          <span class="text-gray-600 ml-2">× {{ item.quantity }}</span>
        </div>
        <span>${{ (item.price * item.quantity).toFixed(2) }}</span>
      </div>
    </div>

    <!-- Summary details -->
    <div class="space-y-2 py-4">
      <div class="flex justify-between">
        <span>Subtotal</span>
        <span>${{ subtotal }}</span>
      </div>

      <div class="flex justify-between">
        <span>Tax ({{ taxRate * 100 }}%)</span>
        <span>${{ taxAmount }}</span>
      </div>

      <div class="flex justify-between">
        <span>Shipping</span>
        <span>${{ shippingCost }}</span>
      </div>

      <div
        v-if="discountAmount > 0"
        class="flex justify-between text-green-600"
      >
        <span>Discount</span>
        <span>-${{ discountAmount }}</span>
      </div>
    </div>

    <!-- Total -->
    <div class="border-t pt-4">
      <div class="flex justify-between font-bold text-lg">
        <span>Total</span>
        <span>${{ totalAmount }}</span>
      </div>

      <div v-if="freeShippingRemaining > 0" class="text-sm text-blue-600 mt-2">
        Add ${{ freeShippingRemaining }} more to qualify for free shipping!
      </div>
    </div>
  </div>
</template>

<script setup>
  import { reactive, computed, ref } from "vue";

  // Constants
  const FREE_SHIPPING_THRESHOLD = 100;
  const taxRate = ref(0.08);
  const shippingRate = ref(10);
  const promoCode = ref("");
  const promoApplied = ref(false);

  // Cart data
  const cart = reactive({
    items: [
      { id: 1, name: "Wireless Earbuds", price: 79.99, quantity: 1 },
      { id: 2, name: "Phone Case", price: 19.99, quantity: 1 },
    ],
  });

  // Computed properties for order calculations
  const subtotal = computed(() => {
    return cart.items
      .reduce((sum, item) => sum + item.price * item.quantity, 0)
      .toFixed(2);
  });

  const subtotalValue = computed(() => {
    return Number(subtotal.value);
  });

  const taxAmount = computed(() => {
    return (subtotalValue.value * taxRate.value).toFixed(2);
  });

  const shippingCost = computed(() => {
    // Free shipping over threshold
    return subtotalValue.value >= FREE_SHIPPING_THRESHOLD
      ? "0.00"
      : shippingRate.value.toFixed(2);
  });

  const discountAmount = computed(() => {
    // Apply 10% discount if promo code is applied
    return promoApplied.value ? (subtotalValue.value * 0.1).toFixed(2) : "0.00";
  });

  const totalAmount = computed(() => {
    return (
      subtotalValue.value +
      Number(taxAmount.value) +
      Number(shippingCost.value) -
      Number(discountAmount.value)
    ).toFixed(2);
  });

  const freeShippingRemaining = computed(() => {
    if (subtotalValue.value >= FREE_SHIPPING_THRESHOLD) return 0;
    return (FREE_SHIPPING_THRESHOLD - subtotalValue.value).toFixed(2);
  });

  // Method to apply promo code
  const applyPromoCode = (code) => {
    if (code === "SAVE10") {
      promoApplied.value = true;
      return true;
    }
    return false;
  };
</script>
```

**Common Pitfalls:**

- Mutating state inside computed properties (they should be pure functions)
- Creating expensive computations without proper memoization
- Not handling edge cases (empty arrays, null values)
- Using computed properties when methods would be more appropriate
- Creating circular dependencies between computed properties

**Frequently Asked Questions:**

1. **Q: Can I modify values within a computed property?**

   A: No, computed properties should be pure functions without side effects. They should only calculate and return a value based on their dependencies. If you need to modify state, use a method or a watcher instead.

2. **Q: When should I use computed vs. methods?**

   A: Use computed properties for derived values that depend on reactive state and should be cached. Use methods when you need to:

   - Perform actions rather than just return values
   - Pass arguments to the function
   - Manually trigger the calculation (not automatically on dependency changes)
   - Create functions that aren't tied to reactive state

3. **Q: Can I make a computed property writable?**

   A: Yes, you can create a writable computed property by providing both a getter and setter:

   ```js
   const fullName = computed({
     get() {
       return `${firstName.value} ${lastName.value}`;
     },
     set(newValue) {
       const names = newValue.split(" ");
       firstName.value = names[0];
       lastName.value = names[1] || "";
     },
   });

   // Usage
   console.log(fullName.value); // "John Doe"
   fullName.value = "Jane Smith"; // Updates firstName and lastName
   ```

### 3. Watchers

**Concise Explanation:**
Watchers allow you to observe changes in reactive data and execute side effects or complex logic in response. Unlike computed properties, watchers are ideal for asynchronous operations, expensive calculations, or when you need to react to specific data changes with side effects.

**Where to Use:**

- Performing API calls when state changes
- Syncing state with localStorage or external systems
- Complex data transformations that need to happen after state changes
- Validating input with debouncing
- Side effects that shouldn't block rendering

**Code Snippet:**

```html
<template>
  <div class="p-6 max-w-md mx-auto bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800 mb-4">User Profile</h1>

    <!-- Username field with validation -->
    <div class="mb-4">
      <label class="block text-sm font-medium text-gray-700 mb-1"
        >Username</label
      >
      <input
        v-model="username"
        class="w-full px-4 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
        :class="{ 'border-red-500': usernameError }"
      />
      <p v-if="usernameError" class="mt-1 text-sm text-red-600">
        {{ usernameError }}
      </p>
      <p v-if="isCheckingUsername" class="mt-1 text-sm text-blue-600">
        Checking availability...
      </p>
    </div>

    <!-- Theme selection with watcher -->
    <div class="mb-4">
      <label class="block text-sm font-medium text-gray-700 mb-1">Theme</label>
      <select
        v-model="theme"
        class="w-full px-4 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
      >
        <option value="light">Light</option>
        <option value="dark">Dark</option>
        <option value="system">System</option>
      </select>
      <p class="mt-1 text-sm text-gray-600">
        Current theme: {{ effectiveTheme }}
      </p>
    </div>

    <!-- Preferences object with deep watcher -->
    <div class="mb-4">
      <h2 class="text-lg font-medium text-gray-700 mb-2">
        Notification Preferences
      </h2>

      <div class="space-y-2">
        <label class="flex items-center">
          <input
            v-model="preferences.email"
            type="checkbox"
            class="h-4 w-4 text-blue-600 rounded"
          />
          <span class="ml-2">Email Notifications</span>
        </label>

        <label class="flex items-center">
          <input
            v-model="preferences.push"
            type="checkbox"
            class="h-4 w-4 text-blue-600 rounded"
          />
          <span class="ml-2">Push Notifications</span>
        </label>

        <label class="flex items-center">
          <input
            v-model="preferences.sms"
            type="checkbox"
            class="h-4 w-4 text-blue-600 rounded"
          />
          <span class="ml-2">SMS Notifications</span>
        </label>
      </div>
    </div>

    <!-- Location with immediate watcher -->
    <div class="mb-4">
      <label class="block text-sm font-medium text-gray-700 mb-1"
        >Location</label
      >
      <select
        v-model="location"
        class="w-full px-4 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
      >
        <option value="">Select a location</option>
        <option value="us">United States</option>
        <option value="ca">Canada</option>
        <option value="uk">United Kingdom</option>
        <option value="au">Australia</option>
      </select>

      <div v-if="timezones.length" class="mt-2">
        <label class="block text-sm font-medium text-gray-700 mb-1"
          >Timezone</label
        >
        <select
          v-model="timezone"
          class="w-full px-4 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
        >
          <option v-for="tz in timezones" :key="tz.value" :value="tz.value">
            {{ tz.label }}
          </option>
        </select>
      </div>
    </div>

    <!-- Status message for saving -->
    <div v-if="saveStatus" :class="`mt-4 p-3 rounded ${saveStatusClass}`">
      {{ saveStatus }}
    </div>

    <button
      @click="saveProfile"
      class="mt-4 w-full px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
    >
      Save Profile
    </button>
  </div>
</template>

<script setup>
  import { ref, reactive, watch, computed, onMounted } from "vue";

  // Reactive state
  const username = ref("");
  const usernameError = ref("");
  const isCheckingUsername = ref(false);
  const theme = ref("system");
  const effectiveTheme = ref("light");
  const location = ref("");
  const timezone = ref("");
  const timezones = ref([]);
  const saveStatus = ref("");

  // Reactive object for preferences
  const preferences = reactive({
    email: true,
    push: true,
    sms: false,
  });

  // Computed property for save status styling
  const saveStatusClass = computed(() => {
    return saveStatus.value.includes("Error")
      ? "bg-red-100 text-red-800"
      : "bg-green-100 text-green-800";
  });

  // Simple watcher for username
  watch(
    username,
    async (newUsername, oldUsername) => {
      // Don't validate empty usernames
      if (!newUsername) {
        usernameError.value = "";
        return;
      }

      // Basic validation
      if (newUsername.length < 3) {
        usernameError.value = "Username must be at least 3 characters";
        return;
      }

      // Simulated API call to check username availability
      if (newUsername !== oldUsername) {
        usernameError.value = "";
        isCheckingUsername.value = true;

        try {
          // Simulate API delay
          await new Promise((resolve) => setTimeout(resolve, 1000));

          // Simulate API response (username 'admin' is taken)
          if (newUsername.toLowerCase() === "admin") {
            usernameError.value = "This username is already taken";
          }
        } finally {
          isCheckingUsername.value = false;
        }
      }
    },
    { immediate: true }
  ); // Run on component creation

  // Watcher for theme with system preference detection
  watch(
    theme,
    (newTheme) => {
      if (newTheme === "system") {
        // Check system preference
        const prefersDark = window.matchMedia(
          "(prefers-color-scheme: dark)"
        ).matches;
        effectiveTheme.value = prefersDark ? "dark" : "light";

        // Also watch for system changes
        const mediaQuery = window.matchMedia("(prefers-color-scheme: dark)");
        const handler = (e) => {
          effectiveTheme.value = e.matches ? "dark" : "light";
          applyTheme(effectiveTheme.value);
        };

        // Setup listener
        mediaQuery.addEventListener("change", handler);

        // Cleanup (would be in onUnmounted in a real component)
        /* onUnmounted(() => {
      mediaQuery.removeEventListener('change', handler)
    }) */
      } else {
        effectiveTheme.value = newTheme;
      }

      // Apply the theme
      applyTheme(effectiveTheme.value);
    },
    { immediate: true }
  );

  // Deep watcher for preferences object
  watch(
    preferences,
    (newPrefs) => {
      // Save to localStorage
      localStorage.setItem("userPreferences", JSON.stringify(newPrefs));

      // Show confirmation
      saveStatus.value = "Preferences saved automatically";
      setTimeout(() => {
        saveStatus.value = "";
      }, 2000);
    },
    { deep: true }
  ); // deep: true is needed to watch nested properties

  // Immediate watcher for location
  watch(
    location,
    async (newLocation) => {
      // Reset timezone when location changes
      timezone.value = "";
      timezones.value = [];

      if (!newLocation) return;

      // Simulate API call to get timezones for location
      try {
        // In a real app, this would be an API call
        await new Promise((resolve) => setTimeout(resolve, 500));

        // Mock response based on location
        const mockTimezones = {
          us: [
            { value: "America/New_York", label: "Eastern Time (ET)" },
            { value: "America/Chicago", label: "Central Time (CT)" },
            { value: "America/Denver", label: "Mountain Time (MT)" },
            { value: "America/Los_Angeles", label: "Pacific Time (PT)" },
          ],
          ca: [
            { value: "America/Toronto", label: "Eastern Time (ET)" },
            { value: "America/Winnipeg", label: "Central Time (CT)" },
            { value: "America/Edmonton", label: "Mountain Time (MT)" },
            { value: "America/Vancouver", label: "Pacific Time (PT)" },
          ],
          uk: [{ value: "Europe/London", label: "Greenwich Mean Time (GMT)" }],
          au: [
            {
              value: "Australia/Sydney",
              label: "Australian Eastern Time (AET)",
            },
            {
              value: "Australia/Adelaide",
              label: "Australian Central Time (ACT)",
            },
            {
              value: "Australia/Perth",
              label: "Australian Western Time (AWT)",
            },
          ],
        };

        timezones.value = mockTimezones[newLocation] || [];

        // Set default timezone if available
        if (timezones.value.length) {
          timezone.value = timezones.value[0].value;
        }
      } catch (error) {
        console.error("Failed to fetch timezones:", error);
      }
    },
    { immediate: true }
  );

  // Function to apply theme (in a real app, this would modify CSS)
  const applyTheme = (theme) => {
    console.log(`Applying theme: ${theme}`);
    // In a real app, this would apply CSS classes or variables
  };

  // Load preferences from localStorage on mount
  onMounted(() => {
    try {
      const savedPrefs = localStorage.getItem("userPreferences");
      if (savedPrefs) {
        const parsed = JSON.parse(savedPrefs);
        // Update our reactive object
        Object.assign(preferences, parsed);
      }
    } catch (e) {
      console.error("Failed to load preferences from localStorage", e);
    }
  });

  // Save all profile data
  const saveProfile = async () => {
    if (usernameError.value) {
      saveStatus.value = "Error: Please fix the username issues";
      return;
    }

    try {
      // Simulate API call
      await new Promise((resolve) => setTimeout(resolve, 1500));

      // Success
      saveStatus.value = "Profile saved successfully!";

      // Clear status after 3 seconds
      setTimeout(() => {
        saveStatus.value = "";
      }, 3000);
    } catch (error) {
      saveStatus.value = `Error: ${error.message || "Failed to save profile"}`;
    }
  };
</script>
```

**Real-World Example:**
In a collaborative document editor, watchers can sync local changes to a backend, debouncing frequent updates to avoid excessive API calls. They can also track user presence, cursor position, and notify other users of changes in real-time.

```html
<script setup>
  import { ref, watch, onMounted, onUnmounted } from "vue";
  import { debounce } from "lodash-es";
  import { saveDocument, trackUserPresence } from "@/api/document";

  // Document state
  const documentId = ref("doc-123");
  const content = ref("");
  const isSaving = ref(false);
  const lastSaved = ref(null);
  const collaborators = ref([]);
  const connectionStatus = ref("connecting");

  // Setup auto-saving with debounce
  const saveContent = debounce(async () => {
    if (!content.value.trim()) return;

    isSaving.value = true;
    try {
      await saveDocument(documentId.value, content.value);
      lastSaved.value = new Date();
    } catch (error) {
      console.error("Failed to save document:", error);
      // In a real app, we'd show an error to the user and retry
    } finally {
      isSaving.value = false;
    }
  }, 1000); // debounce for 1 second

  // Watch for content changes and save
  watch(content, (newContent) => {
    // Don't save empty content
    if (!newContent.trim()) return;

    // Update status
    isSaving.value = true;

    // Trigger debounced save
    saveContent();
  });

  // Setup presence tracking
  let presenceInterval;
  onMounted(() => {
    // Initialize connection
    setupRealTimeConnection();

    // Update user presence every 30 seconds
    presenceInterval = setInterval(async () => {
      try {
        await trackUserPresence(documentId.value, {
          status: "active",
          lastActive: new Date(),
        });
      } catch (error) {
        console.error("Failed to update presence:", error);
      }
    }, 30000);
  });

  // Cleanup on component unmount
  onUnmounted(() => {
    // Clear presence interval
    if (presenceInterval) {
      clearInterval(presenceInterval);
    }

    // Ensure any pending saves are processed
    saveContent.flush();

    // Disconnect from real-time server
    disconnectRealTime();
  });

  // Real-time connection handling
  const setupRealTimeConnection = () => {
    // In a real app, this would connect to WebSockets or similar
    connectionStatus.value = "connected";

    // Mock collaborators data
    collaborators.value = [
      { id: "user-1", name: "Alice", status: "active" },
      { id: "user-2", name: "Bob", status: "idle" },
    ];
  };

  const disconnectRealTime = () => {
    // In a real app, this would disconnect from the real-time server
    connectionStatus.value = "disconnected";
  };
</script>
```

**Common Pitfalls:**

- Creating watchers that trigger other watchers, leading to infinite loops
- Not using the `deep` option when watching objects or arrays
- Using watchers when computed properties would be more appropriate
- Forgetting to clean up side effects (e.g., event listeners) in watch callbacks
- Not handling errors in asynchronous operations

**Frequently Asked Questions:**

1. **Q: What's the difference between watch and watchEffect?**

   A: `watch` allows you to specifically watch one or more reactive sources and access both new and old values. `watchEffect` automatically tracks all reactive dependencies used within it but doesn't provide access to previous values. Use `watch` when you need to compare old and new values or want explicit control over what triggers the effect. Use `watchEffect` for simpler cases where you just need to run side effects based on dependencies.

2. **Q: When should I use the immediate option?**

   A: Use `{ immediate: true }` when you want the watcher callback to run immediately upon creation, not just when the watched value changes. This is useful for initialization logic that depends on reactive state, validating initial values, or fetching data based on props.

3. **Q: How can I stop a watcher?**

   A: Watchers return a stop function that can be called to stop the watcher:

   ```js
   const stopWatcher = watch(source, callback);

   // Later, when you want to stop watching
   stopWatcher();
   ```

   You can also use `onUnmounted` to automatically stop watchers when a component is unmounted.

### 4. Lifecycle Hooks in Composition API

**Concise Explanation:**
Lifecycle hooks in the Composition API are imported functions that you call within the `setup()` function or `<script setup>`. They allow you to register callbacks that execute at specific stages of a component's lifecycle, such as when it's mounted, updated, or destroyed.

**Where to Use:**

- `onMounted`: Initialize third-party libraries, fetch initial data, access DOM
- `onUpdated`: React to component updates after DOM changes
- `onUnmounted`: Clean up resources, event listeners, timers
- `onBeforeMount`/`onBeforeUpdate`: Prepare data before DOM operations
- `onErrorCaptured`: Handle errors from child components
- `onActivated`/`onDeactivated`: Handle components inside `<keep-alive>`

**Code Snippet:**

```html
<template>
  <div class="p-6 max-w-md mx-auto bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800 mb-4">User Dashboard</h1>

    <!-- Loading state -->
    <div v-if="loading" class="flex justify-center py-8">
      <div
        class="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-500"
      ></div>
    </div>

    <!-- Error state -->
    <div v-else-if="error" class="bg-red-100 p-4 rounded text-red-800">
      <p>{{ error }}</p>
      <button
        @click="fetchUserData"
        class="mt-2 px-4 py-1 bg-red-600 text-white rounded"
      >
        Retry
      </button>
    </div>

    <!-- User data -->
    <div v-else-if="user" class="space-y-4">
      <div class="flex items-center">
        <img
          :src="user.avatar"
          :alt="user.name"
          class="w-16 h-16 rounded-full"
          ref="avatarRef"
        />
        <div class="ml-4">
          <h2 class="font-bold text-lg">{{ user.name }}</h2>
          <p class="text-gray-600">{{ user.email }}</p>
        </div>
      </div>

      <div class="border-t pt-4">
        <h3 class="font-medium mb-2">Activity Stats</h3>
        <div ref="chartContainer" class="w-full h-48 bg-gray-50 rounded"></div>
      </div>

      <div class="border-t pt-4">
        <h3 class="font-medium mb-2">Recent Activity</h3>
        <ul class="space-y-2">
          <li
            v-for="activity in user.recentActivity"
            :key="activity.id"
            class="p-2 hover:bg-gray-50 rounded"
          >
            {{ activity.description }}
            <span class="text-sm text-gray-500 block">
              {{ formatDate(activity.timestamp) }}
            </span>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script setup>
  import {
    ref,
    onMounted,
    onUpdated,
    onUnmounted,
    onErrorCaptured,
    onBeforeMount,
  } from "vue";
  import { fetchUser, trackUserVisit } from "@/api/user";
  import { format } from "date-fns";
  import ChartLibrary from "chart-library";

  // Component state
  const user = ref(null);
  const loading = ref(true);
  const error = ref(null);
  const avatarRef = ref(null);
  const chartContainer = ref(null);
  const chart = ref(null);
  const connectionId = ref(null);

  // Format date helper
  const formatDate = (dateString) => {
    return format(new Date(dateString), "MMM d, yyyy h:mm a");
  };

  // Fetch user data
  const fetchUserData = async () => {
    loading.value = true;
    error.value = null;

    try {
      // Simulate API call
      await new Promise((resolve) => setTimeout(resolve, 1000));

      // Mock user data
      user.value = {
        id: 1,
        name: "Jane Smith",
        email: "jane@example.com",
        avatar: "https://ui-avatars.com/api/?name=Jane+Smith",
        stats: {
          posts: 24,
          followers: 142,
          following: 98,
          engagement: [10, 15, 8, 22, 30, 25, 12],
        },
        recentActivity: [
          {
            id: 1,
            description: "Created a new post",
            timestamp: "2025-05-19T14:30:00Z",
          },
          {
            id: 2,
            description: 'Commented on "Vue 3 Tips"',
            timestamp: "2025-05-18T09:45:00Z",
          },
          {
            id: 3,
            description: 'Liked "Composition API Guide"',
            timestamp: "2025-05-17T16:20:00Z",
          },
        ],
      };
    } catch (e) {
      error.value =
        "Failed to load user data: " + (e.message || "Unknown error");
    } finally {
      loading.value = false;
    }
  };

  // Initialize before mounting (data prep, not DOM)
  onBeforeMount(() => {
    console.log("Component is about to mount");
    connectionId.value = `user-${Date.now()}`;
  });

  // Initialize after mounting (DOM is available)
  onMounted(async () => {
    console.log("Component mounted, DOM is ready");

    // Fetch initial data
    await fetchUserData();

    // Track user visit in analytics
    trackUserVisit(connectionId.value);

    // Initialize third-party chart library once data is loaded
    if (user.value && chartContainer.value) {
      initializeChart();
    }

    // Add window event listeners
    window.addEventListener("resize", handleResize);

    // Add custom event for demonstration
    document.addEventListener("custom:event", handleCustomEvent);
  });

  // React to updates
  onUpdated(() => {
    console.log("Component updated");

    // Update chart if data changed and chart exists
    if (chart.value && user.value) {
      chart.value.updateData(user.value.stats.engagement);
    }

    // Demonstrate accessing updated DOM
    if (avatarRef.value) {
      console.log(
        "Avatar element dimensions:",
        avatarRef.value.clientWidth,
        avatarRef.value.clientHeight
      );
    }
  });

  // Cleanup when component is destroyed
  onUnmounted(() => {
    console.log("Component will be unmounted");

    // Clean up chart library
    if (chart.value) {
      chart.value.destroy();
      chart.value = null;
    }

    // Remove event listeners
    window.removeEventListener("resize", handleResize);
    document.removeEventListener("custom:event", handleCustomEvent);

    // Clean up any other resources
    console.log(`Closing connection ${connectionId.value}`);
  });

  // Error handling
  onErrorCaptured((err, instance, info) => {
    // Log the error
    console.error(`Error captured: ${err.message}`, info);

    // Display user-friendly error message
    error.value = "Something went wrong. Please try again.";

    // Return false to prevent error propagation
    return false;
  });

  // Event handlers
  const handleResize = () => {
    if (chart.value) {
      chart.value.resize();
    }
  };

  const handleCustomEvent = (event) => {
    console.log("Custom event received:", event.detail);
  };

  // Initialize chart library (simulated)
  const initializeChart = () => {
    // In a real app, this would use an actual chart library
    chart.value = new ChartLibrary(chartContainer.value, {
      data: user.value.stats.engagement,
      type: "line",
      colors: ["#3B82F6"],
    });
  };
</script>
```

**Real-World Example:**
In a video conferencing application, lifecycle hooks manage the camera and microphone access. `onMounted` initializes the media stream, `onBeforeUnmount` releases hardware resources, and `onErrorCaptured` handles permission denials or device disconnections.

```html
<template>
  <div class="p-6 max-w-2xl mx-auto bg-white rounded-xl shadow-lg">
    <h1 class="text-2xl font-bold mb-4">Video Call</h1>

    <!-- Video display -->
    <div class="relative bg-gray-900 rounded overflow-hidden aspect-video">
      <!-- Local video -->
      <video
        ref="localVideoRef"
        class="absolute bottom-4 right-4 w-1/4 rounded border-2 border-white"
        autoplay
        muted
        playsinline
      ></video>

      <!-- Remote video (full size) -->
      <video
        ref="remoteVideoRef"
        class="w-full h-full"
        autoplay
        playsinline
      ></video>

      <!-- Loading overlay -->
      <div
        v-if="status === 'connecting'"
        class="absolute inset-0 flex items-center justify-center bg-black/50 text-white"
      >
        <div class="text-center">
          <div
            class="animate-spin rounded-full h-12 w-12 border-4 border-white border-t-transparent mx-auto"
          ></div>
          <p class="mt-4">Connecting...</p>
        </div>
      </div>

      <!-- Error overlay -->
      <div
        v-if="error"
        class="absolute inset-0 flex items-center justify-center bg-black/80 text-white"
      >
        <div class="text-center p-6">
          <svg
            class="h-12 w-12 text-red-500 mx-auto"
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
          <p class="mt-4 text-lg font-medium">{{ error }}</p>
          <button
            @click="initializeMedia"
            class="mt-4 px-4 py-2 bg-red-600 text-white rounded"
          >
            Retry
          </button>
        </div>
      </div>
    </div>

    <!-- Controls -->
    <div class="flex justify-center space-x-4 mt-4">
      <button
        @click="toggleMute"
        class="p-3 rounded-full"
        :class="isMuted ? 'bg-gray-200' : 'bg-blue-500 text-white'"
      >
        <svg
          class="h-6 w-6"
          fill="none"
          viewBox="0 0 24 24"
          stroke="currentColor"
        >
          <path
            v-if="isMuted"
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M19 11a7 7 0 01-7 7m0 0a7 7 0 01-7-7m7 7v4m0 0H8m4 0h4m-4-8a3 3 0 01-3-3V5a3 3 0 116 0v6a3 3 0 01-3 3z"
          />
          <path
            v-else
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M15.536 8.464a5 5 0 010 7.072m2.828-9.9a9 9 0 010 12.728M5.586 15.536a5 5 0 017.072 0m-9.9-2.828a9 9 0 0112.728 0M12 18.75V21m0-14.25V3"
          />
        </svg>
      </button>

      <button
        @click="toggleVideo"
        class="p-3 rounded-full"
        :class="isVideoOff ? 'bg-gray-200' : 'bg-blue-500 text-white'"
      >
        <svg
          class="h-6 w-6"
          fill="none"
          viewBox="0 0 24 24"
          stroke="currentColor"
        >
          <path
            v-if="isVideoOff"
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M15 10l4.553-2.276A1 1 0 0121 8.618v6.764a1 1 0 01-1.447.894L15 14M5 18h8a2 2 0 002-2V8a2 2 0 00-2-2H5a2 2 0 00-2 2v8a2 2 0 002 2z"
          />
          <path
            v-else
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M15 10l4.553-2.276A1 1 0 0121 8.618v6.764a1 1 0 01-1.447.894L15 14M5 18h8a2 2 0 002-2V8a2 2 0 00-2-2H5a2 2 0 00-2 2v8a2 2 0 002 2z"
          />
        </svg>
      </button>

      <button @click="endCall" class="p-3 bg-red-600 text-white rounded-full">
        <svg
          class="h-6 w-6"
          fill="none"
          viewBox="0 0 24 24"
          stroke="currentColor"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M16 8l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2M5 3a2 2 0 00-2 2v1c0 8.284 6.716 15 15 15h1a2 2 0 002-2v-3.28a1 1 0 00-.684-.948l-4.493-1.498a1 1 0 00-1.21.502l-1.13 2.257a11.042 11.042 0 01-5.516-5.517l2.257-1.128a1 1 0 00.502-1.21L9.228 3.683A1 1 0 008.279 3H5z"
          />
        </svg>
      </button>
    </div>
  </div>
</template>

<script setup>
  import {
    ref,
    onMounted,
    onBeforeUnmount,
    onErrorCaptured,
    nextTick,
  } from "vue";
  import { connectToRoom, disconnectFromRoom } from "@/api/videocall";

  // Component state
  const localVideoRef = ref(null);
  const remoteVideoRef = ref(null);
  const localStream = ref(null);
  const remoteStream = ref(null);
  const status = ref("idle"); // idle, connecting, connected
  const error = ref(null);
  const isMuted = ref(false);
  const isVideoOff = ref(false);
  const roomConnection = ref(null);

  // Initialize media devices and stream
  const initializeMedia = async () => {
    status.value = "connecting";
    error.value = null;

    try {
      // Request camera and microphone permissions
      localStream.value = await navigator.mediaDevices.getUserMedia({
        video: true,
        audio: true,
      });

      // Once we have the stream, set it to the video element
      await nextTick();

      if (localVideoRef.value) {
        localVideoRef.value.srcObject = localStream.value;
      }

      // Connect to the room
      await connectToCallRoom();

      status.value = "connected";
    } catch (err) {
      console.error("Failed to initialize media:", err);

      if (err.name === "NotAllowedError") {
        error.value =
          "Camera and microphone access denied. Please allow access in your browser settings.";
      } else if (err.name === "NotFoundError") {
        error.value =
          "No camera or microphone found. Please check your device connections.";
      } else {
        error.value = `Failed to start video call: ${
          err.message || "Unknown error"
        }`;
      }

      status.value = "idle";
    }
  };

  // Connect to video call room
  const connectToCallRoom = async () => {
    try {
      // In a real app, this would connect to a signaling server
      roomConnection.value = await connectToRoom("room-123", localStream.value);

      // Setup event handlers for the connection
      roomConnection.value.onRemoteStream = (stream) => {
        remoteStream.value = stream;
        if (remoteVideoRef.value) {
          remoteVideoRef.value.srcObject = stream;
        }
      };

      roomConnection.value.onDisconnect = () => {
        status.value = "idle";
        error.value = "Call ended by the other participant.";
      };

      // For demo: simulate a remote stream after 2 seconds
      setTimeout(() => {
        if (remoteVideoRef.value && localStream.value) {
          // In a real app, this would be the remote peer's stream
          remoteVideoRef.value.srcObject = localStream.value;
        }
      }, 2000);
    } catch (err) {
      throw new Error(`Failed to connect to room: ${err.message}`);
    }
  };

  // Toggle mute
  const toggleMute = () => {
    if (localStream.value) {
      const audioTracks = localStream.value.getAudioTracks();

      if (audioTracks.length > 0) {
        audioTracks.forEach((track) => {
          track.enabled = !track.enabled;
        });

        isMuted.value = !audioTracks[0].enabled;
      }
    }
  };

  // Toggle video
  const toggleVideo = () => {
    if (localStream.value) {
      const videoTracks = localStream.value.getVideoTracks();

      if (videoTracks.length > 0) {
        videoTracks.forEach((track) => {
          track.enabled = !track.enabled;
        });

        isVideoOff.value = !videoTracks[0].enabled;
      }
    }
  };

  // End the call
  const endCall = () => {
    // Clean up streams
    cleanupMedia();

    // Disconnect from room
    if (roomConnection.value) {
      disconnectFromRoom(roomConnection.value);
      roomConnection.value = null;
    }

    status.value = "idle";
  };

  // Clean up media resources
  const cleanupMedia = () => {
    // Stop all tracks in the local stream
    if (localStream.value) {
      localStream.value.getTracks().forEach((track) => {
        track.stop();
      });
      localStream.value = null;
    }

    // Clear video elements
    if (localVideoRef.value) {
      localVideoRef.value.srcObject = null;
    }

    if (remoteVideoRef.value) {
      remoteVideoRef.value.srcObject = null;
    }

    remoteStream.value = null;
  };

  // Check if the browser supports WebRTC
  const checkBrowserSupport = () => {
    if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
      error.value =
        "Your browser does not support video calls. Please try a modern browser like Chrome, Firefox, or Safari.";
      return false;
    }
    return true;
  };

  // Lifecycle hooks
  onMounted(() => {
    // Check browser support
    if (checkBrowserSupport()) {
      // Initialize media on component mount
      initializeMedia();
    }

    // Listen for page visibility changes
    document.addEventListener("visibilitychange", handleVisibilityChange);
  });

  onBeforeUnmount(() => {
    // Clean up all resources when component is destroyed
    cleanupMedia();

    // Disconnect from room
    if (roomConnection.value) {
      disconnectFromRoom(roomConnection.value);
      roomConnection.value = null;
    }

    // Remove event listeners
    document.removeEventListener("visibilitychange", handleVisibilityChange);
  });

  // Handle errors in child components
  onErrorCaptured((err, instance, info) => {
    console.error("Error in child component:", err, info);

    // Show a user-friendly error
    error.value =
      "An error occurred in the application. Please try refreshing the page.";

    // Stop error propagation
    return false;
  });

  // Handle page visibility changes
  const handleVisibilityChange = () => {
    if (document.hidden) {
      // Page is hidden, might want to pause video or mute audio
      console.log("Page hidden, conserving resources");
    } else {
      // Page is visible again
      console.log("Page visible, resuming stream");
    }
  };
</script>
```

**Common Pitfalls:**

- Putting DOM manipulation in `onBeforeMount` (DOM isn't available yet)
- Not properly cleaning up resources in `onUnmounted`
- Using `onUpdated` for state updates, creating infinite loops
- Forgetting that lifecycle hooks don't have access to `this` in Composition API
- Not properly handling errors in lifecycle hooks

**Frequently Asked Questions:**

1. **Q: What is the equivalent of mounted, updated, etc. in the Composition API?**

   A: The Composition API provides direct function equivalents for each lifecycle hook:

   - `beforeCreate` / `created`: Not needed with `<script setup>` (happens automatically)
   - `beforeMount` → `onBeforeMount`
   - `mounted` → `onMounted`
   - `beforeUpdate` → `onBeforeUpdate`
   - `updated` → `onUpdated`
   - `beforeUnmount` → `onBeforeUnmount`
   - `unmounted` → `onUnmounted`
   - `errorCaptured` → `onErrorCaptured`
   - `activated` → `onActivated`
   - `deactivated` → `onDeactivated`

2. **Q: Why don't we need beforeCreate and created hooks in the Composition API?**

   A: With `<script setup>`, the code is automatically executed during component creation, which is effectively the same as using `created`. If you need a `beforeCreate` hook, you can just put code at the top of the `setup` function.

3. **Q: How do I handle async operations in lifecycle hooks?**

   A: Lifecycle hooks support async functions:

   ```js
   onMounted(async () => {
     try {
       const data = await fetchData();
       processData(data);
     } catch (error) {
       handleError(error);
     }
   });
   ```

   However, the component will continue rendering and won't wait for these promises to resolve. For suspending rendering, you would use Vue's Suspense feature.

### 5. Template Refs

**Concise Explanation:**
Template refs provide direct access to DOM elements or child components in Vue. They allow you to interact with the underlying DOM when declarative templates aren't sufficient, or to call methods on child component instances.

**Where to Use:**

- Focusing form elements programmatically
- Initializing third-party libraries that require DOM access
- Measuring element dimensions or positions
- Triggering animations or transitions imperatively
- Calling methods on child component instances

**Code Snippet:**

```html
<template>
  <div class="p-6 max-w-md mx-auto bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800 mb-4">
      Template Refs Example
    </h1>

    <!-- Form with input refs -->
    <form @submit.prevent="submitForm" class="space-y-4">
      <div>
        <label
          for="username"
          class="block text-sm font-medium text-gray-700 mb-1"
        >
          Username
        </label>
        <input
          id="username"
          ref="usernameInput"
          v-model="form.username"
          class="w-full px-4 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
        />
      </div>

      <div>
        <label for="bio" class="block text-sm font-medium text-gray-700 mb-1">
          Bio
        </label>
        <textarea
          id="bio"
          ref="bioTextarea"
          v-model="form.bio"
          rows="3"
          class="w-full px-4 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
        ></textarea>
        <div class="flex justify-between mt-1 text-sm text-gray-500">
          <span>{{ form.bio.length }} / {{ maxBioLength }} characters</span>
          <button
            type="button"
            @click="expandTextarea"
            class="text-blue-600 hover:text-blue-800"
          >
            Expand
          </button>
        </div>
      </div>

      <!-- Image upload with element ref -->
      <div>
        <label class="block text-sm font-medium text-gray-700 mb-1">
          Profile Image
        </label>
        <div
          ref="dropZone"
          class="border-2 border-dashed border-gray-300 rounded p-6 text-center hover:bg-gray-50 cursor-pointer"
          @click="triggerFileInput"
          @dragover.prevent="highlightDropZone"
          @dragleave.prevent="resetDropZone"
          @drop.prevent="handleFileDrop"
        >
          <input
            type="file"
            ref="fileInput"
            class="hidden"
            accept="image/*"
            @change="handleFileSelect"
          />

          <div v-if="uploadedImage">
            <img
              :src="uploadedImage"
              alt="Uploaded preview"
              class="mx-auto h-32 object-cover rounded"
            />
            <button
              type="button"
              @click.stop="removeImage"
              class="mt-2 text-red-600 hover:text-red-800"
            >
              Remove
            </button>
          </div>

          <div v-else>
            <svg
              class="mx-auto h-12 w-12 text-gray-400"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M3 16.5v2.25A2.25 2.25 0 005.25 21h13.5A2.25 2.25 0 0021 18.75V16.5m-13.5-9L12 3m0 0l4.5 4.5M12 3v13.5"
              />
            </svg>
            <p class="mt-1 text-sm text-gray-500">
              Drag and drop an image here, or click to select
            </p>
          </div>
        </div>
      </div>

      <!-- Canvas element with ref -->
      <div>
        <label class="block text-sm font-medium text-gray-700 mb-1">
          Signature (draw below)
        </label>
        <canvas
          ref="signatureCanvas"
          class="border rounded w-full h-32 touch-none"
          @mousedown="startDrawing"
          @mousemove="draw"
          @mouseup="stopDrawing"
          @mouseleave="stopDrawing"
          @touchstart="startDrawing"
          @touchmove="draw"
          @touchend="stopDrawing"
        ></canvas>
        <div class="flex justify-end mt-2">
          <button
            type="button"
            @click="clearSignature"
            class="px-3 py-1 text-sm text-red-600 hover:text-red-800"
          >
            Clear
          </button>
        </div>
      </div>

      <!-- Chart with ref for third-party library -->
      <div>
        <label class="block text-sm font-medium text-gray-700 mb-1">
          Activity Chart
        </label>
        <div
          ref="chartContainer"
          class="border rounded w-full h-48 bg-gray-50"
        ></div>
      </div>

      <!-- Child component with ref -->
      <CustomButton
        ref="submitButtonRef"
        label="Submit"
        :loading="submitting"
        @click="triggerSubmit"
      />
    </form>

    <!-- Element dimensions demo -->
    <div class="mt-8 p-4 bg-gray-50 rounded">
      <h2 class="text-lg font-medium mb-2">Element Dimensions</h2>
      <div
        ref="measureElement"
        class="p-4 bg-blue-100 border border-blue-300 rounded"
      >
        This element's dimensions are being measured
      </div>
      <p class="mt-2 text-sm">
        Width: {{ elementWidth }}px, Height: {{ elementHeight }}px
      </p>
    </div>

    <!-- Multiple refs using v-for -->
    <div class="mt-8">
      <h2 class="text-lg font-medium mb-2">Multiple Element Refs</h2>
      <div class="grid grid-cols-3 gap-2">
        <button
          v-for="(color, index) in colors"
          :key="color"
          :ref="
            (el) => {
              if (el) colorButtons[index] = el;
            }
          "
          @click="highlightButton(index)"
          class="p-2 rounded"
          :class="`bg-${color}-100 hover:bg-${color}-200`"
        >
          {{ color }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
  import { ref, reactive, onMounted, onUnmounted, nextTick } from "vue";
  import CustomButton from "./CustomButton.vue";
  import ChartLibrary from "chart-library"; // Simulated third-party library

  // Form state
  const form = reactive({
    username: "",
    bio: "",
    image: null,
  });

  // Constants
  const maxBioLength = 200;
  const submitting = ref(false);

  // Template refs
  const usernameInput = ref(null);
  const bioTextarea = ref(null);
  const fileInput = ref(null);
  const dropZone = ref(null);
  const signatureCanvas = ref(null);
  const chartContainer = ref(null);
  const submitButtonRef = ref(null);
  const measureElement = ref(null);

  // Canvas state
  const isDrawing = ref(false);
  const canvasContext = ref(null);
  const lastX = ref(0);
  const lastY = ref(0);

  // Other state
  const uploadedImage = ref(null);
  const elementWidth = ref(0);
  const elementHeight = ref(0);
  const colors = ["red", "blue", "green", "yellow", "purple", "pink"];
  const colorButtons = ref([]);

  // Initialize after component is mounted
  onMounted(() => {
    // Auto focus the username input
    usernameInput.value.focus();

    // Initialize canvas
    initializeCanvas();

    // Initialize chart with mock data
    initializeChart();

    // Measure the element dimensions
    updateElementDimensions();

    // Set up resize observer to update dimensions when the element changes
    const resizeObserver = new ResizeObserver(updateElementDimensions);
    resizeObserver.observe(measureElement.value);

    // Set up window resize event for responsive elements
    window.addEventListener("resize", handleResize);

    // Cleanup function (will be called in onUnmounted)
    onUnmounted(() => {
      resizeObserver.disconnect();
      window.removeEventListener("resize", handleResize);
    });
  });

  // Initialize canvas for drawing
  const initializeCanvas = () => {
    if (!signatureCanvas.value) return;

    const canvas = signatureCanvas.value;
    const ctx = canvas.getContext("2d");

    // Set canvas dimensions accounting for device pixel ratio
    const dpr = window.devicePixelRatio || 1;
    const rect = canvas.getBoundingClientRect();

    canvas.width = rect.width * dpr;
    canvas.height = rect.height * dpr;

    // Scale the context for high-DPI displays
    ctx.scale(dpr, dpr);

    // Set drawing style
    ctx.lineJoin = "round";
    ctx.lineCap = "round";
    ctx.lineWidth = 2;
    ctx.strokeStyle = "#000000";

    canvasContext.value = ctx;
  };

  // Canvas drawing functions
  const startDrawing = (e) => {
    isDrawing.value = true;

    // Get coordinates from mouse or touch event
    const { x, y } = getEventCoordinates(e);

    lastX.value = x;
    lastY.value = y;
  };

  const draw = (e) => {
    if (!isDrawing.value) return;

    // Prevent scrolling when drawing on touch devices
    e.preventDefault();

    const ctx = canvasContext.value;
    const { x, y } = getEventCoordinates(e);

    ctx.beginPath();
    ctx.moveTo(lastX.value, lastY.value);
    ctx.lineTo(x, y);
    ctx.stroke();

    lastX.value = x;
    lastY.value = y;
  };

  const stopDrawing = () => {
    isDrawing.value = false;
  };

  const clearSignature = () => {
    const canvas = signatureCanvas.value;
    if (!canvas) return;

    const ctx = canvasContext.value;
    ctx.clearRect(0, 0, canvas.width, canvas.height);
  };

  const getEventCoordinates = (e) => {
    const canvas = signatureCanvas.value;
    const rect = canvas.getBoundingClientRect();

    // Handle both mouse and touch events
    const clientX = e.touches ? e.touches[0].clientX : e.clientX;
    const clientY = e.touches ? e.touches[0].clientY : e.clientY;

    return {
      x: clientX - rect.left,
      y: clientY - rect.top,
    };
  };

  // Initialize chart with third-party library
  const initializeChart = () => {
    if (!chartContainer.value) return;

    // In a real app, this would use an actual charting library
    // For this example, we'll simulate a chart library
    const chart = new ChartLibrary(chartContainer.value, {
      type: "bar",
      data: [12, 19, 8, 5, 15, 10],
      labels: ["Jan", "Feb", "Mar", "Apr", "May", "Jun"],
      colors: ["#3B82F6"],
    });
  };

  // Form methods
  const expandTextarea = () => {
    if (!bioTextarea.value) return;

    // Increase the number of rows
    bioTextarea.value.rows = bioTextarea.value.rows + 2;
  };

  const triggerFileInput = () => {
    if (!fileInput.value) return;

    fileInput.value.click();
  };

  const handleFileSelect = (e) => {
    const file = e.target.files[0];
    if (!file) return;

    processSelectedFile(file);
  };

  const handleFileDrop = (e) => {
    e.preventDefault();

    // Get the dropped file
    const file = e.dataTransfer.files[0];
    if (!file || !file.type.startsWith("image/")) return;

    processSelectedFile(file);
    resetDropZone();
  };

  const processSelectedFile = (file) => {
    // Store the file in the form state
    form.image = file;

    // Create a preview URL
    uploadedImage.value = URL.createObjectURL(file);
  };

  const highlightDropZone = () => {
    if (!dropZone.value) return;

    dropZone.value.classList.add("border-blue-500");
    dropZone.value.classList.add("bg-blue-50");
  };

  const resetDropZone = () => {
    if (!dropZone.value) return;

    dropZone.value.classList.remove("border-blue-500");
    dropZone.value.classList.remove("bg-blue-50");
  };

  const removeImage = () => {
    uploadedImage.value = null;
    form.image = null;

    // Reset the file input
    if (fileInput.value) {
      fileInput.value.value = "";
    }
  };

  // Element measurements
  const updateElementDimensions = () => {
    if (!measureElement.value) return;

    elementWidth.value = measureElement.value.offsetWidth;
    elementHeight.value = measureElement.value.offsetHeight;
  };

  // Handle resize events
  const handleResize = () => {
    // Reinitialize canvas dimensions
    initializeCanvas();

    // Update element measurements
    updateElementDimensions();

    // Refresh chart if needed
    initializeChart();
  };

  // Multiple ref elements
  const highlightButton = (index) => {
    // Reset all buttons
    colorButtons.value.forEach((button) => {
      button.classList.remove("ring-2", "ring-offset-2", "ring-blue-500");
    });

    // Highlight the clicked button
    if (colorButtons.value[index]) {
      colorButtons.value[index].classList.add(
        "ring-2",
        "ring-offset-2",
        "ring-blue-500"
      );
    }
  };

  // Form submission
  const submitForm = () => {
    submitting.value = true;

    // Simulate API call
    setTimeout(() => {
      console.log("Form submitted:", form);
      submitting.value = false;

      // Reset form
      form.username = "";
      form.bio = "";
      form.image = null;
      uploadedImage.value = null;
      clearSignature();

      // Focus back on the username input
      usernameInput.value.focus();
    }, 1500);
  };

  // Method called through the custom button ref
  const triggerSubmit = () => {
    submitForm();
  };
</script>
```

**Real-World Example:**
In a content management system's WYSIWYG editor, template refs access the editor container to initialize a rich text editor library. The editor includes toolbars, formatting controls, and image upload capabilities, all requiring direct DOM access through refs.

```html
<template>
  <div class="p-6 max-w-4xl mx-auto bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800 mb-4">Content Editor</h1>

    <!-- Editor toolbar with button refs -->
    <div class="border-b pb-2 mb-4 flex flex-wrap gap-2">
      <button
        v-for="(tool, index) in editorTools"
        :key="tool.command"
        :ref="
          (el) => {
            if (el) toolbarButtons[index] = el;
          }
        "
        @click="executeCommand(tool.command, tool.value)"
        class="p-2 rounded hover:bg-gray-100"
        :title="tool.label"
      >
        <span v-html="tool.icon"></span>
      </button>
    </div>

    <!-- Rich text editor with ref -->
    <div
      ref="editorRef"
      contenteditable="true"
      class="border rounded p-4 min-h-48 focus:outline-none focus:ring-2 focus:ring-blue-500"
      @input="handleEditorInput"
      @paste="handlePaste"
      @keydown="handleKeyDown"
    ></div>

    <!-- Character count -->
    <div class="mt-2 text-sm text-gray-500 flex justify-between">
      <span>{{ characterCount }} characters</span>
      <button @click="insertImage" class="text-blue-600 hover:text-blue-800">
        Insert Image
      </button>
    </div>

    <!-- Hidden file input for image upload -->
    <input
      type="file"
      ref="imageInputRef"
      class="hidden"
      accept="image/*"
      @change="processImageUpload"
    />

    <!-- Preview with rendered HTML -->
    <div class="mt-8">
      <h2 class="text-lg font-medium mb-2">Preview</h2>
      <div
        ref="previewRef"
        class="border rounded p-4 bg-gray-50 prose"
        v-html="sanitizedContent"
      ></div>
    </div>
  </div>
</template>

<script setup>
  import { ref, reactive, computed, onMounted, onBeforeUnmount } from "vue";
  import DOMPurify from "dompurify"; // For sanitizing HTML

  // Editor state
  const editorRef = ref(null);
  const previewRef = ref(null);
  const imageInputRef = ref(null);
  const content = ref("");
  const toolbarButtons = ref([]);
  const selectedRange = ref(null);

  // Editor tools
  const editorTools = [
    { command: "bold", label: "Bold", icon: "<strong>B</strong>" },
    { command: "italic", label: "Italic", icon: "<em>I</em>" },
    { command: "underline", label: "Underline", icon: "<u>U</u>" },
    { command: "formatBlock", label: "Heading", value: "h2", icon: "H2" },
    { command: "formatBlock", label: "Paragraph", value: "p", icon: "P" },
    { command: "insertUnorderedList", label: "Bullet List", icon: "&bull;" },
    { command: "insertOrderedList", label: "Numbered List", icon: "1." },
    { command: "createLink", label: "Link", icon: "🔗" },
    { command: "justifyLeft", label: "Align Left", icon: "←" },
    { command: "justifyCenter", label: "Align Center", icon: "↔" },
    { command: "justifyRight", label: "Align Right", icon: "→" },
  ];

  // Computed props
  const characterCount = computed(() => {
    // Strip HTML tags to count only text characters
    const div = document.createElement("div");
    div.innerHTML = content.value;
    return div.textContent?.length || 0;
  });

  const sanitizedContent = computed(() => {
    // Sanitize HTML to prevent XSS
    return DOMPurify.sanitize(content.value);
  });

  // Initialize editor
  onMounted(() => {
    // Set initial content
    if (editorRef.value) {
      editorRef.value.innerHTML = "<p>Start typing your content here...</p>";
      content.value = editorRef.value.innerHTML;
    }

    // Add selection change listener to track cursor position
    document.addEventListener("selectionchange", saveSelection);
  });

  // Cleanup
  onBeforeUnmount(() => {
    document.removeEventListener("selectionchange", saveSelection);
  });

  // Editor methods
  const handleEditorInput = () => {
    if (!editorRef.value) return;
    content.value = editorRef.value.innerHTML;
  };

  const executeCommand = (command, value = null) => {
    // Restore selection if it exists
    restoreSelection();

    // Execute command
    if (command === "createLink") {
      const url = prompt("Enter the URL:");
      if (url) {
        document.execCommand(command, false, url);
      }
    } else {
      document.execCommand(command, false, value);
    }

    // Update content
    if (editorRef.value) {
      content.value = editorRef.value.innerHTML;
    }

    // Focus back on editor
    editorRef.value?.focus();
  };

  const saveSelection = () => {
    if (document.activeElement === editorRef.value) {
      const selection = window.getSelection();
      if (selection.rangeCount > 0) {
        selectedRange.value = selection.getRangeAt(0);
      }
    }
  };

  const restoreSelection = () => {
    if (selectedRange.value) {
      const selection = window.getSelection();
      selection.removeAllRanges();
      selection.addRange(selectedRange.value);
    }

    // Focus the editor
    editorRef.value?.focus();
  };

  const handlePaste = (e) => {
    // Prevent default paste behavior
    e.preventDefault();

    // Get plain text from clipboard
    const text = e.clipboardData.getData("text/plain");

    // Insert text at cursor position
    document.execCommand("insertText", false, text);
  };

  const handleKeyDown = (e) => {
    // Implement keyboard shortcuts
    if (e.ctrlKey || e.metaKey) {
      switch (e.key.toLowerCase()) {
        case "b":
          e.preventDefault();
          executeCommand("bold");
          break;
        case "i":
          e.preventDefault();
          executeCommand("italic");
          break;
        case "u":
          e.preventDefault();
          executeCommand("underline");
          break;
      }
    }
  };

  const insertImage = () => {
    // Save current selection
    saveSelection();

    // Trigger file input
    imageInputRef.value?.click();
  };

  const processImageUpload = (e) => {
    const file = e.target.files[0];
    if (!file) return;

    // Create a URL for the image
    const reader = new FileReader();
    reader.onload = (event) => {
      // Restore selection
      restoreSelection();

      // Insert image at cursor position
      document.execCommand("insertImage", false, event.target.result);

      // Update content
      if (editorRef.value) {
        content.value = editorRef.value.innerHTML;
      }

      // Reset file input
      if (imageInputRef.value) {
        imageInputRef.value.value = "";
      }
    };

    reader.readAsDataURL(file);
  };
</script>
```

**Common Pitfalls:**

- Accessing refs before they're available (e.g., before `onMounted`)
- Not handling cases where refs might be null or undefined
- Over-relying on refs for tasks that should be handled declaratively
- Not properly cleaning up refs when components unmount
- Using `.value` incorrectly when accessing ref elements

**Frequently Asked Questions:**

1. **Q: When are template refs populated in the component lifecycle?**

   A: Template refs are set after the component's DOM is mounted. They will be `null` during `onBeforeMount`, but available in `onMounted` and later hooks. Always check if a ref exists before using it, especially in functions that might be called before mounting.

2. **Q: How do I handle multiple elements with refs in v-for loops?**

   A: For Vue 3, use a function as the ref value:

   ```html
   <div
     v-for="(item, index) in items"
     :key="item.id"
     :ref="el => { if (el) itemRefs[index] = el }"
   >
     {{ item.name }}
   </div>
   ```

   And initialize the array in setup:

   ```js
   const itemRefs = ref([]);
   ```

3. **Q: How do I access methods on a child component using refs?**

   A: After getting a reference to the child component, you can call its methods directly:

   ```js
   // Child component
   const modal = ref(null);

   // Call a method on the child
   const showModal = () => {
     modal.value.open();
   };
   ```

   In Vue 3, for component methods to be accessible via refs, they need to be exposed using `defineExpose` in the child component:

   ```js
   // In child component
   defineExpose({
     open,
     close,
     reset,
   });
   ```

### 6. Provide/Inject

**Concise Explanation:**
Provide/Inject is a dependency injection system in Vue that allows a parent component to "provide" data that any of its descendant components can "inject," no matter how deep in the component tree. This avoids prop drilling (passing props through multiple intermediate components).

**Where to Use:**

- Sharing data with deeply nested components
- Theme configuration
- User authentication and authorization context
- Application-wide preferences
- Plugin systems where components need shared functionality

**Code Snippet:**

```html
<!-- App.vue (Root component providing values) -->
<template>
  <div class="min-h-screen" :class="themeClasses">
    <header class="p-4 bg-white dark:bg-gray-800 shadow">
      <div class="max-w-6xl mx-auto flex justify-between items-center">
        <h1 class="text-xl font-bold text-gray-900 dark:text-white">
          Vue Provide/Inject Example
        </h1>

        <div class="flex items-center space-x-4">
          <!-- Theme toggle -->
          <button
            @click="toggleTheme"
            class="p-2 rounded-full bg-gray-100 dark:bg-gray-700"
          >
            <span v-if="theme === 'dark'" class="text-yellow-500">☀️</span>
            <span v-else class="text-gray-700">🌙</span>
          </button>

          <!-- User menu -->
          <div class="relative">
            <button
              @click="userMenuOpen = !userMenuOpen"
              class="flex items-center space-x-2"
            >
              <img
                :src="currentUser.avatar"
                alt="User avatar"
                class="w-8 h-8 rounded-full"
              />
              <span class="text-gray-900 dark:text-white"
                >{{ currentUser.name }}</span
              >
            </button>

            <div
              v-if="userMenuOpen"
              class="absolute right-0 mt-2 w-48 bg-white dark:bg-gray-800 shadow-lg rounded z-10"
            >
              <div class="p-2 border-b border-gray-200 dark:border-gray-700">
                <p class="text-sm text-gray-500 dark:text-gray-400">
                  Signed in as
                </p>
                <p class="font-medium">{{ currentUser.email }}</p>
              </div>
              <div class="p-1">
                <button
                  @click="logout"
                  class="w-full text-left px-4 py-2 text-sm text-red-600 rounded hover:bg-gray-100 dark:hover:bg-gray-700"
                >
                  Sign out
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </header>

    <main class="max-w-6xl mx-auto p-4">
      <!-- Content sections -->
      <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
        <MainContent />
        <Sidebar />
      </div>
    </main>
  </div>
</template>

<script setup>
  import { ref, computed, provide } from "vue";
  import MainContent from "./MainContent.vue";
  import Sidebar from "./Sidebar.vue";

  // Theme state
  const theme = ref(localStorage.getItem("theme") || "light");
  const themeClasses = computed(() => {
    return theme.value === "dark" ? "dark bg-gray-900" : "bg-gray-50";
  });

  // User state
  const currentUser = ref({
    id: 1,
    name: "Jane Doe",
    email: "jane.doe@example.com",
    avatar: "https://ui-avatars.com/api/?name=Jane+Doe",
    role: "admin",
    preferences: {
      notifications: true,
      newsletter: false,
    },
  });
  const userMenuOpen = ref(false);
  const isAuthenticated = ref(true);

  // API base URL
  const apiBaseUrl = "https://api.example.com/v1";

  // Methods
  const toggleTheme = () => {
    theme.value = theme.value === "light" ? "dark" : "light";
    localStorage.setItem("theme", theme.value);
  };

  const logout = () => {
    isAuthenticated.value = false;
    userMenuOpen.value = false;
    // In a real app, this would call an auth service logout method
  };

  // Provide values to descendant components
  // Read-only values
  provide("apiBaseUrl", apiBaseUrl);

  // Reactive values
  provide("theme", theme);
  provide("currentUser", currentUser);
  provide("isAuthenticated", isAuthenticated);

  // Functions
  provide("toggleTheme", toggleTheme);
  provide("logout", logout);

  // Complex reactive object with methods
  provide("userService", {
    user: currentUser,
    updateProfile: (data) => {
      Object.assign(currentUser.value, data);
    },
    updatePreferences: (preferences) => {
      currentUser.value.preferences = {
        ...currentUser.value.preferences,
        ...preferences,
      };
    },
  });
</script>
```

```html
<!-- MainContent.vue (Intermediate component) -->
<template>
  <div class="col-span-2">
    <div class="bg-white dark:bg-gray-800 p-6 rounded-lg shadow">
      <h2 class="text-xl font-semibold mb-4 text-gray-900 dark:text-white">
        Dashboard
      </h2>

      <!-- User greeting that uses injected values -->
      <div class="mb-6 p-4 bg-blue-50 dark:bg-blue-900 rounded">
        <p class="text-blue-800 dark:text-blue-200">
          Welcome back, {{ currentUser.name }}!
        </p>
      </div>

      <!-- Children components that will inject values -->
      <UserProfile />
      <FeatureList />
    </div>
  </div>
</template>

<script setup>
  import { inject } from "vue";
  import UserProfile from "./UserProfile.vue";
  import FeatureList from "./FeatureList.vue";

  // Inject values from ancestor component
  const currentUser = inject("currentUser");
</script>
```

```html
<!-- UserProfile.vue (Deeply nested component) -->
<template>
  <div class="mb-6 p-4 border border-gray-200 dark:border-gray-700 rounded">
    <h3 class="text-lg font-medium mb-2 text-gray-900 dark:text-white">
      Your Profile
    </h3>

    <!-- Display user information with injected values -->
    <div class="flex items-start space-x-4">
      <img
        :src="currentUser.avatar"
        alt="User avatar"
        class="w-16 h-16 rounded-full"
      />

      <div>
        <p class="font-medium text-gray-900 dark:text-white">
          {{ currentUser.name }}
        </p>
        <p class="text-gray-600 dark:text-gray-400">{{ currentUser.email }}</p>
        <p class="text-sm text-gray-500 dark:text-gray-500">
          Role: {{ currentUser.role }}
        </p>
      </div>
    </div>

    <!-- Edit profile form -->
    <div class="mt-4">
      <h4 class="text-sm font-medium mb-2 text-gray-700 dark:text-gray-300">
        Update Profile
      </h4>

      <div class="space-y-3">
        <div>
          <label class="block text-sm text-gray-600 dark:text-gray-400 mb-1">
            Display Name
          </label>
          <input
            v-model="profileForm.name"
            class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded bg-white dark:bg-gray-700 text-gray-900 dark:text-white"
          />
        </div>

        <div>
          <label class="block text-sm text-gray-600 dark:text-gray-400 mb-1">
            Email
          </label>
          <input
            v-model="profileForm.email"
            class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded bg-white dark:bg-gray-700 text-gray-900 dark:text-white"
          />
        </div>

        <div class="flex justify-end">
          <button
            @click="updateProfile"
            class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
          >
            Save Changes
          </button>
        </div>
      </div>
    </div>

    <!-- Preferences section -->
    <div class="mt-6 pt-4 border-t border-gray-200 dark:border-gray-700">
      <h4 class="text-sm font-medium mb-2 text-gray-700 dark:text-gray-300">
        Preferences
      </h4>

      <div class="space-y-2">
        <label class="flex items-center">
          <input
            type="checkbox"
            v-model="preferences.notifications"
            class="h-4 w-4 text-blue-600 rounded"
          />
          <span class="ml-2 text-gray-700 dark:text-gray-300">
            Enable notifications
          </span>
        </label>

        <label class="flex items-center">
          <input
            type="checkbox"
            v-model="preferences.newsletter"
            class="h-4 w-4 text-blue-600 rounded"
          />
          <span class="ml-2 text-gray-700 dark:text-gray-300">
            Subscribe to newsletter
          </span>
        </label>
      </div>

      <div class="mt-3">
        <button
          @click="updatePreferences"
          class="px-4 py-2 bg-gray-200 dark:bg-gray-600 text-gray-800 dark:text-white rounded hover:bg-gray-300 dark:hover:bg-gray-700"
        >
          Save Preferences
        </button>
      </div>
    </div>

    <!-- Theme toggle -->
    <div class="mt-6 pt-4 border-t border-gray-200 dark:border-gray-700">
      <div class="flex items-center justify-between">
        <span class="text-gray-700 dark:text-gray-300">
          Current theme: {{ theme === "dark" ? "Dark" : "Light" }}
        </span>

        <button
          @click="toggleTheme"
          class="px-4 py-2 bg-gray-200 dark:bg-gray-600 text-gray-800 dark:text-white rounded hover:bg-gray-300 dark:hover:bg-gray-700"
        >
          Switch to {{ theme === "dark" ? "Light" : "Dark" }} Mode
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
  import { inject, reactive, computed } from "vue";

  // Inject values from ancestor components
  const currentUser = inject("currentUser");
  const theme = inject("theme");
  const toggleTheme = inject("toggleTheme");
  const userService = inject("userService");

  // Local state
  const profileForm = reactive({
    name: "",
    email: "",
  });

  // Initialize form with current user data
  profileForm.name = currentUser.value.name;
  profileForm.email = currentUser.value.email;

  // Preferences with two-way binding to user preferences
  const preferences = reactive({ ...currentUser.value.preferences });

  // Methods
  const updateProfile = () => {
    userService.updateProfile({
      name: profileForm.name,
      email: profileForm.email,
    });

    // Show success notification (in a real app)
    alert("Profile updated successfully");
  };

  const updatePreferences = () => {
    userService.updatePreferences(preferences);

    // Show success notification (in a real app)
    alert("Preferences updated successfully");
  };
</script>
```

```html
<!-- Sidebar.vue (Another branch of the component tree) -->
<template>
  <div class="bg-white dark:bg-gray-800 p-6 rounded-lg shadow">
    <h2 class="text-xl font-semibold mb-4 text-gray-900 dark:text-white">
      App Info
    </h2>

    <!-- API information using injected value -->
    <div class="mb-4 p-3 bg-gray-100 dark:bg-gray-700 rounded text-sm">
      <p class="text-gray-700 dark:text-gray-300">
        API Base URL: {{ apiBaseUrl }}
      </p>
    </div>

    <!-- Authentication status -->
    <div class="mb-4">
      <div class="flex items-center">
        <div
          class="w-3 h-3 rounded-full mr-2"
          :class="isAuthenticated ? 'bg-green-500' : 'bg-red-500'"
        ></div>
        <span class="text-gray-700 dark:text-gray-300">
          {{ isAuthenticated ? "Authenticated" : "Not authenticated" }}
        </span>
      </div>
    </div>

    <!-- Logout button -->
    <button
      v-if="isAuthenticated"
      @click="logout"
      class="w-full px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700"
    >
      Sign Out
    </button>
  </div>
</template>

<script setup>
  import { inject } from "vue";

  // Inject values from ancestor components
  const apiBaseUrl = inject("apiBaseUrl");
  const isAuthenticated = inject("isAuthenticated");
  const logout = inject("logout");
</script>
```

**Real-World Example:**
An e-commerce application provides a shopping cart service at the root level, which any product component can inject to add items. This avoids passing cart methods through multiple layers of components and centralizes the cart logic.

```html
<!-- CartProvider.vue -->
<script setup>
  import { ref, provide, computed } from "vue";

  // Cart state
  const cartItems = ref([]);
  const isCartOpen = ref(false);

  // Computed properties
  const cartCount = computed(() => {
    return cartItems.value.reduce((count, item) => count + item.quantity, 0);
  });

  const cartTotal = computed(() => {
    return cartItems.value
      .reduce((total, item) => {
        return total + item.price * item.quantity;
      }, 0)
      .toFixed(2);
  });

  // Cart methods
  const addToCart = (product, quantity = 1) => {
    const existingItem = cartItems.value.find((item) => item.id === product.id);

    if (existingItem) {
      existingItem.quantity += quantity;
    } else {
      cartItems.value.push({
        ...product,
        quantity,
      });
    }

    // Show cart briefly
    isCartOpen.value = true;
    setTimeout(() => {
      isCartOpen.value = false;
    }, 3000);
  };

  const removeFromCart = (productId) => {
    const index = cartItems.value.findIndex((item) => item.id === productId);
    if (index !== -1) {
      cartItems.value.splice(index, 1);
    }
  };

  const updateQuantity = (productId, quantity) => {
    const item = cartItems.value.find((item) => item.id === productId);
    if (item) {
      item.quantity = Math.max(1, quantity);
    }
  };

  const clearCart = () => {
    cartItems.value = [];
  };

  const toggleCart = () => {
    isCartOpen.value = !isCartOpen.value;
  };

  // Provide cart service to all descendant components
  provide("cartService", {
    // State
    items: cartItems,
    isOpen: isCartOpen,
    count: cartCount,
    total: cartTotal,

    // Methods
    addToCart,
    removeFromCart,
    updateQuantity,
    clearCart,
    toggleCart,
  });
</script>

<template>
  <slot></slot>
</template>
```

```html
<!-- ProductCard.vue (deeply nested component) -->
<template>
  <div
    class="border rounded overflow-hidden shadow-sm hover:shadow-md transition-shadow"
  >
    <img
      :src="product.image"
      :alt="product.name"
      class="w-full h-48 object-cover"
    />

    <div class="p-4">
      <h3 class="font-medium text-gray-900">{{ product.name }}</h3>
      <p class="text-gray-500 text-sm">{{ product.category }}</p>

      <div class="mt-2 flex items-center justify-between">
        <span class="font-bold">${{ product.price.toFixed(2) }}</span>
        <button
          @click="addToCart(product)"
          class="px-3 py-1 bg-blue-600 text-white rounded hover:bg-blue-700"
        >
          Add to Cart
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
  import { inject } from "vue";

  const props = defineProps({
    product: {
      type: Object,
      required: true,
    },
  });

  // Inject cart service from ancestor component
  const { addToCart } = inject("cartService");
</script>
```

**Common Pitfalls:**

- Not providing default values for injected properties
- Making values non-reactive when providing them
- Creating circular dependencies between providers
- Overusing provide/inject for data that should be passed as props
- Modifying provided values directly in child components

**Frequently Asked Questions:**

1. **Q: How do I make sure injected values are reactive?**

   A: When providing values, make sure to use refs or reactive objects:

   ```js
   // Reactive provide
   const theme = ref("light");
   provide("theme", theme); // Pass the ref directly

   // Non-reactive provide (avoid this)
   provide("theme", "light"); // Passing a string directly
   ```

   And when injecting, don't destructure reactive objects:

   ```js
   // Correct (preserves reactivity)
   const user = inject("user");
   console.log(user.value.name); // Accessing through .value

   // Incorrect (breaks reactivity)
   const { name, email } = inject("user").value;
   ```

2. **Q: How do I provide a default value for injection?**

   A: You can provide a default value as the second argument to `inject`:

   ```js
   // With default
   const theme = inject("theme", "light");

   // With default factory function (for complex defaults)
   const user = inject("user", () => ({
     name: "Guest",
     role: "visitor",
   }));
   ```

3. **Q: Is it better to provide a value or a function to update that value?**

   A: For most cases, provide both the value and functions to update it, especially if multiple components need to modify the value:

   ```js
   // In parent
   const count = ref(0);

   const increment = () => {
     count.value++;
   };

   provide("counter", {
     count,
     increment,
   });

   // In child
   const { count, increment } = inject("counter");
   ```

   This creates a clear API for modifying shared state while maintaining control in the providing component.

## Next Actions: Exercises and Micro-Projects

### 1. Reactive State Management Exercise

Create a to-do application using refs and reactive for state management. Implement features like adding, editing, completing, and filtering tasks. Use computed properties for filtered views and watchers to sync with localStorage.

### 2. Composition API Lifecycle Practice

Build a photo gallery component that fetches images from an API in onMounted, displays them in a grid, and supports pagination. Use lifecycle hooks to clean up resources and handle loading states.

### 3. Template Refs Project

Create a custom form component library with advanced input validation, including a credit card input with automatic formatting, a color picker, and a rich text editor. Use template refs to interact with the DOM elements directly.

### 4. Provide/Inject Exercise

Build a theme system that provides theme variables (colors, fonts, spacing) throughout the application. Include a theme switcher component and persist the user's preference in localStorage.

### 5. Complex State Management Challenge

Create a multi-step form wizard that maintains state across steps, validates each step, and allows navigating between completed steps. Use composition API features to organize the complex state.

### 6. Real-time Dashboard with Watchers

Build a dashboard that displays real-time data, using watchers to update charts and visualizations when the data changes. Include a simulation of data updates with different update frequencies.

## Success Criteria

You've mastered Module 3: Reactive State Management when you can:

1. **Choose Appropriate Reactivity Primitives** - Select the right tool (ref vs. reactive) for different state management needs and understand their differences.

2. **Create Derived State** - Efficiently build computed properties that update automatically when dependencies change.

3. **React to State Changes** - Implement watchers correctly for side effects, with appropriate options for different scenarios.

4. **Manage Component Lifecycle** - Use lifecycle hooks at the correct times for initialization, updates, and cleanup.

5. **Access the DOM When Needed** - Use template refs appropriately and safely when direct DOM access is necessary.

6. **Share State Across Components** - Implement provide/inject patterns to avoid prop drilling while maintaining clear data flow.

## Troubleshooting Guide

### Reactivity Issues

- **Problem**: Changes to state not reflecting in the UI.
  **Solution**: Ensure you're using `.value` when modifying refs in JavaScript code, not destructuring reactive objects, and making proper reactive state updates.

- **Problem**: Losing reactivity after passing state to components.
  **Solution**: Pass the entire ref or reactive object instead of destructuring it first. Use toRefs if you need to destructure.

### Computed Properties Issues

- **Problem**: Computed property not updating when dependencies change.
  **Solution**: Verify all dependencies are truly reactive and that the computed property is accessing reactive values in their reactive form (e.g., `user.value.name` not just a stored `name`).

- **Problem**: Infinite loop with computed properties.
  **Solution**: Ensure computed properties don't modify state or trigger side effects. They should be pure functions that only calculate values.

### Watcher Issues

- **Problem**: Watcher not firing when expected.
  **Solution**: Check if you need the `deep` option for watching nested objects, or `immediate` to run on initialization.

- **Problem**: Watchers causing performance issues.
  **Solution**: Consider using debounce or throttle to limit how often watchers trigger, especially for expensive operations or input-related watchers.

### Lifecycle Hook Issues

- **Problem**: Trying to access DOM before it's available.
  **Solution**: Move DOM manipulation from `onBeforeMount` to `onMounted` where the DOM is guaranteed to be available.

- **Problem**: Memory leaks from event listeners or subscriptions.
  **Solution**: Ensure all setup in `onMounted` is cleaned up in `onUnmounted`, like removing event listeners, canceling subscriptions, and stopping timers.

### Template Refs Issues

- **Problem**: Template ref is null when trying to use it.
  **Solution**: Make sure you're accessing refs after the component is mounted. Use `nextTick` if needed to ensure DOM updates have completed.

- **Problem**: Can't access component methods through refs.
  **Solution**: Make sure the component is using `defineExpose` to expose the methods you need to access.

### Provide/Inject Issues

- **Problem**: Injected value is undefined.
  **Solution**: Verify the ancestor component actually provides the value, and consider adding a default value to the inject call. Check for typos in the injection key.

- **Problem**: Changes to provided values not updating in injected components.
  **Solution**: Ensure you're providing reactive values (refs or reactive objects), not primitive values that lose their reactivity.
