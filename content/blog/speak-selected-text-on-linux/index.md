+++
title = "An unobtrusive text-to-speech workflow in GNOME"
description = "Speak selected text improved with Speech Dispatcher."
date = "2026-06-06"

[taxonomies]
tags = ["accessibility", "text-to-speech"]
+++

For every blind user who can't see their computer screen at all, there are many more low-vision users with some usable vision but not enough to read text on the screen comfortably. Users like this could benefit greatly from using more speech, but it's hard to recommend adopting a full-fledged screen reader. These tools can feel like too much for a user with significant remaining vision. Even beyond the massive set of keyboard shortcuts to learn, many applications are not optimized for assistive tech and can become even harder to navigate. No matter how masterful the user's command of their screen reader, they might conclude they're better off just relying on magnification and their remaining vision .

When text-to-speech becomes not a convenience but a necessity, the user needs their operating system to make it available system-wide, across all applications. Making that a reality on major Linux desktop environments without turning on a screen reader requires some tinkering.

### Other operating systems already do it

Other desktop operating systems figured this out some time ago. macOS has had ["Speak Selection" functionality](https://support.apple.com/en-ca/guide/mac-help/mh27448/mac) for ages. Microsoft introduced a ["Read from here" feature](https://blogs.windows.com/windowsexperience/2020/05/21/whats-coming-in-windows-10-accessibility/) through Magnifier in Windows 10. The [Reader View in Firefox](https://support.mozilla.org/en-US/kb/firefox-reader-view-clutter-free-web-pages) has text-to-speech built in as well.

Screen readers also have some features that are geared towards partially-sighted users. Both NVDA on Windows and VoiceOver on macOS support a mode that reads text under the mouse pointer. But on Linux, [Orca's Mouse Review feature works inconsistently under Wayland](https://discourse.gnome.org/t/mouse-review-is-hit-and-miss/34949) the modern display system for Linux.

### Prior art

A [2021 blog post](https://dev.to/tylerlwsmith/read-selected-text-out-loud-on-ubuntu-linux-45lj) demonstrates how to set up a keyboard shortcut to trigger text-to-speech using an impressively short shell script. The script simply grabs the highlighted text from the clipboard's primary selection buffer, and feeds that directly into the espeak synthesizer.

This works great much of the time, putting aside espeak's very robotic voice characteristic of the 1980s-era speech synthesis techniques. But you'll run into many unhandled edge cases if you use it extensively. For example, it struggles with nonstandard characters, like the Unicode characters used not infrequently to format social media posts. The word "𝐉𝐚𝐯𝐚𝐒𝐜𝐫𝐢𝐩𝐭" is read unintelligibly as "letter 1d409, letter 1d14a, ..." And worse, emoji characters are silently dropped.

My favorite failure mode is when the script encounters text surrounded by double square brackets. The espeak synthesizer interprets what's inside as raw phonetic data. The result is text that is a very exotic flavor of garbled. [[ If you happen to be using it now, this example will demonstrate exactly what I  mean,. ]]

### A native extension for GNOME

I've published a [GNOME shell extension](https://extensions.gnome.org/extension/9659/speak-selection/) that adds some requisite sophistication to the same basic idea. The integration with the GNOME desktop is an improvement, but more valuable is using speech dispatcher. This service is far from new but addresses many of the problems we saw, translating Unicode characters, reading emoji names, and stripping speech synthesizer control characters. Moreover, as a system-wide service sitting between applications and the speech synthesizer, it improves the listening experience as well.  It prevents multiple voices from speaking over each other, as well as maintaining global settings for voice, speech rate, and so on.

Nevertheless, since we're still using fundamentally the same text-capturing method, some limitations remain. In particular, not all text in applications is speakable. Some frameworks, notably Flutter, do not implement the primary selection buffer, so our extension does not recognize when text has been highlighted in applications that use this framework.