<script setup lang="ts">
import { computed, ref } from 'vue'
import ipadFrame from './images/ipad-optimised.png'
import iphoneFrame from './images/iphone-optimised.png'
import laptopFrame from './images/laptop-screen-optimised.png'
import desktopFrame from './images/large-screen-optimised.png'

type DeviceId = 'desktop' | 'laptop' | 'tablet' | 'mobile'

interface Device {
  id: DeviceId
  name: string
  width: number
  height: number
  frameWidth: number
  frameHeight: number
  frame: string
  className: string
  screenLeft: number
  screenTop: number
  scale: number
}

const devices = ref<Device[]>([
  {
    id: 'desktop',
    name: 'Desktop',
    width: 1600,
    height: 992,
    frameWidth: 566,
    frameHeight: 538,
    frame: desktopFrame,
    className: 'device-desktop',
    screenLeft: 28,
    screenTop: 38,
    scale: 0.3181,
  },
  {
    id: 'laptop',
    name: 'Laptop',
    width: 1280,
    height: 802,
    frameWidth: 477,
    frameHeight: 307,
    frame: laptopFrame,
    className: 'device-laptop',
    screenLeft: 60,
    screenTop: 26,
    scale: 0.277,
  },
  {
    id: 'tablet',
    name: 'iPad',
    width: 750,
    height: 1080,
    frameWidth: 246,
    frameHeight: 400,
    frame: ipadFrame,
    className: 'device-tablet',
    screenLeft: 22,
    screenTop: 34,
    scale: 0.27,
  },
  {
    id: 'mobile',
    name: 'iPhone',
    width: 320,
    height: 480,
    frameWidth: 95,
    frameHeight: 196,
    frame: iphoneFrame,
    className: 'device-mobile',
    screenLeft: 11,
    screenTop: 32,
    scale: 0.219,
  },
])

const inputUrl = ref('https://example.com')
const pageUrl = ref('https://example.com')
const hoveredDeviceId = ref<DeviceId | null>(null)
const selectedDeviceId = ref<DeviceId | null>(null)

const selectedDevice = computed(
  () => devices.value.find((d) => d.id === selectedDeviceId.value) ?? null,
)

/* Calibration panel */
const calibrationOpen = ref(false)
const calibTab = ref<DeviceId>('tablet')
const copied = ref(false)
const calibDevice = computed(
  () => devices.value.find((d) => d.id === calibTab.value) ?? devices.value[0],
)

async function copyCalibValues() {
  const d = calibDevice.value
  const text =
    `screenLeft: ${d.screenLeft},\n` +
    `screenTop: ${d.screenTop},\n` +
    `scale: ${d.scale.toFixed(4)},`
  try {
    await navigator.clipboard.writeText(text)
    copied.value = true
    setTimeout(() => (copied.value = false), 1500)
  } catch {
    /* noop */
  }
}

function frameStyle(device: Device) {
  return {
    width: `${device.frameWidth}px`,
    height: `${device.frameHeight}px`,
    backgroundImage: `url(${device.frame})`,
  }
}

function iframeStyle(device: Device) {
  return {
    width: `${device.width}px`,
    height: `${device.height}px`,
    top: `${device.screenTop}px`,
    left: `${device.screenLeft}px`,
    transform: `scale(${device.scale})`,
    transformOrigin: 'top left',
  }
}

function normalizeUrl(value: string): string {
  const trimmed = value.trim()
  if (!trimmed) return ''
  return /^https?:\/\//i.test(trimmed) ? trimmed : `https://${trimmed}`
}

function loadUrl(): void {
  pageUrl.value = normalizeUrl(inputUrl.value)
}

function selectDevice(device: Device): void {
  selectedDeviceId.value = device.id
}

function closeFocusedPreview(): void {
  selectedDeviceId.value = null
}

function openInNewTab(): void {
  if (pageUrl.value) {
    window.open(pageUrl.value, '_blank', 'noopener,noreferrer')
  }
}

function handleDeviceKeydown(event: KeyboardEvent, device: Device): void {
  if (event.key === 'Enter' || event.key === ' ') {
    event.preventDefault()
    selectDevice(device)
  }
}
</script>

