# MediaSDK-Cpp
MediaSDK-Cpp is a C++ library to handle stitching, editing of media from Insta360 Cameras.

### Supported platforms

| Platform | Version                                       |
| :------- | :-------------------------------------------- |
| Windows  | Windows 7 or later, only x64 supported        |
| MacOS    | 10.8.2 or later, only x64 supported           |
| Linux    | Ubuntu, other distributions need to be tested |

### Supported fileformat

| filetype | import format | export format |
| :------- | :------------ | :------------ |
| Video    | insv   	   | mp4		   |
| Image    | insp   	   | jpg           |

## Table of contents

- [Running test demo](#demo)
- [Get Started](#started)
  - [Note](#note)
  - [Image stitcher ](#ImageStitcher)
  - [Video stitcher](#videoStitcher) 
  	- [Create VideoStitcher](#videoStitcher_create)
    - [Set Parameter](#videoStitcher_param)
    - [Set process callback function](#videoStitcher_process_callback)
    - [Set error callback function](#videoStitcher_error_callback)
    - [Start Stitch](#videoStitcher_StartStitch)
    - [Cancel Stitch](#videoStitcher_cancelStitch)
- [Parameters Guidance](#Guidance)
	- [inputpath](#inputpath)
	- [stitchType](#stitchtype)
	- [HDRType](#hdrType)
	- [EnableFlowState](#flowstate)


## <span id="demo">Running test demo</span>

**On Ubuntu**, follow the steps below to build and run **stitcherSDKDemo**.

```bash
$ gcc main.cc -o stitcherSDKDemo -I/path/to/libmediaSDK -lMediaSDK
$ sudo chmod +x  stitcherSDKDemo
$ export LD_LIBRARY_PATH=LD_LIBRARY_PATH:/path/to/libMediaSDK.so
$ ./stitcherSDKDemo -h
```

## <span id="started">Get Started</span>

### <span id="note">Note</span>

The video stitcher runs asynchronously. Before  **`StartStitch`**, you can call the function **`SetStitchProgressCallback`** to set a callback for receiving the current process during stitching, and call the function **`SetStitchStateCallback`** to set an error callback so that you can get the error info  if an error occurs during stitching. After running **StartStitch** function, you must set a **blocking state** to prevent the program from ending. You can end the blocking state by using the status returned by these two callback functions.  It can be referred to **main.cc**.

The image stitcher runs synchronously.


### <span id="ImageStitcher">Image Stitcher</span>
```c++
#include <stitcher/stitcher.h>
#include <stitcher/common.h>
auto imageStitcher = std::make_shared<ins_media::ImageStitcher>();

//set params
imageStitcher->SetInputPath(input_paths); 
imageStitcher->SetStitchType(stitch_type);
imageStitcher->SetHDRType(hdr_type);
imageStitcher->SetOutputPath(output_path);
imageStitcher->SetOutputSize(output_width, output_height);
imageStitcher->EnableFlowState(bEnableFlowState);

//perform stitch
imageStitcher->Stitch();
```


### <span id="videoStitcher">Video Stitcher</span>
##### <span id="videoStitcher_create">Create VideoStitcher</span>
```c++
#include <stitcher/stitcher.h>
#include <stitcher/common.h>

auto videoStitcher = std::make_shared<ins_media::VideoStitcher>();
```
#####  <span id="videoStitcher_param">Set Parameters</span>

```c++
 videoStitcher->SetInputPath(input_paths);
 videoStitcher->SetOutputPath(output_path);
 videoStitcher->SetStitchType(stitch_type);
 videoStitcher->SetOutputSize(output_width, output_height);
 videoStitcher->SetOutputBitRate(output_bitrate);
 videoStitcher->EnableFlowState(bEnableFlowState);
```

##### <span id="videoStitcher_process_callback">Set process callback function</span>
```c++
void process_callback(long lContext, int process, int error) {
    printf("\rprocess = %d%% ", process);
}

...
    
videoStitcher->SetStitchProgressCallback(process_callback, 0);
```
##### <span id="videoStitcher_error_callback">Set error callback function</span>

```c++
void error_callback(long lContext, int error, const char* errinfo) {
	//you can print the error info 
}
...

videoStitcher->SetStitchStateCallback(error_callback, 0);
```

##### <span id="videoStitcher_StartStitch">Start Stitch</span>

```c++
videoStitcher->StartStitch();
```

##### <span id="videoStitcher_cancelStitch"> Cancel Stitch</span>

```c++
videoStitcher->CancelStitch();
```

## <span id="Guidance">Parameters Guidance</span>

#### <span id="inputpath">InputPaths</span>

***On video***  The maximum size of the vector is 2. For 5.7k separated videos like `/path/to/VID_20190123_113402_00_032.insv` and `/path/to/VID_20190123_113402_10_032.insv`, please input urls of both videos with order like this {"`/path/to/VID_20190123_113402_00_032.insv`","`/path/to/VID_20190123_113402_10_032.insv`"},`xxx_00_xx.insv` means the first eye while `xxx_10_xx.insv` means the second.

***On image***  Generally the size of the vector is 1 , if you enable HDR , there might be multiple paths.

#### <span id = "stitchtype">StitchType</span>
StitchType is an enum.
```c++
enum class STITCH_TYPE {
    TEMPLATE, //very fast but not good stitching effect
    OPTFLOW,//slow but best stitching effect
    DYNAMICSTITCH,//fast and keeps a relatively good stitching effect
};
```
####  <span id="hdrType">HDRType</span>

HDRType is an enum. 

```c++
enum class HDR_TYPE {
    ImageHdr_NONE = -1, //do not perform HDR, this is the default value.
    SingleImageHdr = 0, //perform HDR on a single image
    MultiImageHdr_mbb = 1,//perform HDR with multiple images with a ev gap.
    MultiImageHdr_mpl = 2,//same as MultiImageHdr_mbb, with a different algorithm and a little faster.
};
```

#### <span id="flowstate">EnableFlowState</span>

Bool value indicating whether you want to enable the super flowstate feature of Insta360.

