# Physics and Math of Shading

Naty Hoffman (2K) -- Siggraph 2018

Going from the Physics that underly shading to the math that describes it.

## 1. What is light from a physics standpoint?

It is an electromagnetic transverse wave, that means that the electric and magnetic fields propagate forward at normal angles of each other. These two fields are coupled, and can be characterized by:

* Frequency: Number of wavelengths (wiggles) they do in a second.
* Wavelength: The distance between two peaks of the waves.

These wavelengths of light range from Gamma rays (0.1nm wavelength) to Extreme low frequency radio waves (ELF) with wavelengths of 10s of thousands of km.

The range  we can actually see with out eyes is only from 400nm for violet light and 700nm for red light.

So to give a relative size:

* It is even smaller than a strand of spider silk which is about 1 micron in length. 
* Hair is 80 microns 
* Something that is very very tiny but not impossibly to understand tiny.

So far we have imagined our waves to be very simple, to have something like just one wavelength. This is not the case in real life for most light. Probably only true for stuff like laser lights. 

In real life most lights contain many different wavelengths with different amounts of energy in each.

To visualize this we use a **spectral power distribution** (SPD).

--- 

Now imagine that you have 3 SPD's one for red, green and blue that are added together into a new SPD. Now this kind of SPD is what you would see in laser projectors. The actual wave would still be rather simple since it is just the addition of three sine waves.

Most light that you actually look at it nature is not just peaks but a very hilly SPD with a very chaotic wavefront.

#### However even though these two SPD's are different, they result in the same color appearance to humans.

Human colorvision is very lossy. It maps infinite spd to just a 3 dimensional space.

--- 

Going back to the waveform, in a vacuum a wave would continue infinitely. But for rendering we only care about what happens when an electro magnetive wave interacts with matter.

When a wave hits a bunch of atoms or molecules it will polarize them: 

* It stretches and separates the positive and negative charges and generates dipoles
* This absorbs energy from the incoming wave and this energy is then radiated back out
* Kind of like a spring, with the dipoles snapping back
* This energy is sometimes just lost to heat but sometimes all of it is preserved as new waves going in all directions.
  

In a thin sparse gas molecules are far apart to the point that they can be treated individually and they are fairly simple formulas to understand what is happening.

But in more complex and dense matter the dipoles and waves interfere and interact with each other and the whole thing is two complex to accurately simulate.

## 2. Physical (wave) Optics

To tame this complexity it was necessary to simplify and get approximations to perform simulations. 

For example:

* They assume the medium that light travels in is homogeneuous.
* This is obviously an abstraction since matter compoosed of matter cannot be homogeneous at all scales.
* In practice this is good enough for materials with uniform density and composition.

### Homogeneous medium optical properties

These are enclosed in the index of refraction, a complex number with two parts. One encloses the speed of light through the medium, the other describes how much light is absorbed by the medium.

However, you do want to model some spaces with are not homogeneous and wave optics does this by assuming the existance of scattering particles. These are abrupt Index of refraction (IOR ) discontinuities that will scatter the incoming light over various directions.

Similar to individual molecule polarization that we discussed above but instead thinking about many molecules instead of just one atom or particle.

### Classifying the overall appearance of the medium

We can use two properties to classify the medium:

* Absorption (color)
* Scattering (cloudiness)
* Can be thought of as two independent axes in a plot

If a liquid is coloured it indicates it absorbs light much more easily in some wavelengths than in others.

---
Although we have talked a bit about participating media the interest of this talk is object surfaces

From an optical perspective the most important thing about a surface is its roughness.

#### No surface can be perfectly flat, at the atomic level there will always be some irregularities. For irregularities that are smaller than a wavelength we can call them Nanogeometry and these cause diffraction.

We can look at the Huygens-Fresnel Princpile that explains diffraction:

Each point on a planar wave is the center of a new spherical wave that is being emitted. These spherical waves will interfere with each other and create new planar waves. 

This mental picture helps when the wave hits an obstacle, which is when you get diffraction. You can imagine like one side of the spherical wave being diffracted and another would continue as a planar wave. This is because some of the speherical waves would not be cancelled by the other spherical waves clsoe to them since they have been occluded by the geometry.

