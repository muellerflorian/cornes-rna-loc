
Source code to analyse RNA localization in c. elegans. 

Workflow is implemented in Python and relies on ImJoy to provide an easy to use interface.

# Data preparation

Images have to be saved as single channel z-stacks, follow a naming convention permitting to
extract information about the experiment, channel and field of view (fov). To permit
automated analyss, data has to be further organized in a strict folder structure.

## Test data

We provide already processed test data, that allows to see how the data is named and organized 
and verify if the analysis package gives the same results.

You can download these data from [**here**](TODO).

## Organization and naming conventions

RNA detection is performed with FISH-quant. Please consult the dedication documentation
for how to organize and name your data [**here**](https://fish-quant.github.io/fq-imjoy/data/).

For this workflow, please create an additional folder called `axes_enrichment` with a subfolder `annotations` in the `analysis` folder. This folder will contain all results related to the axes enrichment annotation and computation.

```
├─ fq-imjoy-demo/
│  ├─ acquisition                          # Folder with raw images
│  │  ├─ C1-wt_pos0.tif
│  │  ├─ ...
│  ├─ analysis                             # Folder with all analysis results
│  │  ├─ axes_enrichment                   # Folder with results for axes enrichment
│  │  │  ├─ annotations
│  │  │  │  ├─ C1-wt_pos0.tif
│  │  │  ├─ results
│  │  │  │  ├─ ...
│  │  ├─ spot_detection                   # Folder with spot detection results
```

## Image orientation

Images are rotated to insure the same orientation among different images in a data-set,
facilitating the comparision of results.

1. **Bend** is on the left side of the image.
2. **???** is on the upper right side of the image.

![img_orientation.png](img/img_orientation.png){: style="width:400px"}

## Split channels

This workflow requires that each channel is stored as a separate z-stack. 
If your images are stored as multi-channel z-stacks you have to split these images into 
individual channels. This can be done with different software packages, e.g. with [Fiji](https://fiji.sc/).

1. Open image stack in FIJI.
2. Split channels
    a. From menu: `Image` > `Color` > `Split channels`
    a. Save each channels with a unique channel identifier, e.g. `C1-` or `DAPI_`.

# Analysis

The analysis involves 3 steps

1. RNA detection to localize RNAs in 3D.
2. Manually outlining the axes across the embryo along which RNA enrichment is then calculated.
3. Calculate RNA enrichment.

## RNA detection

RNA molecules are detected with FISH-quant running on our computational platform **ImJoy**. Please follow the detailed instructions provided [**here**](https://fish-quant.github.io/fq-imjoy/fq-overview/) for how to install and run FISH-quant. 

Please also read the next section on how to manually draw the axes along which RNA enrichment will be calculated.

__Note for the test data__
For the analysis of the test-data the following parameters can be used to reproduce the RNA detection

1. `Regular expression`: `(?P<channel>.*)-(?P<file_ident>.*)_(?P<fov>.*)\\.(?P<img_ext>.*)`
2. IMPORTANT: disable `Z first` to read these data.
3. `Channels`: name = `FISH', identifier = `C1`
4. `LOG filter`: sigma_xy = 1, sigma_z = 1.25
5. `RNA detection`: min_dist = 2, threshold = 37 

## Manual outline of axes

Once you loaded an image into FISH-quant, you can draw the outline in this tool. For this

1. Double-click on the loaded image. This will open an image viewer.
2. In this viewer, you can perform different annotations.
3. Press on `+ Add layer` and select `vector` from the pulldown menu.
4. Select the vector layer, and press on the pencil button to enale the `Draw mode`
5. As the annotation geometry, select the symbol for `LineString` .
6. Then draw **ONE** line defining the axes of the embryo.
7. When you are satisfied with the annotation, you can export it by pressing on `Export` button.
8. This will open a dialog stating ???, please confirm
9. Save the annotation in the folder `analysis\axes_enrichment\annotation`, and under the same name as the image with the proposed file-extension `json`, e.g. `C1-wt_pos0.json`

## Automatic analysis of enrichment data

This is done with an ImJoy plugin that can be installed from [**here**]().

The strict file-organization permits to batch process an entire folder. You have to specify

1. Path to analysis folder.
2. Image size
3. Maximum distance that an RNA can be away from the outlined axes and still be considered.
4. Size of smoothing average window.
5. Pressing on the plugin name will then process all files where spot detection results and an annotated axes is present.

### Overview of analysis

1. Each RNA is assinged to the closest point on the axes. RNAs that are further away than the defined threshold will be ignored. 
2. Distance along the axis is calculated.
   1. **0**: position of the "bend", i.e. the point that is the furthest on the left.
   2. Measurments start at upper right part, i.e. negative distances are reported for the upper part. 

## Results

Several result files will be saved in the folder `analysis\axes_enrichment\results`. The file-names
start with the name of the image and carry a suffix

* **__axes_enrich.csv**: csv file with all information about the axes enrichment.
  * ax1, ax2: coordinates of the polygon defining the axes
  * dist_orig: distance along axis (with 0 being the bend)
  * n_rna: number of spots for this data-point
  * n_rna_movavg: number of RNAs after calculating the smoothing average.
  
* **__axes_enrich.png**: plot summarizing the analysis results.



