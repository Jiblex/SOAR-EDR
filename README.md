# SOAR-EDR

## Overview
This project aims to integrate Security Orchestration, Automation, and Response (SOAR) with Endpoint Detection and Response (EDR) capabilities to streamline and automate incident response processes. The core of this project is built around leveraging LimaCharlie for endpoint detection and custom rule creation, coupled with Tines for orchestrating automated responses and user notifications.

## Components

### Playbook

<img width="817" alt="Screenshot 2024-09-05 at 23 23 08" src="https://github.com/user-attachments/assets/7d486743-6087-4023-8ecd-c398c8d7601b">
<br>
<br>

### LimaCharlie
LimaCharlie is the EDR tool which is used for detecting incidents based on custom rules intended to detect potential threats or suspicious activity at endpoint. LimaCharlie provides the necessary visibility into any endpoint connected to it, providing alerts once detection rules have been triggered. 

### Tines
Tines acts as the the automation and orchestration layer in this project. When a detection rule is triggered and an alert is created, it is forwarded to Tines. Tines then manages the response workflow, sending notifications to the appropriate communication channels (in this project it is email and or slack) and handling user interactions. 

### Machines
The machine monitored by LimaCharlie in this project is a cloud Windows machine (Vultr). 

## Project Timeline

### Playbook
I first started the project by defining a playbook which clearly shows the workflow of the project, adding to it as I went along if needed or making changes if I later decided something didn't make sense. I would reference back to the playbook when implementing the SOAR aspect of the project. 

### Windows machine setup 
The Windows cloud machine was booted up using Vultr. Once running, the machine was clearly displayed on Vultr:

<img width="1442" alt="Vultr Windows machine" src="https://github.com/user-attachments/assets/a3159c6c-6a31-4010-824d-dd15cc2b29e4">
<br>
<br>

I made a firewall group with a single for the Windows machine to make it so that it only accepts MS RDP on port 3389 from my IP, and setup this firewall group for the Windows machine.

<img width="1424" alt="SOAR-EDR-Firewall-Windows" src="https://github.com/user-attachments/assets/d1118fa7-658f-4647-80d5-f7ba3d2e960e">
<img width="1328" alt="Setting Firewall for Windows machine" src="https://github.com/user-attachments/assets/e6349a42-31b3-4e5c-a1e0-875052727296">
<br>
<br>

## LimaCharlie Installation - EDR
After creating a LimaCharlie project (or per LimaCharlie terms an organization), I looked for the installation instructions. The following screenshot is from the LimaCharlie "Installation Keys" screen, which gives some installation instructions for different machines:

<img width="1792" alt="LC Insta Key" src="https://github.com/user-attachments/assets/da993506-1b1c-40c3-9176-ab0c0e4a5fca">
<br>
<br>

From the instructions, I needed to run the following command: _lc_sensor.exe -i YOUR_INSTALLATION_KEY_

I logged into the Windows machine using the provided administrator password using the remote console provided by Vultr. I copied the "Windows 64 bit" link under "Sensor Downloads", which gave me the .exe file needed for the command, and then after copying my "Sensor Key" (the _YOUR_INSTALLATION_KEY_ variable), I opened PowerShell as admin and ran the command: 

<img width="1331" alt="Windows LC download powershell" src="https://github.com/user-attachments/assets/1c3a2a10-a5ff-490f-a0d3-fb2ce3de8318">
<br>
<br>

After running it I received a success message:

<img width="1376" alt="Windows LC Success" src="https://github.com/user-attachments/assets/18f8573d-40e0-47a4-b281-7b24d5d01e14">
<br>
<br>

From here, when navigating to the "Sensors" screen in LimaCharlie the Windows machine could be seen displayed:

<img width="1792" alt="LC Windows connect" src="https://github.com/user-attachments/assets/0599829b-4ccc-4e5f-8b13-ffd0e23a6add">
<br>
<br>

Clicking on the machine leads the user to a "Sensor Details" window where various propoerties of the sensor are displayed. In particular, the especially useful panes here are "Detections", which shows detections on the machine, "Console", which allows the remote sending of commands to the machine, and "File System" which allows the user to see the machine's file system, which includes downloading of files as well as their various metadata.

