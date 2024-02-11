# Setup

Specify the output in the settings.json such that auxiliary files are not stored in the main.tex directory but instead in the build directory:
```json
"latex-workshop.latex.outDir": "./build",
```


I got the following sections. Write a conlcusion section for me:


\section{Method}

In this section, we detail the methodology employed to determine the optimized integration model
along with the implementation steps undertaken to validate our approach.

\subsection{Dataset Description} 
For our experimentation, we utilized the inD, exiD, and rounD datasets provided by the ika 
of RWTH Aachen University.
The datasets offer vehicle trajectories recorded at German intersections, highway exits, and entries, 
and roundabouts, respectively. 
Additionally, the datasets were provided with a tool that allowed us to visualize them for better understanding. 
The datasets includes 18 columns ranging from the ids of a vehicle, lifetime to all various motion related information.
We will primarily focus on the following columns:
$x_{\text{Center}}$, $y_{\text{Center}}$, $x_{\text{Velocity}}$, $y_{\text{Velocity}}$, $x_{\text{Acceleration}}$, 
$y_{\text{Acceleration}}$.

The columns  $x_{\text{Center}}$ and $y_{\text{Center}}$ represent the current position of the object's center in the coordinate coordinate system. Preprocessing can be done such that the columns describe the distance to the origin of the vehicle.

Velocity information is captured through columns $x_{\text{Velocity}}$ and $y_{\text{Velocity}}$ representing velocities 
in the x-axis and y-axis respectively.

Acceleration data are represented in columns $x_{\text{Acceleration}}$ and $y_{\text{Acceleration}}$ which provide 
information on the acceleration in the x-axis and y-axis respectively.

\subsection{Model Selection Process} 
Our model selection process involved a systematic trial and error approach with a total of 8 different models. 
Each model underwent an evaluation process to assess its ability to accurately predict vehicle acceleration across 
various scenarios. 
Ultimately, the following two linear models emerged as the most suitable choice based on their superior performance metrics.

\begin{align} 
    a_{dis}(k) &= \bar{c}_1 \bigl( s(k) - s(k+1) - v(k) \bigr) -\bar{c}_2 a(k-1) \\
    a_{vel}(k) &= \bar{c}_3 \bigl( v(k) - v(k+1) \bigr) -\bar{c}_4 a(k-1) 
\end{align}

This linear model is then solved by linear regression. 
After training the model on the acceleration set from our dataset, we can rearrange the formulas
to determine the distance and velocity formulas, similarly to the ballistic integration

\begin{align} 
    s(k+1) &= s(k) + v(k) + c_1 a_{dis}(k) + c_2 a(k-1) \\
    v(k+1) &= v(k)        + c_3 a_{vel}(k) + c_4 a(k-1)
\end{align}


The constants can then be determined through these calculations:
\begin{align}
   c_1 &= \frac{1}{\bar{c}_1} \\
   c_2 &= \bar{c}_2 \cdot c_1 \\
   c_3 &= \frac{1}{\bar{c}_3} \\
   c_4 &= \bar{c}_4 \cdot c_3
\end{align}

By rearranging the model as such, we specifically ensure that for both $s(k)$ and $v(k)$ we receive the same 
acceleration, as we are training both models on the same acceleration set.
Thus, we receive a proper integration method for the data set on which we trained the model, which solves the 
problems of the mismatched acceleration in previous attempts.

As seen in the section Dataset Description, we can see, that the column for distance, velocity and acceleration are 
split into their x- and y-components.
Thus our linear models will look like this:\\
Distance Model (Acceleration from distance formula),

{\footnotesize

\begin{align} \label{eq:lin_model_acc_dis}
    \begin{bmatrix}
        a_x(k) \\ 
        a_y(k)       
    \end{bmatrix}_{dis}
    =
    \begin{bmatrix}
       s_x(k) - s_x(k+1) - v_x(k) & -a_x(k-1) \\ 
       s_y(k) - s_y(k+1) - v_y(k) & -a_y(k-1)   
    \end{bmatrix}
    \begin{bmatrix}
        \overline c_1 \\
        \overline c_2 \\
   \end{bmatrix}
\end{align}
}\\
Velocity Model (Acceleration from velocity formula),

