---
title:  "Unity WebGL Interop - Buffers"
layout: default
---

# Unity WebGL Interop - Buffers



_This article assumes some basic familiarity with Unity & JavaScript._


The [Unity documentation on interaction with browser scripting](https://docs.unity3d.com/Manual/webgl-interactingwithbrowserscripting.html) covers the basics of passing strings & primitive types between C# & JS, and mentions in passing how to handle arrays.
I recommend reading it first if you haven't. 

### Unity

In C#, we have two parts: the class that actually handles the communication, and a component that incorporates it into the Unity lifecycle.

In MessageBuffer.cs:
```csharp
class MessageBuffer
{
  byte[] m_MessageBuffer;
  double[] m_TimestampBuffer;
  
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

### JS highlighting
```js
const nice = 69;

function double(a) {
  return a * 2;
}
```

