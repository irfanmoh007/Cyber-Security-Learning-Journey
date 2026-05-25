# Investigating Windows — TryHackMe

**Difficulty**: Easy (but need to spend some time and search a lot)
**Category**: Threat hunting / Analysis

## Honest Reflection
This was kinda easy entry level room but it took some time for me to find the answers cause it is my first time solving this type of windows 
investigation. 

**Question 1 - Whats the version and year of the windows machine?
you can find the answer by just typing systeminfo in the cmd and find the answer 

**Question 2 - Which user logged in last?
go to event viewer , and filter for the event id 4624 successful login ,and you can find the answer 

**Question 3 - When did John log onto the system last?
just like the above quesiton , go to event viewer filter for event id 4624 and scroll until u find john as target username 

**Question 4 - What IP does the system connect to when it first starts?
you can find answer to the question by looking the powershell window when you first login to the system , it will open for a split second and automatically 
close , if not there is another way to find the answer , attackers hide autostarting a process in registry ,  we can go a look at there and can find 
the answer in HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run and click the file inside the folder 

** Question 5 - What two accounts had administrative privileges (other than the Administrator user)?
you can find teh answer by just typing net localgroup administrators 

**Question 6 - Whats the name of the scheduled task that is malicous.
you can go task scheduler or just type this command schtasks

**Question 7 - What file was the task trying to run daily?
go to task scheduler , click the malicious process look at the start a program in the bottom window and find the answer 

**Question 8 - What port did this file listen locally for?
same process as above , look carefully in the start a program field

**Question 9 - When did Jenny last logon?
type this command net user jenny 

## after this , the questions are a bit hard to find cause as a beginner you wouldn't know where do i need to look for finding the answer 
**Question 10 - At what date did the compromise take place?
this took me a little long time  , but successfully found the answer , the answer lies in the previous questions take a close look at the scheduled 
task quesiton and check its history and properties 

**Question 11 - During the compromise, at what time did Windows first assign special privileges to a new logon?
filter the event id 4672 , cause adversary usually creates a new user and escalate their privileges to create backdoors

**Question 12 - What tool was used to get Windows passwords?
for this take a look at powershell command history , or you can go and look at the malicious folder created by the attacker and check there , you will find the answer 

**Question 13 - What was the attackers external control and command servers IP?
local DNS poisoning on a single Windows machine is almost always done via the hosts file.
You navigate to C:\Windows\System32\drivers\etc\hosts

**Question 14 - What was the attackers external control and command servers IP?
if a windows machine is acting as a server , it will store all its files in C:\inetpub\wwwroot\.
in linux it is something like this var/www/html i guess im not sure 

**Question 15 - What was the last port the attacker opened?
check on firewall inbound rules you will find the answer 

**Question 16 - Check for DNS poisoning, what site was targeted?
you can find the answer to the question by looking at the host folder :\Windows\System32\drivers\etc\hosts









