---
layout: post
title:  "Close a window by title in C#"
date: 2015-04-15 20:24:00 +0000
categories: csharp windows
excerpt_separator: <!--end_excerpt-->
---

When you're developing for embedded systems that don't have a mouse or keyboard attached, a misbehaving program that decides to pop up windows at random is suddenly a lot more inconvenient.
<!--end_excerpt-->
Cue the below code snippet, which takes in a window title and sets it's state to minimised, maximised, or normal depending on the parameters you pass in. As usual, this is a Linqpad script. You just need to add a reference to **System.Runtime.InteropServices**, which is part of .net 4 and above.

{% highlight csharp %}
void Main()
{
 var windowTitle = "Untitled - Notepad";

  IntPtr hWnd = FindWindow(null, windowTitle);
    if (!hWnd.Equals(IntPtr.Zero))
    {
        ShowWindowAsync(hWnd, SW_SHOWMINIMIZED);
    }
 
}

// Define other methods and classes here
private const int SW_SHOWNORMAL = 1;
private const int SW_SHOWMINIMIZED = 2;
private const int SW_SHOWMAXIMIZED = 3;

[DllImport("user32.dll")]
private static extern bool ShowWindowAsync(IntPtr hWnd, int nCmdShow);

[DllImport("user32.dll", EntryPoint = "FindWindow")]
private static extern IntPtr FindWindow(string lp1, string lp2);
{% endhighlight %}

This just imports the FindWindow and ShowWindowAsync methods from the user32 assembly. FindWindow is used to find the window we want to close. This return a pointer to the window handle, which we then pass to ShowWindowAsync along with an int indicating what we want to do to the window.

In the above example, I already know the window title, but this is unlikely to be the case in the real world. You can get this with the below snippet which will select out the window title and the process name. You can obviously add a where clause and modify the results based on your needs.

{% highlight csharp %}
Process.GetProcesses().Select (p => new{p.ProcessName, p.MainWindowTitle})
{% endhighlight %}