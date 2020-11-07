---
title:  "Unity WebGL Interop - Buffers"
layout: default
category: technical
tags: unity webgl
---

# Unity WebGL Interop - Buffers


_This article assumes some basic familiarity with Unity & JavaScript._


The [Unity documentation on interaction with browser scripting](https://docs.unity3d.com/Manual/webgl-interactingwithbrowserscripting.html) covers the basics of passing strings & primitive types between C# & JS, and mentions in passing how to handle arrays.
I recommend reading it first if you haven't. 


### Purpose

For this example, what we want to do is receive [MIDI message](https://en.wikipedia.org/wiki/MIDI#Messages) input in Unity from a hardware MIDI controller, via the browser's [Web MIDI API](https://www.w3.org/TR/webmidi/).


### Data

Before we write any code, we need to think about what data we need to communicate between languages.  
We're only receiving data from the browser to Unity, and we want to react to data once per frame.

In our case of MIDI Input, each incoming MIDI message has 3 properties we want to communicate to Unity:
  * The MIDI message itself - 3 bytes
  * A "device index" that tells us which input device the message is from - 1 byte
  * A timestamp in `double` format - 8 bytes 

#### Buffer Layout

When working in c/++, a simple, type-safe way to communicate the above data between languages would be to define a struct in both c# and c/++, and use that to create a single array, which is all you need.

```csharp
struct MidiMessage
{
  public byte StatusAndChannel;
  public byte Data1;
  public byte Data2;
}

struct MidiInputEvent
{
  public double Time;
  public byte DeviceIndex;
  public MidiMessage Message;
}

// other language references this array via pointer
MidiInputEvent[] Buffer;
```

However, to take advantage of [typed arrays in JS](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Typed_arrays), we can't use structs - each piece of data needs to be a primitive type. 
If we switch our data to a structure-of-arrays (SoA) layout, we can do this:

```csharp
struct MidiInputBuffers
{
  // every 4 bytes is laid out like the IndexedMidiMessage struct below.
  // corresponds to a Uint8Array in javascript.
  public byte[] IndexedMessages;

  // corresponds to a Float64Array in javascript
  public double[] Time;

  public MidiInputBuffers(int capacity = 1024)
  {
    // every 4 bytes in this buffer is one message 'element'
    IndexedMessages = new byte[capacity * 4];
    Time = new double[capacity];
  }
}

// every field in a midi message AND the device index is a `byte`, so we can put them together,
// without having to do any reinterpreting of bytes.  This also makes them align on 4 bytes. 
struct IndexedMidiMessage
{
  public byte DeviceIndex;
  public MidiMessage Message;
}
```


### Unity

In C#, we have a few parts. 
First, a static class with the data we're going to share with JavaScript, and some `[DllImport]` methods that call JavaScript functions.  
```csharp
public static class WebMidi
{
    public static byte[] JsSharedMidiInBuffer = new byte[0];
    public static double[] JsSharedTimestampInBuffer = new double[0];

    [DllImport("__Internal")]
    static extern void InitJsInputBuffer(byte[] array, int length);

    [DllImport("__Internal")]
    static extern void InitInputTimestampBuffer(double[] array, int length);

    public static void Initialize(MidiInputBuffers buffer)
    {
        // pass buffers for input messages and their timestamps to JS
        InitJsInputBuffer(buffer.IndexedMessages, buffer.IndexedMessages.Length);
        InitInputTimestampBuffer(buffer.Time, buffer.Time.Length);
    }

    [DllImport("__Internal")]
    public static extern int ConsumeMidiInBuffer();
}
```

A class that wraps the javascript function calls, and incorporates them into the Unity lifecycle.

```csharp
using System;

public class MessageBuffer
{
  byte[] m_MessageBuffer;
  double[] m_TimestampBuffer;
  
  Action<MidiMessage>[] m_InputDeviceHandlers;
  
  static void Initialize()
  {
    // TODO - implement
  }

  static int ConsumeBuffer()
  {
    // TODO - implement
  }
  
  static int GetBufferedMessageCount()
  {
    // TODO - implement
  }
}
```


### Browser
```js
const nice = 69;

function double(a) {
  return a * 2;
}
```

