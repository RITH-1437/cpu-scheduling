# CPU Scheduling Project - SRT Algorithm Presentation Script

## 📋 Introduction (2 minutes)

**Good [morning/afternoon], everyone!**

Today, I'll be presenting a CPU Scheduling Simulator project that demonstrates various scheduling algorithms, with a special focus on the **SRT (Shortest Remaining Time)** algorithm.

**What you're about to see:**

- An interactive Vue.js application that visualizes CPU process scheduling
- Real-time simulation of how processes move through different states
- A deep dive into the SRT algorithm and its advantages

---

## 🎯 Project Overview (2 minutes)

**This project is a CPU Scheduling Simulator built with:**

- **Vue.js 3** - Modern reactive framework
- **Vite** - Fast build tool
- **Interactive UI** - Real-time process visualization

**The simulator supports 4 scheduling algorithms:**

1. **FCFS** - First Come First Serve
2. **SJF** - Shortest Job First
3. **SRT** - Shortest Remaining Time _(Our focus today)_
4. **RR** - Round Robin

---

## 🔄 CPU Process States (3 minutes)

**Before we dive into SRT, let's understand how processes move through different states:**

### The 5 Process States:

1. **NEW** 🆕
   - Process is being created
   - Initial state when a program starts

2. **READY QUEUE** ⏳
   - Process is waiting for CPU allocation
   - Has all resources except CPU time
   - Multiple processes can be in this queue

3. **RUNNING** ▶️
   - Process is currently executing on the CPU
   - Only ONE process can run at a time (single CPU)

4. **WAITING** ⏸️
   - Process is waiting for I/O operation to complete
   - Cannot proceed until I/O is done
   - Examples: Reading from disk, network requests

5. **TERMINATED** ✅
   - Process has finished execution
   - All resources are released

### State Transitions:

- **NEW → READY**: Admitted to the system
- **READY → RUNNING**: CPU dispatcher selects the process
- **RUNNING → WAITING**: Process requests I/O
- **WAITING → READY**: I/O operation completes
- **RUNNING → READY**: Preemption (for SRT and RR)
- **RUNNING → TERMINATED**: Process completes

---

## 🎯 What is SRT? (4 minutes)

### Shortest Remaining Time (SRT)

**Definition:**
SRT is a **preemptive** version of the Shortest Job First (SJF) algorithm. At any given time, the process with the shortest remaining execution time is selected to run.

### Key Characteristics:

✅ **Preemptive** - Can interrupt the currently running process
✅ **Dynamic** - Makes decisions based on remaining time, not total burst time
✅ **Optimal** - Minimizes average waiting time
✅ **Responsive** - Short jobs get priority

### How SRT Works:

```
At every decision point:
1. Check all processes in the Ready Queue
2. Compare their remaining execution times
3. Select the process with the SHORTEST remaining time
4. If a new process arrives with shorter remaining time than current:
   → Preempt the current process
   → Move it back to Ready Queue
   → Start executing the shorter process
```

### Example Scenario:

```
Time 0: P1 arrives (Burst Time: 8)
        → P1 starts running (Remaining: 8)

Time 1: P2 arrives (Burst Time: 4)
        → P2's remaining time (4) < P1's remaining (7)
        → P1 is PREEMPTED
        → P2 starts running

Time 2: P3 arrives (Burst Time: 2)
        → P3's remaining time (2) < P2's remaining (3)
        → P2 is PREEMPTED
        → P3 starts running

Time 4: P3 completes
        → P2 resumes (shortest remaining: 3)

Time 7: P2 completes
        → P1 resumes (remaining: 7)

Time 14: P1 completes
```

---

## 💻 Live Demonstration (5 minutes)

**Now let me show you the simulator in action!**

### Demo Steps:

1. **Launch the Application**

   ```bash
   npm run dev
   ```

2. **Toggle Practice Mode**
   - Click "Toggle Practice Mode" button
   - This activates the interactive simulator

3. **Select SRT Algorithm**
   - Choose "SRT - Shortest Remaining Time" from dropdown
   - Notice the algorithm description

4. **Add Processes**
   - Click "Add Random Process" multiple times
   - Observe: Each process gets:
     - Process ID (P1, P2, P3...)
     - Burst Time (1-5 units)
     - Priority (1-5)
     - Arrival Time (current simulation time)

5. **Start Simulation**
   - Click "Start Simulation"
   - Watch the processes move through states:
     - NEW → READY → RUNNING
     - Preemption occurs when shorter job arrives
     - Check the simulation log

6. **Observe Key Behaviors:**
   - ⚡ **Preemption**: When a shorter process arrives, current process is interrupted
   - 📊 **Visual Updates**: Process boxes change colors based on state
   - 📝 **Log Messages**: Real-time updates showing decisions
   - ⏱️ **Metrics**: Waiting time, turnaround time

### Key Features to Highlight:

**Ready Queue Visualization**

```
Shows all processes waiting for CPU
Sorted by remaining time in SRT mode
```

