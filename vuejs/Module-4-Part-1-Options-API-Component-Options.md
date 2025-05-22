# Module 4: Options API - Part 1: Component Options

## Core Problem

The Options API provides a structured way to organize component logic by type (data, methods, computed, etc.), making it easy to understand component architecture at a glance. While the Composition API offers more flexibility, the Options API remains widely used in existing codebases and provides a more approachable entry point for many developers.

## Key Assumptions

- You have basic knowledge of Vue 3 fundamentals
- You understand HTML, CSS, and JavaScript (ES6+)
- You're familiar with single-file components (.vue files)
- You use Tailwind v4 for styling (no @apply or config files needed)

## Essential Concepts

### 1. Data, Methods, and Computed Properties

**Concise Explanation:**
The backbone of the Options API is organizing component logic into separate options:

- **data**: Reactive state that the component owns and manages
- **methods**: Functions that can be called from templates or other methods
- **computed**: Derived values that update automatically when dependencies change

These three options handle most of a component's core functionality and state management.

**Where to Use:**

- **data**: For any state that belongs to the component and needs to be reactive
- **methods**: For actions, event handlers, and complex logic
- **computed**: For derived values that depend on other state

**Code Snippet:**

```html
<template>
  <div class="p-6 max-w-md mx-auto bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-700">Product Details</h1>

    <!-- Using data and computed properties in templates -->
    <div class="mt-4 space-y-2">
      <div class="flex justify-between">
        <span class="text-gray-500">Product:</span>
        <span class="font-medium">{{ product.name }}</span>
      </div>

      <div class="flex justify-between">
        <span class="text-gray-500">Price:</span>
        <span class="font-medium">${{ product.price.toFixed(2) }}</span>
      </div>

      <div class="flex justify-between">
        <span class="text-gray-500">Quantity:</span>
        <div class="flex items-center space-x-2">
          <button
            @click="decreaseQuantity"
            class="px-2 py-1 bg-gray-200 rounded text-gray-700"
            :disabled="quantity <= 1"
          >
            -
          </button>
          <span>{{ quantity }}</span>
          <button
            @click="increaseQuantity"
            class="px-2 py-1 bg-gray-200 rounded text-gray-700"
          >
            +
          </button>
        </div>
      </div>

      <!-- Using computed property -->
      <div class="flex justify-between border-t pt-2">
        <span class="text-gray-700 font-medium">Total:</span>
        <span class="font-bold" :class="totalPriceClass"
          >${{ totalPrice }}</span
        >
      </div>
    </div>

    <!-- Using methods for event handling -->
    <div class="mt-6">
      <button
        @click="addToCart"
        class="w-full py-2 px-4 bg-blue-600 text-white font-medium rounded hover:bg-blue-700"
      >
        Add to Cart
      </button>

      <div
        v-if="message"
        class="mt-4 p-2 text-center rounded"
        :class="messageClass"
      >
        {{ message }}
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: "ProductDetail",

    // Component state (reactive)
    data() {
      return {
        product: {
          id: "prod-123",
          name: "Premium Headphones",
          price: 149.99,
          inStock: true,
        },
        quantity: 1,
        message: "",
        messageType: "success", // 'success' or 'error'
      };
    },

    // Methods (functions)
    methods: {
      increaseQuantity() {
        this.quantity += 1;
      },

      decreaseQuantity() {
        if (this.quantity > 1) {
          this.quantity -= 1;
        }
      },

      addToCart() {
        // Validation
        if (!this.product.inStock) {
          this.message = "Sorry, this product is out of stock";
          this.messageType = "error";
          return;
        }

        // Simulate API call or store action
        console.log(`Adding ${this.quantity} x ${this.product.name} to cart`);

        // Success message
        this.message = `${this.quantity} item(s) added to cart!`;
        this.messageType = "success";

        // Reset quantity
        this.quantity = 1;

        // Clear message after 3 seconds
        setTimeout(() => {
          this.message = "";
        }, 3000);
      },

      // Other methods can be added here
      applyDiscount(percentage) {
        const discount = this.product.price * (percentage / 100);
        return this.product.price - discount;
      },
    },

    // Computed properties (derived state)
    computed: {
      totalPrice() {
        return (this.product.price * this.quantity).toFixed(2);
      },

      totalPriceClass() {
        return this.quantity >= 5 ? "text-green-600" : "text-gray-800";
      },

      messageClass() {
        return {
          "bg-green-100 text-green-800": this.messageType === "success",
          "bg-red-100 text-red-800": this.messageType === "error",
        };
      },

      // Computed property with a getter and setter
      discountedPrice: {
        get() {
          return this.product.price * 0.9; // 10% discount
        },
        set(newValue) {
          // This updates the original price based on the discounted price
          this.product.price = newValue / 0.9;
        },
      },
    },
  };
</script>
```

**Real-World Example:**
In an e-commerce product configurator, you might use:

- **data** for the base product details, selected options, and UI state
- **methods** for updating configurations, validating selections, and submitting orders
- **computed** for the final price, product availability, and recommended accessories

```html
<template>
  <div class="p-6 bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-semibold text-gray-800">
      {{ product.name }} Configurator
    </h1>

    <!-- Product image -->
    <div
      class="mt-4 aspect-square bg-gray-100 rounded-lg flex items-center justify-center overflow-hidden"
    >
      <img
        :src="selectedConfiguration.imageUrl"
        :alt="product.name"
        class="max-h-full max-w-full object-contain"
      />
    </div>

    <!-- Color options -->
    <div class="mt-6">
      <h2 class="text-sm font-medium text-gray-700">Color</h2>
      <div class="mt-2 flex space-x-2">
        <button
          v-for="color in product.colors"
          :key="color.id"
          @click="selectColor(color)"
          class="w-8 h-8 rounded-full border-2 hover:opacity-75"
          :class="[
            selectedColor.id === color.id
              ? 'border-blue-500'
              : 'border-transparent',
          ]"
          :style="{ backgroundColor: color.hexCode }"
        ></button>
      </div>
    </div>

    <!-- Storage options -->
    <div class="mt-6">
      <h2 class="text-sm font-medium text-gray-700">Storage</h2>
      <div class="mt-2 grid grid-cols-3 gap-2">
        <button
          v-for="storage in product.storageOptions"
          :key="storage.id"
          @click="selectStorage(storage)"
          class="px-4 py-2 text-center rounded"
          :class="[
            selectedStorage.id === storage.id
              ? 'bg-blue-500 text-white'
              : 'bg-gray-100 text-gray-800 hover:bg-gray-200',
          ]"
        >
          {{ storage.label }}
        </button>
      </div>
    </div>

    <!-- Accessories -->
    <div class="mt-6">
      <h2 class="text-sm font-medium text-gray-700">Accessories</h2>
      <div class="mt-2 space-y-2">
        <div
          v-for="accessory in product.accessories"
          :key="accessory.id"
          class="flex items-center justify-between p-3 bg-gray-50 rounded"
        >
          <div class="flex items-center">
            <input
              type="checkbox"
              :id="`accessory-${accessory.id}`"
              :value="accessory.id"
              v-model="selectedAccessories"
              class="h-4 w-4 text-blue-600 rounded"
            />
            <label
              :for="`accessory-${accessory.id}`"
              class="ml-2 text-gray-700"
            >
              {{ accessory.name }}
            </label>
          </div>
          <span class="text-gray-900 font-medium"
            >${{ accessory.price.toFixed(2) }}</span
          >
        </div>
      </div>
    </div>

    <!-- Summary -->
    <div class="mt-8 pt-6 border-t border-gray-200">
      <div class="flex justify-between text-sm">
        <span class="text-gray-500">Base price</span>
        <span>${{ product.basePrice.toFixed(2) }}</span>
      </div>

      <div
        class="flex justify-between text-sm mt-2"
        v-if="storagePriceDifference !== 0"
      >
        <span class="text-gray-500">Storage upgrade</span>
        <span>${{ storagePriceDifference.toFixed(2) }}</span>
      </div>

      <div
        class="flex justify-between text-sm mt-2"
        v-if="accessoriesPrice > 0"
      >
        <span class="text-gray-500">Accessories</span>
        <span>${{ accessoriesPrice.toFixed(2) }}</span>
      </div>

      <div class="flex justify-between mt-4 pt-4 border-t border-gray-200">
        <span class="text-lg font-semibold">Total</span>
        <span class="text-lg font-bold">${{ totalPrice.toFixed(2) }}</span>
      </div>

      <button
        @click="addToCart"
        :disabled="!isAvailable"
        class="mt-6 w-full py-3 px-6 bg-blue-600 text-white font-medium rounded hover:bg-blue-700 disabled:bg-gray-300 disabled:cursor-not-allowed"
      >
        {{ isAvailable ? "Add to Cart" : "Out of Stock" }}
      </button>
    </div>
  </div>
</template>

<script>
  export default {
    name: "ProductConfigurator",

    data() {
      return {
        product: {
          id: "smartphone-x",
          name: "Smartphone X",
          basePrice: 799.99,
          colors: [
            { id: "black", name: "Black", hexCode: "#000000", inStock: true },
            { id: "silver", name: "Silver", hexCode: "#C0C0C0", inStock: true },
            { id: "gold", name: "Gold", hexCode: "#FFD700", inStock: false },
          ],
          storageOptions: [
            { id: "128gb", label: "128GB", price: 0, inStock: true },
            { id: "256gb", label: "256GB", price: 100, inStock: true },
            { id: "512gb", label: "512GB", price: 200, inStock: false },
          ],
          accessories: [
            {
              id: "case",
              name: "Protective Case",
              price: 29.99,
              inStock: true,
            },
            {
              id: "charger",
              name: "Fast Charger",
              price: 49.99,
              inStock: true,
            },
            {
              id: "earbuds",
              name: "Wireless Earbuds",
              price: 129.99,
              inStock: false,
            },
          ],
        },
        selectedColor: null,
        selectedStorage: null,
        selectedAccessories: [],
        isAddingToCart: false,
        cartMessage: "",
      };
    },

    created() {
      // Set default selections when component is created
      this.selectedColor =
        this.product.colors.find((color) => color.inStock) ||
        this.product.colors[0];
      this.selectedStorage =
        this.product.storageOptions.find((storage) => storage.inStock) ||
        this.product.storageOptions[0];
    },

    methods: {
      selectColor(color) {
        this.selectedColor = color;
      },

      selectStorage(storage) {
        this.selectedStorage = storage;
      },

      addToCart() {
        if (!this.isAvailable) return;

        this.isAddingToCart = true;

        // Simulate API call
        setTimeout(() => {
          const order = {
            productId: this.product.id,
            colorId: this.selectedColor.id,
            storageId: this.selectedStorage.id,
            accessories: this.selectedAccessories,
            price: this.totalPrice,
          };

          console.log("Adding to cart:", order);
          this.cartMessage = "Product added to cart!";
          this.isAddingToCart = false;

          // Reset message after 3 seconds
          setTimeout(() => {
            this.cartMessage = "";
          }, 3000);
        }, 1000);
      },

      getAccessoryById(id) {
        return this.product.accessories.find((acc) => acc.id === id);
      },
    },

    computed: {
      selectedConfiguration() {
        // In a real app, you would construct the image URL based on selections
        return {
          imageUrl: `https://example.com/images/${this.product.id}-${this.selectedColor.id}.jpg`,
        };
      },

      storagePriceDifference() {
        return this.selectedStorage ? this.selectedStorage.price : 0;
      },

      accessoriesPrice() {
        return this.selectedAccessories.reduce((total, accId) => {
          const accessory = this.getAccessoryById(accId);
          return total + (accessory ? accessory.price : 0);
        }, 0);
      },

      totalPrice() {
        return (
          this.product.basePrice +
          this.storagePriceDifference +
          this.accessoriesPrice
        );
      },

      selectedAccessoriesDetails() {
        return this.selectedAccessories
          .map((id) => this.getAccessoryById(id))
          .filter((acc) => acc !== undefined);
      },

      isAvailable() {
        // Check if the current configuration is available
        const isColorAvailable =
          this.selectedColor && this.selectedColor.inStock;
        const isStorageAvailable =
          this.selectedStorage && this.selectedStorage.inStock;

        // Check if all selected accessories are in stock
        const areAccessoriesAvailable = this.selectedAccessoriesDetails.every(
          (acc) => acc.inStock
        );

        return (
          isColorAvailable && isStorageAvailable && areAccessoriesAvailable
        );
      },
    },
  };
</script>
```

**Common Pitfalls:**

- **Data Mutability**: Directly mutating nested objects or arrays can cause reactivity issues
- **This Context**: Losing the `this` context in methods, especially in callbacks or event handlers
- **Method vs. Computed**: Using methods when computed properties would be more appropriate
- **Computed Side Effects**: Causing side effects in computed properties (they should be pure)
- **Arrow Functions**: Using arrow functions for methods or computed properties, which affects `this` binding

**Frequently Asked Questions:**

1. **Q: When should I use a method versus a computed property?**

   A: Use **computed properties** for derived values that:

   - Depend on other data
   - Need to be cached until dependencies change
   - Don't require arguments

   Use **methods** when you need to:

   - Perform an action rather than return a value
   - Pass arguments to the function
   - Call the function explicitly (not automatically when dependencies change)
   - Create functions that cause side effects

2. **Q: How does reactivity work with complex data structures?**

   A: Vue automatically makes all properties in your `data` reactive, but there are limitations:

   - Adding new properties to existing objects won't be reactive unless you use `Vue.set` or replace the entire object
   - Similarly, direct array index assignment (e.g., `arr[0] = newValue`) isn't reactive; use `splice` or array methods
   - In Vue 3, these limitations are reduced due to the Proxy-based reactivity system

3. **Q: Can I add data properties after a component is created?**

   A: All properties used by a component should be initialized in the `data` function. If you need to add properties later:

   ```js
   // Vue 2 approach
   this.$set(this.object, "newProp", value);

   // Vue 3 with Options API
   this.object = { ...this.object, newProp: value };
   ```

   A better practice is to initialize all properties with default values (null, empty string, etc.).

### 2. Watchers

**Concise Explanation:**
Watchers (`watch`) observe changes to specific data properties and execute functions in response. Unlike computed properties, which derive values, watchers are ideal for side effects like API calls, localStorage updates, or complex state transitions. Watchers also provide access to both the previous and current values of the watched property.

**Where to Use:**

- Reacting to data changes with side effects
- Making API calls when dependencies change
- Persisting state changes to localStorage/sessionStorage
- Complex validation or data transformations
- Syncing component state with external systems
- Transitioning between states with animations

**Code Snippet:**

```html
<template>
  <div class="p-6 max-w-md mx-auto bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800">Search Users</h1>

    <!-- Search input with debouncing via watcher -->
    <div class="mt-4">
      <div class="relative">
        <input
          v-model="searchQuery"
          type="text"
          placeholder="Search users..."
          class="w-full px-4 py-2 pr-10 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
        />
        <div
          class="absolute inset-y-0 right-0 flex items-center pr-3 pointer-events-none"
        >
          <svg
            v-if="isSearching"
            class="animate-spin h-5 w-5 text-gray-400"
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
          <svg
            v-else
            class="h-5 w-5 text-gray-400"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"
            />
          </svg>
        </div>
      </div>

      <!-- Search statistics -->
      <div class="mt-2 text-sm text-gray-500">
        <span v-if="searchResults.length"
          >Found {{ searchResults.length }} results</span
        >
        <span v-else-if="searchQuery && !isSearching">No results found</span>
        <span v-else-if="!searchQuery">&nbsp;</span>
      </div>
    </div>

    <!-- Results list -->
    <div class="mt-4">
      <div v-if="isSearching" class="py-4 text-center text-gray-500">
        Searching...
      </div>

      <div
        v-else-if="searchResults.length === 0 && searchQuery"
        class="py-4 text-center text-gray-500"
      >
        No users found for "{{ searchQuery }}"
      </div>

      <div v-else-if="searchResults.length" class="divide-y">
        <div
          v-for="user in searchResults"
          :key="user.id"
          class="py-3 flex items-center"
        >
          <img
            :src="user.avatar"
            :alt="user.name"
            class="h-10 w-10 rounded-full object-cover"
          />
          <div class="ml-3">
            <p class="text-sm font-medium text-gray-900">{{ user.name }}</p>
            <p class="text-sm text-gray-500">{{ user.email }}</p>
          </div>
        </div>
      </div>
    </div>

    <!-- Search history -->
    <div v-if="searchHistory.length" class="mt-6">
      <h2 class="text-sm font-medium text-gray-700">Recent Searches</h2>
      <div class="mt-2 flex flex-wrap gap-2">
        <button
          v-for="(term, index) in searchHistory"
          :key="index"
          @click="searchQuery = term"
          class="px-3 py-1 text-sm bg-gray-100 rounded hover:bg-gray-200"
        >
          {{ term }}
        </button>

        <button
          @click="clearHistory"
          class="px-3 py-1 text-sm text-red-600 hover:text-red-800"
        >
          Clear
        </button>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: "SearchUsers",

    data() {
      return {
        searchQuery: "",
        searchResults: [],
        searchHistory: [],
        isSearching: false,
        searchTimeout: null,
        showSuggestions: false,
      };
    },

    watch: {
      // Basic watcher
      searchQuery(newQuery, oldQuery) {
        // Clear previous timeout
        if (this.searchTimeout) {
          clearTimeout(this.searchTimeout);
        }

        // Don't search if query is empty
        if (!newQuery.trim()) {
          this.searchResults = [];
          this.isSearching = false;
          return;
        }

        // Set flag to show loading state
        this.isSearching = true;

        // Debounce the search (wait for user to stop typing)
        this.searchTimeout = setTimeout(() => {
          this.performSearch(newQuery);
        }, 500);
      },

      // Watcher with options
      searchResults: {
        handler(newResults) {
          console.log(`Found ${newResults.length} results`);

          // Update document title to reflect search results
          if (this.searchQuery && newResults.length > 0) {
            document.title = `(${newResults.length}) Results for "${this.searchQuery}"`;
          } else {
            document.title = "User Search";
          }
        },
        deep: true, // Watch all properties of objects in array
      },

      // Watcher with immediate option
      "$route.query.search": {
        handler(newSearchTerm) {
          if (newSearchTerm) {
            this.searchQuery = newSearchTerm;
          }
        },
        immediate: true, // Run when component is created
      },
    },

    methods: {
      async performSearch(query) {
        try {
          // In a real app, this would be an API call
          // Simulating API delay
          await new Promise((resolve) => setTimeout(resolve, 1000));

          // Mock results
          const mockUsers = [
            {
              id: 1,
              name: "John Doe",
              email: "john@example.com",
              avatar: "https://i.pravatar.cc/150?img=1",
            },
            {
              id: 2,
              name: "Jane Smith",
              email: "jane@example.com",
              avatar: "https://i.pravatar.cc/150?img=5",
            },
            {
              id: 3,
              name: "Alice Johnson",
              email: "alice@example.com",
              avatar: "https://i.pravatar.cc/150?img=9",
            },
            {
              id: 4,
              name: "Bob Brown",
              email: "bob@example.com",
              avatar: "https://i.pravatar.cc/150?img=3",
            },
          ];

          // Filter based on query
          this.searchResults = mockUsers.filter(
            (user) =>
              user.name.toLowerCase().includes(query.toLowerCase()) ||
              user.email.toLowerCase().includes(query.toLowerCase())
          );

          // Add to search history if not already there
          if (query.trim() && !this.searchHistory.includes(query)) {
            this.searchHistory = [query, ...this.searchHistory].slice(0, 5);
            this.saveSearchHistory();
          }
        } catch (error) {
          console.error("Search failed:", error);
          this.searchResults = [];
        } finally {
          this.isSearching = false;
        }
      },

      saveSearchHistory() {
        localStorage.setItem(
          "searchHistory",
          JSON.stringify(this.searchHistory)
        );
      },

      clearHistory() {
        this.searchHistory = [];
        localStorage.removeItem("searchHistory");
      },
    },

    created() {
      // Load search history from localStorage
      const savedHistory = localStorage.getItem("searchHistory");
      if (savedHistory) {
        try {
          this.searchHistory = JSON.parse(savedHistory);
        } catch (e) {
          console.error("Failed to parse search history", e);
        }
      }
    },
  };
