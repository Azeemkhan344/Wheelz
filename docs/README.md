## Project overview

This project is a Unity-based 2D, grid-style maze runner featuring a hamster character. It includes a central `GameManager` singleton that orchestrates game state, levels, scoring, coins, lives, timers, and daily rewards, and a `PlayerController` that drives grid-based movement, input, and pickups.

### Main components
- **`GameManager`**: Singleton service that manages global state, level flow, score/coins/lives, time limit, persistence via `PlayerDataManager`, UI handoff via `UIManager`, and leaderboard submission via `LeaderboardManager`.
- **`PlayerController`**: Player movement and interaction controller. Handles grid-based motion, one-tap direction queues, collisions with pickups/obstacles/exit, simple animation flags, and optional speed boosts.

## Setup

### Scene setup
- Add a persistent `GameObject` in your initial scene (e.g., `MainMenu`) and attach `GameManager` to it. Ensure it is not duplicated across scenes.
- Fill `GameManager.levelScenes` with the playable level scene names and include them in Build Settings.
- Create or ensure the following tags and layers exist and are assigned on relevant scene objects:
  - **Tags**: `Coin`, `Star`, `Obstacle`, `Exit`
  - **Layer**: `Wall` (used by `PlayerController` raycasts)

### Player setup
- Add a `Rigidbody2D` (Dynamic or Kinematic as fits your physics setup) to the player object.
- Attach `PlayerController` to the player object.
- Assign optional references in the inspector:
  - `hamsterSpriteRenderer`
  - `hamsterAnimator`
- Tune `moveSpeed` and `gridSize` for your maze layout.

### External managers (required by `GameManager`)
Implement or provide the following singletons referenced by `GameManager`:
- `PlayerDataManager.Instance`: persistence of coins, lives, last played level, high scores, unlocked levels.
- `UIManager.Instance`: presents/hides pause, game over, level complete, and daily reward UI.
- `LeaderboardManager.Instance` (optional): submits scores.

## Quickstart

### Start a game from the Main Menu
- Add a UI Button and hook its OnClick to `GameManager.StartGame`.
- Ensure `GameManager.levelScenes` includes your first level at index 0.

```csharp
// Example: Wiring from code if needed
using UnityEngine;
using UnityEngine.UI;

public class MenuBindings : MonoBehaviour
{
  public Button playButton;
  void Awake() => playButton.onClick.AddListener(() => GameManager.Instance.StartGame());
}
```

### Subscribing to score/coins/lives/time updates
```csharp
void OnEnable()
{
  GameManager.Instance.OnScoreChanged += HandleScoreChanged;
  GameManager.Instance.OnCoinsChanged += HandleCoinsChanged;
  GameManager.Instance.OnLivesChanged += HandleLivesChanged;
  GameManager.Instance.OnTimeChanged += HandleTimeChanged;
}

void OnDisable()
{
  GameManager.Instance.OnScoreChanged -= HandleScoreChanged;
  GameManager.Instance.OnCoinsChanged -= HandleCoinsChanged;
  GameManager.Instance.OnLivesChanged -= HandleLivesChanged;
  GameManager.Instance.OnTimeChanged -= HandleTimeChanged;
}
```

### Pause/Resume from UI
```csharp
public void OnPause() => GameManager.Instance.PauseGame();
public void OnResume() => GameManager.Instance.ResumeGame();
```

### Daily rewards
- `GameManager.CheckDailyReward()` should be called on the main menu to prompt users if a reward is available.
- Claim via `GameManager.ClaimDailyReward()`.

## Links
- API docs: see `docs/api/GameManager.md` and `docs/api/PlayerController.md`.
