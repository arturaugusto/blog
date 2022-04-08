+++
title = "Robots and Sound Communication"
date = 2022-04-08
category = "Tech"

[taxonomies]
tags = ["rust", "stm32", "embedded", "DSP"]
+++

I always found configuring wireless gadgets very annoying.


Pairing/unpairing/repairing bluetooth things on badly designed user interfaces is not my thing.


But what are the options for simple wireless communication?


A surprisingly non-obvious option is the use of sound waves to create a sort of communication. Surprisingly because that is what we use for talking with each other, but not something we consider to use to interact with an embedded electronic system.


Developing speech recognition is not a simple task. But maybe this is not the correct path to achieve a communication channel with hardware for simple applications. Detecting single tones is a very feasible thing to do though.


A handy algorithm that can be used to detect tones even on memory constrained hardware is the Goertzel Algorithm. Opposite to Fast Fourier Transform, It requires much less memory and CPU cycles to compute a predefined set of frequencies.


Based on those assumptions, I decided to prototype a robot controlled by sound wave tones.

This is what I achieved:

<iframe width="560" height="315" src="https://www.youtube.com/embed/JwpZoJkUezs" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


The embedded Rust source running on the STM32 bluepill microcontroller is available at https://github.com/arturaugusto/audio-drone.


For my robot, I defined four frequencies to be detected. Each frequency is associated with some action, like move forward, backward, turn left or right.


I decided to write my own implementation of Goertzel Algorithm in Rust. It was almost a verbatim translation from [this C implementation](https://netwerkt.wordpress.com/2011/08/25/goertzel-filter/). The source code is available at https://github.com/arturaugusto/rt-goertzel


I also made a simple [user interface](https://github.com/arturaugusto/audio-drone-keypad) using javascript and the Browser audio API to play tones, so I don't need to whistle every time like here:


<iframe width="560" height="315" src="https://www.youtube.com/embed/rYuAQTtDHGI" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


