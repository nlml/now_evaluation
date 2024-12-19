# Updated with Dockerfile and faster parallel processing

This fork makes it easier to setup and run the NoW evaluation script. It also processes your predictions faster thanks to using multiprocessing.

## Setup

First, clone this repo, then build the docker image:

```
docker build -t noweval docker/
```

Download all components of the NoW dataset to a folder like this:

```
/datadir/NoW_Evaluation/dataset
├── imagepathsvalidation.txt
├── NoW_Dataset
├── scans
├── scans_lmks_onlypp
└── test
```

## Evaluating with the Docker image

Make your predictions on the validation set, and put them in e.g. `/path/with/your/predictions`

Now just run:

```
docker run --ipc host --rm -v /datadir/NoW_Evaluation/dataset:/dataset -v /path/with/your/predictions:/preds noweval
```

The results will be placed in `/path/with/your/predictions/results/`

Done!

The original README continues below:

# NoW Evaluation

This is the official repository for evaluation on the [NoW Benchmark](https://now.is.tue.mpg.de). The goal of the NoW benchmark is to introduce a standard evaluation metric to measure the accuracy and robustness of 3D face reconstruction methods from a single image under variations in viewing angle, lighting, and common occlusions. 

<p align="center"> 
<img src="content_now_dataset.png">
</p>

## Evaluation metric

Given a single monocular image, the challenge consists of reconstructing a 3D face. Since the predicted meshes occur in different local coordinate systems, the reconstructed 3D mesh is rigidly aligned (rotation, translation, and scaling) to the scan using a set of corresponding landmarks between the prediction and the scan. We further perform a rigid alignment based on the scan-to-mesh distance (which is the absolute distance between each scan vertex and the closest point in the mesh surface) between the ground truth scan, and the reconstructed mesh using the landmarks alignment as initialization. For more details, see the [NoW Website](https://now.is.tue.mpg.de) or the [RingNet paper](https://ps.is.tuebingen.mpg.de/uploads_file/attachment/attachment/509/paper_camera_ready.pdf).

```
Learning to Regress 3D Face Shape and Expression from an Image without 3D Supervision
Soubhik Sanyal, Timo Bolkart, Haiwen Feng, Michael J. Black
Computer Vision and Pattern Recognition (CVPR) 2019
```

## Clone the repository 
```
git clone https://github.com/soubhiksanyal/now_evaluation.git
```
## Installation

Please install the virtual environment

```
mkdir <your_home_dir>/.virtualenvs
python3 -m venv <your_home_dir>/.virtualenvs/now_evaluation
source <your_home_dir>/.virtualenvs/now_evaluation/bin/activate
```

Make sure your pip version is up-to-date:
```
pip install -U pip
```

Install the requirements by using:

```
pip install -r requirements.txt
```

Install mesh processing libraries from [MPI-IS/mesh](https://github.com/MPI-IS/mesh) within the virtual environment.

### Installing Scan2Mesh distance:

Clone the [flame-fitting](https://github.com/Rubikplayer/flame-fitting) repository and copy the required folders by the following comments

```
git clone https://github.com/Rubikplayer/flame-fitting.git
cp -r flame-fitting/smpl_webuser now_evaluation/smpl_webuser
cp -r flame-fitting/sbody now_evaluation/sbody
```

Clone [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page), checkout v3.4.0, and copy the it to the following folder 

```
git clone https://gitlab.com/libeigen/eigen.git
cd eigen
git checkout 3.4.0
cd ..
cp -r eigen now_evaluation/sbody/alignment/mesh_distance/eigen
```

Edit the file 'now_evaluation/sbody/alignment/mesh_distance/setup.py' to set EIGEN_DIR to the location of Eigen. Then compile the code by following command
```
cd now_evaluation/sbody/alignment/mesh_distance
make
```

The installation of Scan2Mesh is followed by the codebase provided by [flame-fitting](https://github.com/Rubikplayer/flame-fitting).
Please check that repository for more detailed instructions on Scan2Mesh installation.

## Evaluation

Download the NoW Dataset and the validation set scans from the [NoW website](https://now.is.tue.mpg.de/download.php), and predict 3D faces for all validation images.

#### Check data setup

Before running the now evaluation, 

***1) check that the predicted meshes can be successfuly loaded by the used mesh loader by running***
```
python check_predictions.py <predicted_mesh_path>
```
Running this loads the `<predicted_mesh_path>` mesh and exports it to `./predicted_mesh_export.obj`. Please check if this file can be loaded by e.g. [MeshLab](https://www.meshlab.net/) or any other mesh loader, and that the resulting mesh looks like the input mesh.

***2) check that the landmarks for the predicted meshes are correct by running***
```
python check_predictions.py <predicted_mesh_path> <predicted_mesh_landmark_path> <gt_scan_path> <gt_lmk_path> 
```
Running this loads the `<predicted_mesh_path>` mesh, rigidly aligns it with the the scan `<gt_scan_path>`, and outputs the aligned mesh to `./predicted_mesh_aligned.obj`, and the cropped scan to `./cropped_scan.obj`. Please check if the output mesh and scan are rigidly aligned by jointly opening them in e.g. [MeshLab](https://www.meshlab.net/).

#### Error computation

To run the now evaluation on the validation set, run
```
python compute_error.py
```

The function in `metric_computation()` in `compute_error.py` is used to compute the error metric. In the most common setting you can run `python compute_error.py --predicted_mesh_folder <predicted_mesh_folder> --dataset_folder <dataset_folder>`. For more options please see `compute_error.py`

The predicted_mesh_folder should in a similar structure as mentioned in the [RingNet](https://now.is.tue.mpg.de/download.php) website.

Prior to computing the point-to-surface distance, a rigid alignment between each predicted mesh and the scan is computed. The rigid alignment computation requires for each predicted mesh a file with following 7 landmarks:

<p align="center"> 
<img src="landmarks_7_annotated.png" width="50%">
</p>


#### Visualization

Visualization of the reconstruction error is best done with a cumulative error curve. To generate a cumulative error plot, call `generating_cumulative_error_plots()` in the `cumulative_errors.py` with the list of output files and the corresponding list method names. 

**Note that ground truth scans are only provided for the validation set. In order to participate in the NoW challenge, please submit the test set predictions to ringnet@tue.mpg.de as described [here](https://now.is.tue.mpg.de/index.html)**.

#### Known issues

The used [mesh loader](https://github.com/MPI-IS/mesh) is unable to load OBJ files with vertex colors appended to the vertices. I.e. if the OBJ contains lines of the following format `v vx vy vz cr cg cb\n`, export the meshes without vertex colors.

## License

By using the NoW dataset or code, you acknowledge that you have read the [license terms](https://now.is.tue.mpg.de/license.html), understand them, and agree to be bound by them. If you do not agree with these terms and conditions, you must not use the code.

## Citing

This codebase was developed for evaluation of the [RingNet project](https://github.com/soubhiksanyal/RingNet). When using the code or NoW evaluation results in a scientific publication, please cite
```
@inproceedings{RingNet:CVPR:2019,
title = {Learning to Regress 3D Face Shape and Expression from an Image without 3D Supervision},
author = {Sanyal, Soubhik and Bolkart, Timo and Feng, Haiwen and Black, Michael},
booktitle = {Proceedings IEEE Conf. on Computer Vision and Pattern Recognition (CVPR)},
month = jun,
year = {2019},
month_numeric = {6}
}
```




