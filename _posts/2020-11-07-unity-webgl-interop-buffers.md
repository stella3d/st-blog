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


#### Unity

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

    public static void Initialize(int bufferCapacity = 1024)
    {
        const int indexedMidiMessageByteSize = 4;
        JsSharedMidiInBuffer = new byte[capacity * indexedMidiMessageByteSize];
        JsSharedTimestampInBuffer = new double[capacity];

        // pass buffers for input messages and their timestamps to JS
        InitJsInputBuffer(JsSharedMidiInBuffer, JsSharedMidiInBuffer.Length);
        InitInputTimestampBuffer(JsSharedTimestampInBuffer, JsSharedTimestampInBuffer.Length);
    }

    [DllImport("__Internal")]
    public static extern int ConsumeMidiInBuffer();
}
```

A definition of the data we want to read in Unity:

```csharp
using System;

public struct MidiMessage
{
  public byte StatusAndChannel;
  public byte Data1;
  public byte Data2;
}

// every 4 bytes in the message input buffer represents one of these
public struct IndexedMidiMessage
{
    public byte DeviceIndex;
    public MidiMessage Message;
}

public struct MidiInputEvent
{
    public double Time;
    public IndexedMidiMessage Data;
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


#### Browser
```js
const nice = 69;

function double(a) {
  return a * 2;
}
```

