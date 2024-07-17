---
layout: posts
title:  "86.7% on Flood Prediction problem using XGBoost"
header:
  image: /assets/images/jonathan-ford-6ZgTEtvD16I-unsplash.jpg
  og_image: /assets/images/jonathan-ford-6ZgTEtvD16I-unsplash.jpg
date:   2024-07-17 09:00:00 +0100
categories: Tabular_Playground, XGBoost, Regression
author: Olayinka Ola
---

In this blog post, I'll discuss my approach to the flood prediction dataset, which is a regression problem. I'm using XGBoost to improve the R2 score. This project is part of Kaggle's Tabular Playground Series, which offers a great opportunity to sharpen machine learning and data science skills using synthetic datasets based on real-world use cases.

### Data Preparation

1. Imported train and test data
2. Removed the ID column as it's not necessary for prediction
    
    ```python
    # load the train dataset
    train = pd.read_csv("/kaggle/input/playground-series-s4e5/train.csv")
    
    # drop the id column
    train = train.drop("id", axis=1)
    
    # load the test dataset
    test_data = pd.read_csv("/kaggle/input/playground-series-s4e5/test.csv")
    
    # drop the id column
    test = test_data.drop("id", axis=1)
    initial_features = list(test.columns)
    
    # print the shape of the train and test dataset
    train.shape, test.shape
    ```
    
3. Used base features provided by the competition
4. Engineered additional features to improve performance (Credit to [AmbrosM](https://www.kaggle.com/competitions/playground-series-s4e5/discussion/499274) for the below add features function)
    
    ```python
    BASE_FEATURES = test.columns
    print(len(BASE_FEATURES))
    
    # Features to be used in the model
    def add_features(df):
        # Statistical features calculated from BASE_FEATURES
        sorted_features = [f"sort_{i}" for i in range(len(initial_features))]
        df['fsum'] = df[BASE_FEATURES].sum(axis=1)
        df['mean'] = df[BASE_FEATURES].mean(axis=1)
        df['std'] = df[BASE_FEATURES].std(axis=1)
        df['median'] = df[BASE_FEATURES].median(axis=1)
        df[sorted_features] = np.sort(df[initial_features], axis=1)
    #     df['special1'] = df['fsum'].isin(np.arange(72, 76))
        return df
    
    # Apply feature engineering to both training and testing datasets
    train = add_features(train)
    test = add_features(test)
    ```
    

### **Exploratory Data Analysis (EDA)**

---

In this [linked]([https://www.kaggle.com/code/madcontender/eda-flood-prediction](https://www.kaggle.com/code/madcontender/eda-flood-prediction)) notebook,  I conducted an EDA with the following findings:

- 20 independent features and 1 target feature (flood probability)
- No missing information in independent variables
- Features have roughly the same number of unique values (17-20)
- Similar distribution and randomness across features
- Positive skew in roughly the same range for all features
    
    ```
    Encroachments                     0.46
    TopographyDrainage                0.46
    InadequatePlanning                0.46
    PopulationScore                   0.45
    Watersheds                        0.45
    Siltation                         0.45
    MonsoonIntensity                  0.44
    DeterioratingInfrastructure       0.44
    IneffectiveDisasterPreparedness   0.44
    Urbanization                      0.44
    DrainageSystems                   0.44
    DamsQuality                       0.44
    CoastalVulnerability              0.44
    PoliticalFactors                  0.44
    WetlandLoss                       0.44
    Deforestation                     0.43
    ClimateChange                     0.43
    RiverManagement                   0.43
    Landslides                        0.43
    AgriculturalPractices             0.42
    ```
    
- Some variability in outliers, especially for moon intensity and landslides
    
    ```
    MonsoonIntensity                  0.34
    DrainageSystems                   0.29
    Siltation                         0.29
    Deforestation                     0.28
    Encroachments                     0.27
    DamsQuality                       0.26
    PopulationScore                   0.25
    InadequatePlanning                0.25
    Urbanization                      0.25
    DeterioratingInfrastructure       0.25
    ClimateChange                     0.24
    CoastalVulnerability              0.24
    TopographyDrainage                0.24
    Watersheds                        0.24
    WetlandLoss                       0.24
    RiverManagement                   0.22
    AgriculturalPractices             0.21
    IneffectiveDisasterPreparedness   0.21
    PoliticalFactors                  0.20
    Landslides                        0.19
    ```
    
- Target variable distribution resembles a normal distribution

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/flood_prediction_histo.png" alt="flood prediction">

- No multicollinearity among features
    
    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/flood_prediction_heatmap.png" alt="flood prediction_heatmap">`
    

**From the Heatmap**

- No two features are highly correlated with each other therefore no multicollinearity among the features.
- 8/20 feature score 0.19 on the correlation heatmap.
- Similar histogram and box plot profiles for the top 8 correlated features
    
    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/flood_prediction_mi.png" alt="histo and boxplot_Viz">
    
    ---
    
    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/flood_prediction_silation.png" alt="histo and boxplot_silation">
    
    ---
    
    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/flood_prediction_landslides.png" alt="histo and boxplot_landslides">
    
    ---
    
    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/flood_prediction_DI.png" alt="histo and boxplot_DI">
    
    ---
    
    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/flood_prediction_PS.png" alt="histo and boxplot_PS">
    
    ---
    
    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/flood_prediction_TD.png" alt="histo and boxplot_TD">
    
    ---
    
    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/flood_prediction_DQ.png" alt="histo and boxplot_DQ">
    
    ---
    
    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/flood_prediction_RM.png" alt="histo and boxplot_RM">
    

### Model Development

1. Started with a naive random forest model -  [Notebook Link]([https://www.kaggle.com/code/madcontender/eda-flood-prediction](https://www.kaggle.com/code/madcontender/navie-flood-prediction))
   - Resulted in a poor private score of 0.37329 (R2)
2. Improved model using XGBoost regression - [Notebook Link]([https://www.kaggle.com/code/madcontender/eda-flood-prediction](https://www.kaggle.com/code/madcontender/flood-prediction-xgb))
    
    ```python
    from xgboost import XGBRegressor
    
    params = {
        "lambda": 39.08510183586842,
        "alpha": 1.0788637557180403,
        "gamma": 0.0007289922088925636,
        "learning_rate": 0.274459641134258,
        "colsample_bynode": 1.0,
        "n_estimators": 250,
        "min_child_weight": 186,
        "max_depth": 83,
        "subsample": 1.0,
        "random_state": 2,
        "n_jobs": -1,
        "tree_method": "hist",
        "device": "cuda"
        
    }
    
    # Instantiate the XGBRegressor, xg_reg
    xg_reg = XGBRegressor(**params)
    
    # Fit to training set
    xg_reg.fit(xs, y)
    
    # Predict labels of test set, y_pred
    y_pred_xg = xg_reg.predict(valid_xs)
    
    mae = mean_absolute_error(valid_y, y_pred_xg)
    mse = mean_squared_error(valid_y, y_pred_xg)
    rmse = np.sqrt(mse)
    r2 = r2_score(valid_y, y_pred_xg)
    
    # Assemble the metrics we're going to write into a collection
    metrics = {"mae": mae, "mse": mse, "rmse": rmse, "r2": r2}
    
    print(metrics)
    ```
    
- Drastically improved the R2 score to 0.86651

## Conclusion

By applying XGBoost regression with optimized hyperparameters, I was able to significantly improve the model's performance from 0.37329 to 0.86651 demonstrates a strong predictive capability for flood probability based on the given features.

---