</script>
```

**Dynamic Form Validation with Watchers:**

```html
<template>
  <div class="p-6 max-w-md mx-auto bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800">Registration Form</h1>

    <form @submit.prevent="submitForm" class="mt-6 space-y-6">
      <!-- Username field -->
      <div>
        <label for="username" class="block text-sm font-medium text-gray-700"
          >Username</label
        >
        <input
          id="username"
          v-model="form.username"
          type="text"
          class="mt-1 w-full px-4 py-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          :class="{ 'border-red-500': errors.username }"
        />
        <p v-if="errors.username" class="mt-1 text-sm text-red-600">
          {{ errors.username }}
        </p>
        <p v-else-if="usernameAvailable" class="mt-1 text-sm text-green-600">
          Username is available!
        </p>
      </div>

      <!-- Email field -->
      <div>
        <label for="email" class="block text-sm font-medium text-gray-700"
          >Email</label
        >
        <input
          id="email"
          v-model="form.email"
          type="email"
          class="mt-1 w-full px-4 py-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          :class="{ 'border-red-500': errors.email }"
        />
        <p v-if="errors.email" class="mt-1 text-sm text-red-600">
          {{ errors.email }}
        </p>
      </div>

      <!-- Password field -->
      <div>
        <label for="password" class="block text-sm font-medium text-gray-700"
          >Password</label
        >
        <input
          id="password"
          v-model="form.password"
          type="password"
          class="mt-1 w-full px-4 py-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          :class="{ 'border-red-500': errors.password }"
        />
        <p v-if="errors.password" class="mt-1 text-sm text-red-600">
          {{ errors.password }}
        </p>

        <!-- Password strength meter -->
        <div v-if="form.password" class="mt-2">
          <div class="flex items-center">
            <div class="flex-1 h-2 bg-gray-200 rounded-full overflow-hidden">
              <div
                class="h-full rounded-full"
                :class="{
                  'w-1/3 bg-red-500': passwordStrength === 'weak',
                  'w-2/3 bg-yellow-500': passwordStrength === 'medium',
                  'w-full bg-green-500': passwordStrength === 'strong',
                }"
              ></div>
            </div>
            <span
              class="ml-2 text-xs"
              :class="{
                'text-red-600': passwordStrength === 'weak',
                'text-yellow-600': passwordStrength === 'medium',
                'text-green-600': passwordStrength === 'strong',
              }"
            >
              {{ passwordStrength }}
            </span>
          </div>
        </div>
      </div>

      <!-- Confirm password field -->
      <div>
        <label
          for="confirmPassword"
          class="block text-sm font-medium text-gray-700"
          >Confirm Password</label
        >
        <input
          id="confirmPassword"
          v-model="form.confirmPassword"
          type="password"
          class="mt-1 w-full px-4 py-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          :class="{ 'border-red-500': errors.confirmPassword }"
        />
        <p v-if="errors.confirmPassword" class="mt-1 text-sm text-red-600">
          {{ errors.confirmPassword }}
        </p>
      </div>

      <!-- Terms checkbox -->
      <div class="flex items-start">
        <div class="flex items-center h-5">
          <input
            id="terms"
            v-model="form.acceptTerms"
            type="checkbox"
            class="h-4 w-4 text-blue-600 border-gray-300 rounded focus:ring-blue-500"
          />
        </div>
        <div class="ml-3 text-sm">
          <label for="terms" class="font-medium text-gray-700"
            >Accept Terms and Conditions</label
          >
          <p v-if="errors.acceptTerms" class="mt-1 text-sm text-red-600">
            {{ errors.acceptTerms }}
          </p>
        </div>
      </div>

      <!-- Form actions -->
      <div>
        <button
          type="submit"
          :disabled="isSubmitting || !isFormValid"
          class="w-full py-2 px-4 bg-blue-600 text-white font-medium rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 disabled:opacity-50"
        >
          <span v-if="isSubmitting">
            <svg
              class="animate-spin -ml-1 mr-2 h-4 w-4 text-white inline-block"
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
  </div>
</template>

<script>
  export default {
    name: "RegistrationForm",

    data() {
      return {
        form: {
          username: "",
          email: "",
          password: "",
          confirmPassword: "",
          acceptTerms: false,
        },
        errors: {
          username: "",
          email: "",
          password: "",
          confirmPassword: "",
          acceptTerms: "",
        },
        isSubmitting: false,
        usernameCheckTimeout: null,
        usernameAvailable: false,
      };
    },

    computed: {
      passwordStrength() {
        const password = this.form.password;
        if (!password) return "";

        // Simple strength calculation
        const hasUppercase = /[A-Z]/.test(password);
        const hasLowercase = /[a-z]/.test(password);
        const hasNumbers = /\d/.test(password);
        const hasSpecialChars = /[!@#$%^&*(),.?":{}|<>]/.test(password);

        const strength =
          (password.length >= 8 ? 1 : 0) +
          (hasUppercase ? 1 : 0) +
          (hasLowercase ? 1 : 0) +
          (hasNumbers ? 1 : 0) +
          (hasSpecialChars ? 1 : 0);

        if (strength <= 2) return "weak";
        if (strength <= 4) return "medium";
        return "strong";
      },

      isFormValid() {
        // Check if all fields are filled and there are no errors
        return (
          this.form.username &&
          this.form.email &&
          this.form.password &&
          this.form.confirmPassword &&
          this.form.acceptTerms &&
          !this.errors.username &&
          !this.errors.email &&
          !this.errors.password &&
          !this.errors.confirmPassword &&
          !this.errors.acceptTerms
        );
      },
    },

    watch: {
      // Watch username field for availability check
      "form.username": function (newUsername) {
        // Clear previous timeout
        if (this.usernameCheckTimeout) {
          clearTimeout(this.usernameCheckTimeout);
        }

        // Reset validation
        this.errors.username = "";
        this.usernameAvailable = false;

        // Basic validation
        if (!newUsername) {
          this.errors.username = "Username is required";
          return;
        }

        if (newUsername.length < 3) {
          this.errors.username = "Username must be at least 3 characters";
          return;
        }

        // Debounce API call
        this.usernameCheckTimeout = setTimeout(() => {
          this.checkUsernameAvailability(newUsername);
        }, 500);
      },

      // Watch email field
      "form.email": function (newEmail) {
        // Basic validation
        this.errors.email = "";

        if (!newEmail) {
          this.errors.email = "Email is required";
          return;
        }

        // Simple email validation
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        if (!emailRegex.test(newEmail)) {
          this.errors.email = "Please enter a valid email address";
        }
      },

      // Watch password field
      "form.password": function (newPassword) {
        // Basic validation
        this.errors.password = "";

        if (!newPassword) {
          this.errors.password = "Password is required";
          return;
        }

        if (newPassword.length < 8) {
          this.errors.password = "Password must be at least 8 characters";
        }

        // Also validate confirm password when password changes
        this.validateConfirmPassword();
      },

      // Watch confirm password field
      "form.confirmPassword": function () {
        this.validateConfirmPassword();
      },

      // Watch terms checkbox
      "form.acceptTerms": function (newValue) {
        this.errors.acceptTerms = newValue
          ? ""
          : "You must accept the terms and conditions";
      },

      // Watch all form fields with deep option
      form: {
        handler(newForm) {
          console.log("Form updated:", JSON.stringify(newForm));
        },
        deep: true, // Watch nested properties
      },
    },

    methods: {
      validateConfirmPassword() {
        this.errors.confirmPassword = "";

        if (!this.form.confirmPassword) {
          this.errors.confirmPassword = "Please confirm your password";
          return;
        }

        if (this.form.password !== this.form.confirmPassword) {
          this.errors.confirmPassword = "Passwords do not match";
        }
      },

      async checkUsernameAvailability(username) {
        try {
          // Simulate API call
          await new Promise((resolve) => setTimeout(resolve, 800));

          // Mock username check (in a real app, this would be an API call)
          const takenUsernames = ["admin", "user", "test", "john"];
          const isAvailable = !takenUsernames.includes(username.toLowerCase());

          if (!isAvailable) {
            this.errors.username = "This username is already taken";
          } else {
            this.usernameAvailable = true;
          }
        } catch (error) {
          console.error("Username check failed:", error);
          this.errors.username = "Failed to check username availability";
        }
      },

      async submitForm() {
        // Final validation
        this.validateAllFields();

        if (!this.isFormValid) {
          return;
        }

        this.isSubmitting = true;

        try {
          // Simulate API call
          await new Promise((resolve) => setTimeout(resolve, 1500));

          alert("Registration successful!");

          // Reset form
          this.form = {
            username: "",
            email: "",
            password: "",
            confirmPassword: "",
            acceptTerms: false,
          };
        } catch (error) {
          console.error("Registration failed:", error);
          alert("Registration failed. Please try again.");
        } finally {
          this.isSubmitting = false;
        }
      },

      validateAllFields() {
        // Trigger all validations
        if (!this.form.username) {
          this.errors.username = "Username is required";
        }

        if (!this.form.email) {
          this.errors.email = "Email is required";
        }

        if (!this.form.password) {
          this.errors.password = "Password is required";
        }

        if (!this.form.confirmPassword) {
          this.errors.confirmPassword = "Please confirm your password";
        }

        if (!this.form.acceptTerms) {
          this.errors.acceptTerms = "You must accept the terms and conditions";
        }
      },
    },
  };
</script>
```

**Real-World Example:**
In a task management application, watchers are essential for syncing with a backend and maintaining UI state:

```html
<template>
  <div class="p-6 bg-white rounded-xl shadow-md">
    <div class="flex justify-between items-center mb-6">
      <h1 class="text-xl font-semibold text-gray-800">{{ board.name }}</h1>

      <div class="flex space-x-2">
        <button
          @click="saveBoard"
          class="px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700"
          :disabled="!isDirty || isSaving"
        >
          <span v-if="isSaving">Saving...</span>
          <span v-else>Save</span>
        </button>

        <button
          @click="toggleBoardSettings"
          class="p-2 text-gray-600 hover:text-gray-800"
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
              d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z"
            />
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"
            />
          </svg>
        </button>
      </div>
    </div>

    <!-- Task lists -->
    <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
      <div
        v-for="list in board.lists"
        :key="list.id"
        class="bg-gray-50 rounded-lg p-4"
      >
        <div class="flex justify-between items-center mb-3">
          <h3 class="font-medium text-gray-700">{{ list.name }}</h3>
          <span class="text-sm text-gray-500">{{ list.tasks.length }}</span>
        </div>

        <!-- Tasks -->
        <div class="space-y-2">
          <div
            v-for="task in list.tasks"
            :key="task.id"
            class="bg-white p-3 rounded shadow-sm"
          >
            <div class="flex items-start">
              <input
                type="checkbox"
                :checked="task.completed"
                @change="toggleTaskCompletion(list.id, task.id)"
                class="mt-1 h-4 w-4 text-blue-600 rounded"
              />
              <div class="ml-3 flex-1">
                <p :class="{ 'line-through text-gray-500': task.completed }">
                  {{ task.title }}
                </p>
                <p v-if="task.description" class="text-sm text-gray-500 mt-1">
                  {{ task.description }}
                </p>
                <div
                  v-if="task.dueDate"
                  class="mt-2 text-xs"
                  :class="{
                    'text-red-600': isOverdue(task.dueDate),
                    'text-orange-600':
                      isDueSoon(task.dueDate) && !isOverdue(task.dueDate),
                    'text-gray-500':
                      !isDueSoon(task.dueDate) && !isOverdue(task.dueDate),
                  }"
                >
                  Due: {{ formatDate(task.dueDate) }}
                </div>
              </div>
            </div>
          </div>

          <!-- Add task input -->
          <div
            v-if="activeListForNewTask === list.id"
            class="bg-white p-3 rounded shadow-sm"
          >
            <input
              v-model="newTaskTitle"
              @keyup.enter="addNewTask"
              @keyup.esc="activeListForNewTask = null"
              placeholder="Task title..."
              class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
              ref="newTaskInput"
            />
            <div class="flex justify-end mt-2 space-x-2">
              <button
                @click="activeListForNewTask = null"
                class="px-3 py-1 text-sm text-gray-600 hover:text-gray-800"
              >
                Cancel
              </button>
              <button
                @click="addNewTask"
                class="px-3 py-1 text-sm bg-blue-600 text-white rounded hover:bg-blue-700"
              >
                Add
              </button>
            </div>
          </div>

          <!-- Add task button -->
          <button
            v-else
            @click="startAddingTask(list.id)"
            class="w-full py-2 text-sm text-gray-600 hover:text-gray-800 text-center border border-dashed rounded hover:bg-gray-50"
          >
            + Add task
          </button>
        </div>
      </div>
    </div>

    <!-- Activity log -->
    <div class="mt-8">
      <h3 class="text-sm font-medium text-gray-700 mb-2">Recent Activity</h3>
      <div class="bg-gray-50 rounded-lg p-4 max-h-40 overflow-y-auto">
        <div
          v-if="activityLog.length === 0"
          class="text-center text-gray-500 py-2"
        >
          No recent activity
        </div>
        <div v-else class="space-y-2">
          <div
            v-for="(activity, index) in activityLog"
            :key="index"
            class="text-sm"
          >
            <span class="text-gray-500"
              >{{ formatTime(activity.timestamp) }}:</span
            >
            <span class="ml-2">{{ activity.message }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: "TaskBoard",

    data() {
      return {
        board: {
          id: "board-1",
          name: "Project Tasks",
          lists: [
            {
              id: "list-1",
              name: "To Do",
              tasks: [
                {
                  id: "task-1",
                  title: "Research competitors",
                  description: "Look at 5 competing products",
                  completed: false,
                  dueDate: "2025-06-10",
                },
                {
                  id: "task-2",
                  title: "Create wireframes",
                  description: "",
                  completed: false,
                  dueDate: "2025-06-15",
                },
              ],
            },
            {
              id: "list-2",
              name: "In Progress",
              tasks: [
                {
                  id: "task-3",
                  title: "Develop MVP",
                  description: "Focus on core features only",
                  completed: false,
                  dueDate: "2025-06-20",
                },
              ],
            },
            {
              id: "list-3",
              name: "Done",
              tasks: [
                {
                  id: "task-4",
                  title: "Project setup",
                  description: "Initialize repository and project structure",
                  completed: true,
                  dueDate: "2025-06-01",
                },
              ],
            },
          ],
        },
        originalBoard: null, // For dirty checking
        activityLog: [],
        isSaving: false,
        activeListForNewTask: null,
        newTaskTitle: "",
        saveTimeout: null,
        boardSettingsOpen: false,
      };
    },

    computed: {
      isDirty() {
        return (
          JSON.stringify(this.board) !== JSON.stringify(this.originalBoard)
        );
      },
    },

    watch: {
      // Watch for changes to the board data
      board: {
        handler(newBoard) {
          // Cancel previous save timeout if there is one
          if (this.saveTimeout) {
            clearTimeout(this.saveTimeout);
          }

          // Set a timeout to auto-save after changes
          if (this.isDirty) {
            this.saveTimeout = setTimeout(() => {
              this.saveBoard();
            }, 3000); // Auto-save after 3 seconds of inactivity
          }
        },
        deep: true, // Watch nested properties of the board
      },

      // Watch for completed tasks to update activity log
      "board.lists": {
        handler(newLists, oldLists) {
          // Check if a task was completed
          newLists.forEach((newList) => {
            const oldList = oldLists.find((list) => list.id === newList.id);

            if (oldList) {
              newList.tasks.forEach((newTask) => {
                const oldTask = oldList.tasks.find(
                  (task) => task.id === newTask.id
                );

                if (oldTask && oldTask.completed !== newTask.completed) {
                  // Log task completion/uncompleting
                  const action = newTask.completed ? "completed" : "reopened";
                  this.addActivityLogEntry(`Task "${newTask.title}" ${action}`);
                }
              });

              // Check if a new task was added
              if (newList.tasks.length > oldList.tasks.length) {
                const newTasks = newList.tasks.filter(
                  (newTask) =>
                    !oldList.tasks.some((oldTask) => oldTask.id === newTask.id)
                );

                newTasks.forEach((task) => {
                  this.addActivityLogEntry(
                    `New task "${task.title}" added to "${newList.name}"`
                  );
                });
              }
            }
          });
        },
        deep: true,
      },

      // Watch for route changes to load correct board
      "$route.params.boardId": {
        handler(newBoardId) {
          if (newBoardId && newBoardId !== this.board.id) {
            this.loadBoard(newBoardId);
          }
        },
        immediate: true,
      },
    },

    created() {
      // Clone board for dirty checking
      this.originalBoard = JSON.parse(JSON.stringify(this.board));

      // Initial activity log entry
      this.addActivityLogEntry("Board loaded");
    },

    methods: {
      toggleTaskCompletion(listId, taskId) {
        const list = this.board.lists.find((list) => list.id === listId);
        if (!list) return;

        const task = list.tasks.find((task) => task.id === taskId);
        if (!task) return;

        task.completed = !task.completed;
      },

      startAddingTask(listId) {
        this.activeListForNewTask = listId;
        this.newTaskTitle = "";

        // Focus the input field after DOM update
        this.$nextTick(() => {
          if (this.$refs.newTaskInput) {
            this.$refs.newTaskInput.focus();
          }
        });
      },

      addNewTask() {
        if (!this.newTaskTitle.trim()) return;

        const list = this.board.lists.find(
          (list) => list.id === this.activeListForNewTask
        );
        if (!list) return;

        // Create a new task
        const newTask = {
          id: `task-${Date.now()}`,
          title: this.newTaskTitle.trim(),
          description: "",
          completed: false,
          dueDate: null,
        };

        // Add to the list
        list.tasks.push(newTask);

        // Reset the form
        this.newTaskTitle = "";
        this.activeListForNewTask = null;

        // The activity log will be updated by the watcher
      },

      async saveBoard() {
        if (!this.isDirty) return;

        this.isSaving = true;

        try {
          // Simulate API call
          await new Promise((resolve) => setTimeout(resolve, 1000));

          // In a real app, you would save to the backend here
          console.log("Board saved:", this.board);

          // Update original board for dirty checking
          this.originalBoard = JSON.parse(JSON.stringify(this.board));

          // Log the save
          this.addActivityLogEntry("Board saved");
        } catch (error) {
          console.error("Failed to save board:", error);

          // In a real app, you would show an error toast here
        } finally {
          this.isSaving = false;
        }
      },

      async loadBoard(boardId) {
        // In a real app, this would be an API call
        // Simulating loading a different board
        console.log(`Loading board ${boardId}...`);

        // Check for unsaved changes
        if (this.isDirty) {
          const shouldSave = confirm(
            "You have unsaved changes. Save before switching boards?"
          );
          if (shouldSave) {
            await this.saveBoard();
          }
        }

        // Simulate board loading
        // In a real app, you would fetch the board data from the backend
      },

      toggleBoardSettings() {
        this.boardSettingsOpen = !this.boardSettingsOpen;

        // In a real app, this would show a modal or side panel
        console.log("Board settings toggled:", this.boardSettingsOpen);
      },

      addActivityLogEntry(message) {
        this.activityLog.unshift({
          message,
          timestamp: new Date(),
        });

        // Limit the log to the most recent 20 entries
        if (this.activityLog.length > 20) {
          this.activityLog = this.activityLog.slice(0, 20);
        }
      },

      isOverdue(dateString) {
        const dueDate = new Date(dateString);
        const today = new Date();

        // Remove time part for comparison
        today.setHours(0, 0, 0, 0);

        return dueDate < today;
      },

      isDueSoon(dateString) {
        const dueDate = new Date(dateString);
        const today = new Date();

        // Remove time part for comparison
        today.setHours(0, 0, 0, 0);

        // Get date 3 days from now
        const threeDaysFromNow = new Date(today);
        threeDaysFromNow.setDate(today.getDate() + 3);

        return dueDate <= threeDaysFromNow && dueDate >= today;
      },

      formatDate(dateString) {
        const date = new Date(dateString);
        return date.toLocaleDateString("en-US", {
          month: "short",
          day: "numeric",
        });
      },

      formatTime(date) {
        return date.toLocaleTimeString("en-US", {
          hour: "2-digit",
          minute: "2-digit",
        });
      },
    },
  };
