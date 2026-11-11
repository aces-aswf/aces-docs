<!-- SPDX-License-Identifier: CC-BY-4.0 -->
<!-- Copyright Contributors to the ACES Documentation -->


Transform IDs and User Names
============================

ACES transforms shall be assigned both a Transform Identifier (`<ACEStransformID>`) and a user-facing name (`<ACESuserName>`).

## Transform Identifiers (Transform IDs)
ACES transforms shall be assigned a Transform Identifier ("Transform ID") contained in an XML tag of `<ACEStransformID>` in the file header. The Transform ID describes the transform and includes versioning information specific to that transform file. 

A Transform ID is required metadata in AMF files. Transform IDs can also be used for tracking in software implementations that are intended to match the reference ACES transforms, where appropriate, such as in code comments implementing a specific ACES Transform.
 
Transform implementations that may be intended to combine a sequence of ACES CTL transforms (e.g. a Color Space Conversion (CSC) + Look Transform (LMT) + Output Transform) should reference each of the relevant ACES Transform IDs in the implementation transform.

## String format for Transform Identifiers¶
The Transform ID serves as a unique identifier for a particular release version of an ACES component. It includes versioning information relevant to three different attributes:

* the version of the Transform ID specification to which it conforms, 
* the version of the ACES system release with which the transform is compatible, and 
* the version of the individual transform itself. 
 
Transform IDs are constructed using a set of specified tokens such that it will be more human-readable than a typical Universally Unique Identifier (UUID) but maintain enough structure that intelligent parsing can still be performed.

All ACES system components, whether Academy-supplied or vendor/user-supplied, shall use the generic versioning string format:

`URN` : `TransformType` . `Namespace` . `Name` . `aACESReleaseMajorVersionNumber` . `vTransformVersionNumber`

The allowed substrings for each of the changeable placeholders are described in the following subclauses. In the following definitions, _italics_ represent a changeable placeholder and **boldface** represents a required character or string. The substrings shall not contain spaces or periods and shall be limited to the ASCII character set.  

### URN¶
`URN` is a unique reference number to identify the version of the Transform ID specification used. For new implementations, it shall conform to the first data row in Table 1. 
 
<div align="center" markdown="1">
<figcaption> <b>Table 1</b> - String specification for Transform ID URN </figcaption>
| |Transform ID URN|
|----:|----|
| Current Version String: | **`urn:ampas:aces:transformId:v2.0:`** |
| Previous values: | `urn:ampas:aces:transformId:v1.5:` |
</div>

!!! note
    Throughout the remainder of this document, `urn:ampas:aces:transformId:v2.0` will be shortened to `URN` for brevity.

### TransformType¶
`TransformType` indicates the type, or category, of transform, and shall be equal to one of the values defined in Table 2.

<div align="center" markdown="1">
<figcaption> <b>Table 2</b> - Options for TransformType</figcaption>
</div>

| Value | Description |
|:----|:----|
| `CSC` | "Color Space Conversion" <br> A transform that converts between defined color encodings, color primary sets, and/or transfer functions. These include "white-paper" conversions between established camera encoding spaces and ACES2065-1. CSCs could also include conversions to and from other established color representation such as perceptual spaces.|
| `Input` | "Input Transform", a.k.a. Input Device Transform (IDT) <br> A transform intended to convert source camera data into ACES2065-1.|
| `Output` | “Output Transform” <br> A transform that converts an ACES scene-referred image to a display-referred output. An Output Transforms is equivalent to the combination of the Reference Rendering Transform and Output Device Transform on ACES engineering diagrams, and prior to ACES 2 used the token RRTODT.|
| `InvOutput` | “Inverse direction of an Output Transform” <br> A transform that converts display-referred imagery into ACES values that reproduce the original display result when the corresponding forward Output Transform is applied. |
| `Look` | “Look Transform”, a.k.a. Look Modification Transform (LMT) <br> A transform that applies modifications to ACES data to alter the default rendering appearance for technical or creative reasons.|
| `InvLook` | “Inverse direction of a Look Transform” <br> The analytical inverse of a Look Transform, when such an inverse exists.|
| `Lib` | “Library Transform” <br> A reusable internal transform intended for use by other core transforms.|
| `Util` | “Utility Transform” <br> A non-core transform intended for testing, diagnostics, or pipeline construction (e.g., `Adjust_Exposure`, `Unity`).|


