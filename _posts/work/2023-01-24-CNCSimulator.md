---
title: "CNC simulator"
last_modified_at: 2023-01-25T15:06:02-05:00
thumbnail: assets/images/work/cnc/CNC-01.jpg
classes: wide
toc: true
toc_sticky: true
excerpt: VR simulation of a CNC machine developed for a vocational school’s industrial technician curriculum.
gallery:
  - url: /assets/images/work/cnc/machine-01.png
    image_path: /assets/images/work/cnc/machine-01.png
    title: "CNC machine 01"
  - url: /assets/images/work/cnc/tools-02.png
    image_path: /assets/images/work/cnc/tools-02.png
    title: "Tools 02"
  - url: /assets/images/work/cnc/machine-02.png
    image_path: /assets/images/work/cnc/machine-02.png
    title: "CNC machine 02"
  - url: /assets/images/work/cnc/machine-03.png
    image_path: /assets/images/work/cnc/machine-03.png
    title: "Machine 03"
categories:
  - Work
tags:
  - pielab
  - simulator
  - unity 
---


{% include video id="3xSwFyj6mxo" provider="youtube" %}
{% include video id="0jPMiMo_6sE" provider="youtube" %}

## Description
VR simulation of a CNC machine developed for a vocational school’s industrial technician curriculum. Created in Unity, 3 person development team.

The CNC machine was based on the machine used for teaching at the school, where they would configure tooling, calibrate and run milling programs.

## Responsibilities

- Tutorial system
- Interaction and tools
- Modelling, texturing, shading

## Implementation
### Tutorial system

The tutorial system presents series of tutorial steps to prepare the machine, displayed on a tablet, which the user can drag around with them. 

<a href="{{site.url}}/assets/images/work/cnc/tablet.png" class="image-popup">
	![Untitled]({{site.url}}/assets/images/work/cnc/tablet.png)
</a>

- Core classes
    - ***Tablet*** - an interactable tablet object, composed of a 3D UI canvas for the user to operate the tutorial system, as well as view the tutorial content.
    - ***Manager*** - a tutorial player, responsible for playing through the selected library, as well as any methods needed for a tutorial action or step
    - ***Library*** - main storage class of a tutorial, composed by a list of tutorial chapters.
    - ***Chapter*** - each chapter represents a list of tutorial steps, mainly used for separating tutorial steps, for easier organization.
	<a href="{{site.url}}/assets/images/work/cnc/tutorial-action-methods.png" class="image-popup">
		![1092-devenv_05-02-2023.png]({{site.url}}/assets/images/work/cnc/tutorial-action-methods.png){: .align-right}
	</a>
    - ***Step*** - localized description of a task needed to be done in the tutorial, video content, as well as the individual actions that are highlighted to the user, needed to complete the task. Operates by lifetime events, called by the manager.
    - ***Action*** - individual action needed to be done by the user, highlights points of interest, checks points of interest to evaluate if action is done properly. Operates by lifetime events, called by the manager.


{% capture code_sample %}
```csharp
[Serializable]
public class SocketToolAction : TutorialAction
{
	[SerializeField] private ToolPieceConnectionPoint connectionPoint;

	[SerializeField] private GameObject highlightObject;
	private GameObject highlightInstance;

#region Tool to connect

	[SerializeField] private ConnectableTool tool;
	
	[SerializeField, HideInInspector] private ToolTypeEnum[] toolTypes;
	[SerializeField, HideInInspector] private int toolTypeIdx;

#endregion

	public override void EnableAction(TutorialStep step)
	{
		base.EnableAction(step);
		CheckYoSelf();
	}

	public override void DisableAction()
	{
		base.DisableAction();
	}

	public override void SubscribeAction(TutorialStep parentStep)
	{
		base.SubscribeAction(parentStep);
		connectionPoint.OnObjectConnected += ToolConnected;
	}

	public override void UnsubscribeAction()
	{
		base.UnsubscribeAction();
		connectionPoint.OnObjectConnected -= ToolConnected;
	}

	protected override void CheckYoSelf()
	{
		base.CheckYoSelf();
		if (CheckForValidConnection())
		{
			SetToCompleted();
		}
	}

	public override void StartHighlighting()
	{
		base.StartHighlighting();
		if (tool)
		{
			tool.StartBlinking(blinkingTime);
		}

		if (highlightObject)
		{
			highlightInstance = Instantiate(highlightObject);

			if (highlightInstance)
				connectionPoint.AddHighlight(highlightInstance, true);
		}
	}
	public override void StopHighlighting()
	{
		base.StopHighlighting();

		connectionPoint.RemoveHighlight();
		if (tool)
		{
			tool.StopBlinking();
		}

		if (connectionPoint.CurrentTool)
		{
			connectionPoint.CurrentTool.StopBlinking();
		}
	}

	private void ToolConnected(ConnectableTool connectedTool)
	{
		//If a tool of the wrong type is placed
		if (CheckToolType(connectedTool)) SetToCompleted();
	}

	bool CheckForValidConnection()
	{
		if (!connectionPoint.CurrentTool) return false;

		return CheckToolType(connectionPoint.CurrentTool);
	}

	bool CheckToolType(ConnectableTool tool)
	{
		ToolTypeEnum[] connectedToolTypes = tool.ToolTypes;

		for (int i = 0; i < connectedToolTypes.Length; i++)
		{
			//Check for the required tool type to connect
			if (toolTypes[toolTypeIdx] == connectedToolTypes[i])
				return true;
		}

		return false;
	}
}
```
{% endcapture %}
{% include collapsible title="C# sample" indent-margin="40px" content=code_sample %}


- Runtime checks
    
    
    During testing with students, they would occasionally do a step prior to it being presented in the tutorial. Rather than having them do the same step again or limiting them to only interact with the relevant objects in the tutorial step, a runtime check system was added to the tutorial.
    
    Before a tutorial step would be activated by the manager, the manager would do a quick check if all the flagged as important steps were already done. If they were, the step would be skipped by the tutorial manager and the next step would activate.
	<a href="{{site.url}}/assets/images/work/cnc/tutorial-runtime-checks.png" class="image-popup">
		![Untitled]({{site.url}}/assets/images/work/cnc/tutorial-runtime-checks.png)
	</a>
    
- Editor tooling
    
    Custom editor code to draw playback buttons, status information, quickly add chapters, steps, actions, as well as custom drawers for some tutorial action components
    
    {% include video id="JQPwUFgm49U" provider="youtube" %}
    
    

## Interaction and tools

The VR and interaction system is based on the SteamVR SDK for Unity. The user is able to interact with the objects by hovering, grabbing, as well as allowing them to slot into each other and the machine itself.

The connectable blocks, screws, and hex keys would all derive and extend the functionality from the base interactable script. View the walkthrough video at the beginning of the page to view how these objects work in VR.

<a href="{{site.url}}/assets/images/work/cnc/tools-01.png" class="image-popup">
	![Untitled]({{site.url}}/assets/images/work/cnc/tools-01.png)
</a>

## Modelling, texturing, shading

Most of the models, barring some basic ones, were modelled and textured by me in 3ds Max, Maya and Substance Painter. Since the target platform was VR >90fps, the final mesh had to be very optimized to not cause performance issues.

To texture the meshes, a high poly mesh was created with some loops cuts and subdivisions, taken to Substance Painter and textured. The final meshes got baked textures from the high poly mesh and were exported from Substance Painter in a Unity PBR format. 

After importing to Unity, a simple PBR shader was setup with support for tinting with a custom color for highlighting objects.

{% include gallery id="gallery" layout="half" caption="Model screenshots" %}