</script>
```

**Common Pitfalls:**

- **Infinite Loops**: Watchers that modify the property they're watching can cause infinite loops
- **Performance Issues**: Not properly debouncing or throttling expensive operations in watchers
- **Unnecessary Watchers**: Using watchers when computed properties would be more appropriate
- **Missing Deep Option**: Not using the `deep` option when watching objects or arrays
- **Memory Leaks**: Not clearing timers or resources set up in watchers

**Frequently Asked Questions:**

1. **Q: What's the difference between a watcher and a computed property?**

   A: The key differences are:

   - **Computed Properties** calculate and return a value based on dependencies
   - **Watchers** execute side effects (like API calls) when a value changes
   - Computed properties are cached and only recalculated when dependencies change
   - Watchers give you access to both the new and old values
   - Use computed for derived data, watchers for side effects

2. **Q: How do I watch nested properties in objects or arrays?**

   A: Use the `deep` option:

   ```js
   watch: {
     myObject: {
       handler(newValue, oldValue) {
         // This will trigger when any nested property changes
         console.log('Object changed')
       },
       deep: true
     }
   }
   ```

   For better performance with large objects, you can watch specific nested properties:

   ```js
   watch: {
     'myObject.specificProperty'(newValue, oldValue) {
       // This will only trigger when specificProperty changes
       console.log('Specific property changed')
     }
   }
   ```

3. **Q: How do I run a watcher when the component is created?**

   A: Use the `immediate` option:

   ```js
   watch: {
     searchQuery: {
       handler(newValue, oldValue) {
         this.fetchResults(newValue)
       },
       immediate: true // Will run when component is created
     }
   }
   ```

   Alternatively, you can call the function directly in the `created` or `mounted` hooks.

### 3. Props and Events

**Concise Explanation:**
Props and events form the communication system between parent and child components in Vue:

- **Props** are data passed from parent to child components (downward communication)
- **Events** are signals emitted from child to parent components (upward communication)

Together, they create a predictable, one-way data flow that makes component relationships explicit and maintainable.

**Where to Use:**

- **Props**: For any data or configuration a component needs from its parent
- **Events**: When a child component needs to inform its parent about changes or actions
- Both are essential for creating reusable, modular components

**Code Snippet:**

```html
<!-- ParentComponent.vue -->
<template>
  <div class="p-6 bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800 mb-4">User Management</h1>

    <!-- Pass props to child component & listen for events -->
    <user-profile
      :user="currentUser"
      :editable="true"
      :theme="theme"
      @update:user="updateUser"
      @delete="confirmDeleteUser"
    />

    <!-- Theme toggle -->
    <div class="mt-6 flex justify-end">
      <button
        @click="toggleTheme"
        class="px-4 py-2 rounded"
        :class="
          theme === 'dark'
            ? 'bg-gray-800 text-white'
            : 'bg-blue-100 text-blue-800'
        "
      >
        Toggle to {{ theme === "dark" ? "Light" : "Dark" }} Theme
      </button>
    </div>

    <!-- User deletion confirmation modal -->
    <div
      v-if="showDeleteModal"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4"
    >
      <div class="bg-white rounded-lg max-w-md w-full p-6">
        <h2 class="text-lg font-semibold text-gray-900">Confirm Deletion</h2>
        <p class="mt-2 text-gray-600">
          Are you sure you want to delete user {{ currentUser.name }}? This
          action cannot be undone.
        </p>

        <div class="mt-4 flex justify-end space-x-3">
          <button
            @click="showDeleteModal = false"
            class="px-4 py-2 text-gray-700 hover:text-gray-900"
          >
            Cancel
          </button>
          <button
            @click="deleteUser"
            class="px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700"
          >
            Delete
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import UserProfile from "./UserProfile.vue";

  export default {
    name: "UserManagement",

    components: {
      UserProfile,
    },

    data() {
      return {
        currentUser: {
          id: 1,
          name: "John Doe",
          email: "john@example.com",
          role: "Admin",
          avatar: "https://i.pravatar.cc/150?img=8",
          joined: "2025-02-15",
        },
        theme: "light",
        showDeleteModal: false,
      };
    },

    methods: {
      updateUser(updatedUser) {
        // Update the user data when the child component emits an update
        this.currentUser = { ...this.currentUser, ...updatedUser };

        // In a real app, you would persist this data to a backend
        console.log("User updated:", this.currentUser);
      },

      confirmDeleteUser() {
        this.showDeleteModal = true;
      },

      deleteUser() {
        // In a real app, you would call an API to delete the user
        console.log("Deleting user:", this.currentUser.id);

        // Hide the modal
        this.showDeleteModal = false;

        // Show a success message (in a real app)
        alert(`User ${this.currentUser.name} has been deleted`);

        // Reset the current user or fetch a new one
        this.currentUser = {
          id: null,
          name: "",
          email: "",
          role: "",
          avatar: "",
          joined: "",
        };
      },

      toggleTheme() {
        this.theme = this.theme === "light" ? "dark" : "light";
      },
    },
  };
</script>
```

```html
<!-- UserProfile.vue (Child Component) -->
<template>
  <div
    class="rounded-lg overflow-hidden"
    :class="[
      theme === 'dark'
        ? 'bg-gray-800 text-white'
        : 'bg-white text-gray-800 border',
    ]"
  >
    <!-- Profile header -->
    <div class="p-6" :class="theme === 'dark' ? 'bg-gray-700' : 'bg-gray-50'">
      <div class="flex items-center space-x-4">
        <img
          :src="user.avatar"
          :alt="user.name"
          class="w-16 h-16 rounded-full object-cover"
        />

        <div>
          <h2 class="text-lg font-semibold">{{ user.name }}</h2>
          <p
            class="text-sm"
            :class="theme === 'dark' ? 'text-gray-300' : 'text-gray-600'"
          >
            {{ user.role }}
          </p>
        </div>

        <div class="ml-auto">
          <button
            v-if="editable"
            @click="isEditing = !isEditing"
            class="p-2 rounded"
            :class="
              theme === 'dark'
                ? 'bg-gray-600 hover:bg-gray-500'
                : 'bg-gray-200 hover:bg-gray-300'
            "
          >
            <span v-if="isEditing">Cancel</span>
            <span v-else>Edit</span>
          </button>
        </div>
      </div>
    </div>

    <!-- Profile details -->
    <div class="p-6">
      <!-- View mode -->
      <div v-if="!isEditing" class="space-y-4">
        <div class="grid grid-cols-2 gap-4">
          <div>
            <p
              class="text-sm"
              :class="theme === 'dark' ? 'text-gray-400' : 'text-gray-500'"
            >
              Email
            </p>
            <p>{{ user.email }}</p>
          </div>

          <div>
            <p
              class="text-sm"
              :class="theme === 'dark' ? 'text-gray-400' : 'text-gray-500'"
            >
              Joined
            </p>
            <p>{{ formatDate(user.joined) }}</p>
          </div>
        </div>

        <!-- Actions -->
        <div class="flex justify-end mt-6 space-x-2">
          <button
            v-if="editable"
            @click="$emit('delete')"
            class="px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700"
          >
            Delete User
          </button>
        </div>
      </div>

      <!-- Edit mode -->
      <div v-else class="space-y-4">
        <div class="grid grid-cols-2 gap-4">
          <div>
            <label
              class="block text-sm mb-1"
              :class="theme === 'dark' ? 'text-gray-400' : 'text-gray-500'"
            >
              Name
            </label>
            <input
              v-model="formData.name"
              type="text"
              class="w-full p-2 rounded border"
              :class="
                theme === 'dark'
                  ? 'bg-gray-700 border-gray-600'
                  : 'bg-white border-gray-300'
              "
            />
          </div>

          <div>
            <label
              class="block text-sm mb-1"
              :class="theme === 'dark' ? 'text-gray-400' : 'text-gray-500'"
            >
              Email
            </label>
            <input
              v-model="formData.email"
              type="email"
              class="w-full p-2 rounded border"
              :class="
                theme === 'dark'
                  ? 'bg-gray-700 border-gray-600'
                  : 'bg-white border-gray-300'
              "
            />
          </div>

          <div>
            <label
              class="block text-sm mb-1"
              :class="theme === 'dark' ? 'text-gray-400' : 'text-gray-500'"
            >
              Role
            </label>
            <select
              v-model="formData.role"
              class="w-full p-2 rounded border"
              :class="
                theme === 'dark'
                  ? 'bg-gray-700 border-gray-600'
                  : 'bg-white border-gray-300'
              "
            >
              <option value="Admin">Admin</option>
              <option value="Editor">Editor</option>
              <option value="Viewer">Viewer</option>
            </select>
          </div>
        </div>

        <!-- Form actions -->
        <div class="flex justify-end mt-6 space-x-2">
          <button
            @click="isEditing = false"
            class="px-4 py-2 rounded"
            :class="
              theme === 'dark'
                ? 'bg-gray-700 hover:bg-gray-600'
                : 'bg-gray-200 hover:bg-gray-300'
            "
          >
            Cancel
          </button>

          <button
            @click="saveChanges"
            class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
          >
            Save Changes
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: "UserProfile",

    // Props - data passed from parent to child
    props: {
      // Basic prop
      user: {
        type: Object,
        required: true,
      },

      // Prop with default value
      editable: {
        type: Boolean,
        default: false,
      },

      // Prop with validator
      theme: {
        type: String,
        default: "light",
        validator: (value) => {
          return ["light", "dark"].includes(value);
        },
      },
    },

    // Component data
    data() {
      return {
        isEditing: false,
        formData: {
          name: "",
          email: "",
          role: "",
        },
      };
    },

    // Lifecycle hooks
    created() {
      // Initialize form data with user props
      this.resetForm();
    },

    // Watch for prop changes
    watch: {
      user: {
        handler(newUser) {
          // Update form data when user prop changes
          this.resetForm();
        },
        deep: true,
      },
    },

    methods: {
      resetForm() {
        // Clone user data to avoid mutating props directly
        this.formData = {
          name: this.user.name,
          email: this.user.email,
          role: this.user.role,
        };
      },

      saveChanges() {
        // Validate form data
        if (!this.formData.name || !this.formData.email) {
          alert("Name and email are required");
          return;
        }

        // Emit event to parent to update user
        // Option 1: Basic event emission
        this.$emit("update:user", {
          name: this.formData.name,
          email: this.formData.email,
          role: this.formData.role,
        });

        // Exit edit mode
        this.isEditing = false;
      },

      formatDate(dateString) {
        if (!dateString) return "";

        const date = new Date(dateString);
        return date.toLocaleDateString("en-US", {
          year: "numeric",
          month: "long",
          day: "numeric",
        });
      },
    },
  };
</script>
```

**Custom Button Component Example:**

```html
<!-- Button.vue -->
<template>
  <button
    :type="type"
    :disabled="disabled || loading"
    :class="[
      baseClasses,
      sizeClasses,
      variantClasses,
      disabled ? disabledClasses : '',
      block ? 'w-full' : '',
      className,
    ]"
    @click="onClick"
  >
    <!-- Loading spinner -->
    <svg
      v-if="loading"
      class="animate-spin -ml-1 mr-2 h-4 w-4"
      xmlns="http://www.w3.org/2000/svg"
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

    <!-- Left icon -->
    <span v-if="iconLeft && !loading" class="mr-2">
      <slot name="icon-left"></slot>
    </span>

    <!-- Button content -->
    <slot>{{ label }}</slot>

    <!-- Right icon -->
    <span v-if="iconRight" class="ml-2">
      <slot name="icon-right"></slot>
    </span>
  </button>
</template>

<script>
  export default {
    name: "BaseButton",

    props: {
      // Button text
      label: {
        type: String,
        default: "Button",
      },

      // Button type attribute
      type: {
        type: String,
        default: "button",
        validator: (value) => {
          return ["button", "submit", "reset"].includes(value);
        },
      },

      // Visual style variant
      variant: {
        type: String,
        default: "primary",
        validator: (value) => {
          return [
            "primary",
            "secondary",
            "success",
            "danger",
            "warning",
            "info",
            "outline",
            "ghost",
          ].includes(value);
        },
      },

      // Button size
      size: {
        type: String,
        default: "md",
        validator: (value) => {
          return ["sm", "md", "lg", "xl"].includes(value);
        },
      },

      // Button state
      disabled: {
        type: Boolean,
        default: false,
      },

      // Loading state
      loading: {
        type: Boolean,
        default: false,
      },

      // Full width button
      block: {
        type: Boolean,
        default: false,
      },

      // Additional CSS classes
      className: {
        type: String,
        default: "",
      },

      // Has left icon
      iconLeft: {
        type: Boolean,
        default: false,
      },

      // Has right icon
      iconRight: {
        type: Boolean,
        default: false,
      },
    },

    computed: {
      baseClasses() {
        return "inline-flex items-center justify-center font-medium rounded focus:outline-none focus:ring-2 focus:ring-offset-2 transition-colors";
      },

      sizeClasses() {
        const sizes = {
          sm: "px-3 py-1.5 text-sm",
          md: "px-4 py-2 text-base",
          lg: "px-6 py-3 text-lg",
          xl: "px-8 py-4 text-xl",
        };
        return sizes[this.size] || sizes.md;
      },

      variantClasses() {
        const variants = {
          primary:
            "bg-blue-600 hover:bg-blue-700 text-white focus:ring-blue-500",
          secondary:
            "bg-gray-600 hover:bg-gray-700 text-white focus:ring-gray-500",
          success:
            "bg-green-600 hover:bg-green-700 text-white focus:ring-green-500",
          danger: "bg-red-600 hover:bg-red-700 text-white focus:ring-red-500",
          warning:
            "bg-yellow-500 hover:bg-yellow-600 text-white focus:ring-yellow-500",
          info: "bg-blue-500 hover:bg-blue-600 text-white focus:ring-blue-400",
          outline:
            "bg-transparent border border-gray-300 text-gray-700 hover:bg-gray-50 focus:ring-gray-500",
          ghost:
            "bg-transparent hover:bg-gray-100 text-gray-700 focus:ring-gray-500",
        };
        return variants[this.variant] || variants.primary;
      },

      disabledClasses() {
        return "opacity-50 cursor-not-allowed";
      },
    },

    methods: {
      onClick(event) {
        // Don't emit click events when disabled or loading
        if (this.disabled || this.loading) {
          event.preventDefault();
          return;
        }

        // Emit the native event
        this.$emit("click", event);
      },
    },
  };
</script>
```

**Using the Custom Button Component:**

```html
<template>
  <div class="p-6 bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800 mb-6">Button Components</h1>

    <div class="grid grid-cols-2 md:grid-cols-4 gap-4 mb-8">
      <!-- Regular button -->
      <base-button
        label="Primary"
        variant="primary"
        @click="handleClick('Primary')"
      />

      <!-- Success button -->
      <base-button
        label="Success"
        variant="success"
        @click="handleClick('Success')"
      />

      <!-- Danger button -->
      <base-button
        label="Danger"
        variant="danger"
        @click="handleClick('Danger')"
      />

      <!-- Outline button -->
      <base-button
        label="Outline"
        variant="outline"
        @click="handleClick('Outline')"
      />
    </div>

    <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-8">
      <!-- Button with left icon -->
      <base-button
        variant="primary"
        icon-left
        @click="handleClick('With left icon')"
      >
        Add User
        <template #icon-left>
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
              d="M12 6v6m0 0v6m0-6h6m-6 0H6"
            />
          </svg>
        </template>
      </base-button>

      <!-- Button with right icon -->
      <base-button
        variant="secondary"
        icon-right
        @click="handleClick('With right icon')"
      >
        Next Step
        <template #icon-right>
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
              d="M14 5l7 7m0 0l-7 7m7-7H3"
            />
          </svg>
        </template>
      </base-button>

      <!-- Loading button -->
      <base-button
        label="Loading..."
        variant="primary"
        :loading="isLoading"
        @click="toggleLoading"
      />
    </div>

    <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-8">
      <!-- Different sizes -->
      <base-button
        label="Small Button"
        variant="primary"
        size="sm"
        @click="handleClick('Small')"
      />

      <base-button
        label="Medium Button"
        variant="primary"
        size="md"
        @click="handleClick('Medium')"
      />

      <base-button
        label="Large Button"
        variant="primary"
        size="lg"
        @click="handleClick('Large')"
      />
    </div>

    <!-- Full width button -->
    <base-button
      label="Block Button"
      variant="success"
      :block="true"
      @click="handleClick('Block')"
    />

    <!-- Click event log -->
    <div v-if="clickLog.length" class="mt-8">
      <h2 class="text-lg font-medium text-gray-700 mb-2">Click Log</h2>
      <div class="bg-gray-50 p-4 rounded-lg max-h-40 overflow-y-auto">
        <div
          v-for="(log, index) in clickLog"
          :key="index"
          class="py-1 text-sm text-gray-600"
        >
          {{ log }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import BaseButton from "./Button.vue";

  export default {
    name: "ButtonDemo",

    components: {
      BaseButton,
    },

    data() {
      return {
        isLoading: false,
        clickLog: [],
      };
    },

    methods: {
      handleClick(buttonType) {
        const timestamp = new Date().toLocaleTimeString();
        this.clickLog.unshift(`[${timestamp}] Clicked: ${buttonType} button`);

        // Limit log length
        if (this.clickLog.length > 10) {
          this.clickLog = this.clickLog.slice(0, 10);
        }
      },

      toggleLoading() {
        if (this.isLoading) return;

        this.isLoading = true;

        const timestamp = new Date().toLocaleTimeString();
        this.clickLog.unshift(`[${timestamp}] Started loading...`);

        // Simulate async operation
        setTimeout(() => {
          this.isLoading = false;

          const completeTimestamp = new Date().toLocaleTimeString();
          this.clickLog.unshift(`[${completeTimestamp}] Loading complete!`);
        }, 2000);
      },
    },
  };
</script>
```

**Real-World Example:**
In an e-commerce application, a product card component might use props and events for catalog displays:

```html
<!-- ProductCard.vue -->
<template>
  <div
    class="group relative rounded-lg overflow-hidden"
    :class="{ 'opacity-60': !product.inStock }"
  >
    <!-- Badge (if on sale) -->
    <div
      v-if="product.discount > 0"
      class="absolute top-2 right-2 z-10 bg-red-600 text-white text-xs font-bold px-2 py-1 rounded"
    >
      -{{ product.discount }}%
    </div>

    <!-- Out of stock badge -->
    <div
      v-if="!product.inStock"
      class="absolute inset-0 bg-gray-900 bg-opacity-60 flex items-center justify-center z-20"
    >
      <p class="text-white font-medium">Out of Stock</p>
    </div>

    <!-- Wishlist button -->
    <button
      class="absolute top-2 left-2 z-10 p-1.5 rounded-full bg-white text-gray-900 hover:text-red-600 transition-colors"
      @click.prevent="toggleWishlist"
    >
      <svg
        class="h-5 w-5"
        :class="{ 'text-red-600 fill-current': product.inWishlist }"
        fill="none"
        viewBox="0 0 24 24"
        stroke="currentColor"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          stroke-width="2"
          d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z"
        />
      </svg>
    </button>

    <!-- Product image -->
    <div class="aspect-w-1 aspect-h-1 bg-gray-200">
      <img
        :src="product.image"
        :alt="product.name"
        class="w-full h-full object-center object-cover group-hover:opacity-90 transition-opacity"
        :class="{ 'opacity-75': !product.inStock }"
      />
    </div>

    <!-- Product info -->
    <div class="px-4 py-3">
      <h3 class="text-sm font-medium text-gray-900 truncate">
        {{ product.name }}
      </h3>

      <div class="mt-1 flex items-center">
        <!-- Rating stars -->
        <div class="flex items-center">
          <div v-for="i in 5" :key="i" class="flex-shrink-0">
            <svg
              class="h-4 w-4"
              :class="{
                'text-yellow-400 fill-current': i <= Math.round(product.rating),
                'text-gray-300 fill-current': i > Math.round(product.rating),
              }"
              viewBox="0 0 20 20"
            >
              <path
                d="M9.049 2.927c.3-.921 1.603-.921 1.902 0l1.07 3.292a1 1 0 00.95.69h3.462c.969 0 1.371 1.24.588 1.81l-2.8 2.034a1 1 0 00-.364 1.118l1.07 3.292c.3.921-.755 1.688-1.54 1.118l-2.8-2.034a1 1 0 00-1.175 0l-2.8 2.034c-.784.57-1.838-.197-1.539-1.118l1.07-3.292a1 1 0 00-.364-1.118L2.98 8.72c-.783-.57-.38-1.81.588-1.81h3.461a1 1 0 00.951-.69l1.07-3.292z"
              />
            </svg>
          </div>
          <span class="ml-1 text-xs text-gray-500"
            >({{ product.reviewCount }})</span
          >
        </div>

        <div class="ml-auto">
          <!-- Price -->
          <div class="flex items-center">
            <span
              v-if="product.discount > 0"
              class="text-xs text-gray-500 line-through mr-1"
            >
              ${{ product.originalPrice.toFixed(2) }}
            </span>
            <span class="text-sm font-medium text-gray-900">
              ${{ product.price.toFixed(2) }}
            </span>
          </div>
        </div>
      </div>
    </div>

    <!-- Action buttons -->
    <div class="px-4 pb-4">
      <button
        @click="addToCart"
        :disabled="!product.inStock"
        class="w-full py-2 text-center font-medium rounded"
        :class="[
          product.inStock
            ? 'bg-blue-600 text-white hover:bg-blue-700'
            : 'bg-gray-300 text-gray-500 cursor-not-allowed',
        ]"
      >
        {{ product.inCart ? "Added to Cart" : "Add to Cart" }}
      </button>

      <button
        v-if="showDetails"
        @click="viewDetails"
        class="w-full mt-2 py-2 bg-transparent border border-gray-300 text-gray-700 hover:bg-gray-50 text-center font-medium rounded"
      >
        View Details
      </button>
    </div>
  </div>
