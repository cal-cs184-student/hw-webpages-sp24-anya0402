#### Task 1:  
<u>Implementation:</u> 
To rasterize triangles, we first need to determine the boundary box to account for edges and find the boundary of where a point would be included and where it won't. The bounding box is defined as the rectangle from $(x_{\text{min}}, y_{\text{min}})$ to $(x_{\text{max}}, y_{\text{max}})$. $x_{\text{min}}$ is determined as the minimum `x` cordinate of the vertices passed in as input by using the `min` and `floor` functions. $x_{\text{max}}$ is similarly calculated but with the `max` function instead. The same process was done for the `y` coordinate. These values represent the smallest and largest `x` and `y` values out of the points given as input. Therefore, our algorithm only looks at points/pixels within the bounding box of the triangle.

Next, we implemented the line test method taught in lecture to determine whether a point laid inside or outside of the triangle. To do this, we initially created a lambda function that determined whether the point is to the left, right, or on the edge/line `AB`. If the point is on the same side of each triangle edge, this meant that the point is inside the triangle. Later on the project, we created a separate helper function defined as `point_inside()` that did the line test calculation for each line of the triangle outside of `rasterize_triangle()` for more convenient usage. Our implementation ensures that the line test would work regardless of the winding order of the vertices.

After that, we now know the boundary box so we're able to iterate over every single pixel using a double for loop. Using our helper function, if the point is in the triangle, we send the according color to the frame buffer using `fill_pixel()`. This color could be a given color or sampled from a texture.

Here is an example of basic triangles with aliasing displayed:
![Rasterizing Triangles](/docs/images/task1_image.png "Rasterizing Triangles")

#### Task 2:
<u>Implementation:</u> 
In traditional rendering and in Task 1, we sampled the point in the middle of the pixel by adding 0.5 to the x and y axis to determine the pixel's color. This can lead to aliasing issues especially along edges and curves, creating jaggies which is displayed with our results from rasterization done in Task 1. With supersampling, multiple sample points are taken within each pixel instead of just sampling one point per pixel. We divide each pixel into 4, 9, and 16 squares — essentially 4, 9, and 16 points inside each pixel. Since each pixel is divided into subsamples, each is essentially treated as their own points/"pixels" and were thus subject to the same sampling method of whether the point was inside or outside the triangle.

However, we aren't able to directly apply this to the screen because we have fewer screen pixels than our data structure. We averaged the color of the sampled points to create a new color for each pixel. The color was based on the "area" of the single sample that was covered by the triangle. To store the supersamples, we used the existing data structure/variable `sample_buffer` which is essentially a `std::vector<Color>` that is the internal color sample buffer containing all samples. Notably, the number of elements in the `buffer = width * height * sample_rate`. 

Supersampling is useful because creates better detail in our triangles by "smoothening out" the edges by generating intermittent colors between high-frequency pixels. Instead of only 1 sample per pixel, the averaging process helps reduce aliasing artifacts and produces smoother transitions between colors, resulting in a better visual representation of the triangle. 

For the rasterization pipeline, we made a few modifications to the rasterization pipeline in the process. Specifically, we utilized the `sample_rate` variable previously defined to specify the number of samples per pixel. In the `resolve_to_framebuffer` function, instead of directly copying colors from the sample buffer to the framebuffer, colors from all sample points within each pixel are averaged to produce the pixel's final color. We also updated `set_sample_rate`, `set_framebuffer_target` and `resolve_to_framebuffer` to account for the changes in `sample_rate`. An important note is that despite supersampling enabling "smoother" and more detailed images, it increases computational cost since more samples need to be processed for each pixel.

Here is the screenshot of basic/test4.svg with the default viewing parameters and sample rate of 1 per pixel:
![Test 4 Rate 1](/docs/images/basic:test4.svg_rate1.png "Rate 1")

Here is the screenshot of basic/test4.svg with the default viewing parameters and sample rate of 4 per pixel:
![Test 4 Rate 4](/docs/images/basic:test4.svg_rate4.png "Rate 4")

Here is the screenshot of basic/test4.svg with the default viewing parameters and sample rate of 9 per pixel:
![Test 4 Rate 9](/docs/images/basic:test4.svg_rate9.png "Rate 9")

