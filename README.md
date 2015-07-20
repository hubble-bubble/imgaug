# ImageAugmenter

The ImageAugmenter is a class to augment your training images for neural networks (and other machine learning methods).
Possible augmentations are:
* Translation (moving the image)
* Scaling (zooming)
* Rotation
* Shear
* Horizontal flipping/mirroring
* Vertical flipping/mirroring

The class is build as a wrapper around scikit-image's AffineTransform.
Most augmentations (all except flipping) are combined into one affine transformation, making the augmentation process reasonably fast.

# Requirements

* numpy
* scikit-image
* scipy (optional, for some of the tests)
* matplotlib (optional, if you want to see a plot containing various example images showing the effects of your chosen augmentation settings)

# Examples

Load an image and apply augmentations to it:
`
image = misc.imread("example.png")
height = image.shape[0]
width = image.shape[1]
augmenter = ImageAugmenter(width, height, # width and height of the image (must be the same for all images in the batch)
                           hflip=True,    # flip horizontally with 50% probability
                           vflip=True,    # flip vertically with 50% probability
                           scale_to_percent=1.3, # scale the image to 70%-130% of its original size
                           scale_axis_equally=False, # allow the axis to be scaled unequally (e.g. x more than y)
                           rotation_deg=25,    # rotate between -25 and +25 degrees
                           shear_deg=10,       # shear between -10 and +10 degrees
                           translation_x_px=5, # translate between -5 and +5 px on the x-axis
                           translation_y_px=5  # translate between -5 and +5 px on the y-axis
                           )

# augment a batch containing only this image
# the input array must have dtype uint8 (0-255)
# the output array will have dtype float32 and can be fed directly into a neural network
augmented_images = augmenter.augment_batch(np.array([image], dtype=np.uint8))
`

Set the minimum and maximum values of the augmentation parameters:
`
image = misc.imread("example.png")
width = image.shape[1]
height = image.shape[0]
augmenter = ImageAugmenter(width, height,
                           scale_to_percent=(0.9, 1.2), # scale the image to 90%-120% of its original size
                           rotation_deg=(10, 25),       # rotate between +10 and +25 degrees
                           shear_deg=(-10, -5),         # shear between -10 and -5 degrees
                           translation_x_px=(0, 5),     # translate between 0 and +5 px on the x-axis
                           translation_y_px=(5, 10)     # translate between +5 and +10 px on the y-axis
                           )

augmented_images = augmenter.augment_batch(np.array([image], dtype=np.uint8))
`

Example with a synthetic image (grayscale):
`
image = [[0, 255, 0],
         [0, 255, 0],
         [0, 255, 0]]
images = np.array([image]).astype(np.uint8)

# set width to 3, height to 3, rotate always exactly by 45 degrees
augmenter = ImageAugmenter(3, 3, rotation_deg=(45, 45))
augmented_images = augmenter.augment_batch(np.array([image], dtype=np.uint8))
`

You can pregenerate some augmentation matrices, which will later on be applied to the images (in random order).
This will speed up the augmentation process a bit, because the transformation matrices can be reused multiple times.
`
augmenter = ImageAugmenter(height, width, rotation_deg=10)
# pregenerate 10,000 matrices
augmenter.pregenerate_matrices(10000)
augmented_images = augmenter.augment_batch(np.array([image], dtype=np.uint8))
`

# Plotting your augmentations

For debugging purposes you can show/plot examples of your augmentation settings (i.e. what images look like if you apply these settings to them).
Use either the method ImageAugmenter.plot_image(image) for that or alternatively ImageAugmenter.plot_images(images, augment).
Example for plot_image:
`
image = misc.imread("example.png")
width = image.shape[1]
height = image.shape[0]
augmenter = ImageAugmenter(width, height, rotation_deg=20)
# This will show the image 50 times, each one with a random augmentation as defined in the constructor,
# i.e. each image being rotated by any random value between -20 and +20 degrees.
augmenter.plot_image(image, nb_repeat=50)
`

# Special use cases

By default the class expects your images to have one of the following two shapes:
* (y, x) for grayscale images
* (y, x, channel) for images with multiple channels (RGB)
If your images have their channel in the first axis instead of the last, i.e. (channel, y, x) then you can
use the parameter channel_is_first_axis in the ImageAugmenter's init function:
augmenter = ImageAugmenter(width, height, channel_is_first_axis=True)

# Tests

The tests and checks are in the tests/ directory.
You can run them using (from within that directory):
`
python TestImageAugmenter.py
python CheckPerformance.py
python CheckPlotImages.py
`
where CheckPerformance.py measures the performance of the class on you machine and CheckPlotImages.py shows some plots with example augmentations.
