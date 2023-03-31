---
title: "Locomotion"
date: 2023-03-31T00:17:31+02:00
slug: ""
description: ""
keywords: []
draft: false
tags: []
math: true
toc: true
---
## The idea
So, for the locomotion technique I thought that the player is using their arms/hands to move, while the further you go down with your hands/controllers the larger your arms get in game. I imagined it something like Hand Walking which can be seen down below.
{{< figure src="../../images/30.gif" title="Hand walking" >}}

These arms are then used to physics based propel/push the player in the desired direction based. And the motion of your hands should be translated into game movement. This motion is doable in many positions and doesn’t need too much space if the enlargement of the arms is factored in. And if I am fast at implementing this locomotion system, I wanted to think about implementing a form of climbing, because it could enhance the playing experience.    
I wanted to use the hand tracking to control the hands. I also wanted to create a copy of the virtual hand and attach it to an arm that I wanted to rig using inverse kinematics. The asset used for this I wanted to find online or in the unity store. From here I will use Unity physics to eventually be able to push myself off the ground and get a motion out of it.     
For the physics-based locomotion I wanted to use an already established solution from the game “Gorilla Tag”. The game has an open git repository and many videos online of how to implement it. My only concern was to get it to work with the hand tracking. 
The gorilla tag implementation uses some different math equations that I am not too familiar with now. For example, Hooke's Law which roughly states that “within certain limits, the force required to stretch an elastic object such as a metal spring is directly proportional to the extension of the spring”. As far as I understand this will help me with the proportion for my locomotion system. 

