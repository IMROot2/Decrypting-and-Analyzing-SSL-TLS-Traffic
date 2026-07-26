# Decrypting-and-Analyzing-SSL-TLS-Traffic
In this project, you'll learn how to use Wireshark to decrypt and analyze SSL/TLS traffic.
## Lab Set-up and Tools

* **Sample PCAP File:** Download a sample PCAP file containing SSL/TLS traffic.
* **Private Key File:** Obtain the private key for the SSL/TLS server used in the sample PCAP file.

### Steps to Open the Capture

1. Open Wireshark.
2. Go to **"File"** > **"Open"** and select the sample PCAP file you downloaded.
3. The file will load, and the captured traffic will be displayed.

## Traffic Capture Analysis

![Wireshark Capture: rsasnakeoil2.pcap](Screenshot%202026-07-23%20154243.png)

### Observation and Analysis
Based on the provided capture file (`rsasnakeoil2.pcap`), the following network behavior and SSL/TLS handshake processes can be analyzed:

* **Local Communication:** The traffic is routed over the loopback interface, as indicated by both the Source and Destination IP addresses being `127.0.0.1`.
* **TCP 3-Way Handshake:** Packets 1 through 3 demonstrate a standard TCP connection establishment (SYN, SYN-ACK, ACK) between the client (Port `38713`) and the server (Port `443`, the default HTTPS port).
* **SSL/TLS Handshake Initiation:** 
  * In Packet 4, the client initiates the secure session by sending an **SSLv2 Client Hello** message.
  * The server responds in Packet 6 with an **SSLv3 Server Hello**, attaches its **Certificate**, and concludes with a **Server Hello Done** message.
* **Key Exchange:** Packet 8 shows the **Client Key Exchange**, which is a critical step for securely exchanging the symmetric session keys (typically using RSA, given the file name context).
* **Encrypted Payload:** Following the **Change Cipher Spec** messages (which confirm the switch to encrypted communication), the subsequent packets (e.g., Packets 11, 19, 21) display **Application Data**. This confirms that the payload is now fully encrypted and secure.
