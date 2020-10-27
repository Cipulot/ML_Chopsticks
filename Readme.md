# 🥢ML Chopsticks🥢
Solution to gather user behavior information when using chopsticks and compile a set of usage statistics based on a ML generated model.

## Inspiration
Following a gimmick that popped up at the end of last [CWF](https://scottkwang.github.io/CodeWithFriends-Spring2020/) edition the project involves chopsticks. The pitch was to create a "5G enabled" chopstick but I then realized that it was a bit impractical (at least for now). Putting the 5G part of the idea aside, this project will enable the user, thanks to ML based models, to gather information about his/her usage behaviour and all sorts of statistics of use.

## General features
The projects aims to develop a system that can be used to collect and process information about user's usage of chopsticks.

To do so the system is based on 2 main components:

* Data collection and model building
* Usage detector and statistics generator

The code flashed on the Tactigon One Board is shared for both, since its main purpose it to collect and expose sensor data (accelerometer and gyroscope) on the BLE Characteristic. The RPi code is Python based and, depending on the component that is required by the user, serves as ML model generator or as gesture and statistics recognition.
This is achieved by implementing an ```if __name__ == "__main__":``` construct in the code, allowing the code to be called "by itself" or imported in other scripts as a module, accessing all its sub functions.

### Data collection and model building
In order to evaluate the movements that the user achieve with chopsticks it's necessary to create a base model around some kind of gesture. In this case a feasible gesture set could be something like: ```GestList = ['picked up', 'put down', 'taking a bite', etc...] ```. As for more "mainstream" ML models it's necessary that you chose those gestures ahead of the model creation and, in order to more easily train the model, record a relatively large and diversified variation of the same gesture, i.e., picking up the chopstick from different angles, take pauses of various duration between one bite and the other.

The raw data is then filtered for unwanted and/or corrupted data that might come through and it's feed to the ML model generator. After the process is complete a final model file is generated and will be used for future use in detection and statistics mode.

Here's a visual representation of the accelerometer data received by the RPi:

![](https://github.com/Cipulot/ML-Chopsticks/blob/main/ML-Chopsticks/media/Sensor%20Graph.gif?raw=true)

## Tech used
For the Tactigon One Board I've used the Arduino IDE to develop a basic BLE peripheral code. The main function gathers IMU data with a 50Hz timing and then puts it into a buffer that will be used to update the BLE characteristic. A simple "if-else" statement is used to check the Bluetooth connection so that if it drops or didn't happened at all the data won't be "pushed out", saving battery.

The RPi, once it's booted up, execute a python code the will search for the Tactigon and connect to it. It then subscribes to the BLE characteristic in notification and waits for new data pushed by the Tactigon One Board. It then feeds, in real-time, every data collected to a custom ML algorithm (scikit-learn library based) that will the following operations:

* Filtering: the raw data read on the characteristic is filtered using some data filtering code that will get rid of possible unwanted/corrupted data that is gathered by the Tactigon One Borad.
* Segmentation: the data comes through at a timing of 50Hz and that is a bit problematic... It has advantages, like more fine detection of movements but introduces a layer of complication for the recognition that later will be executed. This step ensure that a valid interval of data is feed into the recognition algorithm.
* Recognition and flagging: the data segment provided will pass through a custom ML code that will calculate the confidence value for a specific gesture from the pre-trained model that is specified into the code. Once a gesture is recognized it will be flagged by the algorithm, which will later record it in a log file for later use.

## Built with
* [scikit-learn](https://scikit-learn.org/stable/) - installed via pip
* [Python 3.7.7](https://www.python.org/downloads/release/python-377/) - Win64
* Arduino IDE
### Note
The ML algorithm is still a work in progress and, for now, I will not be able to post it 😢 .

## Specifications
The project was developed and tested on the following hardware:
* [Raspberry Pi 3 Model B+](https://www.raspberrypi.org/products/raspberry-pi-3-model-b-plus/?resellerType=home)
* [Tactigon One Board](https://www.nextind.eu/product/the-tactigon-one/)
## Known issues

## To do
The Raspberry Pi 3 is a bit too large to make a "real" portable version of the system. Something with the footprint of the RPi Zero or a cut-down version like the [RPi 3 A+](https://www.raspberrypi.org/products/raspberry-pi-3-model-a-plus/?resellerType=home) would do the trick.

## Reference
The documentation about the Tactigon board can be found [here](https://github.com/TactigonTeam/Docs).

For Tactigon libraries exposed in the Arduino IDE you can look [here](https://www.thetactigon.com/arduino/doxygen/index.html).
