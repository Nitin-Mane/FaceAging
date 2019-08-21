# Using CycleGANs for face aging

### This is assignment 3 of phase 2 of EIP 3.0

Data information
------------------------------------

 - The project uses the UTK_inthewild dataset for training.

 - The training data is a subset of the images in the dataset, filtered by age.

 - Two age groups are used : Young and Old. Each group includs images of people with ages between a range. This range is altered during training. For example, the age range for "Young" was [18-25] during a training phase.

 - All images are resized to 128x128 pixels. OpenCV is used to load the images. Aspect ratio is not maintained.

 - Image data is normalized between -1.0 and 1.0.  


Network information
-------------------------------------

 - Generator Networks
 	- The generators were treated as if they consisted of three parts:
 		- **Encoder**: This is a CNN that takes in a batch of 128x128x3 images and outputs a batch of vectors of shape 32x32x256, with values in range [-1, 1].

 	    - **Slide**: Feature manipulation, that I call 'Slide', in the code and rest of the document. This is a sequence of custom Residual blocks.

 	    - **Decoder**: Another CNN, consisting of Transpose Convolution layers. Outputs a batch of 128x128x3 images in range (-1, 1).

 - Discriminator networks
 	- These are CNNs that take batches of 128x128x3 image vectors with values in range [-1, 1] and output batches of 16x16x1 vectors.

 	- No activations are used in the output layer of the Discriminarors.

 - InstanceNormalization has been used in both Generators and Discriminators.

 - No pooling has been used. Convolutionl layers with strides > 1 are used instead.

 - Padding has been used in some places to maintain required shape.


Training
------------------------------------

 - Training is performed using Tensorflow, with eager execution enabled.

 - Tensorflow's `GradientTape` is used instead of Keras' `model.fit()`.

 - Loss functions have been defined manually. The loss function for generators is changed between training, as summarized later in this section.

 - Optimizers and learning rates are changed during training.

 - A batch size between 6 to 10 is used, changing based on whether the batch can be processed in Google Colab during execution.

 - Training: Phase 1
 	- All layers in the model are trainable. The model is trained useing the standard CycleGAN loss equations until _barely acceptable_ results appear.

 	- The encoder, slide and decoder are saved as separate models.

 	- Adadelta, with learning rate of 1.0 is used as the optimizer.

 - Training: Phase 2
 	- In a separate notebook, the encoder and decoder of one of the generators are trained as a regular encoder-decoder network to recreate a given input.

 	- These are then used as untrainable structures, along with trainable slides for all further training and experiments. They are used in both generators.

 	- The optimizers are changed from Adam to RMSProp, with decaying learning rate starting from 0.001.

 - Training: Phase 3
    - The encoders and decoders obtained from phase 2 are frozen and used with trainable slides.

    - For the first epoch, standard CycleGAN loss equations are used.

    - From the second epoch onwards, identity loss is removed. This was a successful approach to prevent mode collapse, which occured when using identity loss.

    - First epoch used Adadelta as the optimizer, with a learning rate of 1.0.

    - From the second epoch to a time when the generator losses started oscillating, Nadam with learing rate 3e-4 is used.

    - After that point, Adam is used as the optimizer. Learning rates are changed manually between epochs.

    - For the final few epochs, the discriminator learning rates are set to a tenth of the generators' learning rates.


Validation
------------------------------------

 - 8 fixed images of each class are used for human verification of training progress. These are taken from the training set.

 - Validation is performed at the beginning of every epoch, and after every 100 steps in every epoch.
