# Edge-YOLO: anchor-free, embed-friendly, high-performance
## Intro
- In embeded device such as Nvidia Jetson AGX Xavier, Edge-YOLO reaches 34fps with 50.2%AP in COCO2017 dataset and 25.8%AP in VisDrone2019 **(image input size is 640x640, batch=16, including post-process)**
- small object and medium object detect performace is imporved by using RH loss during the last few training epochs.

## Coming Soon
- MNN deployment
- smaller models
- c++ inference code
- mask-edgeyolo for object segmentation task

## Quick Start
### setup

```bash
git clone https://github.com/LSH9832/edgeyolo.git
cd edgeyolo
pip install -r requirements.txt
```
if you use tensorrt, please make sure torch2trt is installed
```
git clone https://github.com/NVIDIA-AI-IOT/torch2trt.git
cd torch2trt
python setup.py install
```

### inference
```
python detect.py --weights edgeyolo_coco.pth --source XXX.mp4 --fp16

# full commands
python detect.py --weights edgeyolo_coco.pth 
                 --source /XX/XXX.mp4     # or dir with images, such as /dataset/coco2017/val2017    (jpg/jpeg, png, bmp, webp is available)
                 --conf-thres 0.25 
                 --nms-thres 0.5 
                 --input-size 640 640 
                 --batch-size 1 
                 --save-dir ./img/coco    # if you press "s", the current frame will be saved in this dir
                 --fp16 
                 --no-fuse                # do not fuse layers
                 --no-label               # do not draw label with class name and confidence
```

### train
- first preparing your dataset and create relative file(./params/dataset/yourdataset.yaml) and change the num of classes in your dataset in file ./params/model/edgeyolo_coco.yaml and add classes.txt to root path of your dataset which contains all the class names(each line writes one class name), make sure your dataset structure as follows:
```
├── dataset_dir
│   ├── annotations
│   │   ├── train.json
│   │   └── val.json
│   ├── train
│   │   ├── xxx.jpg
│   │   └── ...
│   ├── val
│   │   ├── xxx.jpg
│   │   └── ...
│   └── classes.txt
```
- then edit file ./params/train/train_settings.yaml
- finally
```bash
python train.py -f ./params/train/train_settings.yaml
```

### evaluate
```bash
python evaluate.py --weights edgeyolo_coco.pth --dataset /path/to/coco2017 --batch-size 8 --device 0 1 2 3

# full commands
python evaluate.py --weights edgeyolo_coco.pth 
                   --dataset /path/to/coco2017 
                   --val-dir val2017 
                   --anno-file annotations/instances_val2017.json 
                   --batch-size 8 
                   --device 0 1 2 3
                   --class-name classes.txt 
                   --input-size 640 640 
```

### export onnx & tensorrt
```
python pth2onnx.py --weights edgeyolo_coco.pth --simplify

# full commands
python pth2onnx.py --weights edgeyolo_coco.pth 
                   --img-size 640 640 
                   --batch-size 1
                   --opset 11
                   --simplify
```
it generates file **export_output/onnx/edgeyolo_coco_640x640.onnx** and **export_output/onnx/edgeyolo_coco_640x640.yaml**

```
# (workspace: GB)
python onnx2trt.py --onnx export_output/onnx/edgeyolo_coco_640x640.onnx 
                   --yaml export_output/onnx/edgeyolo_coco_640x640.yaml 
                   --workspace 10 
                   --fp16
```
it will generate
- **export_output/tensorrt/edgeyolo_coco_640x640.pt**  for python inference
- **export_output/tensorrt/edgeyolo_coco_640x640.engine**  for c++ inference
- **export_output/tensorrt/edgeyolo_coco_640x640.txt**  for c++ inference

#### for python inference
```
python detect.py --trt --weights export_output/tensorrt/edgeyolo_coco_640x640.pt --source XXX.mp4

# full commands
python detect.py --trt 
                 --weights export_output/tensorrt/edgeyolo_coco_640x640.pt 
                 --source XXX.mp4
                 --legacy         # if "img = img / 255" when you train your train model
                 --use-decoder    # if use original yolox tensorrt model before version 0.3.0
```

#### for c++ inference
it will comming soon


