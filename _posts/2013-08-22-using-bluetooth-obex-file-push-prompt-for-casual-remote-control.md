---
title: Using Bluetooth OBEX file push prompt for casual remote control
date: 2013-08-22 10:41:33.000000000 +05:30
---

Nowadays most of the mobile devices include embeded blue-tooth capability, This article we intercept these blue-tooth devices, even lower end ones for causal remote control. <a href="http://en.wikipedia.org/wiki/OBject_EXchange">IrDA's  OBEX protocol</a> is primarily a proprietary protocol, the specification are only accessible from IrDA by paying a fees. 

[Download libpptmote](https://github.com/linuxexp/libpptmote)
[Download pptmode-win32](https://github.com/linuxexp/pptmote-win32)

By choosing OBEX protocol, we limit our options, and we invite the hassle to implement  little publicly known <a href="http://en.wikipedia.org/wiki/OBject_EXchange">OBEX protocol </a>ourself. But by using OBEX,  we can support even lower end mobile device, since many supports OBEX <a href="http://en.wikipedia.org/wiki/Bluetooth_profiles#File_Transfer_Profile_.28FTP.29">file push profile</a>.

Most of the blue-tooth devices (i.e mobiles) including lower end devices, supports OBEX (OBEX file push), this prompts the paired device to show a dialog, with notice to accept or reject the file (OBEX object), these prompts usually have some timeout associated with them. 

Therefore we are only left with `OBEX accept` as the only definitive option. This doesn't look very useful, but you can use a single option as a remote control for controlling presentation slides. This should vastly improve the presenter's and audience's experience, on average this could provide seamless wireless ppt control over 10-20 meters or more.

## PPTMote 
PPTMote (PPT reMote) is a little program that uses blue-tooth OBEX file prompt's responses to stimulate virtual key presses as required.

To provide seamless experience without any lags "pptmote" is entriely written in native code, in C, uses <a href="http://www.google.co.in/url?sa=t&amp;rct=j&amp;q=wiki%20microsoft%20bluetooth%20stack&amp;source=web&amp;cd=1&amp;cad=rja&amp;ved=0CC4QygQwAA&amp;url=http%3A%2F%2Fen.wikipedia.org%2Fwiki%2FBluetooth_stack%23Microsoft_Windows_stack&amp;ei=u_YVUsimDovPrQfB8YAg&amp;usg=AFQjCNHuRyRV3lWAkBhkNH3kEKN4YsWfow">Microsoft's native Bluetooth stack</a>. Since <a href="http://www.google.co.in/url?sa=t&amp;rct=j&amp;q=wiki%20microsoft%20bluetooth%20stack&amp;source=web&amp;cd=1&amp;cad=rja&amp;ved=0CC4QygQwAA&amp;url=http%3A%2F%2Fen.wikipedia.org%2Fwiki%2FBluetooth_stack%23Microsoft_Windows_stack&amp;ei=u_YVUsimDovPrQfB8YAg&amp;usg=AFQjCNHuRyRV3lWAkBhkNH3kEKN4YsWfow">Microsoft's Bluetooth stack</a> doesnt provide any implicit support for OBEX, pptmote implements the required OBEX support in hard-coded OBEX headers.

<p><a href="http://www.google.co.in/url?sa=t&amp;rct=j&amp;q=wiki%20microsoft%20bluetooth%20stack&amp;source=web&amp;cd=1&amp;cad=rja&amp;ved=0CCkQFjAA&amp;url=http%3A%2F%2Fen.wikipedia.org%2Fwiki%2FBluetooth_stack&amp;ei=u_YVUsimDovPrQfB8YAg&amp;usg=AFQjCNHuRyRV3lWAkBhkNH3kEKN4YsWfow">Bluetooth stack</a> is more or less like a TCP/IP <a href="http://en.wikipedia.org/wiki/Protocol_stack">networking stack</a>, this entire idea is implmented over <a href="http://en.wikipedia.org/wiki/RFCOMM?redirect=no">rfcomm sockets</a>, which provides TCP like capability. (i.e QoS)</p>
<p>For this purpose, I intercepted the bluetooth communication to determine the OBEX protocol, the result are as follows:</p>
<p>On successful communication, the client send a OBEX connect packet, which is as follows</p>

<pre>
0000-0010:  80 00 1a 10-00 ff ff 46-00 13 f9 ec-7b c4 95 3c  .......F ....{..&lt;
0000-001a:  11 d2 98 4e-52 54 00 dc-9e 09                    ...NRT.. ..
80    : Connect
00 1A : length of package, here 26 bytes
10    : obex version 1.0
00    : always zero
ff ff : maximum package length
46    : Header ID
00 13 : Total Header length 0x13
</pre>

<p>After that, the OBEX file prompt is sent,</p>

<pre>
0000-0010:  02 00 1b 01-00 13 00 70-00 70 00 74-00 6d 00 6f  .......p .p.t.m.o
0000-001b:  00 74 00 65-00 00 c3 00-00 00 07                 .t.e.... ...
</pre>

<p>If the user accepts, the required keys are stimulated and At last the OBEX disconnect packet is sent, before closing the rfcomm socket connections.</p>

<pre>
0000-0003:  81 00 03                                         ...
</pre>

<p>The program supports SDP (Service discovery protocol), but doesn't manually tries to find the OBEX port on the pptmote device using service UUIDs, although it'll list all services available on the remote device, user has too manually enter the OBEX port.</p>

## How to run

* Make sure that you change device name of your mob or any obex supporting device to "pptmote"
* Pair the "pptmote" device with your PC
* Start the program, it'll search for the "pptmote" device
* Once found, it'll list all the services available on the "pptmote" device by SDP
* You are supposed to enter the OBEX port number

That's it!, file prompts will be shown on your device, accepting will emulate "n" (0x4E) key on your PC

Close the program, when you are done.
