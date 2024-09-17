# TACO
TACO: Temporal Automated Captions for Observations

## Overview
This dataset comprises pairs of time-series data and their corresponding descriptive texts. These descriptive texts were automatically generated from 1.2 million real-world sensor data series and are designed to describe general, domain-independent characteristics of time-series data. To achieve this, we designed and implemented a novel method for generating descriptive texts from time-series data.

## Relation to Prior Datasets
Several datasets with pairs of time-series data and descriptive texts are known [^1], [^2]. However, descriptions contained in these datasets are either domain-specific or limited to a few basic characteristics of the time-series data. In contrast, the TACO dataset contains domain-independent diverse descriptive texts for each time-series data.

## Method for Making the TACO dataset
### 1. Define time-series classes
Referring to Imani et al. [^3], we defined 21 general characteristics of time-series data as time-series classes. The time-series classes are "Rising", "Falling", "Constant", "Convex", "Concave", "Linear", "Nonlinear", "Smooth", "Noisy", "Simple", "Complex", "Spiky", "Dropout", "Periodic", "Aperiodic", "Symmetry", "Asymmetry", "Step", "NoStep", "High amplitude", and "Low amplitude".

### 2. Define procedures for identifying time-series classes in each time-series data
We defined procedures for identifying time-series classes within each dataset. By assigning these classes to the time-series data, descriptive texts can be generated based on the assigned classes. To identify the time-series classes in each dataset, we prepared sets of score calculation methods and thresholds for each class, which are detailed below.

#### Rising: 
Assigned to time-series data that has a rising trend. Specifically, if the correlation between the timestamps and signals in time-series data exceeds **0.80**, the data is assigned the time-series class "Rising."
 
#### Falling: 
Assigned to time-series data that has a decreasing trend. If the correlation between the timestamps and signals in time-series data is below **-0.80**, the data is assigned the time-series class "Falling."

#### Constant:
Assigned to time-series data that exibits a constant trend. By "constant", we refer to situations where the functions or systems generating the time-series data remain static during the data generation process. Therefore, values including sudden spikes or dropouts can also be considered as constant. 
To implement the score calculation method, we followed these steps:
1. Median Filtering:    
    - Apply a median filter with a filter size of 5% of the length of the time-series data to the original time-series.  
  
2. Segmentation:    
    - Split the time-series into segments. These segments are generated with a window size of 4% of the data length, strided by 2% of the data length.  
  
3. Score Calculation:    
    - Calculate the Wasserstein distance (WSD) between the split segments and output the average value as the final score.    
    - If the output score is greater than **-0.04**, the time-series data is assigned the time-series class "Constant".
  
#### Convex & Concave:
Assigned to time-series data that has a convex or concave shape. Score calculation was conducted in the following steps:

1. Segment the Time-Series Data:
    - Split the time-series data into segments using two different window sizes:    
        - split with windows size of **50%** of the data length, strided by **12.5%** of the window length.    
        - split with windows size of **75%** of the data length, strided by **12.5%** of the window length.   
  
2. Calculate scores for each segment:    
    - For each segmented part as well as the whole time-series, perform the following steps:    
        1. Calculate the MSE between the original time-series data and the data fitted with a linear (first-order) function (`mse_1`).    
        2. Calculate the MSE between the original time-series data and the data fitted with a quadratic (second-order) function. "Convex" or "Concave" is determined by the sign of the coefficient of the quadratic term when fitting the quadratic function (`mse_2`).    
        3. Compute the score for each segment by:  
  
        $$  
        \text{Score} = k \times mse_2 - mse_1
        $$  
  
        \( k \) is set to four.    
        4. If the distance between the position of the extremum of the fitted quadratic function and the center of the time-series data is larger than **0.05** times the length of the time-series data, output zero instead.  
  
3. Output the Maximum Value:    
    - Output the maximum value of the values calculated in step 2. A time-series data is assigned the class "Convex" or "Concave" if the calculated score exceeds **0.01**.  
  
