<script setup>
import { ref, onMounted, computed } from 'vue'

const activeState = ref(null)
const animating = ref(false)
const currentStep = ref(0)
const showPractice = ref(false)

// Practice Mode State
const practiceProcesses = ref([])
const nextProcessId = ref(1)
const selectedAlgorithm = ref('FCFS')
const timeQuantum = ref(2)
const currentTime = ref(0)
const isSimulationRunning = ref(false)
const simulationSpeed = ref(1000)
const simulationLog = ref([])
const cpuProcess = ref(null)
const completedProcesses = ref([])

// Practice Functions
const addProcess = () => {
  const newProcess = {
    id: `P${nextProcessId.value}`,
    arrivalTime: currentTime.value,
    burstTime: Math.floor(Math.random() * 5) + 1,
    remainingTime: 0,
    priority: Math.floor(Math.random() * 5) + 1,
    state: 'new',
    waitingTime: 0,
    turnaroundTime: 0
  }
  newProcess.remainingTime = newProcess.burstTime
  practiceProcesses.value.push(newProcess)
  addLog(`Process ${newProcess.id} created (Burst: ${newProcess.burstTime}, Priority: ${newProcess.priority})`)
  nextProcessId.value++
  
  // Auto move to ready after short delay
  setTimeout(() => {
    if (newProcess.state === 'new') {
      newProcess.state = 'ready'
      addLog(`Process ${newProcess.id} moved to Ready Queue`)
    }
  }, 500)
}

const addCustomProcess = (burstTime, priority) => {
  const newProcess = {
    id: `P${nextProcessId.value}`,
    arrivalTime: currentTime.value,
    burstTime: burstTime,
    remainingTime: burstTime,
    priority: priority,
    state: 'new',
    waitingTime: 0,
    turnaroundTime: 0
  }
  practiceProcesses.value.push(newProcess)
  addLog(`Process ${newProcess.id} created (Burst: ${burstTime}, Priority: ${priority})`)
  nextProcessId.value++
  
  setTimeout(() => {
    if (newProcess.state === 'new') {
      newProcess.state = 'ready'
      addLog(`Process ${newProcess.id} moved to Ready Queue`)
    }
  }, 500)
}

const addLog = (message) => {
  simulationLog.value.unshift({
    time: currentTime.value,
    message: message
  })
  if (simulationLog.value.length > 50) {
    simulationLog.value.pop()
  }
}

const readyQueue = computed(() => {
  return practiceProcesses.value.filter(p => p.state === 'ready')
})

const waitingQueue = computed(() => {
  return practiceProcesses.value.filter(p => p.state === 'waiting')
})

const getNextProcess = () => {
  const ready = readyQueue.value
  if (ready.length === 0) return null
  
  switch (selectedAlgorithm.value) {
    case 'FCFS':
      return ready[0]
    case 'SJF':
      return ready.reduce((min, p) => p.remainingTime < min.remainingTime ? p : min, ready[0])
    case 'SRT':
      return ready.reduce((min, p) => p.remainingTime < min.remainingTime ? p : min, ready[0])
    case 'Priority':
      return ready.reduce((min, p) => p.priority < min.priority ? p : min, ready[0])
    case 'RR':
      return ready[0]
    default:
      return ready[0]
  }
}

let simulationInterval = null
let quantumCounter = 0

const startSimulation = () => {
  if (isSimulationRunning.value) return
  isSimulationRunning.value = true
  quantumCounter = 0
  addLog('Simulation started')
  
  simulationInterval = setInterval(() => {
    simulationStep()
  }, simulationSpeed.value)
}

const pauseSimulation = () => {
  isSimulationRunning.value = false
  if (simulationInterval) {
    clearInterval(simulationInterval)
    simulationInterval = null
  }
  addLog('Simulation paused')
}

const simulationStep = () => {
  currentTime.value++
  
  // Move waiting processes back to ready (simulate I/O completion)
  waitingQueue.value.forEach(p => {
    if (Math.random() < 0.3) {
      p.state = 'ready'
      addLog(`Process ${p.id} I/O complete, moved to Ready Queue`)
    }
  })
  
  // Update waiting time for ready processes
  readyQueue.value.forEach(p => {
    if (p !== cpuProcess.value) {
      p.waitingTime++
    }
  })
  
  // CPU scheduling logic
  if (!cpuProcess.value) {
    const next = getNextProcess()
    if (next) {
      cpuProcess.value = next
      next.state = 'running'
      quantumCounter = 0
      addLog(`Process ${next.id} dispatched to CPU`)
    }
  } else {
    // Preemptive check for Shortest Remaining Time (SRT):
    if (selectedAlgorithm.value === 'SRT') {
      const shorter = readyQueue.value.find(p => p.remainingTime < cpuProcess.value.remainingTime)
      if (shorter) {
        cpuProcess.value.state = 'ready'
        addLog(`Process ${cpuProcess.value.id} preempted (Shorter job ${shorter.id} arrived)`)
        // Do not move the preempted process to the end; leave ordering.
        cpuProcess.value = null
        quantumCounter = 0
        return
      }
    }

    cpuProcess.value.remainingTime--
    quantumCounter++
    
    if (cpuProcess.value.remainingTime <= 0) {
      // Process completed
      cpuProcess.value.state = 'terminated'
      cpuProcess.value.turnaroundTime = currentTime.value - cpuProcess.value.arrivalTime
      completedProcesses.value.push(cpuProcess.value)
      addLog(`Process ${cpuProcess.value.id} completed (TAT: ${cpuProcess.value.turnaroundTime})`)
      cpuProcess.value = null
      quantumCounter = 0
    } else if (selectedAlgorithm.value === 'RR' && quantumCounter >= timeQuantum.value) {
      // Time quantum expired (Round Robin)
      cpuProcess.value.state = 'ready'
      addLog(`Process ${cpuProcess.value.id} preempted (Time quantum expired)`)
      // Move to end of ready queue
      const idx = practiceProcesses.value.indexOf(cpuProcess.value)
      practiceProcesses.value.splice(idx, 1)
      practiceProcesses.value.push(cpuProcess.value)
      cpuProcess.value = null
      quantumCounter = 0
    } else if (Math.random() < 0.1) {
      // Random I/O request
      cpuProcess.value.state = 'waiting'
      addLog(`Process ${cpuProcess.value.id} requested I/O, moved to Waiting`)
      cpuProcess.value = null
      quantumCounter = 0
    }
  }
  
  // Check if all processes completed
  const allCompleted = practiceProcesses.value.length > 0 && 
    practiceProcesses.value.every(p => p.state === 'terminated')
  if (allCompleted) {
    pauseSimulation()
    addLog('All processes completed!')
  }
}

