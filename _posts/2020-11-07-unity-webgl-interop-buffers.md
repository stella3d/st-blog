---
title:  "Unity WebGL Interop - Buffers"
layout: default
category: technical
tags: unity webgl
---

# Unity WebGL Interop - Buffers
_This article assumes some basic familiarity with Unity, C#, & JavaScript._

There's a lot of useful web APIs available in the browser, which we might want to use in Unity.  

The [Unity documentation on interaction with browser scripting](https://docs.unity3d.com/Manual/webgl-interactingwithbrowserscripting.html) covers the basics of passing strings & primitive types between C# & JS, and mentions in passing how to handle arrays.
I recommend reading it first if you haven't. 

There's also the part of Emscripten's documentation that deals with [accessing memory from JavaScript](https://emscripten.org/docs/porting/connecting_cpp_and_javascript/Interacting-with-code.html#interacting-with-code-access-memory).  We are not going to use `getValue()` / `setValue()` as described there - we're going to expand upon these two sentences:
```
You can also access memory 'directly' by manipulating the arrays that represent memory. 
This is not recommended unless you are sure you know what you are doing, 
and need the additional speed over getValue() and setValue().
```

The post is to help you be sure of what you're doing!

### Example Task

For this example, what we want to do is receive [MIDI message](https://en.wikipedia.org/wiki/MIDI#Messages) input in Unity from a hardware MIDI controller, via the browser's [Web MIDI API](https://www.w3.org/TR/webmidi/).


### Data

Before we write any code, we need to think about what data we need to communicate between languages.  
We're only receiving data from the browser to Unity, and we want to react to data once per frame.

In our case, each incoming MIDI message has 3 properties we want to communicate to Unity:
  * The MIDI message itself - 3 bytes
  * A 'device ID' - 1 byte
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


We're going to do 2 things to solve this:

1) Switching our data to a [structure-of-arrays](https://en.wikipedia.org/wiki/AoS_and_SoA) (SoA) layout

2) Grouping data elements by primitive type

```csharp
struct IndexedMidiMessage
{
  // If all the members of your struct are the same primitive type, you can 
  // put them in the same struct. This makes accessing data by index in C# more efficient.
  // in this case, it also helps the midi messages align nicely to 4 bytes.
  public byte DeviceIndex;
  public MidiMessage Message;
}

struct MidiInputBuffers
{
  // corresponds to a Uint8Array in javascript, with 4 times as many elements
  public IndexedMidiMessage[] Messages;

  // corresponds to a Float64Array in javascript
  public double[] Time;

  public MidiInputBuffers(int capacity = 1024)
  {
    IndexedMessages = new byte[capacity];
    Time = new double[capacity];
  }
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
    static extern void InitJsInputBuffer(IndexedMidiMessage[] array, int length);

    [DllImport("__Internal")]
    static extern void InitInputTimestampBuffer(double[] array, int length);

    public static void Initialize(MidiInputBuffers buffer)
    {
        // pass buffers for input messages and their timestamps to JS
        InitJsInputBuffer(buffer.IndexedMessages, buffer.IndexedMessages.Length);
        InitInputTimestampBuffer(buffer.Time, buffer.Time.Length);
    }

    [DllImport("__Internal")]
    // TODO - note about how you must consume / copy the data immediately.
    public static extern int ConsumeMidiInBuffer();
}
```

A class that wraps the javascript function calls, and incorporates them into the Unity lifecycle.

```csharp
using System;

public class MessageBuffer
{ 
  static void Initialize()
  {
    // TODO - implement
  }

  static int ConsumeBuffer()
  {
    // TODO - implement
  }
}
```


### Browser


First, we have to setup [TypedArrays] that correspond to the buffers we defined in c#.


The `byteOffset` parameter 

```js
    // passed (double[], int) from C#. Float64Array = 'double[]'
    InitInputTimestampBuffer: function(byteOffset, length) {
      // this creates a new view on the underlying buffer, instead of allocating new memory
      window.midiInTimestamps = new Float64Array(buffer, byteOffset, length);
    },
```
```js
    // passed (IndexedMidiMessage[], int) from Unity. Uint8Array = 'byte[]'in C#
    InitJsInputBuffer: function(byteOffset, length) {
      // the struct is 4 bytes, so multiply by 4 to get length of the Uint8 view.
      window.midiInMessages = new Uint8Array(buffer, byteOffset, length * 4);
    },
```

```js
// A function like this would be called for every midi input message received
function onMidiMessage(message, deviceIndex) {
    var count = window.midiInBufferCount;
    var buffer = window.midiInMessages;
    var startIndex = count * 4;

    buffer[startIndex] = deviceIndex;
    buffer[startIndex + 1] = message.data[0];
    buffer[startIndex + 2] = message.data[1];
    buffer[startIndex + 3] = message.data[2];

    window.midiInTimestamps[count] = message.timeStamp;
    window.midiInBufferCount += 1;
};
```

