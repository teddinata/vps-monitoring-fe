<!-- Dashboard.vue -->
<template>
  <div class="min-h-screen bg-gradient-to-br from-slate-900 via-blue-900 to-slate-900">
    <div class="container mx-auto px-4 py-8">
      <!-- Header -->
      <div class="mb-8">
        <h1 class="text-4xl font-bold text-white mb-2">VPS Monitor</h1>
        <p class="text-blue-300">Real-time server metrics monitoring</p>
      </div>

      <!-- Connection Status -->
      <div :class="[
        'mb-6 px-4 py-2 rounded-lg inline-flex items-center',
        connectionStatus === 'connected' ? 'bg-green-500/20 text-green-300' : 'bg-red-500/20 text-red-300'
      ]">
        <div class="w-2 h-2 rounded-full mr-2"
          :class="connectionStatus === 'connected' ? 'bg-green-400 animate-pulse' : 'bg-red-400'">
        </div>
        {{ connectionStatus === 'connected' ? 'Connected' : 'Disconnected' }}
      </div>

      <!-- Tambahkan di bagian System Info -->
      <div class="bg-slate-800/50 backdrop-blur rounded-xl p-6 border border-blue-500/20 mb-8">
        <h3 class="text-lg font-semibold text-white mb-4">System Information</h3>
        <div class="grid grid-cols-2 gap-4">
          <div>
            <p class="text-blue-300">Operating System</p>
            <p class="text-white font-medium">{{ metrics.system.os }}</p>
          </div>
          <div>
            <p class="text-blue-300">CPU Model</p>
            <p class="text-white font-medium">{{ metrics.system.cpuModel }}</p>
          </div>
          <div>
            <p class="text-blue-300">Total Cores</p>
            <p class="text-white font-medium">{{ metrics.system.totalCores }} / {{ metrics.system.maxCores }}</p>
          </div>
          <div>
            <p class="text-blue-300">Last Update</p>
            <p class="text-white font-medium">{{ lastUpdateFormatted }}</p>
          </div>
        </div>
      </div>

      <!-- Update CPU Card -->
      <div class="bg-slate-800/50 backdrop-blur rounded-xl p-6 border border-blue-500/20 mb-4">
        <div class="flex justify-between items-start mb-4">
          <div>
            <h3 class="text-lg font-semibold text-white">CPU Usage</h3>
            <p class="text-blue-300 text-sm">Current Load</p>
          </div>
          <div class="text-2xl font-bold text-white">
            {{ cpuUsageFormatted }}%
          </div>
        </div>
        <div class="space-y-2">
          <div class="relative h-3 bg-slate-700 rounded-full overflow-hidden">
            <div class="absolute inset-y-0 left-0 bg-gradient-to-r from-blue-500 to-indigo-500 transition-all duration-500"
              :style="{ width: `${cpuUsageFormatted}%` }">
            </div>
          </div>
          <div class="text-xs text-blue-300">
            Raw Value: {{ metrics.cpu.usage }}%
          </div>
        </div>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
        <!-- Individual CPU Cores -->
        <template v-for="(core, index) in cpuCores" :key="index">
          <div class="bg-slate-800/50 backdrop-blur rounded-xl p-6 border border-blue-500/20">
            <div class="flex justify-between items-start mb-4">
              <div>
                <h3 class="text-lg font-semibold text-white">{{ core.core }}</h3>
                <p class="text-blue-300 text-sm">Core Usage</p>
              </div>
              <div class="text-2xl font-bold text-white">{{ core.usage }}%</div>
            </div>
            <div class="space-y-2">
              <div class="relative h-2 bg-slate-700 rounded-full overflow-hidden">
                <div class="absolute inset-y-0 left-0 bg-gradient-to-r from-blue-500 to-indigo-500 transition-all duration-500"
                  :style="{ width: `${core.usage}%` }">
                </div>
              </div>
              <div class="grid grid-cols-3 gap-2 text-xs">
                <div class="text-blue-300">
                  <span class="block">User</span>
                  <span class="text-white">{{ core.user }}%</span>
                </div>
                <div class="text-blue-300">
                  <span class="block">System</span>
                  <span class="text-white">{{ core.system }}%</span>
                </div>
                <div class="text-blue-300">
                  <span class="block">IO Wait</span>
                  <span class="text-white">{{ core.iowait }}%</span>
                </div>
              </div>
            </div>
          </div>
        </template>
      </div>

      <!-- Memory & Disk Section -->
      <div class="grid grid-cols-1 md:grid-cols-2 gap-6 mb-8">
        <!-- Memory Card -->
        <div class="bg-slate-800/50 backdrop-blur rounded-xl p-6 border border-blue-500/20">
          <h3 class="text-lg font-semibold text-white mb-4">Memory Usage</h3>
          <div class="flex items-center justify-between mb-4">
            <div class="text-blue-300">
              <div class="text-3xl font-bold text-white">{{ memoryUsedGB }} GB</div>
              <div class="text-sm">of {{ memoryTotalGB }} GB Used</div>
              <div class="text-xs mt-1">Raw: {{ memoryUsageRaw }}</div>
            </div>
            <div class="text-right">
              <div class="text-2xl font-bold text-white">{{ memoryUsagePercent }}%</div>
              <div class="text-sm text-blue-300">Usage</div>
              <div class="text-xs text-blue-300">{{ memoryFreeGB }} GB Free</div>
            </div>
          </div>
          <div class="relative h-4 bg-slate-700 rounded-full overflow-hidden">
            <div class="absolute inset-y-0 left-0 bg-gradient-to-r from-green-500 to-emerald-500 transition-all duration-500"
              :style="{ width: `${memoryUsagePercent}%` }">
            </div>
          </div>
        </div>

        <!-- Disk Card -->
        <div class="bg-slate-800/50 backdrop-blur rounded-xl p-6 border border-blue-500/20">
          <h3 class="text-lg font-semibold text-white mb-4">Disk Usage</h3>
          <div class="flex items-center justify-between mb-4">
            <div class="text-blue-300">
              <div class="text-3xl font-bold text-white">{{ metrics.disk?.used }}</div>
              <div class="text-sm">of {{ metrics.disk?.total }} Used</div>
              <div class="text-xs mt-1">Free: {{ metrics.disk?.free }}</div>
            </div>
            <div class="text-right">
              <div class="text-2xl font-bold text-white">{{ metrics.disk?.usage }}</div>
              <div class="text-sm text-blue-300">Usage</div>
            </div>
          </div>
          <div class="relative h-4 bg-slate-700 rounded-full overflow-hidden">
            <div class="absolute inset-y-0 left-0 bg-gradient-to-r from-purple-500 to-pink-500 transition-all duration-500"
              :style="{ width: metrics.disk?.usage.replace('%', '') + '%' }">
            </div>
          </div>
        </div>
      </div>

      <!-- Process Table -->
      <div class="bg-slate-800/50 backdrop-blur rounded-xl p-6 border border-blue-500/20">
        <div class="flex justify-between items-center mb-4">
          <h3 class="text-lg font-semibold text-white">Active Processes</h3>
          <div class="text-sm text-blue-300">Last Update: {{ lastUpdateFormatted }}</div>
        </div>
        <div class="overflow-x-auto">
          <table class="w-full">
            <thead>
              <tr class="text-blue-300 border-b border-blue-500/20">
                <th class="px-4 py-3 text-left">PID</th>
                <th class="px-4 py-3 text-left">User</th>
                <th class="px-4 py-3 text-left">CPU %</th>
                <th class="px-4 py-3 text-left">Memory %</th>
                <th class="px-4 py-3 text-left">Command</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="process in metrics.processes" :key="process.pid"
                class="text-white border-b border-blue-500/10 hover:bg-slate-700/30">
                <td class="px-4 py-3">{{ process.pid }}</td>
                <td class="px-4 py-3">{{ process.user }}</td>
                <td class="px-4 py-3">{{ process.cpu }}%</td>
                <td class="px-4 py-3">{{ process.memory }}%</td>
                <td class="px-4 py-3 font-mono text-sm">{{ process.command }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Dashboard',
  data() {
    return {
      metrics: {
        timestamp: null,
        system: {
          os: '',
          cpuModel: '',
          totalCores: 0,
          maxCores: 0
        },
        cpu: {
          usage: '0'
        },
        memory: {
          total: 0,
          used: 0,
          free: 0
        },
        disk: {
          total: '0',
          used: '0',
          free: '0',
          usage: '0%'
        },
        processes: []
      },
      ws: null,
      connectionStatus: 'disconnected',
      lastUpdate: null
    }
  },
  computed: {
    // Memory Calculations
    memoryTotalGB() {
      return (this.metrics.memory.total / 1024).toFixed(2);
    },
    memoryUsedGB() {
      return (this.metrics.memory.used / 1024).toFixed(2);
    },
    memoryFreeGB() {
      return (this.metrics.memory.free / 1024).toFixed(2);
    },
    memoryUsagePercent() {
      return ((this.metrics.memory.used / this.metrics.memory.total) * 100).toFixed(1);
    },
    memoryUsageRaw() {
      return `${this.metrics.memory.used}MB / ${this.metrics.memory.total}MB`;
    },

    // CPU Usage Details
    cpuUsageFormatted() {
      return parseFloat(this.metrics.cpu.usage).toFixed(1);
    },

    // Disk Calculations
    diskUsagePercent() {
      return parseInt(this.metrics.disk.usage);
    },
    diskDetailsRaw() {
      return `${this.metrics.disk.used} / ${this.metrics.disk.total} (${this.metrics.disk.free} free)`;
    },

    // Timestamp Formatting
    lastUpdateFormatted() {
      if (!this.metrics.timestamp) return '-';
      const date = new Date(this.metrics.timestamp);
      return date.toLocaleString();
    },
    cpuCores() {
      if (!this.metrics.cpu?.cores) return [];
      // Filter hanya CPU 0-3 dan hapus CPU all dan CPU CPU
      return this.metrics.cpu.cores.filter(core => 
        core.core.includes('CPU ') && 
        !core.core.includes('CPU CPU') && 
        !core.core.includes('CPU all')
      );
    }
  },
  methods: {
    connectWebSocket() {
      try {
        this.ws = new WebSocket('wss://server.barengan.shop');
        
        this.ws.onopen = () => {
          this.connectionStatus = 'connected';
          console.log('Connected to WebSocket');
        };

        this.ws.onmessage = (event) => {
          try {
            this.metrics = JSON.parse(event.data);
          } catch (error) {
            console.error('Error parsing message:', error);
          }
        };

        this.ws.onclose = () => {
          this.connectionStatus = 'disconnected';
          setTimeout(() => this.connectWebSocket(), 5000);
        };

        this.ws.onerror = (error) => {
          console.error('WebSocket error:', error);
          this.connectionStatus = 'error';
        };
      } catch (error) {
        console.error('Error setting up WebSocket:', error);
        this.connectionStatus = 'error';
      }
    }
  },
  mounted() {
    this.connectWebSocket();
  },
}
</script>

<style scoped>
.metric-card {
  @apply bg-white overflow-hidden shadow-sm rounded-lg p-6;
}

.metric-title {
  @apply text-sm font-medium text-gray-500 truncate;
}

.metric-value {
  @apply mt-1 text-3xl font-semibold text-gray-900;
}

.progress-bar {
  @apply h-3 rounded-full transition-all duration-500;
}

.connection-status {
  @apply text-sm font-medium px-2 py-1 rounded;
}

.connection-status.connected {
  @apply bg-green-100 text-green-800;
}

.connection-status.disconnected {
  @apply bg-red-100 text-red-800;
}

.connection-status.error {
  @apply bg-yellow-100 text-yellow-800;
}
</style>