const resetSimulation = () => {
  pauseSimulation()
  practiceProcesses.value = []
  completedProcesses.value = []
  cpuProcess.value = null
  currentTime.value = 0
  nextProcessId.value = 1
  simulationLog.value = []
  quantumCounter = 0
  addLog('Simulation reset')
}

const togglePractice = () => {
  showPractice.value = !showPractice.value
  if (!showPractice.value) {
    resetSimulation()
  }
}

const states = [
  { id: 'new', name: 'New', description: 'Process is being created' },
  { id: 'ready', name: 'Ready Queue', description: 'Process waiting for CPU allocation' },
  { id: 'running', name: 'Running', description: 'Process is executing on CPU' },
  { id: 'waiting', name: 'Waiting', description: 'Process waiting for I/O completion' },
  { id: 'terminated', name: 'Terminated', description: 'Process has finished execution' }
]

const algorithms = [
  { name: 'FCFS', full: 'First Come First Serve', description: 'Processes are executed in order of arrival' },
  { name: 'SJF', full: 'Shortest Job First', description: 'Process with shortest burst time executes first' },
  { name: 'SRT', full: 'Shortest Remaining Time', description: 'Process with shortest remaining time executes first' },
  { name: 'RR', full: 'Round Robin', description: 'Each process gets a fixed time quantum' }
]

const transitions = [
  { from: 'new', to: 'ready', label: 'Admitted' },
  { from: 'ready', to: 'running', label: 'Dispatch' },
  { from: 'running', to: 'terminated', label: 'Exit' },
  { from: 'running', to: 'waiting', label: 'I/O Request' },
  { from: 'running', to: 'ready', label: 'Timeout/Preempt' },
  { from: 'waiting', to: 'ready', label: 'I/O Complete' }
]

const flowSteps = [
  { state: 'new', message: '1. Process Created - Enters NEW state' },
  { state: 'ready', message: '2. Process admitted to READY QUEUE' },
  { state: 'running', message: '3. CPU Scheduler dispatches process to RUNNING' },
  { state: 'waiting', message: '4a. Process requests I/O - moves to WAITING' },
  { state: 'ready', message: '4b. I/O Complete - returns to READY QUEUE' },
  { state: 'running', message: '5. Process dispatched again to RUNNING' },
  { state: 'terminated', message: '6. Process completes - moves to TERMINATED' }
]

const startAnimation = () => {
  animating.value = true
  currentStep.value = 0
  animateStep()
}

const animateStep = () => {
  if (currentStep.value < flowSteps.length) {
    activeState.value = flowSteps[currentStep.value].state
    setTimeout(() => {
      currentStep.value++
      animateStep()
    }, 1500)
  } else {
    animating.value = false
    setTimeout(() => {
      activeState.value = null
    }, 1000)
  }
}

const highlightState = (stateId) => {
  if (!animating.value) {
    activeState.value = stateId
  }
}

const clearHighlight = () => {
  if (!animating.value) {
    activeState.value = null
  }
}
</script>