The effect of this will be to slightly soften shadows.

What is more interesting for reflectance is diffraction on a surface that has irregularities in the nanogeometry.

For this case, I am going to look at an optically smooth surface. That is, a surface that has irregularities smaller than an optical wavelength. 

Not hard to polish to this degree!! (pretty cool)

We can then apply this huygens principle to surfaces that have this kind of super small irregularities. And oyu would see the same effects that are expected with diffraction and constructive and destructive interferance that would result in a complexly structured wave. With some of the light scattered in all kinds of directions. 

Obviously, the smaller the nanogeometry the less diffraction there will be and the more of it will be reflected in the regular specular way. (TBD soon)

Directly a function of the heigh of the bumps.

If you have a surface that is super-polished it is possible the scatterin then is very very small (less than 1% of incoming light ). So very small but still measurable.


## 3. Geometric (Ray) Optics

The kind of physics you have in most of computer graphics.

Our first simplification at this level is to ignore nanogeometry.

IF it is smaller than a lightwave it does not exist. 

We treat all optically smooth surfaces as this abstract flat surface.

--- 

It can be shown from the equations governing electromagnetic waves that such a perfect surface will split light into exactly two directions.

# Reflection and Refraction


Most real world surfaces aren't optically smooth. They have irregularites that are much bigger than nanogeometry and a light wavelength but still smaller than a pixel.

We will call this **Microgeometry**

This kind of irregularities do not cause diffraction, it simply tilts the surface in various direction. And since we know that reflection and refraction both depend on the surface normal, it is effectively changing the local normal.

At a very small scale these are only being reflected in one direction, per bump but since a pixel covers many bumps you end up getting some kind of statistical average. 

Rougher surfaces, thouse with more irregularities in the microgeometry scale cause blurrier reflections.

Every surface is angled in a different direction, but they might be in similar directions if the roughness is low, or wildly different ones if the surface is high.

In the macro view that we normally work in we do not model the microgeomery directly, we treat it statistically.

We view the surface reflecting and refracting in multiple direction inside a cone, and the rougher the surface the wider this cone will be. 

So far we have talked about reflected light, but what happens to refracted light. 

### Refracted light

Three main types of materials:

* Metals (Conductors )
* Dielectric (Insulators)
* Semiconductors

In games and in movies you only really see metals and nonmetals and ignore dielectrics.

### Metals

Immediately absorb all refracted light, it gets lost in the sea of free electrons and never returns.

### Non-metals however

Behave like the simplistic homogeneous medium we discussed earlier, there is a participating medium and it is located underneath the surface and it is where all the refracted light goes and participates with.

There refracted light is either scattered again or absorbed or scattered again etc etc. 

If the object is made of something like glass, the light will jsut move through the medium. 

However, if there is enough scattering some of the refracted light will be scattered back out of the surface. 

The re-emitted light comes out at verying distances from the entry point.

This distribution of distances mostly depends on the density and other properties of the scattering particles.

If the pixel size or the shading size or sample area is large when compared to the entry / exit distance. Then we can assume the distances are effectively zero fro shading purposes.

We can't resolve the fact that the light is coming out at a different point that it is coming in. 

So in that case we can compute all shading locally at a single point and the shading color is only affected by light at one point. 

It is convenient to split this very different light points into different shading terms. We term to call the surface reflection as specular and the term from absorption, refraction, scattering and re-refraction as diffuse. 

In the case when the pixel is very small, and the distance of the area of interest is much smaller than the scattering distances we cannot treat the shading as all happening in a single point and we will need some special **subsurface scattering techniques** 

Even diffuse shading is the same as subsurface scattering, the only difference is the ratio of shading resolution to scattering distance. 

It is not true to say that a material is subsurface scattering since they all happen, it's just a function of distance to the camera what system we use. 



## So far we have discusses just the physics, we have to now discuss and quantify this behaviour mathematically. 

The first step is to quantify light as a number.

Radiometry is the measurement of lights. And there are many different radiommetric quantities that are itnegrals over direction or area. 

#### Radiance

Measures the intensity of light over a single ray and it is spectraly varying.

Radiance values are properly explained as SPD's but most people and traditionally we have just used RGB for anything that is spectrally varying 

