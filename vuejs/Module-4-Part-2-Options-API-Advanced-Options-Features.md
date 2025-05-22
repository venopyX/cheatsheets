# Module 4: Options API - Part 2: Advanced Options Features

## Core Problem

Advanced Vue.js Options API features help solve complex UI challenges that basic components can't easily address alone. They extend Vue's capabilities for DOM manipulation, performance optimization, and code reuse patterns beyond standard components.

## Key Assumptions

- You have basic knowledge of Vue 3 fundamentals
- You understand HTML, CSS, and JavaScript (ES6+)
- You're familiar with single-file components (.vue files) and the Options API
- You use Tailwind v4 for styling (no @apply or config files needed)

## Essential Concepts

### 1. Custom Directives

**Concise Explanation:**
Custom directives provide direct access to the DOM when you need low-level control beyond what templates offer. They are perfect for manipulating DOM elements directly without requiring a component. Custom directives operate on a specific element and can respond to its lifecycle.

**Where to Use:**

- When you need direct DOM manipulation
- For reusable DOM behaviors across components
- Input focus management, scroll position handling
- Third-party library integration
- Implementing specialized UI behaviors

**Code Snippet:**

```js
// main.js - Global directive registration
const app = createApp(App);

// Focus directive (v-focus)
app.directive("focus", {
  // Called when the bound element is mounted
  mounted(el) {
    el.focus();
  },
});

// Highlight directive (v-highlight)
app.directive("highlight", {
  // Directive hooks receive these arguments:
  // el: The element the directive is bound to
  // binding: An object containing arguments and modifiers
  // vnode: The virtual node produced by Vue
  mounted(el, binding) {
    // Check if a value is provided
    const color = binding.value || "yellow";

    // Apply the highlight
    el.style.backgroundColor = color;

    // Check if the directive has modifiers
    if (binding.modifiers.bold) {
      el.style.fontWeight = "bold";
    }

    // Check for arguments
    if (binding.arg === "text") {
      el.style.color = color;
      el.style.backgroundColor = "transparent";
    }
  },

  // Called when the bound element is updated (but children not updated)
  updated(el, binding) {
    const color = binding.value || "yellow";

    if (binding.arg === "text") {
      el.style.color = color;
    } else {
      el.style.backgroundColor = color;
    }
  },
});
```

```html
<!-- LocalDirective.vue - Component-local directive -->
<template>
  <div class="p-6 bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800 mb-4">
      Custom Directives Demo
    </h1>

    <!-- Auto-focus directive -->
    <div class="mb-6">
      <label class="block text-sm font-medium text-gray-700 mb-1">
        Auto-Focus Input
      </label>
      <input
        v-focus
        type="text"
        class="w-full px-4 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
        placeholder="This input will auto-focus"
      />
    </div>

    <!-- Highlight directive -->
    <div class="space-y-4 mb-6">
      <h2 class="text-lg font-medium text-gray-700">Highlight Examples</h2>

      <!-- Basic highlight -->
      <p v-highlight class="p-2 rounded">
        This paragraph has the default yellow highlight
      </p>

      <!-- With value -->
      <p v-highlight="'#e0f7fa'" class="p-2 rounded">
        This paragraph has a custom color highlight
      </p>

      <!-- With argument -->
      <p v-highlight:text="'#e91e63'" class="p-2 rounded">
        This text is highlighted with a custom color
      </p>

      <!-- With modifier -->
      <p v-highlight.bold="'#f0f4c3'" class="p-2 rounded">
        This paragraph is highlighted and bold
      </p>
    </div>

    <!-- Click outside directive -->
    <div class="mb-6">
      <h2 class="text-lg font-medium text-gray-700 mb-2">
        Click Outside Example
      </h2>

      <div class="relative">
        <button
          @click="showDropdown = !showDropdown"
          class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
        >
          {{ showDropdown ? "Close Menu" : "Open Menu" }}
        </button>

        <div
          v-if="showDropdown"
          v-click-outside="closeDropdown"
          class="absolute top-full left-0 mt-1 w-48 bg-white rounded shadow-lg z-10"
        >
          <ul class="py-1">
            <li>
              <a href="#" class="block px-4 py-2 hover:bg-gray-100">Profile</a>
            </li>
            <li>
              <a href="#" class="block px-4 py-2 hover:bg-gray-100">Settings</a>
            </li>
            <li>
              <a href="#" class="block px-4 py-2 hover:bg-gray-100">Logout</a>
            </li>
          </ul>
        </div>
      </div>
    </div>

    <!-- Tooltip directive -->
    <div class="mb-6">
      <h2 class="text-lg font-medium text-gray-700 mb-2">Tooltip Example</h2>

      <button
        v-tooltip="'This is a helpful tooltip'"
        class="px-4 py-2 bg-gray-200 text-gray-800 rounded hover:bg-gray-300"
      >
        Hover Me
      </button>
    </div>

    <!-- Permission directive -->
    <div>
      <h2 class="text-lg font-medium text-gray-700 mb-2">Permission Example</h2>

      <button
        v-permission="'admin'"
        class="px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700"
      >
        Admin Only Action
      </button>

      <p v-if="!hasAdminPermission" class="mt-2 text-sm text-gray-500">
        You don't have admin permissions to see this button.
      </p>
    </div>
  </div>
</template>

<script>
  export default {
    name: "DirectivesDemo",

    data() {
      return {
        showDropdown: false,
        hasAdminPermission: false, // In a real app, this would come from auth state
      };
    },

    methods: {
      closeDropdown() {
        this.showDropdown = false;
      },
    },

    // Component-local directives
    directives: {
      // Click outside directive
      clickOutside: {
        mounted(el, binding) {
          el._clickOutside = (event) => {
            // Check if click was outside the element
            if (!(el === event.target || el.contains(event.target))) {
              binding.value(event);
            }
          };
          document.addEventListener("click", el._clickOutside);
        },
        unmounted(el) {
          // Clean up the event listener
          document.removeEventListener("click", el._clickOutside);
        },
      },

      // Tooltip directive
      tooltip: {
        mounted(el, binding) {
          // Create tooltip element
          const tooltip = document.createElement("div");
          tooltip.className =
            "absolute px-2 py-1 text-xs bg-gray-900 text-white rounded opacity-0 -translate-y-2 transition-all pointer-events-none";
          tooltip.textContent = binding.value;
          tooltip.style.bottom = "100%";
          tooltip.style.left = "50%";
          tooltip.style.transform = "translateX(-50%)";
          tooltip.style.marginBottom = "5px";
          tooltip.style.whiteSpace = "nowrap";
          tooltip.style.zIndex = "10";

          // Store tooltip reference
          el._tooltip = tooltip;

          // Set element style for position reference
          el.style.position = "relative";

          // Add tooltip to element
          el.appendChild(tooltip);

          // Event listeners
          el.addEventListener("mouseenter", () => {
            tooltip.style.opacity = "1";
            tooltip.style.transform = "translateX(-50%) translateY(0)";
          });

          el.addEventListener("mouseleave", () => {
            tooltip.style.opacity = "0";
            tooltip.style.transform = "translateX(-50%) translateY(-2px)";
          });
        },
        unmounted(el) {
          // Clean up tooltip element
          if (el._tooltip) {
            el.removeChild(el._tooltip);
          }
        },
      },

      // Permission directive
      permission: {
        mounted(el, binding) {
          // Get required permission
          const requiredPermission = binding.value;

          // Check if user has permission
          const hasPermission = checkUserPermission(requiredPermission);

          // Hide element if user doesn't have permission
          if (!hasPermission) {
            el.style.display = "none";
          }
        },
      },
    },
  };

  // Mock function to check user permissions
  function checkUserPermission(permission) {
    // In a real app, this would check against user roles/permissions
    const userPermissions = ["user"]; // Example: user only has 'user' permission
    return userPermissions.includes(permission);
  }
</script>
```

**Real-World Example:**
In a complex UI for a CRM application, you could use custom directives to:

- `v-autoresize` for text areas that grow with content
- `v-permissions` to conditionally render elements based on user role
- `v-longpress` to detect when a user holds down on an element
- `v-intersect` to implement lazy-loading of images or infinite scrolling
- `v-format-number` to automatically format numbers as currency or percentages

