---
title: "Canny Edge Detector - Verilog/SystemC"
description: "Image processing algorithm for edge detection implemented in both SystemC and Verilog, with SRAM and UART supplementary"
date: "2024-10-15"
author: "Vishnu Ramesh"
math: true
---

Image processing is a great application for hardware design, as typical image processing algorithms involve large amounts of data processing and math operations. This project goes over a **Canny Edge Detector**, which takes an input image and outputs an image with all the edges outlined. This is typically done by finding sharp discontinuities in an image. This algorithm has high computation, as it utilizes Noise Reduction, Gradient Calculation, Non-Maximum Suppresion, and Hysterisis Thresholding. This creates a highly accurate output image that classifies all borders present in the image.

This project was first simulated in SystemC using the Anaconda SystemC package for running simulations. The circuit was then designed in Verilog using Cadence XCelium to simulate and verify the correctness of the design. The Verilog design was also verified at the gate-level design using Synopsys Design Compiler and the [OSU standard cell library](https://www.vlsitechnology.org/html/libraries04.html). The code for this project can be found on [Github](https://github.com/vishnuvardhanRamesh/Edge-Detector).

![Starting Picture](/images/ced_starting_image.png)
![Final Picture](/images/ced_final_image.png)


## Canny Edge Circuit Algorithm
The following sections detail the specific parts of this algorithm.

### Noise Reduction
The first step involves reducing noise in the image, using a Gaussian filter. This will slightly blur the image, but will reduce *false edges* by removing high gradient points caused by random fluctuations, while keeping the true edges visible in the image. 

### Gradient Calculation

To find the edge strength, we can take the Gradient of the image. This involves using Sobel operators to perform the measurement. There are two Sobel matrices, one operating in the X direction (horizontal) and one in the Y direction (vertical). These matrices are particularly good at highlighting areas of high intensity change, corresponding to edges. The gradient can be computed using the equations below.


{{< math.inline >}}
\begin{align*}
\text{SobelX} = \begin{bmatrix}
-1 & 0 & 1 \\
-2 & 0 & 2 \\
-1 & 0 & 1 \\
\end{bmatrix} \\
\text{SobelY} = \begin{bmatrix}
-1 & -2 & -1 \\
0 & 0 & 0 \\
1 & 2 & 1 \\
\end{bmatrix}
\end{align*}
{{</ math.inline >}}
{{< math.inline >}}
\begin{equation}
    Gx[i,j] = \sum_{k=-1}^{1} \sum_{l=-1}^{1} M[i+k,j+l] * SobelX[k,l]
\end{equation}
\begin{equation}
    Gy[i,j] = \sum_{k=-1}^{1} \sum_{l=-1}^{1} M[i+k,j+l] * SobelY[k,l]
\end{equation}
{{</ math.inline >}}

Using Gx and Gy, we can then calculate the magnitude and direction. &alpha; is used for normalization when calculating magnitude. For direction, an approximation algorithm is used for computational efficiency.

```python
Magnitude = (abs(Gx) + abs(Gy)) / alpha

if (Gy < 0):
    Gx = Gx * -1;
    Gy = Gy * -1;
if (Gx >= 0):
    if (Gy <= 0.5*Gx):
        direction = 0
    elif (Gy <= 2.5*Gx):
        direction = 45
    else:
        direction = 90
else:
    if (Gy <= -0.5*Gx):
        direction = 0
    elif (Gy <= -2.5*Gx):
        direction = 135
    else:
        direction = 90  
```



### Non-Maximum Suppresion

Using the currently calculated gradients, the edges may be thick or somewhat blurry. To obtain crisp edges, the following algorithm can be used to further prune out noisy edge markings.

*Note that the below algorithm is done with respect to multiple possible edge directions (0, 45, 90, 135 degrees).*

1. Select pixel C. Find neighbor pixels A and B based on edge direction, and the magnitudes of all pixels (calculated previously)
2. If M(C) >= M(A) and M(C) >= M(B), keep pixel C and discard pixels A and B by setting their magnitudes to 0
3. Otherwise, discard pixel C by setting magnitude to 0.

### Hysteresis Thresholding

As seen above, some of the edges are not well defined, and most edges are not able to be clearly seen. The final step of this algorithm is to improve the image quality by further removing weak edges, and clearly defining connected edges.

1. Define threshold levels T_high and T_low. Strong pixels are greather than T_high, weak pixels are lower than T_low. *Candidate* pixels are between the two values.
2. Keep strong pixels and discard low pixels. Look at candidate pixels and their neighboring pixels in each direction (similar to before)
3. For candidate pixels, label as *strong* if it is connected to a strong pixel, and discard otherwise.

This is the final step of the algorithm that produces a clear image with well-defined edges.

## Conclusion

Canny Edge detection is a image processing algorithm that has higher computational complexity compared to other algorithms, due to its multiple steps. It produces a high quality image, and can be implemented on hardware for high performance. This article goes over the actual implementation of the algorithm, although one should note the complexity of implementing the datapath and controller for transferring data between memory and the image processing module. 


