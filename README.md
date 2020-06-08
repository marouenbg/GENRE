# GEN (GPU Elastic-Net): A MATLAB Package for Massively Parallel Linear Regression with Elastic-Net Regularization

## Table of Contents
1. [Overview](#Overview)
2. [Setup](#Setup)
3. [Input Data Format](#Input-Data-Format)
4. [Interface](#Interface)

## Overview
GEN (GPU Elastic-Net) is a MATLAB package that allows for many instances of linear regression with elastic-net regularization to be performed in parallel on a GPU. The specific objective function that is minimized is shown below.

![objective function](https://latex.codecogs.com/svg.latex?%5Cboldsymbol%7B%5Chat%5Cbeta%7D%20%3D%20%5Cunderset%7B%5Cboldsymbol%7B%5Cbeta%7D%7D%7B%5Cmathrm%7Bargmin%7D%7D%5Cfrac%7B1%7D%7B2N%7D%5Csum_%7Bi%3D1%7D%5E%7BN%7D%20%5Cleft%28%5Cboldsymbol%7By%7D_%7Bi%7D%20-%20%5Csum_%7Bj%3D1%7D%5E%7BP%7D%20%5Cboldsymbol%7BX%7D_%7Bij%7D%5Cboldsymbol%7B%5Cbeta%7D_%7Bj%7D%5Cright%29%5E%7B2%7D%20&plus;%20%5Clambda%20%5Cleft%28%20%5Calpha%20%5Cleft%5C%7C%20%5Cboldsymbol%7B%5Cbeta%7D%20%5Cright%5C%7C_%7B1%7D%20&plus;%20%5Cfrac%7B%20%5Cleft%281%20-%20%5Calpha%20%5Cright%29%5Cleft%5C%7C%20%5Cboldsymbol%7B%5Cbeta%7D%20%5Cright%5C%7C_%7B2%7D%5E%7B2%7D%7D%7B2%7D%20%5Cright%29)

In this equation, ![N](https://latex.codecogs.com/svg.latex?N) represents the number of observations, ![P](https://latex.codecogs.com/svg.latex?P) represents the number of predictors, ![X](https://latex.codecogs.com/svg.latex?%5Cboldsymbol%7BX%7D) is the model matrix containing ![P](https://latex.codecogs.com/svg.latex?P) predictors with ![N](https://latex.codecogs.com/svg.latex?N) observations each, ![y](https://latex.codecogs.com/svg.latex?%5Cboldsymbol%7By%7D) is the vector of ![N](https://latex.codecogs.com/svg.latex?N) observations to which the model matrix is being fit, ![beta](https://latex.codecogs.com/svg.latex?%5Cboldsymbol%7B%5Cbeta%7D) is the vector of ![P](https://latex.codecogs.com/svg.latex?P) model coefficients, ![lambda](https://latex.codecogs.com/svg.latex?%5Clambda) is a scaling factor for the amount of regularization that is applied, and ![alpha](https://latex.codecogs.com/svg.latex?%5Calpha) is a factor in the range [0, 1] that provides a weighting between the L1-regularization and the L2-regularization terms. In order to minimize this objective and obtain the estimated model coefficients contained in ![Beta hat](https://latex.codecogs.com/svg.latex?%5Cboldsymbol%7B%5Chat%5Cbeta%7D), the cyclic coordinate descent optimization algorithm is utilized. This involves minimizing the objective function with respect to one model coefficient at a time. Cycling through all of the model coefficients results in one iteration of cyclic coordinate descent, and iterations are performed until the specified convergence criteria are met. For GEN, the convergence criteria consist of a maximum number of allowed iterations and a tolerance for the maximum coefficient change between iterations of cyclic coordinate descent. For example, if the maximum number of iterations has been performed or if no coefficient changes by at least as much as the specified tolerance between iterations, then the algorithm will stop. Now, when minimizing the objective function with respect to one model coefficient at a time, the following update is obtained for the model coefficient, where ![S](https://latex.codecogs.com/svg.latex?S) is a soft-thresholding function. Moreover, 
![estimated y](https://latex.codecogs.com/svg.latex?%5Cboldsymbol%7B%5Chat%7By%7D%7D%5E%7B%28j%29%7D) represents the estimated value of ![y](https://latex.codecogs.com/svg.latex?%5Cboldsymbol%7By%7D) leaving the current predictor out.

![update](https://latex.codecogs.com/svg.latex?%5Cboldsymbol%7B%5Chat%5Cbeta%7D_%7Bj%7D%20%5Cgets%20%5Cfrac%7BS%28%5Cfrac%7B1%7D%7BN%7D%5Csum_%7Bi%3D1%7D%5E%7BN%7D%5Cboldsymbol%7BX%7D_%7Bij%7D%28%5Cboldsymbol%7By%7D_%7Bi%7D%20-%20%5Cboldsymbol%7B%5Chat%7By%7D%7D_%7Bi%7D%5E%7B%28j%29%7D%29%2C%20%5Clambda%5Calpha%29%7D%7B%5Cfrac%7B1%7D%7BN%7D%5Csum_%7Bi%3D1%7D%5E%7BN%7D%5Cboldsymbol%7BX%7D_%7Bij%7D%5E%7B2%7D%20&plus;%20%5Clambda%281%20-%20%5Calpha%29%7D)

![soft thresholding function](https://latex.codecogs.com/svg.latex?S%28z%2C%20%5Cgamma%29%20%3D%20%5Cbegin%7Bcases%7D%20z%20-%20%5Cgamma%20%26%20%5Ctext%7Bif%20%24z%20%3E%200%24%20and%20%24%5Cgamma%20%3C%20%7Cz%7C%24%7D%20%5C%5C%20z%20&plus;%20%5Cgamma%20%26%20%5Ctext%7Bif%20%24z%20%3C%200%24%20and%20%24%5Cgamma%20%3C%20%7Cz%7C%24%7D%20%5C%5C%200%20%26%20%5Ctext%7Bif%20%24%5Cgamma%20%5Cgeq%20%7Cz%7C%24%7D%20%5Cend%7Bcases%7D)

The description provided above describes the process of performing one model fit, but GEN allows for many of these fits to be performed in parallel on the GPU by using the CUDA parallel programming framework. GPUs have many computational cores, which allows for a large number of threads to execute operations in parallel. In the case of GEN, each GPU thread handles one model fit. For example, if 100 individual model fits need to be performed, then 100 computational threads will be required. Performing the fits in parallel on a GPU rather than in a sequential fashion on a CPU can potentially provide a significant speedup in terms of computational time (speedup varies depending on the GPU that is utilized).

## Setup
In order to utilize GEN, a CUDA-capable NVIDIA GPU along with an available release of MATLAB is required. The MATLAB Parallel Computing Toolbox must also be installed if not already installed in order to allow for the compilation of MEX-files containing CUDA code. Moreover, a C/C++ compiler that is compatible with the installed release of MATLAB must be installed in order to compile MEX-files containing C/C++ code. The compiler compatibility can be found at https://www.mathworks.com/support/requirements/supported-compilers.html. Note that the code was evaluated on both Windows and Linux OS. For Windows, the free community edition of Microsoft Visual Studio 2017 was used as the C/C++ compiler. To download this older version, go to https://visualstudio.microsoft.com/vs/older-downloads/ and create a free Dev Essentials program account with Microsoft. When installing Microsoft Visual Studio 2017, make sure to also check the box for the VC++ 2015 toolset (the 2015 will most likely be followed by a version number). For Linux, the GNU Compiler Collection (GCC) was used as the C/C++ compiler. In addition to a C/C++ compiler, a CUDA toolkit version that is compatible with the installed release of MATLAB must be installed. To determine compatibility, refer to https://www.mathworks.com/help/parallel-computing/gpu-support-by-release.html. Once the compatibility is determined, go to https://developer.nvidia.com/cuda-toolkit-archive and install the particular CUDA toolkit version. Note that the installation process for the toolkit will also allow for the option to install a new graphics driver. If you do not desire to install a new driver, then you must ensure that your current driver supports the toolkit version that is being installed. For driver and toolkit compatability, refer to page 4 of https://docs.nvidia.com/pdf/CUDA_Compatibility.pdf.

Before compiling the code, you should first check to see that MATLAB recognizes your GPU card. To do so, go to the command prompt and type ```gpuDevice```. If successful, the properties of the GPU will be displayed. If an error is returned, then possible causes will most likely be related to the graphics driver or the toolkit version that is installed. Once the GPU is recognized, the next step is to compile the MEX-files that contain the C/CUDA code. Assuming the code repository is already on your system, go to the MATLAB directory that contains the repository folders and add them to your MATLAB path. For Windows OS, type the following commands into the MATLAB command prompt.

```Matlab
cd GEN_GPU_Single_Precision_Code
mexcuda GEN_GPU_single_precision.cu
cd ..\GEN_GPU_Double_Precision_Code
mexcuda GEN_GPU_double_precision.cu
```

The same commands can be used for Linux OS, but the path to the CUDA toolkit library must also be included. This is illustrated by the following commands.

```Matlab
cd GEN_GPU_Single_Precision_Code
mexcuda GEN_GPU_single_precision.cu -L/usr/local/cuda-10.0/lib64
cd ../GEN_GPU_Double_Precision_Code
mexcuda GEN_GPU_double_precision.cu -L/usr/local/cuda-10.0/lib64
```

Note that there might be differences in your path compared to the one shown above, such as in regards to the version of the CUDA toolkit that is being used. In addition, if desired, the ```-v``` flag can be included at the end of each mexcuda command to display compilation details. If the compilation process is successful, then it will display a success message for each compilation in the command prompt. In addition, a compiled MEX-file will appear in each folder. The compilation is process is important, and it is recommended to recompile any time a different release of MATLAB is utilized.

## Input Data Format
As previously stated, GEN allows for many models to run in parallel on the GPU. The data for each model fit needs to be saved as a ```.mat``` file. For example, if there are 100 model fits that need to be performed, then there should be 100 ```.mat``` files. Each file should contain 3 variables that are called ```X```, ```y```, and ```intercept_flag```. ```X``` is the model matrix, ```y``` is the vector that contains the observed data to which the model matrix is being fit, and ```intercept_flag``` is a flag (either 0 or 1) that indicates whether the model matrix includes an intercept term. Note that if an intercept term is desired, then the first column of ```X``` needs to be a column vector of ones, and ```intercept_flag``` should be set to 1. However, if an intercept term is not desired, then the column vector of ones should not be included in ```X```, and ```intercept_flag``` should be set to 0. All of the ```.mat``` files should be saved in a directory. In terms of the naming convention of the files, the code assumes that the file for the first model fit is called ```model_data_1.mat```, the second file is called ```model_data_2.mat```, and so on. However, if desired, this naming convention can be changed by modifying the way the ```filename``` variable is defined in the ```data_organizer.m``` script. Note that GEN allows for either single precision or double precision to be utilized for the model fit calculations. However, the input data for each model fit can be saved as either single or double data type. For example, if the variables in the files are saved as double data type, the model fits can still be performed using either single precision or double precision because GEN converts the input data to the precision that is selected for the model fit calculations before it is passed to the GPU. The model coefficients that GEN returns are converted to the same data type as the original input data. This means that if the data in the model files is saved as double data type and single precision is selected for the GPU calculations, then the returned model coefficients will be converted to double data type.

## Interface
GEN consists of several files. The main program is the ```GEN.m``` script, and it is the only file that the user will need to modify. The inputs to this script are described in detail below.

```precision```: Specifies which precision to use for the model fit calculations on the GPU. ```precision = 'single'``` is recommended due to the fact that there is typically a performance penalty when using double precision on GPUs due to there being fewer FP64 units than FP32 units. Moreover, using double precision requires more memory resources in terms of storage and bandwidth because one value of type double is 64 bits while one value of type single is 32 bits.

```num_fits```: The number of model fits to perform.

```data_path```: Path to the directory containing the model data files described in the previous section.

```save_path```: Path to the directory where the output file containing the computed model coefficients will be saved to.

```output_filename```: The name of the output file containing the computed model coefficients.

```alpha_values_h```: A vector containing ![alpha](https://latex.codecogs.com/svg.latex?%5Calpha) for each model fit.

```lambda_values_h```: A vector containing ![lambda](https://latex.codecogs.com/svg.latex?%5Clambda) for each model fit.

```tolerance_values_h```: A vector containing the tolerance convergence criterion value for each model fit (values such as 1E-4 or 1E-5 are reasonable). For example, if set to 1E-4, then cyclic coordinate descent will stop if the maximum model coefficient change across all of the coefficients between iterations of cyclic coordinate descent is less than this number. Essentially, the model coefficient change between iterations is calculated for each coefficient, and the maximum change across the coefficients is compared to the tolerance value.

```max_iterations_values_h```: A vector containing the maximum number of iterations convergence criterion for each model fit (100,000 iterations is reasonable because we typically expect the tolerance criterion to be met first).

```transformation_flag```: A flag that specifies which transformation option to use for the model fits. ```transformation_flag = 1``` means that each predictor column in the model matrix for each model fit will be standardized on the GPU. The mean of each predictor column is subtracted off from each observation in the column, and each observation in the column is then divided by the standard deviation of the column. Note that the ![One over N](https://latex.codecogs.com/svg.latex?%5Csmall%201/N) variance formula is used when calculating the standard deviation similar to the glmnet software package. Once the model fits are performed, the coefficients are unstandardized before they are returned due to the fact that the original model matrices were unstandardized. A column vector of ones corresponding to an intercept term must be included in every model matrix in order to select this option. The intercept term is not standardized. ```transformation_flag = 2``` means that each predictor column in the model matrix for each model fit will be normalized on the GPU. Each observation in each predictor column will be divided by doing ![normalization](https://latex.codecogs.com/svg.latex?%5Cfrac%7B%5Cboldsymbol%7BX%7D_%7Bij%7D%7D%7B%5Csqrt%7B%5Csum_%7Bi%3D1%7D%5E%7BN%7D%5Cboldsymbol%7BX%7D_%7Bij%7D%5E%7B2%7D%7D%7D)
