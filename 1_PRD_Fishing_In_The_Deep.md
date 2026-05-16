# Fishing In The Deep - Definition

# Definitions

## ⚙️setup

- the goal of this PA is to earn money,
- earn money by catching different fish
- the PA will have 3 rounds of fishing after the 3rd throw is done the PA will transfer to end screen
- Coin counter will be on screen initially
- Player will start with 100 coins (🪙)
- fish counter :
  - will **not** be on screen initially,
  - will be shown when the hook is going down, replacing the coin counter
  - it’ll start with 0, and each caught fish will add 1 to the counter, the counter will have a fill bar that will be filled relatively with each fish caught, showing the player how many fish he caught and how much is left to catch.
- the player will have two upgrade buttons that he can purchase before fishing:
  - max fish - how many fish the player can catch in a single throw
    - will start with 6 fish
    - cost: 7 fish 50🪙, 8 fish 100🪙, 9 fish 200🪙, 10 fish 400🪙
  - max depth - how deep the hook will go at “MAX” throw
    - will start with 5m
    - cost: 10m 50🪙, 15m 75🪙, 20m 150🪙, 30m 300🪙
- the play button will have a gauge that have a min-max-min setup with a slider/pointer on it , the pointer will move on the gauge from side to side in a pendulum motion slowly (ease in& out at the end of each side), where the pointer stops when the player click the play button determines how deep the hook will go
  - max depth is defined by initial setup / upgrades
  - min depth will be the max depth-2m. anything in between will be accordingly.
- fisherman and fishes need to be in 3D but no real assets are required - can be a basic 3D shape

## 🕹️gameplay

- fishing is done by clicking the “Play” button and stopping the pointer on the gauge
- when the fishing start (player press the button), coin counter will be replaced with fish counter
- the hook is going down the selected depth (max depth is not visible in the frame at start)
- Camera following the hook until reaching the desired depth
- Once the hook reached the selected depth:
  - it will hold for a brief moment (0.5 - 1 sec),
  - the first round/fishing will have longer pause (2 seconds) to allow a short tutorial title to show up when the hook is paused, “move the hook to catch fish”, the title disappears and game resumes.
- Then the hook will start moving upwards allowing the player to drag it from side to side catching fish as the hook move to the surface
- when a fish is caught an effect will be played with a bouncy amount of the fish worth
- some fish will have an additional title pop up when they are caught like : rare!, amazing!, legendary! (higher depth better chance) etc..
- the caught fish will be hooked on the hook, showing the player his catch, in addition and the fish counter will be updated
- once the max amount of fish are caught the hook will speed up to surface, not collecting any other fish on the way,
- just before the hook+fish arrive to the surface the fish counter will be replaced by the coin counter.
- once the hook arrived to the surface:
  - the fish are being summed with a jumping animation of the fish,
  - amounts are showing for each fish and attaboys titles if they have any (rare, amazing etc),
  - a coin burst effect from each fish
- once all the fish animation are done a summery will be shown with the total amount (with a slight counter action) of the catch
  - coins will fly from the amount to the coin counter
  - coin counter is updated
  - the game reset to the next round (play button re-appears)