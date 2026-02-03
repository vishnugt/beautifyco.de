<template>
  <div class="formatter-container">
    <!-- Minimized View -->
    <div v-if="!isMaximized" class="minimized-view">
      <div class="controls">
        <div class="indentation-control">
          <label>Indentation:</label>
          <div class="toggle-group">
            <button 
              @click="setIndentation(2)" 
              :class="['toggle-btn', { active: indentation === 2 }]"
              title="2 Spaces">
              2sp
            </button>
            <button 
              @click="setIndentation(4)" 
              :class="['toggle-btn', { active: indentation === 4 }]"
              title="4 Spaces">
              4sp
            </button>
            <button 
              @click="setIndentation(0)" 
              :class="['toggle-btn', { active: indentation === 0 }]"
              title="Minified">
              Min
            </button>
          </div>
        </div>
        <div class="view-options">
          <button @click="formatJson" class="beautify-btn" :disabled="!jsonInput.trim()">Beautify</button>
          <button @click="expandAll" class="action-btn" :disabled="!parsedJson">Expand All</button>
          <button @click="collapseAll" class="action-btn" :disabled="!parsedJson">Collapse All</button>
          <button @click="copyToClipboard" class="action-btn" :disabled="!parsedJson">Copy</button>
          <button @click="toggleMaximize" class="maximize-btn" :disabled="!parsedJson">Maximize</button>
        </div>
      </div>

      <!-- Side by Side Layout -->
      <div class="side-by-side-layout">
        <!-- Input Side -->
        <div class="input-side">
          <textarea
              v-model="jsonInput"
              placeholder="Paste JSON here... (Ctrl+Enter to format)"
              class="input-box"
              @keydown="handleInputKeydown">
          </textarea>
        </div>

        <!-- Output Side -->
        <div class="output-side" v-if="parsedJson">
          <div class="json-viewer">
            <div v-html="renderJsonHtml(parsedJson, 'root', 0)"></div>
          </div>
        </div>

        <!-- Empty State -->
        <div class="empty-output empty-state" style="margin-top: 0" v-else>
          <p>Your formatted JSON will appear here</p>
          <p class="hint">Press Ctrl+Enter to format</p>
        </div>
      </div>

      <p class="error-message" v-if="errorMessage">{{ errorMessage }}</p>
    </div>

    <!-- Fullscreen View -->
    <div v-if="isMaximized" class="fullscreen-view">
      <div class="fullscreen-header">
        <div class="fullscreen-controls">
          <div class="indentation-control">
            <label>Indentation:</label>
            <div class="toggle-group">
              <button 
                @click="setIndentation(2)" 
                :class="['toggle-btn', { active: indentation === 2 }]"
                title="2 Spaces">
                2sp
              </button>
              <button 
                @click="setIndentation(4)" 
                :class="['toggle-btn', { active: indentation === 4 }]"
                title="4 Spaces">
                4sp
              </button>
              <button 
                @click="setIndentation(0)" 
                :class="['toggle-btn', { active: indentation === 0 }]"
                title="Minified">
                Min
              </button>
            </div>
          </div>
          <div class="control-group">
            <div class="view-options" v-if="parsedJson">
              <button @click="expandAll" class="action-btn">Expand All</button>
              <button @click="collapseAll" class="action-btn">Collapse All</button>
              <button @click="copyToClipboard" class="action-btn">Copy</button>
            </div>
          </div>
          <button @click="toggleMaximize" class="minimize-btn">Minimize</button>
        </div>
      </div>

      <div class="fullscreen-content">
        <div class="fullscreen-output" v-if="parsedJson">
          <div class="fullscreen-json-viewer">
            <div v-html="renderJsonHtml(parsedJson, 'root', 0)"></div>
          </div>
        </div>

        <div class="fullscreen-empty" v-else>
          <div class="fullscreen-empty-state">
            <p>No JSON formatted yet</p>
          </div>
        </div>
      </div>

      <p class="fullscreen-error-message" v-if="errorMessage">{{ errorMessage }}</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      jsonInput: "",
      parsedJson: null,
      errorMessage: "",
      indentation: 2,
      expandedNodes: {},
      isMaximized: false  // Start in minimized mode for input
    };
  },
  mounted() {
    document.addEventListener('keydown', this.handleKeydown);
  },
  beforeUnmount() {
    document.removeEventListener('keydown', this.handleKeydown);
  },
  methods: {
    formatJson() {
      try {
        if (!this.jsonInput.trim()) {
          this.errorMessage = "Please enter some JSON to format.";
          this.parsedJson = null;
          return;
        }
        this.parsedJson = JSON.parse(this.jsonInput);
        this.errorMessage = "";
        this.initializeExpandedState();
      } catch (error) {
        this.errorMessage = "Invalid JSON format.";
        this.parsedJson = null;
      }
    },
    initializeExpandedState() {
      this.expandedNodes = {};
      this.setDefaultExpandedState(this.parsedJson, 'root', 0);
      // Set up global toggle function for HTML clicks
      window.toggleNode = (path) => {
        this.toggleNode(path);
      };
    },
    setDefaultExpandedState(obj, path, depth) {
      if (obj !== null && typeof obj === 'object') {
        this.expandedNodes[path] = depth < 2; // Auto-expand first 2 levels

        if (Array.isArray(obj)) {
          obj.forEach((item, index) => {
            this.setDefaultExpandedState(item, `${path}.${index}`, depth + 1);
          });
        } else {
          Object.entries(obj).forEach(([key, value]) => {
            this.setDefaultExpandedState(value, `${path}.${key}`, depth + 1);
          });
        }
      }
    },
    toggleNode(path) {
      this.expandedNodes[path] = !this.expandedNodes[path];
      this.$forceUpdate();
    },
    expandAll() {
      this.setAllExpandedState(this.parsedJson, 'root', true);
      this.$forceUpdate();
    },
    collapseAll() {
      this.setAllExpandedState(this.parsedJson, 'root', false);
      this.$forceUpdate();
    },
    setAllExpandedState(obj, path, expanded) {
      if (obj !== null && typeof obj === 'object') {
        this.expandedNodes[path] = expanded;

        if (Array.isArray(obj)) {
          obj.forEach((item, index) => {
            this.setAllExpandedState(item, `${path}.${index}`, expanded);
          });
        } else {
          Object.entries(obj).forEach(([key, value]) => {
            this.setAllExpandedState(value, `${path}.${key}`, expanded);
          });
        }
      }
    },
    copyToClipboard() {
      const jsonString = this.indentation === 0 ? JSON.stringify(this.parsedJson) : JSON.stringify(this.parsedJson, null, this.indentation);
      navigator.clipboard.writeText(jsonString).then(() => {
        // Could add a toast notification here
      });
    },
    renderJsonHtml(data, path, indentLevel) {
      const isExpanded = this.expandedNodes[path] !== false;

      if (data === null) {
        return `<span class="value null">null</span>`;
      }

      if (typeof data !== 'object') {
        const className = typeof data;
        const displayValue = typeof data === 'string' ? `"${data}"` : data.toString();
        return `<span class="value ${className}">${displayValue}</span>`;
      }

      const isArray = Array.isArray(data);
      const entries = isArray
          ? data.map((item, index) => ({ key: index, value: item }))
          : Object.entries(data).map(([key, value]) => ({ key, value }));

      const openBracket = isArray ? '[' : '{';
      const closeBracket = isArray ? ']' : '}';

      // For minified view, return inline compact representation
      if (this.indentation === 0) {
        let compactHtml = entries.length > 0 ? `<span class="json-toggle" onclick="window.toggleNode('${path}')" data-path="${path}">${isExpanded ? '▼' : '▶'}</span>` : '';
        compactHtml += `<span class="bracket">${openBracket}</span>`;
        
        if (isExpanded && entries.length > 0) {
          entries.forEach((entry, index) => {
            if (!isArray) {
              compactHtml += `<span class="key">"${entry.key}":</span>`;
            }
            compactHtml += this.renderJsonHtml(entry.value, `${path}.${entry.key}`, indentLevel + 1);
            if (index < entries.length - 1) {
              compactHtml += `<span class="comma">,</span>`;
            }
          });
        } else if (!isExpanded && entries.length > 0) {
          compactHtml += `<span class="ellipsis">...</span>`;
        }
        
        compactHtml += `<span class="bracket">${closeBracket}</span>`;
        return compactHtml;
      }

      // Regular view - simple approach
      if (!isExpanded) {
        return `<div class="json-line" style="padding-left: ${indentLevel * this.indentation * 8}px;">
          <span class="json-toggle" onclick="window.toggleNode('${path}')" data-path="${path}">▶</span>
          <span class="bracket">${openBracket}...${closeBracket}</span>
          <span class="item-count">(${entries.length})</span>
        </div>`;
      }

      let html = `<div class="json-line" style="padding-left: ${indentLevel * this.indentation * 8}px;">`;
      if (entries.length > 0) {
        html += `<span class="json-toggle" onclick="window.toggleNode('${path}')" data-path="${path}">▼</span>`;
      }
      html += `<span class="bracket">${openBracket}</span></div>`;
      
      if (entries.length > 0) {
        entries.forEach((entry, index) => {
          html += `<div class="json-line" style="padding-left: ${(indentLevel + 1) * this.indentation * 8}px;">`;
          if (!isArray) {
            html += `<span class="key">"${entry.key}"</span><span class="colon">: </span>`;
          }
          html += this.renderJsonHtml(entry.value, `${path}.${entry.key}`, indentLevel + 1);
          if (index < entries.length - 1) {
            html += `<span class="comma">,</span>`;
          }
          html += `</div>`;
        });
        html += `<div class="json-line" style="padding-left: ${indentLevel * this.indentation * 8}px;">
          <span class="bracket">${closeBracket}</span>
        </div>`;
      }

      return html;
    },
    handleInputKeydown(event) {
      if (event.ctrlKey && event.key === 'Enter') {
        event.preventDefault();
        event.stopPropagation();
        this.formatJson();
      }
    },
    toggleMaximize() {
      this.isMaximized = !this.isMaximized;
      if (this.isMaximized) {
        document.body.style.overflow = 'hidden';
      } else {
        document.body.style.overflow = '';
      }
    },
    handleKeydown(event) {
      if (event.key === 'Escape' && this.isMaximized) {
        event.preventDefault();
        this.toggleMaximize();
      }
      if (event.ctrlKey && event.key === 'Enter') {
        event.preventDefault();
        if (this.isMaximized) {
          // In fullscreen mode, minimize
          this.toggleMaximize();
        } else {
          // In minimized mode, format JSON (same as textarea handler)
          this.formatJson();
        }
      }
    },
    setIndentation(value) {
      this.indentation = value;
      this.updateIndentation();
    },
    updateIndentation() {
      if (this.parsedJson) {
        this.$forceUpdate();
      }
    }
  }
};
</script>