<template>
  <main class="app-shell">
    <header class="site-header">
      <a class="brand" href="/" aria-label="Breakframe home">
        <span class="brand-mark">B</span>
        <span>breakframe</span>
      </a>
      <span class="header-status">
        <span class="status-dot"></span>
        Responsive preview
      </span>
    </header>

    <div class="page-content">
      <section class="hero">
        <div>
          <p class="eyebrow">Device preview studio</p>
          <h1>
            Break the frame.<br />
            <span>See what fits.</span>
          </h1>
          <p class="hero-copy">
            Preview your website across every important screen size. Click a device
            to open a larger view and inspect it in context.
          </p>
        </div>
        <div class="hero-note">
          <span class="command-key">⌘</span>
          Click any screen to focus it
        </div>
      </section>

      <form class="url-bar" @submit.prevent="loadUrl">
        <label class="url-field">
          <span>Website URL</span>
          <div class="input-wrap">
            <span class="input-icon" aria-hidden="true">↗</span>
            <input
              v-model="inputUrl"
              type="url"
              placeholder="https://your-website.com"
              aria-label="Website URL"
            />
          </div>
        </label>
        <button class="load-button" type="submit">
          Load preview
          <span aria-hidden="true">↗</span>
        </button>
      </form>

      <section v-if="selectedDevice" class="focused-view" aria-label="Focused device preview">
        <div class="section-heading">
          <div>
            <p class="eyebrow">Focused preview</p>
            <h2>
              {{ selectedDevice.name }} · {{ selectedDevice.width }} × {{ selectedDevice.height }}
            </h2>
          </div>
          <div class="heading-actions">
            <button class="outline-button" type="button" @click="openInNewTab">
              Open in new tab ↗
            </button>
            <button class="outline-button" type="button" @click="closeFocusedPreview">
              View all devices
            </button>
          </div>
        </div>

        <div class="focused-stage">
          <div
            class="focused-device device-frame"
            :class="[selectedDevice.className, { 'is-calibrating': calibrationOpen }]"
            :style="frameStyle(selectedDevice)"
          >
            <iframe
              v-if="pageUrl"
              :src="pageUrl"
              :title="`${selectedDevice.name} focused website preview`"
              :style="iframeStyle(selectedDevice)"
            ></iframe>
          </div>
        </div>
      </section>

      <section v-else class="device-section" aria-label="Device previews">
        <div class="section-heading">
          <div>
            <p class="eyebrow">Device matrix</p>
            <h2>One page. Every perspective.</h2>
          </div>
          <span class="section-help">Hover to inspect · Click to focus</span>
        </div>

        <div class="device-stage">
          <button
            v-for="device in devices"
            :key="device.id"
            class="device-card"
            :class="[
              device.className,
              {
                'is-hovered': hoveredDeviceId === device.id,
                'is-selected': selectedDeviceId === device.id,
              },
            ]"
            type="button"
            :aria-label="`Open ${device.name} preview, ${device.width} by ${device.height}`"
            @mouseenter="hoveredDeviceId = device.id"
            @mouseleave="hoveredDeviceId = null"
            @focus="hoveredDeviceId = device.id"
            @blur="hoveredDeviceId = null"
            @keydown="handleDeviceKeydown($event, device)"
            @click="selectDevice(device)"
          >
            <span
              class="device-frame"
              :class="{ 'is-calibrating': calibrationOpen }"
              :style="frameStyle(device)"
            >
              <iframe
                v-if="pageUrl"
                :src="pageUrl"
                :title="`${device.name} website preview`"
                :style="iframeStyle(device)"
                tabindex="-1"
                scrolling="no"
              ></iframe>
            </span>

            <span class="device-caption">
              {{ device.name }} {{ device.width }} × {{ device.height }}
            </span>
          </button>
        </div>
      </section>
    </div>

    <!-- Calibration panel -->
    <button
      class="calib-toggle"
      type="button"
      aria-label="Toggle calibration"
      @click="calibrationOpen = !calibrationOpen"
    >
      🔧
    </button>

    <div v-if="calibrationOpen" class="calib-panel">
      <header>
        <strong>Calibrate screen</strong>
        <button type="button" @click="calibrationOpen = false">×</button>
      </header>

      <div class="calib-tabs">
        <button
          v-for="d in devices"
          :key="d.id"
          type="button"
          :class="{ active: calibTab === d.id }"
          @click="calibTab = d.id"
        >
          {{ d.name }}
        </button>
      </div>

      <label>
        <span>Left</span>
        <input v-model.number="calibDevice.screenLeft" type="range" min="0" max="120" step="0.5" />
        <output>{{ calibDevice.screenLeft }}</output>
      </label>

      <label>
        <span>Top</span>
        <input v-model.number="calibDevice.screenTop" type="range" min="0" max="120" step="0.5" />
        <output>{{ calibDevice.screenTop }}</output>
      </label>

      <label>
        <span>Scale</span>
        <input v-model.number="calibDevice.scale" type="range" min="0.1" max="0.6" step="0.001" />
        <output>{{ calibDevice.scale.toFixed(3) }}</output>
      </label>

      <p class="calib-hint">
        Red outline = frame. Blue outline = iframe. Move sliders until the blue
        box sits flush on the transparent screen area of the frame PNG.
      </p>

      <button class="calib-copy" type="button" @click="copyCalibValues">
        {{ copied ? '✓ Copied' : 'Copy values' }}
      </button>
    </div>
  </main>
</template>