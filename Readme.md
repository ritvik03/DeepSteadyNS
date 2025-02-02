# DeepSteadyFlows
Computational Fluid Dynamics (CFD) simulation by the numerical solution of the Navier-Stokes equations is an essential tool in a wide range of applications from engineering design to climate modeling. However, the computational cost and memory demand required by CFD codes may become very high for flows of practical interest, such as in aerodynamic shape optimization. This expense is associated with the complexity of the fluid flow governing equations, which include non-linear partial derivative terms that are of difficult solution, leading to long computational times and limiting the number of hypotheses that can be tested during the process of iterative design. Therefore, I propose DeepSteadyFlows: a convolutional neural network (CNN) based model that efficiently approximates solutions for the problem of non-uniform steady laminar flows. The proposed model is able to learn complete solutions of the Navier-Stokes equations, for both velocity and pressure fields, directly from ground-truth data generated using a state-of-the-art CFD. It leverages the techniques of machine learning along with Navier-Strokes based loss function for steady state for faster convergence. Using DeepSteadyFlows, we found a speedup of up to 3 orders of magnitude compared to the standard CFD approach at a cost of low error rates.

[Presentation Video | Youtube](https://youtu.be/M8JX5Lyayzg)

## Architecture

![architecture](image_assets/architecture.png)

- The architecture is a UNet based architecture with single channel (128x64x1) input scaled between 0 (object) to 1 (empty space).

- This is first scaled to (64x64x8) by passing through 8 convolutional filters with strides (2,1) followed by various encoding layers . Activation function used is Leaky_ReLU (0.2)

- This feature vector then passes through various other encoding layers with stride of (2,2) till the final bottleneck of shape (1x1x512)

- Decoding layer sequence is just opposite of that of encoding layers which upscale the feature vector using Conv2DTranspose with stride (2,2) back to shape of 64x64x8. Activation function used is ReLU

- This decoded feature vector is then passed through final 3 Conv2DTranspose filters to give steady state u, v, p fields

- Each decoding layer has a concatenation operation by corresponding encoding layer using skip connection. This helps in sharing featurewise information between input and output thereby preserving geometrical features and appropriately transforming according to them. Skip connections also help in tackling the problem of vanishing gradients in deeper networks

## Loss function

Loss function is a combination of mean absolute error (MAE) and the steady-state navier-strokes loss function based on the equation mentioned below

<!-- ![continuity-eq](image_assets/continuity.png)
![ns-eq](image_assets/ns_equation.png) -->


<img src="image_assets/continuity.png" height="100px">
<img src="image_assets/ns_equation.png" height="130px">

Partial Differential terms and laplacian transform are computed using tensor gradient formulation from tensorflow library

**MAE** : mean of all absolute errors in prediction of pixel values 

**Steady-state-Navier-loss** : e<sub>x</sub><sup>2</sup> + e<sub>y</sub><sup>2</sup>

- <strong>e<sub>x</sub></strong> : Error in steady state navier-stokes equation in X-direction

- <strong>e<sub>y</sub></strong> : Error in steady state navier-stokes equation in Y-direction

- <strong>continuity_loss</strong> : Square of error in continuity equation in 2 dimensions

### Final loss function :
Loss = MAE +  λ * steady_state_navier_loss + continuity_loss

For a simpler formulation, I have kept the body force fields to 0 and lamda is set to 1

**[Note]:** More tinkering with network architecture and loss hyperparameters required. Also, generalize this to 3D

[Link to trained model](https://drive.google.com/drive/folders/13U5BLoyLj9x_DjnVSvy8scluNLNI0-JM?usp=sharing)

![model](model.png)

## Training
- **Filter count:** (8, 16, 64, 128, 512, 512) (512) (512,512, 128, 64, 16, 8) (3)
- **Optimizer:** Adam (LR = 0.01)
- **Dropout:** 0.2
- **Validation split:** 0.2
- **Metrics of evaluation:** MAE, MAPE, Cosine_proximity

![loss](logs_training_stats/loss.png)
![cos](logs_training_stats/cosine_proximity.png)
<!-- ![mape](logs_training_stats/mape.png) -->

## Results
Following are few of the cherry-picked results obtained

![1](image_assets/1.png)
![2](image_assets/2.png)
![3](image_assets/3.png)
![4](image_assets/4.png)

## Usage
**Generating custon figures**
- run draw.py and draw the outline of shape
- press "m" to fill the shape
- press "escape (esc)" to save the input 

**Prediction of custom figure**
- run predict.py 

[Link to Demo Video | Youtube](https://www.youtube.com/watch?v=aIged107FCk)

![demo](image_assets/demo_mtp2.gif)

## Custom Results
Ignoring some image artifacts, the custom results look fairly realistic too with navier strokes based loss in the ranges of 0.004, hence can be said to be a good approximation of steady state simulations in significantly less time

![6](custom_outputs/semi_circle2.png)
![5](custom_outputs/rectangle2.png)
![4](custom_outputs/plus2.png)
![3](custom_outputs/motorbike.png)
![2](custom_outputs/mandir.png)
![1](custom_outputs/bullet.png)
![0](custom_outputs/v.png)

## Credits
This work is based on an earlier of [DeepCFD](https://arxiv.org/abs/2004.08826). Few of the unused code and insipiration is from the [github repository](https://github.com/mdribeiro/DeepCFD) of the same paper although the architecture and loss functions are completely changed for better convergence. The data used to train the model is also provided in the repository. [Link to data here](https://zenodo.org/record/3666056/files/DeepCFD.zip?download=1) 

## About me
This repository is made by Ritvik Pandey for the academic use as Maters Thesis Project (MTP) for Department of Mechanical Engineering, Indian Institute of Technology Kharagpur (IIT-KGP)

Contact: ritvik.pandey03@gmail.com
