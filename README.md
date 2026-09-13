# PCD_Assignment01
Python Implementation code for image upscaling and downscaling methods.

Downscaling Algorithms
  Downscaling reduces image resolution by mapping a block of original pixels to a single pixel in the newly sized image. The script determines the new dimensions using integer division (//) based on a specified down_factor. It iterates through the target matrix, slicing the original image into discrete blocks defined by h_start:h_end and w_start:w_end boundaries.

1. Max Downscaling: Evaluates each segmented block and extracts the maximum intensity value for each color channel using np.max. This operation tends to preserve bright features and dilate highlights in the resulting image.

2. Mean Downscaling: Calculates the arithmetic average of all pixels within the localized block using np.mean. This algorithm acts as a localized low-pass filter, creating a smoother but potentially softer downscaled image.

3. Min Downscaling: Extracts the minimum channel value within each block using np.min. Functionally opposite to max pooling, this method preserves darker pixels and shadows.

Upscaling Algorithms
  Upscaling increases image dimensions by generating new pixels. The script calculates the new height and width by multiplying the original dimensions by an up_factor.  

1. Nearest Neighbor (nn_upscale):This is a zero-order interpolation method that performs direct, discrete mapping.  The algorithm iterates through the empty upscaled grid and maps each coordinate back to the original image using integer division **(orig_i = i // up_factor)**.  It directly assigns the exact pixel value from the corresponding coordinate in the source array to the new matrix.  While computationally inexpensive, it creates hard blocky artifacts because pixel values are simply duplicated without blending.

2. Bilinear Interpolation (bilinear_upscale):This method creates a smoother output by calculating a weighted average of a 2x2 pixel neighborhood.  It maps the output coordinate to a floating-point input coordinate (orig_i = i / up_factor).  Using floor functions (np.floor), it establishes a 2x2 bounding box containing the upper-left, upper-right, lower-left, and lower-right neighboring pixels.  The algorithm calculates the fractional spatial distances (di, dj) between the floating-point coordinate and the top-left boundary (i_low, j_low).  The final pixel value is linearly interpolated across both axes using the following formula:
**val = (v1 * (1 - di) * (1 - dj)) + (v2 * di * (1 - dj)) + (v3 * (1 - di) * dj) + (v4 * di * dj)**.

3. Bicubic Interpolation (bicubic_upscale):This algorithm evaluates a larger 4x4 pixel neighborhood (16 pixels) to generate smoother curves and sharper edges.  Unlike the nested pixel-by-pixel loops in the bilinear function, this implementation is highly vectorized via NumPy. It precomputes fractional output coordinates (x_out, y_out) and their respective base indices (x_low, y_low) for the entire image simultaneously.  It employs a helper function get_weights(x) utilizing a Keys cubic convolution filter with a defined shape parameter of a = -0.5.  The spatial weights w are calculated based on the absolute distance x:
**For x <= 1:w = (a + 2)x^3 - (a + 3)x^2 + 1**.
**For 1 < x < 2: w = ax^3 - 5ax^2 + 8ax - 4a**.
The code precomputes horizontal (wx) and vertical (wy) weight arrays for offsets ranging from -1 to 2.  Using a nested 4x4 loop, it samples the original image and applies the weights across the entire target array using NumPy broadcasting.  Because the cubic polynomial generates negative weights (causing overshoot for edge sharpening), the final array is passed through np.clip(bicubic_upscaled, 0, 255) to ensure all calculated pixel values remain within the valid 8-bit color spectrum.  
