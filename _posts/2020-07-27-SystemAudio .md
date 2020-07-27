---
layout: post
title: 调出系统音量程序
tags: c#,System,audio
categories: common
renderNumberedHeading: true
grammar_cjkRuby: true
---


```
    /*
     * 弹出系统音量控制器
     * */
    public static void PopupController()
    {
        System.Diagnostics.ProcessStartInfo Info = new System.Diagnostics.ProcessStartInfo();
        Info.FileName = "Sndvol";
        System.Diagnostics.Process.Start(Info);
    }
	```