!!! note "Relationship between Input and Color Space Conversion Transforms"
    An Input Transform and CSC may perform identical mathematical operations, but they typically differ in intent. 

    Historically, an Input Transform describes the math used to bring camera values into ACES as the first step of an ACES pipeline, and may include device-specific or use-specific considerations that are not intended for general or bi-directional use. These might include user-created or custom transforms intended for specific illuminants, or transforms derived empirically from prosumer devices where linear sensor data is not fully accessible.

    A CSC describes a general, reusable and often reversible conversion between two defined color encodings or color representations. This includes conversions between camera encodings and ACES2065-1, and can be identical in function to Input Transforms in cases where cameras already map native sensor data directly to their own colorimetric encoding.
        
    When an Input transform and a CSC are mathematically identical, they can be considered interchangeable. In such cases, the CSC token is preferred for the transform type. It is not required to maintain duplicate transforms that differ only by filename or Transform ID. 

!!! note
    A CSC transform that converts from ACES _to_ a camera manufacturer’s encoding space may be considered an "Inverse Input Transform". However, no `InvInput` TransformType is defined. Therefore, CSC is the appropriate substring value for transforms of this type.


### Namespace¶
`Namespace` shall identify the creator of the transform. This is usually the name of a manufacturer or facility but could also be the name of an individual. 

The namespace `Academy` is reserved for Academy-supplied transforms.
 
Transforms provided by users or vendors to ACES Github repositories shall maintain the namespace of the user or vendor that created it.
 
### Name¶
`Name` shall describe the transform. The level of detail is left up to the user but it should be adequate to uniquely describe the transform and its intended usage. It does not need to try to include the settings for every possible parameter. 
 
For example, if there is a standard that describes an output display type, such as Rec. 2100, that can be used in the filename and Transform ID to imply the primaries to be Rec. 2020 with the transfer function of SMPTE ST 2084. If a transform imposes some sort of variant or limitation making it different from a standard display type, then those differences should be included in the filename and Transform ID to alert the user that different behavior is expected. 
 
#### Output Transforms
For Output Transforms, the `Name` shall describe the device and/or output data encoding in as much detail as necessary. 

ACES 2.0 adopted verbose filenames to provide consistent information to implementers in the filename - including the peak luminance, limiting color primaries and white point, encoding color primaries and white point, and transfer function. 

The generalized format is:

`LimitingColorPrimaries-White` _ `###nit` \_in_ `EncodingColorPrimaries-White` _ `TransferFunction`

Examples:

* `URN:Output.Academy.P3-D65_100nit_in_P3-D65_Gamma2pt2.a2.v1`
* `URN:Output.Academy.P3-D65_500nit_in_Rec2100-D65_ST2084.a2.v1`
* `URN:Output.Academy.P3-D60_300nit_in_XYZ-E_ST2084.a2.v1`

#### CSC Transforms
For Color Space Conversion (CSC) Transforms, `Name` shall describe the device. If the creator of the transforms is the same as the camera manufacturer, then the Name need not repeat the manufacturer name because it should already be in the Namespace. If the Input Transform creator is not the camera manufacturer, then the manufacturer name shall be prepended to the device name in the Name field.

For transforms from one color space to another, the Name shall describe the source color space to the destination color space, using the general form of `fromSpace` `_to_` `toSpace`.
 
Example Input Transform names:

* `URN:CSC.Academy.ACES_to_ACEScg.a2.v1`
* `URN:CSC.Blackmagic.ACES_to_BMDFilm_WideGamut_Gen5.a2.v2`
* `URN:CSC.Red.Log3G10_RWG_to_ACES.a2.v1`
 
#### ACES Major Version Number
This component of the Transform ID shall be a string consisting of the letter ‘**a**’ followed by the major version number of the current ACES system release. This indicates the ACES system version with which a particular transform is compatible. For  new implementations, it shall be equal to the first data row in Table 3. 
 
<div align="center" markdown="1">
<figcaption> <b>Table 3</b> - String specification for ACES Major Version Number </figcaption>
| | ACES Major Version Number|
|----:|----|
| Current Version String: | **`a2`** |
| Previous values: | `a1` |
</div>
 
#### Transform Version Number
The Transform ID shall conclude with the `TransformVersionNumber`, which is a string consisting of the letter ‘**v**’ followed by the version number of that transform.

Output, Input, and Look Transforms may import sub-functions or constants that are included in separate Library-type CTL files. Library files usually contain more than one sub-function or constant. If a change is made to a Library file that will change the resulting output from a transform that depends on that Library function, then the version number of the calling transform’s Transform ID shall also be incremented. However, modifications to a Library file that do not affect the numerical output of a calling function (e.g. adding new subfunctions used by other importing files) do not require the Transform ID of a file importing it to be incremented.
 
Any updates to a Transform file that do not change the output of that transform (e.g. whitespace changes, modifications to code comments, etc.) shall not require the transform version number to be incremented.


## User-Name (ACESuserName)
Though designed to be human-readable, ACES Transform IDs can be overly complex for presentation to end users for selection in menus. Therefore, all transforms shall include a user-friendly name, contained in an XML tag of `<ACESuserName>`. Software applications should reference this more user-friendly name for presentation in their user interfaces.
