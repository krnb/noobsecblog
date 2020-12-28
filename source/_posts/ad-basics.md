---
title: Understanding Active Directory and Kerberos
date: 2020-11-12 11:01:35
tags: [active directory, kerberos]
---


# Understanding Active Directory and Kerberos

In this blog post I will be covering some of the basics of Active Directory and Kerberos. This is to support the future posts which will only be about their exploitation in numerous ways. So let's get started.

## What is Active Directory?



## What is Kerberos?

Kerberos is a network authentication protocol that was developed by the MIT and was named after the guard dog of the underworld, Cerberus. This authentication protocol was adopted by Microsoft which will work hand in hand with the Active Directory.

### Kerberos Authentication

Kerberos authentication works on the basis of Tickets in a client-server architecture. To understand the process of authentication in detail, let's take an example of a client in an organization who wants to access a file server.

There are three main components to keep in mind here:
 - Key Distribution Center (KDC) 
 - Client
 - File server

Step 1:
When a client present in the organization wants to access a resource it sends a request, *AS-REQ*, to the KDC which consists of a timestamp which is encrypted with the clients NTLM hash.

Step 2:
Once the the request reaches the KDC, it verifies the request by promptly decrypting the request and if it were to decrypt it then it sends a response, *AS-REP*, a ticket called *Ticket Granting Ticket* (TGT), which is encrypted using the NTLM hash of the service account under which the KDC is running, *krbtgt*.

A part of the response, TGT, is encrypted using the NTLM hash of krbtgt and other is encrypted using the clients' NTLM hash. Since the client would not have the NTLM hash of the krbtgt account, it cannot decrypt that part.

Step 3:
Now to request access to some resource, a file server here, the client will send the encrypted TGT it had previously received. This request is called as *TGS-REQ*

Step 4:
KDC verifies the request by decrypting the TGT the client sent, if it is able to then it sends the client the *Ticket Granting Service* (TGS). This response is called as *TGS-REP*

Step 5:
Once the client receives the TGS for a particular service, it can then connect to this service and then present the ticket. This process is called *AP-REQ*

To provide access or not is upto the service as KDC will not bother itself with the access control.

There are some more steps but they are optional, usually they will not happen unless the organization has enabled them:
Step 6:
Upon being presented the TGS, the service could perform an optional mutual authentication between the client and the service.

Step 7:
The service could perform a PAC validation request to the KDC. PAC or Privilege Attribute Certificate is a certificate along with Kerberos tickets that contains information regarding the security identifiers, group membership and more. To protect from Man-in-the-middle attack, PAC validation can be performed. Learn more [here](https://docs.microsoft.com/en-us/archive/blogs/openspecification/understanding-microsoft-kerberos-pac-validation)


### To Note

Now that we have understood the basic authentication mechanism of Kerberos, let's talk a little about its' nuances.

The KDC lies with the Domain Controller (DC) of the organization. The DC is aware of all the secrets of the organization including hashes of every account and thus is able to decrypt requests coming in from any client in the organization. Similarly, it can encrypt responses being sent using any services' NTLM hash.

The only check for authentication when a client requests for a TGT is that the KDC is being able to decrypt the request with the clients' NTLM hash which KDC has. It doesn't check if this is a valid request or not, as long as it is able to decrypt the request using the clients' NTLM hash that the requests is or pretends to be from, the KDC will assume it is a legitimate request.

