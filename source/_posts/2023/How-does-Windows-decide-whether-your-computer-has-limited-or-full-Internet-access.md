---
title: >-
  How does Windows decide whether your computer has limited or full Internet
  access
date: 2023-01-24 20:52:23
updated: 2023-01-24 20:52:23
tags: [Windows]
categories: Windows
---

> reference [How does Windows decide whether your computer has limited or full Internet
>   access?](https://devblogs.microsoft.com/oldnewthing/20221115-00/?p=107399)

Windows lets you know when your computer’s Internet connection is limited or absent entirely. What is this sorcery?

Windows attempts to download a file from a dedicated Web server. Depending on which version of Windows, it’s [http://www.msftncsi.com/ncsi.txt](http://www.msftncsi.com/ncsi.txt) or [http://www.msftconnecttest.com/connecttest.txt](http://www.msftconnecttest.com/connecttest.txt). If the download is successful and contains the correct contents, then Windows concludes that you have full Internet access.

If something goes wrong, Windows will report either limited or no Internet access, depending on what exactly went wrong.

You can [read more details on docs.microsoft.com](https://docs.microsoft.com/en-us/troubleshoot/windows-client/networking/internet-explorer-edge-open-connect-corporate-public-network).



Linux distros do something similar… e.g. on a current Fedora version, it’s http://fedoraproject.org/static/hotspot.txt.

Note that as with the Windows version, the protocol is HTTP, not HTTPS – because captive portals completely break TLS, but plaintext HTTP will result in a clean redirect to the portal, allowing the network service to detect the presence of the portal and to bring up a browser window to let the user authenticate.
