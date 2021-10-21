---
title: Kerberoasting
date: 2021-10-15 22:17:51
tags: [active directory, windows, kerberos]
---


# Attacking Kerberos - Kerberoasting

Kerberoasting is a very popular attack in the Active Directory realm since over 6 years now.  Attacking guard dog of Hades by Tim Meddin


## Overview 

In any organization that is using Active Directory, the authentication process is done via Kerberos since a while now. While Kerberos is a secure network authentication protocol, there are several gaps inherently with the way it works. To put it simply, it works as follows:

Step 1 : User creates an encrypted request by taking system time and encrypting it using the password hash. It sends the request to the KDC asking TGT

Step 2 : KDC takes the users password hash, attempts to decrypt the request. If successful, assumes the request came from the legitimate user. Takes the users identifiers and puts it in a packet, then encrypts the packet using KDCs' (krbtgt) own password hash and sends it to the user.

Step 3: Once the user receives the TGT, the user can go ahead and request for service tickets to access some service within the forest by sending the TGT back to the KDC to send it a TGS.

Step 4: Once the KDC receives the TGT, it attempts to decrypt it and if successful, assumes that the TGT is legitimate. It looks up the service requested, creates a service ticket which is encrypted using the service accounts password hash 