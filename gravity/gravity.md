---
layout: page
title: Gravity Model of Spatial Interaction
---

[Gravity Model of Spatial Interaction](C:\github\stevenmontilla.github.io\gravity\models/GravityModel.model3):
This model calculates spatial interaction catchments using the gravity model of spatial interaction.
Catchments are calculated by measuring the maximum potential for interaction between an input feature
and a target feature. The model takes into account three factors that can be customized: attractiveness of target
features (α), friction of distance (β) and emissivity of input features (λ). All input features that have highest
 potential interaction with the same target feature are dissolved into a spatial interaction catchment feature of
  the same layer format as the input features/layer.

![Gravity Model](assets/GravityModelofSpatialInteraction.png)

[Preprocessing of target feature layer for Gravity Model](C:\github\stevenmontilla.github.io\gravity\models/targetFt.model3).

This model takes in the target and input features to be used in the Gravity Model of Spatial Interaction to be preprocessed.
All target features found within the same input feature are aggregated and summarized into a centroid (calculated from dissolved target features).
The output is a target layer is a filtered version of the target features that conserves all attribute fields.

![Target Features Preprocessing](assets/Preprocessing of Target Features for Gravity Model.png)

[Preprocessing Model of Homeland Security Hospital data](C:\github\stevenmontilla.github.io\gravity\models/Homeland.model3):
The only purpose of this model is to prepare Homeland Security Hospital Data to exclude military, psychiatric, closed hospitals and null weights
features with the following expression: "TYPE" != 'MILITARY' AND "TYPE" != 'PSYCHIATRIC' AND "BEDS" > 0 AND "STATUS" = 'OPEN'.
 This model is specifically design for the Homeland Security Hospital data

 ![Target Features Preprocessing](assets/Preprocessing Homeland Security Hospital Data.png)


Data sources and References
[Homeland Security Data]( found [here](https://services1.arcgis.com/Hp6G80Pky0om7QvQ/ArcGIS/rest/services/Hospitals_1/FeatureServer)
Dartmouth Atlas Data:
[Dartmouth Hospital Service Areas](https://atlasdata.dartmouth.edu/downloads/supplemental#boundaries)
Dartmouth Atlas User Agreement:
The data set forth at [this map](assets/) of publication/press
release was obtained from Dartmouth Atlas Data website, which was funded by the Robert Wood Johnson Foundation,
The Dartmouth Clinical and Translational Science Institute, under award number UL1TR001086 from the National Center
for Advancing Translational Sciences (NCATS) of the National Institutes of Health (NIH), and in part, by the National
Institute of Aging, under award number U01 AG046830."


interpretation of model results vis-a-vis the Dartmouth Health Atlas

link to [our map](assets/)!
