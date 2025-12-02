---
title: "STIG Compliance: SCAP, Group Policy Objects, and Evaluate-STIG... Ansible?"
description: ""
pubDate: "Jun 13 2025"
categories: ["Cybersecurity", "DoD", "Windows"]
tags: ["Ansible", "Compliance", "DoD", "DoD Cyber Exchange", "Evaluate-STIG", "NAVSEA", "STIG"]
heroImage: "/images/posts/STIG.jpg"
---
Welcome back to the Windows Server 2022 lab. I set this server up a while back in a [post](https://wbb.afh.mybluehost.me/windows-server-2022-active-directory-lab/) to learn about Active Directory. It's been a little stagnant for a while since I moved all my homelab services away from my Windows and Ubuntu servers to LXCs (I have been really busy). The only VMs I have running now are part of my movement towards a small DoD compliant network - the plan will include a couple of workstations, domain controller, an ACAS solution, etc. Which is all a part of my own personal training to operate efficiently in a DoD environment and just because I am really curious to learn about it all.

**There are a few ways to accomplish STIG compliance, one is publicly accessible on the DoD Cyber Exchange: the SCAP tool, a Powershell script called Evaluate-STIG provided by NAVSEA and requires a CAC for access _(I did not provide that file_, sorry)**, **and Ansible - which I will cover in another post.**

**Links (CAC required):**  
**NIPR:** [https://spork.navsea.navy.mil/nswc-crane-division/evaluate-stig/-/releases](https://spork.navsea.navy.mil/nswc-crane-division/evaluate-stig/-/releases)  
**SIPR:** _If you're on SIPR you should already have this link..._ _lol_  
**Commercial:** [https://intelshare.intelink.gov/sites/NAVSEA-RMF](https://intelshare.intelink.gov/sites/NAVSEA-RMF)

To get started we are going to need a few items from the [DoD Cyber Exchange](https://public.cyber.mil/). If your not familiar with this site, I would definitely check it out and click around - it's a necessity for DoD IT folks.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/06/DOD-Cyber-Exchange-STIG-1024x800.png)

Items we will need for this project are in order below as they are seen on the Windows Server 2022 desktop in most of my screenshots:

Tools:  
[STIG Viewer 2.18](https://dl.dod.cyber.mil/wp-content/uploads/stigs/zip/U_STIGViewer_2-18.zip)  
[](https://dl.dod.cyber.mil/wp-content/uploads/stigs/zip/scc-5.10.2_Windows_bundle.zip)[SCC 5.10.2 Windows](https://dl.dod.cyber.mil/wp-content/uploads/stigs/zip/scc-5.10.2_Windows_bundle.zip)  
[Local Group Policy Object Utility (LGPO.exe) v3.0](https://www.microsoft.com/en-us/download/details.aspx?id=55319) (found under download on the Microsoft Security Compliance Tool site)  
  
Support Files:  
[Compilation - SRG-STIG Library](https://dl.dod.cyber.mil/wp-content/uploads/stigs/zip/U_SRG-STIG_Library_April_2025.zip)  
[Microsoft Windows Server 2022 STIG SCAP Benchmark - Ver 2, Rel 4](https://dl.dod.cyber.mil/wp-content/uploads/stigs/zip/U_MS_Windows_Server_2022_V2R4_STIG_SCAP_1-3_Benchmark.zip)  
[Group Policy Objects (GPO) - April 2025](https://dl.dod.cyber.mil/wp-content/uploads/stigs/zip/U_STIG_GPO_Package_April_2025.zip)

Most of those links are direct links to the tools and supporting files you need. You can find them all across the DoD Exchange site if you know where to look. After you get your tools downloaded and extracted, get them organized in a way that makes sense to you. Below I put the tools to the left and supporting document folders to the right of each they are related too.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/06/DC1-Desktop-Icons-1024x639.png)

The Tools in the first row of my Windows Desktop have a few additional folders to the right. I have added these to ease the understanding of what is happening behind the scenes with files. These are the folders:

Checklist STIG  
SCAP Output  
SCC Benchmarks (Save your downloaded benchmarks here)  
  
_There are more options for [benchmarks](https://public.cyber.mil/stigs/scap/) that can be used with the SCAP tool but for this situation we will use the Windows Server 2022 benchmark to check for compliance._

**SCAP Tool and Benchmarks**

In the scc-5.10.2 folder, you open up the scc.exe - this will be the first tool used to check this Operating System for compliance. The SCAP Output folder on your desktop needs to be set in the **Options** menu of the SCAP Compliance Checker 5.10.2 tool (scc.exe) - click **Output Options** on the left and change the **SCC Results** as seen below to match your **SCAP Output** folder:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/06/SCAP-Output-1024x645.png)

And then you will need to install the Microsoft Windows Server 2022 STIG SCAP Benchmark by clicking install at the top center. When the menu comes up "Select Content File(s) to Install" and browse to your SCC Benchmarks folder you created and select:  
  
**_U\_MS\_Windows\_Server\_2022\_V2R4\_STIG\_SCAP\_1-3\_Benchmark.xml_**  
  
Then click install and you should end up with a visible benchmark in the center under Stream. You can then click Start Scan to the left.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/06/SCAP-Benchmark-Install-1024x645.png)

After you scan completes you should see something similar to below:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/06/SCAP-Scan-Complete-1024x645.png)

Which you can then click "View Results" and you will bring up another menu showing you a percentage score. In my Window Server 2022 VM it's 47.92 - which is a failing score for compliance.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/06/SCAP-Results-1024x645.png)

In order to correct the low compliance score we will need to use the LGPO.exe to apply a quick Group Policy baseline.

**Microsoft's Local Group Policy Object Utility (LGPO.exe) & Security Baselines**

You will need to open the Command Prompt as Administrator and go to the directory with LGPO.exe in it on your Desktop. For me this location is here:

C:\\Users\\Administrator\\Desktop\\LGPO\_30\\LGPO.exe

If you type it in and your path is correct, you should execute the LGPO program as seen below and then scroll up and see the many options available.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/06/LGPO-Tool-1024x643.png)

Let's navigate to the folder by using:

cd C:\\Users\\Administrator\\Desktop\\LGPO\_30\\

That will make the commands slightly cleaner. In the next step make a backup of your current Group Policy with the **/b** modifier:

LGPO.exe /b C:\\Users\\Administrator\\Desktop\\LGPO\_30

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/06/LGPO-GPO-Backup-1024x643.png)

You can see LGPO confirmed the backup location and it is visible in the LGPO folder as well.

Now to begin examining the [Group Policy Objects](https://public.cyber.mil/stigs/gpo/) (GPO) you downloaded above. If you followed my led, it should be right next to your LGPO\_30 folder on the desktop: U\_STIG\_GPO\_Package\_April\_2025.

If you browse through the folder and go into the Reports section you can find a .html file that explains a lot of the GPO settings being applied to the OS. The HTML files:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/06/GPO-HTML-Files-1024x643.png)

Here is a small section of the GPO - you can hide/show as necessary to satiate your quest for knowledge. I opened up the Account Policies/Password Policy section:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/06/GPO-Account-Policies-1024x643.png)

After you have completed a little review of the GPO we can move forward with the application of the **_DoD WinSvr 2022 MS and DC v2r4_** group policy. As seen below, the STIG GPO package we downloaded contains many different folders with DoD approved GPO. The one we will apply is named for the OS we are using in our domain controller. The notepad above shows the exact **path** we will use with LGPO and the **/g** modifier.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/06/STIG-GPO-Package-April-2025-1024x643.png)

LGPO.exe /g "C:\\Users\\Administrator\\Desktop\\U\_STIG\_GPO\_Package\_April\_2025\\DoD WinSvr 2022 MS and DC v2r4"

And you should see the completion of the GPO import with the LGPO tool:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/06/GPO-Applied-CMD-1024x643.png)

Since we have applied all these policies, lets restart the server.

Now if you find yourself locked out of the Windows Server administrator account. You may want to review the GPO you just imported. :D

The Adminstrator account was renamed "X\_Admin"

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/06/Administrator-Password-1024x610.png)

Hopefully you didn't get locked out for long, if at all - I didn't really go into the details of setting up your virtual machine. Mine was only setup with an Administrator account.  
  
Lets jump back to the SCAP tool within the scc-5.10.2 folder (scc.exe) and run another scan on the system.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/06/SCAP-Scan-Compliance-1024x643.png)