## Implementation problems
First, I followed and implemented the video from Justin P Barnett of the livestream where he implemented the movement of “gorilla tag” [Video](https://www.youtube.com/watch?v=KxmMSHP3Mbk). It was easy to follow and had it implemented really fast. The first time playing with everything implemented was very much fun. I was jumping around, playing with my demo cube and just messing around. So, for my first play test this was pretty good, but it was just a copy of something already working so it wasn't my achievement but I have/had fun. This implementation was made with the XR Rig and the controllers. The XR Rig is part of the [Unity XR Framework](https://docs.unity3d.com/Manual/XRPluginArchitecture.html) which was added in Unity 2019.3 and newer versions.
{{< figure src="../../images/first_time_playing.gif" title="My first time playing with the gorilla tag movement" >}}

Because the XR Rig isn’t supporting hand tracking I needed to switch to the OVR Framework. It is part of the official oculus integration plugin for unity. I used that because we all had a quest 2 lend to us for the time of the course. First I imported the plugin then imported the OVR rig preset, set the right settings in the OVR Rig. With this I had working hand tracking. 
{{< figure src="../../images/OVRRig1.png" title="The OVR rig in the editor" class="project_image">}}
{{< figure src="../../images/OVRSettings.png" title="The settings of the OVR rig" class="project_image">}}

Next, I tried to combine the hand tracking with the physics script. This offered the challenge of where I get the collider to connect with the physics script. First, I tried the generated physics capsules from the OVR rig but I couldn’t grab them from the physics script. There were too many generated colliders and they weren’t addressable from the editor, because they are generated on game start. But I needed predictable and addressable collider for the physics script. So, for the start I used the same plains/rectangles as in the tutorial and attached the physics script to them. 
{{< figure src="../../images/Blue_Plains2.png" title="The blue plains" >}}

While testing I discovered that hand tracking has its problems. I had the hand disappear when it was not fully visible or that the hand would disappear while it was moving fast and reappearing when the hand slowed down.     
The hand tracking can’t handle the fast movements for the movement technique. This problem emerged at the time I had the movement already implemented and I never thought about that till then. With this some of my starting motivation couldn’t be applied anymore. And so the movement doesn’t work properly, because the hands disappear when they move too fast and this causes my locomotion not to work. Sometimes the plains would reset to the headset and sometimes it just wouldn’t collide properly. 
{{< figure src="../../images/Glitching_hands.gif" title="An example of the problems with the hand tracking" >}}
{{< figure src="../../images/BluePlainEditor.png" title="One blue plain in the editor" class="project_image">}}

To mediate the problem with the hand tracking I had a couple of choices. I could have abandoned the hand-tracking and used the controllers. This would contradict some of my motivation for the project, but the locomotion would work and could be used. I also could have changed the whole controlling schema and used something like finger controllers. This would have solved my problem with the arm enlargement and wouldn’t use the controllers. But the problem is that later with the extended arm the controls could have further problems. 
So to have it ready for the first deadline I changed the controls from hand tracking to the controllers. With this change most problems with the movement disappeared. Then I rigged the plains to the controllers. For this I needed a ridgidbody and a collider on the controller. This is then dragged into the target in the editor. 
{{< figure src="../../images/physics_controlls.png" title="The physics script in the editor" >}}

For the players comfort I rotated the plains 90 degrees so that the plains are flat if the controllers are upright. I also gave the OVR rig the locomotion technique script to take advantage of the already implemented controls. But because it was not fully what I wanted, I modified it and gave the player smooth turning and disabled/remapped some already given inputs. The additional Inputs can be seen in the [Random Problems](../random_problems/#animations) article.
{{< figure src="../../images/turning.gif" title="Head turning with the thumbstick" >}}

From there I started testing and fixing the script on the parkour course itself. First I needed colliders for the hole course, so that the physics hands could interact/collide with the world. For this I used the unity mesh collider and gave every object in the entity “Environment” a collider. This gave a good cover of the course and so I didn’t need to model them myself.  

## The implementation
For the movement and the rotation of the plains I used PID controllers. These controllers are a type of control loop that is used for automation. They are flexible because they can handle target values and changing external conditions. It can be configerd to respond to changes in different ways. For example, one system might respond with fast, snappy movements, while another uses slow and gentle movements.

This typ of controller can be used when more precision is needed. The basic idea is to calculate a more precise value to control the system. It uses the feedback from the system to calculate the next input. The P stands for proportional, the I for integral and the D for derivative. At the end they are added together to make the input, which can be sent to a specific output.        

Proportional is the main term and the simplest. Here the error is simply multiplied by the gain. The D term acts as a dampener, while the P term acts as a spring. For the derivative a damping force is applied and the rate of change (derivative) of the error value is calculated. The error rate of change simply differs from the system's velocity. The error remains unchanged when the system is stationary. The error changes by -5 m/s when the system moves at +5 m/s. So the D term provides a braking force that intensifies with system speed. And the integral term I adds up the error over time and integrates it. The I term becomes stronger the longer an error exists. The integration is used to eliminate the steady state error. This can occur when the target value is constantly changing or there is a constant external force acting on the system, such as gravity. [Quelle: PID](https://vazgriz.com/621/pid-controllers/).

So simply the PID controller applies acceleration force to the rigidbody to get it to go to a specific point. And it makes the motion smooth by going faster the more it is away from the target or slower if you are close.    
For the movement of the plain the same principles are used as described before. Then these values are smoothed over time and from there the force is calculated. This force is then added as acceleration force to the rigidbody.
```c#
    void PIDMovement()
    {
        //target = to follow 
        
        //p
        float kp = (6f * frequency) * (6f * frequency) * 0.25f;
        //d
        float kd = 4.5f * frequency * damping;
        
        float g = 1 / (1 + kd * Time.fixedDeltaTime + kp * Time.fixedDeltaTime * Time.fixedDeltaTime);
        float ksg = kp * g;
        float kdg = (kd + kp * Time.fixedDeltaTime) * g;
        
        //calculate how much force/acceleration to add to physics hand
        Vector3 force = (target.position - transform.position) * ksg +
                        (playerRigidbody.velocity - _rigidbody.velocity) * kdg;
        
        _rigidbody.AddForce(force, ForceMode.Acceleration);
    }
```

For the rotation the same formula is used like for the force of the movement. From there the torque is calculated, for this the angle of rotation is needed. This angle comes from a quaternien with the rotation direction. 
```c#
    void PIDRotation()
    {
        //p
        float kp = (6f * rotFrequency) * (6f * rotFrequency) * 0.25f;
        //d
        float kd = 4.5f * rotFrequency * rotDamping;
        
        float g = 1 / (1 + kd * Time.fixedDeltaTime + kp * Time.fixedDeltaTime * Time.fixedDeltaTime);
        float ksg = kp * g;
        float kdg = (kd + kp * Time.fixedDeltaTime) * g;
        Quaternion q = target.rotation * Quaternion.Inverse(transform.rotation);

        if (q.w < 0)
        {
            q.x = -q.x;
            q.y = -q.y;
            q.z = -q.z;
            q.w = -q.w;
        }
        else
        {
            //q.x = q.x + 90 ;
        }
        q.ToAngleAxis(out float angle, out Vector3 axis);
        axis.Normalize();
        axis *= Mathf.Deg2Rad;
        Vector3 torque = ksg * axis * angle + -_rigidbody.angularVelocity * kdg;
        
        _rigidbody.AddTorque(torque, ForceMode.Acceleration);
    }
```

Hook's Law is also used which is the law of elasticity. It states that, for relatively small deformations of an object, the displacement or size of the deformation is directly proportional to the deforming force or load. Inside specific restricts, the power expected to extend a flexible item, for example, a metal spring is straightforwardly relative to the expansion of the spring. Hooks Law is the term for this. [Quelle: Hooks Law](https://www.khanacademy.org/science/physics/work-and-energy/hookes-law/a/what-is-hookes-law) [Quelle2: Hooks Law](https://www.britannica.com/science/Hookes-law)

Simply Hooks-law is used for the propulsion of the player, this formula describes the movement of a spring like it stretches and contracts. It come into action when the physics hand is colliding, so every time the player “hits” a surface. 
```c#
    void HookesLaw()
    {
        //How compressed is the spring
        Vector3 displacementFromResting = transform.position - target.position;
        Vector3 force = displacementFromResting * climbForce;
        float drag = GetDrag();
        
        playerRigidbody.AddForce(force, ForceMode.Acceleration);
        playerRigidbody.AddForce(drag * -playerRigidbody.velocity * climbDrag, ForceMode.Acceleration);
    }

    float GetDrag()
    {
        Vector3 handVelocity = (target.localPosition - _previousPosition) / Time.fixedDeltaTime;
         //Inverse, add small number to avoid devide by 0, the faster we move the less drag and the slower more drag
        float drag = 1 / (handVelocity.magnitude + 0.01f);
        drag = Math.Clamp(drag, 0.03f, 1f);

        _previousPosition = transform.position;
        
        return drag;
    }
```
These functions are then called in the Fixed Update function but hooks law is only called when the attached object is colliding. The Fixed Update is used because it is used for every physics interaction in Unity. It allows the physic manipulations to take place at a fixed rate, as opposed to the normal update which is called every frame. 
```c#
    void FixedUpdate()
    {
        PIDMovement();
        PIDRotation();
        if(_isColliding)
        {
            HookesLaw();
        }
    }
```

In the editor the blue planes are set on the controller models and are given the physics script. Also the physics script needs Collider so 2 box colliders are given. On collider is set as a trigger, to trigger events on the course. The other is the collider to move the player, this one really collides with the mash colliders of the environment. 
{{< figure src="../../images/BluePlainEditor.png" title="The physics hand in the editor" class="project_image">}}

The seen hand models in the more current versions is the oculus hand model. This has animations already which I also used. The hand model is set as a child object of the blue plain and the position is set so that the plain is set over the palm of model. To let it now look like the hand is colliding I disabled the mesh renderer of the blue plains, which makes them “invisible”. Now it looks like the hand is colliding even when I have no colliders for the hand itself.  
{{< figure src="../../images/Hand_with_Plain.png" title="The physics hand in the scene" >}}




