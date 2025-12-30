# Frost Of Enless Tommorows - Game Design Document

This project was built in **Unreal Engine 5** with a focus on modular systems and dynamic event management to create an immersive and replayable experience with easy-to-expand systems.

![Status](https://img.shields.io/badge/status-playable-success.svg)
![Engine](https://img.shields.io/badge/engine-Unreal%20Engine%205-0E1128.svg)
![Genre](https://img.shields.io/badge/genre-psychological%20horror-lightgrey.svg)
![Architecture](https://img.shields.io/badge/architecture-event--driven-blueviolet.svg)
![Design](https://img.shields.io/badge/design-modular%20systems-blue.svg)
![Platform](https://img.shields.io/badge/platform-Windows-lightgrey.svg)
![Status](https://img.shields.io/badge/status-prototype-blue.svg)

![Gameplay Pictures](https://github.com/th-efool/Horror_FPS/blob/main/screenshot20251230090517.png)

To Play the Game Directly, you can download and run the executable from here: [🎮game.exe](https://drive.google.com/drive/folders/1bwhD_902zfkpq301nAY7nngzucZofkKe?usp=sharing)

To look at the **Source Code**, 
1) clone the repositry 
2) download [3dAssetsZip](https://drive.google.com/drive/folders/1jhBGBf6ltoCyop8PWUh7BP16EmyjV617?usp=sharing)
 & paste it's contents inside the `Content` **Folder** of the cloned repositry.
3) Open the project inside Unreal Engine 5.5.4.



### Game Concept

Play as **Elliot**, a man trapped in a loop of fractured reflections and fragile cause-and-effect. One moment, you're standing in a rotting house where mirrors weep memories. The next, you're lost in a forest where snowflakes fall upward and butterflies shatter like glass. 

Time splinters. Choices ripple.

Each mirror shows a different version of your past—some you remember, some you swear never happened. Behind every reflection hides a truth... or a lie. Your smallest action sends cracks through reality, changing rooms, people, even the rules of the world.

You begin to suspect you're not just reliving time—you're rewriting it. But the more you change, the less you recognize yourself.

**Time loops. Mirror worlds. A butterfly flapping its wings in your mind.**

*Will you escape the echo, or become another reflection, lost forever?*

---

## Core Systems Overview

### Interaction System

The interaction system consists of three main interfaces:

- **IInteract** - Handles interaction with doors, keys, levers, buttons, and other interactable objects
- **ILightSource** - Used to run:
  - `FlickerLight()` - Creates flickering light effects
  - `DestroyBulb()` - Triggered on first encounter with ghost
- **IPlayer** - Used to trigger functions inside the PlayerCharacter script, such as:
  - `Hallucination()` function, which enables the PostProcess component

### Event Manager System

The **EventManagement GameObject + Script** is used to trigger and manage events throughout the game.

#### EventsEnumeration

Each instance of the EventManagerSystem has variables that can be edited based on the instance in the editor.

Based on which boolean is active, aside from the Event in EventIndex getting triggered, you can send/show/trigger multiple functions based on use-case:
- At one location: show message + sound
- At another: sound + hallucination + light flicker

According to the scene's dynamic needs, we can customize the response.

**OnBeginOverlap()** when condition `ActorHasTag("Player")` is met, triggers the Event in EventIndex and other events based on which boolean is active.

#### User Interface System Events

- **LightFlicker**
- **Hallucinations**
- **BlackOut**
- **SoundPlay**
- **ShowMessage**
- **JumpScareEvent**
- **DisablePlayerTorch**
- **Handshake** - Links two scripts together upon contact
- **Teleport** - For creating optical illusions and switching levels

#### Event Manager Variables

- **EventIndex** (Events Enumeration datatype) - Decides which main event the particular EventObject would trigger before getting destroyed
- **SendMessageToo?** (bool)
- **PlaySoundAsWell?** (bool)
- **DisablePlayerLights?** (bool) and **DisableLightsFor** (float)
- **HallucinationsAsWell?** (bool)

---

## PlayerCharacter Script & GameObject

### Key Functionalities

#### Keys/Possessable PickUp System

The following functions handle UI and player interactions:

- `UILoadChapter(Int LevelIndex, Int Duration)` - Shows opening text of every new chapter
- `UIMessageDisplay(Text Text)` - Shows dialogues and informative texts (e.g., "Picked up Kitchen key", "You need ABC to open this door")
- `UIDisplayDiary(Text Title, Text Entry)` - Triggered when player interacts with DiaryActor/NotesActor to display content
- `UIUnDisplayDiary()` - Hides diary display
- `UINoobsGuide()` - Displays UI showing key controls:
  - **RMB** - Interact
  - **X** - Drop Item
  - **LMB** - UV Flashlight

#### Input System

Responsible for taking keyboard inputs and running functionality related to each function.

There's a **SphereCollider** attached that:
- **OnBeginOverlap()** - Checks for any interactable object (Does Object Implement Interface? == IInteract) to give player heads-up with `UINoobsGuide()` for how to interact with the object
- When user presses **InputActionKey for Interact**, we iterate through all overlapping objects to check for (Does Object Implement Interface? == IInteract). If yes, then trigger `Interact(this)` function utilizing the IInteract interface

#### Visual Components

- **PostProcess Component** - Attached to player, enabled every time we wish to display hallucinations and other effects
- **Two Spot Light Components**:
  - One with shorter cone angle (serves as flashlight)
  - Other with bigger cone angle at low illuminosity (provides general ambient illumination) for a much better experience

#### Data Storage

The data of whether player possesses a key/possessable item is stored in a **Bool Array**, whose indexes correspond to values stored in an Enum (e.g., 0=BedroomKey, 1=KitchenKey, etc.).

When `IInteract()` function is run, it updates the BoolArray set value at `(int)EnumGameObject`.

---

## Special Systems

### Infinite Hallways/Staircase

The illusion is created by teleporting the player character around. These systems work with Event Manager, blackouts, and light flicker to disorient the player and prevent them from noticing the change when we swiftly teleport them in the middle of it.

In infinite hallway there are **four event manager objects** - two on each end. The reason for having two of them is that in case after teleportation, if the player tries to break the game by running backwards, the other event manager will teleport them to one of the inner event manager anchors.

### Collectible Mirror Objects

There are mysterious collectible mirrors placed around. Each mirror does something that can serve as a clue for solving puzzles to unlock the mystery:

- **Reversing Movements** - Going forward makes you go backward, left becomes right, etc. (multiplying input-axis value by -1)
- **Reversing Gravity** - We play with character controller script gravity values
- **Color Vision ↔ Black-White** - Playing with post-processing volume attached to player-character
- **Giving Something leads to something taken away from you** - Wrapping reality, reversing the world in various ways

Each mirror has its own mystery.

### Mirror Rendering Optimization

Mirrors are computationally expensive to render, so we tried avoiding using them wherever possible. In the initial scene where the person sees a ghost and a disoriented reflection of themselves, we achieved this by:

1. Creating a **pawn class character** with the same mesh as the main character
2. When the player character steps into an event manager object, it performs a **handshake** between the player-character and the mirror image pawn
3. The input of one character is fed into the other

---

## Technical Notes

I have tried to implement **event-driven systems** instead of running things in the GameLoop Update function, due to the inefficiency of constant polling.



### ⭐ **Star this repository if you found it interesting!** ⭐

Made with ❤️ using Unreal Engine 5.5.4

</div>
