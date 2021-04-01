# Install Home Assistant and MQTT broker
Home Assistant is an OSS solution for home automation, it supports many types of devices and services,

to send messages to Home Assistant from our AI model we use MQTT protocol,

MQTT requires a broker which receives the messages and transports them to the subscribers
There are remote brokers (for a fee) but we will use a local broker called [Mosquitto](https://mosquitto.org/).
1. Create the following path:
    ```
    ~/docker/ha/config
    ~/docker/mosquitto
    ```

2. Set the paths inside `docker-compose.yml`, change "YOUR_USERNAME" to match your Jetson username, for example: "/home/guy/docker/ha/config:/config" (execute `who` to get the user name)
3. [optional] Set the "Asia/Jerusalem" to your timezone
4. Start the containers by executing the following while pointing at the repo root
    ```
    docker-compose up -d
    ```
5. Home Assistant should now be running, navigate to `http://localhost:8123`
See original instructions [here](https://www.home-assistant.io/docs/installation/docker/#docker-compose)
6. The MQTT broker is now running locally on port 1883

#### If needed, here are a few helpful docker-compose commands:

To restart the container run:

    docker-compose restart

To update the image to the latest, run:

    docker-compose pull
    docker-compose up -d --build homeassistant

### Set Home Assistant to listen to MQTT messages
1. Open Home Asisstant in your browser ([http://localhost:8123](http://localhost:8123))
2. On the left menu, navigate to "Configuration", and then "Integrations"
3. On the right bottom, click "Add integration" type in "MQTT" in the search box and select the shown option
4. For "broker" type `localhost` and for "port" leave it as `1883`, click "Submit"

#### Test Home Assistant with MQTT
1. Open Home Asisstant in your browser ([http://localhost:8123](http://localhost:8123))
2. On the left menu, navigate to "Configuration", and then "Integrations"
3. Select the MQTT box and click "configure"
4. Unser the "Listen to a topic", type "classifier" in the "topic to subscribe to" and "start listening"
5. In terminal, navigate to the [ha](ha/) directory in this repo
6. Run the `test_mqtt.py` file:

    ```
    python3 test_mqtt.py
    ```
7. Go back to Home Assistant, veryify you see "Message 0 received" and below: `Hello MQTT`
8. MQTT is now set and Home Assistant can subscribe to topics

## Configure Shelly button in Home Assistant
See [HA documentation](https://www.home-assistant.io/integrations/shelly/)
1. Open Home Asisstant in your browser ([http://localhost:8123](http://localhost:8123))
2. On the left menu, navigate to "Configuration", and then "Integrations"
3. On the right bottom, click "Add integration" and type in "Shelly", select the Shelly option
4. For localhost, type in the Shelly local ip and click "Submit"
    1. If you are not sure what the IP is, open the Shelly app, select the device and click on "Settings"
    2. Scroll down to "Device information" and expand it, the IP is listed under "Device IP"
7. The device is now read to be use in HA, you can test it by navigating to the ["Overview" page](http://localhost:8123/lovelace/default_view) and you should see the Shelly switch.

## Creating an Home Assistant automation
The next step would be creating an automation that will be triggered by an MQTT message.
1. Browse to your Home Assistant instance
2. In the sidebar click on "Configuration" and select "Automations"
3. Click on the "Add automation" button on the bottom right corner
4. Select "Start with an empty automation"
6. Configure the automation
    1. For the "Name", type in something like "Control Shelly by MQTT"
    2. Trigger
        1. For "trigger type" select "MQTT"
        2. For "topic" type in "classifier"
        3.  For "payload" type in "1"
    3. Action
        1. For "action type" select "Device"
        2. For "device" select the Shelly device from the list
        3. For "Action" select "Turn on <shelly device name>"
7. Click the "Save" button
8. Back in the terminal, navigate to the [ha/](ha/) dir in this repo
9. Edit the `test_mqtt.py` and replace the `Hello MQTT` with `1`
10. Save and close the file
11. Run it, `python3 test_mqtt.py`
12. The Shelly button should now be switched on

Try creating a similar automation for turnning it off.
**Note**: The MQTT messages from the classifier might be sent out in high rate, make sure to aggregate the events or delay the Shelly operation so it won't execute too many times in a second.
