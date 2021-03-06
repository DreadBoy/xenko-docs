# Material attributes

<span class="label label-doc-level">Intermediate</span>
<span class="label label-doc-audience">Artist</span>
<span class="label label-doc-audience">Programmer</span>

**Material attributes** define the core characteristics of the material, such as its diffuse color, diffuse shading model, and so on.

![media/material-attributes-1.png](media/material-attributes-1.png)

There are two types of attribute:

- attributes used as input values for a shading model (for example, the Diffuse attribute provides only color used by the diffuse shading model)
- attributes that can change the shading model (for example, the Diffuse Shading Model [like Lambert] interprets the diffuse attribute color)

Attributes contribute to a layer of a material. If a material is directly used as a model material, all its root attributes are considered part of the first layer.

A material needs to have at least one declared attribute shading model to display something:

- Diffuse Model (with Diffuse color)
- Specular Model (With Specular color, possibly Diffuse Color and MicroSurface)
- Emissive

Attributes are organized in three categories: geometry, shading, and misc.

## Geometry attributes

Geometry attributes define the shape of the material:

- tessellation
- displacement
- surface (macro surface definition like normal maps)
- microSurface (micro surface definition like gloss maps)

### Tessellation

Real-time tessellation uses a HW feature of the GPU to massively subdivide triangles. This increases the realism and potential of deformations of the surface geometry.

| No tessellation  | Flat tessellation | Point normal tessellation 
| --------------  | -------------- | -------------------- 
| ![media/material-attributes-2.png](media/material-attributes-2.png)  | ![media/material-attributes-3.png](media/material-attributes-3.png)  | ![media/material-attributes-4.png](media/material-attributes-4.png)

#### Flat tessellation

Tessellates the mesh uniformly with a flat tessellation.

![media/material-attributes-5.png](media/material-attributes-5.png) 

The following snapshot shows that the flat tessellation is adding extra triangles to the existing triangles, while the curve isn't taken into account:

| No tessellation  | Flat tessellation  
| ---------------- | ----------------- 
| ![media/material-attributes-6.png](media/material-attributes-6.png)  | ![media/material-attributes-7.png](media/material-attributes-7.png)   

| Property               | Description      
| ---------------------- | ----------------------- |
| Triangle Size          | The size of a tessellated triangle in screen-space units        
| Adjacent Edges Average | Adjust the **triangle size** values from the average of adjacent edges values 

#### Point normal tessellation

Tessellates the mesh using the curvature provided by the normals of the mesh.

![media/material-attributes-8.png](media/material-attributes-8.png) 

The following image shows that the point normal tessellation is adding extra triangles to the existing triangles, while keeping the curvature of the mesh into account:

| No tessellation | Point normal tessellation 
| ---------------|  ---------------------- |
| ![media/material-attributes-6.png](media/material-attributes-6.png)  |![media/material-attributes-9.png](media/material-attributes-9.png)         

| Property               | Description 
| ---------------------- | ------------
| Triangle Size          | The size of a tessellated triangle in screen-space units        
| Adjacent Edges Average | Adjust the **triangle size** and **normal curvature** values from the average of adjacent edges values 

### Displacement map

The **displacement map** displaces the geometry of the mesh.

![media/material-attributes-10.png](media/material-attributes-10.png) 
 
Depending on the stage the displacement is applied, the results can be very different:

| Displacement with Vertex Shader  | Tessellation with Displacement  
| ------| ----------------- |
| ![media/material-attributes-11.png](media/material-attributes-11.png)  | ![media/material-attributes-12.png](media/material-attributes-12.png)

| Property         | Description     
| ---------------- | ------------ 
| Displacement Map | The displacement texture as a [Material Color Provider](material-maps.md) 
| Intensity        | The intensity of displacement                                         
| Scale & Bias     | When enabled, the value coming from the texture is considered as a positive value ranging from 0.0 to 1.0 and the shader will apply a scale to get the range -1.0 to 1.0
| Shader Stage     | Specify which shader stage the displacement map should be applied to, Vertex Shader or Domain Shader (used with Tessellation)

## Surface

The **surface** defines the **macro** surface normals.

