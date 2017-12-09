---
layout: post
title:  "Spotify in your Terminal part 1"
comments: true
date:   2017-12-07 14:00:00 +1000
categories: linux terminal mac spotify zsh applescript
---

I recently got turned on to the wonderful world of customisable terminals.

If you haven't tried or heard of `zsh` or [`oh-my-zsh`][1], your terminal is surely missing out

One of the fantastic themes for `oh-my-zsh` is [`powerlevel9k`][2]. This theme is the equivalent of a digital kamehameha.

You can install a variety of plugins and very extensibly customise your prompt.

You can also include your own cool little custom plugins like this *What's Playing?* plugin:

```bash
prompt_music_status(){
  local color='%F{white}'
  running=`osascript -e 'application "Spotify" is running'`;
  if [[ "$running" == "true" ]]; then
    state=`osascript -e 'tell application "Spotify" to player state as string'`;
    if [[ "$state" == "playing" ]]; then
      track=`osascript -e 'tell application "Spotify" to name of current track as string'`;
      duration=`osascript -e 'tell application "Spotify" to ((round (round ((duration of current track) / 1000) / 60) rounding down) as string) & ":" & text -2 thru -1 of ("00" & (round ((duration of current track) / 1000) mod 60))'`;
      echo -n "%{$color%} \uf1bc $track ($duration)";
    fi
  fi
}
```

This magic is made possible by using `AppleScript`, although there are [`spotify`][3] modules out there for use on non-macOS systems.

I'm sticking with AppleScript for the meantime, if you are so inclined, you can read my post on [AppleScript Dictionaries][4]

Throwing that into your `~/.zshrc` and putting `music_status` in your prompt list gives you a nice look like this:

![prompt example][5]

If you're curious, you can find my [`.zshrc`][6] here and I might write a gist up in the future detailing installation instructions.

[1]:https://github.com/robbyrussell/oh-my-zsh
[2]:https://github.com/bhilburn/powerlevel9k
[3]:https://github.com/johnelse/spotify-cli
[4]:/2017/11/applescript-dictionaries/
[5]:/images/shell.png
[6]:https://github.com/the-quill/dotfiles/blob/master/.zshrc
