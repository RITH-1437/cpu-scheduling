# SRT Algorithm - Demo Script (5-7 minutes)

## 🎯 Introduction (30 seconds)

"Hello everyone! Today I'll demonstrate the **SRT (Shortest Remaining Time)** scheduling algorithm using this interactive CPU scheduling simulator."

**What is SRT?**
- Preemptive scheduling algorithm
- Always executes the process with the shortest remaining time
- Optimal for minimizing average waiting time

---

## 💻 Live Demo

### Step 1: Launch Application (10 seconds)

```bash
npm run dev
```

*"Let me start the application... and here we are!"*

---

### Step 2: Enable Practice Mode (5 seconds)

**Action:** Click "Toggle Practice Mode" button

*"First, I'll enable Practice Mode to interact with the simulator."*

---

### Step 3: Select SRT Algorithm (10 seconds)

**Action:** Select **"SRT - Shortest Remaining Time"** from dropdown

*"Now I'm selecting the SRT algorithm. Notice the description: 'Process with shortest remaining time executes first.'"*

---

### Step 4: Add Processes (30 seconds)

**Action:** Click "Add Random Process" 3-4 times

*"Let me add several processes. Watch the Ready Queue:"*
- **P1**: Burst Time = 5
- **P2**: Burst Time = 2  
- **P3**: Burst Time = 6
- **P4**: Burst Time = 3

*"Each process has a different burst time - this is important for seeing how SRT works."*

---

### Step 5: Start Simulation (2 minutes)

**Action:** Click "Start Simulation"

#### Watch and Explain:

**As simulation runs, point out:**

1. **Initial Selection**
   - *"P2 starts first because it has the shortest remaining time (2 units)"*

2. **Preemption Event** (when it happens)
   - *"See this? P1 was running, but when P2 arrived with shorter time, P1 got PREEMPTED"*
   - *"This is the key feature of SRT - it can interrupt a running process"*

3. **Process States**
   - 🟡 **Yellow boxes** = Ready Queue
   - 🟢 **Green box** = Currently Running
   - ⚫ **Gray boxes** = Completed
   
   *"Notice how processes move between these states"*

4. **Simulation Log**
   - *"The log shows every decision: dispatching, preemptions, completions"*
   - Read one entry: *"Process P2 preempted - Shorter job P3 arrived"*

---

### Step 6: Key Observations (1 minute)

**Pause the simulation** (Click "Pause")

*"Let me highlight the important points:"*

✅ **Shortest jobs complete first**
   - "P2 finished before P1, even though P1 arrived earlier"

✅ **Preemption happens**
   - "When a shorter job arrives, the current process is interrupted"

✅ **Dynamic scheduling**
   - "The algorithm constantly evaluates which process should run"

✅ **Waiting time minimized**
   - "Short processes don't wait behind long ones"

---

### Step 7: Show Completion (30 seconds)

**Action:** Resume or let simulation finish

*"As you can see, all processes eventually complete. Look at the turnaround times:"*
- Shortest jobs: Low turnaround time
- Longest jobs: Higher turnaround time

---

## 🎓 Key Takeaways (30 seconds)

**Say:**

"So what did we learn about SRT?"

1. **It's preemptive** - can interrupt running processes
2. **It's optimal** - minimizes average waiting time
3. **It's dynamic** - adapts to new arrivals
4. **It's realistic** - similar to modern OS schedulers

**Trade-off:** More context switching = more overhead

---

## 💡 Quick Comparison (Optional - 30 seconds)

**If you have time:**

**Action:** Reset and select "FCFS"
- Add same processes
- Run simulation

*"Notice the difference? With FCFS, processes run in order of arrival. Compare the completion times - SRT is more efficient!"*

---

## 🎯 Simple Analogy (Optional - 30 seconds)

*"Think of SRT like an express checkout lane at a grocery store:"*
- Person with 2 items goes before person with full cart
- If someone with 1 item arrives, they jump ahead
- Minimizes total waiting time for everyone

---

## ❓ Q&A Prompts

If audience asks:

**Q: "What if a long process never runs?"**
A: "Good question! That's called starvation. Real systems use 'aging' - gradually increasing priority of waiting processes."

**Q: "How does it know remaining time?"**
A: "Real systems estimate based on past behavior. Our simulator knows exact times for demonstration."

**Q: "Is this used in real operating systems?"**
A: "Similar concepts! Linux and Windows use priority-based scheduling inspired by SRT."

---

## 📋 Demo Checklist

**Before demo:**
- [ ] Application is running
- [ ] Browser is open and visible
- [ ] Screen is shared/visible to audience
- [ ] Practice mode is ready

**During demo:**
- [ ] Speak clearly and pace yourself
- [ ] Point to screen when referencing elements
- [ ] Pause after each key point
- [ ] Watch for audience understanding

**After demo:**
- [ ] Show GitHub repository
- [ ] Offer to answer questions
- [ ] Provide access to project if needed

---

## 🎬 Script Timing

| Section | Time | Cumulative |
|---------|------|------------|
| Introduction | 30s | 0:30 |
| Launch & Setup | 25s | 0:55 |
| Add Processes | 30s | 1:25 |
| Run Simulation | 2m | 3:25 |
| Key Observations | 1m | 4:25 |
| Completion | 30s | 4:55 |
| Takeaways | 30s | 5:25 |
| Optional Comparison | 30s | 5:55 |

**Total: 5-7 minutes** (depending on optional sections)

---

## 💬 What to Say at Each Step

### Opening
*"This simulator visualizes how CPU scheduling works. I'll show you SRT - one of the most efficient algorithms."*

### Adding Processes
*"Each box represents a process with a burst time - how long it needs the CPU."*

### During Simulation
*"Watch the Ready Queue - it's sorted by remaining time in SRT mode."*

### Preemption Event
*"THIS is SRT's power - switching to shorter jobs immediately!"*

### Completion
*"Shorter jobs finished first, minimizing overall waiting time."*

### Closing
*"SRT balances efficiency with complexity - perfect for understanding scheduling trade-offs."*

---

## 🚨 Troubleshooting

**If no preemption visible:**
- Add processes with very different burst times (8, 2, 7, 1)

**If simulation too fast:**
- Adjust speed slider before starting

**If confusion about colors:**
- Explain color coding again: Yellow=Ready, Green=Running, Gray=Done

**If technical issue:**
- Have screenshots ready as backup
- Explain the concept verbally with board/slides

---

## 🎤 Confidence Boosters

- **Breathe** - Take your time
- **Smile** - Show enthusiasm
- **Point** - Direct attention to screen
- **Pause** - Let information sink in
- **Ask** - "Does everyone see the preemption?"

---

## ✨ Pro Tips

1. **Add processes before starting** - easier to explain
2. **Pause at key moments** - especially during preemptions
3. **Use the log** - it shows exactly what's happening
4. **Compare burst times** - "See? 2 is less than 5"
5. **Repeat key phrase**: "Shortest Remaining Time First"

---

**YOU'VE GOT THIS! 🚀**

Keep it simple, focus on the demo, and let the visualization do the talking!
