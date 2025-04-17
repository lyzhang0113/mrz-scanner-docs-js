---
layout: default-layout
needAutoGenerateSidebar: true
needGenerateH3Content: true
noTitleIndex: false
title: Dynamsoft MRZ Scanner JavaScript Edition - Reading from Local Images
keywords: Documentation, MRZ Scanner, Dynamsoft MRZ Scanner JavaScript Edition, Local Images, Load
description: Reading from Local Images with the Dynamsoft MRZ Scanner JavaScript Edition
permalink: /codegallery/dwt-mrz/index.html
---

# Reading MRZs from Local Images - JavaScript Edition

So far, the [MRZ Scanner JS User Guide]({{ site.guides }}mrz-scanner.html) addressed how you can use the library to scan MRZs from a camera in an interactive live video scenario. The majority of use cases will involve using the camera of a phone or a laptop directly, but there could be cases where a camera is not needed at all. Although the **MRZ Scanner JavaScript Edition** has the ability to read from local static images via the Load Image feature of the MRZScannerView, that still requires connecting to a valid camera input.

In this article, we will address how you can use the foundational products of the MRZ Scanner (Dynamsoft Label Recognizer, Dynamsoft Code Parser) to implement the ability to read from static images without the need for a camera input.

## Initialization

The first thing that needs to be determined is what the source of the images will be. There are several methods in which you can load images in JavaScript application, but in this article we will be using another Dynamsoft product, [Dynamic Web TWAIN]({{ site.dwt }}about/index.html).

Dynamic Web TWAIN (DWT for short) allows users to acquire images from scanners in a web application, as well as load or download existing documents. One of the advantages of using Dynamic Web TWAIN is that it allows the user to load in PDFs as well as images. So whether you are loading in a local image or acquiring an image from a physical scanner, Dynamic Web TWAIN keeps things simple and implements these features with a few simple API.

### Setting up the Resources

