# Guwahati-3D-map-Mapzen-DEM
A tutorial for 3D map generation in R using mapzen_dem()
![Demo1](https://github.com/Hwoabam/Guwahati-3D-map-Mapzen-DEM/blob/master/Media/Animation/Mapzen.gif)

The packages required are:
```{r}
library(rayshader)
library(geoviz)
library(raster)
library(rgdal)
```
Then the coordinates of the centre of the area of interest are assigned as the centre of the polygon to be used for data extraction alongwith the area in square km and the resolution as max number of tiles. Then mapzen_dem function is used for data exportation.
```{r}
##Lat, long of the centre of the area of interest
lat <- 26.151317
long <- 91.775044
square_km <- 15

##~Set max tiles. This sets the resolution of your DEM. 
##The max is 60, but the image will be slower in R (TEST ON 10)
max_tiles <- 45

#DEM data from Mapzen
dem <- mapzen_dem(lat, long, square_km, max_tiles = max_tiles)
```

Satellite imagery with the help of map key available for registered members of mapbox community or Stamen basemap may be used to overlay the elevation matrix
```{r fig1, fig.height = 15, fig.width = 10, align= "center"}
# If you want to put satellite imagery on, first get a mapkey
# MapKey: https://docs.mapbox.com/help/glossary/access-token
mapbox_key <- "pk.eyJ1IjoiaGFvYmFtIiwiYSI6ImNrY3QybTloODA2NmQyenBiZzJiNXI3ODcifQ.nHov5k4LBu9GwJPXI4SEBg"
#add this option to the following slippy_overlay: api_key = mapbox_key
# or use a Stamen basemap, like I did in this example:
overlay_image <-
slippy_overlay(
       dem,
    image_source = "stamen",
    image_type = "terrain-background",
  )
sunangle <- 270
```
The elevation matrix defined by Bilinear method with varying elevation values and using sphere_shade to shade the plot with defult black and white texture.
```{r fig2, fig.height = 15, fig.width = 10, align= "center"}
#Draw the rayshader scene
elmat = matrix(
  raster::extract(dem, raster::extent(dem), method = 'bilinear'),
  nrow = ncol(dem),
  ncol = nrow(dem) )

#Rayshader texture:
elmat %>%
  sphere_shade(texture = "bw") %>%
  plot_map()
```
![Default texture](https://github.com/Hwoabam/Guwahati-3D-map-Mapzen-DEM/blob/master/Media/Plots/BW.png)

The Stamen basemap is overlain on the elevation matrix defined previously. The resulting plot :
```{r fig3, fig.height = 15, fig.width = 10, align= "center"}
#plot map with overlay image and shadows
elmat %>%
    sphere_shade(texture = "bw") %>%
     add_overlay(overlay_image,alphalayer = 1) %>%
    add_shadow(ray_shade(elmat, zscale=50,maxsearch = 500)) %>%
     plot_map()
```
![final 2D plot with overlay](https://github.com/Hwoabam/Guwahati-3D-map-Mapzen-DEM/blob/master/Media/Plots/Stamen.png)
Using the PLot generated previoulsy and the elevation matrix, providing a zscale=7.5 and viewpoint parameters as desired. The compass is rendered to the west of the plot.
```{r fig5, fig.height = 15, fig.width = 10, align= "center"}
#render in 3d
elmat %>%
    sphere_shade(texture = "bw") %>%
    add_overlay(overlay_image,alphalayer = 0.6) %>%
    plot_3d(elmat, zscale = 7.5, fov = 0, theta = -15, zoom = 0.58, phi = 38, windowsize = c(1000, 800))
render_compass(position = "W" )
render_snapshot(title_text = "Guwahati Topography | Mapzen DEM",title_bar_color = "#F2E1D0", title_color = "black", title_bar_alpha = 1)
```
![3D plot](https://github.com/Hwoabam/Guwahati-3D-map-Mapzen-DEM/blob/master/Media/Snapshots/snapMp.png)

A 24 second video is generated of the animation of the rotation of the plot using ffmpeg function at framerate of 60fps. 
```{r eval=FALSE, include=FALSE}
angles= seq(0,360,length.out = 1441)[-1]
for(i in 1:1440) {
  render_camera(theta=-45+angles[i])
  render_snapshot(filename =
  sprintf("guwahati_Mpz%i.png", i),title_text = "Guwahati Topography | Mapzen DEM",title_bar_color = "#F2E1D0", title_color = "black", title_bar_alpha = 1)
}
rgl::rgl.close()
system("ffmpeg -framerate 60 -i guwahati_Mpz%d.png -vcodec libx264 -an Ghy_Mapzen_2.1.mp4 ")
```
The markdown is provided as under:
![final 2D plot with overlay](https://github.com/Hwoabam/Guwahati-3D-map-Mapzen-DEM/blob/master/Ghy_Mapzen.rmd)


