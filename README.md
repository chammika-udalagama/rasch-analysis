# Rasch Analysis



## My Recipe

### Step 1: Consolidate and Format Data

We need to organise/format the scores so that:

1. We have columns `grader` and `name` (name is the name of the student)
2. Shift the number scale so that it stars from 0.
3. The question columns have names: `q1_response, q2_response,...`
4. Then save this file as `_temp.xlsx`
5. Order should be: `student, rater, q1_response, q2_response,...`

### Step 2: Run the Rasch Model using R’s `TAM` package

- Run the Rasch model using the R `TAM` package.

  - We use the **Multi-faceted Rasch Model** (MFRM). Under this umbrella we use a **Partial Credit Model** (PCM) which allows the multiple categories (0, 1, 2, 3,…).The original Rasch model (called the **dichotomous model**) only allowed true or false.

  - A facet is any latent quality whose influence we want to investigate. We are interested in the facet `rater`. So, we set the modelling equation as:

    ```R
    rasch.formula <- ~ item + rater + item:step
    ```

### Step 3: Merge the fitted Data

- Fitted data will include values of $\theta,\beta ,\tau$  along with their associated standard errors.

- In this step I merge all the data into a single file with all the data necessary  to calculate the various values in the next step.

- The merged file will typically have the following **columns**:

  ```python
  ['name', 'grader',
   'q01_response', 'q02_response', 'q03_response', 'q04_response',   # These are the original responses
   'q01_response_xsi', 'q02_response_xsi', 'q03_response_xsi', 'q04_response_xsi',  
   																																	 # These are the β's
   'student-theta', 'student-theta.se',                              # These are the θ's
   'grader-xsi', 'grader-xsi.se',                                    # These are the α's
   'q01_response:step1_xsi', 'q01_response:step1_xsi.se',            # These are the 𝜏's
   'q02_response:step1_xsi', 'q02_response:step1_xsi.se',
   'q03_response:step1_xsi', 'q03_response:step1_xsi.se',
   'q04_response:step1_xsi', 'q04_response:step1_xsi.se',
   'q01_response:step2_xsi', 'q01_response:step2_xsi.se',
   'q02_response:step2_xsi', 'q02_response:step2_xsi.se',
   'q03_response:step2_xsi', 'q03_response:step2_xsi.se',
   ...
   'q04_response:step5_xsi', 'q04_response:step5_xsi.se',
   'q01_weight', 'q02_weight', 'q03_weight', 'q04_weight'               # These are weights as floats
  ]
  ```

  **Note**:  Columns `q01_...` all have the same values for all rows. I organised these this way so that I can easily perform row-wise calculations using Pandas.



## Theory

### Basics

- Person $n$ has a latent ability $\theta_n$ which we are trying to estimate using the responses to a set of questions (**items**). 
- An item $i$ can have one of the ordered responses $0, 1, 2,\ldots$ upto a maximum of $m_i$ giving it $m_i+1$ **categories**.
- These categories are not equally easy to achieve. The **difficulty** of achieving an response $k$ (which is one of $0, 1,2,3,\ldots m_i$) for the item $i$ is given by $\beta_{ik}$.

#### Dichotomous Model

- The dichotomous model has only two categories for each item ($m_i=1$).

- The probability of a person $n$ achieving a score $x \in (0,1)$  for item $i$ is given by:

$$
p_{ni(k=1)}=\dfrac{\exp(\theta_n-\beta_i)}{1+\exp(\theta_n-\beta_i)}
$$

​	where $\beta_i$ is the difficulty of item $i$.

- It is typical to express this information in terms of a **logit**, which is the log odds of success over failure.
  $$
  \ln\left[\dfrac{p_{ni(k=1)}}{p_{ni(k=0)}}\right] = \theta_n -\beta_i
  $$
  
- Note that if $\theta_n=\beta_i$, $p=50$% and logit zero. 
  The bigger $\theta_n$ is over $\beta_i$ the greater the chance of the person getting item $i$ correct. I.e. a positive logit is associated with greater success.

#### Polytomous Model

- A polytomous model allows for more than two categories. 
Lets indicate these categories as  $0, 1, 2,\ldots$ upto a maximum of $m_i$ giving a total of $m_i+1$ categories. 
- Having multiple categories modifies the previous equation to:

$$
\ln\left[\dfrac{p_{ni(k)}}{p_{ni(k-1)}}\right] = \theta_n -\beta_i - \tau_k
$$

​	where $\tau_k$ is a measure of how difficult it is to move from category $k-1$ to $k$. 
​	More specifically it is the point where there is a $50:50$ probability to go to $k$ or $k-1$.

- When the category thresholds are not equally spaced across items, then the equation becames:
$$
\ln\left[\dfrac{p_{ni(k)}}{p_{ni(k-1)}}\right] = \theta_n -\beta_i - \tau_{ik}
$$

- The first version is called the **Rating Scale Model** (RSM) and typically deals with situations where there is a rating involved (e.g. strongly disagree, disagree, agree, and strongly agree). The second version is called **Partial Credit Model** (PCM) and allows for categories that are not equally space. The fundamental difference between the two is that for RSM all the items have identical category thresholds ($\tau_i$) whereas with PCM every item has its own set of thresholds ($\tau_{ik}$).


- The probability of a person $n$ achieving a category $x\, (\le m_i)$ for item $i$ is given by:
  $$
  \newcommand\ff[1]{\exp{\displaystyle\left[#1(\theta_n-\beta_{ik})-\;\sum_{s=0}^{s=#1}\tau_{is}\right]}}
  p_{ni(k=x)} = \dfrac{\ff{x}}{\displaystyle\sum_{r=0}^{m_i}{\ff{r}}}
  $$


#### Multi-Faceted Rasch Model

- Following the above scheme we can add more facets to account for latent factors that influence the probability of achieving a category. For instance, the severity $\alpha_j$ of a rater $j$ can be incorporated by:

$$
\ln\left[\dfrac{p_{nij(k)}}{p_{nij(k-1)}}\right] = \theta_n -\beta_i -\alpha_j - \tau_{ik}
$$

​	Notice, that when a rater is sever (i.e. large $\alpha$),  the person’s ability $\theta$, must large for there to positive logit score and 	consequently a large probability of achieving a high $k$.

- The corresponding probability for person with ability $\theta_n$, achieving category $k$ for item $i$ with a grader $j$ is given by:

  

$$
\newcommand\ff[1]{\exp{\displaystyle\left[#1(\theta_n-\alpha_{j}-\beta_{ik})-\;\sum_{s=0}^{s=#1}\tau_{is}\right]}}
p_{nij(k=x)} = \dfrac{\ff{x}}{\displaystyle\sum_{r=0}^{m_i}{\ff{r}}}
$$

#### Fitting a Model

- When we use Rasch analysis, we setup a model that include the facets we are interested in and then use a fitting algorithm to find the best value of $\theta, \beta, \alpha, \tau$ that fit the data. The R package `TAM` provides this functionality.



## Extracting Information from the Model

- Once we have the probabilities we can calculate the score we expect as:
  $$
  \text{score for person $n$ for item $i$ by grader $j$} = \sum_{k=0}^{k=m_i}k*p_{nijk}
  $$
  

### Correcting for Rater Severity

