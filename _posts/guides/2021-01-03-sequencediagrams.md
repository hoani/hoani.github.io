---
title: "Sequence Diagrams"
excerpt: "Sequence Diagram Cheatsheet"
toc: true
categories:
  - guide
  - golang-guide
  - rust-guide
  - python-guide
  - cpp-guide
---

Sequence diagrams show process flow.

<figure>
    <img src="/assets/images/posts/guides/sequencediagrams/000_main.png">
</figure>

## Elements

Dispatching calls:
* solid arrows dispatch
* dotted arrows return data
* box the timeline to show lifetime of a dispatch
<figure class="two-thirds">
    <img src="/assets/images/posts/guides/sequencediagrams/001_dispatch.png">
</figure>

Frame
* use frames for:
  * `opt` optional actions, followed by `[condition]`
  * `loop` looping behaviour, followed by `[num loops]` or `[condition]`
  * `alt` alternative actions, followed by `[condition]`
<figure class="two-thirds">
    <img src="/assets/images/posts/guides/sequencediagrams/002_option.png">
</figure>

## Tools
[draw.io](https://app.diagrams.net/)
* Free-form diagrams, good support for sequence diagrams

[websequencediagrams.com](https://www.websequencediagrams.com/)
* Fast and automatically formatted 
* Uses code defintions, like:
```
title MotorControl
actor A
A->+SpeedControl: update(t, setpoint)
SpeedControl->+Sensor: getSample()
Sensor-->-SpeedControl: sensorData
SpeedControl->Estimator: update(t, sensorData)
opt t - tLast > ControlPeriod
SpeedControl->+Estimator: getState()
Estimator-->-SpeedControl: stateData
SpeedControl->+Regulator: regulate(stateData, setpoint)
Regulator-->-SpeedControl: commandTorque
SpeedControl->Motor: actuate(commandTorque)
```