</template>

<script>
  export default {
    name: "ProductCard",

    props: {
      product: {
        type: Object,
        required: true,
        validator(value) {
          // Basic validation for required fields
          return value.id && value.name && value.price !== undefined;
        },
      },
      showDetails: {
        type: Boolean,
        default: true,
      },
      currency: {
        type: String,
        default: "USD",
      },
    },

    computed: {
      formattedCurrency() {
        const currencySymbols = {
          USD: "$",
          EUR: "€",
          GBP: "£",
          JPY: "¥",
        };
        return currencySymbols[this.currency] || this.currency;
      },
    },

    methods: {
      addToCart() {
        if (!this.product.inStock) return;

        // Emit an event to inform the parent about adding to cart
        this.$emit("add-to-cart", {
          productId: this.product.id,
          quantity: 1,
        });
      },

      toggleWishlist() {
        // Emit an event to inform the parent about wishlist update
        this.$emit("toggle-wishlist", this.product.id);
      },

      viewDetails() {
        // Emit an event to inform the parent about viewing details
        this.$emit("view-details", this.product.id);
      },
    },
  };
</script>
```

**Parent Component Using ProductCard:**

```html
<template>
  <div class="p-6 bg-white rounded-xl shadow-md">
    <div class="flex justify-between items-center mb-6">
      <h1 class="text-xl font-medium text-gray-800">Featured Products</h1>

      <div class="flex items-center space-x-2">
        <label for="sort" class="text-sm text-gray-600">Sort by:</label>
        <select id="sort" v-model="sortBy" class="p-1 text-sm border rounded">
          <option value="name">Name</option>
          <option value="price">Price</option>
          <option value="rating">Rating</option>
        </select>
      </div>
    </div>

    <!-- Product grid -->
    <div class="grid grid-cols-1 md:grid-cols-3 lg:grid-cols-4 gap-6">
      <product-card
        v-for="product in sortedProducts"
        :key="product.id"
        :product="product"
        :show-details="true"
        @add-to-cart="handleAddToCart"
        @toggle-wishlist="handleToggleWishlist"
        @view-details="handleViewDetails"
      />
    </div>

    <!-- Notification -->
    <div
      v-if="notification"
      class="fixed bottom-4 right-4 bg-gray-800 text-white px-4 py-2 rounded shadow-lg"
    >
      {{ notification }}
    </div>
  </div>
</template>

<script>
  import ProductCard from "./ProductCard.vue";

  export default {
    name: "ProductCatalog",

    components: {
      ProductCard,
    },

    data() {
      return {
        products: [
          {
            id: 1,
            name: "Wireless Headphones",
            price: 79.99,
            originalPrice: 99.99,
            discount: 20,
            image: "https://example.com/headphones.jpg",
            rating: 4.5,
            reviewCount: 123,
            inStock: true,
            inWishlist: false,
            inCart: false,
          },
          {
            id: 2,
            name: "Smartphone Stand",
            price: 24.99,
            originalPrice: 24.99,
            discount: 0,
            image: "https://example.com/stand.jpg",
            rating: 4.2,
            reviewCount: 42,
            inStock: true,
            inWishlist: true,
            inCart: false,
          },
          {
            id: 3,
            name: "Bluetooth Speaker",
            price: 49.99,
            originalPrice: 69.99,
            discount: 29,
            image: "https://example.com/speaker.jpg",
            rating: 3.8,
            reviewCount: 78,
            inStock: false,
            inWishlist: false,
            inCart: false,
          },
          {
            id: 4,
            name: "USB-C Hub",
            price: 39.99,
            originalPrice: 39.99,
            discount: 0,
            image: "https://example.com/usb-hub.jpg",
            rating: 4.0,
            reviewCount: 56,
            inStock: true,
            inWishlist: false,
            inCart: true,
          },
        ],
        sortBy: "name",
        notification: null,
        notificationTimeout: null,
      };
    },

    computed: {
      sortedProducts() {
        return [...this.products].sort((a, b) => {
          switch (this.sortBy) {
            case "price":
              return a.price - b.price;
            case "rating":
              return b.rating - a.rating;
            default:
              return a.name.localeCompare(b.name);
          }
        });
      },
    },

    methods: {
      handleAddToCart({ productId, quantity }) {
        // Find the product
        const product = this.products.find((p) => p.id === productId);
        if (!product) return;

        // Update product state
        product.inCart = true;

        // Show notification
        this.showNotification(`Added ${product.name} to cart`);

        // In a real app, this would call a store action or API
        console.log(`Adding to cart: ${quantity}x ${product.name}`);
      },

      handleToggleWishlist(productId) {
        // Find the product
        const product = this.products.find((p) => p.id === productId);
        if (!product) return;

        // Toggle wishlist state
        product.inWishlist = !product.inWishlist;

        // Show notification
        const action = product.inWishlist ? "added to" : "removed from";
        this.showNotification(`${product.name} ${action} wishlist`);

        // In a real app, this would call a store action or API
        console.log(
          `Toggling wishlist for: ${product.name}, now: ${product.inWishlist}`
        );
      },

      handleViewDetails(productId) {
        // Find the product
        const product = this.products.find((p) => p.id === productId);
        if (!product) return;

        // In a real app, this would navigate to product detail page
        console.log(`Viewing details for: ${product.name}`);

        // Show notification
        this.showNotification(`Viewing details for ${product.name}`);

        // In a real app, you might use router:
        // this.$router.push(`/product/${productId}`)
      },

      showNotification(message) {
        // Clear existing timeout
        if (this.notificationTimeout) {
          clearTimeout(this.notificationTimeout);
        }

        // Show notification
        this.notification = message;

        // Hide after 3 seconds
        this.notificationTimeout = setTimeout(() => {
          this.notification = null;
        }, 3000);
      },
    },
  };
</script>
```

**Common Pitfalls:**

- **Mutating Props**: Directly changing prop values (props are meant to be one-way)
- **Prop Types**: Not validating prop types or not providing defaults when appropriate
- **Excessive Props**: Passing too many props instead of structuring data more effectively
- **Event Naming**: Inconsistent event naming conventions
- **Prop Drilling**: Passing props through many levels of components instead of using Provide/Inject
- **Missing Validations**: Not validating props or not handling edge cases

**Frequently Asked Questions:**

1. **Q: Can I modify props directly in a child component?**

   A: No, props are meant to be read-only in the child component. This ensures one-way data flow and makes your application more predictable. Instead:

   - Create a local copy in data if you need to modify it:

     ```js
     data() {
       return {
         localValue: this.propValue
       }
     }
     ```

   - Use computed properties for transformations:

     ```js
     computed: {
       processedValue() {
         return this.propValue.toUpperCase()
       }
     }
     ```

   - Emit events to ask the parent to update the prop:
     ```js
     methods: {
       updateValue(newValue) {
         this.$emit('update:value', newValue)
       }
     }
     ```

2. **Q: What's the best practice for prop validation?**

   A: Comprehensive prop validation improves component reliability:

   ```js
   props: {
     // Basic type validation
     username: String,

     // Type with additional options
     age: {
       type: Number,
       required: true,
       validator: value => value >= 0
     },

     // Multiple possible types
     identifier: [String, Number],

     // Complex object with shape validation
     user: {
       type: Object,
       required: true,
       validator: user => {
         return user.id && user.name && user.email
       }
     },

     // Custom validator function for complex rules
     status: {
       type: String,
       default: 'active',
       validator: value => ['active', 'inactive', 'pending'].includes(value)
     }
   }
   ```

   Always provide sensible defaults for optional props and document the expected shape of complex objects.

3. **Q: What's the difference between events and the v-model directive?**

   A: `v-model` is syntactic sugar for binding a value prop and handling an input event:

   ```html
   <!-- Basic v-model -->
   <input v-model="searchText" />

   <!-- Is equivalent to -->
   <input :value="searchText" @input="searchText = $event.target.value" />
   ```

   For custom components, `v-model` binds to a prop called `modelValue` and listens for an `update:modelValue` event:

   ```html
   <!-- Using v-model with a custom component -->
   <custom-input v-model="searchText">
     <!-- Is equivalent to -->
     <custom-input
       :modelValue="searchText"
       @update:modelValue="newValue => searchText = newValue"
     ></custom-input
   ></custom-input>
   ```

   You can also create custom v-model bindings using the `model` option:

   ```js
   // In the component
   export default {
     model: {
       prop: "value",
       event: "change",
     },
     props: {
       value: String,
     },
   };
   ```

### 4. Lifecycle Hooks

**Concise Explanation:**
Lifecycle hooks are special methods that Vue calls at specific stages of a component's existence. They allow you to run code at critical moments: when the component is created, mounted to the DOM, updated, or destroyed. Understanding these hooks is essential for managing resources, fetching data, and interacting with external libraries.

**Where to Use:**

- **beforeCreate/created**: For initializing data, setting up reactive properties
- **beforeMount/mounted**: For DOM interactions, third-party library initialization
- **beforeUpdate/updated**: For reacting to data changes and DOM updates
- **beforeUnmount/unmounted**: For cleanup and resource management
- **errorCaptured**: For catching and handling errors in child components
- **activated/deactivated**: For components inside `<keep-alive>` to handle activation state

**Code Snippet:**

```html
<template>
  <div class="p-6 bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800 mb-4">Component Lifecycle</h1>

    <div class="bg-gray-50 p-4 rounded mb-6">
      <h2 class="font-medium text-gray-700 mb-2">Current State</h2>
      <div class="grid grid-cols-2 gap-2">
        <div class="text-sm text-gray-600">Component Status:</div>
        <div class="text-sm font-medium">{{ status }}</div>

        <div class="text-sm text-gray-600">Mounted:</div>
        <div class="text-sm font-medium">{{ isMounted ? "Yes" : "No" }}</div>

        <div class="text-sm text-gray-600">Update Count:</div>
        <div class="text-sm font-medium">{{ updateCount }}</div>

        <div class="text-sm text-gray-600">Error State:</div>
        <div class="text-sm font-medium">
          {{ hasError ? "Error" : "Normal" }}
        </div>
      </div>
    </div>

    <!-- Lifecycle demonstration controls -->
    <div class="space-y-4">
      <div>
        <label class="block text-sm font-medium text-gray-700 mb-1">
          Counter (updates component)
        </label>
        <div class="flex items-center">
          <button
            @click="counter--"
            class="px-3 py-1 bg-gray-200 rounded-l text-gray-700"
          >
            -
          </button>
          <div class="px-4 py-1 border-t border-b text-center w-16">
            {{ counter }}
          </div>
          <button
            @click="counter++"
            class="px-3 py-1 bg-gray-200 rounded-r text-gray-700"
          >
            +
          </button>
        </div>
      </div>

      <div>
        <label class="block text-sm font-medium text-gray-700 mb-1">
          Error Handling
        </label>
        <button
          @click="triggerError"
          class="px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700"
        >
          Trigger Error
        </button>
      </div>

      <div>
        <label class="block text-sm font-medium text-gray-700 mb-1">
          Component Lifecycle
        </label>
        <button
          @click="unmountComponent"
          v-if="isMounted"
          class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
        >
          Unmount Component
        </button>
        <button
          @click="mountComponent"
          v-else
          class="px-4 py-2 bg-green-600 text-white rounded hover:bg-green-700"
        >
          Mount Component
        </button>
      </div>
    </div>

    <!-- Child component for lifecycle demonstration -->
    <lifecycle-child
      v-if="isMounted"
      :counter="counter"
      :should-error="hasError"
      @error="handleChildError"
    />

    <!-- Lifecycle log -->
    <div class="mt-6">
      <h2 class="font-medium text-gray-700 mb-2">Lifecycle Events</h2>
      <div
        class="bg-gray-800 text-green-400 p-4 rounded font-mono text-sm h-64 overflow-y-auto"
      >
        <div v-for="(log, index) in lifecycleLogs" :key="index" class="py-1">
          <span class="text-gray-400">[{{ log.time }}]</span>
          <span
            :class="{
              'text-yellow-400': log.type === 'warning',
              'text-red-400': log.type === 'error',
              'text-blue-400': log.type === 'info',
            }"
          >
            {{ log.message }}
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import LifecycleChild from "./LifecycleChild.vue";

  export default {
    name: "LifecycleDemo",

    components: {
      LifecycleChild,
    },

    data() {
      return {
        status: "Initializing",
        counter: 0,
        updateCount: 0,
        isMounted: true,
        hasError: false,
        lifecycleLogs: [],
        intervalId: null,
      };
    },

    // Lifecycle Hooks

    beforeCreate() {
      // Called before the instance is created
      // Data observation and event/watcher setup haven't happened yet
      this.logLifecycle("Parent beforeCreate: Instance is being initialized");

      // Note: 'this.status' is undefined here because data() hasn't been processed
      console.log("  Status in beforeCreate:", this.status); // undefined
    },

    created() {
      // Called after the instance is created
      // Data observation, computed properties, methods, and watch/event callbacks have been set up
      this.logLifecycle("Parent created: Instance has been created");
      this.status = "Created";

      // Good place for API calls and initialization that doesn't require DOM
      this.logLifecycle("Parent created: Setting up data interval", "info");
      this.setupInterval();
    },

    beforeMount() {
      // Called before the component is inserted into the DOM
      this.logLifecycle("Parent beforeMount: About to mount to DOM");
      this.status = "Mounting";
    },

    mounted() {
      // Called after the component is inserted into the DOM
      // All DOM nodes are now accessible
      this.logLifecycle("Parent mounted: Component is now in DOM");
      this.status = "Mounted";

      // Good place to initialize libraries that need DOM access
      this.logLifecycle("Parent mounted: DOM is accessible now", "info");
      console.log("  DOM element available:", this.$el);
    },

    beforeUpdate() {
      // Called when data changes, before the DOM is re-rendered
      this.logLifecycle(
        `Parent beforeUpdate: Data changed, update ${
          this.updateCount + 1
        } pending`
      );

      // You can access the old DOM here before it gets updated
      console.log(
        "  Counter in DOM before update:",
        this.$el.querySelector(".counter-value")?.textContent
      );
      console.log("  Counter in data:", this.counter);
    },

    updated() {
      // Called after data changes and the DOM has been re-rendered
      this.updateCount++;
      this.logLifecycle(
        `Parent updated: DOM has been re-rendered (update ${this.updateCount})`
      );

      // The DOM is now in sync with your data
      console.log(
        "  Counter in DOM after update:",
        this.$el.querySelector(".counter-value")?.textContent
      );
    },

    beforeUnmount() {
      // Called right before a component is destroyed/unmounted
      this.logLifecycle(
        "Parent beforeUnmount: Component is about to be destroyed"
      );
      this.status = "Unmounting";

      // Good place to start cleanup
      this.logLifecycle("Parent beforeUnmount: Starting cleanup", "info");
    },

    unmounted() {
      // Called after a component has been destroyed/unmounted
      this.logLifecycle("Parent unmounted: Component has been destroyed");
      this.status = "Unmounted";

      // Clean up resources, event listeners, etc.
      this.logLifecycle("Parent unmounted: Clearing interval", "info");
      this.clearInterval();
    },

    errorCaptured(err, vm, info) {
      // Captures errors from this component and its child components
      this.logLifecycle(
        `Parent errorCaptured: Error in ${vm.$options.name}: ${err.message}`,
        "error"
      );
      this.status = "Error";

      // Decide whether to let the error propagate further
      return false; // Prevent propagation
    },

    methods: {
      logLifecycle(message, type = "normal") {
        const time = new Date().toLocaleTimeString();
        this.lifecycleLogs.unshift({ message, time, type });

        // Limit the number of logs
        if (this.lifecycleLogs.length > 100) {
          this.lifecycleLogs.pop();
        }
      },

      setupInterval() {
        // Set up a data refresh interval
        this.intervalId = setInterval(() => {
          // Simulate data refresh
          this.logLifecycle("Data refresh check", "info");
        }, 10000); // Every 10 seconds
      },

      clearInterval() {
        if (this.intervalId) {
          clearInterval(this.intervalId);
          this.intervalId = null;
        }
      },

      unmountComponent() {
        this.isMounted = false;
      },

      mountComponent() {
        this.isMounted = true;
      },

      triggerError() {
        this.hasError = true;
      },

      handleChildError(error) {
        this.logLifecycle(`Received error from child: ${error}`, "warning");

        // Reset the error state after a delay
        setTimeout(() => {
          this.hasError = false;
        }, 2000);
      },
    },
  };
</script>
```

```html
<!-- LifecycleChild.vue -->
<template>
  <div class="mt-6 border border-gray-200 rounded-lg p-4 bg-gray-50">
    <h2 class="font-medium text-gray-700 mb-2">Child Component</h2>

    <div class="mb-4">
      <div class="text-sm text-gray-600">Counter Value:</div>
      <div class="text-lg font-medium counter-value">{{ counter }}</div>
    </div>

    <div v-if="!shouldError">
      <div class="text-sm text-gray-600">Status:</div>
      <div class="text-sm font-medium">{{ status }}</div>
    </div>

    <!-- This will cause an error when shouldError is true -->
    <div v-if="shouldError">{{ nonExistentMethod() }}</div>
  </div>
</template>

<script>
  export default {
    name: "LifecycleChild",

    props: {
      counter: {
        type: Number,
        default: 0,
      },
      shouldError: {
        type: Boolean,
        default: false,
      },
    },

    data() {
      return {
        status: "Initializing",
        chartInstance: null,
      };
    },

    // Lifecycle hooks

    beforeCreate() {
      console.log("Child beforeCreate");
    },

    created() {
      console.log("Child created");
      this.status = "Created";
    },

    beforeMount() {
      console.log("Child beforeMount");
      this.status = "Mounting";
    },

    mounted() {
      console.log("Child mounted");
      this.status = "Mounted";

      // Simulate initializing a third-party chart library
      this.initChart();
    },

    beforeUpdate() {
      console.log("Child beforeUpdate");
    },

    updated() {
      console.log("Child updated");

      // Update the chart when component updates
      this.updateChart();
    },

    beforeUnmount() {
      console.log("Child beforeUnmount");
      this.status = "Unmounting";
    },

    unmounted() {
      console.log("Child unmounted");
      this.status = "Unmounted";

      // Clean up the chart
      this.destroyChart();
    },

    watch: {
      shouldError(newValue) {
        if (newValue) {
          // Emit an event to parent before we error
          this.$emit("error", "About to trigger an error");
        }
      },
    },

    methods: {
      initChart() {
        console.log("Initializing chart...");

        // Simulate a chart library initialization
        this.chartInstance = {
          render: () => console.log("Chart rendered"),
          update: () => console.log("Chart updated"),
          destroy: () => console.log("Chart destroyed"),
        };

        // Initial render
        this.chartInstance.render();
      },

      updateChart() {
        if (this.chartInstance) {
          this.chartInstance.update();
        }
      },

      destroyChart() {
        if (this.chartInstance) {
          this.chartInstance.destroy();
          this.chartInstance = null;
        }
      },
    },
  };
