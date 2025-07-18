# Shadow - Soul-like ARPG Game

**CPS 4881 Final Project**  
**Project Team:** Zhu Lianjie, Zhang Xue

## Demo Video
[Watch Demo Video](https://www.bilibili.com/video/BV1cDGLeMEf5/?spm_id_from=333.999.0.0&vd_source=8ffffca7886aebad1a4ab0f6ff17ccb9)

## Project Background

### Soul-like ARPG Shadow
As a milestone in the ARPG genre, Soul-like games have produced many well-known titles that have been recognized. Elden Ring won the 2023 TGA Game of the Year award for its outstanding design and innovation. Inspired by these classics, our team decided to create an ARPG game with a cyberpunk near-future worldview, aiming to combine the core features of Soul-like games with unique cyberpunk elements to bring players a new experience.

In the game design, we borrowed heavily from the map design concept of Souls-like games. Souls games are known for their intricate map layouts and high degree of freedom of exploration, which allow players to immerse themselves in the game world and experience unique adventures. Therefore, we have incorporated a lot of vertical stereoscopic map elements into the level design, making the whole game world more three-dimensional and multi-layered.

## Game World Setting

### Cyberpunk Near-future Worldview
Our game is set in a cyberpunk-style near-future metropolis where the contrast between high-tech and low-life is striking, with neon-lit skyscrapers and dark alleyways. Players take on the role of a character on a special mission as they embark on an adventure in a city full of crises and opportunities.

### Vertical Stereoscopic Map Design
In order to enhance the exploratory and challenging nature of the game, we have adopted a vertical three-dimensional structure in the map design. Not only can players explore freely on the ground, but they can also enter the underground world in various ways or climb to the top of a tall structure. This design not only enriches the game's scene and layering, but also increases the depth and fun of exploration.

## Core Gameplay Features

### Difficult Combat
Continuing the difficult combat style of Souls-like games, enemies will have highly intelligent and diverse attack methods, and players will need precise controls and strategies to defeat strong enemies.

### Free Exploration
With an open-ended map design, players can freely choose the path to explore and discover hidden secrets and treasures.

### Cyberpunk Elements
Combining cyberpunk-style visuals and technological elements, the game will have a wealth of futuristic technological equipment and transformation systems.

### Character Development
Diverse character development paths, players can choose different skills and equipment according to their preferences, creating a unique fighting style.

### In-depth Storyline
With a fascinating storyline and rich world viewing, players will gradually uncover the truth of the game world through the main quest and side quests.

## Project Goals

By combining the core features of the Souls genre with a cyberpunk near-future worldview, we wanted to create an ARPG that was both challenging and creative, bringing players a new gaming experience. Whether it's the thrill of combat or the freedom of exploration, we're committed to creating an unforgettable game world.

## Team Arrangement

### Core Development Team
- **Zhu Lianjie**: Coding, Battle planning, Mapping, Mapping planning, Some parts of AI (Melee Enemy), HUD
- **Zhang Xue**: Coding, Some parts of AI (Boss), UI (Main page)
- **Andy Chen**: Guidance

### Other Team Members
- **Zhou Shanghao**: Music
- **Ma Yuehang**: Map design
- **Lei Yutian**: Map Art
- **Chen Junxi**: Character Art

# Zhang Xue's Blueprint Implementation - Shadow ARPG

## Added a feature where melee enemies' bodies gradually disappear after being hit and killed (fixed the bug where they fly twice)

```blueprint
// Enemy Death Timeline
Event Take Damage →
    Current Health <= 0 →
    Set Collision Enabled: No Collision →
    Set Physics: Simulate Physics = False →
    Timeline: Body Fade →
        Track: Alpha (0.0 to 1.0, 3.0 seconds) →
        Set Scalar Parameter Value On Materials →
            Parameter Name: "Opacity" →
            Value: 1.0 - Alpha →
    Timeline Finished →
        Destroy Actor

// Material Setup for Fade
Event BeginPlay →
    Create Dynamic Material Instance →
        Source Material: M_Enemy_Material →
    Set Material: Dynamic Material Instance
```

## Created a new kind of melee enemy (Includes patrol, combat, death, etc. logic)

```blueprint
// BP_Enemy_Dummy Components
Components:
├── Capsule Component
├── Mesh (Character): SK_Mannequin
├── Widget Component (Health Bar)
└── AI Controller Class: BP_SWAIController

// Blackboard Keys
Target: Object
PatrolPoint: Vector  
EnemyState: Enum
LastKnownLocation: Vector

// Behavior Tree Structure
ROOT →
    Selector →
        Blackboard Based Condition: Player In Range →
            BTT_AttackPlayer →
        Sequence: Patrol Logic →
            BTT_MoveTo: PatrolPoint →
            Wait: 3.0 seconds →
            BTT_SetNextPatrolPoint
```

## Fixed the bug with the stairs from the first floor to the second floor

```blueprint
// Stair Collision Fix
Event BeginPlay →
    Set Collision Response To Channel: Pawn = Block →
    Set Collision Object Type: WorldStatic →
    
// Player Movement on Stairs
Event ActorBeginOverlap →
    Other Actor: BP_Player →
    Set Movement Mode: Walking →
    Set Max Step Height: 45.0
```

## When approaching the door imprisoning the boss, a trigger box and UI pop-up display: "Please search the 3 glowing objects in the factory to open the door"

```blueprint
// BP_BossDoorTrigger
Event ActorBeginOverlap →
    Other Actor →
    Cast To BP_Player →
    Is Valid →
        Get Game Instance →
        Get Glowing Objects Collected →
        Branch: Objects Collected < 3 →
            True →
                Create Widget: WBP_BossMessage →
                Set Text: "Please search the 3 glowing objects in the factory to open the door" →
                Add To Viewport →
                Set Timer By Function Name: HideMessage (5.0s)
            False →
                Play Animation: Door Open
```

## Created the boss monster and replaced it with a suitable model later (detects the player, tracks the player, attacks the player, summons reinforcements to join the battle when health is low, death logic)

```blueprint
// Boss Detection System
Event Tick →
    Get All Actors Of Class: BP_Player →
    Get Distance To: Player →
    Branch: Distance < 2000.0 →
        Set Blackboard Value: Target = Player →
        Set Blackboard Value: PlayerDetected = True

// Boss Health-Based Reinforcement
Event Take Damage →
    Current Health - Damage →
    Set Health →
    Branch: Health < (Max Health * 0.3) →
        True →
            Branch: Reinforcements Not Spawned →
                Spawn Actor From Class: BP_Enemy_Dummy →
                    Location: Boss Location + (500, 0, 0) →
                Spawn Actor From Class: BP_Enemy_Dummy →
                    Location: Boss Location + (-500, 0, 0) →
                Set Bool: Reinforcements Spawned = True

// Boss Attack Logic  
BTT_BossAttack →
    Try Activate Ability By Class: GA_BossAttack →
    Get Play Length: Attack Montage →
    Delay: Animation Length →
    Finish Execute: Success
```

## Added two enemies patrolling near the glowing boxes to protect them from being hit by the player

```blueprint
// Guard Spawn System
Event BeginPlay →
    Get All Actors Of Class: BP_GlowingObject →
    For Each Loop: Glowing Objects →
        Spawn Actor From Class: BP_GuardEnemy →
            Location: Object Location + Random Vector (Range: 300-500) →
        Set Blackboard Value: GuardTarget = Current Glowing Object →
        Set Blackboard Value: PatrolRadius = 400.0

// Guard Patrol Logic
BTS_GuardPatrol →
    Get Blackboard Value: GuardTarget →
    Get Actor Location: Target →
    Make Vector: Random Point In Circle →
        Center: Target Location →
        Radius: 400.0 →
    Set Blackboard Value: PatrolPoint
```

## Fixed the bug where the UI above enemies' heads was always displayed in the scene

```blueprint
// Enemy Widget Visibility Control
Event BeginPlay →
    Set Widget Visibility: Hidden

// Camera-Based Display
Event Player Camera Update →
    Get Player Camera Manager →
    Line Trace By Channel →
        Start: Camera Location →
        End: Enemy Location →
    Branch: Hit Result = This Enemy →
        True → Set Widget Visibility: Visible
        False → Set Widget Visibility: Hidden
```

## Fixed the bug where locking onto the enemy with the Q key caused player and monster twitching

```blueprint
// BP_CursorDirection - Smooth Lock-On
Event Q Key Pressed →
    Find Closest Enemy In Range →
    Branch: Valid Enemy Found →
        Timeline: Smooth Camera Turn →
            Track: Alpha (0.0 to 1.0, 0.5s) →
            Lerp: Current Rotation to Target Rotation →
            Set Control Rotation: Lerped Rotation →
        Set Bool: Is Locked On = True

// Anti-Jitter System
Custom Event: OnDirectionChange →
    Detection Sensitivity: 100.0 →
    Branch: Mouse Movement > Sensitivity →
        Update Target Lock →
    Else →
        Maintain Current Lock
```

## Added game music and sound effects; the volume can be adjusted in the UI's audio settings

```blueprint
// WBP_UserInterface Audio Controls
Event Volume Slider Value Changed →
    Get Slider Value →
    Set Sound Class Volume →
        Sound Class: Master →
        Volume: Slider Value →
    Set Sound Class Volume →
        Sound Class: Music → 
        Volume: Slider Value →
    Set Sound Class Volume →
        Sound Class: SFX →
        Volume: Slider Value

// Audio Assets Integration
Sound Assets:
├── Change Page Button: Cue_Ch
├── Main Menu Button: Cue_Me
└── Tab Button: Cue_Tal

// Save Audio Settings
Event Apply Audio Settings →
    Create Save Game Object →
    Set Master Volume: Slider Value →
    Set Music Volume: Music Slider Value →
    Set SFX Volume: SFX Slider Value →
    Save Game To Slot: "AudioSettings"
```

## Fully designed and programmed the game's start screen UI (adjustable resolution, window size, screen brightness, etc.)

```blueprint
// WBP_UserInterface Main Menu System
Event Initialize →
    Populate Resolution Dropdown →
    Set Default Values →
    Bind Button Events

// Resolution Control
Event Resolution Dropdown Changed →
    Get Selected Option →
    Parse Resolution String →
    Set Screen Resolution →
        Size X: Parsed Width →
        Size Y: Parsed Height

// Window Mode Toggle
Event Window Mode Button Clicked →
    Get Current Window Mode →
    Branch: Is Fullscreen →
        True → Set Window Mode: Windowed
        False → Set Window Mode: Fullscreen

// Brightness Control  
Event Brightness Slider Changed →
    Get Slider Value →
    Get Post Process Settings →
    Set Exposure Compensation: Slider Value →
    Apply Post Process Settings

// Settings Save/Load
Event Save Settings →
    Create Save Game Object →
    Set Resolution: Current Resolution →
    Set Window Mode: Current Mode →
    Set Brightness: Current Brightness →
    Save Game To Slot: "GameSettings"

Event Load Settings →
    Load Game From Slot: "GameSettings" →
    Apply Saved Resolution →
    Apply Saved Window Mode →
    Apply Saved Brightness
```
