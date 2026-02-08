# SRT Algorithm - Quick Reference Guide

## 🎯 What is SRT?

**Shortest Remaining Time (SRT)**

- **Preemptive** version of Shortest Job First (SJF)
- Selects process with **shortest remaining execution time**
- Optimal for **minimizing average waiting time**

## 🔄 How It Works

```
Every time unit:
1. Check Ready Queue
2. Find process with minimum remaining time
3. If shorter than current → PREEMPT current process
4. Execute shortest remaining time process
```

## ⚡ Key Features

| Feature          | Description                         |
| ---------------- | ----------------------------------- |
| **Preemption**   | Yes - can interrupt running process |
| **Selection**    | Minimum remaining time              |
| **Optimization** | Minimizes average waiting time      |
| **Overhead**     | High (frequent context switches)    |

## ✅ Advantages

- Minimal average waiting time (theoretically optimal)
- Good response time for short jobs
- Dynamic adaptation to new arrivals
- Short jobs complete quickly

## ❌ Disadvantages

- Risk of starvation for long processes
- High context switching overhead
- Requires knowledge/estimation of burst times
- Complex implementation

## 📊 Example

```
Time  Event
----  -----
0     P1 arrives (Burst: 8) → Starts running
1     P2 arrives (Burst: 4) → P1 PREEMPTED (remaining: 7)
2     P2 running (remaining: 3)
3     P3 arrives (Burst: 2) → P2 PREEMPTED (remaining: 2)
4     P3 running (remaining: 1)
5     P3 completes → P2 resumes
7     P2 completes → P1 resumes
14    P1 completes

Order of completion: P3 → P2 → P1
```

## 🎮 Demo Steps

1. **Start app**: `npm run dev`
2. **Toggle Practice Mode**
3. **Select "SRT"** algorithm
4. **Add processes** (different burst times)
5. **Start simulation**
6. **Observe preemption** when shorter jobs arrive

## 💻 Code Logic (from project)

```javascript
// Select process with shortest remaining time
case 'SRT':
  return ready.reduce((min, p) =>
    p.remainingTime < min.remainingTime ? p : min,
    ready[0]
  )

// Check for preemption
if (selectedAlgorithm.value === 'SRT') {
  const shorter = readyQueue.value.find(
    p => p.remainingTime < cpuProcess.value.remainingTime
  )
  if (shorter) {
    // Preempt current process
    cpuProcess.value.state = 'ready'
    cpuProcess.value = null
  }
}
```

## 📈 Performance Metrics

**Waiting Time**: Time in Ready Queue
**Turnaround Time**: Completion - Arrival
**Response Time**: First CPU access - Arrival

## 🎓 Real-World Uses

- Operating system schedulers
- Web server request handling
- Database query optimization
- Network packet scheduling
- Cloud resource allocation

## 🔄 Process States

```
NEW → READY → RUNNING → TERMINATED
       ↑         ↓
       └─ WAITING
```

States:

- **NEW**: Being created
- **READY**: Waiting for CPU
- **RUNNING**: Executing
- **WAITING**: I/O operation
- **TERMINATED**: Completed

## 🆚 vs Other Algorithms

| Algorithm | Preemptive | Criteria           | Best For          |
| --------- | ---------- | ------------------ | ----------------- |
| FCFS      | No         | Arrival order      | Simplicity        |
| SJF       | No         | Total burst time   | Known durations   |
| **SRT**   | **Yes**    | **Remaining time** | **Min wait time** |
| RR        | Yes        | Time quantum       | Time-sharing      |

## 🎤 Key Talking Points

1. **SRT is optimal** for average waiting time
2. **Preemption** enables responsiveness
3. **Trade-off**: Performance vs overhead
4. **Starvation risk** requires mitigation (aging)
5. **Used in modern** scheduling systems

## 🛠️ Troubleshooting Demo

**No preemption visible?**
→ Add processes with very different burst times (e.g., 8, 2, 5)

**Simulation too fast?**
→ Adjust "Simulation Speed" slider

**Want to see specific scenario?**
→ Use "Add Custom Process" for controlled burst times

## 📱 Quick Test Scenario

```
Add these processes in order:
P1: Burst = 8
P2: Burst = 2 (should preempt P1)
P3: Burst = 5

Expected behavior:
- P1 starts
- P2 preempts P1, completes first
- P3 or P1 continues based on remaining times
```

## 🎯 Presentation Tips

- **Start with diagram** of process states
- **Show live demo** early to engage audience
- **Use analogy**: Like a queue where shorter tasks cut in line
- **Compare** with everyday examples (express checkout lane)
- **Address concerns** about fairness/starvation upfront

## 📝 Formula Sheet

```
Waiting Time (WT) = Turnaround Time - Burst Time
Turnaround Time (TAT) = Completion Time - Arrival Time
Response Time (RT) = First Execution - Arrival Time

Average WT = Σ(WT) / n
Average TAT = Σ(TAT) / n
CPU Utilization = (Total Burst Time / Total Time) × 100%
```

## 🔗 Resources

- Full script: `SRT_PRESENTATION_SCRIPT.md`
- Project README: `README.md`
- Main component: `src/components/CPUSchedulingDiagram.vue`

---

**Practice Run**: 15-20 minutes before presenting!
