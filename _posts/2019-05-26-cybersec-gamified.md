---
layout: post
title: Cyber Security Gamified
updated: 2019-05-25 23:00
image: ctfsec.jpg
tags: [ctf, cyber, security, tech]
---

## initialize

Learning cybersecurity theoretically is not as fun as learning it practically, ofcourse you do need to know theory, before getting practical. Because theory is key to exploiting vulnerabilities in software or hardware. Being practical will help you put that theory to use. The fun thing about cyber security is that it's like puzzle solving on steroids basically. One such way to learn cyber security(whilst having fun) is by playing Capture The Flag competitions or CTF for short. This word kind of stems out from the gaming industry, where you have players competing to capture the maximum number of flags. 

## ctfs

So what are CTF's in term's of cyber security? Like I said before it's just like the game in which you compete for capturing the maximum number of flags. The CTFs are mainly 2 types(3 actually if you take "king of the hill" into consideration). You have "Jeopardy" style and "Attack-Defense" style.

### Jeopardy based CTFs

![jeopardy google ctf]({{ site.baseurl }}/img/jeopardy.jpg)

Jeopardy CTFs have a game style similar to the actual jeopardy game, which is a quiz show. There's a leader board with teams ofcourse. There are a set of tasks which need to be completed. Successful completion of these tasks will yeild a so called "flag" which can be submitted to the game in order to get points. The tasks range from all the domains of security like web, cryptography, pwn(binary exploitation), reverse engineering, forensics, programming and so on. The top teir one's even have tasks based on 1-Day exploits in which you can try your hand at exploiting real applications like Firefox, Chrome, etc. The scoring is mainly of two types, static and dynamic. In static each task has the same points throughout the game. Where as in dynamic, the points go on decreasing for a task as more number of teams solve it(which is a better scoring mechanism imo).

### Attack-Defense based CTF's

![attack defense CMU PPP at defcon]({{ site.baseurl}}/img/atdef.jpg)

Now let me speak about this type. Last year I had the privilage to attend one such competition which our college team "Nullsec" had qualified for. It was the first time we had ever played something like it, altough I knew what it was, but playing it in person was fun. Attack Defense based CTFs are usually onsite CTFs as in you have to actually travel to a physical location(which is kinda cool since you get to see places).Back to the point, in attack defense ctfs, each team has a server assigned to them. These servers have various services running on them, which need to be exploited(attack side) and patched(defense side). Then comes the main flag server. This server is responsible for planting flags on all the teams boxes after regular intervals. The flag server plants a different flag for each of the running services. The job of the attacker is to steal these so called "flags" from the other teams by gaining acces to their boxes. How do they do that? They exploit vulnerabilities in the running services. Each of those services have intentional vulnerablilities. It is the job of defense to patch these vulnerabilities. Everything goes in this game, except attacking the infrastructure itself. Now the flags do expire after certain interval, so you gotta keep stealing, here is where automation plays a key role(and where our team screwed up).

## Learning a lot...

There's a lot to learn especially if you're new to the scene. I'd suggest creating a team, or play for any open team. The reason I say play with a team, is because each and every member has a unique prespective about a problem, that helps a lot(playing alone is no fun, and it won't take you very far). Learn Python or any scripting language(I cannot stress how important this is). Get used to reading documentation(the devil is in the details). Play often and learn from other teams. If you get stuck on a problem, once the game is over you can like read "writeups" which are solutions to the problems which were solved by other teams. Even if the team didn't do a writeup, but still solved it, shoot them an email or contact them on irc, chances are they're more than willing to help you out. Learn your basics well, then build upon that. Staying upto date is key. Recently they've even started adding tasks based on blockchain(yes that can be exploited to). Some techniques get outdated and new ones come into play, and CTFs stress a lot on new techniques. You should be convinced by now of how important it is to keep learning by the amount of times I've used "Learn" in the last few sentences. Usually you need to shutdown your service, for patching, the more time the service is down the more penalties you get, in this case you can use the flag servers interval (or so called tick) time to your advantage, basically patch inbetween ticks.(and pray that you patched it the right way xD).

## It changes your prespective

Over the past three years, learning to do binary exploitation, watching other people do it, playing around with reverse engineering has really changed my prespective.Trust me, you will not even look at malware the sameway after you play CTFs. What is malicious code? It's just code to me now, which will get me from point A to point B in the unintended way. Always remember, there's always a way, always.

## Meet effing wizards

The onsite thing, has the perks of socializing. You get to learn a lot from meeting other people and speaking to them. They may (always) do things differently than you, which may be quite interesting. Not everyone solves problems the same way. Speaking of people, there are a lot of youtubers out there who belong to really amazing CTF teams whose channel you should subscribe to, like Gynvael Coldwind from DragonSector and LiveOverflow from ALLES!.

## The perks

Seeing those points get added on a problem you solved is an undescribable feeling, especially when the problem is hard. You get to learn technology in depth I feel through CTFs, the details really matter in these games. You learn weird tricks, no one in their right mind would have tought of, and kind of blow your own mind in the process. Having indepth knowledge is extremely useful in the cyber security field, it helps you overlook fewer things. I'm gonna be honest with you, it is challenging. Everything is at the begining, but once you do it(this is the hardest part-getting yourself to do something), you'll find it easier. Some may argue that these challenges are unrealistic, but I'd say that it does help you become a better programmer at the end of the day. All this knowledge you learn't will also help you be in a better position for cyber security as a career. Cheers!

### Resources

[CTF Time](https://ctftime.org/)
[Overthewire](https://overthewire.org/wargames/)
[PicoCTF](https://picoctf.com/)
[Gynvael Coldwind](https://www.youtube.com/channel/UCCkVMojdBWS-JtH7TliWkVg)
[Liveoverflow](https://www.youtube.com/channel/UClcE-kVhqyiHCcjYwcpfj9w)


