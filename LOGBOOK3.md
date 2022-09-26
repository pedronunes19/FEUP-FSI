# Trabalho realizado na Semana #3

## Identification

- Exploitable path traversal vulnerability, caused by improper limitation of a pathname to a restricted directory.
- Affects the Zoom client (version 4.6.10, which processes messages including animated GIFs), for Windows, macOS and Linux systems.
- Chat is built on top of XMPP standard (supports a giphy extension) and uses messages with URLs to fetch GIFs.
- Allows directory traversal and arbitrary file write outside Zoom's installation directory.

## Cataloguing

- Discovered and reported by a member of Cisco Talos (see TALOS-2020-1055).
- Disclosed to vendor on 2020-04-16, public released in 2020-06-03.
- CVSS 2.x score of 7.5 (high); CVSS severity 3.0 score of 8.5 (high) CVSS severity 3.1 score of 9.8 (critical).
- Zoom planned of revamping its bug bounty program in the same month the vulnerability was disclosed.

## Exploit

- In Talos Vulnerability Report there is an example of a malicious XMPP message.
- Changing the URL for the retrieval of the giphy file allows the download of an arbitrary file.
- Crafting a special id attribute for the giphy tag allows writing a file in an arbitrary location.
- Altough the vulnerability is well documented in the Talos Report, there are no known Metasploit modules.

## Attacks

- There are no known reports of successful attacks, vulnerabilty patched since 2020-04-21.
- On Windows systems using NTFS, NTFS alternative streams can potentially be abused to change configuration files or affect lock files.
- Arbitrary file write could potentially be abused in conjunction with other vulnerabilities for arbitrary code execution.
- The exploitation doesn't need any form of authentication.
