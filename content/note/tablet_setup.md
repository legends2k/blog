+++
title = "Drawing tablet setup on Windows"
date = 2018-04-24T10:53:43+05:30
tags = ["hardware", "drawing"]
+++

# Steps

1. Disable Windows defaults
    1. Windows 10
        + Run → `services.msc` → _Touch Keyboard and Handwriting Panel Service_; stop and disable
        + Disable _Tablet mode_ under _Tablet mode settings_
    2. Windows 7
        + Disable _Tablet PC services_ from the _Windows Features_ list
        + Disable _Flicks_ in _Pen and Tablet_ settings
2. Install latest manufacturer drivers **before** first plugging in to the system
3. Plug in device and test
4. Enable _Force Proportions_ to avoid disproportionate lines
    + Tablet Area: Full
    + Screen Area: Monitor
    + _Force Proportions_ mightn't be available on non-Wacom driver software like Huion's.
5. [Calibrate _Pressure Curve_](http://www.davidrevoy.com/article182/calibrating-wacom-stylus-pressure-on-krita) in the app (GIMP, Krita, etc.) by setting the pressure curve to get better greys, _if needed_


# References

1. Frenden's [Huion H610 Pro, H610, & K58 Graphics Tablet Review](http://frenden.com/post/87110791272/huion-h610-pro-h610-k58-graphics-tablet-review)
2. GDQuest’s [Free Krita Tutorial for Game Artists](http://gdquest.com/tutorial/art/krita-tutorial-for-game-artists/)
  + [Introduction to graphics tablets](https://www.youtube.com/watch?v=RfTumTdNhho)
  + [Wacom tablet setup](https://www.youtube.com/watch?v=75lUVdq2Dto)
