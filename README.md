# Joint Unsupervised Learning of Optical Flow and Depth by Watching Stereo Videos
This codebase implements the system described in the paper:

UnOS: Unified Unsupervised Optical-flow and Stereo-depth Estimation by Watching Videos

More information can also be found in an earlier version of the paper [Joint Unsupervised Learning of Optical Flow and Depth by Watching Stereo Videos](https://arxiv.org/abs/1810.03654)

Yang Wang, Peng Wang, Zhenheng Yang, Yi Yang, Chenxu Luo and Wei Xu

Please contact Yang Wang (wangyangcharles@gmail.com) if you have any questions.


## Prerequisites
This codebase was developed and tested with Tensorflow 1.2, CUDA 8.0 and Ubuntu 14.04. 

## Acknowledgement
Some of the codes were borrowed from the excellent works of [Tinghui Zhou](https://github.com/tinghuiz/SfMLearner), [Clément Godard](https://github.com/mrharicot/monodepth), [Huangying Zhan](https://github.com/Huangying-Zhan/Depth-VO-Feat), [Chelsea Finn](https://github.com/tensorflow/models/tree/master/research/video_prediction), [Ruoteng Li](https://github.com/liruoteng/OpticalFlowToolkit) and [Martin Kersner](https://github.com/martinkersner/py_img_seg_eval). The files borrowed from [Clément Godard](https://github.com/mrharicot/monodepth) and [Huangying Zhan](https://github.com/Huangying-Zhan/Depth-VO-Feat) are licensed under their original license respectively. 


## Preparing training data
You would need to download all of the [KITTI raw data](http://www.cvlibs.net/datasets/kitti/raw_data.php) and calibration files to train the model. You would also need the training files of [KITTI 2012](http://www.cvlibs.net/datasets/kitti/eval_stereo_flow.php?benchmark=stereo) and [KITTI 2015](http://www.cvlibs.net/datasets/kitti/eval_scene_flow.php?benchmark=stereo) for validating the models.

## Pretrained models
The pretrained models described in the paper can be downloaded [here](https://drive.google.com/open?id=1iOjsq6bePCf8j4InRiMGhMscpNPPQ6TL).

## Training
As described in the paper, the training are organized into three stages sequentially. 

#### Stage 1: only train optical flow
In Stage 1, we only train the PWC-Flow net to learn the optical flow. 
```
python main.py --data_dir=/path/to/your/kitti_raw_data --batch_size=4 --mode=flow --train_test=train  --retrain=True  --train_file=./filenames/kitti_train_files_png_4frames.txt --gt_2012_dir=/path/to/your/kitti_2012_gt --gt_2015_dir=/path/to/your/kitti_2015_gt --trace=/path/to/store-your-model-and-logs
```
After around 200K iterations, you should be able to reach the performance of `Ours(PWC-Only)` described in the paper. You can also download our pretrained model `model-flow`.

#### Stage 2: only train depth and pose
In Stage 2, we train the PWC-Disp and MotionNet to learn the depth and pose.
```
python main.py --data_dir=/path/to/your/kitti_raw_data --batch_size=4  --mode=depth --train_test=train  --retrain=True  --train_file=./filenames/kitti_train_files_png_4frames.txt --gt_2012_dir=/path/to/your/kitti_2012_gt --gt_2015_dir=/path/to/your/kitti_2015_gt --pretrained_model=/path/to/your/pretrained-flow-model-in-stage1  --trace=/path/to/store-your-model-and-logs
```
After around 200K iterations, you sould be able to reach the performance of `Ours(Ego-motion)` described in the paper. You can also download our pretrained model `model-depth`

#### Stage 3: train optical flow, depth, pose and motion segmentation together
In Stage 3, we train everything together.
```
python main.py --data_dir=/path/to/your/kitti_raw_data --batch_size=4  --mode=depthflow --train_test=train  --retrain=True  --train_file=./filenames/kitti_train_files_png_4frames.txt --gt_2012_dir=/path/to/your/kitti_2012_gt --gt_2015_dir=/path/to/your/kitti_2015_gt --pretrained_model=/path/to/your/pretrained-depth-model-in-stage2 --trace=/path/to/store-your-model-and-logs
```
After around 200K iterations, you should be able to reach the performance of `Ours(Full)` described in the paper. You can also download our pretrained model `model-depthflow`

#### Aside: Only train depth using stereo
If you would like to only train depth using the stereo pairs, you can run the following script. This is different from Stage 2 that it only trains PWC-Disp net using stereo pairs. 

```
python main.py --data_dir=/path/to/your/kitti_raw_data --batch_size=4 --mode=stereo --train_test=train  --retrain=True  --train_file=./filenames/kitti_train_files_png_4frames.txt --gt_2012_dir=/path/to/your/kitti_2012_gt --gt_2015_dir=/path/to/your/kitti_2015_gt --trace=/path/to/store-your-model-and-logs
```

After around 100K iterations, you should be able to reach the performance of `Ours(Stereo-only)` described in the paper. You can also download our pretrained model `model-stereo`

#### Notes
- You can specify multiple GPUs training with flag `--num_gpus`
- You can switch to KITTI odometry split by setting `--train_file=./filenames/odo_train_files_png_4frames.txt`
- If you would like to continue to train a model from a previous checkpoint from the same mode, you can set `--retrain=False` 

## Evaluation
The evaluation has already been performed while doing the training. The evaluation results will be printed to the screen.

If you would like to only do a evaluation run, you can set `--train_test=test`.

You can test the pose estimations on sequences 09 and 10 by setting `--eval_pose=09,10` which only works for modes `depth` and `depthflow`.


## Disclaimer
This is the authors' implementation of the system described in the paper and not an official Baidu product.