Compliance is now at 96.88% \[GREEN\]. So that is considered a passing score for this Windows Server 2022 domain controller benchmark when using the SCAP Compliance Checker tool.

If you are curious about this score - details are describe in the **[NIST SP 800-126 Rev. 3](https://csrc.nist.gov/pubs/sp/800/126/r3/final)**:

> Measurement and scoring systems. In SCAP this refers to evaluating specific characteristics of a security weakness (for example, software vulnerabilities and security configuration issues) and, based on those characteristics, generating a score that reflects their relative severity. The SCAP measurement and scoring system specifications are Common Vulnerability Scoring System (CVSS) and Common Configuration Scoring System (CCSS).

But is that all that can be done you may be asking? No, not at all - the rabbit hole can go deeper! Now that we ran our SCAP tool benchmark, we can look deeper into the STIGs with STIG Viewer. **_Looking at my Desktop screenshots - I see I left a folder out!_**

**[U\_SRG-STIG\_Library\_April\_2025](https://public.cyber.mil/stigs/compilations/)**

That file should have been next to the STIG Viewer folder the whole time because we will now need it (top row below). Open up the STIG Viewer and go to the top - under **File > Import STIG** - then locate the **_U\_MS\_Windows\_Server\_2022\_V2R4\_STIG.zip_** in the folder we just added to the top row on the Desktop. It should appear in the STIG Viewer after you open it as seen below:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/06/STIG-Viewer-1024x636.png)

Now you need to go up to **Checklist > Create Checklist - Check Marked STIG(s)** and then you should see a tab appear called **\*New Checklist** and then click on **File >** **Save Checklist As...**  
  
You can save it to the folder on the Desktop called "Checklist STIG" - I called my checklist "Windows Server 2022 Benchmark"

Next go up and click on **Import > XCCDF Results File...** in the STIG Viewer menu and navigate to the folder "SCAP Output" - search for XCCDF to make your search simpler, then select the file with **_XCCDF-Results\_MS\_Windows\_Server\_2022\_STIG_** in the name. You should then see some color coding on your STIGs in the center. This will aid in the investigation of compliant and non-compliant CAT I, CAT II, and CAT III vulnerabilities.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/06/XCCDF-Import-1024x642.png)

