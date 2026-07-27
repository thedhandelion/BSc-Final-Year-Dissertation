This is a write-up of my final year dissertation project for my Cyber Security and Digital Forensics (BSc Hons) degree, which was awarded a First Class mark of 80%.

# Project Overview
"A Critical Study of Misconfiguration-Driven Security and Forensic Risks in Web-Based POS Deployments for SMEs".

Can small businesses accidentally leave customer data exposed without ever being hacked?

This project explored how common security misconfigurations within web-based point-of-sale (POS) systems could lead to the unintended persistence of sensitive transactional data. Rather than focusing on active exploitation, I investigated where data remains after normal operation and how misconfigurations increase both security and forensic risk. 

Many Small and Medium Enterprises (SMEs) process card payment data every day, but they often lack the technical expertise and resources to deploy securely-configured systems. With standards such as PCI DSS, how could misconfigurations cause compliance issues and what implications do these have for SMEs?

After reviewing existing literature, I conducted an experiment in a controlled web-based POS environment and implemented selected misconfiguration scenarios, assessing the extent to which each misconfiguration caused the remanence of card payment data.

# Background and Motivation
I've had an interest in digital forensics long before I developed a passion for cyber security, and this project was the perfect opportunity to use the knowledge I'd acquired throughout my Cyber Security and Digital Forensics degree to combine the two. 

Quite often, research projects view security from an exploitation perspective, 
`"How can payment data be intercepted if transmitted via HTTP?"`, whereas, for this project, I wanted to shift the perspective to forensics, 
`"How can payment data be inadvertently logged in a SIEM if transmitted via HTTP? What implications could this have on a business?"`.

This project originated as a study assessing the security of commercially available mobile POS (mPOS) terminals, which can be bought off-the-shelf, configured and used by a single, non-technical individual. However, after identifying several ethical considerations and research constraints, I refined the focus to SME POS environments. This shift gave me the opportunity to pursue a more impactful study that actually allowed me to integrate my passion for digital forensics. 

That said, this shift meant I was starting afresh two months into the project, leaving me with only four months to complete it instead of the original six, which only made my sense of achievement even greater when it was awarded 80%!

# Research Questions
1. What misconfiguration-related security risks are most prominent in web-based POS deployments used by SMEs?
2. How do existing guidance and compliance frameworks, including Cyber Essentials, OWASP, and PCI DSS, address these risks and where do practical gaps remain for SMEs?
3. To what extent do selected misconfiguration scenarios lead to recoverable residual artefacts in a controlled open-source POS test environment?

# Methodology
This project began with a literature review, in which I analysed not just secondary research, but the currently-available guidance and compliance frameworks and the root causes of misconfigurations in the first place, which will be discussed further in _Framework Evaluation_.

The second part of this project involved a test environment using an open-source POS web app, which allowed me to configure a secure baseline, then implement common misconfigurations one-by-one to examine how and where these cause data remanence after normal operational use.

## Test Environment
For this project, I used two virtual machines (VMs):
- A standard LAMP stack: Ubuntu Server, acting as a lightweight web server (Apache) and database (MySQL), with PHP for the scripting layer. This is the backend server that hosts the web application.
- A thin client: Low-resource Windows 10 VM connected to the Ubuntu Server via LAN. This simulates a shop-floor POS terminal and accesses the OSPOS web app via the web browser.

During the set up process, I had configured everything to a secure baseline, and took a snapshot of each VM. The experiment would involve performing three test transactions in the secure environment and examining potential data remanence locations to verify that no 'sensitive' data would be retrievable. After doing so, I would revert the VMs back to their secure baselines, implement a misconfiguration, perform three further test transactions, and examine those same predicted artefact source locations for data remanence. I would revert back to the secure baseline snapshot prior to every misconfiguration test.

As for the 'sensitive' transactional data I'd be analysing, OSPOS had, unfortunately, lacked the capability to take card payments, and so I repurposed a 'Comments' field in the POS terminal to accept the PAN (Primary Account Number - 16 digit number on the back of a bank card). For the test payments, I used a synthetic PAN, '4111 1111 1111 1111', which is a common Visa test card number. Obviously, using a real payment card for this project would be unethical and insecure, plus the synthetic PAN is easily-recognisable, which is why I did not use a real PAN for this experiment.

# Results and Key Findings
I tested five different misconfiguration scenarios across different areas of the environment:
1. Debug Mode and Verbose Logging - Server-side
2. Insecure Transport             - Network layer
3. Session Storage Misuse         - Client-side
4. Lack of Encryption at Rest     - Server-side
5. Improper Invalidation          - Server-side

## Results

