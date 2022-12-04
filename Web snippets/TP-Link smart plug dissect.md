ref: https://www.softscheck.com/en/blog/tp-link-reverse-engineering/

Reverse Engineering the TP-Link HS110

The TP-Link HS110 Wi-Fi is a cloud-enabled power plug that can be turned on and off remotely via app and offers energy monitoring and scheduling capabilities. As part of ongoing research into Internet of Things security, we performed a security analysis by reverse engineering the device firmware and Android app, sniffing app-to-device and device-to-app communications and fuzzing the proprietary protocols being used.

While cloud communication was found to be reasonably secure for an IoT device, we discovered two insecure proprietary local configuration protocols: A human-readable JSON protocol "encrypted" with an easily reversible autokey XOR cipher and a binary DES-encrypted configuration and debugging protocol (TDDP – TP-Link Device Debug Protocol). TDDP is in use across most of the TP-Link product line including routers and access points and thus merits further research. We also release a Wireshark dissector and two python clients for the proprietary protocols on GitHub.

GitHub Icon View on GitHub