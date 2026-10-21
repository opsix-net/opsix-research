# Reading Images by Their Frequencies

A photograph of a brick wall looks simple until you need to remove a repeating camera pattern without erasing the mortar lines. In the pixel view, both are just changes in brightness. A Fourier transform offers another view: it describes how brightness varies at different spatial frequencies and directions. That perspective can make certain image operations easier to design, but it does not automatically tell you which changes matter. You still need to choose a filter, understand what it removes, and inspect the result.

This article develops the central ideas behind Fourier-based image processing: what a frequency representation means, how filtering relates to convolution, and why transform-based restoration needs care. It also works through a small numerical example whose arithmetic can be checked by hand.

## From pixel locations to spatial frequencies

An image in the spatial domain is a grid of intensity values. A grayscale image might be written as $f[x,y]$, where $x$ and $y$ identify a pixel. Neighboring values that change gradually form smooth regions. Abrupt changes, such as an edge, involve faster spatial variation.

The two-dimensional Fourier transform represents an image as a combination of sinusoidal patterns. Each frequency component has a magnitude and phase. Magnitude indicates the component's strength; phase helps determine how its pattern aligns with the image. Frequency also has direction: variation can be rapid horizontally, vertically, diagonally, or along a combination of directions.

For a finite image of width $N$ and height $M$, the discrete transform is commonly written as

$$
F[u,v] = \sum_{x=0}^{N-1}\sum_{y=0}^{M-1} f[x,y] e^{-j2\pi(ux/N+vy/M)}.
$$

Here $u$ and $v$ index discrete frequency bins, and $j$ is the imaginary unit. Software libraries use particular normalization conventions, so a transform followed by its matching inverse reconstructs the input, but the scale of intermediate coefficients depends on the convention. The continuous transform is a useful mathematical model; actual digital images are finite arrays and use a discrete transform such as the DFT, often computed efficiently with an FFT algorithm.

A constant image has energy at zero frequency, called the DC component. Slowly varying regions tend to contribute strongly at low frequencies. Fine texture and sharp transitions contribute at higher frequencies. This is a useful rule of thumb, not a label attached to each pixel. A sharp edge contains a range of frequencies, and noise can occupy many frequencies.

## What a spectrum display does and does not show

The raw transform places zero frequency at a corner in many common array layouts. A display function such as `fftshift` rearranges the coefficients so zero frequency appears in the center. This is convenient for viewing and for constructing filters centered on low frequencies.

A magnitude image often plots $\log(1+|F[u,v]|)$ because coefficients can vary greatly in size. The logarithm makes weaker components easier to see. It discards phase, however, and therefore is not enough to reconstruct the image. A spectrum plot is a diagnostic visualization, not the transformed image itself.

If the input image is real-valued, its Fourier coefficients have conjugate symmetry. Informally, frequency components occur in paired locations. A filter intended to produce a real-valued output should preserve that symmetry. Symmetric masks around the centered frequency origin usually do so, subject to correct handling of array boundaries and even or odd dimensions.

## Filtering: multiply in one domain, convolve in the other

Frequency-domain filtering multiplies each coefficient by a filter response:

$$
G[u,v] = H[u,v]F[u,v].
$$

Taking the inverse transform of $G$ gives the filtered image. A low-pass filter keeps or emphasizes a region near zero frequency, which tends to smooth rapid changes. A high-pass filter suppresses low frequencies and retains faster variation, which can make edges prominent while also emphasizing noise. A band-pass filter retains a selected range, useful when a texture or repeating pattern occupies a known range of frequencies. A band-stop filter suppresses a selected range, which can help with narrow periodic interference.

The convolution theorem explains why frequency methods are connected to familiar spatial filters: for a finite DFT, circular spatial convolution corresponds to coefficient-wise multiplication in frequency, while spatial multiplication corresponds, up to normalization, to circular convolution in frequency. Linear convolution requires suitable padding. For large kernels or images, FFT-based convolution can be efficient. For small kernels, a direct spatial operation can be simpler and faster. The choice depends on image size, kernel shape, and implementation details.

A filter mask is not just a switch labeled “good pixels” and “bad pixels.” A hard-edged mask can introduce ringing artifacts in the spatial result. Smooth responses, such as Gaussian-shaped low-pass filters, attenuate frequencies gradually and often reduce such artifacts. Filter shape, cutoff, boundary assumptions, and image size all affect the output.

## Worked example: an eight-by-eight stripe image

Consider an 8-by-8 grayscale image that is constant in the vertical direction and repeats across each row:

$$
f[x,y] = 10 + 4\cos(2\pi x/4), \qquad 0\le x,y<8.
$$

