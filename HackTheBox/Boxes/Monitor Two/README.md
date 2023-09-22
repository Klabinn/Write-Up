# Monitor Two
`Web` `Vulnerability Assessment`
<br>
<br>
So the first thing i did is some information gathering.
I immediately tried to connect to its IP, and it turns out there is a homepage.
![Untitled](https://github.com/Klabinn/Write-up/assets/107389203/d3b7362b-c3cf-45f2-a667-36ee022c6bc6)

And the next thing i did is, obviously nmap! Usually i use -sV -O -p- -T5. But my computer is acting out this last few week so this time i only use raw nmap `Not Recommended`
<br>
This is my write-up run, prior to this i use the -sV -O -T5 -p- command to scan the network. 

![Untitled (1)](https://github.com/Klabinn/Write-up/assets/107389203/95cb5e7c-46dc-4343-b818-39697424c2c8)

Since the useful port is scanned then im just gonna use this for reference. 

Ok now we can see that there are 2 open services ssh and http. We already access the http service, for now just keep the ssh in mind.
<br>
Lets go back to the http, if we notice the web uses cacti.
> Cacti is web-based network monitoring, performance, fault and configuration management framework

We can see that it uses cacti 1.2.22, and if we googled cacti 1.2.22 its the very first recomendation CVE-2022-46196 RCE.
https://github.com/FredBrave/CVE-2022-46169-CACTI-1.2.22/tree/main

The exploit is pretty straight forward it works as a reverse shell 

![Untitled (3)](https://github.com/Klabinn/Write-up/assets/107389203/61af99c0-b5c0-4a03-b964-25a7d50feba0)

Set up the listener

![Untitled (2)](https://github.com/Klabinn/Write-up/assets/107389203/f4c146f7-5747-4b64-88ca-d3bba68c788d)

Run the .py to gain shell on the docker.

We are in! the docker atleast...

Now we scroll around the shell, i found some interesting things

![Untitled (4)](https://github.com/Klabinn/Write-up/assets/107389203/b6ddf34e-d328-47c9-8268-734e2d7ab1a8)

![Untitled (5)](https://github.com/Klabinn/Write-up/assets/107389203/2da6ef77-e346-4db8-a70b-ea62d1611055)

here i assume that the password is default and its correct its root root

![Untitled (6)](https://github.com/Klabinn/Write-up/assets/107389203/5dacc033-6db7-44f7-adcf-ede01cdd189c)

Since we find the sus table just dump the data

![Untitled (7)](https://github.com/Klabinn/Write-up/assets/107389203/238b64f3-17a1-4fa1-aefe-a2853b784fa9)

NOW! we have the good stuff

lets hashcat the hashes

![Untitled (8)](https://github.com/Klabinn/Write-up/assets/107389203/e59b634d-2e1a-40fb-9710-1fdd80081c3d)
![Untitled (9)](https://github.com/Klabinn/Write-up/assets/107389203/d88b0eed-55ed-4f5d-90d4-fc64cf2c148b)

We get the pass for marcus now we login to the ssh.

![Untitled (11)](https://github.com/Klabinn/Write-up/assets/107389203/bf93620e-b4ef-4c0e-97c6-d493bb7cf32c)

![Untitled (12)](https://github.com/Klabinn/Write-up/assets/107389203/addd39d0-8c41-4609-9917-b461e7687fc9)

Yay user gained!

## Now Root.
Lets look around and find vulnurable stuff we could play with.

We find that the docker it self is vulnurable

Since the machine is already retired i cant actually check the docker version but the cve i used is CVE-2021-41091

but the version is lower that 20.10.9 version. Because the CVE got patched on that version

https://github.com/UncleJ4ck/CVE-2021-41091

Theres a step by step guide on how to do the exploit. To save some times, im just gonna skip the explenation you could learn the setup on the link above.

But to actually do this exploit you need to escelate /bin/bash restriction using chmod. But you cant do it because the docker is not root.

Now we try to escelate the privilage of the docker :D.

![Untitled (13)](https://github.com/Klabinn/Write-up/assets/107389203/585cab65-8fc8-42d8-9802-acd839fd2a2f)

these are the commands we can play with. Not much, but we have capsh

https://ihsansencan.github.io/privilege-escalation/linux/binaries/capsh.html

the repo above shows that its possible to escelate privilege using capsh. So... we gonna use this method to escelate our docker privilege.

![Untitled (14)](https://github.com/Klabinn/Write-up/assets/107389203/c6bb011c-a758-4a87-b75f-e69db397e030)

okay now we can do the chmod

```chmod u+s /bin/bash```

Now we go back to the ssh and run the exploit to find root privilage directory

![Untitled (15)](https://github.com/Klabinn/Write-up/assets/107389203/3c31c717-02b4-4882-9e53-f538b273836f)

![Untitled (16)](https://github.com/Klabinn/Write-up/assets/107389203/f5df00b0-21d6-4710-a36e-b6e8e067eb9d)

and we gain bash that is root.

![Untitled (17)](https://github.com/Klabinn/Write-up/assets/107389203/eb10fb9e-468c-4220-8177-2847b63c3c25)

And there you go. we get both flags!
