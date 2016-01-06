# Building for Usability

## Abstract
There is a clear need for easy to use security products. We have the crypto - what about
usability? This paper explores five of the most high impact ways that you can build more usable software.

## Content

The need for easy to use encryption is now evident. Due to high profile hacks like the attack on Sony Pictures, mainstream users are beginning to take security seriously. Privacy activists like Snowden and the EFF are calling on technologists to build easy to use encryption products. Journalists and venture capitalists are amplifying that message. It’s clear that there is a social need, and a private market, for easy to use encryption products. The next step is figuring out how we can build more usable applications. 

 '##'Build and Test the Design First’

My talk at Shmoocon was presented in an irreverent fashion but the core message is sincere. Building usable apps requires starting with strong design and then doing user testing. It’s critical to build and test the design before building the backend. This allows you to discover that your feature set needs improvement, or that there are usability issues to correct. It’s much easier to change the design or feature specifications before coding has begun. 

A strong example of this approach is Instagram. Instagram was originally a photo + checkins app called Burbn. The Instagram founders would create code only when necessary, instead using paper and other prototyping tools. Instagram co-founders Mike Krieger and Kevin Systrom would go to a cafe with little iPhone design pads. According to Krieger, “we’d build and throw away entire features. You’d waste three or four pieces of paper, not three weeks of coding.”

Burbn became successful only after user testing led the founders to realize that the Burbn app’s design and feature set was too complicated. They cut several features from Burbn, and created the simpler app that we now know as Instagram. Instagram would never have become a multi-billion dollar success without a strong process for design, prototyping and user testing. 

(“Instagram Co-Founder Mike Krieger’s 8 Principles For Building Products People Want” by Josh Constine. November 30, 2012. http://techcrunch.com/2012/11/30/instagram-co-founder-mike-kriegers-8-principles-for-building-products-people-want/)

(“How Instagram Grew From Foursquare Knock-Off to $1 Billion Photo Empire” by Eric Markowitz. April 10, 2012.  http://www.inc.com/eric-markowitz/life-and-times-of-instagram-the-complete-original-story.html)
 
2.  '##'Design for Your Users’ Threat Model’

“Made for Spies” is awesome if your users are spies. But if they are account executives (at Sony, for example) then we need to account for them being less motivated than the average spy. Let’s use messaging apps as an example. Mainstream users (executives included) need apps that are easy to use or else they will resort to DMing on Facebook and using other insecure communications.

Building for the specific threat model allows us to know when and where we can make trade-offs that may improve usability. For example, an app designed to the highest security specifications would automatically log out the user after a period of inactivity. An app designed for usability would allow the user to stay logged in for longer, or to save the username and password in the login fields. 

Another area where we can easily build for both security and usability is in the user settings. We can allow the users to choose their settings, while informing them whether their choices are high or low security. This kind of design pattern applies to tasks like choosing a password, how long the user can be inactive before the app logs out, or whether or not to use GPS location services.

That’s the approach taken by Blackphone’s operating system PrivatOS, which has strong security defaults but allows the user to override those settings. (Blackphone Support Security Center. Website visited February 6, 2015. https://support.blackphone.ch/customer/portal/topics/660716-security-center/articles)

3.  ‘##Using New Tech for Identity Verification' 
A classic problem with encryption and usability has been verifying other parties. We have new tools to solve for this (as well as a changing user demographic for encrypted products) and that should become part of the conversation about building usable encryption. 

First, let’s look at who is using encryption. Real name communication is becoming an increasingly important use case. Executives, for example, do not (in general) need to be anonymous with each other but do need to maintain privacy standards. Other use cases, like anonymous communication, are important but we are focusing on the real names use case for this example.

Now let’s look at the tools at our disposal. Glimpse, for example, is an end to end encrypted photo and video messaging app. (Full disclosure, I led the team that built this app.) Using an app like this I can send a photo to you, in real time, which absolutely verifies that I am the sender. The Glimpse app has icons to show whether a photo or video has been uploaded from camera roll or whether it is being sent in real time. This kind of iconography is important, in order to evaluate the photo that is being received. This principle of verifying your contacts via video chat or real time photos applies to any encrypted app with a similar feature set. This does not work well for anonymous use cases (obviously) but real name real time encrypted chat is worth our attention. Note that Glimpse is out of the app store while we harden security features.

Once we are using real names or pseudonyms, we also have the option of integrating Twitter and/or Github for identity verification. Keybase successfully incorporates Twitter and Github to verify identity. (“Keybase Docs: Understanding Tracking.” Website visited February 6, 2015. https://keybase.io/docs/tracking) Facebook and LinkedIn are also useful for real name verification but are problematic to integrate into a security-focused application because both companies are facing lawsuits for privacy violations. As a result of privacy concerns, some users have a mistrust of Facebook and LinkedIn’s authentication features. 

4. Test and Iterate with Your User Base

Many apps are only tested by the programmers (or programmers and quality assurance staff) that built the app. It’s important to test with your user base! You can perform usability testing on your target demographic easily and affordably at scale using sites like usertesting.com.  There are other reputable sites for testing your app but UserTesting.com is particularly useful because they work with over a million user testers, allowing you to target your testing for whatever demographic you are targeting. See: http://www.usertesting.com/product/users

5. Test at Bars and Coffee Shops

Sites like usertesting.com are useful because they scale so well. However it can be very helpful to watch people (real people, live in person) using your product. People in bars and coffee shops are relatively willing to start conversations with strangers, making it a very effective and cost efficient way to user test. Instagram founder Krieger call this “the bar exam.” He says:
“Figure out how you would pitch your app or product to a stranger in a bar in 20 seconds, and if it’s too complicated to explain, re-think the simplicity. He said with their precursor to Instagram, called Burbn, they failed the bar test and realized they needed to re-work it.” (“Instagram Co-Founder on Startup Success” November 30, 2012. https://gigaom.com/2012/11/30/instagram-co-founder-on-startup-success-identify-your-early-obsessions/)

Title: Building for Usability
Subtitle:
Primary Author Name: Elissa Shevinsky	
Primary Author Affiliation: CTO of JeKuDo Privacy Company, Author at OR Books
Primary Author Email: Elissabiz@gmail.com
Additional Author Name:
Additional Author Affiliation:
Additional Author Email:
(repeat Additional Author as necessary)
Keywords/Tags: (comma delimited) Usability, User Testing, UI, UX, Drinking, Bourbon, Burbn
Abstract: (limited to 200 words)