\begin{align} \label{eq:lin_model_acc_vel}
    \begin{bmatrix}
        a_x(k) \\ 
        a_y(k) 
    \end{bmatrix}_{vel}
    =
    \begin{bmatrix}
        v_x(k) - v_x(k+1) & -a_x(k+1)    \\ 
        v_y(k) - v_y(k+1) & -a_y(k+1)    \\
    \end{bmatrix}
    \begin{bmatrix}
        \overline c_3 \\
        \overline c_4 \\
   \end{bmatrix}
\end{align}

Afterward, we will compare the accelerations derived from displacement, denoted as 
$\begin{bmatrix} a_x(k) \\ a_y(k) \end{bmatrix}_{dis}$
, with those derived from velocity, denoted as 
$\begin{bmatrix} a_x(k) \\ a_y(k) \end{bmatrix}_{\text{vel}}$
, to assess the effectiveness of the Distance Model's acceleration estimation relative to that of the Velocity Model.
Using the coefficients determined from linear regression from the linear models (5) and (6) we can determine the distance and the velocity using the following matrices incorporating the x and y components.

Distance matrix
{
\footnotesize
\label{eq:distance_matrix}
\begin{align}
    \begin{bmatrix}
        s_x(k+1) \\ 
        s_y(k+1) 
    \end{bmatrix}
    =
    \begin{bmatrix}
        a_x(k) & a_x(k-1)    \\ 
        a_y(k) & a_y(k-1)    \\
    \end{bmatrix}
    \begin{bmatrix}
        c_1 \\
        c_2 \\
    \end{bmatrix}
    +
    \begin{bmatrix}
        s_x(k) + v_x(k) \\ 
        s_y(k) + v_y(k) \\
    \end{bmatrix}
\end{align} 
}

Velocity matrix
\begin{align}
\label{eq:velocity_matrix}
    \begin{bmatrix}
        v_x(k+1) \\ 
        v_y(k+1) 
    \end{bmatrix}
    =
    \begin{bmatrix}
        a_x(k) & a_x(k-1)    \\ 
        a_y(k) & a_y(k-1)    \\
    \end{bmatrix}
    \begin{bmatrix}
        c_3 \\
        c_4 \\
   \end{bmatrix}
    +
    \begin{bmatrix}
        v_x(k) \\
        v_y(k) 
    \end{bmatrix}
\end{align}

Thus, we receive a discrete integration model with matching accelerations for the distance and velocity formula.

\subsection{Training and Testing Procedure} 
For training the model, we used the $LinearRegression()$ function, provided by the sci-kit-learn Python library to train our linear model.
The $train\_test\_split()$ function was utilized, with a test size of 0.3, to ensure a sufficient amount of data 
for evaluation.

\subsection{Evaluation Metrics and Results} 
The performance of our linear model was assessed using common metrics for linear regression, including 
R-squared (r2) and Mean Squared Error (MSE). 
Detailed evaluation results, including comparisons with the old method, will be provided later in the report, 
offering insights into the model's predictive capabilities.

And this is the results section:
\section{Results}

\subsection{Linear Model}

The linear models that predict the acceleration derived from the distance formula
(\ref{eq:lin_model_acc_dis}) and the model derived from the velocity formula (\ref{eq:lin_model_acc_vel}) solved using linear regression have the following test results:


\begin{table}[htbp]
\centering
\caption{Comparison Results: Distance Model and Velocity Model}
\label{tab:model_comparison}
\begin{tabular}{l S[table-format=1.4e-2] S[table-format=1.4e-2] S[table-format=1.4e-2]}
\toprule
 & {MSE} & {MAE} & {$R^2$ Score} \\
\midrule
Distance Model & 3.5392e-03 & 1.1892e-02 & 9.8918e-01 \\
Velocity Model & 3.5370e-03 & 1.1877e-02 & 9.8918e-01 \\
\bottomrule
\end{tabular}
\end{table}

We can see that the predictions for the test set are quite accurate having small MSE and MAE. 
We can also plot our results into graphs to have a better visual understanding of our results

\begin{figure}[h]
    \centering
    \begin{minipage}[b]{0.45\columnwidth}
        \centering
        \includegraphics[width=\columnwidth]{images/figures/Prediction Acceleration (Linear Model, Distance).png}
        \caption{Test results Distance Model}
        \label{fig:minipage3}
    \end{minipage}
    \hfill
    \begin{minipage}[b]{0.45\columnwidth}
        \centering
        \includegraphics[width=\columnwidth]{images/figures/Prediction Acceleration (Linear Model, Velocity).png}
        \caption{Test results Velocity Model}
        \label{fig:minipage4}
    \end{minipage}
