<template>
  <div class="item">
    <div class="item-left">
      <div class="addon-content">
        <div class="details">
          <div class="logo_container">
            <img :src="logoURL || defaultLogo" />
          </div>
          <div class="addon-info">
            <span class="addon-name">{{ name }}</span>
            <AddonFeatures :manifest="manifest" :showText="false" />
          </div>
        </div>
        <div class="actions-wrapper">
          <div class="col">
            <button class="button icon-only visit-url" title="Open addon configuration page in new window"
              :disabled="!isConfigurable" 
              @click="openAddonConfigurationPage" @mousedown.stop @touchstart.stop>
              <img src="/icons/arrow-up-right-12-000000.svg">
            </button>
            <button class="button icon-only copy-url" title="Copy addon manifest URL to clipboard"
              @click="copyManifestURLToClipboard" @mousedown.stop @touchstart.stop>
              <img src="/icons/clipboard-12-000000.svg">
            </button>
            <button class="button icon-only edit-addon" title="Edit addon" 
              @click="openEditManifestModal" @mousedown.stop @touchstart.stop>
              <img src="/icons/edit-12-000000.svg">
            </button>
            <button class="button icon-only delete" title="Remove addon from list" 
              :disabled="!isDeletable"
              @click="removeAddon" @mousedown.stop @touchstart.stop>
              <img src="/icons/trash-2-12-000000.svg">
            </button>
          </div>
        </div>
      </div>
    </div>
    <div class="item-right">
      <span class="priority-display">{{ priorityNumber }}</span>
      <input 
        type="number" 
        class="priority-input-right"
        :value="priorityNumber"
        min="1"
        :max="totalAddons"
        title="Enter new position (1 = highest priority)"
        @keydown.enter="handlePriorityChange($event)"
        @blur="handlePriorityChange($event)"
        @mousedown.stop
        @touchstart.stop
      />
    </div>
  </div>
</template>

<script setup>
  import { ref } from 'vue'
  import AddonFeatures from './AddonFeatures.vue'

  const props = defineProps({
    name: {
      type: String,
      required: true
    },
    idx: {
      type: Number,
      required: true
    },
    manifestURL: {
      type: String,
      required: true
    },
    logoURL: {
      type: String,
      required: false
    },
    isDeletable: {
      type: Boolean,
      required: false,
      default: true
    },
    isConfigurable: {
      type: Boolean,
      required: false,
      default: false
    },
    manifest: {
      type: Object,
      required: false,
      default: () => ({})
    },
    priorityNumber: {
      type: Number,
      required: true
    },
    totalAddons: {
      type: Number,
      required: true
    }
  })
  
  const emits = defineEmits(['delete-addon', 'edit-addon', 'show-toast', 'change-priority'])
  
  const defaultLogo = '/icons/box-48-ffffff.svg'
  
  function handlePriorityChange(event) {
    const input = event.target
    const newValue = parseInt(input.value, 10)
    
    if (isNaN(newValue)) {
      input.value = props.priorityNumber
      return
    }
    
    const clampedValue = Math.max(1, Math.min(newValue, props.totalAddons))
    
    if (clampedValue === props.priorityNumber) {
      input.value = props.priorityNumber
      return
    }
    
    emits('change-priority', props.idx, clampedValue)
  }
  
  async function copyManifestURLToClipboard() {
    try {
      if (navigator.clipboard && navigator.clipboard.writeText) {
        await navigator.clipboard.writeText(props.manifestURL)
        emits('show-toast', {
          message: 'Manifest URL copied to clipboard!',
          duration: 3000,
        })
      } else {
        const textArea = document.createElement('textarea')
        textArea.value = props.manifestURL
        textArea.style.position = 'fixed'
        textArea.style.left = '-999999px'
        textArea.style.top = '-999999px'
        document.body.appendChild(textArea)
        textArea.focus()
        textArea.select()
        
        try {
          const successful = document.execCommand('copy')
          document.body.removeChild(textArea)
          
          if (successful) {
            emits('show-toast', {
              message: 'Manifest URL copied to clipboard!',
              duration: 3000,
            })
          } else {
            throw new Error('Copy command failed')
          }
        } catch (err) {
          document.body.removeChild(textArea)
          console.error('Failed to copy text:', err)
          emits('show-toast', {
            message: 'Failed to copy URL. Please copy manually.',
            duration: 4000,
          })
        }
      }
    } catch (error) {
      console.error('Error copying text to clipboard', error)
      emits('show-toast', {
        message: 'Failed to copy URL. Please try again.',
        duration: 3000,
      })
    }
  }
  
  function openAddonConfigurationPage() {
    const configureURL = props.manifestURL.replace("stremio://", "https://").replace("/manifest.json", "/configure");
    window.open(configureURL, '_blank', 'noopener,noreferrer');
  }
  
  function removeAddon() {
    const addonName = props.name.length > 30 
      ? props.name.substring(0, 27) + '...' 
      : props.name
    
    emits('show-toast', {
      message: `"${addonName}" removed. Sync to Stremio to apply changes.`,
      duration: 4000,
    })
    
    emits('delete-addon', props.idx)
  }
  
  function openEditManifestModal() {
    emits('edit-addon', props.idx)
  }
