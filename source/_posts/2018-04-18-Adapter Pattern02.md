---
layout: post
title:  "Adapter Pattern02"
date:   2018-04-18
categories: Design Pattern
---

- 설명 
  - 기존에 MediaPlayer 인터페이스 : mp3 포맷만 play
  - 새로운 AdvancedMediaPlayer 인터페이스 :  vlc, mp4 포맷 play
  - AudioPlayer로 mp3 뿐만 아니라, vlc, mp4 도 play 하고자 한다
- Class Diagram

![](/image/myAda.png)

- Code

### Interface : Media Player and Advanced Media Player

![](/image/ad03.png)

![](/image/ad04.png)

### concrete classes 

### implementing the *AdvancedMediaPlayer* interface.

![](/image/ad05.png)

![](/image/ad06.png)

### adapter class 

### implementing the *MediaPlayer* interface.

![](/image/ad07.png)

### concrete class 

### implementing the *MediaPlayer* interface.

![](/image/ad08.png)

### Use the AudioPlayer  

### to play different types of audio formats. 

![](/image/ad09.png)

### Result

![](/image/ad02.png)

- Reference
  - <https://www.tutorialspoint.com/design_pattern/adapter_pattern.html>

