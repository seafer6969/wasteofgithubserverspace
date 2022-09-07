# A tutorial on how to stream application audio to discord using pulseaudio on linux.  
### Things you'll need:  
1. pulseaudio and all its many modules  
2. pavucontrol  
3. pagraphcontrol (not NEEDED but trust me you'll need it)  

To begin, you will need to take stock of your audio devices and application processes. For me, my only used audio device is my headset (Logitech G533) and all my use will come from there. If you're using `pagraphcontrol` (please use it) then you should organize your graph so it looks readable and undistracted. (figs 1 and 2)  
![scrot-2022-09-07 18:38 -0x2800007](https://user-images.githubusercontent.com/74340978/188999154-ba3ea1bc-b17e-4844-b3bf-61ff22e5398b.jpg)  
![scrot-2022-09-07 18:39 -0x2800007](https://user-images.githubusercontent.com/74340978/188999159-29aa10a5-1f89-493e-9764-8dec0b984936.jpg)  
  
As you can see, audio goes from source (input device) to WEBRTC (discord) to sink (output device) very cleanly and we have removed unused monitors and sinks. As a side note, to avoid problems that may arise when switching outputs on some application, the application whose audio you intend to stream should not be opened until we're done configuring the modules.  

We'll be using the `pacmd` command to manage the three modules we need for this task: `module-loopback`, `module-null-sink`, and `module-combine-sink`. Step one is to create a false sink to which we will redirect our application audio and microphone audio.  
The command: `pacmd load-module module-null-sink sink_name=[YOUR SINK NAME HERE]` creates this virtual sink we'll be using. You can name the sink whatever you want, but it will always show up as "Null Output" in pagraphcontrol unless you give it a description.  
The command: `pacmd update-sink-proplist [YOURSINKNAME] device.description=[YOURSINKDESC]` takes your argument for `device.description` and makes it the visible name in both pagraphcontrol and pavucontrol. (fig 3)  
![scrot-2022-09-07 18:49 -0x2800007](https://user-images.githubusercontent.com/74340978/188999242-4abcdf6c-fac9-44a7-9fa4-10c7dc4d9e5b.jpg)

Up next we will be creating a loopback which duplicates the source which represents our microphone and allows us to divert that cloned audio to another sink. This will default to your default output sink which will likely cause you to be able to hear yourself, so be warned.  
The command: `pacmd load-module module-loopback` does this. With pagraphcontrol you can simply drag the arrow from module-loopback to your default sink to the virtual sink we created.  
Our next step is to combine the virtual sink with our default sink, this way we'll be able to hear the application audio instead of only directing it to the stream. To do this you'll need the name of your virtual stream (which i hope you made easy to remember) and the *alsa_output* name of your default sink.  
The command: `pacmd list-sinks | grep '[PIECE OF DEFAULT SINK NAME HERE'` will show you the name you need.  
The command: `pacmd load-module module-combine-sinks slaves=[DEFAULT SINK NAME],[VIRTUAL SINK NAME]` will create *yet another new sink* called `Simultaneous output to [default sink] and [virtual sink]` which you will now change to be your fallback sink by right clicking the sink and selecting "Set as default" in pagraphcontrol or the checkmark icon on it in pavucontrol. (fig 4)  
![scrot-2022-09-07 18:59 -0x2800007](https://user-images.githubusercontent.com/74340978/188999296-07cb01e5-e07c-4f92-8f39-0439e626d3b2.jpg)

For the last step, you'll be changing the input device for discord to the virtual sink source. Discord won't show you this source in Voice & Video settings, so you will do this through pavucontrol. (fig 5)  
![scrot-2022-09-07 19:15](https://user-images.githubusercontent.com/74340978/188999364-483fd8ad-7bd9-45c8-adc8-d0513179754d.jpg)

For the last step, you'll be changing the input device for discord to the virtual sink source. Discord won't show you this source in Voice & Video settings, so you will do this through pavucontrol. (fig 5)  
