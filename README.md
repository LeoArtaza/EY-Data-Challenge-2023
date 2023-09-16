# EY Data Challenge 2023
In this data competition, the goal was to predict rice harvest based on satellite images from the Sentinel-2 space mission. The training data consisted of the following locations, with the white dots representing the test data:

![image](https://github.com/LeoArtaza/EY-Data-Challenge-2023/assets/57342159/396ded6b-124f-4274-b53b-f9e40ebb6b08)

First, the images were downloaded through STAC's API, using a custom bounding box for each area and applying cloud filtering to capture only the relevant data:

![image](https://github.com/LeoArtaza/EY-Data-Challenge-2023/assets/57342159/5b9e9b95-22fa-47e5-9184-530cadfdeb13)

To extract information from the photos, median values (which could also be mean, max, or min) were calculated for each of the satellite bands and for each image. Additionally, the data was resampled to every 5 days because the satellite's passage over the area is not consistent. Missing values due to cloudy days were also interpolated, resulting in the following dataset:

![image](https://github.com/LeoArtaza/EY-Data-Challenge-2023/assets/57342159/7a38d7d2-8a6d-4c0a-9cb8-7b17e58b3f7b)

It's worth noting that various metrics were calculated based on the original bands, such as the NDVI (Normalized Difference Vegetation Index), NDWI (Normalized Difference Water Index), NDBSI (Normalized Difference Bare Soil Index), and NDRE (Normalized Difference Red Edge Index). Each of these metrics provides different information about the crop's status:

$\text{NDVI} = \frac{\text{NIR} - \text{Red}}{\text{NIR} + \text{Red}}$  
$\text{NDWI} = \frac{\text{Green} - \text{NIR}}{\text{Green} + \text{NIR}}$  
$\text{NDBSI} = \frac{(\text{Red} + \text{SWIR}) - (\text{NIR} + \text{Blue})}{(\text{Red} + \text{SWIR}) + (\text{NIR} + \text{Blue})}$  
$\text{NDRE} = \frac{\text{NIR} - \text{Red Edge}}{\text{NIR} + \text{Red Edge}}$  

A regression task on rice yield is performed using the time series as data. Because our data samples are 2D (sequence length, satellite_bands) this task is ideal for a neural network, for which both a CNN and also a Transformer architecture were implemented with the idea of finding patterns in the time series. Additionally, aggregated data in tabular form was incorporated into the neural network, in parallel with the time series, with statistical descriptors of the time series such as standard deviation, skewness, kurtosis, etc.

![image](https://github.com/LeoArtaza/EY-Data-Challenge-2023/assets/57342159/8c942034-db3a-42b4-b7a2-7d38ab577416)

However, with the aggregated data in tabular form, a gradient boosting algorithm can be used, such as CatBoost, or LightGBM (the 2 best performers for this case). This method turned out to be surprisingly more effective. This could be either because the images, and therefore the time series data, are highly noisy and thus not that useful, or simply the fact that the evolution of these metrics over time is not a determining factor of rice yield.

![image](https://github.com/LeoArtaza/EY-Data-Challenge-2023/assets/57342159/a3cf735d-a183-4dee-b30b-d6d970ac7031)
