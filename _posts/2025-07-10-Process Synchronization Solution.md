---
title: "Process Synchronization Solution: 8-Process Dependency Problem"
date: 2025-07-10 00:26:00 +0000
categories: [Operating System, theory]
tags: [Semaphore, Synchronization]
author: srodi
---
## Problem

We have 8 processes with the following dependencies:
- P1 and P2 can execute in arbitrary order
- P3 and P4 must wait for both P1 and P2 to finish
- P5 must wait for both P3 and P4 to finish
- P6 must wait for P5 to finish
- P7 must wait for P6 to finish
- P8 must wait for both P5 and P6 to finish (can run concurrently with P7)

## Dependency Graph
```
P1 ──┐
     ├─→ P3 ──┐
P2 ──┤       ├─→ P5 ──┬─→ P6 ──→ P7
     └─→ P4 ──┘       └────└──→ P8
```

## Solution with Unlimited Semaphores

### Step-by-Step Construction

Start with P1 and P2 (no dependencies):
```
P1: CODE EXEC
P2: CODE EXEC
```

Add P3 and P4 (wait for P1 and P2):
- s1: P1 must signal twice (for P3 and P4)
- s2: P2 must signal twice (for P3 and P4)

```
P1: CODE EXEC → V(s1) → V(s1)
P2: CODE EXEC → V(s2) → V(s2)
P3: P(s1) → P(s2) → CODE EXEC
P4: P(s1) → P(s2) → CODE EXEC
```

Add P5 (waits for P3 and P4):
- s3, s4: Each signals once
```
P1: CODE EXEC → V(s1) → V(s1)
P2: CODE EXEC → V(s2) → V(s2)
P3: P(s1) → P(s2) → CODE EXEC → V(s3)
P4: P(s1) → P(s2) → CODE EXEC → V(s4)
P5: P(s3) → P(s4) → CODE EXEC
```

Add P6 (waits for P5):
- s5: Signals once
```
P1: CODE EXEC → V(s1) → V(s1)
P2: CODE EXEC → V(s2) → V(s2)
P3: P(s1) → P(s2) → CODE EXEC → V(s3)
P4: P(s1) → P(s2) → CODE EXEC → V(s4)
P5: P(s3) → P(s4) → CODE EXEC → V(s5)
P6: P(s5) → CODE EXEC
```

Add P7 (waits for P6):
- s6: Signals once
```
P1: CODE EXEC → V(s1) → V(s1)
P2: CODE EXEC → V(s2) → V(s2)
P3: P(s1) → P(s2) → CODE EXEC → V(s3)
P4: P(s1) → P(s2) → CODE EXEC → V(s4)
P5: P(s3) → P(s4) → CODE EXEC → V(s5)
P6: P(s5) → CODE EXEC → V(s6)
P7: P(s6) → CODE EXEC
```

Add P8 (waits for P5 and P6):
- s5: Signals one more
- s6: Signals one more

```
P1: CODE EXEC → V(s1) → V(s1)
P2: CODE EXEC → V(s2) → V(s2)
P3: P(s1) → P(s2) → CODE EXEC → V(s3)
P4: P(s1) → P(s2) → CODE EXEC → V(s4)
P5: P(s3) → P(s4) → CODE EXEC → V(s5) → V(s5) 
P6: P(s5) → CODE EXEC → V(s6) → V(s6)
P7: P(s6) → CODE EXEC
P8: P(s5) → P(s6) → CODE EXEC
```
**Semaphores used**: 6 semaphores (s1 = 0, s2 = 0, s3 = 0, s4 = 0, s5 = 0, s6 = 0)
