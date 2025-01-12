---
id: overview
title: DICOM Service
sidebar_label: Overview
---
The DICOM Service is a plugin that allows **interfacing with systems that use the webDICOM standard for exchanging DICOM (Digital Imaging and COmmunications in Medicine) documents**. Those systems are called **PACS (Picture archiving and communication system)**.

The service has a **custom interface**, that is, it does not expose the endpoints defined by the webDICOM standard. In fact, the main purpose of the DICOM Service is to abstract and hide the presence of a PACS that interfaces with your system. This allows you to deal with interaction with the PACS **without knowledge of the webDICOM standard**. It also allows you to decouple the part of your system that generates and manages DICOM files with the saving and viewing part (see also **DICOM Viewer** for further details). The overall advantage is making interaction with the reference PACS more flexible and easier.

:::info
Technical nomenclature closely related to the DICOM standard will be used. In order to better understanding the following chapters, it is strongly recommended to refer to the official documentation of the [DICOM standard](https://www.dicomstandard.org/), with a focus on the communication procedures defined by the [webDICOM standard](https://www.dicomstandard.org/using/dicomweb).
:::

## Usage
The DICOM Service **exposes a subset of capabilities compared to those offered by the webDICOM standard**. In particular, the service allows to:
* Create a new workitem
* Retrieve the list of workitems
* Delete a specific workitem
* Upload an exam result (i.e. a DICOM file)
* Retrieve the thumbnail of a specific exam
* Retrieve the list of the exams

The previous capabilities are based on the interface defined by the webDICOM standard. In addition to these, the service provides additional capabilities that extend the capabilities of the webDICOM standard. Specifically, these are:
* Retrieve the OTP used to access specific exam data
* Getting notified when a new exam has been uploaded
* Generate a unique UID that can be assigned to DICOM entities
