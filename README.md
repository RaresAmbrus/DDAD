# DDAD - Dense Depth for Autonomous Driving

<a href="https://www.tri.global/" target="_blank">
 <img align="right" src="/media/figs/tri-logo.png" width="20%"/>
</a>

- [DDAD depth challenge](#ddad-depth-challenge)
- [How to Use](#how-to-use)
- [Dataset details](#dataset-details)
- [Dataset stats](#dataset-stats)
- [Sensor placement](#sensor-placement)
- [Evaluation metrics](#evaluation-metrics)
- [IPython notebook](#ipython-notebook)
- [References](#references)
- [Privacy](#privacy)
- [License](#license)

DDAD is a new autonomous driving benchmark from TRI (Toyota Research Institute) for long range (up to 250m) and dense depth estimation in challenging and diverse urban conditions. It contains monocular videos and accurate ground-truth depth (across a full 360 degree field of view) generated from high-density LiDARs mounted on a fleet of self-driving cars operating in a cross-continental setting. DDAD contains scenes from urban settings in the United States (San Francisco, Bay Area, Cambridge, Detroit, Ann Arbor) and Japan (Tokyo, Odaiba).

![](media/figs/ddad_viz.gif)

## DDAD depth challenge

The [DDAD depth challenge](https://eval.ai/web/challenges/challenge-page/902/overview) consists of two tracks: self-supervised and semi-supervised monocular depth estimation. We will evaluate all methods against the ground truth Lidar depth, and we will also compute and report depth metric per semantic class. The winner will be chosen based on the abs_rel metric. The winners of the challenge will receive cash prizes and will present their work at the CVPR 2021 Workshop [“Frontiers of Monocular 3D Perception”](https://sites.google.com/view/mono3d-workshop). Please check below for details on the [DDAD dataset](#dataset-details), [notebook](ipython-notebook) for loading the data and a description of the [evaluation metrics](#evaluation-metrics).

## How to Use

The data can be downloaded here: [train+val](https://tri-ml-public.s3.amazonaws.com/github/DDAD/datasets/DDAD.tar) (257 GB, md5 checksum: `c0da97967f76da80f86d6f97d0d98904`) and test (coming soon). To load the dataset, please use the [TRI Dataset Governance Policy (DGP) codebase](https://github.com/TRI-ML/dgp). The following snippet will instantiate the dataset:

```python
from dgp.datasets import SynchronizedSceneDataset

# Load synchronized pairs of camera and lidar frames.
dataset =
SynchronizedSceneDataset('<path_to_dataset>/ddad.json',
    datum_names=('lidar', 'CAMERA_01', 'CAMERA_05'),
    generate_depth_from_datum='lidar',
    split='train'
    )

# Iterate through the dataset.
for sample in dataset:
  # Each sample contains a list of the requested datums.
  lidar, camera_01, camera_05 = sample[0:3]
  point_cloud = lidar['point_cloud'] # Nx3 numpy.ndarray
  image_01 = camera_01['rgb']  # PIL.Image
  depth_01 = camera_01['depth'] # (H,W) numpy.ndarray, generated from 'lidar'
```

The [DGP](https://github.com/TRI-ML/dgp) codebase provides a number of functions that allow loading one or multiple camera images, projecting the lidar point cloud into the camera images, intrinsics and extrinsics support, etc. Additionally, please refer to the [Packnet-SfM](https://github.com/TRI-ML/packnet-sfm) codebase (in PyTorch) for more details on how to integrate and use DDAD for depth estimation training/inference/evaluation and state-of-the-art pretrained models. 

## Dataset details

DDAD includes high-resolution, long-range [Luminar-H2](https://www.luminartech.com/technology) as the LiDAR sensors used to generate pointclouds, with a maximum range of 250m and sub-1cm range precision. Additionally, it contains six calibrated cameras time-synchronized at 10 Hz, that together produce a 360 degree coverage around the vehicle. The six cameras are 2.4MP (1936 x 1216), global-shutter, and oriented at 60 degree intervals. They are synchronized with 10 Hz scans from our Luminar-H2 sensors oriented at 90 degree intervals (datum names: `camera_01`, `camera_05`, `camera_06`, `camera_07`, `camera_08` and `camera_09`) - the camera intrinsics can be accessed with `datum['intrinsics']`. The data from the Luminar sensors is aggregated into a 360 point cloud covering the scene (datum name: `lidar`). Each sensor has associated extrinsics mapping it to a common vehicle frame of reference (`datum['extrinsics']`).

The training and validation scenes are 5 or 10 seconds long and consist of 50 or 100 samples with corresponding Luminar-H2 pointcloud and six image frames including intrinsic and extrinsic calibration. The training set contains 150 scenes with a total of 12650 individual samples (75900 RGB images), and the validation set contains 50 scenes with a total of 3950 samples (23700 RGB images).


<p float="left">
  <img src="/media/figs/pano1.png" width="32%" />
  <img src="/media/figs/pano2.png" width="32%" />
  <img src="/media/figs/pano3.png" width="32%" />
</p>
<img src="/media/figs/odaiba_viz_rgb.jpg" width="96%">
<img src="/media/figs/hq_viz_rgb.jpg" width="96%">
<img src="/media/figs/ann_viz_rgb.jpg" width="96%">

## Dataset stats

### Training split

| Location      | Num Scenes (50 frames)     |  Num Scenes (100 frames)  | Total frames |
| ------------- |:-------------:|:-------------:|:-------------:|
| SF            | 0  |  19 | 1900 |
| ANN           | 23  | 53 | 6450 |
| DET           |  8  | 0  | 400 |
| Japan         | 16  | 31  | 3900 |

Total: `150 scenes` and `12650 frames`.

### Validation split

| Location      | Num Scenes (50 frames)     |  Num Scenes (100 frames)  | Total frames |
| ------------- |:-------------:|:-------------:|:-------------:|
| SF            | 1  |  10 | 1050 |
| ANN           | 11  | 14 | 1950 |
| Japan         | 9  | 5  | 950 |

Total: `50 scenes` and `3950 frames`.

USA locations: ANN - Ann Arbor, MI; SF - San Francisco Bay Area, CA; DET - Detroit, MI; CAM - Cambridge, Massachusetts. Japan locations: Tokyo and Odaiba.

### Test split

The test split consists of 3080 images with associated intrinsic calibration. The data can be downloaded from [here](https://tri-ml-public.s3.amazonaws.com/github/DDAD/datasets/DDAD_test.tar). 200 images from the test split have associated panoptic labels, similar to the DDAD validation split. The ground truth depth and panoptic labels will not be made public. To evaluate your method on the DDAD test split, please submit your results to the [DDAD depth challenge](https://eval.ai/web/challenges/challenge-page/902/overview), as a single zip file with the same file name convention as the test split (i.e. 000000.png ... 003079.png). Each entry in the zip file should correspond to the DDAD test split image with the same name, and it should be a 16bit single channel PNG image. Each prediction can be either at full image resolution or downsampled. If the resolution of the predicted depth is different from that of the input image, the evaluation script will upsample the predicted depth to the input image resolution using nearest neighbor interpolation.

## Sensor placement

The figure below shows the placement of the DDAD LiDARs and cameras. Please note that both LiDAR and camera sensors are positioned so as to provide 360 degree coverage around the vehicle. The data from all sensors is time synchronized and reported at a frequency of 10 Hz. The data from the Luminar sensors is reported as a single point cloud in the vehicle frame of reference with origin on the ground below the center of the vehicle rear axle, as shown below. For instructions on visualizing the camera images and the point clouds please refer to this [IPython notebook](media/notebooks/DDAD.ipynb).

![](media/figs/ddad_sensors.png)

## Evaluation metrics

Please refer to the the [Packnet-SfM](https://github.com/TRI-ML/packnet-sfm) codebase for instructions on how to compute detailed depth evaluation metrics.

## IPython notebook

The associated [IPython notebook](notebooks/DDAD.ipynb) provides a detailed description of how to instantiate the dataset with various options, including loading frames with context, visualizing rgb and depth images for various cameras, and displaying the lidar point cloud.

[![](media/figs/notebook.png)](notebooks/DDAD.ipynb)

## References

Please use the following citation when referencing DDAD:

#### 3D Packing for Self-Supervised Monocular Depth Estimation (CVPR 2020 oral)
*Vitor Guizilini, Rares Ambrus, Sudeep Pillai, Allan Raventos and Adrien Gaidon*, [**[paper]**](https://arxiv.org/abs/1905.02693), [**[video]**](https://www.youtube.com/watch?v=b62iDkLgGSI)
```
@inproceedings{packnet,
  author = {Vitor Guizilini and Rares Ambrus and Sudeep Pillai and Allan Raventos and Adrien Gaidon},
  title = {3D Packing for Self-Supervised Monocular Depth Estimation},
  booktitle = {IEEE Conference on Computer Vision and Pattern Recognition (CVPR)},
  primaryClass = {cs.CV}
  year = {2020},
}
```


## Privacy

To ensure privacy the DDAD dataset has been anonymized (license plate and face blurring) using state-of-the-art object detectors.


## License

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.
