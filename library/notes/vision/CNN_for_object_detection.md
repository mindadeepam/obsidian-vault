#vision #cnns #objectdetection

# Basic Concepts

#### IoU
[andrewng lecture](https://www.coursera.org/learn/convolutional-neural-networks/lecture/p9gxz/intersection-over-union)
#### Anchor boxes
[andrewng lecture](https://www.coursera.org/learn/convolutional-neural-networks/lecture/yNwO0/anchor-boxes
#### mAP:
  We use a metric called “*mean average precision*” (mAP)
  Compute average precision (AP) separately for each class, then average over classes.
  A detection is a true positive if it has IoU with a ground-truth box greater than some threshold (usually 0.5) (mAP@0.5)
  Combine all detections from all test images to draw a precision / recall curve for each class; AP is area under the curve. mAP is the mean for all classes.
  TL;DR mAP is a number from 0 to 100; high is good.

#### Non max supression
[andrewng lecture](https://www.coursera.org/learn/convolutional-neural-networks/lecture/dvrjH/non-max-suppression) 

Likely the model is able to find multiple bounding boxes for the same object. Non-max suppression helps avoid repeated detection of the same instance. After we get a set of matched bounding boxes for the same object category: Sort all the bounding boxes by confidence score. Discard boxes with low confidence scores. _While_ there is any remaining bounding box, repeat the following: Greedily select the one with the highest score. Skip the remaining boxes with high IoU (i.e. > 0.5) with previously selected one.
```
Input:
    boxes: List of bounding boxes (x_min, y_min, x_max, y_max)
    scores: List of confidence scores for each bounding box
    iou_threshold: IoU threshold for suppression

Output:
    selected_boxes: List of bounding boxes after NMS

Algorithm:
1. Initialize an empty list `selected_boxes`
2. Sort `boxes` by `scores` in descending order
3. While `boxes` is not empty:
    a. Select the box with the highest score (first box in the sorted list)
    b. Add this box to `selected_boxes`
    c. Remove this box from the `boxes` list
    d. For each remaining box in `boxes`:
        i. Calculate IoU between the selected box and the current box
        ii. If IoU > iou_threshold:
            - Remove the current box from `boxes`
4. Return `selected_boxes`
```
  
  ![[non-max-suppression.png|Fig. 3. Multiple bounding boxes detect the car in the image. After non-maximum suppression, only the best remains and the rest are ignored as they have large overlaps with the selected one)]]

---

# Single Stage Detectors


## YOLO(you only look once): 2015

[paper link](https://arxiv.org/pdf/1506.02640)
### Key Contributions:
YOLO is the first single-shot detector, reformulating object detection as a single regression problem, predicting both bounding boxes and class probabilities directly from the image. (no region proposals, no anchor boxes, etc)
![[Screenshot 2024-08-15 at 2.36.19 PM.png|300x200]]

### Architecture:

YOLO uses a fully convolutional network. It is inspired from Googlenet/NiN architectures. It has 2 versions fast and normal, faster one has fewer layers.

> Our network has 24 convolutional layers **followed by 2 fully connected layers**. Instead of the inception modules used by GoogLeNet, we simply use 1 × 1 reduction layers followed by 3 × 3 convolutional layers, similar to NiN. The full network is shown in Figure 3.

![[Screenshot 2024-08-15 at 2.34.05 PM.png|YOLO's simplicity allows for very fast inference.|700x400]]

For evaluating YOLO on PASCAL VOC, we use S = 7, B = 2. PASCAL VOC has 20 labelled classes so C = 20. Our final prediction is a 7 × 7 × 30 tensor.
The final output of our network is the 7 × 7 × 30 tensor of predictions.
### Training:

First the model pretrained on Imagenet for classification then its trained for detection. Then, they add four convolutional layers and two fully connected layers with randomly initialized weights. Detection often requires fine-grained visual information so we increase the input resolution of the network from 224×224 to 448×448.

During detection, YOLO’s training process uses a unified loss function, combining mean-squared error loss for bounding box prediction and cross-entropy loss for classification. The network is trained end-to-end on images, with ground truth labels providing bounding boxes and class labels.

The image is divided into a grid, and each cell in the grid predicts a fixed number of bounding boxes, along with confidence scores for each box and class probabilities. Non maximum supresion is applied on top.

For each image, each cell in the grid predicts $B$ (typically 2) bounding boxes(or bb predictor/anchor boxes), along with a confidence score for each box and $C$ class probabilities (1 to C for C classes). 

Hence the shape of output from one image: 

$$
[B*(x,y,h,w,b, c_1,c_2,c_3.., c_n)] -> B*(5+C)
$$


where
- $b$ = box confidence. These confidence scores reflect how confident the model is that the box contains an object and also how accurate it thinks the box is that it predicts. Formally we define confidence as  $Pr(Object) ∗ \text{IOU}_{\text{pred}}^{\text{truth}}.$ If no object exists in that cell, the confidence scores should be zero. Otherwise we want the confidence score to equal the intersection over union (IOU) between the predicted box and the ground truth.
- $c_1, c_2, .., c_n$ = (conditional) class probabilities. conditional because these probabilities are conditioned on the grid cell containing an object, ie its target is 1 when there is an object and we dont backpropagate when there isnt.
- $x,y$ = are the centre coordinates. (bw 0 and 1)
  **$t_x$ and $t_y$ are the network’s raw predictions** for the center coordinates of the box. These predictions are passed through a sigmoid function, so they are constrained to fall between 0 and 1, ensuring the predicted box stays within the grid cell. The final center coordinates $(b_x, b_y)$ are computed as:
  ($b_x = \sigma(t_x) + c_x$) and ($b_y = \sigma(t_y) + c_y$)
  here $c_x , c_y$ are topleft coordinates of grid.
  
- $h, w$ = is the height and width of box relative to the image. (bw 0 and 1)
  YOLO predicts these values as a scaling of the predefined anchor box dimensions.  **$t_w$ and $t_h$ are the network’s raw predictions** for the width and height. Instead of constraining these with a sigmoid, YOLO uses an exponential transformation. The final width and height of the bounding box are computed as:
  ($b_w = p_w \cdot e^{t_w}$) and ($b_h = p_h \cdot e^{t_h}$)
  where:
	  - $p_w$ and $p_h$ are the predefined dimensions (width and height) of the anchor box associated with that prediction.
	  - $e^{t_w}$ and $e^{t_h}$ allow the predicted width and height to scale up or down from the anchor box size.

This formulation ensures that the predicted box sizes remain positive and can vary significantly based on the predictions.


![[Screenshot 2024-08-19 at 6.16.42 PM.png|300x300]]



At training time we only want one **bounding box predictor** (out of the $B$) to be responsible for each object. We assign one predictor to be “responsible” for predicting an object based on which prediction has the highest current IOU with the ground truth. This leads to specialization between the bounding box predictors. Each predictor gets better at predicting certain sizes, aspect ratios, or classes of object, improving overall recall.

Most boxes wont contain any object. This pushes the “confidence” scores of those cells towards zero, often overpowering the gradient from cells that do contain objects. This can lead to model instability, causing training to diverge early on. To stabilize training, they increase the loss from bounding box coordinate predictions and decrease the loss from confidence predictions for boxes that don’t contain objects by multipliying by constants ($\lambda_{coord}=5, \lambda_{noobj}=0.5$)

To avoid overfitting we use dropout and extensive data augmentation. A dropout layer with rate = .5 after the first connected layer prevents co-adaptation between layers. For data augmentation we introduce random scaling and translations of up to 20% of the original image size. We also randomly adjust the exposure and saturation of the image by up to a factor of 1.5 in the HSV color space.
### Loss

![[Screenshot 2024-08-15 at 3.31.36 PM.png|The loss function in yolov1.|500x400]]

1. **Localization Loss** (Bounding Box Regression):
   This part of the loss ensures that the predicted bounding boxes are accurate. It measures the error in the position (center coordinates $x, y$) and size (width $w$, height $h$) of the predicted bounding boxes with respect to the ground truth boxes.
   - **Position Loss**: This term penalizes the difference between the predicted and true bounding box center coordinates. It usually uses a squared error loss.
   - **Size Loss**: This term penalizes the difference in the width and height of the predicted and true bounding boxes, also using squared error. However, YOLO v1 applies the square root to the width and height to avoid dominance of large objects in the loss.

   Mathematically, the localization loss is:
   $$
   \lambda_{\text{coord}} \sum_{i=0}^{S^2} \sum_{j=0}^{B} \mathbb{1}_{ij}^{\text{obj}} \left( \left( x_i - \hat{x}_i \right)^2 + \left( y_i - \hat{y}_i \right)^2 + \left( \sqrt{w_i} - \sqrt{\hat{w}_i} \right)^2 + \left( \sqrt{h_i} - \sqrt{\hat{h}_i} \right)^2 \right)
   $$

   - $S$: Grid size (e.g., $7 \times 7$)
   - $B$: Number of bounding boxes predicted per grid cell (typically 2)
   - $\mathbb{1}_{ij}^{\text{obj}}$: Indicator function, 1 if the $j$-th bounding box in grid cell $i$ is responsible for detecting an object.

2. **Confidence Loss**:
   This term accounts for how confident the model is about the presence of an object in a predicted bounding box. The confidence score is the product of the Intersection over Union (IoU) between the predicted box and the ground truth box and the probability of the object. The loss penalizes incorrect confidence scores for both object-containing and object-free cells.
   - **Object Confidence Loss**: Penalizes the difference between the predicted confidence score and the IoU for cells that contain objects.
   - **No Object Confidence Loss**: Penalizes the confidence score in cells that do not contain objects, ensuring the model does not predict objects where none exist.

   The confidence loss is:
   $$
   \sum_{i=0}^{S^2} \sum_{j=0}^{B} \mathbb{1}_{ij}^{\text{obj}} \left( C_i - \hat{C}_i \right)^2 + \lambda_{\text{noobj}} \sum_{i=0}^{S^2} \sum_{j=0}^{B} \mathbb{1}_{ij}^{\text{noobj}} \left( C_i - \hat{C}_i \right)^2
   $$

   - $C_i$: Predicted confidence score
   - $\hat{C}_i$: Ground truth confidence score $=$ $IOU$ bw gt and predicted bbox.
   - $\lambda_{\text{noobj}}$: Weight term that reduces the impact of the loss from cells not containing objects (typically set to 0.5).


3. **Classification Loss**:
   This part of the loss penalizes errors in the predicted class probabilities. YOLO v1 predicts the class probabilities for each grid cell and uses a standard squared error loss to measure the difference between the predicted and true class labels. This loss is applied only to the cells that contain objects.

   The classification loss is:
   $$
   \sum_{i=0}^{S^2} \mathbb{1}_{i}^{\text{obj}} \sum_{c \in \text{classes}} \left( p_i(c) - \hat{p}_i(c) \right)^2
   $$

   - $p_i(c)$: Predicted class probability for class $c$ in cell $i$
   - $\hat{p}_i(c)$: Ground truth class probability for class $c$. 

**Overall Loss Function**

The overall loss function is a weighted sum of these three components:
$$
\text{Loss} = \lambda_{\text{coord}} (\text{Localization Loss}) + (\text{Confidence Loss}) + (\text{Classification Loss})
$$
- $\lambda_{\text{coord}}$: A constant to increase the weight of the bounding box coordinates loss (usually set to 5 in YOLO v1).
- $\lambda_{\text{noobj}}$: A constant to decrease the weight of the no-object confidence loss.

These terms balance localization accuracy, confidence score correctness, and classification performance.

	Note that the loss function only penalizes classification error if an object is present in that grid cell (hence the conditional class probability discussed earlier). It also only penalizes bounding box coordinate error if that predictor is “responsible” for the ground truth box (i.e. has the highest IOU of any predictor in that grid cell).

### Inference

At **test** time we multiply the conditional class probabilities and the individual box confidence predictions, $$Pr(Classi|Object) ∗ Pr(Object) ∗ \text{IOU}_{\text{pred}}^{\text{truth}}$$ $$= Pr(Object) *\text{IOU}_{\text{pred}}^{\text{truth}}$$which gives us class-specific confidence scores for each box. These scores encode both the probability of that class appearing in the box and how well the predicted box fits the object. lets denote this with $p_i(c)$.

Other than this, Non maximal supression is applied to remove redundant predictions
### performance:
![[Screenshot 2024-08-15 at 3.38.31 PM.png|500x200]]

![[Screenshot 2024-08-15 at 3.39.20 PM.png|Table 1. yolo is faster than faster-rcnn but not as good.|500x250]]


### Limitations:

YOLO imposes strong spatial constraints on bounding box predictions since each grid cell only predicts two boxes and can only have one class. This spatial constraint limits the number of nearby objects that our model can predict. Our model struggles with small objects that appear in groups, such as flocks of birds. 
Since our model learns to predict bounding boxes from data, it struggles to generalize to objects in new or unusual aspect ratios or configurations. Our model also uses relatively coarse features for predicting bounding boxes since our architecture has multiple downsampling layers from the input image. 
Finally, while we train on a loss function that approximates detection performance, our loss function treats errors the same in small bounding boxes versus large bounding boxes. A small error in a large box is generally benign but a small error in a small box has a much greater effect on IOU. Our main source of error is incorrect localizations.



## YOLO v2

[paper](https://arxiv.org/pdf/1612.08242)

![[Screenshot 2024-08-19 at 8.36.24 PM.png]]
### YOLOv2: You Only Look Twice (2016)

#### Key Contributions:
YOLOv2, also known as YOLO9000, builds on YOLOv1 and introduces several enhancements to improve object detection performance. It reformulates object detection with the same single-shot, end-to-end approach but with significant improvements in accuracy, speed, and scalability.

#### Architecture:

YOLOv2 uses a more advanced network architecture called Darknet-19. Here are its main components:

- **Backbone Network**: YOLOv2's backbone is Darknet-19, which consists of 19 convolutional layers and 5 max-pooling layers. It replaces the original YOLO’s simpler network with a deeper and more complex network for better feature extraction.

- **Convolutional Layers**: Darknet-19 uses a combination of 3 × 3 and 1 × 1 convolutions. The 1 × 1 convolutions serve as bottleneck layers to reduce dimensionality before applying 3 × 3 convolutions, improving efficiency and reducing the number of parameters.

- **Batch Normalization**: All convolutional layers in YOLOv2 use batch normalization. This helps in stabilizing the learning process and speeding up convergence.

- **High Resolution Classifier**: YOLOv2 introduces a high-resolution classifier to improve detection accuracy. The network is initially trained on a low-resolution dataset and then fine-tuned on a higher resolution dataset to capture finer details.

- **Anchor Boxes**: YOLOv2 introduces anchor boxes to handle multiple object shapes better. It uses k-means clustering on the training data to determine the anchor box sizes. Each grid cell predicts multiple bounding boxes using these pre-defined anchor sizes.

- **Direct Location Prediction**: YOLOv2 refines the prediction of bounding box coordinates. It predicts the bounding box offsets relative to anchor boxes, improving localization.

#### Training:

1. **Pre-training and Fine-tuning**:
   - YOLOv2 is first trained on ImageNet for classification tasks and then fine-tuned on object detection tasks with a dataset like COCO.
   - The network is initially trained with low-resolution images (e.g., 224 × 224) and then fine-tuned with higher-resolution images (e.g., 448 × 448) to improve detection performance.

2. **Unified Loss Function**:
   - YOLOv2 employs a loss function that combines localization, confidence, and classification losses.

   The overall loss function is a weighted sum of:
   - **Localization Loss**:
     Measures the error in predicted bounding box coordinates. It is similar to YOLOv1 but includes adjustments for anchor boxes.
     $$
     \text{Localization Loss} = \lambda_{\text{coord}} \sum_{i=0}^{S^2} \sum_{j=0}^{B} \mathbb{1}_{ij}^{\text{obj}} \left( \left( x_i - \hat{x}_i \right)^2 + \left( y_i - \hat{y}_i \right)^2 + \left( \sqrt{w_i} - \sqrt{\hat{w}_i} \right)^2 + \left( \sqrt{h_i} - \sqrt{\hat{h}_i} \right)^2 \right)
     $$
   - **Confidence Loss**:
     Measures the accuracy of objectness score and intersection over union (IoU) between predicted and ground truth boxes.
     $$
     \text{Confidence Loss} = \sum_{i=0}^{S^2} \sum_{j=0}^{B} \mathbb{1}_{ij}^{\text{obj}} \left( C_i - \hat{C}_i \right)^2 + \lambda_{\text{noobj}} \sum_{i=0}^{S^2} \sum_{j=0}^{B} \mathbb{1}_{ij}^{\text{noobj}} \left( C_i - \hat{C}_i \right)^2
     $$
   - **Classification Loss**:
     Measures the difference between predicted and actual class probabilities.
     $$
     \text{Classification Loss} = \sum_{i=0}^{S^2} \mathbb{1}_{i}^{\text{obj}} \sum_{c \in \text{classes}} \left( p_i(c) - \hat{p}_i(c) \right)^2
     $$

3. **Data Augmentation**:
   - YOLOv2 employs various data augmentation techniques, including random cropping, scaling, and hue adjustments, to make the model robust to variations in input data.

#### Inference:

- **Bounding Box Predictions**:
  Each grid cell predicts multiple bounding boxes (using anchor boxes), class probabilities, and confidence scores.
  
  The final prediction for each bounding box is given by:
  $$
  \text{Box Score} = Pr(Object) \times \text{IOU}_{\text{pred}}^{\text{truth}}
  $$
  This score is combined with class probabilities to determine the final detection.

- **Non-Maximum Suppression**:
  After bounding box predictions are made, non-maximum suppression (NMS) is applied to eliminate redundant and overlapping bounding boxes, ensuring that each object is detected once.

#### Performance:

YOLOv2 achieves better accuracy and speed compared to YOLOv1. It can detect objects from a larger number of classes (up to 9000 with YOLO9000) and handles small objects better due to its improved architecture and anchor boxes.

#### Limitations:

- **Small Object Detection**:
  YOLOv2 still struggles with detecting very small objects or objects in dense scenes.
- **Class Prediction**:
  YOLOv2 has constraints due to the grid-based prediction and anchor boxes, which can limit its ability to predict overlapping or closely packed objects accurately.


YOLOv2 (also called YOLO9000) is a significant improvement over YOLOv1. Here’s a breakdown:

1. Architecture

	•	Backbone: YOLOv2 uses a custom architecture called Darknet-19, which has 19 convolutional layers and 5 max-pooling layers. It replaces YOLOv1’s simpler architecture and improves feature extraction.
	•	Batch Normalization: Introduced after each convolutional layer, this helps regularize the network, leading to faster convergence and better accuracy.
	•	Anchor Boxes: Instead of predicting the exact coordinates directly, YOLOv2 uses anchor boxes (similar to Faster R-CNN) to predict offsets, improving localization accuracy for objects of varying sizes. these anchor boxes are selected by kmeans clustering in the training data to learn better priors compared to heuristics.

2. Loss Function

	•	Bounding Box Loss: Similar to YOLOv1 but uses anchor boxes. YOLOv2 predicts the offsets relative to these predefined anchor boxes.
	•	Objectness Loss: A binary cross-entropy loss that evaluates whether a grid cell contains an object.
	•	Class Prediction Loss: Softmax cross-entropy loss used to predict class probabilities for each anchor box. If YOLOv2 is operating in the YOLO9000 mode (predicting 9,000 classes), hierarchical softmax is used.

The total loss is a combination of these terms, and YOLOv2 tries to balance localization, classification, and objectness.

3. Training

	•	Pretraining: The network is pretrained on the ImageNet dataset to learn robust feature extraction in the Darknet-19 backbone.
	•	Multi-Scale Training: During training, YOLOv2 randomly resizes input images to various scales (320x320 to 608x608). This teaches the network to be more robust to different image sizes and improves detection at multiple scales.
	•	Data Augmentation: Techniques like random cropping, flipping, and color jittering are used to improve generalization and reduce overfitting.

4. Dataset Curation

	•	VOC & COCO: YOLOv2 was primarily trained on the PASCAL VOC and MS COCO datasets. These datasets provide a variety of object categories and scenarios.
	•	YOLO9000 (Additional Training): YOLOv2 was extended to YOLO9000, which trained on both detection data (like COCO/VOC) and classification data from ImageNet, allowing the model to predict over 9,000 object classes. This was achieved using a hierarchical classification structure to predict rare or unseen categories.

5. Inference

	•	Detection Pipeline: During inference, YOLOv2 resizes input images to a fixed size (e.g., 416x416) and passes them through the network. The network outputs a grid of predictions where each grid cell predicts several anchor boxes. Predictions are filtered using non-maximum suppression (NMS) to remove redundant boxes and refine object locations.
	•	Speed: YOLOv2 retains YOLOv1’s fast inference speed, operating at around 40–45 FPS on a standard GPU for 416x416 resolution inputs, while improving detection accuracy compared to YOLOv1.

6. Limitations

	•	Localization Accuracy: Although anchor boxes improve localization, YOLOv2 still struggles with small objects compared to more advanced models like Faster R-CNN.
	•	Class Imbalance: Predicting a large number of classes (as in YOLO9000) introduces class imbalance issues. While the hierarchical softmax helps, it’s still difficult for the model to predict rare classes accurately.
	•	Accuracy vs. Speed Tradeoff: YOLOv2 prioritizes speed, which results in lower accuracy for highly complex scenarios with overlapping objects, crowded scenes, or tiny objects.

7. Improvements Over YOLOv1:

	• More accurate: Thanks to the use of anchor boxes and the Darknet-19 backbone, YOLOv2 achieves higher mean Average Precision (mAP) compared to YOLOv1.
	•	Better multi-scale detection: Multi-scale training allows YOLOv2 to detect objects of various sizes more effectively.
	•	YOLO9000: YOLOv2 can generalize to 9,000 classes, even in scenarios where only limited detection data is available, making it more versatile for real-world applications.





# Multistage Detectors

## Concepts:

#### Bounding Box Regression

Given a predicted bounding box coordinate $\mathbf{p} = (p_x, p_y, p_w, p_h)$ (center coordinate, width, height) and its corresponding ground truth box coordinates $\mathbf{g} = (g_x, g_y, g_w, g_h)$, the regressor is configured to learn scale-invariant transformation between two centers and log-scale transformation between widths and heights. All the transformation functions take $\mathbf{p}$ as input.

$$
\hat{g}_x = p_w d_x(\mathbf{p}) + p_x
$$

$$
\hat{g}_y = p_h d_y(\mathbf{p}) + p_y
$$

$$
\hat{g}_w = p_w \exp(d_w(\mathbf{p}))
$$

$$
\hat{g}_h = p_h \exp(d_h(\mathbf{p}))
$$

![Image illustrating transformation between predicted and ground truth bounding boxes.|300x200](https://lilianweng.github.io/posts/2017-12-31-object-recognition-part-3/RCNN-bbox-regression.png)

An obvious benefit of applying such transformation is that all the bounding box correction functions, $d_i(\mathbf{p})$ where $i \in \{x, y, w, h\}$, can take any value between \(-\infty, +\infty\). The targets for them to learn are:

$$
t_x = (g_x - p_x)/p_w
$$

$$
t_y = (g_y - p_y)/p_h
$$

$$
t_w = \log(g_w/p_w)
$$

$$
t_h = \log(g_h/p_h)
$$

A standard regression model can solve the problem by minimizing the SSE loss with regularization:

$$
L_{\text{reg}} = \sum_{i \in \{x,y,w,h\}} (t_i - d_i(\mathbf{p}))^2 + \lambda \|\mathbf{w}\|^2
$$

The regularization term is critical here and RCNN paper picked the best λ by cross validation. It is also noteworthy that not all the predicted bounding boxes have corresponding ground truth boxes. For example, if there is no overlap, it does not make sense to run bbox regression. Here, only a predicted box with a nearby ground truth box with at least 0.6 IoU is kept for training the bbox regression model.


#### **Hard Negative Mining**

We consider bounding boxes without objects as negative examples. Not all the negative examples are equally hard to be identified. For example, if it holds pure empty background, it is likely an “_easy negative_”; but if the box contains weird noisy texture or partial object, it could be hard to be recognized and these are “_hard negative_”.

The hard negative examples are easily misclassified. We can explicitly find those false positive samples during the training loops and include them in the training data so as to improve the classifier.

---

## Overfeat - [2014]

- 2014: [paper](https://arxiv.org/pdf/1312.6229) [code](https://github.com/sermanet/OverFeat)
- lilian blog section [link](https://lilianweng.github.io/posts/2017-12-15-object-recognition-part-2/#overfeat),
- andrewng [lecture](https://www.coursera.org/learn/convolutional-neural-networks/lecture/6UnU4/convolutional-implementation-of-sliding-windows)

Overfeat  is a pioneer model of integrating the object detection, localization and classification tasks all into one convolutional neural network. The main idea is to (i) do image classification at different locations on regions of multiple scales of the image in a sliding window fashion, and (ii) predict the bounding box locations with a regressor trained on top of the same convolution layers.

The Overfeat model architecture is very similar to [AlexNet](https://lilianweng.github.io/posts/2017-12-15-object-recognition-part-2/#alexnet-krizhevsky-et-al-2012). It is trained as follows:

![The training stages of the Overfeat model.](https://lilianweng.github.io/posts/2017-12-15-object-recognition-part-2/overfeat-training.png)

1. Train a CNN model (similar to AlexNet) on the image classification task.
2. Then, we replace the top classifier layers by a regression network and train it to predict object bounding boxes at each spatial location and scale. The regressor is class-specific, each generated for one image class.
   - Input: Images with classification and bounding box.
   - Output: (xleft,xright,ytop,ybottom), 4 values in total, representing the coordinates of the bounding box edges.
   - Loss: The regressor is trained to minimize l2 norm between generated bounding box and the ground truth for each training example.

At the detection time,

1. Perform classification at each location using the pretrained CNN model.
2. Predict object bounding boxes on all classified regions generated by the classifier.
3. Merge bounding boxes with sufficient overlap from localization and sufficient confidence of being the same object from the classifier.

---

## **Deformable Parts Model (DPM)** - [2008, 2010]

**Key Contributions**:
DPM introduced the idea of modeling objects as collections of deformable parts. The model uses a combination of root filters (representing the whole object) and part filters (smaller parts of the object) that can shift slightly to handle deformations.

**Architecture**:
The object model consists of a coarse root filter (covering the object) and several part filters (for finer details). DPM uses HOG (Histogram of Oriented Gradients) features for object representation and relies on a star-structured graph model to model relationships between the root and parts.

**Training**:
Training uses a latent SVM (Support Vector Machine). Initially, only the root filter is trained. Once the object location is identified, part positions are added as latent variables, and the entire model is jointly optimized. The final prediction combines responses from both root and part filters.

**Limitations**:
DPM's handcrafted features and optimization through SVM are computationally expensive and don't generalize well compared to deep learning methods.

---

## R-CNN family

### **RCNN (Regions with CNN features)** - [2014]

[rcnn paper](https://arxiv.org/abs/1311.2524)

![[Screenshot 2024-08-15 at 8.07.05 PM.png|convnet is the same (but each region is passed separately), while reg and svms are per class artifacts|600x300]]

**Key Contributions**:
RCNN was a breakthrough in applying deep learning to object detection. It proposed using a pre-trained CNN to extract features from region proposals and then classify them using a linear SVM.

**Architecture**:
RCNN starts with region proposals (using Selective Search: [paper](https://ivi.fnwi.uva.nl/isis/publications/2013/UijlingsIJCV2013/UijlingsIJCV2013.pdf), [explained here](https://www.geeksforgeeks.org/selective-search-for-object-detection-r-cnn/)) and warps each region to a fixed size (e.g., 224x224). These regions are passed through a CNN (e.g., AlexNet or VGG) pre-trained on ImageNet to extract features. The extracted features are then classified using a linear SVM for object class and refined for bounding box localization.

**Training**:The training process is multi-step:

1. The CNN is trained on ImageNet for image classification.

   ![[Screenshot 2024-08-15 at 7.54.02 PM.png|400x200]]
2. For each region proposal, image is warped to cnninput size, then CNN features are extracted, and SVM classifiers and bbox regressors are trained per class.

   ![[Screenshot 2024-08-15 at 7.54.11 PM.png|400x200]]
   ![[Screenshot 2024-08-15 at 7.54.17 PM.png|400x200]]
   ![[Screenshot 2024-08-15 at 7.54.24 PM.png|400x200]]
   ![[Screenshot 2024-08-15 at 7.54.30 PM.png|400x200]]
3. A bounding box regressor is trained separately to refine predicted box coordinates.
   ![[Screenshot 2024-08-15 at 7.54.37 PM.png|400x200]]

**Limitations**:

- Slow at test-time: need to run full forward pass of CNN for each region proposal
- SVMs and regressors are post-hoc: CNN features not updated in response to SVMs and regressors
- Complex multistage training pipeline

---

### **Fast R-CNN** - [2015]

[fast-rcnn paper](https://arxiv.org/abs/1504.08083)]

**Key Contributions**:
Fast R-CNN addresses RCNN’s inefficiency by sharing computation. It uses a single CNN to extract a feature map for the entire image, then classifies each region of interest (RoI) using a RoI pooling layer.

**Architecture**:
Fast R-CNN processes the whole image through a CNN (e.g., VGG16) to generate a feature map. For each region proposal, the RoI pooling layer extracts a fixed-size feature, which is then fed to fully connected layers for classification and bounding box regression.

![[Screenshot 2024-08-15 at 8.54.51 PM.png|500x250]]

![[Screenshot 2024-08-15 at 8.57.49 PM.png|500x250]]

**Training**:
Fast R-CNN uses a single-stage training procedure. A multi-task loss function is optimized, combining classification and bounding box regression. End-to-end training allows for more efficient learning and faster inference.

**Limitations**:
Although it improved speed significantly, Fast R-CNN still relies on an external region proposal algorithm (e.g., Selective Search), which is slow.

---

### **Faster R-CNN** - [2016]

[faster rcnn paper](https://arxiv.org/abs/1506.01497)

**Key Contributions**:
Faster R-CNN replaces the external region proposal mechanism with a Region Proposal Network (RPN), making the process fully end-to-end and significantly faster.

**Architecture**:
Faster R-CNN uses a two-stage network. First, an image is passed through a CNN to generate a feature map. Then, the RPN takes the feature map and predicts objectness scores and bounding box proposals. These proposals are then passed to an RoI pooling layer, followed by fully connected layers for final classification and box regression.

**Training**:
The RPN and the detection network share the same CNN backbone. The network is trained end-to-end with a multi-task loss that jointly optimizes object classification and bounding box regression.

**Limitations**:
While Faster R-CNN is faster than its predecessors, it is still relatively slow for real-time applications due to its two-stage nature.

---

### **Mask R-CNN** - [2017]

[mask rcnn (segmenatation) paper](https://arxiv.org/abs/1703.06870)

**Key Contributions**:
Mask R-CNN extends Faster R-CNN by adding a third branch for pixel-level segmentation, enabling instance segmentation along with object detection.

**Architecture**:
Mask R-CNN builds on Faster R-CNN, adding a small fully convolutional network branch to predict masks for each RoI, in parallel with classification and bounding box regression branches. The network uses RoIAlign instead of RoIPooling for better pixel alignment.

**Training**:
Mask R-CNN is trained end-to-end with a combined loss function. The loss includes terms for object classification, bounding box regression, and mask prediction. The mask branch is a binary classification task, predicting whether each pixel in the RoI belongs to the object.

**Limitations**:
Mask R-CNN is computationally heavier than Faster R-CNN due to the additional mask prediction branch, making it less suitable for real-time applications.

---

***Summary of Models in the R-CNN family***

Here I illustrate model designs of R-CNN, Fast R-CNN, Faster R-CNN and Mask R-CNN. You can track how one model evolves to the next version by comparing the small differences.

![](https://lilianweng.github.io/posts/2017-12-31-object-recognition-part-3/rcnn-family-summary.png)

# Resources

- Deeplearning.ai course object recognition section: [link](https://www.coursera.org/learn/convolutional-neural-networks/home/week/3)
- Lilian blog object recognition series part-3 RCNN family [link](https://lilianweng.github.io/posts/2017-12-31-object-recognition-part-3) [The whole series is great!!]
- cs291 winter15-16 lecture 8 [link](https://cs231n.stanford.edu/slides/2016/winter1516_lecture8.pdf)
- cs291 2024 lecture-9 [link](https://cs231n.stanford.edu/slides/2024/lecture_9.pdf)
