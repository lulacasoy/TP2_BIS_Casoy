Tp2_Bis_Casoy
================
Luciana Casoy
2023-09-24

------------------------------------------------------------------------

``` r
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.2     ✔ readr     2.1.4
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.0
    ## ✔ ggplot2   3.4.3     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.2     ✔ tidyr     1.3.0
    ## ✔ purrr     1.0.1     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(skimr)
library(readr)
library(dplyr)
library(sf)
```

    ## Linking to GEOS 3.9.3, GDAL 3.5.2, PROJ 8.2.1; sf_use_s2() is TRUE

``` r
library(ggplot2)
```

## VOY A TRABAJAR CON UNA BASE DE DATOS DE PROPERATI DE OPERACIONES DE COMPRA/VENTA EN AMBA DURANTE JUNIO 2021

``` r
properati_amba <- readr::read_csv("C:/Users/Usuario/OneDrive - Amplity Health/Escritorio/Lula/TP2_CASOY_MEU/DATA/amba_properati2021.csv")
```

    ## Rows: 19279 Columns: 13
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (6): provincia, partido, currency, title, property_type, operation_type
    ## dbl (7): created_on, rooms, surface_total, surface_covered, price, lat, lon
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

## DECIDO QUEDARME SOLAMENTE CON LOS INMUEBLES DE CABA, Y DENTRO DE ESTE ÁREA, CON LAS OPERACIONES DE VENTA QUE ESTÁN EN USD. CREO UNA NUEVA COLUMNA CON EL VALOR POR M2 DE LAS PROPIEDADES PARA PODER HACER UNA VISUALIZACION DE LOS VALORES DEL SUELO EN EL MERCADO DE LA CIUDAD DE BUENOS AIRES

``` r
properati_caba <- properati_amba %>%
filter(provincia == "CABA") %>%
filter(operation_type=="Venta") %>%
filter(currency=="USD") %>%
mutate(valor_m2=(price/surface_total))
properati_caba <-rename(properati_caba, Tipo_propiedad = property_type)
```

## USO GEOM_POINT FACETADO PARA MOSTRAR GRAFICOS COMPARATIVOS ENTRE LAS DISTINTAS COMUNAS DONDE CADA PUNTO REPRESENTA UNA PROPIEDAD, SU COLOR INDICA EL TIPO DE PROPIEDAD Y EN LOS EJES VEMOS TAMAÑOS DE PROPÍEDADES EN Y Y VALORES POR M2 EN X.

``` r
ggplot(data = properati_caba) +
  geom_point(mapping = aes(x=valor_m2,
                           y=surface_total, color=Tipo_propiedad))+
  facet_wrap(~partido, nrow=6) +
  labs(title = "Inmuebles en venta CABA - Junio 2021",  subtitle = "Comparativa por Comunas", 
        y = "Superficie Inmueble",  x = "Valor en USD por m2", 
        caption = "FUENTE: Properati")
```

![](TP2_Bis_Casoy_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

## AHORA RE AGRUPO EL DATA SET CON UN VALOR PROMEDIO DEL VALOR DEL M2 POR COMUNA PARA PODER PLASMARLO EN UN MAPA DE CABA

``` r
properati_caba <-rename(properati_caba, COMUNAS = partido)
properati_caba <-aggregate ( properati_caba$valor_m2, list(properati_caba$COMUNAS), FUN=mean)
properati_caba <-rename(properati_caba, COMUNAS = Group.1, Valor_m2 = x)
```

## TRAIGO EL DATA SET CON LOS POLIGONOS DE LAS COMUNAS PARA PODER ARMAR EL MAPA

``` r
comunas_caba <-read_sf("C:/Users/Usuario/OneDrive - Amplity Health/Escritorio/Lula/TP2_CASOY_MEU/DATA/comunas/comunas_wgs84.shp")
```

``` r
comunas_caba$COMUNAS <- paste("Comuna", comunas_caba$COMUNAS)
```

## UNO LOS DOS DATA SETS, EL DE LOS POLIGONOS CON EL QUE CONTIENE EL VALOR DE LAS PROPIEDADES

``` r
mapa_comunas_properati <- left_join(comunas_caba, properati_caba, by="COMUNAS")
```

``` r
ggplot(mapa_comunas_properati) +
geom_sf(aes(fill= Valor_m2), color="white",) +
  geom_sf_label(aes(label = COMUNAS), size = 2) +
 theme_void()+
labs(title = "Valores del m2: Inmuebles en venta CABA - Junio 2021",  subtitle = "Comparativa por Comunas", caption = "FUENTE: Properati")
```

    ## Warning in st_point_on_surface.sfc(sf::st_zm(x)): st_point_on_surface may not
    ## give correct results for longitude/latitude data

![](TP2_Bis_Casoy_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->
