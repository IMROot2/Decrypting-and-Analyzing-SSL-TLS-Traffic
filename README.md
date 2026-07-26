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
### Steps to Inspect Decrypted Traffic

1. Configure Wireshark to decrypt the SSL/TLS traffic by loading the server's private RSA key (as configured in the previous exercises).
2. Apply a display filter (e.g., `tls` or `ssl`) in the filter bar to focus on the encrypted sessions.
3. Locate a packet containing **Application Data** (e.g., Packet 11) in the packet list pane.
4. Double-click the packet to open the detailed view, or expand the packet details pane in the main window.
5. Expand the **Transport Layer Security** tree to inspect the record layer and reveal the underlying decrypted protocols.

## Decrypted Packet Analysis (Packet 11)

![Wireshark Packet Details: Packet 11](Screenshot%202026-07-23%20155507.png)

### Observation and Analysis
By examining the detailed view of Packet 11, we can observe the following results after successfully applying the SSL/TLS decryption keys:

* **Protocol Stack Visibility:** The frame encompasses several protocols, displayed sequentially at the top as `eth:ethertype:ip:tcp:tls:http`. This explicitly confirms that the encrypted TLS tunnel is carrying HTTP traffic.
* **Connection Details:** The packet is sent from the client (Source Port `38713`) to the server (Destination Port `443`) over the local interface (`127.0.0.1`).
* **SSLv3 Record Layer:** The payload is identified as **Application Data (23)** running under SSL version 3.0. 
* **Successful Decryption Verification:** The most critical observation in this capture is the presence of the `[Application Data Protocol: Hypertext Transfer Protocol]` tag, alongside the dedicated **Hypertext Transfer Protocol** tree at the bottom. This proves that Wireshark successfully used the provided private key to decrypt the ciphertext (shown in the "Encrypted Application Data" string) and successfully exposed the underlying plain-text HTTP communication.
### Steps to Follow the TCP Stream

1. Right-click on any packet belonging to the SSL/TLS session in the main packet list.
2. From the context menu, select **"Follow"** > **"TCP Stream"**.
3. A new window will open displaying the raw data exchanged during the entire TCP conversation between the client and the server.

## TCP Stream Analysis (Raw Data)

![Wireshark Follow TCP Stream](Screenshot%202026-07-23%20155833.png)

### Observation and Analysis
By examining the "Follow TCP Stream" window, we gain insight into the raw data transmitted over the wire before the application-layer decryption is fully applied to the stream view. Key observations include:

* **Traffic Direction:** The stream uses color-coding to indicate direction. The red text represents data sent from the client to the server, while the blue text represents data sent from the server back to the client.
* **Plaintext Handshake Data:** The readable blue text clearly exposes the details of the server's X.509 Digital Certificate. We can identify the Issuer/Subject details such as `Snake Oil, Ltd`, `Certificate Authority`, and `www.snakeoil.dom`. This data is visible in plain text because the server sends its certificate *before* the symmetric encryption keys are established during the TLS handshake.
* **Encrypted Payload:** The majority of the stream appears as garbled, unreadable characters (e.g., dots and random symbols). This represents the actual Application Data (the HTTP traffic) which was encrypted over the network. 
* **Conclusion:** This view proves that while the network packet capture contains the encrypted ciphertext, simply following the raw TCP stream is insufficient to read the payload. To view the readable HTTP data, one must rely on the packet details pane (as shown in the previous step) or use the "Follow TLS Stream" / "Follow HTTP Stream" features when the private key is actively applied.
### Steps to View the Decrypted HTTP Stream

1. Right-click on a decrypted packet (e.g., Packet 11) in the packet list.
2. From the context menu, select **"Follow"** > **"HTTP Stream"** (or "TLS Stream", depending on your Wireshark version).
3. A new window will open displaying the fully decrypted application-layer conversation.

## Decrypted HTTP Stream Analysis

![Wireshark Follow HTTP Stream](Screenshot%202026-07-23%20160439.png)

### Observation and Analysis
By examining the "Follow HTTP Stream" window, we can observe the ultimate result of our SSL/TLS decryption configuration. Unlike the TCP stream which showed garbled ciphertext, this view presents the communication in complete, readable plain text:

* **Cleartext Recovery:** The successful decryption allows us to view the underlying HTTP protocol exactly as it was transmitted before encryption.
* **Client Request (Red Text):** The client sends a standard `GET / HTTP/1.1` request. We can easily read the HTTP headers, such as the `Host: localhost` and the `User-Agent` indicating the request was made using a Mozilla Firefox browser on Linux.
* **Server Response (Blue Text):** The server responds with a `HTTP/1.1 200 OK` status code. The headers reveal the server's identity (`Apache/2.0.55 (Debian)`).
* **HTML Payload:** Following the response headers, the actual HTML code of the requested web page is fully visible. We can see it is a default Debian Apache placeholder page (`<TITLE>Placeholder page</TITLE>`). 
* **Conclusion:** This successfully demonstrates how possessing the server's private RSA key allows an analyst or incident responder to decrypt, reconstruct, and inspect secure TLS traffic for analysis or troubleshooting purposes.
* ## Conclusion

This project successfully demonstrates the fundamental process of intercepting and decrypting SSL/TLS network traffic. By utilizing Wireshark and a known private RSA key, we transitioned from analyzing raw, encrypted ciphertext to fully reconstructing the readable HTTP application-layer stream. 

**Key Takeaways:**
* Successfully navigated and analyzed `.pcap` files using Wireshark.
* Understood the phases of the TCP 3-way handshake and the SSL/TLS secure session establishment.
* Demonstrated how possessing the server's private key allows for the decryption of payload data.
* Highlighted the difference between raw TCP streams and decrypted Application Data streams.

This hands-on analysis serves as a practical foundation for network troubleshooting, deep packet inspection, and incident response investigations.
