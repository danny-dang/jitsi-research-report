# Jitsi Research Report for Tailwind Business Solutions
________________
## Understanding Jitsi
**1. Client**
- This is where the user interact with the application, works as user interface connecting user and server.
- Web & Android & iOs: [https://github.com/jitsi/jitsi-meet](https://github.com/jitsi/jitsi-meet)
- Electrton Desktop (Mac & Windows & Linux): [https://github.com/jitsi/jitsi-meet-electron](https://github.com/jitsi/jitsi-meet-electron)

**2. Prosody server**
- [https://prosody.im/](https://prosody.im/)
- XMPP server for Jitsi writen in Lua
- Where all the components of Jitsi connects to and communicate using xmpp protocol

**3. Jicofo**
- [https://github.com/jitsi/jicofo](https://github.com/jitsi/jicofo)
- Manage media sessions between each of the participants and the videobridge.
- Works as the signalling focus
- Manage room including: setup rooms, invite users to room, kick user, mute user ...

**4. Jitsi Video Bridge (JVB)**
- [https://github.com/jitsi/jitsi-videobridge](https://github.com/jitsi/jitsi-videobridge)
- A video conferencing solution supporting WebRTC that allows multiuser video communication.
- Transmit streams between users and server and users.
- Do most of the hard work of Jitsi stacks, need the most resources, especially network bandwidth of the server

**5. Jibri** (Optional)
- [https://github.com/jitsi/jibri](https://github.com/jitsi/jibri)
- Provides services for recording or streaming a Jitsi Meet conference

**6. Jigasi** (Optitonal)
- [https://github.com/jitsi/jigasi](https://github.com/jitsi/jigasi)
- Allows regular SIP clients to join Jitsi Meet conferences hosted by Jitsi Videobridge
- Make call and users can dial in the meeting conference
## Deployment of Jitisi on AWS EC2

### Requirements

- 1 meeting: 10 people, 720p -1080p
- Total simultaneous meetings: 10 meetings
- Type of server, number of instances
### Network Bandwidth Required Estimation
The most important specification for a Jitsi server (Jisit Video Bridge mainly) is the network performance.

Network bandwidth that the server handles for 1 meeting of 10 participants:

***For video conference at 720p:***
- Network fluctuation: 1.2 (sometimes clients will send more or less bandwidth, we will multiply this number to get the ideal bandwidth)
- In: (2.5 + 0.6 + 0.2) * 10 * 1.2 ~= 40 Mbit/s
	- one 720p stream: 2.5 Mbit/s
	- one 360p stream: 0.6 Mbit/s
	- one 180p stream: 0.2 Mbit/s
- Out: (2.5 + 9 * 0.2 ) * 10 * 1.2 ~= 51 Mbit/s
	- one 720p stream
	- nine 180p streams
- Total: 40 + 51 = **91 Mbit/s**

Network bandwidth that the server handles for 10,000 meetings: 91 * 10,000 = 1,060,000 Mbit/s = **910 Gbit/s**

***For video conference at 1080p:***
- Network fluctuation: 1.2 (sometimes clients will send more or less bandwidth, we will multiply this number to get the ideal bandwidth)
- In: (5 + 0.6 + 0.2) * 10 *1.2 ~= 70 Mbit/s
	- one 1080p stream: 5 Mbit/s
	- one 360p stream: 0.6 Mbit/s
	- one 180p stream: 0.2 Mbit/s
- Out: (5 + 9 * 0.2 ) * 10 * 1.2 ~= 82 Mbit/s
	- one 1080p stream
	- nine 180p streams
- Total: 70 + 82 = **152 Mbit/s**

Network bandwidth that the server handles for 10,000 meetings: 152 * 10,000 = 1,520,000 Mbit/s = **1520 Gbit/s**

Reference: [Calculation details](https://docs.google.com/document/d/1iNw-0a8fPM8KvjcCmp_VlumwKrXoLezFCqp_kHcocTM/edit?usp=sharing)

### Recommendation for server scaling
**1. One server for Jitsi Client, Front-end Customer Facing website, Backend Server for Customer Facing**

**2. One database server to store users, system data**
- Recommend: AWS RDS, Mongodb, ....

**3. 10 - 15 maximum instances in AWS Auto Scaling Group for jitsi Video Bridge**

Recommended type of server for best efficent:
- Name: C5N 18xlarge
- Memory: 192.0 GiB
- CPU : 72 vCPUs
- Network Performance: 100 Gbit/s
- Recommended minimum instances: 2
- Recommended maximum instances: 15
- Price: $3317.76 (London)

Recommended type of server for cost saving (but less network stable)
- Name: C5N 2xlarge
- Memory: 21.0 GiB
- CPU : 8 vCPUs
- Network Performance: Up to 25 Gbit/s
- Recommended minimum instances: 20
- Recommended maximum instances: 122
 Price: $368.64 (London)

Elastic Load Balancer will be applied to distribute network bandwidths across instances.
Auto Scaling will be applied to scale the number of instances.

References:
- [https://community.jitsi.org/t/aws-scale-for-big-deployment/22617/24](https://community.jitsi.org/t/aws-scale-for-big-deployment/22617/24)
- [https://jitsi.org/blog/new-tutorial-video-scaling-jitsi-meet-in-the-cloud/](https://jitsi.org/blog/new-tutorial-video-scaling-jitsi-meet-in-the-cloud/)
- [https://community.jitsi.org/t/working-multi-jitsi-meet-multi-videobridge-setup/24704](https://community.jitsi.org/t/working-multi-jitsi-meet-multi-videobridge-setup/24704)
- [https://www.youtube.com/watch?v=LyGV4uW8km8](https://www.youtube.com/watch?v=LyGV4uW8km8)

--> You'll need a very strong DevOps skill for this
## Improvement of Jitsi audio quality
### Recommendation
Configure these settings in config.js
```
disableAP: true, //audio processing
disableAEC: true,  //Acoustic Echo Cancellation
disableNS: true,  //Noise Suppression
disableAGC: true,  //Automatic Gain Control
disableHPF: true,  //Highpass Filter
stereo: true,
```
AP, AEC, NS, AGC, HPF are meant for optimizing video conferencing, disabling these options will bring back the original audio.

The bitrate will improve from ~64 kbps to 80 - 128 kbps

Reference: [https://community.jitsi.org/t/higher-audio-quality/31441/6](https://community.jitsi.org/t/higher-audio-quality/31441/6)


## Improvement of Jitsi video quality
Change the constraint in the `config.js`
```
constraints: {
    video: {
        height: {
            ideal: 1080,
            max: 1080,
            min: 240
        }
    }
}
```
Changing video quality works now, you just need to change in the config.js, reference: [https://community.jitsi.org/t/1080-jitsi-server-available-to-test-with/18796/23](https://community.jitsi.org/t/1080-jitsi-server-available-to-test-with/18796/22)

## Data security and privacy
### EE2E for both 1x1 calls and video bridge calls
Reference: [https://jitsi.org/blog/e2ee/](https://jitsi.org/blog/e2ee/)
Browser support:
- Chromium Browser (Edge, Chrome, Opera and Brave)
- Safari not supported

Desktop using [Jitsi Electron](https://github.com/jitsi/jitsi-meet-electron)
- Mac
- Windows
- Linux

______________________BELOW ARE RESEARCHING IN PROGRESS!

www.mainsite.com
1. Call external api, render video steam right in the mainsite
2. Open a new window, like messenger : www.meet.mainsite.com/roomname&jwt=token
www.meet.mainsite.com
[https://github.com/nvonahsen/jitsi-token-moderation-plugin](https://github.com/nvonahsen/jitsi-token-moderation-plugin)

___
3. Main site: Register as a Moderator -> Login as Mod -> Create room -> token {moderator: true} -> Join meeting -> Mod of the meeting
4. Main site: Register as a Participant -> Login as Participant -> token{moderator: false}-> Join Meeting -> Participant of the meeting

- Web UI: React.js [https://github.com/jitsi/jitsi-meet](https://github.com/jitsi/jitsi-meet)
- Prosody Server:  [https://github.com/nvonahsen/jitsi-token-moderation-plugin]
- Jicofo: 
- Jitsi Video Bridge: 

**Quick install:** 
- Pros: more customization
- Cons: a lot of times, more effort require, need a strong devops guide

**Docker** 
- Pros: Easy to install, quicker, work with any OS
- Cons: less customzation have on server
