**Parallel Bitonic Sorting on the GPU by a Compute Shader**

This is a plugin for the Unreal Engine that realises parallel sorting on the GPU via compute shaders. It sorts three-dimensional vectors according to their distance to the current camera position. Currently only textures with maximum size of 1024\*1024 are supported, that is, 1 Million points can be sorted in total at maximum.

The texture/problem size has to be set manually in "ComputeShaderUsageExample.h" and "BitonicSortingKernelComputeShader.usf":

```CPP
#define BITONIC_BLOCK_SIZE 1024
#define TRANSPOSE_BLOCK_SIZE 16
```
```CPP
const UINT NUM_ELEMENTS = 1024 * 1024;
const UINT BITONIC_BLOCK_SIZE = 1024;
```

To sort a dataset with the compute shader, the compute shader has to be used as follows:

```CPP
mComputeShader = new FComputeShader(1.0f, GetUpperPowerOfTwo(pointsPerAxis), GetUpperPowerOfTwo(pointsPerAxis), currentWorld->Scene->GetFeatureLevel());

// Send unsorted point position data and point color data to compute shader
mComputeShader->SetPointPosDataReference(mPointPosDataPointer);
mComputeShader->SetPointColorDataReference(mColorDataPointer);

// Execute parallel sorting compute shader
mComputeShader->ExecuteComputeShader(FVector4(currentCamPos));
```

Furthermore, the created textures have to be converted to usable textures via a pixel shader:
```CPP
mPixelShader = new FPixelShader(FColor::Green, currentWorld->Scene->GetFeatureLevel());

// Render sorted point positions to render target for the material shader
mPixelShader->ExecutePixelShader(mComputeShaderRT, mComputeShader->GetSortedPointPosTexture(), FColor::Red, 1.0f);
mCastedRT = Cast<UTexture>(mComputeShaderRT);

// Render sorted point colors to render target for the material shader
mPixelShader->ExecutePixelShader(mComputeShaderRT2, mComputeShader->GetSortedPointColorsTexture(), FColor::Red, 1.0f);
mCastedColorRT = Cast<UTexture>(mComputeShaderRT2);
```

To see the plugin in action, see my point cloud renderer plugin for UE4: https://github.com/ValentinKraft/UE4_GPUPointCloudRenderer/tree/WithComputeShaderSort_4.18
A detailed tutorial might follow as well.
Tested in Unreal 4.18.

[![Demo](https://img.youtube.com/vi/jVZaz-0Y4Ek/0.jpg)](https://www.youtube.com/watch?v=jVZaz-0Y4Ek)
