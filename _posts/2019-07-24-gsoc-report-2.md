---
lang: en-US
title: "Purr Data x GSoC: Phase 2"
categories:
  - Programming
tags:
  - purr-data
  - gsoc
---

## Things done

Shamefully, I must admit that this month of GSoC doesn't really achieve anything noteworthy. Below is a shady but complete list of the changes happened in July.

- (small) Features:
  - Better styling for the K12 panel.
  - Added a menu entry to show the K12 demo Pd patches.
  - Grouped buttons in different categories to different lines. Added a mechanism to easily customise this in `k12abs.json`.
- Bugfixes:
  - User can no longer select the elements in the K12 frame with the mouse cursor.
  - Right clicking on canvas now pops up the menu with the correct coordinates.
  - Manually toggling editmode now also changes the Perform/Build Button.
  - The K12 launcher on Mac now works everywhere in addition to `/Applications/` as long as it is placed in the same directory as the real Pd-l2ork app.
- General improvements:
  - Improved the `make clean` process to prevent introducing redundant files to the repo.

Now K12 Mode looks like this (screenshots taken on Mac).

![k12-dmg](/images/k12-dmg.png)
![k12-full](/images/k12-full.png)

## What next?

Fortunately, the next steps are quite clear.

- Solve the z-order problem. It is rather easy to seperate the nlets from the border and render them in proper order; the only problem remaining is how to select them both with a single mouse click.
- Implement nlet tooltips. Modifying the `canvas_doclick()` function in `g_editor.c` and adding a few callbacks in `pdgui.js` should suffice. The mechanism will be similar to the Magic Glass.
- Implement the K12 launcher for Linux.
- Add new abstractions to the `k12abs.json` list.
- Toggle legacy mode based on K12 Mode.
- Bugfixes:
  - Fix the bug where K12 Mode and non-K12 Mode opens a Pd file in the same window size (meaning 200px in K12 Mode is taken up by the K12 panel).