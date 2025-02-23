# AIS based prediction of vessel status

## Repository Link

[https://github.com/annewoelfl/AIS]

## Description

AIS data is transmitted and recorded in vast quantities.
Therefore it is a convenient source for training models that are envisioned to perform tasks like predicting:
- the current (or past) status of a vessel
- the future path or reaction of the vessel
- movement patterns of different vessel types

with the purpose to have a useful application for safety or security evaluations (early warning for coastal authorities, detecting illegal fishing or as additional information for other mariners on their gadgets, e.g. radar) or maybe to develop realistic behaviour patterns for synthetic traffic in simulators.

The scope of this project here is limited to a specific binary classification: Between vessels engaged in fishing and vessels underway using engine. For this task the models are trained on either 1-min records with 10sec intervals or 2h records with 1min intervals containing only course, heading and speed.

The 15-min timesets were created and used primarily on a separate branch. Only incorporating the best models to main.

The datasets come from the Danish Maritime Authority under http://web.ais.dk/aisdata/

### Task Type

[Time series binary classification]

### Results Summary

- **Best Model:** [LSTM_3_2 on the 15min dataset}
- **Evaluation Metric:** [accuracy, precision, recall, f1-score]
- **Result:** [89%, , , ]

## Documentation

1. **[Literature Review](0_LiteratureReview/README.md)**
2. **[Dataset Characteristics](1_DatasetCharacteristics/exploratory_data_analysis.ipynb)**
3. **[Baseline Model](2_BaselineModel/baseline_model.ipynb)**
4. **[Model Definition and Evaluation](3_Model/model_definition_evaluation)**
5. **[Presentation](4_Presentation/README.md)**

## Cover Image

![Project Cover Image](CoverImage/cover_image.png)
