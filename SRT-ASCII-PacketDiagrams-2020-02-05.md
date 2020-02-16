> **NOTE**: This document reproduces the Packet Structure section of the 
> SRT-TechnicalOverview-2019-09-18.md document, with packet diagrams converted to ASCII 
> format, and field descriptions reworked to comply with this proposed IETF standard:
> 
>   **Describing Protocol Data Units with Augmented Packet Header Diagrams**
>   
>   https://tools.ietf.org/html/draft-mcquistin-augmented-ascii-diagrams-02



# Packet Structure

## Table of Contents

**[Packet Structure](#packet-tructure)**

**[Data Packets](#data-packets)**

**[Control Packets](#control-packets)**

**[Handshake Packets](handshake-packets)**
> 
> [KM Error Packets](#km-error-response-packets)
> 
> [ACK Packets](#ack-packets)
> 
> [Keep-alive Packets](#keep-alive-packets)
> 
> [NAK Control Packets](#nak-control-packets)
> 
> [SHUTDOWN Control Packets](#shutdown-control-packets)
> 
> [ACKACK Control Packets](#ackack-control-packets)
> 
> [Extended Control Message Packets](#extended-control-message-packets)



SRT maintains UDT's UDP packet structure, but with some modifications.
Refer to Section 2 of the UDT IETF Internet Draft (draft-gg-udt-03.txt)
for more details.

Every UDP packet carrying SRT traffic contains an SRT header
(immediately after the UDP header). In all versions, the SRT header
contains four major 32-bit fields:

  -  PH_SEQNO
  -  PH_MSGNO
  -  PH_TIMESTAMP
  -  PH_ID

SRT has two kinds of packets, where the first bit in the PH_SEQNO field
in the packet header distinguishes between data (0) and control (1)
packets.

**NOTE**: *Changes in the packet structure were introduced in SRT
version 1.3.0. For the purpose of promoting compatibility with earlier
versions, old and new packet structures are presented here. Packet
diagrams in this document are in network bit order (big-endian).*


## Data Packets

### libsrt 1.1.x

An SRT Data packet based on libsrt 1.1.x (HSv4 UDT4 + SRT Extensions) 
is formatted as follows:

        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |            SrcPort            |            DstPort            |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ UDP Header
       |              Len              |            ChkSum             |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |0|                    Packet Sequence Number                   |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |FF |O|KK |                   Message Number                    |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ UDT Header
       |                           Time Stamp                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                     Destination Socket ID                     |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |                                                               :
       :                              Data                             :
       :                                                               |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   where:

   Source Port (SrcPort): 16 bits.  Value: ???
   This is a fixed-width field . . .

   Destination Port (DstPort): 16 bits.  Value: ???
   This is a fixed-width field . . .

   Length (Len): 16 bits.  Value: ???
   This is a fixed-width field . . .
   
   Check Sum (ChkSum): 16 bits.  Value: ???
   This is a fixed-width field . . .
   
   Packet Type ( ): 1 bit.  Value: 0
   This is a fixed-width field used to distinguish between 
   data (0) and control (1) packets.

   Packet Sequence Number ( ): 31 bits.  Value: 1..2^31-1
   This is a fixed-width field . . .
   
   Position (FF): 2 bits.  Value: ???
   This is a fixed-width field providing the position of a packet 
   in a message, where:

      10 = 1st
      00 = middle
      01 = last
      11 = single
   
   Delivery (O): 1 bits.  Value: ???
   This is a fixed-width field that indicates whether the message 
   should be delivered in order (1) or not (0). In File/Message mode (original UDT with 
   UDT_DGRAM) when this bit is clear then a message that is sent later (but reassembled 
   before an earlier message which may be incomplete due to packet loss) is allowed to be 
   delivered immediately, without waiting for the earlier message to be completed. This 
   is not used in Live mode because there‘s a completely different function used for 
   data extraction when TSBPD mode is on.
   
   Data Encrypted (KK): 2 bits.  Value: ???
   This is a fixed-width field that indicates whether or not 
   data is encrypted:

      00: not encrypted
      01: encrypted with even key
      10: encrypted with odd key
   
   Message Number (MsgNum): 26 bits.  Value: 1..2^27-1
   This is a fixed-width field . . .

   Time Stamp (TS): 32 bits.  Value: ???
   This is a fixed-width field usually containing the time (in microseconds) when a 
   packet was sent, although the real interpretation may vary depending on the type.

   Destination Socket ID (DestSockID): 32 bits.  Value: ???
   This is a fixed-width field providing the socket ID to which a packet should be 
   dispatched, although it may have the special value 0 when the packet is a 
   connection request.

   Data: n bits.  Value: ???
   This is a variable-width field . . .

### >=libsrt 1.3.0

An SRT Data packet based on versions of libsrt >=1.3.0 is formatted as follows:

        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |            SrcPort            |            DstPort            |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ UDP Header
       |              Len              |            ChkSum             |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |0|                    Packet Sequence Number                   |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |FF |O|KK |R|                  Message Number                   |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ UDT Header
       |                           Time Stamp                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                     Destination Socket ID                     |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |                                                               :
       :                              Data                             :
       :                                                               |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   where:

   Source Port (SrcPort): 16 bits.  Value: ???
   This is a fixed-width field . . .

   Destination Port (DstPort): 16 bits.  Value: ???
   This is a fixed-width field . . .

   Length (Len): 16 bits.  Value: ???
   This is a fixed-width field . . .
   
   Check Sum (ChkSum): 16 bits.  Value: ???
   This is a fixed-width field . . .
   
   Packet Type ( ): 1 bit.  Value: 0
   This is a fixed-width field used to distinguish between 
   data (0) and control (1) packets.

   Packet Sequence Number (PktSeqNum): 31 bits.  Value: 1..2^31-1
   This is a fixed-width field . . .
   
   Position (FF): 2 bits.  Value: ???
   This is a fixed-width field providing the position of a packet 
   in a message, where:

      10 = 1st
      00 = middle
      01 = last
      11 = single
   
   Delivery (O): 1 bits.  Value: ???
   This is a fixed-width field that indicates whether the message 
   should be delivered in order (1) or not (0). In File/Message mode (original UDT with 
   UDT_DGRAM) when this bit is clear then a message that is sent later (but reassembled 
   before an earlier message which may be incomplete due to packet loss) is allowed to be 
   delivered immediately, without waiting for the earlier message to be completed. This 
   is not used in Live mode because there‘s a completely different function used for 
   data extraction when TSBPD mode is on.
   
   Data Encrypted (KK): 2 bits.  Value: ???
   This is a fixed-width field that indicates whether or not 
   data is encrypted:

      00: not encrypted
      01: encrypted with even key
      10: encrypted with odd key
   
   Retransmitted (R): 1 bit. This is a fixed-width field/flag is clear (0) when a packet 
   is transmitted the very first time, and is set (1) if the packet is retransmitted.
  
   Message Number (MsgNum): 26 bits.  Value: 1..2^27-1
   This is a fixed-width field . . .

   Time Stamp (TS): 32 bits.  Value: ???
   This is a fixed-width field usually containing the time (in microseconds) when a 
   packet was sent, although the real interpretation may vary depending on the type.

   Destination Socket ID (DestSockID): 32 bits.  Value: ???
   This is a fixed-width field providing the socket ID to which a packet should be 
   dispatched, although it may have the special value 0 when the packet is a 
   connection request.

   Data: n bits.  Value: ???
   This is a variable-width field . . .


## Control Packets

An SRT Control packet is formatted as follows (UDP header not shown):

        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |1|            Type             |           Reserved            |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                        Additional Info                        |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ UDT Header
       |                           Time Stamp                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                     Destination Socket ID                     |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |                                                               :
       :                   Control Information Field                   :
       :                                                               |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   where:
   
   Packet Type ( ): 1 bit. Value: 0
   This is a fixed-width field used to distinguish between data (0) and 
   control (1) packets.

   Type ( ): 15 bits.  Value: ???
   This is a fixed-width field used to indicate message type 
   (see table below)

   Reserved ( ): 16 bits.  Value: ???
   This is a fixed-width field used to indicate message extended type 
   (see table below)

   Additional info ( ): 32 bits.  Value: ???
   This is a fixed-width field used in some control messages 
   as extra space for data. Its interpretation depends on the particular message. 
   Handshake messages don‘t use it (undefined???).

   Time Stamp (TS): 32 bits.  Value: ???
   This is a fixed-width field usually containing the time (in microseconds) when a 
   packet was sent, although the real interpretation may vary depending on the type.

   Destination Socket ID (DestSockID): 32 bits.  Value: ???
   This is a fixed-width field providing the socket ID to which a packet should be 
   dispatched, although it may have the special value 0 when the packet is a 
   connection request.

   Control Information Field (CIF): n bits.  Value: ???
   This is a variable-width field . . .

For Control packets the first three fields (Word 0) are interpreted respectively
(using network bit order) as:

  -  Word 0:    
      -  Bit 0: Packet Type (set to 1 for control packets)    
      -  Bits 1-15: Message Type    
      -  Bits 16-31: Message Extended type

| **Type** | **Extended Type** | **Description**                                 |
| -------- | ----------------- | ----------------------------------------------- |
| 0        | 0                 | HANDSHAKE                                       |
| 1        | 0                 | KEEPALIVE                                       |
| 2        | 0                 | ACK                                             |
| 3        | 0                 | NAK (Loss Report)                               |
| 4        | 0                 | Congestion Warning                              |
| 5        | 0                 | Shutdown                                        |
| 6        | 0                 | ACKACK                                          |
| 7        | 0                 | Drop Request                                    |
| 8        | 0                 | Peer Error                                      |
| 0x7FFF   | -                 | Message Extension: defined by Reserved field    |
| 0x7FFF   | 1                 | SRT_HSREQ: SRT Handshake Request                |
| 0x7FFF   | 2                 | SRT_HSRSP: SRT Handshake Response               |
| 0x7FFF   | 3                 | SRT_KMREQ: Encryption Keying Material Request   |
| 0x7FFF   | 4                 | SRT_KMRSP: Encryption Keying Material Response  |

The Extended Message mechanism is theoretically open for further
extensions. SRT uses some of them for its own purposes. This will be
referred to later in the section on the SRT Extended Handshake.

## Handshake Packets

Handshake control packets (“packet type” bit = 1) are used to establish
a connection between two peers in a point-to-point SRT session. Original
versions of SRT relied on handshake extensions to exchange certain
parameters immediately after a connection was opened, but as of version
1.3 an integrated mechanism ensures all parameters are exchanged as part
of the handshake itself. 


### libsrt 1.1.x

An SRT Handshake packet based on libsrt 1.1.x (HSv4 UDT4 + SRT Extensions) is formatted 
as follows:

        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |1|            Type             |           Reserved            |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                        Additional Info                        |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ SRT Header
       |                           Time Stamp                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                     Destination Socket ID                     |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |                          UDT Version                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                          Socket Type                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                 Initial Packet Sequence Number                |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                      Maximum Packet Size                      |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ CIF
       |                   Maximum Flow Window Size                    |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                        Connection Type                        |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                           Socket ID                           |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                           SYN Cookie                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                                                               :
       :                        Peer IP Address                        :
       :                                                               |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---

   where:
   
   Packet Type ( ): 1 bit. Value: 0
   This is a fixed-width field used to distinguish between data (0) and 
   control (1) packets.

   Type ( ): 15 bits.  Value: ???
   This is a fixed-width field used to indicate message type 


   Reserved ( ): 16 bits.  Value: ???
   This is a fixed-width field used to indicate message extended type 


   Additional info ( ): 32 bits.  Value: ???
   This is a fixed-width field used in some control messages 
   as extra space for data. Its interpretation depends on the particular message. 
   Handshake messages don‘t use it.

   Time Stamp (TS): 32 bits.  Value: ???
   This is a fixed-width field usually containing the time (in microseconds) when a 
   packet was sent, although the real interpretation may vary depending on the type.

   Destination Socket ID (DestSockID): 32 bits.  Value: ???
   This is a fixed-width field providing the socket ID to which a packet should be 
   dispatched, although it may have the special value 0 when the packet is a 
   connection request.

   UDT Version ( ): 32 bits. {4}  This is a fixed-width field . . .

   Socket Type (SockType): 32 bits.  Value: {SOCK_STREAM(1), SOCK_DGRAM(2)} 
   This is a fixed-width field . . .  SOCK_STREAM is not used

   Initial Packet Sequence Number (ISN): 32 bits.  Value: ???
   This is a fixed-width field . . .

   Maximum Packet Size (MSS): 32 bits.  Value: ???
   This is a fixed-width field . . .

   Maximum Flow Window Size (FC): 32 bits.  Value: ???
   This is a fixed-width field . . .

   Connection Type ( ): 32 bits.  Value: ???
   This is a fixed-width field . . .

   Socket ID ( ): 32 bits.  Value: ???
   This is a fixed-width field . . .

   SYN Cookie ( ): 32 bits.  Value: ???
   This is a fixed-width field . . .

   Peer IP Address ( ): n bits.  Value: ???
   This is a ???-width field . . . should be fixed width to accommodate IPv4
   or IPv6 addresses


#### HSv4 SRT Extended Handshake (HS) Packet

An SRT HSv4 Extended Handshake packet based on libsrt 1.1.x (HSv4 UDT4 + SRT Extensions) 
is formatted as follows:

        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |1|            Type             |              Ext              |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                        Additional Info                        |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ 
       |                           Time Stamp                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                     Destination Socket ID                     |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                          SRT Version                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                           SRT Flags                           |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |          TsbPd Resv           |          TsbPdDelay           |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                           Reserved                            |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ 
       |                                                               |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                                                               |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   where:
   
   Packet Type ( ): 1 bit. Value: 0
   This is a fixed-width field used to distinguish between data (0) and 
   control (1) packets.

   Type ( ): 15 bits.  Value: ???
   This is a fixed-width field used to indicate message type 

   Ext: 16 bits. Value: {HSREQ(1), HSRSP(2)} 
   This is a fixed-width field used to describe a handshake extension.

   Additional info ( ): 32 bits. Value: {undefined} 
   This is a fixed-width field used in some control messages as extra space for data. 
   Its interpretation depends on the particular message. 
   Handshake messages don‘t use it.

   Time Stamp (TS): 32 bits.  Value: ???
   This is a fixed-width field usually containing the time (in microseconds) when a 
   packet was sent, although the real interpretation may vary depending on the type.

   Destination Socket ID (DestSockID): 32 bits.  Value: ???
   This is a fixed-width field providing the socket ID to which a packet should be 
   dispatched, although it may have the special value 0 when the packet is a 
   connection request.

   SRT Version ( ): 32 bits. Value: {<10300h}  
   This is a fixed-width field . . .

   SRT Flags ( ): 32 bits.  Value: ???
   This is a fixed-width field . . .

   TsbPd Resv ( ): 16 bits.  Value: ???
   This is a fixed-width field . . . Time-stamp based Packet 
   delivery Reserved

   TsbPdDelay ( ): 16 bits.  Value: ???
   This is a fixed-width field . . . Time-stamp based Packet 
   delivery Delay

   Reserved ( ): ??? bits. Value: {0} 
   This is a ???-width field . . . 
   


#### HSv4 SRT Extended Keying Material (KM) Packet

An SRT HSv4 Extended Handshake packet based on libsrt 1.1.x (HSv4 UDT4 + SRT Extensions) 
is formatted as follows:

        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |1|            Type             |            Reserved           |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                        Additional Info                        |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ 
       |                           Time Stamp                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                     Destination Socket ID                     |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |0|  V  |   Pt  |              Sign             |   Resv    |KK |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                              KEKI                             |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |     Cipher    |     Auth      |      SE       |     Resv1     |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |             Recv2             |     Slen/4    |     klen/4    |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ kmext
       |                                                               :
       :                             Salt                              :
       :                                                               |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                                                               :
       :                    Wrap[((KK+1/2)*Klen)+8]                    :
       :                                                               |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---


   where:
   
   Packet Type ( ): 1 bit. Value: 0
   This is a fixed-width field used to distinguish between data (0) and 
   control (1) packets.

   Type ( ): 15 bits.  Value: {0x7fff}
   This is a fixed-width field used to indicate message type 

   Reserved ( ): 16 bits. Value: {KMREQ(3), KMRSP(4)} 
   This is a fixed-width field used to describe a handshake extension???

   Additional info ( ): 32 bits. Value: {undefined} 
   This is a fixed-width field used in some control messages as extra space for data. 
   Its interpretation depends on the particular message. 
   Handshake messages don‘t use it.

   Time Stamp (TS): 32 bits.  Value: ???
   This is a fixed-width field usually containing the time (in microseconds) when a 
   packet was sent, although the real interpretation may vary depending on the type.

   Destination Socket ID (DestSockID): 32 bits.  Value: ???
   This is a fixed-width field providing the socket ID to which a packet should be 
   dispatched, although it may have the special value 0 when the packet is a 
   connection request.

   0 ( ): 1 bit. Value: {0}  
   This is a fixed-width field . . .

   V ( ): 2 bits. Value: {1}  
   This is a fixed-width field . . .

   PT ( ): 3 bits. Value: {2}  
   This is a fixed-width field . . .

   Sign ( ): ??? bits. Value: {2029h}  
   This is a fixed-width field . . .

   Resv ( ): ??? bits. Value: {0}  
   This is a fixed-width field . . .

   Data Encrypted (KK): 2 bits.  Value: ???
   This is a fixed-width field that indicates whether or not 
   data is encrypted:

      00: not encrypted
      01: encrypted with even key
      10: encrypted with odd key   

   KEKI ( ): 32 bits. Value: {0}  
   This is a fixed-width field . . .

   Cipher ( ): 8 bits. Value: {2}  
   This is a fixed-width field . . .

   Auth ( ): 8 bits. Value: {0}  
   This is a fixed-width field . . .

   SE ( ): 8 bits. Value: {2}  
   This is a fixed-width field . . .

   Resv1 ( ): 8 bits. Value: {0}  
   This is a fixed-width field . . .

   Recv2 ( ): 16 bits. Value: {0}  
   This is a fixed-width field . . .

   Slen/4 ( ): 8 bits. Value: {4}  
   This is a fixed-width field . . . Slen(bytes)/4

   klen/4 ( ): 8 bits. Value: {4,6,8}  
   This is a fixed-width field . . . klen(bytes)/4

   Salt[Slen] ( ): ??? bits. Value: { }  
   This is a variable-width field . . . 

   Wrap[((KK+1/2)*Klen)+8] ( ): ??? bits. Value: { }  
   This is a variable-width field . . .


### libsrt >=1.3.0

An SRT Handshake version 5 (HSv5) packet based on libsrt >=1.3.0 is formatted 
as follows:

        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |1|            Type             |           Reserved            |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                        Additional Info                        |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                           Time Stamp                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                     Destination Socket ID                     |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                          UDT Version                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |            EncrFld            |            ExtFld             |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                 Initial Packet Sequence Number                |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                      Maximum Packet Size                      |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                   Maximum Flow Window Size                    |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                        Connection Type                        |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                           Socket ID                           |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                           SYN Cookie                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                                                               :
       :                        Peer IP Address                        :
       :                                                               |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |            Ext Type           |           Ext Size            |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                          SRT Version                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                           SRT Flags                           |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |          TsbPd Resv           |          TsbPdDelay           |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |            Ext Type           |           Ext Size            |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |0|  V  |   Pt  |              Sign             |   Resv    |KK |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                              KEKI                             |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |     Cipher    |     Auth      |      SE       |     Resv1     |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |             Recv2             |     Slen/4    |     klen/4    |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                                                               :
       :                             Salt                              :
       :                                                               |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                                                               :
       :                    Wrap[((KK+1/2)*Klen)+8]                    :
       :                                                               |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   where:
   
   Packet Type ( ): 1 bit. Value: 0
   This is a fixed-width field used to distinguish between data (0) and 
   control (1) packets.

   Type ( ): 15 bits.  Value: ???
   This is a fixed-width field used to indicate message type 


   Reserved ( ): 16 bits.  Value: ???
   This is a fixed-width field used to indicate message extended type 


   Additional info ( ): 32 bits.  Value: ???
   This is a fixed-width field used in some control messages 
   as extra space for data. Its interpretation depends on the particular message. 
   Handshake messages don‘t use it.

   Time Stamp (TS): 32 bits.  Value: ???
   This is a fixed-width field usually containing the time (in microseconds) when a 
   packet was sent, although the real interpretation may vary depending on the type.

   Destination Socket ID (DestSockID): 32 bits.  Value: ???
   This is a fixed-width field providing the socket ID to which a packet should be 
   dispatched, although it may have the special value 0 when the packet is a 
   connection request.

   UDT Version ( ): 32 bits. {4}  This is a fixed-width field . . .

   EncrFld ( ): 16 bits.  Value: {0,2,3,4} 
   This is a fixed-width field . . .  

   ExtFld ( ): 16 bits.  Value: {0.7,4a17h} 
   This is a fixed-width field . . .  

   Initial Packet Sequence Number (ISN): 32 bits.  Value: ???
   This is a fixed-width field . . .

   Maximum Packet Size (MSS): 32 bits.  Value: ???
   This is a fixed-width field . . .

   Maximum Flow Window Size (FC): 32 bits.  Value: ???
   This is a fixed-width field . . .

   Connection Type ( ): 32 bits.  Value: ???
   This is a fixed-width field . . .

   Socket ID ( ): 32 bits.  Value: ???
   This is a fixed-width field . . .

   SYN Cookie ( ): 32 bits.  Value: ???
   This is a fixed-width field . . .

   Peer IP Address ( ): n bits.  Value: ???
   This is a ???-width field . . . should be fixed width to accommodate IPv4
   or IPv6 addresses
   
   Ext Type: 16 bits. Value: { } 
   This is a fixed-width field used to describe a handshake extension.
   Ext Type = SRT_CMD_HSREQ(1)
   
   Ext Size: 16 bits. Value: {3} 
   This is a fixed-width field used to describe a handshake extension.

   SRT Version ( ): 32 bits. Value: {>=10300h}  
   This is a fixed-width field . . .

   SRT Flags ( ): 32 bits.  Value: ???
   This is a fixed-width field . . .

   TsbPd Resv ( ): 16 bits.  Value: ???
   This is a fixed-width field . . . Time-stamp based Packet 
   delivery Reserved
   TsbPd Resv = 0

   TsbPdDelay ( ): 16 bits.  Value: {20..8000}
   This is a fixed-width field . . . Time-stamp based Packet 
   delivery Delayc   
   Ext Type: 16 bits. Value: { } 
   This is a fixed-width field used to describe a handshake extension.
   Ext Type = SRT_CMD_KMREQ(3)
   
   Ext Size: 16 bits. Value: { } 
   This is a fixed-width field used to describe a handshake extension.
   Ext Size(bytes/4)

   0 ( ): 1 bit. Value: {0}  
   This is a fixed-width field . . .

   V ( ): 2 bits. Value: {1}  
   This is a fixed-width field . . .

   PT ( ): 3 bits. Value: {2}  
   This is a fixed-width field . . .

   Sign ( ): ??? bits. Value: {2029h}  
   This is a fixed-width field . . .

   Resv ( ): ??? bits. Value: {0}  
   This is a fixed-width field . . .

   Data Encrypted (KK): 2 bits.  Value: ???
   This is a fixed-width field that indicates whether or not 
   data is encrypted:

      00: not encrypted
      01: encrypted with even key
      10: encrypted with odd key   

   KEKI ( ): 32 bits. Value: {0}  
   This is a fixed-width field . . .

   Cipher ( ): 8 bits. Value: {2}  
   This is a fixed-width field . . .

   Auth ( ): 8 bits. Value: {0}  
   This is a fixed-width field . . .

   SE ( ): 8 bits. Value: {2}  
   This is a fixed-width field . . .

   Resv1 ( ): 8 bits. Value: {0}  
   This is a fixed-width field . . .

   Recv2 ( ): 16 bits. Value: {0}  
   This is a fixed-width field . . .

   Slen/4 ( ): 8 bits. Value: {4}  
   This is a fixed-width field . . . Slen(bytes)/4

   klen/4 ( ): 8 bits. Value: {4,6,8}  
   This is a fixed-width field . . . klen(bytes)/4

   Salt[Slen] ( ): ??? bits. Value: { }  
   This is a variable-width field . . . 

   Wrap[((KK+1/2)*Klen)+8] ( ): ??? bits. Value: { }  
   This is a variable-width field . . .

## KM Error Response Packets

Key Message Error Response control packets (“packet type” bit = 1) are
used to exchange error status messages between peers. Refer to the
**Encryption** section later in this document for details.

### libsrt 1.1.x

#### HSv4 SRT Extended Keying Material (KM) Packet

An SRT HSv4 SRT Extended Keying Material (KM) packet based on libsrt 1.1.x 
is formatted as follows:

        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |1|            Type             |            Reserved           |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                        Additional Info                        |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ 
       |                           Time Stamp                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                     Destination Socket ID                     |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                             KmErr                             |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   where:
   
   Packet Type ( ): 1 bit. Value: 0
   This is a fixed-width field used to distinguish between data (0) and 
   control (1) packets.

   Type ( ): 15 bits.  Value: {0x7fff}
   This is a fixed-width field used to indicate message type 

   Reserved ( ): 16 bits. Value: {KMRSP(4)} 
   This is a fixed-width field used to describe a handshake extension.

   Additional info ( ): 32 bits. Value: {undefined} 
   This is a fixed-width field used in some control messages as extra space for data. 
   Its interpretation depends on the particular message. 
   Handshake messages don‘t use it.

   Time Stamp (TS): 32 bits.  Value: ???
   This is a fixed-width field usually containing the time (in microseconds) when a 
   packet was sent, although the real interpretation may vary depending on the type.

   Destination Socket ID (DestSockID): 32 bits.  Value: ???
   This is a fixed-width field providing the socket ID to which a packet should be 
   dispatched, although it may have the special value 0 when the packet is a 
   connection request.

   KmErr ( ): 32 bits.  Value: {SRT_KM_NOSECRET,SRT_KM_BADSECRET}
   This is a fixed-width field 

### libsrt >=1.3.0

#### HSv5 KM Error Extension

An SRT HSv5 KM Error Extension packet based on libsrt 1.1.x 
is formatted as follows:

        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |            Ext Type           |           Ext Size            |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                             KmErr                             |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   where:
   
   Ext Type: 16 bits. Value: { } 
   This is a fixed-width field used to describe a handshake extension.
   Ext Type = SRT_CMD_KMRSP(4)
   
   Ext Size: 16 bits. Value: {1} 
   This is a fixed-width field used to describe a handshake extension.

   KmErr: 32 bits. Value: { } 
   This is a fixed-width field used to describe a handshake extension.


## ACK Packets

Acknowledgement (ACK) control packets (“packet type” bit = 1) are used
to provide data packet delivery status and RTT information. Refer to the
**SRT Data Transmission and Control** section later in this document for
details.

> ??? Same for libsrt 1.1.x and >=1.3.0 ???

An SRT ACK packet is formatted as follows:

        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |1|            Type             |           Reserved            |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                        Additional Info                        |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ SRT Header
       |                           Time Stamp                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                     Destination Socket ID                     |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |            Last Acknowledged Packet Sequence Number           |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                              RTT                              |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                          RTT Variance                         |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                     Available Buffer Size                     |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ACK
       |                    Packets Receiving Rate                     |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                    Estimated Link Capacity                    |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                        Receiving Rate                         |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---


   where:
   
   Packet Type ( ): 1 bit. Value: 0
   This is a fixed-width field used to distinguish between data (0) and 
   control (1) packets.

   Type ( ): 15 bits.  Value: ACK{2}
   This is a fixed-width field used to indicate message type 

   Reserved ( ): 16 bits.  Value: ???
   This is a fixed-width field used to indicate message extended type 

   Additional info ( ): 32 bits.  Value: ACK Sequence Number
   This is a fixed-width field used in some control messages 
   as extra space for data. Its interpretation depends on the particular message. 

   Time Stamp (TS): 32 bits.  Value: ???
   This is a fixed-width field usually containing the time (in microseconds) when a 
   packet was sent, although the real interpretation may vary depending on the type.

   Destination Socket ID (DestSockID): 32 bits.  Value: ???
   This is a fixed-width field providing the socket ID to which a packet should be 
   dispatched, although it may have the special value 0 when the packet is a 
   connection request.
   
   Last Acknowledged Packet Sequence Number (excluding): 32 bits. Value: { }  
   This is a fixed-width field providing the Last Acknowledged Packet Sequence Number.
   
   RTT ( ): 32 bits. Value: { }  
   This is a fixed-width field providing the Round Trip Time in microseconds.
   
   RTT Variance ( ): 32 bits. Value: { }  
   This is a fixed-width field providing the RTT variance.
   
   Available Buffer Size ( ): 32 bits. Value: { }  
   This is a fixed-width field providing the Available Buffer Size in packets.
   
   Packets Receiving Rate ( ): 32 bits. Value: { }  
   This is a fixed-width field providing the Packets Receiving Rate in packets/sec.
   
   Estimated Link Capacity ( ): 32 bits. Value: { }  
   This is a fixed-width field providing the Estimated Link Capacity
   
   Receiving Rate ( ): 32 bits. Value: { }  
   This is a fixed-width field providing the Receiving Rate in bits per second.
      
   

### Keep-alive Packets

Keep-alive control packets (“packet type” bit = 1) are exchanged
approximately every 10 ms to enable SRT streams to be automatically
restored after a connection loss.

> ??? Same for libsrt 1.1.x and >=1.3.0 ???

An SRT Keep-alive packet is formatted as follows:

        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |1|            Type             |           Reserved            |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                        Additional Info                        |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ SRT Header
       |                           Time Stamp                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                     Destination Socket ID                     |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |                              None                             |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   where:
   
   Packet Type ( ): 1 bit. Value: 0
   This is a fixed-width field used to distinguish between data (0) and 
   control (1) packets.

   Type ( ): 15 bits.  Value: ACK{2}
   This is a fixed-width field used to indicate message type 

   Reserved ( ): 16 bits.  Value: ???
   This is a fixed-width field . . . 

   Additional info ( ): 32 bits.  Value: {undefined}
   This is a fixed-width field used in some control messages 
   as extra space for data. Its interpretation depends on the particular message. 

   Time Stamp (TS): 32 bits.  Value: ???
   This is a fixed-width field usually containing the time (in microseconds) when a 
   packet was sent, although the real interpretation may vary depending on the type.

   Destination Socket ID (DestSockID): 32 bits.  Value: ???
   This is a fixed-width field providing the socket ID to which a packet should be 
   dispatched, although it may have the special value 0 when the packet is a 
   connection request.
   
   None (excluding): 32 bits. Value: {0}  
   This is a fixed-width field . . .
   







### NAK Control Packets

Negative acknowledgement (NAK) control packets (“packet type” bit = 1)
are used to signal failed data packet deliveries. Refer to the **SRT
Data Transmission and Control** section later in this document for
details.

> ??? Same for libsrt 1.1.x and >=1.3.0 ???

An SRT NAK packet is formatted as follows:

        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |1|            Type             |           Reserved            |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                        Additional Info                        |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ SRT Header
       |                           Time Stamp                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                     Destination Socket ID                     |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |0|                 Lost packet sequence number                 |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |1|                    List of lost packets                     |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ Loss List
       |0|                            Up to                            |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |0|                 Lost packet sequence number                 |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---

   where:
   
   Packet Type ( ): 1 bit. Value: 0
   This is a fixed-width field used to distinguish between data (0) and 
   control (1) packets.

   Type ( ): 15 bits.  Value: ACK{2}
   This is a fixed-width field used to indicate message type 

   Reserved ( ): 16 bits.  Value: ???
   This is a fixed-width field . . . 

   Additional info ( ): 32 bits.  Value: {undefined}
   This is a fixed-width field used in some control messages 
   as extra space for data. Its interpretation depends on the particular message. 

   Time Stamp (TS): 32 bits.  Value: ???
   This is a fixed-width field usually containing the time (in microseconds) when a 
   packet was sent, although the real interpretation may vary depending on the type.

   Destination Socket ID (DestSockID): 32 bits.  Value: ???
   This is a fixed-width field providing the socket ID to which a packet should be 
   dispatched, although it may have the special value 0 when the packet is a 
   connection request.
   
   0 ( ): 1 bits. Value: {0}  
   This is a fixed-width field . . .
   
   Lost packet sequence number ( ): 31 bits. Value: {2h}  
   This is a fixed-width field . . .
   
   1 ( ): 1 bits. Value: {1}  
   This is a fixed-width field . . .
   
   List of lost packets ( ): 31 bits. Value: {2h}  
   This is a fixed-width field . . .
   
   0 ( ): 1 bits. Value: {0}  
   This is a fixed-width field . . .
   
   Up to ( ): 31 bits. Value: {Bh}  
   This is a fixed-width field . . .
   
   0 ( ): 1 bits. Value: {0}  
   This is a fixed-width field . . .
   
   Lost packet sequence number ( ): 31 bits. Value: {Eh}  
   This is a fixed-width field . . .
   
   








### SHUTDOWN Control Packets

Shutdown control packets (“packet type” bit = 1) are used to initiate
the closing of an SRT connection.

> ??? Same for libsrt 1.1.x and >=1.3.0 ???

An SRT SHUTDOWN Control packet is formatted as follows:

        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |1|            Type             |           Reserved            |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                        Additional Info                        |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ SRT Header
       |                           Time Stamp                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                     Destination Socket ID                     |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |                              None                             |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   where:
   
   Packet Type ( ): 1 bit. Value: 0
   This is a fixed-width field used to distinguish between data (0) and 
   control (1) packets.

   Type ( ): 15 bits.  Value: SHUTDOWN{5}
   This is a fixed-width field used to indicate message type 

   Reserved ( ): 16 bits.  Value: ???
   This is a fixed-width field . . . 

   Additional info ( ): 32 bits.  Value: {undefined}
   This is a fixed-width field used in some control messages 
   as extra space for data. Its interpretation depends on the particular message. 

   Time Stamp (TS): 32 bits.  Value: ???
   This is a fixed-width field usually containing the time (in microseconds) when a 
   packet was sent, although the real interpretation may vary depending on the type.

   Destination Socket ID (DestSockID): 32 bits.  Value: ???
   This is a fixed-width field providing the socket ID to which a packet should be 
   dispatched, although it may have the special value 0 when the packet is a 
   connection request.
   
   None (excluding): 32 bits. Value: {0}  
   This is a fixed-width field . . .
   

### ACKACK Control Packets

ACKACK control packets (“packet type” bit = 1) are used to acknowledge
the reception of an ACK, and are instrumental in the ongoing calculation
of RTT. Refer to the **SRT Data Transmission and Control** section later
in this document for details.

> ??? Same for libsrt 1.1.x and >=1.3.0 ???

An SRT ACKACK Control packet is formatted as follows:

        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |1|            Type             |           Reserved            |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                        Additional Info                        |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ SRT Header
       |                           Time Stamp                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                     Destination Socket ID                     |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |                              None                             | CIF
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---

   where:
   
   Packet Type ( ): 1 bit. Value: 0
   This is a fixed-width field used to distinguish between data (0) and 
   control (1) packets.

   Type ( ): 15 bits.  Value: ACKACK{6}
   This is a fixed-width field used to indicate message type 

   Reserved ( ): 16 bits.  Value: ???
   This is a fixed-width field . . . 

   Additional info ( ): 32 bits.  Value: {ACK Sequence Number}
   This is a fixed-width field used in some control messages 
   as extra space for data. Its interpretation depends on the particular message. 

   Time Stamp (TS): 32 bits.  Value: ???
   This is a fixed-width field usually containing the time (in microseconds) when a 
   packet was sent, although the real interpretation may vary depending on the type.

   Destination Socket ID (DestSockID): 32 bits.  Value: ???
   This is a fixed-width field providing the socket ID to which a packet should be 
   dispatched, although it may have the special value 0 when the packet is a 
   connection request.
   
   None ( ): 32 bits. Value: {0}  
   This is a fixed-width field . . .
   

### Extended Control Message Packets

Extended Control Message packets (“packet type” bit = 1) are repurposed
from the original UDT User control packets. They are used in the SRT
extended handshake, either through separate messages, or inside the
handshake. Note that they are not intended to be used as user
extensions.

> ??? Same for libsrt 1.1.x and >=1.3.0 ???

An SRT Extended Control Message packet is formatted as follows:

        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |1|            Type             |           Reserved            |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                        Additional Info                        |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ SRT Header
       |                           Time Stamp                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                     Destination Socket ID                     |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
       |                                                               :
       :                   Control Information Field                   :
       :                                                               |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   where:
   
   Packet Type ( ): 1 bit. Value: 0
   This is a fixed-width field used to distinguish between data (0) and 
   control (1) packets.

   Type ( ): 15 bits.  Value: USER{7fffh}
   This is a fixed-width field used to indicate message type 

   Reserved ( ): 16 bits.  Value: ???
   This is a fixed-width field . . . 

   Additional info ( ): 32 bits.  Value: {user defined}
   This is a fixed-width field used in some control messages 
   as extra space for data. Its interpretation depends on the particular message. 

   Time Stamp (TS): 32 bits.  Value: ???
   This is a fixed-width field usually containing the time (in microseconds) when a 
   packet was sent, although the real interpretation may vary depending on the type.

   Destination Socket ID (DestSockID): 32 bits.  Value: ???
   This is a fixed-width field providing the socket ID to which a packet should be 
   dispatched, although it may have the special value 0 when the packet is a 
   connection request.
   
   Control Information Field ( ): N bits. Value: {user defined}  
   This is a variable-width field . . .
   