```html
<!-- CRM Example -->
<template>
  <div class="p-6 bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800 mb-6">Customer Record</h1>

    <!-- Resizable notes field -->
    <div class="mb-6">
      <label class="block text-sm font-medium text-gray-700 mb-1">
        Customer Notes
      </label>
      <textarea
        v-model="customerNotes"
        v-autoresize
        class="w-full px-4 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
        placeholder="Enter notes about this customer..."
      ></textarea>
    </div>

    <!-- Financial information with number formatting -->
    <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-6">
      <div class="p-4 bg-gray-50 rounded">
        <h3 class="text-sm font-medium text-gray-500">Lifetime Value</h3>
        <p
          class="text-2xl font-bold"
          v-format-number:currency="customer.lifetimeValue"
        ></p>
      </div>

      <div class="p-4 bg-gray-50 rounded">
        <h3 class="text-sm font-medium text-gray-500">Open Invoices</h3>
        <p
          class="text-2xl font-bold"
          v-format-number:currency="customer.openInvoices"
        ></p>
      </div>

      <div class="p-4 bg-gray-50 rounded">
        <h3 class="text-sm font-medium text-gray-500">Growth Rate</h3>
        <p
          class="text-2xl font-bold"
          v-format-number:percentage="customer.growthRate"
        ></p>
      </div>
    </div>

    <!-- Admin actions with permission directive -->
    <div class="border-t pt-4 mt-4">
      <h2 class="text-lg font-medium text-gray-700 mb-2">Actions</h2>

      <div class="flex space-x-4">
        <button
          class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
        >
          Contact Customer
        </button>

        <button
          v-permission="'manager'"
          class="px-4 py-2 bg-yellow-600 text-white rounded hover:bg-yellow-700"
        >
          Apply Discount
        </button>

        <button
          v-permission="'admin'"
          class="px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700"
        >
          Delete Record
        </button>
      </div>
    </div>

    <!-- Long-press action example -->
    <div class="mt-6">
      <div
        v-longpress="activateQuickActions"
        class="p-4 border border-dashed rounded text-center cursor-pointer"
      >
        Hold to activate quick actions
      </div>

      <div v-if="showQuickActions" class="mt-2 p-4 bg-gray-50 rounded">
        <div class="text-sm font-medium mb-2">Quick Actions</div>
        <div class="flex flex-wrap gap-2">
          <button class="px-3 py-1 bg-gray-200 rounded text-sm">Call</button>
          <button class="px-3 py-1 bg-gray-200 rounded text-sm">Email</button>
          <button class="px-3 py-1 bg-gray-200 rounded text-sm">
            Schedule
          </button>
          <button class="px-3 py-1 bg-gray-200 rounded text-sm">Note</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: "CustomerRecord",

    data() {
      return {
        customerNotes:
          "Customer prefers email communication and has expressed interest in our premium plan.",
        showQuickActions: false,
        customer: {
          name: "Acme Corporation",
          lifetimeValue: 24500.75,
          openInvoices: 3200,
          growthRate: 0.12,
        },
      };
    },

    methods: {
      activateQuickActions() {
        this.showQuickActions = true;

        // Auto-hide after 5 seconds
        setTimeout(() => {
          this.showQuickActions = false;
        }, 5000);
      },
    },

    directives: {
      // Auto-resize textarea
      autoresize: {
        mounted(el) {
          // Set initial height
          el.style.height = "auto";
          el.style.height = el.scrollHeight + "px";

          // Function to update height
          const resize = () => {
            el.style.height = "auto";
            el.style.height = el.scrollHeight + "px";
          };

          // Add input event listener
          el.addEventListener("input", resize);

          // Store the reference to the function for cleanup
          el._autoResizeFunc = resize;
        },

        updated(el) {
          // Update height on content change
          el.style.height = "auto";
          el.style.height = el.scrollHeight + "px";
        },

        unmounted(el) {
          // Remove the event listener
          el.removeEventListener("input", el._autoResizeFunc);
        },
      },

      // Format numbers as currency or percentage
      formatNumber: {
        mounted(el, binding) {
          const format = binding.arg || "decimal";
          const value = binding.value || 0;

          let formattedValue;
          switch (format) {
            case "currency":
              formattedValue = new Intl.NumberFormat("en-US", {
                style: "currency",
                currency: "USD",
              }).format(value);
              break;
            case "percentage":
              formattedValue = new Intl.NumberFormat("en-US", {
                style: "percent",
                minimumFractionDigits: 1,
                maximumFractionDigits: 1,
              }).format(value);
              break;
            default:
              formattedValue = new Intl.NumberFormat("en-US").format(value);
          }

          el.textContent = formattedValue;
        },

        updated(el, binding) {
          const format = binding.arg || "decimal";
          const value = binding.value || 0;

          let formattedValue;
          switch (format) {
            case "currency":
              formattedValue = new Intl.NumberFormat("en-US", {
                style: "currency",
                currency: "USD",
              }).format(value);
              break;
            case "percentage":
              formattedValue = new Intl.NumberFormat("en-US", {
                style: "percent",
                minimumFractionDigits: 1,
                maximumFractionDigits: 1,
              }).format(value);
              break;
            default:
              formattedValue = new Intl.NumberFormat("en-US").format(value);
          }

          el.textContent = formattedValue;
        },
      },

      // Long press directive
      longpress: {
        mounted(el, binding) {
          // Time threshold in ms (700ms = 0.7s)
          const longPressThreshold = 700;

          // Variables to track press duration
          let pressTimer = null;

          // Start the press
          const startPress = (e) => {
            // Prevent context menu on long-press (mobile)
            if (e.type === "contextmenu") {
              e.preventDefault();
              return;
            }

            pressTimer = setTimeout(() => {
              // Call the bound function
              binding.value();
            }, longPressThreshold);
          };

          // Cancel the press
          const cancelPress = () => {
            if (pressTimer) {
              clearTimeout(pressTimer);
              pressTimer = null;
            }
          };

          // Add event listeners
          el.addEventListener("mousedown", startPress);
          el.addEventListener("touchstart", startPress);
          el.addEventListener("contextmenu", startPress); // For mobile

          // Cancel on release or move
          el.addEventListener("mouseup", cancelPress);
          el.addEventListener("mouseleave", cancelPress);
          el.addEventListener("touchend", cancelPress);
          el.addEventListener("touchcancel", cancelPress);

          // Store references for cleanup
          el._longpress = {
            startPress,
            cancelPress,
          };
        },

        unmounted(el) {
          // Remove event listeners
          const { startPress, cancelPress } = el._longpress || {};
          if (startPress && cancelPress) {
            el.removeEventListener("mousedown", startPress);
            el.removeEventListener("touchstart", startPress);
            el.removeEventListener("contextmenu", startPress);

            el.removeEventListener("mouseup", cancelPress);
            el.removeEventListener("mouseleave", cancelPress);
            el.removeEventListener("touchend", cancelPress);
            el.removeEventListener("touchcancel", cancelPress);
          }
        },
      },
    },
  };
</script>
```

**Common Pitfalls:**

- **Not cleaning up**: Forgetting to remove event listeners or observers in `unmounted`
- **Modifying DOM outside directives**: Causing conflicts with Vue's virtual DOM
- **Overusing directives**: Using directives for things better handled by components
- **Not using binding parameters effectively**: Missing opportunities to make directives flexible
- **Direct manipulation of element content**: Better to use v-html or templates

**Frequently Asked Questions:**

1. **Q: When should I use a directive vs. a component?**

   A: Use a directive when:

   - You need direct DOM manipulation
   - The behavior doesn't include its own template
   - You're enhancing an existing element

   Use a component when:

   - You need a full template with markup
   - You have complex state management
   - The UI pattern includes both behavior and presentation

2. **Q: Can custom directives access the component instance?**

   A: No, directives are isolated from the component instance. They receive:

   - **el**: The element they're attached to
   - **binding**: An object with value, argument, modifiers, etc.
   - **vnode**: The virtual node (providing some context)
   - **prevVNode**: The previous virtual node (only in updated hook)

   If a directive needs component data, pass it via the directive value:

   ```html
   <div
     v-my-directive="{ text: componentProperty, callback: methodName }"
   ></div>
   ```

3. **Q: What lifecycle hooks are available for directives?**

   A: Directive hooks (Vue 3):

   - **created**: Called before attributes or event listeners are applied
   - **beforeMount**: Called before the element is inserted into the DOM
   - **mounted**: Called when the element and its component are mounted
   - **beforeUpdate**: Called before the containing component updates
   - **updated**: Called after the containing component has updated
   - **beforeUnmount**: Called before the directive's element is removed
   - **unmounted**: Called when the element and its directive are unmounted

### 2. Filters (Deprecated but Legacy)

**Concise Explanation:**
Filters in Vue 2 provided a way to apply text formatting in templates. While deprecated in Vue 3, understanding them is important for maintaining legacy applications. In Vue 3, computed properties or methods are recommended instead.

**Where to Use (in Vue 2 or legacy applications):**

- Text formatting in templates (dates, currencies, uppercase/lowercase)
- Simple text transformations
- Number formatting

**Code Snippet (Vue 2 syntax):**

```html
<template>
  <div class="p-6 bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800 mb-4">Product Details</h1>

    <div class="space-y-4">
      <!-- Using filters in Vue 2 -->
      <div class="flex justify-between">
        <span class="text-gray-600">Product Name:</span>
        <span class="font-medium">{{ product.name | uppercase }}</span>
      </div>

      <div class="flex justify-between">
        <span class="text-gray-600">Price:</span>
        <span class="font-medium">{{ product.price | currency }}</span>
      </div>

      <div class="flex justify-between">
        <span class="text-gray-600">Created Date:</span>
        <span class="font-medium">{{ product.createdAt | formatDate }}</span>
      </div>

      <!-- Chaining filters -->
      <div class="flex justify-between">
        <span class="text-gray-600">Description:</span>
        <span class="font-medium"
          >{{ product.description | truncate(50) | capitalize }}</span
        >
      </div>
    </div>

    <div class="mt-6 pt-6 border-t">
      <h2 class="text-lg font-medium text-gray-700 mb-3">Vue 3 Alternatives</h2>

      <!-- Using computed properties in Vue 3 -->
      <div class="space-y-4">
        <div class="flex justify-between">
          <span class="text-gray-600">Product Name:</span>
          <span class="font-medium">{{ uppercaseName }}</span>
        </div>

        <div class="flex justify-between">
          <span class="text-gray-600">Price:</span>
          <span class="font-medium">{{ formattedPrice }}</span>
        </div>

        <div class="flex justify-between">
          <span class="text-gray-600">Created Date:</span>
          <span class="font-medium">{{ formattedDate }}</span>
        </div>

        <!-- Using methods in Vue 3 -->
        <div class="flex justify-between">
          <span class="text-gray-600">Description:</span>
          <span class="font-medium"
            >{{ capitalizeAndTruncate(product.description, 50) }}</span
          >
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: "ProductDetails",

    data() {
      return {
        product: {
          name: "wireless headphones",
          price: 149.99,
          createdAt: "2025-03-15T14:22:18Z",
          description:
            "high-quality wireless headphones with noise cancellation and long battery life.",
        },
      };
    },

    // Vue 2 Filters
    filters: {
      uppercase(value) {
        if (!value) return "";
        return value.toUpperCase();
      },

      currency(value) {
        if (!value) return "$0.00";
        return "$" + parseFloat(value).toFixed(2);
      },

      formatDate(value) {
        if (!value) return "";
        const date = new Date(value);
        return date.toLocaleDateString("en-US", {
          year: "numeric",
          month: "long",
          day: "numeric",
        });
      },

      truncate(value, length = 30) {
        if (!value) return "";
        if (value.length <= length) return value;
        return value.substr(0, length) + "...";
      },

      capitalize(value) {
        if (!value) return "";
        return value.charAt(0).toUpperCase() + value.slice(1);
      },
    },

    // Vue 3 alternatives - computed properties
    computed: {
      uppercaseName() {
        return this.product.name.toUpperCase();
      },

      formattedPrice() {
        return "$" + parseFloat(this.product.price).toFixed(2);
      },

      formattedDate() {
        const date = new Date(this.product.createdAt);
        return date.toLocaleDateString("en-US", {
          year: "numeric",
          month: "long",
          day: "numeric",
        });
      },
    },

    // Vue 3 alternatives - methods
    methods: {
      truncate(value, length = 30) {
        if (!value) return "";
        if (value.length <= length) return value;
        return value.substr(0, length) + "...";
      },

      capitalize(value) {
        if (!value) return "";
        return value.charAt(0).toUpperCase() + value.slice(1);
      },

      capitalizeAndTruncate(value, length) {
        const truncated = this.truncate(value, length);
        return this.capitalize(truncated);
      },
    },
  };
</script>
```

**Vue 3 Migration Examples:**

```html
<!-- Vue 2 with Filters -->
<template>
  <p>{{ message | capitalize }}</p>
  <p>Price: {{ price | currency }}</p>
</template>

<script>
  export default {
    filters: {
      capitalize(value) {
        if (!value) return "";
        return value.charAt(0).toUpperCase() + value.slice(1);
      },
      currency(value) {
        return "$" + parseFloat(value).toFixed(2);
      },
    },
  };
</script>
```

```html
<!-- Vue 3 with Computed Properties -->
<template>
  <p>{{ capitalizedMessage }}</p>
  <p>Price: {{ formattedPrice }}</p>
</template>

<script>
  export default {
    computed: {
      capitalizedMessage() {
        const value = this.message;
        if (!value) return "";
        return value.charAt(0).toUpperCase() + value.slice(1);
      },
      formattedPrice() {
        return "$" + parseFloat(this.price).toFixed(2);
      },
    },
  };
</script>
```

```html
<!-- Vue 3 with Methods -->
<template>
  <p>{{ capitalize(message) }}</p>
  <p>Price: {{ formatCurrency(price) }}</p>
</template>

<script>
  export default {
    methods: {
      capitalize(value) {
        if (!value) return "";
        return value.charAt(0).toUpperCase() + value.slice(1);
      },
      formatCurrency(value) {
        return "$" + parseFloat(value).toFixed(2);
      },
    },
  };
</script>
```

**Common Pitfalls:**

- **Trying to use filters in Vue 3**: They're removed and will cause errors
- **Complex logic in filters**: Filters should be simple transforms
- **Mutating values**: Filters should not mutate original values
- **Performance with many filters**: Chaining multiple filters can impact performance

**Frequently Asked Questions:**

1. **Q: Why were filters deprecated in Vue 3?**

   A: Filters were removed for several reasons:

   - They added special syntax that deviated from JavaScript
   - They created confusion with JavaScript's array filters
   - Most filter functionality can be achieved with computed properties or methods
   - They created an additional API surface to learn without significant benefits

