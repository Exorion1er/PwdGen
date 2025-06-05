<template>
  <div class="password-generator">
    <div class="password-display" ref="containerRef">
      <div class="password-text" ref="passwordTextRef">{{ password }}</div>
    </div>
    <div class="button-container">
      <button class="action-button refresh-button" @click="generatePassword" title="Generate new password">
        <span class="action-text">Refresh</span>
        <svg class="refresh-icon" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path
            d="M17.65 6.35C16.2 4.9 14.21 4 12 4C7.59 4 4 7.59 4 12C4 16.41 7.59 20 12 20C15.73 20 18.84 17.45 19.73 14H17.65C16.83 16.33 14.61 18 12 18C8.69 18 6 15.31 6 12C6 8.69 8.69 6 12 6C13.66 6 15.14 6.69 16.22 7.78L13 11H20V4L17.65 6.35Z"
            fill="currentColor" />
        </svg>
      </button>
      <button class="action-button copy-button" @click="copyToClipboard" title="Copy to clipboard">
        <span class="action-text">Copy</span>
        <svg v-if="!copied" class="copy-icon" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path
            d="M16 1H4C2.9 1 2 1.9 2 3V17H4V3H16V1ZM19 5H8C6.9 5 6 5.9 6 7V21C6 22.1 6.9 23 8 23H19C20.1 23 21 22.1 21 21V7C21 5.9 20.1 5 19 5ZM19 21H8V7H19V21Z"
            fill="currentColor" />
        </svg>
        <svg v-else class="check-icon" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M9 16.17L4.83 12l-1.42 1.41L9 19 21 7l-1.41-1.41L9 16.17z" fill="currentColor" />
        </svg>
      </button>
    </div>

    <div class="settings-box">
      <div class="setting length-setting">
        <div class="length-label-row">
          <span class="length-label">LENGTH</span>
          <span class="length-value" @click="toggleLengthInput" v-if="!isLengthInputVisible">({{ settings.length
            }})</span>
          <input v-else type="number" v-model.number="settings.length" class="length-input" ref="lengthInputRef" />
        </div>
        <input type="range" v-model.number="settings.length" :min="4" :max="100" class="slider" />
        <div class="length-minmax">
          <span>{{ 4 }}</span>
          <span>{{ 100 }}</span>
        </div>
      </div>
      <div class="setting">
        <label>
          <input type="checkbox" v-model="settings.includeUppercase">
          Uppercase (A-Z)
        </label>
      </div>
      <div class="setting">
        <label>
          <input type="checkbox" v-model="settings.includeLowercase">
          Lowercase (a-z)
        </label>
      </div>
      <div class="setting">
        <label>
          <input type="checkbox" v-model="settings.includeNumbers">
          Numbers (0-9)
        </label>
      </div>
      <div class="setting">
        <label>
          <input type="checkbox" v-model="settings.includeSpecial">
          Special Characters (!@#$%^&*)
        </label>
      </div>
      <div class="setting required-chars">
        <label>
          Required Characters:
          <input type="text" v-model="settings.requiredChars" placeholder="Type characters to include"
            class="required-chars-input" />
        </label>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted, watch, computed, onBeforeUnmount, nextTick } from 'vue'

const password = ref('')
const copied = ref(false)
const passwordTextRef = ref<HTMLElement | null>(null)
const containerRef = ref<HTMLElement | null>(null)
const isLengthInputVisible = ref(false)
const lengthInputRef = ref<HTMLInputElement | null>(null)
const MOUSE_DISTANCE_THRESHOLD = 100 // pixels

const settings = reactive({
  length: 16,
  includeUppercase: true,
  includeLowercase: true,
  includeNumbers: true,
  includeSpecial: true,
  requiredChars: ''
})

const generatePassword = () => {
  const uppercase = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
  const lowercase = 'abcdefghijklmnopqrstuvwxyz'
  const numbers = '0123456789'
  const special = '!@#$%^&*'

  let chars = ''
  if (settings.includeUppercase) chars += uppercase
  if (settings.includeLowercase) chars += lowercase
  if (settings.includeNumbers) chars += numbers
  if (settings.includeSpecial) chars += special

  if (chars === '') {
    password.value = 'Please select at least one character type'
    return
  }

  // Calculate remaining length after including required characters
  const remainingLength = settings.length - settings.requiredChars.length

  if (remainingLength < 0) {
    password.value = 'Password length too short for required characters'
    return
  }

  // Generate random part of the password
  let result = ''
  for (let i = 0; i < remainingLength; i++) {
    result += chars.charAt(Math.floor(Math.random() * chars.length))
  }

  // Insert required characters at random positions
  const requiredChars = settings.requiredChars.split('')
  for (const char of requiredChars) {
    const insertPos = Math.floor(Math.random() * (result.length + 1))
    result = result.slice(0, insertPos) + char + result.slice(insertPos)
  }

  password.value = result
}

