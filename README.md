# CyberDefenders-PhishStrike-Challenge-Walkthrough
CyberDefenders — PhishStrike Challenge Walkthrough

Intro:
This repository documents my experience tackling the CyberDefenders.org "PhishStrike" challenge, which focuses on analyzing a phishing email and associated malware samples. The challenge involved tasks such as dissecting email headers, identifying malicious URLs and IP addresses, analyzing malware behavior in sandboxed environments, and uncovering command and control infrastructure. Through this hands-on exercise, I gained practical experience in threat hunting, malware analysis, and understanding common attack vectors used in phishing campaigns. The documented steps include clear explanations and references to the tools and resources used throughout the analysis process, providing a comprehensive overview of the techniques and insights gained from the challenge.

Here is the link for the following challenge: https://cyberdefenders.org/blueteam-ctf-challenges/phishstrike/

Scenario:
As a cybersecurity analyst at an educational institution, you receive an alert about a phishing email targeting faculty members. The email appears to be from a trusted contact and claims a $625,000 purchase, providing a link to download an invoice.
Your task is to investigate the email using Threat Intel tools. Analyze the email headers and inspect the link for malicious content. Identify any Indicators of Compromise (IOCs) and document your findings to prevent potential fraud and educate faculty on phishing recognition.
To perfom comphrensive analysis we will use accessory tools such as:
- MXToolBox [https://mxtoolbox.com/]: This tool helps us perform a detailed analysis of the email headers. It offers easy-to-read insights about the sender and any potential anomalies that we can explore.

- URLhaus [https://urlhaus.abuse.ch/]: After analyzing the headers, we may uncover some suspicious URLs. URLhaus is a service where we can gather intelligence about these URLs by checking them against a database of known malicious domains, giving us valuable context about potential malware hosted on them.

- Virus Total [https://www.virustotal.com/gui/]: After identifying details about the malware, we can submit the file hashes to VirusTotal to get comprehensive scan results and analysis.

- MalwareBazaar [https://bazaar.abuse.ch/]: This is a repository used to share malware samples with the infosec community. Here, we can search for additional reports about the uploaded samples to understand the malware’s behavior.

- AnyRun [https://app.any.run/]: used in this case for interactive online malware analysis. It allows cybersecurity specialists to detect, monitor, and research cyber threats in real-time by providing an interactive sandbox environment where they can influence the virtual machine and control the analysis process. It helps analyze suspicious files, examine malware behavior, and collect threat intelligence.

- CyberChef [https://gchq.github.io/CyberChef/]: open source and freeware HEX editor.
  
And hey, if you find this walkthrough helpful — whether it levels-up your skills, gets you through a stumbling block, or serves as a handy reference — please feel free to share it and give it a thumbs up!
Thanks and let's get going with the investigation!

!BEFORE STARTING!:
Safety first! When working with lab/challenge files from CyberDefenders (or any educational lab/challenge/range), it’s important to be responsible and stay safe by interacting with potentially malicious files in a dedicated, isolated virtual machine environment. I deeply suggest to either download the file from a comparmentalized VM segregated from you main network or go for REMnux, a specialized Linux distribution for malware analysis. Below the link to the relevant documentation and how to download and set it up in your preferred VM.
[https://docs.remnux.org/install-distro/get-virtual-appliance?source=post_page-----756553475c73---------------------------------------]

WALKTHROUGH:

Step 1 - We will need to download the file containing the email, and once downloaded, we will need to analyze it. 
Have a look at it first how it looks and take notes of the key details.

Step 2 - Then we will have to view the email in source format: I've used Visual Studio Code in restricted mode. Restricted Mode in VS Code prevents automatic code execution from untrusted sources. To disable it, you can modify the settings: File > Preferences > Settings > Search for "Trust" and then disable the checkbox that says "Controls whether or not workspace trust is enabled within VS Code". Alternatively, set "security.workspace.trust.enabled": false in your settings.json file. We can then have a look at the questions:

Q1) Identifying the sender's IP address with specific SPF and DKIM values helps trace the source of the phishing email. What is the sender's IP address that has an SPF value of softfail and a DKIM value of fail?

![Q1_phishing_cyberdefenders](https://github.com/user-attachments/assets/4680a56e-15a8-4129-949e-81cd528ca6cc)

As seen in the picture we can learly identify the sender IP address, which is 18.208.22.104

Q2) Understanding the return path of an email is essential for tracing its origin. What is the return path specified in this email?

![Q2_phishing_cyberdefenders](https://github.com/user-attachments/assets/f7fdcc02-4665-4a15-9ec3-795199223604)

Search for "Return-Path" in the email source using Ctrl+F to find the specified return path. 
The answer is erikajohana.lopez@uptc.edu.co

Q3) Identifying the source of malware is critical for effective threat mitigation and response. What is the IP address of the server hosting the malicious file related to malware distribution?

![Q3_phishing_cyberdefenders](https://github.com/user-attachments/assets/7f6776a3-c06a-4bbc-b6eb-8fc56b979417)

Scrolling down into the email body we cam clearly see a malicious link to an Ip address with the name loader/install.exe with the associated IP of 107.175.247.199

Q4) Identifying malware that exploits system resources for cryptocurrency mining is critical for prioritizing threat mitigation efforts. The malicious URL can deliver several malware types. Which malware family is responsible for cryptocurrency mining?

![Q4_phishing_cyberdefenders](https://github.com/user-attachments/assets/796112fb-c52e-4607-84e5-cfe8f792d69b)

Grabbing the IP address and the malicious .exe file, we can check for details in the URLhaus webiste [https://urlhaus.abuse.ch/] to collect some intelligence about the file. From the tags, we’ll notice that this URL is associated with several different malware types. To answer Question 4, we are interested in the tag associated with cryptocurrency mining — CoinMiner as shown in the screenshot below.

![Q4_phishing_cyberdefenders-2](https://github.com/user-attachments/assets/cb82ac40-df98-4d1f-8a73-2f1a5aa92ca3)

Q5) Identifying the specific URLs malware requests is key to disrupting its communication channels and reducing its impact. Based on the previous analysis of the cryptocurrency malware sample, what does this malware request the URL?
For example, let’s navigate to VirusTotal and search the CoinMiner SHA256 hash. We’ll check the Relations tab under Contacted URLs to understand what URLS the malware communicates with based on previous analysis on the service. We can then grab the SHA256  hash of the file and check on virus total [https://www.virustotal.com/gui/home/upload] and under the section "RELATIONS"  we can ses to which URLs the malware communicates with. Looking at the URLs, one is familiar (http://107.175.247.199/loader/server.exe) whil the other one is the response to our question.

![Q5_phishing_cyberdefenders](https://github.com/user-attachments/assets/74888a7d-4e23-4f7a-a524-1bea59e1a1c4)

Q6) Understanding the registry entries added to the auto-run key by malware is crucial for identifying its persistence mechanisms. Based on the BitRAT malware sample analysis, what is the executable's name in the first value added to the registry auto-run key?
This step will require a bit more analysis to find the answer. We will now shift our focus on the BitRat malware analysis.

![Q6_phishing_cyberdefenders-1](https://github.com/user-attachments/assets/b6ce92fa-2107-4170-9e80-d70a24faa91b)

We will analyze it throught MalwareBazaar platform (https://bazaar.abuse.ch/) to eventually find out the link for the AnyRun Sandbox application which will give us a more comphrensive and live iteration of the malware.

![Q6_phishing_cyberdefenders-2](https://github.com/user-attachments/assets/4b158e1b-16b4-407b-a5e9-3a832ba8e34c)

Below the screenshots for the AnyRun.app 

![Q6_phishing_cyberdefenders-3](https://github.com/user-attachments/assets/53ee49b2-f81d-4fa6-9220-1d1c57b670c4)
![Q6_phishing_cyberdefenders-4](https://github.com/user-attachments/assets/cd07c2e4-4ae4-490b-96d5-5eade1a77b21)

Here we can find the MITRE matrix to help us gather in-dept details about the malware, specifically the T1547.001 MITRE ATT&CK technique (Registry Run Keys) [https://attack.mitre.org/techniques/T1547/001/]. The executable name (e.g., C:\Users\admin\AppData\Roaming\Ozndcoodb\Jzwvix.exe) will be listed as the value change. We gonna grab the value and type into the slot to get our answer checked.

Q7) Identifying the SHA-256 hash of files downloaded from a malicious URL is essential for tracking and analyzing malware activity. Based on the BitRAT analysis, what is the SHA-256 hash of the file previously downloaded and added to the autorun keys?
Continuing our BitRAT analysis on VirusTotal, let’s find the SHA-256 file hash of the executable we found in the previous question via the URLhaus platform.

![Q7_phishing_cyberdefenders-1](https://github.com/user-attachments/assets/49ad8770-fae9-41cf-a856-c09e7e35cb77)

Q8) Analyzing the HTTP requests made by malware helps in identifying its communication patterns. What is the URL in the HTTP request used by the loader to retrieve the BitRAT malware?
We will have to go back to VirusTotal and check the "Relations" section to find the requested URL from the malware executable.

![Q8_phishing_cyberdefenders-2](https://github.com/user-attachments/assets/98a208a3-b473-42a4-a1db-663bebf97b50)

Or conversly we can get the URL from the HTTP requests table in the AnyRun.app (below).

![Q8_phishing_cyberdefenders](https://github.com/user-attachments/assets/4cef28fd-d3b6-4871-b71a-ac03dc2a8361)

Q9) Introducing a delay in malware execution can help evade detection mechanisms. What is the delay (in seconds) caused by the PowerShell command according to the BitRAT analysis?
This one got me working a bit as I had to go back and analyze the malware behaviour. Basically:
- Malware often employs delays to evade detection by security mechanisms. By introducing a pause before executing its payload, it can avoid triggering alarms or being detected by security software that monitors for immediate malicious activity.
- Analyzing execution delays helps in understanding how the malware operates and what techniques it uses to maintain persistence or execute its tasks stealthily.
Then we can proceed with the analysis by analyzing the Shell commands displayed to focus in particularly on this command ("C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -enc UwB0AGEAcgB0AC0AUwBsAGUAZQBwACAALQBTAGUAYwBvAG4AZABzACAANQAwAA==) which is a Base64-encoded string, in other words it has been ofbfuscated to hide its true intent from casual observer.
So how will we read this string? Simple, using an Hex Editor by the likes of CyberChef [https://gchq.github.io/CyberChef/] or by downloading the freeware HxD Hex editor software on your machine [https://mh-nexus.de/en/hxd/].
By decoding the string into readable characters we can see that the delay time is  set to 50 seconds.

![Q9_phishing_cyberdefenders](https://github.com/user-attachments/assets/dc358ef9-9246-4b95-8743-f5f249e028c9)

Q10) Tracking the command and control (C2) domains used by malware is essential for detecting and blocking malicious activities. What is the C2 domain used by the BitRAT malware?
After reviewing the network connections on VirusTotal, we might think that we’ve already discovered the command and control (C2) URL, but none of the domains that we have uncovered so far fit the format that the question is looking for. We will need a thorough analysis.
On the MalwareBazaar page for the BitRAT sample, scroll down to the Vendor Threat Intelligence section and choose the Hatching Triage entry to see an overview of their findings. What do you notice?

![1_cKn0mx88JWA6FM9I1vReNQ](https://github.com/user-attachments/assets/a8b9bd66-d6cd-406f-b5f2-67975b808191)

Conversely, we can achhieve the saem result via the analysis on the AnyRun.app (below).
In the ANY.RUN analysis, examine the "Connections" section to identify the Command and Control domain (C2). The C2 domain used by BitRAT is gh9st.mywire.org.

![Q10_phishing_cyberdefenders-1](https://github.com/user-attachments/assets/52868beb-ad4b-450c-948b-b7f8cdf4a99d)
![Q10_phishing_cyberdefenders-2](https://github.com/user-attachments/assets/a1cf3553-90be-471c-b44a-746aa692e6bc)

Q11) Understanding how malware exfiltrates data is essential for detecting and preventing data breaches. According to the AsyncRAT analysis, what is the Telegram Bot ID used by this malware?
Here we’ll apply the same process we did in the previous question, this time selecting the AsyncRAT link to view the sample on MalwareBazaar. During the AsyncRAT analysis, we have examined threat intelligence vendor reports and we discovered a very compelling and detailed report from Hackign Triage. Let’s analyze their full report to extract anything that will help us get closer to the answer. 

![Q11_phishing_cyberdefenders-1](https://github.com/user-attachments/assets/20d8ff09-bf98-4a4c-9b39-a001839cc570)

In the task section, under "Behavioral," scroll to the network area. Identify the Telegram Bot ID used by AsyncRAT, such as bot5610920260. This ID is used for communication via the Telegram API.

![Q11_phishing_cyberdefenders-2](https://github.com/user-attachments/assets/5e3a837b-7764-43d5-bc4e-b46a7c7bbef9)
![Q11_phishing_cyberdefenders-3](https://github.com/user-attachments/assets/c0f33348-93c0-4abe-8de9-25882873d825)
![Q11_phishing_cyberdefenders-4](https://github.com/user-attachments/assets/d9f3ab62-9add-4d80-851b-6f4426af86f8)


CONCLUSION
Great job on completing the PhishStrike challenge! Starting with a single email, we utilized MXToolBox to investigate a spoofed trusted contact and uncovered a suspicious URL within the email body. By leveraging tools like URLhaus and VirusTotal, we gathered threat intelligence on three different malware samples delivered by the malicious server, allowing us to analyze their behaviors comprehensively. Additionally, we consulted external reports to understand potential data exfiltration methods. With our objectives met, we now possess the crucial information needed to safeguard our institution from this threat. Let’s wrap up the PhishStrike challenge!

I'd like to thank CyberDefenders for creating such an engaging and realistic lab scenario. This challenge provided a fantastic opportunity to sharpen investigative skills, starting with minimal information and expanding into a thorough analysis. It’s always rewarding to connect the dots between artifacts, tools, and external research to uncover the full scope of a threat. In real-world scenarios, where speed and accuracy are critical, platforms like VirusTotal and URLhaus prove invaluable for identifying and mitigating threats efficiently. Exercises like this are not just practice—they’re essential preparation for the challenges we face every day in cybersecurity.

I hope you found this walkthrough insightful as well! If you found this content helpful, please consider giving it a clap! Your feedback is invaluable and motivates me to continue supporting your journey in the cybersecurity community. Remember, cybersecurity is a team sport, and we’re all in this together! For more insights and updates on cybersecurity analysis, follow me on Substack! [https://substack.com/@atlasprotect?r=1f5xo4&utm_campaign=profile&utm_medium=profile-page]

