## Diaphragm
- In theory, when imaging a scene through a thin lens, only the points at a certain distance can be on focus, all the others appear blurred into circles 
- However, as long as such _circles_ are _smaller than the size of the photosensing elements_, the image will still look on-focus (i.e. the light is collected by a single pixel of the camera sensor)
If the circle of confusion is contained within a single pixel, the image on that pixel will be sharp (and not blurred). That is not the case if the pixel does not contain the circle of confusion. 

- The _range of distances_ across which the _image appears on focus_ - due to blur circles being small enough - determines the __DOF (Depth of Field) of the imaging apparatus__. 
- The diaphragm controls the amount of light that I'm letting through the camera. 
	- Large hole -> larger cones (large blur circle)
	- Small hole -> small cone (small blur circle)

- Cameras often deploy an adjustable diaphragm (iris) to control the amount of light gathered through the effective aperture of the lens. 
- We close the diaphragm => increase depth of field => not enough light => increase exposure time => moving object => motion blur => still scenes

## Focusing Mechanism
To focus on objects at diverse distances: 
- mechanism that allows the lens (or lens subsystem) to _translate along the optical axis_ with respect to the _fixed_ position of the image plane

![[lenses_3.png]]
- At the _first end_ position (image on the left) (v=f) _the camera is focused at infinity_.
- The mechanism allows the lens to be translated farther away from the image plane up to a certain maximum value (the second end position), which determines the minimum focusing distance. 
	- The focus is closer to the camera. 

In cameras, there's an inverted relationship between $v$ and $u$. 

# Image digitization
Generally speaking, the image plane of a camera consists of a _planar sensor_ (a matrix of sensors) which converts the _irradiance_ at any point into an _electric quantity_ (e.g. a voltage) 
- Transduction from light to an electric quantity => “records” the light coming from a scene point

![[sampling_quantization.png]]

Such a continuous “electric” image is sampled and quantized to end up with a digital image suitable to visualization and processing by a computer 
- The continuous voltages in the sampled image are gonna be quantized into a fixed number of levels

- __Sampling__: the planar continuous image is _sampled_ along both the horizontal and vertical directions to pick up a 2D array (matrix) of N×M samples known as pixels:
![[image_matrix.png]]

- __Quantization__ (of colors): the continuous range of values _associated with pixels_ is _quantized_ into $l = 2^m$ discrete levels known as gray-levels. 
	- $m$ is the number of bits used to represent a pixel, with the size (in bits) of a grayscale image given by: $B = N \times M \times m$. 
	- colour digital images are instead typically represented within computers using 3 bytes per pixels (one byte for each of the RGB channels).
		- More memory to occupy.
	- The more the better => more pixels + more bits per pixel => higher quality image. 

## Image quality 
- The more bits we spend for its representation, the higher the quality of the digital image (we get a closer approximation to the ideal continuous image)
- This applies to both sampling (details) as well as quantization (color accuracy) parameters

## Camera sensors
- The sensor is a 2D array of photodetectors (photogates or photodiodes) 
- During exposure time, each detector converts the incident light into a proportional electric charge (i.e. photons to electrons) 
- The companion circuitry reads-out the charge to generate the output signal, which can be either digital or analog 
- For digital: the camera includes also the necessary ADC circuitry
![[sensor_model_2.png]]

- Nowdays, there is never a continuous image in practice => _the image is sensed directly as a sampled signal_.

Today, the two main sensor technologies are: 
- __CCD__ (Charge Coupled Devices) 
- __CMOS__ (Complementary Metal Oxide Semiconductor) 
	- Can be found on smartphones (since they are little and cheaper)

Unlike CCD, _CMOS_ technology allows the _electronic circuitry_ to be _integrated within the same chip_ as the sensor (“one chip camera”) 
- This provides more compactness, less power consumption and often lower system cost. 

Unlike CCD, CMOS sensors allow an _arbitrary window_ to be _read-out_ without having to _receive the full image_. 
- This can be useful to _inspect_ or track at a higher speed a small Region Of Interest (ROI) within the image 

_CCD technology_ typically provides _higher Signal-to-Noise Ratio_ (SNR), higher _Dynamic Range (DR)_ and better uniformity. 

- CCD/CMOS sensors are sensitive to light ranging from near-ultraviolet (200 nm) through the visible spectrum (380-780 nm) up to the near infrared (1100 nm). The sensed intensity at a pixel results from integration over the range of wavelengths of the spectral distribution of the incoming light multiplied by the spectral response function of the sensor ==> CCD/CMOS sensor cannot sense colour. 
- To create a colour sensor, an array of optical filters (__Colour Filter Array__) is _placed in front of the photodetectors_, so as to render each pixel sensitive to a specific range of wavelengths. 
	- In the most common Bayer CFA _green filters_ are _twice as much_ as red and blue ones to _mimic the higher sensitivity_ of the human eye in the green range. 
	- To obtain an RGB triplet at each pixel, missing samples are interpolated from neighbouring pixels (de-mosaicking). However, the _true resolution of the sensor is smaller_ due to the green channel being subsampled by a factor of 2, the blue and red ones by 4. 
![[camera_sensor.png]]
_A “full resolution” – though more expensive - colour sensor_ can be achieved by deploying an _optical prism_ to split the incoming light beam into 3 RGB beams sent to 3 distinct sensors equipped with corresponding filters.

## Signal to Noise Ratio
__Signal-to-Noise Ratio (SNR)__: the intensity measured at a pixel under perfectly static conditions _varies_ due to the presence of _random noise_ (i.e. a pixel value is not deterministic but rather a random variable).

The main noise sources are: 
- __Photon Shot Noise__: the time between photon arrivals at a pixel is governed by a Poisson statistics and thus the number of photons collected during exposure time is not constant. 
- __Electronic Circuitry Noise__: it is generated by the electronics which reads-out the charge and amplifies the resulting voltage signal. 
- __Quantization Noise__: related to the final ADC conversion (in digital cameras). 
- __Dark Current Noise__: a random amount of charge due to _thermal excitement_ is observed at each pixel even though the sensor is not exposed to light. 

The SNR can be thought of as quantifying the strength of the “true” signal with respect to the unwanted fluctuations induced by noise (i.e. the higher the better). It should be measured according to standard procedures and it is usually expressed either in decibels or bits:
$$
SNR_{db} = 20 \cdot log_{10} (SNR); \ SNR_{bit} = log_2 (SNR)
$$


## Dynamic Range
__Dynamic Range (DR)__: if the sensed amount of light is too small, the “true” signal cannot be distinguished from noise 
- given $E_{min}$ - the minimum _detectable_ irradiation (which depends on noise) 
- given $E_{max}$ - the saturation irradiation (i.e. the amount of light that would fill up the capacity of a photodetector)
The DR of a sensor is defined as $DR = \dfrac{E_{max}}{E_{min}}$ and, like the SNR, it is often specified in decibels or bits

- As it is the case of the SNR, also for the DR the higher is the better. Indeed, the higher the DR the better is the ability of the sensor to simultaneously capture in one image both the dark and bright structures of the scene. 
- An active research field in image processing deals with creating High Dynamic Range (HDR) images by combining together a sequence of images of the same subject taken under different exposure times.

