# Offline Gallery

## Acknowledgment

Offline Gallery is a portable platform to present browser-based projects. It is built on a Raspberry pi and can be powered by any portable battery, like the ones we use to charge our smartphones when traveling or hiking in the mountain. For that reason, thanks to its portability, this virtual gallery can be installed (or hidden) literally anywhere (forests, public and private buildings, urban and natural environments, over or underwater—using a specific antenna, etc.). The public of these exhibitions can connect to the site-specific artworks within the perimeter of the box using their own mobile devices (smartphone, tablet, laptop, etc.). 
 
All these features allow the Offline Gallery to be present in spaces that are normally not suitable (or even allowed) for the promotion of the arts—although they are often the main source of inspiration for the artists. One of the main challenges of working with interfaces in the field of artistic research is how not to end up turning their diffractive potential into mere "reflection", understood as the quality that displaces the same thing to another place. With the Offline Gallery, the motif and the voice of the artist can finally meet in the same space! Artists and their audiences can meet and express themselves freely in the "virtual" digital space, while the passerby sharing that same physical space remains undisturbed.
 
How? The Offline Gallery offers an open network. The public connected to this network will automatically receive a pop-up window, like the captive portal window in public hotspots. Previously installed in the Offline Gallery, the artwork is then shared only with the public around its perimeter. There is no need to download annoying and suspicious applications or follow a registration process, the public's interaction with the artwork is completely anonymous, leaving no trace and collecting no data. 
 
The use of the Offline Gallery as a platform for hybrid physical and digital exhibitions results in a more aesthetic, less disembodied experience of the artwork. By physically placing the audience in the context of the artwork, this project confronts the prejudice against digital artifacts as promoters of alienating and disembodied experiences. The Offline Gallery offers the possibility of merging the source of inspiration in the "real" and the artist's work in the "virtual".

## Installation

### Supported Hardware and Operating System

Offline Gallery is tested on Raspberry Pi 3B and Raspberry Pi 4.

