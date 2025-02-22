<template>
  <div class="formatter-container">
    <h2>JSON Formatter</h2>

    <!-- Input Textarea -->
    <textarea
        v-model="jsonInput"
        placeholder="Paste JSON here..."
        class="input-box">
    </textarea>

    <!-- Formatting Options -->
    <div class="options">
      <label>Indentation:</label>
      <select v-model="indentation">
        <option :value="2">2 Spaces</option>
        <option :value="4">4 Spaces</option>
      </select>
      <button @click="formatJson">Format</button>
    </div>

    <!-- Output -->
    <pre class="output-box" v-if="formattedJson">{{ formattedJson }}</pre>
    <p class="error-message" v-if="errorMessage">{{ errorMessage }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      jsonInput: "",
      formattedJson: "",
      errorMessage: "",
      indentation: 2 // Default indentation
    };
  },
  methods: {
    formatJson() {
      try {
        this.formattedJson = JSON.stringify(JSON.parse(this.jsonInput), null, this.indentation);
        this.errorMessage = ""; // Clear errors if successful
      } catch (error) {
        this.errorMessage = "Invalid JSON format.";
        this.formattedJson = "";
      }
    }
  }
};
</script>

<style>
.formatter-container {
  max-width: 600px;
  margin: 20px auto;
  text-align: center;
}

.input-box {
  width: 100%;
  height: 150px;
  padding: 10px;
  font-family: monospace;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.options {
  margin: 10px 0;
}

.output-box {
  background: #f4f4f4;
  border: 1px solid #ddd;
  padding: 10px;
  white-space: pre-wrap;
  text-align: left;
}

.error-message {
  color: red;
  font-weight: bold;
}
</style>
