## kicad-laser-min

The application will take a PCB file generated by KICAD and create paths isolating all nets. 
This is isolation routing for PCBs. 

### Operation
The application uses a line Voronoi segmentation technique on the BW image of one layer of PCB. The Voronoi algorithm has been implemented in C++
by colorizing each track a different color. Each track is dilated until regions meet. Finally, an edge detection algorithm will produce the contours.
The contours produce the PCB region isolation. A contour following algorithm will transform the edge into gcode. 

The KICAD pcb to image conversion uses `<pxmm>` scale (which defaults to 30 pixels per mm).

Edge dilation takes time (why it is implemented in C++). The time is dependant on the `<pxmm>` scale.

Drill holes for pads and vias will be added to the end gcode. The holes will be original size. 
Please note that the holes are circular (too much work to create oval gcode).

<img src="images/output.png">
From left to right - BW Image of tracks, Colourized Dilation of tracks, Edge Detection of dilation.

## Note
The application is provided as is. I tested it with my own generated pcbs and worked well. However, your testing is also appreciated.
Please review outputs, especially gcode (as I am not an expert on any of the file formats used).

```KICAD Version used :
    Application: Pcbnew
    Version: 5.1.6-c6e7f7d~86~ubuntu18.04.1, release build
```

### Usage

```Usage: 
    Option: -m         Process map.png directly
            -f         Process Front Copper Layer. (default Bottom Copper Layer)
            -p<pxmm>   Change pixels per mm (default 30)
```

### Files
kicadpcb2contour.cpp

### Output Files
    map.png       - BW output image of Kicad PCB conversion
    cpp_image.png - your mind on LSD. The colorised image dilation of PCB tracks.
    trace.png     - Edge detection of the dilated image
    kic.gcode     - Final gcode



### Dependencies
OpenCV libraries (version 4.2 used)

### Compilation
`g++ -g kicadpcb2contour.cpp -o kicadpcb2contour -I /usr/local/include/opencv4/opencv -I /usr/local/include/opencv4 -L /usr/local/lib -l opencv_dnn -l opencv_gapi -l opencv_highgui -l opencv_ml -l opencv_objdetect -l opencv_photo -l opencv_stitching -l opencv_video -l opencv_calib3d -l opencv_features2d -l opencv_flann -l opencv_videoio -l opencv_imgcodecs -l opencv_imgproc -l opencv_core`

The output is from ```pkginfo``` on include and libraries for ```opencv4```.

### Docker image( To come )

### Errors

Opencv follow contour sometimes does not make closed contours. This is a problem in PCB making, as it would short-circuit PCB tracks. 
However, as shown the inter-contour distance tolerance is of 0.0685mm. I have a laser cutter with an engraving width of 0.2mm (i.e. a
possible 0.1mm engraving width on the centre of track to a track coninciding from left or right side)

Contours do not touch.
<img src="images/error-1.png">

However, distance is small enough to still function.
<img src="images/error-2.png">

OpenCV follow contours creates two contours from both sides. 
One could propose new findcontour method. No real problem here, rather than double time to process pcb.
<img src="images/error-3.png">

```findcountour``` also creates small artefacts, which don't deter from final quality but may slow rendering of PCB.