2. **Q: What's the best way to migrate filters in a Vue 2 application?**

   A: Choose the most appropriate approach for each filter:

   - For displaying formatted data, use computed properties
   - For transformations needing parameters, use methods
   - For global filters, create utility functions in a separate file

   Example migration:

   ```js
   // Global filter in Vue 2
   Vue.filter("currency", function (value) {
     return "$" + parseFloat(value).toFixed(2);
   });

   // Migration to a utility function
   // utils/filters.js
   export function currency(value) {
     return "$" + parseFloat(value).toFixed(2);
   }

   // In a component
   import { currency } from "@/utils/filters";

   export default {
     methods: {
       currency, // Register as a method
     },
   };
   ```

3. **Q: Are there any filter-like features in Vue 3?**

   A: No, but for template formatting you can:

   - Use the `v-bind` syntax with JavaScript expressions:

     ```html
     <span>{{ price.toFixed(2) }}</span>
     ```

   - Use computed properties for derived values:

     ```js
     computed: {
       formattedPrice() {
         return '$' + this.price.toFixed(2)
       }
     }
     ```

   - Use utility functions in templates:
     ```html
     <span>{{ formatPrice(product.price) }}</span>
     ```

### 3. Render Functions

**Concise Explanation:**
Render functions provide a JavaScript alternative to templates, giving you full programmatic control over component output. They use the virtual DOM directly and are useful for complex rendering logic that's difficult to express in templates.

**Where to Use:**

- When creating highly dynamic components
- When the output depends on complex conditional logic
- When building higher-order components (HOCs)
- When creating library components that need programmatic flexibility
- When performance is critical and templates are too restrictive

**Code Snippet:**

```html
<script>
  import { h } from "vue";

  export default {
    name: "RenderFunctionDemo",

    props: {
      level: {
        type: Number,
        default: 1,
        validator: (level) => level >= 1 && level <= 6,
      },
      title: {
        type: String,
        required: true,
      },
    },

    // Render function
    render() {
      // Use h function to create virtual DOM elements
      // h(tag, props, children)
      return h(
        // Dynamic tag name based on prop
        "h" + this.level,
        // Props/attributes
        {
          class: "title-component",
          style: {
            fontSize: 28 - this.level * 2 + "px",
            marginBottom: "0.5em",
            fontWeight: 600,
            color: "#374151",
          },
        },
        // Children
        this.title
      );
    },
  };
</script>
```

```html
<template>
  <div class="p-6 bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800 mb-6">
      Render Functions Demo
    </h1>

    <!-- Using the custom title component -->
    <div class="space-y-4 mb-6">
      <div class="p-4 bg-gray-50 rounded">
        <dynamic-title :level="1" title="Level 1 Heading"></dynamic-title>
        <p class="text-gray-600">
          This is a paragraph following a level 1 heading.
        </p>
      </div>

      <div class="p-4 bg-gray-50 rounded">
        <dynamic-title :level="2" title="Level 2 Heading"></dynamic-title>
        <p class="text-gray-600">
          This is a paragraph following a level 2 heading.
        </p>
      </div>

      <div class="p-4 bg-gray-50 rounded">
        <dynamic-title :level="3" title="Level 3 Heading"></dynamic-title>
        <p class="text-gray-600">
          This is a paragraph following a level 3 heading.
        </p>
      </div>
    </div>

    <!-- Using components with render functions only -->
    <div class="space-y-4">
      <h2 class="text-lg font-medium text-gray-700 mb-2">Dynamic List</h2>

      <div class="flex space-x-4 mb-4">
        <button
          @click="listType = 'ordered'"
          :class="[
            'px-3 py-1 rounded text-sm',
            listType === 'ordered'
              ? 'bg-blue-600 text-white'
              : 'bg-gray-200 text-gray-800',
          ]"
        >
          Ordered List
        </button>

        <button
          @click="listType = 'unordered'"
          :class="[
            'px-3 py-1 rounded text-sm',
            listType === 'unordered'
              ? 'bg-blue-600 text-white'
              : 'bg-gray-200 text-gray-800',
          ]"
        >
          Unordered List
        </button>

        <button
          @click="listType = 'custom'"
          :class="[
            'px-3 py-1 rounded text-sm',
            listType === 'custom'
              ? 'bg-blue-600 text-white'
              : 'bg-gray-200 text-gray-800',
          ]"
        >
          Custom List
        </button>
      </div>

      <!-- Dynamic list component with render function -->
      <dynamic-list
        :type="listType"
        :items="listItems"
        @item-click="handleItemClick"
      ></dynamic-list>
    </div>
  </div>
</template>

<script>
  import { h } from "vue";
  import DynamicTitle from "./DynamicTitle.vue";

  // Dynamic list with render function
  const DynamicList = {
    props: {
      type: {
        type: String,
        default: "unordered",
        validator: (type) => ["ordered", "unordered", "custom"].includes(type),
      },
      items: {
        type: Array,
        default: () => [],
      },
    },

    emits: ["item-click"],

    render() {
      // Create list items
      const children = this.items.map((item, index) => {
        return h(
          "li",
          {
            key: item.id || index,
            class: "py-2 border-b last:border-0",
            onClick: () => this.$emit("item-click", item),
          },
          item.text || item
        );
      });

      // Determine list type
      if (this.type === "ordered") {
        return h(
          "ol",
          {
            class: "pl-5 list-decimal space-y-1",
          },
          children
        );
      } else if (this.type === "unordered") {
        return h(
          "ul",
          {
            class: "pl-5 list-disc space-y-1",
          },
          children
        );
      } else {
        // Custom list with checkboxes
        const customChildren = this.items.map((item, index) => {
          return h(
            "div",
            {
              key: item.id || index,
              class: "flex items-center p-2 border-b last:border-0",
            },
            [
              h("input", {
                type: "checkbox",
                class: "h-4 w-4 text-blue-600 rounded mr-3",
                checked: item.completed || false,
                onChange: () => this.$emit("item-click", item),
              }),
              h(
                "span",
                {
                  class: item.completed
                    ? "line-through text-gray-500"
                    : "text-gray-800",
                },
                item.text || item
              ),
            ]
          );
        });

        return h(
          "div",
          {
            class: "border rounded divide-y",
          },
          customChildren
        );
      }
    },
  };

  export default {
    name: "RenderFunctionDemo",

    components: {
      DynamicTitle,
      DynamicList,
    },

    data() {
      return {
        listType: "unordered",
        listItems: [
          { id: 1, text: "Learn Vue.js fundamentals", completed: true },
          { id: 2, text: "Master render functions", completed: false },
          { id: 3, text: "Build advanced components", completed: false },
          { id: 4, text: "Create a full application", completed: false },
        ],
      };
    },

    methods: {
      handleItemClick(item) {
        // Toggle completion for custom list
        if (this.listType === "custom") {
          item.completed = !item.completed;
        } else {
          console.log("Clicked item:", item);
        }
      },
    },
  };
</script>
```

**Advanced Render Function Example:**

```html
<script>
  import { h } from "vue";

  export default {
    name: "DataTable",

    props: {
      columns: {
        type: Array,
        required: true,
      },
      data: {
        type: Array,
        default: () => [],
      },
      sortable: {
        type: Boolean,
        default: false,
      },
      striped: {
        type: Boolean,
        default: false,
      },
    },

    data() {
      return {
        sortBy: null,
        sortDirection: "asc",
      };
    },

    computed: {
      sortedData() {
        if (!this.sortBy) return this.data;

        return [...this.data].sort((a, b) => {
          const aValue = a[this.sortBy];
          const bValue = b[this.sortBy];

          if (typeof aValue === "string") {
            return this.sortDirection === "asc"
              ? aValue.localeCompare(bValue)
              : bValue.localeCompare(aValue);
          } else {
            return this.sortDirection === "asc"
              ? aValue - bValue
              : bValue - aValue;
          }
        });
      },
    },

    methods: {
      toggleSort(column) {
        if (!this.sortable || !column.sortable) return;

        if (this.sortBy === column.key) {
          this.sortDirection = this.sortDirection === "asc" ? "desc" : "asc";
        } else {
          this.sortBy = column.key;
          this.sortDirection = "asc";
        }
      },

      renderSortIndicator(column) {
        if (!this.sortable || !column.sortable) return null;

        // Not sorted by this column
        if (this.sortBy !== column.key) {
          return h("span", { class: "ml-1 text-gray-400" }, "↕");
        }

        // Sorted by this column
        return h(
          "span",
          { class: "ml-1 text-blue-600" },
          this.sortDirection === "asc" ? "↑" : "↓"
        );
      },

      renderCell(column, row, rowIndex) {
        // Check if the column has a custom render function
        if (column.render) {
          return column.render(h, {
            row,
            column,
            index: rowIndex,
            value: row[column.key],
          });
        }

        // Default cell rendering
        return h(
          "td",
          {
            class: "px-4 py-2",
          },
          row[column.key]
        );
      },
    },

    render() {
      // Create table header
      const tableHeader = h("thead", { class: "bg-gray-100" }, [
        h(
          "tr",
          {},
          this.columns.map((column) =>
            h(
              "th",
              {
                class: [
                  "px-4 py-2 text-left text-sm font-medium text-gray-700",
                  this.sortable && column.sortable
                    ? "cursor-pointer hover:bg-gray-200"
                    : "",
                ],
                onClick: () => this.toggleSort(column),
              },
              [column.title, this.renderSortIndicator(column)]
            )
          )
        ),
      ]);

      // Create table body
      const tableBody = h(
        "tbody",
        {},
        this.sortedData.map((row, rowIndex) =>
          h(
            "tr",
            {
              class: this.striped && rowIndex % 2 ? "bg-gray-50" : "",
              key: row.id || rowIndex,
            },
            this.columns.map((column) => this.renderCell(column, row, rowIndex))
          )
        )
      );

      // Empty state if no data
      if (this.sortedData.length === 0) {
        return h(
          "div",
          { class: "text-center py-4 text-gray-500" },
          "No data available"
        );
      }

      // Return the table component
      return h("div", { class: "overflow-x-auto" }, [
        h("table", { class: "min-w-full border-collapse" }, [
          tableHeader,
          tableBody,
        ]),
      ]);
    },
  };
</script>
```

**JSX Syntax Alternative:**

