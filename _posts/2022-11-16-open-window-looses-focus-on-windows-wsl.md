---
title: Open Window Loses Focus Issue on Windows, the Reason Is Wsl
categories: Windows WSL
featured_image: /assets/img/windows-wsl-focus-issue/1_e43YYm_XZI-inn_0-g4K3w.webp
featured_image_alt: Linux penguin on the windows os logo
comments: true

---


I had an issue that was very annoying and disturbing, and I didn’t notice it until I focused very well on it and had to diagnose it to find what’s the problem really is and find a solution (if we can call it that) to it.

## The symptoms :
Any window you are using will lose focus and if you were typing into it, you will lose the ability to do so until you take the mouse and click on it in order to be able to type again. And this happens frequently until it becomes really annoying. (Sorry, I got mad).

## The analysis :
After noticing the issue, I had to find a way to diagnose this and see why I was losing the window focus.

I had the idea to see what window has focus or what window/program is taking focus temporarily. So, I [found this PowerShell](https://stackoverflow.com/questions/46351885/how-to-grab-the-currently-active-foreground-window-in-powershell/70010344#70010344) script that prints the windows that currently have a focus in the console, and the good thing is that it runs continuously (while(1), who said infinite loops are bad).

```powershell
Add-Type  @"
using System;
using System.Runtime.InteropServices;
using System.Text;
public class APIFuncs
{
    [DllImport("user32.dll", CharSet = CharSet.Auto, SetLastError = true)]
    public static extern int GetWindowText(IntPtr hwnd,StringBuilder
    lpString, int cch);
    [DllImport("user32.dll", SetLastError=true, CharSet=CharSet.Auto)]
    public static extern IntPtr GetForegroundWindow();
    [DllImport("user32.dll", SetLastError=true, CharSet=CharSet.Auto)]
    public static extern Int32 GetWindowThreadProcessId(IntPtr hWnd,out
    Int32 lpdwProcessId);
    [DllImport("user32.dll", SetLastError=true, CharSet=CharSet.Auto)]
    public static extern Int32 GetWindowTextLength(IntPtr hWnd);
}
"@

while(1)
{
    $w = [apifuncs]::GetForegroundWindow()
    $len = [apifuncs]::GetWindowTextLength($w)
    $sb = New-Object text.stringbuilder -ArgumentList ($len + 1)
    $rtnlen = [apifuncs]::GetWindowText($w,$sb,$sb.Capacity)
    write-host "Window Title: $($sb.tostring())"
    sleep 1
}
```

The result was the following :

![An output of the powershell script](/assets/img/windows-wsl-focus-issue/1_kAwWQ0JgpgYHLRbRAZ36zQ.webp)


As you can see, this “RemoteApp” is taking over the focus from my “Google Chrome” Browser.

And after a couple of Google searches, I found out that this app is the Windows Remote Desktop Connection (RDP) app.

So, I opened the process explorer (you can find it and install it [here](https://learn.microsoft.com/en-us/sysinternals/downloads/process-explorer)) This is a verbose and detailed alternative for Windows Task Manager. I filtered by the name of the RDP process, which is (mstsc.exe). And found this :

![screenshot of the search](/assets/img/windows-wsl-focus-issue/1_S505ppO_Y4g7o2YF0Lmd8w.webp)

I was worried because I thought maybe someone has access to my computer over RDP. So, I check the [history of RDP](https://www.anyviewer.com/how-to/view-connection-history-remote-desktop-windows-10-2578.html) connections in my PC and it was empty, then I checked if [RDP was even enabled on my PC](https://www.anyviewer.com/how-to/how-to-check-if-remote-desktop-is-enabled-0007.html) but found nothing as well.

So, I figured that if you double-click on the process in Process Explorer you can see what command line launched the process, and here was the Eureka moment.


![The details of the RDP](/assets/img/windows-wsl-focus-issue/1_AIsDrLcne7G9YhjYyqen5Q.webp)


It was WSL (Windows Subsytem of linux, which I personally need to dev) and I have various tools that need WSL, like Docker.

I didn’t do any further research on why WSL needs an RDP connection, but it was stealing my focus (literally). Found other people complaining about the same problem. (like [here](https://youtrack.jetbrains.com/issue/IDEA-282196/Constant-loss-of-editor-focus-when-using-Windows-11-and-WSL), the last comment states that it has nothing to do with any JetBrains tools which I confirm, it’s not a JetBrains issue, I use Rider and IntelliJ.)

## The solution
I couldn’t find anything useful or a reasonable fix apart from restarting WSL using a command line to shut it down (be careful, if you have apps using WSL this will kill them or make them misbehave):

`wsl --shutdown`

you can run the following command to make sure all WSL are shut down :

`wsl --list --verbose`

And then run the following to restart it (or restart any app that uses it, it will launch it automatically) :

`wsl`

Thanks for reading this far, this was a quick post that maybe will help someone in the future. If you know a fix that I couldn’t find or have an insight about this issue, please put it in a comment or reach out to me, I can add it to this post with credits (of course!).