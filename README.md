
Convolutional Neural Network Applications on Surface-Based Temporospatial Neuroimaging Data
===========================================================================================



Semih Cantürk<sup>1</sup>, Cassiano Becker<sup>1</sup>, Edward Benjamin Fineran<sup>2</sup> and Victor M. Preciado<sup>1</sup>

**1** University of Pennsylvania, Department of Electrical and Systems Engineering (Philadelphia, PA, United States)


**2** University of Pennsylvania, Department of Computer and Information Science (Philadelphia, PA, United States)

## 1. Abstract

In statistical learning approaches to pattern detection and recognition, one of the most successful methods is the convolutional neural network (CNN). CNN’s are known for their ability of positional invariance, the ability to recognize and relate patterns regardless of their positions in the visual space through localized filters. CNNs are adept at pattern recognition in 2D and hence images; however they are not directly applicable to problems with irregular data structures and extra dimensions both in time and space, the type of data that is commonly found in representation and analysis of brain activity as in temporospatial fMRI imaging. In order to extend the pattern recognition capabilities of CNNs to temporospatial brain data analysis, we propose a mesh CNN (mCNN) adapted to  semi-regular mesh data structures neuroimaging data is commonly represented on. The mCNN borrows approaches from graph theory to apply vertex domain mesh convolutions, paralleling the conventional convolution operation in CNNs but extending it to further dimensions in time and representation space. We then applied the mCNN on a motor classification task from the Human Connectome Project (HCP), where subjects were given visual cues to perform basic movements as their fMRI activations were measured. The preliminary results of the study suggest that the mCNN is able to attain comparable performance with conventional MLP’s with significantly less data, and is likely to surpass the already significant success of the MLP approach.

## 2. Introduction

Convolutional neural networks (Krizhevsky et al. 2012; LeCun & Bengio 1998) have been the leading method in solving pattern recognition problems in the recent years. CNNs use of shared filters render them location invariant, meaning they can detect and recognize local feature regardless of their positions in the visual field. The conventional CNN approaches are also translatable into three dimensions (3D) in cases where the detection of spatial and/or temporal relations in the third dimension is trivial; such is the case with pattern detection in dynamic images and 3D volume. However, a trivial translation is not possible in dynamic 3D polygon meshes, where the problem at hand involves irregular polygonal surface structures in 3D space as well as temporal relations over the 3D space.

In neuroimaging, a common and interpretable way of visualizing cerebral activity is through the mapping of functionality data to the brain morphometry domain. This mapping is usually done in three dimensions (3D) by modeling the 3D geometry of the cortical surface through polygon meshes and projecting the activations on the model. The 3D surface representations has been used to visualize cortical thickness (Fischl & Dale 2000), electroencephalography (EEG) (Heinonen et al. 1999) and functional magnetic resonance imaging (fMRI) (Chen & Li 2012). However, most of these brain functionality visualizations serve solely as interpretability tools for medical professionals, and are not used in mathematical and statistical analysis of the brain functionality. As a result, the applications of machine learning methods, and particularly of CNNs which can make use of the visualized spatiotemporal relations, on 3D mesh representations of brain activity are limited in number of scope. While methods to apply CNNs to cortical meshes have recently been proposed, the applications have been on toy discriminatory tasks. For example, (Seong et al. 1999) have used CNNs for sex discrimination based on cortical surface representations.

This article proposes a form of CNN that is adapted to pattern recognition in spatiotemporal 3D triangular mesh data, referred as the mesh CNN (mCNN); it also examines in depth the translation of spatiotemporal fMRI data from 3D cortical visualization into regular data structures that a multi-layer mCNN is able to make use of, while simultaneously preserving the spatiotemporal relations essential for the discriminative ability of the model. The cortical mesh is first treated as an icosahedral sphere, to which it is analogous in terms of both topology and triangular mesh structure. The mCNN is then designed to traverse and preprocess the mesh, and apply subsequent convolutions and pooling in order to detect detect patterns both in 3D space and in time. This mCNN is then applied on a motor task classification problem derived from the Human Connectome Project (HCP; Elam & Van Essen 2013). The mCNN is then compared in classification accuracy to a two-layer MLP, which had been used in previous work for the same discriminatory task.

<div id="MPFigureElement:FCFFB5DF-C200-4B80-CB8E-860A6C7473D5">

![alt text](./figures/FIG1.png)

</div>

## 3. Method

### 3.1. Mesh CNN 

