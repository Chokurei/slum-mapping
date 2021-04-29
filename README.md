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
Save to ./src/ori/cities/Shenzhen/dataset/2.0/image_full/Shenzhen.tif
```
no Create VRT
Calculate from Layer -> Shenzhen (as example)
Resolution (current: userdefined) -> 2 (as example)
```
<img width="574" alt="Screen Shot 2021-04-29 at 21 52 58" src="https://user-images.githubusercontent.com/16301109/116553606-47ece380-a935-11eb-8041-a04fc4945f78.png">

#### 4.1.3 Clip Into Small
Shapefile with related Google Satellite imagery
<img width="1524" alt="Screen Shot 2021-04-29 at 22 03 51" src="https://user-images.githubusercontent.com/16301109/116555113-f2193b00-a936-11eb-8896-75a672cfef72.png">








