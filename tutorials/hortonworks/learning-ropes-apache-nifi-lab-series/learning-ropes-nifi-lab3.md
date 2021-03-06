---
layout: tutorial
title: Lab 3 Ingest NextBus Live Stream Routes to the DataFlow
tutorial-id: 640
tutorial-series: Basic Development
tutorial-version: hdp-2.4.0
intro-page: false
components: [ nifi ]
---

# Lab 3: Ingest NextBus Live Stream Routes to the DataFlow

## Introduction
In this tutorial, You will replace the section of our dataflow that generates the simulation of vehicle location XML data with a new section that ingests a live stream of data from NextBus San Francisco Muni Agency on route OceanView into our NiFi DataFlow.

In this lab, you will build the Ingest NextBus SF Muni Live Stream section of the dataflow:

![](/assets/learning-ropes-nifi-lab-series/lab3-ingest-nextbus-live-stream-nifi-lab-series/complete_dataflow_lab3_live_stream_ingestion.png)

Feel free to download the [Lab3-NiFi-Learn-Ropes.xml](https://raw.githubusercontent.com/hortonworks/tutorials/hdp/assets/learning-ropes-nifi-lab-series/lab3-template/Lab3-NiFi-Learn-Ropes.xml) template file or if you prefer to build the dataflow from scratch, continue on to the lab.

## Pre-Requisites
- Completed Lab 0: Download, Install and Start NiFi
- Completed Lab 1: Build A Simple NiFi DataFlow
- Completed Lab 2: Enhance the DataFlow with Geo Location Enrichment

## Outline
- NextBus Live Feed
- Step 1: Attach NextBus Live Stream to the DataFlow
- Step 2: Run the NiFi DataFlow
- Summary
- Further Reading

## NextBus Live Feed

NextBus Live Feed provides the public with live information regarding passenger information, such as vehicle location information, prediction times on transit vehicles, routes of vehicles and different agencies (San Francisco Muni, Unitrans City of Davis, etc). We will learn to use NextBus's API to access the XML Live Feed Data and create an URL. In this URL we will specify parameters in a query string. The parameters for the lab will include the vehicle location, agency, route and time.

After viewing the Live Feed Documentation, we created the following URL for the GetHTTP processor:

~~~
http://webservices.nextbus.com/service/publicXMLFeed?command=vehicleLocations&a=sf-muni&r=M&t=0
~~~

Let’s break apart the parameters, so we can better understand how to create custom URLs. There are 4 parameters:

- commands: command = vehicleLocations
- agency: a=sf-muni
- route: r=M
- time: t=0

Refer to [NextBus’s Live Feed Documentation](https://www.nextbus.com/xmlFeedDocs/NextBusXMLFeed.pdf) to learn more about each parameter.

### Step 1: Attach NextBus Live Stream to the DataFlow

### GetHTTP

1\. Delete GetFile, UnpackContent and ControlRate processors. We will replace them with the GetHTTP processor.

2\. Add the **GetHTTP** processor and drag it to the place where the previous three processors were located. Connect GetHTTP to EvaluateXPath processor located above SplitXML. When the Create Connection window appears, select **success** checkbox. Click **Apply**.

3\. Open GetHTTP Config Property Tab window. We will need to copy and paste Nextbus XML Live Feed URL into the property value. Add the property listed in **Table 1**.

**Table 1:** Update GetHTTP Properties Tab

| Property  | Value  |
|:---|---:|
| `URL`  | `http://webservices.nextbus.com/service/publicXMLFeed?command=vehicleLocations&a=sf-muni&r=M&t=0` |
| `Filename`  | `vehicleLoc_SF_OceanView.xml` |

![getHTTP_liveStream_config_property_tab_window](/assets/learning-ropes-nifi-lab-series/lab3-ingest-nextbus-live-stream-nifi-lab-series/getHTTP_liveStream_config_property_tab_window.png)

4\. Now that each property is updated. Navigate to the **Scheduling tab** and change the **Run Schedule** from 0 sec to `1 sec`, so that the processor executes a task every 1 second. Therefore, overuse of system resources is prevented.

5\. Open the processor config **Settings** tab, change the processor's Name from GetHTTP to `IngestVehicleLoc_SF_OceanView`. Click **Apply** button.

### Modify PutFile in Geo Enrich Section

1\. Open PutFile Configure **Properties Tab**. Change the Directory property value from the previous value to the value shown in **Table 2**:

**Table 2:** Update PutFile Properties Tab

| Property  | Value  |
|:---|---:|
| `Directory`  | `/tmp/nifi/output/nearby_neighborhoods_liveStream`

**Directory** is changed to a new location for the real-time data coming in from NextBus live stream.

![modify_putFile_in_geo_enrich_section](/assets/learning-ropes-nifi-lab-series/lab3-ingest-nextbus-live-stream-nifi-lab-series/modify_putFile_in_geo_enrich_section.png)

2\. Click **Apply**.


### Step 2: Run the NiFi DataFlow

Now that we added NextBus San Francisco Muni Live Stream Ingestion to our dataflow , let's run the dataflow and verify if we receive the expected results in our output directory.

1\. Go to the actions toolbar and click the start button ![start_button_nifi_iot](/assets/learning-ropes-nifi-lab-series/lab1-build-nifi-dataflow/start_button_nifi_iot.png). Your screen should look like the following:

![complete_dataflow_lab3_live_stream_ingestion](/assets/learning-ropes-nifi-lab-series/lab3-ingest-nextbus-live-stream-nifi-lab-series/complete_dataflow_lab3_live_stream_ingestion.png)

2\. Let's verify the data in output directory is correct. Navigate to the following directories and open a random one to check the data.

~~~
cd /tmp/nifi/output/nearby_neighborhoods_liveStream
ls
vi 58849126478211
~~~

Did you receive neighborhoods similar to the image below?

![nextbus_liveStream_output_lab3](/assets/learning-ropes-nifi-lab-series/lab3-ingest-nextbus-live-stream-nifi-lab-series/nextbus_liveStream_output_lab3.png)

## Summary

Congratulations! You learned how to use NextBus's API to connect to their XML Live Feed for vehicle location data. You also learned how to use the GetHTTP processor to ingest a live stream from NextBus San Francisco Muni into NiFi!

## Further Reading

- [NextBus XML Live Feed](https://www.nextbus.com/xmlFeedDocs/NextBusXMLFeed.pdf)
- [Hortonworks NiFi User Guie](http://docs.hortonworks.com/HDPDocuments/HDF1/HDF-1.2.0.1/bk_UserGuide/content/index.html)
- [Hortonworks NiFi Developer Guide](http://docs.hortonworks.com/HDPDocuments/HDF1/HDF-1.2.0.1/bk_DeveloperGuide/content/index.html)