</script>
```

**Data Fetching in Lifecycle Hooks:**

```html
<template>
  <div class="p-6 bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800 mb-4">User Dashboard</h1>

    <!-- Loading state -->
    <div v-if="loading" class="flex items-center justify-center py-12">
      <div class="animate-spin rounded-full h-12 w-12 border-4 border-blue-500 border-t-transparent"></div>
    </div>

    <!-- Error state -->
    <div v-else-if="error" class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded">
      <p class="font-bold">Error</p>
      <p>{{ error }}</p>
      <button
        @click="fetchData"
        class="mt-2 px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700"
      >
        Try Again
      </button>
    </div>

    <!-- Data display -->
    <div v-else-if="user" class="space-y-6">
      <!-- User profile -->
      <div class="flex items-center space-x-4">
        <img
          :src="user.avatar"
          :alt="user.name"
          class="w-16 h-16 rounded-full object-cover"
        />
        <div>
          <h2 class="text-lg font-semibold">{{ user.name }}</h2>
          <p class="text-gray-600">{{ user.email }}</p>
        </div>
      </div>

      <!-- Stats -->
      <div class="grid grid-cols-3 gap-4">
        <div class="bg-blue-100 p-4 rounded-lg">
          <div class="text-sm text-blue-800">Posts</div>
          <div class="text-2xl font-bold text-blue-900">{{ stats.posts }}</div>
        </div>
        <div class="bg-green-100 p-4 rounded-lg">
          <div class="text-sm text-green-800">Followers</div>
          <div class="text-2xl font-bold text-green-900">{{ stats.followers }}</div>
        </div>
        <div class="bg-purple-100 p-4 rounded-lg">
          <div class="text-sm text-purple-800">Following</div>
          <div class="text-2xl font-bold text-purple-900">{{ stats.following }}</div>
        </div>
      </div>

      <!-- Recent activity -->
      <div>
        <h3 class="text-lg font-medium mb-3">Recent Activity</h3>
        <div v-if="activities.length === 0" class="text-gray-500 text-center py-4">
          No recent activity
        </div>
        <div v-else class="space-y-3">
          <div
            v-for="(activity, index) in activities"
            :key="index"
            class="flex items-start p-3 rounded bg-gray-50"
          >
            <div
              class="h-8 w-8 rounded-full flex items-center justify-center mr-3"
              :class="{
                'bg-blue-100 text-blue-800': activity.type === 'post',
                'bg-green-100 text-green-800': activity.type === 'follow',
                'bg-yellow-100 text-yellow-800': activity.type === 'like'
              }"
            >
              <span v-if="activity.type === 'post'>✍️</span>
              <span v-else-if="activity.type === 'follow'">👤</span>
              <span v-else-if="activity.type === 'like'">👍</span>
            </div>
            <div>
              <p>{{ activity.message }}</p>
              <p class="text-sm text-gray-500">{{ formatDate(activity.timestamp) }}</p>
            </div>
          </div>
        </div>
      </div>

      <!-- Refresh button -->
      <div class="flex justify-end">
        <button
          @click="fetchData"
          class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
          :disabled="loading"
        >
          Refresh Data
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'UserDashboard',

  data() {
    return {
      user: null,
      stats: {
        posts: 0,
        followers: 0,
        following: 0
      },
      activities: [],
      loading: true,
      error: null,
      dataInterval: null
    }
  },

  // Fetch data when the component is created
  created() {
    console.log('Created: Setting up initial data fetch')

    // Fetch the data
    this.fetchData()
  },

  // Set up polling when the component is mounted
  mounted() {
    console.log('Mounted: Setting up data polling interval')

    // Set up interval for data refresh
    this.dataInterval = setInterval(() => {
      console.log('Auto-refreshing data...')
      this.fetchData(true) // true = silent update (no loading indicator)
    }, 60000) // Refresh every minute
  },

  // Clean up when the component is unmounted
  beforeUnmount() {
    console.log('BeforeUnmount: Cleaning up interval')

    // Clear the interval
    if (this.dataInterval) {
      clearInterval(this.dataInterval)
    }
  },

  // Error handling
  errorCaptured(err, vm, info) {
    console.error('Error captured in dashboard:', err, info)

    // Set error state
    this.error = `An unexpected error occurred: ${err.message}`
    this.loading = false

    // Prevent error propagation
    return false
  },

  methods: {
    async fetchData(silent = false) {
      // Don't show loading indicator for silent updates
      if (!silent) {
        this.loading = true
        this.error = null
      }

      try {
        // Simulate API calls
        await this.fetchUserProfile()
        await this.fetchUserStats()
        await this.fetchUserActivities()

        this.loading = false
      } catch (err) {
        console.error('Error fetching data:', err)
        this.error = err.message || 'Failed to fetch data'
        this.loading = false
      }
    },

    async fetchUserProfile() {
      // Simulate API delay
      await new Promise(resolve => setTimeout(resolve, 800))

      // Mock API response
      this.user = {
        id: 1,
        name: 'Jane Doe',
        email: 'jane.doe@example.com',
        avatar: 'https://i.pravatar.cc/150?img=5'
      }
    },

    async fetchUserStats() {
      // Simulate API delay
      await new Promise(resolve => setTimeout(resolve, 500))

      // Mock API response
      this.stats = {
        posts: 42,
        followers: 1024,
        following: 256
      }
    },

    async fetchUserActivities() {
      // Simulate API delay
      await new Promise(resolve => setTimeout(resolve, 700))

      // Mock API response
      this.activities = [
        {
          type: 'post',
          message: 'You published a new post: "Getting Started with Vue 3"',
          timestamp: new Date(Date.now() - 3600000) // 1 hour ago
        },
        {
          type: 'follow',
          message: 'John Smith started following you',
          timestamp: new Date(Date.now() - 86400000) // 1 day ago
        },
        {
          type: 'like',
          message: 'Alice Johnson liked your post: "Vue Component Patterns"',
          timestamp: new Date(Date.now() - 172800000) // 2 days ago
        }
      ]
    },

    formatDate(date) {
      if (!date) return ''

      const now = new Date()
      const diff = now - date // difference in milliseconds

      // Less than a minute ago
      if (diff < 60000) {
        return 'Just now'
      }

      // Less than an hour ago
      if (diff < 3600000) {
        const minutes = Math.floor(diff / 60000)
        return `${minutes} minute${minutes > 1 ? 's' : ''} ago`
      }

      // Less than a day ago
      if (diff < 86400000) {
        const hours = Math.floor(diff / 3600000)
        return `${hours} hour${hours > 1 ? 's' : ''} ago`
      }

      // Less than a week ago
      if (diff < 604800000) {
        const days = Math.floor(diff / 86400000)
        return `${days} day${days > 1 ? 's' : ''} ago`
      }

      // Format as date
      return date.toLocaleDateString('en-US', {
        year: 'numeric',
        month: 'short',
        day: 'numeric'
      })
    }
  }
}
</script>
```

**Third-Party Library Integration:**

```html
<template>
  <div class="p-6 bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800 mb-4">Sales Analytics</h1>

    <!-- Chart controls -->
    <div class="flex flex-wrap gap-4 mb-6">
      <div>
        <label class="block text-sm font-medium text-gray-700 mb-1">
          Time Period
        </label>
        <select v-model="period" class="px-3 py-2 border rounded">
          <option value="7days">Last 7 Days</option>
          <option value="30days">Last 30 Days</option>
          <option value="90days">Last 90 Days</option>
          <option value="year">This Year</option>
        </select>
      </div>

      <div>
        <label class="block text-sm font-medium text-gray-700 mb-1">
          Chart Type
        </label>
        <select v-model="chartType" class="px-3 py-2 border rounded">
          <option value="line">Line Chart</option>
          <option value="bar">Bar Chart</option>
          <option value="pie">Pie Chart</option>
        </select>
      </div>

      <div>
        <label class="block text-sm font-medium text-gray-700 mb-1">
          Data Series
        </label>
        <div class="flex flex-wrap gap-2">
          <label class="inline-flex items-center">
            <input
              type="checkbox"
              v-model="series.revenue"
              class="h-4 w-4 text-blue-600 rounded"
            />
            <span class="ml-2 text-sm">Revenue</span>
          </label>
          <label class="inline-flex items-center">
            <input
              type="checkbox"
              v-model="series.orders"
              class="h-4 w-4 text-blue-600 rounded"
            />
            <span class="ml-2 text-sm">Orders</span>
          </label>
          <label class="inline-flex items-center">
            <input
              type="checkbox"
              v-model="series.customers"
              class="h-4 w-4 text-blue-600 rounded"
            />
            <span class="ml-2 text-sm">Customers</span>
          </label>
        </div>
      </div>
    </div>

    <!-- Chart container -->
    <div class="bg-gray-50 p-4 rounded-lg">
      <div v-if="loading" class="flex items-center justify-center h-80">
        <div
          class="animate-spin rounded-full h-12 w-12 border-4 border-blue-500 border-t-transparent"
        ></div>
      </div>

      <div v-else ref="chartContainer" class="h-80 w-full"></div>
    </div>

    <!-- Data summary -->
    <div class="mt-6 grid grid-cols-1 md:grid-cols-3 gap-4">
      <div class="p-4 bg-blue-50 rounded-lg">
        <div class="text-sm text-blue-800">Total Revenue</div>
        <div class="text-2xl font-bold text-blue-900">
          ${{ formatNumber(summaryData.revenue) }}
        </div>
        <div class="text-sm text-blue-700">
          <span v-if="summaryData.revenueChange > 0" class="text-green-600"
            >↑ {{ summaryData.revenueChange }}%</span
          >
          <span v-else-if="summaryData.revenueChange < 0" class="text-red-600"
            >↓ {{ Math.abs(summaryData.revenueChange) }}%</span
          >
          <span v-else>0%</span>
          vs previous period
        </div>
      </div>

      <div class="p-4 bg-yellow-50 rounded-lg">
        <div class="text-sm text-yellow-800">Total Orders</div>
        <div class="text-2xl font-bold text-yellow-900">
          {{ formatNumber(summaryData.orders) }}
        </div>
        <div class="text-sm text-yellow-700">
          <span v-if="summaryData.ordersChange > 0" class="text-green-600"
            >↑ {{ summaryData.ordersChange }}%</span
          >
          <span v-else-if="summaryData.ordersChange < 0" class="text-red-600"
            >↓ {{ Math.abs(summaryData.ordersChange) }}%</span
          >
          <span v-else>0%</span>
          vs previous period
        </div>
      </div>

      <div class="p-4 bg-green-50 rounded-lg">
        <div class="text-sm text-green-800">New Customers</div>
        <div class="text-2xl font-bold text-green-900">
          {{ formatNumber(summaryData.customers) }}
        </div>
        <div class="text-sm text-green-700">
          <span v-if="summaryData.customersChange > 0" class="text-green-600"
            >↑ {{ summaryData.customersChange }}%</span
          >
          <span v-else-if="summaryData.customersChange < 0" class="text-red-600"
            >↓ {{ Math.abs(summaryData.customersChange) }}%</span
          >
          <span v-else>0%</span>
          vs previous period
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  // Import Chart.js
  import Chart from "chart.js/auto";

  export default {
    name: "SalesAnalytics",

    data() {
      return {
        // Chart configuration
        period: "30days",
        chartType: "line",
        series: {
          revenue: true,
          orders: true,
          customers: false,
        },

        // Data state
        loading: true,
        chart: null,
        chartData: null,
        summaryData: {
          revenue: 0,
          revenueChange: 0,
          orders: 0,
          ordersChange: 0,
          customers: 0,
          customersChange: 0,
        },
      };
    },

    // Watch for configuration changes to update the chart
    watch: {
      period() {
        this.fetchData();
      },

      chartType() {
        this.updateChart();
      },

      series: {
        handler() {
          this.updateChart();
        },
        deep: true,
      },
    },

    // Use the created hook to fetch initial data
    created() {
      console.log("Created: Fetching initial chart data");
      this.fetchData();
    },

    // Use the mounted hook to initialize the chart after the DOM is ready
    mounted() {
      console.log("Mounted: Setting up chart instance");
      // Chart will be initialized after data is loaded
    },

    // Use the beforeUnmount hook to clean up the chart instance
    beforeUnmount() {
      console.log("BeforeUnmount: Destroying chart instance");
      if (this.chart) {
        this.chart.destroy();
        this.chart = null;
      }
    },

    methods: {
      async fetchData() {
        this.loading = true;

        try {
          // Simulate API delay
          await new Promise((resolve) => setTimeout(resolve, 1200));

          // Simulate API response - in a real app, this would be a fetch call
          this.chartData = this.generateMockData();

          // Calculate summary data
          this.calculateSummary();

          // Initialize or update the chart
          this.$nextTick(() => {
            this.initOrUpdateChart();
            this.loading = false;
          });
        } catch (error) {
          console.error("Error fetching data:", error);
          this.loading = false;
        }
      },

      initOrUpdateChart() {
        const ctx = this.$refs.chartContainer;

        if (!ctx) {
          console.error("Chart container not found");
          return;
        }

        // Destroy existing chart if it exists
        if (this.chart) {
          this.chart.destroy();
        }

        // Create new chart
        this.chart = new Chart(ctx, this.getChartConfig());
      },

      updateChart() {
        if (!this.chart || !this.chartData) return;

        const config = this.getChartConfig();

        // Update chart type and data
        this.chart.config.type = config.type;
        this.chart.data = config.data;
        this.chart.options = config.options;

        // Update the chart
        this.chart.update();
      },

      getChartConfig() {
        // Base configuration
        const config = {
          type: this.chartType,
          data: {
            labels: this.chartData.labels,
            datasets: [],
          },
          options: {
            responsive: true,
            maintainAspectRatio: false,
            interaction: {
              mode: "index",
              intersect: false,
            },
            plugins: {
              legend: {
                position: "top",
              },
              tooltip: {
                callbacks: {
                  label: function (context) {
                    let label = context.dataset.label || "";
                    if (label) {
                      label += ": ";
                    }
                    if (context.parsed.y !== null) {
                      if (context.dataset.yAxisID === "revenue") {
                        label += "$" + context.parsed.y.toLocaleString();
                      } else {
                        label += context.parsed.y.toLocaleString();
                      }
                    }
                    return label;
                  },
                },
              },
            },
          },
        };

        // Add datasets based on selected series
        if (this.series.revenue) {
          config.data.datasets.push({
            label: "Revenue",
            data: this.chartData.revenue,
            borderColor: "#3b82f6",
            backgroundColor: "rgba(59, 130, 246, 0.5)",
            yAxisID: "revenue",
            borderWidth: 2,
          });
        }

        if (this.series.orders) {
          config.data.datasets.push({
            label: "Orders",
            data: this.chartData.orders,
            borderColor: "#f59e0b",
            backgroundColor: "rgba(245, 158, 11, 0.5)",
            yAxisID: "count",
            borderWidth: 2,
          });
        }

        if (this.series.customers) {
          config.data.datasets.push({
            label: "New Customers",
            data: this.chartData.customers,
            borderColor: "#10b981",
            backgroundColor: "rgba(16, 185, 129, 0.5)",
            yAxisID: "count",
            borderWidth: 2,
          });
        }

        // Special config for pie chart
        if (this.chartType === "pie") {
          // For pie charts, we need to restructure the data
          const lastIndex = this.chartData.labels.length - 1;
          const datasets = [];

          if (this.series.revenue) {
            datasets.push({
              label: "Revenue",
              data: [this.chartData.revenue[lastIndex]],
              backgroundColor: ["rgba(59, 130, 246, 0.5)"],
            });
          }

          if (this.series.orders) {
            datasets.push({
              label: "Orders",
              data: [this.chartData.orders[lastIndex]],
              backgroundColor: ["rgba(245, 158, 11, 0.5)"],
            });
          }

          if (this.series.customers) {
            datasets.push({
              label: "New Customers",
              data: [this.chartData.customers[lastIndex]],
              backgroundColor: ["rgba(16, 185, 129, 0.5)"],
            });
          }

          config.data = {
            labels: datasets.map((d) => d.label),
            datasets: [
              {
                data: datasets.map((d) => d.data[0]),
                backgroundColor: datasets.map((d) => d.backgroundColor[0]),
                hoverOffset: 4,
              },
            ],
          };
        } else {
          // Add scales configuration for line/bar charts
          config.options.scales = {
            revenue: {
              type: "linear",
              display: this.series.revenue,
              position: "left",
              title: {
                display: true,
                text: "Revenue ($)",
              },
            },
            count: {
              type: "linear",
              display: this.series.orders || this.series.customers,
              position: "right",
              title: {
                display: true,
                text: "Count",
              },
              grid: {
                drawOnChartArea: false,
              },
            },
          };
        }

        return config;
      },

      calculateSummary() {
        // Calculate summary data from chart data
        const data = this.chartData;
        const lastIndex = data.labels.length - 1;
        const midIndex = Math.floor(data.labels.length / 2);

        // Current values (latest data point)
        this.summaryData.revenue = data.revenue[lastIndex];
        this.summaryData.orders = data.orders[lastIndex];
        this.summaryData.customers = data.customers[lastIndex];

        // Calculate percent change from middle to latest
        const calculateChange = (current, previous) => {
          if (previous === 0) return 0;
          return Math.round(((current - previous) / previous) * 100);
        };

        this.summaryData.revenueChange = calculateChange(
          data.revenue[lastIndex],
          data.revenue[midIndex]
        );

        this.summaryData.ordersChange = calculateChange(
          data.orders[lastIndex],
          data.orders[midIndex]
        );

        this.summaryData.customersChange = calculateChange(
          data.customers[lastIndex],
          data.customers[midIndex]
        );
      },

      generateMockData() {
        // Generate mock data based on selected period
        let days;
        switch (this.period) {
          case "7days":
            days = 7;
            break;
          case "30days":
            days = 30;
            break;
          case "90days":
            days = 90;
            break;
          case "year":
            days = 365;
            break;
          default:
            days = 30;
        }

        const labels = [];
        const revenue = [];
        const orders = [];
        const customers = [];

        // Generate dates for labels
        for (let i = days; i >= 0; i--) {
          const date = new Date();
          date.setDate(date.getDate() - i);
          labels.push(
            date.toLocaleDateString("en-US", { month: "short", day: "numeric" })
          );
        }

        // Generate mock data with some randomness but a general trend
        let baseRevenue = 5000;
        let baseOrders = 100;
        let baseCustomers = 20;

        for (let i = 0; i <= days; i++) {
          // Add some daily variation
          const dailyVariation = 0.2; // 20% variation
          const revenueVariation =
            baseRevenue *
            (1 + (Math.random() * dailyVariation - dailyVariation / 2));
          const ordersVariation =
            baseOrders *
            (1 + (Math.random() * dailyVariation - dailyVariation / 2));
          const customersVariation =
            baseCustomers *
            (1 + (Math.random() * dailyVariation - dailyVariation / 2));

          revenue.push(Math.round(baseRevenue + revenueVariation));
          orders.push(Math.round(baseOrders + ordersVariation));
          customers.push(Math.round(baseCustomers + customersVariation));

          // Apply a slight upward trend
          baseRevenue *= 1.01;
          baseOrders *= 1.005;
          baseCustomers *= 1.008;
        }

        return { labels, revenue, orders, customers };
      },

      formatNumber(value) {
        // Format large numbers with commas
        return value.toLocaleString();
      },
    },
  };
