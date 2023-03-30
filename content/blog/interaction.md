---
title: "Interaction"
date: 2023-03-31T00:18:03+02:00
slug: ""
description: ""
keywords: []
draft: true
tags: []
math: false
toc: false
---
## The idea
For the interaction I wanted to implement something that uses hand tracking too. I wanted to implement a solution in which you stretch your hand to an object and your hand moves further away from you the longer you stay in this position. For extra clarity while grabbing, your hand should change the size the further it is away from the player. This enables the player to grab objects even when they are further away and still seeing the object clearly. This implementation is a little like the go-go arms with additional scaling, like seeing down in the gif. [Video_Link](https://www.youtube.com/watch?app=desktop&v=WhA8n4IXeoY)
{{< figure src="../../images/gogo.gif" title="GoGo Arms" >}}
The Image shows my original scatch for the interaction technique. 
{{< figure src="../../images/go-rgo..png" title="Original sketch" >}}
I believed some of the challenges could be the implementation of the scaling of the hands and the distinguishment of the gesture to grab and the gesture to move. And if I had implemented climbing, I needed to differentiate between an object and a climbable surface. But this implementation of interaction would allow the user to grab items far away even without moving and to reach something even when you don’t have that much space to play or are sitting.    
For this type of interaction I wanted to remodel the current parkour to better show off the possibilities of movement and interaction. So, I wanted to build some obstacles in different heights and some obstacles that are narrower. I also wanted to make some interactable not reachable so that you need to use the extend arms to interact with them.


## The implementation idea
With the same hand tracking API and Rig I wanted to use for the locomotion, I also wanted to use it to implement the go-go hands. I wanted to enlarge the arms the further they are away from the headset or closer to the ground. This distinction could be made on whether the player is standing or sitting. The enlargement should be proportional to the distance with a reasonable multiplication factor so that the arms won’t get too big or not big enough. This factor could be made adjustable and or set over the course of development. When the player is at the max extension of their arm(s), which probably will be when the controllers are stopping at a certain length and doesn’t have the object already in hand or collided with a wall. And while extending the arms the hands should also be scaled down proportional to the distance they have traveled away from the player.

## Implementation problems
I first thought about how to determine the magnification factor. I wanted to implement it based on the distance of the controllers to the headset or the floor, but this would work because when do I determine the magnification of the arms because it can't be while moving and shouldn’t happen on accident.    
While thinking about how to implement the magnification and writing it down, I thought about why I wanted it in the first place. And that was to enable people to play in tight spaces. So, then I thought about a slider so that the player can choose the distance of the hands from the player. In the end I found a better way, I just made the extended hands freely movable.  
First I needed a way to move the physics hands without actually moving them because this causes them to bounce around because of the physics script. So, I added placeholder objects and rigged the physics script to them. 
{{< figure src="../../images/ExtendedHand1.png" title="Extended Hand 1" >}}
{{< figure src="../../images/ExtendedHand2.png" title="Extended Hand 2" >}}

And with the extended hand script I moved the placeholder object which then moves the physics hands because the placeholder is the target of the physics script. I then figured out how to move the placeholder. This is done by changing the local position of the placeholder according to the direction of the controller thumb stick. If the stick is moved horizontal the local x position of the placeholder is changed. And because of the rotation of the plains and hands if the stick is moved vertically the local z position is changed. The z position changes the distance to the player. And because the player controls the extended arm with the controllers itself a change in the y direction wasn’t needed. 
``` c#
    void Update(){
        var rightTriggerValue = OVRInput.Get(OVRInput.Axis1D.PrimaryIndexTrigger, controller); 
        stickPosition = OVRInput.Get(OVRInput.Axis2D.PrimaryThumbstick, controller);

        if(rightTriggerValue > 0.95f){
            if (stickPosition.x != 0){
                localPositionExtended.x += (stickPosition.x * 0.3f);
                handPlaceholder.transform.localPosition = localPositionExtended;
            }

            if (stickPosition.y != 0){
                localPositionExtended.z += (stickPosition.y * 0.1f);
                handPlaceholder.transform.localPosition = localPositionExtended;
            }
        }
    }
```
Now I needed a way to let the hands appear and disappear. Because if they were there the hole time the player would not have the need for the normal hands and the original movement would be gone. So, I expended the extended hand script I already used for the movement. I tracked when the player made a specific input on either controller, by given the OVRInput function a reference to the used controller. This is done by setting the controller in the editor. From there I set the placeholder and the blue plain active or inactive. 
``` c#
    void Update(){
        if (OVRInput.GetDown(OVRInput.Button.Two, controller)){
            if (!triggerd){
                triggerd = true;       
                handPlaceholder.SetActive(true);
                physicsHand.SetActive(true);
                Debug.Log("Extend Aktiv");
            }
            else{
                physicsHand.SetActive(false);
                handPlaceholder.SetActive(false);
                Debug.Log("Extend Deaktiv");
                triggerd = false;
            }
        }
    }
```
But it wasn’t that easy at the start because every time I used the binded button on the controller the player would be teleported to the beginning of the course and the hands would fling to the player. At first I thought it was a bug with the disappearing of the extended hands. But I didn’t think about a scripted teleportation, because for testing I was always close to the starting flag. So, I thought the teleportation was a byproduct of a bug from the disappearing hands. I searched for a long time and also showed the problem to my professor. In the end I found out that the button was already bound by the locomotion technique script. It was in the update function and at first it was used the OVRInput.Button.Two.
``` c#
    if (OVRInput.Get(OVRInput.Button.PrimaryThumbstick) || OVRInput.Get(OVRInput.Button.SecondaryThumbstick)){
        {
            if (parkourCounter.parkourStart)
            {
                this.transform.position = parkourCounter.currentRespawnPos;
            }
        }
    }
```
The Grab was a given script by the professor, but I had to modify it to fit my needs. The script takes the object that is in the trigger and parents it to the hand. Then a bool is set true to mark the object as selected. If the trigger is then pressed again the object is released to the world again. 
```c#
    void Update()
    {
        triggerValue = OVRInput.Get(OVRInput.Axis1D.PrimaryHandTrigger, controller);

        if (isInCollider || isSelected)
        {
            if (!isSelected && triggerValue > 0.95f)
            {
                isSelected = true;
                prevParent = selectedObj.transform.parent;
                selectedObj.transform.parent.transform.SetParent(transform);
            }
            else if (isSelected && triggerValue < 0.95f)
            {
                isSelected = false;
                selectedObj.transform.parent.transform.parent = null;
                
            }
        }
    }
```


## The implementation

#### GoGo extended arms
At the end I didn’t measure the distance and just set them the right distance in the editor to start with and then modify the localPosition in my Script to move them according to the controller thumb stick movement. These extended hands are child objects of the physical hands, so that they stay with the controllers. I gave the player no max distance with the extended hands and didn’t scale them, because there are no small objects in this “game”. I also gave the player the ability to spawn and despawn the the extended hands. Here only the hand which presses the button is affected.
{{< figure src="../../images/extended_hands_and_spawn.gif" title="Extended Arms" >}}

``` c#
    [SerializeField] private GameObject handPlaceholder;
    [SerializeField] private GameObject physicsHand;
    [SerializeField] private OVRInput.Controller controller;

    public Vector3 localPositionExtended;
    public bool triggerd = false;
    public Vector2 stickPosition;
    
    void Start(){
        handPlaceholder.SetActive(false);
        physicsHand.SetActive(false);
        localPositionExtended = handPlaceholder.transform.localPosition;
    }

    void Update(){
        var rightTriggerValue = OVRInput.Get(OVRInput.Axis1D.PrimaryIndexTrigger, controller); 
        stickPosition = OVRInput.Get(OVRInput.Axis2D.PrimaryThumbstick, controller);

        if(rightTriggerValue > 0.95f){
            if (stickPosition.x != 0){
                localPositionExtended.x += (stickPosition.x * 0.3f);
                handPlaceholder.transform.localPosition = localPositionExtended;
            }

            if (stickPosition.y != 0){
                localPositionExtended.z += (stickPosition.y * 0.1f);
                handPlaceholder.transform.localPosition = localPositionExtended;
            }
        }

        if (OVRInput.GetDown(OVRInput.Button.Two, controller)){
            if (!triggerd){
                triggerd = true;       
                handPlaceholder.SetActive(true);
                physicsHand.SetActive(true);
                Debug.Log("Extend Aktiv");
            }
            else{
                physicsHand.SetActive(false);
                handPlaceholder.SetActive(false);
                Debug.Log("Extend Deaktiv");
                triggerd = false;
            }
        }
    }
```

#### Grab
As opposed to the beginning where I wanted to let the extended hands grab the objective, the main hand uses the given grip script to move the objective now. This gives more control to the user to set it in the right way. Also creates another challenge where the user needs to stand on their extended hand to reach the interactable.
{{< figure src="../../images/grab.gif" title="Grab gif">}}

``` c#
    public OVRInput.Controller controller;
    private float triggerValue;
    private bool isInCollider;
    private bool isSelected = false;
    private GameObject selectedObj;
    private Transform prevParent;
    public SelectionTaskMeasure selectionTaskMeasure;

    void Update(){
        triggerValue = OVRInput.Get(OVRInput.Axis1D.PrimaryHandTrigger, controller);

        if (isInCollider || isSelected){
            if (!isSelected && triggerValue > 0.95f){
                isSelected = true;
                prevParent = selectedObj.transform.parent;
                selectedObj.transform.parent.transform.SetParent(transform);
            }
            else if (isSelected && triggerValue < 0.95f){
                isSelected = false;
                selectedObj.transform.parent.transform.parent = null;
            }
        }
    }

    void OnTriggerEnter(Collider other){
        if (other.gameObject.CompareTag("objectT")){
            if(!isSelected){
                Debug.Log("is Colliding");
                isInCollider = true;
                selectedObj = other.gameObject;
            }
        }
        else if (other.gameObject.CompareTag("selectionTaskStart")){
            if (!selectionTaskMeasure.isCountdown){
                selectionTaskMeasure.isTaskStart = true;
                selectionTaskMeasure.StartOneTask();
            }
        }
        else if (other.gameObject.CompareTag("done")){
            selectionTaskMeasure.isTaskStart = false;
            selectionTaskMeasure.EndOneTask();
        }
    }

    private void OnTriggerStay(Collider other){
        if (other.gameObject.CompareTag("objectT")){
            if(!isSelected){
                isInCollider = true;
                selectedObj = other.gameObject;
            }
        }
    }

    void OnTriggerExit(Collider other){
        if (other.gameObject.CompareTag("objectT")){
            isInCollider = false;
            selectedObj = null;
        }
    }
```