```jsx
// With Babel JSX plugin
import { defineComponent } from "vue";

export default defineComponent({
  name: "AlertComponent",

  props: {
    type: {
      type: String,
      default: "info",
      validator: (type) =>
        ["info", "success", "warning", "error"].includes(type),
    },
    message: {
      type: String,
      required: true,
    },
    dismissible: {
      type: Boolean,
      default: false,
    },
  },

  data() {
    return {
      visible: true,
    };
  },

  methods: {
    dismiss() {
      this.visible = false;
      this.$emit("dismiss");
    },
  },

  render() {
    // Don't render if not visible
    if (!this.visible) return null;

    // Map type to styles
    const typeStyles = {
      info: "bg-blue-100 text-blue-800 border-blue-200",
      success: "bg-green-100 text-green-800 border-green-200",
      warning: "bg-yellow-100 text-yellow-800 border-yellow-200",
      error: "bg-red-100 text-red-800 border-red-200",
    };

    // Map type to icons
    const icons = {
      info: (
        <svg class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
          <path
            fill-rule="evenodd"
            d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-7-4a1 1 0 11-2 0 1 1 0 012 0zM9 9a1 1 0 000 2v3a1 1 0 001 1h1a1 1 0 100-2v-3a1 1 0 00-1-1H9z"
            clip-rule="evenodd"
          />
        </svg>
      ),
      success: (
        <svg class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
          <path
            fill-rule="evenodd"
            d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z"
            clip-rule="evenodd"
          />
        </svg>
      ),
      warning: (
        <svg class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
          <path
            fill-rule="evenodd"
            d="M8.257 3.099c.765-1.36 2.722-1.36 3.486 0l5.58 9.92c.75 1.334-.213 2.98-1.742 2.98H4.42c-1.53 0-2.493-1.646-1.743-2.98l5.58-9.92zM11 13a1 1 0 11-2 0 1 1 0 012 0zm-1-8a1 1 0 00-1 1v3a1 1 0 002 0V6a1 1 0 00-1-1z"
            clip-rule="evenodd"
          />
        </svg>
      ),
      error: (
        <svg class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
          <path
            fill-rule="evenodd"
            d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z"
            clip-rule="evenodd"
          />
        </svg>
      ),
    };

    // Render with JSX
    return (
      <div
        class={`p-4 border rounded flex items-start ${typeStyles[this.type]}`}
      >
        <div class="flex-shrink-0 mr-3">{icons[this.type]}</div>

        <div class="flex-1">
          {this.message}
          {this.$slots.default?.()}
        </div>

        {this.dismissible && (
          <button
            onClick={this.dismiss}
            class="ml-3 flex-shrink-0 text-gray-500 hover:text-gray-700 focus:outline-none"
          >
            <svg class="h-4 w-4" viewBox="0 0 20 20" fill="currentColor">
              <path
                fill-rule="evenodd"
                d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z"
                clip-rule="evenodd"
              />
            </svg>
          </button>
        )}
      </div>
    );
  },
});
```

**Common Pitfalls:**

- **Not providing keys for list items**: Missing keys can cause performance issues and bugs
- **Accessing instance properties**: Directly using `this` in the wrong context
- **Not handling slots properly**: Forgetting to include slots or using them incorrectly
- **Verbose code**: Render functions can be harder to read than templates
- **Mixing with templates**: Confusion when combining render functions with template-based components

**Frequently Asked Questions:**

1. **Q: When should I use render functions instead of templates?**

   A: Use render functions when:

   - You need high-level of dynamic rendering that's difficult with v-if/v-for
   - You're creating components with variable tags/structures
   - You're building a component library that needs to be very flexible
   - You need fine-grained control over rendering
   - You have logic that's easier to express in JavaScript than in template syntax

2. **Q: Are render functions more performant than templates?**

   A: Not necessarily. Template optimization in Vue's compiler is very good:

   - Templates are pre-compiled into optimized render functions
   - Manual render functions bypass this optimization step
   - Complex render functions can be less efficient if not carefully written
   - They can be faster for certain edge cases, but this shouldn't be the primary reason to use them

3. **Q: How do I handle slots in render functions?**

   A: Slots are accessed from the context's slots property:

   ```js
   render() {
     // Default slot
     const defaultSlot = this.$slots.default ? this.$slots.default() : 'Default content'

     // Named slot
     const headerSlot = this.$slots.header?.() || h('h2', 'Default Header')

     // Scoped slot
     const itemSlots = this.items.map(item =>
       this.$slots.item?.(item) || h('div', item.text)
     )

     return h('div', [
       headerSlot,
       h('div', { class: 'items-container' }, itemSlots),
       defaultSlot
     ])
   }
   ```

### 4. Transitions and Animations

**Concise Explanation:**
Vue's transition system provides ways to animate entering/leaving elements and transitioning between states. It integrates with CSS transitions/animations and JavaScript animation libraries, making it easy to add motion to your applications.

**Where to Use:**

- Element appearance/disappearance (modals, notifications)
- List item additions/removals
- Page transitions
- State-based UI changes
- Hover/click effects
- Loading states
- Expanding/collapsing sections

**Code Snippet:**

```html
<template>
  <div class="p-6 bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800 mb-6">Transitions Demo</h1>

    <!-- Basic Transition -->
    <div class="mb-8">
      <h2 class="text-lg font-medium text-gray-700 mb-3">Basic Transition</h2>
      <div class="flex items-center space-x-4">
        <button
          @click="showBox = !showBox"
          class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
        >
          {{ showBox ? "Hide" : "Show" }} Box
        </button>

        <transition name="fade">
          <div v-if="showBox" class="h-16 w-16 bg-blue-500 rounded"></div>
        </transition>
      </div>
    </div>

    <!-- Different Transitions -->
    <div class="mb-8 grid grid-cols-1 md:grid-cols-3 gap-6">
      <div>
        <h3 class="text-md font-medium text-gray-700 mb-2">Slide</h3>
        <button
          @click="showSlide = !showSlide"
          class="mb-2 px-3 py-1 bg-green-600 text-white text-sm rounded hover:bg-green-700"
        >
          Toggle
        </button>

        <transition name="slide">
          <div
            v-if="showSlide"
            class="p-4 bg-green-100 border border-green-200 rounded"
          >
            Slide transition
          </div>
        </transition>
      </div>

      <div>
        <h3 class="text-md font-medium text-gray-700 mb-2">Scale</h3>
        <button
          @click="showScale = !showScale"
          class="mb-2 px-3 py-1 bg-yellow-600 text-white text-sm rounded hover:bg-yellow-700"
        >
          Toggle
        </button>

        <transition name="scale">
          <div
            v-if="showScale"
            class="p-4 bg-yellow-100 border border-yellow-200 rounded"
          >
            Scale transition
          </div>
        </transition>
      </div>

      <div>
        <h3 class="text-md font-medium text-gray-700 mb-2">Rotate</h3>
        <button
          @click="showRotate = !showRotate"
          class="mb-2 px-3 py-1 bg-purple-600 text-white text-sm rounded hover:bg-purple-700"
        >
          Toggle
        </button>

        <transition name="rotate">
          <div
            v-if="showRotate"
            class="p-4 bg-purple-100 border border-purple-200 rounded"
          >
            Rotate transition
          </div>
        </transition>
      </div>
    </div>

    <!-- Mode Transitions -->
    <div class="mb-8">
      <h2 class="text-lg font-medium text-gray-700 mb-3">Transition Modes</h2>
      <div class="flex items-center space-x-4">
        <button
          @click="toggleComponent"
          class="px-4 py-2 bg-gray-600 text-white rounded hover:bg-gray-700"
        >
          Switch Component
        </button>

        <div class="overflow-hidden relative h-16 w-64 border rounded">
          <transition name="fade" mode="out-in">
            <div
              v-if="activeComponent === 'A'"
              key="a"
              class="absolute inset-0 flex items-center justify-center bg-red-100"
            >
              Component A
            </div>
            <div
              v-else
              key="b"
              class="absolute inset-0 flex items-center justify-center bg-blue-100"
            >
              Component B
            </div>
          </transition>
        </div>
      </div>
    </div>

    <!-- Transition List -->
    <div class="mb-8">
      <h2 class="text-lg font-medium text-gray-700 mb-3">List Transitions</h2>

      <div class="mb-4 flex space-x-2">
        <input
          v-model="newItem"
          @keyup.enter="addItem"
          type="text"
          placeholder="Add item..."
          class="flex-1 px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
        />
        <button
          @click="addItem"
          class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
        >
          Add
        </button>
      </div>

      <transition-group
        tag="ul"
        name="list"
        class="border rounded divide-y overflow-hidden"
      >
        <li
          v-for="(item, index) in items"
          :key="item.id"
          class="flex items-center justify-between p-3 bg-white"
        >
          <span>{{ item.text }}</span>
          <button
            @click="removeItem(index)"
            class="text-red-500 hover:text-red-700"
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
        </li>
      </transition-group>
    </div>

    <!-- JavaScript Hooks -->
    <div>
      <h2 class="text-lg font-medium text-gray-700 mb-3">JavaScript Hooks</h2>

      <button
        @click="showJS = !showJS"
        class="px-4 py-2 bg-indigo-600 text-white rounded hover:bg-indigo-700"
      >
        Toggle with JS Animation
      </button>

      <transition
        @before-enter="onBeforeEnter"
        @enter="onEnter"
        @after-enter="onAfterEnter"
        @leave="onLeave"
        :css="false"
      >
        <div
          v-if="showJS"
          class="mt-4 p-4 bg-indigo-100 border border-indigo-200 rounded"
          ref="jsBox"
        >
          Using JavaScript hooks for animation
        </div>
      </transition>
    </div>
  </div>
</template>

<script>
  import gsap from "gsap"; // Assuming GSAP is installed

  export default {
    name: "TransitionsDemo",

    data() {
      return {
        // Basic transitions
        showBox: true,
        showSlide: true,
        showScale: true,
        showRotate: true,

        // Mode transitions
        activeComponent: "A",

        // List transitions
        newItem: "",
        items: [
          { id: 1, text: "Learn Vue Transitions" },
          { id: 2, text: "Build animated components" },
          { id: 3, text: "Refine animation timing" },
        ],
        nextId: 4,

        // JavaScript hooks
        showJS: false,
      };
    },

    methods: {
      toggleComponent() {
        this.activeComponent = this.activeComponent === "A" ? "B" : "A";
      },

      addItem() {
        if (this.newItem.trim()) {
          this.items.push({
            id: this.nextId++,
            text: this.newItem,
          });
          this.newItem = "";
        }
      },

      removeItem(index) {
        this.items.splice(index, 1);
      },

      // JavaScript animation hooks for GSAP
      onBeforeEnter(el) {
        el.style.opacity = 0;
        el.style.height = "0px";
        el.style.overflow = "hidden";
      },

      onEnter(el, done) {
        gsap.to(el, {
          duration: 0.5,
          height: "auto",
          opacity: 1,
          padding: "1rem",
          onComplete: done,
        });
      },

      onAfterEnter(el) {
        console.log("Animation completed");
      },

      onLeave(el, done) {
        gsap.to(el, {
          duration: 0.5,
          height: 0,
          opacity: 0,
          padding: 0,
          onComplete: done,
        });
      },
    },
  };
</script>

<style>
  /* Basic fade transition */
  .fade-enter-active,
  .fade-leave-active {
    transition: opacity 0.3s ease;
  }

  .fade-enter-from,
  .fade-leave-to {
    opacity: 0;
  }

  /* Slide transition */
  .slide-enter-active,
  .slide-leave-active {
    transition: all 0.3s ease;
  }

  .slide-enter-from,
  .slide-leave-to {
    transform: translateX(20px);
    opacity: 0;
  }

  /* Scale transition */
  .scale-enter-active,
  .scale-leave-active {
    transition: all 0.3s ease;
  }

  .scale-enter-from,
  .scale-leave-to {
    transform: scale(0.9);
    opacity: 0;
  }

  /* Rotate transition */
  .rotate-enter-active,
  .rotate-leave-active {
    transition: all 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  }

  .rotate-enter-from,
  .rotate-leave-to {
    transform: rotateY(90deg);
    opacity: 0;
  }

  /* List transitions */
  .list-enter-active,
  .list-leave-active {
    transition: all 0.5s ease;
  }

  .list-enter-from {
    opacity: 0;
    transform: translateY(-30px);
  }

  .list-leave-to {
    opacity: 0;
    transform: translateX(30px);
  }

  /* Move transition for remaining items when item is deleted */
  .list-move {
    transition: transform 0.5s ease;
  }

  /* Optional: Ensure leaving items don't affect layout during animation */
  .list-leave-active {
    position: absolute;
  }
</style>
```

**Advanced Page Transitions Example:**

