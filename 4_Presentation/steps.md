1. **Introduction**: Briefly introduce the problem you are solving, the dataset used, and the type of task (classification, regression, clustering, etc.).
-->description see chat?
-->dataset danish accumulation of lots of AIS reports (maybe show distribution over a map, example from 30.11. and 30.05.? and then the selected areas?)

2. **Literature Review**: Indicate solutions you've found that are related to your project.
--> see literatur review

3. **Dataset Characteristics**: Provide an overview of your dataset, including any preprocessing and feature engineering steps:

   - grouping by vessel
   - filtering by area (area selection where there is open sea, no confusing port, and seperately one anchorage area)
   - creating XXmin sets of equal Navigational Status (variable), these contain reports of several things at irregular times, each entry has: **Timestamp**,Type of mobile, MMSI, Latitude, Longitude, **Navigational status(TARGET)**, ROT, **SOG**, **COG**, **Heading**, IMO, Callsign, Name, Ship type, Cargo type, Width, Length, Type of position fixing device, Draught, Destination, ETA, Data source type, A, B, C, D, Time_Group
    - from the above I plan to use the items in bold writing because the model should be as general as possible and features like coordinates would work against that.
   - Missing values: Interpolation for Columns with Less Than 50% Missing Values: For each file, after deleting the files with more or equal than 50% of a columns missing, we’ll check the percentage of missing values per column and apply interpolation after time for those columns where less than 50% of the values are missing.
   - standardizing the sets to exactly the same length and time interval (example for 15min: 14:50 exactly and 10s interval)
   - deleting columns from tables that are not needed like positions.
   - setting the initial course to zero for the datasets to be better comparable. That means from every course and heading value the initial course value is subtracted
   - deleting not needed columns

4. **Baseline Model**: Quickly recap your baseline model, its performance, and why it was chosen.
- random forest (here I split two-ways: 1 model looks at every line and 1 model looks at every time-set (aggregated) sepearately)
- tested on 15min and 2h sets
- results were: between 0.68 and 0.78 in accuracy, but some classes were not recognized at all. (Reminder:insert confusion matrix image)
- approaches with the position (Lat/Lon) still in held better results, but still certain classes were not recognized
- added a lot of visualizations and stuff to understand input - output and feature importance (Reminder: add feature importance image)

