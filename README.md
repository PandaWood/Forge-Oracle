# Forge Oracle

Predict the next Forge of Potential results in Valheim from your character save, before you spend an idol.

Forge of Potential rolls are not random at the moment you click. Each item and target level has a seed stored in your `.fch` character file. Each attempt turns that seed into one roll, then saves a new seed. Reloading cannot change an outcome, so upcoming results can be read ahead of time.

Unofficial fan tool. Not affiliated with or endorsed by Iron Gate AB. Single-player only. Read-only: it never modifies your save.

## Use it in the browser

Open `index.html` (or the hosted page, if you publish one).

1. Drop your character file onto the page. Everything runs locally in your browser; the file is never uploaded.
2. Pick one of the slots found in your save, or type an item's internal name (for example `Hammer`) and its current level.
3. Read the next ten upgrade steps for that item: row 1 is current level to +1, row 2 is +1 to +2, and so on, each with its predicted result and roll. The rows after the first assume every earlier step succeeded, since a failure breaks the item.

The character name is read from the save automatically. It only matters for an item and level that has no stored seed yet, where the starting seed comes from the name.

On Windows, character files live in:

```
%USERPROFILE%\AppData\LocalLow\IronGate\Valheim\characters_local
```

The game writes seeds to the file only when it saves, so quit to the main menu after an attempt before loading the file.

## Use it from the command line

`forge_predict.py` needs only Python 3. It prints the same list as the page.

```
python forge_predict.py --item Hammer --level 3 --fch Thor.fch
```

| Option | Meaning |
| --- | --- |
| `--item` | The item's internal name, not its display name. Look names up in the [Valheim item ID list](https://www.valheim.tools/item-ids) |
| `--level` | The item's current level. Row 1 is current to +1, row 2 is +1 to +2, and so on |
| `--fch` | Your character file. Stored seeds are used and the character name is read from it |
| `--name` | Character name. Only needed without `--fch`, or if the name could not be read |
| `--count` | How many upgrade steps to list (default 10) |

`fch_tail.py` prints the save version, checksum status and the raw seed tables from a `.fch` file. It is a debugging aid.

## How it works

These details were read from the game's own code and then checked against real saves.

1. The key is `hash(item name) + target level`, using Valheim's string hash.
2. The seed is the one stored for that key in the save. If there is none, it is `hash(character name) + key`.
3. Unity's random generator is seeded with it and one value between 0 and 1 is drawn. That is the roll.
4. A second draw, taken modulo 99999999, is stored as the new seed for that key. This happens on every attempt, before the result is judged.
5. A roll at or below the idol's upgrade chance succeeds. A higher roll fails, and a failed upgrade destroys the item.

Things that follow from this:

- Reloading a save does not reroll an attempt.
- Two items with the same internal name at the same target level share one sequence.
- A different item or a different level is a separate sequence.
- After a success, the next upgrade of that item uses the next level's slot, which has a fresh sequence.

### Save format (version 47, game 1.0.20)

```
[u32 payloadLength][payload][u32 64][SHA-512 of payload]
```

Near the end of the payload, after the build-menu blob, come two lists. The second is the Forge seed list: a count followed by `(u32 key, u32 seed)` pairs. The first is a separate list of drop counters, which the Forge does not use.

## What is verified

- The seed-to-roll math matched 14 consecutive real hammer attempts, including every following seed.
- The key and seed layout match the game's decompiled `Player.GetItemUpgradeChance`.
- The character name detection returned the right name on every save tested.

## Known limits

- The success cutoff is only bracketed: a roll of 0.5535 succeeded and 0.6602 failed. The page marks anything in that gap as unclear.
- Only the wood idol has been tested. Other idols may use different chances.
- The built-in item list is a best guess at internal names. A wrong name gives a plausible but wrong prediction. Check names against the [Valheim item ID list](https://www.valheim.tools/item-ids).
- Name detection looks for the name stored next to your character ID. A character with no crafted items, or with many items crafted by someone else, may need the name entered by hand.
- A game patch can change the math or the save format.

If a prediction ever misses, open an issue with the save from before the attempt, the save from after it, and what happened.

## Files

| File | Purpose |
| --- | --- |
| `index.html` | The browser tool, a single self-contained file |
| `forge_predict.py` | Command line predictor |
| `fch_tail.py` | Save inspector |

## License

Add a license before publishing, for example MIT.