```html
<template>
  <div class="relative">
    <!-- Page transition wrapper -->
    <transition
      :name="transitionName"
      @before-leave="beforeLeave"
      @after-enter="afterEnter"
    >
      <component
        :is="currentView"
        :key="$route.path"
        class="page-container"
      ></component>
    </transition>

    <!-- Loading overlay -->
    <div
      v-if="isLoading"
      class="fixed inset-0 bg-white bg-opacity-80 flex items-center justify-center z-10"
    >
      <div class="loader">
        <svg
          class="animate-spin h-10 w-10 text-blue-600"
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
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: "PageTransitionWrapper",

    props: {
      currentView: {
        type: Object,
        required: true,
      },
    },

    data() {
      return {
        isLoading: false,
        transitionName: "slide-left",
        previousRoute: null,
      };
    },

    watch: {
      $route(to, from) {
        // Store previous route
        this.previousRoute = from;

        // Determine transition direction based on navigation
        const toDepth = to.path.split("/").length;
        const fromDepth = from.path.split("/").length;

        if (to.meta.transition) {
          // Use route-specific transition if defined
          this.transitionName = to.meta.transition;
        } else if (fromDepth < toDepth) {
          // Going deeper in navigation - slide left
          this.transitionName = "slide-left";
        } else if (fromDepth > toDepth) {
          // Going back in navigation - slide right
          this.transitionName = "slide-right";
        } else {
          // Same level - fade
          this.transitionName = "fade";
        }

        // Show loading if component needs to load data
        this.isLoading = !!to.meta.requiresLoading;
      },
    },

    methods: {
      beforeLeave(el) {
        // Prevent scrolling during transition
        document.body.classList.add("overflow-hidden");
      },

      afterEnter(el) {
        // Resume scrolling after transition
        document.body.classList.remove("overflow-hidden");

        // Hide loading state
        this.isLoading = false;

        // Scroll to top on new page
        window.scrollTo(0, 0);
      },
    },
  };
</script>

<style>
  .page-container {
    position: absolute;
    width: 100%;
    min-height: 100vh;
    transition-property: all;
  }

  /* Fade transition */
  .fade-enter-active,
  .fade-leave-active {
    transition-duration: 300ms;
  }

  .fade-enter-from,
  .fade-leave-to {
    opacity: 0;
  }

  /* Slide left transition */
  .slide-left-enter-active,
  .slide-left-leave-active {
    transition-duration: 300ms;
  }

  .slide-left-enter-from {
    transform: translateX(100%);
  }

  .slide-left-leave-to {
    transform: translateX(-100%);
    opacity: 0;
  }

  /* Slide right transition */
  .slide-right-enter-active,
  .slide-right-leave-active {
    transition-duration: 300ms;
  }

  .slide-right-enter-from {
    transform: translateX(-100%);
  }

  .slide-right-leave-to {
    transform: translateX(100%);
    opacity: 0;
  }
</style>
```

**Modal Transition Example:**

```html
<template>
  <div>
    <!-- Modal trigger -->
    <button
      @click="showModal = true"
      class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
    >
      Open Modal
    </button>

    <!-- Modal with transitions -->
    <transition name="modal">
      <div
        v-if="showModal"
        class="fixed inset-0 flex items-center justify-center z-50"
      >
        <!-- Backdrop -->
        <div
          class="fixed inset-0 bg-black bg-opacity-50"
          @click="showModal = false"
        ></div>

        <!-- Modal content -->
        <div
          class="bg-white rounded-lg shadow-xl w-full max-w-md m-3 z-10"
          role="dialog"
          aria-modal="true"
        >
          <div class="px-6 py-4 border-b">
            <div class="flex justify-between items-center">
              <h3 class="text-lg font-medium text-gray-900">Modal Title</h3>
              <button
                @click="showModal = false"
                class="text-gray-400 hover:text-gray-500 focus:outline-none"
              >
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
                    d="M6 18L18 6M6 6l12 12"
                  />
                </svg>
              </button>
            </div>
          </div>

          <div class="px-6 py-4">
            <p class="text-gray-700">
              This modal uses transitions for a smooth user experience.
            </p>
            <p class="mt-2 text-gray-600">
              Click outside or the X to close it.
            </p>
          </div>

          <div class="px-6 py-3 bg-gray-50 flex justify-end rounded-b-lg">
            <button
              @click="showModal = false"
              class="px-4 py-2 bg-gray-200 text-gray-800 rounded hover:bg-gray-300 mr-2"
            >
              Cancel
            </button>
            <button
              @click="confirmModal"
              class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
            >
              Confirm
            </button>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
  export default {
    name: "ModalTransition",

    data() {
      return {
        showModal: false,
      };
    },

    methods: {
      confirmModal() {
        // Handle confirm action
        console.log("Modal confirmed");
        this.showModal = false;
      },
    },
  };
</script>

<style>
  .modal-enter-active,
  .modal-leave-active {
    transition: all 0.3s ease;
  }

  .modal-enter-from {
    opacity: 0;
  }

  .modal-enter-from .bg-white {
    transform: scale(0.9);
  }

  .modal-leave-to {
    opacity: 0;
  }

  .modal-leave-to .bg-white {
    transform: scale(0.9);
  }

  .bg-white {
    transition: transform 0.3s ease;
  }
</style>
```

**Real-World Example:**
In an e-commerce application, transitions are crucial for enhancing the user experience:

- Product list animations when filtering/sorting
- Smooth cart updates when adding/removing items
- Image gallery transitions
- Page transitions between product listing and details
- Checkout step animations
- Form validation feedback animations

**Common Pitfalls:**

- **Performance issues**: Too many animated elements or complex transitions
- **Layout shifts**: Transitions that cause unexpected layout changes
- **Missing key attributes**: Not using keys when transitioning between elements
- **Accessibility issues**: Animations that can't be disabled for users with vestibular disorders
- **Over-animation**: Too many animations can be distracting or annoying

**Frequently Asked Questions:**

1. **Q: What's the difference between transition and transition-group?**

   A: The key differences are:

   - **transition**: For single element/component transitions
   - **transition-group**: For lists of elements/components

   Specific differences:

   - `<transition-group>` renders a real element (default is `<span>`, changeable with `tag` prop)
   - `<transition>` is just a wrapper that doesn't render to the DOM
   - `<transition-group>` requires unique `key` attributes for each child
   - `<transition-group>` has additional CSS classes (.move) for when items change positions

2. **Q: How do I animate route changes in a Vue application?**

   A: Use nested routes with `<transition>`:

   ```html
   <template>
     <div>
       <transition name="fade" mode="out-in">
         <router-view :key="$route.path"></router-view>
       </transition>
     </div>
   </template>
   ```

   For more dynamic transitions based on route:

   ```js
   <script>
   export default {
     computed: {
       transitionName() {
         // Return different transition names based on routes
         return this.$route.meta.transition || 'fade'
       }
     }
   }
   </script>

   <template>
     <transition :name="transitionName" mode="out-in">
       <router-view :key="$route.path"></router-view>
     </transition>
   </template>
   ```

3. **Q: Can I reuse transitions across components?**

   A: Yes, you can reuse transitions by:

   - Using global CSS classes for transitions:

     ```css
     /* Global styles */
     .fade-enter-active,
     .fade-leave-active {
       transition: opacity 0.3s;
     }
     .fade-enter-from,
     .fade-leave-to {
       opacity: 0;
     }
     ```

   - Creating a reusable transition component:

     ```html
     <template>
       <transition
         name="custom-transition"
         :css="!disableCss"
         :duration="duration"
         @before-enter="beforeEnter"
         @enter="enter"
         @leave="leave"
       >
         <slot></slot>
       </transition>
     </template>

     <script>
       export default {
         name: "FadeTransition",
         props: {
           duration: {
             type: [Number, Object],
             default: 300,
           },
           disableCss: {
             type: Boolean,
             default: false,
           },
         },
         methods: {
           beforeEnter(el) {
             this.$emit("before-enter", el);
           },
           enter(el, done) {
             this.$emit("enter", el, done);
             if (this.disableCss) done();
           },
           leave(el, done) {
             this.$emit("leave", el, done);
             if (this.disableCss) done();
           },
         },
       };
     </script>

     <style>
       .custom-transition-enter-active,
       .custom-transition-leave-active {
         transition: all 0.3s ease;
       }
       .custom-transition-enter-from,
       .custom-transition-leave-to {
         opacity: 0;
         transform: translateY(-20px);
       }
     </style>
     ```

### 5. Keep-alive Components

**Concise Explanation:**
`<keep-alive>` preserves component state and avoids re-rendering when components are toggled. This is especially useful for expensive components or when you need to maintain component state when switching between tabs, routes, or conditional displays.

**Where to Use:**

- Tab interfaces
- Wizard or multi-step forms
- List/detail views
- Cached routes
- Conditionally rendered components that need to preserve state

**Code Snippet:**

