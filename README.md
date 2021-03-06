# Centripetal-SGD

Update: Pytorch implementation released. Fixed a bug of Pytorch implementation in csgd/csgd_prune.py, which was related to pruning the last conv layer which is followed by an FC layer. This bug only resulted in an error if the last-layer feature maps were flattened as input to FC. For models with Global Average Pooling like ResNet-56, this bug was harmless. The critical codes for C-SGD training (csgd/csgd_train.py) and pruning (csgd/csgd_prune.py) have been refactored and cleaned, such that the readability has significantly improved. The Tensorflow codes also work, but I would not suggest you read them. A little trick: using smaller centripetal strength on the scaling factor of BN improves the performance in some of the cases.

This repository contains the codes for the following CVPR-2019 paper 

[Centripetal SGD for Pruning Very Deep Convolutional Networks with Complicated Structure](http://openaccess.thecvf.com/content_CVPR_2019/html/Ding_Centripetal_SGD_for_Pruning_Very_Deep_Convolutional_Networks_With_Complicated_CVPR_2019_paper.html).

This demo will show you how to globally slim ResNet-56 and DenseNet-40 on CIFAR-10 in PyTorch and Tensorflow.

Citation:

	@inproceedings{ding2019centripetal,
  		title={Centripetal sgd for pruning very deep convolutional networks with complicated structure},
  		author={Ding, Xiaohan and Ding, Guiguang and Guo, Yuchen and Han, Jungong},
  		booktitle={Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition},
  		pages={4943--4953},
  		year={2019}
	}

## Introduction

Filter pruning, a.k.a. network slimming or channel pruning, aims to remove some filters from a CNN so as to slim it with acceptable performance drop. We seek to make some filters increasingly close and eventually identical for network slimming. To this end, we propose Centripetal SGD (C-SGD), a novel optimization method, which can train several filters to collapse into a single point in the parameter hyperspace. When the training is completed, the removal of the identical filters can trim the network with NO performance
loss, thus no finetuning is needed. By doing so, we have partly solved an open problem of constrained filter pruning on CNNs with complicated structure, where some layers must be pruned following others.


## PyTorch Example Usage

We iteratively train a ResNet-56 (with 16-32-64 channels) and slim it into 13/16, 11/16 and 5/8 of the original width.

1. Install PyTorch. Clone this repo and enter the directory. Modify PYTHONPATH or you will get an ImportError.
```
export PYTHONPATH='WHERE_YOU_CLONED_THIS_REPO'
```

2. Modify 'CIFAR10_PATH' in dataset.py to the directory of your CIFAR-10 dataset. If the dataset is not found in that directory, it will be automatically downloaded.

3. Train the original ResNet-56 and iteratively slim it.
```
python csgd/csgd_rc56.py
```

4. Check the shape of pruned weights and the validation accuracy.
```
python display_hdf5.py csgd_exps/rc56_slim_5-8_csgd/itr0/pruned.hdf5
python display_hdf5.py csgd_exps/rc56_slim_5-8_csgd/itr1/pruned.hdf5
python display_hdf5.py csgd_exps/rc56_slim_5-8_csgd/itr2/pruned.hdf5
cat csgd_exps/rc56_slim_5-8_csgd/itr2/log.txt
```


## Tensorflow Example Usage

1. Install Tensorflow-gpu-1.11

2. Prepare the CIFAR-10 dataset in tfrecord format. Please follow https://github.com/tensorflow/models/blob/master/research/slim/datasets/download_and_convert_cifar10.py, download the CIFAR-10 dataset, convert it to tfrecord format, rename the two output files as train.tfrecords and validation.tfrecords, and modify the value of DATA_PATH in tf_dataset.py.

3. Prune a DenseNet-40 to 3 filters per layer based on the magnitude of kernels and finetune it. Then evaluate the model.

```
python csgd_standalone.py magnitude1
python csgd_standalone.py eval magnitude1_trained.hdf5
```

4. Train a DenseNet-40 using C-SGD and trim it to obtain the same final structure. Then evaluate the model.

```
python csgd_standalone.py csgd1
python csgd_standalone.py eval dc40_csgd1_itr0_prunedweights.hdf5
```

## Contact
dxh17@mails.tsinghua.edu.cn

Google Scholar Profile: https://scholar.google.com/citations?user=CIjw0KoAAAAJ&hl=en

My open-sourced papers and repos: 

**State-of-the-art** channel pruning (preprint, 2020): [Lossless CNN Channel Pruning via Gradient Resetting and Convolutional Re-parameterization](https://arxiv.org/abs/2007.03260) (https://github.com/DingXiaoH/ResRep)

CNN component (ICCV 2019): [ACNet: Strengthening the Kernel Skeletons for Powerful CNN via Asymmetric Convolution Blocks](http://openaccess.thecvf.com/content_ICCV_2019/papers/Ding_ACNet_Strengthening_the_Kernel_Skeletons_for_Powerful_CNN_via_Asymmetric_ICCV_2019_paper.pdf) (https://github.com/DingXiaoH/ACNet)

Channel pruning (CVPR 2019): [Centripetal SGD for Pruning Very Deep Convolutional Networks with Complicated Structure](http://openaccess.thecvf.com/content_CVPR_2019/html/Ding_Centripetal_SGD_for_Pruning_Very_Deep_Convolutional_Networks_With_Complicated_CVPR_2019_paper.html) (https://github.com/DingXiaoH/Centripetal-SGD)

Channel pruning (ICML 2019): [Approximated Oracle Filter Pruning for Destructive CNN Width Optimization](http://proceedings.mlr.press/v97/ding19a.html) (https://github.com/DingXiaoH/AOFP)

Unstructured pruning (NeurIPS 2019): [Global Sparse Momentum SGD for Pruning Very Deep Neural Networks](http://papers.nips.cc/paper/8867-global-sparse-momentum-sgd-for-pruning-very-deep-neural-networks.pdf) (https://github.com/DingXiaoH/GSM-SGD)
