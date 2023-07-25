---
layout: post
title: Vulkan Renderer
lead: Hobby project
---

The goal of my current (HEAVY WIP) [project](https://github.com/OlesenJonas/Testingine) is to create a nice to work with and flexible framework for writing graphics code, based on Vulkan.

For that i've decided to make heavy use of C++ 20's designated initializers in combination with structs for parameters as in the example below:\
(inspired by Sebastian Aaltonen's tweets, and now also part of his presentation [Modern Mobile Rendering @ HypeHype](https://enginearchitecture.realtimerendering.com/2023_conference/#:~:text=Talks-,Modern%20Mobile%20Rendering%20%40%20HypeHype,-HypeHype%E2%80%99s%20new%20renderer))

{% highlight c++ %}
auto irradianceTexHandle = resourceManager->createTexture({
    .debugName = "hdriIrradiance",
    .type = Texture::Type::tCube,
    .format = Texture::Format::r16g16b16a16_float,
    .allStates = ResourceState::SampleSource | ResourceState::Storage,
    .initialState = ResourceState::Undefined,
    .size = {irradianceRes, irradianceRes},
});
{% endhighlight %}

Other features include an archetype based [Entity Component System](https://github.com/OlesenJonas/Testingine/tree/main/src/libraries/intern/ECS):
{% highlight c++ %}
auto skybox = engine.ecs.createEntity();
{
    auto* transform = skybox.addComponent<Transform>();
    auto* renderInfo = skybox.addComponent<RenderInfo>();
    renderInfo->mesh = defaultCube;
    renderInfo->materialInstance = cubeSkyboxMatInst;
}
{% endhighlight %}

aswell as a fully bindless pipeline setup based on HLSL, which simplifies resource management and enables writing pretty sweet shader code in my opinion.
As an example, here's the compute shader for converting an equirectangular texture to a cubemap: 
{% highlight hlsl %}
#include "../Bindless/Setup.hlsl"

DefineShaderInputs(
    Handle< Texture2D<float4> > sourceTex;
    Handle< RWTexture2DArray<float4> > targetTex;
);

float2 sampleSphericalMap(float3 v){...}

[numthreads(8, 8, 1)]
void main(uint3 id : SV_DispatchThreadID)
{
    int3 size;
    RWTexture2DArray<float4> targetTex = shaderInputs.targetTex.get();
    targetTex.GetDimensions(size.x, size.y, size.z);
    
    float2 uv = float2(id.xy + 0.5) / size.xy;
    uv.y = 1.0 - uv.y;
    uv = 2.0 * uv - 1.0;

    float3 localPos;
    if(id.z == 0)
        localPos = float3(1.0, uv.y, -uv.x); // pos x
    else if(id.z == 1)
        localPos = float3(-1.0, uv.y, uv.x); // neg x
    else if(id.z == 2)
        localPos = float3(uv.x, 1.0, -uv.y); // pos y
    else if(id.z == 3)
        localPos = float3(uv.x, -1.0, uv.y); // neg y
    else if(id.z == 4)
        localPos = float3(uv.x, uv.y, 1.0); // pos z
    else if(id.z == 5)
        localPos = float3(-uv.x, uv.y, -1.0); // neg z

    float3 color =
        shaderInputs.sourceTex.get().SampleLevel(
            LinearRepeatSampler,
            sampleSphericalMap(normalize(localPos)),
            0
        ).xyz;

    targetTex[id] = float4(color, 1.0);
}
{% endhighlight %}