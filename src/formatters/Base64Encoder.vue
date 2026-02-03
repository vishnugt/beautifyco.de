<script>
export default {
  data() {
    return {
      inputText: "",
      encodedText: "",
      errorMessage: ""
    };
  },
  methods: {
    encodeBase64() {
      try {
        if (!this.inputText.trim()) {
          this.errorMessage = "Please enter some text to encode.";
          this.encodedText = "";
          return;
        }
        this.encodedText = btoa(this.inputText);
        this.errorMessage = "";
      } catch (error) {
        this.errorMessage = "Error encoding text.";
        this.encodedText = "";
      }
    },
    copyToClipboard() {
      navigator.clipboard.writeText(this.encodedText).then(() => {
        // Could add toast notification
      });
    },
    clearAll() {
      this.inputText = "";
      this.encodedText = "";
      this.errorMessage = "";
    },
    handleInputKeydown(event) {
      if (event.ctrlKey && event.key === 'Enter') {
        event.preventDefault();
        this.encodeBase64();
      }
    }
  }
};
</script>

<template>
  <div class="encoder-container">

    <!-- Side by Side Layout -->
    <div class="side-by-side-layout">
      <!-- Input Side -->
      <div class="input-side">
        <textarea
            v-model="inputText"
            placeholder="Enter text to encode... (Ctrl+Enter to encode)"
            class="input-box"
            @keydown="handleInputKeydown">
        </textarea>
      </div>

      <!-- Output Side -->
      <div class="output-side" v-if="encodedText && !errorMessage">
        <textarea
            v-model="encodedText"
            placeholder="Encoded output will appear here..."
            class="output-box">
        </textarea>
      </div>
      
      <!-- Empty State -->
      <div class="empty-output empty-state" style="margin-top: 0" v-else>
        <p>Your encoded text will appear here</p>
        <p class="hint">Press Ctrl+Enter to encode</p>
      </div>
    </div>
    
    <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>
  </div>
</template>

<style>
.encoder-container {
  max-width: 1300px;
  margin: 0 auto;
  padding: 12px;
  height: calc(100vh - 80px);
  display: flex;
  flex-direction: column;
  position: relative;
  box-sizing: border-box;
  overflow: hidden;
}

/* Responsive layout for larger displays */
@media (min-width: 1800px) {
  .encoder-container {
    max-width: 1800px;
  }
}

@media (min-width: 2200px) {
  .encoder-container {
    max-width: 2000px;
  }
}


.controls {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 16px;
  margin-bottom: 12px;
  flex-wrap: wrap;
  padding: 12px;
  background: #f1f5f9;
  border-radius: 8px;
  border: 1px solid #e2e8f0;
}

.side-by-side-layout {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
  flex: 1;
  min-height: 0;
  overflow: hidden;
}

.input-side,
.output-side,
.empty-output {
  flex: 1;
  width: 100%;
  min-height: 0;
}


.input-box {
  flex: 1;
  width: 100%;
  padding: 12px;
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
  font-size: 12px;
  border: 1px solid #e2e8f0;
  border-radius: 6px;
  resize: none;
  transition: border-color 0.2s ease;
  background: #ffffff;
  line-height: 1.3;
  min-height: 0;
  box-sizing: border-box;
}

.input-box:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

.output-box {
  flex: 1;
  width: 100%;
  background: #f8fafc;
  border: 1px solid #e2e8f0;
  border-radius: 6px;
  padding: 12px;
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
  font-size: 12px;
  line-height: 1.3;
  min-height: 0;
  resize: none;
  color: #1e293b;
  transition: border-color 0.2s ease;
  box-sizing: border-box;
}

.output-box:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

.empty-state {
  flex: 1;
  width: 100%;
  background: #f1f5f9;
  border: 1px dashed #d1d5db;
  border-radius: 6px;
  padding: 12px;
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
  font-size: 12px;
  line-height: 1.3;
  min-height: 0;
  resize: none;
  color: #64748b;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  font-style: italic;
}

.empty-state p {
  margin: 4px 0;
  font-size: 13px;
}

.empty-state .hint {
  font-size: 11px;
  color: #64748b;
}

.view-options {
  display: flex;
  gap: 8px;
}

.encode-btn {
  background: #3b82f6;
  color: white;
  border: none;
  padding: 6px 14px;
  border-radius: 6px;
  font-weight: 500;
  font-size: 12px;
  cursor: pointer;
  transition: all 0.2s ease;
  box-shadow: 0 1px 3px rgba(79, 70, 229, 0.2);
}

.encode-btn:hover {
  background: #4338ca;
  transform: translateY(-1px);
  box-shadow: 0 2px 6px rgba(79, 70, 229, 0.3);
}

.action-btn {
  background: #f3f4f6;
  color: #1e293b;
  border: 1px solid #d1d5db;
  padding: 5px 10px;
  border-radius: 4px;
  font-size: 11px;
  cursor: pointer;
  transition: all 0.2s ease;
  font-weight: 500;
}

.action-btn:hover {
  background: #e5e7eb;
  border-color: #9ca3af;
}

.error-message {
  color: #dc2626;
  font-weight: 500;
  margin-top: 10px;
  padding: 8px 12px;
  background: #fef2f2;
  border: 1px solid #fecaca;
  border-radius: 6px;
  text-align: center;
}

@media (max-width: 1024px) {
  .side-by-side-layout {
    grid-template-columns: 1fr;
  }
  
  .controls {
    flex-direction: column;
    gap: 12px;
  }
}
</style>
