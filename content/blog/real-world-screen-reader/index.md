+++
title = "Audio annotations for the real world"
description = "Your phone always knows where you are, so why don't you"
date = "2026-06-19"

[taxonomies]
tags = ["accessibility", "text-to-speech"]
+++

Maps date back to a time when the most efficient way to describe a place was to draw it on paper. But for quite a few years, we've had the ability to layer that information on top of the real world, thanks to our smartphones. Your phone knows what's around you because it knows your exact GPS location together with information about all nearby venues, at least those that appear in online maps.

I remember this feature showing up on Windows Phone over a decade ago, through an app called Nokia City Lens. You could hold up your phone to a city street, and it would add floating labels on top of all the places it knew about.

![Nokia City Lens showing points of interest](./citylens.webp)
(source: [WIRED](https://www.wired.com/2012/05/hands-on-nokia-city-lens-beta-augmented-reality-for-your-lumia-phone/))

Less well-publicized was a Microsoft research project called Soundscape, which  did this augmented reality overlay as audio rather than video. Instead of drawing labels on top of a camera view, your phone speaks markers as spatial audio. The labels sound as if they emanate from different places and directions.

This turns out to be a pretty nifty navigational aid, especially if you can't easily read signs. Walking down the street with earbuds in and phone in your pocket, you hear the names of things around you as you approach them, with a decent sense of exactly where they are.

{{ youtube(id="v5DXykmOdJ8") }}

### From research to open-source project

Useful as this technology was, there wasn't much commercial potential, and [Microsoft discontinued Soundscape](https://www.microsoft.com/en-us/research/blog/microsoft-soundscape-new-horizons-with-a-community-driven-approach/) in 2023. But as there had been a dedicated fanbase among low-vision users, they first [dumped the source code](https://github.com/microsoft/soundscape) on GitHub, under an open-source license. After some initial concern as to whether anyone would step up to run the servers and republish the app, no fewer than three different groups picked it up. Each has taken Soundscape in different directions.

A solo developer published [VoiceVista](https://apps.apple.com/us/app/voicevista/id6450388413), a closed-source fork that adds all sorts of bells and whistles, like a "breadcrumb" mode. Separately, a Scottish nonprofit published a long-requested [Android version](https://play.google.com/store/apps/details?id=org.scottishtecharmy.soundscape&hl=en_US), which was no small feat. And finally, there is [Soundscape Community](https://soundscape.services/), the initiative I helped get off the ground and spent several years as a volunteer developer. They've established an open source community of contributors and users, and have been finding new uses and collaborations in adaptive sports, among others

### What makes Soundscape special

It's hard to explain what makes Soundscape feel so empowering, compared to audio turn-by-turn directions that are so commonplace. Most of all, there's a much greater sense of agency: Soundscape is not telling you where to go -- you follow whatever path you want, and it dutifully speaks up about what's around you, exactly when it's most relevant.

Soundscape is also unusual assistive tech because it's providing a visual aid without using any visual input. It relies entirely on GPS position and compass heading, not the camera. This makes it feel a bit like a spatial superpower. It can tell you the name of a park that has no marker, or the name of a street that has no sign. This makes it a natural complement to smart glasses that can describe a visual scene.
