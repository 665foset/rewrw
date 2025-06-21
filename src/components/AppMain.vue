<template>
  <main class="main-content">
    <div class="container main-card">
      <h2 class="section-title">AC Control Dashboard</h2>
      <p class="intro-text">
        Monitor and control your air conditioning system from anywhere. Adjust temperature, view current mode, and manage devices with ease.
      </p>

      <div class="ac-control-section">
        <div class="status-display">
          <div class="status-item current-temp-display">
            <p class="status-label">Current Temperature:</p>
            <p class="status-value">{{ currentTemp.toFixed(1) }}°C</p>
          </div>
          <div class="status-item target-temp-display">
            <p class="status-label">Target Temperature:</p>
            <p class="status-value">{{ targetTemp.toFixed(1) }}°C</p>
          </div>
          <div class="status-item mode-display">
            <p class="status-label">Mode:</p>
            <p class="status-value">{{ currentMode }}</p>
          </div>
          <div class="status-item device-status-display">
            <p class="status-label">Device Status:</p>
            <p class="status-value">{{ displayedDeviceStatus }}</p>
          </div>
          <div class="status-item device-id-display">
            <p class="status-label">Device ID:</p>
            <p class="status-value">{{ selectedDeviceId }}</p>
          </div>
        </div>

        <div class="control-panel">
          <div class="device-selection-controls">
            <p class="control-label">Select Device:</p>
            <div class="device-dropdown-group">
              <div v-if="loadingDevice" class="loading-indicator">Loading...</div>
              <Dropdown v-else v-model="selectedDeviceId" :options="deviceIds" placeholder="Select a Device" class="device-dropdown" />
            </div>
            <div v-if="errorDevice" class="error-message">Error: {{ errorDevice }}</div>
          </div>

          <div class="temp-controls">
            <p class="control-label">Set Temperature:</p>
            <div class="temp-slider-group">
              <button @click="decreaseTemp" class="temp-button minus-button">-</button>
              <input
                  type="range"
                  v-model.number="targetTemp"
                  min="16"
                  max="30"
                  step="0.5"
                  class="temp-slider"
                  :style="{ background: sliderBackground }"
              />
              <button @click="increaseTemp" class="temp-button plus-button">+</button>
            </div>
            <div class="save-button-container">
              <Button label="Save Temperature" icon="pi pi-save" @click="saveTemperature" :disabled="loadingSave" class="p-button-primary save-temp-button" />
              <div v-if="loadingSave" class="loading-indicator">Saving...</div>
              <div v-if="errorSave" class="error-message">Save Error: {{ errorSave }}</div>
              <div v-if="successSave" class="success-message">Temperature saved!</div>
            </div>
          </div>
        </div>
      </div>

      <p class="outro-text">
        This interactive dashboard allows you to control the temperature and switch between different AC units, while displaying the current operating mode.
      </p>
    </div>
  </main>
</template>

<script setup>
import { ref, watch, onMounted, computed, onUnmounted } from 'vue';
import Dropdown from 'primevue/dropdown';
import Button from 'primevue/button';

const API_BASE_URL = import.meta.env.VITE_API_BASE_URL || '/api';

const currentTemp = ref(25.0);
const targetTemp = ref(22.0);
const currentMode = ref('N/A');
const deviceIds = ref(null);
const deviceStatuses = ref([]);
const selectedDeviceId = ref(null);

const loadingDevice = ref(false);
const errorDevice = ref(null);
const loadingSave = ref(false);
const errorSave = ref(null);
const successSave = ref(false);

let refreshInterval = null;

const displayedDeviceStatus = computed(() => {
  if (selectedDeviceId.value && deviceStatuses.value) {
    const device = deviceStatuses.value.find(d => d.deviceId === selectedDeviceId.value);
    return device ? device.status : 'Unknown';
  }
  return 'N/A';
});

const fetchDeviceIDs = async () => {
  loadingDevice.value = true;
  errorDevice.value = null;
  try {
    const response = await fetch(`${API_BASE_URL}/Devices`);
    if (!response.ok) {
        throw new Error(`HTTP Error: ${response.status} ${response.statusText}`);
    } else {
      const data = await response.json();
      deviceIds.value = data.map(device => device.deviceId);
      deviceStatuses.value = data.map(device => ({
        deviceId: device.deviceId,
        status: device.connectionState,
      }));
      if (deviceIds.value.length > 0 && !selectedDeviceId.value) {
        selectedDeviceId.value = deviceIds.value[0];
      }
    }
  } catch (error) {
    console.error("Error fetching device IDs:", error);
    errorDevice.value = error.message;
    deviceIds.value = [];
    selectedDeviceId.value = null;
  } finally {
    loadingDevice.value = false;
  }
};