```html
<template>
  <div class="p-6 bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800 mb-6">Keep-Alive Demo</h1>

    <!-- Tab navigation -->
    <div class="border-b mb-6">
      <nav class="flex -mb-px">
        <button
          v-for="tab in tabs"
          :key="tab.id"
          @click="currentTab = tab.id"
          :class="[
            'py-3 px-4 text-center',
            currentTab === tab.id
              ? 'border-b-2 border-blue-500 text-blue-600 font-medium'
              : 'text-gray-500 hover:text-gray-700 hover:border-gray-300',
          ]"
        >
          {{ tab.name }}
        </button>
      </nav>
    </div>

    <!-- Tab content with keep-alive -->
    <keep-alive>
      <component :is="currentTabComponent" :key="currentTab"></component>
    </keep-alive>

    <!-- Activation state demonstration -->
    <div class="mt-6 bg-gray-50 p-4 rounded">
      <h2 class="text-md font-medium text-gray-700 mb-2">
        Component Activation Log
      </h2>
      <div class="h-40 overflow-y-auto border rounded p-2 bg-white">
        <div
          v-for="(log, index) in activationLog"
          :key="index"
          class="py-1 text-sm"
          :class="{
            'text-blue-600': log.includes('activated'),
            'text-red-600': log.includes('deactivated'),
          }"
        >
          {{ log }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  // Tab components
  const ProfileTab = {
    name: "ProfileTab",
    template: `
    <div>
      <h2 class="text-lg font-medium text-gray-800 mb-4">User Profile</h2>
      
      <div class="space-y-4">
        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1">Name</label>
          <input 
            v-model="profile.name" 
            type="text" 
            class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
        </div>
        
        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1">Email</label>
          <input 
            v-model="profile.email" 
            type="email" 
            class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
        </div>
        
        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1">Bio</label>
          <textarea 
            v-model="profile.bio" 
            rows="3" 
            class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
          ></textarea>
        </div>
      </div>
    </div>
  `,
    data() {
      return {
        profile: {
          name: "John Doe",
          email: "john@example.com",
          bio: "Software developer with a passion for Vue.js",
        },
      };
    },
    activated() {
      this.$emit(
        "log",
        `ProfileTab activated at ${new Date().toLocaleTimeString()}`
      );
    },
    deactivated() {
      this.$emit(
        "log",
        `ProfileTab deactivated at ${new Date().toLocaleTimeString()}`
      );
    },
  };

  const SettingsTab = {
    name: "SettingsTab",
    template: `
    <div>
      <h2 class="text-lg font-medium text-gray-800 mb-4">User Settings</h2>
      
      <div class="space-y-4">
        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1">Theme</label>
          <select 
            v-model="settings.theme" 
            class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
          >
            <option value="light">Light</option>
            <option value="dark">Dark</option>
            <option value="system">System Default</option>
          </select>
        </div>
        
        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1">Language</label>
          <select 
            v-model="settings.language" 
            class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
          >
            <option value="en">English</option>
            <option value="fr">French</option>
            <option value="es">Spanish</option>
            <option value="de">German</option>
          </select>
        </div>
        
        <div class="flex items-center">
          <input 
            id="notifications" 
            v-model="settings.notifications" 
            type="checkbox" 
            class="h-4 w-4 text-blue-600 focus:ring-blue-500 border-gray-300 rounded"
          />
          <label for="notifications" class="ml-2 block text-sm text-gray-700">
            Enable notifications
          </label>
        </div>
      </div>
    </div>
  `,
    data() {
      return {
        settings: {
          theme: "light",
          language: "en",
          notifications: true,
        },
      };
    },
    activated() {
      this.$emit(
        "log",
        `SettingsTab activated at ${new Date().toLocaleTimeString()}`
      );
    },
    deactivated() {
      this.$emit(
        "log",
        `SettingsTab deactivated at ${new Date().toLocaleTimeString()}`
      );
    },
  };

  const NotificationsTab = {
    name: "NotificationsTab",
    template: `
    <div>
      <h2 class="text-lg font-medium text-gray-800 mb-4">Notifications</h2>
      
      <div class="space-y-4">
        <div class="flex justify-between mb-4">
          <button 
            @click="markAllAsRead"
            class="px-3 py-1 text-sm bg-blue-600 text-white rounded hover:bg-blue-700"
          >
            Mark All as Read
          </button>
          
          <button 
            @click="refreshNotifications"
            class="px-3 py-1 text-sm bg-gray-200 text-gray-800 rounded hover:bg-gray-300"
          >
            Refresh
          </button>
        </div>
        
        <div v-if="loading" class="py-8 flex justify-center">
          <svg class="animate-spin h-8 w-8 text-blue-600" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
          </svg>
        </div>
        
        <div v-else-if="notifications.length === 0" class="py-8 text-center text-gray-500">
          <p>No notifications to display</p>
        </div>
        
        <div v-else class="border rounded divide-y">
          <div 
            v-for="notification in notifications"
            :key="notification.id"
            class="p-3 flex items-start"
            :class="{'bg-blue-50': !notification.read}"
          >
            <div class="flex-shrink-0 mr-3">
              <div 
                class="h-8 w-8 rounded-full flex items-center justify-center"
                :class="{
                  'bg-blue-100 text-blue-600': notification.type === 'message',
                  'bg-green-100 text-green-600': notification.type === 'success',
                  'bg-yellow-100 text-yellow-600': notification.type === 'warning',
                  'bg-red-100 text-red-600': notification.type === 'error'
                }"
              >
                <svg v-if="notification.type === 'message'" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 10h.01M12 10h.01M16 10h.01M9 16H5a2 2 0 01-2-2V6a2 2 0 012-2h14a2 2 0 012 2v8a2 2 0 01-2 2h-5l-5 5v-5z" />
                </svg>
                <svg v-else-if="notification.type === 'success'" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
                </svg>
                <svg v-else-if="notification.type === 'warning'" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
                </svg>
                <svg v-else-if="notification.type === 'error'" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
                </svg>
              </div>
            </div>
            
            <div class="flex-1">
              <div class="flex justify-between">
                <p class="text-sm font-medium" :class="notification.read ? 'text-gray-700' : 'text-gray-900'">
                  {{ notification.title }}
                </p>
                <span class="text-xs text-gray-500">{{ formatTime(notification.time) }}</span>
              </div>
              <p class="text-sm text-gray-600">{{ notification.message }}</p>
            </div>
            
            <button 
              v-if="!notification.read"
              @click="markAsRead(notification.id)"
              class="ml-2 text-xs text-blue-600 hover:text-blue-800"
            >
              Mark as read
            </button>
          </div>
        </div>
      </div>
    </div>
  `,
    data() {
      return {
        loading: false,
        notifications: [
          {
            id: 1,
            type: "message",
            title: "New Message",
            message: "You received a new message from Sarah",
            time: new Date(Date.now() - 300000), // 5 minutes ago
            read: false,
          },
          {
            id: 2,
            type: "success",
            title: "Task Completed",
            message: 'Your task "Update documentation" was completed',
            time: new Date(Date.now() - 3600000), // 1 hour ago
            read: true,
          },
          {
            id: 3,
            type: "warning",
            title: "Storage Space",
            message: "You are using 80% of your storage space",
            time: new Date(Date.now() - 86400000), // 1 day ago
            read: false,
          },
        ],
      };
    },
    methods: {
      markAsRead(id) {
        const notification = this.notifications.find((n) => n.id === id);
        if (notification) {
          notification.read = true;
        }
      },

      markAllAsRead() {
        this.notifications.forEach((notification) => {
          notification.read = true;
        });
      },

      refreshNotifications() {
        this.loading = true;

        // Simulate API call
        setTimeout(() => {
          // Add a new notification
          this.notifications.unshift({
            id: Date.now(),
            type: "message",
            title: "New Update",
            message: "Application was updated to version 2.0",
            time: new Date(),
            read: false,
          });

          this.loading = false;
        }, 1000);
      },

      formatTime(date) {
        if (!date) return "";

        const now = new Date();
        const diff = now - date; // difference in milliseconds

        // Less than a minute ago
        if (diff < 60000) {
          return "Just now";
        }

        // Less than an hour ago
        if (diff < 3600000) {
          const minutes = Math.floor(diff / 60000);
          return `${minutes}m ago`;
        }

        // Less than a day ago
        if (diff < 86400000) {
          const hours = Math.floor(diff / 3600000);
          return `${hours}h ago`;
        }

        // Less than a week ago
        if (diff < 604800000) {
          const days = Math.floor(diff / 86400000);
          return `${days}d ago`;
        }

        // Format as date
        return date.toLocaleDateString();
      },
    },
    activated() {
      this.$emit(
        "log",
        `NotificationsTab activated at ${new Date().toLocaleTimeString()}`
      );
    },
    deactivated() {
      this.$emit(
        "log",
        `NotificationsTab deactivated at ${new Date().toLocaleTimeString()}`
      );
    },
  };

  export default {
    name: "KeepAliveDemo",

    components: {
      ProfileTab,
      SettingsTab,
      NotificationsTab,
    },

    data() {
      return {
        currentTab: "profile",
        tabs: [
          { id: "profile", name: "Profile" },
          { id: "settings", name: "Settings" },
          { id: "notifications", name: "Notifications" },
        ],
        activationLog: [],
      };
    },

    computed: {
      currentTabComponent() {
        const componentMap = {
          profile: "ProfileTab",
          settings: "SettingsTab",
          notifications: "NotificationsTab",
        };

        return componentMap[this.currentTab] || "ProfileTab";
      },
    },

    methods: {
      logActivation(message) {
        this.activationLog.unshift(message);

        // Limit log length
        if (this.activationLog.length > 20) {
          this.activationLog.pop();
        }
      },
    },

    // Key lifecycle hooks for the parent component
    mounted() {
      this.logActivation(
        `Parent component mounted at ${new Date().toLocaleTimeString()}`
      );
    },
  };
</script>
```

**Selective Caching with Include/Exclude:**

```html
<template>
  <div class="p-6 bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800 mb-6">
      Advanced Keep-Alive Usage
    </h1>

    <!-- Tab navigation -->
    <div class="border-b mb-6">
      <nav class="flex -mb-px">
        <button
          v-for="tab in tabs"
          :key="tab.id"
          @click="currentTab = tab.id"
          :class="[
            'py-3 px-4 text-center',
            currentTab === tab.id
              ? 'border-b-2 border-blue-500 text-blue-600 font-medium'
              : 'text-gray-500 hover:text-gray-700 hover:border-gray-300',
          ]"
        >
          {{ tab.name }}
        </button>
      </nav>
    </div>

    <!-- Selectively cache only specific components -->
    <keep-alive :include="['ProfileTab', 'SettingsTab']">
      <component :is="currentTabComponent" @refresh="refresh"></component>
    </keep-alive>

    <!-- Debug information -->
    <div class="mt-6 bg-gray-50 p-4 rounded">
      <h2 class="text-md font-medium text-gray-700 mb-2">Cache Status</h2>
      <div class="space-y-2">
        <div class="flex">
          <span class="font-medium w-32">Current Tab:</span>
          <span>{{ currentTab }}</span>
        </div>
        <div class="flex">
          <span class="font-medium w-32">Cached:</span>
          <span>{{ isCurrentTabCached ? "Yes" : "No" }}</span>
        </div>
        <div class="flex">
          <span class="font-medium w-32">Last Refresh:</span>
          <span>{{ lastRefresh }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  // Dynamic import for components
  const ProfileTab = () => import("./ProfileTab.vue");
  const SettingsTab = () => import("./SettingsTab.vue");
  const AnalyticsTab = () => import("./AnalyticsTab.vue");

  export default {
    name: "SelectiveCaching",

    components: {
      ProfileTab,
      SettingsTab,
      AnalyticsTab,
    },

    data() {
      return {
        currentTab: "profile",
        tabs: [
          { id: "profile", name: "Profile" },
          { id: "settings", name: "Settings" },
          { id: "analytics", name: "Analytics" },
        ],
        lastRefresh: "Not refreshed yet",
      };
    },

    computed: {
      currentTabComponent() {
        const componentMap = {
          profile: "ProfileTab",
          settings: "SettingsTab",
          analytics: "AnalyticsTab",
        };

        return componentMap[this.currentTab] || "ProfileTab";
      },

      isCurrentTabCached() {
        // Check if current tab is in the include list
        return ["ProfileTab", "SettingsTab"].includes(this.currentTabComponent);
      },
    },

    methods: {
      refresh() {
        this.lastRefresh = new Date().toLocaleTimeString();
      },
    },
  };
</script>
```

**Max Components Caching Example:**

