# ASPECTOS METEOROLOGICOS DEL DEPARTAMENTO DE JUNIN
SUBIR ARCHIVOS 

###  1. PRIMERO INSTALAMOS LOS PAQEUTES QUE UTILIZAREMOS

install.packages ("geojsonio")
install.packages ("plainview")
install.packages ("rgdal")
install.packages ("googledrive")
install.packages("mapedit")
install.packages("tidyverse")
install.packages("sf")
install.packages("sp")
install.packages("raster")
install.packages("ggplot2")
install.packages("rgee")
install.packages("mapview")
install.packages("mapedit")
install.packages("lubridate")
install.packages("earthengine-api")

### 2. RESETEAR EL ESPACIO DONDE SE VA A TRABAJAR

arch <- "C:/DATA R/"
setwd(arch)

### 3. LLAMAR A TODAS LAS LIBRERIAS QUE UTILIZAREMOS

library(tidyverse)
library(sf)
library(sp)
library(raster)
library(ggplot2)
library(rgee)
library(mapview)
library(mapedit)
library(lubridate)

### 4. INICIAR SECION CON TU CUENTA GOOGLE EARTH ENGINE

ee_Initialize("nombre de usuario",drive = TRUE)

### 5. CARGAR EL SHAPE PROVINCIAS

prov <- st_read("Provincias.shp")
prov %>% filter(PRONOM98 %in% c("JUNIN"))

### 6. EXTRAEMOS DATOS DE ALTURA 

demperu<-raster("C:/data/demperu.tiff")
plot(demperu)





