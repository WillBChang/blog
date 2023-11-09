---
date: 2023-11-06
title: Fix Ghost Selecting in Context Menu
description: When you hold down the right-click button and move the cursor within the context menu, and then release the button, it will automatically activate the selected item. This can be quite bothersome if you unintentionally rush through this process, leading to unexpected actions. This issue can be addressed by adjusting the timing of displaying the context menu after releasing the right-click button. You can achieve this by using a Complex Modification rule in Karabiner Elements.

tags:
  - macOS
  - mouse
  - karabiner-elements
---

# {{ $frontmatter.title }}

## Solution

1. Install [Karabiner-Elements](https://karabiner-elements.pqrs.org/).
   ```bash
   brew install --cask karabiner-elements
   ```
2. Follow [How to change mouse buttons](https://karabiner-elements.pqrs.org/docs/help/how-to/mouse-button/) to enable
   the mouse in Karabiner-Elements.
3. Import the Complex Modification
   rule: [Fix ghost select in context menu](https://ke-complex-modifications.pqrs.org/#fix-ghost-select-in-context-menu)
   and enable it.

## How did I solve it?

I experienced this issue several times per week, mainly while using Intellij IDEA. I thought it was the IDEA's bug.
After doing some google searches I found the same issue which was created 7 years
ago.  [Mac OS X Sierra: Mouse right click context menu too close to click location - triggers highlighted action unintentionally : JBR-1477](https://youtrack.jetbrains.com/issue/JBR-1477/Mac-OS-X-Sierra-Mouse-right-click-context-menu-too-close-to-click-location-triggers-highlighted-action-unintentionally).
I need to know if what exactly is the issue, then I started doing debugging.

### How to trigger this bug?

The bug in IDEA is triggered very quickly, I thought I was unintentionally clicked the left-click button. So I lifted my
index finger and click the right-click button several times. **The bug still appeared!** Not my index finger's problem!

I repeated the right click quickly and moved the mouse randomly, then I found a pattern to trigger it. Quickly do the
following process: **Hold down the right-click button, move cursor slightly right, then release the button.**

Then I tried to the process above slowly to test if it still happens. The answer is **Yes**.

### Does this behavior happen in other software?

**Yes**, I tried the process in Google Chrome and even in Finder. The bug still occurs.

I thought it might be the problem of Logi Options+, after uninstalling it. The bug still occurs.

### Does this behavior relate to hardware?

**No**, I used the trackpad in Macbook Air, it still happens.

### Does this behavior happen in others' computer?

**Yes**, I found [How to disable "right click acting as left click"](https://apple.stackexchange.com/questions/367395/how-to-disable-right-click-acting-as-left-click).
It seems like a feature for macOS.

### How do I disable this feature?

First, I need to know what exactly going on when I do the process. I
found [Karabiner-EvenViewer](https://karabiner-elements.pqrs.org/), it can watch the button/keyboard events. It turns out it's just right click. 

```json
[
    {
        "type": "down",
        "name": {
            "pointing_button": "button2"
        },
        "usagePage": "9 (0x0009)",
        "usage": "2 (0x0002)",
        "misc": ""
    },
    {
        "type": "up",
        "name": {
            "pointing_button": "button2"
        },
        "usagePage": "9 (0x0009)",
        "usage": "2 (0x0002)",
        "misc": ""
    }
]
```

I was thinking about how to control the mouse behavior and found a library [AltF02/mouse-rs](https://github.com/AltF02/mouse-rs) and the Apple Developer Documentation [Mouse, Keyboard, and Trackpad](https://developer.apple.com/documentation/appkit/mouse_keyboard_and_trackpad/). It will definitely take some time to do this. Is there a simpler way?

**YES**! Do you remember that I used **Karabiner EventViewer** to capture the button clicks? I think it's very likely to use **Karabiner Elements** to modify the right lick behavior.

A tutorial documentation [How to change mouse buttons](https://karabiner-elements.pqrs.org/docs/help/how-to/mouse-button/) shows how to enable it. After enable it I did a test with **Simple Modifications** rule to test if it really works. The context menu is showing when right click button `down`, if I make the context menu shows on button `up`, this should not activate the item in context menu anymore. By using [to_after_key_up | Karabiner-Elements](https://karabiner-elements.pqrs.org/docs/json/complex-modifications-manipulator-definition/to-after-key-up/) I wrote the Complex Modifications Rule:

```json
{
   "title": "Fix ghost select in context menu",
   "rules": [
      {
         "description": "Fix ghost select in context menu",
         "manipulators": [
            {
               "from": {
                  "pointing_button": "button2"
               },
               "to_after_key_up": [
                  {
                     "pointing_button": "button2"
                  }
               ],
               "type": "basic"
            }
         ]
      }
   ]
}
```

- By using `frontmost_application_if`, the rule will only work on the applications you set.
- By using `frontmost_application_unless`, the rule will work everywhere except the applications you set.

You can use Karabiner-EventViewer → Frontmost Application to get the bundle id of the application.

```diff
{
  "title": "Fix ghost select in context menu",
  "rules": [
    {
      "description": "Fix ghost select in context menu",
      "manipulators": [
        {
+         "conditions": [
+           {
+             "bundle_identifiers": [
+               "^cc\\.ffitch\\.shottr$"
+             ],
+             "type": "frontmost_application_unless"
+           }
+         ],
          "from": {
            "pointing_button": "button2"
          },
          "to_after_key_up": [
            {
              "pointing_button": "button2"
            }
          ],
          "type": "basic"
        }
      ]
    }
  ]
}
```