```html
<template>
  <div class="p-6 bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800 mb-6">
      Memory-Optimized Caching
    </h1>

    <!-- Item selection -->
    <div class="mb-6">
      <label class="block text-sm font-medium text-gray-700 mb-2"
        >Select an item to view:</label
      >
      <div class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 gap-3">
        <button
          v-for="item in items"
          :key="item.id"
          @click="currentItemId = item.id"
          :class="[
            'px-3 py-2 text-sm rounded transition-colors',
            currentItemId === item.id
              ? 'bg-blue-600 text-white'
              : 'bg-gray-100 text-gray-800 hover:bg-gray-200',
          ]"
        >
          Item {{ item.id }}
        </button>
      </div>
    </div>

    <!-- Item details with LRU caching (max 3 components) -->
    <keep-alive :max="3">
      <component
        :is="'ItemDetail'"
        :item-id="currentItemId"
        :key="currentItemId"
        @cache-status="updateCacheStatus"
      ></component>
    </keep-alive>

    <!-- Cache info -->
    <div class="mt-6 bg-gray-50 p-4 rounded">
      <h2 class="text-md font-medium text-gray-700 mb-2">Cache Information</h2>
      <p class="text-sm text-gray-600 mb-2">
        Using LRU (Least Recently Used) caching strategy. Max 3 components in
        cache.
      </p>
      <div class="overflow-x-auto">
        <table class="min-w-full divide-y divide-gray-200">
          <thead class="bg-gray-100">
            <tr>
              <th
                class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
              >
                Item ID
              </th>
              <th
                class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
              >
                Cache Status
              </th>
              <th
                class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
              >
                Last Accessed
              </th>
            </tr>
          </thead>
          <tbody class="bg-white divide-y divide-gray-200">
            <tr v-for="status in cacheStatusList" :key="status.id">
              <td class="px-4 py-2 text-sm text-gray-800">
                Item {{ status.id }}
              </td>
              <td class="px-4 py-2">
                <span
                  :class="[
                    'inline-flex text-xs px-2 py-1 rounded-full',
                    status.cached
                      ? 'bg-green-100 text-green-800'
                      : 'bg-gray-100 text-gray-800',
                  ]"
                >
                  {{ status.cached ? "Cached" : "Not Cached" }}
                </span>
              </td>
              <td class="px-4 py-2 text-sm text-gray-600">
                {{ status.lastAccessed || "Never" }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
  // Item detail component
  const ItemDetail = {
    name: "ItemDetail",

    props: {
      itemId: {
        type: Number,
        required: true,
      },
    },

    data() {
      return {
        loading: true,
        item: null,
      };
    },

    created() {
      this.fetchItemData();
    },

    activated() {
      // Notify parent about activation (for cache tracking)
      this.$emit("cache-status", {
        id: this.itemId,
        cached: true,
        lastAccessed: new Date().toLocaleTimeString(),
      });
    },

    methods: {
      fetchItemData() {
        this.loading = true;

        // Simulate API call
        setTimeout(() => {
          this.item = {
            id: this.itemId,
            title: `Item ${this.itemId}`,
            description: `This is the detailed description for item ${this.itemId}.`,
            createdAt: new Date().toLocaleDateString(),
            stats: {
              views: Math.floor(Math.random() * 1000),
              likes: Math.floor(Math.random() * 100),
              shares: Math.floor(Math.random() * 50),
            },
          };

          this.loading = false;
        }, 500);
      },
    },

    template: `
    <div class="border rounded p-4">
      <div v-if="loading" class="py-12 flex justify-center">
        <svg class="animate-spin h-8 w-8 text-blue-600" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
          <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
          <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
        </svg>
      </div>
      
      <div v-else>
        <h2 class="text-lg font-medium text-gray-800 mb-3">{{ item.title }}</h2>
        <p class="text-gray-600 mb-4">{{ item.description }}</p>
        
        <div class="text-sm text-gray-500 mb-4">Created on {{ item.createdAt }}</div>
        
        <div class="grid grid-cols-3 gap-4">
          <div class="border rounded p-3 bg-gray-50 text-center">
            <div class="text-lg font-bold text-gray-700">{{ item.stats.views }}</div>
            <div class="text-xs text-gray-500">Views</div>
          </div>
          
          <div class="border rounded p-3 bg-gray-50 text-center">
            <div class="text-lg font-bold text-gray-700">{{ item.stats.likes }}</div>
            <div class="text-xs text-gray-500">Likes</div>
          </div>
          
          <div class="border rounded p-3 bg-gray-50 text-center">
            <div class="text-lg font-bold text-gray-700">{{ item.stats.shares }}</div>
            <div class="text-xs text-gray-500">Shares</div>
          </div>
        </div>
      </div>
    </div>
  `,
  };

  export default {
    name: "MemoryOptimizedCaching",

    components: {
      ItemDetail,
    },

    data() {
      return {
        currentItemId: 1,
        items: Array.from({ length: 10 }, (_, i) => ({ id: i + 1 })),
        cacheStatusList: [],
      };
    },

    created() {
      // Initialize cache status list
      this.cacheStatusList = this.items.map((item) => ({
        id: item.id,
        cached: false,
        lastAccessed: null,
      }));
    },

    methods: {
      updateCacheStatus({ id, cached, lastAccessed }) {
        const status = this.cacheStatusList.find((s) => s.id === id);
        if (status) {
          status.cached = cached;
          status.lastAccessed = lastAccessed;
        }
      },
    },
  };
</script>
```

**Common Pitfalls:**

- **Memory leaks**: Keeping too many components alive can lead to memory issues
- **Missing cleanup**: Not handling resource cleanup in deactivated/activated hooks
- **DOM events**: Event listeners persisting when component is cached
- **Network requests**: Not canceling API requests when deactivated
- **Key management**: Forgetting to use keys with dynamic components
- **State confusion**: State unexpectedly persisted between navigation

**Frequently Asked Questions:**

1. **Q: When should I use keep-alive vs not using it?**

   A: Use `<keep-alive>` when:

   - The component is expensive to re-render
   - You need to preserve user input or state when switching views
   - You want to maintain scroll position
   - Component initialization is costly (heavy data loading, complex calculations)

   Don't use `<keep-alive>` when:

   - The component should always show fresh data
   - Memory usage is a concern
   - The component state shouldn't persist
   - Component initialization is simple and fast

2. **Q: How do I know if my application is using too much memory with keep-alive?**

   A: Monitor for these signs:

   - Browser performance degradation over time
   - Increased memory usage in browser dev tools
   - Lag when switching between cached components

   Use these strategies to manage memory:

   - Use the `max` property to limit cached instances
   - Use `include`/`exclude` to be selective about what's cached
   - Implement manual cache invalidation when needed

   ```html
   <!-- Limit to 10 most recently used components -->
   <keep-alive :max="10">
     <component :is="currentView"></component>
   </keep-alive>
   ```

3. **Q: How do I clean up resources when a component is deactivated?**

   A: Use the activated/deactivated lifecycle hooks:

   ```js
   export default {
     data() {
       return {
         intervalId: null,
         dataSubscription: null,
       };
     },

     activated() {
       // Restart timers, subscribe to events
       this.intervalId = setInterval(this.refreshData, 30000);
       this.dataSubscription = this.$store.subscribe(this.handleStoreChanges);
     },

     deactivated() {
       // Clear timers, unsubscribe from events
       clearInterval(this.intervalId);
       this.dataSubscription(); // Assuming this unsubscribes
     },
   };
   ```

   For asynchronous operations:

   ```js
   export default {
     data() {
       return {
         abortController: null,
       };
     },

     activated() {
       // Create new controller for this activation
       this.abortController = new AbortController();
       this.fetchData();
     },

     deactivated() {
       // Cancel pending requests
       if (this.abortController) {
         this.abortController.abort();
       }
     },

     methods: {
       async fetchData() {
         try {
           const response = await fetch("/api/data", {
             signal: this.abortController.signal,
           });
           // Process data
         } catch (error) {
           if (error.name !== "AbortError") {
             console.error("Fetch error:", error);
           }
         }
       },
     },
   };
   ```

### 6. Component Inheritance

**Concise Explanation:**
Component inheritance in Vue allows you to extend components, inheriting their options and overriding or extending specific behaviors. While Vue encourages composition over inheritance, understanding inheritance patterns helps with creating consistent component systems and reusing functionality.

**Where to Use:**

- Creating base components that get extended with specific variations
- Form control extensions (base input → specialized inputs)
- Layout components with common functionality
- Consistent UI components with shared behavior
- Progressive enhancement of third-party components

**Code Snippet:**

```html
<!-- BaseButton.vue (Parent Component) -->
<template>
  <button
    :type="type"
    :disabled="disabled || loading"
    :class="[
      'px-4 py-2 rounded transition-colors focus:outline-none focus:ring-2',
      variantClasses,
      sizeClasses,
      disabled ? 'opacity-50 cursor-not-allowed' : '',
      className,
    ]"
    @click="handleClick"
  >
    <!-- Loading spinner -->
    <svg
      v-if="loading"
      class="animate-spin -ml-1 mr-2 h-4 w-4 inline-block"
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

    <!-- Button content -->
    <slot>{{ label }}</slot>
  </button>
</template>

<script>
  export default {
    name: "BaseButton",

    props: {
      type: {
        type: String,
        default: "button",
      },
      label: {
        type: String,
        default: "Button",
      },
      variant: {
        type: String,
        default: "primary",
        validator: (value) =>
          [
            "primary",
            "secondary",
            "success",
            "danger",
            "warning",
            "info",
          ].includes(value),
      },
      size: {
        type: String,
        default: "md",
        validator: (value) => ["sm", "md", "lg"].includes(value),
      },
      disabled: {
        type: Boolean,
        default: false,
      },
      loading: {
        type: Boolean,
        default: false,
      },
      className: {
        type: String,
        default: "",
      },
    },

    computed: {
      variantClasses() {
        const variants = {
          primary:
            "bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500",
          secondary:
            "bg-gray-600 text-white hover:bg-gray-700 focus:ring-gray-500",
          success:
            "bg-green-600 text-white hover:bg-green-700 focus:ring-green-500",
          danger: "bg-red-600 text-white hover:bg-red-700 focus:ring-red-500",
          warning:
            "bg-yellow-500 text-white hover:bg-yellow-600 focus:ring-yellow-500",
          info: "bg-blue-500 text-white hover:bg-blue-600 focus:ring-blue-400",
        };

        return variants[this.variant] || variants.primary;
      },

      sizeClasses() {
        const sizes = {
          sm: "text-sm px-3 py-1",
          md: "text-base px-4 py-2",
          lg: "text-lg px-6 py-3",
        };

        return sizes[this.size] || sizes.md;
      },
    },

    methods: {
      handleClick(event) {
        if (this.disabled || this.loading) {
          event.preventDefault();
          return;
        }

        this.$emit("click", event);
      },
    },
  };
</script>
```

```html
<!-- IconButton.vue (Child Component extending BaseButton) -->
<template>
  <button
    :type="type"
    :disabled="disabled || loading"
    :class="[
      'rounded transition-colors focus:outline-none focus:ring-2 inline-flex items-center justify-center',
      variantClasses,
      sizeClasses,
      roundedClasses,
      disabled ? 'opacity-50 cursor-not-allowed' : '',
      className,
    ]"
    @click="handleClick"
  >
    <!-- Loading spinner -->
    <svg
      v-if="loading"
      class="animate-spin h-4 w-4"
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

    <!-- Icon -->
    <slot v-else name="icon">
      <svg
        v-if="icon"
        class="h-5 w-5"
        fill="none"
        viewBox="0 0 24 24"
        stroke="currentColor"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          stroke-width="2"
          :d="iconPath"
        />
      </svg>
    </slot>

    <!-- Label (only for non-circle buttons) -->
    <span v-if="!circle && !loading && label" class="ml-2">{{ label }}</span>
  </button>
</template>

<script>
  import BaseButton from "./BaseButton.vue";

  export default {
    name: "IconButton",

    extends: BaseButton,

    props: {
      icon: {
        type: String,
        default: "",
      },
      circle: {
        type: Boolean,
        default: false,
      },
      rounded: {
        type: String,
        default: "default",
        validator: (value) => ["default", "full", "none"].includes(value),
      },
    },

    computed: {
      // Override and extend parent's computed properties
      sizeClasses() {
        if (this.circle) {
          const sizes = {
            sm: "h-8 w-8",
            md: "h-10 w-10",
            lg: "h-12 w-12",
          };

          return sizes[this.size] || sizes.md;
        }

        // Use parent's size classes for non-circle buttons
        return BaseButton.computed.sizeClasses.call(this);
      },

      // Add new computed property
      roundedClasses() {
        if (this.circle) {
          return "rounded-full";
        }

        const roundedOptions = {
          default: "rounded",
          full: "rounded-full",
          none: "rounded-none",
        };

        return roundedOptions[this.rounded] || "rounded";
      },

      // Calculate icon path based on icon name
      iconPath() {
        const iconPaths = {
          plus: "M12 4v16m8-8H4",
          minus: "M20 12H4",
          check: "M5 13l4 4L19 7",
          x: "M6 18L18 6M6 6l12 12",
          trash:
            "M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16",
          edit: "M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z",
        };

        return iconPaths[this.icon] || iconPaths.plus;
      },
    },
  };
</script>
```

