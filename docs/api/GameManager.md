## GameManager API

`GameManager` is a `MonoBehaviour` singleton that coordinates game flow, levels, UI signaling, persistence, and player progression.

### Singleton
- **`static GameManager Instance { get; }`**: Global access to the single instance.

### State and configuration
- **`enum GameState { Menu, Playing, Paused, GameOver, LevelComplete }`**
- **`GameState CurrentState { get; private set; }`**
- **`int currentLevelIndex`**: The active level index (0-based). Updated on scene load.
- **`string[] levelScenes`**: Ordered list of level scene names. Must be in Build Settings.
- **`float timeLimitPerLevel`**: Seconds allowed per level.

### Player progression
- **`int Score { get; private set; }`**
- **`int Lives { get; private set; }`**
- **`int Coins { get; private set; }`** (total across sessions)
- **`int CurrentLevelCoins { get; private set; }`** (collected in current level)

### Events
- **`event Action<int> OnScoreChanged`**
- **`event Action<int> OnLivesChanged`**
- **`event Action<int> OnCoinsChanged`** (total coins)
- **`event Action<int> OnCurrentLevelCoinsChanged`**
- **`event Action<GameState> OnGameStateChanged`**
- **`event Action<float> OnTimeChanged`** (remaining time in seconds)

Subscribe/unsubscribe in `OnEnable/OnDisable` to avoid leaks.

```csharp
void OnEnable()
{
  var gm = GameManager.Instance;
  gm.OnScoreChanged += UpdateScoreUI;
  gm.OnLivesChanged += UpdateLivesUI;
  gm.OnTimeChanged += UpdateTimerUI;
}

void OnDisable()
{
  var gm = GameManager.Instance;
  gm.OnScoreChanged -= UpdateScoreUI;
  gm.OnLivesChanged -= UpdateLivesUI;
  gm.OnTimeChanged -= UpdateTimerUI;
}
```

### Public methods

- **`void StartGame()`**
  - Loads the last played level (from `PlayerDataManager`) or level 0 and enters `Playing`.
  - Resets `Score`, `CurrentLevelCoins`, timer; emits change events.

- **`void AddScore(int amount)`**
  - Increments `Score` by `amount` and emits `OnScoreChanged`.

- **`void CollectCoin(int amount = 1)`**
  - Increments `CurrentLevelCoins` and total `Coins` by `amount`.
  - Emits `OnCurrentLevelCoinsChanged` and `OnCoinsChanged` and persists coins.

- **`void CollectStar()`**
  - Shortcut that calls `AddScore(100)`.

- **`void LoseLife()`**
  - Decrements `Lives` and persists. If `Lives <= 0`, enters `GameOver` and shows UI; otherwise restarts the current level.

- **`void GrantExtraLife(int amount = 1)`**
  - Increments `Lives` and persists. If recovering from game over, hides game over UI and restarts.

- **`void LevelCompleted()`**
  - Enters `LevelComplete`, persists coins, updates high score, unlocks next level, sets last played, submits leaderboard, and shows level complete UI.

- **`void NextLevel()`**
  - Loads the next level if available; otherwise returns to main menu.

- **`void RestartLevel()`**
  - Reloads the current level, resetting level-local variables and timer.

- **`void PauseGame()` / `void ResumeGame()`**
  - Toggles `Paused`/`Playing`, sets `Time.timeScale`, and updates UI via `UIManager`.

- **`void GoToMainMenu()`**
  - Ensures normal timescale, sets `Menu`, and loads the `MainMenu` scene.

- **`void SetState(GameState newState)`**
  - Centralized state setter; emits `OnGameStateChanged` when the state actually changes.

- **`void AddCoins(int amount)`**
  - Adds to total `Coins`, emits `OnCoinsChanged`, and persists.

- **Daily reward**
  - **`void CheckDailyReward()`**: Inspect the last reward time from `PlayerPrefs` and prompt via `UIManager` if available or show remaining cooldown.
  - **`void ClaimDailyReward()`**: Adds the configured reward to `Coins`, sets the last-claim timestamp, saves, and hides the prompt.

### Usage examples

#### Starting the game and reacting to state
```csharp
public class MenuUI : MonoBehaviour
{
  public void OnPlayClicked() => GameManager.Instance.StartGame();

  void OnEnable() => GameManager.Instance.OnGameStateChanged += HandleState;
  void OnDisable() => GameManager.Instance.OnGameStateChanged -= HandleState;

  void HandleState(GameManager.GameState state)
  {
    // Swap panels based on state
  }
}
```

#### Awarding points and coins from gameplay objects
```csharp
void OnEnemyDefeated()
{
  GameManager.Instance.AddScore(250);
}

void OnSecretCoinCacheFound(int coins)
{
  GameManager.Instance.CollectCoin(coins);
}
```

#### Pause and resume bindings
```csharp
public void OnPauseButton() => GameManager.Instance.PauseGame();
public void OnResumeButton() => GameManager.Instance.ResumeGame();
```

### External dependencies
- `PlayerDataManager.Instance`: `LoadLives(default)`, `LoadCoins()`, `SaveLives(lives)`, `SaveCoins(coins)`, `GetLastPlayedLevel()`, `SetLastPlayedLevel(idx)`, `UpdateHighScore(levelIdx, score)`, `MarkLevelComplete(levelIdx)`, `UnlockLevel(levelIdx)`.
- `UIManager.Instance`: `ShowGameOverScreen()`, `HideGameOverScreen()`, `ShowPauseScreen()`, `HidePauseScreen()`, `ShowLevelCompleteScreen(score, levelCoins)`, `ShowDailyRewardPrompt(amount, isAvailable, remainingCooldown)`.
- `LeaderboardManager.Instance`: `SubmitScore(score)`.

### Notes and best practices
- Ensure `GameManager` exists before other systems access it; place it in your bootstrap scene and mark it `DontDestroyOnLoad`.
- Keep `levelScenes` synchronized with Build Settings order.
- Always unsubscribe from events to avoid dangling handlers when scenes unload.