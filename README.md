# Using DeepLabCut for extracting poses from chimpanzee videos

This document explains the process of training a model for estimating poses using DeepLabCut (http://www.mousemotorlab.org/deeplabcut)

## Requirements
* Miniconda
* Spyder3
* DeepLabCut

## Procedure

In general, the process works like this:

* Install the required software
* Convert videos to required format
* Load videos into DeepLabCut
* Label some frames with pose features
* Train neural network model
* Evaluate and update model

Below will be further elaboration of each step

### Installing the required software

In order to use DeepLabCut, you'll need a Python3 environment with specific version of packages. The easiest way to do this is to install miniconda (https://conda.io)

Once you've installed miniconda, you can use the environment file provided in this git repo.

The other piece of software you'll need is Spyder3 (only needed on the desktop, not on the server).


### Convert videos to required format

DeepLabCut needs to have files in either .mp4 or .avi. You might need to use ffmpeg or another program to convert these. e.g.

    ffmpeg -i INPUT2.wmv OUTPUT.mp4

### Loading the videos into DeepLabCut

Once you have DeepLabCut installed, the next step is to set up your project. One good source for extra information about this is the paper (https://www.nature.com/articles/s41596-019-0176-0) and the project documentation (https://github.com/AlexEMG/DeepLabCut).

Here are the commands that you need to run to set up the project:

    import deeplabcut as dlc

    # create a new project
    dlc.create_new_project(PROJECT_NAME,
                           EXPERIMENTER,
                           [VIDEO1_PATH, VIDEO2_PATH, ... ])`

This will create a folder called PROJECT_NAME-EXPERIMENTER-DATE with the video files and a config file in it.

Then, whenever you want to work with the project you just need to specify where the config file for the project is.

    project = "path/to/project/config.yaml"


### Label frames with pose features

The next main step is to extract frames from the videos and manually label them with body parts. By default it extracts 20 frames from each video. This can be adjusted though.

First you should edit the config.yaml file and add the bodyparts that will be labelled.

After importing DeepLabCut and specifying the config file, run the following command.

    dlc.extract_frames(project)

Then, once the frames have been extracted, running the following command will bring up the graphical interface which you will use for labelling.

    dlc.label_frames(project)

Confirm that the labels are correct with:

    dlc.correct_labels(project)


### Train neural network

Training the network is the longest step. If you're just testing it on a laptop, you should use far fewer iterations than recommended. When you want to train it properly, use 100,000 to 200,000 iterations.

First create a training data set:

    dlc.create_training_dataset(project, num_shuffles=1)

Train the network (on a laptop for testing purposes):

    dlc.train_network(project, shuffle=1, trainingsetindex=0, gputouse=None, max_snapshots_to_keep=5, displayiters=100, saveiters=100, maxiters=500)


For actual training, leaving the commands out should be sufficient:

    dlc.train_network(project)


### Evaluate the trained network

The following command will evaluate the trained network in comparison to the manual labels.

    dlc.evaluate_network(project, plotting=True)


### Analyse videos and plot results

You can use the trained network to analyze new videos and save the csv of the poses.

    dlc.analyze_videos(project, ["/full/path/to/video_folder"], shuffle=1, save_as_csv=True)

You can also, for example, plot the labels on the analyzed videos:

    dlc.create_labeled_video(project, ["/full/path/to/video1", "/full/path/to/video2"]


However, the main data to be analyzed will be in the csv files.



## Next steps

Once you have CSV files of pose data, you will need to analyze the data through another program looking for e.g. poses of interest. One way to do this would be to load the CSV into R and define some relations between the configurations between the labels to look for. For example, it would be possible to create a function that returns 'TRUE' if the points of interest are in a straight line (e.g. wrist, elbow, shoulder) and to then give a timestamp for each occurance of this gesture.