const fetchDeviceThreshold = async (deviceId) => {
  if (!deviceId) {
      targetTemp.value = 22.0;
      return;
  }
  try {
    const response = await fetch(`${API_BASE_URL}/Thresholds/${deviceId}`);
    if (!response.ok) {
      if (response.status === 404) {
        targetTemp.value = 22.0;
        console.warn(`No threshold found for device ${deviceId}. Default temperature set.`);
      } else {
        throw new Error(`HTTP Error: ${response.status} ${response.statusText}`);
      }
    } else {
      const data = await response.json();
      targetTemp.value = parseFloat(data.temperature.toFixed(1));
    }
  } catch (error) {
    console.error("Error fetching device threshold:", error);
    errorDevice.value = error.message;
    targetTemp.value = 22.0;
  }
};

const fetchDeviceData = async (deviceId) => {
  if (!deviceId) {
    currentTemp.value = 0.0;
    currentMode.value = 'N/A';
    return;
  }
  loadingDevice.value = true;
  errorDevice.value = null;
  try {
    const response = await fetch(`${API_BASE_URL}/Devices/${deviceId}/status`);
    if (!response.ok) {
      throw new Error(`HTTP Error: ${response.status} ${response.statusText}`);
    } else {
      const data = await response.json();
      currentMode.value = data.coolingStatus || 'N/A';
      currentTemp.value = parseFloat(data.temperature.toFixed(1));
      const deviceIndex = deviceStatuses.value.findIndex(d => d.deviceId === deviceId);
      if (deviceIndex !== -1) {
          deviceStatuses.value[deviceIndex].status = data.connectionState;
      }
    }
  } catch (error) {
    console.error("Error fetching device data:", error);
    errorDevice.value = error.message;
    currentTemp.value = 0.0;
    currentMode.value = 'N/A';
  } finally {
    loadingDevice.value = false;
  }
};

const startDataRefresh = () => {
    if (refreshInterval) {
        clearInterval(refreshInterval);
    }
    if (selectedDeviceId.value) {
        fetchDeviceData(selectedDeviceId.value);
        refreshInterval = setInterval(() => {
            if (selectedDeviceId.value) {
                fetchDeviceData(selectedDeviceId.value);
            }
        }, 5000);
    }
};

const saveTemperature = async () => {
  loadingSave.value = true;
  errorSave.value = null;
  successSave.value = false;
  try {
    const payload = {
      deviceId: selectedDeviceId.value,
      temperature: targetTemp.value,
    };
    const response = await fetch(`${API_BASE_URL}/Thresholds`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(payload),
    });

    if (!response.ok) {
      const errorText = await response.text();
      throw new Error(`Temperature save error: ${response.status} ${response.statusText} - ${errorText}`);
    }

    const result = await response.json();
    console.log("Temperature saved successfully:", result);
    successSave.value = true;
    setTimeout(() => successSave.value = false, 3000);
  } catch (error) {
    console.error("Error saving temperature:", error);
    errorSave.value = error.message;
  } finally {
    loadingSave.value = false;
  }
};

const increaseTemp = () => {
  if (targetTemp.value < 30) {
    targetTemp.value = parseFloat((targetTemp.value + 0.5).toFixed(1));
  }
};

const decreaseTemp = () => {
  if (targetTemp.value > 16) {
    targetTemp.value = parseFloat((targetTemp.value - 0.5).toFixed(1));
  }
};

const sliderBackground = computed(() => {
  const min = 16;
  const max = 30;
  const percentage = ((targetTemp.value - min) / (max - min)) * 100;
  return `linear-gradient(to right, #3b82f6 0%, #3b82f6 ${percentage}%, #d1d5db ${percentage}%, #d1d5db 100%)`;
});

watch(selectedDeviceId, (newDeviceId) => {
  if (newDeviceId) {
    fetchDeviceThreshold(newDeviceId);
    startDataRefresh();
  } else {
    if (refreshInterval) {
        clearInterval(refreshInterval);
        refreshInterval = null;
    }
  }
}, { immediate: true });

onMounted(() => {
  fetchDeviceIDs();
});

onUnmounted(() => {
  if (refreshInterval) {
    clearInterval(refreshInterval);
  }
});
</script>

<style scoped>
.main-content {
  flex-grow: 1;
  padding: 2rem;
  margin-top: 2rem;
  margin-bottom: 2rem;
}

.main-card {
  padding: 2rem;
  background-color: white;
  border-radius: 0.75rem;
  box-shadow: 0 10px 15px rgba(0, 0, 0, 0.1);
}

.section-title {
  font-size: 1.875rem;
  font-weight: 700;
  color: #1f2937;
  margin-bottom: 1.5rem;
  border-bottom: 2px solid #60a5fa;
  padding-bottom: 0.5rem;
}

.intro-text, .outro-text {
  font-size: 1.125rem;
  line-height: 1.75rem;
  margin-bottom: 1.5rem;
}

.outro-text {
  margin-top: 1.5rem;
}

.ac-control-section {
  display: flex;
  flex-direction: column;
  gap: 2rem;
  padding: 1.5rem;
  background-color: #eff6ff;
  border-radius: 0.5rem;
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.06);
}