5. **Model Definition and Evaluation**:
  - working on LSTM
    - with position the model had moderate results, but as it should be possible to apply it anywhere, it is necessary to ignore posittion and see only movement
    - only movement (starting point as 0) did not have any good results (not learning), even worse when class weights were introduced, commented out class weights again
    - tried normalizing each column separately - not very different
    - tried looking at only differentiating between "Under way using engine" and "Engaged in fishing". -with class weights: medium results - without class weights, with undersampling: medium results. 
    - after some unsatisfying results I found that datasets from all vessels got thrown together in one database, predictions were then made not with the set of one vessel but with one timestamp, which was not what I intended, so I changed the approach by combining each 15min-dataset of one vessel into a vector which is then combined in a database where each row (each vector) represents a 15min time-slot of one vessel with a uniform nav-status. This instantly got better results. (aggregated values only)
    - tried keras tuner with this vectorized model. Frustratingly results were not better than the starting point.
    Noticed the last layer used "softmax", which was probably from the earlier multi-classification approach. I changed it to "sigmoid" and let the tuner and the LSTM without tuner run again. Result is worse, reverting to "softmax"
    - noticed that even when clearly stated that it is binary classification, chatGPT tended to use softmax in suggestions quite often, the results don't differ very much, so it seems using sigmoid is not so important after all
    - reviewing exactly how the data is "vectorized", maybe information goes missing.
    - tried instead only to flatten the input datasets (each vessels 15-min set goes into one row but this time complete, not in the form of mean values or slope). Interesting that it runs as fast as before (must be a lot more values) Results were very much the same as before.
    - Trying a different architecture (CNN for example?) - CNN brought same result (more overfitting)
    - I thought I had already introduced the subtracting of the initial course (so that it doesn't get evaluated which initial direction the vessel is going) I can't find it again
    - wasn'sure if I normalized the values correctly, revisited that: results got significantly better for 1_5
    - build a new tuner approach (the old had not really given any fruitful results as can be seen above)
    - added subtraction of initial course and cyclical encoding of course values to 1_5, 1_6 and the baseline models
    - added flattening again (1_5f) (see above, it was somehow removed), this time I notice strong divergence of training and validation loss and accuracy. with the flattened data the model tends to overfit much stronger.
    - many attempts have now always revolved around the accuracy of 0,8.
    - maximum confusion, noticed that previously used models 1_5, 1_5f, 1_6 were no LSTMs after all? LSTMed 1_5f again. Similar results. Just takes longer. And a lot of overfitteing (since flattening)
    - As I kind of reached a dead end with the earlier approaches and could not reliably say that the data is processed the right way / a meaningful way, I try a new approach from scratch. It has around the same results, but more even distribution of false pos/neg
    - also put the filtering out of columns and the subtraction of the initial course in the preprocessing process (seperate folders) for better control
    - ran baseline model again with current configuration of dataset and it got same results as the NN with aggregated values, so either I manage to get some good results out of a Deep Learning model with every timestep considered or I can abandon that (Deep Learning with the aggregated features is definitely overkill)
    - experimented with image recognition instead (first result not so good, 0,75)
    - Processing the files and images also had the advantage that i could look through them easily and notice any special features
    - When looking at the images i noticed fishing vessels having high speed and straight course i have to check if maybe those are wrong datasets. Notice that high speeds were present about one hundred knots have to check that too. Maybe with a refined dataset i can achieve better results.
    - The checks revealed that there were not so many of these assumed too fast vessels. (~10 files)
    - Additionally this was the first time i could really visually see what the model was doing when i saw the images off the time sets by quickly looking through them i could see that there is more variance in the engine sets than i thought and i saw what could have been the problem before that is of course there are a lot of vessels going straight ahead with eight knots or something in the one category and as well in the other category for those cases it would really be necessary to look at longer time slots to be able to differentiate them.
    - so with the existing models i think that i am at the end of the developments of models with a small time slot and could experiment further with longer time sets, e.g. 2h with minute steps.
    - I set up the two hours database which was quite easily done only one click but it took a long time like four hours to compute.
    - On this new dataset i tried the old models and disappointingly i didn't get better results.
    - I tried to visualize the new data said again in the map tiles.
    - This time i would say the difference between the two classes is clearer mostly because of speed.
    - In the map tiles i introduced some changes i removed the background it is now transparent. I introduced color coding for speed called speed color coded trajectories.
    - Iran these images through the image model again results were first around zero point seven and later a little bit better than the fifteen minutes sets at around zero point eight but still not satisfying.
    - The color coding was a good step i think because no additionally to the path you can also see that speed of the vessel in the images.
    - As all the models did not produce any better results than the fifteen minutes sets i tried again to write a new model (2_2) from script with the help of chatGPT
    - I did not expect much of the new approach but the first time i ran it over the database it seemed to produce the breakthrough i had hoped for. Results are now 0.99 in accuracy, f1 score and precision. Recall at 0.98. Intensive trials with the code showed no problems so the result seems valid.
    - By looking through the changes i noticed for the first time that a sliding window approach was used. That might have been the most important optimization.
    - A detailed description of how the now best performing model is handling the data is included in the file.
    - Tried training the 2_2 with 15min sets and a sequence length of 30 --> 0.85, not perfect but still the best within the 15min set models.
    - Tried 2_2 with 15min sets early stopping to 15 and sequence length to 45 (thats the half of the time of the set, like in the 2h sets with 60)
    - Was interested to see how the model trained on the 2h sets would perform on the 15 min set (to evaluate the need for a different training for the dynamic approach (see future work below)), but as the data input structure is different, it was not possible to directy use the models interchangeably, to have a useable product, I would need to produce a dynamic model somehow.
    - trying to assess the wrongly categorized csv and then display the corresponding color-coded trajectory so that the results are easily visible.
  - Crunched a lot of data to get about 1000 datasets for sailing-engine or multi-classifier
  - built sailing-engine binary classifier
  - to check validity built sanity-check branch
  - sanity check done by just putting random underway-using-engine files in both folders. The randomly selected files of one class should have given an output accuracy of about 0.5.
  - sadly the output accuracy was way higher (~0.8X), that implies there is something fucky with the code
  - found out that
  - using train_test_split() to create a test set, then within .fit(), you're splitting again (validation_split=0.2) This means 20% of the training set is used for validation, which may lead to data leakage.
  - splitting sequences at the sequence level (train_test_split(X, y, ...)), multiple sequences from the same vessel (or same file) could be in both the training and validation/test sets.
  - additionally noticed that sequences are not starting with course zero and are thus not really comparable
  - try to correct these potential sources for the overperformance

8. **Results**: Present the results in a clear and easy-to-understand format. Use tables, charts, or any other visual aids that you find appropriate.

9. **Challenges and Errors**:
    - Challenge with data size (one single day was several GB) and even after crunching down 12 days I didn't have enough sets for specific vessels, esp. sailing and anchoring.
    - Challenge with missing values and standardization of datasets took a long time.
    - Both of the above points lead to the model not being very flexible (e.g. changing the timeslot from 15min to a shorter or longer period would make all the data preprocessing necessary again, although I think the script for that is quite easily usable, but it would take time to calculate)
    - Hyperparameter tuning (also with keras tuner) seemed not to be worth the effort as the results were not significantly better than the starting point.
    - Difficulty with the shape of the data, should one use aggregated or flattened data, how are the timestamps seen by the model? And what is the best model architecture to use?
    - for a long time i did not understand how the data had to be processed and how it is used in the model (time series for binary classification) until at last it worked and i let the working model be explained to me again.

10. **Discussion**: Reflect on the performance of your models compared to the baseline, and discuss any limitations and future work.
    - Future Work
       - More vessel categories
       - Approach the timeslot more dynamically (e.g. for a radar application you rarely have 2h of information, but it slowly builds up, so you could step by step improve the quality of the prediction the more data it collects, starting at 5min, then 15, then 30 or even fluent without fixed steps "sliding window which is growing bigger with time"??) (training could maybe work by creating varying timesets from 5min to 1 or 2h?, then scaling the timesteps and the sliding window dynamically?)
       - more features like wind
       - Predict vessels movement like in literature review source number three.

12. **Q&A**: Be prepared to answer questions from the audience or instructor.
