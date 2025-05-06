---
layout: default-layout
needAutoGenerateSidebar: true
needGenerateH3Content: true
noTitleIndex: true
title: MRZ Scanner JavaScript Edition - Release Notes
keywords: Documentation, MRZ Scanner JavaScript Edition, Release Notes
breadcrumbText: Release Notes
description: MRZ Scanner JavaScript Edition Documentation Release Notes
permalink: /releasenotes/index.html
---

# Release Notes

## 2.0.0 (03/13/2025)

The **MRZ Scanner JavaScript Edition** has been redesigned and redeveloped to now include a **ready-to-use, fully developed UI** to ease the development process while providing even better functionality.

## Highlighted Features

- Automatic detection and parsing of MRZs in passports and IDs
- Support for the following MRTD formats: TD3 (Passport), TD2 (ID), and TD1 (ID)
- Ready-to-use UI to simplify the development process
- Supports an interactive video scenario (capturing via video) as well as static images (jpg/png)
- Modular, view-based design for easy maintenance and customization

## Views

MRZ Scanner JavaScript Edition is organized into configurable UI views. Below is a quick overview of the two main views:

> Learn more about these views and how to configure them in the [User Guide]({{ site.guides }}mrz-scanner.html) and the [Customization Guide]({{ site.guides }}mrz-scanner-customization.html).

### MRZ Scanner View

- Fully configurable camera view
- Resolution/camera select dropdown to allow for quick improvements
- Newly developed scan guide frame to enable easy capture of the MRZ document
- Green highlight overlay over the MRZ area once recognized
- Flash/Torch support
- Allows the user to load in an image from the photo library

### MRZ Result View

- Scrollable field view to display the parsed MRZ info
- Displays the original image of the MRZ document
- Allows the user to edit almost all of the parsed fields
- Two buttons to allow the user to re-scan or move on