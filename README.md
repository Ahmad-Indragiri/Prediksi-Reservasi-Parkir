# Parking Reservation Fulfillment Prediction

This project focuses on developing a machine learning model to predict whether a parking reservation will be successfully fulfilled or not, based on various features associated with the reservation.

## Project Overview

Parking reservation systems often encounter scenarios where a booked slot isn't utilized (e.g., user cancellation or no-shows). This model attempts to identify patterns from historical reservation data to forecast the likelihood of a reservation being "Fulfilled" (original status `Selesai` - Completed or `Dikonfirmasi_Belum_Digunakan` - Confirmed_Not_Used) versus "Not Fulfilled" (original status `Dibatalkan_Pengguna` - Canceled_by_User or `Tidak_Hadir` - No_Show). Accurate predictions can aid in better parking space management and potentially optimize availability.

## Dataset

The dataset used is `reservasi_parkir.csv`, which is **synthetic data generated by AI** (sourced from `/kaggle/input/synthetic-parking-reservation-data-ai-generated/reservasi_parkir.csv` as per the notebook).

Key features in the dataset include:
- Reservation details (ID, parking slot, user ID)
- Target reservation times (date, entry time, exit time, duration)
- Vehicle details (type, parking-level)
- Booking creation time
- Final reservation status (the initial target variable)
- Total cost
- External conditions (weather forecast, special event information)
- Estimated occupancy at the time of booking

The original target variable `status_akhir_reservasi` was transformed into a binary target variable `target`:
- **1 (Fulfilled)**: If the original status was 'Selesai' or 'Dikonfirmasi_Belum_Digunakan'.
- **0 (Not Fulfilled)**: If the original status was 'Dibatalkan_Pengguna' or 'Tidak_Hadir'.

## Methodology

The model development process involved the following key stages:

1.  **Setup and Data Loading**: Importing necessary libraries (Pandas, NumPy, Scikit-learn, XGBoost, etc.) and loading the dataset.
2.  **Exploratory Data Analysis (EDA) & Target Variable Preparation**:
    * Inspecting dataset information, missing values, and descriptive statistics.
    * Analyzing the distribution of the original target variable (`status_akhir_reservasi`).
    * Creating the new binary target variable (`target`) for classification.
3.  **Feature Engineering & Preprocessing**:
    * Dropping irrelevant ID columns.
    * Converting date/time columns to datetime objects.
    * Engineering new features from datetime columns:
        * `pembuatan_reservasi_hour`, `pembuatan_reservasi_dayofweek`, `pembuatan_reservasi_month`
        * `booking_lead_days` (time difference between reservation creation and target entry).
        * `jam_masuk_hour`, `jam_keluar_hour`.
    * Identifying numerical and categorical features.
    * Creating preprocessing pipelines:
        * **Numerical Features**: Missing values imputed with the median, followed by scaling with `StandardScaler`.
        * **Categorical Features**: Missing values imputed with the mode, followed by encoding with `OneHotEncoder`.
    * Combining pipelines using `ColumnTransformer`.
    * Splitting data into training and testing sets (`train_test_split`) with stratification on the target variable.
4.  **Model Training & Selection**:
    * Training several classification models:
        * Logistic Regression
        * Random Forest Classifier
        * XGBoost Classifier
    * Each model was trained using the established preprocessing pipeline.
    * Evaluating models based on metrics: Accuracy, F1-Score, Precision, Recall, and ROC-AUC.
    * Generating Classification Reports and Confusion Matrices for each model.
    * Performing cross-validation (`cross_val_score`) using F1-score as the primary comparison metric.
5.  **Saving the Best Model**:
    * Based on evaluation results, the Random Forest Classifier was selected as the best-performing model.
    * The best model was retrained on the entire training dataset.
    * The trained model was saved to `model_reservasi_parkir.pkl` using `joblib`.

## Results

