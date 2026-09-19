+++
title = "Let's snap Spiel"
description = "Packaging a text-to-speech framework."
date = "2026-08-09"

[taxonomies]
tags = ["accessibility", "text-to-speech"]
+++

[Spiel](https://project-spiel.org/) is a proposed replacement for the aging speech-dispatcher service on Linux. It's essentially the glue between applications that need to generate speech, like screen readers, and text-to-speech engines like [Piper](https://rhasspy.github.io/piper-samples/) that can generate it.

One selling point of Spiel over speech-dispatcher is that its architecture is better suited with modern sandboxed apps like snaps. But as far as I can tell, Spiel's distribution has been limited to flatpaks. Let's fix that.

### Using snap interfaces

Spiel's architecture puts TTS engines in their own processes, called speech providers, that communicate with client applications via D-Bus. When new speech providers are installed, any application using Spiel libraries can find it automatically using D-Bus service discovery.

Snaps support this kind of interface through [plugs and slots](https://snapcraft.io/docs/explanation/interfaces/all-about-interfaces/). The speech provider snap defines a D-Bus service, and the snap's executable entrypoint is configured to activate based on that connection. This means the speech provider daemon will only run when an application triggers it via a D-Bus message.
```yaml
# speech-provider-piper snap
slots:
  speech-provider:
    interface: dbus
    bus: session
    name: ai.piper.Speech.Provider

apps:
  speech-provider-piper:
    command: usr/local/bin/speech-provider-piper
    daemon: simple
    daemon-scope: user
    activates-on: [speech-provider]
```

A client application like Orca would define a corresponding plug with the same service name:
```yaml
# orca snap
plugs:
  speech-provider-piper:
    interface: dbus
    bus: session
    name: ai.piper.Speech.Provider
```

Once both snaps are installed, users can hook up the application to the speech provider manually:
```
$ snap connect orca:speech-provider-piper speech-provider-piper:speech-provider
```

### Defining voice packs

There's a third layer of packaging required here, beyond the speech providers and client applications: voices. TTS engines typically support many languages, with potentially many voice options for each. For a modern TTS engine like Piper, an installed voice can use hundreds of megabytes of disk space for the underlying AI model. It's therefore best to let users choose which subset of voices they'd like to install.

Snaps have a couple different concepts related to packaging content separately from an application. [Content snaps](https://snapcraft.io/docs/reference/interfaces/content-interface/) are published independently from the snaps they support, potentially even by a different author, and use interfaces to share resources. They're most valuable when the packaged content is used by multiple snaps, such as [GNOME runtimes](https://github.com/ubuntu/gnome-sdk). Since the Piper voice models will only be used by the speech provider snap, this isn't the case here.

On the other hand, [components](https://snapcraft.io/docs/explanation/how-snaps-work/snap-components/) are designed to package optional parts of a single application. This is the technique used by other snaps that have selectively-installable ML models, such as [gemma4](https://github.com/canonical/gemma4-snap). Components are also much faster to build than separate content snaps, because they are all built in the same build container as the snap they support. Since our Piper speech provider will have dozens of supported languages and locales, this can make a big difference here.

Defining components involves simply including a top-level `components` key and a part that writes the desired data into the component's directory. For Piper voices, we download the voice model and its metadata individually, because cloning the whole repository of voices would incur several gigabytes of network traffic and disk usage.
```yaml
# speech-provider-piper snap
components:
  voices-en-gb-alan-medium:
    type: standard
    summary: en-GB alan voice for Piper
    description: en-GB alan voice for Piper
    version: '0.1'

parts:
  voices-en-gb-alan-medium:
    plugin: dump
    source: ./component-base
    build-packages:
      - curl
      - ca-certificates
    override-build: |
      craftctl default
      set -eu
      mkdir -p "$CRAFT_PART_INSTALL/en_GB-alan-medium"
      curl -L --fail --retry 3 \
        -o "$CRAFT_PART_INSTALL/en_GB-alan-medium/en_GB-alan-medium.onnx" \
        "https://huggingface.co/rhasspy/piper-voices/resolve/main/en/en_GB/alan/medium/en_GB-alan-medium.onnx"
      curl -L --fail --retry 3 \
        -o "$CRAFT_PART_INSTALL/en_GB-alan-medium/en_GB-alan-medium.onnx.json" \
        "https://huggingface.co/rhasspy/piper-voices/resolve/main/en/en_GB/alan/medium/en_GB-alan-medium.onnx.json"
    organize:
      en_GB-alan-medium/en_GB-alan-medium.onnx: (component/voices-en-gb-alan-medium)/en_GB-alan-medium.onnx
      en_GB-alan-medium/en_GB-alan-medium.onnx.json: (component/voices-en-gb-alan-medium)/en_GB-alan-medium.onnx.json
```

### Next steps

The Orca screen reader has [experimental support for Spiel](https://github.com/GNOME/orca/blob/main/README.md#spiel-text-to-speech-support), and it could take advantage of these packages on systems that support snaps. Orca itself wouldn't need to be packaged as a snap to do so, though having an Orca snap would make installation convenient. However, Orca's unusual interaction patterns could make it tricky to run as a sandboxed app.

You can find snapcraft.yaml build configuration files and testing scripts [on GitHub](https://github.com/steinbro/spiel-snaps). Try it out, and add a few more languages and voices to the Piper speech provider while you're at it.