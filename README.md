# SI_regressor[Bold Byte Coders]

### Решение задачи регрессии на основе показателя 'SI'

<!-- GETTING STARTED -->
## Getting Started

### Installation
  ```sh
  pip install requirements.txt
  ```
<!-- Implementation -->
## Implementation(нужно будет последовательно запускать все ячейки кода из файла, ниже приведены примеры функций ячеек)

### Загрузка и предобработка данных

  ```sh
  df_1 = pd.read_excel('1400.xlsx', sheet_name='Smile-IC50-CC50')
  df_1['SI'] = df_1['CC50-MDCK, mmg/ml']/df_1['IC50, mmg/ml']
  df_1['S_leng'] = df_1['SMILES'].str.len ()
  df_1 = df_1.drop(['Title', 'Pictures', 'Hydrogen bond acceptors', 'Hydrogen bond donors', 'Polar SA'], axis=1)
  # df_1.describe()
  df_1 = df_1[df_1['IC50, mmg/ml'] < 250]
  df_1 = df_1[df_1['CC50-MDCK, mmg/ml'] < 500]
  df_1 = df_1[df_1['SI'] <= 100]
  df_1 = df_1[(df_1['S_leng'] <= 200) & (0 <= df_1['S_leng'])]
  df_1 = df_1[df_1['Molecular weight'] <= 1250]
  df_1 = df_1.dropna()
  ```
  ### Оставляем только колонку SI и SMILES
  ```sh
  df_1 = df_1.drop(['CC50-MDCK, mmg/ml', 'IC50, mmg/ml', 'S_leng', 'Molecular weight'], axis=1)
  ```
  <!-- Получение набора данных по выбранному набору дескрипторов -->
  ### Получение набора данных по выбранному набору дескрипторов
  ```sh
  def morgan_fingerprint(df):
    mols = df['SMILES'].to_list()
    mols = list(map(Chem.MolFromSmiles, mols))
    descrs = [morgan_binary_features_generator(mol) for mol in mols]
    df_with_new_desc = pd.DataFrame(descrs)
    df_with_new_desc['target'] = list(df['SI'])
    return df_with_new_desc
  ```

  <!-- Разбиение new_df на тест и стади -->
  ### Разбиение new_df на test и train
  
  ```sh
  def data_set_cut(new_df, fraction):
      train_data, test_data = train_test_split(new_df, train_size=fraction, random_state=42)
      X_train = train_data.drop(columns=['target'])
      y_train = train_data['target']
      X_test = test_data.drop(columns=['target'])
      y_test = test_data['target']
      return X_train, y_train, X_test, y_test
  ```

  <!-- Метод главных компонент -->
  ### Метод главных компонент
  
  ```sh
  def reduce_dimensionality(df, variance_threshold):
      if variance_threshold == 1:
          return df
      # Разделите DataFrame на матрицу признаков X и столбец target y
      X = df.drop(columns=['target'])
      y = df['target']
  
      # Создайте объект PCA и обучите его
      pca = PCA()
      pca.fit(X)
  
      # Определите количество компонент, чтобы объяснить заданный уровень дисперсии
      explained_variance = pca.explained_variance_ratio_
      cumulative_variance = explained_variance.cumsum()
  
      n_components = np.argmax(cumulative_variance >= variance_threshold) + 1
  
      # # Создайте график зависимости дисперсии от количества компонент
      # plt.figure(figsize=(10, 6))
      # plt.plot(range(1, len(explained_variance) + 1), cumulative_variance, marker='o', linestyle='--', color='b')
      # plt.xlabel('Number of Components')
      # plt.ylabel('Cumulative Explained Variance')
      # plt.title('Cumulative Explained Variance vs. Number of Components')
      # plt.grid()
  
      # Создайте новый объект PCA с выбранным количеством компонент
      pca_selected = PCA(n_components=n_components)
  
      # Примените PCA к данным
      X_reduced = pca_selected.fit_transform(X)
  
      # Создайте DataFrame с сокращенными признаками и добавьте столбец target обратно
      df_reduced = pd.DataFrame(X_reduced, columns=[f'PC{i+1}' for i in range(n_components)])
      df_reduced['target'] = y
      df_reduced.dropna(inplace=True)
      return df_reduced
  ```
  <!-- Модели -->
  ### Модели
  ```sh
  def apply_model(model, X_train, X_test, y_train, y_test):
      y_pred = model(X_train, X_test, y_train, y_test)
      mape, rmse, mse, mae, predict_volume = results(y_test, y_pred)
      # graf(y_test, y_pred)
      return mape, rmse, mse, mae, predict_volume, model.__name__
  
  def apply_models(X_train, y_train, X_test, y_test, skip_svr=False):
      models = [
          XGBRegressor_model,
          linear_regression_model,
          lasso_regression_model,
          ridge_regression_model,
          random_forest_regression_model,
          gradient_boosting_regression_model,
          svr_regression_model
          Neural_model
      ]
      results_list = []
      for model in models:
          if skip_svr and model == svr_regression_model:
              continue
          # print(f'Работа с моделью {model}')
          results = apply_model(model, X_train, X_test, y_train, y_test)
          results_list.append(results)
      return results_list
  ```
  <!-- Запуск -->
  ### Запуск
  ```sh
  dataframes = {
    "Дескриптор 1": new_df_1,
    # "Дескриптор 2": new_df_2,
    "Дескриптор 3": new_df_3
  }
  
  best_metrics = {
      metric: [float('inf'), '', '', '', ''] if metric != 'predict_volume' else [float('-inf'), '', '', '', '']
      for metric in ["mape", "rmse", "mse", "mae", "predict_volume"]
  }
  
  pca_levels = [0.7, 0.75, 0.8, 0.85, 0.9, 0.95, 1]
  fraction_levels = [0.7, 0.75, 0.8, 0.85, 0.9, 0.95]
  
  for df_name, new_df in dataframes.items():
      for pca_level in pca_levels:
          pca_df = reduce_dimensionality(new_df, pca_level)
          for fraction in fraction_levels:
              X_train, y_train, X_test, y_test = data_set_cut(pca_df, fraction)
              skip_svr = df_name == "Дескриптор 2"
              results_list = apply_models(X_train, y_train, X_test, y_test, skip_svr)
              for mape, rmse, mse, mae, predict_volume, model_name in results_list:
                  for metric, value in zip(["mape", "rmse", "mse", "mae", "predict_volume"], [mape, rmse, mse, mae, predict_volume]):
                      if metric == 'predict_volume':
                          if value > best_metrics[metric][0]:
                              best_metrics[metric] = [value, model_name, df_name, pca_level, fraction]
                      else:
                          if value < best_metrics[metric][0]:
                              best_metrics[metric] = [value, model_name, df_name, pca_level, fraction]
  
  print("Лучшие результаты:")
  for metric in ["mape", "rmse", "mse", "mae", "predict_volume"]:
      print(f"{metric.upper()}: {best_metrics[metric][0]} в модели {best_metrics[metric][1]} с дескриптором {best_metrics[metric][2]} с уровнем дисперсии {best_metrics[metric][3]} c тренировочной выборкой {best_metrics[metric][4]}")
  ```