</script>
```

**Real-World Example:**
In a video streaming application, lifecycle hooks are critical for managing player state, tracking analytics, and optimizing performance. For example, you would use:

- `created` to initialize player settings and prepare playlists
- `mounted` to initialize the video player with the DOM element
- `beforeUnmount` to save playback position and viewing history
- `errorCaptured` to log and handle player errors gracefully
- `activated`/`deactivated` to pause/resume video when navigating between pages

**Common Pitfalls:**

- **DOM Manipulation Too Early**: Trying to access DOM elements before `mounted`
- **Duplicate Setup**: Initializing the same resources in multiple hooks
- **Missing Cleanup**: Not cleaning up resources in `beforeUnmount`
- **Heavy Processing in Hooks**: Doing CPU-intensive work in hooks that block rendering
- **Not Using Async/Await**: Blocking the main thread with synchronous API calls
- **Hook Ordering**: Assuming hooks will execute in a specific order across components

**Frequently Asked Questions:**

1. **Q: When should I use created vs mounted?**

   A: Use each hook for its specific purpose:

   - **created**: For initializing non-DOM data, setting up reactive properties, or making API calls that don't require DOM access

     ```js
     created() {
       // Initialize data
       this.initialized = true

       // Set up reactive properties
       this.formData = { ...this.initialValues }

       // Make API calls
       this.fetchUserData()
     }
     ```

   - **mounted**: For DOM interactions, initializing third-party libraries that need DOM access, or setting up event listeners

     ```js
     mounted() {
       // Access DOM elements
       const element = this.$refs.container

       // Initialize third-party libraries
       this.chart = new Chart(this.$refs.canvas, this.chartConfig)

       // Set up DOM event listeners
       window.addEventListener('resize', this.handleResize)
     }
     ```

2. **Q: How do I handle cleanup properly in Vue components?**

   A: Use the `beforeUnmount` lifecycle hook to clean up resources:

   ```js
   beforeUnmount() {
     // Remove event listeners
     window.removeEventListener('resize', this.handleResize)

     // Destroy third-party instances
     if (this.chart) {
       this.chart.destroy()
     }

     // Clear intervals/timeouts
     clearInterval(this.pollingInterval)
     clearTimeout(this.debounceTimeout)

     // Cancel pending API requests
     if (this.currentRequest) {
       this.currentRequest.cancel()
     }
   }
   ```

   Not cleaning up properly can lead to memory leaks, especially in single-page applications where components may be created and destroyed frequently.

3. **Q: How do lifecycle hooks work with async operations?**

   A: Lifecycle hooks themselves are not async, but you can use async/await inside them:

   ```js
   async created() {
     try {
       this.loading = true
       const data = await this.fetchData()
       this.items = data
     } catch (error) {
       this.error = error.message
     } finally {
       this.loading = false
     }
   }
   ```

   For operations that need to happen in sequence, you can use async/await. Just be aware that the component will continue its lifecycle regardless of pending promises.

   For operations that should block rendering, consider using the `v-if` directive with a loading state.

### 5. Mixins

**Concise Explanation:**
Mixins are a flexible way to distribute reusable functionality across Vue components. They allow you to extract common component options into separate objects that can be "mixed in" to multiple components. While they've largely been replaced by composables in Vue 3, mixins remain important in many codebases and in the Options API.

**Where to Use:**

- Sharing common methods across multiple components
- Reusing lifecycle hooks in a consistent way
- Adding standard computed properties to several components
- Creating pluggable functionality that components can opt into
- Centralizing common state or behavior patterns

**Code Snippet:**

```js
// mixins/formMixin.js
export const formMixin = {
  data() {
    return {
      isSubmitting: false,
      errors: {},
      successMessage: "",
      touched: {},
    };
  },

  methods: {
    /**
     * Mark a field as touched
     * @param {string} field - Field name
     */
    touch(field) {
      this.touched[field] = true;
    },

    /**
     * Validate form data
     * @returns {boolean} - True if valid
     */
    validate() {
      // Reset errors
      this.errors = {};

      // This method should be implemented in the component
      if (typeof this.validateFields !== "function") {
        console.warn("validateFields method not implemented");
        return true;
      }

      // Call component's validation logic
      const errors = this.validateFields();

      // If no errors, return true
      if (!errors || Object.keys(errors).length === 0) {
        return true;
      }

      // Otherwise, set errors and return false
      this.errors = errors;

      // Mark all fields as touched
      Object.keys(errors).forEach((field) => {
        this.touch(field);
      });

      return false;
    },

    /**
     * Submit the form
     * @param {Event} event - Form submit event
     */
    async submitForm(event) {
      if (event) {
        event.preventDefault();
      }

      // Validate form
      if (!this.validate()) {
        return;
      }

      // Set submitting state
      this.isSubmitting = true;
      this.successMessage = "";

      try {
        // This method should be implemented in the component
        if (typeof this.submit !== "function") {
          throw new Error("submit method not implemented");
        }

        // Call component's submit logic
        const result = await this.submit();

        // Set success message
        this.successMessage = result?.message || "Form submitted successfully";

        // Reset form if needed
        if (this.resetAfterSubmit) {
          this.resetForm();
        }

        // Emit success event
        this.$emit("success", result);

        return result;
      } catch (error) {
        // Handle validation errors from API
        if (error.response?.data?.errors) {
          this.errors = error.response.data.errors;
        } else {
          // Set generic error
          this.errors._generic = error.message || "An error occurred";
        }

        // Emit error event
        this.$emit("error", error);

        // Re-throw for component handling if needed
        throw error;
      } finally {
        this.isSubmitting = false;
      }
    },

    /**
     * Reset the form
     */
    resetForm() {
      // Reset errors and success message
      this.errors = {};
      this.successMessage = "";
      this.touched = {};

      // This method can be implemented in the component
      if (typeof this.resetFields === "function") {
        this.resetFields();
      }
    },

    /**
     * Check if a field has an error
     * @param {string} field - Field name
     * @returns {boolean} - True if error exists
     */
    hasError(field) {
      return Object.prototype.hasOwnProperty.call(this.errors, field);
    },

    /**
     * Get error message for a field
     * @param {string} field - Field name
     * @returns {string} - Error message
     */
    getError(field) {
      return this.errors[field] || "";
    },

    /**
     * Check if a field is touched
     * @param {string} field - Field name
     * @returns {boolean} - True if touched
     */
    isTouched(field) {
      return Object.prototype.hasOwnProperty.call(this.touched, field);
    },

    /**
     * Check if a field is invalid
     * @param {string} field - Field name
     * @returns {boolean} - True if invalid
     */
    isInvalid(field) {
      return this.isTouched(field) && this.hasError(field);
    },
  },
};
```

```html
<!-- ContactForm.vue (Using the mixin) -->
<template>
  <div class="p-6 bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800 mb-4">Contact Us</h1>

    <form @submit="submitForm" class="space-y-4">
      <!-- Name field -->
      <div>
        <label for="name" class="block text-sm font-medium text-gray-700"
          >Name</label
        >
        <input
          id="name"
          v-model="form.name"
          type="text"
          @blur="touch('name')"
          class="mt-1 block w-full px-3 py-2 border rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
          :class="{ 'border-red-500': isInvalid('name') }"
        />
        <p v-if="isInvalid('name')" class="mt-1 text-sm text-red-600">
          {{ getError("name") }}
        </p>
      </div>

      <!-- Email field -->
      <div>
        <label for="email" class="block text-sm font-medium text-gray-700"
          >Email</label
        >
        <input
          id="email"
          v-model="form.email"
          type="email"
          @blur="touch('email')"
          class="mt-1 block w-full px-3 py-2 border rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
          :class="{ 'border-red-500': isInvalid('email') }"
        />
        <p v-if="isInvalid('email')" class="mt-1 text-sm text-red-600">
          {{ getError("email") }}
        </p>
      </div>

      <!-- Subject field -->
      <div>
        <label for="subject" class="block text-sm font-medium text-gray-700"
          >Subject</label
        >
        <select
          id="subject"
          v-model="form.subject"
          @blur="touch('subject')"
          class="mt-1 block w-full px-3 py-2 border rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
          :class="{ 'border-red-500': isInvalid('subject') }"
        >
          <option value="">Select a subject</option>
          <option value="general">General Inquiry</option>
          <option value="support">Technical Support</option>
          <option value="billing">Billing Question</option>
          <option value="feedback">Feedback</option>
        </select>
        <p v-if="isInvalid('subject')" class="mt-1 text-sm text-red-600">
          {{ getError("subject") }}
        </p>
      </div>

      <!-- Message field -->
      <div>
        <label for="message" class="block text-sm font-medium text-gray-700"
          >Message</label
        >
        <textarea
          id="message"
          v-model="form.message"
          rows="4"
          @blur="touch('message')"
          class="mt-1 block w-full px-3 py-2 border rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
          :class="{ 'border-red-500': isInvalid('message') }"
        ></textarea>
        <p v-if="isInvalid('message')" class="mt-1 text-sm text-red-600">
          {{ getError("message") }}
        </p>
      </div>

      <!-- Generic error -->
      <div
        v-if="errors._generic"
        class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded"
      >
        {{ errors._generic }}
      </div>

      <!-- Success message -->
      <div
        v-if="successMessage"
        class="bg-green-100 border border-green-400 text-green-700 px-4 py-3 rounded"
      >
        {{ successMessage }}
      </div>

      <!-- Submit button -->
      <div class="flex justify-end">
        <button
          type="submit"
          :disabled="isSubmitting"
          class="px-4 py-2 bg-blue-600 text-white font-medium rounded hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 disabled:opacity-50"
        >
          <span v-if="isSubmitting">
            <svg
              class="animate-spin -ml-1 mr-2 h-4 w-4 text-white inline-block"
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
            Sending...
          </span>
          <span v-else>Send Message</span>
        </button>
      </div>
    </form>
  </div>
</template>

<script>
  import { formMixin } from "@/mixins/formMixin";

  export default {
    name: "ContactForm",

    // Use the form mixin
    mixins: [formMixin],

    data() {
      return {
        form: {
          name: "",
          email: "",
          subject: "",
          message: "",
        },
        resetAfterSubmit: true,
      };
    },

    methods: {
      // Implement the validateFields method required by the mixin
      validateFields() {
        const errors = {};

        if (!this.form.name.trim()) {
          errors.name = "Name is required";
        }

        if (!this.form.email.trim()) {
          errors.email = "Email is required";
        } else if (!this.isValidEmail(this.form.email)) {
          errors.email = "Please enter a valid email address";
        }

        if (!this.form.subject) {
          errors.subject = "Please select a subject";
        }

        if (!this.form.message.trim()) {
          errors.message = "Message is required";
        } else if (this.form.message.length < 10) {
          errors.message = "Message must be at least 10 characters";
        }

        return errors;
      },

      // Implement the submit method required by the mixin
      async submit() {
        // Simulate API call
        await new Promise((resolve) => setTimeout(resolve, 1500));

        // In a real app, this would be a fetch or axios call
        console.log("Submitting form:", this.form);

        // Return successful response
        return {
          success: true,
          message: "Your message has been sent. We'll get back to you soon!",
        };
      },

      // Implement the resetFields method
      resetFields() {
        this.form = {
          name: "",
          email: "",
          subject: "",
          message: "",
        };
      },

      // Helper method for email validation
      isValidEmail(email) {
        // Basic email validation
        return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
      },
    },
  };
</script>
```

**Authorization Mixin Example:**

```js
// mixins/authMixin.js
export const authMixin = {
  data() {
    return {
      isLoadingAuth: false,
      authError: null,
    };
  },

  computed: {
    // Check if user is authenticated
    isAuthenticated() {
      return !!this.$store.state.auth.user;
    },

    // Get current user
    currentUser() {
      return this.$store.state.auth.user || {};
    },

    // Get user's roles
    userRoles() {
      return this.currentUser.roles || [];
    },

    // Check if user is admin
    isAdmin() {
      return this.userRoles.includes("admin");
    },
  },

  methods: {
    /**
     * Check if user has a specific role
     * @param {string|string[]} roles - Role(s) to check
     * @returns {boolean} - True if user has the role
     */
    hasRole(roles) {
      // If not authenticated, return false
      if (!this.isAuthenticated) return false;

      // Convert single role to array
      const rolesToCheck = Array.isArray(roles) ? roles : [roles];

      // Check if user has any of the roles
      return rolesToCheck.some((role) => this.userRoles.includes(role));
    },

    /**
     * Check if user has a specific permission
     * @param {string|string[]} permissions - Permission(s) to check
     * @returns {boolean} - True if user has the permission
     */
    hasPermission(permissions) {
      // If not authenticated, return false
      if (!this.isAuthenticated) return false;

      // Get user permissions
      const userPermissions = this.currentUser.permissions || [];

      // Convert single permission to array
      const permissionsToCheck = Array.isArray(permissions)
        ? permissions
        : [permissions];

      // Check if user has any of the permissions
      return permissionsToCheck.some((permission) =>
        userPermissions.includes(permission)
      );
    },

    /**
     * Require authentication
     * @param {Function} callback - Callback to execute if authenticated
     * @returns {boolean} - True if authenticated
     */
    requireAuth(callback) {
      if (!this.isAuthenticated) {
        // Store the current location for redirect after login
        this.$store.dispatch("auth/setRedirectPath", this.$route.fullPath);

        // Redirect to login page
        this.$router.push({ name: "login" });
        return false;
      }

      // Execute callback if provided
      if (typeof callback === "function") {
        callback();
      }

      return true;
    },

    /**
     * Require specific role
     * @param {string|string[]} roles - Role(s) to require
     * @param {Function} callback - Callback to execute if has role
     * @returns {boolean} - True if has role
     */
    requireRole(roles, callback) {
      // First check if authenticated
      if (!this.requireAuth()) return false;

      // Then check if has role
      if (!this.hasRole(roles)) {
        // Redirect to unauthorized page
        this.$router.push({ name: "unauthorized" });
        return false;
      }

      // Execute callback if provided
      if (typeof callback === "function") {
        callback();
      }

      return true;
    },
  },
};
```

**Using Multiple Mixins:**

```html
<template>
  <div class="p-6 bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800 mb-4">User Profile</h1>

    <!-- Loading state -->
    <div
      v-if="isLoadingAuth || isLoadingProfile"
      class="py-12 flex justify-center"
    >
      <div
        class="animate-spin rounded-full h-12 w-12 border-4 border-blue-500 border-t-transparent"
      ></div>
    </div>

    <!-- Authentication required -->
    <div v-else-if="!isAuthenticated" class="py-6 text-center">
      <p class="text-gray-600 mb-4">You need to sign in to view your profile</p>
      <button
        @click="$router.push({ name: 'login' })"
        class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
      >
        Sign In
      </button>
    </div>

    <!-- Error state -->
    <div
      v-else-if="profileError || authError"
      class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded"
    >
      <p class="font-bold">Error</p>
      <p>{{ profileError || authError }}</p>
      <button
        @click="loadProfile"
        class="mt-2 px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700"
      >
        Try Again
      </button>
    </div>

    <!-- Profile content -->
    <div v-else-if="profile" class="space-y-6">
      <!-- User info -->
      <div class="flex items-center space-x-4">
        <img
          :src="profile.avatar || '/default-avatar.png'"
          :alt="profile.name"
          class="w-16 h-16 rounded-full object-cover"
        />
        <div>
          <h2 class="text-lg font-semibold">{{ profile.name }}</h2>
          <p class="text-gray-600">{{ profile.email }}</p>
          <div class="mt-1 flex space-x-2">
            <span
              v-for="role in userRoles"
              :key="role"
              class="px-2 py-1 text-xs rounded"
              :class="{
                'bg-blue-100 text-blue-800': role === 'user',
                'bg-purple-100 text-purple-800': role === 'editor',
                'bg-red-100 text-red-800': role === 'admin',
              }"
            >
              {{ role }}
            </span>
          </div>
        </div>
      </div>

      <!-- Profile form -->
      <form @submit="submitForm" class="space-y-4">
        <!-- Profile fields -->
        <div>
          <label
            for="displayName"
            class="block text-sm font-medium text-gray-700"
            >Display Name</label
          >
          <input
            id="displayName"
            v-model="form.displayName"
            type="text"
            @blur="touch('displayName')"
            class="mt-1 block w-full px-3 py-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
            :class="{ 'border-red-500': isInvalid('displayName') }"
          />
          <p v-if="isInvalid('displayName')" class="mt-1 text-sm text-red-600">
            {{ getError("displayName") }}
          </p>
        </div>

        <div>
          <label for="bio" class="block text-sm font-medium text-gray-700"
            >Bio</label
          >
          <textarea
            id="bio"
            v-model="form.bio"
            rows="3"
            @blur="touch('bio')"
            class="mt-1 block w-full px-3 py-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
            :class="{ 'border-red-500': isInvalid('bio') }"
          ></textarea>
          <p v-if="isInvalid('bio')" class="mt-1 text-sm text-red-600">
            {{ getError("bio") }}
          </p>
        </div>

        <!-- Admin-only settings section -->
        <div v-if="isAdmin" class="pt-4 border-t">
          <h3 class="text-lg font-medium mb-2">Admin Settings</h3>

          <div class="space-y-2">
            <label class="flex items-center">
              <input
                type="checkbox"
                v-model="form.isPublic"
                class="h-4 w-4 text-blue-600 rounded focus:ring-blue-500"
              />
              <span class="ml-2 text-sm text-gray-700"
                >Make profile public</span
              >
            </label>

            <label class="flex items-center">
              <input
                type="checkbox"
                v-model="form.emailNotifications"
                class="h-4 w-4 text-blue-600 rounded focus:ring-blue-500"
              />
              <span class="ml-2 text-sm text-gray-700"
                >Receive email notifications</span
              >
            </label>
          </div>
        </div>

        <!-- Generic error -->
        <div
          v-if="errors._generic"
          class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded"
        >
          {{ errors._generic }}
        </div>

        <!-- Success message -->
        <div
          v-if="successMessage"
          class="bg-green-100 border border-green-400 text-green-700 px-4 py-3 rounded"
        >
          {{ successMessage }}
        </div>

        <!-- Form actions -->
        <div class="flex justify-end space-x-3">
          <button
            type="button"
            @click="resetForm"
            class="px-4 py-2 border border-gray-300 rounded hover:bg-gray-50"
          >
            Cancel
          </button>

          <button
            type="submit"
            :disabled="isSubmitting"
            class="px-4 py-2 bg-blue-600 text-white font-medium rounded hover:bg-blue-700 disabled:opacity-50"
          >
            <span v-if="isSubmitting">
              <svg
                class="animate-spin -ml-1 mr-2 h-4 w-4 text-white inline-block"
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
              Saving...
            </span>
            <span v-else>Save Changes</span>
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
  import { formMixin } from "@/mixins/formMixin";
  import { authMixin } from "@/mixins/authMixin";
  import { profileMixin } from "@/mixins/profileMixin"; // Another mixin

  export default {
    name: "UserProfile",

    // Use multiple mixins
    mixins: [formMixin, authMixin, profileMixin],

    data() {
      return {
        form: {
          displayName: "",
          bio: "",
          isPublic: false,
          emailNotifications: true,
        },
      };
    },

    created() {
      // Load profile data when component is created
      if (this.isAuthenticated) {
        this.loadProfile();
      }
    },

    watch: {
      // Reload profile when user changes
      currentUser(newUser) {
        if (newUser) {
          this.loadProfile();
        }
      },
    },

    methods: {
      // Load profile data
      async loadProfile() {
        if (!this.requireAuth()) return;

        this.isLoadingProfile = true;
        this.profileError = null;

        try {
          // Simulate API call
          await new Promise((resolve) => setTimeout(resolve, 1000));

          // In a real app, this would fetch from an API
          this.profile = {
            name: this.currentUser.name,
            email: this.currentUser.email,
            avatar: this.currentUser.avatar,
            displayName: this.currentUser.name,
            bio: "Lorem ipsum dolor sit amet, consectetur adipiscing elit.",
            isPublic: false,
            emailNotifications: true,
          };

          // Initialize form with profile data
          this.form = {
            displayName: this.profile.displayName,
            bio: this.profile.bio,
            isPublic: this.profile.isPublic,
            emailNotifications: this.profile.emailNotifications,
          };
        } catch (error) {
          this.profileError = error.message || "Failed to load profile";
        } finally {
          this.isLoadingProfile = false;
        }
      },

      // Validate form fields (required by formMixin)
      validateFields() {
        const errors = {};

        if (!this.form.displayName.trim()) {
          errors.displayName = "Display name is required";
        }

        if (this.form.bio && this.form.bio.length > 200) {
          errors.bio = "Bio cannot exceed 200 characters";
        }

        return errors;
      },

      // Submit form (required by formMixin)
      async submit() {
        // Check authentication
        if (!this.requireAuth()) return;

        // Simulate API call
        await new Promise((resolve) => setTimeout(resolve, 1500));

        // Update profile data
        this.profile = {
          ...this.profile,
          displayName: this.form.displayName,
          bio: this.form.bio,
          isPublic: this.form.isPublic,
          emailNotifications: this.form.emailNotifications,
        };

        // Return success
        return {
          success: true,
          message: "Profile updated successfully",
        };
      },

      // Reset form fields (optional for formMixin)
      resetFields() {
        // Reset to original profile data
        if (this.profile) {
          this.form = {
            displayName: this.profile.displayName,
            bio: this.profile.bio,
            isPublic: this.profile.isPublic,
            emailNotifications: this.profile.emailNotifications,
          };
        } else {
          this.form = {
            displayName: "",
            bio: "",
            isPublic: false,
            emailNotifications: true,
          };
        }
      },
    },
  };
