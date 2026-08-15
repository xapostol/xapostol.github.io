---
{"dg-publish":true,"permalink":"/arvr-design-challenge/design/learning-journey/learning-journey-breakdown/","created":"2026-08-11T21:31:32.493-07:00","dg-note-properties":{}}
---

# Overview{ #mled-overview}


This document outlines the overall training experience centered around **Torque Wrench** education. Any gaps or suggested improvements should be added to the [[ARVR Design Challenge/Design/Learning Journey/🏫 Learning Journey Breakdown#^mled-future-improvements \| Future Improvements Section]]!

> [!Assumptions]+
> - Learner may move around the outlined area using their controllers (i.e. - navigate and rotate) and their body in physical space, but must stay within the designated interaction space.
> 	- Standard movement controls: **teleport** vs **smooth locomotion** & **smooth** vs **incremented turns**.
> - The learner has standard movement controls and can access information about the tutorial progress through #PrimaryController_Button_YB.


# Experience{ #mled-experience}


This section goes over the larger training flow and the specific checkpoints learners will reach to master the **Torque Wrench**!

## Scene Mapping

### Setting

![AutoShop2037_MikaelJavaid-Camua.jpg\|700](/img/user/ARVR%20Design%20Challenge/__Resources/Media/AutoShop2037_MikaelJavaid-Camua.jpg)![GarageSet_PhilippeCaseiro.jpg\|324](/img/user/ARVR%20Design%20Challenge/__Resources/Media/GarageSet_PhilippeCaseiro.jpg)![Garage_kukichart.jpg\|373](/img/user/ARVR%20Design%20Challenge/__Resources/Media/Garage_kukichart.jpg)

**Auto Workshop**
Multiple cars raised up in the background. Layout is busy, but follows best practices for a safe work environment.
### Interaction Zone
- Simple (10 x 10) zone with a lifted car, wheel chest height to learner, and torque wrench placed on a mobile work-stand.
- Notepad is placed on the mobile work-stand. Message from their boss asking the learner to finish the last wheel, as they had to step out for an emergency.
	- Includes number of steps till training completion, diagram of the car, its wheels, and specific torque specifications for each bolt). Also indicates the mode the learner is in (i.e. - `Learning, Practice, or Endless`).
	- UI pop-up version is also accessible from #PrimaryController_Button_YB.
	- Learner presses #Controller_Button_Trigger to interact with the UI.
- Pace should be guided, but never forced.
### Atmosphere
- #Particle dust floating through the air, very minimal, but enough to give an atmosphere
- #AudioSFX of air vents and machine hums; only diegetic, soft sfx to maintain learner focus

## Specifications

### 00. Preface - Experience State

Here are the training experience states that define learner progression. These map to scene transitions and activity triggers that are explicitly indicated through visual, haptic, and audio cues.

```C#
public enum ExperienceState { Intro, GuidedExploration, Practice, Complete, Endless };
```

| Enum                | State                                                                                            | Constraints/Exit                                                                  |
| ------------------- | ------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------- |
| `Intro`             | Learner begins in the workshop. Note pad and wrench visible.                                     | Learner interacts with note pad or presses #PrimaryController_Button_YB to begin. |
| `GuidedExploration` | Active learning phase: 3-lug-nut sequence (under, over, correct). Guided step-by-step via hints. | All 3 lug-nuts completed successfully → transitions to `Practice`.                |
| `Practice`          | Learner applies skills to 4 lug-nuts on the same wheel, then a 4-wheel sequence.                 | All 4 wheels tightened to correct spec → transitions to `Complete`.               |
| `Complete`          | Learner finishes the 4-wheel sequence.                                                           | Learner may continue to `Endless` or exit.                                    |
| `Endless`           | Open-ended practice mode: varying lug-nut counts and torque specs.                               | No exit condition — persists until learner manually exits or resets.              |

### 01. Activity - Understanding

This first section of the training experience will focus on familiarizing the learner with the scene and tool context. This section also indicates how to check tutorial progress via the notepad on the desk or by pressing #PrimaryController_Button_YB for a UI pop-up.

- [x] **Checking Progress**
	- <span style="color:rgb(138, 177, 125)">Activate</span>
	    - 1x Press #PrimaryController_Button_YB
	        - While in any state of the experience, the learner may open a UI wrist menu version of the 'notepad' on their primary controller wrist.
	        - This is also available in physical space via the 'notepad'
	- <span style="color:rgb(238, 32, 77)">Deactivate</span>
	    - 1x Press #PrimaryController_Button_YB
		    - Pressing the same button closes the UI.
	- <span style="color:rgb(255, 217, 140)">Effects</span>
	    - #Visual_Dynamic semi-transparent UI wrist menu
	    - #Haptics_Light on the designated controller
	    - #AudioSFX of subtle **UI toggle** sound
- [ ] **Initiating the Experience**
	- <span style="color:rgb(138, 177, 125)">Activate</span>
	    - Picking up the **Torque Wrench**
		    - If in `Intro` state and learner picks up the torque wrench, the training experience starts triggering `GuidedExploration`.
	- <span style="color:rgb(255, 217, 140)">Effects</span>
	    - #Visual_Animation of notepad text fades out → next step fades in on the notepad
	    - #AudioSFX of soft chime
	    - #Haptics_Light on controller upon selection

### 02. Activity - Exploration

The second section will guide the learner through interacting with the torque wrench, teaching them how to handle and set torque values. This section actively guides the learner through tightening lug-nuts, each step hinting at the next with highlights, haptics, and audio cues.

The learner will set 3 different lug-nuts, following the star pattern. It will also actively show what happens when the torque value is **under (1st)**, **over (2nd)**, and **right on (3rd)** the correct torque value. 

- [ ] **Understanding the Wrench**
	- <span style="color:rgb(100, 250, 204)">Pre-Reqs</span>
	    - `ExperienceState == GuidedExploration`
	- <span style="color:rgb(138, 177, 125)">Activate</span>
	    - As learner nears an interaction (i.e. - wrench or lug nut), context hints appear.
	- <span style="color:rgb(238, 32, 77)">Deactivate</span>
	    - Prompt fades when learner completes the action.
	- <span style="color:rgb(255, 217, 140)">Effects</span>
	    - #Visual_Animation floating UI indicators (i.e. - grip hand, thumbstick) and informational text fade in as learner approaches interaction points
	    - #AudioSFX of **gentle chime** for new hint; **soft chime-up** for success
	    - #Haptics_Light when hint appears, stronger on success
- [ ] **Torque Guidance**
	- <span style="color:rgb(138, 177, 125)">Activate</span>
	    - When `TorqueWrench.TorqueValue` ≠ `LugNut.RequiredValue` and cranking completes, expected behaviors initiate. View 'Behaviors' Section of [[ARVR Design Challenge/Design/Torque Wrench/🔧 Torque Wrench Breakdown\|🔧 Torque Wrench Breakdown]] for more information.
	        1. **Undershoot** --> lug nut spins off, rolls away
	        2. **Overshoot** --> lug nut strips, wheel shifts orientation
	        3. **Achieved** --> lug nut settles
	- <span style="color:rgb(255, 217, 140)">Effects</span>
	    - #Visual_Animation floating UI indicators (i.e. - grip hand, thumbstick) and informational text appear over interaction points
- [ ] **Transition to Practice**
	- <span style="color:rgb(138, 177, 125)">Activate</span>
	    - After successfully tightening the last lug-nut in `GuidedExploration`
	- <span style="color:rgb(255, 217, 140)">Effects</span>
	    - #Visual_Animation: notepad updates → “Now try all 4 lug-nuts on the wheel.”
	    - #Visual_Dynamic UI shifts to 'Practice Mode'
	    - #AudioSFX Confirmed **success chime**
	    - #Haptics_Light and controller pulse

### 03. Activity - Practice 

The last section allows the learner to practice what they've learned by changing and manipulating the lug-nuts on a single wheel, and then a 4-wheel sequence. Each set of lug-nuts has a different value that must be tightened correctly.

After the learner has done all 4 wheels, they reach the `Complete`state where they're affirmed of their progress (i.e. - visualized on the notepad). This activates an `Endless` where they can continue to practice what they've learned.

- [ ] **Completing 4-Wheel Sequence**
	- <span style="color:rgb(138, 177, 125)">Activate</span>
	    - When all 4 wheels are tightened to correct torque
	- <span style="color:rgb(255, 217, 140)">Effects</span>
	    - #Visual_Animation car lowers slightly → “All wheels secured!”
	    - #AudioSFX full success suite (UI chime + distant **clank** of car settling)
	    - #Haptics_Medium → #Haptics_Heavy pulse
- [ ] **Continuing or Resetting the Experience**
	- <span style="color:rgb(138, 177, 125)">Activate</span>
	    - 1x Press “Continue Practicing” on Notepad (Physical or UI Version)
	- <span style="color:rgb(238, 32, 77)">Deactivate</span>
	    - 1x Press “Exit Practice” on Notepad (Physical or UI Version)
	- <span style="color:rgb(255, 217, 140)">Effects</span>
	    - #Visual_Dynamic UI shifts to 'Endless Mode'
	    - #AudioSFX **car revving** sfx
    
## Validation

### Process

In order to validate the effectiveness of the activity, there are a few checks and groups I would consider running the experience with. The main testing philosophy is to 'fail-fast', so testing frequently and often with different groups gets us closer to the end goal faster!

#### 1. Playtesting
- **Cycle 1 - Internal**
	- Ensure state transitions, tool interactions, and feedback timing with development team (design, engineering, audio).
	- Check for any immediate gaps.
- **Cycle 2 - New Learners**
	- Observe non-technical learners (no VR or torque wrench experience)
	- In this section, I'd measure for completion rate, time-to-first-correct-torque, and recovery from errors / blockers.
- **Cycle 3 - Experts**
	- Test with technicians to check accuracy of interactions, procedures (e.g., star pattern), and skill transfer (perceived).

#### 2. Performance
- **Objective**
	- Step completion rate, average torque error margin, # of retries per wheel, time per step.
- **Subjective**
	- Post-session surveys (presence/confidence scales).
- **Misses**
	- Flag any interaction where >20% of learners missed the intended workflow (i.e. - skipping hints, mis-timing wrench release, clunking grabbing).

#### 3. Accessibility
- **Motor**
	- Button hold timings, alternative inputs (e.g., thumbstick vs. grip), additional ways to complete a task.
- **Cognitive**
	- UI readability, cognitive workload during multi-step tasks, timing with hints.
- **Physical**
	- Comfort in the headset, locomotion comfort (teleport vs. smooth), ease of use for notepad (UI wrist menu).  

# Future Improvements{ #mled-future-improvements}

## Pre-Interview (08/11/2026)

### 01. Existing Specifications
Include real specifications and interaction procedures provided by organization and other industry best practices (i.e. - have flow of interactions follow specified safety procedures from handbooks, guides, etc.).

### 02. Multiple Activities
Have a true 'endless' mode where the learner can change out different car tires for multiple vehicles. This expanded activity would also educate on attaching different torque wrench heads to set the correct lug-nut size.

### 03. Modular Design System
Utilize Tags to create a Modular Design System via an HTML Webpage or a scene directly integrated into the Unity System
- Can take advantage of Unity game-objects, scripts, and existing interaction systems