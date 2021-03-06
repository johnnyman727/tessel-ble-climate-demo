# Tessel Climate/BLE Demo

This application demonstrates how to stream climate data from your Tessel to your BLE-equipped iPhone or iPad

<div id="top_of_page"></div>

## Table of Contents
<ul>
	<li><a href="#user-content-getting_started">Getting Started</a></li>
	<li><a href="#user-content-feature_list">Feature List</a></li>
	<li><a href="#user-content-screenshots">Screenshots</a></li>
	<li><a href="#user-content-requirements">Requirements and Assumptions</a></li>
	<ul>
		<li>Software Requirements</li>
		<li>Hardware Requirements</li>
		<li>Assumptions</li>
	</ul>
	<li><a href="#user-content-how_it_works">How it Works</a></li>
	<li><a href="#user-content-limitations">Limitations</a></li>
</ul>

</div>

## Getting Started
<div id="getting_started"><a href="#user-content-top_of_page">(Back to top)</div>

1. `git clone https://github.com/rbobbins/tessel-ble-climate-demo.git`
1. `cd path/to/tessel-ble-climate-demo`

1. On your Tessel:

	* Plug your BLE module into port A
	* Plug your climate module into port B
	* `tessel run climate.js`
	* Once you see the following 2 lines (in any order), your Tessel is ready to connect and transmit climate data.
	
	```
		Connected to climate-si7020
		Connected to ble113a.
	```

1. On your Mac:

	* Open the project in XCode. It resides in in `PROJECT_DIR/tessel-ble-climate/tessel-ble-climate.xcodeproj`
	* Plug in your iPhone or iPad. App will not work on the iOS simulator. (See "Requirements and Assumptions > Assumptions" for explanation.)
	* Build for your iPhone or iPad
	* Make sure your device has bluetooth turned on. If it's off, your device will present a helpful alert the first time you open the app.
	* On the app, tap the "Scan" button to search for your Tessel

## Feature List
<div id="feature_list"><a href="#user-content-top_of_page">(Back to top)</div>

* When iPhone/iPad is connected to Tessel, it shows a live display of climate data
* User can scan for (and then connect to) Tessel
* User can stop scan, or kill connection to Tessel
* If the app doesn't find the Tessel within 5 seconds, it asks if the user wants to continue scanning for it.
* When connection to the Tessel gets killed due to an error (ie. connection time out), app automatically tries to reconnect
* User can see log of all events that happen with the Tessel.
	* Log updates automatically every time the connection changes
	* Log does not update automatically when new climate data is received; user can manually refresh log by clicking the "Refresh button"
	* User can clear log at any time


## Screenshots
<div id="screenshots"><a href="#user-content-top_of_page">(Back to top)</div>

_Click to enlarge_


<a href="https://raw.githubusercontent.com/rbobbins/tessel-ble-climate-demo/master/screenshots/success_case.png">
	<img src="https://raw.githubusercontent.com/rbobbins/tessel-ble-climate-demo/master/screenshots/success_case.png" width="40%"/>
</a>
<a href="https://raw.githubusercontent.com/rbobbins/tessel-ble-climate-demo/master/screenshots/scanning_case.png">
	<img src="https://raw.githubusercontent.com/rbobbins/tessel-ble-climate-demo/master/screenshots/scanning_case.png" width="40%"/>
</a>

## Requirements and Assumptions
<div id="requirements"><a href="#user-content-top_of_page">(Back to top)</div>

### Hardware Requirements
* Tessel, with the following modules:
	* ble-ble113a module
	* climate-si7020 module (you could probably make it work with the older climate-si7005 module by swapping out the node package that's included in this repo)
* iOS device with BLE compatability. Must be running iOS8
* Mac

### Software Requirements
* XCode 6.1
* Tessel CLI

### Assumptions
* You're a registered iOS developer. Unfortunately, Apple requires that you be a registered iOS dev in order to build any app on a physical device. BLE only works on physical devices, not on the iOS simulator.


## How it works
<div id="how_it_works"><a href="#user-content-top_of_page">(Back to top)</div>

To start, the Tessel BLE module advertises that it's available for connection over BLE.

When the user taps the "Scan" button on the iOS app, her iPhone/iPad searches for all available BLE peripheral devices. When it finds one who's name is "Tessel BLE113A Module", it stops scanning and tries to connect to that peripheral. Upon successful connection, the Tessel starts broadcasting climate data. At 1 second intervals, the Tessel writes temperature data to its first characteristic and humidity data to its second characteristic.

After an iDevice/Tessel connection is made, the iDevice searches for the "Data Tranceiving" service on the Tessel:

    CBUUID *dataServiceUUID = [CBUUID UUIDWithString:kTesselDataTransceivingServiceUUID];
    [peripheral discoverServices:@[dataServiceUUID]];

After finding the Data Tranceiving service, the iDevice searches for the characteristics that broadcast climate data:

    CBUUID *tempCharacteristicUUID = [CBUUID UUIDWithString:kTesselTemperatureCharacteristicUUID];
    CBUUID *humidityCharacteristicUUID = [CBUUID UUIDWithString:kTesselHumidityCharacteristicUUID];
    [peripheral discoverCharacteristics:@[tempCharacteristicUUID, humidityCharacteristicUUID]
                             forService:service];

The UUID's for services and characteristics are baked into the firmware of the ble-ble113a. Thus, it's safe to use the UUIDs as hardcoded values in the iOS app. If you're curious, the complete list of services and their properties is available in the Tessel's [GATT Profile](https://github.com/tessel/ble-ble113a/blob/master/lib/profile.json)

Once the iOS app knows about the Tessel's characteristics, it subscribes to notifications.

	[peripheral setNotifyValue:YES forCharacteristic:characteristic]

Every time the BLE module broadcasts climate data (at the 1 second interval we specified), the app will be notified:

	- (void)peripheral:(CBPeripheral *)peripheral 
	didUpdateValueForCharacteristic:(CBCharacteristic *)characteristic 
	error:(NSError *)error 

Each time this method is called, we parse the raw byte data into an `NSNumber` and notify the ViewController. The ViewController updates the large temperature and humidity displays, making sure the numbers are formatted in human-readable values.

## Limitations
<div id="limitations"><a href="#user-content-top_of_page">(Back to top)</div>

* App does not record climate changes that happen when it's running in the background
* App has a mostly universal layout, but it looks cramped if you're running an iPhone 5s or earlier in landscape mode. 