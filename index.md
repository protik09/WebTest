My wanderings into the world of Mechatronics
============================================

Introduction
------------

As a pure Mechanical engineer I never had much time for electrical systems and
their ilk. It was considered anathema to the very core concept of "mechanical"
engineering. But my roommate at University worked in robotics and he
occasionally needed help when he was swamped. I always wanted to learn how to
solder, so I watched a couple youtube videos and off I went. Got an Arduino
starter kit for my birthday - turned an LED on and off and never looked back; I
was addicted.

An interesting thing I discovered in my musings is that because I was learning
in an unstructured environment, i.e., I was teaching myself, I couldn’t be
trusted to remember everything. Right from outset, I was obsessed about
documenting everything I noticed. I wrote down all my projects in a little lab
book. My internal policy became that I should be able to leave my projects for
years and know exactly what I did and why I did it when I checked my notes. But
then I moved cities, and my book was misplaced in the ensuing chaos. I’ve been
away from this tinkering for over a year now. Thus this is an erstwhile
continuation of my dear departed notes. This is, after all, the digital age.

Recently I was given the task of building a small and simple motor controller
(not one of those advanced PID controllers). The task – Use a potentiometer to
change the speed and direction of a motor, including a microcontroller into the
loop.

 Equipment used
---------------

After some time spent researching on the internet and familiarizing myself with
the best way to accomplish the task, I took stock of the equipment I had. The
traditional way for directional motor control was to use an H-Bridge. I didn’t
have one. But I knew that an H-Bridge effectively amounts to 4 MOSFETS (for a
single channel). As we can see below:

![Figure 1]( <http://www.bristolwatch.com/ele/moshbridge/mosh7.png> “Figure 1”)
