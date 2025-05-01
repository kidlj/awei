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
    $ sudo reboot

Note that you can adjust the level of smoothing more granually by changing the number at the end of this command. The `0` disables font smoothing, while `1` enables light font smoothing, `2` enables default medium smoothing, and `3` enables strong smoothing. For example, if after disabling font smoothing you want to return to the default setting, replace the `0` at the end of the command with a `2`.

### Set Hostname

    $ sudo scutil --set ComputerName "NewHostName"
    $ sudo scutil --set HostName "NewHostName"
    $ sudo scutil --set LocalHostName "NewHostName"
