---
layout: post
title: Unity调用C++ DLL文件，更改系统音量 
tags: C#,unity,
categories: common
renderNumberedHeading: true
grammar_cjkRuby: true
---

Unity 更改系统音量，使用C++制作为Dll文件，C#调用。

```
using UnityEngine;
using System.Runtime.InteropServices;
using UnityEngine.UI;


//修改系统变量
public class UDAudioSystem : MonoBehaviour
{

    [DllImport("SystemVolumeControl")]
    private static extern int GetSystemVolume();
    [DllImport("SystemVolumeControl")]
    private static extern bool SetSystemVolume(int b);

    static int m_systemVolume = 0;

    private void Start()
    {
        GetAudioVolume();
    }

    private void Update()
    {
# if UNITY_EDITOR
        if (Input.GetKeyDown(KeyCode.PageUp))
        {
            m_systemVolume++;
            SetAudioVolume(m_systemVolume);
        }
        if (Input.GetKeyDown(KeyCode.PageDown))
        {
            m_systemVolume--;
            SetAudioVolume(m_systemVolume);
        }
#endif
    }


    public Slider _button;



    //得到系统音量
    public  int GetAudioVolume()
    {
        m_systemVolume = GetSystemVolume();
        return m_systemVolume;
    }
    /// <summary>
    /// 设置声音音量
    /// </summary>
    public  void SetAudioVolume(int volume)
    {
        m_systemVolume = volume;
        m_systemVolume = (int)Mathf.Clamp(m_systemVolume, 0, 100);
        SetSystemVolume(m_systemVolume);
    }
}

```

![SystemVolumeControl](./attachments/SystemVolumeControl.dll)