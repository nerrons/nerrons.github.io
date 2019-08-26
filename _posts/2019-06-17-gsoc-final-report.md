---
lang: en-US
title: "Purr Data x GSoC: Final Report"
categories:
  - Programming
tags:
  - purr-data
  - gsoc
---

# Things done
Below is a list of merge requests submitted during the coding period.
- [General improvements for the building process.](https://git.purrdata.net/jwilkes/purr-data/merge_requests/295)
- [Building phase changes for K12 Mode.](https://git.purrdata.net/jwilkes/purr-data/merge_requests/297)
- [Minimal K12 Mode.](https://git.purrdata.net/jwilkes/purr-data/merge_requests/301)
- [Hovering tooltips for inlets and outlets.](https://git.purrdata.net/jwilkes/purr-data/merge_requests/308)

### Building Phase
First we need a way to convenietly start Purr Data in K12 Mode.
- On Windows, create desktop and Start Menu shortcuts that starts Purr Data in K12 Mode.
  - Makefile copies the K12 abstractions to `@extra`.
  - Inno Setup now creates new shortcuts to launch Purr Data in K12 Mode on the desktop and the Start Menu. Relevent changes are in `pd-inno.ss.in` and `pd-inno-light.ss.in`.
- On Mac, create a minimal K12 Launcher app in the .dmg that starts Purr Data in K12 Mode. The user needs to put the launcher in the same directory as the Purr Data app.
  - Makefile copies the K12 abstractions to `@extra`.
  - The K12 launcher simply invokes the Pd-l2ork app with a `-k12` flag. A separate app is chosen over a shell script or anything similar because apps have customized icons. Relevent changes are in the `darwin_app` Makefile and the `k12-launcher` folder.
- (No changes to the Linux building phase since it's already taken care of by Pd-l2ork 1.0.)

Below is a screenshot on how the dmg looks on macOS.

![k12-dmg](/images/k12-dmg.png)

### Minimal K12 Mode
Now we add a working K12 Mode with functional panel and buttons.
- In `index.js`, parse the `-k12` flag when starting Purr Data and set up K12 Mode with the API of `pdgui.js`. Also adds a menu entry for opening the K12 Demos folder.
- In `pd_canvas.html`, adds styles for the K12 panel and buttons. The markup is also modified to include the structure of the panel.
  - Buttons will be shoved into `#k12-buttons-control` and `#k12-buttons-sound`. To change Control/Sound, we simply set the visibility of the two divs.
- In `pd_canvas.js`, initialize the K12 Panel when opening a new canvas.
  - Throughout the file, code that are based on x-coordinates are accompanied by an additional K12 Offset, which will mitigate the horizontal bias introduced by the width of the K12 Panel.
  - When initializing the canvas and K12 Mode is on:
    - Add the Build/Perform and Control/Sound buttons to the K12 Panel;
    - Populate the K12 Panel with different categories of abstractions, which order is defined in the `k12_category_list` and `k12_buttons_list` variables. Placeholders will automatically be added to differentiate the categories.
- In `pdgui.js`:
  - Like `pd_canvas.js`, introduce horizontal offsets.
  - Add APIs for setting and getting K12 mode related information.

With the minimal K12 Mode, Purr Data will look like this:

![k12-full](/images/k12-full.png)

### xlet Tooltips
Then we implement xlet tooltips: when the user hovers their mouse cursor over any xlet, they should see the tooltip with relevant information popping up. To get the content of the tooltip, we parse the `-help.pd` files in the K12 folder. This way when the information needs to be updated, no changes in the source code will be necessary; just provide the updated Pd patch.

Below is how it works.
1. When starting Pd in K12 mode, a parser goes over all `-help.pd` files to extract the information in the `pd META` canvas. The content of the tooltip is stored in a global object called `k12_xlet_tt`, structured like this:
```
{
  "abstraction1": {
    "i0": "content...",
    "i1": "content...",
    "o0": "content..."
  },
  "abstraction2": {
    ...
  }, ...
}
```
2. When opening a .pd file and Purr Data is creating gobjs on the svg, the `gui_gobj_new` function from JS receives the rtext-text of the graph object from the C side (which is essentially the name of the K12 abstraction), and stores it in the DOM using the `ttid` attribute. This way, Purr Data knows which K12 abstraction is our new gobj representing, making it possible to retrieve tooltip content from `k12_xlet_tt`. Relevant changes include:
   - In `pdgui.js`, an additional parameter for `gui_gobj_new` to receive the rtext-text;
   - In `g_graph.c`'s `graph_vis` function , a call to `rtext_gettext` before vmessing `gui_obj_new`.
3. C code now already has a (very complex) mechanism for detecting whether the mouse cursor is hovering over any xlet, and if so, it will vmess `gui_gobj_highlight_io` to play the cute animation. We just modify `gui_highlight_io` to retrieve tooltip content from `k12_xlet_tt` using the `ttid` attribute, and then display the tooltip div at the appropriate position. Relevant changes include:
   - In `pd_canvas.html`, there's a new `<div>` to contain the tooltip.
   - `pdgui.js`'s `gui_gobj_highlight_io` function now shows the tooltip div when the cursor is hovering over xlets. For inlets, the tooltip will be located below the cursor, because there will be guaranteed space; the opposite for outlets. 
   - `pdgui.js`'s `gui_gobj_configure_io` function hides the tooltip div, since this function is called every time when the cursor is moving out of the xlet.

Below is the screenshot of the xlet tooltip feature.

![xlet-tt-demo](/images/xlet-tt-demo.gif)

# What next?
A few things in [the proposal](https://drive.google.com/open?id=1brEdq5efSpqipSECx2JNtK0P4p3STLns) were not yet realized.
- Enhancing the image quality of the K12 abstraction background images, either by upscaling the .png file or converting them to .svg files.
- Touchscreen support.

Some known bugs.
- As can be seen in the xlet tooltip screenshot, the border of the abstraction has different thickness, the top and the left a bit thicker than the bottom and the right. That is because the `ggee/image` used by the K12 abstractions draws the image with a slight offset towards the bottom-right (the top-left inlets are not entirely covered up by the white background of the image).
- For some unknown reason, when creating certain K12 abstractions, the rtext-text doesn't get passed through `gui_vmess` to `gui_gobj_new`. When this happens, the `<g>` will not have a correct `ttid` attribute and thus won't display the tooltip. I fail to find a pattern when this will happen. It's always the same abstractions no matter how many times the Pd patch is opened. Below is a violent printing of how the `vmess` actually happened.

![xlet-tt-bug](/images/xlet-tt-bug.png)

Also there are a lot of enhancements to work on.
- Move the CSS styles in `pd_canvas.html` to either the themes or a separate CSS file.
- When the xlet is on the right-most edge of the canvas, show the tooltip on the left of the cursor.
- Integrate the new K12 abstractions from Pd-l2ork 1.0.
- It might be a good idea to separate `<g>`s that draw the border and the xlets of an abstraction, so the content of GOP (especially images) can extend out of the borders without covering up the xlets.

# One more thing
My mentors, Ivica Ico Bukvic and Jonathan Wilkes, are great. Not only they answer my questions in great detail and review my code thoroughly, they do have faith in me when I am troubled by real-life problems and could not perform as well. I am extremely grateful about their mentoring and will keep contributing to Purr Data after GSoC. Thank you, and all the great minds who realized Pd.