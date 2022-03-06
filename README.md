### YOLOv5-Pytorch-TensorRT

### 1 Training in Pytorch
### 1.1 Download YOLOv5-Pytorch
```
cd ~/Desktop
mkdir workspace
cd workspace
git clone https://github.com/ultralytics/yolov5
```

### 1.2 Prepare dataset
```
cd ~/Desktop/workspace
mkdir dataset
cd dataset
mkdir annotations
mkdir images
mkdir ImageSets
cd ImageSets
mkdir Main
```

### 1.3 Place your images and annotations into images and annotations folder respectively
### 1.4 Put the ```split_train_val.py``` and ```voc2yolo_label.py```into dataset folder
### 1.5 Change ```voc2yolo_label.py``` line 8-13 to your config
```
classes = ["obstacle", "human", "injury"]   # 改成自己的类别
image_dir = "/home/laitathei/Desktop/workspace/dataset/images/"
labels_dir = "/home/laitathei/Desktop/workspace/dataset/labels/"
annotations_dir = "/home/laitathei/Desktop/workspace/dataset/annotations/"
ImageSets_Main_dir = "/home/laitathei/Desktop/workspace/dataset/ImageSets/Main/"
dataset_dir = "/home/laitathei/Desktop/workspace/dataset/"
```

### 1.5 Dataset folder structures
```
├── dataset
   ├── annotations
   │   ├── xxx.xml
   │   
   ├── ImageSets
   │   ├── Main
   │       ├── test.txt
   │       ├── train.txt
   │       ├── trainval.txt
   │       ├── val.txt
   │   
   ├── labels
   │   ├── test
   │       ├── xxx.txt
   |       ├──   ...
   │   ├── val 
   │       ├── xxx.txt
   |       ├──   ...
   │   ├── train
   │       ├── xxx.txt
   |       ├──   ...
   │   
   ├── images
   │   ├── test
   │       ├── xxx.jpg
   |       ├──   ...
   │   ├── val 
   │       ├── xxx.jpg
   |       ├──   ...
   │   ├── train
   │       ├── xxx.jpg
   |       ├──   ...
   ├── train.txt
   ├── train_dummy.txt
   ├── test.txt
   ├── test_dummy.txt
   ├── valid.txt
   ├── valid_dummy.txt
   ├── split_train_val.py
   └── voc2yolo_label.py
```

### 1.6 Prepare your dataset yaml file, download the official weight from https://github.com/ultralytics/yolov5/releases/tag/v6.1 and place it into yolov5/weight folder
```
cd ~/Desktop/workspace/yolov5
mkdir weight
cd weight
wget https://github.com/ultralytics/yolov5/releases/download/v6.1/yolov5n.pt
cd ~/Desktop/workspace/yolov5/data
gedit custom_dataset.yaml
```

### 1.7 custom_dataset.yaml content and change your configuration such as number of class and class names
```
train: /home/laitathei/Desktop/workspace/dataset/train.txt
val: /home/laitathei/Desktop/workspace/dataset/val.txt
test: /home/laitathei/Desktop/workspace/dataset/test.txt
test_xml: /home/laitathei/Desktop/workspace/dataset/annotations

# Classes
nc: 3  # number of classes
names: ['obstacle','human', 'injury']  # class names
```

### 1.8 Train your dataset
```
cd ~/Desktop/workspace/yolov5/
python3 train.py --weights /home/laitathei/Desktop/workspace/yolov5/weight/yolov5n.pt --cfg /home/laitathei/Desktop/workspace/yolov5/models/yolov5n.yaml --data /home/laitathei/Desktop/workspace/yolov5/data/custom_dataset.yaml --epochs 100
```

### 1.9 Train result, best.pt and last.pt are FP32 format
```
100 epochs completed in 0.091 hours.
Optimizer stripped from runs/train/exp/weights/last.pt, 3.9MB
Optimizer stripped from runs/train/exp/weights/best.pt, 3.9MB

Validating runs/train/exp/weights/best.pt...
Fusing layers... 
Model Summary: 213 layers, 1763224 parameters, 0 gradients, 4.2 GFLOPs
               Class     Images     Labels          P          R     mAP@.5 mAP@.5:.95: 100%|██████████| 2/2 [00:00<00:00,  3.06it/s]                                                                      
                 all         55        215      0.986      0.977      0.994      0.916
            obstacle         55        215      0.986      0.977      0.994      0.916
Results saved to runs/train/exp
```

### 2 TensorRT conversion
### 2.1 Convert ```best.pt```->```best.wts```->```best.engine``` in TensorRTx
### 2.2 Put the ```gen_wts.py``` into yolov5 folder
### 2.3 Convert ```best.wts```->```best.engine``` by ```gen_wts.py```
```
cd ~/Desktop/workspace/yolov5
python3 gen_wts.py -w ./runs/train/exp/weights/best.pt -o ./runs/train/exp/weights/best.wts

## Below content will show if program success
YOLOv5 🚀 v6.1-11-g63ddb6f torch 1.10.2+cu113 CPU
```

### 2.4 Convert ```best.wts```->```best.engine``` by TensorRTx cmake and best.engine is FP16 format
```
cd ~/Desktop/workspace/tensorrtx/yolov5
mkdir build
cd build

## Place the best.wts into build folder
## update CLASS_NUM in yololayer.h if your model is trained on custom dataset
## before 
static constexpr int CLASS_NUM = 80; // line 20
static constexpr int INPUT_H = 640;  // line 21  yolov5's input height and width must be divisible by 32.
static constexpr int INPUT_W = 640;  // line 22
## after 
static constexpr int CLASS_NUM = 3; // line 20
static constexpr int INPUT_H = 640; // line 21  yolov5's input height and width must be divisible by 32.
static constexpr int INPUT_W = 640; // line 22

cmake ..
make

## Below content will show if program success
[100%] Linking CXX executable yolov5
[100%] Built target yolov5

sudo ./yolov5 -s best.wts best.engine n
## Below content will show if program success
Loading weights: best.wts
Building engine, please wait for a while...
Build engine successfully!
```

### 2.5 Inference the engine with ROS and realsensen D455 camera
### 2.6 Put ```inference_ros_trt_v2.py``` into tensorrtx/yolov5 folder
```
cd ~/Desktop/workspace/tensorrtx/yolov5
python3 inference_ros_trt_v2.py

## change the realsense camera topic, PLUGIN_LIBRARY, and engine_file_path if required
```
