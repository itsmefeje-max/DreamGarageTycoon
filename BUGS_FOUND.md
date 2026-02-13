# Bug Audit Findings

## 1) Receipt processing can error when `Backpack` is unavailable
- In `ReceiptRouter`, the car purchase path directly parents a cloned tool to `player.Backpack` without checking that the backpack exists yet.
- `MarketplaceService.ProcessReceipt` can run before character/backpack initialization in some join timing scenarios.
- Impact: runtime error during receipt handling, causing receipt retries and potentially delayed grants.
- Fix: use `player:FindFirstChildOfClass("Backpack")` or `player:WaitForChild("Backpack", timeout)` and fail safely if missing.

## 2) ATM "Max" autofill uses the wrong stat
- In `WithdrawLogic`, the Max button reads `leaderstats.Cash`.
- But the server-side withdraw path (`TransactionHandler`) withdraws from `leaderstats.ATMBalance`.
- Impact: Max button often fills an amount larger than ATM funds, producing false/annoying client-side and server-side denials.
- Fix: read `ATMBalance` for withdraw max.

## 3) Duplicate ATM max behavior bug in standalone max helper
- `Max Helper` also reads from `leaderstats.Cash`.
- This repeats the same wrong-source bug as above for UI flows that use this helper script.
- Impact: user enters invalid max amount relative to ATM balance.
- Fix: replace `Cash` with `ATMBalance`.

## 4) Unreachable branch in help button input handling
- In `Click then help`, code checks `MouseMovement` *inside* a branch that already requires `MouseButton1` or `Touch`.
- Impact: the intended mouse-move label behavior in that block can never execute.
- Fix: move mouse-movement logic to its own condition or rely on `MouseEnter`/`MouseLeave` only.
