# Toon-Shader

I learned to create a post proccessing toon shader in unreal engine 4.

## Outline Effect
The outline effect is a simple process (theoretically), but not so much when it comes to the nodes. We essentially grab the scene normal lines and depth lines and we compare then to find the differences. The simple formula is `Center Pixel - Offset Pixel = Toon Lines`.

So what we need to do is sample the pixel above, to the left, right, and down from the origin pixel. We start off by getting our `texture coordinate` which is our orgin pixel. Then we need a `SceneTexture:SceneDepth` node to get exactly 1 pixel. Now in order to get the corresponding pixels around we can use a `vector2` and multiply that to our `SceneTexture:SceneDepth` and then we add it to our `texture coordinate` (In this picture we are grabbing the left pixel coordinate). We then want to mask that out with just the `R` channel because we only want the gray scale. Finally we want to take that offset pixel and subtract that from our origin pixel.

![](https://user-images.githubusercontent.com/29154540/158484068-160c2b9b-b218-4cba-a052-d47f83aa1a51.png)

This is what it ends up looking like.

![](https://user-images.githubusercontent.com/29154540/158484413-f0aae679-a298-417f-a361-8e71cff3cbcc.png)

Now we pretty much copy and paste this 3 more times for each offsetted pixel(This is a little hard to see because my map is exposed to direct sun light).

![](https://user-images.githubusercontent.com/29154540/158485094-6d12bd5a-44dc-49f8-a9fc-ed7e012c0e83.png)

For the normal lines, we can pretty much copy and paste the entire code. What we need to change is the `SceneTexture:SceneDepth` to `SceneTexture:WorldNormal`. Now remember how we masked out the only the `R` channel. We need now `RGB` masking because we need to compare the `X`, `Y`, and `Z` of the scene.

![](https://user-images.githubusercontent.com/29154540/158485824-217eb431-62bd-451b-b075-425d53943750.png)

Here is the full Code.

![](https://user-images.githubusercontent.com/29154540/158482180-72adbd58-4d81-41fd-b0c4-0eeb4e0ad771.png)
![](https://user-images.githubusercontent.com/29154540/158482182-f410c96b-c106-4f6d-87ff-eabe5ebfd6e5.png)

Now when we combine them we can get some nice offseted lines!

![](https://user-images.githubusercontent.com/29154540/158486341-1b6b0907-931e-4ba0-aba3-0dab8bb52afe.png)

Although I noticed a problem, it was affecting the skybox as well. I didn't really want that so I masked out the sky. It's essentially a distance mask so anything that is less than `10 000` will have that affect other wise it will just return the regular scene. In addition we also want to mask out the depth objects. We also want to add these two masks to our cell shading shader at the end as well.

![](https://user-images.githubusercontent.com/29154540/158486473-c7b70d67-21cf-4028-bf66-7e8087a2eb02.png)

Finally putting everything together we get toon lines!!

![](https://user-images.githubusercontent.com/29154540/158486638-addff8a4-8e33-43d2-870e-89048ec55848.png)


## Cell Shading - Single Tone
The single tone shader only has a light and dark toning process. The cell shading is done by extracting the light info with the `SceneTexture:PostProcessInput0` node and the `SceneTexture:DiffuseColor` node. 

![](https://user-images.githubusercontent.com/29154540/158470259-5ec05161-d236-4242-b2c6-ec2e396f7a04.png)

After extracting the light info, I use an `if` node to determine the lighting tint. I set a default of `0.5` and anything above that will recieve a light tint, and anything below a dark tint.

![](https://user-images.githubusercontent.com/29154540/158472876-cc60a89d-e790-4b81-bf16-c75d37008719.png)
![](https://user-images.githubusercontent.com/29154540/158472906-7f588d3c-e91f-4df8-b88d-224587d031ed.png)

This is what it looks like with no colour(Left) and with colour(Right) 

![](https://user-images.githubusercontent.com/29154540/158482534-9ba4be75-7c54-499c-a246-c1233df2f840.png)
![](https://user-images.githubusercontent.com/29154540/158482438-e7a83e11-1947-4bb0-ba99-dd53941c3c6f.png)

## Cell Shading - Multi Tone
The multi tone shader has 3 levels of light to process, the brightest is done automatically by the scene, the mid-level, and the shadows.

The light extraction is done the same was as the single tone. Although, you have more control with how the light is proccessed based on some thresholds. This includes the `Dark Intensity`, `Dark Threshold`, `Mid-Level Intensity`, and `Mid-Level Threshold`. There is also an optional colour selection, this just takes the output of the light and shadow and multiplies a `Float4` to it.

![](https://user-images.githubusercontent.com/29154540/158480795-65e5e65d-6dd4-46ba-bbc4-20a611ca7366.png)

This is what it looks like with no colour(Left) and with colour(Right) 

![](https://user-images.githubusercontent.com/29154540/158481230-2c60bbdb-da87-4ed1-b849-3813ebbaff7b.png)
![](https://user-images.githubusercontent.com/29154540/158481405-5f223485-e0a3-4020-a325-4af1455df174.png)

Done!! This is my process of making a comic style look, it was very interesting to work on this and I learned a lot in the process. For this to work you can just add a `PostProcessVolume` into your scene(scale it as you wish) and in the `Rendering Features`  add the two material instances into the array.

![](https://user-images.githubusercontent.com/29154540/158486904-df47c9f6-dcd7-4416-bde9-ecd7c85c9947.png)
