
Los datos de la población de México fueron adquiridos a partir del [idbr package for R](https://github.com/walkerke/idbr), que permite el acceso a indicadores demográficos del [United States Census: International Database (IDB)](https://www.census.gov/data/developers/data-sets/international-database.html) y descargarlos en formato de datasets en R. En el caso de México, los datos de población están disponibles desde 1980 a 2100 (proyectado). 



##### Instalar paquetes/Cargar librerias

```{r}
if(!require('pacman')) install.packages('pacman')
pacman::p_load(idbr, tidyverse, animation, kableExtra)
```

* **pacman**: Cargar todas las librerias a la vez
* **tidyverse:** Preparar el dataframe (*tidyr*), manipular el dataframe (*dplyr*, *stringr*), crear graficos (*ggplot2*)
* **animation:** Animación de los graficos en formato GIF
* **kableExtra:** Crear tablas visualmente atractivas

##### Colocar la Census API Key

La API Key puede ser obtenida del US Census Bureau en http://api.census.gov/data/key_signup.html.

```{r}

idb_api_key("Inserta la API Key")

```

##### Conseguir los datos de población a partir del idbr package

Solicitar los datos de población de México por genero, desde el año 1980 al año 2100, de acuerdo a la publicación de la [IDB](https://www.census.gov/data-tools/demo/idb/#/country?YR_ANIM=2021&FIPS_SINGLE=MX). 

```{r}
# Datos de la poblacion masculina.
Hombres <- idb1('Mexico', 1980:2100, sex = 'male') %>%
  mutate(POP = POP * -1, SEX = 'Hombres')

# Datos de la población femenina.
Mujeres <- idb1('Mexico', 1980:2100, sex = 'female') %>%
  mutate(SEX = 'Mujeres')

```

Por último, crear un data frame con ambos resultados.

```{r}

mexico_pop <- rbind(Hombres, Mujeres) %>%
  mutate(abs_pop = abs(POP))

# Guardar el data frame en formato csv.
write.csv(mexico_pop, 'Mexico-Population-1980-2100.csv')


```


```{r}

# Revisión de las dimesiones del dataframe
dim(mexico_pop)

```
Lo anterior nos muestran las carcteristicas del dataframe: 24442 registros y 9 atributos. A continuación se muestran los primeros 6 registros, con la finalidad de observar cuál es el estado de los datos despues de la aplicación de *dplyr*.

```{r}

kable(head(mexico_pop)) %>% 
  kable_styling(bootstrap_options = 'striped', font_size = 12, full_width = FALSE)

```

##### Construir la animación a partír de un bucle

```{r}

# Comienzo de la creación del GIF
saveGIF({
  
  # Crear un bucle de acuerdo al rango de los datos (1980:2100)
  for (i in 1980:2100) {
    
    title <- as.character(i)
    
    year_data <- filter(mexico_pop, time == i)
    
    # Crear el grafico con ggplot
    Pyramid_Mex <- ggplot(year_data, aes(x = AGE, y = POP, fill = SEX, width = 1)) +
      coord_fixed() + 
      coord_flip() +
      # Constucción de los gráficos de barras para hombres y mujeres
      geom_bar(data = subset(year_data, SEX == "Mujeres"), stat = "identity") +
      geom_bar(data = subset(year_data, SEX == "Hombres"), stat = "identity") +
      # Modificar el eje Y del gráfico d acuerdo a los datos del dataframe
      scale_y_continuous(breaks = seq(-1400000, 1400000, 200000),
                         labels = paste0(as.character(c(seq(1.4, 0, -0.2), c(seq(0.2, 1.4,0.2)))), "m"),
                         limits = c(min(mexico$POP), max(mexico$POP))) +
      scale_fill_manual(values = c('#4682b4', '#ee7989')) +
# Tema del grafico (Tema oscuro)
      theme(
        plot.background = element_rect(fill = "#232227", colour = "#232227"),
        axis.ticks = element_blank(),
        axis.title.y = element_blank(),
        legend.title = element_blank(),
        panel.background = element_rect("#111111", colour = 'white'),
        panel.border = element_blank(),
        strip.background = element_blank(),
        strip.text.x = element_blank(),
        panel.grid.minor = element_blank(),
        panel.grid.major = element_blank(), 
        axis.text = element_text(size = 14, color = 'white'),
        legend.key.size = unit(0.75, 'cm'),
        legend.text = element_text(
          size = 16,
          face = 'bold',
          color = 'white'
        ),
        plot.title = element_text(
          size = 22,
          hjust = 0.5,
          face = 'bold',
          color = 'white'
        ),
        plot.subtitle = element_text(
          size = 22,
          hjust = 0.5,
          face = 'bold',
          color = 'white'
        ),
        axis.title.x = element_text(
          size = 16,
          face = 'bold',
          color = 'white'
        ),
        plot.caption = element_text(
          size = 15,
          hjust = 0.5,
          face = 'italic',
          color = 'white'
        )
      ) +
      # Agregar títulos y subtítulos al gráfico y a los ejes 
      labs(
        title = 'Estructura de la población de México 1980 - 2100\n',
        subtitle = title,
        y = '\nPoblación',
        caption  = '\nFuente: US Census Bureau IDB; idbr R package'
      ) + 
      # Adecuar el recuadro de leyenda al tema
      theme(legend.position = "bottom", legend.title = element_blank(), 
            legend.background	= element_rect(fill = "#232227")) + 
      guides(fill = guide_legend(reverse = TRUE))
     print(Pyramid_Mex)
    
  }

  # Gyardar el gráfico con formato con extensión GIF
}, movie.name = 'mexico-pyramid-black.gif', interval = 0.2, ani.width = 800, ani.height = 700)

    
```

<p align = 'center'>
<img src = 'grafico' />
</p>
