# PyTorch_YOLOv4-tiny

This project is based on [WongKinYiu/PyTorch_YOLOv4 u3_preview branch](https://github.com/WongKinYiu/PyTorch_YOLOv4/tree/u3_preview) with some modification

- You can use this project to train 416x416 YOLOv4-tiny.
[GitHub Issues](https://github.com/WongKinYiu/ScaledYOLOv4/issues/41)

- View our experiment environment infos [here](https://github.com/e96031413/PyTorch_YOLOv4-tiny/blob/main/experiment-info.md)

### 02/08更新：
新增detect.py計算FPS功能 (detect.py的第8行、138~140行、171行)
```
# line 8
FPS_avg = []

# line 138~140
from statistics import mean 
print('%sDone. (FPS:%.1f)' % (s, 1/(t2 - t1)))
FPS_avg.append(1/(t2-t1))

# line 171
print('Avg FPS: (%.1f)' % (mean(FPS_avg)))
```

解決test.py執行時遇到numpy必須要1.17版本及No module named 'pycocotools'的方法:
```bash
#移除所有numpy
pip uninstall numpy
#安裝1.17版本numpy
pip install numpy==1.17
#安裝pycocotools
pip install pycocotools
#執行test.py
python test.py --data coco2017.data --cfg yolov4-tiny.cfg --weights yolov4-tiny.pt --img 416 --iou-thr 0.7 --batch-size 8
```

### 01/10更新：
新增支援yolov4.conv.137的Pre-trained weight功能(在model.py第456~456行)
```
    elif file == 'yolov4.conv.137':
        cutoff = 137
```

### 12/29更新：
新增YOLOv4-tiny的RouteGroup功能

[Feature-request: YOLOv4-tiny #1350](https://github.com/ultralytics/yolov3/issues/1350#issuecomment-651602149)

新增ReLU6(在utils/layers.py)、ReLU(在utils/layers.py)、DepthWise Convolution(在models.py)、ShuffleNetUnit(在models.py)

**如何透過[u5版本](https://github.com/WongKinYiu/PyTorch_YOLOv4/tree/u5)的yaml檔案進行backbone修改？**

[Yolov4 with Efficientnet b0-b7 Backbone](https://shihyung1221.medium.com/yolov4-with-efficientnet-b0-b7-backbone-529d0ce67cf0)

[Yolov4 with MobileNet V2/V3 Backbone](https://shihyung1221.medium.com/yolov4-with-mobilenet-v2-v3-backbone-c18c0f10bc29)

[Yolov4 with Resnext50/ SE-Resnet50 Backbones](https://shihyung1221.medium.com/yolov4-with-resnext50-se-resnet50-backbones-c324242c48f4)

[Yolov4 with Resnet Backbone](https://shihyung1221.medium.com/yolov4-with-resnet-backbone-eb141b6e79ca)

[Yolov4 with VGG Backbone](https://shihyung1221.medium.com/yolov4-with-vgg-backbone-ae0cedab4f0f)

### 12/28更新：
新增以下四篇Paper的程式碼(在models.py、utils/layers.py)

SE Block paper : [arxiv.org/abs/1709.01507](arxiv.org/abs/1709.01507)

CBAM Block paper : [arxiv.org/abs/1807.06521](arxiv.org/abs/1807.06521)

ECA Block paper : [arxiv.org/abs/1910.03151](arxiv.org/abs/1910.03151)

Funnel Activation for Visual Recognition : [arxiv.org/abs/2007.11824](arxiv.org/abs/2007.11824)


### 12/1更新：
修改了PyTorch_YOLOv4的u3_preview當中，models.py第355行(支援pre-trained weight)、train.py第67行(支援32倍數的解析度)、dataset.py第262及267行(處理dataset相對路徑的問題)
models.py line 355
```python
def load_darknet_weights(self, weights, cutoff=-1):
    # Parses and loads the weights stored in 'weights'
    # Establish cutoffs (load layers between 0 and cutoff. if cutoff = -1 all are loaded)
    file = Path(weights).name
    if file == 'darknet53.conv.74':
        cutoff = 75
    elif file == 'yolov3-tiny.conv.15':
        cutoff = 15
    elif file == 'yolov4-tiny.conv.29':
        cutoff = 29
	.
	.
	.
	.
```
train.py line 67
```python
    gs = 32  # (pixels) grid size # 原本gs = 64改成32
    assert math.fmod(imgsz_min, gs) == 0, '--img-size %g must be a %g-multiple' % (imgsz_min, gs)
```
utils/dataset.py  line 262 and 267
```python
class LoadImagesAndLabels(Dataset):  # for training/testing
    def __init__(self, path, img_size=416, batch_size=16, augment=False, hyp=None, rect=False, image_weights=False,
                 cache_images=False, single_cls=False):
        path = str(Path(path))  # os-agnostic
        parent = str(Path(path).parent) + os.sep                              # add this
        assert os.path.isfile(path), 'File not found %s. See %s' % (path, help_url)
        with open(path, 'r') as f:
            self.img_files = [x.replace('/', os.sep) for x in f.read().splitlines()  # os-agnostic
                              if os.path.splitext(x)[-1].lower() in img_formats]
            self.img_files = [x.replace('./', parent) if x.startswith('./') else x for x in self.img_files]    # add this
```
透過[u3_preview](https://github.com/WongKinYiu/PyTorch_YOLOv4/tree/u3_preview?rgh-link-date=2020-11-24T04%3A40%3A32Z)版本經由以下指令訓練300個Epochs並進行testing得到的AP結果：
```
CUDA_VISIBLE_DEVICES=0 python train.py --data coco2017.data --cfg yolov4-tiny.cfg --weights 'yolov4-tiny.conv.29' --name yolov4-tiny --img 416 416 416
python test.py --data coco2017.data --cfg yolov4-tiny.cfg --weights weights/best.pt --img 416 --batch-size 8
```
darknet版本測出是40.2
```
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.221
 Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets=100 ] = 0.394
 Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets=100 ] = 0.219
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.077
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.263
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.340
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=  1 ] = 0.217
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 10 ] = 0.368
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.400
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.151
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.484
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.612
```

## Requirements

```
pip install -r requirements.txt
```
※ For running Mish models, please install https://github.com/thomasbrandon/mish-cuda

## Training

```
CUDA_VISIBLE_DEVICES=0 python train.py --data coco2017.data --cfg yolov4-tiny.cfg --weights 'yolov4-tiny.conv.29' --name yolov4-tiny --img 416
```

## Testing

```
CUDA_VISIBLE_DEVICES=0 python test.py --data coco2017.data --cfg yolov4-tiny.cfg --weights yolov4-tiny.pt --img 416 --augment
```

## Citation

```
@article{bochkovskiy2020yolov4,
  title={{YOLOv4}: Optimal Speed and Accuracy of Object Detection},
  author={Bochkovskiy, Alexey and Wang, Chien-Yao and Liao, Hong-Yuan Mark},
  journal={arXiv preprint arXiv:2004.10934},
  year={2020}
}
```

```
@inproceedings{wang2020cspnet,
  title={{CSPNet}: A New Backbone That Can Enhance Learning Capability of {CNN}},
  author={Wang, Chien-Yao and Mark Liao, Hong-Yuan and Wu, Yueh-Hua and Chen, Ping-Yang and Hsieh, Jun-Wei and Yeh, I-Hau},
  booktitle={Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition Workshops},
  pages={390--391},
  year={2020}
}
```

## Acknowledgements

* [https://github.com/AlexeyAB/darknet](https://github.com/AlexeyAB/darknet)
* [https://github.com/ultralytics/yolov3](https://github.com/ultralytics/yolov3)
* [https://github.com/ultralytics/yolov5](https://github.com/ultralytics/yolov5)
* [HaloTrouvaille/YOLO-Multi-Backbones-Attention](https://github.com/HaloTrouvaille/YOLO-Multi-Backbones-Attention/tree/1f425d379783b6d132b44f14ecfd251d8e2448fa)
* [SpursLipu/YOLOv3v4-ModelCompression-MultidatasetTraining-Multibackbone](https://github.com/SpursLipu/YOLOv3v4-ModelCompression-MultidatasetTraining-Multibackbone)