The cosine samples across one row are $1, 0, -1, 0$, repeated twice. Thus the pixel values are $14, 10, 6, 10, 14, 10, 6, 10$. The mean is $10$, since the deviations $4,0,-4,0,4,0,-4,0$ sum to zero.

Use the unnormalized forward DFT convention in the equation above. There are 64 pixels, so the constant part contributes $10\times64=640$ to the DC coefficient $F[0,0]$. The cosine repeats every four pixels, so it has horizontal frequency-bin index $u=8/4=2$, along with its conjugate partner at $u=8-2=6$. It is constant vertically, so $v=0$.

The cosine coefficient at each of those two bins is $4\times64/2=128$. One way to verify the factor is that a real cosine splits equally between its positive and negative complex-exponential components. Therefore, the only nonzero coefficients are $F[0,0]=640$, $F[2,0]=128$, and $F[6,0]=128$.

Now apply an ideal low-pass filter that retains only the DC coefficient. Under the matching inverse DFT, the output at every pixel is $640/64=10$. The image becomes uniformly gray, so the stripes disappear. If the filter instead retains the DC coefficient and both bins at indices 2 and 6, the inverse reconstructs the original values. If it removes only the DC component, the result is the zero-mean stripe pattern: $4,0,-4,0$, repeated. These outcomes follow directly from the coefficients, not from a visual guess.

This tiny example illustrates the promise and limitation of frequency filtering. The periodic stripes are easy to isolate because their frequency is cleanly separated from the mean. A real image usually contains overlapping content. Removing a frequency that represents unwanted interference may also remove legitimate image structure at the same frequency.

## Restoration is an inference problem

Deconvolution attempts to reverse blur. If an observed image is modeled as a sharp image convolved with a blur kernel, then in frequency space the observation is approximately the sharp image's spectrum multiplied by the kernel's spectrum. A tempting inverse is to divide by the kernel response. But if that response is near zero, division can amplify noise dramatically. Practical restoration therefore often requires regularization, a noise model, or prior knowledge about the image.

The chapter passage introduces deconvolution as a way to restore blurred images. The important engineering qualification is that restoration is not guaranteed simply by applying an inverse transform. The blur and noise model matter, and the recovered image may be uncertain. Filtering can reveal or suppress patterns; it cannot recover information that was never captured without assumptions.

## Where this perspective is useful

In microscopy, frequency inspection can expose periodic illumination artifacts or help characterize repeated structures. In remote sensing, filters can separate broad land-cover variation from finer texture, while careful validation guards against removing meaningful features. In industrial inspection, a known repeating sensor pattern may appear as localized peaks in a spectrum. In computer vision, Fourier tools can support blur analysis, texture characterization, and efficient convolution.

The same reasoning transfers to engineering data beyond images. Any sampled signal can be examined for periodic components, but the axes and units must be interpreted correctly. For images, frequency is measured in cycles per pixel unless pixel spacing is supplied. If horizontal and vertical pixel sizes differ, a circular mask in pixel-index coordinates does not correspond to a circular cutoff in physical units.

## Assumptions and practical pitfalls

A discrete Fourier transform treats its finite input as one period of a repeating signal. Opposite image borders are therefore implicitly adjacent. A mismatch at those borders can create spectral leakage, spreading energy across bins. Padding, windowing, or boundary-aware spatial methods may help, though each changes the analysis and has tradeoffs.

Color images need an explicit strategy. Transforming channels independently is possible, but the meaning of a filter may differ between color spaces. Converting to grayscale simplifies the task but discards color information. Integer image data also needs sensible conversion and range handling before filtering; inverse transforms can have small imaginary round-off components, and output values may need clipping or rescaling. Taking an absolute value of a reconstructed image is not a universal substitute for handling those details.

Finally, test a filter on synthetic patterns and representative real data. Compare the input and output at the same display scale, inspect difference images, and record parameters. A smoother or sharper-looking result is not automatically a more accurate measurement.

## Exercises with short answers

1. An image's mean brightness increases while its fine texture stays unchanged. Which transform component must change? **Answer:** The DC component changes; the texture-related coefficients may remain the same.

2. Why can a high-pass filter make a noisy image look worse? **Answer:** Noise often contains rapid spatial variation, so retaining high frequencies can retain or emphasize it.

3. A periodic interference pattern has the same frequency as a real repeating structure. Can a frequency mask distinguish them by frequency alone? **Answer:** No. They overlap in frequency, so other information or a different method is needed.

Fourier methods are most useful when treated as a way to ask precise questions about spatial variation. The transform gives a new representation; filter design, boundary assumptions, and validation determine whether that representation leads to a trustworthy result.

---

Published 2026-10-21.

**Source:** Understanding Fourier Transforms with Python, section **Chapter 6: Fourier Transform in Image Processing**.

This is an original explanatory note; the source book is not redistributed.
