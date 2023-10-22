# SI_regressor[Bold Byte Coders]

### Решение задачи регрессии на основе показателя 'SI'

<!-- GETTING STARTED -->
## Getting Started

### Installation
  ```sh
  pip install requirements.txt
  ```
<!-- Implementation -->
## Implementation
  ```sh
  df_1 = pd.read_excel('1400.xlsx', sheet_name='Smile-IC50-CC50')
  df_1['SI'] = df_1['CC50-MDCK, mmg/ml']/df_1['IC50, mmg/ml']
  df_1['S_leng'] = df_1['SMILES'].str.len ()
  df_1 = df_1.drop(['Title', 'Pictures', 'Hydrogen bond acceptors', 'Hydrogen bond donors', 'Polar SA'], axis=1)
  # df_1.describe()
  ```
