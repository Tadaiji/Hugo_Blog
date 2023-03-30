---
title: "Random problems"
date: 2023-03-28T21:12:17+02:00
slug: "random_problems"
description: ""
keywords: []
draft: true
tags: []
math: false
toc: false
---
## The course:
The course was given to us with the project and here we should implement our techniques.
{{< figure src="../../images/Parkour_Top.png" title="Top view of the course" >}}
{{< figure src="../../images/Course_side2.png" title="Side view of the course" >}}
This parkour course had patches of coins on the track and the track wasn’t only straight. For my implementation I added some additional coins to the track, mostly on top of buildings. I also added stairs to one building to give the player something to climb.
We also had a testing scene which only had a plane in the beginning. While testing I expanded that. Here I first started testing my implementations and later on the course itself.
{{< figure src="../../images/TestingArea.png" title="My testing area" >}}



## The "Quest Link"
Because I was developing on a laptop at first, I had the problem that the quest link was not working. This is because my laptop has no separate GPU which is needed for the link. But without the link I had the problem that I needed to export a build each time I wanted to test something and because these builds are standalone, I couldn’t observe the changes of the game objects. This led to the problem that if the build had bugs, I could only observe them in-game and couldn’t get information form the unity editor. At the beginning this wasn’t really a problem because I used an in-game debug console and the implementations weren’t that hard and could get fixed but not comfortably. And with future implementations I really needed a live overview of what is happening with my game objects. So, I tried to switch to my pc, but I had trouble getting it to work and couldn’t install new graphic drivers. And these problems of problems ate away at my time. After uninstalling my old GPU drivers I installed the lates and I got it to work. Now next was setting up the Unity editor to work with the quest in the editor. For this I followed a tutorial online and the setup was easy. Now I could debug on the fly, which saved me much time and many headaches. But the connection just broke was and still is very unstable and breaks often. 

## Animations

At the late stages of the development, I let my professor try the game for the first time. While he was playing, he had questions about the controls and what he could do. And there it struck me I had played and developed the game for some time so I knew all controls and what is possible, but when a new player plays the game for the first time they couldn’t know what to do and what is possible. So I made some in-game Animations which are placed before the start, which tell the player what they can do. 
{{< figure src="../../images/tutorial_piller.png" title="The ingame tutorials" >}}

These were simply done by take game-objects which represent what I wanted to show and some text above them.
{{< figure src="../../images/hand_beschoeren.gif" title="Hands (de)spawn" >}}

Then I took the game-objects and the gave them an animator.
{{< figure src="../../images/Animator.png" title="Animator" >}}

This then allowed me to create and animation clip with Unitys inbuild animation editor. 
{{< figure src="../../images/Animation.png" title="Animation ingame editor" >}}

This animation editor can save attributes of my choosing as keyframes and if I change them at a later point on the timeline it sets a new keyframe. Now unity interpolates between these keyframes and makes a smooth movement.
{{< figure src="../../images/animation.gif" title="Animation" >}}

This animation is then saved and can now be controlled from a script or in the unity editor. I controlled the animation from a script. I wrote a little script that start and stops the animations, when the player enters or exits a trigger around it. This is done to save some processing power when the player isn’t around the object.
{{< figure src="../../images/tutorial_piller_collider.png" title="The ingame tutorial with visible collider" >}}
``` c#
{
    private void OnTriggerEnter(Collider other){
        if (other.gameObject.layer == 8)
        {
            if (anim.isPlaying)
            {return; }
            anim.Play();
        }
    }

    private void OnTriggerStay(Collider other){
        if (other.gameObject.layer == 8)
        {
            if (anim.isPlaying)
            { return; }
            anim.Play();
        }
    }

    private void OnTriggerExit(Collider other){
if (!anim.isPlaying)
       		{return;}
       	anim.Stop()
     }
}
```