<img width="1792" alt="Click on windows" src="https://github.com/user-attachments/assets/5d63c902-9479-4a78-b77d-28230450e3c6">
<br>
<br>

## Testing Timeline

Once connected, I ran the file LaZagne.exe (purpose explained in the next section), to check that the event was being picked up LimaCharlie. 

<img width="1792" alt="Screenshot 2024-09-07 at 20 03 43" src="https://github.com/user-attachments/assets/eba8b69d-24c3-4b08-8370-015f810d36d0">
<br>
<br>

## Creating a Rule on LimaCharlie
For the first rule, I decided to create a rule which flags the execution of LaZagne, a password recovery tool. As I did not know how to write rules on LimaCharlie at this point in time, I decided to go find a rule made for detection and response related to credential recovery, with the additional detail that can be seen in the previous screenshot, that the rule should target a "NEW_PROCESS" and "EXISTING_PROCESS" (the latter being included after reading documentation). The following <a href="https://github.com/refractionPOINT/rules/blob/master/Sigma/dr_rules/windows_process_creation/win_susp_process_creations.yml"> rule </a> was found after looking at for sample rules on the <a href="https://github.com/limacharlieio/limacharlieio"> LimaCharlie github page </a>. With the help of LimaCharlie documentation as well as studying the rule, I got the rough hang of how rules can be written. 

The following is the rule I wrote to detect execution of LaZagne.exe and the response to this detection: 

<img width="1710" alt="Detection rule;" src="https://github.com/user-attachments/assets/7864284b-c2bd-4b1d-9055-e796df56a42a">

<img width="1685" alt="Screenshot 2024-09-07 at 20 22 34" src="https://github.com/user-attachments/assets/3e6cce42-e5cb-464b-9680-94239abd1772">
<br>
<br>

The detection states that the event must be a new or existing process, and must be a windows device, and one of the following must be true (all not case sensitive):
- the command must end with lazagne.exe
- the command line contains the string "lazagne" at any index
- the hash of the file matches the provided hash (hash acquired from the event timeline)

If this statement evaluates to True, a detection is triggered. The response simply shows the response event generated with its metadata. 

LimaCharlie provides an option to test created rules at the bottom of the rule creation window called "Target Event" where an event can be pasted and checked to see whether it triggers the specific rule. Pasting the event provided in the previous event screenshot, the rule did flag the event as a detection. Success. 

## Setting up Tines - SOAR

Now that LimaCharlie was able to generate alerts, the EDR aspect was working; it was time to implement the SOAR aspect. First, I went to create a new, what Tines calls, "story". A story in Tines is effectively like creating a playbook using different blocks of logic. Once I had created my new story, I started off by placing a webhook block, which is responsible for connecting LimaCharlie and Tines.

<img width="1792" alt="Webhook" src="https://github.com/user-attachments/assets/2c81e060-5155-4f50-a740-45be2cb28694">
<br>
<br>

A webhook is a block that is able to recieve events through Webhooks (HTTP callbacks), in this case it recieves alerts from LimaCharlie. To do this, I copied the Webhook URL and used this URL to add the Webhook as an output in LimaChalie in the "Outputs" section. 

<img width="1792" alt="LC Outputs" src="https://github.com/user-attachments/assets/bb7d7524-f9c8-4320-8d35-f2c507f0aafc">
<br>
<br>

Selecting Detections:

<img width="1792" alt="Detections select" src="https://github.com/user-attachments/assets/bb93c50f-58d7-4ab7-a21e-507383c2460f">
<br>
<br>

Selecting Tines:

<img width="1792" alt="Tines select" src="https://github.com/user-attachments/assets/eb5f7d48-57d3-4941-ac16-c55bf221ad70">
<br>
<br>

Once this was done, I was going to test the connection, and unexpectedly LimaCharlie forwarded me to a page that helped me out with the testing:

<img width="1792" alt="Webhook in LC" src="https://github.com/user-attachments/assets/daf319a5-8350-4349-95c5-43606685b9c6">
<br>
<br>

