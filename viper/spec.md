# Viper Clone - Game Specification (v1.1)

This document outlines the functional requirements for a 2-player "Viper Clone" game.

## 1. General Game Overview

* **Game Type:** 2-Player, competitive, real-time.
* **Objective:** Be the last player alive. Players are eliminated if their "head" collides with the game boundaries (walls), their own trail, or the opponent's trail.
* **Core Mechanic:** Each player controls a continuously moving entity ("viper") that leaves a persistent trail behind it.

## 2. Game Display & User Interface

* **Game Area:** The game should be rendered on a full-screen display area that dynamically adapts to the browser window size.
    * Background Color: Solid Black (`#000000`).
* **Score Display:**
    * Player 1 (P1) Score: Displayed in the top-left corner. Recommended color: Yellow.
    * Player 2 (P2) Score: Displayed in the top-right corner. Recommended color: Red.
    * Appearance: Text should be clearly legible, e.g., using a pixel-art font like 'Press Start 2P', size 28px, bold, with a subtle shadow.
* **Game Over Screen:**
    * Displayed when both players are eliminated or a game-ending condition is met.
    * Clearly indicates "Game Over" (e.g., "GAME OVER").
    * Provides an instruction to restart the game (e.g., "Press R to restart").
    * Visually distinct, for example, with a semi-transparent overlay on the game area.

## 3. Player Mechanics

* **Number of Players:** 2 (Player 1 and Player 2).
* **Player Representation:** Each player is represented by a "viper" that draws a continuous line (trail) as it moves.
    * **Player 1 (P1):**
        * Trail Color: Yellow (e.g., `#FFFF00`).
        * Initial Position: Starts near the bottom-left of the game area (e.g., 50 units from the left, 50 units from the bottom).
        * Initial Direction: Moves diagonally upwards and to the right (e.g., at an angle of -45 degrees or `-Math.PI / 4` radians relative to the positive x-axis).
    * **Player 2 (P2):**
        * Trail Color: Red (e.g., `#FF0000`).
        * Initial Position: Starts near the bottom-right of the game area (e.g., 50 units from the right, 50 units from the bottom).
        * Initial Direction: Moves diagonally upwards and to the left (e.g., at an angle of -135 degrees or `-3 * Math.PI / 4` radians relative to the positive x-axis).
* **Movement:**
    * Speed: Constant and configurable (e.g., 2.5 units of distance per game update).
    * Vipers move continuously in their current direction.
* **Turning:**
    * Players can change their viper's direction to the left or right relative to its current heading.
    * Turn Rate: The change in angle per game update when a turn key is held should be configurable (e.g., 4 degrees).
* **Trail (Line) Properties:**
    * Thickness: Configurable (e.g., 4 units).
    * Appearance: Trails should have rounded ends and joins for a smoother look.
* **Gaps in Trail:**
    * Vipers should periodically and automatically stop drawing their trail for a short distance, creating gaps.
    * Gap Length: The length of these gaps should be configurable (e.g., 20 units of distance).
    * Gap Frequency: Gaps should appear after the viper has traveled a configurable distance drawing its trail (e.g., every 150 units of distance).

## 4. Collision System

* **General:** Collision detection should determine if a player's head intersects with game boundaries or any player's trail.
* **Wall Collision:**
    * If a player's head touches or goes outside the game area boundaries, that player is eliminated.
    * Upon wall collision, if the player was not in a gap, their trail should be visibly drawn up to the point of impact with the boundary.
* **Trail Collision (Opponent):**
    * If a player's head collides with the opponent's trail, that player is eliminated.
    * Collision detection should be sensitive enough to register contact with the opponent's trail color. A configurable tolerance for color matching can be considered.
* **Self-Collision (Own Trail):**
    * If a player's head collides with their own previously drawn trail, that player is eliminated.
    * This check should use a very strict (minimal) tolerance for color matching.
    * **Important Nuance for Self-Collision:** Players should not immediately collide with the rounded end of their own trail segment that was drawn in the immediately preceding game update, especially during sharp turns. The system must differentiate between hitting this "fresh cap" and genuinely crossing an older part of the trail.
    * Self-collision detection should be active from the very first frame of the game; there should be no artificial grace period or distance threshold before it becomes active.
* **Collision Integrity:**
    * Collision checks must consider the entire path segment a player moves in a single game update to prevent "tunneling" (passing through thin obstacles at high speed).
    * Areas of the canvas that are transparent or nearly transparent (e.g., due to anti-aliasing against the background) should not trigger trail collisions.

## 5. Scoring System

* **Earning Points:** Players should earn points based on survival time (e.g., 1 point for approximately each second they remain alive).
* **Score Reset:** Scores for both players must be reset to zero at the beginning of each new game.

## 6. Game Flow & State Management

* **Initialization:**
    * At the start of a new game, all relevant game state variables must be reset. This includes, but is not limited to: player scores, player positions and angles, trail status (including gap logic), and player alive/dead status.
    * The game area should be cleared, and any "Game Over" messages hidden.
* **Game Loop:**
    * The game must run in a continuous loop, updating at a consistent rate for smooth animation and responsive controls.
    * Each update cycle should:
        1.  Process player inputs for turning.
        2.  Update player positions and trail data.
        3.  Perform collision checks (walls, opponent trails, self trails).
        4.  Update player alive/dead status based on collisions.
        5.  Draw all game elements (trails, player heads if distinct, scores).
        6.  Update scores for surviving players.
        7.  If a game-ending condition is met (e.g., both players eliminated), transition to the "Game Over" state.
* **Game Over State:**
    * When the game ends, all active viper movement and trail drawing should cease.
    * The "Game Over" screen should be displayed.
    * The game should await a restart command.
* **Restart:**
    * Pressing the designated restart key (e.g., 'R') while the "Game Over" screen is active must trigger the game initialization process and start a new game.

## 7. Controls

* **Player 1:**
    * Turn Left: 'Q' key.
    * Turn Right: 'W' key.
* **Player 2:**
    * Turn Left: 'O' key.
    * Turn Right: 'P' key.
* **Game Restart:**
    * 'R' key (active only when the "Game Over" screen is displayed).

## 8. Configurable Parameters

The game should allow for easy configuration of the following parameters:

* Player movement speed.
* Thickness of player trails.
* Background color of the game area.
* The rate at which players turn (e.g., degrees or radians per game update when a turn key is held).
* Distinct trail colors for Player 1 and Player 2.
* Initial starting positions (e.g., offsets from edges) and initial movement angles for both players.
* Parameters defining trail gaps: length of gaps and frequency of appearance (based on distance traveled).
* A general color matching tolerance for detecting collisions with opponent trails.

## 9. Key Gameplay Considerations

* **Fairness in Self-Collision:** The system for handling self-collision needs to be robust. It must prevent players from dying by merely touching the rounded end of the trail segment they just created, especially during intended sharp turns. However, it must reliably detect when a player's head crosses an older, established part of their own trail.
* **Collision Reliability:** Collision detection should be consistent and feel fair to the players, accurately reflecting visual interactions on screen.