<style>
.formatter-container {
  width: 100%;
  height: 100vh;
  position: relative;
}


/* Minimized View Styles */
.minimized-view {
  max-width: 1300px;
  margin: 0 auto;
  padding: 12px;
  height: calc(100vh - 80px);
  display: flex;
  flex-direction: column;
  box-sizing: border-box;
  overflow: hidden;
}

/* Responsive layout for larger displays */
@media (min-width: 1800px) {
  .minimized-view {
    max-width: 1800px;
  }
}

@media (min-width: 2200px) {
  .minimized-view {
    max-width: 2000px;
  }
}


.controls {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  margin-bottom: 16px;
  padding: 12px 16px;
  background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 100%);
  border-radius: 12px;
  border: 1px solid #e2e8f0;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
}

.side-by-side-layout {
  display: grid;
  grid-template-columns: 50% 50%;
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
  min-width: 0;
  max-width: 100%;
  overflow: hidden;
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

.json-viewer {
  flex: 1;
  width: 100%;
  background: #f8fafc;
  border: 1px solid #e2e8f0;
  border-radius: 6px;
  padding: 12px;
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
  font-size: 12px;
  overflow-y: auto;
  overflow-x: auto;
  line-height: 1.3;
  min-height: 0;
  min-width: 0;
  box-sizing: border-box;
  word-wrap: break-word;
  overflow-wrap: break-word;
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
  overflow-y: auto;
  line-height: 1.3;
  min-height: 0;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  color: #64748b;
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

/* Fullscreen View Styles */
.fullscreen-view {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: #ffffff;
  z-index: 10000;
  display: flex;
  flex-direction: column;
}

.fullscreen-header {
  padding: 16px 24px;
  background: #f1f5f9;
  border-bottom: 1px solid #e1e5e9;
  flex-shrink: 0;
}

.fullscreen-controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 20px;
  flex-wrap: wrap;
}

.control-group {
  display: flex;
  align-items: center;
  gap: 12px;
}

.fullscreen-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 0;
}