const copyToClipboard = async () => {
  try {
    await navigator.clipboard.writeText(password.value)
    copied.value = true
    setTimeout(() => {
      copied.value = false
    }, 2000)
  } catch (err) {
    console.error('Failed to copy:', err)
  }
}

const adjustFontSize = () => {
  if (!passwordTextRef.value || !containerRef.value) return

  const container = containerRef.value
  const text = passwordTextRef.value

  // Start with max size
  text.style.fontSize = '3.5rem'
  text.style.whiteSpace = 'nowrap'

  // Get the actual available width
  const containerRect = container.getBoundingClientRect()
  const buttonRect = container.querySelector('.copy-button')?.getBoundingClientRect()
  const buttonWidth = buttonRect ? buttonRect.width : 32
  const availableWidth = containerRect.width - buttonWidth - 64 // 64px for container padding

  // If text is wider than available space, reduce font size proportionally
  if (text.scrollWidth > availableWidth) {
    const ratio = availableWidth / text.scrollWidth
    const newSize = Math.max(0.8, ratio * 3.5)
    text.style.fontSize = `${newSize}rem`

    // Only allow wrapping when we've hit the minimum font size
    if (newSize <= 0.8) {
      text.style.whiteSpace = 'normal'
    }
  }
}

const handleGlobalMouseMove = (event: MouseEvent) => {
  if (!isLengthInputVisible.value || !lengthInputRef.value) return

  const rect = lengthInputRef.value.getBoundingClientRect()
  const centerX = rect.left + rect.width / 2
  const centerY = rect.top + rect.height / 2

  const distance = Math.sqrt(
    Math.pow(event.clientX - centerX, 2) +
    Math.pow(event.clientY - centerY, 2)
  )

  if (distance > MOUSE_DISTANCE_THRESHOLD) {
    isLengthInputVisible.value = false
  }
}

const toggleLengthInput = () => {
  isLengthInputVisible.value = !isLengthInputVisible.value
  if (isLengthInputVisible.value) {
    // Focus the input after it's rendered
    nextTick(() => {
      lengthInputRef.value?.focus()
      // Add global mouse move listener when input becomes visible
      document.addEventListener('mousemove', handleGlobalMouseMove)
    })
  } else {
    // Remove global mouse move listener when input is hidden
    document.removeEventListener('mousemove', handleGlobalMouseMove)
  }
}

onMounted(() => {
  generatePassword()
  const resizeObserver = new ResizeObserver(() => {
    requestAnimationFrame(adjustFontSize)
  })
  if (containerRef.value) {
    resizeObserver.observe(containerRef.value)
  }

  // Initial font size adjustment
  requestAnimationFrame(adjustFontSize)

  // Clean up observer on component unmount
  onBeforeUnmount(() => {
    resizeObserver.disconnect()
    // Clean up the event listener when component is unmounted
    document.removeEventListener('mousemove', handleGlobalMouseMove)
  })
})

watch(settings, () => {
  generatePassword()
}, { deep: true })

watch(password, () => {
  // Adjust font size after password changes
  requestAnimationFrame(adjustFontSize)
})
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Roboto+Mono:wght@400&family=Inter:wght@300&display=swap');

.password-generator {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
  box-sizing: border-box;
  padding: 2rem 0;
  position: relative;
}

.password-display {
  margin-bottom: 2rem;
  display: flex;
  align-items: center;
  width: 100%;
  padding: 0 2vw;
  box-sizing: border-box;
  position: relative;
  height: 5rem;
  min-height: unset;
  z-index: 1;
}

.password-text {
  font-size: clamp(0.8rem, 8vw, 3.5rem);
  font-family: 'Roboto Mono', monospace;
  color: #ffffff;
  white-space: nowrap;
  font-weight: 500;
  line-height: 1.2;
  flex: 1;
  text-align: center;
  word-break: break-all;
  overflow-wrap: break-word;
  max-width: 100%;
}

.button-container {
  display: flex;
  gap: 1rem;
  margin: 0 auto 2rem auto;
  position: relative;
  z-index: 2;
}

.action-button {
  background: rgba(26, 26, 26, 0.90);
  border: none;
  color: rgba(255, 255, 255, 0.5);
  cursor: pointer;
  padding: 0.5rem;
  transition: all 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.25rem;
  width: auto;
  height: 2.5rem;
}

.action-text {
  font-family: 'Inter', sans-serif;
  font-weight: 300;
  font-size: 1.75rem;
  letter-spacing: 0.5px;
  text-transform: uppercase;
  line-height: 1;
}

.refresh-icon {
  width: 32px;
  height: 32px;
}

.copy-icon,
.check-icon {
  width: 28px;
  height: 28px;
}

.action-button:hover {
  color: rgba(255, 255, 255, 0.8);
}