**CPU Process Display**

```
Displays currently running process
Shows remaining time decreasing
```

**Simulation Log**

```
Time 0: Process P1 created (Burst: 5, Priority: 3)
Time 0: Process P1 moved to Ready Queue
Time 1: Process P1 dispatched to CPU
Time 2: Process P2 created (Burst: 2, Priority: 4)
Time 2: Process P2 moved to Ready Queue
Time 3: Process P1 preempted (Shorter job P2 arrived)
Time 3: Process P2 dispatched to CPU
```

---

## 📊 SRT vs Other Algorithms (3 minutes)

### Comparison Table:

| Algorithm | Preemptive? | Selection Criteria | Best For             |
| --------- | ----------- | ------------------ | -------------------- |
| **FCFS**  | ❌ No       | Arrival time       | Simple, fair systems |
| **SJF**   | ❌ No       | Total burst time   | Known job lengths    |
| **SRT**   | ✅ Yes      | Remaining time     | Minimizing wait time |
| **RR**    | ✅ Yes      | Time quantum       | Time-sharing systems |

### SRT Advantages:

✅ **Minimal Average Waiting Time**

- Theoretically optimal for minimizing waiting time
- Short jobs complete quickly

✅ **Good Response Time**

- Short jobs don't wait behind long ones
- Immediate feedback for quick tasks

✅ **Dynamic Adaptation**

- Adjusts to new arrivals automatically
- No job waits indefinitely (if properly implemented)

### SRT Disadvantages:

❌ **Starvation Risk**

- Long processes may wait indefinitely
- If short processes keep arriving

❌ **High Context Switching**

- Frequent preemptions
- Overhead from saving/restoring process state

❌ **Need to Know Burst Times**

- Requires estimation in real systems
- May use historical data or heuristics

❌ **Complex Implementation**

- More overhead than non-preemptive algorithms
- Requires precise timing mechanisms

---

## 🛠️ Implementation Highlights (3 minutes)

### Core SRT Logic in Our Project:

```javascript
// SRT Algorithm Selection
const getNextProcess = () => {
  const ready = readyQueue.value;
  if (ready.length === 0) return null;

  switch (selectedAlgorithm.value) {
    case "SRT":
      // Find process with minimum remaining time
      return ready.reduce(
        (min, p) => (p.remainingTime < min.remainingTime ? p : min),
        ready[0],
      );
  }
};

// Preemption Check (runs every time unit)
if (selectedAlgorithm.value === "SRT") {
  // Check if any ready process has shorter remaining time
  const shorter = readyQueue.value.find(
    (p) => p.remainingTime < cpuProcess.value.remainingTime,
  );

  if (shorter) {
    // Preempt current process
    cpuProcess.value.state = "ready";
    addLog(`Process ${cpuProcess.value.id} preempted 
           (Shorter job ${shorter.id} arrived)`);
    cpuProcess.value = null;
  }
}
```

### Process Data Structure:

```javascript
{
  id: 'P1',                    // Process identifier
  arrivalTime: 0,              // When process entered system
  burstTime: 5,                // Total execution time needed
  remainingTime: 5,            // Time left to complete
  priority: 3,                 // For priority scheduling
  state: 'ready',              // Current state
  waitingTime: 0,              // Total time spent waiting
  turnaroundTime: 0            // Completion time - arrival time
}
```

### Simulation Flow:

```
1. Time increments by 1 unit each step
2. Update waiting times for all ready processes
3. Check for I/O completion (waiting → ready)
4. CPU Scheduling Decision:
   - If CPU is idle: Select next process (SRT logic)
   - If CPU is busy: Check for preemption
5. Execute current process (decrease remaining time)
6. Check for completion or I/O request
7. Update logs and metrics
8. Repeat
```

---

## 📈 Performance Metrics (2 minutes)

### Key Metrics Tracked:

**1. Waiting Time**

- Time spent in Ready Queue
- Lower is better
- SRT typically minimizes this

**2. Turnaround Time**

- Completion Time - Arrival Time
- Total time from submission to completion
- Includes waiting + execution time

**3. Response Time**

- First time process gets CPU - Arrival Time
- Important for interactive systems
- SRT provides good response for short jobs

### Example Calculation:

```
Process P1:
- Arrival Time: 0
- Burst Time: 5
- Starts at: 2 (waited 2 units)
- Preempted at: 4 (ran 2 units)
- Resumes at: 10 (waited 6 more units)
- Completes at: 13 (ran remaining 3 units)

Waiting Time = 2 + 6 = 8 units
Turnaround Time = 13 - 0 = 13 units
Response Time = 2 - 0 = 2 units
```

---

## 🎓 Real-World Applications (2 minutes)

### Where SRT Concepts Are Used:

**1. Operating Systems**

- Linux Completely Fair Scheduler (CFS)
- Windows Process Scheduling
- Priority-based scheduling with time estimates

**2. Web Servers**

- Request queuing
- Prioritizing short API calls
- Load balancing

