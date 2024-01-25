---
title: "🏋🏻 Building a Choice Predictor with Streamlit and XGBoost)"
excerpt: "Building an interactive web application where a user makes binary choices and an ML algorithm learns from these decisions to predict the user's next selection with real-time feedback on the algorithm's prediction accuracy.
<br/><img src='https://github.com/m-guseva/m-guseva.github.io/blob/master/images/thumb_choicePredictor.jpg?raw=true' width='300'>"
collection: portfolio
---


# Building a Choice Predictor with Streamlit and XGBoost


## Background
In my research experiments I asked people to make many binary choices between "heads" and "tails" of a coin (for more details, see our publication [here](https://www.frontiersin.org/articles/10.3389/fpsyg.2023.1113654/full)). The participants were specifically instructed to make these choices randomly, without following any pattern. We looked at how well people performed the task and explored what happens in their brain while doing it.

As a side hobby project I decided to build a similar experiment in a web application using streamlit, which is handy a Python library that turns data scripts into shareable web apps. The program (I call it the ✨choice predictor✨) asks users to make multiple binary choices (between 1 or 0), and the algorithm learns from these choices to predict the next one.
The full code can be found in [this github repo](https://github.com/m-guseva/choice-predictor). Here I explain step by step how I designed this application.

The programm is deployed and can be accessed here: **[choicepredictor.streamlit.app/](choicepredictor.streamlit.app/)**

[<img src="https://github.com/m-guseva/choice-predictor/blob/main/Layout.jpg?raw=true" width="900"/>](image.png)


## Procedure in a nutshell
1. The user can choose the algorithm that will be used for the prediction. Possible choices: xgboost, logistic regression or random forest. The default algorithm is xgboost and will be used if no choice is made.
2. The user makes a set amount of choices by repeatedly pressing one of two buttons to create an initial training set.
3. As soon as this initial set is gathered, the algorithm does a parameter search to find optimal number of features.
4. This optimal number of features is used to make a prediction for the user's next choice. 
5. After every X steps (X is determined by the `interValToTest` constant), a new parameter search is run and the newly found optimal parameter is used in the updated model for the following predictions.

Additionally, some metrics are displayed, e.g. the % of correct predictions, with an indicator of whether and by how much the percentage changed compared to the previous one. A diagram illustrates the percentage metric over time.


## Packages
```python
import pandas as pd
import streamlit as st
import numpy as np
from xgboost import XGBRegressor
from itertools import chain
from sklearn.linear_model import LogisticRegression
import seaborn as sns
import matplotlib.pyplot as plt
from sklearn.ensemble import RandomForestClassifier
```

##  Initialize variables and  streamlit configs
``` python
st.set_page_config(layout="wide")

# How long should model learn in the beginning before making first prediction:
initSeqLen =18
# Interval in which the  model should be retrained
interValToTest = 1

sequence = []
button_sequence = []
sequence_str= []
perc_correctPredictions = []
```

Because streamlit is structured such that it refreshes anytime a widget is changed or a button is pressed, we need to make use of state session variables to keep the information across reloads.

```python
# Initialize session state variables
if 'button_sequence' not in st.session_state:
    st.session_state.button_sequence = []
if 'which_model' not in st.session_state:
    st.session_state.which_model = []
if 'perc_correctPredictions' not in st.session_state:
    st.session_state.perc_correctPredictions = []
if 'featureList' not in st.session_state:
    st.session_state.featureList = []
if 'predictionList' not in st.session_state:
    st.session_state.predictionList = []
```

## Basic streamlit interface

This is what is happening at the front end, it's very simple. The layout contains 6 columns. In the leftmost column is an image of  a fortuneteller that I generated with Midjourney. Column 2 is dynamic and contains the initial model choice radio buttons that disappear after one choice is made, later this column displays the choice and the predictions. The choice buttons are in columns 3 and 4. 
There is also a metric that shows the % correct predictions (column 2) and a diagram in column 6.

``` python
st.title("Choice predictor")
string = '''
1. Choose the algorithm to use for prediction. If you don't choose anything, then the default algorithm will be xgboost. 
2. Start making your sequence of choices by pressing the buttons 1 or 0 many times. Make sure to vary
your choices because if you  use only :red[one] of the buttons, the algorithm will throw an error!
3. After the algorithm has learned, it will start predicting your choices, the prediction accuracy will be shown on the right of the screen.
'''
st.markdown(string)

# Create layout
col1,col2, col3, col4, col5,col6 = st.columns([0.05,0.05,0.3,0.2,0.3,0.3])
col3.image("fortuneTeller.png")

# Buttons
left_symbol = "1"
right_symbol = "0"

# Make some space above buttons for better layout
for _ in range(3):  
    for col in [col3, col4]:
        col.write(" ")
        col.write(" ")

left_button = col3.button(left_symbol,key=1,use_container_width=True)
right_button = col4.button(right_symbol,key=0,use_container_width=True)
```

### What happens when the buttons are triggered
When either of the buttons is triggered, a function `gatherChoiceSequence()` is called, that concatenates all choices into one list of integers. If the person exclusively uses only of the buttons, the algorithm cannot learn as it need variability in the training set, an error is displayed and the user is prompted to refresh the app.

``` python
if left_button:
    sequence = gatherChoiceSequence(left_symbol)
elif right_button:
    sequence = gatherChoiceSequence(right_symbol)

# Handle case where person presses only one button
if (len(sequence) >= initSeqLen) & (len(set(sequence[:initSeqLen])) == 1):
    with col2:
        st.error("It seems like you have chosen only one of the options! 😱  The algorithm needs you to make choices between both 1 and 0! ☝🏻 Please refresh the page to start anew.", icon="‼️")
        st.stop()
```
```python
def gatherChoiceSequence(symbol):
    sequence = st.session_state.button_sequence.append(symbol)
    sequence_str = ' '.join(map(str, st.session_state.button_sequence))
    sequence = [int(x) for x in sequence_str if x.strip().isdigit()]
    return sequence
```
The variable which_model determines the model used in the prediction, it is taken from the session state variable (which was instantiated by the radio buttons).
Next `procedure()` is called, which outputs the predicted value given the choice sequence up to that point. This function is explained in detail in the next section. the function `uiElements()` controls the display of relevant metrics and will be explained in the end.

``` python
which_model  = st.session_state.which_model 
prediction = procedure(sequence)
uiElements(sequence)
```

## What is the prediction procedure?
When the sequence is passed to `procedure()`, it first performs a parameter search for the amount of features to use in the model, by calling `whichFeatures()` (will be explained in detail below).

 After the sequence length exceedes initSeqlen (i.e. the user created the necessary initial choice set), a prediction is made, based on the optimal parameter `n_features` as determined in the previous step. This prediciton is appended to the session state variable `predictionList`.

``` python
def procedure(sequence):
    # Determine how many features to use:
    n_features = whichFeatures(sequence) 
    prediction = []
    # Make prediction:
    if len(sequence)>initSeqLen:
        prediction = predictNextChoice(sequence, features=n_features)
        st.session_state.predictionList.append(prediction)
    else:
        st.session_state.predictionList.append([])
    return prediction
```

## How is the prediction done?

The prediction is returned via `predictNextChoice()` using the sequence and number of features as arguments.

```python
def predictNextChoice(sequence, features):
    #1. Create supervised sequence
    supervised_sequence = split_sequence_to_df(sequence, features).values

    # 2. Split into features and target
    X_train, y_train = supervised_sequence[:, 0:-1], supervised_sequence[:, -1] 
    
    #3. Fit model to X_train and y_train and create prediction
    model = XGBRegressor(n_estimators =1000, learning_rate = 0.05, n_jobs = 1)
    model.fit(X_train, y_train, verbose=True)
    last_choices = np.array(sequence [-features:]).reshape(1,features)
    predicted_choice = transform_predictions(model.predict(last_choices))
    return predicted_choice
```
Let's look at this function step by step:
### 1. Create supervised sequence
First we create a supervised sequence. Given that a choice sequence is essentially a time series, we can use lagged versions of the sequence as our features. This is done with `split_sequence_to_df()` 
[INSERT img of split sequence df]. 

```python
def split_sequence_to_df(sequence, n_steps):
    X, y = list(), list()
    for i in range(len(sequence)):
        end_ix = i + n_steps
        if end_ix > len(sequence) - 1:
            break
        seq_x = sequence[i:end_ix]
        seq_y = sequence[end_ix]
        X.append(seq_x)
        y.append(seq_y)
    X_df = pd.DataFrame(X, columns=[f'lag_{i}' for i in range(n_steps)])
    y_df = pd.DataFrame(y, columns=['output'])
    return pd.concat([X_df, y_df], axis=1)
```

### 2. Split into features and target
Next the our variable supervised_sequence which contains the lagged arrays is split into predictors X_train and target y_train.

### 3. Fit model to X_train and y_train and create prediction
Then an XGBRegressor model is fit to predictors and target. Then we take the last choices in our sequence (as determined by number of lagged features) to base our predictions on. We create the prediction and transform it. This step is necessary because the predictions don't come in a binary format, so if the number is <0.5 it becomes 0, else 1.

```python
def transform_predictions(predictions):
    predictions = [0 if pred < 0.5 else 1 for pred in predictions]
    return predictions
```

## Parameter search

The last ingredient is to do a parameter search to determine how many lags to use as features. If our lag is too short, we might not capture autocorrelations that go back more than one step behind (e.g. in a sequence like `1,1,0,0,1,1,0,0,...`). If our lag is too high, then we end up with  very little samples from which the algorithm can learn, since you can shift a sequence only so far.

### At what points in time  is parameter search executed?
This function organizes when the parameter search is being executed:

```python
def whichFeatures(sequence):
    '''Calculates the best number of features to use in model. It takes the first choices up to initSeqLen 
     and calculates initBestFeature. As soon as our sequence grows to a multiplicative of initSeqLen, it recalculates the
      best number of features and returns the new one. The calculated features are saved in a st.session variable so
       that they survive each rerunning of the script. '''
    valuesToTest = [1,2,3]
    initBestFeature = []
    featureToUse = []
    if len(sequence) == initSeqLen:
        initBestFeature = searchFeatureSpace(sequence, valuesToTest)
        st.session_state.featureList = [initBestFeature]
        return initBestFeature
    # Retrain after every 'intervalToTest' steps
    if len(sequence) % interValToTest == 0 and len(sequence) > initSeqLen:
        bestFeature = searchFeatureSpace(sequence, valuesToTest)
         # save feature in session state variable featureList
        st.session_state.featureList.append(bestFeature)
    if len(sequence)>initSeqLen:
        featureToUse = st.session_state.featureList[-1]
    return featureToUse
```
It does nothing while the user still inputs the initial choices up until `initSeqLen`. Then if len(sequenc) == initSeqlen, it starts the parameter search by calling `searchFeatureSpace()`. The best feature is saved in the session state variable `featureList` and returned.

We repeat the parameter search every `ìntervalToTest` steps. The best feature is saved in the running featureList variable. The function returns the last element of this session state variable to use in the prediction procedure.

### How exactly is Parameter search done?
Parameter search is done with the function `searchFeatureSpace()` :

```python
def searchFeatureSpace(sequence, feature_space =[1,2,3,4,5]):
    '''Takes sequence and tests different combinations of features, then outputs the best feature 
    (which is the one with the highest validation accuracy)'''
    validation_accuracies = {}
    for features in feature_space:
        # Reformulate dataset into supervised lagged values set
        supervised_values = split_sequence_to_df(sequence, features).values
        # Run model
        model,validation_accuracy = TrainModel(supervised_values, which_model)
        validation_accuracies[features] = validation_accuracy
    bestFeature = max(validation_accuracies, key=validation_accuracies.get)
    return bestFeature
```
The function takes a sequence and a feature space, with a default list of  1 to 5. It cycles through each element of this list and splits the sequence according to the lag that corresponds to the element in feature space. 

Then it trains the model on the train set (supervised_values) and validates it using walk forward validation (see explanation below). The validation accuracy for each of the features in question is saved in the list and the best feature is simply the one with the highest validaiton accuracy.

```python
def TrainModel(supervised_values, which_model):
    '''
    possible which_model arguments: 'logreg', 'xgboost', 'randomForest'
    Trains the predictive model and evaluates it by walk-forward validation
    '''    
    # Determine change_index
    change_index = np.where(np.diff(supervised_values[:, -1]) != 0)[0]    
    # necessary to determine the size of initial train chunk, because if target is only 1s or 0s model won't run
    train, validate = split_train_validate(supervised_values, change_index[0]+2)
    history = train.copy()
    predictions = []

    # Walk-forward validation
    for t in range(len(validate)):     
        # Get approprieat validation row to validate predictions:
        X_validate = validate[t, 0:-1].reshape(1, -1)
        y_validate = validate[t,-1].reshape(1,-1)
        # Configure model:
        if which_model == 'logreg':
            model = LogisticRegression(max_iter=2000)
        elif which_model =='xgboost':
            model = XGBRegressor(n_estimators =1000, learning_rate = 0.05, n_jobs = 1)
        elif which_model == 'randomForest':
            model = RandomForestClassifier(n_estimators=100, max_depth=None, random_state=None)
        X_train, y_train = history[:, 0:-1], history[:, -1] 
        if which_model == 'xgboost':
            model.fit(X_train, y_train, early_stopping_rounds=5, eval_set=[(X_validate,y_validate)], verbose=False)
        else:
            model.fit(X_train, y_train)
        # Make predictions for the current time step
        yhat = model.predict(X_validate)
        predictions.append(yhat[0])
        # Update the history with the observed value
        history = np.vstack((history, validate[t, :])) #append validation set row to history

    # Evaluate the model accuracy
    if which_model == 'logreg':
        validation_accuracy = np.sum(predictions == validate[:, -1]) / len(predictions)
    elif which_model =='xgboost':
        validation_accuracy = np.sum(transform_predictions(predictions) == validate[:, -1]) / len(predictions)
    elif which_model == 'randomForest':
        validation_accuracy = np.sum(transform_predictions(predictions) == validate[:, -1]) / len(predictions)
    return model,validation_accuracy
```

The training is executed with the function `TrainModel()` #TODO maybe rename as WalkForwardValidation
This function takes the supervised_values and an argument `which_model` which specifies the type of model to use (logreg, xgboost and randomForest are possible). Then it splits the supervised_values into train and validate for use in the walkforward validation procedure.
#### What is Walk forward validation?
Walk-forward validation is a good choice for time series data. The process involves training a model on a subset of the data (from time point t0 to tn) and making predictions for the next time point, tn+1. Following this, the predicted value for tn+1 is compared to the actual observed value. The data point for tn+1 is then incorporated into the training set, and the model is re-fitted, now incorporating this additional data point.

This iterative cycle continues, with the model being updated and predictions made for each subsequent time step. At each step, the accuracy of the model's predictions is assessed by comparing them to the actual values. The average prediction accuracy is then calculated across all steps, providing an overall measure of the model's performance on the entire time series.

The function returns the model object and validation accuracy. 


## Display Stats

The following functions handle the display of choices, predictions and statistics about prediction accuracy.

`uIElements()` manages the display of elements in column 2 and partly 1. First, it prompts the user to select an algorithm and instructs the user to make choices. Once the initial choices is made, it displays the current choice and the computer's prediction. It also informs the user to continue making choices before the prediction begins, with a countdown dynamically indicating the remaining choices before the prediction begins.

```python
def uiElements(sequence):
    if len(sequence) <1:
        col2.write(" ")        
        if (not left_button) and (not right_button):
            with col2:
                modelChoice_radio = st.radio("1️⃣ Pick an algorithm:", ["xgboost", "logreg", "randomForest"], horizontal=False)
                st.session_state.which_model = modelChoice_radio
        else:
            st.write(" ")
        col2.write("2️⃣ Then start making your choices by pressing the buttons on the right!")
    else:
        with col2:
            st.metric(label="▫️YOUR CHOICE▫️", value=str(sequence[-1]))
    if len(sequence)<=initSeqLen+1:
        col1.write("💬 Continue making choices, so I can learn!")
        col1.markdown("Number of choices before I begin predicting:  " + ":red["+str(initSeqLen+2-len(sequence))+"]")
    else:
        # Display computer's prediction:
        col2.metric(label="✨WHAT I PREDICTED ✨", value=int(st.session_state.predictionList[-2][0]))
        displayStats()
    return
```

The `displayStats()` functions uses all the values from the session state variable predictionList and the user's sequence so far to calculate the % correct predictions overall. At each time step this metric is appended to a session state variable perc_correctPredictions and used to display in column 2 as well as to plot a line chart to show the prediction success over time.

```python
def displayStats():
    with col6:
        # Flatten prediction list session state variable:
        plist = [int(item) for item in chain(*st.session_state.predictionList)]
        # Create df with predicted and chosen values. 
        data = pd.DataFrame({"pred":plist[:-1],"vals":sequence[initSeqLen+1:]})
        # Calculate percent of cases where pred-vals == 0 (i.e. correct predictions from the computer)
        perc_correct = ((data.pred-data.vals == 0).sum())/len(data.pred)
        # Update session state variable per_correctPredictions
        st.session_state.perc_correctPredictions.append(perc_correct)
        st.write("% correct predictions after every trial ⬇️")
        st.line_chart(st.session_state.perc_correctPredictions)
        
    with col2:
        if len(st.session_state.perc_correctPredictions)>=2:
            st.divider()
            delta = st.session_state.perc_correctPredictions[-1]-st.session_state.perc_correctPredictions[-2]
            st.metric(label="% correct predictions", value= np.round(perc_correct,2), delta=np.round(delta, 2))
    return

```

---
## What else I want to implement in the future
This program is a work in progress, as I have multiple ideas how to improve it and what to add, including
- Creating a simulation of a user who made a very logn choice sequence of say 1000 choices and evaluating algorithm performance
- Test whether it makes sense to retrain the model with the whole sequence or perhaps it's enough to only use the past X choices
- I am convinced that being able to view the computer's prediction accuracy makes the person be better at beating the computer. I want to test the prediction accuracy when people do the task blindly without any feedback (for example at the very end).
- Make a version with more than two choice options, e.g. all digits from 0 to 9

So stay tuned for updates 😎✌🏻