Here is the screenshot of basic/test4.svg with the default viewing parameters and sample rate of 16 per pixel:
![Test 4 Rate 16](/docs/images/basic:test4.svg_rate16.png "Rate 16")

We zoom in on a very skinny corner of the pink triangle. When we only sample once per pixel, entire pieces of the triangle are missing but as we sample more times per pixel, more of the triangle "appears"/"fills in". This is because of the averaging color effect that reduces aliasing - for example, if a sample was 50% covered by the triangle, it would have closer to 50% the saturation of the triangle color. 

#### Task 3:
Here is the screenshot of our robot:
![Custom Cubeman](/docs/images/robot_image.png "Our Cubeman")

For this version of the cubeman, we wanted to make it wave. To do this, we moved one arm down closer to its side, and made the other arm bend so that it is like the cubeman is waving. We also changed the colors of the objects that make up cubeman to make it look more human-like with clothes.

#### Task 4:
Barycentric coordinates help us to linearly interpolate coordinates so that a texture or color mapping is smooth and even without jagged lines. When we use barycentric coordinates, we are finding the exact distance of the point and what combination of values is needed to get what we want at that point. We can take this image of a colored triangle as an example.
![Color Triangle](/docs/images/triangle_color.png "Color Triangle")
We start of as each corner of the triangle being red, green, and blue. Then, barycentric coordinates helps us calculate the exact ratio of how much red, green, and blue we need at a specific point in the triangle, and colors it that color. This happens for the whole triangle until it is fully smoothly shaded.

Here is a screenshot of the colorwheel generated by the interpolated coordinates:
![Color Wheel](/docs/images/colorwheel.png "Color Wheel")
One issue we came across with generating this image is that there seems to be a thin white line going through the circle when first rendered. However, if we move the circle slightly to the side, either the white line changes position, or it goes away altogether, which is shown in the image above. We first thought this was an issue with our bounding box since it could affect the edges, which may cause the line to occur. We tried flooring the values and also changing them from floats the doubles, but nothing was able to fix it. We think it could be an issue internally with the rendering and creation of the window, since there are positions where the full circle is colored in and interpolated correctly.

#### Task 5:
Pixel sampling is how translate from x-y coordinates for pixels to texel u-v coordinates. It's essentialy how we map textures onto triangles of the vector graphic - we're given the corresponding texture coordinates so we sample the color of a given point from a texture. When converting x-y to u-v, this results in a decimal value but we can only sample from integers. If we are using the nearest pixel sampling method, we sample from the nearest integer `u` and `v` coordinates. If we are using the bilinear method, we sample from the four closest u-v coordinates and use linear interpolation to create a weighted sample of neighbors in the horizontal and vertical directions for the new color. 

The closer the point is to one of the corners, that corner's color will have a stronger influence on the fill color. Pixel sampling typically gives us better rendering since textures appear smoother as a result of averaging texels - less jumps between neighboring pixels. 

Here is the screenshot of nearest sampling at 1 sample per pixel:
![Nearest 1 Sample](/docs/images/task5_nearest_1_sample.png "Nearest: 1 Sample Per Pixel")

Here is the screenshot of nearest sampling at 16 samples per pixel:
![Nearest 16 Sample](/docs/images/task5_nearest_16_samples.png "Nearest: 16 Samples Per Pixel")

Here is the screenshot of bilinear sampling at 1 sample per pixel:
![Bilinear 1 Sample](/docs/images/task5_bilinear_1_sample.png "Bilinear: 1 Sample Per Pixel")

Here is the screenshot of bilinear sampling at 16 samples per pixel:
![Bilinear 16 Sample](/docs/images/task5_bilinear_16_samples.png "Bilinear: 16 Samples Per Pixel")

At lower sampling rates, the differences between the two methods are less obvious but as the sampling rate increases, bilinear sampling is generally better, especially when there are large variations in texel colors within a small area (ex: detailed textures) as pictured above. The nearest sampling at one sample per pixel is extremely pixelated in the magnified image but bilinear sampling "smoothens" this out even if we sample at one sample per pixel. Bilinear sampling at 16 samples per pixel is very "smooth" since the area is relatively small. We noticed that the largest difference between nearest and bilinear sampling is when there are significant changes in texel colors within a pixel. This makes sense because bilinear sampling will interpolate between texels whereas nearest sampling is unable to do so, potentially resulting in aliasing artifacts. 

#### Task 6:
