+++
title = "Drawing tablet Windows setup"
date = 2018-04-24T10:53:43+05:30
tags = ["tablet", "drawing", "setup"]
draft = true
+++

1. Disable Windows defaults
    1. Windows 10
        + Run → services.msc → Touch Keyboard and Handwriting Panel Service; stop and disable
        + Disable _Tablet mode_ under _Tablet mode settings_
    2. Windows 7
        + Make sure to disable Tablet PC services from the _Windows Features_ list
        + Pen and Tablet settings disable _Flicks_
2. Install latest manufacturer drivers **before** first plug-in of the tablet
3. Plug-in device and test
4. Enable _Force Proportions_ to avoid disproportionate lines
    + Tablet Area: Full
    + Screen Area: Monitor
    + This mightn't be available on non-Wacom driver suits like Huion's.
5. [Calibrate the _Pressure Curve_](http://www.davidrevoy.com/article182/calibrating-wacom-stylus-pressure-on-krita) in the app (GIMP, Krita, etc.) by setting the pressure curve to get better greys, _if needed_


# References

1. GDQuest's [Introduction to graphics tablets](https://www.youtube.com/watch?v=RfTumTdNhho) and [Wacom tablet setup](https://www.youtube.com/watch?v=75lUVdq2Dto)
2. Frenden's [Huion H610 Pro, H610, & K58 Graphics Tablet Review](http://frenden.com/post/87110791272/huion-h610-pro-h610-k58-graphics-tablet-review)