<template>
  <div class="cpu-scheduling-container">
    <!-- Header -->
    <header class="header">
      <div class="header-content">
        <div class="logo-section">
          <div class="cpu-icon">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <rect x="4" y="4" width="16" height="16" rx="2"/>
              <rect x="9" y="9" width="6" height="6"/>
              <line x1="9" y1="1" x2="9" y2="4"/>
              <line x1="15" y1="1" x2="15" y2="4"/>
              <line x1="9" y1="20" x2="9" y2="23"/>
              <line x1="15" y1="20" x2="15" y2="23"/>
              <line x1="20" y1="9" x2="23" y2="9"/>
              <line x1="20" y1="14" x2="23" y2="14"/>
              <line x1="1" y1="9" x2="4" y2="9"/>
              <line x1="1" y1="14" x2="4" y2="14"/>
            </svg>
          </div>
          <div>
            <h1>CPU Scheduling</h1>
            <p class="subtitle">Process Flow Diagram</p>
          </div>
        </div>
        <button 
          class="animate-btn" 
          @click="startAnimation" 
          :disabled="animating"
        >
          <svg viewBox="0 0 24 24" fill="currentColor" width="20" height="20">
            <path d="M8 5v14l11-7z"/>
          </svg>
          {{ animating ? 'Animating...' : 'Animate Flow' }}
        </button>
        <button 
          class="practice-btn" 
          @click="togglePractice"
        >
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" width="20" height="20">
            <path d="M14.7 6.3a1 1 0 0 0 0 1.4l1.6 1.6a1 1 0 0 0 1.4 0l3.77-3.77a6 6 0 0 1-7.94 7.94l-6.91 6.91a2.12 2.12 0 0 1-3-3l6.91-6.91a6 6 0 0 1 7.94-7.94l-3.76 3.76z"/>
          </svg>
          {{ showPractice ? 'Back to Diagram' : 'Practice Mode' }}
        </button>
      </div>
    </header>

    <!-- Practice Mode Screen -->
    <div v-if="showPractice" class="practice-mode">
      <div class="practice-header">
        <h2>🎮 CPU Scheduling Practice Mode</h2>
        <p>Create processes and watch how the CPU scheduler handles them!</p>
      </div>

      <!-- Control Panel -->
      <div class="practice-controls">
        <div class="control-group">
          <label>Algorithm:</label>
          <select
            v-model="selectedAlgorithm"
            :disabled="isSimulationRunning"
            class="algorithm-select"
            :class="'select-' + selectedAlgorithm.toLowerCase()"
          >
            <option value="FCFS">FCFS - First Come First Serve</option>
            <option value="SJF">SJF - Shortest Job First</option>
            <option value="SRT">SRT - Shortest Remaining Time</option>
            <option value="RR">RR - Round Robin</option>
          </select>
        </div>
        
        <div v-if="selectedAlgorithm === 'RR'" class="control-group">
          <label>Time Quantum:</label>
          <input type="number" v-model.number="timeQuantum" min="1" max="10" :disabled="isSimulationRunning"/>
        </div>

        <div class="control-group">
          <label>Speed (ms):</label>
          <input type="range" v-model.number="simulationSpeed" min="200" max="2000" step="100"/>
          <span>{{ simulationSpeed }}ms</span>
        </div>

        <div class="control-buttons">
          <button class="btn btn-add" @click="addProcess">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" width="18" height="18">
              <circle cx="12" cy="12" r="10"/>
              <line x1="12" y1="8" x2="12" y2="16"/>
              <line x1="8" y1="12" x2="16" y2="12"/>
            </svg>
            Add Process
          </button>
          <button class="btn btn-start" @click="startSimulation" :disabled="isSimulationRunning || practiceProcesses.length === 0">
            <svg viewBox="0 0 24 24" fill="currentColor" width="18" height="18">
              <path d="M8 5v14l11-7z"/>
            </svg>
            Start
          </button>
          <button class="btn btn-pause" @click="pauseSimulation" :disabled="!isSimulationRunning">
            <svg viewBox="0 0 24 24" fill="currentColor" width="18" height="18">
              <rect x="6" y="4" width="4" height="16"/>
              <rect x="14" y="4" width="4" height="16"/>
            </svg>
            Pause
          </button>
          <button class="btn btn-reset" @click="resetSimulation">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" width="18" height="18">
              <path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 8"/>
              <path d="M3 3v5h5"/>
            </svg>
            Reset
          </button>
        </div>
      </div>

      <!-- Time Display -->
      <div class="time-display">
        <span class="time-label">⏱️ Current Time:</span>
        <span class="time-value">{{ currentTime }}</span>
      </div>

      <!-- Visual Process States -->
      <div class="practice-visualization">
        <!-- New State -->
        <div class="practice-state new-practice-state">
          <div class="state-header">
            <div class="state-indicator new-indicator"></div>
            <h3>NEW</h3>
          </div>
          <div class="process-list">
            <div 
              v-for="p in practiceProcesses.filter(p => p.state === 'new')" 
              :key="p.id"
              class="process-block new-block"
            >
              <span class="process-id">{{ p.id }}</span>
              <span class="process-info">B:{{ p.burstTime }}</span>
            </div>
            <div v-if="practiceProcesses.filter(p => p.state === 'new').length === 0" class="empty-state">
              Empty
            </div>
          </div>
        </div>

        <!-- Arrow -->
        <div class="practice-arrow">→</div>

        <!-- Ready Queue -->
        <div class="practice-state ready-practice-state">
          <div class="state-header">
            <div class="state-indicator ready-indicator"></div>
            <h3>READY QUEUE</h3>
            <span class="queue-count">{{ readyQueue.length }}</span>
          </div>
          <div class="process-list queue-list">
            <div 
              v-for="p in readyQueue" 
              :key="p.id"
              class="process-block ready-block"
            >
              <span class="process-id">{{ p.id }}</span>
              <span class="process-info">R:{{ p.remainingTime }} P:{{ p.priority }}</span>
            </div>
            <div v-if="readyQueue.length === 0" class="empty-state">
              Empty
            </div>
          </div>
        </div>

        <!-- Arrow -->
        <div class="practice-arrow">→</div>

        <!-- CPU (Running) -->
        <div class="practice-state running-practice-state">
          <div class="state-header">
            <div class="state-indicator running-indicator"></div>
            <h3>CPU (RUNNING)</h3>
          </div>
          <div class="cpu-display">
            <div v-if="cpuProcess" class="process-block running-block active-cpu">
              <div class="cpu-animation"></div>
              <span class="process-id">{{ cpuProcess.id }}</span>
              <span class="process-info">Remaining: {{ cpuProcess.remainingTime }}</span>
            </div>
            <div v-else class="empty-cpu">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" width="40" height="40">
                <rect x="4" y="4" width="16" height="16" rx="2"/>
                <rect x="9" y="9" width="6" height="6"/>
              </svg>
              <span>Idle</span>
            </div>
          </div>
        </div>

        <!-- Arrow to Waiting -->
        <div class="practice-arrow down-arrow">↓</div>

        <!-- Waiting State -->
        <div class="practice-state waiting-practice-state">
          <div class="state-header">
            <div class="state-indicator waiting-indicator"></div>
            <h3>WAITING (I/O)</h3>
            <span class="queue-count">{{ waitingQueue.length }}</span>
          </div>
          <div class="process-list">
            <div 
              v-for="p in waitingQueue" 
              :key="p.id"
              class="process-block waiting-block"
            >
              <span class="process-id">{{ p.id }}</span>
              <div class="io-spinner"></div>
            </div>
            <div v-if="waitingQueue.length === 0" class="empty-state">
              Empty
            </div>
          </div>
        </div>

        <!-- Arrow -->
        <div class="practice-arrow">→</div>

        <!-- Terminated -->
        <div class="practice-state terminated-practice-state">
          <div class="state-header">
            <div class="state-indicator terminated-indicator"></div>
            <h3>TERMINATED</h3>
            <span class="queue-count">{{ completedProcesses.length }}</span>
          </div>
          <div class="process-list completed-list">
            <div 
              v-for="p in completedProcesses" 
              :key="p.id"
              class="process-block terminated-block"
            >
              <span class="process-id">{{ p.id }}</span>
              <span class="process-info">TAT:{{ p.turnaroundTime }} WT:{{ p.waitingTime }}</span>
            </div>
            <div v-if="completedProcesses.length === 0" class="empty-state">
              Empty
            </div>
          </div>
        </div>
      </div>

      <!-- Process Table -->
      <div class="process-table-section">
        <h3>📊 Process Table</h3>
        <div class="table-container">
          <table class="process-table">
            <thead>
              <tr>
                <th>Process</th>
                <th>Arrival</th>
                <th>Burst</th>
                <th>Remaining</th>
                <th>Priority</th>
                <th>State</th>
                <th>Waiting</th>
                <th>Turnaround</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="p in practiceProcesses" :key="p.id" :class="'row-' + p.state">
                <td><strong>{{ p.id }}</strong></td>
                <td>{{ p.arrivalTime }}</td>
                <td>{{ p.burstTime }}</td>
                <td>{{ p.remainingTime }}</td>
                <td>{{ p.priority }}</td>
                <td><span class="state-badge" :class="'badge-' + p.state">{{ p.state.toUpperCase() }}</span></td>
                <td>{{ p.waitingTime }}</td>
                <td>{{ p.state === 'terminated' ? p.turnaroundTime : '-' }}</td>
              </tr>
            </tbody>
          </table>
          <div v-if="practiceProcesses.length === 0" class="no-processes">
            No processes. Click "Add Process" to create one!
          </div>
        </div>
      </div>

      <!-- Statistics -->
      <div v-if="completedProcesses.length > 0" class="statistics-section">
        <h3>📈 Statistics</h3>
        <div class="stats-grid">
          <div class="stat-card">
            <span class="stat-label">Completed</span>
            <span class="stat-value">{{ completedProcesses.length }}</span>
          </div>
          <div class="stat-card">
            <span class="stat-label">Avg Waiting Time</span>
            <span class="stat-value">{{ (completedProcesses.reduce((sum, p) => sum + p.waitingTime, 0) / completedProcesses.length).toFixed(2) }}</span>
          </div>
          <div class="stat-card">
            <span class="stat-label">Avg Turnaround</span>
            <span class="stat-value">{{ (completedProcesses.reduce((sum, p) => sum + p.turnaroundTime, 0) / completedProcesses.length).toFixed(2) }}</span>
          </div>
          <div class="stat-card">
            <span class="stat-label">Throughput</span>
            <span class="stat-value">{{ (completedProcesses.length / currentTime).toFixed(3) }}/unit</span>
          </div>
        </div>
      </div>

      <!-- Event Log -->
      <div class="event-log-section">
        <h3>📋 Event Log</h3>
        <div class="event-log">
          <div v-for="(log, index) in simulationLog" :key="index" class="log-entry">
            <span class="log-time">[T={{ log.time }}]</span>
            <span class="log-message">{{ log.message }}</span>
          </div>
          <div v-if="simulationLog.length === 0" class="no-logs">
            Events will appear here...
          </div>
        </div>
      </div>
    </div>

    <!-- Main Diagram (shown when not in practice mode) -->
    <main v-else class="diagram-section">
      <div class="flow-diagram">
        <!-- Start Node -->
        <div class="start-end-node start">
          <span>START</span>
        </div>

        <div class="arrow-down">
          <svg viewBox="0 0 24 60" fill="none" stroke="currentColor" stroke-width="2">
            <line x1="12" y1="0" x2="12" y2="50"/>
            <polyline points="6,44 12,54 18,44"/>
          </svg>
        </div>

        <!-- New State -->
        <div 
          class="state-node new-state" 
          :class="{ active: activeState === 'new' }"
          @mouseenter="highlightState('new')"
          @mouseleave="clearHighlight"
        >
          <div class="state-icon">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <circle cx="12" cy="12" r="10"/>
              <line x1="12" y1="8" x2="12" y2="16"/>
              <line x1="8" y1="12" x2="16" y2="12"/>
            </svg>
          </div>
          <div class="state-content">
            <h3>NEW</h3>
            <p>Process Created</p>
          </div>
        </div>

        <div class="arrow-down labeled">
          <span class="arrow-label">Admitted</span>
          <svg viewBox="0 0 24 60" fill="none" stroke="currentColor" stroke-width="2">
            <line x1="12" y1="0" x2="12" y2="50"/>
            <polyline points="6,44 12,54 18,44"/>
          </svg>
        </div>

        <!-- Main Flow Area -->
        <div class="main-flow-area">
          <!-- Ready Queue -->
          <div 
            class="state-node ready-state queue-node" 
            :class="{ active: activeState === 'ready' }"
            @mouseenter="highlightState('ready')"
            @mouseleave="clearHighlight"
          >
            <div class="state-icon">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <rect x="3" y="3" width="18" height="18" rx="2"/>
                <line x1="8" y1="8" x2="16" y2="8"/>
                <line x1="8" y1="12" x2="16" y2="12"/>
                <line x1="8" y1="16" x2="12" y2="16"/>
              </svg>
            </div>
            <div class="state-content">
              <h3>READY QUEUE</h3>
              <p>Waiting for CPU</p>
            </div>
            <div class="queue-processes">
              <div class="mini-process">P1</div>
              <div class="mini-process">P2</div>
              <div class="mini-process">P3</div>
            </div>
          </div>

          <!-- Scheduler Decision -->
          <div class="scheduler-section">
            <div class="arrow-right">
              <span class="arrow-label-horiz">Scheduler<br/>Dispatch</span>
              <svg viewBox="0 0 80 24" fill="none" stroke="currentColor" stroke-width="2">
                <line x1="0" y1="12" x2="70" y2="12"/>
                <polyline points="64,6 74,12 64,18"/>
              </svg>
            </div>
            
            <div class="decision-diamond">
              <div class="diamond-content">
                <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <circle cx="12" cy="12" r="3"/>
                  <path d="M12 1v4M12 19v4M1 12h4M19 12h4"/>
                </svg>
                <span>CPU<br/>Scheduler</span>
              </div>
            </div>
          </div>

          <!-- Running State -->
          <div 
            class="state-node running-state" 
            :class="{ active: activeState === 'running' }"
            @mouseenter="highlightState('running')"
            @mouseleave="clearHighlight"
          >
            <div class="state-icon running-icon">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <rect x="4" y="4" width="16" height="16" rx="2"/>
                <rect x="9" y="9" width="6" height="6"/>
              </svg>
              <div class="pulse-ring"></div>
            </div>
            <div class="state-content">
              <h3>RUNNING</h3>
              <p>CPU Executing</p>
            </div>
          </div>

          <!-- Waiting State (below) -->
          <div 
            class="state-node waiting-state" 
            :class="{ active: activeState === 'waiting' }"
            @mouseenter="highlightState('waiting')"
            @mouseleave="clearHighlight"
          >
            <div class="state-icon">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <circle cx="12" cy="12" r="10"/>
                <polyline points="12 6 12 12 16 14"/>
              </svg>
            </div>
            <div class="state-content">
              <h3>WAITING</h3>
              <p>I/O Operations</p>
            </div>
          </div>

          <!-- Terminated State -->
          <div 
            class="state-node terminated-state" 
            :class="{ active: activeState === 'terminated' }"
            @mouseenter="highlightState('terminated')"
            @mouseleave="clearHighlight"
          >
            <div class="state-icon">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <circle cx="12" cy="12" r="10"/>
                <polyline points="9 12 11 14 15 10"/>
              </svg>
            </div>
            <div class="state-content">
              <h3>TERMINATED</h3>
              <p>Process Complete</p>
            </div>
          </div>

          <!-- Connection Lines -->
          <svg class="flow-connections" viewBox="0 0 800 400" preserveAspectRatio="none">
            <!-- Running to Terminated -->
            <path d="M 580 120 L 700 120" class="flow-line exit-line"/>
            <polygon points="695,115 705,120 695,125" class="arrow-head exit-arrow"/>
            <text x="640" y="108" class="line-label">Exit</text>
            
            <!-- Running to Waiting -->
            <path d="M 540 160 L 540 250 L 540 280" class="flow-line io-line"/>
            <polygon points="535,275 540,285 545,275" class="arrow-head io-arrow"/>
            <text x="555" y="220" class="line-label">I/O Request</text>
            
            <!-- Waiting to Ready -->
            <path d="M 460 320 L 200 320 L 200 180" class="flow-line complete-line"/>
            <polygon points="195,185 200,175 205,185" class="arrow-head complete-arrow"/>
            <text x="300" y="310" class="line-label">I/O Complete</text>
            
            <!-- Running to Ready (Timeout) -->
            <path d="M 480 100 L 350 100 L 350 120 L 280 120" class="flow-line timeout-line"/>
            <polygon points="285,115 275,120 285,125" class="arrow-head timeout-arrow"/>
            <text x="360" y="90" class="line-label">Timeout / Preempt</text>
          </svg>
        </div>

        <!-- End Node -->
        <div class="start-end-node end">
          <span>END</span>
        </div>
      </div>

      <!-- Animation Status -->
      <div v-if="animating" class="animation-status">
        <div class="status-content">
          <div class="status-icon">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <circle cx="12" cy="12" r="10"/>
              <polyline points="12 6 12 12 16 14"/>
            </svg>
          </div>
          <p>{{ flowSteps[currentStep]?.message || 'Animation Complete!' }}</p>
        </div>
        <div class="progress-bar">
          <div class="progress-fill" :style="{ width: ((currentStep + 1) / flowSteps.length * 100) + '%' }"></div>
        </div>
      </div>
    </main>

    <!-- Scheduling Algorithms Section (only show when not in practice) -->
    <section v-if="!showPractice" class="algorithms-section">
      <h2>
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M12 2L2 7l10 5 10-5-10-5z"/>
          <path d="M2 17l10 5 10-5"/>
          <path d="M2 12l10 5 10-5"/>
        </svg>
        Scheduling Algorithms
      </h2>
      <div class="algorithms-grid">
        <div
          v-for="algo in algorithms"
          :key="algo.name"
          class="algorithm-card"
          :class="`algo-card-${algo.name.toLowerCase()}`"
        >
          <div class="algo-header">
            <span :class="`algo-badge algo-badge-${algo.name.toLowerCase()}`">{{ algo.name }}</span>
            <h4>{{ algo.full }}</h4>
          </div>
          <p>{{ algo.description }}</p>
        </div>
      </div>
    </section>

    <!-- Legend Section (only show when not in practice) -->
    <section v-if="!showPractice" class="legend-section">
      <h2>
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <rect x="3" y="3" width="18" height="18" rx="2"/>
          <line x1="3" y1="9" x2="21" y2="9"/>
          <line x1="9" y1="21" x2="9" y2="9"/>
        </svg>
        Flowchart Symbols
      </h2>
      <div class="legend-grid">
        <div class="legend-item">
          <div class="legend-symbol oval"></div>
          <span>Start / End</span>
        </div>
        <div class="legend-item">
          <div class="legend-symbol rectangle"></div>
          <span>Process State</span>
        </div>
        <div class="legend-item">
          <div class="legend-symbol diamond"></div>
          <span>Decision</span>
        </div>
        <div class="legend-item">
          <div class="legend-symbol queue"></div>
          <span>Queue</span>
        </div>
        <div class="legend-item">
          <div class="legend-symbol arrow"></div>
          <span>Transition</span>
        </div>
      </div>
    </section>

    <!-- State Descriptions (only show when not in practice) -->
    <section v-if="!showPractice" class="states-section">
      <h2>
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <circle cx="12" cy="12" r="10"/>
          <line x1="12" y1="16" x2="12" y2="12"/>
          <line x1="12" y1="8" x2="12.01" y2="8"/>
        </svg>
        Process States
      </h2>
      <div class="states-grid">
        <div 
          v-for="state in states" 
          :key="state.id" 
          class="state-card"
          :class="state.id + '-card'"
          @mouseenter="highlightState(state.id)"
          @mouseleave="clearHighlight"
        >
          <h4>{{ state.name }}</h4>
          <p>{{ state.description }}</p>
        </div>
      </div>
    </section>

    <!-- Footer -->
    <footer class="footer">
      <p>CPU Process Scheduling - Operating System Concepts</p>
    </footer>
  </div>
