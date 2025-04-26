---
title: Mac
---

### Ctrl+w everywhere

    $ vim ~/Library/KeyBindings/DefaultKeyBinding.dict

    {
        "^w" = "deleteWordBackward:";
    }

### VScode key repeating

    $ defaults delete com.microsoft.VSCode ApplePressAndHoldEnabled
    $ defaults write com.microsoft.VSCode ApplePressAndHoldEnabled -bool false

### Font Smoothing

    $ defaults -currentHost write -g AppleFontSmoothing -int 0
    $ reboot

Note that you can adjust the level of smoothing more granually by changing the number at the end of this command. The "0" disables font smoothing, while "1" enables light font smoothing, "2" enables default medium smoothing, and "3" enables strong smoothing. For example, if after disabling font smoothing you want to return to the default setting, replace the "0" at the end of the command with a "2."

```css
html {
    -webkit-font-smoothing: antialiased;
}
```

for Safari, save this with a .css extension somewhere and select it as the custom stylesheet in Preferences > Advanced.

`defaults write com.apple.Safari AppleFontSmoothing -int 0` would also disable subpixel rendering in the UI.
