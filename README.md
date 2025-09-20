# Nigeria-Geopolitical-Map
R script for creating Map of Ngieria with Geopolitical Zones and States

############################################################
# Map of Ngieria with Geopolitical Zones and States
#
# Author: Adewuyi et al 2025
# Date: September 2025
# Version: 1.0
# Citation: 
#
# Description:
# This script generates a map of Nigeria showing its 36 states
# and the Federal Capital Territory (FCT), grouped by the six
# official geopolitical zones. State boundaries are obtained
# from the Natural Earth dataset via the rnaturalearth package.
#
# Features:
# - Assigns each state to its geopolitical zone
# - Labels all states using centroids (with ggrepel to avoid overlaps)
# - Custom colour palette for each zone (excluding red/green)
# - Debugging step to check state name consistency
#
# Requirements:
# R >= 4.0
# Packages: sf, ggplot2, dplyr, rnaturalearth, rnaturalearthdata, ggrepel
#
# Usage:
# 1. Install required packages (see README.md).
# 2. Run the script to generate the map.
# 3. Save the output with ggsave() if desired.
#
############################################################

# Load libraries
library(sf)
library(ggplot2)
library(dplyr)
library(rnaturalearth)
library(rnaturalearthdata)
library(ggrepel)

# Load Nigeria shapefile (Natural Earth states)
nigeria <- ne_states(country = "Nigeria", returnclass = "sf")

# Print unique state names for debugging
print(unique(nigeria$name))

# Define the geopolitical zones and their states
geopolitical_zones <- list(
  "North Central" = c("Benue", "Kogi", "Kwara", "Nassarawa", "Niger", "Plateau", "Federal Capital Territory"),
  "North East"    = c("Adamawa", "Bauchi", "Borno", "Gombe", "Taraba", "Yobe"),
  "North West"    = c("Jigawa", "Kaduna", "Kano", "Katsina", "Kebbi", "Sokoto", "Zamfara"),
  "South East"    = c("Abia", "Anambra", "Ebonyi", "Enugu", "Imo"),
  "South South"   = c("Akwa Ibom", "Bayelsa", "Cross River", "Delta", "Edo", "Rivers"),
  "South West"    = c("Ekiti", "Lagos", "Ogun", "Ondo", "Osun", "Oyo")
)

# Assign geopolitical zones to states
nigeria <- nigeria %>%
  mutate(Geopolitical_Zone = case_when(
    name %in% geopolitical_zones[["North Central"]] ~ "North Central",
    name %in% geopolitical_zones[["North East"]]    ~ "North East",
    name %in% geopolitical_zones[["North West"]]    ~ "North West",
    name %in% geopolitical_zones[["South West"]]    ~ "South West",
    name %in% geopolitical_zones[["South East"]]    ~ "South East",
    name %in% geopolitical_zones[["South South"]]   ~ "South South",
    TRUE ~ NA_character_
  ))

# Compute centroids for state labels
nigeria_centroids <- nigeria %>%
  st_centroid() %>%
  mutate(lon = st_coordinates(geometry)[, 1],
         lat = st_coordinates(geometry)[, 2])

# Define custom colours for each geopolitical zone
zone_colors <- c(
  "North West"   = "#1f78b4",
  "North East"   = "#ff6347",
  "North Central"= "#ffcc00",
  "South West"   = "#ff7f00",
  "South East"   = "#6a3d9a",
  "South South"  = "#b15928"
)

# Plot the map
map_plot <- ggplot(data = nigeria) +
  geom_sf(aes(fill = Geopolitical_Zone), color = "black", size = 0.2) +
  geom_text_repel(data = nigeria_centroids, 
                  aes(x = lon, y = lat, label = name), 
                  size = 3, color = "black", fontface = "bold", max.overlaps = 50) +
  scale_fill_manual(values = zone_colors, na.value = "grey90") +
  theme_minimal() +
  labs(fill = "Geopolitical Zone")

# Display plot
print(map_plot)

# Save plot (uncomment to save as file)
# ggsave("nigeria_geopolitical_map.png", map_plot, width = 10, height = 8, dpi = 300)

############################################################
# End of Script
############################################################
