# Image Editor in Java for jpg files with a JavaFX GUI

## Which are the main aspects of my project?

- it uses 16 threads to apply a filter in order to cut down on time
- you can choose which filter you want to apply to the image from the gui (it's important to note that only one filter can be applied at a time)
- you can apply multiple filters consecutively without having to move the file from the output folder to the input folder
- you can choose to copy the metadata from the input file, keeping details from the camera
- except for the gui, it uses only java native libraries, this includes the calculations done to each pixel (hence why I've decided to go with 16 threads)
- for the 20 filters, I've used the Factory design pattern, which means there is no endless "if else" and adding a new filter doesn't intervene with the rest of them
- lossy compression is set to 0 so no quality is lost (sizing goes up just a tad)
- I've coded various methods for checking paths so the program doesn't break if the input files aren't found (this includes the paths for the src folders)

` If you want to see some of the images, you can check out: `https://drive.google.com/drive/folders/1gRENFJv9aT3yPnJnu8ERoQtnzC15GD_L

## Installation
### If you want to try it out, you'll need the following:
- Visual Studio Code
- Extension Pack for Java (VSC extension)
- I work with jdk 17, haven't tested it on older versions so this is what I recommended
- javafx-sdk-20 (https://gluonhq.com/products/javafx/) 
- Run the code once from src/Gui/Main.java for VSC to create its own configuration in the launch.json file
- Next you'll need to go the top configuration which has "projectName": "ImageEditorV3WithGUI_ + a new hash" and add the following line
"vmArgs": "--module-path path_to_where_javafx-sdk-20_was_downloaded --add-modules javafx.controls,javafx.fxml"

## GUI
As previously mentioned, the gui was made using javafx. The control elements are: buttons, check-boxes, radio-buttons, text-fields, one slider and one combobox.
The "Input image name", "Output image name" and "Watermark" labels change color as well as text after pressing the Start button. So if the input image isn't found or 
the incorrect extension is typed, the labels will change color from black to red and the text will become "Error: Image not found/not jpg!". Nonetheless, if the file is found 
and it's a jpg, the text will change to blue and "Image found.". For the output text-field as well as the watermark we have similar behaviour. This is achieved using the Controller 
java file. The slider is another element with its own unique behaviour. You can drag it up and down (the red 0 will get swapped for the new value) changing the parameter for 8 filters. 
The styling is accomplished using a css file. Buttons have padding, are rounded with colors swapping when hovering over or pressing them. Both the text-boxes and buttons have shadows for a 
better look. When pressing the exit button or closing the window you'll be greeted with a smaller window (an allert) that also uses css styling for different text, buttons and image. Another 
element which is not visibile at first is an image-view. When pressing the Start button, the input image will be shown for a short time untill the process is completed and gets replaced by the new image.

## Applying filters

EditorMain is the java file which receives the paths for the input, output and watermark images, their presence being confirmed in the Controller class.
Each thread receives the locations for the pixels that are being processed (from packImageSizeFactory where we have another Factory with classes returning the width and height (start and end) for each segment relative to the image), the buffered image as well as the filter type and value. Then each pixel is divided into 3 values R, G and B and depending on the filter, it works with the raw values (for example for greyscale, we calculate the average without the need of an external library). An interesting filter is Paint which is inspired by an algorithm found in "Computer Science: An Interdisciplinary Approach by 
Robert Sedgewick, Kevin Wayne". For each pixel we find the most frequent R, G and B values in a square surrounding the pixel and then we replace it. It is important to mention that the new values are not taken into consideration later on during the process, only the original ones as well  as the fact that it consumes a lot of the cpu's resources (which is why it is in red).
I have decided that the 16'th thread is the one responsible for calling the class that writes the new image and I made sure that it waits untill the the other threads have finished. 
ImageWriterClass as its name suggests writes the new image with 0 compression (I like my images clear, but you can change it on line 53) with or without the metadata. In the Temp folder we have temporary files which can be deleted as well as a text file with information like date and time, filter name and value, time taken and number of pixels. Throughout the project we have various try and catch statements as well as printing for paths.


## The app

### Running the app using VS Code

In order for VS Code to execute the code, a small addition needs to be made to the configurations array with the path to the javafx libraries.

![Launch File Screenshot](https://drive.google.com/uc?export=view&id=1lqvLcJ950M6j9L9YUYNb9nkccoU5Cu0q)

### The interface

The GUI contains 2 main sections, one for choosing the wanted filter, the other for entering the file names for input and output. The second section also contains an input for adding a watermark, although it is optional.

![GUI Screenshot](https://drive.google.com/uc?export=view&id=1ln82CZFiDQlBEr4L8PZJOCPiOp1s7ffH)

If the input image file is found, but an ouput name is not entered, then the image will be shown bellow the main section and a warning message will replace the initial label.

![GUI Screenshot](https://drive.google.com/uc?export=view&id=1lvNkV-mDJZejta3oGHSEgjOBaJk-V46X)

After entering a valid name for the output image and pressing the start button, the app will create a new image based on the selected options and, once completed, will replace the initial image in the GUI. 

![GUI Screenshot](https://drive.google.com/uc?export=view&id=1lwlkyjIiV7munp1uue14gBmrwLhPKcuy)

Upon pressing the exit button, a confirmation window is shown to either confirm or cancel.

![GUI Screenshot](https://drive.google.com/uc?export=view&id=1lrfwymvrAk_yrzJ8kqJV4JzUAEW00qzI)

### Folder structure for images

The Images folder contains 3 subfolders used for reading the input image (this is where house.jpg needs to be placed), writing the output image and for reading and writing images when multiple filters are applied to the same input file (the "Continue editing the same image" checkbox is selected).

![Folder Structure Screenshot](https://drive.google.com/uc?export=view&id=1m0xJ_MtBoHfq_WbWdX8EFbuVDdE4H_Qq)

### Terminal

Inside the terminal, the application prints useful information such as the timings for each thread. In the example from above, inverting the colors on a 4000x3000 image takes roughly 150-260ms. 
The last thread is the one which also writes the image. The writing process has a 2 second delay which I've added because of an issue with either java or javafx not being able to read and place the new image in the GUI right after it got written in memory. So the real timing for inverting the colors and creating the new image takes about 1 second.

![Terminal Screenshot](https://drive.google.com/uc?export=view&id=1lnF0_3qesS-bvCpjORqX9hHlKGV5caIj)