| Scenario | Implementation | Artefact Location | Retrieval Process |
| :----: | :----: | :----: | :----: |
| Debug Mode & Verbose Logging | Debug Mode Enabled, Logging Threshold Increased | Application Logs (3 files containing PAN) | `Grep` on log directory |
| Insecure Transport | HTTPS disabled | Plaintext PAN in network packets | Wireshark capture and packet analysis |
| Session Storage Misuse | PAN stored in client browser's local storage | Browser storage | Chrome developer tools |
| Lack of Encryption at Rest | PAN stored in database unencrypted | Database table and raw InnoDB file | SQL queries and `Strings` |
| Improper Invalidation | Row "deleted" with SQL | Still recoverable in underlying InnoDB file | File system analysis |

## Key Findings
There were two key findings from these tests. Firstly, and most importantly, cryptography is one of the most important factors when securely configuring systems, as the severity of many misconfigurations would be drastically decreased if sufficient encryption mechanisms were implemented. For example, the effects of verbose logging (sensitive data in log files) and improper invalidation ("deleted" data persisting in underlying storage) would be far less detrimental if the 'sensitive data' was encrypted, as it would not only be harder to identify, but also much harder to exfiltrate, thus keeping an SME compliant with regulations and standards like GDPR and the PCI DSS requirements.

The second key finding is that "deleted" does not necessarily mean 'securely erased', as shown when the 'deleted' PAN data from the database table was still present in underlying storage. This misconception is common amongst non-technical users and, in SME environments, can cause compliance issues. What is also interesting is that this result was inconsistent, and data recovery was dependant on subsequent database usage. Inconsistent results are certainly worth noting as a single configuration/operational audit may miss insecure practices, and operation may continue with said insecure practices going unnoticed. This emphasises the importance of regular audits and security checks.

# Framework Evaluation
During the literature review stage of my dissertation, I evaluated four sources of formal guidance that are available to SMEs: NCSC's Cyber Essentials, OWASP Top 10, OWASP ASVS, and PCI DSS. 

Overall, I found Cyber Essentials to be too broad, covering a range of basic vulnerabilities that are useful for beginners, but lacks configuration-focused guidance that extends beyond changing default passwords and securing firewalls.  
OWASP, on the other hand, is far more extensive, and provides not only a list of common vulnerabilities through its Top 10, but links those vulnerabilities to their Application Security Verification Standard (ASVS), which provides a comprehensive review checklist that can be used for securing web applications. However, a non-technical SME would struggle to use ASVS alone and secure a web application, and if they try, the likelihood of misconfigurations remains high. 
Finally, I looked at PCI's Data Security Standard (PCI DSS), which provides security requirements rather than technical guidance and, credit where credit's due, this is comprehensive and a valuable source of information. However, from an SME-standpoint, these requirements may not be easily understood by smaller businesses and achieving compliance may present further technical challenges for SMEs.  
In conclusion, I found that while no single source provides full guidance coverage, each offers different forms of information and guidelines. Therefore, in my opinion, the best alternative for SMEs would be to combine all four frameworks, bridging the gaps and providing broad cyber security understanding, compliance requirements, and control checklists. That said, this could just introduce further complexity and overburden small businesses, but it's the strongest solution to the underlying problem: most frameworks are conceptual and don't offer practical implementation guidance.

# Limitations
Put plainly; this project is not as strong as it could have been, for several reasons:
- I used one POS platform
- I managed to choose one that doesn't actually take card payment data and repurposed application fields don't reflect the behaviour of POS deployments with transaction-handling capabilities
- I used synthetic PAN data - necessary for ethical reasons, but not representative of real-world operation
- I used a small set of five misconfiguration scenarios

Overall, the results should be interpreted as indicative rather than definitive, and more could have been done to have strengthened this project... 

# My Thoughts
...That said, I can't help but be proud of what I did achieve, and I feel that I have identified a real research gap. Most importantly, I am truly pleased to have found a project I was genuinely interested in, and am still very passionate about. I am looking forward to looking deeper into this area, performing more (and better) experiments and tests to see what more I can find. 

In the future, I hope to build onto this project, running more tests on multiple POS platforms (including proprietary systems and those with transaction-handling capabilities!), testing more misconfigurations, and examining more artefact sources (such as short-term volatile memory, long-term persistence, and data carving). I really hope to use this first project as a foundation to build something truly impactful in the future.

# Additional Information
## Tools Used
- VMWare
- Ubuntu Server
- Windows 10
- Apache, MySQL, PHP
- OSPOS
- Wireshark

## Degree and Grade
University of the West of England  
Bachelor of Science with Honours  
Cyber Security and Digital Forensics  
Dissertation Classification: First Class (80%)  
