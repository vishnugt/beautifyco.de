<template>
  <div class="jwt-decoder-container">

    <!-- Side by Side Layout -->
    <div class="side-by-side-layout">
      <!-- Input Side -->
      <div class="input-side">
        <textarea
            v-model="jwtInput"
            placeholder="Enter JWT token here... (Ctrl+Enter to decode)"
            class="input-box"
            @keydown="handleInputKeydown"
        ></textarea>
      </div>

      <!-- Output Side -->
      <div class="output-side" v-if="decodedHeader || decodedPayload">
        <div class="output-section">
          <div v-if="decodedPayload" class="output-container">
            <h4>Payload</h4>
            <div class="json-viewer">
              <pre>{{ formatJson(decodedPayload) }}</pre>
            </div>
          </div>
          
          <div v-if="signature" class="output-container">
            <h4>Signature</h4>
            <div class="signature-box">
              <code>{{ signature }}</code>
            </div>
          </div>
        </div>
      </div>
      
      <!-- Empty State -->
      <div class="empty-output empty-state" style="margin-top: 0" v-else>
        <p>Your decoded JWT will appear here</p>
        <p class="hint">Press Ctrl+Enter to decode</p>
      </div>
    </div>
    
    <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      jwtInput: "",
      decodedHeader: null,
      decodedPayload: null,
      signature: "",
      errorMessage: ""
    };
  },
  methods: {
    decodeJwt() {
      try {
        if (!this.jwtInput.trim()) {
          this.errorMessage = "Please enter a JWT token to decode.";
          this.clearOutput();
          return;
        }
        
        const token = this.jwtInput.trim();
        const parts = token.split('.');
        
        if (parts.length !== 3) {
          this.errorMessage = "Invalid JWT format. JWT should have 3 parts separated by dots.";
          this.clearOutput();
          return;
        }
        
        // Decode header
        try {
          const headerDecoded = this.base64UrlDecode(parts[0]);
          this.decodedHeader = JSON.parse(headerDecoded);
        } catch (e) {
          this.errorMessage = "Invalid JWT header.";
          this.clearOutput();
          return;
        }
        
        // Decode payload
        try {
          const payloadDecoded = this.base64UrlDecode(parts[1]);
          this.decodedPayload = JSON.parse(payloadDecoded);
        } catch (e) {
          this.errorMessage = "Invalid JWT payload.";
          this.clearOutput();
          return;
        }
        
        // Store signature (don't decode it as it's binary)
        this.signature = parts[2];
        this.errorMessage = "";
        
      } catch (error) {
        this.errorMessage = "Error decoding JWT token.";
        this.clearOutput();
      }
    },
    
    base64UrlDecode(str) {
      // Add padding if needed
      const padded = str + '='.repeat((4 - str.length % 4) % 4);
      // Replace URL-safe characters
      const base64 = padded.replace(/-/g, '+').replace(/_/g, '/');
      return atob(base64);
    },
    
    formatJson(obj) {
      return JSON.stringify(obj, null, 2);
    },
    
    copyToClipboard() {
      const payload = JSON.stringify(this.decodedPayload, null, 2);
      navigator.clipboard.writeText(payload).then(() => {
        // Could add toast notification
      });
    },
    
    clearAll() {
      this.jwtInput = "";
      this.clearOutput();
      this.errorMessage = "";
    },
    
    clearOutput() {
      this.decodedHeader = null;
      this.decodedPayload = null;
      this.signature = "";
    },
    
    handleInputKeydown(event) {
      if (event.ctrlKey && event.key === 'Enter') {
        event.preventDefault();
        this.decodeJwt();
      }
    }
  }
};
</script>

<style>
.jwt-decoder-container {
  max-width: 1300px;
  margin: 0 auto;
  padding: 12px;
  height: 100vh;
  display: flex;
  flex-direction: column;
  position: relative;
}


.controls {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 16px;
  margin-bottom: 12px;
  flex-wrap: wrap;
  padding: 12px;
  background: #f9fafb;
  border-radius: 8px;
  border: 1px solid #e1e5e9;
}

.side-by-side-layout {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
  flex: 1;
  min-height: 0;
  height: calc(100vh - 140px);
}

.input-side,
.output-side,
.empty-output {
  display: flex;
  flex-direction: column;
  min-height: 0;
}

.input-box {
  flex: 1;
  width: 100%;
  padding: 12px;
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
  font-size: 12px;
  border: 1px solid #e1e5e9;
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
  border-color: #4f46e5;
  box-shadow: 0 0 0 4px rgba(79, 70, 229, 0.1);
}

.view-options {
  display: flex;
  gap: 8px;
}

.empty-output {
  flex: 1;
  width: 100%;
  min-height: 0;
}

.empty-state {
  flex: 1;
  width: 100%;
  background: #f9fafb;
  border: 1px dashed #d1d5db;
  border-radius: 6px;
  padding: 12px;
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
  font-size: 12px;
  line-height: 1.3;
  min-height: 0;
  color: #6b7280;
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
  color: #9ca3af;
}

.decode-btn {
  background: #4f46e5;
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

.decode-btn:hover {
  background: #4338ca;
  transform: translateY(-1px);
  box-shadow: 0 2px 6px rgba(79, 70, 229, 0.3);
}

.action-btn {
  background: #f3f4f6;
  color: #374151;
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

.output-section {
  flex: 1;
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 12px;
  overflow-y: auto;
  padding: 12px;
  background: #fafafa;
  border: 1px solid #e1e5e9;
  border-radius: 6px;
  box-sizing: border-box;
}

.output-container {
  min-height: 120px;
}

.output-container h4 {
  margin: 0 0 8px 0;
  color: #374151;
  font-weight: 600;
  font-size: 14px;
  text-align: left;
}

.json-viewer {
  background: #ffffff;
  border: 1px solid #d1d5db;
  border-radius: 4px;
  padding: 8px;
  text-align: left;
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
  font-size: 11px;
  min-height: 80px;
  overflow-y: auto;
  line-height: 1.3;
}

.json-viewer pre {
  margin: 0;
  color: #374151;
  white-space: pre-wrap;
  word-wrap: break-word;
}

.signature-box {
  background: #fef3c7;
  border: 1px solid #f59e0b;
  border-radius: 4px;
  padding: 8px;
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
  font-size: 11px;
  word-break: break-all;
  color: #92400e;
  min-height: 40px;
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