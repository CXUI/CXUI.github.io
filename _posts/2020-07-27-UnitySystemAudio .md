---
layout: post
title: unity中修改系统音量
tags: unity,c#,audio
categories: common
renderNumberedHeading: true
grammar_cjkRuby: true
---

在unity中修改系统音量，通过快捷键的方式调整音量
```
 //修改系统音量
public class UDAudioSystem : MonoBehaviour
{
    //通过调整音量的快捷键来调整音量大小
    #region System
    [DllImport("user32.dll")]
    static extern void keybd_event(byte bVk, byte bScan, UInt32 dwFlags, UInt32 dwExtraInfo);

    [DllImport("user32.dll")]
    static extern Byte MapVirtualKey(UInt32 uCode, UInt32 uMapType);

 

    private const byte VK_VOLUME_MUTE = 0xAD;
    private const byte VK_VOLUME_DOWN = 0xAE;
    private const byte VK_VOLUME_UP = 0xAF;
    private const UInt32 KEYEVENTF_EXTENDEDKEY = 0x0001;
    private const UInt32 KEYEVENTF_KEYUP = 0x0002;


    private void Start()
    {
        keybd_event(0x41, 0x1e, 0x0000, 0);   // 测试 按下 a
 
    }



    /// <summary>
    /// 改变系统音量大小，增加
    /// </summary>
    public void SystemVolumeUp()
    {
        keybd_event(VK_VOLUME_UP, MapVirtualKey(VK_VOLUME_UP, 0), KEYEVENTF_EXTENDEDKEY, 0);
        keybd_event(VK_VOLUME_UP, MapVirtualKey(VK_VOLUME_UP, 0), KEYEVENTF_EXTENDEDKEY | KEYEVENTF_KEYUP, 0);
        Debug.Log(string.Format("VK_VOLUME_UP:{0}  MapVirtualKey(VK_VOLUME_UP, 0):{1}  KEYEVENTF_EXTENDEDKEY | KEYEVENTF_KEYUP:{2} KEYEVENTF_EXTENDEDKEY:{3} ", VK_VOLUME_UP, MapVirtualKey(VK_VOLUME_UP, 0), KEYEVENTF_EXTENDEDKEY | KEYEVENTF_KEYUP, KEYEVENTF_EXTENDEDKEY));
    }

    /// <summary>
    /// 改变系统音量大小，减小
    /// </summary>
    public void SystemVolumeDown()
    {
        keybd_event(VK_VOLUME_DOWN, MapVirtualKey(VK_VOLUME_DOWN, 0), KEYEVENTF_EXTENDEDKEY, 0);
        keybd_event(VK_VOLUME_DOWN, MapVirtualKey(VK_VOLUME_DOWN, 0), KEYEVENTF_EXTENDEDKEY | KEYEVENTF_KEYUP, 0);
        Debug.Log(string.Format("VK_VOLUME_DOWN:{0} MapVirtualKey(VK_VOLUME_DOWN, 0):{1}  KEYEVENTF_EXTENDEDKEY | KEYEVENTF_KEYUP:{2} KEYEVENTF_EXTENDEDKEY:{3} ", VK_VOLUME_DOWN, MapVirtualKey(VK_VOLUME_DOWN, 0), KEYEVENTF_EXTENDEDKEY | KEYEVENTF_KEYUP, KEYEVENTF_EXTENDEDKEY));
    }

    /// <summary>
    /// 改变系统音量大小，静音
    /// </summary>
    public void SystemMute()
    {
        keybd_event(VK_VOLUME_MUTE, MapVirtualKey(VK_VOLUME_MUTE, 0), KEYEVENTF_EXTENDEDKEY, 0);
        keybd_event(VK_VOLUME_MUTE, MapVirtualKey(VK_VOLUME_MUTE, 0), KEYEVENTF_EXTENDEDKEY | KEYEVENTF_KEYUP, 0);
    }
    #endregion
    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.A))
        {
            Debug.Log("aaaaaaaaaaa");
        }

        if (Input.GetKeyDown(KeyCode.UpArrow))
            SystemVolumeUp();
        if (Input.GetKeyDown(KeyCode.DownArrow))
            SystemVolumeDown();
        if (Input.GetKeyDown(KeyCode.Space))
            SystemMute();
    }
}
```