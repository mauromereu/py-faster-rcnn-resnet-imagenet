# Faster RCNN with ResNet for ImageNet Detection

Based on Faster RCNN, the repository aims to reproduce the ImageNet Detection results in ResNet paper (Deep Residual Learning for Image Recognition).

So far, it achieves mAP 52.2% (vs. 60.5% in the paper) on val2 of ImageNet 2015 Detection dataset *without* the use of Box refinement, Global context and Multi-scale testing.

## What has been done
  * Add ImageNet Detection imdb and roidb
  * Use ResNet-101
  * Multi-gpus training and testing
  * Fix some problems when training Faster RCNN over ImageNet
  
## What has NOT been done
  - Box refinement
  - Global context 
  - Multi-scale testing

If you are interested in these, please feel free to submit PRs.

## How to use this code
* Clone this repository and set up the environment as described in the original [README.md](README_Faster_RCNN.md) of Faster RCNN
* Download the models I trained. Populate them into `output/faster_rcnn_alt_opt/imagenet_2015_trainval1_woextra`.

  [resnet-101_rpn_stage1_iter_320000.caffemodel](https://drive.google.com/open?id=0B7c5Ix-XO7hqNXRjSUNta0wxcmc)

  [resnet-101_fast_rcnn_stage1_iter_320000.caffemodel](https://drive.google.com/open?id=0B7c5Ix-XO7hqVkRyMXdvQ29sNUE)

* Generate RPN proposals
  
  `python tools/rpn_generate.py --def models/imagenet/ResNet-101/faster_rcnn_alt_opt/rpn_test.pt --net output/faster_rcnn_alt_opt/imagenet_2015_trainval1_woextra/resnet-101_rpn_stage1_iter_320000.caffemodel --imdb imagenet_2015_val2 --gpu 0 1 2 3`

* Use Fast RCNN to detect objects with these proposals generated by RPN
  
  `./tools/test_net.py --gpu 0 1 2 3 --def models/imagenet/ResNet-101/fast_rcnn/test.prototxt --net output/faster_rcnn_alt_opt/imagenet_2015_trainval1_woextra/resnet-101_fast_rcnn_stage1_iter_320000.caffemodel --imdb imagenet_2015_val2 --cfg experiments/cfgs/fast_rcnn.yml --rpn_file output/default/imagenet_2015_val2/resnet-101_rpn_stage1_iter_320000/proposals_test`


Once it finishes, you could see the final mAP over val2.

## How to train it by yourself
**Hardware required:** 4 or 8 GPU cards are required to train the model because the batch sizes used in the original paper are 8 for RPN and 16 for Fast RCNN.

**Please note:** because the improvement of stage 3 and 4 is minior, only the first two stages are trained. With 8 Titanx GPU cards, it takes about four days to train.

* Download ImageNet Detection 2015 datasets, and unzip it into directory `data`. It should have this basic structure
  ```Shell
  ImageNet2015/Annotations
  ImageNet2015/Data
  ImageNet2015/ImageSets
  # others...
  ```

* Run `cp trainval1_woextra.txt val2.txt data/ImageNet2015/ImageSets/DET`
* Download the pretranied model of ResNet-101 [here](https://onedrive.live.com/?authkey=%21AAFW2-FVoxeVRck&id=4006CBB8476FF777%2117887&cid=4006CBB8476FF777), and populate it into `data/imagenet_models` with name `ResNet-101.caffemodel`
* Run `experiments/scripts/faster_rcnn_alt_opt.sh 0,1,2,3,4,5,6,7 ResNet-101 imagenet` to train. `0,1,2,3,4,5,6,7` is GPU ids. Please replace it if you use other GPUs.
