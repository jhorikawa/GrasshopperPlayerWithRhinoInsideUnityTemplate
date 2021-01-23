# GrasshopperPlayerWithRhinoInsideUnityTemplate

A project template for Unity interacting with Grasshopper using Rhino.Inside.

## To Add Custom UI (ex. Toggle)

### Step 1. Register Callback to receive data from Unity.

GrasshopperUnity.cs
```csharp

public GameObject togglePanelPrefab;

void Start(){
  ...
  ...
  Rhino.Runtime.HostUtils.RegisterNamedCallback("FromGHCreateToggle", FromGHCreateToggle);
  ...
  ...
}

void FromGHCreateToggle(object sender, Rhino.Runtime.NamedParametersEventArgs args)
{
    if (Application.isPlaying)
    {
        string id = "";
        string toggleName = "";
        bool val;
        if (args.TryGetString("id", out id))
        {
            args.TryGetString("name", out toggleName);
            args.TryGetBool("value", out val);

            var togglePanelObj = (GameObject)Instantiate(togglePanelPrefab, uiParent.transform);
            togglePanelObj.name = id;
            TogglePanel togglePanel = togglePanelObj.GetComponent<TogglePanel>();
            togglePanel.toggleLabel.text = toggleName;
            togglePanel.toggle.isOn = val;
            togglePanel.toggle.onValueChanged.AddListener((value) =>
            {
                SendToggleValue(id, value);
            });

        }
    }
}
```

```csharp
public void SendToggleValue(string id, bool val)
{
    using (var args = new Rhino.Runtime.NamedParametersEventArgs())
    {
        args.Set("toggleValue", val);
        Rhino.Runtime.HostUtils.ExecuteNamedCallback("ToGH_Toggle_" + id, args);
    }
}
```

TogglePanel.cs
```csharp
using UnityEngine.UI;
public class TogglePanel : MonoBehaviour
{
    public Toggle toggle;
    public Text toggleLabel;
}
```


### Step 2. Create Callback execution code to send data from Unity to Grasshopper.

```csharp
public void SendToggleValue(string id, bool val)
{
    using (var args = new Rhino.Runtime.NamedParametersEventArgs())
    {
        args.Set("toggleValue", val);
        Rhino.Runtime.HostUtils.ExecuteNamedCallback("ToGH_Toggle_" + id, args);
    }
}
```


### Step 3. C# code to send toggle value from GH to Unity.
```csharp
using System;
using System.Collections;
using System.Collections.Generic;

using Rhino;
using Rhino.Geometry;

using Grasshopper;
using Grasshopper.Kernel;
using Grasshopper.Kernel.Data;
using Grasshopper.Kernel.Types;

using Grasshopper.Kernel.Special;
using Grasshopper.GUI.Canvas;

public class Script_Instance : GH_ScriptInstance
{

  private void RunScript(bool toggle, bool init, ref object Value)
  {

    if(toggle != prevVal){
      prevVal = toggle;
      tglVal = toggle;
    }

    var inputs = Component.Params.Input;
    var finput = inputs[0];
    if(finput.SourceCount > 0){
      var tglComp = finput.Sources[0];
      if(tglComp.GetType() == typeof(GH_BooleanToggle)){
        var tgl = (GH_BooleanToggle) tglComp;
        var name = tgl.NickName;
        var val = tgl.Value;

        if(name != prevName || init || forceUpdate){

          using(var args = new Rhino.Runtime.NamedParametersEventArgs()){
            Component.PingDocument -= OnPingDocument;
            Component.PingDocument += OnPingDocument;
            Component.ObjectChanged -= OnObjectChanged;
            Component.ObjectChanged += OnObjectChanged;
            Grasshopper.Instances.ActiveCanvas.DocumentChanged -= OnDocumentChanged;
            Grasshopper.Instances.ActiveCanvas.DocumentChanged += OnDocumentChanged;

            args.Set("id", Component.InstanceGuid.ToString());
            args.Set("name", name);
            args.Set("value", val);
            Rhino.Runtime.HostUtils.ExecuteNamedCallback("FromGHClearUI", args);
            Rhino.Runtime.HostUtils.ExecuteNamedCallback("FromGHCreateToggle", args);

            Register(Component);
          }

          prevName = name;
          forceUpdate = false;
        }
      }
    }


    Value = tglVal;
  }

  // <Custom additional code> 
  bool tglVal = false;
  bool prevVal = false;
  IGH_Component comp = null;
  bool forceUpdate = false;

  string prevName = "";

  void Register(IGH_Component component)
  {
    Rhino.Runtime.HostUtils.RegisterNamedCallback("ToGH_Toggle_" + component.InstanceGuid.ToString(), ToGrasshopper);

    comp = component;
  }

  void ToGrasshopper(object sender, Rhino.Runtime.NamedParametersEventArgs args)
  {
    bool val;

    if (args.TryGetBool("toggleValue", out val)){
      tglVal = val;

      if(Component.Params.Input[0].SourceCount > 0){
        var c = Component.Params.Input[0].Sources[0];
        if(c.GetType() == typeof(Grasshopper.Kernel.Special.GH_BooleanToggle)){
          var toggle = (Grasshopper.Kernel.Special.GH_BooleanToggle) c;
          toggle.Value = tglVal;
        }
      }
    }

    comp.ExpireSolution(true);
  }

  public void OnPingDocument(object sender, GH_PingDocumentEventArgs e){
    if(e.Document == null){
      using(var args = new Rhino.Runtime.NamedParametersEventArgs()){
        args.Set("id", comp.InstanceGuid.ToString());
        Rhino.Runtime.HostUtils.ExecuteNamedCallback("FromGHClearUI", args);
      }
    }
  }

  public void OnObjectChanged(object sender, GH_ObjectChangedEventArgs e){
    if(e.Type == GH_ObjectEventType.Enabled){
      if(comp.Locked){
        using(var args = new Rhino.Runtime.NamedParametersEventArgs()){
          forceUpdate = true;
          args.Set("id", comp.InstanceGuid.ToString());
          Rhino.Runtime.HostUtils.ExecuteNamedCallback("FromGHClearUI", args);
        }
      }
    }

    comp.ExpireSolution(true);
  }

  public void OnDocumentChanged(object sender, GH_CanvasDocumentChangedEventArgs e){
    using(var args = new Rhino.Runtime.NamedParametersEventArgs()){
      args.Set("id", comp.InstanceGuid.ToString());
      Rhino.Runtime.HostUtils.ExecuteNamedCallback("FromGHClearUI", args);

      forceUpdate = true;

      if(comp != null){
        comp.ExpireSolution(true);
      }
    }
  }
  // </Custom additional code> 
}
```


Tutorial Video: https://youtu.be/geC94_pkPrs
