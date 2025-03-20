
![Rplot](https://github.com/user-attachments/assets/7ee34335-6194-42f2-9652-6cb212031e6a)

# README: Bivariate Choropleth Map for Human and Cow Density

This R script generates a bivariate choropleth map comparing human population density and cow density using a hexagonal grid. The map is visualized using a custom color palette and includes a legend explaining the color coding system.

### Requirements

The following R packages are required to run the code:

- `tidyverse`
- `sf`
- `pals`
- `patchwork`

To install these packages, you can use the following command:

```R
install.packages(c("tidyverse", "sf", "pals", "patchwork"))
```

### Overview

This code does the following:

1. Loads a hexagonal grid shapefile and population data.
2. Merges the population data with the hexagonal grid.
3. Creates a bivariate choropleth map using two datasets: human population density and cow density.
4. Customizes the legend and map colors.
5. Combines the legend and map into one figure using `patchwork`.

### Code Breakdown

#### 1. Load Packages

The code begins by loading the necessary libraries:

```R
library(tidyverse)
library(sf)
library(pals)
library(patchwork)
```

#### 2. Load Basemap

A hexagonal grid shapefile is loaded as the base map:

```R
hex = read_sf('https://github.com/BjnNowak/frex_db/raw/main/map/hex_grid.gpkg')
```

#### 3. Create Initial Map

A simple map of the hexagonal grid is plotted:

```R
ggplot(hex) + geom_sf()
```

#### 4. Load Data

The human population density and cow density datasets are loaded:

```R
human_pop <- read_csv('https://raw.githubusercontent.com/BjnNowak/frex_db/main/data/human/human_pop_density.csv')
herd <- read_csv('https://raw.githubusercontent.com/BjnNowak/frex_db/main/data/herds/herds_density_municipality.csv')
```

#### 5. Merge Data with Map

The population and herd data are merged with the hexagonal grid:

```R
data <- hex %>%
  left_join(human_pop) %>%
  left_join(herd)
```

#### 6. Prepare Data for Bivariate Choropleth

The code calculates a combined cow density column and assigns classes for both human population and cow density based on tier thresholds (25 and 50):

```R
data <- data %>%
  mutate(cow_km2 = milk_cow_km2 + meat_cow_km2) %>%
  mutate(cl_A = case_when(
    human_km2 < t1 ~ 'A1',
    human_km2 < t2 ~ 'A2',
    TRUE ~ 'A3'
  )) %>%
  mutate(cl_B = case_when(
    cow_km2 < t1 ~ 'B1',
    cow_km2 < t2 ~ 'B2',
    TRUE ~ 'B3'
  )) %>%
  mutate(cl = paste0(cl_A, cl_B))
```

#### 7. Create Bivariate Choropleth Map

A choropleth map is created using the calculated classes for human and cow density:

```R
p1 <- ggplot(data, aes(fill = cl)) +
  geom_sf()
```

#### 8. Assign Colors

A custom color palette is applied to the choropleth map:

```R
pal <- stevens.pinkblue(9)

pal_bivar <- c(
  'A1B1' = pal[1],
  'A1B2' = pal[2],
  'A1B3' = pal[3],
  'A2B1' = pal[4],
  'A2B2' = pal[5],
  'A2B3' = pal[6],
  'A3B1' = pal[7],
  'A3B2' = pal[8],
  'A3B3' = pal[9]
)
```

#### 9. Final Map Visualization

The final choropleth map is rendered with the specified color palette:

```R
p2 <- p1 +
  scale_fill_manual(values = pal_bivar) +
  guides(fill = 'none') +
  theme_void()
```

#### 10. Create Legend

A legend showing the relationship between human and cow density is created using `geom_tile` and `geom_point`:

```R
leg_tile <- ggplot(data = tib, aes(x = var_A, y = var_B, fill = value)) +
  geom_tile() +
  scale_fill_manual(values = pal) +
  guides(fill = 'none') +
  coord_fixed() +
  labs(x = "Human density", y = "Animal density") +
  theme_minimal() +
  theme(axis.text = element_blank())

leg_pts <- ggplot(data = tib, aes(x = var_A, y = var_B, fill = value)) +
  geom_point(pch = 21, size = 60, color = "grey90") +
  scale_fill_manual(values = pal) +
  guides(fill = 'none') +
  coord_fixed() +
  labs(x = "Human density", y = "Animal density") +
  theme_minimal() +
  theme(axis.text = element_blank())
```

#### 11. Combine Legend and Map

The final map and legend are combined into a single plot using `patchwork`:

```R
leg_tile + p2

layout <- c(
  area(t = 8, l = 1, b = 10, r = 2),
  area(t = 1, l = 1, b = 10, r = 10)
)

leg_tile + p2 + 
  plot_layout(design = layout)
```

### Output

The final output is a bivariate choropleth map that shows the relationship between human population density and cow density, along with a custom legend.

### Customizing the Code

- **Thresholds:** You can adjust the thresholds for human and cow density to change the classes used in the choropleth map.
- **Color Palette:** The color palette (`pal`) can be modified to suit different visual preferences.

### License

This script is open source and available for modification and redistribution under the MIT License.
