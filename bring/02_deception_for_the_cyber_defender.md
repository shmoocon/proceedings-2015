# Deception for the Cyber Defender
### To Err is Human; to Deceive, Divine

## Abstract

Since the first conflict between man, deception has played an integral role. Today on the network battlefield attackers enjoy many advantages and frequently employ deception as a powerful tool to accomplish their objectives. In this paper we discuss how to turn the tables on the attacker and employ deception strategies that deceive both human attackers and the code they employ to best defend your assets.  We do so by mapping traditional and well-developed military battlefield deception techniques and principles onto the cyber domain.

## Introduction

Using deception in computer network attack and defense is not a new concept.  Malicious hackers have used spoofed packet headers, social engineering, phishing, and other deception techniques for years to deceive users and network devices in order to gain unauthorized access to target systems and networks.  Defenders, however, have taken less advantage of deception in their efforts to keep their networks safe.  Deception in the defense has largely been limited to honeypots or honeynets, servers or network segments that appear to be lucrative targets for hackers but are really unused systems designed to waste attackers' time and capture their tools.  In the past, honeypots have primarily been designed to trap opportunistic attackers and have had limited usefulness. 

In a world where persistent attackers are operating with knowledge of 0-day vulnerabilities and IDS evasion techniques, deceptive defensive tactics may deserve a second look.  Honeynets can be useful if they are well-designed and implemented, and if they are properly monitored by network defenders.  They can buy time for defenders to harden suspected target systems and to evict intruders.  Even better, they might help reveal how attackers gained initial access and their general tactics and techniques.  In a perfect world, a honeynet would be so well-conceived and constructed that it convinces attackers that they got what they are looking for, or that what they are looking for doesn't exist anywhere in the target network.  The best way to protect against persistent attackers who can evade technical controls may be to manipulate the attacker's perceptions.  

We argue that there are many effective ways to use deception in the offense and the defense.  After all, armies have been doing this for millennia -- if we can understand how militaries think about deception, and how it has been used historically, we can draw lessons that will help us effectively use deception to defend our systems and networks.

##Basics of Deception for the Cyber Defender

>*"Thou fraud (deception) in other activities be detestable, in the management 
of war it is laudable and glorious, and he who overcomes an enemy by fraud is 
as much to be praised as he who does so by force." - Niccolo Machiavelli, 
Discourses, 1517*
	
Deception is the construction of a false reality for an adversary via intentionally "leaked" false information or other measures.  The objective of a deception event is to affect an adversary's behavior such that his action (or inaction) provides an advantage to you or your allies.  Deception in computer network operations is different from classic military deception in that the target might be either an enemy decision maker OR an electronic system in an adversary's network.  For example, a network defender might try to deceive malware by modifying virtual machine settings to try to convince the software that it is running directly on hardware to improve the ability to analyze it.

Military operators think in terms of effects.  Staffs generally begin planning by considering the specific effect they want to have on an adversary (example effects are degrade, delay, destroy, disrupt, divert, neutralize, and suppress) and then think about the tools they have at their disposal and how those tools can be used to achieve the desired effect.  The same should be true for your defensive deception campaign.  What sort of effect do you want to achieve, and on who (or what) do you want to achieve those effects?  Table 1, below, provides a sampling of cyber effects and how an attacker or a defender might approach each one.

TABLE 1. EXAMPLE DECEPTION EFFECTS FOR CYBER ATTACKERS AND DEFENDERS

| Effect | Attacker | Defender |
|:-------|:---------|:---------|
| Fail to observe | Prevent the defender from detecting the attack. | Prevent the attacker from discovering their target. |
| Reveal | Trick the defender into providing access. | Trick the attacker into revealing their presence. |
| Waste Time | Focus the defender's attention on the wrong aspects of the incident. | Focus the attacker's efforts on the wrong target. |
| Underestimate | Induce the defender to think the attack is unsophisticated, not targeted. | Induce the attacker into thinking that the sought after thing is not here. |
| Disengage | Induce the defender into thinking that the attack is contained or completed. | Induce the attacker into thinking that they have already achieved their goal. |
| Misdirect | Focus the defender on a different attacker. | Encourage the attacker to target a different victim. |
| Misattribute | Induce the defender into thinking that the attacker is someone else. | Induce the attacker into thinking that they’ve compromised the wrong network. |


It is useful to consider the level at which your deception operation will be applied.  Military planners divide operations into three levels of warfare: strategic, operational, and tactical.  Strategic deception are those high-level techniques that apply to the entirety of a large, perhaps geographically distributed, organization.  It might be used to disguise basic objectives, intentions, strategies, and capabilities of the organization as a whole.  Operational deception bridges the gap between strategic and tactical.  It might be used to confuse an adversary regarding specific operations or activities you are preparing to conduct.  Tactical deception is used to mislead others while they are actively involved in competition with you, your interests, or your forces.  It is important to understand that these categorizations are not based on the type of deception being practiced, but rather on the overall objective of the deception.

## Deception Maxims

Over the years a handful of deception maxims, or general principles, have been developed in military circles that are also very relevant to cyber deception operations.  Here we will briefly describe some of these maxims.
 
