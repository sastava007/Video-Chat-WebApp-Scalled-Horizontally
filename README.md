
# Video Chat WebApp

## Features of the application: 

 - Several available rooms and possibility to jump across them.
 - Realtime user status (online/absent/unavailable)
 - Private chat with anyone else in the room & get notified whether the other user is already talking with someone else or has closed the chat.
 - Be able to start a video conference within the same private chat. 
 - Horizontally scaled the two instances of app server using ***Load Balancer*** and ***Redis Adapter***.

## Technology Stack

 - Backend: NodeJS + Express
 - VueJS as frontend framework
 - Redis as a database in memory
 - Docker & Docker Compose

## Application Architecture
As we are running two diferrent instances of app server, with each server maintaining the data of it's own users locally so in case if two users want to communicate with each other but they are connected with different instances then it's not going to happen. So we need a shared memory which is accessible by both app servers and here come's the Redis into picture. 

<p align="center"> 
<img height="350" src="https://i.imgur.com/IKloc5K.png">
</p>

In case of Docker, we'll be running 3 containers the first two for app servers and third one for Redis. 
<p align="center"> 
<img height="320" src="https://i.imgur.com/75bP04a.png">
</p>




## Interested in knowing how Video Communication works?


I'm using [**_WebRTC_**](https://webrtc.org/) is a free and open project that provides web and mobile applications with Real-Time Communications (RTC) capabilities via simple APIs. WebRTC enables peer to peer communications even tough it still needs a server for the _signaling_ process. *In this case the private chat rooms will be used as the signaling mechanism between the two users*

![enter image description here](https://i.imgur.com/9ybiMpY.png)



## Installation Guide

 - Fork this repository, please star too.
 - Enter the respective folder, and run `npm install`
 - Fix dependency issues, if any using `npm audit fix`
 - Run `npm run serve`

Stuck somewhere, need help? Feel free to write to me at _[sastava007@gmail.com](mailto:sastava007@gmail.com)_
