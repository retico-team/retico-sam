# retico-sam
A ReTico module for SAM. See below for more information on the models.

As suggested in [SAM](https://github.com/facebookresearch/segment-anything) it is highly recommended to use SAM with a GPU, but CPU only is possible, it will just take much longer to run.

## Installation and Requirements

To use the SAM modules you first need to install the retico-sam package:
* Install retico-sam:
```pip install retico-sam```

  or, for the latest development version from GitHub:
```pip install git+https://github.com/retico-team/retico-sam.git```

This installs [SAM](https://github.com/facebookresearch/segment-anything), and FastSAM through [Ultralytics](https://docs.ultralytics.com/models/fast-sam/).

You should also download a checkpoint for whichever model version you would like to use. The default model for SAM is 'vit_h', which can be found [here](https://dl.fbaipublicfiles.com/segment_anything/sam_vit_h_4b8939.pth). To use it you must include the path to the checkpoint when creating the SAM module, as demonstrated in the example. If you would like to use a different model you must download the corresponding checkpoint and pass the matching model name.

For FastSAM, pass the name of the FastSAM weights, `"FastSAM-s.pt"` or `"FastSAM-x.pt"`, which Ultralytics downloads automatically, or the path to your own checkpoint.

## Modules

### `SAMModule` and `FastSAMModule`
Subscribe to `ImageIU` and output `DetectedObjectsIU` with the detected objects as bounding boxes or segmentation masks. `SAMModule` is imported from `retico_sam` and `FastSAMModule` from `retico_sam.fastsam`.

**Model Options:** `'h' (vit_h)`, `'l' (vit_l)`, `'b' (vit_b)`

#### Arguments
* `model` (str): model type, defaults to `'h'`; only used by `SAMModule`.
NOTE: the user must match the model type with the corresponding checkpoint
* `path_to_chkpnt` (str): the path to the model checkpoint, no default.
* `use_bbox` (bool): output bounding boxes; defaults to `False`
* `use_seg` (bool): output segmentation masks; defaults to `False`

Exactly one of `use_bbox` and `use_seg` must be set to `True`.

### Hugging Face `SAMModule`
The same as `SAMModule`, but loads SAM from Hugging Face, so no checkpoint has to be downloaded. It is imported from `retico_sam.hfsam`.

#### Arguments
* `model` (str): the Hugging Face model, defaults to `facebook/sam-vit-base`
* `show` (bool): currently unused; defaults to `False`
* `use_bbox` (bool): output bounding boxes; defaults to `False`
* `use_seg` (bool): output segmentation masks; defaults to `False`

### Example
**Note:** The example uses `WebcamModule` and `ExtractObjectsModule` from [retico-vision](https://github.com/retico-team/retico-vision) and `DebugModule` from `retico_core`. The Extract Objects module allows the user to choose how many detected objects to extract
* `num_obj_to_display` (int): the number of detected objects to extract

```python
import os

os.environ['PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION'] = 'python'

from retico_core.debug import DebugModule
from retico_vision.vision import WebcamModule, ExtractObjectsModule
from retico_sam import SAMModule
## OR for FastSAM
# from retico_sam.fastsam import FastSAMModule
## OR for Hugging Face SAM
# from retico_sam.hfsam import SAMModule

path_var = "sam_vit_h_4b8939.pth"  # replace with the path to your checkpoint

webcam = WebcamModule()
sam = SAMModule(model='h', path_to_chkpnt=path_var, use_bbox=True)
# sam = FastSAMModule(path_to_chkpnt="FastSAM-x.pt", use_bbox=True)  # if fastsam
# sam = SAMModule(use_bbox=True)  # if hfsam
extractor = ExtractObjectsModule(num_obj_to_display=20)
debug = DebugModule()

webcam.subscribe(sam)
sam.subscribe(extractor)
extractor.subscribe(debug)

webcam.run()
sam.run()
extractor.run()
debug.run()

print("Network is running")
input()

webcam.stop()
sam.stop()
extractor.stop()
debug.stop()
```