Among the evaluated models:
-   **Random Forest** and **XGBoost** demonstrated exceptional performance on the test data, achieving perfect scores (1.00) for Accuracy, F1-Score, Precision, Recall, and ROC-AUC.
-   **Logistic Regression** also performed very well, with an Accuracy of 0.9750 and an F1-Score of 0.9851.
-   Cross-validation results (mean F1-score) were:
    -   Logistic Regression: 0.9971
    -   Random Forest: 1.0000
    -   XGBoost: 1.0000

It's important to note that these perfect scores were achieved on **synthetic data**. Performance on real-world data might differ. The Random Forest model was chosen for saving.

## Files in this Repository

-   `prediksi-reservasi-parkir (1).ipynb`: The Jupyter Notebook containing all analysis, preprocessing, model training, and evaluation steps.
-   `model_reservasi_parkir.pkl`: The saved, trained Random Forest model file.
-   `README.md`: This file, providing an overview of the project.
-   (Optional: You might want to add a `requirements.txt` file)

## Libraries Used

-   Pandas
-   NumPy
-   Matplotlib
-   Seaborn
-   Scikit-learn
-   XGBoost
-   Joblib

## How to Run

1.  Ensure all required libraries are installed. You can install them using pip:
    ```bash
    pip install pandas numpy matplotlib seaborn scikit-learn xgboost joblib
    ```
2.  Open and run the `prediksi-reservasi-parkir.ipynb` notebook in a Jupyter environment (e.g., Jupyter Notebook or JupyterLab).
3.  The dataset `reservasi_parkir.csv` needs to be available at the appropriate path (the notebook assumes a Kaggle environment path). If running locally, adjust the dataset path accordingly.

## How to Use the Saved Model

The `model_reservasi_parkir.pkl` model can be loaded and used to make predictions on new data. Ensure the new data has the same format and features as the data used to train the model (after feature engineering and before preprocessing).

## Potential Future Enhancements

* Hyperparameter tuning for the selected model.
* Experimenting with other model architectures or more advanced ensemble techniques.
* Validating the model's performance on a real-world dataset.
* Deploying the model as an API service.

Example usage:
```python
import joblib
import pandas as pd
# import numpy as np # Only needed if your preprocessor's feature identification relies on np.number

# Load the model
loaded_model = joblib.load('model_reservasi_parkir.pkl')

# Example new data (ensure it has all 14 feature columns expected by the model)
# Columns: ['durasi_reservasi_menit', 'tipe_kendaraan', 'lantai_parkir_target',
# 'hari_reservasi_target', 'total_biaya_rupiah', 'prediksi_cuaca_pemesanan',
# 'info_acara_khusus_pemesanan', 'estimasi_okupansi_saat_pesan',
# 'pembuatan_reservasi_hour', 'pembuatan_reservasi_dayofweek',
# 'pembuatan_reservasi_month', 'booking_lead_days', 'jam_masuk_hour', 'jam_keluar_hour']

new_data = pd.DataFrame({
    'durasi_reservasi_menit': [120],
    'tipe_kendaraan': ['Mobil'],          # Car
    'lantai_parkir_target': ['P1'],       # Parking Level P1
    'hari_reservasi_target': ['Senin'],   # Monday
    'total_biaya_rupiah': [10000],
    'prediksi_cuaca_pemesanan': ['Cerah'], # Clear (weather)
    'info_acara_khusus_pemesanan': ['Tidak_Ada'], # No_Event
    'estimasi_okupansi_saat_pesan': [0.5],
    'pembuatan_reservasi_hour': [10],
    'pembuatan_reservasi_dayofweek': [0], # Monday=0
    'pembuatan_reservasi_month': [3],
    'booking_lead_days': [1.5],
    'jam_masuk_hour': [14],
    'jam_keluar_hour': [16]
})

# Make predictions
prediction = loaded_model.predict(new_data)
prediction_proba = loaded_model.predict_proba(new_data)

print(f"Prediction: {'Fulfilled' if prediction[0] == 1 else 'Not Fulfilled'}")
print(f"Probabilities (Not Fulfilled, Fulfilled): {prediction_proba[0]}")
