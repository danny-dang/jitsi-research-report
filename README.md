# jitsi-research-report
Research Report Jitsi Project Upwork

## Deployment of Jitisi on AWS EC2

### Requirements

- 1 meeting: 10 people, 720p - 1080p
- Total simultaneous meetings: 10 meetings
- Type of server, number of instances
### Recommendation
The most important specification for a Jitsi server is the network performance.

Network bandwidth that the server handles for 1 meeting of 10 participants:
- In: (3.3 + 0.9 + 0.3) * 10 = 45 Mbit/s
	- one 720p stream: 3.3 Mbit/s
	- one 360p stream: 0.9 Mbit/s
	- one 180p stream: 0.3 Mbit/s
- Out: (3.4 + 9 * 0.3 ) * 10 = 61 Mbit/s
	- one 720p stream
	- nine 180p streams
- Total: 45 + 61 = **106 Mbit/s**

Network bandwidth that the server handles for 10,000 meetings: 106 * 10,000 = 1,060,000 Mbit/s = **1060 Gbit/s**

Reference: [Calculation details](https://docs.google.com/document/d/1iNw-0a8fPM8KvjcCmp_VlumwKrXoLezFCqp_kHcocTM/edit?usp=sharing)

Recommended type of server:
- Name: C5N 18xlarge
- Memory: 192.0 GiB
- CPU : 72 vCPUs
- Network Performance: 100 Gigabit
- Recommend number of instances: 10
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

______________________BELOW ARE RESEARCH IN PROGRESS

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

### Installation
**Quick install:** 
- Pros: more customization
- Cons: a lot of times, more effort require, need a strong devops guide

**Docker** 
- Pros: Easy to install, quicker, work with any OS
- Cons: less customzation have on server
