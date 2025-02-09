---
title: Anomaly Detector JavaScript client library quickstart 
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 09/22/2020
ms.author: mbullwin
ms.custom: devx-track-js
---

Get started with the Anomaly Detector client library for JavaScript. Follow these steps to install the package start using the algorithms provided by the service. The Anomaly Detector service enables you to find abnormalities in your time series data by automatically using the best-fitting models on it, regardless of industry, scenario, or data volume.

Use the Anomaly Detector client library for JavaScript to:

* Detect anomalies throughout your time series data set, as a batch request
* Detect the anomaly status of the latest data point in your time series
* Detect trend change points in your data set.

[Library reference documentation](https://go.microsoft.com/fwlink/?linkid=2090788) | [Library source code](https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/anomalydetector) | [Package (npm)](https://www.npmjs.com/package/%40azure/ai-anomaly-detector) | [Find the code on GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/javascript/AnomalyDetector)

## Prerequisites

* Azure subscription - [Create one for free](https://azure.microsoft.com/free/cognitive-services)
* The current version of [Node.js](https://nodejs.org/)
* Once you have your Azure subscription, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesAnomalyDetector"  title="Create an Anomaly Detector resource"  target="_blank">create an Anomaly Detector resource </a> in the Azure portal to get your key and endpoint. Wait for it to deploy and click the **Go to resource** button.
    * You will need the key and endpoint from the resource you create to connect your application to the Anomaly Detector API. You'll paste your key and endpoint into the code below later in the quickstart.
    You can use the free pricing tier (`F0`) to try the service, and upgrade later to a paid tier for production.

## Setting up

[!INCLUDE [anomaly-detector-environment-variables](../environment-variables.md)]

### Create a new Node.js application

In a console window (such as cmd, PowerShell, or Bash), create a new directory for your app, and navigate to it. 

```console
mkdir myapp && cd myapp
```

Run the `npm init` command to create a node application with a `package.json` file. 

```console
npm init
```

Create a file named `index.js` and import the following libraries:

[!code-javascript[Import statements](~/cognitive-services-quickstart-code/javascript/AnomalyDetector/anomaly_detector_quickstart.js?name=imports)]

Create variables your resource's Azure endpoint and key. If you created the environment variable after you launched the application, you will need to close and reopen the editor, IDE, or shell running it to access the variable. Create another variable for the example data file you will download in a later step, and an empty list for the data points. Then create a `ApiKeyCredentials` object to contain the key.

[!code-javascript[Initial endpoint and key variables](~/cognitive-services-quickstart-code/javascript/AnomalyDetector/anomaly_detector_quickstart.js?name=vars)]

### Install the client library

Install the `ms-rest-azure` and `azure-cognitiveservices-anomalydetector` NPM packages. The csv-parse library is also used in this quickstart:

```console
npm install @azure/ai-anomaly-detector @azure/ms-rest-js csv-parse
```

Your app's `package.json` file will be updated with the dependencies.

## Object model

The Anomaly Detector client is an [AnomalyDetectorClient](/javascript/api/@azure/cognitiveservices-anomalydetector/anomalydetectorclient) object that authenticates to Azure using your key. The client can do anomaly detection on an entire dataset using [entireDetect()](/javascript/api/@azure/cognitiveservices-anomalydetector/anomalydetectorclient#entiredetect-request--servicecallback-entiredetectresponse--), or on the latest data point using [LastDetect()](/javascript/api/@azure/cognitiveservices-anomalydetector/anomalydetectorclient#lastdetect-request--msrest-requestoptionsbase-). The [ChangePointDetectAsync](https://go.microsoft.com/fwlink/?linkid=2090788) method detects points that mark changes in a trend. 

Time series data is sent as series of [Points](/javascript/api/@azure/cognitiveservices-anomalydetector/point) in a [Request](/javascript/api/@azure/cognitiveservices-anomalydetector/request) object. The `Request` object contains properties to describe the data ([Granularity](/javascript/api/@azure/cognitiveservices-anomalydetector/request#granularity) for example), and parameters for the anomaly detection. 

The Anomaly Detector response is a [LastDetectResponse](/javascript/api/@azure/cognitiveservices-anomalydetector/lastdetectresponse), [EntireDetectResponse](/javascript/api/@azure/cognitiveservices-anomalydetector/entiredetectresponse), or [ChangePointDetectResponse](https://go.microsoft.com/fwlink/?linkid=2090788) object depending on the method used. 

## Code examples 

These code snippets show you how to do the following with the Anomaly Detector client library for Node.js:

* [Authenticate the client](#authenticate-the-client)
* [Load a time series data set from a file](#load-time-series-data-from-a-file)
* [Detect anomalies in the entire data set](#detect-anomalies-in-the-entire-data-set) 
* [Detect the anomaly status of the latest data point](#detect-the-anomaly-status-of-the-latest-data-point)
* [Detect the change points in the data set](#detect-change-points-in-the-data-set)

## Authenticate the client

Instantiate a [AnomalyDetectorClient](/javascript/api/@azure/cognitiveservices-anomalydetector/anomalydetectorclient) object with your endpoint and credentials.

[!code-javascript[Authentication](~/cognitive-services-quickstart-code/javascript/AnomalyDetector/anomaly_detector_quickstart.js?name=authentication)]

## Load time series data from a file

Download the example data for this quickstart from [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/AnomalyDetector/request-data.csv):
1. In your browser, right-click **Raw**.
2. Click **Save link as**.
3. Save the file to your application directory, as a .csv file.

This time series data is formatted as a .csv file, and will be sent to the Anomaly Detector API.

Read your data file with the csv-parse library's `readFileSync()` method, and parse the file with `parse()`. For each line, push a [Point](/javascript/api/@azure/cognitiveservices-anomalydetector/point) object containing the timestamp, and the numeric value.

[!code-javascript[Read the data file](~/cognitive-services-quickstart-code/javascript/AnomalyDetector/anomaly_detector_quickstart.js?name=readFile)]

## Detect anomalies in the entire data set 

Call the API to detect anomalies through the entire time series as a batch with the client's [entireDetect()](/javascript/api/@azure/cognitiveservices-anomalydetector/anomalydetectorclient#entiredetect-request--msrest-requestoptionsbase-) method. Store the returned [EntireDetectResponse](/javascript/api/@azure/cognitiveservices-anomalydetector/entiredetectresponse) object. Iterate through the response's `isAnomaly` list, and print the index of any `true` values. These values correspond to the index of anomalous data points, if any were found.

[!code-javascript[Batch detection function](~/cognitive-services-quickstart-code/javascript/AnomalyDetector/anomaly_detector_quickstart.js?name=batchCall)]

## Detect the anomaly status of the latest data point

Call the Anomaly Detector API to determine if your latest data point is an anomaly using the client's [lastDetect()](/javascript/api/@azure/cognitiveservices-anomalydetector/anomalydetectorclient#lastdetect-request--msrest-requestoptionsbase-) method, and store the returned [LastDetectResponse](/javascript/api/@azure/cognitiveservices-anomalydetector/lastdetectresponse) object. The response's `isAnomaly` value is a boolean that specifies that point's anomaly status.  

[!code-javascript[Last point detection function](~/cognitive-services-quickstart-code/javascript/AnomalyDetector/anomaly_detector_quickstart.js?name=lastDetection)]

## Detect change points in the data set

Call the API to detect change points in the time series with the client's [detectChangePoint()](https://go.microsoft.com/fwlink/?linkid=2090788) method. Store the returned [ChangePointDetectResponse](https://go.microsoft.com/fwlink/?linkid=2090788) object. Iterate through the response's `isChangePoint` list, and print the index of any `true` values. These values correspond to the indices of trend change points, if any were found.

[!code-javascript[detect change points](~/cognitive-services-quickstart-code/javascript/AnomalyDetector/anomaly_detector_quickstart.js?name=changePointDetection)]

## Run the application

Run the application with the `node` command on your quickstart file.

```console
node index.js
```

[!INCLUDE [anomaly-detector-next-steps](../quickstart-cleanup-next-steps.md)]
