# Micronesia
### Sub-Kernel Kit for Insider Threat Detection

## Abstract

Bootkits have long been used in an offensive manner by adversaries in order to maintain cold-state persistence. Micronesia is an extended bootkit to allow for self-surveillance upon a host system. The purpose of the kit is to monitor for insider-threat potential on a local machine. At current, resources invested in this problem space for anti-leak/insider-threat detection are primarily invested in exterior-host communications. They rely heavily upon heuristics and detection of anomalous traffic movement. A notable example can be seen in various organizations where sensitive documents in high-side networks are hashed. These hashes are then matched against low-side traffic with hopes of taint marking against data leakage. A knowledgeable adversary however can easily obscure communications so as to be ineffective to such methods. Micronesia is a bootkit solution that allows for not only detection of a potential insider threat, but allows for the compromised system to react with both self-surveillance and deterrence.

## Introduction 

Bootkits are often utilized as a tool for offensive measures. Micronesia, however, is a defensive bootkit solution intended to not only detect insider attempts at booting a live operating system, but also to provide host-level introspection capabilities to identify and track activities within such an environment. The motivation behind the development of the kit was to explore the reintroduction of directed intelligence efforts as a viable solution as opposed to wasteful alternatives such as mass-scale analysis upon many systems

## Insider Threats 

Insider threats aren’t a new phenomenon, they have always existed and will continue to exist as they exploit the human factor. There have been so many instances of insider threats that the FBI has put effort into documenting notable cases and providing guidance on how organizations can attempt to deter them[^1]. An insider exists in a privileged position and is able to and can cause harm to an organization. 

There have been countless studies and materials [^2] [^3] [^4] in the past indicating how damaging insider threats can be for an organization. Whether intentional or unintentional, individuals in privileged positions have the ability to cause an effect upon the integrity of an organization. The SpectorSoft 2014 Insider Threat Survey[^5] states that ~55% of respondents had incidents last longer than a week before discovery. The survey further states that typically a full week elapses before the threat is properly addressed. Note, that the SpectorSoft survey encompasses both unintentional and intentioned incidents. An intentioned insider can be considered an insider that is purposeful in their malice against the organization – this definition contrasts an unintentional insider where data leakage is accidental.

Not all insider threat cases are the same and there will always be incidents that fall out of scope of systems designed to address data leakage. An intentioned threat will come prepared with tools to deceive and hide activities. This is where our focus lies, targeting the toolsets used by an insider threat. A common tool in the arsenal is the utilization of “live operating systems”[^6] [^7] to conceal oneself.
Live Operating Systems can be used to establish environments that are inherently evasive to analysis systems, specifically the type of analyses offered by Data Leakage Prevention (DLP) systems[^8]. 

## Secure Live Operating Systems 

This isn’t our first foray into the Live OS space[^5]. However, the focus now comes from a different vantage point. These bootable systems can be used to mask an insider’s activities and deceive detection systems. They are the type of environments a purposed insider would benefit from once established as doing so provides concealment of activities and eases exfiltration operations.

What these tools offer to an insider is the ability to establish a new working environment that is separate and secluded from the typical operating environment of a system. Such an environment typically entails disallowing manipulation of the base system and thus reduces or negates any artifacts left behind by an insider. Furthermore, the operating system purposefully hosts itself in a volatile medium (random access memory) so as to not persist on shutdown. There are other protection techniques employed, but in essence are all forms and tactics of anti-forensics. Once the environment is established, existing analysis systems are only able to evaluate exterior communications of the host system. 

The problem space can be divided out into different segments that occur chronologically. The most immediate segment would be the actual booting of the Live OS. Employing a preventative policy, one can simply block boots from Universal Serial Bus (USB) devices for instance. However, this is akin to “hot-gluing USB ports”. 

A purposeful insider can and would continue to either enable physical access to the machine or to pick a different target entirely. Preventative policy is not enough to fully dissuade an insider. This is where surveillance methods need to be employed to not only implement activity monitoring capabilities but also determine the order and severity of leakage. The goal is to determine what this insider has with respect to sensitive materials and his/her destinations for such the exfiltration of such materials. Enough evidence gathering would allow for sufficient case-building against the insider.
Venturing forth, our goal is to utilize insight from an offensive mind-set and build capabilities towards to further the defensive side. We work from previous breach incidents and grab inspiration from specific tools and technologies used in aiding a motivated insider threat’s attempts at data exfiltration. 

### Tails

From The Amnesic Incognito Live System (Tails) site:

> Tails is a live OS, that you can start on almost any computer from a DVD, USB stick, > or SD card. It aims at preserving your privacy and anonymity, and helps you to:
> use the Internet anonymously and circumvent censorship; all connections to the Internet are forced to > go through the Tor network; leave no trace on the computer you are using unless you ask it explicitly;
> use state-of-the-art cryptographic tools to encrypt your files, emails and instant messaging.

As a target, Tails was chosen for its notoriety and broad support. We felt that Tails was an appropriate target to build the bootkit against because of its continued endorsements. It helps that Tails is touted as a crucial piece of tradecraft by Edward Snowden[^9]. 

