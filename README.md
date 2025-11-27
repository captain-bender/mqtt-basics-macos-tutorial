# MQTT Tutorial on macOS  
## Eclipse Mosquitto Broker + Python venv + Paho MQTT

This repo is a minimal, step-by-step tutorial for **macOS** that shows how to:

1. Create and use a **Python virtual environment** (`venv`).
2. Install and run the **Eclipse Mosquitto** MQTT broker.
3. Use the **Paho MQTT** Python client library.
4. Exchange basic messages between:
   - one **subscriber** (listening), and  
   - one **publisher** (sending messages).
5. Experiment with **QoS levels** and **retained messages** in MQTT.

This tutorial assumes **no prior MQTT experience** and only very basic Terminal familiarity.

---

## 1. Prerequisites

### 1.1. Install Homebrew (recommended)

On macOS, the easiest way to install Mosquitto is with **Homebrew**.

Open **Terminal** and run:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

After installation, follow any instructions Homebrew prints (e.g., adding it to your PATH).  
Then verify:

```bash
brew --version
```

### 1.2. Install Python 3 (if needed)

Many macOS systems already have Python 3, but you can also install it via Homebrew:

```bash
brew install python
```

Check:

```bash
python3 --version
```

You should see something like:

```text
Python 3.11.x
```

---

## 2. Clone or download this repository

You can either:

- **Clone with Git**:

  ```bash
  git clone https://github.com/captain-bender/mqtt-basics-macos-tutorial.git
  cd mqtt-basics-macos-tutorial
  ```

or

- **Download as ZIP** from GitHub, unzip it, and then in Terminal `cd` into the unzipped folder.

All the following commands assume you are inside the project folder.

---

## 3. Create and activate a virtual environment (venv)

A virtual environment keeps the Python packages for this project isolated.

### 3.1. Create the venv

```bash
python3 -m venv .venv
```

This creates a folder called `.venv` in the current directory.

### 3.2. Activate the venv (macOS / Linux style)

```bash
source .venv/bin/activate
```

If activation worked, your prompt should look similar to:

```text
(.venv) your-mac:~/path/to/mqtt-macos-tutorial user$
```

Whenever you see `(.venv)` at the beginning of your prompt, it means the virtual environment is active.

To **deactivate** it later, you can run:

```bash
deactivate
```

---

## 4. Install Python dependencies (Paho MQTT)

With the virtual environment **activated**, run:

```bash
pip install -r requirements.txt
```

