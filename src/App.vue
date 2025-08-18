<script>

import JsonFormatter from "./formatters/JsonFormatter.vue";
import Base64Decoder from "./formatters/Base64Decoder.vue";
import Base64Encoder from "./formatters/Base64Encoder.vue";
import JwtDecoder from "./formatters/JwtDecoder.vue";
import UuidGenerator from "./formatters/UuidGenerator.vue";

const app = {
  components: {
    JsonFormatter,
    Base64Decoder,
    Base64Encoder,
    JwtDecoder,
    UuidGenerator
  },
  data() {
    return {
      isFocused: false,
      selectedFormatter: "JsonFormatter",
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
          id: "JwtDecoder",
          name: "JWT Decoder"
        },
        {
          id: "UuidGenerator",
          name: "UUID Generator"
        }]
    }
  },
  computed: {
    filteredScripts() {
      return this.scripts
        .map((script, index) => ({ ...script, originalIndex: index }))
        .filter((script) => {
          const searchTerm = this.searchQuery.toLowerCase();
          const scriptNumber = (script.originalIndex + 1).toString();
          return script.id.toLowerCase().includes(searchTerm) || 
                 script.name.toLowerCase().includes(searchTerm) ||
                 scriptNumber.includes(searchTerm);
        });
    },
    currentScriptName() {
      const currentScript = this.scripts.find(script => script.id === this.selectedFormatter);
      return currentScript ? currentScript.name : "Select Tool";
    },
    placeholderText() {
      return `${this.currentScriptName} (ctrl + space for other options)`;
    }
  },
  mounted() {
    window.addEventListener("keydown", this.handleShortcut);
  },
  beforeUnmount() {
    window.removeEventListener("keydown", this.handleShortcut);
  },
  watch: {
    isFocused(value) {
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
      this.selectedFormatter = script.id;
      this.searchQuery = "";
      this.isFocused = false;
    },
  }
}
export default app;
</script>

<template>
  <div class="search-container">
    <input ref="searchInput" v-model="searchQuery" @focus="isFocused = true" @blur="isFocused = false"
           :placeholder="placeholderText" class="search-box"
           @keydown.enter="processFirstResult"/>
    <ul v-if="isFocused && filteredScripts.length" class="results-list">
      <li v-for="script in filteredScripts" :key="script.id" class="result-item"
          @mousedown="selectScript(script)">
        {{ script.originalIndex + 1 }}. {{ script.name }}
      </li>
    </ul>
  </div>
  <div>
    <!-- Dynamically render the selected formatter -->
    <component :is="selectedFormatter" v-if="selectedFormatter"></component>
  </div>

</template>

<style>
* {
  box-sizing: border-box;
}

html, body {
  margin: 0;
  padding: 0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen', 'Ubuntu', 'Cantarell', sans-serif;
  background: #fafafa;
  color: #374151;
  height: 100vh;
  overflow: hidden;
}

.search-container {
  position: relative;
  max-width: 800px;
  margin: 20px auto;
  padding: 0 24px;
  text-align: center;
}

.search-box {
  width: 100%;
  padding: 12px 16px;
  font-size: 15px;
  border: 2px solid #e1e5e9;
  border-radius: 10px;
  outline: none;
  box-sizing: border-box;
  transition: all 0.2s ease;
  background: #ffffff;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.search-box:focus {
  border-color: #4f46e5;
  box-shadow: 0 0 0 4px rgba(79, 70, 229, 0.1), 0 4px 12px rgba(0, 0, 0, 0.1);
}

.search-box::placeholder {
  color: #9ca3af;
  font-weight: 400;
}

.results-list {
  position: absolute;
  top: 100%;
  margin-top: 8px;
  list-style: none;
  padding: 8px;
  background: #ffffff;
  border: 1px solid #e1e5e9;
  border-radius: 8px;
  width: 094%;
  max-height: 280px;
  overflow-y: auto;
  box-sizing: border-box;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
  z-index: 100;
}

.results-list li {
  text-align: left;
  padding: 12px 16px;
  cursor: pointer;
  transition: all 0.2s ease;
  border-radius: 8px;
  margin-bottom: 4px;
  font-size: 14px;
  font-weight: 500;
  color: #374151;
}

.results-list li:last-child {
  margin-bottom: 0;
}

.results-list li:hover {
  background: #f3f4f6;
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.results-list li:active {
  background: #4f46e5;
  color: white;
  transform: translateY(0);
}

@media (max-width: 768px) {
  .search-container {
    margin: 20px auto;
    padding: 0 16px;
  }
  
  .search-box {
    padding: 10px 14px;
    font-size: 14px;
  }
  
  .results-list {
    max-height: 240px;
  }
}
</style>