Once sent, clicking on the webhook brings up some buttons (as can be seen in the first picture of this section) where clicking the "Events" button will bring up a window showing all received events, thereby making it easy to see whether an event was received. Now that the connection was made, I wanted to begin by sending the received alert from LimaCharlie to Slack. After making a new workspace and creating an alert channel, I used a Slack template block in Tines and configured it to simply send: "A detection rule has been triggered!" to the alert channel whenever the Webhook received an alert. To make the connection from Tines to Slack, I needed to add Slack in the Tines credentials (which the template conveniently prompts you for) and then add the channel ID to which I wanted the message sent to into the Slack template as seen below:

<img width="1792" alt="Tines rule triggerd" src="https://github.com/user-attachments/assets/87d06417-fd5d-4dae-a762-d880458085d9">
<br>
<br>

Once done, I simply executed LaZagne.exe to create an alert:

<img width="1173" alt="Slack alert test" src="https://github.com/user-attachments/assets/79e628c1-806e-41fb-95de-63ca49e9a7b6">
<br>
<br>

Success! It felt great to see something that happened on the Windows computer travel all the way to Slack. Next, as shown in the playbook, I needed to implement email. As can be seen in previous pictures, Tines has an email block making this functionality very simple. All I needed to do was specify the email to which to send the alert, as well as what the body should contain. 

<img width="1792" alt="Adding email" src="https://github.com/user-attachments/assets/28d6402f-721b-4105-be07-e2acaf37dd93">
<br>
<br>

The contents of the email, and the slack message, needed to be the variables specified in the playbook. To access these variables (which are within the alert sent by LimaCharlie) in Tines, I needed to refer to them through the _retrieve_detections_ variable (the name of the Webhook). Clicking on the Webhook and then events, allows one to see all of the fields which can be accessed and how. For instance, to access the author varibale, it must be referred to as _retrieve_detections.body.author_.

<img width="1148" alt="Screenshot 2024-09-09 at 18 20 13" src="https://github.com/user-attachments/assets/b242bc47-9186-433e-81de-997c487bb6f7">
<br>
<br>

Combined with some HTML, this is what the final message looked like:

<img width="309" alt="Screenshot 2024-09-09 at 18 23 21" src="https://github.com/user-attachments/assets/0d295b15-520c-42f5-b020-f6cea8e02353">
<br>
<br>

