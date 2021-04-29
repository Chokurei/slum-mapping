# Slum Mapping

## 4. Get Started
### 4.1 Data
#### 4.1.1 Vector Ground Truth (ArcGIS)
Find here: [OneDrive](https://1drv.ms/u/s!ApTa4c0QeLyMgQK06-V_VmxDQQpB?e=L3YzNo)

Created by ArcQIS, saved with extension xx.kmz. Take Shenzhen.kmz as an example in following contents.
#### 4.1.2 Raster Ground Truth (QGIS)
**Add Vector Layer**
```
Layer -> Add Layer-> Add Vector Layer
Source: path/to/Shenzhen.kmz
Select Vector Layers to Add: Geometry type: Polygon
```
<img width="1524" alt="Screen Shot 2021-04-24 at 23 40 10" src="https://user-images.githubusercontent.com/16301109/115962546-e0aaea00-a556-11eb-9455-960b92d676c1.png">

Save shapefile to "./src/ori/cities/Shenzhen/shapefile/shenzhen.shp"

**Add Google Satellite Layer**

Find [Google Maps URL](https://mt1.google.com/vt/lyrs%3Ds%26x%3D%7Bx%7D%26y%3D%7By%7D%26z%3D%7Bz%7D)
```
Plugins -> Tile+ -> Tile+
```
<img width="461" alt="Screen Shot 2021-04-29 at 21 27 14" src="https://user-images.githubusercontent.com/16301109/116550475-c8114a00-a931-11eb-81a0-afc5bee26496.png">

Add Google Satellite by URL, zoom level information [here](https://blog.csdn.net/mengdong_zy/article/details/73949818).
```
Browser -> XYZ Tiles -> New Connection -> xxx
```
<img width="912" alt="Screen Shot 2021-04-29 at 21 32 50" src="https://user-images.githubusercontent.com/16301109/116551333-cc8a3280-a932-11eb-8407-a6e1c3d54ce7.png">


**Extract Google Satellite Imagery**

Extract target Google Satellite Imagery based on related shapefile
```
XYZ Tiles：Google Satellite -> Export layer -> To file 
```
Save to "./src/ori/cities/Shenzhen/dataset/2.0/image_full/Shenzhen.tif"
```
no Create VRT
Calculate from Layer -> Shenzhen (as example)
Resolution (current: userdefined) -> 2 (as example)
```
<img width="574" alt="Screen Shot 2021-04-29 at 21 52 58" src="https://user-images.githubusercontent.com/16301109/116553606-47ece380-a935-11eb-8041-a04fc4945f78.png">

#### 4.1.3 Clip Into Small Pieces
Load shapefile with related Google Satellite imagery in GIS (just for visualization).

Google Satellite imager dir:  "./src/ori/cities/Shenzhen/dataset/2.0/image_full/Shenzhen.tif"

Shapefile dir: "./src/ori/cities/Shenzhen/shapefile/shenzhen.shp"

<img width="1524" alt="Screen Shot 2021-04-29 at 22 22 05" src="https://user-images.githubusercontent.com/16301109/116557403-5e953980-a939-11eb-95bb-faf6a4e14b5f.png">

Clip Google Satellite imagery and related ground truth (shapefile) into small pieces

Run:
```
$ python ./utils/preprocess.py
```
Arguments:
```python
# city name
CITY = 'Shenzhen' 
# resolution
RESO = '2.0'
VRT_CLIP = True
# tile shape size
if VRT_CLIP:
    tile_size_x = 5000
    tile_size_y = 5000
```

Dataset saved in "./src/ori/cities/Shenzhen/dataset/2.0/", directory tree:
```
.
├── anno
│   ├── Shenzhen_0.tfw
│   ├── Shenzhen_0.tif
│   ├── ...
│   ├── Shenzhen_41.tfw
│   └── Shenzhen_41.tif
├── anno_full
│   ├── Shenzhen.tfw
│   └── Shenzhen.tif
├── image
│   ├── Shenzhen_0.tfw
│   ├── Shenzhen_0.tif
│   ├── ...
│   ├── Shenzhen_41.tfw
│   └── Shenzhen_41.tif
└── image_full
    └── Shenzhen.tif
```

#### 4.1.4 Upload in GPU Server

SSH connect to server, run
```
$ ssh -l guo sakurag04.cw503.net
```
Make training data dirs in server, run
```
$ cd ~/Work/slum-mapping/src
$ mkdir Shenzhen_2.0_train && cd Shenzhen_2.0_train
$ mkdir image label
$ touch train.txt test.txt
```
Upload dataset from local, run
```
$ scp -r ./src/ori/cities/Shenzhen/dataset/2.0/anno/* guo@sakurag04.cw503.net:~/Work/slum-mapping/src/label/
$ scp -r ./src/ori/cities/Shenzhen/dataset/2.0/image/* guo@sakurag04.cw503.net:~/Work/slum-mapping/src/image/
```
### 4.2 Model Training
In server

### 4.2.1 Data Extraction
Choose training tiles names, adn edit train.txt and test.txt in "./src/";

Edit "./utils/extractor.py", extracted data saved in "./dataset/shenzhen_2.0_train_rand/";

Debug args parameters in extractor.py:
```python
    parser.add_argument('-data', type=str, default="Shenzhen_2.0_train",
                        help='data dir for processing')
    parser.add_argument('-data_usage', type=str, default="train", \
                        choices=['train', 'test', 'trans'],\
                        help='data usage for training, testing, or transfer learning?')
    parser.add_argument('-split', type=list, default=[0.8, 0.1, 0.1],
                        help='train, val, and test partition')
    parser.add_argument('-mode', type=str, default="slide-rand",
                        choices=['slide-stride', 'vector', 'slide-rand'],
                        help='croping mode ')
    parser.add_argument('-img_rows', type=int, default=224,
                        help='img rows for croping ')
    parser.add_argument('-img_cols', type=int, default=224,
                        help='img cols for croping ')
    parser.add_argument('-stride', type=int, default=224,
                        help='img cols for croping ')
    parser.add_argument('-nb_crop', type=int, default=400,
                        help='random crop number')
```

### 4.2.2 Training
Edit ./FPN.py, choose training dataset and related modes:
```python
    parser.add_argument('-train', type=lambda x: (str(x).lower() == 'true'), \
                        default=True, help='train or not?')
    parser.add_argument('-train_data', type=str, default='Shenzhen_2.0_train-rand',
                        help='training data dir name')
    parser.add_argument('-is_multi', type=lambda x: (str(x).lower() == 'true'), default=False,
                        help='multi-class or not')
    parser.add_argument('-trigger', type=str, default='iter', choices=['epoch', 'iter'],
                        help='trigger type for logging')
    parser.add_argument('-interval', type=int, default=10,
                        help='interval for logging')
    parser.add_argument('-terminal', type=int, default=200,
                        help='terminal for training ')
    parser.add_argument('-save_best', type=lambda x: (str(x).lower() == 'true'), default=True,
                        help='only save best val_loss model')
```
Run
```
$ python ./FPN.py
```
Log results saved in "./logs/"; trained model saved in ./checkpoint/

### 4.2 Inference
Take testing Guangzhou_2.0 as an example;

Prepare Guangzhou_2.0_test as introduced in 4.2.1;

Edit "./vissin_Area.py", debug args parameters:
```python
    parser.add_argument('-data', type=str, default="Guangzhou_2.0_test",
                        help='data dir for processing')
    parser.add_argument('-checkpoints', nargs='+', type=str, default=[
        'FPNUNet_epoch_100_Jun06_10_47.pth',
    ],
```
Run:
```
$ python ./vissin_Area.py
```
Results will bed saved in "./result/area-binary".

**Convert into Transparent Mode**
```
$ python ./utils/transparent.py
```
Final results saved in "./result/Guangzhou/2.0_trans/"
### 4.3 Visualization
<img width="1524" alt="Screen Shot 2021-04-29 at 23 29 24" src="https://user-images.githubusercontent.com/16301109/116567869-fa777300-a942-11eb-861e-a940eb2df1ff.png">

CM | Predicted (True) | Predicted (False)
--- | --- | ---
Actual (True) | Green | Red
Actual (False) | Blue | No Color
















