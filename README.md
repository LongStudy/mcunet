# MCUNet: Tiny Deep Learning on IoT Devices 

###  [website](http://mcunet.mit.edu/) | [paper](https://arxiv.org/abs/2007.10319) | [demo](https://www.youtube.com/watch?v=YvioBgtec4U&feature=emb_logo)


## News

- **(2022/06/14)** We refactor the MCUNet repo as a standalone repo (previous repo: https://github.com/mit-han-lab/tinyml)
- Our projects are covered by: [MIT News](https://news.mit.edu/2020/iot-deep-learning-1113), [WIRED](https://www.wired.com/story/ai-algorithms-slimming-fit-fridge/), [Morning Brew](https://www.morningbrew.com/emerging-tech/stories/2020/12/07/researchers-figured-fit-ai-ever-onto-internet-things-microchips), [Stacey on IoT](https://staceyoniot.com/researchers-take-a-3-pronged-approach-to-edge-ai/), [Analytics Insight](https://www.analyticsinsight.net/amalgamating-ml-and-iot-in-smart-home-devices/), [Techable](https://techable.jp/archives/142462), etc.


## Overview

Microcontrollers are low-cost, low-power hardware. They are widely deployed and have wide applications.

![teaser](assets/figures/applications.png)

But the tight memory budget (50,000x smaller than GPUs) makes deep learning deployment difficult.

![teaser](assets/figures/memory_size.png)

MCUNet is a **system-algorithm co-design** framework for tiny deep learning on microcontrollers. It consists of **TinyNAS** and **TinyEngine**. They are co-designed to fit the tight memory budgets.

With system-algorithm co-design, we can significantly improve the deep learning performance on the same tiny memory budget.

![teaser](assets/figures/overview.png)

Our **TinyEngine** inference engine could be a useful infrastructure for MCU-based AI applications. It significantly **improves the inference speed and reduces the memory usage** compared to existing libraries like [TF-Lite Micro](https://www.tensorflow.org/lite/microcontrollers), [CMSIS-NN](https://arxiv.org/abs/1801.06601), [MicroTVM](https://tvm.apache.org/2020/06/04/tinyml-how-tvm-is-taming-tiny), etc. It improves the inference speed by **1.5-3x**, and reduces the peak memory by **2.7-4.8x**.

![teaser](assets/figures/latency_mem.png)



## Model Zoo

We provide the model zoo for ImageNet and Visual Wake Words (VWW). 

### Usage

You can build the pre-trained PyTorch `fp32` model or the `int8` quantized model in TF-Lite format.

```python
from mcunet.model_zoo import model_id_list, build_model, download_tflite
print(model_id_list)  # the list of models in the model zoo

# pytorch fp32 model
model, image_size, description = build_model(model_id="mcunet-320kb-in", pretrained=True)  # you can replace model_id with any other option from model_id_list

# download tflite file to tflite_path
tflite_path = download_tflite(model_id="mcunet-320kb-in")
```


### Evaluate

To evaluate the accuracy of PyTorch `fp32` models, run:

```bash
python eval_imagenet.py --net_id mcunet-320kb-in --data-dir PATH/TO/IMAGENET/val
```

To evaluate the accuracy of TF-Lite `int8` models, run:

```bash
python eval_tflite.py \
    --data-dir PATH/TO/IMAGENET/val \
    --tflite_path assets/tflite/mcunet-320kb-1mb_imagenet.tflite
```

### Model List

- Note that all the **latency**, **SRAM**, and **Flash** usage are profiled with **TinyEngine**.
- Here we only provide the `int8` quantized modes. `int4` quantized models (as shown in the paper) can further push the accuracy-memory trade-off, but lacking a general format support.

The ImageNet model list:

| net_id                                          | Model Stats.                                  | TF-Lite Stats.                | TinyEngine Stats.             | Top-1 Acc.                 | Top-5 Acc.                 | Link                                                         |
| ----------------------------------------------- | --------------------------------------------- | ----------------------------- | ----------------------------- | -------------------------- | -------------------------- | ------------------------------------------------------------ |
| Proxyless-s <br />for TF-Lite<br />(w0.25-r112) | MACs: 10.7M <br />Param: 0.57M<br />Act: 98kB | SRAM: 288kB<br />Flash: 860kB | SRAM: 114kB<br />Flash: 701kB | FP: 44.9%<br />int8: 43.8% | FP: 70.0%<br />int8: 69.0% | [json](assets/configs/proxyless-w0.25-r112_imagenet.json)<br />[ckpt](https://hanlab.mit.edu/projects/tinyml/mcunet/release/proxyless-w0.25-r112_imagenet.pth)<br />[tflite]( |



### ImageNet models

#### Comparison on STM32F746 (constraints: 320kB SRAM, 1MB Flash)

We first compare the baseline networks (scaled ProxylessNASMobile, <u>Proxyless-s</u>) with MCUNet on STM32F746. The baseline networks are scaled to fit the hardware constraints (we compoundly scale the width and resolution of the baseline networks and report the best accuracy under the constraints).

The memory usage is measured on STM32F746 board. TinyEngine can reduce the Flash and SRAM usage and fit a larger model; TinyNAS can design network that has superior accuracy under the same memory budget.

| Model                                             | Model Stats.                                   | TF-Lite Stats.                                    | TinyEngine Stats.                                  | Top-1 Acc.                  | Top-5 Acc.                 | Link                                                         |
| ------------------------------------------------- | ---------------------------------------------- | ------------------------------------------------- | -------------------------------------------------- | --------------------------- | -------------------------- | ------------------------------------------------------------ |
| Proxyless-s <br />for TF-Lite<br />(w0.25-r112)   | MACs: 10.7M <br />Param: 0.57M<br />Act: 98kB  | SRAM: 288kB<br />Flash: 860kB | SRAM: 114kB<br />Flash: 701kB  | FP: 44.9%<br />int8: 43.8%  | FP: 70.0%<br />int8: 69.0% | [json](assets/configs/proxyless-w0.25-r112_imagenet.json)<br />[ckpt](https://hanlab.mit.edu/projects/tinyml/mcunet/release/proxyless-w0.25-r112_imagenet.pth)<br />[tflite](https://hanlab.mit.edu/projects/tinyml/mcunet/release/proxyless-w0.25-r112_imagenet.tflite) |
| Proxyless-s <br />for TinyEngine<br />(w0.3-r176) | MACs: 38.3M <br />Param: 0.75M<br />Act: 242kB | SRAM: 526kB<br />Flash:  1063kB | SRAM: 292kB<br />Flash: 892kB  | FP: 57.0%<br />int8:  56.2% | FP: 80.2%<br />int8: 79.7% | [json](assets/configs/proxyless-w0.3-r176_imagenet.json)<br />[ckpt](https://hanlab.mit.edu/projects/tinyml/mcunet/release/proxyless-w0.3-r176_imagenet.pth)<br />[tflite](https://hanlab.mit.edu/projects/tinyml/mcunet/release/proxyless-w0.3-r176_imagenet.tflite) |
|                                                   |                                                |                                                   |                                                    |                             |                            |                                                              |
| TinyNAS<br />for TinyEngine<br />(**MCUNet**)     | MACs: 81.8M <br />Param: 0.74M<br />Act: 333kB | SRAM: 560kB<br />Flash:  1088kB | SRAM: 293kB<br />Flash: 897kB | FP: 62.2%<br />int8:  61.8% | FP: 84.5%<br />int8: 84.2% | [json](assets/configs/mcunet-320kb-1mb_imagenet.json)<br />[ckpt](https://hanlab.mit.edu/projects/tinyml/mcunet/release/mcunet-320kb-1mb_imagenet.pth)<br />[tflite](https://hanlab.mit.edu/projects/tinyml/mcunet/release/mcunet-320kb-1mb_imagenet.tflite) |

#### MCUNet under different memory constraints

We provide the MCUNet models under different memory constraints:

- STM32 F412 (Cortex-M4, 256kB SRAM/1MB Flash)
- STM32 F746 (Cortex-M7, 320kB SRAM/1MB Flash)
- STM32 H743 (Cortex-M7, 512kB SRAM/2MB Flash)

The memory usage is measured on the corresponding devices.

##### 1. Int8 Models

Int8 quantization is the most widely used quantization and default setting in our experiments.

| Constraints    | Model Stats.                                   | TF-Lite Stats.                                    | TinyEngine Stats.                                  | Top-1 Acc.                  | Top-5 Acc.                 | Link                                                         |
| -------------- | ---------------------------------------------- | ------------------------------------------------- | -------------------------------------------------- | --------------------------- | -------------------------- | ------------------------------------------------------------ |
| 256kB<br />1MB | MACs: 67.3M <br />Param: 0.73M<br />Act: 325kB | SRAM: 546kB<br />Flash:  1081kB | SRAM: 242kB<br />Flash: 878kB | FP: 60.9%<br />int8: 60.3%  | FP: 83.3%<br />int8: 82.6% | [json](assets/configs/mcunet-256kb-1mb_imagenet.json)<br />[ckpt](https://hanlab.mit.edu/projects/tinyml/mcunet/release/mcunet-256kb-1mb_imagenet.pth)<br />[tflite](https://hanlab.mit.edu/projects/tinyml/mcunet/release/mcunet-256kb-1mb_imagenet.tflite) |
| 320kB<br />1MB | MACs: 81.8M <br />Param: 0.74M<br />Act: 333kB | SRAM: 560kB<br />Flash:  1088kB | SRAM: 293kB<br />Flash: 897kB | FP: 62.2%<br />int8:  61.8% | FP: 84.5%<br />int8: 84.2% | [json](assets/configs/mcunet-320kb-1mb_imagenet.json)<br />[ckpt](https://hanlab.mit.edu/projects/tinyml/mcunet/release/mcunet-320kb-1mb_imagenet.pth)<br />[tflite](https://hanlab.mit.edu/projects/tinyml/mcunet/release/mcunet-320kb-1mb_imagenet.tflite) |
| 512kB<br />2MB | MACs: 125.9M <br />Param: 1.7M<br />Act: 413kB | SRAM: 863kB<br />Flash:  2133k | SRAM: 456kB<br />Flash: 1876kB | FP: 68.4%<br />int8: 68.0%  | FP: 88.4%<br />int8: 88.1% | [json](assets/configs/mcunet-512kb-2mb_imagenet.json)<br />[ckpt](https://hanlab.mit.edu/projects/tinyml/mcunet/release/mcunet-512kb-2mb_imagenet.pth)<br />[tflite](https://hanlab.mit.edu/projects/tinyml/mcunet/release/mcunet-512kb-2mb_imagenet.tflite) |

##### 

* We can further reduce the memory usage with lower precision like `int4` (as shonw in the paper). However, `int4` quantization also does NOT bring further speed gain compared to `int8` due to the instruction set.\

## Requirement

- Python 3.6+

- PyTorch 1.4.0+

- Tensorflow 1.15 (if you want to test TF-Lite models; CPU support only)

## Acknowledgement

We thank [MIT Satori cluster](https://mit-satori.github.io/) for providing the computation resource. We thank MIT-IBM Watson AI Lab, SONY, Qualcomm, NSF CAREER Award #1943349 and NSF RAPID Award #2027266 for supporting this research.

Part of the code is taken from [once-for-all](https://github.com/mit-han-lab/once-for-all) project for development.


## Citation
If you find the project helpful, please consider citing our paper:

```
@article{lin2020mcunet,
  title={Mcunet: Tiny deep learning on iot devices},
  author={Lin, Ji and Chen, Wei-Ming and Lin, Yujun and Gan, Chuang and Han, Song},
  journal={Advances in Neural Information Processing Systems},
  volume={33},
  year={2020}
}

@inproceedings{
  lin2021mcunetv2,
  title={MCUNetV2: Memory-Efficient Patch-based Inference for Tiny Deep Learning},
  author={Lin, Ji and Chen, Wei-Ming and Cai, Han and Gan, Chuang and Han, Song},
  booktitle={Annual Conference on Neural Information Processing Systems (NeurIPS)},
  year={2021}
} 
```


## Related Projects

[TinyTL: Reduce Memory, Not Parameters for Efficient On-Device Learning](https://arxiv.org/abs/2007.11622) (NeurIPS'20)

[Once for All: Train One Network and Specialize it for Efficient Deployment](https://arxiv.org/abs/1908.09791) (ICLR'20)

[ProxylessNAS: Direct Neural Architecture Search on Target Task and Hardware](https://arxiv.org/pdf/1812.00332.pdf) (ICLR'19)

[AutoML for Architecting Efficient and Specialized Neural Networks](https://ieeexplore.ieee.org/abstract/document/8897011) (IEEE Micro)

[AMC: AutoML for Model Compression and Acceleration on Mobile Devices](https://arxiv.org/pdf/1802.03494.pdf) (ECCV'18)

[HAQ: Hardware-Aware Automated Quantization](https://arxiv.org/pdf/1811.08886.pdf)  (CVPR'19, oral)