</template>

<style scoped>
.cpu-scheduling-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #0f0f1a 0%, #1a1a2e 50%, #16213e 100%);
  color: #fff;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

/* Header Styles */
.header {
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  padding: 1.5rem 2rem;
  position: sticky;
  top: 0;
  z-index: 100;
}

.header-content {
  max-width: 1400px;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.logo-section {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.cpu-icon {
  width: 50px;
  height: 50px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 10px;
}

.cpu-icon svg {
  width: 100%;
  height: 100%;
  color: white;
}

.header h1 {
  font-size: 1.8rem;
  font-weight: 700;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin: 0;
}

.subtitle {
  color: rgba(255, 255, 255, 0.6);
  font-size: 0.9rem;
  margin: 0;
}

.animate-btn {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.8rem 1.5rem;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border: none;
  border-radius: 10px;
  color: white;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(102, 126, 234, 0.4);
}

.animate-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.6);
}

.animate-btn:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

/* Diagram Section */
.diagram-section {
  padding: 3rem 2rem;
  max-width: 1400px;
  margin: 0 auto;
}

.flow-diagram {
  background: rgba(255, 255, 255, 0.03);
  border-radius: 20px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  padding: 2rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 600px;
  position: relative;
}

/* Start/End Nodes */
.start-end-node {
  width: 120px;
  height: 50px;
  border-radius: 25px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: 0.9rem;
  letter-spacing: 1px;
}

