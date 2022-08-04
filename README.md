# Integrate ADT and 3d Scenes with IoT Central


https://user-images.githubusercontent.com/17155996/182733010-627941d8-e7d7-4ea8-a459-c5ecae14a379.mp4


IoT Central as we know is a No Code aPaaS which allows users to quickly connect IoT Devices to cloud and start realizing use cases like Condition Monitoring, Remote Operations etc. Each device that gets connected already has Twin model defined, which makes integration with ADT as the logical next step. Here I demonstrate how you can not only integrate with ADT and start sending telemetry and properties to ADT but also how you can embed the ADT 3d Scenes back in IoT Central to close the loop and provide a richer experience to your users. 
This enables the users to walk through a plant or a mine, observe state of different assets, quickly identify assets which need attention and then drill down in a particular asset to view it in detail.

![image](https://user-images.githubusercontent.com/17155996/182738201-f8d72171-6641-4d15-acf5-e63ff811ffca.png)

### Step 1 - Upload the respective DTDL models into the ADT
For the device groups/models that you would like to integrate with ADT, upload their DTDL Models into the ADT. Once uploaded create a default/empty twin of the asset with a **dtid** same as the device id from IoT Central.

<img width="960" alt="image" src="https://user-images.githubusercontent.com/17155996/182540804-7f67393d-8d4b-403a-975a-5bbdb1ca8924.png">

1. [Data Export](https://docs.microsoft.com/en-us/azure/iot-central/core/howto-export-to-event-hubs?tabs=connection-string%2Cjavascript)
1. [Event Hub Trigger](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-event-hubs-trigger?tabs=in-process%2Cfunctionsv2%2Cextensionv5&pivots=programming-language-csharp)
1. [Patch Twins Function]()
1. [Embedder App](https://github.com/malichishti/adt-3d-embedder-app)
1. [IoT Central Dashboard](https://docs.microsoft.com/en-us/azure/iot-central/core/howto-manage-dashboards#create-a-dashboard)

#### Links to help:
* [Manage models](https://docs.microsoft.com/en-us/azure/digital-twins/how-to-manage-model)
* [Create twins](https://docs.microsoft.com/en-us/azure/digital-twins/quickstart-azure-digital-twins-explorer#add-another-twin)
* Models I used can be found in /DTDLs directory

### Step 2 - Create a Data export to Event Hub
In IoT Central create a Data Export for all your devices to an Event Hub destination as per the instructions [here](https://docs.microsoft.com/en-us/azure/iot-central/core/howto-export-to-event-hubs?tabs=connection-string%2Cjavascript). 
Configure the Data Export to export all of the Telemetry to the Even Hub.
<img width="959" alt="image" src="https://user-images.githubusercontent.com/17155996/182542107-49a7a2d9-9cbf-4f47-a142-c40d38e7921e.png">

### Step 3 - Azure function to pass data from Event Hub to ADT
Create an Azure Function with a trigger on Event Hub. This would be responsible to receive the telemetry from Event Hub and Patch it in ADT for the respective twin. You can find the source code for this Function at **/Azure Function - Event Hub to ADT/TelemetryTrigger.cs** You can see it finds the deviceId from the message which is used to link to the corresponding twin via dtid. Next it sends a Patch to ADT for all telemetry received.

The Azure Function requires following Applications settings defined:

1. EVENTHUB - this is a connection string to the Event Hub
1. ADT_SERVICE_URL - This is the ADT Instance URL e.g. https://my-twins3.api.aue.digitaltwins.azure.net

**Please Note:** The Azure Function must have **Azure Digital Twins Data Owner** access to the ADT. You can achieve this by giving a System assigned identity to the Azure function and then adding a Role access for it on ADT.

### Step 4 - Setup 3d Models
Now that we have Twins integrated with Device telemetry from IoT Central, next step is to create 3d Scenes and map telemetry and behaviors in the 3d models. You can follow the instructions [here](https://docs.microsoft.com/en-us/azure/digital-twins/how-to-use-3d-scenes-studio). Sample models can be found in this [repository](https://github.com/malichishti/ADT-Models).

### Step 5 - Create Embedder App
In this step you create an Azure App Service using https://github.com/malichishti/adt-3d-embedder-app This would enable you to embed any specific 3d Scene using an iFrame.

### Step 6 - Embed a 3d Scene inside IoT Central Dashboard
You are now ready to embed the 3d Scene in IoT Central:
1. [Create a new dashboard](https://docs.microsoft.com/en-us/azure/iot-central/core/howto-manage-dashboards#create-a-dashboard)
1. In he edit view, add a **External content** Tile
1. In the properties of the tile add a URL to your embedder app including a querystring parameter called **sceneId** e.g. `https://3d-embedder.azurewebsites.net/?sceneId=3b614761fd1187ace3bac52ebb241c54`
<img width="937" alt="image" src="https://user-images.githubusercontent.com/17155996/182735779-926a6fed-eabf-47ef-bab1-0b8db1a6ef2f.png">

Now you will be able to see the 3d scene in your dashboard.

You can also setup navigation/drilldown between 3d scenes by using the Link Widget in 3d Scene Build view and passing a link to the embedder app.

<img width="960" alt="image" src="https://user-images.githubusercontent.com/17155996/182736256-f4c9ec2a-faaf-4b4a-92f9-0a4ee37becf9.png">