The units of radiance are Watt/(steradian * m^2)

Since we assume that shading can be handled locally vs subsurface scattering case.

Then light response only depends on the ligh and view directions.

Since you can parametrize each of those with two numbers you have a 4 dimensional function we call:

#### Bi-directional reflectance distribution function

BRDF

Function of light direction and view direction.

Note that BRDF are only defined for light and view vectors above the microscopic level of the surface. 

## 4.0 The reflectance Equation

It basically says that out going radiance from a point equal the integral of incoming radiance times the BRDF times the cosine factor. Over the hemisphere of incoming directions.

Weighted average over all incoming directions. Take all light coming in, use BRDF to weigh more light in one direction than in others and then just adding it all together.

### Looking at the surface / Specular term

It is the light that gets directly reflected. 

We will be looking at it from the perspective of microfacet theory.

* Microfacet theory is a way to derive BRDF for surface reflection from this non-optically falt surfaces.
* The assumption is that we have imperfections that are small with respect to light wavelength
* Each point is a perfect mirror
* A perfect mirror will reflect each incoming ray of light, ignoring refraction for the moment, into one outgoing direction, the normal. In this case the microfacet normal.

Only the microfacets that have their normal oriented in the right direction to bounce L into V will reflect visible light. All that are aiming in some other direction will aim it to the wrong place that is not v.

Will not contribute to BRDF.

This perfect direction is called the half-vector H. It can be interpreted as jut the direction the microfacets need to be pointing towards. 

---

Yet not all microfacets that have an N = H will contribute, some will be blocked by other microfacets by the light direction and others by the view direction:

* Blocked by light direction: shadowing
* Blocked by view direction: masking

In reality, blocked light will continue to bounce and some of it will eventually escape towards V, but we just ignore this effect.

This is the form of a general Microfacet specular BRDF:

$f(l,v) = \frac{F(l,h)G(l,v,h)D(h)}{4(n* L)(n * v)}$


It has a component structure which makes it very useful.

### 4.1 Fresnel Reflectance

Its the fraction of incoming light that is reflected as oposed to refracted. From an optically flat surface for a given surface at a given angle. It varies based on the light angle and the surface normal.

Tells us basically of the light that hits the microfacets that are aligned to the half vector, what percentage is reflected.

This can be graphed for a given material as a function of the angle of incidence:

* the angle between the view and the normal ( l and H ) 

We do not care about the microfacets for which there microfacets are not aligned with H. 

Fresnel reflectance has a very specific kind of behaviour  since it is a function of the angle between the view and the normal we can see that at the edges it is much brighter than at the middle. 

IT can be divided into about 3 areas:

* barely changes from 0 to 40 degrees
* Changes somewhate from 45 to 75
* Goes very rapidly to 1 from 75 to 90 

The vast majority of pixels are in area 1 and 2 and only a very small sample are in area 3.

Over most of the visible surface the fresnel reflectance is similar to the value it has at 0 degrees it is useful to treat this value as the surface's characteristic specular color.

Metals as a general rule have very bright specular except gold's blue value.

##### F0 for dielectrics

On the other hand dielectric have dark specular colors and they are not chromatic. They also typically have adiffuse color.

Unlike metals which have only the specular color.

Vast majority of dielectrics fall between plastic and glass, so people just put it at 0.04;

We need to also model the angular variation in some way and most people just use the frehnel schlick approximation which is 

$F_{Schlick}(F0, l, h) = F_0 + (1-F_0)(1 - (l * h ))^5$

This approximation is useful because it parameterizes using F0 

### 4.2 Normal distribution function

D(h)

We want to know how many of the microfacets are actually pointing in the direction of h.

GGX is the one that is typically used now since it gives a strong highlight with a halo arond it. 

Surface deformation also affects the NDF

Geometry shadowing/masking function.

What concentration of microfacets are actually transferring light, they are not shadowed or masked and are actually transferring light from the l direction to the v direction. 

Only the smith function is both mathematically valid and physically realistic. 

The dividing factors are correction factors with respect to the frames involved


## 5.0 Diffuse term

Typically people used to use the lambert term for this

Diffuse and specular roughness should be completely different from one another




















