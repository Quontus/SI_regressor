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
