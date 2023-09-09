# EY Data Challenge 2023
In this data competition, the goal was to predict rice harvest based on satelite images from the Sentinel-2 space mission.
These were the spots for the training data, with the white dots being the test data:

![image](https://github.com/LeoArtaza/EY-Challenge-2023/assets/57342159/bf107c5e-aba0-4112-9c66-81153e672dad)

First, we download the images through STAC's API, using a custom bounding box for each area, and using cloud filtering to capture only what we are interested in:

![image](https://github.com/LeoArtaza/EY-Challenge-2023/assets/57342159/4a63a638-07ba-4c6a-ad4b-856af4cac1e1)

To make use of the photos' information, we take the median values (could also be mean, max, or min) for each of the satellite bands, for each of the images. I also need to resample by 5 days, because the passing of the satellite through the area is not consistent. Then I interpolate through missing values due to cloudy days. We end up with this data:

![image](https://github.com/LeoArtaza/EY-Challenge-2023/assets/57342159/b75c8f7c-efea-48ab-a5a5-a4efea961e14)

Notice that I also calculated multiple metrics which are based on the original bands, such as the NDVI (Normalized Difference Vegetation Index) and NDWI (Normalized Difference Water Index), NDBSI (Normalized Difference Bare Soil Index), and NDRE (Normalized Difference Red Edge Index), each providing different information about the status of the crop:

$\text{NDVI} = \frac{\text{NIR} - \text{Red}}{\text{NIR} + \text{Red}}$  
$\text{NDWI} = \frac{\text{Green} - \text{NIR}}{\text{Green} + \text{NIR}}$  
$\text{NDBSI} = \frac{(\text{Red} + \text{SWIR}) - (\text{NIR} + \text{Blue})}{(\text{Red} + \text{SWIR}) + (\text{NIR} + \text{Blue})}$  
$\text{NDRE} = \frac{\text{NIR} - \text{Red Edge}}{\text{NIR} + \text{Red Edge}}$  

With these time series, we need to perform a regression problem on rice yield. Because our data samples are 2D (sequence length, n_features) this task is ideal for a neural network, for which I coded a CNN and also a Transformer architecture with the idea of finding patterns in the time series. Additionally, I also added aggregated data of the time series in tabular form to the neural network in parallel to the time series, with statistical descriptors such as standard deviation, skewness, kurtosis, etc.

![image](https://github.com/LeoArtaza/EY-Challenge-2023/assets/57342159/668adcb6-15e7-463d-90c0-9b2f9c7afadd)

However, with the aggregated data in tabular form, we can use a gradient boosting algorithm, such as CatBoost, or LightGBM (the 2 best performers for this case). This method turned out to be surprisingly more effective. This could be either because the images, and therefore the time series data, are highly noisy, or simply the fact that the evolution of these metrics over time is not a determinant factor of rice yield.

![image](https://github.com/LeoArtaza/EY-Challenge-2023/assets/57342159/32b2099c-149b-4f6e-b521-7634db836d10)
