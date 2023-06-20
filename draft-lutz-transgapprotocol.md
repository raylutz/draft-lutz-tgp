---
###
###
title: "TransGapProtocl"
abbrev: "TGP"
category: info

docname: draft-lutz-transgapprotocol-latest
submissiontype: "independent"
number:
date:
consensus: true
v: 3
area: AREA
workgroup: WG Working Group
keyword:

venue:
  group: WG
  type: Working Group
  mail: 
  arch: 
  github: raylutz/draft-lutz-tgp
  latest: 

author:
 -
    fullname: Raymond Lutz
    organization: Citizens Oversight
    email: raylutz@citizensoversight.org

normative:

informative:


--- abstract

TODO Abstract


--- middle

# Introduction

Provide a standard methodology for using removable media, typically flash memory "thumb drives" or "jump drives" to securely track data provided to, from, and between air-gapped computer systems.



## **Goals and Characteristics of the TGP:**

1. Provide a standard methodology for using removable media, typically flash memory "thumb drives" or "jump drives" to securely track data provided to, from, and between air-gapped computer systems. 

2. Internal to one removable media can be viewed as a file system. In that file system can be a "security_log" folder, and a "content" folder. At the highest level, these folders can be zipped, but in many cases, the content is already zipped and may be quite large in size. Thus, additional zipping is not recommended. 

3. The content folder can include any content using a file system layout that is convenient for the application. \

4. The security log folder provides an implementation of an append-only log with security information which is hash-linked together. \

5. For the election systems application, the security log will contain the following:
    1. Public key of the master system
    2. Nonce random number
    3. signature of the above.
------- device is plugged into controlled system

    4. ID of the controlled system
    5. public key of the controlled system.
    6. symmetric encryption key encrypted by EMS public key.
    7. RATS EAT block providing verification data from the controlled device.
    8. response Nonce.
    9. signature by the controlled device seeded by Nonce of (b). 
------- device is plugged into master system 
------- public keys are read from controlled devices and published. 

    10. data request from the master system.
    11. Nonce random number
    12. configuration data
    13. signature by master system 
------- device is plugged into the controlled system
    14. ID of the controlled system
    15. Initial RATS EAT block by controlled system providing verification data.
    16. root hash of the hash-list of all images scanned.
    17. Final RATS EAT block by controlled system providing verification data.
    18. signature by controlled system 
------ device is plugged into the master system
    19. final closing record added to the device.
    20. master system signs everything. 

6. The security log, in its entirety, for each controlled device is an artifact that is published to a transparency service.

## **Current Concept: Use modified TLS protocol	**


Transport Layer Security (TLS) is defined by RFC-8446[^1]. TLS allows client/server applications to communicate over the Internet in a way that is designed to prevent eavesdropping, tampering, and message forgery. It is heavily used today for all "https" secure browser interactions. 
 
The concept will be to mimic an internet channel by using a flash-memory device with a conventional file system, the TGP device. On that device will be the TLS folder. There will be individual files in those folders for each of the sets of messages.

The initialization is performed in an air-gapped secure facility and all devices are handled by staff to allow the devices to be initialized prior to deployment.

The master device (the EMS in this case) is identified as the Client, and the controlled scanner device is the Server. The TGP device is first plugged into the Client and it creates the file "ClientHello" in the TLS folder. 


While in an air-gapped facility, the TGP Device is then plugged into the controlled device, the scanner that will be used in the precinct, which is identified as the Server in this protocol. That device writes the file "ServerHello" to the TGP device in the TLS folder. Included in ServerHello is the public key of the Server device, with proof that it has the corresponding private key. 

The device is unplugged from the "Server" controlled device (precinct scanner) and it is returned to the EMS. If everything checks out, it writes the file ServerFinished, which includes proof that the server has control of the private key that corresponds to the public key. At this point, the TGP device is ready for use in the election. The public keys of all scanner devices are then published to a transparency server, for future use in audits.

At this point, a secure channel is created for application data written to the TGP device. 


```
Figure 1 below shows the basic full TLS handshake:

       Client                                           Server

Key  ^ ClientHello
Exch | + key_share*
     | + signature_algorithms*
     | + psk_key_exchange_modes*
     v + pre_shared_key*       -------->
                                                  ServerHello  ^ Key
                                                 + key_share*  | Exch
                                            + pre_shared_key*  v
                                        {EncryptedExtensions}  ^  Server
                                        {CertificateRequest*}  v  Params
                                               {Certificate*}  ^
                                         {CertificateVerify*}  | Auth
                                                   {Finished}  v
                               <--------  [Application Data*]
     ^ {Certificate*}
Auth | {CertificateVerify*}
     v {Finished}              -------->
       [Application Data]      <------->  [Application Data]
```


Before the TGP device is disconnected from the Client device, configuration data can be written using the secure keys established in the protocol. This configuration information is to configure the Server device so it can operate correctly in the election. This first set of data blocks from the Client will typically provide the election configuration in terms of the precincts and styles supported by the machine and the contests and options on each style and how to interpret the targets, for example. That assumes a ballot-aware scanner. If it is a "dumb" scanner then there may be very little if any configuration information.

The initial file mentioned above may be initialized later or replaced, while not erasing the original information.

The device is moved to the Server device and it is deployed. The Server device performs the final phase of the protocol and establishes a corresponding set of keys for the communication.

As ballots are scanned, the data is encrypted using the keys established and written to the TGP device. This means that there will not be a large time delay when the election is closed for encrypting data, and it mean that data will not exist twice on the device. 


<!-- Footnotes themselves at the bottom. -->
## Notes

[^1]:
     [https://datatracker.ietf.org/doc/html/rfc8446](https://datatracker.ietf.org/doc/html/rfc8446) 



# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
