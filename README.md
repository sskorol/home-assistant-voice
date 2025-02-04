# Home Assistant Voice Preview Setup

This repo will guide you through a local Docker-based HA setup with [Voice Assistant](https://www.home-assistant.io/voice-pe/) support.

![image](https://github.com/user-attachments/assets/72785be6-45fc-48cd-b61f-aa8bf3b719e9)

https://github.com/user-attachments/assets/ff3d1837-b677-42a4-9926-9ef42d434af3

## Initial setup (Ubuntu)

```shell
git pull https://github.com/sskorol/home-assistant-voice.git && cd home-assistant-voice
docker compose pull
```

- You can comment/remove the `mosqutto` service if you don't use MQTT.
- If you plan to use **Ollama** locally, you may want to explicitly [install](https://ollama.com/download) it and map the `.ollama` folder to the container.
- Check supported **Whisper**/**Piper** environment vars to pull the correct models based on your language preference (EN is the default).
- **Piper** voices can be tested [here](https://rhasspy.github.io/piper-samples/).
- GPU is enabled by default for **Whisper**/**Piper**/**Ollama** images. You can remove `deploy` blocks if you plan to run on the CPU. Note that this guide doesn't cover the CUDA setup. Follow the official [NVIDIA tutorials](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) for that.

```shell
docker compose up -d
```

## Voice Assistant PE

Follow [this guide](https://voice-pe.home-assistant.io/getting-started/) to connect your hardware. Note that if you are as lucky as me and don't have a stable BLE device on your PC, you'll likely get infinite Bluetooth errors on your HA instance, preventing you from establishing the initial connection with Voice Assistant. I recommend you do not waste time, and use your cell phone to connect to the HA instance on your PC and set up Voice Assistant via mobile BLE.

## Whisper / Piper / Ollama

- Whisper and Piper can be added to HA via Wyoming Protocol. As HA shares the network with your host OS, ASR / TTS services will be accessible via `localhost` on 10300 and 10200 ports.
![Screenshot](https://github.com/user-attachments/assets/213a63d8-a13d-407b-9c5a-3217924f18a4)

- Ollama will also be accessible on `http://localhost:11434`. If you've already downloaded models locally, you'll immediately see them as we mapped the model folder with the container.
![Screenshot](https://github.com/user-attachments/assets/598f5894-11df-47d9-b244-2a74dd0993b6)

I recommend starting with Llama 3.2. Don't forget to tune the prompt and enable assistance:

![Screenshot](https://github.com/user-attachments/assets/ca4b64e4-b1d2-4249-a583-53360f695d65)

Just so you know, if you want to add a new model, you need to add it as a separate service.
- Now you can see newly added services in the Voice Assistant config:

![Screenshot](https://github.com/user-attachments/assets/53cf45b5-c851-4dd1-83fd-a09ade94f1c0)

- Expose your smart home devices to voice assistant (alias will serve as a short command to access your device):

![Screenshot](https://github.com/user-attachments/assets/7a63477b-9f1a-4d6d-838a-a1bfbc65211b)

- Ensure your ESPHome is linked with Voice Assist PE and connected to the newly configured virtual Voice Assistant in HA:

![Screenshot](https://github.com/user-attachments/assets/8d7ba8ae-6cdb-4ff5-841f-011d4eae0ce1)

- Test ASR / TTS and chatting capabilities via web UI:

![Screenshot](https://github.com/user-attachments/assets/eda3fac9-a4b4-4322-b580-01b6a087a60c)

Now you are ready to interact with your devices via Voice PE hardware and have casual conversations with Llama 3.2. Well, not really...

### Known Issues

You'll likely almost immediately face TTS failures on long responses from LLMs. There are several similar bugs raised across HA repos about this.

One workaround you can apply is [patching](https://github.com/esphome/home-assistant-voice-pe/compare/dev...sskorol:home-assistant-voice-pe:fix/tts_timeout) the `AudioReader` timeout in the VA PE firmware code. Note that to build the firmware, you must provide your Wi-Fi credentials in `secrets.yaml`. You should also update the repo reference to ensure your changes are applied. Otherwise, the code will be pulled from the original dev branch.

Fork `https://github.com/esphome/home-assistant-voice-pe.git`.
```shell
git clone https://github.com/YOUR_USERNAME/home-assistant-voice-pe.git && cd home-assistant-voice-pe
```
Update `secrets.yaml`:
```shell
nano secrets.yaml
# Paste your WiFi creds:
wifi_ssid: "..."
wifi_password: "..."
```

- Apply the above patch.
- Push changes to your fork.
- Update `external_components` URL to point to your fork.
- Build and flash the updated firmware:
```shell
docker run --rm -v "$(pwd)":/config -it ghcr.io/esphome/esphome run --device IP_OF_YOUR_VA_PE home-assistant-voice.yaml
```

Also note that changes in containers' environment variables (e.g., new models or voices) require a complete stop. New values won't be automatically picked up on restart.
