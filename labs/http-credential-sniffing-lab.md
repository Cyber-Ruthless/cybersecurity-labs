# HTTP Credential Sniffing with Wireshark

A lab demonstrating why submitting login credentials over plain HTTP (instead of HTTPS) is insecure — capturing a login form submission in transit and recovering the username and password directly from the packet capture.

> ⚠️ **Disclaimer:** Performed entirely in an isolated lab environment (Windows 11 client, Windows Server 2019, Parrot Security) that I own and control, against a test site (`moviescope.com`) intentionally served over unencrypted HTTP. Do not capture or inspect traffic on networks or systems you do not own or have explicit written authorization to test.

---

## Lab Environment

| Component | Details |
|---|---|
| Client | Windows 11 |
| Target site | `moviescope.com` (served over **HTTP**, not HTTPS) |
| Capture tool | Wireshark |
| Capture file | `praaa.pcapng` |

---

## 1. Logging Into the Target Site

A login was performed on the `moviescope.com` test site using the credentials `sam` / a test password. Note the address bar explicitly flags the connection as **"Not secure"** — the site is served over plain HTTP, and the browser's password manager prompt offers to save the credential, unaware it was just sent unencrypted.

![Logging into moviescope.com over HTTP on Windows 11](../images/website%20on%20windows%2011.png)

---

## 2. Capturing and Filtering the Traffic in Wireshark

With a Wireshark capture already running during the login, the traffic was filtered down using:

```
http.request.method == POST
```

This isolates the exact HTTP POST request generated when the login form was submitted. Wireshark's **Find Packet** feature was then used to search the capture for the string `pwd`, pinpointing the request instantly among hundreds of other packets.

Drilling into the request's HTML form data reveals the submitted values in plain text:

```
Form item: "txtusername" = "sam"
Form item: "txtpwd" = "test"
Form item: "btnlogin" = "Login"
```

Because the form was submitted over HTTP instead of HTTPS, every field — including the password — travels across the network completely unencrypted and readable by anyone capturing traffic on the same segment.

![Wireshark capture revealing the plaintext username and password from the POST request](../images/password%20sniffed.png)

---

## How This Attack Works

HTTP transmits all request data — including form fields like usernames and passwords — as plain, unencrypted text. Anyone able to observe network traffic between the client and server (e.g. on a shared Wi-Fi network, a compromised switch, or via ARP spoofing to redirect traffic) can simply filter for POST requests in a tool like Wireshark and read credentials directly out of the form data, exactly as shown here. HTTPS (TLS) is what closes this gap — it encrypts the entire request, including form data, so an observer only sees ciphertext.

---

## Key Takeaways / Mitigations

- **Always enforce HTTPS** for any page handling authentication — redirect HTTP to HTTPS and use HSTS to prevent downgrade.
- **Never trust "Not secure" warnings as cosmetic** — browsers flag this precisely because credentials and session data are exposed in transit.
- **Avoid logging into sites over untrusted/shared networks** without confirming HTTPS is in use.
- **Use network segmentation and switch port security** to limit who can passively observe traffic on a given segment in the first place.

---

## Tools Referenced

- [Wireshark](https://www.wireshark.org/)