.settings-box {
  background: rgba(46, 46, 46, 0.90);
  padding: 0.1rem 0.6rem 0 0.6rem;
  border-radius: 0;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  border: 1px solid #fff;
  width: fit-content;
  min-width: unset;
  margin: 0 auto;
  max-width: 90%;
  position: relative;
  z-index: 2;
}

.setting {
  margin-bottom: 0.5rem;
  color: #fff;
  width: 100%;
  box-sizing: border-box;
}

.setting label {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  cursor: pointer;
  padding: 0.1rem 0;
  border-radius: 4px;
  transition: box-shadow 0.2s ease;
}

.setting label:hover {
  box-shadow: 0 0 0 1px rgba(224, 224, 224, 0.2);
}

.setting input[type="checkbox"] {
  width: 1.2rem;
  height: 1.2rem;
  accent-color: #bdbdbd;
  border-radius: 0;
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  border: 1px solid #bdbdbd;
  background: transparent;
  cursor: pointer;
  position: relative;
  transition: all 0.2s;
}

.setting input[type="checkbox"]:checked {
  background: #bdbdbd;
}

.setting input[type="checkbox"]:checked::after {
  content: '';
  position: absolute;
  left: 5px;
  top: 3px;
  width: 4px;
  height: 8px;
  border: solid #1a1a1a;
  border-width: 0 2px 2px 0;
  transform: rotate(45deg);
}

.setting input[type="checkbox"]:hover {
  border-color: #e0e0e0;
}

.setting input[type="checkbox"]:focus {
  outline: none;
}

.setting input[type="number"] {
  width: 4rem;
  padding: 0.3rem;
  border: 1px solid #2c3e50;
  border-radius: 4px;
  background: #2c3e50;
  color: #fff;
}

.length-setting {
  margin-top: 1rem;
  display: flex;
  flex-direction: column;
  align-items: stretch;
  gap: 0.2rem;
}

.length-label-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 0.95rem;
  color: #bdbdbd;
  margin-bottom: -0.2em;
  letter-spacing: 1px;
}

.length-label {
  font-weight: 500;
  text-transform: uppercase;
}

.length-value {
  font-family: inherit;
  font-size: 0.95rem;
  color: #bdbdbd;
  font-weight: 500;
}

.length-minmax {
  display: flex;
  justify-content: space-between;
  font-size: 0.85rem;
  color: #bdbdbd;
  margin-top: -0.4em;
  padding: 0;
}

.slider {
  width: 100%;
  min-width: 80px;
  accent-color: #e0e0e0;
  height: 6px;
  border-radius: 3px;
  background: linear-gradient(to right, #444 0%, #444 var(--percent, 50%), #e0e0e0 var(--percent, 50%), #e0e0e0 100%);
  margin-top: 4px;
}

/* Chrome, Safari, Edge */
.slider::-webkit-slider-runnable-track {
  height: 6px;
  border-radius: 3px;
  background: linear-gradient(to right, #444 0%, #444 calc(var(--percent, 50%)), #e0e0e0 calc(var(--percent, 50%)), #e0e0e0 100%);
}

.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 14px;
  height: 14px;
  border-radius: 2px;
  background: #e0e0e0;
  cursor: pointer;
  transition: background 0.2s;
  margin-top: -4px;
  position: relative;
  z-index: 2;
}

/* Firefox */
.slider::-moz-range-thumb {
  width: 14px;
  height: 14px;
  border-radius: 2px;
  background: #e0e0e0;
  cursor: pointer;
  transition: background 0.2s;
  position: relative;
  z-index: 2;
}

.slider::-moz-range-progress {
  background-color: #444;
  height: 6px;
  border-radius: 3px;
}

.slider::-moz-range-track {
  background-color: #e0e0e0;
  height: 6px;
  border-radius: 3px;
}

/* IE */
.slider::-ms-fill-lower {
  background: #444;
  border-radius: 3px;
}

.slider::-ms-fill-upper {
  background: #e0e0e0;
  border-radius: 3px;
}

.slider:focus {
  outline: none;
}

.length-input {
  width: 60px;
  background: transparent;
  border: 1px solid #bdbdbd;
  color: #bdbdbd;
  font-size: 0.95rem;
  padding: 2px 4px;
  border-radius: 4px;
  text-align: center;
  font-family: inherit;
}

.length-input:focus {
  outline: none;
  border-color: #e0e0e0;
}

.required-chars {
  margin-top: 0.5rem;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
  padding-top: 0.5rem;
  width: 100%;
  box-sizing: border-box;
}

.required-chars label {
  display: flex;
  flex-direction: column;
  width: 100%;
}

.required-chars-input {
  width: 100%;
  display: block;
  background: transparent;
  border: 1px solid #bdbdbd;
  color: #bdbdbd;
  font-size: 0.95rem;
  padding: 4px 8px;
  border-radius: 4px;
  margin-top: 0.5rem;
  font-family: inherit;
  box-sizing: border-box;
}
</style>