.start-end-node.start {
  background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);
  box-shadow: 0 4px 15px rgba(56, 239, 125, 0.3);
}

.start-end-node.end {
  background: linear-gradient(135deg, #eb3349 0%, #f45c43 100%);
  box-shadow: 0 4px 15px rgba(235, 51, 73, 0.3);
  margin-top: 2rem;
}

/* Arrows */
.arrow-down {
  height: 60px;
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
}

.arrow-down svg {
  height: 60px;
  width: 24px;
  color: rgba(255, 255, 255, 0.5);
}

.arrow-down.labeled .arrow-label {
  position: absolute;
  left: 30px;
  top: 20px;
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.7);
  background: rgba(102, 126, 234, 0.2);
  padding: 2px 8px;
  border-radius: 4px;
}

/* State Nodes */
.state-node {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 1rem 1.5rem;
  border-radius: 15px;
  background: rgba(255, 255, 255, 0.05);
  border: 2px solid rgba(255, 255, 255, 0.1);
  transition: all 0.3s ease;
  cursor: pointer;
  min-width: 200px;
}

.state-node:hover, .state-node.active {
  transform: scale(1.05);
  border-color: currentColor;
  box-shadow: 0 0 30px currentColor;
}

.state-icon {
  width: 45px;
  height: 45px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}

