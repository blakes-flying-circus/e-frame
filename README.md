# e-paper dashboard

The idea is to have a static e-ink display that shows helpful dashboard about the house and vehicles that my fiance and I can reference throughout the day. 

Was inspired by [this project](https://github.com/kimmobrunfeldt/eink-weather-display). 

Here are some other examples:    
https://github.com/jtiscione/e-paper-frame
https://github.com/ondiiik/meteoink

# Software
## ESPHome
[ESPHome](https://esphome.io/) is a system to control your microcontrollers by simple yet powerful configuration files and control them remotely through Home Automation systems.

I went with ESPHome for this project as it has a extensive set of supported features and sensors. It also allows me to focus on building instead of getting caught up in C 😅. 

I did take a look at [espressif idf](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/index.html#) and considering my lack of C expertise I got scared. 

The beauty of ESPHome is in a single yaml config for the micro-controller I can define a [waveshare display](https://esphome.io/components/display/waveshare_epaper.html), a [native api component](https://esphome.io/components/api) to talk directly with home assistant, and [wifi component](https://esphome.io/components/wifi) for ensuring the micro-controller is always connected.

I won't be going over setup of ESPHome as there are multiple ways to set it up and you should choose the way that works best for you. See their get started documentation. For clarity I have mine [setup using docker](https://esphome.io/guides/getting_started_command_line).

For micro-controller configuration see e-paper.yaml. 

## Home assistant
[Home Assistant](https://www.home-assistant.io/) is free and open-source software used for home automation. It serves as an integration platform and smart home hub, allowing users to control smart home devices.

I've used home assistant off and on for a few years now. I've had my ups and downs managing it and keeping it running, but my latest dockerized config seems to be holding up well. 

[Home assistant also has community integrations](https://www.hacs.xyz/) that are available for install, and just so happened to have one for [Toyota](https://github.com/widewing/ha-toyota-na) and [Niu](https://github.com/marcelwestrahome/home-assistant-niu-component) our two vehicles that I was keen to have charge and fuel status of.

We recently purchased a Rav4 Prime which is a plugin electric vehicle (PHEV). I was hoping to display ev charge status and fuel status as an easy quick check when heading out of the house. We have the same use case for our Niu electric scooter. The niu HACS integration worked like a charm. Unfortunately the same cannot be said about the toyota integration. See below for details on setup for toyota ev data.

## ha-toyota-na setup
Seems like toyota DMCA'd the [original api](https://github.com/toyotha/toyota-na) which was created before support for EV info could be added. Thanks to the kind souls on [this issue](https://github.com/widewing/ha-toyota-na/issues/16). I was able to create a private version of the api and setup the HACS integration to display EV info in Home assistant ✨. Below are the steps to get that working. 

1. Pull [toyota-na 2.1.1](https://pypi.org/project/toyota-na/) from pypi. 
2. Follow the instructions posted by [PenitentTangent2401 here](https://github.com/widewing/ha-toyota-na/issues/16#issuecomment-2203381925) to add EV info to the toyota-na api. 
3. Push this version to a private repo. 
4. Install this dependency on your home assistant. Because my home assistant is run in docker I had to run the following commands post ha-toyota-na HACS install. Note I had to uninstall the public package as it was installed with the HACS integration. I need to look into adding this private library to my docker-compose.yaml 
```
docker exec -it homeassistant /bin/sh # get into homeassistant container

pip uninstall toyota-na #uninstalls the public package

python3 -m pip install git+https://your-user:your-token@github.com/your-user/your-repo.
git #needs a github token and installs augmented version with EV data
```
5. After these have been run you need to add the following to the const.py file in ha-toyota-na in HACS. This can be done more robustly by forking the repo and installing your own version or [waiting for support](https://github.com/widewing/ha-toyota-na/pull/76).
```
{
    "state_class": SensorStateClass.MEASUREMENT,
    "icon": "mdi:gauge",
    "feature": VehicleFeatures.ChargeDistance,
    "name": "EV Range",
    "unit": "MI_OR_KM",
},
{
    "state_class": SensorStateClass.MEASUREMENT,
    "icon": "mdi:gauge",
    "feature": VehicleFeatures.ChargeDistanceAC,
    "name": "EV Range AC",
    "unit": "MI_OR_KM",
},
{
    "state_class": SensorStateClass.MEASUREMENT,
    "icon": "mdi:gauge",
    "feature": VehicleFeatures.ChargeLevel,
    "name": "EV Battery Level",
    "unit": PERCENTAGE,
},
```
6. Now restart container/homeassistant and you should be able to pull the EV info into homeassistant. 

# Hardware
- [Waveshare 7.5inch E-Ink Display](https://www.amazon.com/dp/B075R4QY3L)
- [ESP32 ESP-WROOM-32 Development Board](https://www.amazon.com/gp/product/B09J95SMG7?th=1)
- power supply (batteries tbd)
  - Thinking of using [these chargers](https://www.amazon.com/HiLetgo-Lithium-Charging-Protection-Functions/dp/B07PKND8KG), with [these batteries](https://www.amazon.com/2000mAh-Rechargable-Protection-Insulated-Development/dp/B08T6QS58J?crid=2YJE37S17L3FT&dib=eyJ2IjoiMSJ9.rZvn_YuZa4q5jGue6fCkbXot6tkeWv5VWeuQRh8IOhO93PqubNLj1ANxBMpLURvVT6sb5HeGIkiK_eGCcSoVYzEw8F3kSLXlgsFKH9pg-auHuzAZiEmWRjSRLqSeFRTXzsw1gsBFarc0H_iEqCB1_h6aGg7FPqZFffZJCfE53wJ5wHhBOeykHq7Sd9mZLprIafcSy8NRorAC6kEK6RI4zVno02NN99PHWeC--lmgYH943XUGxciSiqCk6D_LKS2uRcIKtoL0ESiAlpv4GiT9fxh663qwriZXsyso1J7lXco.eJ0eByYKRxuvLDDCwd_k3tlNLwJggOryjyAl6VAv560&dib_tag=se&keywords=lipo+battery&qid=1726718108&sprefix=lipo+battery,aps,173&sr=8-10)

TODOS:
- need to figure out housing once POC is functional

