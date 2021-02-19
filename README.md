# Pickup/Dropoff Zones in Boston
An analysis of Boston Pickup/Dropoff location data to inform the choice of designated rideshare zones

## Background

The Boston Transportation Department is interested in piloting designated pickup/dropoff zones for rideshare activity. Limiting rideshare pickups and dropoffs to reserved spaces will improve safety and traffic, and has potentially dramatic impact in some particularly clogged regions of Boston.

The data for this project comes from a tool built by [SharedStreets](https://sharedstreets.io/) for the Greater Boston Area. The tool displays rideshare activity aggregated for anonimization purposes originally provided by Uber and Lyft. The data was exported in the form of a geojson and trimmed to only include activity occuring within the city boundaries of Boston. The activity is captured in an attribute called "periodAvgCount" which gives the quantity of pick-up and drop-off instances, normalized for the time period and region chosen, and aggregated over a small street segment.

## Clustering Analysis

In seeking hotspots of PUDO activity, we interpreted “clusters” as areas of high density surrounded by regions of relatively low density.  This suggests the use of a [DBSCAN algorithm](https://scikit-learn.org/stable/modules/generated/sklearn.cluster.DBSCAN.html#sklearn.cluster.DBSCAN), rather than a more traditional approach such as K-means clustering.
- DBSCAN stands for Density-Based Spatial Clustering of Applications with Noise
- DBSCAN has the advantage that clusters may be of any size and shape, rather than being forced to be convex, and the areas separating clusters are considered “noise”
- Two key parameters to define “density”: For each sample to be included in the cluster there must be a minimum number of samples `min_samples` within a distance `eps`
- “A cluster is a set of core samples that can be built by recursively taking a core sample, finding all of its neighbors that are core samples, finding all of their neighbors that are core samples, and so on. A cluster also has a set of non-core samples, which are samples that are neighbors of a core sample in the cluster but are not themselves core samples. Intuitively, these samples are on the fringes of a cluster.”
- So we have two ways to adjust our clusters: we can require more/less activity, or we can broaden/tighten the neighborhood of samples
- For more information on clustering algorithms [see here](https://scikit-learn.org/stable/modules/clustering.html)

## Methods

This analysis focuses only on pick-up activity, expecting rideshare pick-ups to require more dwell time at the site of the pick-up. Drop-off analysis would follow the same procedure.

1. Read in trimmed data from Shared Streets (above) and Boston neighborhood boundary shapefile
2. Divide City of Boston into regions based on neighborhood.  The regions are chosen based on proximity and similar PUDO activity patterns
3. For each region, the DBSCAN algorithm is run using parameters chosen to produce clusters of a manageable size (a few blocks or less).  Because activity may be more concentrated or more spread-out depending on the neighborhood, the parameters are chosen manually for each neighborhood.  The samples are weighted by the `periodAverageCount` from Shared Streets.
4. The clustering analysis outputs a csv file containing the clusters for each region.  Different regions have different numbers of clusters.  Each cluster contains the following attributes:
`lat`, `lon`: The latitude and longitude of the cluster midpoint
`neighborhood`: The region of the cluster
`clusters`: The number of core samples included in the cluster.  This is a measure of how wide-spread the cluster is.
`density`: The sum of periodAverageCount for all samples contained in the cluster.  This is a measure of the raw activity.
`spread`: 100*`density`/`clusters`.  This is a summary measure--high spread indicates a relatively intense, concentrated cluster, low spread indicates a relatively weak, diffuse cluster.