**"Jones Dilemma":**  This maxim states that deception becomes more difficult as the number of sources available to confirm the real increases.  As you shape your deception operation, you must consider your adversary's sources of information outside those that you can influence and the likelihood that s/he will use those sources.  As an attacker, do you have multiple ways to observe the target?  Often, attackers don't think this through because deceptive defensive operations are not expected.  As a defender, are you cross-correlating different sources of information (IDS vs. PCAP vs. Netflow)?  What assumptions do you make when things don’t line up (do you assume your sensors are malfunctioning, or that you are being deceived)?

**Types of Deception Operations:**  When considering your deception plan, it is helpful to understand the two primary types of deception: Ambiguity Deception (A-Type), whereby you increase doubt by providing multiple possible truths, and Misdirection Deception (M-Type), in which you endeavor to decrease doubt by focusing the target on a particular falsehood.  For example, as an attacker with a piece of malware containing bogus IP addresses designed to look like C&C points, you have the choice of using known IPs used in a variety of existing malware campaigns to increase ambiguity, or you can use IPs that all point to systems used by a single campaign as an attempt at misdirection.  Usually, Misdirection Deceptions are more effective than Ambiguity Deceptions because the target is less likely to realize that he or she is being deceived. 

**Axelrod's Contribution:** This maxim suggests that you should husband your deception assets.  Use deception tools and capabilities sparingly and always hold some in reserve, in case you need to use them to bolster your deception efforts in the future.

**The Monkey's Paw:** You should watch for unanticipated reactions to your deception efforts, particularly by friendly forces.  The name refers to an old tale about an enchanted monkey hand that grants wishes, but in unexpected and often undesirable ways.  The best deceptions are those that have the intended effect on your adversary without affecting allies.

**Don't Make It Too Easy:**  Plan the placement of your deceptive material so that the enemy has to work for it.  As a cyber attacker, make the malware analyst unpack your malware or fiddle with some poorly designed “encryption” or encoding to get to your list of phony IP addresses.  They will be much more likely to think that the product of their efforts is something valuable that you didn't want them to find.

**Magruder's Principle:**  This maxim refers to confirmation bias.  A deception is most likely to be believed if it reinforces the target's pre-existing beliefs rather than forcing the target to change their beliefs.  Appear to be something that your adversary expects to find. 

**Limits of Human Information Processing:**  Consider that there are fundamental limits in the way humans process data.  For example, you can take advantage of the fact that people will often draw conclusions based on an insufficient number of data points (referred to as the law of small numbers).  People are also susceptible to conditioning and can be lulled into ignoring events that should draw attention.  People often assume that unlikely events are impossible (Occam's Razor), and deceptions need only be as effective as demanded by the bandwidth of the tool that is used to observe them (a rubber tank looks like a real tank in aerial imagery).  For example, security teams may stop carefully investigating an IDS event if they learn that it fires on a regular basis and the circumstances are usually innocuous.  Attackers can similarly be lulled into thinking that certain kinds of systems usually do not contain interesting data and are not worth the effort to compromise. 

**Carefully Sequence your Deception Events:**  Deception events should tell a story over time, rather than try to convince an adversary with a single event.  The riskiest, or most difficult to believe components of the deception effort should be left to the end when they will be seen to reinforce perceptions caused by earlier, simpler efforts.

**Feedback:**  One of the most challenging aspects of any deception effort is the feedback mechanism.  You must have a way to determine whether your deception events are being witnessed by the target and he believes them.  For example, as a network attacker, you might try to eavesdrop on your target's incident handling team as they react to your deception (or not).

## Principles of Military Deception

U.S. Military doctrine lists a series of deception principles that, unlike the above maxims, should be applied to any deception operation.  Those principles are listed here.

* Focus - the deception must target the adversary decision maker capable of taking the desired actions.  As discussed earlier, the target of cyber deception can be human, or it can be devices or algorithms used in network attack or defense.
* Objective - to cause an adversary to take (or not to take) specific actions, not just to believe certain things.  
* Centralized Planning and Control - military deception operations should be centrally planned and directed.  U.S. Joint Publication 3-13.4 provides a detailed deception planning process than can be modified to inform defensive deception operations.
* Security - deny knowledge of a force’s intent to deceive and the execution of that intent to adversaries.  Operations security is important in deception operations, just like in other operations.
* Timeliness - a deception operation requires careful timing.  Events should be sequenced to maximize the intended effect of the deception on the target.
* Integration - fully integrate each deception with the operation that it is supporting.  The deception plan should be integral to the operation and planned in parallel to ensure maximum benefit.

## Conclusion

Deception, if properly applied, can be a powerful tool for the network defender.  We contend that most deception operations, like much of network security in general, is applied as an afterthought and is not well coordinated with network construction or defense.  Military planners have used deception for millennia to improve their chances of success.  We believe that existing military history and doctrine can be a rich source of inspiration and techniques for the network defender.

##Disclaimer

The views expressed in this paper are those of the authors and do not reflect the official policy or position of Drawbridge Networks, West Point, the Department of the Army, the Department of Defense, or the United States Government.

We are not lawyers, nor are we giving legal advice.  Please consult your legal advisor before even considering deception activities.

Primary Author Name:  Tom Cross
Primary Author Affiliation: Drawbridge Networks
Primary Author Email: tom@tomcross.info
Additional Author Name: Dave Raymond
Additional Author Affiliation: West Point
Additional Author Email: david.raymond@usma.edu
Additional Author Name: Greg Conti
Additional Author Affiliation: West Point
Additional Author Email: gregory.conti@usma.edu
Keywords/Tags: deception, honeypot, honeynet, military, cyber operations