.fullscreen-output,
.fullscreen-empty {
  padding: 32px;
  display: flex;
  flex-direction: column;
  flex: 1;
  min-height: 0;
}

.fullscreen-input-box {
  flex: 1;
  padding: 20px;
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
  font-size: 16px;
  border: 2px solid #e1e5e9;
  border-radius: 12px;
  resize: none;
  transition: border-color 0.2s ease;
  background: #ffffff;
  line-height: 1.6;
  min-height: 0;
}

.fullscreen-json-viewer {
  flex: 1;
  background: #f8fafc;
  border: 2px solid #e1e5e9;
  border-radius: 12px;
  padding: 20px;
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
  font-size: 12px;
  overflow-y: auto;
  line-height: 1.6;
  min-height: 0;
}


.fullscreen-empty-state {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f1f5f9;
  border: 2px dashed #d1d5db;
  border-radius: 12px;
  color: #64748b;
  font-style: italic;
  font-size: 18px;
}

/* Common Styles */
.indentation-control {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 6px 12px;
  background: rgba(255, 255, 255, 0.7);
  border-radius: 8px;
  border: 1px solid #e2e8f0;
}

.indentation-control label {
  font-weight: 600;
  color: #374151;
  font-size: 13px;
  white-space: nowrap;
}

.toggle-group {
  display: flex;
  border-radius: 8px;
  overflow: hidden;
  border: 1px solid #d1d5db;
  background: white;
  min-width: 150px;
}

