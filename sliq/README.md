# SLIQ

SLIQ is a tile-based puzzle game where you swipe numbered tiles across a rotating board to score by matching tile colors with the border. It blends strategy with timing as the colored border rotates and new tiles drop in.

## Game Instructions

- **Goal**: Reach the target score before time runs out or the board fills up.
- **Tiles**: Values 1–4. Each swipe left/right moves the tile and reduces its value by 1.
- **Scoring**: When a tile’s color matches the border color beneath it, it drops through and scores points equal to its current value.
- **Rotation**: The border rotates clockwise on a timer. You can also rotate manually using the lever.
- **New Tiles**: Fresh tiles drop in when the board rotates.
- **Stars**: Levels award 0–3 stars based on performance (roughly 33%, 65%, and 100% of target).
- **Progression**: 3 bundles (Easy/Medium/Hard), 12 levels each, with star-based unlocks. Free Play unlocks after completing all 36 levels.

## How To Run

1. Open `Sliq.xcodeproj` in Xcode.
2. Select an iOS Simulator (or device) and build/run.

## Project Layout

- `Sliq/Scenes/` — Menu, level select, settings, gameplay scenes
- `Sliq/Models/` — Board, tiles, border system, game state
- `Sliq/Assets/Textures/` — Core art assets (tiles, UI, characters)

## Asset Showcase

Below is a visual tour of most of the in-game assets.

### App Icon + UI Elements

<img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/icon.png" width="96"> <img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/icon-no-background.png" width="96"> <img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/icon-no-background-no-bear.png" width="96"> <img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/lever.png" width="72"> <img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/lever_flip.png" width="72">

### Characters + Feedback

<img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/teddy.png" width="96"> <img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/teddy-alive.png" width="96"> <img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/empty-head.png" width="64"> <img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/filled-head.png" width="64"> <img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/body.png" width="64"> <img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/guts.png" width="64"> <img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/beating-heart.gif" width="64">

### Tiles (Values 0–4)

<img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/0-tile.png" width="72"> <img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/1-tile.png" width="72"> <img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/2-tile.png" width="72"> <img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/3-tile.png" width="72"> <img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/4-tile.png" width="72"> <img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/tile.png" width="72">

### Boards + UI Screens

<img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/empty-board.png" width="160">

### Industrial Theme Elements

<img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/pipes.png" width="200"> <img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/pipes-no-belt.png" width="200">

### Tile Animation GIFs

<img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/gifs/1-tile-left.gif" width="72">
<img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/gifs/2-tile-left.gif" width="72">
<img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/gifs/3-tile-left.gif" width="72">
<img src="https://github.com/BowerHarry/sliq-iOS/blob/main/Sliq/Assets/Textures/gifs/4-tile-left.gif" width="72">

### In-game screengrabs

#### Landing screen
<img src="https://github.com/user-attachments/assets/1fa967ef-1381-47f8-b504-65dcf3311f95" width="200">

#### Level select
<img width="200" height="500" alt="IMG_3231" src="https://github.com/user-attachments/assets/a046598f-0050-4780-bf12-7625218f21b5" />

#### Gameplay
<img src="https://github.com/user-attachments/assets/3c45a873-6fcd-4301-8b58-7265683e1366" width="200">