</script>
```

**Global Mixin Example:**

```js
// Global mixin (use with caution!)
// main.js
import Vue from "vue";
import App from "./App.vue";

// Global mixin
Vue.mixin({
  created() {
    // Log component creation for debugging
    const componentName = this.$options.name || "Anonymous";
    console.log(`Component created: ${componentName}`);
  },

  methods: {
    // Format date for all components
    formatDate(date, format = "short") {
      if (!date) return "";

      const dateObj = new Date(date);

      switch (format) {
        case "short":
          return dateObj.toLocaleDateString();
        case "long":
          return dateObj.toLocaleDateString(undefined, {
            year: "numeric",
            month: "long",
            day: "numeric",
          });
        case "relative":
          // Calculate relative time
          const now = new Date();
          const diff = now - dateObj;

          // Less than a minute
          if (diff < 60000) return "just now";

          // Less than an hour
          if (diff < 3600000) {
            const minutes = Math.floor(diff / 60000);
            return `${minutes} minute${minutes !== 1 ? "s" : ""} ago`;
          }

          // Less than a day
          if (diff < 86400000) {
            const hours = Math.floor(diff / 3600000);
            return `${hours} hour${hours !== 1 ? "s" : ""} ago`;
          }

          // Less than a week
          if (diff < 604800000) {
            const days = Math.floor(diff / 86400000);
            return `${days} day${days !== 1 ? "s" : ""} ago`;
          }

          // Default to short format
          return dateObj.toLocaleDateString();
        default:
          return dateObj.toLocaleDateString();
      }
    },
  },
});

new Vue({
  render: (h) => h(App),
}).$mount("#app");
```

**Real-World Example:**
In an e-commerce application, you might create mixins for:

- Product handling (formatting prices, calculating discounts)
- Cart management (adding/removing items, calculating totals)
- User authentication (checking permissions, handling login/logout)
- Form handling (validation, submission, error handling)
- Analytics tracking (sending events on component interaction)

**Common Pitfalls:**

- **Name Collisions**: When mixins have properties or methods with the same names
- **Implicit Dependencies**: Mixins that rely on specific component structure without making it clear
- **Overuse**: Creating mixins for small pieces of functionality better handled by utility functions
- **Debugging Difficulty**: Tracing the source of properties/methods in a component with many mixins
- **Order Matters**: Mixins are applied in order, which can lead to unexpected behavior

**Frequently Asked Questions:**

1. **Q: When should I use a mixin vs. a component?**

   A: Use the right tool for the right job:

   - **Use a mixin** when you want to share behavior without UI elements

     - Reusable methods across different components
     - Shared lifecycle hooks
     - Common data handling logic

   - **Use a component** when you want to reuse both behavior and UI
     - Reusable UI elements
     - Visual patterns that repeat across the application
     - Self-contained chunks of the interface

2. **Q: What are the strategies for resolving mixin conflicts?**

   A: When mixins and components define the same options, Vue uses these merge strategies:

   - **Data objects** are merged. Component data takes precedence when there are conflicts.

   - **Lifecycle hooks** from mixins and components are preserved and called in order:

     1. Mixin hooks (in the order they're included)
     2. Component hooks

   - **Methods, computed properties, and components** from the component take precedence over mixin versions.

   To avoid conflicts:

   - Use namespacing for mixin methods (e.g., `formValidate` instead of `validate`)
   - Keep mixins focused on specific functionality
   - Document dependencies and expected usage

3. **Q: Should I use mixins or composables in Vue 3?**

   A: In Vue 3 projects, composables are generally preferred over mixins because:

   - **Namespace Conflicts**: Composables have explicit imports and returns, avoiding naming conflicts
   - **Transparency**: The component explicitly shows what it's using from each composable
   - **Type Safety**: Composables work better with TypeScript
   - **Testability**: Composables are easier to test in isolation

   However, in Options API code, mixins remain a valid pattern. You should consider:

   - For new Options API code, use composables that expose reactive data and methods, then use those in the Options API
   - For existing code with mixins, you can continue to use them, or gradually migrate to composables

### 6. Migration to Composition API

**Concise Explanation:**
The Composition API is an alternative to the Options API that offers more flexible code organization, better TypeScript support, and more powerful reusability patterns. Migrating from Options API to Composition API involves restructuring your component logic from option-based organization to function-based composition.

**Where to Use:**

- When refactoring complex components with related logic spread across multiple options
- When creating highly reusable logic that's easier to share between components
- When you need better TypeScript support
- When working with large team codebases where explicit dependency tracking helps maintainability
- Gradually in existing Options API projects, starting with the most complex components

**Code Snippet:**

**Before (Options API):**

```html
<template>
  <div class="p-6 bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800 mb-4">Product Search</h1>

    <!-- Search input -->
    <div class="mb-6">
      <div class="flex">
        <input
          v-model="searchQuery"
          type="text"
          placeholder="Search products..."
          class="flex-1 px-4 py-2 border rounded-l focus:outline-none focus:ring-2 focus:ring-blue-500"
          @keyup.enter="searchProducts"
        />
        <button
          @click="searchProducts"
          class="px-4 py-2 bg-blue-600 text-white rounded-r hover:bg-blue-700"
        >
          Search
        </button>
      </div>

      <!-- Filters -->
      <div class="mt-3 flex flex-wrap gap-2">
        <select
          v-model="selectedCategory"
          class="px-3 py-1 border rounded text-sm"
        >
          <option value="">All Categories</option>
          <option
            v-for="category in categories"
            :key="category"
            :value="category"
          >
            {{ category }}
          </option>
        </select>

        <select v-model="sortBy" class="px-3 py-1 border rounded text-sm">
          <option value="relevance">Relevance</option>
          <option value="price_asc">Price: Low to High</option>
          <option value="price_desc">Price: High to Low</option>
          <option value="rating">Rating</option>
        </select>

        <button
          @click="toggleAdvancedFilters"
          class="px-3 py-1 border rounded text-sm hover:bg-gray-100"
        >
          {{ showAdvancedFilters ? "Hide Filters" : "Advanced Filters" }}
        </button>
      </div>

      <!-- Advanced filters -->
      <div
        v-if="showAdvancedFilters"
        class="mt-3 p-3 border rounded bg-gray-50"
      >
        <div class="grid grid-cols-1 md:grid-cols-3 gap-3">
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1"
              >Price Range</label
            >
            <div class="flex items-center space-x-2">
              <input
                v-model.number="minPrice"
                type="number"
                min="0"
                placeholder="Min"
                class="w-full px-3 py-1 border rounded text-sm"
              />
              <span>to</span>
              <input
                v-model.number="maxPrice"
                type="number"
                min="0"
                placeholder="Max"
                class="w-full px-3 py-1 border rounded text-sm"
              />
            </div>
          </div>

          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1"
              >Rating</label
            >
            <div class="flex items-center space-x-1">
              <span
                v-for="i in 5"
                :key="i"
                @click="minRating = i"
                class="cursor-pointer"
              >
                <svg
                  class="h-5 w-5"
                  :class="{
                    'text-yellow-400 fill-current': i <= minRating,
                    'text-gray-300': i > minRating,
                  }"
                  viewBox="0 0 20 20"
                >
                  <path
                    d="M9.049 2.927c.3-.921 1.603-.921 1.902 0l1.07 3.292a1 1 0 00.95.69h3.462c.969 0 1.371 1.24.588 1.81l-2.8 2.034a1 1 0 00-.364 1.118l1.07 3.292c.3.921-.755 1.688-1.54 1.118l-2.8-2.034a1 1 0 00-1.175 0l-2.8 2.034c-.784.57-1.838-.197-1.539-1.118l1.07-3.292a1 1 0 00-.364-1.118L2.98 8.72c-.783-.57-.38-1.81.588-1.81h3.461a1 1 0 00.951-.69l1.07-3.292z"
                  />
                </svg>
              </span>
              <span class="ml-1 text-sm text-gray-700">& Up</span>
            </div>
          </div>

          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1"
              >Availability</label
            >
            <label class="flex items-center">
              <input
                type="checkbox"
                v-model="inStockOnly"
                class="h-4 w-4 text-blue-600 rounded"
              />
              <span class="ml-2 text-sm text-gray-700">In Stock Only</span>
            </label>
          </div>
        </div>
      </div>
    </div>

    <!-- Results loading -->
    <div v-if="isLoading" class="py-12 flex justify-center">
      <div
        class="animate-spin rounded-full h-12 w-12 border-4 border-blue-500 border-t-transparent"
      ></div>
    </div>

    <!-- Results error -->
    <div
      v-else-if="error"
      class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded"
    >
      <p class="font-bold">Error</p>
      <p>{{ error }}</p>
      <button
        @click="searchProducts"
        class="mt-2 px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700"
      >
        Try Again
      </button>
    </div>

    <!-- No results -->
    <div
      v-else-if="products.length === 0"
      class="py-12 text-center text-gray-500"
    >
      <p class="text-lg">No products found</p>
      <p class="text-sm">Try adjusting your search criteria</p>
    </div>

    <!-- Results grid -->
    <div v-else class="grid grid-cols-1 md:grid-cols-3 gap-6">
      <div
        v-for="product in products"
        :key="product.id"
        class="border rounded-lg overflow-hidden hover:shadow-md transition-shadow"
      >
        <img
          :src="product.image"
          :alt="product.name"
          class="w-full h-48 object-cover object-center"
        />
        <div class="p-4">
          <h2 class="font-medium text-gray-900">{{ product.name }}</h2>
          <p class="text-sm text-gray-500">{{ product.category }}</p>

          <div class="mt-2 flex items-center">
            <div class="flex items-center">
              <svg
                v-for="i in 5"
                :key="i"
                class="h-4 w-4"
                :class="{
                  'text-yellow-400 fill-current': i <= product.rating,
                  'text-gray-300': i > product.rating,
                }"
                viewBox="0 0 20 20"
              >
                <path
                  d="M9.049 2.927c.3-.921 1.603-.921 1.902 0l1.07 3.292a1 1 0 00.95.69h3.462c.969 0 1.371 1.24.588 1.81l-2.8 2.034a1 1 0 00-.364 1.118l1.07 3.292c.3.921-.755 1.688-1.54 1.118l-2.8-2.034a1 1 0 00-1.175 0l-2.8 2.034c-.784.57-1.838-.197-1.539-1.118l1.07-3.292a1 1 0 00-.364-1.118L2.98 8.72c-.783-.57-.38-1.81.588-1.81h3.461a1 1 0 00.951-.69l1.07-3.292z"
                />
              </svg>
              <span class="ml-1 text-xs text-gray-500"
                >({{ product.reviewCount }})</span
              >
            </div>

            <div class="ml-auto">
              <span class="text-lg font-bold text-gray-900"
                >${{ product.price.toFixed(2) }}</span
              >
            </div>
          </div>

          <div class="mt-4">
            <button
              @click="addToCart(product)"
              class="w-full py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
              :disabled="!product.inStock"
            >
              {{ product.inStock ? "Add to Cart" : "Out of Stock" }}
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Pagination -->
    <div
      v-if="products.length && totalPages > 1"
      class="mt-6 flex justify-between items-center"
    >
      <button
        @click="previousPage"
        :disabled="currentPage === 1"
        class="px-4 py-2 border rounded text-sm disabled:opacity-50"
      >
        Previous
      </button>

      <div class="text-sm">Page {{ currentPage }} of {{ totalPages }}</div>

      <button
        @click="nextPage"
        :disabled="currentPage === totalPages"
        class="px-4 py-2 border rounded text-sm disabled:opacity-50"
      >
        Next
      </button>
    </div>
  </div>
</template>

<script>
  export default {
    name: "ProductSearch",

    data() {
      return {
        // Search and filters
        searchQuery: "",
        selectedCategory: "",
        sortBy: "relevance",
        showAdvancedFilters: false,
        minPrice: null,
        maxPrice: null,
        minRating: 0,
        inStockOnly: false,

        // Products data
        products: [],
        isLoading: false,
        error: null,

        // Pagination
        currentPage: 1,
        totalPages: 1,

        // Available categories (would come from an API in a real app)
        categories: [
          "Electronics",
          "Clothing",
          "Home & Kitchen",
          "Books",
          "Toys",
        ],
      };
    },

    watch: {
      // Reset to first page when filters change
      selectedCategory() {
        this.currentPage = 1;
        this.searchProducts();
      },

      sortBy() {
        this.searchProducts();
      },

      minPrice() {
        this.debouncedSearch();
      },

      maxPrice() {
        this.debouncedSearch();
      },

      minRating() {
        this.searchProducts();
      },

      inStockOnly() {
        this.searchProducts();
      },
    },

    created() {
      // Initialize debounce function
      this.debouncedSearch = this.debounce(this.searchProducts, 500);

      // Load initial products
      this.searchProducts();
    },

    methods: {
      async searchProducts() {
        this.isLoading = true;
        this.error = null;

        try {
          // In a real app, this would be an API call
          await new Promise((resolve) => setTimeout(resolve, 1000));

          // Generate mock products
          const mockProducts = this.generateMockProducts();

          // Filter products based on search criteria
          let filtered = [...mockProducts];

          // Apply text search
          if (this.searchQuery) {
            const query = this.searchQuery.toLowerCase();
            filtered = filtered.filter(
              (p) =>
                p.name.toLowerCase().includes(query) ||
                p.description.toLowerCase().includes(query)
            );
          }

          // Apply category filter
          if (this.selectedCategory) {
            filtered = filtered.filter(
              (p) => p.category === this.selectedCategory
            );
          }

          // Apply price filters
          if (this.minPrice !== null) {
            filtered = filtered.filter((p) => p.price >= this.minPrice);
          }
          if (this.maxPrice !== null) {
            filtered = filtered.filter((p) => p.price <= this.maxPrice);
          }

          // Apply rating filter
          if (this.minRating > 0) {
            filtered = filtered.filter((p) => p.rating >= this.minRating);
          }

          // Apply availability filter
          if (this.inStockOnly) {
            filtered = filtered.filter((p) => p.inStock);
          }

          // Apply sorting
          switch (this.sortBy) {
            case "price_asc":
              filtered.sort((a, b) => a.price - b.price);
              break;
            case "price_desc":
              filtered.sort((a, b) => b.price - a.price);
              break;
            case "rating":
              filtered.sort((a, b) => b.rating - a.rating);
              break;
            // Default is relevance (no additional sorting)
          }

          // Calculate pagination
          const pageSize = 6;
          this.totalPages = Math.ceil(filtered.length / pageSize);

          // Get current page items
          const startIndex = (this.currentPage - 1) * pageSize;
          this.products = filtered.slice(startIndex, startIndex + pageSize);
        } catch (err) {
          this.error = "Failed to load products. Please try again.";
          console.error("Search error:", err);
        } finally {
          this.isLoading = false;
        }
      },

      toggleAdvancedFilters() {
        this.showAdvancedFilters = !this.showAdvancedFilters;
      },

      previousPage() {
        if (this.currentPage > 1) {
          this.currentPage--;
          this.searchProducts();
        }
      },

      nextPage() {
        if (this.currentPage < this.totalPages) {
          this.currentPage++;
          this.searchProducts();
        }
      },

      addToCart(product) {
        // In a real app, this would dispatch to a store
        console.log("Adding to cart:", product);

        // Show a toast notification
        this.$emit("notification", {
          type: "success",
          message: `${product.name} added to cart`,
        });
      },

      // Utility method for debouncing
      debounce(fn, delay) {
        let timeout;
        return function () {
          const context = this;
          const args = arguments;
          clearTimeout(timeout);
          timeout = setTimeout(() => fn.apply(context, args), delay);
        };
      },

      // Generate mock product data
      generateMockProducts() {
        const products = [];

        // Categories with corresponding image URLs
        const categoryImages = {
          Electronics:
            "https://source.unsplash.com/featured/300x200?electronics",
          Clothing: "https://source.unsplash.com/featured/300x200?clothing",
          "Home & Kitchen": "https://source.unsplash.com/featured/300x200?home",
          Books: "https://source.unsplash.com/featured/300x200?books",
          Toys: "https://source.unsplash.com/featured/300x200?toys",
        };

        // Product names by category
        const productsByCategory = {
          Electronics: [
            "Wireless Headphones",
            "Smartphone",
            "Laptop",
            "Tablet",
            "Smart Watch",
            "Bluetooth Speaker",
          ],
          Clothing: [
            "T-Shirt",
            "Jeans",
            "Sneakers",
            "Hoodie",
            "Dress",
            "Jacket",
          ],
          "Home & Kitchen": [
            "Coffee Maker",
            "Blender",
            "Toaster",
            "Cooking Pot",
            "Knife Set",
            "Dining Table",
          ],
          Books: [
            "Mystery Novel",
            "Sci-Fi Collection",
            "Cookbook",
            "Biography",
            "Self-Help Guide",
            "History Book",
          ],
          Toys: [
            "Action Figure",
            "Board Game",
            "Puzzle",
            "Remote Control Car",
            "Stuffed Animal",
            "Building Blocks",
          ],
        };

        // Generate 30 mock products
        for (let i = 1; i <= 30; i++) {
          // Random category
          const category =
            this.categories[Math.floor(Math.random() * this.categories.length)];

          // Random product name from the category
          const productsInCategory = productsByCategory[category];
          const name =
            productsInCategory[
              Math.floor(Math.random() * productsInCategory.length)
            ];

          // Random price between $10 and $1000
          const price = Math.round((10 + Math.random() * 990) * 100) / 100;

          // Random rating between 1 and 5, rounded to nearest 0.5
          const rating = Math.round(Math.random() * 4 * 2) / 2 + 1;

          // Random review count between 0 and 500
          const reviewCount = Math.floor(Math.random() * 500);

          // 80% chance of being in stock
          const inStock = Math.random() < 0.8;

          products.push({
            id: i,
            name: `${name} ${i}`,
            category,
            price,
            description: `This is a great ${name.toLowerCase()} with many features.`,
            image: categoryImages[category],
            rating,
            reviewCount,
            inStock,
          });
        }

        return products;
      },
    },
  };