In high-level architecture, the mCNN is analogous to conventional CNNs: it is composed of repeating convolutional and pooling layers, followed by full layers with a softmax output layer. However, the main differentiators of the mCNN from the conventional CNN are the low-level implementations of the convolutional layers and the mesh pooling layers. While the traditional convolution and pooling layers deal with 2D data in multidimensional array form which are then trivially treated as tensors, the mCNN convolution and pooling layers involve low level architectural adaptations to 3D mesh data in time series form in order to build the tensors.

<div id="MPFootnotesElement:74A7B852-E6CA-4AC2-866B-AC75A8D5A5E2">

</div>

![alt text](./figures/brain.gif)

**Figure 3:** Example of partial mesh traversal over cortical surface

#### 3.1.1. Convolution Layer 

In conventional CNNs, the convolution filters are defined by regular shapes; a square is often used due to its trivial translatability to 2D array data structures. These filters are then convolved samples from the data which are congruent, which are called “patches". This sampling operation is referred to as “striding”, where patches from data are called iteratively in a predefined order.

The mCNN defines this convolution operation as a _vertex domain mesh convolution_. The convolutions associate each data point with a patch, defined by all vertices within a specific graph distance from the point, ordered by a) graph distance from the center vertex which contains the associated data value, and b)the following ordering for vertices of the same graph distance/order:

1.  The vertex with the closest euclidean distance in 3D, vertex A, is queued first.
2.  At this point, the verticess of the same order will form a circle, with each vertex having two neighbors of the same order. Then, of its two neighbors, the one with the closest euclidean distance to vertex A is queued, determining whether the order is clockwise or counterclockwise.
3.  From then on, the order is trivial. In the direction defined in 2), all unvisited vertices are visited in order.
4.  The list of all unseen neighbors of the vertices from the current layer is delegated as the next layer.

Then, to account for the time dimension, a time window H is selected **(Figures 1, 2A)**: The convolution tensor for one example contains the patches from all vertices (ordered by vertex ID), for all time samples within the time window H. Multiple examples (270 for our motor task classification experiment) are generated from a single patient by slicing running windows of width H through the patient data, which include 284 consecutive time samples **(Figure 1A)**. The convolution tensors are constructed by flattening the patch dimension for each vertex n to form a row vector. This results in a ![alt text](./figures/eq2.svg) tensor, where l is the length of a single patch and Nis the total number of vertices in the mesh. A single HCP patient data then produces 270 of these tensors. Each filter then also assumes the ![alt text](./figures/eq3.svg) shape, with the convolutional layer possessing f number of filters. The convolution operation then, dimensionally speaking, takes place as the following:

<div id="MPEquationElement:C465A36C-87B5-45AF-BBC9-9CE1B0736758">

![alt text](figures/eq1.svg)

</div>

<div id="MPFigureElement:0371341F-7701-456D-A24A-EDAA49A57EA3">

![alt text](./figures/FIG2.png)

</div>

**Figure 1A** demonstrates the spatiotemporal nature of the data: Each vertex has a position in 3D space and is connected to neighbor vertices via edges, and also contains a vector of 284 consecutive fMRI activation values that change over time. It also includes information about the HCP experiment procedure; the activation over time plot shows when the cues are given to the subjects, and which body part was moved with each cue. **Figure 1A** is then connected to **Figure 1B** by showing how a H second window is translated into a tensor dimension. **1B** also visualizes the tensor operations in the convolution layer, while **Figure 2A** illustrates the implementation of this tensorization process in detail. **Figure 1C** then displays the high-level architecture of the mCNN, which is very similar to that of a conventional CNN.

#### 3.1.2. Pooling Layer

In the mCNN, the convolution layers are followed by the pooling layers which downsample the outputs of the convolution process. This is done in order to reduce the dimensionality of the data, curbing potential overfitting and reducing the computational cost of the mCNN. The mCNN pooling process is centered around the triangular structure of the surface mesh. **Figures 1A** and **2A** provide close-ups of the mesh structure: each vertex is connected to six other vertices, forming six triangle faces.

Pooling is done via a mapping between two cortical surface meshes which retain the same overall shape, but have a vertex ratio of 7:1, meaning that on average seven vertices from the original mesh is assigned to a single vertex in the “pooled” mesh. The pooling algorithm is as follows:

1.  The simplified pooling mesh is created via uniform mesh sampling over the original mesh with the specified vertex ratio, as shown in **Figure 2B**. This mesh is visually similar to the original mesh, but with less complexity due to the reduction in vertices.
2.  Then, a mapping is created between the “complex” and “simplified” meshes. The vertex in the complex mesh that has the most similar position in the ambient space is assigned to each vertex in the simplified mesh.
3.  Each point in the simplified mesh is then assigned the precomputed “patch” that is associated with its counterpart in the complex mesh. This results in the aforementioned seven-to-one mapping for the simplified mesh. If there are less than seven vertices in a patch, the row vector is padded with zeros (for max-pooling) or the average of the vector (for mean pooling).
4.  Each vertex in the simplified mesh is assigned the maximum or average of the seven associated vertices in the patch depending on the type of pooling, completing the pooling process.