**3. Database Query Optimization**

- Query execution planning
- Short queries get priority
- Reduces overall latency

**4. Network Packet Scheduling**

- Quality of Service (QoS)
- Low-latency packet prioritization
- Real-time communication systems

**5. Cloud Computing**

- Container orchestration (Kubernetes)
- Serverless function scheduling
- Resource allocation in datacenters

---

## 🎮 Interactive Features (2 minutes)

### Our Simulator Offers:

**1. Custom Process Creation**

- Add processes with specific burst times
- Set custom priorities
- Control arrival times

**2. Algorithm Comparison**

- Switch between FCFS, SJF, SRT, RR
- Compare performance
- Observe different behaviors

**3. Adjustable Parameters**

- **Time Quantum** (for Round Robin)
- **Simulation Speed** (1s, 0.5s, 0.2s)
- **Process Characteristics**

**4. Real-Time Monitoring**

- Live process state visualization
- Color-coded states:
  - 🟦 NEW - Blue
  - 🟡 READY - Yellow
  - 🟢 RUNNING - Green
  - 🟠 WAITING - Orange
  - ⚫ TERMINATED - Gray

**5. Detailed Logging**

- Every state transition logged
- Timestamp for each event
- Easy debugging and analysis

---

## 🔧 Technical Stack (1 minute)

**Frontend Framework:**

- Vue.js 3 with Composition API
- Reactive state management
- Computed properties for derived data

**Build Tool:**

- Vite for fast development
- Hot Module Replacement (HMR)
- Optimized production builds

**Key Vue Features Used:**

- `ref()` - Reactive state
- `computed()` - Derived values
- `onMounted()` - Lifecycle hooks
- Template directives for rendering

---

## 🎯 Conclusion (2 minutes)

### What We've Learned:

✅ **SRT Algorithm**

- Preemptive scheduling based on remaining time
- Optimal for minimizing average waiting time
- Dynamic and responsive to new arrivals

✅ **CPU Process States**

- Five states: New, Ready, Running, Waiting, Terminated
- State transitions and their triggers
- The role of the scheduler

✅ **Practical Application**

- How scheduling affects system performance
- Trade-offs between different algorithms
- Real-world usage in modern systems

### Key Takeaways:

1. **SRT is optimal** for average waiting time but has overhead
2. **Preemption enables responsiveness** but increases context switching
3. **No algorithm is perfect** - each has trade-offs
4. **Understanding scheduling** helps in system design and optimization

---

## 💡 Future Enhancements

**Potential Improvements:**

- Add more algorithms (Priority Scheduling, Multilevel Queue)
- Implement aging to prevent starvation
- Add Gantt chart visualization
- Export simulation results to CSV
- Add statistical analysis (average waiting time, throughput)
- Multi-core CPU simulation
- Real-time process arrival simulation

---

## ❓ Q&A Session

**Common Questions to Prepare For:**

**Q: How does SRT prevent starvation?**
A: In pure SRT, starvation can occur. Solutions include:

- Aging: Gradually increase priority of waiting processes
- Maximum wait time limits
- Hybrid approaches combining SRT with other criteria

**Q: Why not always use SRT?**
A: Because of:

- High context switching overhead
- Difficulty estimating burst times in practice
- Potential for starvation
- Complexity of implementation

**Q: How do real OSes estimate burst time?**
A: Using:

- Historical averages
- Exponential averaging of past behavior
- User/process hints
- Static priority assignments

**Q: What happens if two processes have the same remaining time?**
A: Tie-breaking rules:

- Use arrival time (FCFS for ties)
- Use process priority
- Use process ID
- Implementation-dependent

---

## 📝 Demo Checklist

**Before Presentation:**

- [ ] Install dependencies: `npm install`
- [ ] Test the application: `npm run dev`
- [ ] Verify all algorithms work
- [ ] Prepare example scenarios
- [ ] Check simulation log visibility
- [ ] Test on presentation screen/projector

**During Demo:**

- [ ] Open localhost URL
- [ ] Toggle Practice Mode
- [ ] Select SRT algorithm
- [ ] Add 3-4 processes with varying burst times
- [ ] Start simulation and pause to explain
- [ ] Show preemption happening
- [ ] Display simulation log
- [ ] Compare with FCFS or SJF
- [ ] Reset and repeat if needed

---

## 📚 References

- **Operating System Concepts** by Silberschatz, Galvin, Gahne
- **Modern Operating Systems** by Andrew S. Tanenbaum
- Vue.js 3 Documentation
- CPU Scheduling Algorithms (Academic Papers)

---

## 🙏 Thank You!

**Questions? Try it yourself!**

```bash
# Clone and run the project:
git clone <repository-url>
cd CPU-Scheduling
npm install
npm run dev
```

**Repository:** [Your GitHub Repository Link]
**Contact:** [Your Email/Contact]

---

**End of Presentation Script**

_Estimated Total Time: 30-35 minutes_
_Adjust timing based on your presentation requirements_
