---
layout: post
title: UI 悬浮
tags: 新建,模板,小书匠
categories: common
renderNumberedHeading: true
grammar_cjkRuby: true
---

Unity悬浮窗

```
using UnityEngine;
using UnityEngine.EventSystems;

//器官认知中悬浮窗
public class UDOrganDetailsWindows : MonoBehaviour, IDragHandler,IPointerDownHandler,IPointerUpHandler,IEndDragHandler{
    public RectTransform canvas;          //得到canvas的ugui坐标
    private RectTransform imgRect;        //得到图片的ugui坐标
    Vector2 offset = new Vector3();    //用来得到鼠标和图片的差值
    public static UDOrganDetailsWindows instance;
    private void Awake()
    {
        if (instance==null)
        {
            instance = this;
        }
        imgRect = GetComponent<RectTransform>();
        UDEventControl.UDEventContr().eventDatas = OnPointerDown;
    }

    public void OnDestroy()
    {
        UDEventControl.UDEventContr().eventDatas = null;
    }
    private void Update()
    {
        if (_isDrag)
        {
            RestrictMoveInWindow(imgRect);
        }
      
    }
    public Vector2 _minRange;//限制区域的最小值
    public Vector2 _maxRange;//限制区域的最大值
    public int _downRestrictedArea = 0;// >0 悬浮窗可移动的区域
    public bool _isDrag = false;
    public void RestrictMoveInWindow(RectTransform originTrans)
    {
        Vector3 pos = originTrans.localPosition;
        _minRange.y = -(canvas.sizeDelta.y/2 - imgRect.sizeDelta.y)+ _downRestrictedArea;
        pos.x = Mathf.Clamp(originTrans.localPosition.x, _minRange.x, _maxRange.x);
        pos.y = Mathf.Clamp(originTrans.localPosition.y, _minRange.y, _maxRange.y);
        originTrans.localPosition = pos;
    }
    //当鼠标按下时调用 接口对应  IPointerDownHandler
    public void OnPointerDown(PointerEventData eventData)
    {
        GetOffset(eventData);
    }
    //得到偏移量
    private void GetOffset(PointerEventData eventData)
    {
        Vector2 mouseDown = eventData.position;    //记录鼠标按下时的屏幕坐标
        Vector2 mouseUguiPos = new Vector2();   //定义一个接收返回的ugui坐标
        //RectTransformUtility.ScreenPointToLocalPointInRectangle()：把屏幕坐标转化成ugui坐标
        //canvas：坐标要转换到哪一个物体上，这里img父类是Canvas，我们就用Canvas
        //eventData.enterEventCamera：这个事件是由哪个摄像机执行的
        //out mouseUguiPos：返回转换后的ugui坐标
        //isRect：方法返回一个bool值，判断鼠标按下的点是否在要转换的物体上
        bool isRect = RectTransformUtility.ScreenPointToLocalPointInRectangle(canvas, mouseDown, eventData.enterEventCamera, out mouseUguiPos);
        if (isRect)   //如果在
        {
            //计算图片中心和鼠标点的差值
            offset = imgRect.anchoredPosition - mouseUguiPos;
        }
        Debug.Log("鼠标按下");
    }

    public void OnDrag(PointerEventData eventData)
    {

        _isDrag = true;
        Vector2 mouseDrag = eventData.position;   //当鼠标拖动时的屏幕坐标
        Vector2 uguiPos = new Vector2();   //用来接收转换后的拖动坐标
        //和上面类似
        bool isRect = RectTransformUtility.ScreenPointToLocalPointInRectangle(canvas, mouseDrag, eventData.enterEventCamera, out uguiPos);

        if (isRect)
        {
            //设置图片的ugui坐标与鼠标的ugui坐标保持不变
            imgRect.anchoredPosition = offset + uguiPos;
        }
    }

    //当鼠标抬起时调用  对应接口  IPointerUpHandler
    public void OnPointerUp(PointerEventData eventData)
    {
        offset = Vector2.zero;
    }

    //当鼠标结束拖动时调用   对应接口  IEndDragHandler
    public void OnEndDrag(PointerEventData eventData)
    {
        offset = Vector2.zero;
        _isDrag = false;
    }

}
```