\end{figure}

The graph seems to fit our low MSE and MAE values, as all predicted points are lying close to the red line which indicates
a perfect prediction.
Interestingly we can see that most inaccuracies lie in the middle where the acceleration is close to zero.
Our hypothesis is that if our acceleration is close to zero the model has the least amount of information for inference, leading
to a more scattered prediction in this region.

\subsection{Equivalence of accelerations}

Note that the both results from the linear regression look quite similar. 
This is due to the fact, that we specifically trained both models to fit the same dataset. 
The similarity can be seen in the following figure.
The left shows the how the predicted accelerations of both our linear models match up, while on the right we can see how the 
accelerations match up for the previously used ballistic integration method

\begin{figure}[h]
    \centering
    \begin{minipage}[b]{0.45\columnwidth}
        \centering
        \includegraphics[width=\columnwidth]{images/figures/Acceleration Equivalence (Linear Model).png}
        \caption{Caption for Figure 1}
        \label{fig:minipage3}
    \end{minipage}
    \hfill
    \begin{minipage}[b]{0.45\columnwidth}
        \centering
        \includegraphics[width=\columnwidth]{images/figures/Acceleration Equivalence (Ballistic integration).png}
        \caption{Caption for Figure 2}
        \label{fig:minipage4}
    \end{minipage}
    \caption{Caption for the entire figure}
    \label{fig:combined_figure_2}
\end{figure}

In the following we can see the metrics of our results for the equivalence of the accelerations

\begin{table}[htbp]
\centering
\caption{Model Performance Comparison}
\label{tab:model_comparison_lin_v_bal}
\begin{tabular}{lccc}
\toprule
Model & MSE & MAE \\
\midrule
Linear Model & 1.8141e-06 & 7.5061e-04 \\
Ballistic Integration & 2.0698e+02 & 5.0158e-01 \\
\bottomrule
\end{tabular}
\end{table}

We can clearly see that the accelerations from both our linear models match up better than the previously used
ballistic integration method.

\subsection{Predicion of the movement}
Using the coefficients determine by the linear regression we can rearrange the model such that the model predicts 
the distance (\ref{eq:distance_matrix}) and the velocity (\ref{eq:velocity_matrix}).
The following shows metrics for the prediciton of the distance and the velocity using the coefficients

\begin{table}[t]
    \centering
    \caption{Regression Metrics}
    \begin{tabular}{lccc}
    \toprule
 & {MSE} & {MAE} & {$R^2$ Score} \\
    \midrule
    Distance Formula & $1.3867 \times 10^{1}$ & $3.7239 \times 10^{0}$ & $9.8926 \times 10^{-1}$ \\
    Velocity Formula & $2.2635 \times 10^{0}$ & $1.5045 \times 10^{0}$ & $9.3378 \times 10^{-1}$ \\
    \bottomrule
    \end{tabular}
    \label{tab:regression_metrics}
\end{table}


As we can see all metrics are conditioned well. 
Though we can see that the MSE and MAE values the error is quite some bit zero.
This means that our prediction for the distance and velocity is not perfect. 
For a better understanding, we also visualized our findings

\begin{figure}[htbp]
    \centering
    \begin{minipage}[b]{0.45\columnwidth}
        \centering
        \includegraphics[width=\columnwidth]{images/figures/Prediction Distance using the coefficients.png}
        \caption{Caption for Figure 1}
        \label{fig:minipage3}
    \end{minipage}
    \hfill
    \begin{minipage}[b]{0.45\columnwidth}
        \centering
        \includegraphics[width=\columnwidth]{images/figures/Prediction Velocity using the coefficients.png}
        \caption{Caption for Figure 2}
        \label{fig:minipage4}
    \end{minipage}
    \caption{Caption for the entire figure}
    \label{fig:combined_figure_2}
\end{figure}

Again, we can see that the prediction matches up with the ground truth very well. 
These results can then be visualized using the drone-dataset-tool provided with the dataset.
Results comparing the prediction of the moving cars to the ground truth can be found here.
\textbf{Insert url here}

