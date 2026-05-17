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
            <div class="addon-toggle-container">
              <label class="addon-toggle-switch" :title="isAddonVisible ? 'Visible on Home screen (Click to hide)' : 'Hidden from Home screen (Click to show)'">
                <input type="checkbox" :checked="isAddonVisible" @change="toggleVisibility">
                <span class="addon-toggle-slider">
                  <img src="/icons/home-24-000000.svg" class="switch-icon">
                </span>
              </label>
            </div>
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
      <span class="drag-handle" aria-label="Reorder addon">
        <img src="/icons/move-32-000000.svg" alt="" aria-hidden="true" />
      </span>
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
  import { ref, computed } from 'vue'
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
  
  const emits = defineEmits(['delete-addon', 'edit-addon', 'show-toast', 'change-priority', 'toggle-addon-visibility'])
  
  const defaultLogo = '/icons/box-48-ffffff.svg'

  const isAddonVisible = computed(() => {
    if (!props.manifest.catalogs || props.manifest.catalogs.length === 0) return true;
    return props.manifest.catalogs.some(catalog => {
      if (!Array.isArray(catalog.extra)) return true;
      return !catalog.extra.some(e => e && e.isRequired === true);
    });
  })

  function toggleVisibility() {
    emits('toggle-addon-visibility', props.idx)
  }
  
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
      // Modern clipboard API - works on most browsers including mobile
      if (navigator.clipboard && navigator.clipboard.writeText) {
        await navigator.clipboard.writeText(props.manifestURL)
        emits('show-toast', {
          message: 'Manifest URL copied to clipboard!',
          duration: 3000,
        })
      } else {
        // Fallback for older browsers/mobile
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
    // Prepare toast notification before emitting delete
    const addonName = props.name.length > 30 
      ? props.name.substring(0, 27) + '...' 
      : props.name
    
    // Emit to parent to show toast (since this component will be destroyed)
    emits('show-toast', {
      message: `"${addonName}" removed. Sync to Stremio to apply changes.`,
      duration: 4000,
    })
    
    // Then emit the delete event
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
  cursor: default;
  align-items: center;
  border-radius: 5px;
  padding: 10px 13px;
  margin-bottom: 11px;
  border: 1px solid #ccc;
  justify-content: space-between;
  flex-wrap: wrap;
  position: relative; /* Needed for absolute positioning of drag handle on mobile */
  flex-direction: row; /* Keep horizontal layout on desktop */
}

@media (max-width: 768px) {
  .sortable-list .item {
    padding: 10px;
    /* Giữ nguyên flex-direction: row từ .item mặc định để phần số thứ tự luôn ở bên phải */
  }
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
  gap: 8px 6px; /* row-gap 8px, column-gap 6px */
  flex-wrap: wrap; /* Allow buttons to wrap to next line if needed */
  align-items: center;
  min-width: auto;
  flex-shrink: 1;
}

.actions-wrapper {
  display: flex;
  align-items: center;
  gap: 0; /* No gap needed, drag-handle has its own margin */
  flex-shrink: 0; /* Prevent shrinking on mobile */
  width: 100%;
  justify-content: space-between; /* Push buttons left, drag handle right */
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
  filter: brightness(0); /* Make icons black */
  pointer-events: none; /* Prevent images from intercepting click events */
}

/* Modern Toggle Switch Styling */
.addon-toggle-container {
  display: flex;
  align-items: center;
  padding: 0 4px;
}

.addon-toggle-switch {
  position: relative;
  display: inline-block;
  width: 46px;
  height: 24px;
  margin: 0;
}

.addon-toggle-switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.addon-toggle-slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #dc3545; /* Hidden state (Red) */
  transition: .3s;
  border-radius: 24px;
  display: flex;
  align-items: center;
  padding: 0 4px;
}

.addon-toggle-slider:before {
  position: absolute;
  content: "";
  height: 18px;
  width: 18px;
  left: 3px;
  bottom: 3px;
  background-color: white;
  transition: .3s;
  border-radius: 50%;
  z-index: 2;
  box-shadow: 0 2px 4px rgba(0,0,0,0.2);
}

