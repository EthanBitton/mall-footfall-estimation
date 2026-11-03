# Estimating Shopping-Mall Footfall from Mobile-Phone Pedestrian Flows

> CentraleSupélec · AI track · Client: **MyTraffic** (footfall-analytics startup) · 2023 · Team of 5

MyTraffic turns mobile-network GPS pings into pedestrian flows on a nationwide grid of tiles. The data is aggregated so that it stays GDPR-compliant, because raw location data must be deleted after 3 months. The client's question: **how much footfall does a shopping mall get if it is not yet in their database?** We built a geospatial feature pipeline and regression models that estimate a mall's footfall from the shape of its polygon and the surrounding street flows. With only ~10 labeled malls, the model gives a **useful order of magnitude**: R² ≈ 0.81 in-sample, with leave-one-out errors from ~20% to much larger. Most of the work was about extracting signal from very small data and being honest about its limits.

Code and data are not published: the client's data is confidential.

---

## Business problem

MyTraffic's retail clients choose store locations and evaluate acquisitions using footfall estimates. Street-level pedestrian flows were available. Footfall *inside* a mall was only known for malls whose owners shared their counts.

**Question:** given only a mall's outline (a polygon) and the tile-level pedestrian flows around it, can we infer its footfall and rebuild its history?

## Approach

```mermaid
flowchart LR
    A["Tile-level<br/>pedestrian flows"] --> C["Spatial join<br/>tiles within buffered polygon"]
    B["Mall polygon<br/>vertices"] --> C
    C --> D["Features<br/>distance-weighted flow sum, mean, std,<br/>area, area², perimeter, compactness"]
    D --> E["Regression<br/>least squares, gradient descent, polynomial"]
    E --> F["Footfall estimate"]
```

1. **Geospatial pipeline.** We computed polygon geometry with Shapely, then **buffered** each polygon so that flows on adjacent streets are also captured. Flows from the tiles inside the buffered polygon are then aggregated.
2. **Distance weighting.** Each tile is weighted by its distance to the polygon's centroid, relative to the farthest boundary point. The assumption is that passers-by near the core of the mall are more likely to enter.
3. **Feature engineering.** For each polygon we computed:
   - the weighted flow sum, mean flow and standard deviation;
   - area, area² and perimeter;
   - a **compactness index**: the polygon's area compared with a circle of the same perimeter. It separates regular mall footprints from irregular ones.
4. **Models.** We fitted the following, in order:
   - ordinary least squares;
   - linear regression by **gradient descent implemented from scratch**, validated on a toy problem;
   - polynomial regression with scikit-learn;
   - a size-based split into two models, abandoned because each half had too few samples;
   - **leave-one-out cross-validation** to make the most of the data.
5. **Client-facing visualization.** We built interactive Folium maps with marker clusters and heatmap layers, so the client can see where pedestrian flows concentrate around a site.

## Results

| Model | Result |
|---|---|
| 7-feature linear regression | R² = 0.81 (in-sample, 6 malls with non-zero footfall) |
| Leave-one-out cross-validation | Held-out error from ~20% on some malls to very large on others |

The model is good enough to give an **order of magnitude** that can be applied to any polygon in France. It is not reliable enough for site-level decisions.

## My contribution

- Data processing, visulisation of each mall on maps 
- Prediction models : linear regression with gradient descent (made by hand without libraries for learning purpose); Polynomial regression with scikit learn  

## Stack

`Python` · `Shapely` · `Folium` · `NumPy` · `scikit-learn`