#### Linear & Nonlinear:
1. Median Filtering:    
    - Apply a median filter to the original time-series data with a filter size of **2.5%** of the data length.  
  
2. Linear Fit:    
    - Fit a linear function using the median filtered data.  
  
3. Score Calculation:    
    - Calculate the MSE between the median filtered data and the fitted linear function. If the MSE is smaller than **0.0035**, assign the class "Linear". If the MSE is larger than **0.02**, assign the class "Nonlinear".

#### Smooth:
1. Convolution with a Kernel:    
    - Convolve the original data using a kernel with a length of **0.02** times the length of the time-series data.    
    - All elements of the kernel are 1 / (length of the kernel).  
  
2. Score Calculation:    
    - Calculate the MSE between the convolved (smoothed) data and the original data.    
    - Assign the class "Smooth" if the MSE is smaller than **0.0006**.  

#### Noisy:
1. Apply Median Filter:    
    - Apply a median filter to the original data with a filter size of **2.5%** of the length of the original data.  
  
2. Subtract Median Filtered Data:    
    - Subtract the median filtered data from the original data to obtain the residual data.  
  
3. Apply Median Filter to Residual Data:    
    - Apply a median filter to the residual data with a filter size of **2.5%** of the length of the data.
    - If the value is larger than **0.0001**, the class "Noisy" is assigned to the time-series data.
  
#### Simple & Complex:
1. Apply Median Filter:    
    - Apply a median filter to the original data with a filter size of **2.5%** of the length of the original data.  
  
2. Calculate First Order Difference:    
    - Calculate the first order difference for the median filtered data.  
  
3. Segment the Time-Series Data:    
    - Split the output time-series into segments with a window size of **4%** of the length of the time-series data, strided by **2%** of the data length.  
  
4. Calculate WSD between segments:    
    - For each segment, calculate the WSD between that segment and the segments that are next to it.  
    - Take the average value of these WSDs.  
    - If the average WSD is larger than **0.14**, assign the class "Complex" to the time-series data.  
    - If the average WSD is smaller than **0.055**, assign the class "Simple" to the time-series data.
  
#### Spiky & Dropout:
1. Segment the Time-Series Data:    
    - Segment the time-series data using a window size of **10%** of the length of the time-series data with no overlap.  
  
2. Set Threshold for Each Segment:    
    - For each segment, set a threshold by:  
        - Calculating the median value of the segment with a filter size of **2%** of the length of the time-series data.  
        - Calculating the standard deviation of the median filtered signal.  
        - Multiplying the standard deviation by a constant value to obtain an adjustment value.  
        - Setting the threshold by adding the adjustment value to the median value of the segment when calculating scores for the "Spiky" class.  
        - Setting the threshold by subtracting the adjustment value from the median value of the segment when calculating scores for the "Dropout" class.  
  
3. Calculte scores based on the threshold:    
    - For each segment, calculate the maximum difference between the original time-series data and the threshold.  
    - If the maximum difference exceeds **0.10**, assign the time-series data the class "Spiky" or "Dropout":  
        - Assign the class "Spiky" if the threshold was set by adding the adjustment value.  
        - Assign the class "Dropout" if the threshold was set by subtracting the adjustment value.


#### Periodic & Aperiodic:
1. Fit a Linear Function:    
    - Fit a linear function to the original signal and calculate the fitting error by the MSE ('mse_linear').  
  
2. Create a Periodic Curve:    
    - Calculate the autocorrelation coefficient for the original signal.  
    - Detect the negative points of the second difference of the autocorrelation coefficient as peaks.  
    - Create a periodic signal by treating the segment up to the peak point as one complete cycle and then repeating this cycle.  
  
3. Calculate the Error of the Periodic Curve:    
    - Calculate the fitting error between the periodic curve and the original signal ('mse_periodic').  
  