The first thing to do when creating a web application that utilizes DWT is to include the Resources folder of DWT. In this folder, the template for the MRZ Reader needs to also be included since it will be referenced by the MRZ Reader. To get a copy of the DWT Resources, please download the [30-day free trial](https://www.dynamsoft.com/web-twain/downloads) of Dynamic Web TWAIN.

Alternatively, you can download the [full sample code](https://github.com/Dynamsoft/mrz-scanner-javascript/tree/main/samples/dwt-mrz) from the Github repository of the MRZ Scanner Samples. Please use this sample as reference when building your own application.

### Initializing the Input Source and Capture Vision Instance

Let's now introduce the full code needed to implement the basic functionalities of Dynamic Web TWAIN as well as the initialization of the MRZ reader using the [foundational API]({{ site.dcvb_js_api }}index.html) of Dynamsoft Capture Vision.

```html
<!DOCTYPE html>
<html>

<head>
    <title>Hello World</title>
    <script type="text/javascript" src="Resources/dynamsoft.webtwain.initiate.js"></script>
    <script type="text/javascript" src="Resources/dynamsoft.webtwain.config.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/dynamsoft-capture-vision-bundle@2.6.1000/dist/dcv.bundle.js"></script>
</head>

<body>
	<input type="button" value="Scan" onclick="AcquireImage();" />
    <input type="button" value="Load" onclick="LoadImage();" />
    <input type="button" value="Read MRZ" onclick="readMRZ();" />
    <div id="dwtcontrolContainer" style="width: 350px; height: 380px;"></div>

    <script type="text/javascript">
		var DWTObject;
        var cvRouter;

		Dynamsoft.DWT.RegisterEvent("OnWebTwainReady", function () {
            initializeMRZ();
			DWTObject = Dynamsoft.DWT.GetWebTwain('dwtcontrolContainer'); // Get the Dynamic Web TWAIN object that is embedded in the div with id 'dwtcontrolContainer'
		});

        /* The objective of this function is to initialize the Capture Vision instance and apply the necessary settings for reading MRZ */
        async function initializeMRZ() {
            // Inialize License Key
            Dynamsoft.License.LicenseManager.initLicense("MRZ_LICENSE_KEY_HERE");
            
            // Load MRZ Resources
            Dynamsoft.Core.CoreModule.loadWasm(["DLR", "DCP"]);
            Dynamsoft.DCP.CodeParserModule.loadSpec("MRTD_TD3_PASSPORT");
            Dynamsoft.DCP.CodeParserModule.loadSpec("MRTD_TD1_ID");
            Dynamsoft.DCP.CodeParserModule.loadSpec("MRTD_TD2_ID");
            Dynamsoft.DLR.LabelRecognizerModule.loadRecognitionData("MRZ");

            // Create Capture Vision Router Instance with MRZ templates
            cvRouter = await Dynamsoft.CVR.CaptureVisionRouter.createInstance();
            await cvRouter.initSettings("./Resources/mrz-template.json");

            console.log("MRZ Initialized Successfully");
        }

        /* This function is used to acquire an image from a physical scanner using the DWT API */
		function AcquireImage() {
            if (DWTObject) {
				DWTObject.SelectSourceAsync().then(function(){
					return DWTObject.AcquireImageAsync({
					        IfCloseSourceAfterAcquire: true // Scanner source will be closed automatically after the scan.
						});
				}).catch(function (exp) {
					alert(exp.message);
				});
            }
        }

        //Callback functions for async APIs
        function OnSuccess() {
            console.log('successful');
        }

        function OnFailure(errorCode, errorString) {
            if(errorCode != -2326)
			alert(errorString);
        }

        /* The fucntion that is triggered when the user clicks the "Load" button */
        function LoadImage() {
            if (DWTObject) {
                DWTObject.IfShowFileDialog = true; // Open the system's file dialog to load image
                DWTObject.LoadImageEx("", Dynamsoft.DWT.EnumDWT_ImageType.IT_ALL, OnSuccess, OnFailure); // Load images in all supported formats (.bmp, .jpg, .tif, .png, .pdf). OnSuccess or OnFailure will be called after the operation
            }
        }

        /* A function to check if there are any images in the DWT buffer */
        function checkIfImagesInBuffer() {
            if (DWTObject.HowManyImagesInBuffer == 0) {
                console.log("There is no image in the buffer.")
                return false;
            }
            else
                return true;
        }
    </script>
</body>

</html>
```

## Implementing the MRZ Reader

Now that we introduced the code to initialize DWT and the MRZ Reader, as well as defined the functions that implement the scanning and loading capabilities of Web TWAIN, it's time now to implement the MRZ reading portion of the code. Continuing the **script** from the previous section:

```js
// Used by the Button on page, to trigger a MRZ read with the current select page
async function readMRZ() {
    if (!checkIfImagesInBuffer()) {
        return;
    }
    if (!cvRouter) {
        console.log("cvRouter is not initialized.");
        return;
    }
    console.log("Reading MRZ on Index " + DWTObject.CurrentImageIndexInBuffer);
    let imgURL = DWTObject.GetImageURL(DWTObject.CurrentImageIndexInBuffer);
    let capturedResults = await cvRouter.capture(imgURL, "ReadPassportAndId");
    const recognizedResults = capturedResults.textLineResultItems;
    const parsedResults = capturedResults.parsedResultItems;
    console.log(parsedResults);
    if (!parsedResults || !parsedResults.length) {
        console.log("No Result");
        return;
    }
    
    let extractedResults = JSON.stringify(extractDocumentFields(parsedResults[0]));
    console.log(extractedResults); // print the result to the console
    extractedResults = extractedResults.split(",").join("\n");
    extractedResults = extractedResults.replace(/{"text":|[{"}]/g, "");
    document.getElementById("divNoteMessage").innerHTML = extractedResults; // print the result to the result div
}

/**
 * Extracts and returns document fields from the parsed MRZ result
 *
 * @param {Object} result - The parsed result object containing document fields.
 * @returns {Object} An object with key-value pairs of the extracted fields.
 */
function extractDocumentFields(result) {
    if (!result || result.exception) {
    return {};
    }

    const fieldWithStatus = (fieldName, raw=false) => ({
    text: raw ? result.getFieldRawValue(fieldName) : result.getFieldValue(fieldName),
    });

    const parseDate = (yearField, monthField, dayField) => {
        const year = result.getFieldValue(yearField);
        const currentYear = new Date().getFullYear() % 100;
        const baseYear =
            yearField === "expiryYear" ? (parseInt(year) >= 60 ? "19" : "20") : parseInt(year) > currentYear ? "19" : "20";
    
        return {
            text: `${baseYear}${year}-${result.getFieldValue(monthField)}-${result.getFieldValue(dayField)}`,
        };
    };

    // Add special case for Spanish IDs
    const getDocumentNumber = (codeType) => {
    const documentType = mapDocumentType(codeType);
    const documentNumberField =
    documentType === "passport" && codeType === "MRTD_TD3_PASSPORT"
        ? "passportNumber"
        : result.getFieldRawValue("nationality") === "ESP" ? 
        "optionalData1" : "documentNumber";

    const primaryNumber = fieldWithStatus(documentNumberField);
    const longNumber = fieldWithStatus("longDocumentNumber");

    return primaryNumber?.text ? primaryNumber : longNumber;
    };

    // Document Type and Name
    const codeType = result.codeType;

    return {
    Surname: fieldWithStatus("primaryIdentifier"),
    "Given Name": fieldWithStatus("secondaryIdentifier"),
    Nationality: fieldWithStatus("nationality", true),
    "Document Number": getDocumentNumber(codeType),
    "Document Type": documentTypeLabel(codeType),
    "Issuing State": fieldWithStatus("issuingState", true),
    Sex: fieldWithStatus("sex", true),
    "Date of Birth (YYYY-MM-DD)": parseDate("birthYear", "birthMonth", "birthDay"),
    "Date of Expiry (YYYY-MM-DD)": parseDate("expiryYear", "expiryMonth", "expiryDay"),
    "Document Type": JSON.parse(result.jsonString).CodeType,
    };
}

function mapDocumentType(codeType) {
    switch (codeType) {
    case "MRTD_TD1_ID":
        return "td1";

    case "MRTD_TD2_ID":
    case "MRTD_TD2_VISA":
    case "MRTD_TD2_FRENCH_ID":
        return "td2";

    case "MRTD_TD3_PASSPORT":
    case "MRTD_TD3_VISA":
        return "passport";

    default:
        throw new Error(`Unknown document type: ${codeType}`);
    }
}

function documentTypeLabel(codeType) {
    switch (codeType) {
    case "MRTD_TD1_ID":
        return "ID (TD1)";

    case "MRTD_TD2_ID":
        return "ID (TD2)";
    case "MRTD_TD2_VISA":
        return "ID (VISA)";
    case "MRTD_TD2_FRENCH_ID":
        return "French ID (TD2)";

    case "MRTD_TD3_PASSPORT":
        return "Passport (TD3)";
    case "MRTD_TD3_VISA":
        return "Visa (TD3)";

    default:
        throw new Error(`Unknown document type: ${codeType}`);
    }
}
```

By completing the script with the code above, you can now go ahead and test the app that you just created. Please note that in order to properly function, the sample will need to be hosted on a server. 

If you are using VS Code, a quick and easy way to serve the project is using the [Five Server VSCode extension](https://marketplace.visualstudio.com/items?itemName=yandeu.five-server). Simply install the extension, open the `hello-world.html` file in the editor, and click "Go Live" in the bottom right corner of the editor. This will serve the application at `http://127.0.0.1:5500/hello-world.html`. Alternatively, you can also use IIS or Github Pages to quickly serve your application.

## Final Comments

This sample was built based on the Hello World sample code of Web TWAIN as well as the Hello World code of the previous version of the MRZ Scanner that was dependent fully on the foundational API of Dynamsoft Capture Vision. Here are some references to check out:

- [Dynamic Web TWAIN User Guide]({{ site.dwt }}general-usage/index.html)
- [MRZ Scanner User Guide using Foundational DCV API](https://www.dynamsoft.com/mrz-scanner/docs/web/guides/mrz-scanner-v1.1.html?ver=1.1&matchVer=true&cVer=true)
- [Dynamic Web TWAIN API Reference]({{ site.dwt }}info/api/)
- [DCV Foundational API Reference]({{ site.dcvb_js_api }}index.html)

If there are any questions regarding this sample or your own implementation, do not hesitate to get in touch with the [Dynamsoft Support Team](https://www.dynamsoft.com/company/contact/).