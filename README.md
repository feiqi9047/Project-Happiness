# Project Happiness

What determines happiness? Can we predict how happy a country is?

## Data

- Data is collected from Knoema's World Happiness Index for 2018, which sourced the original data from the World Happiness Report published by the Sustainable Development Solutions Network
- Surveying 156 countries, the report provides the rankings of polled happiness score amongst other life-evaluation questions
- Rankings are taken from nationally representative samples of ~1,000 respondents per country
- The happiness score is ranked on a scale of 1 to 10. Each variable measures a population-weighted average score on a scale from 0 to 10

The model aims to predict Happiness Score for a given country. The independent variables in the model consist of the following by country:
- Positive Emotions
- Negative Emotions
- GDP per Capita
- Health/Life Expectancy
- Freedom
- Social Support
- Government Confidence
- Generosity
- Perception of Corruption

### Data Cleaning

- Organized the schema by pivoting the dataframe to make each country as the index, while rearranging each independent variable into its own column
- Dropped all other columns except for the selected independent variables
- Dropped all NaN values, which left the dataframe with 123 countries that have workable data
- Snippet of resulting dataframe:
<img width="880" alt="Screen Shot 2019-04-18 at 11 34 46 AM" src="https://user-images.githubusercontent.com/44821660/56374819-0cf63800-61d2-11e9-9887-60d31061e36a.png">

## Initial Observations

<img width="527" alt="Screen Shot 2019-04-18 at 11 38 14 AM" src="https://user-images.githubusercontent.com/44821660/56374873-27301600-61d2-11e9-8e86-0d2babf31f03.png">

- Strong positive correlations were observed between Happiness Score (dependent variable) with GDP per Capita, Health/Life Expectancy, and Social Support. Negative correlations with the dependent variable were shown for Negative Emotions and Perception of Corruption

<img width="763" alt="Screen Shot 2019-04-18 at 11 38 22 AM" src="https://user-images.githubusercontent.com/44821660/56375148-c35a1d00-61d2-11e9-916e-38f4f63711c0.png">
There are some outliers detected for the variables Freedom, Social Support, Generosity, and Perception of Corruption. These outliers are kept in the dataset since eliminating them would mean excluding countries with geopolitical instabilities.

## Where is Happiness?

I created an interactive world heatmap on Plotly using Happiness Score (access via link below):

https://plot.ly/~feiqi9047/3/

Feel free to drag/pull and hover!

The top 10 happiest and unhappiest countries are shown below based on Happiness Score:
<img width="734" alt="Screen Shot 2019-04-18 at 1 34 22 PM" src="https://user-images.githubusercontent.com/44821660/56379854-ba6f4880-61de-11e9-964b-53e66a7adb5c.png">
<img width="738" alt="Screen Shot 2019-04-18 at 1 34 29 PM" src="https://user-images.githubusercontent.com/44821660/56379863-be02cf80-61de-11e9-9901-ee8fca414e09.png">

## Understanding the Target Variable

I wanted to see what the R<sup>2</sup> looks like with all of the original independent variables in the dataset:
- SKlearn gave me a baseline R<sup>2</sup> of 0.77
- OLS gave me a R<sup>2</sup> of 0.81
<img width="340" alt="Screen Shot 2019-04-18 at 2 23 45 PM" src="https://user-images.githubusercontent.com/44821660/56382708-feb21700-61e5-11e9-8248-abe0ca0ab343.png">

### Checking for Multicollinearity

Multicollinearity was detected for GDP per Capita. However, given its high correlation with the dependent variable, I decided to keep it in the model. 
<img width="806" alt="Screen Shot 2019-04-18 at 2 29 06 PM" src="https://user-images.githubusercontent.com/44821660/56382837-59e40980-61e6-11e9-9df5-ac6175e8b36e.png">

### Checking for Interactions

Given all possible combinations of independent variables, I found that Social Support is a major confounding variable. I decided to split up Social Support into 3 categories: High Social Support Environment, Medium Social Support Environment, and Low Social Support Environment.
<img width="729" alt="Screen Shot 2019-04-18 at 2 31 46 PM" src="https://user-images.githubusercontent.com/44821660/56383111-0b833a80-61e7-11e9-88a3-66245ab7fd49.png">

Out of the three independent variables that had the highest interactions with Social Support, Health/Life Expectancy and Perception of Corruption exhibit clear interactions while Negative Emotions does not. To interpret this:
- The effect on Happiness Score are more strongly affected by level of Health/Life Expectancy in High Social Support environments than in low social support environments
- The effect on Happiness Score are more strongly affected by Perception of Corruption in High Social Support environments than in low social support environments

Given the above, I decided to include the two interactions in my model. 

### Checking for Non-Linearity 

Examining the relationships between Happiness Score and each independent variable more closely, there appeared to be non-linear relationships between Happiness Score with Health/Life Expectancy and Perception of Corruption:

<img width="792" alt="Screen Shot 2019-04-18 at 1 40 56 PM" src="https://user-images.githubusercontent.com/44821660/56380228-a11acc00-61df-11e9-87df-f8093d32604a.png">

