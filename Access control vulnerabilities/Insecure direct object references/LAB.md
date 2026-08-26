# Insecure direct object references

**Source:** [PortSwigger](https://portswigger.net/web-security/access-control/lab-insecure-direct-object-references)  
**Category:** Access control  
**Difficulty:** APPRENTICE

## Objective
Solve the lab by finding the password for the user `carlos`, and logging into their account.

## Vulnerability
This lab stores user chat logs directly on the server's file system, and retrieves them using static URLs.

## Exploitation

### Steps
1. Enter the live chat and exchange a few test messages. The responses appear to be random phrases.
2. Click View transcript to download a .txt file containing the message transcript. The first downloaded file was named 3.txt, the next one 4.txt, and so on.
3. Intercept the View transcript request. The header shows `POST /download-transcript HTTP/2`, which suggests the location on the server where these sequentially named files are stored.
4. In the browser, navigate to `/download-transcript/1.txt`. The conversation file is downloaded, and from it we can extract the password. The file looks like this:

```txt
CONNECTED: -- Now chatting with Hal Pline --
You: Hi Hal, I think I've forgotten my password and need confirmation that I've got the right one
Hal Pline: Sure, no problem, you seem like a nice guy. Just tell me your password and I'll confirm whether it's correct or not.
You: Wow you're so nice, thanks. I've heard from other people that you can be a right ****
Hal Pline: Takes one to know one
You: Ok so my password is iisdungx8q99brrr7srw. Is that right?
Hal Pline: Yes it is!
You: Ok thanks, bye!
Hal Pline: Do one!
```

5. Check the 2.txt file in the same manner - it is empty.
6. Attempt to log in as `carlos:iisdungx8q99brrr7srw` - successful.

### Payload
`/download-transcript/1.txt`

### Shortcut
1. Directly access /download-transcript/1.txt to retrieve Carlos's password.

## Impact
An attacker can access sensitive information (such as passwords) belonging to other users by guessing or enumerating sequential file names, leading to account takeover.

## Mitigation
- Implement proper access controls on the server side to verify that the requesting user is authorized to access the requested resource.
- Use unpredictable, hard-to-guess identifiers (e.g., UUIDs) for file names instead of sequential integers.
- Avoid exposing internal file paths or structures directly to the client.

## Tools
- Burp Suite

## Notes
- The lab demonstrates how insecure direct object references (IDOR) can expose sensitive data when file names are predictable.
- Always validate user permissions before serving any resource, even if it seems harmless or static.