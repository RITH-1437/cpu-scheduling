# SRT Presentation - Speaker Notes

## Opening (30 sec)

- Introduce yourself
- "Today: CPU Scheduling with focus on SRT algorithm"
- Show project running on screen

## What is CPU Scheduling? (1 min)

- OS decides which process gets CPU time
- Like managing a checkout line
- Must be fair but efficient
- Multiple algorithms, each with trade-offs

## Process States Overview (2 min)

**Point to the diagram on screen**

- NEW → Process created
- READY → Waiting for CPU (in queue)
- RUNNING → Currently executing (only 1 at a time)
- WAITING → Blocked on I/O
- TERMINATED → Finished

**Key point**: "Scheduler decides: Ready → Running"

## SRT Definition (1 min)

"Shortest Remaining Time First"

- **Preemptive** - can interrupt current process
- **Always picks** process with least time left
- **Optimal** for minimizing wait time

## SRT Example (2 min)

**Write on board or show slide:**

```
T0: P1 (burst=8) arrives → starts
T1: P2 (burst=4) arrives → P1 has 7 left
    → P2 shorter → PREEMPT P1
T2: P3 (burst=2) arrives → P2 has 3 left
    → P3 shorter → PREEMPT P2
T4: P3 done → P2 resumes (3 left)
T7: P2 done → P1 resumes (7 left)
T14: All done
```

**Ask**: "Why this order? Because it minimizes total waiting!"

## Live Demo (5 min)

### Setup

- "Let me show you this in action"
- Open browser: localhost:5173
- Click **"Toggle Practice Mode"**

### Add Processes

- Select **"SRT"** from dropdown
- Click **"Add Random Process"** 3 times
- Point out: "See the burst times? 5, 2, 4"

### Run Simulation

- Click **"Start Simulation"**
- **Pause after few seconds**
- Point to screen:
  - "Look - Ready Queue shows waiting processes"
  - "CPU shows current running process"
  - "Log shows what's happening"

### Show Preemption

- Resume simulation
- **Pause when preemption occurs**
- Read log: "Process P1 preempted - Shorter job P2 arrived"
- "This is SRT in action!"

### Show Completion

- Let simulation finish
- Point to metrics: "See turnaround times"

## Advantages (1 min)

✅ "Why use SRT?"

- Minimum average wait time (proven optimal)
- Short jobs finish fast
- Good for responsive systems
- Dynamic - adapts to new arrivals

## Disadvantages (1 min)

❌ "Nothing's perfect..."

- Long processes can starve (wait forever)
- Lots of context switching (overhead)
- Need to know/estimate burst times
- Complex to implement

**Real solution**: Use aging or hybrid approaches

## Comparison (1 min)

**Show table or say:**

- FCFS: Simple, but long waits
- SJF: Good, but not preemptive
- **SRT**: Best wait time, but overhead
- Round Robin: Fair, but not optimal

"Each algorithm fits different needs"

## Real World (1 min)

"Where do we see this?"

- Operating systems (Linux, Windows)
- Web servers (short requests first)
- Databases (quick queries prioritized)
- Cloud computing (container scheduling)

"Not pure SRT, but similar concepts"

## Code Walkthrough (2 min - optional)

**If technical audience:**

```javascript
// Point to screen/slide
"Here's the core logic:"

getNextProcess() {
  return ready.reduce((min, p) =>
    p.remainingTime < min.remainingTime ? p : min
  )
}

"Simple! Find minimum remaining time."

// Preemption check
if (shorter exists)
  preempt current
  dispatch shorter
```

## Interactive (1 min)

"Want to try different scenarios?"

- Add custom burst times
- Compare algorithms
- Adjust simulation speed

**If time**: Let audience suggest burst times

## Performance Metrics (1 min)

**Point to completed processes:**

- **Waiting Time**: Time in ready queue
- **Turnaround Time**: Start to finish
- **Response Time**: First CPU access

"SRT optimizes average waiting time"

## Key Takeaways (1 min)

1. SRT = Shortest Remaining Time First
2. Preemptive → responsive
3. Optimal for wait time
4. Trade-off: overhead & complexity
5. Understanding helps design better systems

## Questions Prep

**Q: Can processes starve?**
A: Yes! Long process can wait forever if short ones keep arriving.
Solution: Aging (increase priority over time)

**Q: How know burst time?**
A: Real systems estimate using history, exponential averaging, or user hints

**Q: Why not always use SRT?**
A: Overhead of frequent context switching, starvation risk, complexity

**Q: Same remaining time?**
A: Tie-breaker: arrival time (FCFS), priority, or process ID

**Q: Multi-core CPU?**
A: Each core runs independent process, scheduling becomes more complex

## Closing (30 sec)

- "CPU scheduling is fundamental to OS"
- "SRT shows the optimization vs overhead trade-off"
- "Thank you! Questions?"
- Show GitHub repo/contact info

---

## Demo Backup Plan

**If live demo fails:**

1. Have screenshots ready
2. Walk through static example
3. Show video recording
4. Use a diagram/flowchart

## Time Check Points

- 5 min: Should be at SRT definition
- 10 min: Should be starting demo
- 15 min: Should be at comparison
- 20 min: Wrapping up, Q&A

## Energy Tips

- **Smile** - enthusiasm is contagious
- **Make eye contact** - engage audience
- **Pause** after key points
- **Use hands** to emphasize
- **Vary pace** - slow for complex parts

## Technical Checklist

✓ Laptop charged
✓ Browser open to localhost
✓ Application running and tested
✓ Backup slides/screenshots
✓ GitHub repo link ready
✓ Pointer/remote if available
✓ Water bottle nearby

## Common Mistakes to Avoid

❌ Don't say "um" too much - pause instead
❌ Don't turn back to screen for long
❌ Don't apologize for "simple" project
❌ Don't rush through demo
❌ Don't ignore questions until end

## Engagement Tricks

- Ask: "Who's used task manager?"
- Poll: "Which is faster: FCFS or SRT?"
- Invite: "Suggest a burst time!"
- Relate: "Like express checkout at grocery"

## If Running Short on Time

**Cut these sections:**

- Code walkthrough
- Extended comparison
- Some real-world examples
  **Keep these:**
- SRT definition & example
- Live demo
- Advantages/disadvantages

## If Have Extra Time

**Add these:**

- Let audience control demo
- Show algorithm comparison
- Discuss aging/starvation solutions
- Live Q&A with examples

---

**REMEMBER:**

- Breathe
- You know this material
- Mistakes happen - smile and continue
- Focus on key message: SRT = optimal wait time with trade-offs

**GOOD LUCK! 🚀**
