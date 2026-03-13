# iTerm 2 configuration

There are other good terminals out there, but I keep using iTerm because I've been using it for a
long time.

## Basic config and color schemes

All the settings here are under the "Profiles" tab in Settings.

### Font

- ComicShannsMono Nerd Font (https://www.nerdfonts.com/font-downloads)
- Size 15 or 16 because my eyes are old.

### Colors

A large variety of themes are available to browse at https://iterm2colorschemes.com/.

I used to use Solarized (and I still use Solarized Light in VS Code for its low-contrast design).
However, I don't use it anymore in the terminal because it has coöpted the "bold" ANSI colors for
its expanded set of grays, which causes scripts not to render in the correct colors.

As a result, I've switched to one of the following two color schemes:

- Selenized: uses the same general hues as Solarized but a higher-contrast design and correct bolds.
- Night Owl: has a much higher contrast overall, but uses a nice set of dark pastels that I think is
  pretty.

Other themes of high repute that I've strongly considered:

- Catpuccin (several variants)
- Monokai Classic / Pro (several variants)
- Dracula / Dracula+ (several variants)

## Profile switching

If I'm going to spend a lot of time on a remote machine, I like to use a different color scheme when
in an ssh connection to the remote box, so I can quickly tell where I am. This can be set up on
iTerm using [Automatic Profile Switching](https://iterm2.com/documentation-automatic-profile-switching.html).
There are other methods, but this one is the most reliable for switching back to the local
theme if the ssh session gets disconnected.

1. In Settings -> Profiles, set up the Default profile for how you want for the local machine's
   terminal to look.
2. Make a second profile and set it up how you want the remote machine's terminal to look.
3. In the "Advanced" tab on the remote profile, create a new rule entry under "Automatic Profile
   Switching". If the current environment matches this rule, iTerm will automatically switch to this
   profile. See the
   [documentation](https://iterm2.com/documentation-automatic-profile-switching.html) for the full
   rule syntax and figure out what works in your case.
4. Finally, for this to work, you need to enable [shell
   integration](https://iterm2.com/documentation-shell-integration.html) on all environments.
   - Locally, this is easy: under the "general" tab (in both profiles), click the "Load shell
     integration automatically" checkbox.
   - On the remote machine, ssh into it and then select "Install shell integration" from the iTerm 2
     menu. It will run some curl/bash scriets to install the necessary stuff into the .zshrc on the
     remote machine.

Shell integration also enables several other features, some of which may be annoying. The most
annoying for me is "command selection", which selects commands when you click inside a terminal. To
disable this, click a command and toggle the feature off using the icon on the top right of the
selected region.
