# Stacking the Deck

This repo contains a **host** and a **strategy client** to play the [Stacking the Deck](https://cs.nyu.edu/courses/fall25/CSCI-GA.2965-001/stacking.html) game described in the assignment.

- `host.py` — runs the game server on **localhost (127.0.0.1)**.
- `client.py` — connects to the host and plays one role (**Arranger** or **Chooser**) using three strategy functions you implement.

Both scripts communicate using newline-delimited JSON over TCP.

---

## Files

- `host.py` — game coordinator (Judge Judy + referee)
- `client.py` — player bot with the three strategy functions

---

## How to Run

### 1) Start the host

```bash
python host.py <PORT> <K>
```
- `<PORT>`: any free local port, e.g. `1213`
- `<K>`: size of **V** (the Arranger’s inserted values), must be `1..16`  
  (Host ensures **V** contains at least one `1`.)

Example:
```bash
python host.py 1213 5
```

### 2) Start the two clients (same machine)

Start **Chooser**:
```bash
python client.py <PORT> "<NAME>" Chooser
# e.g.
python client.py 1213 "Charles" Chooser
```

Start **Arranger**:
```bash
python client.py <PORT> "<NAME>" Arranger
# e.g.
python client.py 1213 "Alice" Arranger
```

> The game begins once both roles have connected.

---

## What the Host Does

1. Builds the multiset of values for two suits Ace..8 → values `1..8` twice (length 16).
2. Randomly chooses **V** (size `K`, includes at least one `1`). The remaining values form **S_pool**.
3. Sends **S_pool** to **Chooser** → Chooser returns an ordering **S′** (timeout: 120s).
4. Sends **S′** and **V** to **Arranger** → Arranger returns a **final sequence** (length 16) that:
   - is a permutation of `S′ ∪ V`,
   - **ends with 1**,
   - **preserves the relative order of S′** (as a subsequence).
5. Sends the final sequence to Chooser → Chooser picks **start** ∈ `{1..8}` (timeout: 60s).
6. Simulates the dealing and prints the **annotated sequence** (revealed cards in parentheses) and the **winner**.

Timeouts or rule violations end the round immediately with the appropriate winner.

---

## Strategy API (implement these in `client.py`)

`client.py` exposes three functions you should customize. Defaults are **random placeholders** so the game runs out-of-the-box.

```python
def arrange_S(S_pool: List[int]) -> List[int]:
    """
    Input:  S_pool — the multiset remaining after removing V (length 16-K)
    Output: S' — an ordering (permutation) of S_pool
    """
    ...

def insert_V(S_prime: List[int], V: List[int]) -> List[int]:
    """
    Input:  S' and V
    Output: Final sequence of length 16 that:
        - is a permutation of S' ∪ V
        - preserves S' relative order (as a subsequence)
        - ends with 1
    """
    ...

def choose_start(sequence: List[int]) -> int:
    """
    Input:  The Arranger’s final sequence (length 16)
    Output: An integer in [1..8] to start the dealing
    """
    ...
```

> Implement your actual logic in these functions. Everything else in `client.py` handles the protocol and message passing.

---

## Example Session

Terminal A (host):
```bash
python host.py 1213 5
```

Terminal B (chooser bot):
```bash
python client.py 1213 "C" Chooser
```

Terminal C (arranger bot):
```bash
python client.py 1213 "A" Arranger
```

The host will print:
- Clients connected and HELLO received
- S_pool sent → S′ received
- V sent → final sequence received (validated)
- Start requested → received
- **Annotated sequence** and **Winner**

Clients will print a short result summary at the end.

---

## Validation Rules (Host-side)

- **S′** must be a **reordering** of **S_pool** (multiset equality).
- **Final sequence** must:
  - contain exactly the multiset `S′ ∪ V`,
  - **end with `1`**,
  - **preserve S′ as a subsequence** (same relative order).
- **Chooser time limit** for S′: **120s**.
- **Arranger time limit** for final sequence: **120s**.


