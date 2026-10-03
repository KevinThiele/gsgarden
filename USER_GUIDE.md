# Garden app — how to use it

Open the app at **https://kevinthiele.github.io/gsgarden/** on any computer, phone or tablet.

## The golden rule: press Save

**Nothing is kept until you press the Save button** at the top of the list. Adding, editing, renaming and deleting only change what you see on screen.

A message above the list tells you where you are:

- **Unsaved changes - press Save to keep them** (orange) — you've changed something that isn't saved yet
- **Saving...** — wait a moment
- **Saved at 3:45 pm** (green) — done
- **Save failed** (red) — nothing was saved, but your changes are still on screen. Check your signal and press Save again.
- **Not saved - the list was changed on another device** (red) — see [Using more than one device](#things-to-watch-out-for).

If you try to close or reload the page with unsaved changes, the browser asks you to confirm first.

## What's on the screen

- **List** and **Map** tabs along the top.
- **Number of species and varieties** — how many plants are in the list.
- **Show delete buttons** — a tick box that reveals the delete (bin) icons. See [Deleting](#deleting).
- The save message, when there's something to report.
- A **search** box. See [Finding a plant](#finding-a-plant).
- The list itself, in alphabetical order, with two buttons at the top: **Add new species or variety** and **Save**.

## Finding a plant

Type in the search box above the list — part of a name is enough, e.g. `geran`. The list shrinks to matching plants as you type, and shows how many match. It also finds plants by source, so typing `woodbridge` shows everything from Woodbridge.

Click **Clear** to show the whole list again.

## Looking at a plant

Each line in the list is a species or variety. Click the **+** box to its left to show its plantings; click the **−** box to hide them again. Plants you have opened stay open while you work.

Each planting shows:

- **Source** — where it came from, e.g. a nursery name, "original" or "transplant". "Unknown" means no source was entered.
- **Planted** — the date planted, if one was entered.
- **Location** — the baseline number and x/y position, if a baseline was entered.

## Adding a new species or variety

1. Click **Add new species or variety**.
2. Type the name, and any of: source, date planted, baseline, x and y.
3. Click **Add**.
4. Press **Save**.

The new plant is added with one planting, made from the details you entered, and shown opened. If you were searching, the search is cleared so you can see it.

## Adding another planting of a plant you already have

Use this when you buy more of something, transplant or divide a plant, or plant the same thing in several places.

1. Open the plant with **+**.
2. Click **Add new planting** underneath its plantings.
3. Fill in the source, date planted, baseline, x and y — whatever you know.
4. Click **Save** in the box.
5. Press **Save** at the top of the list.

Don't add the plant again with **Add new species or variety** — that makes a second plant with the same name.

## Changing a plant's name

1. Click the **pencil** next to the plant's name.
2. Type the new name and click **Rename**.
3. Press **Save**.

## Changing a planting

1. Open the plant with **+**, then click the **pencil** next to the planting.
2. Change any of the fields and click **Save** in the box. To remove a value, such as a wrong date, clear the box.
3. Press **Save** at the top of the list.

## Deleting

1. Tick **Show delete buttons**. Red bin icons appear.
2. Click a bin:
   - next to a **plant name** — deletes the plant *and all its plantings*
   - next to a **planting** — deletes just that planting
3. A box asks you to confirm. Click **Delete**.
4. Press **Save**.

Untick **Show delete buttons** when finished, so you don't hit a bin by accident.

## Using it in the garden with poor signal

The app keeps a copy of the plant list on each device, so if there's no signal when you open it, you'll see the list as it was last time.

You can make changes without signal, but **saving needs a connection** — pressing Save will show **Save failed**. If that happens:

- **keep the page open** — don't close or reload it
- when you have signal again, press **Save**

If you close the page before saving, the changes are lost.

## Things to watch out for

**Using more than one device.** Before making changes on a device, **reload the page** so it has the latest list. If the list was saved from another device after you opened it, pressing **Save** shows a warning: "The plant list was saved from another device…". You then choose:

- **Cancel** (usually the right choice) — nothing is saved and your changes stay on screen. Jot down what you changed, reload the page to get the latest list, make your changes again, and press **Save**.
- **OK** — your version is saved and *replaces* the other device's. Anything changed on the other device since you opened the page here is lost. Only choose this if you know the other device's changes don't matter.

**Baseline, x and y must be numbers**, such as `150`. If you type anything else, the app tells you and doesn't save it.

**Give every plant a different name.** The app tells plants apart by name. Two plants with exactly the same name open and close together in the list, and can get mixed up if the app ever has to combine two versions of the data. For another of the same plant, use **Add new planting** instead. The list currently has two `Geranium` and two `Salvia` entries — if each pair is really the same plant, add the second one's details as a new planting of the first, then delete the second.

**Entries ending in `[local …]` and `[github …]`.** If you ever see a plant listed twice like this, the app found two different versions of it and kept both rather than guess. Compare them, rename the correct one back to its proper name (pencil), delete the other (bin), and press **Save**.

## Not working yet

- **Map** — unfinished. The background plan image (`block plan.png`) isn't in the repo yet, so the map is blank apart from a single test dot. Plants are not drawn on it yet.

## If something goes wrong

- **The list is empty or won't load** — check you have signal and reload. If it's still empty, the app may not be able to reach GitHub; ask whoever looks after the app.
- **Save failed** — your changes are still on screen. Keep the page open, check your signal and press Save again.
- **Not saved - the list was changed on another device** — see [Using more than one device](#things-to-watch-out-for): note your changes, reload, and make them again.
- **Changes made on another device have disappeared** — another device saved over them, usually by choosing **OK** at the warning. They aren't lost: every save is kept in the repo history. Ask whoever looks after the app to bring back the earlier version (see the README).
- **Data was saved wrongly or deleted by mistake** — nothing is lost for good. Every save is kept in the repo history, so an earlier version can be brought back. See the README for how.
