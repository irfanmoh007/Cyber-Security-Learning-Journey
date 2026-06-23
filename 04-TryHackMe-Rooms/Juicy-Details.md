##Juicy details - room 
**Difficulty : Easy
**Concept : Analysis , Forensics 

##Task 1 
What tools did the attacker use? (Order by the occurrence in the log)  
for this use , you need to analyse the user agent header , since the log is huge , i used this command to filter it out kinda , cat access.log | cut -d " " -f 12,13,14,15 | uniq
and you can find the answer 

What endpoint was vulnerable to a brute-force attack? 
just filter the logs using grep hydra , you will find the answer

What endpoint was vulnerable to SQL injection?   
for this too , just filter it out using grep sqlmap

What parameter was used for the SQL injection?  you can find the answer just looking at the sqlmap logs , for hint it is url encoded , the starting letter is the parameter

What endpoint did the attacker try to use to retrieve files? (Include the /) grep using feroxbuster , you know we are retrieving files , so there is your hint 


##Task 2
What section of the website did the attacker use to scrape user email addresses?
just grep it using review , you will find the section of the website and the answer 

Was their brute-force attack successful? If so, what is the timestamp of the successful login? (Yay/Nay, 11/Apr/2021:09:xx:xx +0000) 
use the command , grep hydra and grep 200 and you will find the answer 

What user information was the attacker able to retrieve from the endpoint vulnerable to SQL injection? , you know the answer from the first question and when you look at the vsfpdlogs , you will find another answer

What files did they try to download from the vulnerable endpoint? (endpoint from the previous task, question #5)
you will find the answer when you cat vsfpd log

What service and account name were used to retrieve files from the previous question? (service, username)
this too , the answer lies in the vsfpd logs

What service and username were used to gain shell access to the server? (service, username)
just use authlog and you will easily find the answer 
