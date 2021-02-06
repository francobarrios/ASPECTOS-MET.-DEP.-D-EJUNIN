# ASPECTOS METEOROLOGICOS DEL DEPARTAMENTO DE JUNIN

###  1. PRIMERO INSTALAMOS LOS PAQEUTES QUE UTILIZAREMOS

          install.packages ("geojsonio")       
          install.packages ("rgdal")
          install.packages ("googledrive")  
          install.packages("mapedit") 
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
![WhatsApp Image 2021-02-05 at 6 03 44 PM](https://user-images.githubusercontent.com/78610845/107098285-db05f980-67dc-11eb-8d98-818bdcc35b71.jpeg)

### 7. FILTRAR LO DATOS SOLO DE JUNIIN

        junin <- prov %>% 
          filter(PROCOD98 == 1205)

### 8. VISUALIZAR EL MAPA DE JUNIN

        mapview(junin)
        
![WhatsApp Image 2021-02-05 at 6 03 44 PM (2)](https://user-images.githubusercontent.com/78610845/107098673-c8d88b00-67dd-11eb-92e8-2ba813b3defd.jpeg)

        mapview(demperu)
        
![WhatsApp Image 2021-02-05 at 6 03 44 PM (1)](https://user-images.githubusercontent.com/78610845/107098575-857e1c80-67dd-11eb-9912-a94f12e824e2.jpeg)

        library(plainview)
        library(rgdal)
        mapview(list(demperu, junin))

### 9. CREAR UNA MASCARA

        sp_junin <- as(junin, "Spatial"

### 10. HACEMOS UN CORTE EN LA PROVINCIA DE JUNIN Y VISUALIZAMOS

        junin_c<- crop(demperu, sp_junin)        
        plot(junin_c)
        
![WhatsApp Image 2021-02-05 at 6 03 44 PM (3)](https://user-images.githubusercontent.com/78610845/107098729-f4f40c00-67dd-11eb-8e76-d865ea0b7e37.jpeg)

        junin_dem <- mask(junin_c, sp_junin)
        plot(junin_dem)
![WhatsApp Image 2021-02-05 at 6 03 44 PM (4)](https://user-images.githubusercontent.com/78610845/107098820-2a005e80-67de-11eb-942f-897fa96db36a.jpeg)


### 11. CONOCIENDO LA ALTITUD MEDIA

        altitud_media <- raster::extract(junin_dem, 
                                         sp_junin, 
                                         fun = mean)

### 12.  CONOCIENDO EL AREA RELATIVA DE JUNIN

        area <- mapview(junin_dem) %>% editMap()
        
![WhatsApp Image 2021-02-05 at 12 19 53 PM](https://user-images.githubusercontent.com/78610845/107100150-8b75fc80-67e1-11eb-861c-7779a5711316.jpeg)        
        
        area_sf <- area$all
        plot(area_sf)
        
![WhatsApp Image 2021-02-05 at 12 19 53 PM (1)](https://user-images.githubusercontent.com/78610845/107100206-ab0d2500-67e1-11eb-8005-d4eaa37008ad.jpeg)        
        
        mapview(list(junin_dem, area_sf))

### 13. sf -> ee

        area_ee <- sf_as_ee(area_sf)
        area_ee
        
![WhatsApp Image 2021-02-05 at 12 19 53 PM (2)](https://user-images.githubusercontent.com/78610845/107100239-bf512200-67e1-11eb-814e-1015aa3dc148.jpeg)

### 14. EXTRAYENDO DATOS DE GOOGLE EARTH CON RGEE 

### PRECIPITACIONES DE UN AÑO

        mapview(list(pp_area, junin_dem))
        mapview(pp_area)
        
 ![WhatsApp Image 2021-02-05 at 12 19 53 PM (3)](https://user-images.githubusercontent.com/78610845/107100275-dc85f080-67e1-11eb-8244-e70138222119.jpeg)       
        
        mapview(junin_dem

![WhatsApp Image 2021-02-05 at 12 19 53 PM (4)](https://user-images.githubusercontent.com/78610845/107100305-f1628400-67e1-11eb-978f-dd176074445b.jpeg)

        ee_pp <- ee$ImageCollection("ECMWF/ERA5/MONTHLY")$
          filterDate("2018-01-01", "2019-01-01")$
          first()
        pp_stack <- ee_as_raster(imag  = ee_pp,
                                 region = area_ee$geometry())
        pp_area <- pp_stack[[5]]
        plot(pp_area)

### PRESION (Pa)

        presion_area <- pp_stack[[6]]
        plot(presion_area)

### 15. SELECCIONANDO PUNTOS EN EL AREA DE JUNIN

        puntos <- mapview(junin_dem) %>% 
          editMap()

        puntos_sf <- puntos$all
        puntos_sf
        plot(junin_dem,
             main = "junin")
             
![WhatsApp Image 2021-02-05 at 12 19 53 PM (5)](https://user-images.githubusercontent.com/78610845/107100932-c1b47b80-67e3-11eb-87b8-f18111bdf427.jpeg)

        plot(puntos_sf, add = T)
     
![WhatsApp Image 2021-02-05 at 12 19 53 PM (6)](https://user-images.githubusercontent.com/78610845/107101056-17892380-67e4-11eb-922a-d6f7a176de4e.jpeg)


### 16.AGREGANDO UNA COLUMAN DE PRECIPITACION DE LOS PUNTOS SELECCIONADOS

        puntos_sf$pp <- raster::extract(pp_area, puntos_sf)
        puntos_sf$Altitud <- raster::extract(junin_dem, puntos_sf)
        puntos_sf$Presión <- raster::extract(presion_area, puntos_sf)
        puntos_sf

### 17. RELACIONANDO ELEMENTOS CLIMATICOS

        comparar <- puntos_sf %>%
          as_tibble() %>% 
          dplyr::select(pp, Altitud, Presión)

      comparar

### 18. REALIZAMOS UN SCATER PLOT DE ALTITUD VS PRESION

        plot(comparar$Altitud, comparar$Presión,
             main = "Altitude vs Surface Pressure",
             ylab = "Surface Pressure(Pa)",
             xlab = " Altitude (m)")
     
### 19. CONOCIENDO LA PRESIPITACION INSTANTANEA POR HORA

            pp_hour <- ee$ImageCollection("JAXA/GPM_L3/GSMaP/v6/operational")$
              filterDate("2018-08-06", "2018-08-07")$
              first()
            pp2_stack <- ee_as_raster(imag  = ee_pp,
                                      region = area_ee$geometry())
            pp_area_hour <- pp2_stack[[1]]
            plot(pp_area_hour)
            mapview(list(pp_area_hour, junin_dem))
            mapview(pp_area_hour)

### 20. CONOCIENDO EL ALBEDO INSTANTANEA

          albedo <- ee$ImageCollection("NASA/GLDAS/V021/NOAH/G025/T3H")$
            filterDate("2018-08-06", "2018-08-07")$
            first()
          albedo_stack <- ee_as_raster(imag  = albedo,
                                       region = area_ee$geometry())
          albedo_area <- albedo_stack[[1]]
          plot(albedo_area)
          
![albedo](https://user-images.githubusercontent.com/78610845/107101448-5b305d00-67e5-11eb-965b-18de490238dd.png)

          mapview(list(albedo_area, junin_dem))
          mapview(albedo_area)
          
![mapview albedo](https://user-images.githubusercontent.com/78610845/107101501-8c109200-67e5-11eb-92b1-b8e1674c713d.png)

### 21. AÑADIENDO COLUMNA DE ALBEDO DE LOS PUNTOS SELECCIONADOS

          puntos_sf$Albedo <- raster::extract(albedo_area, puntos_sf)

          comparar <- puntos_sf %>%
            as_tibble() %>% 
            dplyr::select(pp, Altitud, Presión, Albedo)

          puntos_sf
          comparar

### 22. EXTRAYENDO DATA DE GOOGLE EARTH ENGINE

        terraclimate <- ee$ImageCollection("IDAHO_EPSCOR/TERRACLIMATE")$
          filterDate("2010-01-01","2017-12-31")$
          map(function(x) x$reproject("EPSG:4326")$select("pr"))
  
### 23. EXTRAYENDO DATOS DE PRESIPITACION 

          x = junin %>% sf::st_set_crs(4326)
          sfsf::st_set_crs(4326)
          ppt <- ee_extract(terraclimate, x,
                            fun = ee$Reducer$mean())
          head(t(ppt))

### 24. GENERANDO OTRO FORMATO

      ppt1 <- ppt[5:length(ppt)]
      library(tidyverse)
## otro formato 
        ppt_2 <- pivot_longer(ppt1, everything(), 
                              names_to = "month",
                              values_to = "pr")

### 25. CREANDO UNA COLUMNA DE FECHAS

        Fecha <- seq(as.Date("2010-01-01"),
                     by = "month",
                     length.out = 96)
        ppt_2$fechas <- Fecha

### 26. PLOTEAMOS

        ggplot(ppt_2, aes(fechas, pr))+
          geom_line(col = "blue")+
          ggtitle("Precipitación de junin")+
          xlab("Fecha")+
          ylab("Precipitación (mm)")
          
![precipitacion de junin (1)](https://user-images.githubusercontent.com/78610845/107101544-b2cec880-67e5-11eb-9cfb-4e85aaf67753.png)