input:checked + .addon-toggle-slider {
  background-color: #28a745; /* Visible state (Green) */
}

input:focus + .addon-toggle-slider {
  box-shadow: 0 0 1px #28a745;
}

input:checked + .addon-toggle-slider:before {
  transform: translateX(22px);
}

.switch-icon {
  width: 12px !important;
  height: 12px !important;
  position: absolute;
  z-index: 3;
  transition: .3s;
  pointer-events: none;
  filter: brightness(0) saturate(100%) invert(48%) sepia(13%) saturate(3207%) hue-rotate(314deg) brightness(95%) contrast(112%); /* Initial red-ish icon color */
}

/* Position icon inside the slider circle */
input:not(:checked) + .addon-toggle-slider .switch-icon {
  left: 6px;
  filter: grayscale(1) brightness(0.5);
}

input:checked + .addon-toggle-slider .switch-icon {
  left: 28px;
  filter: brightness(0) saturate(100%) invert(45%) sepia(93%) saturate(1418%) hue-rotate(88deg) brightness(101%) contrast(106%); /* Green-ish icon */
}

.dark .addon-toggle-slider {
  background-color: #555;
}

.dark input:checked + .addon-toggle-slider {
  background-color: #218838;
}

.delete img {
  width: 20px;
  height: 20px;
  filter: brightness(0) saturate(100%) invert(25%) sepia(85%) saturate(3500%) hue-rotate(345deg); /* Red color for delete */
  pointer-events: none; /* Prevent images from intercepting click events */
}

.dark .visit-url img,
.dark .copy-url img,
.dark .edit-addon img {
  filter: brightness(0); /* Keep icons black even in dark mode */
}

.dark .delete img {
  filter: brightness(0) saturate(100%) invert(35%) sepia(85%) saturate(4000%) hue-rotate(345deg); /* Brighter red for dark mode */
}

.drag-handle {
  cursor: move;
  touch-action: none; /* Disable scrolling when touching drag handle */
  user-select: none;
  -webkit-user-select: none;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 6px;
  margin: 0;
}

.drag-handle img {
  width: 24px;
  height: 24px;
  filter: brightness(0); /* Make icon black */
  pointer-events: none;
}

.dark .drag-handle img {
  filter: brightness(0) invert(1); /* Make icon white in dark mode */
}

.drag-handle:hover {
  opacity: 0.7;
}

@media (max-width: 768px) {
  .sortable-list .item {
    /* Keep flex-direction: row */
    padding: 10px;
  }

  .item-right {
    padding-left: 10px; /* Tiết kiệm không gian trên mobile */
    min-width: 60px;
  }

  .item .details {
    margin-bottom: 10px;
    align-self: flex-start; /* Force left alignment */
  }

  .item .details img {
    margin-right: 12px;
    margin-bottom: 8px;
  }

  .actions-wrapper {
    width: 100%;
    margin-top: 10px;
  }

  .col {
    flex-direction: row;
    gap: 8px 6px;
    justify-content: flex-start;
    min-width: auto;
    flex-wrap: wrap; /* Allow wrapping on mobile */
    flex-shrink: 1;
  }

  .button {
    padding: 6px;
    min-width: 32px; /* Ensure buttons don't get too small */
    flex-shrink: 0; /* Don't let individual buttons shrink */
  }
}

@media (max-width: 480px) {
  .item .details {
    flex-direction: row; /* Keep horizontal on small screens */
    align-items: center;
  }

  .item .details img {
    margin-bottom: 0; /* Remove bottom margin */
    margin-right: 12px; /* Keep right margin */
  }

  .actions-wrapper {
    margin-top: 6px;
  }

  .col {
    gap: 6px 4px; /* Minimal gap for very small screens */
    flex-wrap: wrap; /* Allow buttons to wrap */
  }

  .button {
    padding: 4px; /* Smaller padding on very small screens */
    min-width: 28px; /* Slightly smaller minimum on tiny screens */
  }

  .button img {
    width: 16px; /* Slightly smaller icons on very small screens */
    height: 16px;
  }
}
</style>