.state-icon svg {
  width: 28px;
  height: 28px;
}

.state-content h3 {
  font-size: 1rem;
  font-weight: 700;
  margin: 0;
  letter-spacing: 0.5px;
}

.state-content p {
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.6);
  margin: 0;
}

/* State Colors */
.new-state {
  color: #38ef7d;
  border-color: rgba(56, 239, 125, 0.3);
}

.new-state .state-icon {
  background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);
}

.ready-state {
  color: #667eea;
  border-color: rgba(102, 126, 234, 0.3);
}

.ready-state .state-icon {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.running-state {
  color: #f093fb;
  border-color: rgba(240, 147, 251, 0.3);
}

.running-state .state-icon {
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
}

.running-icon .pulse-ring {
  position: absolute;
  width: 100%;
  height: 100%;
  border-radius: 12px;
  border: 2px solid #f093fb;
  animation: pulse 1.5s ease-out infinite;
}

@keyframes pulse {
  0% {
    transform: scale(1);
    opacity: 1;
  }
  100% {
    transform: scale(1.5);
    opacity: 0;
  }
}

.waiting-state {
  color: #ffecd2;
  border-color: rgba(255, 236, 210, 0.3);
}

.waiting-state .state-icon {
  background: linear-gradient(135deg, #f5af19 0%, #f12711 100%);
}

.terminated-state {
  color: #a8edea;
  border-color: rgba(168, 237, 234, 0.3);
}

.terminated-state .state-icon {
  background: linear-gradient(135deg, #a8edea 0%, #fed6e3 100%);
}

/* Queue Node */
.queue-node {
  flex-direction: column;
  text-align: center;
  padding: 1.5rem;
}

.queue-node .state-icon {
  margin-bottom: 0.5rem;
}

.queue-processes {
  display: flex;
  gap: 0.5rem;
  margin-top: 0.5rem;
}

.mini-process {
  width: 30px;
  height: 30px;
  background: rgba(102, 126, 234, 0.3);
  border: 1px solid #667eea;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.7rem;
  font-weight: 600;
}

/* Main Flow Area */
.main-flow-area {
  display: grid;
  grid-template-columns: 200px 180px 200px 200px;
  grid-template-rows: auto auto;
  gap: 1rem;
  align-items: center;
  justify-items: center;
  margin: 2rem 0;
  position: relative;
  padding: 2rem;
}

.ready-state {
  grid-column: 1;
  grid-row: 1;
}

.scheduler-section {
  grid-column: 2;
  grid-row: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.running-state {
  grid-column: 3;
  grid-row: 1;
}

.terminated-state {
  grid-column: 4;
  grid-row: 1;
}

.waiting-state {
  grid-column: 3;
  grid-row: 2;
  margin-top: 2rem;
}

/* Decision Diamond */
.decision-diamond {
  width: 80px;
  height: 80px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  transform: rotate(45deg);
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4px 15px rgba(102, 126, 234, 0.4);
}

.diamond-content {
  transform: rotate(-45deg);
  text-align: center;
  font-size: 0.6rem;
  font-weight: 600;
}

.diamond-content svg {
  width: 20px;
  height: 20px;
  margin-bottom: 2px;
}

/* Arrows */
.arrow-right {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 1rem;
}

.arrow-right svg {
  width: 80px;
  height: 24px;
  color: rgba(255, 255, 255, 0.5);
}

.arrow-label-horiz {
  font-size: 0.65rem;
  color: rgba(255, 255, 255, 0.7);
  text-align: center;
  margin-bottom: 5px;
}

/* Flow Connections SVG */
.flow-connections {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

.flow-line {
  stroke: rgba(255, 255, 255, 0.3);
  stroke-width: 2;
  fill: none;
}

.arrow-head {
  fill: rgba(255, 255, 255, 0.5);
}

.line-label {
  fill: rgba(255, 255, 255, 0.6);
  font-size: 10px;
}

.exit-line { stroke: #a8edea; }
.exit-arrow { fill: #a8edea; }

.io-line { stroke: #f5af19; }
.io-arrow { fill: #f5af19; }

.complete-line { stroke: #667eea; }
.complete-arrow { fill: #667eea; }

.timeout-line { stroke: #f093fb; }
.timeout-arrow { fill: #f093fb; }

/* Animation Status */
.animation-status {
  margin-top: 2rem;
  padding: 1.5rem;
  background: rgba(102, 126, 234, 0.1);
  border: 1px solid rgba(102, 126, 234, 0.3);
  border-radius: 15px;
  width: 100%;
  max-width: 500px;
}

.status-content {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1rem;
}

.status-icon svg {
  width: 30px;
  height: 30px;
  color: #667eea;
  animation: spin 2s linear infinite;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.status-content p {
  font-size: 1rem;
  margin: 0;
}

.progress-bar {
  height: 6px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 3px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  transition: width 0.3s ease;
}

/* Algorithms Section */
.algorithms-section {
  padding: 3rem 2rem;
  max-width: 1400px;
  margin: 0 auto;
}

.algorithms-section h2,
.legend-section h2,
.states-section h2 {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  font-size: 1.5rem;
  margin-bottom: 1.5rem;
  color: #fff;
}

.algorithms-section h2 svg,
.legend-section h2 svg,
.states-section h2 svg {
  width: 28px;
  height: 28px;
  color: #667eea;
}

.algorithms-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1.5rem;
}

.algorithm-card {
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 15px;
  padding: 1.5rem;
  transition: all 0.3s ease;
}

/* Make all algorithm cards use the FCFS gradient background */
.algorithms-grid .algorithm-card.algo-card-fcfs,
.algorithms-grid .algorithm-card.algo-card-sjf,
.algorithms-grid .algorithm-card.algo-card-priority,
.algorithms-grid .algorithm-card.algo-card-rr {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%) !important;
  border-color: transparent !important;
  color: #fff !important;
  box-shadow: 0 10px 30px rgba(102, 126, 234, 0.25) !important;
}

.algorithm-card:hover {
  transform: translateY(-5px);
  border-color: #667eea;
  box-shadow: 0 10px 30px rgba(102, 126, 234, 0.2);
}

.algo-header {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 0.75rem;
}

.algo-badge {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 0.3rem 0.8rem;
  border-radius: 20px;
  font-size: 0.75rem;
  font-weight: 700;
}

.algorithms-grid .algo-badge.algo-badge-fcfs,
.algorithms-grid .algo-badge.algo-badge-sjf,
.algorithms-grid .algo-badge.algo-badge-priority,
.algorithms-grid .algo-badge.algo-badge-rr {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%) !important;
  color: #fff !important;
}

.algo-header h4 {
  font-size: 1rem;
  margin: 0;
}

.algorithm-card p {
  color: rgba(255, 255, 255, 0.6);
  font-size: 0.9rem;
  margin: 0;
  line-height: 1.5;
}

/* Legend Section */
.legend-section {
  padding: 2rem 2rem 3rem;
  max-width: 1400px;
  margin: 0 auto;
}

.legend-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 2rem;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.legend-symbol {
  width: 40px;
  height: 25px;
}

.legend-symbol.oval {
  border-radius: 12px;
  background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);
}

.legend-symbol.rectangle {
  border-radius: 6px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.legend-symbol.diamond {
  width: 25px;
  height: 25px;
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
  transform: rotate(45deg);
}

.legend-symbol.queue {
  border-radius: 6px;
  background: repeating-linear-gradient(
    90deg,
    #667eea 0px,
    #667eea 10px,
    rgba(102, 126, 234, 0.3) 10px,
    rgba(102, 126, 234, 0.3) 12px
  );
}

.legend-symbol.arrow {
  background: linear-gradient(90deg, transparent 0%, #fff 50%, transparent 100%);
  height: 3px;
  position: relative;
}

.legend-symbol.arrow::after {
  content: '';
  position: absolute;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
  border-left: 8px solid #fff;
  border-top: 5px solid transparent;
  border-bottom: 5px solid transparent;
}

.legend-item span {
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.8);
}

/* States Section */
.states-section {
  padding: 2rem 2rem 3rem;
  max-width: 1400px;
  margin: 0 auto;
}

.states-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
}

.state-card {
  padding: 1rem 1.25rem;
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.05);
  border-left: 4px solid;
  transition: all 0.3s ease;
  cursor: pointer;
}

.state-card:hover {
  transform: translateX(5px);
  background: rgba(255, 255, 255, 0.08);
}

.state-card h4 {
  font-size: 0.95rem;
  margin: 0 0 0.25rem 0;
}

.state-card p {
  font-size: 0.8rem;
  color: rgba(255, 255, 255, 0.6);
  margin: 0;
}

.new-card { border-color: #38ef7d; color: #38ef7d; }
.ready-card { border-color: #667eea; color: #667eea; }
.running-card { border-color: #f093fb; color: #f093fb; }
.waiting-card { border-color: #f5af19; color: #f5af19; }
.terminated-card { border-color: #a8edea; color: #a8edea; }

/* Footer */
.footer {
  text-align: center;
  padding: 2rem;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
  color: rgba(255, 255, 255, 0.5);
  font-size: 0.9rem;
}

/* Responsive Design */
@media (max-width: 900px) {
  .main-flow-area {
    grid-template-columns: 1fr;
    grid-template-rows: auto;
  }

  .ready-state,
  .scheduler-section,
  .running-state,
  .terminated-state,
  .waiting-state {
    grid-column: 1;
    grid-row: auto;
  }

  .flow-connections {
    display: none;
  }

  .header-content {
    flex-direction: column;
    gap: 1rem;
    text-align: center;
  }

  .logo-section {
    flex-direction: column;
  }
}

/* Practice Button */
.practice-btn {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.8rem 1.5rem;
  background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);
  border: none;
  border-radius: 10px;
  color: white;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(56, 239, 125, 0.4);
}

.practice-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(56, 239, 125, 0.6);
}

/* Practice Mode Styles */
.practice-mode {
  padding: 2rem;
  max-width: 1400px;
  margin: 0 auto;
}

.practice-header {
  text-align: center;
  margin-bottom: 2rem;
}

.practice-header h2 {
  font-size: 2rem;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin-bottom: 0.5rem;
}

/* Styling for the algorithm select control */
.algorithm-select {
  background: rgba(255,255,255,0.03);
  color: #fff;
  border: 1px solid rgba(255,255,255,0.08);
  padding: 0.5rem 0.6rem;
  border-radius: 6px;
}
.algorithm-select.select-fcfs {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%) !important;
}
.algorithm-select.select-sjf {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%) !important;
}
.algorithm-select.select-priority {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%) !important;
}
.algorithm-select.select-rr {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%) !important;
}

/* Try to style options (limited browser control) */
.algorithm-select option {
  color: #000;
}

.practice-header p {
  color: rgba(255, 255, 255, 0.7);
}

/* Control Panel */
.practice-controls {
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 15px;
  padding: 1.5rem;
  margin-bottom: 2rem;
  display: flex;
  flex-wrap: wrap;
  gap: 1.5rem;
  align-items: center;
}

.control-group {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.control-group label {
  font-weight: 600;
  color: rgba(255, 255, 255, 0.8);
}

.control-group select,
.control-group input[type="number"] {
  padding: 0.5rem 1rem;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  background: rgba(255, 255, 255, 0.1);
  color: white;
  font-size: 0.9rem;
}

.control-group select:focus,
.control-group input:focus {
  outline: none;
  border-color: #667eea;
}

.control-group input[type="range"] {
  width: 120px;
  accent-color: #667eea;
}

.control-buttons {
  display: flex;
  gap: 0.75rem;
  margin-left: auto;
}

.btn {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.6rem 1.2rem;
  border: none;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-add {
  background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);
  color: white;
}

.btn-start {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.btn-pause {
  background: linear-gradient(135deg, #f5af19 0%, #f12711 100%);
  color: white;
}

.btn-reset {
  background: rgba(255, 255, 255, 0.1);
  color: white;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
}

/* Time Display */
.time-display {
  text-align: center;
  margin-bottom: 2rem;
  padding: 1rem;
  background: rgba(102, 126, 234, 0.1);
  border-radius: 10px;
  border: 1px solid rgba(102, 126, 234, 0.3);
}

.time-label {
  font-size: 1.1rem;
  margin-right: 1rem;
}

.time-value {
  font-size: 2rem;
  font-weight: 700;
  color: #667eea;
}

/* Practice Visualization */
.practice-visualization {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: flex-start;
  gap: 1rem;
  margin-bottom: 2rem;
  padding: 2rem;
  background: rgba(255, 255, 255, 0.03);
  border-radius: 20px;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.practice-state {
  background: rgba(255, 255, 255, 0.05);
  border-radius: 15px;
  padding: 1rem;
  min-width: 180px;
  max-width: 220px;
  border: 2px solid rgba(255, 255, 255, 0.1);
  transition: all 0.3s ease;
}

.state-header {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 1rem;
  padding-bottom: 0.5rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.state-indicator {
  width: 12px;
  height: 12px;
  border-radius: 50%;
}

.new-indicator { background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%); }
.ready-indicator { background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); }
.running-indicator { background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%); }
.waiting-indicator { background: linear-gradient(135deg, #f5af19 0%, #f12711 100%); }
.terminated-indicator { background: linear-gradient(135deg, #a8edea 0%, #fed6e3 100%); }

.state-header h3 {
  font-size: 0.85rem;
  margin: 0;
  flex-grow: 1;
}

.queue-count {
  background: rgba(255, 255, 255, 0.2);
  padding: 0.2rem 0.5rem;
  border-radius: 10px;
  font-size: 0.75rem;
  font-weight: 700;
}

.process-list {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  min-height: 60px;
}

.queue-list {
  flex-direction: column;
}

.process-block {
  padding: 0.5rem 0.75rem;
  border-radius: 8px;
  font-size: 0.8rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.25rem;
  min-width: 50px;
  transition: all 0.3s ease;
}

.process-id {
  font-weight: 700;
}

.process-info {
  font-size: 0.65rem;
  opacity: 0.8;
}

.new-block {
  background: rgba(56, 239, 125, 0.2);
  border: 1px solid #38ef7d;
  color: #38ef7d;
}

.ready-block {
  background: rgba(102, 126, 234, 0.2);
  border: 1px solid #667eea;
  color: #667eea;
  width: 100%;
  flex-direction: row;
  justify-content: space-between;
}

.running-block {
  background: rgba(240, 147, 251, 0.2);
  border: 1px solid #f093fb;
  color: #f093fb;
}

.waiting-block {
  background: rgba(245, 175, 25, 0.2);
  border: 1px solid #f5af19;
  color: #f5af19;
}

.terminated-block {
  background: rgba(168, 237, 234, 0.2);
  border: 1px solid #a8edea;
  color: #a8edea;
}

.empty-state {
  color: rgba(255, 255, 255, 0.3);
  font-size: 0.8rem;
  padding: 1rem;
  text-align: center;
  width: 100%;
}

.practice-arrow {
  font-size: 2rem;
  color: rgba(255, 255, 255, 0.5);
  display: flex;
  align-items: center;
}

.down-arrow {
  transform: rotate(90deg);
}

/* CPU Display */
.cpu-display {
  min-height: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.active-cpu {
  position: relative;
  padding: 1rem;
  animation: cpuPulse 1s ease-in-out infinite;
}

@keyframes cpuPulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.05); }
}

.cpu-animation {
  position: absolute;
  inset: -4px;
  border: 2px solid #f093fb;
  border-radius: 10px;
  animation: cpuGlow 1.5s ease-in-out infinite;
}

@keyframes cpuGlow {
  0%, 100% { opacity: 1; box-shadow: 0 0 10px #f093fb; }
  50% { opacity: 0.5; box-shadow: 0 0 20px #f093fb; }
}

.empty-cpu {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  color: rgba(255, 255, 255, 0.3);
}

/* I/O Spinner */
.io-spinner {
  width: 16px;
  height: 16px;
  border: 2px solid rgba(245, 175, 25, 0.3);
  border-top-color: #f5af19;
  border-radius: 50%;
  animation: ioSpin 1s linear infinite;
}

@keyframes ioSpin {
  to { transform: rotate(360deg); }
}

/* Process Table */
.process-table-section {
  background: rgba(255, 255, 255, 0.03);
  border-radius: 15px;
  padding: 1.5rem;
  margin-bottom: 2rem;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.process-table-section h3,
.statistics-section h3,
.event-log-section h3 {
  margin-bottom: 1rem;
  color: #fff;
}

.table-container {
  overflow-x: auto;
}

.process-table {
  width: 100%;
  border-collapse: collapse;
}

.process-table th,
.process-table td {
  padding: 0.75rem 1rem;
  text-align: left;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.process-table th {
  background: rgba(102, 126, 234, 0.2);
  font-weight: 600;
}

.process-table tr:hover {
  background: rgba(255, 255, 255, 0.05);
}

.row-running {
  background: rgba(240, 147, 251, 0.1);
}

.row-terminated {
  opacity: 0.6;
}

.state-badge {
  padding: 0.25rem 0.6rem;
  border-radius: 20px;
  font-size: 0.7rem;
  font-weight: 700;
}

.badge-new { background: rgba(56, 239, 125, 0.2); color: #38ef7d; }
.badge-ready { background: rgba(102, 126, 234, 0.2); color: #667eea; }
.badge-running { background: rgba(240, 147, 251, 0.2); color: #f093fb; }
.badge-waiting { background: rgba(245, 175, 25, 0.2); color: #f5af19; }
.badge-terminated { background: rgba(168, 237, 234, 0.2); color: #a8edea; }

.no-processes {
  text-align: center;
  padding: 2rem;
  color: rgba(255, 255, 255, 0.5);
}

/* Statistics */
.statistics-section {
  background: rgba(255, 255, 255, 0.03);
  border-radius: 15px;
  padding: 1.5rem;
  margin-bottom: 2rem;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 1rem;
}

.stat-card {
  background: rgba(102, 126, 234, 0.1);
  border-radius: 12px;
  padding: 1rem;
  text-align: center;
  border: 1px solid rgba(102, 126, 234, 0.3);
}

.stat-label {
  display: block;
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.7);
  margin-bottom: 0.5rem;
}

.stat-value {
  font-size: 1.5rem;
  font-weight: 700;
  color: #667eea;
}

/* Event Log */
.event-log-section {
  background: rgba(255, 255, 255, 0.03);
  border-radius: 15px;
  padding: 1.5rem;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.event-log {
  max-height: 200px;
  overflow-y: auto;
  padding: 1rem;
  background: rgba(0, 0, 0, 0.3);
  border-radius: 10px;
  font-family: 'Consolas', 'Monaco', monospace;
  font-size: 0.85rem;
}

.log-entry {
  padding: 0.3rem 0;
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
}

.log-time {
  color: #667eea;
  margin-right: 0.75rem;
}

.log-message {
  color: rgba(255, 255, 255, 0.8);
}

.no-logs {
  color: rgba(255, 255, 255, 0.3);
  text-align: center;
  padding: 1rem;
}

/* Responsive Practice Mode */
@media (max-width: 900px) {
  .practice-controls {
    flex-direction: column;
    align-items: stretch;
  }

  .control-buttons {
    margin-left: 0;
    flex-wrap: wrap;
  }

  .practice-visualization {
    flex-direction: column;
    align-items: center;
  }

  .practice-state {
    max-width: 100%;
    width: 100%;
  }

  .practice-arrow {
    transform: rotate(90deg);
  }

  .down-arrow {
    transform: rotate(90deg);
  }
}
</style>