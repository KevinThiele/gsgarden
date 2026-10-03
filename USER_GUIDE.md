# Garden app — how to use it

Open the app at **https://kevinthiele.github.io/gsgarden/** on any computer, phone or tablet.

## The golden rule: press Save

**Nothing is kept until you press the Save button** at the top of the list. Adding, editing, renaming and deleting only change what you see on screen. If you close or reload the page before saving, those changes are lost.

There is no "saved" message — the page simply stays as it is. To check a save worked, reload the page: if your changes are still there, it saved.

## What's on the screen

- **List** and **Map** tabs along the top.
- **Number of species and varieties** — how many plants are in the list.
- **Show delete buttons** — a tick box that reveals the delete (bin) icons. See [Deleting](#deleting).
- The list itself, in alphabetical order, with two buttons at the top: **Add new species or variety** and **Save**.

## Looking at a plant

Each line in the list is a species or variety. Click the **+** box to its left to show its plantings; click the **−** box to hide them again. Plants you have opened stay open while you work.

Each planting shows:

- **Source** — where it came from, e.g. a nursery name, "original" or "transplant". "Unknown" means no source was entered.
- **Planted** — the date planted, if one was entered.
- **Location** — the baseline number and x/y position, if a baseline was entered.

## Adding a new species or variety

1. Click **Add new species or variety**.
2. Type the name, and any of: source, date planted, baseline, x and y.
3. Click **Add**. A pop-up confirms what is being added — click **OK**.
4. Press **Save**.

The new plant is added with one planting, made from the details you entered.

## Changing a plant's name

1. Click the **pencil** next to the plant's name.
2. Type the new name and click **Rename**.
3. Press **Save**.

## Changing a planting

1. Open the plant with **+**, then click the **pencil** next to the planting.
2. Change any of the fields and click **Save** in the box.
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

You can make changes without signal, but **saving needs a connection**. If you're offline:

- **keep the page open** — don't close or reload it
- when you have signal again, press **Save**

If you close the page before saving, the changes are lost.

## Things to watch out for

**Using more than one device.** The last save wins. If the app is open on both your phone and your computer, and you save on one and then the other, the second save overwrites the first. Before making changes on a device, **reload the page** so it has the latest list.

**Clearing a field doesn't clear it.** When changing a planting, leaving a box empty keeps the old value. You can change a source, date or location, but you can't remove one in the app.

**x and y must be numbers.** Type plain numbers such as `150`. Anything else may cause an error or a strange value.

**Give every plant a different name.** The app tells plants apart by name. Two plants with exactly the same name can cause the wrong one to be deleted or details to get mixed up. Add something to tell them apart, e.g. `Salvia (front bed)`.

**Don't leave the name empty** when adding a plant — the app will let you, but you'll get a blank line in the list.

**Double quotes show as single quotes** in the list, e.g. `Achillea "Moonshine"` appears as `Achillea 'Moonshine'`. The name itself is unchanged.

**The plant count doesn't update when you add a plant.** It catches up when you reload the page.

**Entries ending in `[local …]` and `[github …]`.** If you ever see a plant listed twice like this, the app found two different versions of it and kept both rather than guess. Compare them, rename the correct one back to its proper name (pencil), delete the other (bin), and press **Save**.

## Not working yet

- **Add new planting** — the button under each plant does nothing yet. For now, a new planting can't be added to an existing plant from the app.
- **Map** — unfinished. The background plan image (`block plan.png`) isn't in the repo yet, so the map is blank apart from a single test dot. Plants are not drawn on it yet.

## If something goes wrong

- **The list is empty or won't load** — check you have signal and reload. If it's still empty, the app may not be able to reach GitHub; ask whoever looks after the app.
- **A save seems to have done nothing** — reload to check. If the change is gone, the save failed; make the change again and press Save while you have good signal.
- **Data was saved wrongly or deleted by mistake** — nothing is lost for good. Every save is kept in the repo history, so an earlier version can be brought back. See the README for how.
