
# Jitsi Research Report for Tailwind Business Solutions
## Table of content
1. [Understanding Jitsi](#understanding-jitsi)
2. [Network bandwidth, AWS Recommendation](#network-bandwidth-aws-recommendation)
3. [Improvement of Jitsi audio quality](#improvement-of-jitsi-audio-quality)
4. [Improvement of Jitsi audio quality](#improvement-of-jitsi-video-quality)
5. [Data security and privacy](#data-security-and-privacy)
6. [Recommended Articles](#recommended-articles)


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
## Network bandwidth, AWS Recommendation

### Requirements

- 1 meeting: 10 people, 720p -1080p
- Total simultaneous meetings: 10 meetings
- Type of server, number of instances
### Network Bandwidth Required Estimation
The most important specification for a Jitsi server (Jisit Video Bridge mainly) is the network performance.

Network bandwidth that the server handles for 1 meeting of 10 participants with **simulcast enabled**:

> **Simulcast enabled**: Each client sends 3 quality videos to the server: best, normal, low.  The server then sends 1 best quality stream of the main speaker, n-1 low quality streams of all other participants to all participants. The server will send the normal quality videos to the user who enable grid view. It can easily be noticed that the quality of the video changes when switching between user videos

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

Auto Scaling will be applied to scale the number of instances depending on the bandwidth and cpu usage.

### Deploying in large scale

1. [Scale Jitsi Meet in the cloud](https://jitsi.org/blog/new-tutorial-video-scaling-jitsi-meet-in-the-cloud/)
	- This tutorial shows the general idea of using HAProxy to load balance Jitsi Meet and Jitsi Video Bridge on the cloud (AWS), and how you should distribute your servers.
2. [How to Load Balance Jitsi Meet](https://www.youtube.com/watch?v=LyGV4uW8km8)
	- This tutorial show how to load balance jitsi meet technically
3. [How Jitsi implement geographical bridge cascading for different regions](https://www.youtube.com/watch?v=32CpoFGKHM8&t=364s)

## Improvement of Jitsi audio quality
### Recommendation

Jitsi Audio Codecs: **Opus**

Increate target average audio bitrate in `config.js`, default to 20000:
```
// Valid values are in the range 6000 to 510000
opusMaxAverageBitrate: 20000
```
Configure these settings in `config.js` to give back original audio
```
disableAP: true, //audio processing
disableAEC: true,  //Acoustic Echo Cancellation
disableNS: true,  //Noise Suppression
disableAGC: true,  //Automatic Gain Control
disableHPF: true,  //Highpass Filter
stereo: true,
```
AP, AEC, NS, AGC, HPF are meant for optimizing video conferencing

The bitrate will improve from ~64 kbps to 80 - 128 kbps

Reference: [https://community.jitsi.org/t/higher-audio-quality/31441/6](https://community.jitsi.org/t/higher-audio-quality/31441/6)


## Improvement of Jitsi video quality

Jitsi video codecs: **VP8** (default) or **H.264**

Change between VP8 or H.264 can be done in `config.js`
```
preferH264: true
```
Change the constraint in the `config.js` will increase the ideal resolution
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
Reference: [https://community.jitsi.org/t/1080-jitsi-server-available-to-test-with/18796/23](https://community.jitsi.org/t/1080-jitsi-server-available-to-test-with/18796/22)

## Data security and privacy
### EE2E for both 1x1 calls and video bridge calls
Jitsi now is developing and testing E2EE

Reference: [https://jitsi.org/blog/e2ee/](https://jitsi.org/blog/e2ee/)

Browser support:
- Chromium Browser (Edge, Chrome, Opera and Brave)
- Safari not supported

Desktop using [Jitsi Electron](https://github.com/jitsi/jitsi-meet-electron) supports:
- Mac
- Windows
- Linux
### Security and Data distribution
The two components which are inevitable for such a large scale system are Reverse Proxy, and a Load Balancer.
1. Reverse Proxy:

> Reverse Proxy mask the real location of the servers clients are accessing. A reverse proxy is a must-have for sites with millions of visitors, because they use many servers. All of the site’s traffic must pass through a reverse proxy and only then access the servers as to not overload them. It also brings two or more servers into the same URL space.
2. Load Balancer:
> A Load Balancer distribute the incoming traffic evenly among the different servers to prevent any single server from becoming overloaded. In the event that a server fails completely, other servers can step up to handle the traffic.

Recommendation:
- Cloudflare:
   - [Cloudflare CDN](https://www.cloudflare.com/cdn/) 
   - [Cloudflare Load Balancing](https://www.cloudflare.com/load-balancing/)
 - HAProxy (Recommended by Jitsi):
     -  [HAProxy](http://www.haproxy.org/)
     - [How Jitsi use HAProxy for Cloud Scaling](https://jitsi.org/blog/new-tutorial-video-scaling-jitsi-meet-in-the-cloud/)
### Secure authentication (multifactor)
Authentication can be implemented so that only authenticated users can create a new conference room. I'm proposing 2 recommendations:
1. Authentication using `mod_auth_custom_http` prosody plugin. 
     - I found an article about someone implemented this plugin for 2-factor authentication using [PrivacyIDEA](https://www.privacyidea.org/): [https://jitsi.github.io/handbook/docs/devops-guide/secure-domain](https://jitsi.github.io/handbook/docs/devops-guide/secure-domain)
     - This mod sends a POST request to your server or authentication  server. Hence, it can be a 2FA if it is using 2FA.
 2. Authentication using `jitsi-token-moderation-plugin` [prosody plugin](https://github.com/nvonahsen/jitsi-token-moderation-plugin) + existing jitsi token feature (Recommended):
     - The user can login into your customer-facing website using any kind of authentication (2FA, etc.), then the website generate a token, this token can be used to login to jitsi meet and join a room.
     - Jitsi's current token feature support for authentication. For example, when open a room, the user needs to have the token, otherwise he will be rejected. However, this feature doesn't support for role-based, which means, anyone log in first will be the moderator, the rest are participants. When the moderator lose connection or log out, the moderator role will be handed to a participant, and the previous mod won't be able to become a mod again when he returns.
     - A moderator will have extra features such as: see the stats, kicks participant, mute participant, etc.
     - `jitsi-token-moderation-plugin` comes into place, this plugin split the role based on token. Anyone who has token of moderator will be the moderator regardless of who join the meeting room first.
     
     - Example:
         - A moderator -> Login as Mod in customer-facing website -> the customer-facing website give him a token `{moderator: true}` -> Join meeting -> he becomes mod of the meeting (he won't lose the moderator role even if he leave the meeting and returns)
         - A participant -> Login as Participant in customer-facing website->the customer-facing website give him a token `{moderator: false}` -> Join Meeting -> participant of the meeting and will never become a moderator
### Recording
[Jibri](https://github.com/jitsi/jibri) can be used to record the meeting. 

Currently, Jitsi has supported saving recorded files to Dropbox where you can easily configure it in `config.js`
```
fileRecordingsEnabled: true,
dropbox: {
    appKey: '<APP_KEY>' // Specify your app key here.
    // A URL to redirect the user to, after authenticating
    // by default uses:
    // 'https://jitsi-meet.example.com/static/oauth.html'
    redirectURI: 'https://jitsi-meet.example.com/subfolder/static/oauth.html'
},
```
You can also specify where you want to store your recorded files such as S3.

[Tutorial on integrate S3 Storage for Jibri](https://www.jibritutorials.com/?p=3662)
## Recommended Articles
1. [Jitsi for swiss higher education - success story](https://community.jitsi.org/t/jitsi-for-swiss-higher-education-success-story/24486)
   - A Swiss company providing video conferencing service.
   - This company uses 32 servers for video bridges, 8VCPUs and 8GB RAM each
   - 1000 conferences a day, up to 40 people
   - Each server handle averge of 4 - 5 conference, 20 participant simultaneously 
   - [Their share about how they deployed their system ](https://community.jitsi.org/t/working-multi-jitsi-meet-multi-videobridge-setup/24704)
2. [How Jitsi optimize their system](https://www.youtube.com/watch?v=SAaa8jYdtx4)

