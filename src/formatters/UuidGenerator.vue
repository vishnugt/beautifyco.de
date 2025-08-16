<template>
  <div class="uuid-generator-container">
    <div class="controls">
      <button @click="generateUuids" class="generate-btn">Generate 10 UUIDs</button>
      <button @click="copyAllUuids" v-if="uuids.length > 0" class="copy-all-btn">Copy All</button>
      <button @click="clearAll" v-if="uuids.length > 0" class="clear-btn">Clear</button>
    </div>
    
    <div v-if="uuids.length > 0" class="uuid-list">
      <div v-for="(uuid, index) in uuids" :key="index" class="uuid-item">
        <span class="uuid-number">{{ index + 1 }}.</span>
        <code class="uuid-value">{{ uuid }}</code>
        <button @click="copyUuid(uuid, index)" class="copy-uuid-btn" :class="{ 'copied': copiedStates[index] }">
          {{ copiedStates[index] ? '✓' : 'Copy' }}
        </button>
      </div>
    </div>
    
    <div v-if="uuids.length === 0" class="empty-state">
      <p>Click "Generate 10 UUIDs" to create new UUIDs</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      uuids: [],
      copiedStates: {}
    };
  },
  mounted() {
    this.generateUuids();
  },
  methods: {
    generateUuids() {
      this.uuids = [];
      this.copiedStates = {};
      
      for (let i = 0; i < 10; i++) {
        this.uuids.push(this.generateUuid());
      }
    },
    
    generateUuid() {
      // Generate UUID v4
      return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
        const r = Math.random() * 16 | 0;
        const v = c == 'x' ? r : (r & 0x3 | 0x8);
        return v.toString(16);
      });
    },
    
    copyUuid(uuid, index) {
      navigator.clipboard.writeText(uuid).then(() => {
        this.copiedStates[index] = true;
        this.$forceUpdate();
        
        // Reset copied state after 2 seconds
        setTimeout(() => {
          this.copiedStates[index] = false;
          this.$forceUpdate();
        }, 2000);
      });
    },
    
    copyAllUuids() {
      const allUuids = this.uuids.join('\n');
      navigator.clipboard.writeText(allUuids).then(() => {
        // Show success feedback for all items
        this.uuids.forEach((_, index) => {
          this.copiedStates[index] = true;
        });
        this.$forceUpdate();
        
        // Reset all copied states after 2 seconds
        setTimeout(() => {
          this.copiedStates = {};
          this.$forceUpdate();
        }, 2000);
      });
    },
    
    clearAll() {
      this.uuids = [];
      this.copiedStates = {};
    }
  }
};
</script>

<style>
.uuid-generator-container {
  max-width: 1000px;
  margin: 0 auto;
  padding: 24px;
  text-align: center;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  position: relative;
}


.controls {
  margin: 20px 0;
  display: flex;
  justify-content: center;
  gap: 12px;
  flex-wrap: wrap;
}

.generate-btn {
  background: #4f46e5;
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 8px;
  font-weight: 600;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.2s ease;
  box-shadow: 0 2px 4px rgba(79, 70, 229, 0.2);
}

.generate-btn:hover {
  background: #4338ca;
  box-shadow: 0 4px 8px rgba(79, 70, 229, 0.3);
  transform: translateY(-1px);
}

.copy-all-btn {
  background: #f3f4f6;
  color: #374151;
  border: 1px solid #d1d5db;
  padding: 12px 20px;
  border-radius: 8px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.copy-all-btn:hover {
  background: #e5e7eb;
  border-color: #9ca3af;
}

.clear-btn {
  background: #f3f4f6;
  color: #374151;
  border: 1px solid #d1d5db;
  padding: 12px 20px;
  border-radius: 8px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.clear-btn:hover {
  background: #e5e7eb;
  border-color: #9ca3af;
}

.uuid-list {
  margin-top: 24px;
  text-align: left;
  flex: 1;
  max-height: 70vh;
  overflow-y: auto;
}

.uuid-item {
  display: flex;
  align-items: center;
  padding: 12px 16px;
  margin-bottom: 8px;
  background: #f9fafb;
  border: 1px solid #e1e5e9;
  border-radius: 8px;
  transition: all 0.2s ease;
}

.uuid-item:hover {
  background: #f3f4f6;
  border-color: #d1d5db;
}

.uuid-number {
  color: #6b7280;
  font-weight: 500;
  font-size: 14px;
  margin-right: 12px;
  min-width: 25px;
}

.uuid-value {
  flex: 1;
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
  font-size: 14px;
  color: #374151;
  background: transparent;
  border: none;
  padding: 4px 8px;
  border-radius: 4px;
  user-select: all;
}

.copy-uuid-btn {
  background: #f3f4f6;
  color: #374151;
  border: 1px solid #d1d5db;
  padding: 6px 12px;
  border-radius: 6px;
  font-size: 12px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
  min-width: 50px;
}

.copy-uuid-btn:hover {
  background: #e5e7eb;
  border-color: #9ca3af;
}

.copy-uuid-btn.copied {
  background: #d1fae5;
  color: #065f46;
  border-color: #a7f3d0;
}

.copy-uuid-btn.copied:hover {
  background: #a7f3d0;
}

.empty-state {
  margin-top: 40px;
  padding: 40px 20px;
  text-align: center;
  color: #6b7280;
  border: 2px dashed #e1e5e9;
  border-radius: 12px;
  background: #fafafa;
}

.empty-state p {
  margin: 0;
  font-size: 16px;
  font-weight: 500;
}

@media (max-width: 768px) {
  .uuid-item {
    flex-direction: column;
    align-items: stretch;
    gap: 8px;
  }
  
  .uuid-number {
    min-width: auto;
    margin-right: 0;
  }
  
  .uuid-value {
    text-align: center;
    word-break: break-all;
  }
  
  .copy-uuid-btn {
    align-self: center;
    min-width: 80px;
  }
}
</style>