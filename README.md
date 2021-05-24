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

# Module 1: IP Intelligence
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
curl -H "X-Forwarded-For: 134.119.218.243" -k https://10.1.10.145/xff-test
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

2. Navigate to **Security > Event Logs > Application > Requests** and review the entries. You should see a Sev3 Alert for the attempted access to uri: **/xff-test** from a malicious IP.
![image](https://user-images.githubusercontent.com/38420010/119356456-bb6ce100-bca6-11eb-91e7-316e32571e48.png)

3. In the violation details you can see the entire request details including the XFF Header even though this site was using strong TLS for encryption.

`Attackers often use proxies to add in source IP randomness. Headers such as XFF are used to track the original source IP so the packets can be returned. In this example the HTTP request was sent from a malicious IP but through a proxy that was not known to be malicious. The request passed right through our Global Layer 3 IPI policy but was picked up at Layer 7 due to the WAF’s capabilities. This demonstrates the importance of implementing security in layers.`

## Exercise 1.2: Add a Geolocation Policy
Another practical control to implement early on in your WAF deployment is Geolocation blocking or fencing. If we know that our application is only supposed to be accessed from certain countries or not accessed from others, now is the time to get that configured and enforced.

`Much like our Layer 7 IPI Policy, with Advanced WAF the Geolocation logic happens at the policy level. You may have many policies each with their own unique configuration per application or you may use a parent policy that has baseline settings.`

### Geolocation
**For demonstration purposes you will now disable the Layer 3 Global IPI policy to ensure Layer 7 Geolocation & IPI events occur.**
1. Browse to **Security > Network Firewall > IP Intelligence > Policies** and set the Global Policy Assignment to **None** and click **Update**.
2. Open **Security > Application Security > Geolocation Enforcement**
3. Select all Geolocations **except the United States and N/A** and move them to Disallowed Geolocations. **Save** and then **Apply Policy**.
![image](https://user-images.githubusercontent.com/38420010/119356800-1ef70e80-bca7-11eb-8486-093801f0641e.png)
`N/A covers all RFC1918 private addresses. If you aren’t dropping them at your border router (layer 3), you may decide to geo-enforce at ASM (Layer 7) if no private IP’s will be accessing the site.`
4. Navigate to **Security > Event Logs > Application > Requests** and review the entries in the event log that contain both IPI and Geolocation violations.
![image](https://user-images.githubusercontent.com/38420010/119356892-3afab000-bca7-11eb-921e-0ff198b705b8.png)
`You can also perform Geolocation Enforcement with LTM policies attached to Virtual Servers even if you are only licensed for Advanced WAF. Blocking decisions made here would not be reflected in the Application Requests WAF Log but can be still be logged.`


**This completes Exercise 1.2

Congratulations! You have just completed Lab 1 by implementing an IPI policy globally at Layer 3 and at Layer 7 via WAF policy for a specific application. Next you added Geolocation Enforcement to the policy and learned that this can be done via WAF policy or LTM policy. This follows our best-practice guidance for getting started with Application Security.**

# Module 2: Bot Defense
## Exercise 2.1: Bot Defense with Signatures¶

The next logical step in our configuration is to deal with automated traffic. While Advanced WAF has some deep Bot Defense capabilities, we will start with Bot Signatures. A good goal during your initial deployment would be to get transparent BOT profiles deployed across your various application Virtual Servers so you can start to analyze your “normal” loads of automated traffic. This can be very surprising to an organization or a developer that thought they had a lot more “real users”.

### Objective
  * Create a Bot Defense logging profile
  * Create and apply a transparent Bot Defense Profile with Signatures
  * Test and verify logs
  * Add a signature to the whitelist
  * Estimated time for completion: **20 minutes**

### Create Logging Profile
1. Navigate to **Security > Event Logs > Logging Profiles** and click **Create** to a new Logging Profile with the settings shown in the screenshot below. Click **Create**.
![image](https://user-images.githubusercontent.com/38420010/119357266-a775af00-bca7-11eb-9473-232baace0399.png)
2. Navigate to **Security > Bot Defense > Bot Defense Profiles** and click **Create**.
3. Name: **webgoat_bot**
4. Profile Template: **Relaxed**
5. Click the Learn more link to see an explanation of the options. These will be explored further in the 241 lab but for now we are going with **Relaxed** aka **Challenge-Free Verification**.
![image](https://user-images.githubusercontent.com/38420010/119357409-d429c680-bca7-11eb-8889-27572035d2e2.png)
6. Click on the **Bot Mitigation Settings** tab and review the default configuration.
7. Click on the **Signature Enforcement** tab and review the signatures and staging status.
8. Click **Save**.

### Apply the Policy and Logging Profile
1. Navigate to **Local Traffic > Virtual Servers** click on **insecureApp1_vs** then go to the **Security Tab > Policies** (top middle of screen).
`To clearly demonstrate just the Bot Defense profile, please disable all security policy on the virtual server. The ipi_tester script should still be running!`
2. Navigate to **Local Traffic > Virtual Servers > insecureApp1_vs > Security > Policies** and disable the **Application Security Policy** and **enable the Bot Defense Profile** and **Bot_Log Profile**.
3. Click **Update**

![image](https://user-images.githubusercontent.com/38420010/119358317-c759a280-bca8-11eb-9725-f60b6ddbe1c4.png)

4. Navigate to **Security > Event Logs > Bot Defense > Bot Requests** and review the event logs. Notice curl (the bot being used in our ipi_tester script) is an untrusted bot in the HTTP Library category of Bots.

![image](https://user-images.githubusercontent.com/38420010/119358572-0556c680-bca9-11eb-812f-248f097248fb.png)

5. On the top middle of the screen under the **Bot Defense** Tab, click on **Bot Traffic** for a global view of all Bot Traffic. In this lab we only have one site configured.

![image](https://user-images.githubusercontent.com/38420010/119358885-55ce2400-bca9-11eb-920e-8bfd37fee102.png)

6. Click on the **insecurApp1_vs** Virtual Server and explore the analytics available under **View Detected Bots** at the bottom of the screen.

![image](https://user-images.githubusercontent.com/38420010/119358826-4bac2580-bca9-11eb-903c-338599a74cf2.png)

### Whitelisting a Bot & Demonstrating Rate-Limiting¶
1. Navigate to **Security > Bot Defense > Bot Defense Profiles > juiceshop_bot > Bot Mitigation Settings**
2. Under **Mitigation Settings** change Unknown Bots to **Rate Limit** with a setting of **5** TPS. **5** is a very aggressive rate-limit and used for demo purposes in this lab.
`In the “real world” you will need to set this to a value that makes sense for your application or environment to ensure the logs do not become overwhelming. If you don’t know, it’s usually pretty safe to start with the default of 30.`
3. Under **Mitigation Settings Exceptions** click **Add Exceptions** and search for curl and click **Add**.
![image](https://user-images.githubusercontent.com/38420010/119359128-95950b80-bca9-11eb-9ece-4c4fd8cf5bfd.png)
4. Change the Mitigation Setting to **None** and then **Save** the profile.
![image](https://user-images.githubusercontent.com/38420010/119359150-9a59bf80-bca9-11eb-8434-04d89bd71af8.png)
5. Navigate to **Security > Event Logs > Bot Defense > Bot Requests** and review the event logs.
6. Notice the whitelisted bot’s class was changed to unknown and we set curl to not alarm but the requests are still being alarmed. What gives?
![image](https://user-images.githubusercontent.com/38420010/119359260-ba897e80-bca9-11eb-9779-90a60e904923.png)
7. Click the down arrow under **Mitigation Action** and note the reason for the alarm.
`Even though we have whitelisted this bot we can still ensure that it is rate-limited to prevent stress on the application and any violations to that rate-limit will be Alarmed. This bot is currently violating the rate-limit of 5 TPS.`
![image](https://user-images.githubusercontent.com/38420010/119359356-d1c86c00-bca9-11eb-8033-94a3164e3879.png)

### Testing Additional User-Agents
1. Navigate to **Local Traffic > Virtual Servers > Virtual Server List > security-testing-overlay-vs > Resources** tab and under **iRules** click **Manage** and add the **ua_tester** iRule and click **Finished**.
![image](https://user-images.githubusercontent.com/38420010/119359583-12c08080-bcaa-11eb-963f-6c424263b000.png)
`What you just added is an iRule that inserts poorly spoofed User-Agents. Our ipi_tester script has been sending traffic through this Virtual Server all along and spoofing source IP’s to the main site via the ipi_tester iRule.`
2. Navigate to **Security > Event Logs > Bot Defense > Bot Requests** and review the event logs.
3. All the **Unknown** bots are getting rate-limited and the known browsers that do not match the appropriate signatures, such as the spoofed Safari request in this example, are being marked as **Suspicious or Malicious**.

**This completes Lab 2**
  
**Congratulations! You have just completed Lab 2 by implementing a signature based bot profile. Implementing bot signatures is the bare minimum for bot mitigation and not a comprehensive security strategy. This is a excellent step in getting started with WAF and will provide actionable information on automated traffic. You can use this information to take next steps such as implementing challenges and blocking mode. At a very minimum, share this information with your Application teams. Automated traffic can negatively affect the bottom line especially in cloud environments where it’s pay to play. See our 241 class on Elevated WAF Security for more info on advanced bot mitigation techniques.**








