</script>

<style scoped>
.item {
  list-style: none;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: stretch;
  cursor: default;
  border-radius: 5px;
  padding: 10px 13px;
  margin-bottom: 11px;
  border: 1px solid #ccc;
  position: relative;
}

.dark .item {
  border: 1px solid #434242;
}

.item-left {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.item-right {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding-left: 20px;
  min-width: 80px;
}

.priority-display {
  font-size: 36px;
  font-weight: 700;
  color: #2c5f8d;
  line-height: 1;
}

.dark .priority-display {
  color: #5a9fd4;
}

.priority-input-right {
  width: 60px;
  padding: 6px 10px;
  font-size: 16px;
  text-align: center;
  border: 1px solid #ccc;
  border-radius: 4px;
  background-color: #fff;
  color: #333;
  -moz-appearance: textfield;
}

.priority-input-right::-webkit-outer-spin-button,
.priority-input-right::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

.priority-input-right:focus {
  outline: none;
  border-color: #2c5f8d;
  box-shadow: 0 0 0 2px rgba(44, 95, 141, 0.2);
}

.dark .priority-input-right {
  background-color: #333;
  border-color: #555;
  color: #e0e0e0;
}

.dark .priority-input-right:focus {
  border-color: #5a9fd4;
  box-shadow: 0 0 0 2px rgba(90, 159, 212, 0.2);
}

.addon-content {
  display: flex;
  flex-direction: column;
  width: 100%;
}

.item .details {
  display: flex;
  align-items: center;
  flex: 1;
  margin-bottom: 10px;
}

.addon-info {
  display: flex;
  flex-direction: column;
  gap: 6px;
  flex: 1;
}

.addon-name {
  font-weight: 500;
}

.item .details img {
  height: 60px;
  width: 60px;
  pointer-events: none;
  margin-right: 12px;
  object-fit: contain;
  object-position: center;
  border-radius: 30%;
  background-color: #262626;
}

.col {
  display: flex;
  gap: 6px;
  flex-wrap: nowrap;
  align-items: center;
  min-width: auto;
  flex-shrink: 1;
}

.actions-wrapper {
  display: flex;
  align-items: center;
  gap: 0;
  flex-shrink: 0;
  width: 100%;
  justify-content: flex-start;
}

.button {
  border-radius: 6px;
  cursor: pointer;
  padding: 8px;
  background-color: #2c5f8d;
  border: none;
  transition: background-color 0.2s ease, opacity 0.2s ease, transform 0.1s ease, box-shadow 0.15s ease;
}

.button:hover:not(:disabled) {
  background-color: #234a6f;
}

.button:active:not(:disabled) {
  background-color: #1a3a56;
  transform: scale(0.95);
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.2);
}

.button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
  opacity: 0.5;
  transform: none;
}

.dark .button {
  background-color: #3a6a96;
}

.dark .button:hover:not(:disabled) {
  background-color: #2d5478;
}

.dark .button:active:not(:disabled) {
  background-color: #234260;
  transform: scale(0.95);
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.3);
}

.dark .button:disabled {
  background-color: #555555;
  opacity: 0.4;
}

.icon-only {
  display: flex;
  justify-content: center;
  align-items: center;
}

.visit-url img,
.copy-url img,
.edit-addon img {
  width: 20px;
  height: 20px;
  filter: brightness(0);
  pointer-events: none;
}

.delete img {
  width: 20px;
  height: 20px;
  filter: brightness(0) saturate(100%) invert(25%) sepia(85%) saturate(3500%) hue-rotate(345deg);
  pointer-events: none;
}

.dark .visit-url img,
.dark .copy-url img,
.dark .edit-addon img {
  filter: brightness(0);
}

.dark .delete img {
  filter: brightness(0) saturate(100%) invert(35%) sepia(85%) saturate(4000%) hue-rotate(345deg);
}

@media (max-width: 768px) {
  .item .details img {
    margin-right: 12px;
    margin-bottom: 8px;
  }

  .col {
    flex-direction: row;
    gap: 6px;
    justify-content: flex-start;
    min-width: auto;
    flex-wrap: nowrap;
    flex-shrink: 1;
  }

  .button {
    padding: 6px;
    min-width: 32px;
    flex-shrink: 0;
  }
}

@media (max-width: 480px) {
  .item .details {
    flex-direction: row;
    align-items: center;
  }

  .item .details img {
    margin-bottom: 0;
    margin-right: 12px;
  }

  .col {
    gap: 3px;
    flex-wrap: nowrap;
  }

  .button {
    padding: 4px;
    min-width: 28px;
  }

  .button img {
    width: 16px;
    height: 16px;
  }
}
</style>
