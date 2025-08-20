---
title: Penetration Testing(PEN TESTING)
image: https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSGYguzr-1PQ14veXvw8NG9L6a7ESoxzA_HSg&s
tags: [ cybersecurity, cloud, containers ,]
categories: [Security]
---

# PENETRATION TESTING

## Definition

A penetration test (pen test) is an authorized simulated attack performed on a computer system to evaluate its security. Penetration testers use the same tools, techniques, and processes as attackers to find and demonstrate the business impacts of weaknesses in a system. Penetration tests usually simulate a variety of attacks that could threaten a business. They can examine whether a system is robust enough to withstand attacks from authenticated and unauthenticated positions, as well as a range of system roles. With the right scope, a pen test can dive into any aspect of a system.

## What are the benefits of penetration testing?

Ideally, software and systems were designed from the start with the aim of eliminating dangerous security flaws. A pen test provides insight into how well that aim was achieved. Pen testing can help an organization

- Find weaknesses in systems
- Determine the robustness of controls
- Support compliance with data privacy and security regulations (e.g., PCI DSS, HIPAA, GDPR)
- Provide qualitative and quantitative examples of current security posture and budget priorities for management

![alt text](https://media.giphy.com/media/v1.Y2lkPTc5MGI3NjExMzE2dmtwd3FseTl0ZzI3MXFhb2p2OHJwMDR6OWEwbGdoZG51N3JzciZlcD12MV9naWZzX3NlYXJjaCZjdD1n/ECWfWVzZpEiDWwdPET/giphy.gif)

## Penetration Testing

### How much access is given to pen testers?

Depending on the goals of a pen test, testers are given varying degrees of information about, or access to, the target system. In some cases, the pen testing team takes one approach at the start and sticks with it. Other times, the testing team evolves its strategy as its awareness of the system increases during the pen test. There are three levels of pen test access:

- **Opaque box**. The team doesn’t know anything about the internal structure of the target system. It acts as hackers would, probing for any externally exploitable weaknesses.
- **Semi-opaque box**. The team has some knowledge of one or more sets of credentials. It also knows about the target’s internal data structures, code, and algorithms. Pen testers might construct test cases based on detailed design documents, such as architectural diagrams of the target system.
- **Transparent box**. Pen testers have access to systems and system artifacts including source code, binaries, containers, and sometimes even the servers running the system. This approach provides the highest level of assurance in the smallest amount of time.

### What are the phases of pen testing?

Pen testers simulate attacks by motivated adversaries. To do this, they typically follow a plan that includes the following steps:

- **Reconnaissance**. Gather as much information about the target as possible from public and private sources to inform the attack strategy. Sources include internet searches, domain registration information retrieval, social engineering, nonintrusive network scanning, and sometimes even dumpster diving. This information helps pen testers map out the target’s attack surface and possible vulnerabilities. Reconnaissance can vary with the scope and objectives of the pen test; it can be as simple as making a phone call to walk through the functionality of a system.
- **Scanning**. Pen testers use tools to examine the target website or system for weaknesses, including open services, application security issues, and open source vulnerabilities. Pen testers use a variety of tools based on what they find during reconnaissance and during the test.
- **Gaining access**. Attacker motivations can include stealing, changing, or deleting data; moving funds; or simply damaging a company’s reputation. To perform each test case, pen testers determine the best tools and techniques to gain access to the system, whether through a weakness such as SQL injection or through malware, social engineering, or something else.
- **Maintaining access**. Once pen testers gain access to the target, their simulated attack must stay connected long enough to accomplish their goals of exfiltrating data, modifying it, or abusing functionality. It’s about demonstrating the potential impact.

### What are the types of pen testing?

A comprehensive approach to pen testing is essential for optimal risk management. This entails testing all the areas in your environment:

- **Web apps**. Testers examine the effectiveness of security controls and look for hidden vulnerabilities, attack patterns, and any other potential security gaps that can lead to a compromise of a web app.
- **Mobile apps**. Using both automated and extended manual testing, testers look for vulnerabilities in application binaries running on the mobile device and the corresponding server-side functionality. Server-side vulnerabilities include session management, cryptographic issues, authentication and authorization issues, and other common web service vulnerabilities.
- **Networks**. This testing identifies common to critical security vulnerabilities in an external network and systems. Experts employ a checklist that includes test cases for encrypted transport protocols, SSL certificate scoping issues, use of administrative services, and more.
- **Cloud**. A cloud environment is significantly different than traditional on-premises environments. Typically, security responsibilities are shared between the organization using the environment and the cloud services provider. Because of this, cloud pen testing requires a set of specialized skills and experience to scrutinize the various aspects of the cloud, such as configurations, APIs, various databases, encryption, storage, and security controls.
- **Containers**. Containers obtained from Docker often have vulnerabilities that can be exploited at scale. Misconfiguration is also a common risk associated with containers and their environment. Both of these risks can be uncovered with expert pen testing.
- **Embedded devices (IoT)**. Embedded / Internet of Things (IoT) devices such as medical devices, automobiles, in-home appliances, oil rig equipment, and watches have unique software testing requirements due to their longer life cycles, remote locations, power constraints, regulatory requirements, and more. Experts perform a thorough communication analysis along with a client/server analysis to identify defects that matter most to the relevant use case.
- **Mobile devices**. Pen testers use both automated and manual analysis to find vulnerabilities in application binaries running on the mobile device and the corresponding server-side functionality. Vulnerabilities in application binaries can include authentication and authorization issues, client-side trust issues, misconfigured security controls, and cross-platform development framework issues. Server-side vulnerabilities can include session management, cryptographic issues, authentication and authorization issues, and other common web service vulnerabilities.
- **APIs**. Both automated and manual testing techniques are used to cover the OWASP API Security Top 10 list. Some of the security risks and vulnerabilities testers look for include broken object level authorization, user authentication, excessive data exposure, lack of resources / rate limiting, and more.
- **CI/CD pipeline**. Modern DevSecOps practices integrate automated and intelligent code scanning tools into the CI/CD pipeline. In addition to static tools that find known vulnerabilities, automated pen testing tools can be integrated into the CI/CD pipeline to mimic what a hacker can do to compromise the security of an application. Automated CI/CD pen testing can discover hidden vulnerabilities and attack patterns that go undetected with static code scanning.

### What are the types of pen testing tools?

There is no one-size-fits-all tool for pen testing. Instead, different targets require different sets of tools for port scanning, application scanning, Wi-Fi break-ins, or direct penetration of the network. Broadly speaking, the types of pen testing tools fit into five categories:

- Reconnaissance tools for discovering network hosts and open ports
- Vulnerability scanners for discovering issues in-network services, web applications, and APIs
- Proxy tools such as specialized web proxies or generic man-in-the-middle proxies
- Exploitation tools to achieve system footholds or access to assets
- Post exploitation tools for interacting with systems, maintaining and expanding access, and achieving attack objectives

### How does pen testing differ from automated testing?

Although pen testing is mostly a manual effort, pen testers do use automated scanning and testing tools. But they also go beyond the tools and use their knowledge of the latest attack techniques to provide more in-depth testing than a vulnerability assessment (i.e., automated testing).

- **Manual pen testing**. Manual pen testing uncovers vulnerabilities and weaknesses not included in popular lists (e.g., OWASP Top 10) and tests business logic that automated testing can overlook (e.g., data validation, integrity checks). A manual pen test can also help identify false positives reported by automated testing. Because pen testers are experts who think like adversaries, they can analyze data to target their attacks and test systems and websites in ways automated testing solutions following a scripted routine cannot.
  
- **Automated testing**. Automated testing generates results faster and needs fewer specialized professionals than a fully manual pen testing process. Automated testing tools track results automatically and can sometimes export them to a centralized reporting platform. Also, the results of manual pen tests can vary from test to test, whereas running automated testing repeatedly on the same system will produce the same results.

### What are the pros and cons of pen testing?

With the frequency and severity of security breaches increasing year after year, organizations have never had a greater need for visibility into how they can withstand attacks. Regulations such as PCI DSS and HIPAA mandate periodic pen testing to remain current with their requirements. With these pressures in mind, here are some pros and cons for this type of defect discovery technique:

#### Pros of pen testing

- Finds holes in upstream security assurance practices, such as automated tools, configuration and coding standards, architecture analysis, and other lighter-weight vulnerability assessment activities
- Locates both known and unknown software flaws and security vulnerabilities, including small ones that by themselves won’t raise much concern but could cause material harm as part of a complex attack pattern
- Can attack any system, mimicking how most malicious hackers would behave, simulating as close as possible a real-world adversary

#### Cons of pen testing

- Is labor-intensive and costly
- Does not comprehensively prevent bugs and flaws from making their way into production


**Thank You for Your Support!**  
Please consider showing your support if this tutorial has been helpful to you in your microservices journey. Your support means a lot to me and keeps me motivated to keep learning and developing.


[![normanf](https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png)](https://www.buymeacoffee.com/normanf)
