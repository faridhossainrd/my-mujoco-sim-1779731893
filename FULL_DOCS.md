# PIB Robot — MuJoCo Simulator Complete Documentation

## Overview

The PIB robot is a humanoid upper body with a head, two articulated arms, and two gripper hands. It is mounted on a fixed torso — there are no legs. All joints use `axis="0 0 1"` in local body frames with complex rotations, so the actual motion directions are **non-obvious**. Use the empirically verified directions below.

---

## Joint & Actuator Reference (CRITICAL for simulation)

### Actuator Index Map (`data.ctrl[i]`)

Actuators are ordered exactly as in the XML `<actuator>` section. The ctrl range is **[-1, 1] for ALL actuators**.

```
 0: head_horizontal          — POSITIVE = turn LEFT,  NEGATIVE = turn RIGHT
 1: head_vertical            — POSITIVE = tilt BACK (look up), NEGATIVE = tilt FORWARD (look down)
 2: shoulder_vertical_left   — NEGATIVE = raise arm FORWARD/UP, POSITIVE = move arm BACKWARD
 3: shoulder_horizontal_left — lateral sweep; NEGATIVE = arm moves OUT to left side
 4: upper_arm_left           — NEGATIVE = raises arm FORWARD, POSITIVE = arm DOWN/BACK
 5: elbow_left               — POSITIVE = bend elbow (forearm moves LEFT and slightly up)
 6: forearm_left             — forearm roll/twist
 7: wrist_left               — wrist rotation
 8: left_thumb_opposition
 9: left_thumb_proximal
10: left_thumb_distal
11: left_index_proximal
12: left_index_distal
13: left_middle_proximal
14: left_middle_distal
15: left_ring_proximal
16: left_ring_distal
17: left_pinky_proximal
18: left_pinky_distal
19: shoulder_vertical_right  — POSITIVE = raise arm FORWARD/UP, NEGATIVE = move arm BACKWARD
20: shoulder_horizontal_right — lateral sweep; POSITIVE = arm moves OUT to right side
21: upper_arm_right          — POSITIVE = arm BACKWARD/DOWN, NEGATIVE = arm FORWARD
22: elbow_right              — POSITIVE = bend elbow (forearm moves RIGHT)
23: forearm_right            — forearm roll/twist
24: wrist_right              — wrist rotation
25: right_thumb_opposition
26: right_thumb_proximal
27: right_thumb_distal
28: right_index_proximal
29: right_index_distal
30: right_middle_proximal
31: right_middle_distal
32: right_ring_proximal
33: right_ring_distal
34: right_pinky_proximal
35: right_pinky_distal
```

---

## CRITICAL — Arm Raising Requires MULTIPLE Actuators Together

Single joints alone **cannot** raise the arms above shoulder height — gravity overwhelms individual actuators.
To raise an arm you **MUST** combine `shoulder_vertical` + `upper_arm` + `shoulder_horizontal` together at high ctrl values.

---

## How to Set Controls via `data.ctrl` in `main.js`

Add your simulation control code inside the render loop in `src/main.js`, within the `if (!this.params["paused"])` block, **BEFORE** the `mujoco.mj_step()` call:

```javascript
if (this.params["scene"] === "pib_robot.xml") {
  const t = this.data.time;
  // Example: raise left arm forward — combine shoulder + upper arm
  this.data.ctrl[2] = -1.0;  // shoulder_vertical_left NEGATIVE = forward
  this.data.ctrl[4] = -0.8;  // upper_arm_left NEGATIVE = forward assist
  this.data.ctrl[3] = -0.3;  // shoulder_horizontal_left slightly out
}
```

---

## Common Pose Cheat Sheet (VERIFIED EMPIRICALLY)

| Pose | Key ctrls |
|---|---|
| Arms at sides (rest) | all 0 |
| Left arm raised forward | `[2]=-1.0, [4]=-0.8, [3]=-0.3` |
| Right arm raised forward | `[19]=1.0, [21]=-0.8, [20]=0.3` |
| T-pose | `[3]=-1.0,[2]=-0.5,[4]=-0.5` (L); `[20]=1.0,[19]=0.5,[21]=-0.5` (R) |
| Wave left arm | `[2]=-0.8+0.3*sin(t*3), [5]=0.6, [4]=-0.5` |
| Head nod | `[1]=0.5*sin(t*2)` |
| Head shake | `[0]=0.5*sin(t*3)` |
| Close left hand (fist) | `[9..18]` = 1.0 (all finger joints) |
| Close right hand (fist) | `[26..35]` = 1.0 (all finger joints) |

---

## Important Notes

- Robot has **NO legs** — fixed upper body mounted on a static base.
- **LEFT arm** uses NEGATIVE values to raise forward; **RIGHT arm** uses POSITIVE (NOT symmetric!).
- Raising an arm requires combining `shoulder_vertical` + `upper_arm` + `shoulder_horizontal`.
- ctrl is clamped to **[-1, 1]**. Use 0.8–1.0 to overcome gravity.
- Use `this.data.time` for time-based animation (smooth motion with `sin`/`cos`).
- For smooth easing: `(1 - Math.cos(phase * Math.PI)) / 2`.

---

## Physics Simulation Loop

The simulation loop runs in `src/main.js`. Structure:

```javascript
// Inside render loop, before mj_step:
if (!this.params["paused"]) {
  const t = this.data.time;

  if (this.params["scene"] === "pib_robot.xml") {
    // Set your ctrl values here
    this.data.ctrl[0] = 0.3 * Math.sin(t * 2);  // head sway
  }

  mujoco.mj_step(this.model, this.data);
}
```