This will install the [Paho MQTT](https://pypi.org/project/paho-mqtt/) client library required by the example scripts.

You can verify:

```bash
pip list
```

You should see `paho-mqtt` in the list.

---

## 5. Install Eclipse Mosquitto MQTT broker (macOS via Homebrew)

Install Mosquitto with Homebrew:

```bash
brew install mosquitto
```

After installation, check it’s available:

```bash
mosquitto -h
```

You should see the help text.

---

## 6. Use the included Mosquitto config

From version 2.0 onward, Mosquitto is more restrictive by default.  
To keep things simple for a local tutorial, this repo includes a minimal config file that:

- listens on port **1883** (the default MQTT port), and  
- allows **anonymous** local connections.

> **Important:** This is fine for a local tutorial, but **not recommended for production**.

The config file is in:

```text
mosquitto/mosquitto.conf
```

The provided one is a template. You can rename it and create a new one with just 2 lines of code as it is the following (example [mosquitto.conf](./mosquitto.conf)):
It contains:

```conf
listener 1883
allow_anonymous true
```

---

## 7. Start the Mosquitto broker

From the **project root** (same folder as `README.md`), run:

```bash
mosquitto -c "$(pwd)/mosquitto/mosquitto.conf" -v
```

Explanation:

- `-c` tells Mosquitto which configuration file to use.
- `-v` enables **verbose** logging so you can see connections and messages.
- `$(pwd)` expands to the full path of the current directory.

You should now see Mosquitto output like:

```text
1710000000: mosquitto version x.x.x starting
1710000000: Opening ipv4 listen socket on port 1883.
```

Leave this Terminal window **open**. The broker must keep running.

---

## 8. Basic example: subscriber (listener)

Open a **new Terminal** window or tab.  
Go to the project folder and activate the venv:

```bash
cd /path/to/mqtt-macos-tutorial
source .venv/bin/activate
```

Then run the subscriber:

```bash
python src/subscriber.py
```

You should see something like:

```text
[SUBSCRIBER] Connecting to broker at localhost:1883...
[SUBSCRIBER] Connected with result code 0
[SUBSCRIBER] Subscribed to topic: test/topic
Waiting for messages... Press Ctrl+C to exit.
```

Leave this running.

---

## 9. Basic example: publisher (sender)

Open **another Terminal** window or tab, go to the project folder, and activate the venv again:

```bash
cd /path/to/mqtt-macos-tutorial
source .venv/bin/activate
python src/publisher.py
```

You should see:

```text
[PUBLISHER] Connecting to broker at localhost:1883...
[PUBLISHER] Connected with result code 0
[PUBLISHER] Publishing message 1: Hello MQTT! (#1)
[PUBLISHER] Publishing message 2: Hello MQTT! (#2)
...
```

On the **subscriber** window, you should see:

```text
[SUBSCRIBER] Message received!
    Topic: test/topic
    Payload: Hello MQTT! (#1)
[SUBSCRIBER] Message received!
    Topic: test/topic
    Payload: Hello MQTT! (#2)
...
```

---

## 10. QoS and retained examples (separate scripts)

The basic example uses **QoS 0** and **non-retained** messages.  
To explore more MQTT features, this repo includes:

- `src/qos_retained_subscriber.py`
- `src/qos_retained_publisher.py`

These demonstrate:

- QoS 0, 1, and 2
- Retained messages

### 10.1. Run the QoS + retained subscriber

In a Terminal with the virtual environment active:

```bash
python src/qos_retained_subscriber.py
```

Expected output (simplified):

```text
[QOS-SUB] Connecting to broker at localhost:1883...
[QOS-SUB] Connected with result code 0
[QOS-SUB] Subscribed to test/+/qos with QoS 2
[QOS-SUB] Subscribed to test/retained with QoS 2
Waiting for messages... Press Ctrl+C to exit.
```

This subscriber listens on:

- `test/+/qos`  → will match `test/0/qos`, `test/1/qos`, `test/2/qos`
- `test/retained`

For every received message it prints:

- the topic,
- the payload,
- the **QoS** used to deliver the message,
- whether the message is **retained** or not.

### 10.2. Run the QoS + retained publisher

In another Terminal (with venv active):

```bash
python src/qos_retained_publisher.py
```

Expected behaviour:

- Publishes **three messages** on:
  - `test/0/qos` with QoS 0
  - `test/1/qos` with QoS 1
  - `test/2/qos` with QoS 2
- Publishes **one retained message** on:
  - `test/retained` with QoS 1 and `retain=True`

On the subscriber window you will see each message come in with its QoS, and you can check the `retained` flag in the output.

---

## 11. Testing the retained message

To see the effect of a retained message clearly:

1. Make sure the broker is running.
2. Run the **retained publisher** once:

   ```bash
   python src/qos_retained_publisher.py
   ```

   This sets a retained message on `test/retained`.

3. Stop any existing QoS subscriber (Ctrl + C).
4. Now **start the QoS subscriber** again:

   ```bash
   python src/qos_retained_subscriber.py
   ```

Notice that **immediately after subscribing**, the subscriber will receive the **retained** message, even though the publisher is no longer running.

The output will say something like:

```text
[QOS-SUB] Message received
    Topic: test/retained
    Payload: This is a retained status message.
    QoS: 1
    Retained: True
```

That is the key idea of retained messages:  
the broker keeps the **last retained message** on a topic and delivers it instantly to **new subscribers**.

---

## 12. Stopping everything

- To stop the **publisher scripts**: go to their Terminal window and press `Ctrl + C`.
- To stop the **subscriber scripts**: go to their Terminal window and press `Ctrl + C`.
- To stop the **broker** (Mosquitto): go to the broker Terminal window and press `Ctrl + C`.

---

## 13. Troubleshooting

**1. `ModuleNotFoundError: No module named 'paho'`**

- Make sure your **virtual environment is activated** (`(.venv)` visible in the prompt).
- Run:

  ```bash
  pip install -r requirements.txt
  ```

**2. `Connection Refused` error in Python clients**

- Check that Mosquitto is running and listening:
  - Is the broker Terminal open with no obvious errors?
  - Did you pass the correct path to `mosquitto.conf`?

- Confirm the broker is listening on port 1883 with this config:
  - `listener 1883`
  - `allow_anonymous true`

**3. Port 1883 already in use**

- Maybe you have another MQTT broker or service using this port.
- You can change the port number in `mosquitto/mosquitto.conf`, e.g.:

  ```conf
  listener 1884
  allow_anonymous true
  ```

- Then update the Python scripts to use port `1884`.

---

## 14. How this example works (in simple terms)

- The **Mosquitto broker** is the **middleman** that receives and routes messages.
- The **subscriber** tells the broker:  
  “I am interested in messages on topic `test/topic` (or similar).”
- The **publisher** sends messages to the broker on that topic.
- The broker forwards those messages to any client subscribed to that topic.

QoS levels change **how hard MQTT tries to deliver** a message:

- **QoS 0** – *at most once*: fire-and-forget, no acknowledgement.
- **QoS 1** – *at least once*: the broker will retry until it knows the client got it (can result in duplicates).
- **QoS 2** – *exactly once*: more expensive handshake to ensure no duplicates.

Retained messages let the broker remember the **last known value** for a topic and give it immediately to new subscribers.

This pattern is called **publish/subscribe**, and it’s very popular in IoT and distributed systems.
