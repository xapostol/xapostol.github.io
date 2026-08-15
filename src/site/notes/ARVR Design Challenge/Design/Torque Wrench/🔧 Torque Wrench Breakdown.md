---
{"dg-publish":true,"permalink":"/arvr-design-challenge/design/torque-wrench/torque-wrench-breakdown/","created":"2026-08-11T20:23:26.766-07:00","dg-note-properties":{}}
---

# Overview{ #twd-overview}


This document outlines how the **Torque Wrench** should function through-out the training experience, specifically in relation to the learner and the interactive components (i.e. - lug-nuts). Any gaps or suggested improvements should be added to the [[ARVR Design Challenge/Design/Torque Wrench/🔧 Torque Wrench Breakdown#^twd-future-improvements \| Future Improvements Section]]!

> [!Assumptions]+
> - For the purposes of this V1 interaction, the **Twist Torque Wrench** will be the only wrench type used.
> - Members will be able to navigate around the experience in the standardized way ~ rotating with one hand (usually primary hand), and locomotion (teleportation or smooth locomotion) with the other (usually secondary hand).
# Experience{ #twd-experience}


This section goes over the input/output methods a learner will use to familiarize themselves with the torque wrench ~ including visual mapping of controls, specification of interactions, and expected behaviors.
## Controller Mapping

![Quest3_Controllers.png\|594](/img/user/ARVR%20Design%20Challenge/__Resources/Media/Quest3_Controllers.png)
### Primary Controller
- #PrimaryController_Button_Grip : Grabbing and holding objects.
	- Utilizing the grip fingers (index to ring) mimics and reinforces the patterns we use outside of the headset.
- #PrimaryController_Thumbstick_Click : Releasing objects.
	- This button only has one function, as we want to encourage holding onto the torque wrench as long as possible, and in a way that is most realistic to everyday interaction. This button was intentionally set inconveniently.
### Secondary Controller
- #SecondaryController_Button_Grip : Activates additional states of already held objects.
	- Chosen to maintain learner expectations from #PrimaryController_Button_Grip. Allows us to mimic grabbing the torque wrench with two hands and twisting in ways we might naturally do.
- #SecondaryController_Thumbstick_Left & #SecondaryController_Thumbstick_Right : Changes values.
	- Similar to how a learner may scroll on their phone or mouse, the thumbstick mimics selection in a more accessible way. Added this as an additional way to interact with torque value selection and give learners more finite control.
## Specifications

### 00. Preface - Torque Wrench States

Here's an overview of the various torque wrench states. I've also listed the anticipated constraints and how we may educate the learners on navigating them.

```C#
public enum WrenchState { Idle, AdjustingTorque, LockedOnLug, Cranking };
```

| Enum              | State                                                                                                                                                                                     | Constraints/Exit                                                                                     |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| `Idle`            | Default state, or after dropping the wrench.                                                                                                                                              | No input for torque adjustment or cranking active.                                                   |
| `AdjustingTorque` | Holding #SecondaryController_Button_Grip at wrench head and swiping #SecondaryController_Thumbstick_Left or #SecondaryController_Thumbstick_Right.                                        | Prevents locking onto lug nut or cranking.                                                           |
| `LockedOnLug`     | Pressing #PrimaryController_Thumbstick_Click while wrench head is within range of a lug nut.                                                                                              | Prevents torque adjustment and cranking is allowed/applied directly to the lug nut.                  |
| `Cranking`        | Holding #PrimaryController_Button_Grip and rotating controller. <br>- In `AdjustingTorque` ~ changes torque setpoint<br>- In `ReadyToUse` or `LockedOnLug` ~  tightens or loosens lug nut | Torque limit reached --> Success/Fail & Wrench manually released while held, if not in `LockedOnLug` |

### 01. Interactions - Handling

These are the basic interactions a learner will need to handle and manipulate the torque wrench.

- [ ] **Equipping the Wrench**
	- <span style="color:rgb(138, 177, 125)">Activate</span>
		- 1x Press #PrimaryController_Button_Grip 
			- If the learner does not have the wrench equipped in any given hand, and the wrench is within trigger range of a hand, pressing the #PrimaryController_Button_Grip once equips the wrench to that hand.
	- <span style="color:rgb(255, 217, 140)">Effects</span>
		- #Highlight wrench handle when hand is within ra<span style="color:rgb(255, 192, 0)"></span>nge
		- #Haptics_Light when wrench reaches hand
		- #AudioSFX of blunt force when wrench reaches hand
	- <span style="color:rgb(0, 176, 240)">Logic Example</span>
		```C#
		bool _canGrabWrench = Controller.InRange(Wrench) && !TorqueWrench.Equipped;
		
		if (_canGrabWrench && Controller.Grip(type = ActivationType.SinglePress)) { Controller.EquipWrench(); }
		```
- [ ] **Dropping the Wrench**
	- <span style="color:rgb(138, 177, 125)">Activate</span>
		- 3 Second Hold #PrimaryController_Thumbstick_Click 
			- If the learner has the wrench equipped, and the wrench head is not within trigger range of a lug nut, pressing and holding the #PrimaryController_Thumbstick_Click for 3 seconds drops the wrench.
	- <span style="color:rgb(255, 217, 140)">Effects</span>
		 - #Haptics_Light --> #Haptics_Medium as the hold counts from 0 to hold limit, no haptics after limit
		 - #AudioSFX of wind when wrench is released from hand
		 - #AudioSFX of metal when wrench hits the ground
	- <span style="color:rgb(0, 176, 240)">Logic Example</span>
		```C#
		bool _canDropWrench = Controller.IsHolding(Wrench) && !TorqueWrench.InRange(LugNut);
		
		if (_canDropWrench && Controller.Trigger(type = ActivationType.Hold, time = 3f)) { Controller.DropWrench(); }
		```

### 02. Interactions - Adjusting Values

These interactions are mainly focused on adjusting the torque value itself and require additional, realistic manipulation of the torque wrench object.

- [ ] **Setting Torque Value**
	- <span style="color:rgb(138, 177, 125)">Activate</span>
		- Hold #SecondaryController_Button_Grip 
			- While the learner has the wrench equipped, and the other hand is within range of the wrench head, holding the #SecondaryController_Button_Grip grabs the head.
			- This sets the `TorqueWrench.Mode == AdjustingTorque`, allowing the learner to adjust the torque value.
	- <span style="color:rgb(238, 32, 77)">Deactivate</span>
		- Release #SecondaryController_Button_Grip 
			- While `TorqueWrench.Mode == AdjustingTorque`, releasing the #SecondaryController_Button_Grip resets `TorqueWrench.Mode == Idle`, locking in whatever value was last set.<span style="color:rgb(0, 176, 80)"></span>
	- <span style="color:rgb(255, 217, 140)">Effects</span>:
		- #Highlight wrench head when other hand is within range
		- #Haptics_Medium when setting `TorqueWrench.Mode == AdjustingTorque`
		- #AudioSFX of blunt force when wrench is grabbed with other hand
- [ ] **Changing the Torque Value**
	- <span style="color:rgb(100, 250, 204)">Pre-Reqs</span>
		- `TorqueWrench.Mode == AdjustingTorque` for the following to work.
	- <span style="color:rgb(138, 177, 125)">Activate</span>
		1. Hold #PrimaryController_Button_Grip and Rotate Controller
			- The learner may hold #PrimaryController_Button_Grip and rotate their hand to change the value of the torque relative to the direction of rotation (left to decrease and right to increase).
			- Like a wrench, they may release the #PrimaryController_Button_Grip to reset their wrist position, and then hold and rotate as many times as they'd like to change the value.
		2. Swipe #SecondaryController_Thumbstick_Left or #SecondaryController_Thumbstick_Right
			- The learner can also change the torque value by swiping the #SecondaryController_Thumbstick_Left (decrease) or swiping the #SecondaryController_Thumbstick_Right (increase).
	- <span style="color:rgb(255, 217, 140)">Effects</span>
		 - #Haptics_Light as the torque value changes
		 - #AudioSFX of clicking as the torque value changes
		 - #Visual_Dynamic of wrench handle rotating by increments
### 03. Interactions - Components

The last set of interactions are focused on the torque wrench and fitting on the lug nut component.

- [ ] **Setting Wrench Head on Lug Nut**
	- <span style="color:rgb(138, 177, 125)">Activate</span>
		- 1x Press #PrimaryController_Thumbstick_Click
			- If the learner is holding the wrench, and the wrench head is within trigger range of a given lug nut, pressing #PrimaryController_Thumbstick_Click once locks the torque head to the lug nut.
			- This sets the `TorqueWrench.Mode == LockedOnLug`, allowing the learner to move their controller freely, the head always locked on the nut.
	- <span style="color:rgb(238, 32, 77)">Deactivate</span>
		- 1x Press #PrimaryController_Thumbstick_Click 
			- While `TorqueWrench.Mode == LockedOnLug`, pressing the #PrimaryController_Thumbstick_Click resets `TorqueWrench.Mode == Idle`, releasing the torque head from the locked lug nut.
	- <span style="color:rgb(255, 217, 140)">Effects</span>
		- #Highlight lug nut when wrench head is within range
		- #Haptics_Medium when the wrench mode changes
		- #AudioSFX of metal settling when wrench is set in place
- [ ] **Rotating Lug Nut**
	- <span style="color:rgb(100, 250, 204)">Pre-Reqs</span>
		- `TorqueWrench.Mode == Idle` for the following to work.
	- <span style="color:rgb(138, 177, 125)">Activate</span>
		- Hold #PrimaryController_Button_Grip and Rotate Controller
			- The learner can hold #PrimaryController_Button_Grip and rotate their hand to tighten or loosen the nut, relative to the direction of rotation (left to loosen, right to tighten).
			- Like a wrench, they may release the #PrimaryController_Button_Grip to reset their wrist position, and then hold and rotate as many times as they'd like to tighten or loosen the nut.
	- <span style="color:rgb(255, 217, 140)">Effects</span>
		- #Haptics_Light <--> #Haptics_Heavy as the lug nut tightens and loosens
		- #Visual_Dynamic of lug nut tightening and loosening, drag being added as the cranking gets tighter
		- #AudioSFX of lug nut squeaking as it tightens, and stretching as it gets closer to the desired value

## Behaviors
### Learning Metrics

Here are some of the expected states a learner may reach when interacting with the torque wrench. These are great opportunities to affirm and give visual, haptic, auditory feedback on how they're progressing!

- [ ] **Torque Undershot**
	- <span style="color:rgb(200, 250, 250)">Summary</span>
		- Learner has set the `TorqueWrench.TorqueValue < LugNut.RequiredValue`. As they crank, they eventually reach the max value before the torque wrench clicks.
		- The lug not is too loose and eventually falls off. A new lug nut appears in its place.
	- <span style="color:rgb(255, 217, 140)">Effects</span>
		- #Visual_Animation of lug nut too loose and falling off, rolling endlessly to a scale of 0
		- #AudioSFX of torque wrench squeaking tighter and reaching the 'max click'
		- #AudioSFX of lug nut falling onto the ground and rolling endlessly away
- [ ] **Torque Exceeded**
	-  <span style="color:rgb(200, 250, 250)">Summary</span>
		- Learner has set the `TorqueWrench.TorqueValue > LugNut.RequiredValue`. As they crank, they eventually reach the max value before the torque wrench clicks, hearing gnarly metal stripping.
		- The lug nut has been stripped and messes with the orientation of the wheel. A new lug nut reappears in its place.
	- <span style="color:rgb(255, 217, 140)">Effects</span>
		- #Visual_Animation of lug nut too tight and shifting the orientation of the wheel as it hits max; it pops off the wheel quickly
		- #AudioSFX of torque wrench squeaking tighter and reaching the 'max click'
		- #AudioSFX of nut squeaking as it tightens, eventually becoming sharp and painful as metal scratches
- [ ] **Torque Achieved**
	-  <span style="color:rgb(200, 250, 250)">Summary</span>
		- Learner has set the `TorqueWrench.TorqueValue == LugNut.RequiredValue`. As they crank, they eventually reach the max value before the torque wrench clicks.
		- The lug nut is set on snug. The learner may continue.
	- <span style="color:rgb(255, 217, 140)">Effects</span>
		- #Visual_Animation of lug nut fitting in perfectly and 'shinning'
		- #AudioSFX of torque wrench squeaking tighter and reaching the 'max click'
		- #AudioSFX of UI 'success'
# Future Improvements{ #twd-future-improvements}

## Pre-Interview (08/11/2026)

### 01. Accessibility Options
Additional accessibility options to set type of hold and interaction. Following best practices and practicality for engaging with the controller, but what if a learner has a mismatch in capability when inside the headset vs real-life?
- What if they get dizzy in the headset easily?
- What is finger dexterity is strong for the work itself, but not as precise for holding different buttons?