Going into the events tab in the Webhook allows one to re-emit events, this way making testing easier. The email and slack messages now looked as follows (note that the variables won't look be the same as I took pictures as I went along as well as after finishing some parts):

<img width="1155" alt="Msg Field Names" src="https://github.com/user-attachments/assets/6eff813e-8457-4288-a584-18ce166443f9">
*Note that HTML formatting does not work within Slack message in Tines. 
<br>
<br>
<img width="1453" alt="Screenshot 2024-09-09 at 18 28 37" src="https://github.com/user-attachments/assets/962c937d-3123-481b-b0b1-4b972aa4128b">
<br>
<br>

Notice that the contents of the messages here are different to that in the playbook as I decided to change some things as I went through the project. I left the old playbook in to show that playbooks may be changed during development process as sometimes one may miss some details or change their mind. 

The new message content now is:
Message:
- Title
- Time
- Machine
- Source IP
- Username
- File Path
- Command Line
- Sensor ID (LimaCharlie)
- Detection Link

# User Prompt

Next step in the playbook is to create a user prompt asking whether the user would like to isolate the machine. This prompt will be responsible for the automated response functionality. As per the playbook, the prompt will ask the user whether they want to isolate the machine on which the detection occured. To do this I used the Page block. Tines makes it so that a new Page is generated for each Webhook event, which can be accessed via a dynamic URL. I included this dynamic URL as an extra field in the slack message named "Take Action", corresponding to the _PAGE.user_prompt_ variable. 

<img width="1792" alt="Msg with info" src="https://github.com/user-attachments/assets/d67a96d9-f5e2-4755-a9a6-f28b31fca0ff">
<br>
<br>

<img width="273" alt="Screenshot 2024-09-10 at 22 55 51" src="https://github.com/user-attachments/assets/b48e95ca-9844-434f-9c7b-20c1973b637c">
<br>
<br>

I included the metadata on the Page again (same syntax as email) in case the user wants to re-review the info before making their decision to avoid extra clicks. Under the metadata I included a boolean in the form of a yes and no button, and made it required to have one or the other selected to submit the form. After clicking on one of the buttons, the user can now submit the form. 

<img width="398" alt="Screenshot 2024-09-10 at 23 24 41" src="https://github.com/user-attachments/assets/d45c25e4-9042-4c96-bc61-3232feb37c19">
<br>
<br>

I then tested this functionality by re-emitting an event, to make sure I can access the form from slack. 

<img width="1175" alt="Screenshot 2024-09-10 at 23 31 24" src="https://github.com/user-attachments/assets/9203c138-dd65-4ed0-ab64-a2e64fc0ac1d">
<br>
<br>

Clicking on the link:

<img width="502" alt="Screenshot 2024-09-10 at 23 33 54" src="https://github.com/user-attachments/assets/2cc942ac-3dc0-4606-9267-57d7b6d23e56">
<br>
<br>

No issues there, so I moved onto adding functionality to the yes and no buttons. To do this I used the Trigger block, which takes an action based on a trigger (in this case this "yes" or "no"), effectively acting as an if statement. Given that the user prompt has a boolean responsible for the buttons, the output of the form is either true or false making things simple:
- If the output is false: send a slack message asking the user to investigate the incident.
- If the output is true: send LimaCharlie a request to isolate the machine and then send a slack message confirming the isolation of the machine.

<img width="865" alt="Screenshot 2024-09-10 at 23 51 49" src="https://github.com/user-attachments/assets/4c3fd964-144b-4adf-a17a-a047d0769e3c">
<br>
<br>

To send the isolation request as well get the isolation status confirmation, I used the HTTP Request blocks to send the requests to LimaCharlie. In order to be able to have the requests perform actions on LimaCharlie, I needed to make a new credential for LimaCharlie on Tines using my organization's API. The API token in this case lasts only an hour, this way I do not need to worry about it being misused from Tines while I am not working on the project. Tines makes it really easy to input a new token if the previous is no longer valid, which I liked a lot. Once implemented, I decided to run LaZagne.exe again, which triggered a Slack message and email, clicked the "Take Action" link, and selected "Yes" before submitting the form. 

<img width="776" alt="Screenshot 2024-09-10 at 23 55 46" src="https://github.com/user-attachments/assets/c53801c7-61cf-4a04-ba2d-fb6458d17f41">
<br>
<br>

Slack then sent me a confirmation message that the machine was isolated:
<img width="1171" alt="Screenshot 2024-09-10 at 23 56 14" src="https://github.com/user-attachments/assets/8d43118c-4c0b-4bde-99d3-abf906440277">
<br>
<br>

Going to LimaCharlie and clicking on the Windows machine in the "Sensors" tab clearly showed that the machine had been isolated:
<img width="1789" alt="Screenshot 2024-09-10 at 23 57 11" src="https://github.com/user-attachments/assets/77c35a02-1066-4f24-b580-919f24426e32">
<br>
<br>

To make sure this was the case, I opened the Windows machine console and pinged Google:

<img width="416" alt="Screenshot 2024-09-11 at 00 08 48" src="https://github.com/user-attachments/assets/83339379-5967-48f1-8936-fc91a530adc8">
<br>
<br>

This confirmed that LimaCharlie properly isolated the machine. Next I made Slack was sending a investigation notice if I clicked "No".

<img width="1164" alt="Screenshot 2024-09-11 at 00 10 27" src="https://github.com/user-attachments/assets/2eb55541-76c0-478e-b1bb-834ea46868a4">
<br>
<br>

## Wrap-Up and Future Additions

At this point everything worked and I had completed all the functionality I set out to implement at the beginning of the project. My plan remains to add a Linux computer to this project, and potentially a Mac computer. In addition, I want to write more rules to address other scenarios. I may decide at any point to add more automation and response functionality through Tines, which I would document after this section. I will not be prioritizing documenting these things on here right away, but if anyone is feeling really curious you are free to reach out to me for an update if you don't see anything beyond this section. If anyone finds any typos or has any comments or advice, I would really appreciate it. 
































