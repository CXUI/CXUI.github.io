---
layout: post
title: Unity 模仿win 鼠标多选框
tags: unity
categories: common
renderNumberedHeading: true
grammar_cjkRuby: true
---

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MuiltSele : MonoBehaviour {

    public enum keyType
    {
        Radio,//单选
        MultiSelect,//多选
        SameType,//同类型选取
    }

    public GameObject cuec;
    public Material mat;
    private Vector2 FirstMouseposition;
    private Vector2 SecondMousePosition;
    private bool StartRender = false;
    private Renderer[] gameObjects;
    private keyType curType;

    private Renderer CurPickObj;
    private int uptime = 0;

    public List<GameObject> PickObj = new List<GameObject>();
    public List<GameObject> NoPickObj = new List<GameObject>();
    public float SameTypeDir = 10;//同类型距离

    private void Start()
    {
        for (int i = 0; i < 100; i++)
        {
            GameObject go = GameObject.Instantiate(cuec);
            go.transform.position = Vector3.forward * Random.Range(-50, 50) + Vector3.left * Random.Range(-50, 50);
        }
        UpdataObjArr();
    }

    public void UpdataObjArr()
    {
        gameObjects = GameObject.FindObjectsOfType<Renderer>();
    }

    private void Update()
    {
        if (Input.GetMouseButtonDown(0))
        {
            StartRender = true;
            FirstMouseposition = Input.mousePosition;
            PickGameObject();
        }
        else if (Input.GetMouseButtonUp(0))
        {
            if ((FirstMouseposition - SecondMousePosition).sqrMagnitude < 0.2f)
            {
                SameTypePick();
            }
            else
            {
                ChangeTwoPoint();
                PickGameObject();
            }
            StartRender = false;
            FirstMouseposition = SecondMousePosition = Vector2.zero;
        }
        SecondMousePosition = Input.mousePosition;
    }

    #region 绘图

    private void OnPostRender()
    {
        if (StartRender)
        {
            mat.SetPass(0);
            GL.LoadOrtho();
            GL.Begin(GL.LINES);
            DrawLine(FirstMouseposition.x, FirstMouseposition.y, SecondMousePosition.x, SecondMousePosition.y);
            GL.End();
        }
    }

    private void DrawLine(float x1, float y1, float x2, float y2)
    {
        GL.Vertex(new Vector3(x1 / Screen.width, y1 / Screen.height, 0));
        GL.Vertex(new Vector3(x2 / Screen.width, y1 / Screen.height, 0));
        GL.Vertex(new Vector3(x2 / Screen.width, y1 / Screen.height, 0));
        GL.Vertex(new Vector3(x2 / Screen.width, y2 / Screen.height, 0));
        GL.Vertex(new Vector3(x2 / Screen.width, y2 / Screen.height, 0));
        GL.Vertex(new Vector3(x1 / Screen.width, y2 / Screen.height, 0));
        GL.Vertex(new Vector3(x1 / Screen.width, y2 / Screen.height, 0));
        GL.Vertex(new Vector3(x1 / Screen.width, y1 / Screen.height, 0));
    }

    #endregion 绘图

    //同类型或者单选
    private void SameTypePick()
    {
        PickObj.Clear();
        NoPickObj.Clear();
        var ra = Camera.main.ScreenPointToRay(Input.mousePosition);
        RaycastHit hit;
        if (Physics.Raycast(ra, out hit))
        {
            var newobj = hit.transform.GetComponent<Renderer>();
            if (CurPickObj == newobj && Time.frameCount - uptime < 30)
            {
                float sdis = SameTypeDir * SameTypeDir;
                for (int i = 0; i < gameObjects.Length; i++)
                {
                    var tt = gameObjects[i];
                    float dis = (tt.transform.position - newobj.transform.position).sqrMagnitude;
                    if (dis < sdis)
                    {
                        PickObj.Add(tt.gameObject);
                        SetSelect(tt, true);
                    }
                    else
                    {
                        NoPickObj.Add(tt.gameObject);
                        SetSelect(tt, false);
                    }
                }
                curType = keyType.SameType;
                CurPickObj = null;
            }
            else
            {
                curType = keyType.Radio;
                CurPickObj = newobj;
                SetSelect(CurPickObj, true);

                uptime = Time.frameCount;
                PickObj.Add(CurPickObj.gameObject);

                for (int i = 0; i < gameObjects.Length; i++)
                {
                    if (!CurPickObj == gameObjects[i])
                    {
                        NoPickObj.Add(gameObjects[i].gameObject);
                    }
                }
            }
        }
        else
        {
            CurPickObj = null;
        }
    }

    //触摸选择
    private void PickGameObject()
    {
        PickObj.Clear();
        NoPickObj.Clear();
        for (int i = 0; i < gameObjects.Length; i++)
        {
            var tt = gameObjects[i];
            if (Area(tt.transform))
            {
                SetSelect(tt, true);
                PickObj.Add(tt.gameObject);
            }
            else
            {
                SetSelect(tt, false);
                NoPickObj.Add(tt.gameObject);
            }
        }
        if (PickObj.Count > 0)
        {
            curType = keyType.SameType;
        }
    }

    private void ChangeTwoPoint()
    {
        if (FirstMouseposition.x > SecondMousePosition.x)
        {
            float position1 = FirstMouseposition.x;
            FirstMouseposition.x = SecondMousePosition.x;
            SecondMousePosition.x = position1;
        }

        if (FirstMouseposition.y > SecondMousePosition.y)
        {
            float position2 = FirstMouseposition.y;
            FirstMouseposition.y = SecondMousePosition.y;
            SecondMousePosition.y = position2;
        }
    }

    public bool Area(Transform obj)
    {
        Vector3 position = Camera.main.WorldToScreenPoint(obj.transform.position);
        if (position.x >= FirstMouseposition.x &
            position.y >= FirstMouseposition.y &
            position.x <= SecondMousePosition.x &
            position.y <= SecondMousePosition.y)
        {
            return true;
        }
        return false;
    }

    //设置选取状态
    public void SetSelect(Renderer obj, bool se)
    {
        if (se)
        {
            obj.material.color = Color.red;
        }
        else
        {
            obj.material.color = Color.white;
        }
    }
}

```