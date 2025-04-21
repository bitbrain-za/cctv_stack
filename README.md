# cctv_stack
Config for frigate on Jetson Nano with ALPR

# References

- [Frigate](https://blakeblackshear.github.io/frigate/)
- [Frigate Plate Recogniser](https://github.com/ljmerza/frigate_plate_recognizer.git)

# Frigate Configuration

Find your jetpack version: `sudo apt-cache show nvidia-jetpack | grep "Version"`
Modify the docker compose to use the correct image:
```yaml
    image: ghcr.io/blakeblackshear/frigate:stable-tensorrt-jp4 # for jetpack 4.x
    image: ghcr.io/blakeblackshear/frigate:stable-tensorrt-jp5 # for jetpack 5.x
```

## Select a model

The list can be found [here](https://docs.frigate.video/configuration/object_detectors#generate-models)
In the docker compose file, edit the `YOLO_MODELS` environment variable.
Edit the `model` field in the `frigate_config/config.yml` file to match the model you selected.

## Add your devices

Modify the contents of `frigate_config/config.yml` to match your setup.
Match the go2rtc stream names to the camera names.

# Automatic License Plate Recognition

## Create an account

Register for a free account at [Plate Recognizer](https://app.platerecognizer.com/)

## Build docker image for ALPR

Run this command to build the docker image for ALPR

```bash
docker build -t  frigate_alpr  https://github.com/ljmerza/frigate_plate_recognizer.git
```

## Configure ALPR

Modify the contents of `alpr_config/config.yml` to match your setup:
    - Change the `frigate_url` to point to your frigate instance
    - Change the `mqtt_server` to point to your mqtt broker
    - Change the `mqtt_port` to point to your mqtt broker
    - Change the `mqtt_username` to your mqtt username
    - Change the `mqtt_password` to your mqtt password
    - modify the array of cameras to include any that you want to add ALPR to
    - Put your plate recognizer token in the `token` field
    - Change the `regions` to the regions you want to use. You can find a list of regions [here](https://guides.platerecognizer.com/docs/tech-references/country-codes)
    - Add any plates you want to watch in the `watched_plates` array. 

```yaml

frigate:
  frigate_url: http://<replace with your firgate device IP>:5000
  mqtt_server: 123.123.123.123 # replace with your broker details
  mqtt_port: 1883
  mqtt_username: test # replace with your mqtt username
  mqtt_password: test # replace with your mqtt password
  main_topic: frigate
  return_topic: plate_recognizer
  frigate_plus: false
  camera:
    - driveway
  objects:
    - car
    - motorcycle
  min_score: .7
  watched_plates: #list of plates to watch.
    -  xxxxxx
  fuzzy_match: 0.8
plate_recognizer:
  token: xxxxxxxxxx
  regions:
    - za
logger_level: INFO

```

# Deploying the stack

Run

```bash
docker-compose up -d
```