![media/material-attributes-13.png](media/material-attributes-13.png) 

### Normal map

The **normal map** provides per-pixel normal perturbation of the normal of the mesh.

![media/material-attributes-14.png](media/material-attributes-14.png) 

Normal mapping ([Wikipedia page](http://en.wikipedia.org/wiki/Normal_mapping)) is widely used to enhance the realism of a low poly mesh:

| Flat | Using a Normal Map   
| -----| ----------- 
| ![media/material-attributes-15.png](media/material-attributes-15.png)  | ![media/material-attributes-16.png](media/material-attributes-16.png)  

| Property     | Description 
| ------------ | ---------------
| Normal Map   | The normal map color provider
| Scale & Bias | When enabled, the value coming from the texture is considered as a positive value ranging from 0.0 to 1.0 and the shader will apply a scale to get the range -1.0 to 1.0. 
| Normal xy    | When enabled, assume that only the (x,y) are valid and z = 1.0

### MicroSurface

Defines the behavior of the micro surface fine-grained normals.

#### Glossiness Map

A map that provides per-pixel information for glossiness.

- A value of 1.0 means the surface is highly glossy (the coarse normal isn't perturbed)
- A value of 0.0 means the surface is very rough (the coarse normal is highly perturbed in several directions)

![media/material-attributes-17.png](media/material-attributes-17.png)

The screenshots below show different levels of glossiness on a material:

- Diffuse = #848484, Lambert
- Specular Metalness = 1.0, GGX

| Glossiness = 0.0 | 0.25 | 0.5  | 0.8  | 1.0 
| -- | --- | ------- | ----------- | ----- |
| ![media/material-attributes-18.png](media/material-attributes-18.png)  | ![media/material-attributes-19.png](media/material-attributes-19.png)  | ![media/material-attributes-20.png](media/material-attributes-20.png)  | ![media/material-attributes-21.png](media/material-attributes-21.png)  | ![media/material-attributes-22.png](media/material-attributes-22.png)  

| Property       | Description
| -------------- | -- |
| Glossiness Map | The glossiness map color provider
| Invert         | Inverts the glossiness value (eg a value of 1.0 produces zero glossiness instead of maximum). This effectively turns the glossiness attribute into a **roughness** attribute, similar to other game engines

## Shading attributes

The **shading attributes** define the main color characteristics of the material and how it reacts to the lights.

> [!Note]
>To display a material, you need to select at least one shading model (diffuse, specular or emissive model) in the model attributes.

### Diffuse

The **diffuse** is the basic color of the material. A pure diffuse material is completely non-reflective and "flat" in appearance.

![media/material-attributes-25.png](media/material-attributes-25.png) 

The final diffuse contribution is calculated like this:

- the **diffuse** defines the color used by the diffuse model
- the **diffuse model** defines which shading model is used for rendering the diffuse component

Currently, the Diffuse attribute supports only a **Diffuse Map**.

![media/material-attributes-23.png](media/material-attributes-23.png)

#### Diffuse Model

The **diffuse model** determines how the diffuse material reacts to light. 

Currently, the only supported diffuse model is the **Lambert** model. Under this model, light is reflected equally in all directions with an intensity following a cosine angular distribution (angle between the normal and the light):

![media/material-attributes-24.png](media/material-attributes-24.png)

> [!Note]
> A pure Lambertian material doesn't exist in reality. A material has always a little specular reflection. This effect is more visible at grazing angles (a mostly diffuse surface becomes shiny at grazing angle).

| Property      | Description  
| ------------- | ----------- 
| Diffuse Map   | The diffuse map color provider                                           
| Diffuse Model | The shading model for diffuse lighting

### Specular

A **specular** is a point of light reflected in a material.

#### Specular Color

The specular color can be defined using two common workflows:

- Metalness workflow: The specular color is calculated by using the diffuse color as a base color.
- Specular workflow: The specular color is defined separately from the diffuse color.

#### Metalness Map

The metalness workflow is easy to use as it simplifies parametrization between the diffuse and specular color.

By taking into into account the fact that almost all materials always have some "metalness"/reflectance in them, the metalness workflow allows to provide realistic materials with a minimal and easy to grasp parametrization.

With the metalness workflow, the final specular color is calculated by mixing between a fixed low reflectance color and the diffuse color.

- When the metalness color = 0.0, the effective specular color is equal to 0.02, while the diffuse color is unchanged. It means that the material is not metal but exhibits some reflectance and is sensitive to the Fresnel effect.
- When the metalness color = 1.0, the effective specular color equal to the diffuse color, and the diffuse color is set to 0. The material is then considered as a pure metal.

![media/material-attributes-26.png](media/material-attributes-26.png) 

 In the screenshots below, using a material with the following attributes, we can see the result of the metalness factor on the material:

- Glossiness = 0.8
- Diffuse = #848484, Lambert
- Specular GGX

| Pure Diffuse (No Metalness)  | Metalness = 0.0    | Metalness = 1.0 
| ---------------------------- | ------------------ | ---------------
|  ![media/material-attributes-27.png](media/material-attributes-27.png)  | ![media/material-attributes-28.png](media/material-attributes-28.png)  | ![media/material-attributes-29.png](media/material-attributes-29.png)  |
| - The diffuse color is dominant | - The diffuse color is dominant   | - The diffuse color is not visible
| - The specular color is not visible   | - The specular color is visible (0.02) | - The specular color is visible 
| - No Fresnel:  | - Fresnel effect is slightly visible: | - Fresnel effect is visible: 
| ![media/material-attributes-30.png](media/material-attributes-30.png)   | ![media/material-attributes-31.png](media/material-attributes-31.png)  | ![media/material-attributes-32.png](media/material-attributes-32.png) 

#### Specular Map

The specular workflow provides more control on the actual specular color but requires to carefully modify the diffuse color accordingly.

Unlike the metalness workflow, It allows to have a different specular color from the diffuse color even in low reflectance scenarios, allowing some materials with special behavior.

> [!Note]
> With the layering system, it's still possible to combine a metalness and specular workflow into the same material.

#### Specular Model

A pure specular surface produces a highlight of a light in a mirror direction. In practice,a broad range of specular materials, not entirely smooth, can reflect the light not in a single direction.

The default specular model used to support these scenarios is the **microfacet** model, also known as [Cook-Torrance (academic paper)](http://www.cs.columbia.edu/~belhumeur/courses/appearance/cook-torrance.pdf).

![media/material-attributes-33.png](media/material-attributes-33.png) 

The microfacet is defined by the following formula, where Rs is the resulting specular reflectance:

![media/material-attributes-34.png](media/material-attributes-34.png) 

| Property            | Description                                                        
| ------------------- | ------- 
| Fresnel             | Defines the amount of the incoming light that is reflected and transmitted. The models supported are: <br>**Schlick**: An approximation of the Fresnel effect (default)</br> <br>**None**: Use the color of the material as-is without taking into account the Fresnel effect</br> 
| Visibility          | Defines the visibility between of the microfacets between (0, 1). Also known as the geometry attenuation - Shadowing and Masking - in the original Cook-Torrance. Xenko simplifies the formula to use the visibility term instead : <br>![media/material-attributes-35.png](media/material-attributes-35.png)</br>      <br>and <br>![media/material-attributes-36.png](media/material-attributes-36.png)</br>        <br>**Schlick GGX** (default)</br> <br> **Implicit**: The microsurface is always visible and doesn't generate any shadowing or masking</br> <br>**Cook-Torrance**</br>  <br>**Kelemen**</br> <br>**Neumann**</br> <br>**Smith-Beckmann**</br> <br>**Smith-GGX correlated**</br>  <br>**Schlick-Beckmann**</br> 
| Normal Distribution | <br>Defines how the normal is distributed. The glossiness attribute is used by this part of the function to modify the distribution of the normal.</br> <br>**GGX** (default) </br> <br>**Beckmann**</br>  <br>**Blinn-Phong**</br> 

### Emissive

An **emissive** material is a surface that emits a light.

![media/material-attributes-37.png](media/material-attributes-37.png) 

With HDR and a bloom post-processing effect, we can observe the influence of an emissive material:

![media/material-attributes-38.png](media/material-attributes-38.png) 

| Property     | Description                                                               
| ------------ | -------------- 
| Emissive Map | The emissive map color provider      
| Intensity    | A factor to multiply by the color of the color provider   
| Use Alpha    | Use the alpha of the emissive map as the main alpha color of the material (instead of using the alpha of the diffuse map by default)

## Misc attributes

### Occlusion

The Occlusion Map is the default occlusion attribute. The occlusion map use geometry occlusion information baked into a texture to modulate the ambient and direct lighting.

![media/material-attributes-39.png](media/material-attributes-39.png) 

These screenshots demonstrate the use of occlusion maps and cavity maps:

| Occlusion Map  | Cavity Map    | Final Composition    
| ------- | ------ | ------- 
| ![media/material-attributes-40.png](media/material-attributes-40.png)  | ![media/material-attributes-41.png](media/material-attributes-41.png)  | ![media/material-attributes-42.png](media/material-attributes-42.png)  
| Coarse occlusion of the ambient light  | Fine-grained occlusion of direct light  | Result                       

| Property  | Description 
| --------- | ---- 
| Occlusion Map             | The occlusion map scalar provider that determines how much ambient light is accessible on the material. A value of 1.0 means that the material is fully lit by ambient lighting. A value of 0.0 means that the material is not lighted by the ambient lighting 
| Direct Lighting Influence | Applies to Occlusion Map and influences direct lighting  |
| Cavity Map                | The cavity map scalar provider is multiplied with direct lighting. It lets you define very fine grained cavity where direct light can't enter. The cavity map is usually defined for thin concave cavity
| Diffuse Cavity            | A factor for diffuse lighting influence of the cavity map. A value of 1.0 means the cavity map fully influences the diffuse lighting 
| Specular Cavity           | A factor for specular lighting influence of the cavity map. A value of 1.0 means the cavity map fully influences the specular lighting       

### Transparency

#### Additive

The additive transparency takes into account the diffuse and diffuse/emissive alpha.

![media/material-attributes-47.png](media/material-attributes-47.png) 

- If the **Alpha** property is less than 0.5, only the specular highlights are visible. The material itself is completely invisible.
  
  | Alpha = 0.25   | Alpha = 0.5  
  | -------------- | -----------
  | ![media/material-attributes-48.png](media/material-attributes-48.png)  | ![media/material-attributes-49.png](media/material-attributes-49.png)  |      
  | We only see the specular highlight in additive mode  | Transparency is fully additive. Specular highlights at maximum 

- If the **Alpha** <= 1.0, the material is semi-opaque with the diffuse/emissive component. If the diffuse component has an alpha, it's transparent.
  
  | Alpha = 0.75 | Alpha = 1.0 
  | -------------- | ---------------------- |
  | ![media/material-attributes-50.png](media/material-attributes-50.png)  | ![media/material-attributes-51.png](media/material-attributes-51.png)          
  | Specular highlights, diffuse with alpha and semi-opaque diffuse          | Specular highlights, diffuse with alpha and opaque diffuse  

| Property | Description 
| -------- | -----------
| Alpha    | The alpha value is interpreted like this:<br> Alpha <= 0.5, the material is rendered in additive mode without the diffuse component (only specular highlights)</br> <br>Alpha <= 1.0, the material is rendered in semi-opaque mode with the diffuse/emissive component. If the diffuse component has an alpha, it's displayed as transparent</br>|
| Tint     | Apply a color tint to the transparency layer   

#### Cuttoff

Renders a material when the current alpha color is above the threshold you specify with the **Alpha** slider.

![media/material-attributes-43.png](media/material-attributes-43.png) 

The following screenshots show the influence of the cutoff Alpha value.

| Alpha = 0.01 | Alpha = 0.5     | Alpha = 1.0    
| -------------| ---- | ------ 
| ![media/material-attributes-44.png](media/material-attributes-44.png)  | ![media/material-attributes-45.png](media/material-attributes-45.png)  | ![media/material-attributes-46.png](media/material-attributes-46.png)

## Layers

See [Material layers](material-layers.md).           

## See also

* [Material maps](material-maps.md)
* [Material layers](material-layers.md)
* [Materials for developers](materials-for-developers.md)