```html
<!-- Usage Example -->
<template>
  <div class="p-6 bg-white rounded-xl shadow-md">
    <h1 class="text-xl font-medium text-gray-800 mb-6">
      Component Inheritance Demo
    </h1>

    <div class="mb-8">
      <h2 class="text-lg font-medium text-gray-700 mb-3">Base Button</h2>
      <div class="flex flex-wrap gap-3">
        <base-button>Default</base-button>
        <base-button variant="secondary">Secondary</base-button>
        <base-button variant="success">Success</base-button>
        <base-button variant="danger">Danger</base-button>
        <base-button size="sm">Small</base-button>
        <base-button size="lg">Large</base-button>
        <base-button :loading="true">Loading</base-button>
        <base-button disabled>Disabled</base-button>
      </div>
    </div>

    <div class="mb-8">
      <h2 class="text-lg font-medium text-gray-700 mb-3">Icon Buttons</h2>
      <div class="flex flex-wrap gap-3">
        <icon-button icon="plus">Add Item</icon-button>
        <icon-button icon="edit" variant="secondary">Edit</icon-button>
        <icon-button icon="check" variant="success">Approve</icon-button>
        <icon-button icon="trash" variant="danger">Delete</icon-button>
      </div>
    </div>

    <div class="mb-8">
      <h2 class="text-lg font-medium text-gray-700 mb-3">
        Circle Icon Buttons
      </h2>
      <div class="flex flex-wrap gap-3">
        <icon-button icon="plus" :circle="true"></icon-button>
        <icon-button
          icon="edit"
          variant="secondary"
          :circle="true"
        ></icon-button>
        <icon-button
          icon="check"
          variant="success"
          :circle="true"
        ></icon-button>
        <icon-button icon="trash" variant="danger" :circle="true"></icon-button>
        <icon-button
          icon="x"
          variant="warning"
          :circle="true"
          size="sm"
        ></icon-button>
        <icon-button
          icon="plus"
          variant="info"
          :circle="true"
          size="lg"
        ></icon-button>
      </div>
    </div>

    <div>
      <h2 class="text-lg font-medium text-gray-700 mb-3">
        Custom Icons with Slots
      </h2>
      <div class="flex flex-wrap gap-3">
        <icon-button variant="primary">
          <template #icon>
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
                d="M3 8l7.89 5.26a2 2 0 002.22 0L21 8M5 19h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z"
              />
            </svg>
          </template>
          Send Email
        </icon-button>

        <icon-button variant="secondary" :circle="true">
          <template #icon>
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
                d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z"
              />
            </svg>
          </template>
        </icon-button>
      </div>
    </div>
  </div>
</template>

<script>
  import BaseButton from "./BaseButton.vue";
  import IconButton from "./IconButton.vue";

  export default {
    name: "ComponentInheritanceDemo",

    components: {
      BaseButton,
      IconButton,
    },
  };
</script>
```

**Inheritance with Mixins Example:**

```html
<!-- BaseFormField.vue -->
<script>
  export default {
    name: "BaseFormField",

    props: {
      label: {
        type: String,
        default: "",
      },
      name: {
        type: String,
        required: true,
      },
      value: {
        type: [String, Number, Boolean, Array],
        default: "",
      },
      disabled: {
        type: Boolean,
        default: false,
      },
      readonly: {
        type: Boolean,
        default: false,
      },
      required: {
        type: Boolean,
        default: false,
      },
      error: {
        type: String,
        default: "",
      },
      helpText: {
        type: String,
        default: "",
      },
    },

    methods: {
      updateValue(value) {
        this.$emit("input", value);
      },
    },
  };
</script>
```

```html
<!-- TextInput.vue -->
<template>
  <div class="mb-4">
    <label
      v-if="label"
      :for="name"
      class="block text-sm font-medium text-gray-700 mb-1"
    >
      {{ label }}
      <span v-if="required" class="text-red-600">*</span>
    </label>

    <div class="relative">
      <div
        v-if="prefixText || $slots.prefix"
        class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none"
      >
        <span v-if="prefixText" class="text-gray-500 sm:text-sm"
          >{{ prefixText }}</span
        >
        <slot v-else name="prefix"></slot>
      </div>

      <input
        :id="name"
        :name="name"
        :type="type"
        :value="value"
        :disabled="disabled"
        :readonly="readonly"
        :required="required"
        :placeholder="placeholder"
        :maxlength="maxlength"
        :min="min"
        :max="max"
        :step="step"
        :autocomplete="autocomplete"
        @input="updateValue($event.target.value)"
        :class="[
          'block w-full rounded-md shadow-sm focus:ring-2 focus:ring-blue-500 focus:border-blue-500 sm:text-sm border',
          error ? 'border-red-300' : 'border-gray-300',
          prefixText || $slots.prefix ? 'pl-10' : '',
          suffixText || $slots.suffix ? 'pr-10' : '',
          disabled ? 'bg-gray-100 text-gray-500 cursor-not-allowed' : '',
        ]"
      />

      <div
        v-if="suffixText || $slots.suffix"
        class="absolute inset-y-0 right-0 pr-3 flex items-center pointer-events-none"
      >
        <span v-if="suffixText" class="text-gray-500 sm:text-sm"
          >{{ suffixText }}</span
        >
        <slot v-else name="suffix"></slot>
      </div>
    </div>

    <p v-if="error" class="mt-1 text-sm text-red-600">{{ error }}</p>
    <p v-else-if="helpText" class="mt-1 text-sm text-gray-500">
      {{ helpText }}
    </p>
  </div>
</template>

<script>
  import BaseFormField from "./BaseFormField.vue";

  export default {
    name: "TextInput",

    extends: BaseFormField,

    props: {
      type: {
        type: String,
        default: "text",
        validator: (value) =>
          [
            "text",
            "email",
            "password",
            "number",
            "tel",
            "url",
            "search",
            "date",
            "time",
            "datetime-local",
          ].includes(value),
      },
      placeholder: {
        type: String,
        default: "",
      },
      maxlength: {
        type: [Number, String],
        default: null,
      },
      min: {
        type: [Number, String],
        default: null,
      },
      max: {
        type: [Number, String],
        default: null,
      },
      step: {
        type: [Number, String],
        default: null,
      },
      autocomplete: {
        type: String,
        default: "off",
      },
      prefixText: {
        type: String,
        default: "",
      },
      suffixText: {
        type: String,
        default: "",
      },
    },
  };
</script>
```

**Mixin vs. Extends Pattern:**

```js
// formFieldMixin.js
export const formFieldMixin = {
  props: {
    label: {
      type: String,
      default: "",
    },
    name: {
      type: String,
      required: true,
    },
    value: {
      type: [String, Number, Boolean, Array],
      default: "",
    },
    error: {
      type: String,
      default: "",
    },
  },

  methods: {
    updateValue(value) {
      this.$emit("input", value);
    },
  },
};
```

```html
<!-- Using the mixin approach -->
<template>
  <div class="mb-4">
    <label
      v-if="label"
      :for="name"
      class="block text-sm font-medium text-gray-700 mb-1"
    >
      {{ label }}
    </label>

    <select
      :id="name"
      :name="name"
      :value="value"
      @change="updateValue($event.target.value)"
      class="block w-full rounded-md border-gray-300 shadow-sm focus:ring-2 focus:ring-blue-500 focus:border-blue-500 sm:text-sm"
    >
      <option v-if="placeholder" value="" disabled>{{ placeholder }}</option>
      <option
        v-for="option in options"
        :key="option.value"
        :value="option.value"
      >
        {{ option.label }}
      </option>
    </select>

    <p v-if="error" class="mt-1 text-sm text-red-600">{{ error }}</p>
  </div>
</template>

<script>
  import { formFieldMixin } from "@/mixins/formFieldMixin";

  export default {
    name: "SelectInput",

    mixins: [formFieldMixin],

    props: {
      options: {
        type: Array,
        required: true,
      },
      placeholder: {
        type: String,
        default: "Select an option",
      },
    },
  };
</script>
```

**Common Pitfalls:**

- **Deep inheritance chains**: Making maintenance and debugging difficult
- **Tight coupling**: Creating components that are too interdependent
- **Namespace collisions**: Naming conflicts between parent and child components
- **Implicit dependencies**: Child components assuming specific parent behavior
- **Overriding complexity**: Confusion about which methods or computed properties are being used

**Frequently Asked Questions:**

1. **Q: When should I use inheritance vs. composition?**

   A: The general guideline is "favor composition over inheritance", but each has its place:

   Use **inheritance** when:

   - You're creating specialized versions of a component
   - The child is truly a "type of" the parent
   - The relationship is clear and unlikely to change
   - You need to override specific behaviors

   Use **composition** when:

   - You're combining multiple behaviors
   - The relationship is "has a" rather than "is a"
   - The components may be used independently
   - You want more flexibility and less coupling

   Example of composition:

   ```html
   <template>
     <div>
       <label-component :label="label" :required="required" />
       <input-component v-model="localValue" :disabled="disabled" />
       <error-component :error="error" />
     </div>
   </template>
   ```

2. **Q: What's the difference between extends and mixins?**

   A: Both allow you to reuse component logic, but with different priorities:

   **extends**:

   - Establishes a parent-child inheritance relationship
   - Child extends a single parent component
   - Options from the parent are merged into the child
   - Child options take precedence in conflicts
   - Better for "is-a" relationships

   **mixins**:

   - Adds reusable functionality to components
   - A component can use multiple mixins
   - Options from mixins are merged into the component
   - Component options take precedence in conflicts
   - Better for sharing behavior across different components

3. **Q: How do I handle lifecycle hooks in inherited components?**

   A: Lifecycle hooks are merged rather than overridden:

   - All hooks from parent/mixins are called first, then the component's hook
   - For example, if both parent and child have a `mounted` hook, the parent's runs first, then the child's

   Example:

   ```js
   // Parent component
   export default {
     mounted() {
       console.log('Parent mounted')
     }
   }

   // Child component
   export default {
     extends: Parent,
     mounted() {
       console.log('Child mounted')
       // Output: "Parent mounted" then "Child mounted"
     }
   }
   ```

   If you need to run code in a specific order or conditionally call the parent's functionality:

   ```js
   // Using super-like pattern with mixins
   const ParentMethods = {
     methods: {
       initialize() {
         console.log("Parent initialize");
       },
     },
   };

   export default {
     mixins: [ParentMethods],
     methods: {
       initialize() {
         // Call parent method first
         ParentMethods.methods.initialize.call(this);

         // Then do child-specific initialization
         console.log("Child initialize");
       },
     },
   };
   ```

## Key Takeaways

1. **Custom Directives**: Provide direct DOM manipulation capabilities for specialized behaviors and UI interactions that are reusable across components.

2. **Filters (Legacy)**: While deprecated in Vue 3, understanding filters helps maintain Vue 2 applications and properly migrate to computed properties or methods.

3. **Render Functions**: Give you low-level control over component output using JavaScript instead of templates, essential for creating highly dynamic components.

4. **Transitions and Animations**: Vue's built-in transition system makes animating element appearance/disappearance and list changes straightforward and declarative.

5. **Keep-alive Components**: Preserve component state between renders, optimizing performance and user experience for toggled components or tabbed interfaces.

6. **Component Inheritance**: Allows extending base components to create specialized versions, though Vue generally recommends composition over inheritance for most cases.

By mastering these advanced Options API features, you'll have the tools to solve complex UI challenges and build sophisticated Vue applications.
