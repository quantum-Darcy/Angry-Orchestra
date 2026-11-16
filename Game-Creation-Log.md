# Game Creation Log

-Decided on game concept and features I wanted to include

-Listed all features and choose the order I would implement them.
 -Background, music and sprites
 -Sprite Tracking
 -Collision detection
 -You lost screen
 -Increasing difficulty

-Background, music and sprites: Background and sprites where taken from scratch's in-built library, except from Mozart sprite which was screenshotted from  [Internet archieve](https://archive.org/details/wolfgang-amadeus-mozart-vol-1) and the music was downloaded from [
Mozart - Eine Kleine Nachtmusik / Serenade No. 13 - KV 525.mp3](https://freemusicarchive.org/music/gregor-quendel/classical-music-collection/mozart-eine-kleine-nachtmusik-serenade-no-13-kv-525mp3-1/)

# Problems fixed without Claude

- When you lose I wanted the music to stop however when I programmed the music to stop when caught the start music command was in a forever loop causing the music to restart. So first I made it so when the caught message was broadcasted it caused a different code block to loop the caught message forever. However this caused the score counter to increase forever on the lose screen which I didn't like, so used hide variable to remove the score on the you lose screen

- You weren't able to move your sprite at the start of the game which caused you to lose instantly so I set a delay on the attackers to reduce the chances of you instantly losing. However this created an exploit that you could get rid of the attackers without losing as I made the delay to long so I reduced it to the perfect amount of 2 down from 4.



> **⚠️ AI-generated content:** Below is a summary created by AI on what problems I encountered and how I used Claude if I needed to get unstuck, based on a tutoring session with me (Darcy) on 15-16 November 2026. I built and debugged the game myself in Scratch; Claude acted as a tutor. It mostly gave hints and guiding questions rather than finished code. Each section below says exactly how the AI helped.

---

## 1. Sprite Tracking

### Problem: The trumpet didn't turn back towards Mozart after hitting the edge
**What happened:** The trumpet was supposed to turn towards Mozart whenever it touched the edge of the stage, but it never changed direction.

**Cause:** The `if touching edge?` check wasn't inside a loop, so Scratch only checked it once, when the green flag was clicked. Both branches of the `if/else` then went into their own `forever` loops, so the script got stuck and never checked again.

**How I fixed it:** My first attempt wrapped everything in an outer `forever` loop, but it still didn't work, because the `forever` loops *inside* the `if/else` never finished, so the outer loop never got another turn. I removed the inner loops and ended up with:
- one `forever` loop
- inside it: `if touching edge? then point towards Mozart`
- then `move 2 steps`

**How AI helped:** Claude didn't give me the fix. It asked me to trace the script block by block and work out which blocks could ever notice the edge. When my second attempt failed, it explained that a `forever` inside a `forever` acts like a trapdoor that never lets the script out, and asked what each branch needed to do once per loop. I worked out the final script myself.

**What I learned:** A check only keeps working if it's inside a loop. Most games use the same shape: one loop that checks, then acts, every time round (in Python, a `while True:` loop).

---

## 2. Collision Detection

### Problem: Hits weren't detected while dragging Mozart with the mouse
**What happened:** When I dragged Mozart onto the trumpet or the drum with the mouse, the game didn't register that he'd been hit.

**Cause:** While you drag a sprite, Scratch hides the real sprite and moves a picture of it around. The sprite only really moves when you let go, so `touching` can't detect anything mid-drag.

**How I fixed it:** I stopped dragging Mozart and moved him with code instead, so his real position is always on the stage.

**How AI helped:** Claude explained why dragging doesn't count as movement and suggested a test: drop Mozart on top of the trumpet and see whether the hit registers when you let go. It pointed me to the right Motion block, but left building the script to me.

### Problem: The lose screen only worked sometimes
**What happened:** When the trumpet hit Mozart, the game sometimes went to the lose screen and sometimes didn't.

**Cause:** It was the same dragging problem. I was still clicking and dragging Mozart while testing, so the hit only counted if I happened to let go while he was overlapping the trumpet.

**How I fixed it:** I moved Mozart with code only, so every hit is now detected.

**How AI helped:** Claude explained that a bug that only happens "sometimes" means something differs between the runs that work and the ones that don't, and asked me to find that difference. It listed possible suspects (dragging, pixel-perfect collision, another script switching the backdrop back) and suggested tests for each. I recognised that dragging was the cause.

### Debugging technique: checking whether the code is actually running
**How AI helped:** Claude showed me how to add a temporary `say Hit!` block inside the `if` to check whether the collision code is actually being reached. This splits the bug into two possibilities: the hit isn't being detected, or the hit is detected but the code that follows doesn't work.

---

## 3. Lose Screen

### Problem: Getting hit didn't properly end the game
**What happened:** My lose code was `stop all sounds` plus `switch backdrop`, but the game kept going.

**Cause:** `stop all sounds` only stops the sound. Every other script keeps running: the chasers keep moving, and the Stage's music loop just starts the song again.

**How AI helped:** Claude asked me to trace what every *other* script was doing at the moment of the hit, and hinted that the Control category has a block that stops a lot more than sounds.

### Problem: Only the sprite that hit Mozart disappeared
**What happened:** I wanted both the trumpet and the drum to disappear when either one hit Mozart, but only the one that touched him vanished. The other kept moving.

**Cause:** Each chaser had its own `if touching Mozart then hide` check, so each one only reacted to itself. A sprite can only hide itself, not another sprite.

**How I fixed it:**
- In Mozart's collision check (`touching Drum? or touching Trumpet?`), I added `broadcast caught`.
- The trumpet and the drum each got a separate script: `when I receive caught` then `hide`.
- I added `show` at the start of their green-flag scripts so they reappear when the game restarts.

**How AI helped:** Claude first gave me a worked example of broadcasting in a different game (one sprite announces an event and every sprite listening reacts). When I asked for more help, it gave me step-by-step instructions for the trumpet and left the drum for me to do the same way. It also warned me that the sprites would stay hidden on the next play without a `show` block.

**What I learned:** Broadcasting lets one sprite notice an event and tell all the others, instead of every sprite checking for itself.

---

## 4. Increasing Difficulty

### Goal: Each time the song loops, the chasers speed up, Mozart says they're getting angrier, and the music gets louder

**Detecting the end of each loop of the song:** On the Stage, I used `play sound until done` inside a loop. The spot just after that block is the exact moment each loop of the song ends, so anything placed there happens once per loop.

**Bug found:** I originally used `change volume by -10`, which makes the music *quieter*. It needed a positive number to get louder.

**Final solution:**
- I made a variable called `speed` set to "For all sprites", because the Stage changes it while the trumpet and drum read it.
- In the Stage's green-flag script, `set speed to 2` comes *before* the loop, so speed resets each game.
- Inside a `repeat until` loop: `play sound Mozart until done`, then `change speed by 1`, `change volume by` a positive number, and `broadcast angrier`.
- The trumpet and drum use `move (speed) steps` instead of `move 2 steps`.
- Mozart has a `when I receive angrier` script that makes him say they're getting angrier.

**Key question I asked: did `change speed by 1` need to go inside an `if` block?**
No. `play sound until done` makes the script wait until the song finishes, so reaching the next line already means one loop of the song has ended. There's nothing to check.

**What I learned:** Use an `if` when something might or might not be true at any moment. Use the **order of the blocks** when one thing always happens after another.

**How AI helped:** Claude broke the feature into four smaller problems. It pointed out that a fixed number like `2` can never change, which led me to variables. It gave me a worked example from a different game (a ball that gets faster each time it bounces) and asked me to apply the same pattern to mine. It then gave me a plan with gaps for me to fill in, explained why an `if` wasn't needed, and confirmed my structure for the Stage script.

---

*Summary of AI use: Claude acted as a tutor throughout. It mostly asked questions, gave hints and showed worked examples from other games, and gave more direct step-by-step instructions only when I asked for more help. I built, tested and debugged all the Scratch code myself. This log was written by Claude from that session.*
