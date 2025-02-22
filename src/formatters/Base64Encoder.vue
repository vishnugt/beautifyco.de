<script>
export default {
  data() {
    return {
      inputText: "",
      encodedText: ""
    };
  },
  methods: {
    encodeBase64() {
      this.encodedText = btoa(this.inputText);
    }
  }
};
</script>

<template>
  <div class="encoder-container">
    <h2>Base64 Encoder</h2>
    <textarea v-model="inputText" placeholder="Enter text to encode"></textarea>
    <button @click="encodeBase64">Encode</button>
    <textarea v-model="encodedText" readonly placeholder="Encoded output"></textarea>
  </div>
</template>

<style>
.encoder-container {
  display: flex;
  flex-direction: column;
  gap: 10px;
  max-width: 400px;
  margin: auto;
}

textarea {
  width: 100%;
  height: 80px;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 5px;
  resize: none;
}

button {
  padding: 10px;
  border: none;
  background: #007bff;
  color: white;
  border-radius: 5px;
  cursor: pointer;
}

button:hover {
  background: #0056b3;
}
</style>