- **Hardware:** Raspberry Pi 3B, Raspberry Pi 4 (Purchase links: [Raspberry Pi 3B](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/), [Raspberry Pi 4](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/))
- **Operating System:** Raspberry Pi OS (32-bit) is recommended. (Download: [Raspberry Pi OS](https://www.raspberrypi.org/software/))

A 32 GB SD card is suggested for optimal performance.

### Writing The Image to SD Card

Before proceeding with Offline Gallery setup, follow these steps to set up the Raspberry Pi OS:

1. Download the Raspberry Pi OS image from the official website.
2. Use a tool like BalenaEtcher or Raspberry Pi Imager to write the Raspberry Pi OS image to the SD card. Detailed instructions can be found [here](https://www.raspberrypi.org/documentation/installation/installing-images/README.md).

### Software Installation
Once the Raspberry Pi OS is set up, follow these steps to install the necessary packages for Offline Gallery:

1. Update the Raspberry Pi OS by running the following commands in the Terminal application:
    ```bash
    sudo apt update
    sudo apt upgrade -y
    ```

2. After updating, reboot your Raspberry Pi for changes to take effect:
    ```bash
    sudo reboot
    ```

3. Install the required packages for creating an access point:
    ```bash
    sudo apt install dnsmasq 
    sudo apt install hostapd
    ```

4. Disable dnsmasq and hostapd services temporarily until configurations are done:
    ```bash
    sudo systemctl stop dnsmasq
    sudo systemctl stop hostapd
    ```
### Set up Node.js and npm Dependency
Before the internet connection is established, it's necessary to install Node.js and npm.

1. Install Node.js by following the instructions provided [here](https://www.golinuxcloud.com/install-nodejs-and-npm-on-raspberry-pi/):
    ```bash
    sudo curl -fsSL https://deb.nodesource.com/setup_17.x | bash -
    sudo apt install nodejs
    ```

2. Verify the Node.js installation:
    ```bash
    node --version
    ```

3. Create a directory on the Desktop for the project:
    ```bash
    sudo mkdir -p /home/offlinegallery/Desktop/Node/PiWiFi
    ```

4. Navigate to the directory:
    ```bash
    cd /home/offlinegallery/Desktop/Node/PiWiFi
    ```

5. Initialize a Node.js project:
    ```bash
    sudo npm init
    ```

6. Install Express as a dependency:
    ```bash
    sudo npm install express
    ```

### Network Settings

Now, configure the network settings.

1. Assign a static IP address to the Raspberry Pi (e.g., `192.168.4.1`). Verify that the IP address is not in use:
    ```bash
    ping 192.168.4.1
    ```

2. Edit the `dhcpcd.conf` file to configure the static IP address:
    ```bash
    sudo nano /etc/dhcpcd.conf
    ```
    Add the following lines to the end of the file:
    ```
    interface wlan0
    static ip_address=192.168.4.1/24
    nohook wpa_supplicant
    ```

3. Restart `dhcpcd`:
    ```bash
    sudo service dhcpcd restart
    ```

4. Rename the `dnsmasq` file to remove unnecessary information:
    ```bash
    sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
    ```
5. Edit the `dnsmasq.conf` file:
    ```bash
    sudo nano /etc/dnsmasq.conf
    ```
    Add the following lines to the file:
    ```
    interface=wlan0
    dhcp-range=192.168.4.2,192.168.4.255,255.255.255.0,15m
    address=/#/192.168.4.1
    ```
6. Update the configuration:
    ```bash
    sudo systemctl reload dnsmasq
    ```

7. Configure the access point:
    ```bash
    sudo nano /etc/hostapd/hostapd.conf
    ```
    Add the following configurations to the file:
    ```
    interface=wlan0
    driver=nl80211
    ssid=Offline Gallery
    channel=7
    hw_mode=g
    ```
8. Edit the file to declare the location of the configuration file:
    ```bash
    sudo nano /etc/default/hostapd
    ```
    Replace the commented line starting with `#DAEMON_CONF`:
    ```
    DAEMON_CONF="/etc/hostapd/hostapd.conf"
    ```

9. Enable the hostapd service:
    ```bash
    sudo systemctl unmask hostapd
    sudo systemctl enable hostapd
    sudo systemctl start hostapd
    ```

10. Check the status of hostapd and dnsmasq:
    ```bash
    sudo systemctl status hostapd
    sudo systemctl status dnsmasq
    ```
    
11. Enable IP forwarding by editing the `sysctl.conf` file:
    ```bash
    sudo nano /etc/sysctl.conf
    ```
    Uncomment the line:
    ```
    net.ipv4.ip_forward=1
    ```

12. For traffic on `eth0`, run:
    ```bash
    sudo iptables -t nat -A  POSTROUTING -o eth0 -j MASQUERADE
    ```

13. Redirect all inbound traffic to `192.168.4.1:3000`:
    ```bash
    sudo iptables -t nat -I PREROUTING -d 192.168.4.1 -p tcp --dport 80 -j DNAT --to-destination 192.168.4.1:3000
    ```

14. Save the `iptables` rules:
    ```bash
    sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
    ```

15. Edit the `rc.local` file to install the `iptables` rules on boot:
    ```bash
    sudo nano /etc/rc.local
    ```
    Add the following line above `exit 0`:
    ```
    iptables-restore < /etc/iptables.ipv4.nat
    ```

16. Reboot the Raspberry Pi and check if the access point named "Offline Gallery" is visible:
    ```bash
    sudo reboot
    ```
### Configure the Node.js Server


1. Navigate to the project directory:
    ```bash
    cd /home/offnet/Desktop/Node/PiWiFi
    ```

2. Create a Node.js file:
    ```bash
    sudo nano app.js
    ```
    Add the following code:
    ```javascript
    const express = require('express');
    const app = express();
    const port = 3000;

    var hostName = 'offlinegallery.wifi';

    app.use((req, res, next) => {
        if (req.get('host') != hostName) {
            return res.redirect(`http://${hostName}`);
        }
        next();
    })

    app.get('/', (req, res, next) => {
        res.sendFile('/home/offnet/Desktop/Node/PiWiFi/offlinegallery/html/index.html');
    })

    app.use(express.static('/home/offnet/Desktop/Node/PiWiFi/offlinegallery/html/sources'));

    app.listen(port, () => {
        console.log(`Server listening on port ${port}`)
    })
    ```

3. Create directories and an HTML file:
    ```bash
    mkdir -p offlinegallery/html/sources
    cd offlinegallery/html
    touch index.html
    ```

4. Edit the `index.html` file and add your HTML content. Upload the images, videos, and PDFs into the sources directory.

 	```bash
    sudo nano index.html
    ```
 Example of an html file: 
 ```html
 <!DOCTYPE html>
<html>
  <head>
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <style>
      body {
        color: #333;
        background-image: url("sources/background.jpeg");
      }
      .header {
        margin-top: 20px;
        text-align: center;
      }
      .column {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        max-width: 70%;
        padding: 0px;
        margin: 20px auto auto auto;
      }
      .column img {
        margin-top: 15px;
        width: 100%;
        max-height: 100%;
        border: 5px solid rgb(40, 6, 209);
      }
      .artwork p {
        font-weight: bold;
        text-align: center;
      }
    </style>
  </head>
  <body>
    <div class="header">
      <img src="/welcome_to.gif" style="width: 100%" />
      <img src="./sources/cafe_gallerie.gif" style="width: 100%" />
    </div>
    <div class="column">
      <div class="artwork">
        <img src="/apples_cezanne.jpg" style="width: 100%" />
        <p>The Basket of Apples – Paul Cézanne</p>
      </div>
      <div class="artwork">
        <img src="/basquet_caravaggio.jpg" style="width: 100%" />
        <p>Basket of Fruit – Caravaggio</p>
      </div>
      <div class="artwork">
        <img src="/campbells_warhol.jpg" style="width: 100%" />
        <p>Campbell's Soup Can - Andy Warhol</p>
      </div>
      <div class="artwork">
        <img src="/still_life_matisse.jpg" style="width: 100%" />
        <p>Still Life with a Pewter Jug and Pink Statuette – Henri Matisse</p>
      </div>
      <div class="artwork">
        <img src="/sunflowers_van_gogh.jpg" style="width: 100%" />
        <p>Sunflowers – Van Gogh</p>
      </div>
      <div class="artwork">
        <img src="/vertumnus_arcimboldo.jpg" style="width: 100%" />
        <p>Vertumnus - Giuseppe Arcimboldo</p>
      </div>
      <div class="artwork">
        <img src="/violin_braque.jpg" style="width: 100%" />
        <p>Violin and Candlestick – Georges Braque</p>
      </div>
      <div class="artwork">
        <img src="white_flower_okeeffe.jpg" style="width: 100%" />
        <p>The White Flower - Georgia O'Keeffe</p>
      </div>
    </div>
  </body>
</html>
 ```
5. Create a service to run the server automatically:
    ```bash
    sudo nano offlinegallerywifi.service
    ```
    Add the following code:
    ```
    [Unit]
    Description=Offline Gallery WiFi Hotspot Service
    After=network.target

    [Service]
    WorkingDirectory=/home/offnet/Desktop/Node/PiWiFi
    ExecStart=/usr/bin/nodejs /home/offnet/Desktop/Node/PiWiFi/app.js
    Restart=on-failure
    User=root
    Environment=PORT=3000

    [Install]
    WantedBy=multi-user.target
    ```

    ```bash
    sudo cp offlinegallerywifi.service /etc/systemd/system
    sudo systemctl enable piwifi.service
    ```
    
    You can test before booting
     ```bash
    sudo systemctl start piwifi
     ```
     The server will run on the boot automatically.

When the server runs, the redirection command returns a 302 status code. It triggers sign in (captive portal page).
You can connect the "Offline Gallery" in the list of Wifi Access Points from another device. The captive portal is going to appear on your device.



### Details


#### Connecting with the SSH
The default IP Address: 192.168.4.1


Additionally, you can connect to the raspberry pi via SSH to configure the details.


#### Folder Structure
Here is an example structure of the Node file for the Captive Portal that is realized in this documentation:

```bash
├── Node
│   └── PiWiFi
│       ├── offlinegallery
│       │   ├── sources            (includes the media files)
│       │   │   └── welcome_to.gif
│       │   │   └── cafe_gallerie.gif
│       │   │   └── apples_cezanne.jpg
│       │   │   └── basquet_caravaggio.jpg
│       │   │   └── campbells_warhol.jpg
│       │   │   └── still_life_matisse.jpg
│       │   │   └── sunflowers_van_gogh.jpg
│       │   │   └── vertumnus_arcimboldo.jpg
│       │   │   └── violin_braque.jpg
│       │   │   └── white_flower_okeeffe.jpg
│       │   │   └── background.jpeg    (background image)
│       │   ├── index.html         (includes web elements)
│       ├── package.lock   
│       ├── package.json    
│       └── app.js                (Node file)
└──
```


## Contributing

Contributions are welcome! If you find any issues or have suggestions for improvement, please open an issue or submit a pull request on the GitHub repository.

## References
[RaspberryPiHotspot](https://github.com/TomHumphries/RaspberryPiHotspot)

## License

This project is licensed under the terms of the [GNU General Public License](https://www.gnu.org/licenses/gpl-3.0.en.html) (Version 3.0).