.toggle-btn {
  padding: 8px 16px;
  border: none;
  background: white;
  color: #64748b;
  font-size: 12px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
  border-right: 1px solid #e2e8f0;
  position: relative;
  min-width: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
  white-space: nowrap;
}

.toggle-btn:last-child {
  border-right: none;
}

.toggle-btn:hover:not(.active) {
  background: #f1f5f9;
  color: #374151;
}

.toggle-btn.active {
  background: linear-gradient(135deg, #3b82f6 0%, #2563eb 100%);
  color: white;
  font-weight: 600;
  box-shadow: inset 0 1px 3px rgba(0, 0, 0, 0.1);
}

.toggle-btn.active:hover {
  background: linear-gradient(135deg, #2563eb 0%, #1d4ed8 100%);
}

.format-btn {
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

.format-btn:hover {
  background: #4338ca;
  transform: translateY(-1px);
  box-shadow: 0 2px 6px rgba(79, 70, 229, 0.3);
}

.view-options {
  display: flex;
  align-items: center;
  gap: 8px;
  flex-wrap: wrap;
}

.action-btn {
  background: linear-gradient(135deg, #ffffff 0%, #f8fafc 100%);
  color: #374151;
  border: 1px solid #d1d5db;
  padding: 7px 12px;
  border-radius: 6px;
  font-size: 12px;
  cursor: pointer;
  transition: all 0.2s ease;
  font-weight: 500;
  white-space: nowrap;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
}

.action-btn:hover:not(:disabled) {
  background: linear-gradient(135deg, #f1f5f9 0%, #e2e8f0 100%);
  border-color: #9ca3af;
  transform: translateY(-1px);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.action-btn:disabled {
  background: #f8fafc;
  color: #9ca3af;
  border-color: #e2e8f0;
  cursor: not-allowed;
  opacity: 0.6;
}

.maximize-btn {
  background: linear-gradient(135deg, #059669 0%, #047857 100%);
  color: white;
  border: none;
  padding: 7px 14px;
  border-radius: 6px;
  font-size: 12px;
  cursor: pointer;
  transition: all 0.2s ease;
  font-weight: 600;
  white-space: nowrap;
  box-shadow: 0 1px 3px rgba(5, 150, 105, 0.3);
}

.maximize-btn:hover:not(:disabled) {
  background: linear-gradient(135deg, #047857 0%, #065f46 100%);
  transform: translateY(-1px);
  box-shadow: 0 2px 6px rgba(5, 150, 105, 0.4);
}

.maximize-btn:disabled {
  background: #9ca3af;
  cursor: not-allowed;
  opacity: 0.6;
}

.beautify-btn {
  background: linear-gradient(135deg, #3b82f6 0%, #2563eb 100%);
  color: white;
  border: none;
  padding: 7px 14px;
  border-radius: 6px;
  font-size: 12px;
  cursor: pointer;
  transition: all 0.2s ease;
  font-weight: 600;
  white-space: nowrap;
  box-shadow: 0 1px 3px rgba(59, 130, 246, 0.3);
}

.beautify-btn:hover:not(:disabled) {
  background: linear-gradient(135deg, #2563eb 0%, #1d4ed8 100%);
  transform: translateY(-1px);
  box-shadow: 0 2px 6px rgba(59, 130, 246, 0.4);
}

.beautify-btn:disabled {
  background: #9ca3af;
  cursor: not-allowed;
  opacity: 0.6;
}

.minimize-btn {
  background: #dc2626;
  color: white;
  border: none;
  padding: 6px 14px;
  border-radius: 6px;
  font-weight: 500;
  font-size: 12px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.minimize-btn:hover {
  background: #b91c1c;
}

.input-box:focus,
.fullscreen-input-box:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

/* JSON Rendering Styles */
.json-line {
  line-height: 1.4;
  margin: 0;
  padding: 1px 0;
}

.json-content {
  margin: 0;
}

.json-line {
  line-height: 1.4;
  margin: 0;
  padding: 1px 0;
}

.toggle-spacer {
  width: 14px;
  display: inline-block;
}

.json-toggle {
  cursor: pointer;
  user-select: none;
  color: #64748b;
  font-size: 12px;
  width: 16px;
  text-align: center;
  transition: color 0.2s ease;
  display: inline-block;
  margin-right: 4px;
}

.json-toggle:hover {
  color: #1e293b;
}

.no-toggle {
  width: 16px;
}

.bracket {
  color: #64748b;
  font-weight: bold;
  margin-left: 4px;
}

.key {
  color: #dc2626;
  font-weight: 500;
  margin-left: 4px;
}

.array-index {
  color: #7c3aed;
  font-weight: 500;
  margin-left: 4px;
}

.value {
  margin-left: 4px;
}

.value.string {
  color: #059669;
}

.value.number {
  color: #dc2626;
}

.value.boolean {
  color: #7c3aed;
}

.value.object {
  color: #64748b;
}

.item-count {
  color: #64748b;
  font-style: italic;
  font-size: 12px;
  margin-left: 8px;
}

.comma {
  color: #64748b;
}

.colon {
  color: #64748b;
  margin: 0 2px;
}

.ellipsis {
  color: #9ca3af;
  font-style: italic;
}

/* Ensure consistent layout regardless of JSON content */
.json-viewer > div {
  max-width: 100%;
  overflow-wrap: break-word;
  word-break: break-word;
}

.placeholder-text {
  color: #64748b;
  font-size: 13px;
  font-style: italic;
  padding: 6px 12px;
  background: rgba(100, 116, 139, 0.05);
  border-radius: 6px;
  border: 1px dashed #cbd5e1;
}

.error-message,
.fullscreen-error-message {
  color: #dc2626;
  font-weight: 500;
  margin-top: 10px;
  padding: 8px 12px;
  background: #fef2f2;
  border: 1px solid #fecaca;
  border-radius: 6px;
  text-align: center;
}

.fullscreen-error-message {
  margin: 0 24px 24px 24px;
}

@media (max-width: 1024px) {
  .side-by-side-layout {
    grid-template-columns: 1fr;
  }

  .fullscreen-controls {
    flex-direction: column;
    gap: 12px;
  }

  .controls {
    flex-direction: column;
    gap: 12px;
  }
}
</style>
