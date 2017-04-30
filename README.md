# Tensorflow Andriod With audio output

It is an android application for the visually impaired based on deep learning. It captures images using the camera on the phone, do some image pre-processing, then a convolutional neural network(retrained Google Inception V3) will classify the input image, after that there is a text to speech process, and the classification result will be announced loudly to the user.
Right now it recognizes 30 kinds of objects, with 92% test accuracy. You can do much better than this by modifing/retraining the model.

![Aaron Swartz](https://github.com/jxtz518/Tensorflow_Andriod_With_Audio_Output/raw/master/demo.png)

Most basic code is modified from https://github.com/miyosuda/TensorFlowAndroidDemo.

My enviroment/install:
1. Ubuntu 16.04
2. Android Studio
3. Android phone with Talk Back

Run the demo:
Download all files. Open it in Android Studio, connect your phone, then build it, everything should be good :D

Build the demo with your customized model for classifing new categories:

(Here I highly recommond you read https://codelabs.developers.google.com/codelabs/tensorflow-for-poets/#0 first before continue.)

Easiest way is to make use of docker provided by Tensorflow:
Install docker: https://docs.docker.com/engine/installation/, then run following command in the terminal to confirm Docker installation has worked. 

 docker run hello-world
 
 Start a terminal using Terminal.app.

 docker run -it gcr.io/tensorflow/tensorflow:latest-devel 

Exit your Docker instance
 
 cd $HOME
 mkdir tf_files
 cd tf_files
 mkdir photos
 
For each category you want to detect and classify, collect as many images as you can and put it in a folder. Name the folder after its category. Put all folders under /photos. Run docker again:
 
 docker run -it -v $HOME/tf_files:/tf_files  gcr.io/tensorflow/tensorflow:latest-devel
 ls /tf_files/ 
 # Should see: photos 
 
Get tensorflow: 

 cd /tensorflow
 git pull

Retrain the model: 
 python tensorflow/examples/image_retraining/retrain.py \
  --bottleneck_dir=/tf_files/bottlenecks \
  --how_many_training_steps 5000 \
  --model_dir=/tf_files/inception \
  --output_graph=/tf_files/retrained_graph.pb \
  --output_labels=/tf_files/retrained_labels.txt \
  --image_dir /tf_files/photos
 
Wait for a little bit long time, we get retrained_graph.pb and retrained_labels.txt. The first one is retrained model, second one is lables. However they can not be used in Andriod at this stage, do the following:

We first need to optimize it first, using a tool named optimize_for_inference(In docker):
 
 ./configure # you can select all default values
 bazel build tensorflow/python/tools:optimize_for_inference

This step takes forrrrrrrrrrrrrrever :D Be patiennnnnnnnnnnnnnt.

 bazel-bin/tensorflow/python/tools/optimize_for_inference \
  --input=/tf_files/retrained_graph.pb \
  --output=/tf_files/retrained_graph_optimized.pb \
  --input_names=Mul \
  --output_names=final_result
  
Exit docker. Delete the previous ImageNet model from our Android app’s assets folder and place the new model (~/tf_files/retrained_graph_optimized.pb and ~/tf_files/retrained_labels.txt) instead.  

Modify part of the app's ClassifierActivity.java: 

 private static final int INPUT_SIZE = 299;
 private static final int IMAGE_MEAN = 128;
 private static final float IMAGE_STD = 128;
 private static final String INPUT_NAME = "Mul";
 private static final String OUTPUT_NAME = "final_result";

 private static final String MODEL_FILE =
  "file:///android_asset/retrained_graph_optimized.pb";
 private static final String LABEL_FILE =
  "file:///android_asset/retrained_labels.txt";
 
For more details, go to http://nilhcem.com/android/custom-tensorflow-classifier.
Build it again in Android Studio, you are ready to test it on your phone! For the audio output, turn on the talk back. The app will read out our classification result loudly.


 




 

 