Tails as a bootable media seeks to replace every computation layer except for the sub-kernel space. Thus the solution is to gain and maintain control over the one layer that persists between typical system use and malicious usage of a Live OS. 

The old adage, “the best defense is a good offense”, remains true. In approaching a solution for the problem space, techniques in offensive boot kits were adopted and refashioned. Boot kits have typically been used in the past for malicious means; we often see them used as a platform to maintain control mechanisms in situations where an operating system typically cannot control. Similar system controls are sought and achieved with the Micronesia kit. 

## Micronesia Kit

Micronesia is a combination of different sub-kernel frameworks. Employing coreboot[10] allows us to modularize the payload components. Combining coreboot and SeaBIOS[11]/Tianocore[12] is sufficient to achieve full-boot functionality expected of a system. The Micronesia bootkit itself is a combination of patches to open-source BIOS/UEFI implementations along with a custom payload. 
This allows us to secure control over the sub-kernel and to implement preventative policies. However, as mentioned prior, preventative measures are not a catch-all policy and ultimately in addressing an insider a reactive solution should be deployed. Regardless of what operating system is booted, Micronesia is able to persist and acts as a first-line defense against such maneuvers. 
Secure Boot[13] is a standard that allows for the firmware of the system to check the signature of the boot software. In doing so, a white-list of approved boot software can be created. In the context of the Micronesia Kit, Secure Boot would be relevant if the end-goal was the development of only a preventative system. In environments where preventative policies are “good enough”, secure boot should be enabled and left to cater to those policies. The converse of that is that when performing in defensive posture, there can be cases where it is necessary to disable secure boot in order to entice an insider deeper into a controlled environment. Without secure boot, an insider now faces less restrictions in environments that can be booted into, what this means for the defense is that introspection can then occur which would not have been allowed otherwise.. However, there are environments where there is a need for active intelligence gathering on an insider threat and that would mean relinquishing full preventative capabilities and employing reactive policies in the form of the OS-level kit (Stampede Agent).
Beyond the above, we desire the ability to monitor every facet of the system post-threat determination. Specifically, we want intelligence on activities in the protected Live OS container. In order to garner these abilities, we trampoline an agent from our controlled sub-kernel layer into Live OS layer.
### Stampede Agent – The Hand That Blocks Also Strikes
The Stampede Agent is the component that currently allows for introspection into a Live OS environment. It allows the Micronesia kit to gain the capability of not just blocking an insider, but to strike back with surveillance.
Starting from Micronesia, we are able to trampoline from the sub-kernel space back into the Tails Live OS environment. We bring with us the Stampede agent that allows for discrete full-system monitoring of Live OS space that was previously not possible. This entire phase is known as the trampolining of the stampede agent. Doing so has not only gained us the ability to passively monitor but to interrupt actions performed in the Live OS space.
In summary, the insider’s attempt to boot into Tails will be logged and the action is tracked. Optionally, the stampede kit can be activated once the Live OS has established itself. From there stampede will allow accountability and surveillance upon the insider. From the defense’s perspective this allows for strong case-building against the target because not only has the malice been detected but the exfiltrated materials can be tracked and addressed in real-time.
## Stay Tuned
There are some other interesting things that can be performed when playing in the sub-kernel space. While Micronesia does not typically fit the mold of what we at Exodus engineer on a daily basis it was nonetheless a fun problem space to address. 


## References

Works Cited:

* [^1] http://www.fbi.gov/about-us/investigate/counterintelligence/insider_threat_brochure
* [^2] https://www.defcon.org/images/defcon–22/dc–22-presentations/Schrodinger/DEFCON–22-Tess-Schrodinger-Raxacoricofallapatorius-With-Love-Case-Studies.pdf
* [^3] https://www.defcon.org/images/defcon-17/dc-17-presentations/defcon-17-antonio_rucci-insider_threats.pdf
* [^4] https://www.us-cert.gov/sites/default/files/publications/Combating%20the%20Insider%20Threat_0.pdf
* [^5] http://media.scmagazine.com/documents/90/spectorsoft–2014-insider-threa_22395.pdf
* [^6] http://en.wikipedia.org/wiki/Live_CD 
* [^7] http://en.wikipedia.org/wiki/Live_USB
* [^8] http://en.wikipedia.org/wiki/Data_loss_prevention_software 
* [^8] http://blog.exodusintel.com/2014/07/23/silverbullets_and_fairytails/
* [^9] http://www.wired.com/2014/04/tails/
* [^10] http://www.coreboot.org/Welcome_to_coreboot
* [^11] http://www.coreboot.org/SeaBIOS
* [^12] http://www.coreboot.org/TianoCore
* [^13] https://technet.microsoft.com/en-us/library/hh824987.aspx
* [^14] https://www.exodusintel.com/files/micronesia.pdf

#### Metaadata

Tags: Data Leakage Prevention, Insider Threat, Bootkit, Surveillance, Preventative, Reactive, Counter Technology

**Primary Author Name**: Loc Nguyen  
**Primary Author Affiliation**: Exodus Intelligence  
**Primary Author Email**: loc@exodusintel.com  
