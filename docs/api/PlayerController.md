## PlayerController API

`PlayerController` is a `MonoBehaviour` that implements grid-based movement, simple one-touch direction queuing, pickup handling, and basic animation flags. It integrates with `GameManager` for state gating and scoring.

### Inspector fields
- **`float moveSpeed`**: Units per second for movement between grid cells.
- **`SpriteRenderer hamsterSpriteRenderer`**: Optional; flips on left/right.
- **`Animator hamsterAnimator`**: Optional; sets `IsMoving` bool.
- **`float gridSize`**: World units per grid cell (default `1`).

### Lifecycle and behavior
- Movement is paused unless `GameManager.CurrentState == Playing`.
- Movement occurs toward a `targetGridPosition` (grid center to grid center).
- Input: a single tap/click queues a 90° turn in the sequence Right → Up → Left → Down. If at grid center and path is clear, the turn happens immediately.
- Walls are detected using a short `Physics2D.Raycast` against `LayerMask.GetMask("Wall")`.
- Collisions:
  - Tag `Coin`: `GameManager.CollectCoin(1)` and destroys the coin.
  - Tag `Star`: `GameManager.CollectStar()` and destroys the star.
  - Tag `Obstacle`: calls `GameManager.LoseLife()`.
  - Tag `Exit`: stops motion and calls `GameManager.LevelCompleted()`.

### Public methods
- **`void ApplySpeedBoost(float multiplier, float duration)`**
  - Temporarily multiplies `moveSpeed` by `multiplier` for `duration` seconds via a coroutine.
  - Example: `player.ApplySpeedBoost(1.5f, 3f);`

### Required components and setup
- `Rigidbody2D` is required on the same `GameObject`.
- Recommended 2D colliders on player and interactables (`isTrigger` for pickups/exit).
- Ensure the `Wall` layer exists and is assigned to solid level geometry.
- Ensure tags `Coin`, `Star`, `Obstacle`, `Exit` exist and are assigned to the relevant objects.
- Add an `EventSystem` to your scene to support UI tap filtering.

### Examples

#### Trigger a temporary speed boost
```csharp
public class PowerupSpeed : MonoBehaviour
{
  public float boost = 1.5f;
  public float seconds = 5f;

  void OnTriggerEnter2D(Collider2D other)
  {
    var player = other.GetComponent<PlayerController>();
    if (player == null) return;
    player.ApplySpeedBoost(boost, seconds);
    Destroy(gameObject);
  }
}
```

#### Customizing input (advanced)
You can replace the one-tap cycling logic by calling into private helpers or forking the controller. As a simple approach, you may expose a method that sets a desired direction when at a grid center.

```csharp
// Example snippet when extending the controller
public void TrySetDirection(Vector2 desiredDirection)
{
  // if (IsDirectionClear(desiredDirection)) { moveDirection = desiredDirection; UpdateSpriteDirection(); }
}
```

### Notes
- The controller stops when a turn would immediately collide with a wall; consider adding pathfinding for smoother turns.
- Animation hooks are minimal by design; drive richer animations based on velocity or state as needed.
- Ensure your `gridSize` matches your tile/pixel grid to avoid drift.