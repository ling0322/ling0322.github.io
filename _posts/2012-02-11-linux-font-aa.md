--- 
layout: post
title: Linux下Simsun字体英文关AA的设置
---

在~目录下建立一个.fonts.conf文件里面的内容为

    <match target="font">
    <test name="pixelsize" compare="less_eq"><double>16.5</double></test>
    <test name="pixelsize" compare="more_eq"><double>10.5</double></test>
    <test qual="any" name="family"><string>SimSun</string></test>
    <edit name="antialias" mode="assign"><bool>false</bool></edit>
    </match>


就是将[11px, 16px]之间的宋体强制点阵，解决宋体的网页英文看上去各种不爽的问题