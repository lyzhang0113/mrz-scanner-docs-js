---
layout: default-layout
needAutoGenerateSidebar: true
needGenerateH3Content: true
noTitleIndex: false
title: Dynamsoft MRZ Scanner JavaScript Edition
keywords: Documentation, MRZ Scanner, Dynamsoft MRZ Scanner JavaScript Edition, Customization
description: Customizing the Dynamsoft MRZ Scanner
permalink: /guides/mrz-scanner-customization.html
---

# Customizing the MRZ Scanner JavaScript Edition

>**Prerequisite**
>
>Before going into the ways that you can customize the MRZ Scanner, please read the [MRZ Scanner JavaScript Edition User Guide]({{ site.guides }}mrz-scanner.html).

This guide expands on the User Guide that explored the MRZ Scanner Hello World sample project. Here we explore ways to customize the UI as well as the performance of the MRZ Scanner. We will walk through the three main configuration interfaces - **`MRZScannerConfig`**, **`MRZScannerViewConfig`**, and **`MRZResultViewConfig`**. These configuration interfaces make customizing the MRZ Scanner as easy as adding or changing a few properties in the instance constructor. Every sample is a variation on the previous Hello World sample with a few additional properties defined in the configuration interfaces, and so we only show the differing portion rather than all the code.

## `MRZScannerConfig` Overview

The **`MRZScannerConfig`** interface is capable of configuring almost all customization options applicable to MRZ scanning use cases with the MRZ Scanner. The MRZ Scanner uses passes an `MRZScannerConfig` object to the constructor when creating an MRZ Scanner instance. `MRZScannerConfig` contains the following properties:

1. **`license`** - the license key is the only property whose ***value must be specified when instantiating the MRZ Scanner instance***. If the license is undefined, invalid, or expired, the MRZ Scanner cannot proceed with scanning, and instead displays a pop-up error message instructing the user to contact the site administrator to resolve this license issue.

