# VapourSynth-Retinex

Retinex.dll | 2014 | mawen1250

VapourSynth plugin

namespace: retinex

functions: MSRCP

## About Retinex

The Retinex theory and algorithm mainly aims at simulating the color constancy feature of HVS(Human Visual System).

The light perceived by visual receptors can be separated into illuminance and reflectance. Retinex estimates the illuminance and derive the reflectance from the light, the filtered result of which is an image represents the reflectance characteristics of the scene, regardless of the illuminance.

Retinex is a very powerful filter in dynamic range compression, contrast enhancement, color constancy, etc.

## MSRCP

### Description

MSR(Multi Scale Retinex) is the most successful implementation of Retinex, based on center/surround theory.

MSRCP(Multi Scale Retinex with Chromaticity Preservation) is based on MSR. It applies MSR on intensity image(Y plane), and adjust UV/RGB guided by Y to preserve chromaticity.

As MSRCP preserves chromaticity, it is excellent for dynamic range compression and contrast enhancement, while it doesn't eliminate color cast. To implement the full color constancy feature of Retinex, it is recommended to use MSRCR(Multi Scale Retinex with Color Restoration) instead.

This function accept 8-16bit integer Gray/YUV/RGB/YCoCg input. Sub-sampled format is not supported. If you want to process YUV420/YUV422 clip, convert it to YUV444 or RGB first.

### Usage

```python
retinex.MSRCP(clip input, float[] sigmaS=[25,80,250], float lower_thr=0, float upper_thr=0, bool fulls, bool fulld=fulls)
```

- input:<br />
    clip to process

- sigma: (Default: [25,80,250])<br />
    sigma of Gaussian function to apply Gaussian filtering.<br />
    Assign an array of multiple sigma to apply MSR.<br />
    It is recommended by most researches to use 3 different scales (balance between speed and quality).

- lower_thr: (Default: 0)<br />
    After applying MSR, a normalization is applied to the value range [Floor,Ceil] of the image.<br />
    This parameter define the ratio of pixel number to determine the Floor of value range. 0 means using the minimum pixel value in the image.<br />
    Valid range is [0,1), and the sum of lower_thr and upper_thr should be less than 1.

- upper_thr: (Default: 0)<br />
    After applying MSR, a normalization is applied to the value range [Floor,Ceil] of the image.<br />
    This parameter define the ratio of pixel number to determine the Ceil of value range. 0 means using the maximum pixel value in the image.<br />
    Valid range is [0,1), and the sum of lower_thr and upper_thr should be less than 1.<br />
    Increase it if there are some extreme bright parts in the output image which makes the whole image too dark.

- fulls: (Default: True for RGB/YCoCg input, False for YUV/Gray input)<br />
    Determine the value range of input clip. True means full range/PC range, and False means limited range/TV range.
    
- fulld: (Default: fulls)<br />
    Determine the value range of output clip. True means full range/PC range, and False means limited range/TV range.<br />
    Set different value for fulls and fulld will result in range conversion.<br />
    When fulls and fulld are the same, it is safe to assign either True or False for any kinds of input clip except YUV. When fulls=False it will automatically determine the Floor and Ceil of input image, which may produce a stronger filter result under some circumstances.

### Example

TV range YUV420P8 input, filter in TV range YUV444P16

```python
v = core.fmtc.resample(v, csp=vs.YUV444P16)
v = core.retinex.MSRCP(v)
```

JPEG image(PC range YUV420P8 with MPEG-1 chroma placement) input, filter in PC range RGB48

```python
i = core.lsmas.LWLibavSource(r'Image.jpg')
i = core.fmtc.resample(i, csp=vs.YUV444P16, fulls=True, cplace="MPEG1")
i = core.fmtc.matrix(i, mat="601", fulls=True, csp=vs.RGB48)
i = core.retinex.MSRCP(i)
```
