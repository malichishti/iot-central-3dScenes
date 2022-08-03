# Integrate ADT and 3d Scenes with IoT Central
IoT Central as we know is a No Code aPaaS which allows users to quickly conenct IoT Devices to cloud and start realizing use cases like Condition Monitoring, Remote Operations etc. Each device that gets connected already has Twin model defined, which makes integration with ADT as the logical next step. Here I demonstrate how you can not only integrate with ADT and start sending telemetry and properties to ADT but also how you can embed the ADT 3d Scenes back in IoT Central to close the loop and provide a richer experience to your users. 
This enables the users to walk through a plant or a mine, observe state of different assets, quickly identify assets which need attention and then drill down in a particular asset to view it in detail.

diagra

### Step 1 - Upload the respective DTDL models into the ADT
For the device groups/models that you would like integrate with ADT, upload their DTDL Models into the ADT. Once uploaded create a default/empty twin of the asset with a **dtid** same as the device id from IoT Central.

<img width="960" alt="image" src="https://user-images.githubusercontent.com/17155996/182540804-7f67393d-8d4b-403a-975a-5bbdb1ca8924.png">

#### Links to help:
* [Manage models](https://docs.microsoft.com/en-us/azure/digital-twins/how-to-manage-model)
* [Create twins](https://docs.microsoft.com/en-us/azure/digital-twins/quickstart-azure-digital-twins-explorer#add-another-twin)
* Models I used can be found in /DTDLs directory

### Step 2 - Create a Data export to Event Hub
In IoT Central create a Data Export for all your devices to an Event Hub destination as per the instructions [here](https://docs.microsoft.com/en-us/azure/iot-central/core/howto-export-to-event-hubs?tabs=connection-string%2Cjavascript). 
Configure the Data Export to export all of the Telemetry to the Even Hub.
<img width="959" alt="image" src="https://user-images.githubusercontent.com/17155996/182542107-49a7a2d9-9cbf-4f47-a142-c40d38e7921e.png">

### Step 3 - Azure function to pass data from Event Hub to ADT
Create an Azure Function with a trigger on Event Hub. This would be responsible to receie the telemetry from Event Hub and Patch it in ADT for the respective twin. You can find the source code for this Function at **/Azure Fucntion - Event Hub to ADT/TelemetryTrigger.cs** You can see it finds the deviceId from the message which is used to link to the corrsponding twin via dtid. Next it sends a Patch to ADT for all telemetry received.

The Azure Function requires following Applications settings defined:

1. EVENTHUB - this is a conenction string to the Event Hub
1. ADT_SERVICE_URL - This is the ADT Instance URL e.g. https://my-twins3.api.aue.digitaltwins.azure.net

**Please Note:** The Azure Function must have **Azure Digital Twins Data Owner** access to the ADT. You can achieve this by giving a System assigned identity to the Azure function and then adding a Role access for it on ADT.

### Step 4 - Setup 3d Models
Now that we have Twins integrated with Device telemetry from IoT Central, next step is to create 3d Scenes and map telemetry and behaviours in the 3d models. 


