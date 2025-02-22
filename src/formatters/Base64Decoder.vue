<template>
  <div class="base64-decoder">
    <h2>Base64 Decoder</h2>
    <textarea
        v-model="base64Input"
        placeholder="Enter Base64 encoded text..."
        class="input-box"
    ></textarea>
    <button @click="decodeBase64" class="decode-btn">Decode</button>
    <div v-if="decodedOutput" class="output-box">
      <h3>Decoded Output:</h3>
      <pre>{{ decodedOutput }}</pre>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      base64Input: "",
      decodedOutput: "",
    };
  },
  methods: {
    decodeBase64() {
      try {
        this.decodedOutput = atob(this.base64Input); // Decode Base64
      } catch (e) {
        this.decodedOutput = "Invalid Base64 string";
      }
    },
  },
};
</script>

<style>
.base64-decoder {
  text-align: center;
  max-width: 500px;
  margin: auto;
}

.input-box {
  width: 100%;
  height: 100px;
  padding: 10px;
  font-size: 16px;
}

.decode-btn {
  margin-top: 10px;
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
}

.output-box {
  margin-top: 20px;
  background: #f4f4f4;
  padding: 10px;
  border-radius: 5px;
}
</style>