2. **`container`** - pass a DOM element to this property to contain the entire MRZ Scanner UI within that DOM element. This property is optional. When not specified, (e.g. in our [Hello World sample]({{ site.guides }}mrz-scanner.html#quick-start-hello-world)) the MRZ Scanner automatically creates its own container upon instantiation and uses that container instead.

3. **`templateFilePath`** - A template file is a JSON file that contains a series of algorithm parameter settings (called Capture Vision templates) that is usually used for very specific and customized scanning and parsing scenarios. The `templateFilePath` points to the location of the JSON file. The MRZ Scanner comes with a default template file, but you may choose to use a custom template to target specialized use cases. We recommend contacting the [Dynamsoft Technical Support Team](https://www.dynamsoft.com/company/contact/) for assistance with template customization. Simply host the custom template file on the hosting server of the web application and use the `templateFilePath` to define its location to the MRZ Scanner.

4. **`utilizedTemplateNames`** - this property defines the names of Capture Vision template(s) defined in the non-default template file pointed to by `templateFilePath`. These names must be declared in this property when using a custom template with `templateFilePath`.

5. **`engineResourcePaths`** - The engine files of the library make up the core of the library and define the operation as well as the UI. `engineResourcePaths` defines the location of the engine files in case they are being referenced from another different location. This property is typically used with frameworks such as **React**, **Angular**, or any other framework that makes use of a package manager like **npm** or **yarn**.

6. **`scannerViewConfig`** - This is the configuration interface for the `MRZScannerView`, which is responsible for the main scanning functionality as well as the camera UI. We explain the configuration properties nested in this object in the [`MRZScannerViewConfig` overview](#mrzscannerviewconfig-overview).

7. **`resultViewConfig`** - This is the configuration interface for the `MRZResultView`, which is responsible for displaying the scanned MRZ document and its parsed data after a successful scan. You can find the breakdown of the `MRZResultView` settings in the [MRZResultView overview](#mrzresultviewconfig-overview).

8. **`mrzFormatType`** - This configures the available MRTD formats that the MRZ Scanner can read. The formats set in `mrzFormatType` are the formats that appear in the format selector box within the **`MRZScannerView`**. By default, the library will include all of the supported MRTD formats.To learn more about the different MRTD formats the library supports, visit the introduction page for [more details]({{ site.introduction }}index.md#supported-mrz-formats).

9. **`showResultView`** - This toggles the visibility of the **`MRZResultView`**. If `false`, the MRZ Scanner immediately closes upon a successful scan rather than going into the `MRZResultView`, then proceeds to the next step of the web application's workflow (outside the purview of the MRZ Scanner SDK). In the case of the Hello World sample, the next step just takes the user back to the landing page.

Next, we go over the different ways that these properties can be used to customize the scanner with a few samples.

### Setting the MRTD formats

> Prerequisite: [Introduction to MRZ Formats]({{ site.introduction }}index.html#supported-mrz-formats)

The MRZ Scanner reads all three MRZ formats, but it can optionally restrict the MRZ format that it reads. For example, you may want to configure MRZ scanner to only read **TD1** and passport (**TD3**) document types, while **ignoring TD2** documents. Here is a quick snippet based on the Hello World code (from the [User Guide]({{ site.guides }}mrz-scanner.html)) that sets the specific MRZ formats to read using the `mrzFormatType1` property:

```ts
const mrzScanner = new Dynamsoft.MRZScanner({
   license: "YOUR_LICENSE_KEY_HERE",
   mrzFormatType: ["passport", "td1"], // setting it to just TD3 (passport) and TD1
});
```

After changing the `**mrzFormatType**`, the format selector box of the MRZScannerView reflects the two formats selected above instead of all three formats available by default. If you only set a single MRZ format with `**mrzFormatType**`, the format selector box of the MRZScannerView *does not appear, even if showFormatSelector is set to `true`*.

### Hiding the Result View

You may not want to use the Result View that the MRZ Scanner provides. If your workflow does not require the user to view or verify the scanned results, you may opt to forego the Result View after the SDK finishes a scan, then exit the MRZ Scanner portion of your application workflow. Here is how to configure the MRZScanner to hide the result view:

```ts
const mrzScanner = new Dynamsoft.MRZScanner({
   license: "YOUR_LICENSE_KEY_HERE",
   showResultView: false,
});
```

## MRZScannerViewConfig Overview

`**MRZScannerViewConfig**` controls the UI elements of the `**MRZScannerView**`, which is the view responsible for scanning operations. Here are its three properties in detail:

1. `**cameraEnhancerUIPath**` - defines the path to a custom HTML user interface file for the `**MRZScannerView**`, which is based on the UI for the Dynamsoft Camera Enhancer SDK used by the MRZ Scanner. Setting the path to a custom file allows this custom UI to take effect for any `**MRZScanner**` instance created within your application. We recommend you to reach out to the [Dynamsoft Technical Support Team](https://www.dynamsoft.com/company/contact/) to assist you in creating such a custom UI.

2. `**container**` - assigns a specific DOM element to contain the `**MRZScannerView**` in. By default, when not specified, the MRZScanner creates its own container for this View automatically.

3. `**showScanGuide**` - toggles the *scan guide* (defaults to `true`). Other than the actual camera view, one of the main elements in the `**MRZScannerView**` is a *scan guide frame*. When enabled, placing the MRZ document within the boundaries of the scan guide frame allows the library to quickly and accurately recognize the MRZ and decipher it. When enabled, the MRZ Scanner does not read outside the confines of the scan guide, so enabling the scan guide reduces the need to read the entire camera frame.
<!-- 4.  There are three frames, one for each MRTD format that the library supports. Going from left to right, the first scan guide corresponds to the TD3 (Passport) format, the second is the TD2 (ID) format, and the third is the TD1 (ID) format.

<div align="center">
   <img src="../assets/imgs/mrz-scan-guides.png" alt="Scan Guide Frames" width="80%" />
</div> -->

4. **showUploadImage** - In addition to scanning via a camera, the MRZScanner also has the ability to read MRZs off static images from the device's local library. However, if you would like to disable that feature in your own implementation, then all you need to do is set this property to *false* as the icon to load an image will be displayed by default at the top of the **MRZScannerView**.

5. **showFormatSelector** - The **MRZScannerView** comes with a selector box towards the bottom of the view that allows the user to disable or enable certain MRTD formats. The formats that show up in this selector box are defined by the **mrzFormatType** property of the **MRZScannerConfig**. Please note that a single format must be selected at any time. The scan guide frame (if shown) will also change based on which format(s) are selected - which is explained more in this [section](#changing-the-scan-guide-frame). By default, the format selector box is set to appear, so setting this to *false* will make it go away.

<div align="center">
   <img src="../assets/imgs/format-selector.png" alt="Scan Guide Frames" width="40%" />
</div>

6. **showSoundToggle** - The MRZ Scanner also has the ability to provide the user a beep sound once a MRZ is recognized depending on the browser being used as not all browsers support this feature. By default, the sound toggle icon will be displayed at the top of the MRZScannerView and it will be disabled (grey). To hide this feature altogether, set this property to *false*.

7. **enableMultiFrameCrossFilter** - The multi-frame result cross filter is a feature that ensures the library relays the most accurate results. It is disabled by default, so if you choose to enable it, please note that there could be a slight time cost.

> **Important Note**
>
> Not every UI element of the MRZScannerView can be controlled by the MRZScannerViewConfig, namely the **torch/flash button**. The torch button will always show up in the **MRZScannerView**.

### Using the MRZScannerViewConfig

Now that we have gone through all the properties that make up the MRZScannerViewConfig, let's see them in action:

```ts
const mrzScanner = new Dynamsoft.MRZScanner({
   license: "YOUR_LICENSE_KEY_HERE",
   scannerViewConfig: {
      showScanGuide: false, // hides the scan guide frame; true by default
      showUploadImage: false, // hides the load image icon that shows up in the toolbar at the top of the view; true by default
      showFormatSelector: false, // hides the format selector box if more than two MRZ types are assigned; true by default
      showSoundToggle: false, // hides the sound icon that allows the user to control whether a beep is played once an MRZ is recognized; true by default
      enableMultiFrameCrossFilter: false, // turning the filter off could improve the speed but at the cost of result accuracy; true by default
   }
});
```

### Changing the Scan Guide Frame

As we have shown above, the library comes with different frames depending on the type of MRZ document that is being scanned. However, in this section we will discuss which frames are displayed when multiple MRTD formats are selected. Here are the current rules for the guide frames that are implemented in the library:

1. If **passport** is selected at any time (or if it is the only selection), then the guide frame for passport (TD3) will be displayed.

2. If both **ID (TD1)** and **ID (TD2)** are selected or only **ID (TD1)**, but not passport, then the frame for ID (TD1) will be displayed.

3. If **ID (TD2)** is the only format selected, then the guide frame for ID(TD2) is displayed.

If you have any questions about this or would like to implement your own customization, please contact the [Dynamsoft Support Team](https://www.dynamsoft.com/company/contact/).

## MRZResultViewConfig Overview

**MRZResultViewConfig** is used to display the parsed MRZ results into readable fields, saving time and resources needed to build your own viewer. In this section, we will explore the different UI elements of the result view and how they can be configured.

1. **container** - In order to contain the **MRZResultView** within a specific DOM element, then that DOM element must be assigned to this property. If it is not specified (like in the Hello World sample) then a container is automatically created.

2. **toolbarButtonsConfig** - The result view comes with a toolbar at the bottom (in portrait mode) or at the right side (in landscape mode) that has two buttons by default. The first button is a **Re-take button** that allows the user to go back to the MRZScannerView and scan another MRZ document. The other button is a **Done button** that closes the scanner and destroys the instance. In the next section, we will explain how these buttons can be customized via **toolbarButtonsConfig**.

3. **showOriginalImage** - By default, the MRZResultView displays a cropped image of the MRZ document at the top. If you would like to hide this image so that it is only the parsed result info that appears, then all you need to do is set this property to *false*.

4. **allowResultEditing** - In certain cases, the MRZ result info that is parsed by the library might come out incorrect or not exactly match the info that is present on the MRZ document. In order to deal with those cases, the library offers the capability for the user to edit the result fields after cross-checking them with the info present on the document itself.

5. **onDone** - In order to change the action(s) that are taken once the user clicks the *Done* button in the, you must use the *onDone* callback function of the **MRZResultViewConfig**. The function comes with the MRZResult object that represents the full MRZ result in case the developer would like to do something with the result after the scanner closes (once *Done* is clicked). Please read the section on changing the Done actions to learn how to implement this callback. 

### Using the MRZResultViewConfig

Now that we have learned about the properties of the MRZResultViewConfig interface, let's now demonstrate how to use it in a simple code snippet:

```ts
const mrzScanner = new Dynamsoft.MRZScanner({
   license: "YOUR_LICENSE_KEY_HERE",
   scannerViewConfig: {
      /* see the MRZScannerViewConfig section to learn how to set this */
   },
   resultViewConfig: {
      showOriginalImage: false, // hides the cropped image of the MRZ document in the result view; true by default
      allowResultEditing: false, // disables the ability to edit the result fields should the parsed information not match the MRZ document; true by default
      toolbarButtonsConfig: {
         retake: {
            icon: "path to a png/svg file" // Changes the icon image of the retake button
            label: "Re-scan", // Change the text label of the retake button to the provided string; string is "Re-take" by default
            isHidden: true, // Hides the retake button; false by default
            className: "custom class name" // to implement a custom css to the done button, you can assign a custom css class to the button here
         },
         done: {
            icon: "path to a png/svg file" // Changes the icon image of the retake button
            label: "Return Home", // Change the text label of the done button to the provided string; string is "Done" by default
            isHidden: true, // Hides the done button; false by default
            className: "custom class name" // to implement a custom css to the done button, you can assign a custom css class to the button here
         }
      }
   }
});
```

### Changing the Done actions

The default action once the user clicks the *Done* is that the scanner closes and the user is taken back to the landing page. However, this action can be changed via the **onDone** callback function of the **MRZResultViewConfig**. Let's demonstrate that:

```ts
const mrzScanner = new Dynamsoft.MRZScanner({
   license: "YOUR_LICENSE_KEY_HERE",
   scannerViewConfig: {
      /* see the MRZScannerViewConfig section to learn how to set this */
   },
   resultViewConfig: {
      onDone: (result) => {
         console.log(result.status.message); // print the result status message to the console
         console.log(result.status.code); // print the result status code
         console.log(result.data.firstName); // print the first name from the MRZ info
      }
   }
});
```

In the above code, we simply just print the result status code, status message, and the first name to the console once the user clicks *Done* and the scanner closes. Of course, the action doesn't have to be this simple as this is just a code snippet. You can choose to navigate to a new page as a next step in your workflow, or really whatever the situation calls for.
