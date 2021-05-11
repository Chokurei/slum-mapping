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

Save shapefile to "./src/ori/cities/Shenzhen/shapefile/shenzhen.shp", CRS in EPSG:3857 - WGS84 (meters instead of degrees).

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
XYZ Tiles：Google Satellite -> Export layer -> To file ；or XYZ Tiles: New conection-> URL：http://www.google.cn/maps/vt?lyrs=s@189&gl=cn&x={x}&y={y}&z={z} 
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
    
shapefile_root = 'to/your/path/slum-mapping/src/ori/cities/%s/shapefile/%s.shp'%(CITY, CITY)
dataset_dir = 'to/your/path/slum-mapping/src/ori/cities/%s/dataset/%s/'%(CITY, RESO)
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

SSH connect to reedbush server, run
```
$ ssh -l p75001 reedbush.cc.u-tokyo.ac.jp
```
Make training data dirs in server, run
```
$ cd /lustre/gp75/p75001/Work/slum-mapping/src
$ mkdir Shenzhen_2.0_train && cd Shenzhen_2.0_train
$ mkdir image label
$ touch train.txt test.txt
```
Upload dataset from local via sftp, run on local terminal
```
$ sftp p75001@reedbush.cc.u-tokyo.ac.jp
sftp> lcd xx # Change the local directory
sftp> cd xx # Change the remote directory
sftp> put # # Transfer file from local root to remote server
```
an example for uploading annotation, run on local terminal
```
$ sftp p75001@reedbush.cc.u-tokyo.ac.jp
sftp> lcd ./src/ori/cities/Shenzhen/dataset/2.0/anno/
sftp> cd /lustre/gp75/p75001/Work/slum-mapping/src/Shenzhen_2.0_train/label/
sftp> put
```
an example for uploading image, run on local terminal
```
$ sftp p75001@reedbush.cc.u-tokyo.ac.jp
sftp> lcd ./src/ori/cities/Shenzhen/dataset/2.0/image/
sftp> cd /lustre/gp75/p75001/Work/slum-mapping/src/Shenzhen_2.0_train/image/
sftp> put
```

### 4.2 Model Training
In server

#### 4.2.1 Data Extraction
Choose training tiles names, and edit train.txt and test.txt in "./src/shenzhen_2.0_train/";

Edit "./utils/run_extractor.sh" for your own requirement
```
#!/bin/sh
#PBS -q l-debug
#PBS -W group_list=gp75
#PBS -l select=4:mpiprocs=8:ompthreads=4
#PBS -l walltime=00:10:00
cd $PBS_O_WORKDIR
. /etc/profile.d/modules.sh
module purge
module load anaconda3/2019.10 cuda10/10.0.130 intel openmpi/3.1.4/intel
export PYTHONUSERBASE=/lustre/gp75/p75001/packages
export LD_LIBRARY_PATH=/lustre/app/acc/anaconda3/2019.10/lib
python ./extractor.py -data Shenzhen_2.0_train -data_usage train -mode slide-rand -nb_crop 400
```
run run_extractor.sh in ./utils/
```
$ qsub run_extractor.sh
```
extracted data saved in "./dataset/shenzhen_2.0_train_rand/";

#### 4.2.2 Training
Edit ./run_FPN.sh, choose training dataset and related modes:
```
#!/bin/sh
#PBS -q l-regular
#PBS -W group_list=gp75
#PBS -l select=4:mpiprocs=8:ompthreads=4
#PBS -l walltime=100:00:00
cd $PBS_O_WORKDIR
. /etc/profile.d/modules.sh
module purge
module load anaconda3/2019.10 cuda10/10.0.130 intel openmpi/3.1.4/intel
export PYTHONUSERBASE=/lustre/gp75/p75001/packages
export LD_LIBRARY_PATH=/lustre/app/acc/anaconda3/2019.10/lib
python ./FPN.py -train_data Shenzhen_2.0_train-rand -terminal 1200
```
Run
```
$ qsub ./run_FPN.sh
```
Check running status:
```
$ rbstat
```

Log results saved in "./logs/"; trained model saved in ./checkpoint/

### 4.2 Inference
Take testing Guangzhou_2.0 as an example;

Prepare Guangzhou_2.0_test as introduced in 4.2.1;

Edit "./run_inference.sh", debug args parameters:
```
#!/bin/sh
#PBS -q l-regular
#PBS -W group_list=gp75
#PBS -l select=4:mpiprocs=8:ompthreads=4
#PBS -l walltime=00:40:00
cd $PBS_O_WORKDIR
. /etc/profile.d/modules.sh
module purge
module load anaconda3/2019.10 cuda10/10.0.130 intel openmpi/3.1.4/intel
export PYTHONUSERBASE=/lustre/gp75/p75001/packages
export LD_LIBRARY_PATH=/lustre/app/acc/anaconda3/2019.10/lib
python ./vissin_Area.py -data Guangzhou_2.0_test -checkpoints FPN_epoch_300_May11_00_16.pth
```
Run:
```
$ qsub run_inference.sh
```
Results will bed saved in "./result/area-binary".

**Convert into Transparent Mode**
```
$ python ./utils/transparent.py
```
Final results saved in "./result/Guangzhou/2.0_trans/"
### 4.3 Visualization
<img width="1524" alt="Screen Shot 2021-04-29 at 23 29 24" src="https://user-images.githubusercontent.com/16301109/116567869-fa777300-a942-11eb-861e-a940eb2df1ff.png">

Confusion Matrix | Predicted (True) | Predicted (False)
--- | --- | ---
Actual (True) | Green | Red
Actual (False) | Blue | No Color
















