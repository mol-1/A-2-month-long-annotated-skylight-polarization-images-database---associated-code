This is the Readme file to explain processing code associated with our database of skylight polarization images taken at La Timone, Marseille, France. Processing codes are here to calibrate it, reduce image size, and process data to get Angle and Degree of Polarization. It can also help to explain data structure provided.

Enhanced code, after first version (https://github.com/mol-1/Long-Term-Skylight-Polarization-Measurement-Device). This ne version add a time drift correction (5 min over the two month) to compensate for orientation estimation drift that was observed in first version. 


Version 1.1, 24/04/2024

Database download : 

POUGHON, Leo; AUBRY, Vincent; MONNOYER, Jocelyn; VIOLLET, Stéphane; SERRES, Julien, 2024, "A 2 month-long annotated skylight polarization images database.", https://doi.org/10.57745/9L2YUB, Recherche Data Gouv, VERSION PROVISOIRE 

Associated publications, please cite them when using our code and database  :

To come (future publication in BMC Research Notes)


Associated Code on Github : https://github.com/mol-1/A-2-month-long-annotated-skylight-polarization-images-database---associated-code


 
License : CC-BY-SA



![Alt text](annotated_weather.png?raw=true "Annotation values of weather over month")


For code execution, please update the paths in the beginning of each jupyter notebook to match with your own system and install libs specified in requirements.txt. 
You can download all (or only some days by choosing viewing annotated_weather.png figure) timestamped ".npy"  (like 2022-08-02_raw.npy) in a local "PolarizedDatabaseM" folder alongside with manual annotation information (like 2022-08-02_raw_annotations.npy), others numpy files in a calib folder in "PolarizedDatabaseM/calib" folder,  and eventually the "ManipeResultatsRed.tar.gz" archive in a "ManipeResultats" folder. All codes (jupyter notebooks and python files) should be located in a "PolarizedDatabaseCodeM" folder.

This code was executed with python 3.9.6. For parts where processing is parallelized, please ajust the "num_processes" variable to the amount of cores available to your system (in linux you can use "nproc" command in terminal).




Workflow :

ManipeResultats folder -> 1-calibration_orientation_fisheye_with_sun_redac.ipynb -> PolarizedDatabaseM/calib folder 
                       -> 2_reduction_to_PolarizedDatabase.ipynb -> PolarizedDatabase folder
                       
PolarizedDatabaseM folder -> 2_b_PolarizedDatabase_Manual_Annotation.ipynb -> PolarizedDatabase folder
                       
PolarizedDatabaseM folder -> 3_calculation_PolarizedDatabaseProcessed.ipynb -> PolarizedDatabaseProcessedM folder

PolarizedDatabaseProcessedM folder -> 4_Database_Use_example.ipynb 





    First a "ManipeResultatsM" folder was obtained from the system described in the paper. It contain several "acquisitions" and each acquisition is composed of several images taken with auto and fixed exposure time, named after as "image_type" ranging from 0 to 10, with images less than 5 are auto exposure one and 5 and above fixed exposures of growing values with number. As these data are heavy, only three days days are provided on the public in an archive, named "ManipeResultatsRed" 

    The notebook "1-calibration_orientation_fisheye_with_sun_redac.ipynb" explains how the calibration is done and generate files in specified "calib" path in another folder "PolarizedDatabase/calib". It compares sun position detected on sensor on clear day with ephemerids and keeps best calibration. It creates following files in calib folder :

 
dist_center.npy  -> distorsion center of optic, obtained with Scaramuzza's Matlab camera calibration toolbox
lat_lon.npy  -> local position of camera, hard coded in code


f.npy  -> estimated focal lenght of optic, in pixels (~520 pixels * 3.45 um -> ~1.8mm)
orientation_pixels_ENU.npy  -> each pixels rays incidence angles in East-North-Up frame
rotation.npy  -> rotation value between camera and ENU frames, ZXY euler angles
rot_mat.npy -> rotation matrix

drift.npy -> chosen drift values (drift in seconds, start and end date)

A time compensation is also added to counteract time drift on local clock of experiment.





    The notebook 2-reduction_to_PolarizedDatabase.ipynb takes data from "ManipeResultats" folder, crop images and reduce their size to 16 bits (in reality, 12 bits range from IMX250MYR sensor on PHX050S-QC camera, bitshifted to 4 bytes). It creates following files in calib folder :

params_crop.npy
alpha_crop.npy
theta_crop.npy


It also creates files with images grouped for each day on numpy files like "2022-07-29_raw.npy" in PolarizedDatabase folder. These images are associated with exposure time (in µs), timestamp, and image_type (numbers corresponding to fixed or auto exposure values) variables.




    The notebook 2_b_PolarizedDatabase_Manual_Annotation.ipynb can help user to define custom annotations for a choosen day, annotation associated to each image "group" (grouped by "acquisition" : 1 acquisition each 10 minutes in our case) and to create corresponding lists containing timestamps and corresponding annotation. All days in our example were annotated manually with the following sky descriptions (for example, one can choose "c" for clear sky, "m" for mist, ect) :

user_labels={'dark_sky': 'n',
 'clear_sky': 'c',
 'mist': 'm',
 'few_clouds': 'f',
 'scattered_clouds': 's',
 'broken_clouds': 'b',
 'overcast_clouds': 'o'}

For each day (like 2022-08-02_raw.npy) a .npy file is created with annotations for each timestamp (like 2022-08-02_raw_annotations.npy). User can easyly reannotate to add more classes if wanted as annotation is a bit subjective ...


    Finally, code from 3_calculation_PolarizedDatabaseProcessed.ipynb is the only one really usefull to execute for final database user, the two previous ones are useful to understand how the PolarizedDatabase database was made, and this third one allows end user to create locally his own PolarizedDatabaseProcessed.
    
    It take each image, and generate AOP and DOP images from it for each color, and a simulation of equivalent rayleigh image. Eaach one are registered in folders, and a .csv file is created for each day to give Image_folder_path, Timestamp, Hour_Local, Minute_Local, Second_Local, Date, Sun_azimuth, Sun_zenith_angle, Sun_hour_angle, Sun_declination, Camera_exposure_time, and Image_type.
    
    In each Image_folder_path, one can find (and use) :
    
    AOP_B.npy  AOP_Gb.npy  AOP_Gr.npy  AOP_R.npy  AOP_s.npy, AOP values for R, Gb, Gr, B, and rayleigh simulation values,
    DOP_B.npy  DOP_Gb.npy  DOP_Gr.npy  DOP_R.npy  DOP_s.npy  DOP values for R, Gb, Gr, B, and rayleigh simulation values,
    (where 'B', 'Gb', 'Gr', and 'R' correspond to color bayer filters on sensors : 'B' is blue, 'Gb' and 'Gr' are green filters next to blue and red ones, and 'R' red one. 's' correspond to rayleigh simuation.)
    
    expo.npy  for exposure
    fig.png  for processed figure
    image.npy  for raw image
    time.npy for timestamp
    
    
    

Once the processed database is done, one can use it, like in the example provided on 4_Database_Use_example.ipynb to train a simple neural network able to estimate sun's elevation in an image (courtesy of Thomas Kronland-Martinet and Fauzi Akbar for this last code). Please note that it is just an example and that results are not exactly interpretable because of the low variety of the data especially for classes like dark sky that always have low elevation so the network only train with the same data... Please also note that without data augmentation to rotate images, sun's path don't nagigate over the whole hemisphere but only its course over the provided month, so be careful about the biais it may introduce on a trained network that could not recognise all sun's positions on sky without data augmentation.





centroide_images.py, ephemeride.py and rayleigh.py are additionnal functions used by our notebooks to detect centroids of sun, calculate ephemerids with astropy, and calculate a simple rayleigh model for the sky.


For rayleigh sky simulation, it is recommended to use 'OpenSky" simulator (https://github.com/MoutenetA/OpenSky) that is more rigorous. Future versions of the present work may use it instead of the provided rayleigh.py file that isn't exactly accurate due to a rotation to put polarization in camera frame. Simulation is given "as reference".




