---
title: "Purr Data x GSoC: Phase 1"
categories:
  - Programming
tags:
  - purr-data
  - gsoc
---

I am very grateful that my proposal got selected by Purr Data. From my perspective, I put a lot of
efforts into researching the project idea and the code base, as well as communicating with the
maintainers of the project, and finally produced a promising proposal that is concrete and should be easy
to follow. However, I must admit that I wasn't really doing a good job in Phase 1. Due to personal
matters, I couldn't focus on getting the job done, postponed the work frequently and didn't have much
discussion with my mentor. Now, even though the K12 Mode is roughly working as planned, it isn't as good
as I've expected. I apologize for the lack of quality in my work.

That being said, I still have about ten days to catch up, and I, apparently, do want to continue working
on Purr Data and GSoC. Here, I will give a brief summary on the progress I've made and the next steps
before the first evaluation. The changes I have commited to the Purr Data GitLab repo can be found
[here](https://git.purrdata.net/nerrons/purr-data/merge_requests/2).

## Changes inside the build system

+ For Windows: Makefile copies the K12 abstractions to `@extra`. Inno Setup now creates new shortcuts 
  to launch Purr Data in K12 Mode on the desktop and the Start Menu. Relevent changes are in `pd-inno.ss.in` and `pd-inno-light.ss.in`.
+ For macOS: During build time, a separate, minimal app called Pd-l2ork-K12 will be created and copied
  into the dmg installer, so it can be dragged into the `/Applications` folder along with the actual
  app. The app simply invokes the Pd-l2ork app with a `-k12` flag. A separate app is chosen over a shell
  script or anything similar because apps have customized icons. Also, the K12 abstractions folder will
  be copied to `@extra`. Relevent changes are in the `darwin_app` Makefile and the `k12-launcher` folder.
+ For Linux: I haven't made any changes to the Linux build system, but it should be be similar to and
  easier than macOS (just adding an executable shell script with an icon).

## Changes in the front end

+ `index.js`: Now reads `-k12` flag and sets up K12 mode in pdgui accordingly.
+ `pdgui.js`: Now has two more exported functions that return 1. whether K12 mode is on; 2. the
  horizontal bias in pixels introduced by the width of the K12 frame (called the k12 offset). Besides, a
  few places in this file that uses the width of the window and the svg are modified to respect that
  offset.
+ `pd_canvas.html`: Now has a new `<div id="k12-frame">` right before the `patchsvg`. The build/perform
  button and the control/signal button are predefined in the HTML, and the K12 abstraction buttons will
  be appended to the div whenever a new canvas is created. The HTML file also serves as a temporary
  place to hold K12 related CSS for now.
+ `pd_canvas.js`: Firstly, like `pdgui.js`, the offset will be considered when the mouse is clicked on
  the canvas window; secondly, during initialization, `pd_k12abs.json` will be read and converted to
  buttons on the K12 frame.
+ `pd_k12abs.json`: Contains information about K12 abstractions (category, name and description).

Now the canvas looks like this when K12 Mode is on. The flashy background color is just to make it easier
to see the margins.

![k12-tiger](/images/k12-tiger.png)

## What next?

There are still a few things in the proposed timeline that didn't get realized, including:

+ Hide menu entries that are too complicated.
+ Add a hidden `preset_hub` for all new canvas windows.
+ Fix the margins on the K12 frame (currently the abstraction buttons part has a larger margin).
+ Add the new K12 abstractions in the latest Pd-l2ork update to the JSON file.

Fortunately, these jobs don't require very sophisticated and original solutions, only careful
implementations and design-thinking, which would be more "suitable" for the situation I'm currently stuck
in. I am still confident that I can get the job done.
