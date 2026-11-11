---
title: Reference Gamut Compression Specification
---

<!-- SPDX-License-Identifier: CC-BY-4.0 -->
<!-- Copyright Contributors to the ACES Documentation -->


Reference Gamut Compression (RGC) Specification
================


Scope
----------------
This document introduces a Reference Gamut Compression (RGC) operator, published in ACES 1.3, which may be applied to ACES image data to “heal” pixel values outside the AP1 gamut. The Reference Gamut Compression algorithm is intended to replace the Blue Light Artifact LMT, which is now deprecated.



Introduction
----------------
A common complaint from users of ACES has been the artifacts resulting from out of gamut values in source images. These artifacts are most known for appearing in highly saturated, bright LED light sources such as police car lights, stoplights, etc - but also appear frequently in LED sources used to light scenes. In an ACES workflow, these artifacts appear at two stages - first in the conversion from camera raw RGB via an Input Transform (IDT) into ACES AP0 - and second in the conversion from ACES AP0 into ACES AP1 (ACEScg and ACEScct). These out of gamut pixel values are problematic when their negative components cause issues in compositing, and may also produce visual artifacts when viewed through an ACES Output Transform.

A Look Modification Transform (LMT) referred to as the [blue light artifact fix](https://github.com/ampas/aces-core/blob/v1.3/transforms/ctl/lmt/LMT.Academy.BlueLightArtifactFix.ctl) was created as a temporary solution, but this affected all pixels in the image, rather than just the problem areas. A new solution was needed which preserved colors within a “zone of trust”, only altering the most saturated values.



Specification
----------------
ACEScg values within the AP1 gamut are positive. Values outside of the AP1 gamut are negative in one or two of their components.

The gamut compression algorithm runs per-pixel.

ACES RGB values shall be converted to gamut compressed ACES RGB values using the following steps.

### Step 1 – Convert ACES 2065-1 RGB to ACEScg RGB

ACES 2065-1 RGB values shall be converted to ACEScg RGB values using the transformation matrix ($TRA_1$) calculated and applied using the methods provided in Section 4 of SMPTE RP 177:1993.
!!! note
    Equation 1 shows the relationship between ACES R, G, and B values and ACEScg RGB values. NPM, rounded to 10 decimal places, is derived using the color space chromaticity coordinates specified in SMPTE S 2065-1, Academy Specification S-2014-004, and the methods provided in Section 3.3 of SMPTE RP 177:1993.

$$
\begin{bmatrix}R_{ACEScg}\\G_{ACEScg}\\B_{ACEScg}\end{bmatrix}=TRA_1\cdot \begin{bmatrix}R_{ACES}\\G_{ACES}\\B_{ACES}\end{bmatrix}
$$

$$
TRA_1=\begin{bmatrix}\phantom{-}1.4514393161 & -0.2365107469 & -0.2149285693 \\-0.0765537734 & \phantom{-}1.1762296998 & -0.0996759264\\\phantom{-}0.0083161484 & -0.0060324498 & \phantom{-}0.9977163014 \\\end{bmatrix}
$$

$$
TRA_1 = {NPM_{AP1}}^{-1}\cdot NPM_{AP0}
$$

<div align="center"><i>Equation 1</i></div>

### Step 2 – Calculate *A*
Calculate $A$ using Equation 2.

$$
A = \text{MAX}(R_{ACEScg}, G_{ACEScg}, B_{ACEScg})
$$

<div style="padding-left: 90px;" markdown="1">   
	where: <br>
	$\textrm{MAX()}$ returns the maximum of the arguments
</div>

<div align="center"><i>Equation 2</i></div>

!!! note
    *A* is used as representative of the achromatic value of the pixel.

### Step 3 – Calculate *d<sub>n</sub>*

Calculate $d_n$ using Equation 3.

$$
d_n = \begin{cases}\frac{A - RGB_{ACEScg}}{\text{ABS} \left ( A \right )},& \text{if } A\neq0\\0,& \text{otherwise}\end{cases}
$$


<div style="padding-left: 90px;" markdown="1"> 
	where :
	$\textrm{ABS()}$ returns the absolute value of the argument
</div>

<div align="center"><i>Equation 3</i></div>

!!! note
    $d_n$ represents the normalized distances of the channels from the achromatic, or “inverse RGB ratios”.

### Step 4 – Calculate *d<sub>c</sub>*

Compress $d_n$ to produce $d_c$  using Equation 4.

$$
d_c = \begin{cases}d_n,& \text{if }d_n < t\\t + \frac{d_n-t}{\left ( 1+d_p \right ) ^{\frac{1}{p}}}\end{cases}
$$

<div style="padding-left: 90px;" markdown="1"> 
	where :
</div>

$$
l = \begin{bmatrix}1.147 \\ 1.264 \\ 1.312\end{bmatrix}\ t = \begin{bmatrix}0.815 \\ 0.803 \\ 0.880\end{bmatrix}\ p = 1.2
$$

$$
s = \frac{l - t}{\left (\left (\frac{1 - t}{l - t} \right )^{-p} - 1\right )^\frac{1}{p}}
$$

$$
d_p = \left ( \frac{d_n - t}{s} \right )^{p}
$$

<div align="center"><i>Equation 4</i></div>

!!! note
    The compression function compresses values at limit ($l$) to 1.0. This results in values beyond this limit remaining outside AP1 after compression. A function which compressed all values to within AP1 would require a significant proportion of the space between threshold ($t$) and 1.0 to be reserved for values which are highly unlikely to occur, thus compressing other values more than necessary. Limit ($l$) values have been chosen so that the encoding gamuts of all digital cinema cameras with official ACES IDTs (ARRI, RED, Canon, Sony, Panasonic) will compress to within AP1. Threshold ($t$) values have been derived from the boundary of the ACEScg values for the ColorChecker Classic 24 (as specified in ISO 17321-1). The exponent ($p$) has been set to an average value, chosen following user testing, since there is no objective measure of the correctness of this value.

### Step 5 – Calculate *RGB<sub>c</sub>*

Calculate $RGB_c$  using Equation 5.

$$
RGB_c = A - d_c \times \text{ABS}\left ( A \right )
$$

<div style="padding-left: 90px;" markdown="1"> 
	where :
	$\textrm{ABS()}$ returns the absolute value of the argument
</div>

<div align="center"><i>Equation 5</i></div>

### Step 6 – Convert *RGB<sub>c</sub>* to ACES2065-1 RGB

Gamut compressed ACEScg RGB values ($RGB_c$) shall be converted to ACES2065-1 RGB values using the transformation matrix ($TRA_2$) calculated and applied using the methods provided in Section 4 of SMPTE RP 177:1993.

!!! note
	Equation 6 shows the relationship between ACEScg R, G, and B values and ACES2065-1 RGB values. NPM, rounded to 10 decimal places, is derived using the color space chromaticity coordinates specified in SMPTE S 2065-1, Academy Specification S-2014-004, and the methods provided in Section 3.3 of SMPTE RP 177.

$$
\begin{bmatrix}R_{ACES}\\G_{ACES}\\B_{ACES}\end{bmatrix}=TRA_2\cdot \begin{bmatrix}R_c\\G_c\\B_c\end{bmatrix}
$$

$$
TRA_2=\begin{bmatrix} \phantom{-}0.6954522414 & \phantom{-}0.1406786965 & \phantom{-}0.1638690622 \\ \phantom{-}0.0447945634 & \phantom{-}0.8596711185 & \phantom{-}0.0955343182 \\ -0.0055258826 & \phantom{-}0.0040252103 & \phantom{-}1.0015006723 \\\end{bmatrix}
$$

$$
TRA_2 = {NPM_{AP0}}^{-1} \cdot NPM_{AP1}
$$

<div align="center"><i>Equation 6</i></div>

### Gamut Decompression

While the Reference Gamut Compression has a closed form inverse, its use is not normally recommended.

Gamut compressed ACES RGB values may be converted back to the original ACES RGB values using the same steps as for compression above, but using Equation 4b in place of Equation 4.

$$
d_c = \begin{cases}t + s \times \left ( \frac{d_p}{1-d_p} \right ) ^{\frac{1}{p}},& \text{if }t \leq d_n \leq t + s\\d_n,& \text{otherwise}\end{cases}
$$

<div align="center"><i>Equation 4b</i></div>

!!! note
    $l$, $t$, $p$, $s$ and $d_p$ are as defined in Equation 4.



Tracking
----------------

The Reference Gamut Compression is defined as a Look Transform (LMT) in CTL and has the following ACES Transform ID:

```
<ACEStransformID>urn:ampas:aces:transformId:v2.0:Look.Academy.ReferenceGamutCompress.a2.v1</ACEStransformID>
```

This is trackable via a `lookTransform` element in an AMF file. If the RCG is used in the viewing pipeline, the `lookTransform` will be listed in the associated AMF. If the AMF is accompanying rendered media, the applied flag should be used to track whether or not the RGC has been “baked in”.

If using the RGC in a viewing pipeline, this `lookTransform` should appear directly after the IDT, first in the list of any LMTs, to make sure other operations benefit from the gamut compression. 

The Transform ID should be included in any exported AMFs, with the applied flag set as appropriate, and the description set to the ACESuserName to enable proper tracking. Currently, only the Reference (i.e. static) Gamut Compression is trackable via AMF.



Appendix A - History / Research
----------------

The Architecture Virtual Working Group, chaired by Carol Payne (Netflix) and Matthias Scharfenberg (ILM), to investigate gamut mapping in ACES began its work in January 2020, with a proposal outlining the main issue as:


<div style="padding-left: 30px;" markdown> 
    Users of ACES are experiencing problems with out of gamut colors and the resulting artifacts (loss of texture, intensification of color fringes). This issue occurs at two stages in the pipeline. 

 * Conversion from camera raw RGB or from the manufacturer’s encoding space into ACES AP0 
 * Conversion from ACES AP0 into the working color space ACES AP1
</div>


It was acknowledged early on in the group that this artifacting can also occur in VFX/Color grading, as well as the Output Transform stages in the pipeline.

The working group chairs set the scope:

 * Propose transforms between color spaces that avoid or reduce color clipping. Solutions for this may include: 
 * Proposing a suitable color encoding space for digital motion-picture cameras.
 * Proposing a suitable working color space.
 * Propose a suitable gamut mapping/compression algorithm that performs well with wide gamut, high dynamic range, scene referred content that is robust and invertible.

The group started out investigating the working and encoding spaces (ACES 2065-1 and ACEScg). However, it was agreed early on that although the possibility of creating a new ACES working space which mitigates common gamut issues should not be discounted, it would require a very strong case as to the benefits. Changing a core component of ACES would potentially introduce backwards compatibility issues, and would also be based only on the situation at the current time. It raised the possibility of having to change the working space repeatedly in future. Thus, the focus moved on to the third option - a suitable algorithm to solve the artifacting while maintaining as much of the current ACES standards and structure as possible. The gamut mapping approach chosen is one of compression. It deals with the ACES image data “as is”, and simply strives to convert that into less problematic image data. 

Based on the history above, the general working assumptions were:

* Samples are relative scene exposure values (i.e. scene-referred linear data) with no assumed min/max value range boundaries
* The gamut mapping operator is per-pixel only (i.e. not spatial or temporal)

The stated ideals for a gamut compression algorithm were:

* Exposure invariance — $f(a \cdot RGB) = a \cdot f(RGB)$
* Source gamut agnosticism
* Monotonicity
* Simplicity – suited to a fast shader implementation
* Invertibility (see caveats in Appendix II)
* Colors in a “zone of trust” will be left unaltered

While a suitable algorithm should be able to map arbitrary gamut A into arbitrary gamut B, it should not be a requirement that all source data must be contained within gamut A. Nor is it necessarily a requirement that the output should be entirely bounded by gamut B. Indeed, allowing extreme source values to be mapped to output values close to, but not within, the target gamut means that the compression function does not need to tend to the horizontal at the boundary. This means that its inverse will not tend to the vertical, which is beneficial for invertibility.

Because the unreal colors which occur are a result of the mismatch between a camera and human observer (among other causes) and are outliers in the residual error of a transform optimized for a subset of important “memory” colors, what they “should” look like is somewhat undefined. The important thing is to remap them into values which are plausible rather than “accurate”.

What was determined to be outside the scope:

* Colorimetric accuracy or spectral plausibility of input device transforms (IDTs) 
* Display gamut mapping. (Required modifications to the RRT/ODT will need to be addressed by a subsequent group.)
* Customizing for specific input/output gamuts
* Working in bounded or volume-based gamuts
* Actions which could limit creative choices further down the line (e.g. excessive desaturation)

### User Testing

Once the working group settled on the baseline algorithm and its properties a set of targeted, small scale user tests were conducted to ensure the foundations of the work were solid. The testing was composed of two groups - VFX compositors and colorists. Between these two disciplines every major use case for the gamut compression algorithm could be tested and measured. The group gathered an open repository of test images that clearly exhibited the problem to be solved. It then derived a set of test scenarios for each group ranging from keying, blur, grain matching, hue adjustment, and more. The tests were conducted in Nuke and Resolve, on both SDR and HDR monitors. 

<figure markdown>
![User Testing Footage Examples](../guides/rgc-implementation/images/rgc-collage.png)
  <figcaption>User Testing Footage Examples</figcaption>
</figure>

Overall, the results of the user testing were positive and uncovered no major issues in the algorithm functionality. 75% of compositors and 96% of colorists stated that using the algorithm helped them complete their work and achieve their creative goals. For full user testing results, please refer to the [working group historical repository](https://github.com/ampas/aces-vwg-gamut-mapping-2020).



Appendix B  – Implementation Considerations
----------------

### Invertibility

Invertibility of the transformation is an aspect that was discussed at length. The consensus was that while an inverse transform should be defined it comes with the caveat that gamut expansion can create undesirable results when used with highly saturated pixel values, such as those added as part of graphics or CG rendered imagery As this operation is considered more of a “pixel healing” technical operation, inversion should not be a required part of the workflow. It is more akin to a despill after pulling a key, or a bit of sharpening on a scale operation.

### 3D LUT Approximation

If there is an unavoidable requirement to implement the Reference Gamut Compression on legacy systems using a 3D lookup table (LUT) special consideration must be given to the shaper function. Common functions for implementing ACES transforms as 3D LUTs are the Log2 shaper or ACEScct. However neither of those are designed to cover the negative value range required to map the negative components of out of gamut colors into a normalized 0 to 1 domain. For a successful 3D LUT implementation the normalizing shaper function must also cover a significant range of negative values.

Due to the residual error a 3D LUT approximation of the transform should be considered to be non-invertible.

Systems used as part of the finishing pipeline should use a mathematical implementation of the RGC, rather than a LUT. But for preview purposes, a well constructed LUT may be acceptable, particularly if it is a concatenation of all the required transforms, such as an on-set preview LUT transforming camera log to Rec.709.

### Parametric Version

Some implementations may also choose to offer a parametric variation of the RGC. This is not officially endorsed as part of ACES, and should be treated simply as another grading operator. It should not be used as a replacement for the RGC since its parameters cannot be tracked by AMF and must instead be stored in the project files of the implementing application.

Suggested parameter names, and default values for a parametric version are given in Section 9 of the [RGC Implementation guide](../guides/rgc-implementation/index.md#parametric-version-implementation-specifications).



Appendix C: Illustrations
----------------

<figure markdown>
![Distance Limit (CIExy Chromaticity Plot)](./images/rgc-distance.jpg){ width="480" }
  <figcaption>Distance Limit (CIExy Chromaticity Plot)</figcaption>
</figure>

<figure markdown>
![Gamut Compression](./images/rgc-cc24.jpg){ width="480" }
  <figcaption>Compression Threshold and ColorChecker 24 Patches (CIExy Chromaticity Plot)</figcaption>
</figure>

<figure markdown>
![Before Gamut Compression (ACES Rec. 709 Output Transform)](./images/rgc-before.jpg)
  <figcaption>Before Gamut Compression (ACES Rec. 709 Output Transform)</figcaption>
</figure>

<figure markdown>
![Gamut Compression](./images/rgc-compression.gif){ width="640" }
  <figcaption>Gamut Compression</figcaption>
</figure>

Note that the red value has also been moved slightly, because although it was not negative, its normalized distance was close to 1.0, and beyond the protected threshold for that channel (0.815).

<figure markdown>
![After Gamut Compression (ACES Rec. 709 Output Transform)](./images/rgc-after.jpg)
  <figcaption>After Gamut Compression (ACES Rec. 709 Output Transform)</figcaption>
</figure>

<figure markdown>
![Gamut Compression Polar Chromaticity Grid](./images/rgc-polar-plot.gif){ width="450" }
  <figcaption>Gamut Compression Polar Chromaticity Grid</figcaption>
</figure>



References
----------------
The following standards, specifications, articles, presentations, and texts are referenced in this text:

* [ACES Gamut Mapping Architecture VWG - Technical Documentation Deliverable](https://paper.dropbox.com/doc/tZHiuOCj0RdYw8PPkrTam)
* [ISO 17321-1:2012 - Colour characterisation of digital still cameras (DSCs) - Part 1: Stimuli, metrology and test procedures](https://www.iso.org/standard/56537.html)
* [RP 177:1993 - SMPTE Recommended Practice - Derivation of Basic Television Color Equations](https://doi.org/10.5594/SMPTE.RP177.1993)
* [S-2014-004: ACEScg — A Working Space for CGI Render and Compositing](../../encodings/acescg.md)




<!-- Include section numbering -->
<style>
    @import "../../stylesheets/sections.css"
</style>