This is where the rabbit hole deepens! Each one of those green **Vul IDs** contain issues that must be addressed and documented to ensure your network/assets have authority to operate (ATO). They contain a lot of different modifications that must be made to the system you are checking for compliance - and that could take a whole other post to document that. But if you are already attuned to editing the registry you can handle that pretty easy.

**Evaluate-STIG** (compliance, the popular way)

Evaluate-STIG is simply a Powershell script that automates a lot of the process of STIG compliance. According to an article at [NAVSEA](https://www.navsea.navy.mil/Media/News/article/1946720/nswc-crane-employee-develops-software-tool-to-increase-cybersecurity-cost-avoid/), "Dan Ireland combined and expanded sample code provided by colleagues Nick Hurley and Rickey Beem to create the Evaluate-STIG (Security Technical Implementation Guide) tool, a Windows Powershell tool with the ability to highly automate the process of documenting system compliance."

Powershell excellence is something to applaud. Many of us struggle through it on the daily, lol! As seen at the top of the page (blue box), you can [download](https://intelshare.intelink.gov/sites/NAVSEA-RMF) the tool if you have a CAC from the NAVSEA Intelshare site. As seen below I have put the ZIP and corresponding hash file on the desktop and checked the hash with the following command:

certutil -hashfile C:\\Users\\Administrator\\Desktop\\Evaluate-STIG\_1.2504.2.zip SHA256

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/06/Evaluate-STIG-Hash-1024x636.png)

The hash checks out with the SHA256 in the text file provided by the NAVSEA team. Now we can begin the work! Inside the Evaluate-STIG folder is a nice **_User Guide_** that spells out a lot of the commands - definitely read over that if you can download the file. The one I am using is supplied in the guide:

.\\Evaluate-STIG.ps1 -ScanType Unlcassified -Output CKL,CKLB -AllowDeprecated

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/06/Evalute-STIG-Command-1024x220.png)

The command above performs an Unclassified (default) scan, saving the results as both **.CKL** and **.CKLB** files, and enables deprecated STIG scanning.

As you can see, the Powershell script runs pretty effortlessly. It was tailored to run this way so you can push the script out to different assets and execute locally (or remotely) via Powershell or a BASH script.

Going to the folder where the CKL/CKLB files were saved to we can double click and open our a CKL file in STIG Viewer to see compliance levels.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/06/Evaluate-STIG-CKL-CKLB-1024x642.png)

I chose the WinServer2022\_V2R4 CKL file. You may have to find the STIG Viewer program after you double click on a CKL. Below you can see how the CKL looks after you open it. You can sort through all the different Vulnerabilities and fix them as applicable.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/06/WinServer2022_V2R4-CKL-1024x642.png)

Much of the documentation side I have not gone into for this post on STIG compliance, such as Answer Files, Non-Finding exceptions - but just know it is crucial for ATO. There is just so much to cover for these tools, documentation and compliance - more than I can put together in one post! For a basic overview this should suffice!

And more importantly I feel this is my gateway opportunity into Ansible. As seen on the [RedHat blog](https://www.redhat.com/en/blog/disa-releases-first-ansible-stig), DISA released the first Ansible STIG in 2023. So I am quite behind on getting to Ansible and now is the time!!  
  
I've already spun up two control nodes for Ansible - one RHEL 9.5 VM and one Ubuntu 22.04 container in Proxmox:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/06/Proxmox-Ansible-VMs-1024x711.png)

See the [DoD Cyber Exchange](https://public.cyber.mil/stigs/supplemental-automation-content/) for a few of the options to utilize with Ansible. Now it's time to figure out this network setup and get to learning about Playbooks!
