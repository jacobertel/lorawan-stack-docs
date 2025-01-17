---
title: "Troubleshooting Devices"
description: ""
---

This section provides help for common issues and frequently asked questions you may have when adding devices. 

<!--more-->

### I receive an error when registering an end device.

Here are some common errors and solutions:

- **ID already taken**: Another end device in the application is using this ID. Choose a different Device ID. See [ID and EUI constraints]({{< ref "reference/id-eui-constraints" >}}) for more information.
- **An end device with JoinEUI `<join-eui>` and DevEUI `<dev-eui>` is already registered as `<device-id>` in application `<application-id>`**: Within {{% tts %}} deployment, only a single end device can be registered with a certain combination of DevEUI and JoinEUI/AppEUI. If you previously registered an end device with this combination of DevEUI and JoinEUI, you can delete the existing device and repeat the registration. You can also register the device using another JoinEUI, but keep in mind that you will need to configure that JoinEUI in the device as well.
- **An end device with JoinEUI `<join-eui>` and DevEUI `<dev-eui>` is already registered in another tenant**: Within {{% tts %}} deployment, only a single end device can be registered with a certain combination of DevEUI and JoinEUI/AppEUI, across all tenants. If someone already registered an end device with this combination of DevEUI and JoinEUI, you can contact the manufacturer to check if your EUIs are correct and/or provide you new EUIs. You can also register the device using another JoinEUI, but keep in mind that you will need to configure that JoinEUI in the device as well.
- **Duplicate identifiers**: This error occurs when an end device is deleted from the Identity Server, but the device entry persists in the Join Server, Network Server and Application Server databases. In this case, the device will not appear in {{% tts %}} Console, so you need to check if it is present in the Join Server/Network Server/Application Server components using the following command:

    ```bash
    curl -X 'GET' 'https://thethings.example.com/api/v3/<js/ns/as>/applications/<application_id>/devices/<end-device_id>' -H 'accept: application/json' -H 'Authorization: Bearer <api-key>'
    ```

    If the device entry is present in the above mentioned components, you can delete it with:

    ```bash
    curl 'https://thethings.example.com/api/v3/<js/ns/as>/applications/<application_id>/devices/<end-device_id>' -X DELETE -H 'Accept: application/json' -H 'Authorization: Bearer <api-key>'
    ```

### How do I see device events?

Device event logs can be found in the console in the device's general information page. See [Working with Events]({{< ref "getting-started/events" >}}) for other ways of subscribing to events.

### My device won't join. What do I do?

- Double check your DevEUI, JoinEUI or AppEUI, LoRaWAN and Regional Parameters Version, root keys (AppKey, and with LoRaWAN 1.1 or higher, NwkKey)
- Check gateway and device events for traffic from your device. Below are a few common issues
    - **MIC mismatch** error in the device events: Possible mismatch of AppKey in device firmware to the AppKey registered in {{% tts %}} console. Update the AppKey in the console accordingly
    - **Uplink channel not found** error in the gateway events: Indicates there is a mismatch of the frequency plans. Double check frequency plan settings in your end device and gateways (they must be the same LoRaWAN band)
    - **DevNonce has already been used** error in the device events: Indicates a duplicate use of Devnonce. Generally happens when the device has sent too many unsuccessful Join Requests
    - **Uplink channel not found** error in the device events: The device is transmitting Join Requests in the non-default channels of the band which is not in line with the LoRaWAN Specification. Contact the end-device manufacturer
- Double check your network connection. If there is a slow connection from the server to the gateway, the join accept message may be sent too late (this can happen when a gateway uses 3G as a backhaul). If using the CLI, run `ttn-lw-cli gateways connection-stats <gateway-id>` to see the round trip time (RTT) for your gateway
- Check for duplicate use of JoinNonce (or AppNonce)
- Adjust ADR and link check settings to conditions which the device is located in

### No downlinks are reaching my device. What do I do?

- Check gateway and application events for traffic from your device
- Check duty cycle restrictions
- Device clock drift often occurs when SF12 is used
- Check your antenna connections

### {{% tts %}} is no longer receiving uplinks from my device. What do I do?

- Check [gateway events](#how-do-i-see-gateway-events) and [device events](#how-do-i-see-device-events) for traffic from your device
- Check for FCnt mismatch on ABP devices
