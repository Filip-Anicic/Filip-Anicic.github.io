---
title: "VR adventure game"
last_modified_at: 2023-01-25T15:06:02-05:00
thumbnail: assets\images\work\sde\SDE-thumbnail.png
classes: wide
toc: true
toc_sticky: true
excerpt: VR simulation of a CNC machine developed for Learnmark for a vocational school’s industrial technician curriculum
categories:
  - Work
tags:
  - pielab
  - simulator
  - unity 
---


## Description
VR simulation of a CNC machine developed for Learnmark for a vocational school’s industrial technician curriculum

Created in Unreal Engine 5, main developer role, 3D modelling and texturing done by a contractor, 20 full days of development.

The main working station was based on the same hydraulic station at the school, where they would do the assemblies and experiments of hydraulic circuits.

## Responsibilities

- Project management
- Hydraulic simulation
- Interaction & Locomotion
    - Grabbing
    - Drawing hoses
- Level & Environment design

## Implementation
### Hydraulic simulation

The simulator is based on a simple graph structure and a breath first search analysis and graph construction. The core of the system is written in C++ plugin

- Core classes
    - ***Hydraulic Subsystem*** - Unreal subsystem that constructs networks and updates individual devices with it’s own update loop.
    - ***Network*** - simple class that represents a hydraulic circuit network, stores all ports in the network, open ports, ports that stop at a device.
    - ***Device*** - Actor class that represents a device with ports in the hydraulic circuit, updated by the subsystem.
    - ***Port component*** - component that makes up the hydraulic circuit network, unique ID, contains it’s neighboring connections.
    - ***Connections*** - structs of non-directional and uni-directional references to connected ports.


{% capture code_sample %}
```cpp
void USDESubsystem::ConstructNetworks()
{
	//Clear Networks
	Networks.Empty();

	//Mark all ports dirty
	for(const auto& PortComponent:PortComponents)
	{
		PortComponent->PortData.bIsDirty = true;
	}

	TArray<FGuid> Traversed;
	TArray<FGuid> Visited;
	TArray<FSDEUniConnection> DeadEnds;

	//BFS for networks
	for (int i = 0; i < OriginPortComponents.Num(); ++i)
	{
		//check if dirty
		if (!Visited.Contains(OriginPortComponents[i]->PortID))
		{
			TQueue<FSDEUniConnection> Queue;
			Visited.Add(OriginPortComponents[i]->PortID);
			OriginPortComponents[i]->PortData.bIsDirty = false;
			TArray<USDEPortComponent*> ConnectedPorts;
			//ConnectedPorts.Add(PortComponent);
			Queue.Enqueue(OriginPortComponents[i]->PortID);
			while(!Queue.IsEmpty())
			{
				FSDEUniConnection UniConnection = *Queue.Peek();
				Queue.Pop();
				Traversed.Push(UniConnection.PortID);

				TArray<FSDEUniConnection> PortConnections = PortMap[UniConnection.PortID].connections;
				bool deadEnd = true;
				for(const auto uniConnection: PortConnections)
				{
					
					if(!Visited.Contains(uniConnection.PortID))
					{
						deadEnd = false;
						Visited.Add(uniConnection.PortID);
						Queue.Enqueue(uniConnection.PortID);
					}
				}
				if(deadEnd)
					DeadEnds.Add(UniConnection);				
			}
		}
		

		USDENetwork* Network = NewObject<USDENetwork>();
		for (auto PortID : Traversed)
		{
			USDEPortComponent* PortComponent = PortMap[PortID].component;
			if(PortComponent)
			{
				Network->PortComponents.Add(PortComponent);
				PortComponent->Network = Network;
				PortComponent->PortData.bIsDirty = false;
			}
		}
		for (auto UniConnection : DeadEnds)
		{
			USDEPortComponent* PortComponent = PortMap[UniConnection.PortID].component;
			if(PortComponent)
			{
				Network->PortComponents.Add(PortComponent);
				PortComponent->Network = Network;
				PortComponent->PortData.bIsDirty = false;

				//If connection goes from external to a Device plug, we stopped at a device that is blocking the path
				if(UniConnection.ConnectionType == ExternalConnection && PortComponent->PortData.PortType == Device)
					Network->DeviceEnds.Add(PortComponent);
				//Else we are at a dead end
				else
					Network->DeadEnds.Add(PortComponent);
			}
		}
		Networks.Add(Network);
	}

	OnNetworksReconstruct.Broadcast();
}
```
{% endcapture %}
{% include collapsible title="C++ sample" indent-margin="40px" content=code_sample %}


- Device implementation 

    {% include gallery id="gallery-devices" layout="half" caption="Directional valve and Piston blueprint implementation." %}
    
- Debug tooling
    
    Debug tooling and editor scripts were made to accelerate building and testing the simulator codebase.
    
	{% include video id="0c2YT3OLWrI" provider="youtube" %}
    
    

### Interaction

The interaction system is an extension of the Unreal VR template functionality, support hovering, drawing hoses, gameplay tags, setting active state, etc.

Controller trigger pull interacts with the held component, or a interactable object nearby, else it just notifies any of the VR pawn’s components if they are interested in a trigger pull.

<a href="{{site.url}}/assets/images/work/sde/Interact.png" class="image-popup">
	![Interact]({{site.url}}/assets/images/work/sde/Interact.png)
</a>

#### Drawing hoses

The hose drawing component works based on the VR pawn’s interaction events, handling starting drawing, adding points, finishing drawing and cancelling drawing.

<a href="{{site.url}}/assets/images/work/sde/HoseDrawComponent.png" class="image-popup">
	![HoseDrawComponent]({{site.url}}/assets/images/work/sde/HoseDrawComponent.png)
</a>

### Level & Environment design

<a href="{{site.url}}/assets/images/work/sde/SDE-comparison.jpg" class="image-popup">
	![Left: hydraulic station from the school. Right: digital replica of the hydraulic station]({{site.url}}/assets/images/work/sde/SDE-comparison.jpg){: .align-center}
</a>
<p>Left: hydraulic station from the school. Right: digital replica of the hydraulic station</p>{: .align-center}

{% include video id="p3ffdVJpBRo" provider="youtube" %}


The table and hydraulic component modelling was done by a hired contractor, while the environment was prepared by myself, with Marketplace assets. 

The target platform was Meta/Oculus Quest 2, but because of the tight time constraints and shader issues, the final build was on Windows.