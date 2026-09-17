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
  frame: string
  className: string
}

const devices: Device[] = [
  {
    id: 'desktop',
    name: 'Desktop',
    width: 1440,
    height: 900,
    frame: desktopFrame,
    className: 'device-desktop',
  },
  {
    id: 'laptop',
    name: 'Laptop',
    width: 1280,
    height: 800,
    frame: laptopFrame,
    className: 'device-laptop',
  },
  {
    id: 'tablet',
    name: 'iPad',
    width: 768,
    height: 1024,
    frame: ipadFrame,
    className: 'device-tablet',
  },
  {
    id: 'mobile',
    name: 'iPhone',
    width: 390,
    height: 844,
    frame: iphoneFrame,
    className: 'device-mobile',
  },
]

const inputUrl = ref('https://example.com')
const pageUrl = ref('https://example.com')
const hoveredDeviceId = ref<DeviceId | null>(null)
const selectedDeviceId = ref<DeviceId | null>(null)

const selectedDevice = computed(() =>
  devices.find((device) => device.id === selectedDeviceId.value) ?? null,
)

function normalizeUrl(value: string): string {
  const trimmed = value.trim()

  if (!trimmed) {
    return ''
  }

  return /^https?:\/\//i.test(trimmed)
    ? trimmed
    : `https://${trimmed}`
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
            Preview your website across every important screen size.
            Click a device to open a larger view and take your own screenshot.
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

      <section
        v-if="selectedDevice"
        class="focused-view"
        aria-label="Focused device preview"
      >
        <div class="section-heading">
          <div>
            <p class="eyebrow">Focused preview</p>

            <h2>
              {{ selectedDevice.name }}
              ·
              {{ selectedDevice.width }} × {{ selectedDevice.height }}
            </h2>
          </div>

          <div class="heading-actions">
            <button class="outline-button" type="button" @click="openInNewTab">
              Open in new tab ↗
            </button>

            <button
              class="outline-button"
              type="button"
              @click="closeFocusedPreview"
            >
              View all devices
            </button>
          </div>
        </div>

        <div class="focused-stage">
          <div
            class="focused-device"
            :class="selectedDevice.className"
          >
            <iframe
              :src="pageUrl"
              :title="`${selectedDevice.name} focused website preview`"
            ></iframe>

            <img
              :src="selectedDevice.frame"
              :alt="`${selectedDevice.name} device frame`"
            />
          </div>
        </div>
      </section>

      <section
        v-else
        class="device-section"
        aria-label="Device previews"
      >
        <div class="section-heading">
          <div>
            <p class="eyebrow">Device matrix</p>
            <h2>One page. Every perspective.</h2>
          </div>

          <span class="section-help">
            Hover to inspect · Click to focus
          </span>
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
            <!-- The iframe must be above the PNG frame. -->
            <span class="device-preview-window">
              <iframe
                :src="pageUrl"
                :title="`${device.name} website preview`"
                tabindex="-1"
              ></iframe>
            </span>

            <!-- The PNG supplies the bezel and device body. -->
            <img
              class="device-frame-image"
              :src="device.frame"
              :alt="`${device.name} device frame`"
            />

            <span class="device-caption">
              {{ device.name }}
              {{ device.width }} × {{ device.height }}
            </span>
          </button>
        </div>
      </section>
    </div>
  </main>
</template>