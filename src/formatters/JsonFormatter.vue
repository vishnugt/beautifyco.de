<template>
  <div class="formatter-container">
    <!-- Minimized View -->
    <div v-if="!isMaximized" class="minimized-view">
      <div class="controls" v-if="parsedJson">
        <div class="view-options">
          <button @click="expandAll" class="action-btn">Expand All</button>
          <button @click="collapseAll" class="action-btn">Collapse All</button>
          <button @click="copyToClipboard" class="action-btn">Copy</button>
          <button @click="toggleMaximize" class="maximize-btn">Maximize</button>
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
            <select v-model="indentation" @change="updateIndentation">
              <option :value="2">2 Spaces</option>
              <option :value="4">4 Spaces</option>
            </select>
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
        // Auto-maximize after successful formatting
        if (!this.isMaximized) {
          this.toggleMaximize();
        }
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
      const jsonString = JSON.stringify(this.parsedJson, null, this.indentation);
      navigator.clipboard.writeText(jsonString).then(() => {
        // Could add a toast notification here
      });
    },
    renderJsonHtml(data, path, indentLevel) {
      const indent = '  '.repeat(indentLevel);
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
      const containerType = isArray ? 'array' : 'object';

      let html = `<div class="json-node">`;
      html += `<div class="node-content" style="padding-left: ${indentLevel * 20}px;">`;
      html += `<span class="toggle-btn" onclick="window.toggleNode('${path}')">${isExpanded ? '▼' : '▶'}</span>`;
      html += `<span class="bracket">${isExpanded ? openBracket : openBracket + '...'}</span>`;

      if (!isExpanded) {
        html += `<span class="item-count">(${entries.length} ${isArray ? 'items' : 'keys'})</span>`;
      }

      html += `</div>`;

      if (isExpanded) {
        html += `<div class="children">`;
        entries.forEach((entry) => {
          html += `<div class="key-value-pair" style="padding-left: ${(indentLevel + 1) * 20}px;">`;
          if (!isArray) {
            html += `<span class="key">"${entry.key}":</span>`;
          } else {
            html += `<span class="array-index">${entry.key}:</span>`;
          }
          html += `<span style="margin-left: 8px;">`;
          html += this.renderJsonHtml(entry.value, `${path}.${entry.key}`, indentLevel + 1);
          html += `</span>`;
          html += `</div>`;
        });
        html += `<div class="closing-bracket" style="padding-left: ${indentLevel * 20}px;">`;
        html += `<span class="bracket">${closeBracket}</span>`;
        html += `</div>`;
        html += `</div>`;
      }

      html += `</div>`;
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
      // Only handle if this component is active (has focus or is maximized)
      if (event.key === 'Escape' && this.isMaximized) {
        event.preventDefault();
        this.toggleMaximize();
      }
      if (event.ctrlKey && event.key === 'Enter' && this.isMaximized) {
        event.preventDefault();
        this.toggleMaximize();
      }
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
  border: 1px solid #e1e5e9;
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
  background: #fafafa;
  border: 1px solid #e1e5e9;
  border-radius: 6px;
  padding: 12px;
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
  font-size: 12px;
  overflow-y: auto;
  line-height: 1.3;
  min-height: 0;
  box-sizing: border-box;
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
  overflow-y: auto;
  line-height: 1.3;
  min-height: 0;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: left;
  color: #6b7280;
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
  background: #f9fafb;
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
  background: #fafafa;
  border: 2px solid #e1e5e9;
  border-radius: 12px;
  padding: 20px;
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
  font-size: 16px;
  overflow-y: auto;
  line-height: 1.6;
  min-height: 0;
}


.fullscreen-empty-state {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f9fafb;
  border: 2px dashed #d1d5db;
  border-radius: 12px;
  color: #6b7280;
  font-style: italic;
  font-size: 18px;
}

/* Common Styles */
.indentation-control {
  display: flex;
  align-items: center;
  gap: 8px;
}

.indentation-control label {
  font-weight: 500;
  color: #374151;
}

.indentation-control select {
  padding: 8px 12px;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  font-size: 14px;
  background: white;
}

.format-btn {
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

.format-btn:hover {
  background: #4338ca;
  transform: translateY(-1px);
  box-shadow: 0 2px 6px rgba(79, 70, 229, 0.3);
}

.view-options {
  display: flex;
  gap: 8px;
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

.maximize-btn {
  background: #059669;
  color: white;
  border: none;
  padding: 5px 10px;
  border-radius: 4px;
  font-size: 11px;
  cursor: pointer;
  transition: all 0.2s ease;
  font-weight: 500;
}

.maximize-btn:hover {
  background: #047857;
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
  border-color: #4f46e5;
  box-shadow: 0 0 0 4px rgba(79, 70, 229, 0.1);
}

/* JSON Rendering Styles */
.json-node {
  line-height: 1.5;
}

.node-content {
  display: flex;
  align-items: center;
  margin: 2px 0;
}

.toggle-btn {
  cursor: pointer;
  user-select: none;
  color: #6b7280;
  font-size: 12px;
  width: 16px;
  text-align: center;
  transition: color 0.2s ease;
}

.toggle-btn:hover {
  color: #374151;
}

.no-toggle {
  width: 16px;
}

.bracket {
  color: #6b7280;
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
  color: #6b7280;
}

.item-count {
  color: #9ca3af;
  font-style: italic;
  font-size: 12px;
  margin-left: 8px;
}

.key-value-pair {
  margin: 1px 0;
  display: flex;
  align-items: flex-start;
}

.closing-bracket {
  margin: 2px 0;
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
