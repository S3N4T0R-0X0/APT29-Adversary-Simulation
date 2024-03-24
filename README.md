# APT29 Adversary Simulation

This is a simulation of attack by the Cozy Bear group (APT-29) targeting diplomatic missions.
The campaign began with an innocuous and legitimate event. In mid-April 2023, a diplomat within the Polish Ministry of Foreign Affairs emailed his legitimate flyer to various embassies advertising the sale of a used BMW 5-series sedan located in Kyiv. The file was titled BMW 5 for sale in Kyiv - 2023.docx.
I relied on palo alto to figure out the details to make this simulation: https://unit42.paloaltonetworks.com/cloaked-ursa-phishing/

![GIF_20240228_033834_929](https://github.com/S3N4T0R-0X0/APT-29-Adversary-Simulation/assets/121706460/7ed2d156-4e1e-4b4a-9018-bf84a0e21352)


1.DOCX file: created DOCX file includes a Hyperlink that leads to downloading further HTML (HTML smuggling file)

2.HTML Smuggling: The attackcers  use the of HTML smuggling to obscure the ISO file

3.LNK files: When the LNK files (shortcut) are executed they run a legitimate EXE and open a PNG file. However, behind the scenes, encrypted shellcode is read into memory and decrypted.

4.ISO file: The ISO file contains a number of LNK files that are masquerading as images. These LNK files are used to execute the malicious payload.

5.DLL hijacking: The EXE file loads a malicious DLL via DLL hijacking, which allows the attacker to execute arbitrary code in the context of the infected process.

6.Shellcode injection: The decrypted shellcode is then injected into a running Windows process, giving the attacker the ability to execute code with the privileges of the infected process.

7.Payload execution: The shellcode decrypts and loads the final payload inside the current process.

8.Dropbox C2: This payload beacons to Dropbox and Primary/Secondary C2s based on the Microsoft Graph API.


![Screenshot from 2024-03-11 15-43-36](https://github.com/S3N4T0R-0X0/APT29-Adversary-Simulation/assets/121706460/f12854a1-0278-4be6-8376-8aa48ef2785e)


## The first stage (delivery technique)

First the attackers created DOCX file includes a Hyperlink that leads to downloading further HTML (HTML smuggling file)
The advantage of the hyperlink is that it does not appear in texts, and this is exactly what the attackers wanted to exploit


![Screenshot from 2024-03-01 19-18-51](https://github.com/S3N4T0R-0X0/APT29-Adversary-Simulation/assets/121706460/bf7fb22b-e089-45fc-831d-1b98d330d72b)


HTML Smuggling used to obscure ISO file and the ISO contains a number of LNK files masquerading as images
command line to make payload base64 to then put it in the HTML smuggling file:
`base64 paylode.iso -w 0` and i added a picture of the BMW car along with the text content of the phishing message in the HTML file 


![Screenshot from 2024-03-01 19-39-42](https://github.com/S3N4T0R-0X0/APT29-Adversary-Simulation/assets/121706460/4e17f341-bd19-45a0-8b1f-1e6f172c2600)

## The Second stage (implanting technique)

We now need to create a PNG image that contains images of the BMW car, but in the background when the image is opened, the malware is running in the background,
at this stage i used the WinRAR program to make the image open with Command Line execution via CMD when opening the image and I used an image in icon format

![20240302_194641](https://github.com/S3N4T0R-0X0/APT29-Adversary-Simulation/assets/121706460/d80c62f9-6515-404d-971a-f2aa49957998)

After using WinRaR for this compressed file, we will make a short cut of this file and put it in another file with the actual images then we will convert it to an ISO file through the PowerISO program

Note: This iso file is the one to which we will make base64 for this iso file and put in the html smuggling file before make hyperlink and place it in the docx file


![Screenshot from 2024-03-02 15-39-55](https://github.com/S3N4T0R-0X0/APT29-Adversary-Simulation/assets/121706460/566bae43-effb-4379-ade5-0f942ed81d52)

## The third stage (execution technique)

Because i put the command line in the setup (run after extraction) menu in the Advanced SFX options for the WinRaR program now when the victim open the ISO file to see the high-quality images for the BMW car according to the phishing message he had previously received he will execute the payload with opening the actual image of the BMW car


https://github.com/S3N4T0R-0X0/APT29-Adversary-Simulation/assets/121706460/effcdebb-c8d2-4609-b250-639a39fd3189

## The fourth stage (Data Exfiltration) over Dropbox API C2 Channe

The attackers used the Dropbox C2 (Command and Control) API as a means to establish a communication channel between their payload and the attacker's server. By using Dropbox as a C2 server, attackers can hide their malicious activities among the legitimate traffic to Dropbox, making it harder for security teams to detect the threat.
First, we need to create a Dropbox account and activate its permissions, as shown in the following figure.

![Screenshot from 2024-03-12 16-10-13](https://github.com/S3N4T0R-0X0/APT29-Adversary-Simulation/assets/121706460/b1eb85c4-9b4e-40b2-8e3e-5fe76c5a664d)

After that, we will go to the settings menu to generate the access token for the Dropbox account, and this is what we will use in Dropbox C2.


![photo_2024-03-12_16-22-54](https://github.com/S3N4T0R-0X0/APT29-Adversary-Simulation/assets/121706460/45f8dec5-5c1d-42f8-9a10-7735745de1c4)

This script integrates Dropbox API functionality to facilitate communication between the compromised system and the attacker-controlled server,
thereby  hiding the traffic within legitimate Dropbox communication, and take the access token as input prompts the user to enter an AES key 
(which must be 16, 24, or 32 bytes long) and encrypts the token using AES encryption in ECB mode. It then base64 encodes the encrypted token and returns it.


![171053992557140444](https://github.com/S3N4T0R-0X0/APT29-Adversary-Simulation/assets/121706460/29737154-6e6c-43d9-ae0d-2dffe977445b)

I used payload written by Python only to test C2 (testing payload.py), if there were any problems with the connection (just for test connection) before writing the actual payload.

## The fifth stage (payload with DLL hijacking) and injected Shellcode

this payload uses the Dropbox API to upload data, including command output to Dropbox. By leveraging the Dropbox API and providing an access token the payload hides its traffic within the legitimate traffic of the Dropbox servic and If the malicious DLL fails to load, it prints a warning message but continues executing without it.

![Screenshot from 2024-03-23 15-17-27](https://github.com/S3N4T0R-0X0/APT29-Adversary-Simulation/assets/121706460/e928ef81-a262-4912-afa3-646c3efc839d)

1.DLL Injection: The payload utilizes DLL hijacking to load a malicious DLL into the address space of a target process.

2.Shellcode Execution: Upon successful injection, the malicious DLL executes shellcode stored within its DllMain function.

3.Memory Allocation: The VirtualAlloc function is employed to allocate memory within the target process, where the shellcode will be injected.

4.Shellcode Injection: The shellcode is copied into the allocated memory region using memcpy, effectively injecting it into the process.

5.Privilege Escalation: If the compromised process runs with elevated privileges, the injected shellcode inherits those privileges, allowing the attacker to perform privileged operations.

![Screenshot from 2024-03-23 15-16-20](https://github.com/S3N4T0R-0X0/APT29-Adversary-Simulation/assets/121706460/d60575bb-0ade-4e76-bfda-bde710e76c34)

## Final result: payload connect to Dropbox C2 server

https://github.com/S3N4T0R-0X0/APT29-Adversary-Simulation/assets/121706460/43b900b9-b96e-417a-91cd-284e4ccbd7e6