</script>
```

**After (Composition API):**

```html
<template>
  <div class="p-6 bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800 mb-4">Product Search</h1>

    <!-- Search input -->
    <div class="mb-6">
      <div class="flex">
        <input
          v-model="searchQuery"
          type="text"
          placeholder="Search products..."
          class="flex-1 px-4 py-2 border rounded-l focus:outline-none focus:ring-2 focus:ring-blue-500"
          @keyup.enter="searchProducts"
        />
        <button
          @click="searchProducts"
          class="px-4 py-2 bg-blue-600 text-white rounded-r hover:bg-blue-700"
        >
          Search
        </button>
      </div>

      <!-- Filters -->
      <div class="mt-3 flex flex-wrap gap-2">
        <select
          v-model="selectedCategory"
          class="px-3 py-1 border rounded text-sm"
        >
          <option value="">All Categories</option>
          <option
            v-for="category in categories"
            :key="category"
            :value="category"
          >
            {{ category }}
          </option>
        </select>

        <select v-model="sortBy" class="px-3 py-1 border rounded text-sm">
          <option value="relevance">Relevance</option>
          <option value="price_asc">Price: Low to High</option>
          <option value="price_desc">Price: High to Low</option>
          <option value="rating">Rating</option>
        </select>

        <button
          @click="toggleAdvancedFilters"
          class="px-3 py-1 border rounded text-sm hover:bg-gray-100"
        >
          {{ showAdvancedFilters ? "Hide Filters" : "Advanced Filters" }}
        </button>
      </div>

      <!-- Advanced filters -->
      <div
        v-if="showAdvancedFilters"
        class="mt-3 p-3 border rounded bg-gray-50"
      >
        <div class="grid grid-cols-1 md:grid-cols-3 gap-3">
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1"
              >Price Range</label
            >
            <div class="flex items-center space-x-2">
              <input
                v-model.number="minPrice"
                type="number"
                min="0"
                placeholder="Min"
                class="w-full px-3 py-1 border rounded text-sm"
              />
              <span>to</span>
              <input
                v-model.number="maxPrice"
                type="number"
                min="0"
                placeholder="Max"
                class="w-full px-3 py-1 border rounded text-sm"
              />
            </div>
          </div>

          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1"
              >Rating</label
            >
            <div class="flex items-center space-x-1">
              <span
                v-for="i in 5"
                :key="i"
                @click="minRating = i"
                class="cursor-pointer"
              >
                <svg
                  class="h-5 w-5"
                  :class="{
                    'text-yellow-400 fill-current': i <= minRating,
                    'text-gray-300': i > minRating,
                  }"
                  viewBox="0 0 20 20"
                >
                  <path
                    d="M9.049 2.927c.3-.921 1.603-.921 1.902 0l1.07 3.292a1 1 0 00.95.69h3.462c.969 0 1.371 1.24.588 1.81l-2.8 2.034a1 1 0 00-.364 1.118l1.07 3.292c.3.921-.755 1.688-1.54 1.118l-2.8-2.034a1 1 0 00-1.175 0l-2.8 2.034c-.784.57-1.838-.197-1.539-1.118l1.07-3.292a1 1 0 00-.364-1.118L2.98 8.72c-.783-.57-.38-1.81.588-1.81h3.461a1 1 0 00.951-.69l1.07-3.292z"
                  />
                </svg>
              </span>
              <span class="ml-1 text-sm text-gray-700">& Up</span>
            </div>
          </div>

          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1"
              >Availability</label
            >
            <label class="flex items-center">
              <input
                type="checkbox"
                v-model="inStockOnly"
                class="h-4 w-4 text-blue-600 rounded"
              />
              <span class="ml-2 text-sm text-gray-700">In Stock Only</span>
            </label>
          </div>
        </div>
      </div>
    </div>

    <!-- Results loading -->
    <div v-if="isLoading" class="py-12 flex justify-center">
      <div
        class="animate-spin rounded-full h-12 w-12 border-4 border-blue-500 border-t-transparent"
      ></div>
    </div>

    <!-- Results error -->
    <div
      v-else-if="error"
      class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded"
    >
      <p class="font-bold">Error</p>
      <p>{{ error }}</p>
      <button
        @click="searchProducts"
        class="mt-2 px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700"
      >
        Try Again
      </button>
    </div>

    <!-- No results -->
    <div
      v-else-if="products.length === 0"
      class="py-12 text-center text-gray-500"
    >
      <p class="text-lg">No products found</p>
      <p class="text-sm">Try adjusting your search criteria</p>
    </div>

    <!-- Results grid -->
    <div v-else class="grid grid-cols-1 md:grid-cols-3 gap-6">
      <div
        v-for="product in products"
        :key="product.id"
        class="border rounded-lg overflow-hidden hover:shadow-md transition-shadow"
      >
        <img
          :src="product.image"
          :alt="product.name"
          class="w-full h-48 object-cover object-center"
        />
        <div class="p-4">
          <h2 class="font-medium text-gray-900">{{ product.name }}</h2>
          <p class="text-sm text-gray-500">{{ product.category }}</p>

          <div class="mt-2 flex items-center">
            <div class="flex items-center">
              <svg
                v-for="i in 5"
                :key="i"
                class="h-4 w-4"
                :class="{
                  'text-yellow-400 fill-current': i <= product.rating,
                  'text-gray-300': i > product.rating,
                }"
                viewBox="0 0 20 20"
              >
                <path
                  d="M9.049 2.927c.3-.921 1.603-.921 1.902 0l1.07 3.292a1 1 0 00.95.69h3.462c.969 0 1.371 1.24.588 1.81l-2.8 2.034a1 1 0 00-.364 1.118l1.07 3.292c.3.921-.755 1.688-1.54 1.118l-2.8-2.034a1 1 0 00-1.175 0l-2.8 2.034c-.784.57-1.838-.197-1.539-1.118l1.07-3.292a1 1 0 00-.364-1.118L2.98 8.72c-.783-.57-.38-1.81.588-1.81h3.461a1 1 0 00.951-.69l1.07-3.292z"
                />
              </svg>
              <span class="ml-1 text-xs text-gray-500"
                >({{ product.reviewCount }})</span
              >
            </div>

            <div class="ml-auto">
              <span class="text-lg font-bold text-gray-900"
                >${{ product.price.toFixed(2) }}</span
              >
            </div>
          </div>

          <div class="mt-4">
            <button
              @click="addToCart(product)"
              class="w-full py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
              :disabled="!product.inStock"
            >
              {{ product.inStock ? "Add to Cart" : "Out of Stock" }}
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Pagination -->
    <div
      v-if="products.length && totalPages > 1"
      class="mt-6 flex justify-between items-center"
    >
      <button
        @click="previousPage"
        :disabled="currentPage === 1"
        class="px-4 py-2 border rounded text-sm disabled:opacity-50"
      >
        Previous
      </button>

      <div class="text-sm">Page {{ currentPage }} of {{ totalPages }}</div>

      <button
        @click="nextPage"
        :disabled="currentPage === totalPages"
        class="px-4 py-2 border rounded text-sm disabled:opacity-50"
      >
        Next
      </button>
    </div>
  </div>
</template>

<script setup>
  import { ref, reactive, computed, watch, onMounted, onUnmounted } from "vue";
  import { useProductSearch } from "@/composables/useProductSearch";
  import { useDebounce } from "@/composables/useDebounce";

  // Emits
  const emit = defineEmits(["notification"]);

  // Extract product search functionality into a composable
  const {
    products,
    isLoading,
    error,
    categories,
    searchProducts,
    generateMockProducts,
  } = useProductSearch();

  // Search and filters state
  const searchQuery = ref("");
  const selectedCategory = ref("");
  const sortBy = ref("relevance");
  const showAdvancedFilters = ref(false);
  const minPrice = ref(null);
  const maxPrice = ref(null);
  const minRating = ref(0);
  const inStockOnly = ref(false);

  // Pagination state
  const currentPage = ref(1);
  const totalPages = ref(1);

  // Create debounced search function
  const { debounce } = useDebounce();
  const debouncedSearch = debounce(() => {
    searchProducts({
      query: searchQuery.value,
      category: selectedCategory.value,
      sortBy: sortBy.value,
      minPrice: minPrice.value,
      maxPrice: maxPrice.value,
      minRating: minRating.value,
      inStockOnly: inStockOnly.value,
      page: currentPage.value,
    });
  }, 500);

  // Watch for filter changes
  watch(selectedCategory, () => {
    currentPage.value = 1;
    searchProducts({
      query: searchQuery.value,
      category: selectedCategory.value,
      sortBy: sortBy.value,
      minPrice: minPrice.value,
      maxPrice: maxPrice.value,
      minRating: minRating.value,
      inStockOnly: inStockOnly.value,
      page: currentPage.value,
    });
  });

  watch(sortBy, () => {
    searchProducts({
      query: searchQuery.value,
      category: selectedCategory.value,
      sortBy: sortBy.value,
      minPrice: minPrice.value,
      maxPrice: maxPrice.value,
      minRating: minRating.value,
      inStockOnly: inStockOnly.value,
      page: currentPage.value,
    });
  });

  watch([minPrice, maxPrice], () => {
    debouncedSearch();
  });

  watch([minRating, inStockOnly], () => {
    searchProducts({
      query: searchQuery.value,
      category: selectedCategory.value,
      sortBy: sortBy.value,
      minPrice: minPrice.value,
      maxPrice: maxPrice.value,
      minRating: minRating.value,
      inStockOnly: inStockOnly.value,
      page: currentPage.value,
    });
  });

  // UI action methods
  const toggleAdvancedFilters = () => {
    showAdvancedFilters.value = !showAdvancedFilters.value;
  };

  const previousPage = () => {
    if (currentPage.value > 1) {
      currentPage.value--;
      searchProducts({
        query: searchQuery.value,
        category: selectedCategory.value,
        sortBy: sortBy.value,
        minPrice: minPrice.value,
        maxPrice: maxPrice.value,
        minRating: minRating.value,
        inStockOnly: inStockOnly.value,
        page: currentPage.value,
      });
    }
  };

  const nextPage = () => {
    if (currentPage.value < totalPages.value) {
      currentPage.value++;
      searchProducts({
        query: searchQuery.value,
        category: selectedCategory.value,
        sortBy: sortBy.value,
        minPrice: minPrice.value,
        maxPrice: maxPrice.value,
        minRating: minRating.value,
        inStockOnly: inStockOnly.value,
        page: currentPage.value,
      });
    }
  };

  const addToCart = (product) => {
    // In a real app, this would use a composable or store action
    console.log("Adding to cart:", product);

    // Show a toast notification
    emit("notification", {
      type: "success",
      message: `${product.name} added to cart`,
    });
  };

  // Load initial products on mount
  onMounted(() => {
    searchProducts({
      query: searchQuery.value,
      category: selectedCategory.value,
      sortBy: sortBy.value,
      minPrice: minPrice.value,
      maxPrice: maxPrice.value,
      minRating: minRating.value,
      inStockOnly: inStockOnly.value,
      page: currentPage.value,
    });
  });
</script>
```

**Product Search Composable (for Composition API version):**

```js
// composables/useProductSearch.js
import { ref, computed } from "vue";

export function useProductSearch() {
  // State
  const products = ref([]);
  const isLoading = ref(false);
  const error = ref(null);
  const totalPages = ref(1);
  const categories = [
    "Electronics",
    "Clothing",
    "Home & Kitchen",
    "Books",
    "Toys",
  ];

  // Search products
  const searchProducts = async ({
    query = "",
    category = "",
    sortBy = "relevance",
    minPrice = null,
    maxPrice = null,
    minRating = 0,
    inStockOnly = false,
    page = 1,
  } = {}) => {
    isLoading.value = true;
    error.value = null;

    try {
      // In a real app, this would be an API call
      await new Promise((resolve) => setTimeout(resolve, 1000));

      // Generate mock products
      const mockProducts = generateMockProducts();

      // Filter products based on search criteria
      let filtered = [...mockProducts];

      // Apply text search
      if (query) {
        const queryLower = query.toLowerCase();
        filtered = filtered.filter(
          (p) =>
            p.name.toLowerCase().includes(queryLower) ||
            p.description.toLowerCase().includes(queryLower)
        );
      }

      // Apply category filter
      if (category) {
        filtered = filtered.filter((p) => p.category === category);
      }

      // Apply price filters
      if (minPrice !== null) {
        filtered = filtered.filter((p) => p.price >= minPrice);
      }
      if (maxPrice !== null) {
        filtered = filtered.filter((p) => p.price <= maxPrice);
      }

      // Apply rating filter
      if (minRating > 0) {
        filtered = filtered.filter((p) => p.rating >= minRating);
      }

      // Apply availability filter
      if (inStockOnly) {
        filtered = filtered.filter((p) => p.inStock);
      }

      // Apply sorting
      switch (sortBy) {
        case "price_asc":
          filtered.sort((a, b) => a.price - b.price);
          break;
        case "price_desc":
          filtered.sort((a, b) => b.price - a.price);
          break;
        case "rating":
          filtered.sort((a, b) => b.rating - a.rating);
          break;
        // Default is relevance (no additional sorting)
      }

      // Calculate pagination
      const pageSize = 6;
      totalPages.value = Math.ceil(filtered.length / pageSize);

      // Get current page items
      const startIndex = (page - 1) * pageSize;
      products.value = filtered.slice(startIndex, startIndex + pageSize);
    } catch (err) {
      error.value = "Failed to load products. Please try again.";
      console.error("Search error:", err);
    } finally {
      isLoading.value = false;
    }
  };

  // Generate mock product data
  const generateMockProducts = () => {
    const products = [];

    // Categories with corresponding image URLs
    const categoryImages = {
      Electronics: "https://source.unsplash.com/featured/300x200?electronics",
      Clothing: "https://source.unsplash.com/featured/300x200?clothing",
      "Home & Kitchen": "https://source.unsplash.com/featured/300x200?home",
      Books: "https://source.unsplash.com/featured/300x200?books",
      Toys: "https://source.unsplash.com/featured/300x200?toys",
    };

    // Product names by category
    const productsByCategory = {
      Electronics: [
        "Wireless Headphones",
        "Smartphone",
        "Laptop",
        "Tablet",
        "Smart Watch",
        "Bluetooth Speaker",
      ],
      Clothing: ["T-Shirt", "Jeans", "Sneakers", "Hoodie", "Dress", "Jacket"],
      "Home & Kitchen": [
        "Coffee Maker",
        "Blender",
        "Toaster",
        "Cooking Pot",
        "Knife Set",
        "Dining Table",
      ],
      Books: [
        "Mystery Novel",
        "Sci-Fi Collection",
        "Cookbook",
        "Biography",
        "Self-Help Guide",
        "History Book",
      ],
      Toys: [
        "Action Figure",
        "Board Game",
        "Puzzle",
        "Remote Control Car",
        "Stuffed Animal",
        "Building Blocks",
      ],
    };

    // Generate 30 mock products
    for (let i = 1; i <= 30; i++) {
      // Random category
      const category =
        categories[Math.floor(Math.random() * categories.length)];

      // Random product name from the category
      const productsInCategory = productsByCategory[category];
      const name =
        productsInCategory[
          Math.floor(Math.random() * productsInCategory.length)
        ];

      // Random price between $10 and $1000
      const price = Math.round((10 + Math.random() * 990) * 100) / 100;

      // Random rating between 1 and 5, rounded to nearest 0.5
      const rating = Math.round(Math.random() * 4 * 2) / 2 + 1;

      // Random review count between 0 and 500
      const reviewCount = Math.floor(Math.random() * 500);

      // 80% chance of being in stock
      const inStock = Math.random() < 0.8;

      products.push({
        id: i,
        name: `${name} ${i}`,
        category,
        price,
        description: `This is a great ${name.toLowerCase()} with many features.`,
        image: categoryImages[category],
        rating,
        reviewCount,
        inStock,
      });
    }

    return products;
  };

  return {
    products,
    isLoading,
    error,
    totalPages,
    categories,
    searchProducts,
    generateMockProducts,
  };
}
```

**Debounce Composable (for Composition API version):**

```js
// composables/useDebounce.js
import { ref } from "vue";

export function useDebounce() {
  // Create a debounce function
  const debounce = (fn, delay) => {
    let timeout = null;

    return (...args) => {
      clearTimeout(timeout);
      timeout = setTimeout(() => fn(...args), delay);
    };
  };

  // Debounced value version
  const debounceValue = (value, delay) => {
    const debouncedValue = ref(value.value);
    let timeout = null;

    const update = () => {
      clearTimeout(timeout);
      timeout = setTimeout(() => {
        debouncedValue.value = value.value;
      }, delay);
    };

    watch(value, () => {
      update();
    });

    return debouncedValue;
  };

  return {
    debounce,
    debounceValue,
  };
}
```

**Key Differences in Migration:**

1. **Script Setup vs Options Object:**

   - Options API groups code by type (data, methods, computed)
   - Composition API groups code by feature (search, pagination, cart)

2. **State Management:**

   - Options API uses `data()` to return an object
   - Composition API uses `ref()` and `reactive()` for individual reactive values

3. **Methods vs Functions:**

   - Options API defines methods in the methods section
   - Composition API uses regular functions declared in the setup function

4. **Lifecycle Hooks:**

   - Options API: `created()`, `mounted()`, etc.
   - Composition API: `onMounted()`, `onUnmounted()`, etc.

5. **Computed Properties:**

   - Options API defines them in the computed section
   - Composition API uses the `computed()` function

6. **Watchers:**

   - Options API uses the watch section
   - Composition API uses the `watch()` function

7. **Event Handling:**

   - Options API: `this.$emit('event')`
   - Composition API: `emit('event')` (from `defineEmits()`)

8. **Props and Emits:**

   - Options API: `props: { }` and events emitted with `this.$emit`
   - Composition API: `defineProps()` and `defineEmits()`

9. **Template Refs:**

   - Options API: `this.$refs.elementName`
   - Composition API: `const elementName = ref(null)` then use `elementName.value`

10. **Code Organization:**
    - Options API tightly couples all component logic together
    - Composition API lets you extract reusable logic into composables

**Migration Strategies:**

1. **Incremental Migration:**

   ```js
   // Component with mixed APIs
   export default {
     props: ["initialCount"],

     // Options API part
     data() {
       return {
         otherData: "example",
       };
     },

     // Composition API part
     setup(props) {
       const count = ref(props.initialCount);
       const increment = () => {
         count.value++;
       };

       return {
         count,
         increment,
       };
     },

     // Options API methods still work with data from setup()
     methods: {
       reset() {
         this.count = this.initialCount;
       },
     },
   };
   ```

2. **Feature-by-Feature:**

   - Start by extracting complex features into composables
   - Keep simple components in Options API
   - Gradually migrate as needed

3. **New Components, New API:**
   - Leave existing Options API components alone
   - Write all new components with Composition API
   - Refactor old components when making significant changes

**Real-World Example:**
In a dashboard application:

- The Options API version might have all user profile, notification, and settings logic mixed together
- The Composition API version would split these into separate composables:
  - `useUserProfile()` - Profile data and actions
  - `useNotifications()` - Notification system
  - `useSettings()` - User preferences

**Common Pitfalls:**

- **Forgetting `.value`**: In Composition API, forgetting to use `.value` when reading or updating refs
- **Missing return**: Forgetting to return values from setup(), making them inaccessible in the template
- **This context**: Trying to use `this` within the setup() function
- **Mixing patterns**: Creating confusing code by inconsistently mixing APIs
- **Over-composition**: Creating too many tiny composables that obscure the component logic

**Frequently Asked Questions:**

1. **Q: Do I need to migrate everything to Composition API?**

   A: No, both APIs are fully supported in Vue 3. You should consider migrating when:

   - You need better TypeScript support
   - Your components have complex, related logic spread across options
   - You want to extract and reuse logic between components
   - Your team prefers the more functional approach

2. **Q: What's the easiest way to start using Composition API in an existing project?**

   A: Start with composables for cross-cutting concerns:

   - Create utility composables like `useLocalStorage`, `useDebounce`, etc.
   - Extract complex logic from existing components into composables
   - Use these composables in your Options API components through the setup option

   ```js
   import { useLocalStorage } from './composables/useLocalStorage'

   export default {
     setup() {
       const { value: theme, updateValue: setTheme } = useLocalStorage('theme', 'light')

       return { theme, setTheme }
     },
     // Rest of the component in Options API
     data() { ... },
     methods: { ... }
   }
   ```

3. **Q: How do I handle component instance properties like $refs and $emit in Composition API?**

   A: These instance properties are replaced by explicit imports and declarations:

   - **Template refs**: Use the `ref()` function

     ```js
     const inputElement = ref(null);
     // In template: <input ref="inputElement">
     // Access with: inputElement.value
     ```

   - **Emitting events**: Use `defineEmits()`

     ```js
     const emit = defineEmits(["update", "delete"]);

     function updateItem() {
       emit("update", { id: 1, name: "Updated" });
     }
     ```

   - **Props**: Use `defineProps()`

     ```js
     const props = defineProps({
       item: { type: Object, required: true },
     });
     ```

   - **Slots**: Use `useSlots()`

     ```js
     import { useSlots } from "vue";

     const slots = useSlots();
     const hasFooter = computed(() => !!slots.footer);
     ```

## Key Takeaways for Migration

1. **Start Small**: Begin with extracting utility functions into composables
2. **Identify Problem Areas**: Look for components with complex, tangled logic
3. **Group by Feature**: Organize code by what it does, not by API categories
4. **Use Script Setup**: For the cleanest Composition API syntax in Single-File Components
5. **Leverage TypeScript**: The Composition API works particularly well with TypeScript
6. **Be Consistent**: Agree on patterns within your team to maintain code consistency
7. **Document Your Approach**: Create examples and guidelines for your specific project

The Composition API provides a more powerful and flexible way to organize your Vue components, especially as they grow in complexity. While the Options API remains a valid choice, understanding both patterns gives you more tools to write maintainable Vue applications.
