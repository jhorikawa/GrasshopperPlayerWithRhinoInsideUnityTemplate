# GrasshopperPlayerWithRhinoInsideUnityTemplate

A project template for Unity interacting with Grasshopper using Rhino.Inside.



### C# code to send slider from GH to Unity.
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
  private void RunScript(double num, bool init, ref object Value)
  {
    if(num != prevVal){
      prevVal = num;
      sliderVal = num;
    }

    var inputs = Component.Params.Input;
    var finput = inputs[0];
    if(finput.SourceCount > 0){
      var slicomp = finput.Sources[0];
      if(slicomp.GetType() == typeof(GH_NumberSlider)){
        var slider = (GH_NumberSlider) slicomp;
        double min = Convert.ToDouble(slider.Slider.Minimum);
        double max = Convert.ToDouble(slider.Slider.Maximum);
        double val = Convert.ToDouble(slider.Slider.Value);
        var name = slider.NickName;
        int decim = slider.Slider.DecimalPlaces;
        int type = (int) slider.Slider.Type;
        Print(forceUpdate.ToString());

        if(forceUpdate || min != prevMin || max != prevMax || name != prevName || decim != prevDecim || type != prevType || init){
          using(var args = new Rhino.Runtime.NamedParametersEventArgs()){
            Component.PingDocument -= OnPingDocument;
            Component.PingDocument += OnPingDocument;
            Component.ObjectChanged -= OnObjectChanged;
            Component.ObjectChanged += OnObjectChanged;
            Grasshopper.Instances.ActiveCanvas.DocumentChanged -= OnDocumentChanged;
            Grasshopper.Instances.ActiveCanvas.DocumentChanged += OnDocumentChanged;

            args.Set("id", Component.InstanceGuid.ToString());
            args.Set("name", name);
            args.Set("min", min);
            args.Set("max", max);
            args.Set("decimal", decim);
            args.Set("value", val);
            args.Set("type", type);
            Rhino.Runtime.HostUtils.ExecuteNamedCallback("FromGHClearUI", args);
            Rhino.Runtime.HostUtils.ExecuteNamedCallback("FromGHCreateSlider", args);

            Register(Component);
          }

          prevMin = min;
          prevMax = max;
          prevName = name;
          prevDecim = decim;
          prevType = type;
          forceUpdate = false;
        }
      }
    }


    Value = sliderVal;
  }

  // <Custom additional code> 
  double sliderVal = 0.0;
  double prevVal = 0.0;
  IGH_Component comp = null;

  double prevMin = double.PositiveInfinity;
  double prevMax = double.PositiveInfinity;
  string prevName = "";
  int prevDecim = -1;
  int prevType = -1;
  bool forceUpdate = false;

  void Register(IGH_Component component)
  {
    Rhino.Runtime.HostUtils.RegisterNamedCallback("ToGH_Slider_" + Component.InstanceGuid.ToString(), ToGrasshopper);

    comp = component;
  }

  void ToGrasshopper(object sender, Rhino.Runtime.NamedParametersEventArgs args)
  {
    double val;
    if (args.TryGetDouble("sliderValue", out val)){
      sliderVal = val;

      if(Component.Params.Input[0].SourceCount > 0){
        var c = Component.Params.Input[0].Sources[0];
        if(c.GetType() == typeof(Grasshopper.Kernel.Special.GH_NumberSlider)){
          var slider = (Grasshopper.Kernel.Special.GH_NumberSlider) c;
          slider.SetSliderValue(Convert.ToDecimal(sliderVal));
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

### C# code to send toggle value from GH to Unity.
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

### C# code to send mesh from GH to Unity.
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

using System.Linq;
using System.Drawing;
using Grasshopper.GUI.Canvas;

public class Script_Instance : GH_ScriptInstance
{
  private void RunScript(DataTree<Mesh> meshes, DataTree<object> mats, ref object outMesh, ref object outMat)
  {
    if(comp == null){
      Component.PingDocument -= OnPingDocument;
      Component.PingDocument += OnPingDocument;
      Component.ObjectChanged -= OnObjectChanged;
      Component.ObjectChanged += OnObjectChanged;
      Grasshopper.Instances.ActiveCanvas.DocumentChanged -= OnDocumentChanged;
      Grasshopper.Instances.ActiveCanvas.DocumentChanged += OnDocumentChanged;

      comp = Component;
    }
    prevMeshes = meshes;

    DataTree<GH_Material> ghMats = new DataTree<GH_Material>();
    ClearAllMeshes(meshes);

    List <int> tempDataTreeCounts = new List<int>();
    for(int i = 0; i < meshes.BranchCount; i++){
      GH_Path path = meshes.Paths[i];
      tempDataTreeCounts.Add(meshes.Branches[i].Count);

      List<object> matList = mats.Branches[Math.Min(i, mats.BranchCount - 1)];
      for(int n = 0; n < meshes.Branches[i].Count; n++){
        Mesh mesh = meshes.Branches[i][n];
        object mat = matList[Math.Min(n, matList.Count - 1)];
        using(var args = new Rhino.Runtime.NamedParametersEventArgs())
        {
          Color color = Color.White;
          Color emission = Color.Black;
          Color specular = Color.DarkGray;
          double transparency = 0f;
          double shine = 50;


          if(mat != null && mat.GetType() == typeof(Rhino.Display.DisplayMaterial)){
            var material = (Rhino.Display.DisplayMaterial) mat;
            color = material.Diffuse;
            emission = material.Emission;
            transparency = material.Transparency;
            shine = material.Shine;
            specular = material.Specular;
          }else if(mat != null && mat.GetType() == typeof(System.Drawing.Color)){
            color = (System.Drawing.Color) mat;
            shine = 0.5;
          }

          args.Set("id", Component.InstanceGuid.ToString() + "-" + i.ToString() + "_" + n.ToString());
          args.Set("mesh", new Mesh[]{mesh});
          args.Set("diffuse", color);
          args.Set("emission", emission);
          args.Set("specular", specular);
          args.Set("transparency", transparency);
          args.Set("shine", shine);
          Rhino.Runtime.HostUtils.ExecuteNamedCallback("FromGHMesh", args);

          var displayMat = new Rhino.Display.DisplayMaterial(color);
          displayMat.Specular = specular;
          displayMat.Emission = emission;
          displayMat.Transparency = transparency;
          displayMat.Shine = shine;

          var ghmat = new GH_Material(displayMat);
          ghMats.Add(ghmat, path);
        }
      }
    }

    prevDataTreeCounts = tempDataTreeCounts;

    outMesh = meshes;
    outMat = ghMats;
  }

  // <Custom additional code> 
  IGH_Component comp = null;
  DataTree<Mesh> prevMeshes = new DataTree<Mesh>();
  List<int> prevDataTreeCounts = new List<int>();

  public void OnPingDocument(object sender, GH_PingDocumentEventArgs e){
    if(e.Document == null){
      ClearAllMeshes(prevMeshes);
    }
  }

  public void OnObjectChanged(object sender, GH_ObjectChangedEventArgs e){
    if(e.Type == GH_ObjectEventType.Enabled){
      if(comp.Locked){
        ClearAllMeshes(prevMeshes);
      }
    }

    comp.ExpireSolution(true);
  }

  public void OnDocumentChanged(object sender, GH_CanvasDocumentChangedEventArgs e){
    ClearAllMeshes(prevMeshes);
    if(comp != null){
      comp.ExpireSolution(true);
    }
  }

  public void ClearAllMeshes(DataTree<Mesh> meshes){
    int branchCount = Math.Max(meshes.BranchCount, prevDataTreeCounts.Count);
    for(int i = 0; i < branchCount; i++){
      int meshCount = 0;
      int prevCount = 0;
      if(meshes.BranchCount > i){
        meshCount = meshes.Branches[i].Count;
      }
      if(prevDataTreeCounts.Count > i){
        prevCount = prevDataTreeCounts[i];
      }
      int itemCount = Math.Max(meshCount, prevCount);


      for(int n = 0; n < itemCount; n++){
        using(var args = new Rhino.Runtime.NamedParametersEventArgs()){
          args.Set("id", comp.InstanceGuid.ToString() + "-" + i.ToString() + "_" + n.ToString());
          Rhino.Runtime.HostUtils.ExecuteNamedCallback("FromGHClearMesh", args);
        }
      }
    }
  }
  // </Custom additional code> 
}
```

Tutorial Video: https://youtu.be/geC94_pkPrs
