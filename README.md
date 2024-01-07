# Game stream (GeForce Now style) with AMD GPU : A quick'n'dirty guide
(Also works for other GPU such as Intel and Nvidia)

You want to rock your epic 4k HDR 120 Hz OLED in the living room and squeeze every bit of power from your computer through your gigabit LAN ?

Software-only solution, no dummy HDMI plug. No EDID flashing.

Here is a little how-to DIY stream.

## What is needed

[**Sunshine**](https://github.com/LizardByte/Sunshine), that will do the actual streaming.

[**Virtual-Display-Driver**](https://github.com/itsmikethetech/Virtual-Display-Driver) to create... a virtual display with the resolution of your choice to match your TV, up to 8k 500 Hz with HDR. (Windows 11 22H2+ needed for HDR)

[**MultiMonitorTool**](https://www.nirsoft.net/utils/multi_monitor_tool.html) from Nirsoft to disable/enable that virtual screen, and make it the primary display when needed.

[**ViGEmBus**](https://github.com/nefarius/ViGEmBus) that integrates fully with Sunshine to bring remote pads.
I'm not sure on this one as I installed it earlier, I believe it is now downloaded automatically with Sunshine's installer.
If your remote pad doesn't work, then check that ViGEmBus is present and has been installed with Sunshine.

And of course [**Moonlight**](https://github.com/moonlight-stream) as a client on any platform. ([downloads are here](https://moonlight-stream.org))

## Let's do this
For convenience's sake, I will use fixed paths in C:\, feel free to use your owns.

I will also interchangeably use *display* for a *monitor*, *monitor* for a *screen* and *screen* for a *display*.

### Sunshine

- Download and install Sunshine.
- Set it up, choose which resolutions you want to make available.
- Pair it and try it with a Moonlight client, check that it works ok from your default screen.

---

### Virtual-Display-Driver
- Download and install Virtual-Display-Driver.
- Set it up to match desired resolution, refresh rate and HDR.
Check that you can find the virtual screen with correct settings in Windows > System > Monitors.

---

### MultiMonitorTool
- Download and install MultiMonitorTool.

There are 2 ways of setting the monitors with Monitor Tools :

1. #### You can use MultiMonitor to load prepared configurations
(preferred, used in the powershell script later)

- In System > Display, set the virtual display on, and make it the default screen.

*(note : new windows will now spawn in this virtual display, you can click on their icon in the taskbar and take them to your screen with Win+Shift+Left/Right keys)*

- Then open a command line and use :
`C:\MultiMonitor\MultiMonitorTool.exe /SaveConfig gaming.conf`

- Disable the display in the System > Display window and set things back to normal.

- Then open a command line and use :
`C:\MultiMonitor\MultiMonitorTool.exe /SaveConfig standard.conf`

We'll use MultiMonitor to switch between those profiles.

2. #### You can use MultiMonitor to manually enable and set to primary each screen.
In this case, try it by yourself and update the Powershell script below.
The downside of this method is that by default, screens' positions will likely get messed up, hence the preferred #1 method.

Assuming your actual screen is 1 and your VirtualDisplay is screen 3 :
```
# Setting on for gaming
C:\MultiMonitor\MultiMonitorTool.exe /enable 3
C:\MultiMonitor\MultiMonitorTool.exe /SetPrimary 3

# Getting back to normal
C:\MultiMonitor\MultiMonitorTool.exe /SetPrimary 1
C:\MultiMonitor\MultiMonitorTool.exe /disable 3
```
(you can also use the `\\.\DISPLAY` format. You will have info on which is which with `C:\Program Files\Sunshine\tools\dxgi-info.exe`)

---

## Use them altogether with a small Powershell script :

```
# Load the gaming conf you made earlier (virtual display on and set as primary)
& "C:\MultiMonitor\MultiMonitorTool.exe" /LoadConfig C:\MultiMonitor\gaming.conf

# Start Sunshine
Start-Process -FilePath "C:\Program Files\Sunshine\sunshine.exe" -WorkingDirectory "C:\Program Files\Sunshine\"

# Wait a few secs for it to be launched
Start-Sleep -Seconds 5

# Move back its window to our #1 display so we can read what's going on and stop it
& "C:\MultiMonitor\MultiMonitorTool.exe" /MoveWindow \\.\DISPLAY1 Title "C:\Program Files\Sunshine\sunshine.exe"

# Check it's running
while ((Get-Process -Name Sunshine -ErrorAction SilentlyContinue)) {
  Start-Sleep -Seconds 1
}

# When stopped, rollback to normal display configuration
& "C:\MultiMonitor\MultiMonitorTool.exe" /LoadConfig C:\MultiMonitor\standard.conf
```

You might get an error about Windows refusing to run a unsigned Powershell script. You can get rid of that using `Set-ExecutionPolicy Unrestricted` in an Administrator Powershell console.

This modification is sensitive security-wise, use at your own risks and check what it does before doing so.

---

## HDR and colourspaces note

You may experience funny colours on the virtual display when using multiple HDR displays and a colourspace (P709) different of the Virtual Display (10-bit/P2020).

You can check this within Sunshine logs, when launching Sunshine on your main screen and Virtual Display, if you see these colourspaces are different, then I fixed this with this setting in Sunshine's conf :

In `C:\Program Files\Sunshine\config\sunshine.conf`, add this line
`encoderCscMode=1`

Not sure what it does, [as it is not really documented](https://docs.lizardbyte.dev/projects/sunshine/en/latest/source/src/video.html#_CPPv4N5video8config_t14encoderCscModeE), but that did the trick for me !

## Profit !
Again, this is only a quick'n'dirty guide with how I used many pieces of free and open source software to achieve home game-streaming (except Nirsoft's free closed-source but nonetheless awesome software).

If you liked what their work can achieve, please consider contributing or donating to [Virtual-Display-Driver](https://github.com/itsmikethetech/Virtual-Display-Driver), [Sunshine](https://app.lizardbyte.dev/Sunshine/), [IddSampleDriver](https://github.com/roshkins/IddSampleDriver?tab=readme-ov-file#fork-with-hdr), [Moonlight](https://github.com/moonlight-stream), and [ViGEmBus](https://github.com/nefarius/ViGEmBus?tab=readme-ov-file#sponsors). (no particular order)

Thank you and happy streaming !
