# Documentation For Lane Module.

# Prerequisites

- Python 3.7 or above
- pip3

# **Dependencies**

- cycler==0.11.0
- fonttools==4.29.1
- kiwisolver==1.3.2
- matplotlib==3.5.1
- numpy==1.22.2
- opencv-python==4.5.5.62
- packaging==21.3
- Pillow==9.0.1
- pyparsing==3.0.7
- python-dateutil==2.8.2
- six==1.16.0

## Installation of the dependencies

[requirement.txt](https://www.notion.so/requirement-txt-eca4e30d0b1d467ba50cc002cb8c7029)

I’ve equipped the module with a ****[requirement.txt](https://www.notion.so/requirement-txt-eca4e30d0b1d467ba50cc002cb8c7029) file, Which includes all the dependencies along with their appropriate versions.

To install all the dependencies, all that is needed to be done, in your environment, is :

```bash
pip3 install -r requirement.txt
```

# About Code

[edge_detection.py](https://www.notion.so/edge_detection-py-5a317f56e0bd4e6e84fb9ac891ca469b)

[lane.py](https://www.notion.so/lane-py-34d4f31b26ff479995a3f1a67050fc19)

Make sure all the files are in the same directory.

[edge_detection.py](https://www.notion.so/edge_detection-py-5a317f56e0bd4e6e84fb9ac891ca469b) will be a collection of methods that helps isolate lane line edges and lane lines.

[lane.py](https://www.notion.so/lane-py-34d4f31b26ff479995a3f1a67050fc19) is where we will implement a Lane class that represents a lane on a road or highway.

## 1. Thresholding

The first part of the lane detection process is to apply thresholding to each video frame so that we can eliminate things that make it difficult to detect lane lines. By applying thresholding, we can isolate the pixels that represent lane lines.

### Thresholding Steps :

- Convert the video frame from BGR (blue, green, red) color space to HLS (hue, saturation, lightness).
- Perform Sobel edge detection on the L (lightness) channel of the image to detect sharp discontinuities in the pixel intensities along the x and y axis of the video frame.
- Perform binary thresholding on the S (saturation) channel of the video frame.
- Perform binary thresholding on the R (red) channel of the original BGR video frame.
- Perform the bit-wise AND operation to reduce noise in the image caused by shadows and variations in the road colour.

The `get_line_markings(self, frame=None)` method in [lane.py](https://www.notion.so/lane-py-34d4f31b26ff479995a3f1a67050fc19) performs all the steps I have mentioned above.

If you un-comment this line (Line No. 791 in [lane.py](https://www.notion.so/lane-py-34d4f31b26ff479995a3f1a67050fc19) ) `cv2.imshow("Image", lane_line_markings)` , you can see the output. 

If you want to configure thresholding in a better way, you can try and do a hit and trial method of the following values :

`_, s_binary = edge.threshold(s_channel, (5, 255))` In line no. 583 of [lane.py](https://www.notion.so/lane-py-34d4f31b26ff479995a3f1a67050fc19)  Try and change the 5 value to get the most optimised solution.

However, it is currently optimised to get results according to our current light and road conditions.

## 2. Apply Perspective Transformation

From the perspective of the camera mounted on a car below, the lane lines make a trapezoid-like shape. We can’t properly calculate the radius of curvature of the lane because, from the camera’s perspective, the lane width appears to decrease the farther away you get from the car. Hence we need to apply a perspective transformation. Fortunately open cv provides help with this task.

### Perspective Transformation Steps :

- For the first step of perspective transformation, we need to identify a region of interest (ROI). This step helps remove parts of the image we’re not interested in.
- Once we have the region of interest, we use OpenCV’s `getPerspectiveTransform` and `warpPerspective` methods to transform the trapezoid-like perspective into a rectangle-like perspective.

To view the Region of interest un-comment the following line : `lane_obj.plot_roi(plot=True)` in line no. 794.

<aside>
💡 We might need to reconfigure the Region of interest (ROI) Based on where the Camera is mounted. 
The Current Region is configured according to the videos, which were previously recorded.
To configure the Region of Interest, change the co-ordinates in lines : 65 to 70.

`self.roi_points = np.float32([
(int(0.475*width), int(0.6*height)),  # Top-left corner
(223, height-1),  # Bottom-left corner
(int(0.9*width), height-1),  # Bottom-right corner
(int(0.6183*width), int(0.6*height))  # Top-right corner
])` 

To find the new co-ordinates, You can run [lane.py](https://www.notion.so/lane-py-34d4f31b26ff479995a3f1a67050fc19) from the previous section. With the image displayed, hover your cursor over the image and find the four key corners of the trapezoid.

</aside>

To view the Transformed Frame, change the value from False to True in `warped_frame = lane_obj.perspective_transform(plot=True)` in line no. 798.
Don’t forget to change it back to False as it will display one frame at a time.

## 3. Identification of lane pixels

We start lane line pixel detection by **generating a histogram** to locate areas of the image that have high concentrations of white pixels.

In [lane.py](https://www.notion.so/lane-py-34d4f31b26ff479995a3f1a67050fc19), make sure to change the parameter value in this line of code (inside the main() method) from False to True so that the histogram will display. 

**Set Sliding Windows** for White Pixel Detection

The next step is to use a sliding window technique where we start at the bottom of the image and scan all the way to the top of the image. If we have enough lane line pixels in a window, the mean position of these pixels becomes the centre of the next sliding window.

Once we have identified the pixels that correspond to the left and right lane lines, we draw a `polynomial best-fit line` through the pixels. This line represents our best estimate of the lane lines.

## 4. Overlay Lane Lines on the original Image.

Once the lane lines have been identified, the information needs to be overlaid on the original image.

Change the parameter on this line form False to True and run [lane.py](https://www.notion.so/lane-py-34d4f31b26ff479995a3f1a67050fc19).

`frame_with_lane_lines = lane_obj.overlay_lane_lines(plot=True)` in line no. 814
Make sure to change it back to False, as this functionality is available only for debugging.

## 5. Calculate curvature of the lane

The formula has been added in the function itself.

If the output is **negative** it indicates the curvature on the **Left** side.

If the output is **positive**, it indicated the curvature on the **Right** side.

The lower the curvature , sharper the the curve (Turn) and vice-versa.

## 6. Car offset from Centre of the lane

This functionality is available for use, but has been disabled in the code as it wasn’t needed.

However, it’s been kept in the code considering future requirements.

The formula has been added in the code.

To enable this function, change the parameter value from False to True in the following code in line no . 814 in [lane.py](https://www.notion.so/lane-py-34d4f31b26ff479995a3f1a67050fc19)

`lane_obj.calculate_car_position(print_to_terminal=False)`

## 7. Calculate Lane Width

This function has been merged with the function above (`calculate_car_position()`)

This function has been called back on line no. 821 to fetch the lane width data.

To make any changes to the formula, change lines : 149 to 158 of [lane.py](https://www.notion.so/lane-py-34d4f31b26ff479995a3f1a67050fc19)

```python
road_width = float(np.abs(bottom_left-bottom_right) * self.XM_PER_PIX)
        self.road_width = road_width

        if print_to_terminal == True:
            print(str(center_offset) + 'cm')
            print(str(road_width) + 'm')
            print(type(road_width))
        self.center_offset = center_offset

        return center_offset, road_width
```

## 8. Video Capture

The main code about video capture is written in lines 742 to 750 of [lane.py](https://www.notion.so/lane-py-34d4f31b26ff479995a3f1a67050fc19).

```python
# Load a video
    cap = cv2.VideoCapture() # either enter integers representing your camera of enter the path of your video file
		#examples
		# cap = cv2.VideoCapture(1) or cap = cv2.VideoCapture(0)
		# cap = cv2.VideoCapture('/home/pranav/vision-analysis/lane-final/video_feed.mp4')

    # Create a VideoWriter object so we can save the video output
    fourcc = cv2.VideoWriter_fourcc(*'mp4v')
    result = cv2.VideoWriter(output_filename,
                             fourcc,
                             output_frames_per_second,
                             file_size)
```

The main things which are needed to be changed here are the parameters to VideoCapture() function

I have provided the examples in the above code snippet for reference.

For eg. `cv2.VideoCapture(1)` represents laptops webcam. (It can vary).

The integer needs to be selected based on the final device, which is going to be used.

To use the code with video file, replace the integer with path to the video file. Example :  `cv2.VideoCapture('/home/pranav/vision-analysis/lane-final/video_feed.mp4')` .

## 9. Inserting Data to csv file.

This particular task is spread over a few lines of code.

However, the main part is from the lines 838 to 852

```python
								counter += 1
                if counter == 25:
                    lane_curvature = (sum(curve_data_list))/len(curve_data_list)
                    lane_width = (sum(width_data_list))/len(width_data_list)
                    width_data_list.clear()
                    curve_data_list.clear()
                    with open('/home/pranav/Desktop/data.csv','w') as csvfile:
                        csv_data = [{'Lane curvature':lane_curvature,'Lane Width':lane_width,'Road Type':'Cement'}]
                        fields = ['Lane curvature','Lane Width','Road Type']
                        writer = csv.DictWriter(csvfile, fieldnames = fields) 
                        writer.writeheader() 
                        writer.writerows(csv_data) 
                    counter = 0
                    lane_curvature = 0
                    lane_width = 0
```

In the above code, I have taken average of 25 values of lane_curvature and lane_width to be inserted into the csv file, to reduce the error. You can change the value (25) in line 839 `if counter == 25:` in order to configure the time taken to insert values to csv file.

As the path for csv file will vary from computer to computer, Please change the path in line no. 844 `with open('<Enter Path here>') as csvfile:` . Enter the path of the csv file, where the data needs to be inserted. Currently its configured for local computer.

# References

**Github Repo :** https://github.com/Pranavoro/lane-detection

**Online Documentation   :**  [lane-detection](https://www.notion.so/Documentation-For-Lane-Module-5d0f676022184955b2df45817be48138)