.status-display {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.status-item {
  background-color: white;
  padding: 1rem;
  border-radius: 0.5rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  text-align: center;
}

.status-label {
  font-size: 0.9rem;
  color: #4b5563;
  margin-bottom: 0.5rem;
}

.status-value {
  font-size: 1.8rem;
  font-weight: 700;
  color: #1e40af;
  margin: 0;
}

.control-panel {
  display: grid;
  grid-template-columns: 1fr;
  gap: 2rem;
}

@media (min-width: 768px) {
  .control-panel {
    grid-template-columns: 1fr 1fr;
  }
}

.control-label {
  font-size: 1.1rem;
  font-weight: 600;
  color: #1f2937;
  margin-bottom: 0.8rem;
  text-align: center;
}

@media (min-width: 768px) {
  .control-label {
    text-align: left;
  }
}

.temp-controls {
  padding: 1rem;
  background-color: white;
  border-radius: 0.5rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.temp-slider-group {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.temp-button {
  background-color: #3b82f6;
  color: white;
  border: none;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  font-size: 1.5rem;
  font-weight: bold;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.temp-button:hover {
  background-color: #2563eb;
}

.temp-slider {
  flex-grow: 1;
  -webkit-appearance: none;
  appearance: none;
  height: 8px;
  border-radius: 5px;
  outline: none;
  transition: opacity .2s;
}

.temp-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #3b82f6;
  cursor: pointer;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.temp-slider::-moz-range-thumb {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #3b82f6;
  cursor: pointer;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.save-button-container {
  margin-top: 1.5rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
}

.save-temp-button {
  width: fit-content;
  padding: 0.75rem 1.5rem;
  font-weight: 600;
  background-color: #2563eb !important;
  color: white !important;
  border-color: #2563eb !important;
  font-family: 'Nunito', sans-serif !important;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1) !important;
  transition: background-color 0.3s ease, transform 0.2s ease !important;
}

.save-temp-button:hover {
  background-color: #1d4ed8 !important;
  transform: translateY(-1px) !important;
}


.device-selection-controls {
  padding: 1rem;
  background-color: white;
  border-radius: 0.5rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  text-align: center;
}

.device-dropdown-group {
    display: flex;
    align-items: center;
    justify-content: center;
    flex-wrap: wrap;
}

@media (min-width: 768px) {
  .device-dropdown-group {
    justify-content: flex-start;
  }
}


.device-dropdown.p-dropdown {
  flex-grow: 1;
  background-color: white !important;
  color: #1f2937 !important;
  border: 1px solid #d1d5db !important;
  border-radius: 0.5rem !important;
  font-family: 'Nunito', sans-serif !important;
  box-shadow: none !important;
}

.device-dropdown :deep(.p-dropdown-label) {
    color: #1f2937 !important;
    font-family: 'Nunito', sans-serif !important;
}

.device-dropdown :deep(.p-dropdown-label.p-placeholder) {
    color: #6b7280 !important;
}

.device-dropdown :deep(.p-dropdown-trigger .p-dropdown-trigger-icon) {
    color: #4b5563 !important;
}

:deep(.p-dropdown-panel) {
  background-color: white !important;
  color: #1f2937 !important;
  border: 1px solid #d1d5db !important;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1) !important;
  font-family: 'Nunito', sans-serif !important;
  z-index: 9999 !important;
}

:deep(.p-dropdown-panel .p-dropdown-items) {
  background-color: white !important;
}

:deep(.p-dropdown-panel .p-dropdown-items .p-dropdown-item) {
  color: #1f2937 !important;
  font-family: 'Nunito', sans-serif !important;
}

:deep(.p-dropdown-panel .p-dropdown-items .p-dropdown-item:hover) {
  background-color: #e0f2fe !important;
  color: #1e40af !important;
}

:deep(.p-dropdown-panel .p-dropdown-items .p-dropdown-item.p-highlight) {
  background-color: #90cdf4 !important;
  color: white !important;
}


.confirm-id-button {
    background-color: #60a5fa !important;
    color: white !important;
    border-color: #60a5fa !important;
    font-family: 'Nunito', sans-serif !important;
    font-weight: 600;
    padding: 0.75rem 1rem !important;
    transition: background-color 0.3s ease, transform 0.2s ease !important;
}

.confirm-id-button:hover {
    background-color: #3b82f6 !important;
    transform: translateY(-1px) !important;
}


.loading-indicator {
  color: #2563eb;
  font-size: 0.9rem;
  margin-top: 0.5rem;
}

.error-message {
  color: #dc2626;
  font-size: 0.9rem;
  margin-top: 0.5rem;
}

.success-message {
  color: #16a34a;
  font-size: 0.9rem;
  margin-top: 0.5rem;
}

@media (min-width: 768px) {
  .temp-controls, .device-selection-controls {
    text-align: left;
  }
  .save-button-container {
    align-items: flex-start;
  }
}

@media (max-width: 768px) {
  .section-title {
    font-size: 1.5rem;
  }

  .intro-text, .outro-text {
    font-size: 1rem;
  }
  .status-value {
    font-size: 1.4rem;
  }
  .device-dropdown {
    width: 100%;
    max-width: none;
  }
}
</style>
