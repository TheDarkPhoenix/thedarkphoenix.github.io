---
layout: post
title: Luminous ball
permalink: /projects/luminous-ball
categories:
  - Software
excerpt: |
  Visual motion tracking pointing device for PC
  <center><img width="600" src="/pics/10_luminous_ball/kontroler_compressed.jpg"></center>
  <br>
date: 2017-05-26
order_number: 13
---

Motion controller for PC.

{% include figure.html image="/pics/10_luminous_ball/kontroler.jpg" width="300" height="300" %}

The controller was inspired by the PlayStation Move and consists of a flashlight with a ball. The ball's movements are tracked on a webcam and translated into mouse cursor movements. By turning the flashlight on and off a mouse click is simulated.

To analyze the image and detect the ball I used the OpenCV library and the detection method as in the [https://forbot.pl/blog/opencv-2-wykrywanie-obiektow-id4888](https://forbot.pl/blog/opencv-2-wykrywanie-obiektow-id4888)

{% include video.html id="R8JSr6QHfQk" title="Luminous ball demo" %}

{% include button.html text="Github repository" icon="github" link="https://github.com/macstepien/LightOrb" color="#0366d6" %}