I proceeded to account for these non-linear relationships in the model using polynomial transformations.

For Health/Life Expectancy, I tested its transformation to the powers of 2, 3, and 4:

<img width="330" alt="Screen Shot 2019-04-18 at 2 39 14 PM" src="https://user-images.githubusercontent.com/44821660/56383572-1b4f4e80-61e8-11e9-919f-abbdab42080e.png">
<img width="319" alt="Screen Shot 2019-04-18 at 2 42 46 PM" src="https://user-images.githubusercontent.com/44821660/56383625-489bfc80-61e8-11e9-97a8-b4b4f80b80d3.png">

The best fitting line is the degree 2 transformation. Checking the residual distribution proved that such transformation is useable.

Repeating the same steps for Perception of Corruption:

<img width="316" alt="Screen Shot 2019-04-18 at 2 46 43 PM" src="https://user-images.githubusercontent.com/44821660/56383861-e55e9a00-61e8-11e9-9fc1-d8dd971d36cb.png">
<img width="313" alt="Screen Shot 2019-04-18 at 2 47 15 PM" src="https://user-images.githubusercontent.com/44821660/56383866-e98ab780-61e8-11e9-9637-548f86c8e189.png">

Although degree 3 seemed to be the best fitting line, the residuals for degree 2 showed more of a random distribution. I decided to include the degree 2 transformation in my model.

## What Does my Model Look Like?

Inclusive of all the interaction terms and variable transformations, my OLS output showed a R<sup>2</sup> 0.86:

<img width="360" alt="Screen Shot 2019-04-18 at 2 50 54 PM" src="https://user-images.githubusercontent.com/44821660/56384118-95cc9e00-61e9-11e9-9dce-099da1cbbf16.png">

There is a slight improvement from my baseline model. However, this model showed the presence of insignificant p-values and counterintuitive coefficients for my variables.

## Training my Model

After many iterations of including and dropping different variables, I needed to make a trade-off between generating the highest Adj. R<sup>2</sup> and generating reasonable coefficients. Hence, I dropped all variables with p-values > 0.05, variables where the coefficients were not explanatory, and those that are already included in the model as part of an interaction term or transformed variable (these included Generosity, Negative Emotions, Social Support, Perception of Corruption, Positive EMotions, Health/Life expectancy to the second degree, and Perception of Corruption to the second degree).

The final model yielded a R<sup>2</sup> of 0.84 and an Adj. R<sup>2</sup> of 0.83:

<img width="338" alt="Screen Shot 2019-04-18 at 3 01 09 PM" src="https://user-images.githubusercontent.com/44821660/56384612-d4af2380-61ea-11e9-970f-986321752af7.png">

## Interpreting the Model

In my multiple-regression model, ~83.7% of the variability in the Happiness Score can be explained by the following variables:
- For every 1 point increase in the score of GDP per Capita, Happiness Score goes up by 0.65
- For every 1 point increase in the score of Health and Life Expectancy, Happiness Score goes down by 4.36 (This inverse relationship could be due to increased financial pressure for retirement given longer expected life. Typically longer life expectancy is observed in developed countries with stronger economies, where, on average, employments are more competitive.)
- For every 1 point increase in the score of Freedom, Happiness Score goes up by 2.52
- For every 1 point increase in the score of Government Confidence, Happiness Score goes down by 1.20 (This seems counterintuitive and will need to be further researched)
- Health and Life Expectancy in high Social Support environments increases Happiness Score by 5.01 than that in low Social Support environments
- Perception of Corruption in high Social Support environments lowers Happiness Score by 3.64 than that in low Social Support environments (A possible explanation for this could be related to managing expectations. People in low social support countries tend to perceive their governments as more corrupt, and vice versa. It is possible that their low expectations and general distrust in the government prevented as dramatic of a decrease in their overall happiness as compared to those in high social support environments)

### Validating my Model

Since I do not have 2019 data, I cannot make predictions to approximate future happiness. However, I still wanted to test if my model is robust.

Using the same variables from 2017 and 2016, I used my model to see the Actual Happiness Score and the "Predicted Happiness Score" for each country in those years. Then, I plotted out the Margin of Error between the Actual Happiness Scores and what my model "predicted". (Note that negative MOE indicates that I "underpredicted" the Happiness Score, where as positive MOE indicates that I "overpredicted" the Happiness Score):

<img width="796" alt="Screen Shot 2019-04-18 at 3 23 16 PM" src="https://user-images.githubusercontent.com/44821660/56385712-f1992600-61ed-11e9-81a2-c493c52c2fa1.png">
<img width="812" alt="Screen Shot 2019-04-18 at 3 23 26 PM" src="https://user-images.githubusercontent.com/44821660/56385728-f52cad00-61ed-11e9-94ec-e279679fab96.png">

An interesting observation here is that countries with the least political stability, weakest economy, and lowest level of development tend to have the highest positive MOE. Whereas more developed countries tend to exhibit less variability in their Happiness Scores over 2016-2018.


