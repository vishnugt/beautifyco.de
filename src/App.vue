<script>

import JsonFormatter from "./formatters/JsonFormatter.vue";
import Base64Decoder from "./formatters/Base64Decoder.vue";
import Base64Encoder from "./formatters/Base64Encoder.vue";

const app = {
  components: {
    JsonFormatter,
    Base64Decoder,
    Base64Encoder
  },
  data() {
    return {
      isFocused: true,
      selectedFormatter: null,
      searchQuery: "",
      scripts: [{
        id: "Base64Decoder",
        name: "Base 64 Decoder"
      },
        {
          id: "Base64Encoder",
          name: "Base 64 Encoder"
        },
        {
          id: "JsonFormatter",
          name: "JSON Formatter"
        },
        {
          id: "jwtDecoder",
          name: "JWT Decoder"
        }]
    }
  },
  computed: {
    filteredScripts() {
      return this.scripts.filter(script => {
        return script.id.toLowerCase().includes(this.searchQuery.toLowerCase());
      });
    }
  },
  mounted() {
    window.addEventListener("keydown", this.handleShortcut);
  },
  beforeUnmount() {
    window.removeEventListener("keydown", this.handleShortcut);
  },
  watch: {
    watchFocus(value) {
      if (value) {
        this.$refs.searchInput.focus();
      } else {
        this.$refs.searchInput.blur();
      }
    }
  },
  methods: {
    handleShortcut(event) {
      if (event.ctrlKey && event.code === "Space") {
        event.preventDefault(); // Prevent default browser behavior
        this.isFocused = !this.isFocused;
      }
    },
    processFirstResult() {
      if (this.filteredScripts.length > 0) {
        this.selectScript(this.filteredScripts[0]); // Select the first script
      }
    },
    selectScript(script) {
      this.selectedFormatter = script.id
      this.$refs.searchInput.blur();
      this.searchQuery = "";
      this.isFocused = false;
    },
  }
}
export default app;
</script>

<template>
  <div class="search-container">
    <input ref="searchInput" v-model="searchQuery" @focus="isFocused = true"
           placeholder="Press Ctrl + Space to start" class="search-box"
           @keydown.enter="processFirstResult"/>
    <ul v-if="isFocused && filteredScripts.length" class="results-list">
      <li v-for="script in filteredScripts" :key="script.id" class="result-item"
          @click="selectScript(script)">
        {{ script.name }}
      </li>
    </ul>
  </div>
  <div>
    <!-- Dynamically render the selected formatter -->
    <component :is="selectedFormatter" v-if="selectedFormatter"></component>
  </div>

</template>

<style>
.search-container {
  position: relative;
  left: 20%;
  width: 60%;
  text-align: center;
}

/* Style the input box */
.search-box {
  width: 100%;
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 5px;
  outline: none;
  box-sizing: border-box;
}

/* Style the results list */
.results-list {
  margin-top: 10px;
  list-style: none;
  padding: 10px;
  background: #ffffff; /* White background for a clean look */
  border: 1px solid #d1d5db; /* Light gray border */
  border-radius: 8px;
  width: 100%;
  overflow-y: auto;
  box-sizing: border-box;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1); /* Subtle shadow */
}

/* Style each list item */
.results-list li {
  padding: 12px;
  cursor: pointer;
  transition: background 0.2s, transform 0.1s;
  width: 100%;
  border-bottom: 1px solid #e5e7eb; /* Light separator */
  font-size: 16px;
  color: #333; /* Darker text for readability */
}

/* Last item should not have a border at the bottom */
.results-list li:last-child {
  border-bottom: none;
}

/* Hover effect: Darker background */
.results-list li:hover {
  background: #f3f4f6; /* Light gray */
  transform: scale(1.02); /* Slightly enlarges row */
}

/* Active row styling */
.results-list li:active,
.results-list li.selected {
  background: #2563eb; /* Blue highlight */
  color: white; /* White text */
  font-weight: bold;
}

</style>

