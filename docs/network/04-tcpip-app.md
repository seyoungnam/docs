# 04 TCP/IP Applications

## TCP and UDP

* Ethernet frame

| Dest MAC | Srce MAC | Dest IP | Srce IP | Dest Port | Srce Port | Sequence | ACK | Data | FCS |

* IP Packet - The chunk of data left the moment the frame comes off the network card

| Dest IP | Srce IP | Dest Port | Srce Port | Sequence | ACK | Data |

* TCP & UDP 

| Dest Port | Srce Port | Sequence | ACK | Data |

<br>

## Understanding Ports

| dest IP  | srce IP    | dest Port   | srce Port   |

* The destination port number is set by the type of application you are using
  * Web server : 80
  * FTP server : 21
  * email server : 110 
* **Port numbers 0 to 1023** are called **well known ports**, used for fixed applications and locked in stone
* The **source port number** is generated as an ephemeral port by the client
  * It's incrementally generated
  * It's got to be a number **1024 up to 65,000**

<br>

