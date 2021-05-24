# WAF - Getting started with WAF, Bot Detection and Threat Campaigns

[Original Lab Guide](https://clouddocs.f5.com/training/community/waf/html/waf141/labinfo/labinfo.html)

This class will focus on a best practice approach to getting started with F5 WAF and application security. This introductory class will give you guidance on deploying WAF services in a successive fashion. This 141 class focuses entirely on the negative security model aspects of WAF configuration.  
This is the 1st class in a three part lab series (141,241,341) based on: [Succeeding with Application Security](https://support.f5.com/csp/article/K07359270) which closely maps to this visualization of layered Application Security.

![image](https://user-images.githubusercontent.com/38420010/119345353-0da70580-bc99-11eb-94fe-59eabaf3c240.png)

## Lab Environment & Topology

`All work is done from the Linux Client, which can be accessed via RDP (Remote Desktop Client) or SSH. No installation or interaction with your local system is required.`

### Environment
  
Linux Client:  
  
  * BURP Community Edition - Packet Crafting
  * curl - command line webclient. Very useful for debugging and request crafting
  * Postman - API Development and request crafting
  
Linux server:  
  
  * WebGoat - deliberately insecure application that allows interested developers just like you to test vulnerabilities commonly found in Java-based applications

### Lab Topology
  
The network topology implemented for this lab is very simple. The following components have been included in your lab environment:

1 x Ubuntu Linux 20.04 client.  
1 x F5 BIG-IP VE (v16.0.1) running Advanced WAF with IP Intelligence & Threat Campaign Subscription Services.  
1 x Ubuntu Linux 20.04 server.  


## Exercise 1.1: IP Intelligence Policies
### Objective

  * Configure Global IPI Profile & Logging
  * Review Global IPI Logs
  * Configure Custom Category and add an IP
  * Create your first WAF Policy and implement IPI w/ XFF inspection
  * Estimated time for completion: 30 minutes.

### Create Your 1st L3 IPI Policy
An IPI policy can be created and applied globally, at the virtual server (VS) level or within the WAF policy itself. We will follow security best-practice by applying IPI via a Global Policy to secure Layer 3 device-wide and within the Layer 7 WAF policy to protect the App by inspecting the HTTP X-Forwarded-For Header.  
![image](https://user-images.githubusercontent.com/38420010/119348500-3e893980-bc9d-11eb-8836-e57471dc73a8.png)

In this first lab, we will start by enabling a Global IPI Policy; much like you would do, as a day 1 task for your WAF:  
  
1. RDP to the Linux Client by choosing the RDP access method from your UDF environment page. You will be presented with the following prompt where you will enter the password only. Please use f5student username:

![image](https://user-images.githubusercontent.com/38420010/119348695-83ad6b80-bc9d-11eb-84a6-ad49b8747f0c.png)

2. Once logged in, launch Chrome Browser. You can double-click the icon or right click and choose execute but do not click multiple times. It does take a few moments for the browser to launch the first time.
3. Click the bigip01 bookmark and login to TMUI. It is normal to see a certificate warning that you can safely click through. Or you can use TMUI access via UDF.
4. On the Main tab, click Local Traffic > Virtual Servers and you will see the Virtual Servers that have been pre-configured for your lab. Essentially, these are the listening IP’s that receive requests for your application and proxy the requests to the backend “real” servers.

![image](https://user-images.githubusercontent.com/38420010/119349079-020a0d80-bc9e-11eb-91ed-d77da464c136.png)

  * **insecureApp1_vs** - Main Site - Status of green indicates a healthy backend pool of real servers
  * **security-testing-overlay-vs** - Will be used later to send spoofed traffic to the main site

5. On the Main tab, click Security > Network Firewall > IP Intelligence > Policies.
![image](https://user-images.githubusercontent.com/38420010/119349565-94121600-bc9e-11eb-9588-d95a3e714f33.png)

6. Click on the **Create** button.
7. For the name: **global_ipi**
8. Under **IP Intelligence Policy Properties** For the Default Log Action choose **yes** to **Log Category Matches.**
9. Browse to the inline Help tab at the top left of the GUI and examine the Default Log Action settings. Inline help is very useful when navigating the myriad of options available within any configuration screen.
10. To the right of the screen, click **Add** under the categories section.
11. Repeat this process and add the following additional categories: **phishing, scanners, spam_sources, & denial_of_service**. Outside of this lab, you would want to enable additional categories for protection.

![image](https://user-images.githubusercontent.com/38420010/119351114-69c15800-bca0-11eb-9264-d5ebeeca6c1e.png)

12. Commit the Changes to the System.
13. Under **Global Policy Assignment > IP Intelligence Policy** click on the dropdown and select the **global_ipi** policy and click Update.

### Setup Logging for Global IPI
1. In the upper left of the GUI under the **Main** tab, navigate to **Security > Event Logs > Logging Profiles** and click on **global-network**
2. Under the Network Firewall section configure the IP Intelligence publisher to use **local-db-publisher**
3. Check **Log GEO Events**
4. Click **Update**
![image](https://user-images.githubusercontent.com/38420010/119351448-bf960000-bca0-11eb-8704-5fd25bfb1f29.png)

### Test
1. On the Linux Client, open a terminal and **cd** to **Agility2020wafTools**
2. Run the following command to send some traffic to the site: **./ipi_tester**
`The script should continue to run for the remainder of Lab 1 & 2. Do NOT stop the script.`
3. Navigate to **Security > Event Logs > Network > Ip Intelligence** and review the entries. Notice the Geolocation Data as well as the Black List Class to the right of the log screen.

![image](https://user-images.githubusercontent.com/38420010/119351937-5b277080-bca1-11eb-882e-bb0d893f125b.png)

### Create Custom Category
1. Navigate to: **Security > Network Firewall > IP Intelligence > Blacklist Categories** and click **Create**.
2. Name: **my_bad_ips** with a match type of **Source**
3. Click **Finished**
4. Click the checkbox next to the name **my_bad_ips** and then at the bottom of the GUI, click **Add To Category**.
![image](https://user-images.githubusercontent.com/38420010/119352108-8d38d280-bca1-11eb-8734-52f48cb89621.png)
5. Enter the ip address: **134.119.218.243** or any of the other malicious IP’s showing up in the IP Intelligence logs, and set the seconds to **3600** (1 hour)
6. Click **Insert Entry**
7. Navigate to **Security > Network Firewall > IP Intelligence > Policies** and click **global_ipi**
8. Under **Categories** click **Add** and select your new custom category **my_bad_ips** from the drop-down. Click **Done Editing** and **Commit Changes to System**.
![image](https://user-images.githubusercontent.com/38420010/119352376-e6086b00-bca1-11eb-8c5a-8213f0ea3e55.png)
9. Navigate back to **Security > Event Logs > Network > Ip Intelligence** and review the entries under the column **Black List Class**. You will see entries for your custom category **my_bad_ips**.
![image](https://user-images.githubusercontent.com/38420010/119352409-f1f42d00-bca1-11eb-9885-2f5c43833c35.png)

**This concludes the Layer 3 IPI policy lab section.**  
  
**To recap, you have just configured a Global IP Intelligence policy and added a custom category.
This policy is inspecting Layer 3 only and is a best-practice first step to securing your Application traffic.**  
  
**We will now configure a Layer 7 WAF policy to inspect the X-Forwarded-For HTTP Header.**

### Create your WAF Policy
1. Navigate to **Security > Application Security > Security Policies** and click the Plus (+) button.
2. Name the policy: **webgoat_waf**
3. Select Policy Template: **Rapid Deployment Policy** (accept the popup)
4. Select Virtual Server: **insecureApp1_vs**
5. Logging Profiles: **Log all requests**
6. Notice that the Enforcement Mode is already in **Transparent Mode** and Signature Staging is **Enabled**
7. Click **Save**.
![image](https://user-images.githubusercontent.com/38420010/119353162-d76e8380-bca2-11eb-90bb-0daf3cd54a9a.png)

### Configure L7 IPI
1. Navigate to **Security > Application Security > Policy Building > Learning and Blocking Settings** and expand the **IP Addresses and Geolocations** section.
`These are the settings that govern what happens when a violation occurs such as Alarm and Block. We will cover these concepts later in the lab but for now the policy is still transparent so the blocking setting has no effect.`
![image](https://user-images.githubusercontent.com/38420010/119355305-64b2d780-bca5-11eb-8102-9f4e614507e7.png)
2. Navigate to **Security > Application Security > IP Addresses > IP Intelligence** and enable **IP Intelligence** by checking the box.
3. Notice at the top left drop-down that you are working within the webgoat_waf policy context. Enable **Alarm** and **Block** for each category.
![image](https://user-images.githubusercontent.com/38420010/119355409-84e29680-bca5-11eb-8572-5f8809e3ae8e.png)
4. Click **Save** and **Apply Policy**. You will get an “Are you sure” popup that you can banish by clicking **Do not ask for this confirmation again**.
5. Enable XFF inspection in the WAF policy by going to **Security > Application Security > Security Policies > Policies List** > and click on webgoat_waf policy.
6. Finally, scroll down under **General Settings** and click **Enabled** under **Trust XFF Header**.
7. Click **Save** and **Apply Policy**

### Test XFF Inspection
1. Open a new terminal or terminal tab on the Client (the ipi_tester script should still be running) and run the following command to insert a malicious IP into the XFF Header:

```bash
curl -H "X-Forwarded-For: 134.119.218.243" -k https://juiceshop.f5agility.com/xff-test
```
If that IP has rotated out of the malicious DB, you can try one of these alternates:
* 80.191.169.66 - Spam Source
* 85.185.152.146 - Spam Source
* 220.169.127.172 - Scanner
* 222.74.73.202 - Scanner
* 62.149.29.36 - Spam Source
* 82.200.247.241 - Phishing
* 134.119.219.93 - Spam Source
* 218.17.228.102 - Spam Source
* 220.169.127.172 - Scanner



























