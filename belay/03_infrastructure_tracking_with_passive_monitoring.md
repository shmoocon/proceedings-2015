# Infrastructure Tracking with Passive Monitoring and Active Probing

## Abstract

Threat intelligence is crucial in our industry to proactively monitor for attacks, detect active breaches, and analyze incidents post-mortem. Intelligence is created by researching, tracking, and interpreting attacker movements with a focus on preemptively countering malicious campaigns as soon as they emerge. In this talk, we will describe tools and methodologies we use in-house to provide context on evil at Internet scale. We will also present concrete use cases on how to leverage threat intelligence, both open source and proprietary, to track internet threats and pivot around specific indicators to further the investigative effort. Our use case of choice will be the new Zeus GameOver variant that re-emerged last summer and which we've been tracking for several months. The various aspects of campaign tracking include command and control infrastructure, preferred hosting providers, domain registration practices, and compromised client behaviors.

## Introduction

Threat intelligence is crucial in our industry to proactively monitor for attacks, detect active breaches, and analyze incidents post-mortem. Intelligence is derived by researching, tracking, and interpreting attacker movements and ideally focuses on preemptively countering malicious campaigns before they emerge. 

This paper will describe tools and methodologies we use in-house to provide context on evil at Internet scale. Also discussed are concrete use cases on how to leverage threat intelligence, both open source and proprietary, to track internet threats and pivot around specific indicators to further the investigative effort. The use cases will cover the Zbot proxy network as well as the new Zeus GameOver variant that re-emerged in the summer of 2014 and which OpenDNS has been tracking for several months. The various aspects of campaign tracking include command and control infrastructure, preferred hosting providers, domain registration patterns, and compromised client behaviors.

## Network Intelligence

Intelligence used to keep computers secure from potential attacks, stop ongoing attacks, and investigate detected attacks comes in two distinct types: host based and network based. Intelligence should be timely, ingestible, and actionable information about threats or situations; it assists in taking action (strategic, operational, or tactical). In computer security it is often based on artifacts or indicators known to be related to threats.

Examples of host based indicators include but aren’t limited to: registry keys, process mutexes, service names, directory and file paths, and file attributes such as SHA256 hash and compile time. Network based indicators include but aren’t limited to: domains, IP addresses, URLs, HTTP user-agents, BGP prefixes, ASNs and whois information about IPs and domains. Two distinct methods of analyzing raw data for deriving intelligence are active probing and passive monitoring.  Each of these methods has their own strengths, weaknesses, and applicable scenarios.

## Active Probing

Active probing is defined as identifying the current state of an entity. For example, identifying what IP address a domain resolves to, what ASN an IP address belongs to, or what name servers a domain is currently using.  Active probing can be broken down into two sub types, direct and indirect.

#### Direct

Direct active probing involves communicating with the system or artifact being investigated. Some examples of this are port scanning a system to see what services it offers, analyzing a system’s service banners, and collecting content from the service (e.g. HTTP or FTP) on a system. Direct active probing is considered “noisy” as the communications can be logged by the system being investigated. Attacker controlled systems are able to block the source IP addresses direct active probing originates from. Returning malicious or misleading information is also the prerogative of attacker controlled systems.

#### Indirect

Indirect active probing involves communicating with authoritative third party systems. These third party systems are typically required for proper Internet operation and as such are generally trustworthy. Examples of third party systems include the DNS (for resource records associated with a domain and a domain’s authoritative name servers), IP and Domain Whois servers (for IP and domain ownership details as well as domain creation times), and BGP looking glass routers (for relationships between operators of netblocks).

## Passive Monitoring 

Besides active probing, passive monitoring is another method for gathering network intelligence. In the context of this paper, passive monitoring is defined as the previous state of things or patterns derived from previously observed behaviors. For example: Which other domains have the same registrant email address as this one? At which hour in the day does a domain receive the most DNS queries? Have we ever seen any DNS queries for any subdomains of this domain? Which IP has a domain been previously resolving to? These are all questions passive monitoring can assist in answering. Some sources of passive monitoring data include historic whois information, passive DNS reconstruction, logged system interactions, previously seen malware analyses, and inbound traffic to sinkholed domains.

## Zbot Case Study

In this case study, we covered the Zbot fast flux proxy network, a “hosting as a service” infrastructure for malware CnCs that has been under our watch for over a year. We covered this proxy network at BlackHat, DefCon, and BotConf, and for ShmooCon, we released new results regarding abused registrars and registrant email addresses used to register the malware domains.

Despite having been around for at least a couple of years, this malware hosting infrastructure is still alive and actively used by a variety of malware families. The last to date is TinyBanker (Tinba), the lightweight banking trojan, which has been observed using the Zbot fast flux infrastructure to host its CnC domains since approximately November of 2014. One of many notable facts about our studies is that we saw common patterns between the Zbot proxy network and the newGOZ infrastructure, both at the DNS and IP level. For instance, TodayNIC and Melbourne IT (two registrars from China and Australia, respectively), are among the top abused registrars for both Zbot and newGOZ CnC domains, which is quite suggestive of shared TTPs within the same or between bad actor groups.

Furthermore, the newGOZ CnC domains had been hosted for a short period of time on the Zbot proxy network when it re-emerged in July of 2014. This could have been an early testing phase or a quick bootstrap platform before newGOZ moved to more dedicated setups, as our talk shows.

