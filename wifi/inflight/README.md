# Inflight Wi-Fi

On a recent flight, with Wi-Fi enabled, I noticed that one of the services offered was "Free Inflight Wi-Fi". It appeared that if you either had an account with the airline or were operating with a specific mobile carrier that this service would be provided to you. However, there was something interesting. When accessing the Wi-Fi from a mobile device it seemed the user could operate their device freely, texting, watch movies, browse the internet, etc... However, upon connecting to the same Wi-Fi with a Laptop things would behave differently. While the Wi-Fi was still "free" the user would be prompted from the "Captive Portal" that they would be given a certain amount of **minutes** of connection, but an advertisement would need to be watched to "replenish" their time. While the mobile device access has no such requirement. This somewhat made sense as a laptop or "larger device" may put more of a load on the network, but this raised a question.

**How does the access point know what kind of device the user is operating?**

The thought of MAC (Media Access Control) address came to mind, since the access point/captive portal could likely determine the vendor and assume what sort of device (mobile/laptop) based on the detected address. However, MAC addresses have the ability to be modified. So, in theory, if one were to obtain an approved connection on their mobile device they could copy the MAC value and change their laptop to match the mobile device MAC. The access point would not know the different and now allow the laptop an advertisement free experience, similar to mobile. 

The theoretical flow would look like this, assuming an iPhone is in use and the user has software installed which enables the user to modify their MAC address on the laptop.

1. Connect to the Wi-Fi access point on the aircraft with mobile device and follow the "standard flow" of getting connected. 
1. Navigate to "Settings" on the iPhone.
1. Go to "Wi-Fi".
1. Click on the "i" icon on the Access Point you are connected to.
1. You should see "Wi-Fi Address". Copy the value, it will be in the format of `XX:XX:XX:XX:XX:XX`.
1. Disconnect the mobile device from the Wi-Fi network.
1. Run `ip a` and determine the name of your wireless adapter.
1. On the laptop turn off the Wi-Fi adapter.

    ```bash
    sudo ip link set dev DEVICE_ID down
    ```

1. Execute `macchanger` against the adapter.

    ```bash
    # Setting to the mobile device MAC
    sudo macchanger --mac XX:XX:XX:XX:XX:XX DEVICE_NAME
    ```
1. Re-enable the Wi-Fi device.
    
    ```bash
    sudo ip link set dev DEVICE_ID up
    ```

1. Now it should be possible to access the Wi-Fi access point and receive the same privileges which the mobile device had.

1. Your MAC address should revert to its original upon reboot, or the following can be executed.

    ```bash
    sudo ip link set dev DEVICE_ID down
    sudo macchanger -p DEVICE_ID
    sudo ip link set dev DEVICE_ID up
    ```

[Back to Home](https://blog.the1ntern.net)