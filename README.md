# RASR
---
Re-entry Analyses from Serendipitous Radar data
ver 3.0

Benjamin Miller <benjamin.g.miller@utexas.edu>  
Yash Sarda <ysarda@utexas.edu>  
Robby Keh <robbykeh@utexas.edu>  

Computational Astronautical Sciences and Technologies (CAST), @ The University of Texas at Austin
12/26/2020

## 1. SUMMARY
---
This algorithm searches the NOAA NEXRAD Level II Doppler Radar archive for phenomena indicating meteoroid falls.  For a given set of dates, archive files from across all available radar sites are retrieved \[1] and unwrapped using the Python ARM Radar Toolkit by \[2].  Latitude/longitude/altitude data is returned for all detections.  The original problem was motivated by \[3], and was developed using the [ARES](https://ares.jsc.nasa.gov/meteorite-falls/) database.  The current algorithm uses a PyTorch convolutional neural network object detection algorithm to identify fall velocity signatures, based on the Detecto framework. Tested on Ubuntu 20.04 locally.

> \[1] Staniewicz, S., Keh, R. (2018). Maual_Input_Scraper.py. The University of Texas at Austin
>
> \[2] Helmus, J.J. & Collis, S.M., (2016). The Python ARM Radar Toolkit (Py-ART), a Library for Working with Weather Radar Data in the Python Programming Language. Journal of Open Research Software. 4(1), p.e25. DOI: http://doi.org/10.5334/jors.119
>
> \[3] Fries, M. & Fries, J., (2010). Doppler Weather Radar as a Meteorite Recovery Tool. Meteoritics & Planetary Sciences 45, Nr. 9, 1476-1487. DOI: 10.1111/j.1945-5100.2010.01115.x


## 2. REQUIREMENTS
---
Python (3.8)  
arm_pyart (1.11)  
beautifulsoup4 (4.9.3)  
detecto (1.2.0)  
matplotlib (3.2.3)
netCDF4 (1.5.3)    
numpy (1.19.2)  
pip (20.3.3)
pymap3d (2.4.3)
requests (2.24.0)  
torch (1.7.1)  
tqdm (4.54.1)  

Check envs/ for conda environment file:
~~~
cd envs
conda env create --file rasrenv.yml
~~~


## 3. TRAINING
---
For training on full size images, add them and their respective labels to 2500/train and 2500/test. Aim for anywhere between a 70/30 to 90/10  train-to-test split. Otherwise, adjust the learning rate, epochs, and directories as needed. Run the training script as below:
~~~
conda activate rasr
python training/torchtrain.py
conda deactivate
~~~

## 4. TEST
---
To test if your environment is set up properly, run the following:
~~~
conda activate rasr
python get/rasr_get_test.py
python detect/torchdetect_test.py
conda deactivate
~~~
You should get example detection images and a json file with relevant data in test/. Similarly, you can test the full code:
~~~
conda activate rasr
python get/rasr_get.py x
python detect/torchdetect.py x
conda deactivate
~~~
**Include x to specify the number of processes, useful for running with parallel computing capable machines. If you omit x, RASR will automatically detect how many CPUs you have available and proceed from there.

## 5. OPERATION
---
The full algorithm is designed to be compiled into a set of executables, for ease of use and universality. To this end, we use PyInstaller:
~~~
cd detect
pyinstaller rasr_detect.py
cp -rf dist/rasr_detect ../rd
~~~
~~~
cd get
pyinstaller rasr_get.py
cp -rf dist/rasr_get ../rg
~~~
These commands will create the executable and libraries in a folder called 'dist/name' (i.e, RASR/detect/dist/rasr_detect) and then move them to folders called rg and rd. You can then compress rg and rd and send them anywhere. As long as they are in the same directory, and you run them from one level outside rd/rg, they will work (i.e, move your working directory to the folder that holds rg and rd). Run them as follows:
~~~
./rg/rasr_get x
./rd/rasr_detect x
~~~
RASR should be run daily, either manually with a bash file (not recommended) or as part of a cron job. Note that sometimes, you must manually download arm-pyart/pyart from their GitHub and add that to the rd folder because it sometimes compiles said package incorrectly.