## New Zeus Gameover Case Study

Domain generation algorithms (DGAs) have been used by malware since the late 2000’s. Popularized by Conficker variants in 2008, a domain generation algorithm can be either letter or word based. Typically, a letter based DGA randomly generates a series of letters (or numbers) and appends a TLD to the letters to generate a domain name. Word based algorithms are similar but make use of a dictionary instead of a list of characters. Often times, a malware’s DGA will generate hundreds or thousands of domains per periodwhich are time durations where an algorithm will change from one set of domains to another. Most DGAs are seeded on the date or time. DGAs can be thought of as a shared secret between malware and a botnet’s controlling party (bot master). Each period, the controlling party needs to only register a few of the domains in that period’s set. This leads to thousands of potential command and control domains (and DNS queries issued by compromised systems) with only a few domains resolving, or being “live”.

The new Zeus Gameover, dubbed newGOZ by researchers, emerged in the summer of 2014 immediately after a coordinated effort to take down its predecessor (oldGOZ). newGOZ uses a letter based DGA with a period of 1 day. It also employed the use of salts in the algorithm. Two known salts have been identified from malware samples. Combined, the two salts totaled 11,000 possible command and control domains per day. We tracked the newGOZ command and control infrastructure for a series of 62 days, from October 7, 2014 through December 7, 2014. After collecting and analyzing data about the infrastructure, some interesting results were discovered.

Out of the 11,000 possible command and control domains generated in the 62 day study, only 251 domains resolved to IP addresses. Both researchers studying the botnet and malicious controllers of the botnet are included in this number. Of the 251 domains which resolved, patterns appeared within the TTL of the domain name. As the administrators of a domain’s authoritative name server set this TTL value, it can provide insight into malicious actors’ procedures. 110 of the 251 domains had a TTL value set to 300 (seconds). These domains were identified as malicious. A table of the remaining TTLs follows. Note that some domains had more than one TTL as they changed owner (from malicious actor to researcher).

| Domain Count| TTL              |  Intent      |
|:----------- |:-----------------|:-------------|
| 110         | 300              | Malicous     |
| 81          | 10800            | Researcher   |
| 58          | 666              | Researcher   |
| 9           | 3600             | Researcher   |
| 5           | 1800             | Researcher   |
| 4           | 600              | ?            |
| 1           | 7200             | ?            |
| 1           | 14400            | ?            |

The 251 command and control domains used 31 different domains as authoritative zones. 21 of those zones had both an ‘ns1’ and ‘ns2’ named system. 5 of the 31 different zones likely belonged to researchers while 4 of the zones were eventually parked. 12 registrars were used to register the 251 domains. Dynadot and 1 & 1 Internet AG were both the most used registrars, however all domains registered through these registrars belonged to security researchers. The third and forth most used registrars where TodayNic and Melbourne IT. All of the domains registered through both of these registrars were malicious.

Looking at the registrant email address found in whois information for both the command and control domains and their authoritative domains, 99 different email addresses were observed. These do not include the email addresses used by researchers. Many of the email addresses were hidden through the registrar’s whois protection service. Of those emails addresses that were not hidden, free email account providers were used. These providers included Yahoo, GMX, and AOL amongst others. An interesting pattern appeared in some of the email addresses where multiple words were used to create usernames. What seemed to be typos (including additional letters or rearranged letters) of previously used email addresses were later used in whois information. An example follows: medicallassers@ymail.com was used to register a domain, then later medicallaserss@ymail.com (note the double ‘s’ at the end of the username) was used to register an additional domain. Attempts to generalize this pattern were inconclusive.

From all command and control domains and those domains’ name servers, 86 unique IP addresses were seen. Many of the IP addresses in this set hosted multiple command and control and name server systems. 54 unique locations were identified using IP whois information; 3 of the locations have been successfully identified as researchers. While the remaining consist of VPSs, ISPs, and compromised systems. Graphing the semantic networks between command and control domain names, their IP addresses, and their name server’s IP addresses created 6 distinct groupings. The largest group was identified as the malicious botnet while the other groups belong to researchers we have confirmed and groups we believe to be researchers given hosting, registrar, and TTL preferences. 

As of November 12, 2014, no new domains have resolved belonging to malicious actors, although researcher sinkholes are still active. Client queries for active command and control domains are still at volumes of 100 to 200 queries per hour.
        	
## Conclusion

In talk, we covered various aspects of network intelligence and how we use it to effectively investigate and mitigate malware campaigns. We discussed two use cases that represent two recent malicious campaigns: the first is the Zbot fast flux proxy network which is currently live and operating and the second is the new GameOver Zeus DGA malware that was active between July and November 2014. 

The slides used to present this material are available online and can be found [here](http://www.slideshare.net/OpenDNS/shmoocon-2015-presentation).

#### Metadata

Tags: Botnets, Gameover Zeus, Threat Tracking, Network Intelligence

**Primary Author Name**: Anthony Kasza  
**Primary Author Affiliation**: OpenDNS  
**Primary Author Email**: ak@opendns (@anthonykasza on Twitter)

**Additional Author Name**: Dhia Mahjoub  
**Additional Author Affiliation**: OpenDNS  
**Additional Author Email**: dhia@opendns.com (@DhiaLite on Twitter)  