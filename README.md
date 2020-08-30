# ros_detection_legs 

![tag:status:status:development](https://raw.githubusercontent.com/PouceHeure/markdown_tags/v1.0/tags/status/status_development/status_development_red.png)

![tag:category:deep-learning](https://raw.githubusercontent.com/PouceHeure/markdown_tags/v1.0/tags/category/deep-learning/deep-learning_blue.png)
![tag:category:robotic](https://raw.githubusercontent.com/PouceHeure/markdown_tags/v1.0/tags/category/robotic/robotic_blue.png)
![tag:lib:ROS](https://raw.githubusercontent.com/PouceHeure/markdown_tags/v1.0/tags/lib/ROS/ROS_blue.png)
![tag:language:python3](https://raw.githubusercontent.com/PouceHeure/markdown_tags/v1.0/tags/language/python3/python3_blue.png)
![tag:lib:tensorflow](https://raw.githubusercontent.com/PouceHeure/markdown_tags/v1.0/tags/lib/tensorflow/tensorflow_blue.png)

- [ros_detection_legs](#ros_detection_legs)
  - [Goal](#goal)
  - [Use](#use)
    - [Deep-learning](#deep-learning)
      - [prepocessing data](#prepocessing-data)
      - [train model](#train-model)
    - [ROS](#ros)
  - [Architecture](#architecture)
    - [Extract data](#extract-data)
    - [Preprocessing data](#preprocessing-data)
      - [segmentation](#segmentation)
    - [Training](#training)
    - [Prediction](#prediction)

## Goal 
Extract legs positions from lidar data, like this: 

[![youtube_presentation](https://img.youtube.com/vi/KcfxU6_UrOo/0.jpg)](https://www.youtube.com/watch?v=KcfxU6_UrOo)

https://www.youtube.com/watch?v=KcfxU6_UrOo

## Use

### Deep-learning

:warning: a model is already training, saved in **./model/** folder 
:pencil: if you want to change some parameters, please update [./src/ros_detection_legs/deep_learning/config/parameters.json](./src/ros_detection_legs/deep_learning/config/parameters.json)
#### prepocessing data

run prepocessing script: 
```
$ python3 src/preprocessing.py
```
once run data are stored in **./data/train/**

#### train model

run training script: 
```
$ python3 src/training.py 
```

### ROS

1. compile package
```
# inside your ros workingspace
$ catkin build ros_detection_legs
```

2. run [detector_node](./nodes/detecor_node.py) 
```
# source devel before
$ rosrun ros_detection_legs detector_node.py
```

3. use [ros_pygame_radar2D](https://github.com/PouceHeure/ros_pygame_radar_2D)
```
# clone project to workspace
$ git clone https://github.com/PouceHeure/ros_pygame_radar_2D

# compile package 
$ catkin build ros_pygame_radar_2D

# run radar_node.py 
$ rosrun ros_pygame_radar_2D radar_node.py
```

## Architecture

### Extract data
- package: **ros_lidar_recorder** https://github.com/PouceHeure/ros_lidar_recorder
- data labeling tool: **lidar_tool_label** https://github.com/PouceHeure/lidar_tool_label
- dataset: https://github.com/PouceHeure/dataset_lidar2D_legs

![graph_data_acquisition](.doc/graph/data_acquisition.png)

### Preprocessing data

This schema defines princpals steps: 

![graph_processing](.doc/graph/prepocessing_steps.png)

We can create clusters with a segmentation method, by this way data will gathering if there are closed. 

#### segmentation

Before segmentation, we have to found a way how to compute a distance between 2 polar points. 

- First approach, convert all polar points to cartesien and apply a classic norm. 

- Second approach, find a general expression from polar point to cartesien distance. 

![distance(p1,p2) = \sqrt{r_1^2*r_2^2 - 2*_1*r_2*cos(\theta_1-\theta_2)}](.doc/equation/eq_distance.svg)

A second distance is computed: 

![distance_{radius}(p1,p2) = abs(r_1 - r_2)](.doc/equation/eq_distance_radius.svg)

Once expressions are defined, we have to define hyper-parameters:  

- **limit_distance** 
- **limit_radius**

![graph_segmenation](.doc/graph/segmentation.png)

### Training 

model used: RNN with LSTM cells. 

more information about LSTM: https://www.tensorflow.org/api_docs/python/tf/keras/layers/LSTM

### Prediction

A ros node, **detector_node** subscribes to **/scan** topic. Once data are pusblished to this topic, the node uses the training model to predict legs positions. Legs positions are published to **/radar** topic. 

![graph_prediction](.doc/graph/prediction_ros.png)

Like the training, data need to be tranformed. So before prediction clusters are created, directly in the subscriber callback function. 

![graph_prediction](.doc/graph/prediction.png)