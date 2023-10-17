![view drawio](https://github.com/anirudh-muthuswamy/Image_Enhancement_and_Manipulation_Java/assets/126427302/3b9f15c2-42da-4773-97e6-a4cc09a00c55)



***Code Overview:***

This document gives a high level overview of the current design choices and implementation for our 
project. We adopt the model view controller Architecture to give a basic structure to the project.
Furthermore, in order to make the GUI based view, we evolve our design choice by incorporating a Model View View Model approach. 
We have a Read Only Model class that has methods to only read relevant data (such as the BufferedImage stream). 
This is useful as the view needs to have the data that it needs to display.

**VIEW CLASS**

The View class is where the Actual user interacts with the program. 
The program supports 2 types of interactions. Text based and GUI based. 

1. The text based interaction can be called by passing the -text flag while running the JAR file.
	It supports all the commands as mentioned later in this document.

2. The GUI based interaction is the default interaction medium. ie it requires no flag to be to loaded while running the jar file. 
	We use the JAVA SWING framework package to create our GUI.
	The design of the GUI View is as follows :
	a. Main Frame : this is the main parent frame that contains all other components of the GUI. (Component Type : JFrame)
	b. Menu Items : On the top of our program window, we create a Menu Bar which provides the user all the functionalites supported in the program.
		The remainder of the Main Frame is divided into 2 parts(leftPanel and imagePanel) in a columnwise fashion, creating demarkation between the 			
		image canvas and the left panel hich has to more panels aligned row wise(imageListPanel and HistogramPanel).(Component Type : JMenuBar)
		The Menu Items have Menu Options such as FILE ,FILTER ,TRANSFORM ,MODIFY. Each of these sub menu's have clickable menu options which are 				
		used to execute the command.

	c. ImagePanel : This is the panel which houses the current image loaded in the program. Whenever an image is chosen from the imageList Panel,
		this Panel is refreshed as well. The Image is wrapped inside of a JLabel(imageLabel). This panel gets scrollable bars whenever the image size is 		
		larger than the main frame.(Component Type : JScrollPane). 
	d. LeftPanel :This panel is used to create a the ImageList and Histogram panels. The ImageList is a scrollPanel and is responsible for populating all 		
		the images currently loaded in the program.(Component Type : JPane)
	e. ImageList : This component is responsible for displaying all the image names currently loaded in the model. (Component Type : JScrollPane)
	f. HistogramPanel : This componenent wraps around the histogram of the currently loaded image. This componenent is also refreshed with the ImagePanel 		
		in order to maintain the same state as the loaded image.
	g. Whenever an error is encoutered, Our GUI pops up an error message dialog box which is a JOPtion Pane. It handles errors like selecting features 			
		when the image is not loaded, Saving an image with an unsupported extension, Combing images with different dimensions and without giving a new 			
		alias to the new combined image. 

Additional Features: 
	As the Text based controller supported multiple images to be worked on concurrently, we try to implement this in our GUI implementation as well.
	The ImagePanel is used to display all currently loaded images in the program. With each image being selected, the image name is queried into the Model 	
	and the corresponding ImageBuffer is returned. This Buffer is user to show the image as well as create the histogram.


**CONTROLLER CLASS:**

In order to support GUI Operations, We have a new controller called the GUIController. 
The GUI Controller performs the exact same function as the earlier text based controller did, However as the user inputs are not text based, 
a new controller is required to take in inputs from the Swing UI framework.

We create a new Interfaces called 'Features', which represents all the features supported by the application.
The GUI controller implements this interface. and is also sent to the GUIView for method calls to the controller and futher the model.

The controller class is where the program is started from, each command given by the user if first sent to the controller.
We implement the Command Design Pattern which allows less modifications to be made in order to support newly added commands. 
This design change leads us to have a seperate interface called 'Command Interface' which is implemented by each of the supported function. 

The Model calls of the respective features are done from their individual classes. 

The controller class has a method 'runController' which is responsible for redirecting the given command to the correct model method. 
With the help of a while loop, we keep the controller active throughout the session. Invalid commands are not executed and are simply ignored, 
the user is prompted to give a valid command if the controller comes accross an invalid command.


We support the following commands: 

1. load image-path image-name: Load an image from the specified path and refer it by image-name henceforth.

2. save image-path image-name: Save the image with the given name to the specified path.

3. greyscale <red,green,blue,luma,intensity,value>-component image-name dest-image-name: Create a greyscale image with the requested component of the image with the given name and refer to it henceforth by dest-immage-name. 

4. horizontal-flip image-name dest-image-name: Flip an image horizontally to create a new image, refer to it as dest-image-name.

5. vertical-flip image-name dest-image-name: Flip an image vertically to create a new image, refer to it as dest-image-name.

6. brighten increment image-name dest-image-name: brighten the image by the given increment to create a new image, referred to henceforth by dest-image-name.

7. rgb-split image-name dest-image-name-red dest-image-name-green dest-image-name-blue: split the given image into three greyscale images containing its red, green and blue components respectively.

8. rgb-combine image-name red-image green-image blue-image: Combine the three greyscale images into a single image that gets its red, green and blue components from the three images respectively.

9. run script-file: Load and run the script commands in the specified file.

10. sepia image-name dest-image-name : Apply the Sepia Image transform on the image-name to create a new image and refer to it as dest-image-name.

11. sharpen image-name dest-image-name : Apply the Sharpen Filter on the image-name to create a new image and refer to it as dest-image-name.

12. blur image-name dest-image-name : Apply the Blur Filter on the image-name to create a new image and refer to it as dest-image-name.

13. dither image-name dest-image-name : Apply the Dither Operation on the image-name to create a new image and refer to it as dest-image-name.

14.  greyscale image-name dest-image-name :Along with the prior greyscale command, we also support a this command which defaults the component argument to luma which uses the greyscale transform operation.


**MODEL CLASS:**

We include a read only version of the Model that is used by the View to display the image and get information regarding the currently loaded images. 
This read only model contains methods like : getImageNames and getImage.
the getImage method takes in the name of the image the user wants displayed on the view and returns the buffered image data.
the getImageNames is used to populate the imageList panel which is used to support the usage of multiple images. 


1. The Image is loaded from the storage and converted into an Input Stream and sent to the Model.
Similarly, for the saving of an image, the Model sends a ByteArrayOutputStream which is then saved to the disk in the controller. 
This makes the model independent of Image Type Support , as long as a controller can parse an image from the disk and convert it to a Stream, our model would work with all image types. 

2. we use int[][][] as a data structure to store pixels and instead use a List of List of Objects. 
In response to this, we have a ImagePixel Interface which is implemented by the PixelRGB class. 
This PixelRGB class is the building block for our Image. 

3. Due to the fact that the model always recieves a generic InputStream in the load method and sends a generic OutputStream in the save method, our model becomes independent of image type. 
This means that our earlier design of having separate image classes for each image type are no longer required. We see this as an opportunity to improve our design choice 
	by having the ImageImpl class only house image pixel data and  meta data. Each Operation performed on the Image is now classified as either a Modification, 
	Transformation or Filter Operation. This leads us to having lesser coupling between our classes. 

In future if we need to add an operation, All we need is to create a new method in one of the image modifation/filter or transform class . 
Furthermore, this makes our Image Operations independent of the Image Impl class as well. In future if we have another 
ImageImpl class, our Image Manipulation operations will be supported for that class as well. 

Other features in our model class: 

We use a Hashmap to store the imagename and the corresponding Image Object. This is useful for us as we can find if a imagename is present in our program at any given time or not. 

We now discuss about the Classes and Interfaces present in our Model:
0. ModelImpl class implements Model interface : The Main Class and Interface that interacts with the contoller. This class has seperate methods for each of the features supported by the program. This is the only class which is visible to the client. The client interacts with the model only using this file.

1. PixelRGB class implements ImagePixel Interface: This is building block of our Image class. We use this PixelRGB class to represent red, green, blue components of the pixel. This class also has method called getData() which returns the components  of the pixel. This Interface is package private.

2. ImageImpl class implements Image Interface: This class represents Image Object. It stores the name of the image, height of image, width of image and the List of List of PixelRGB data Object.
It also has a method which takes in the height, width and component of the image pixel and returns the component value.This Interface is package private.

3. ImageModifyImpl class modifies ImageModify interface : This class represents all the Modification Operations supported by the application. Operations such as Horizontal and Vertical Flip, RGB Split and RGB Combine are implemented in this class. This Interface is package private.

4. ImageTransformImpl class modifies ImageTransform interface :This class represents all the Tranform Operations supported by the application. Operations such as GreyScale, Sepia and Brighten. We have a private helper method applyTransform which takes in a transformation matrix and an Image and returns a List of List of PixelRGB values for the new transformed Image. This leads to abstraction of the transform function. In the future, given any transformation matrix and an image Object, we can create a new transformation operation in minimal lines of code. This Interface is package private.

5. ImageFilterImpl class modifies ImageFilter interface :This class represents all the Filter Operations supported by the application. Operations such as Dither, sharpen, Blur are implemented in this class. 
Similar to the applyTransform method in ImageTransformImpl, we have a applyFilter Method which does a similar operation given a Filter matrix and an Image. This Interface is package private.


**Instructions to run script.**

1.  Write all the commands in a txt file and save it in the root of this directory.
    (for eg. Name of script : script.txt and it should be saved at the same level as the src and test folders.)
2.  Open the Main.java file and run the program.
3.  For single line commands directly enter the command in the Intellij prompt.
	(for eg. load image-filepath imagename ; brighten 10 imagename newimagename)
4.  In order to run a script file use the "run" argument as a prefix to the filename. 
    (for eg. to run a file "script.txt" , the command used should be "run script.txt")


Instructions to run JAR File:

1. Open terminal in the res folder. 
2. for GUI interactions : run the jar file using the command  : "java -jar Image_Enhancement.jar"
3. For text based interactions : run the jar file using the commands : "java -jar Image_Enhancement.jar -text"
4. For running scripts present in text file : run the jar file using the command: "java -jar Image_Enhancement.jar -file <filename>.txt"



**TEST IMAGE SOURCE :**
shorturl.at/jyAK3