The original surface mesh is composed of 32,492 vertices; this number reduces to approximately 5400 after the first convolution and pooling, and to 900 after the second. The exact numbers are dependent on the uniform mesh sampling done on the original mesh to generate these simpler iterations, and the current figures stand at 5356 and 914 vertices for the first two pooling layers, respectively. **Figure 2B** demonstrates this pooling process both in vertex-level and mesh-level. In the study, we tested both max pooling and mean pooling approaches, and found mean pooling to perform slightly better on average.

### 3.2. Implementation

The mCNN implementation was based on the HIPS Convnet, which in turn is a CNN built for MNIST modeled on the LeNet-5 <span class="citation" data-reference-id="MPCitation:7FA37B1B-CAA6-49C8-A405-7E883C089B4C">(LeCun et al. 1998, p. )</span>. The current architecture consists of two convolution and pooling layer pairs, followed by two dense layers and a softmax output - the architecture is shown in detail **Figure 1C**. As mentioned in Section 3.1.2, mean pooling was preferred over max pooling in the pooling layers. The network uses conventional backpropagation and batch gradient descent. It is important to note that this architecture is currently suboptimal, and several architectural changes are proposed and investigated to improve mCNN performance. This section will be updated with the appropriate changes once implemented. The proposed changes are as follows:

* Deepening the network to more than two convolution-pooling layer pairs.

* Adding normalization and ReLU units between the convolution and pooling layers.

* Further hyper-parameter optimization after the aforementioned changes are implemented: batch size, learning rate etc.

### 3.3. Applications to Motor Task Classification

The implemented mCNN was then applied to a motor task classification problem using cortical fMRI activation time series from healthy adults, obtained from the HCP database. The experiment data was collected as follows: Subjects were given cues to move different body parts (left hand, left foot, right hand, right foot, tongue) during their fMRI scanning. The timings of the cues as well as a small portion of fMRI data can be found in **Figure 1A**. The the scans then measure the hemodynamic response of discrete brain regions corresponding to each cortical mesh point, by a technique known as blood-oxygen-level dependent (BOLD) imaging. BOLD imaging is based on the differentiating the magnetic properties of oxygenated and deoxygenated blood; as active areas of the brain require more energy, resulting in different ratios of oxygenated and deoxygenated blood which are then detected by the fMRI scan.

The cortical surface possesses the same topology as a sphere, enabling direct applicability of the mCNN to the cortical surface mesh. In the HCP data, this surface mesh is icosahedral, and consists of 32,492 data points per hemisphere. The activation values for each data point are evaluated in every time step, where any two time steps are 0.72 seconds apart. Each experiment per patient involves 284 time steps. The activation data is then normalized for each point.

This classification problem was previously explored through a MLP approach. A two-layer MLP with dropout and softmax output was used, with the best performing configuration having (400, 100) vertices for the respective layers.



## 4. Bibliography

Chen, S. & Li, X., 2012. Functional magnetic resonance imaging for imaging neural activity in the human brain: The annual progress. _Computational and mathematical methods in medicine._

Elam, J.S. & Van Essen, P.D., 2013. Human Connectome Project. In _Encyclopedia of computational neuroscience._

Fischl, B. & Dale, A.M., 2000. Measuring the thickness of the human cerebral cortex from magnetic resonance images. _Proceedings of the National Academy of Sciences._

Heinonen, T., Lahtinen, A. & Häkkinen, V., 1999. Implementation of three-dimensional EEG brain mapping. _Computers and Biomedical Research._

Krizhevsky, A., Sutskever, I. & Hinton, G.E., 2012. 1 ImageNet Classification with Deep Convolutional Neural Networks. _Advances In Neural Information Processing Systems._

LeCun, Y. et al., 1998. Gradient-based learning applied to document recognition. _Proceedings of the IEEE._

LeCun, Y. & Bengio, Y., 1998. Convolutional Networks for Images, Speech, and Time Series. In _The handbook of brain theory and neural networks._

Seong, S.-B., Pae, C. & Park, H.-J., 2018. Geometric Convolutional Neural Network for Analyzing Surface-Based Neuroimaging Data. _Frontiers in Neuroinformatics._