4. Compare the Errors:
    - Subtract the 'mse_linear' from the 'mse_periodic'.
    - If the value is smaller than **0.01**, the time-series data is assigned the class "Periodic". If the value is larger than **0.01**, the time-series data is assigned the class "Aperiodic".
  

#### Symmetry & Asymmetry:
1. Generate Padded Original Signals:    
    - Pad the original data at the front with the first value of the data and padding widths ranging from 1 to length of the data to create multiple padded data.
    - Likewise, pad the original data at the back with the last value of the data and padding widths ranging from 1 to length of the data.
  
2. Generate Flipped Signals:    
    - For each padded original data, generate a flipped signal by left-right flipping the padded data.  
  
3. Calculate Errors:    
    - For each pair of padded original data and its flipped signal, calculate the maximum error between them.  
  
4. Find Minimum Errors:    
    - Calculate the minimum error among the errors calculated for each pair.
    - If the minimum error is smaller than **0.15**, the time-series data is assigned the class "Symmetry". If the value is larger than **0.70**, the data is assigned the class "Asymmetry".
  
#### Step & Nostep:
1. Prepare Kernels:    
    - The kernel sizes are **5%**, **10%**, **15%**, and **20%** of the length of the time-series data.  
    - Each kernel has **-1** for the first half and **1** for the second half.  
  
2. Convolve the Data with Each Kernel:    
    - For each kernel, convolve the time-series data.  
    - During convolution, if the difference between the maximum value and the minimum value of the segment being convolved is larger than **0.15**, the output is the convolved value. Otherwise, the output is zero.  
  
3. Select the Largest Convolved Value:    
    - Among the convolved values obtained using different kernels, take the largest convolved value as the final output.
    - If the largest convolved value is larger than **0.90**, assign the class "Step". If the largest convolved value is smaller than 0.60, assign the class "Nostep".

#### High amplitude & Low amplitude:
1. Split the Time-Series Data into Segments:  
    - The window size is one-third of the length of the time-series data.  
    - The stride length is **2.5% **of the length of the time-series data.  
  
2. Calculate Variance for Each Segment:  
    - For each segment, calculate the variance.  
  
3. Calculate the Maximum Variance:  
    - Find the maximum variance among the segments.  
    - If the maximum variance is larger than **0.15**, assign the class "High amplitude".  
    - If the maximum variance is smaller than **0.03**, assign the class "Low amplitude".  


### 3. Generate descriptive texts based on the assigned time-series classes
We generated descriptive texts based on the combination of these classes. We first defined pairs of time-series classes and corresponding base
captions. For example, for the class “Rising”, the base caption is “The signal has a rising trend”. Likewise, for “Smooth”, the base caption is “The signal has a smooth shape”. Then, we prepared base descriptive texts for each time-series data by combining the base captions corresponding to the time-series classes assigned to that data. For instance, for a time-series data with classes “Rising” and “Smooth”, the base descriptive texts are “The signal has a rising trend. The signal has a smooth shape”. Finally, we rephrased the base descriptive texts using the Llama-3-8B-Instruct [^4] to generate descriptive texts.


[^1]: H. Jhamtani and T. Berg-Kirkpatrick, "Truth-Conditional Captioning of Time Series Data," in Proc. Conference on Empirical Methods in Natural Language Processing (EMNLP), 2021, pp. 719–733. [URL](https://aclanthology.org/2021.emnlp-main.55.pdf)

[^2]: A. Spreafico and G. Carenini, "Neural Data-Driven Captioning of Time-series Line Charts," AVI '20: Proceedings of the 2020 International Conference on Advanced Visual Interfaces, 2020, pp. 1-5. [URL](https://dl.acm.org/doi/10.1145/3399715.3399829)





[^3]: S. Imani, S. Alaee, and E. Keogh, "Putting the human in the time series analytics loop," in Proc. World Wide Web Conference (WWW), 2019, pp. 635–644. [URL](https://doi.org/10.1145/3308560.3317308)

[^4]: https://huggingface.co/meta-llama/Meta-Llama-3-8B-Instruct
