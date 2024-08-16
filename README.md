---
jupyter:
  kernelspec:
    display_name: Python 3 (ipykernel)
    language: python
    name: python3
  language_info:
    codemirror_mode:
      name: ipython
      version: 3
    file_extension: .py
    mimetype: text/x-python
    name: python
    nbconvert_exporter: python
    pygments_lexer: ipython3
    version: 3.10.12
  nbformat: 4
  nbformat_minor: 5
---

::: {#f842ba93-f137-4b90-a236-bc325122d610 .cell .markdown}
# Propriedades das imagens geoespaciais
:::

::: {#0562570c-5e6f-4f77-95dc-33366972a7a1 .cell .markdown}
### Características das bandas
:::

::: {#d7b3c18d-093a-4abf-9a6c-4a4428e565d6 .cell .markdown}
A página de desenvolvedor do **Google** fornece informações detalhadas
sobre o catálogo *Harmonized Sentinel-2 MSI: MultiSpectral Instrument,
Level-2A*, assim como os demais. A tabela de referência contendo
informações sobre as bandas das imagens pode ser obtida no endereço
<https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_SR_HARMONIZED#bands>.
:::

::: {#6098d509-93e0-493b-b00d-3a4c1bc81fbb .cell .code execution_count="86"}
``` python
import requests
from bs4 import BeautifulSoup
import pandas as pd

# Define a URL da página web
url = "https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_SR_HARMONIZED#bands"

# Faz uma requisição HTTP para a URL
response = requests.get(url)

# Verifica se a requisição foi bem-sucedida
if response.status_code == 200:
    # Cria um objeto BeautifulSoup para analisar o código fonte HTML
    soup = BeautifulSoup(response.text, 'html.parser')

    # Encontra a tabela de bandas usando a classe 'eecat'
    bands_table = soup.find('table', class_='eecat')

    # Se a tabela for encontrada, extraia as informações
    if bands_table:
        # Cria listas para armazenar as informações das bandas
        band_names = []
        band_units = []
        band_min = []
        band_max = []
        band_scale = []
        band_pixel_size = []
        band_wavelength = []
        band_description = []

        # Itera sobre as linhas da tabela
        for row in bands_table.find_all('tr'):
            # Itera sobre as células da linha
            cells = row.find_all('td')
            # Verifica se a linha tem pelo menos 8 células
            if len(cells) >= 8:
                # Extrai as informações das células
                band_names.append(cells[0].text.strip())
                band_units.append(cells[1].text.strip())
                band_min.append(cells[2].text.strip())
                band_max.append(cells[3].text.strip())
                band_scale.append(cells[4].text.strip())
                band_pixel_size.append(cells[5].text.strip())
                band_wavelength.append(cells[6].text.strip())
                band_description.append(cells[7].text.strip())

        # Cria um DataFrame Pandas com as informações das bandas
        bands_df = pd.DataFrame({
            'Name': band_names,
            'Units': band_units,
            'Min': band_min,
            'Max': band_max,
            'Scale': band_scale,
            'Pixel Size': band_pixel_size,
            'Wavelength': band_wavelength,
            'Description': band_description
        })

        # Filtrando as bandas que começam com 'B'
        df = bands_df[bands_df['Name'].str.startswith('B')]

        # Selecionando as colunas desejadas
        df = df[['Name', 'Scale', 'Pixel Size', 'Wavelength', 'Description']]
        # Exibe o DataFrame Pandas
        print(df)
    else:
        print("A tabela com a classe 'eecat' não foi encontrada na página web.")
else:
    print("Erro ao acessar a página web:", response.status_code)
```

::: {.output .display_data}
```{=html}

            <style>
                .geemap-dark {
                    --jp-widgets-color: white;
                    --jp-widgets-label-color: white;
                    --jp-ui-font-color1: white;
                    --jp-layout-color2: #454545;
                    background-color: #383838;
                }

                .geemap-dark .jupyter-button {
                    --jp-layout-color3: #383838;
                }

                .geemap-colab {
                    background-color: var(--colab-primary-surface-color, white);
                }

                .geemap-colab .jupyter-button {
                    --jp-layout-color3: var(--colab-primary-surface-color, white);
                }
            </style>
            
```
:::

::: {.output .stream .stdout}
       Name   Scale Pixel Size                       Wavelength  Description
    0    B1  0.0001  60 meters    443.9nm (S2A) / 442.3nm (S2B)     Aerosols
    1    B2  0.0001  10 meters    496.6nm (S2A) / 492.1nm (S2B)         Blue
    2    B3  0.0001  10 meters        560nm (S2A) / 559nm (S2B)        Green
    3    B4  0.0001  10 meters      664.5nm (S2A) / 665nm (S2B)          Red
    4    B5  0.0001  20 meters    703.9nm (S2A) / 703.8nm (S2B)   Red Edge 1
    5    B6  0.0001  20 meters    740.2nm (S2A) / 739.1nm (S2B)   Red Edge 2
    6    B7  0.0001  20 meters    782.5nm (S2A) / 779.7nm (S2B)   Red Edge 3
    7    B8  0.0001  10 meters      835.1nm (S2A) / 833nm (S2B)          NIR
    8   B8A  0.0001  20 meters      864.8nm (S2A) / 864nm (S2B)   Red Edge 4
    9    B9  0.0001  60 meters      945nm (S2A) / 943.2nm (S2B)  Water vapor
    10  B11  0.0001  20 meters  1613.7nm (S2A) / 1610.4nm (S2B)       SWIR 1
    11  B12  0.0001  20 meters  2202.4nm (S2A) / 2185.7nm (S2B)       SWIR 2
:::
:::

::: {#2a72b10d-1e7c-4e39-ba65-5d7a976f72b6 .cell .markdown}
### Propriedades gerais
:::

::: {#5ebcafb6-0b5f-475b-b689-375adb09c504 .cell .markdown}
Embora nem todas as características das bandas descritas na tabela de
referência possam ser acessadas diretamente, diversas propriedades das
imagens podem ser exibidas através de métodos fornecidos pelo **Google
Earth Engine**, para extração dos metadados.
:::

::: {#a7f7bf45-0bbb-45bb-88b7-ce4bec5a4bc7 .cell .markdown}
Para acesso ao catálogo de imagens geoespaciais do **Google Earth
Engine**, repetimos os procedimentos descritos anteriormente
:::

::: {#d6c889d0-b35e-4eac-915e-638b1e20c625 .cell .markdown}
### Importamos as bibliotecas
:::

::: {#88137d5a-7f55-49d6-b61a-5c9dd6067db8 .cell .code execution_count="87"}
``` python
import ee
import geemap
import rasterio
from rasterio.plot import show
import matplotlib.pyplot as plt
from matplotlib.patches import Polygon
```

::: {.output .display_data}
```{=html}

            <style>
                .geemap-dark {
                    --jp-widgets-color: white;
                    --jp-widgets-label-color: white;
                    --jp-ui-font-color1: white;
                    --jp-layout-color2: #454545;
                    background-color: #383838;
                }

                .geemap-dark .jupyter-button {
                    --jp-layout-color3: #383838;
                }

                .geemap-colab {
                    background-color: var(--colab-primary-surface-color, white);
                }

                .geemap-colab .jupyter-button {
                    --jp-layout-color3: var(--colab-primary-surface-color, white);
                }
            </style>
            
```
:::
:::

::: {#fce744bb-597e-4aa0-9258-58abab4f9d8f .cell .markdown}
### Inicializamos a API
:::

::: {#90303401-8359-4c8e-a566-769185415112 .cell .code execution_count="88"}
``` python
# Processo de autenticação
ee.Authenticate()

# Inicializamos a biblioteca
ee.Initialize()
```

::: {.output .display_data}
```{=html}

            <style>
                .geemap-dark {
                    --jp-widgets-color: white;
                    --jp-widgets-label-color: white;
                    --jp-ui-font-color1: white;
                    --jp-layout-color2: #454545;
                    background-color: #383838;
                }

                .geemap-dark .jupyter-button {
                    --jp-layout-color3: #383838;
                }

                .geemap-colab {
                    background-color: var(--colab-primary-surface-color, white);
                }

                .geemap-colab .jupyter-button {
                    --jp-layout-color3: var(--colab-primary-surface-color, white);
                }
            </style>
            
```
:::
:::

::: {#1ba6ac6a-0002-48a4-9999-7a5ce70a46ba .cell .markdown}
### Definição de parâmetros
:::

::: {#8e3b8e9f-94bd-4d60-a242-0f8bd15a39fe .cell .markdown}
*Intervalo temporal*
:::

::: {#f7a3afdc-c959-462a-bc55-da3f30959276 .cell .code execution_count="89"}
``` python
data_inicial = '2024-04-01'
data_final = '2024-07-31'
```

::: {.output .display_data}
```{=html}

            <style>
                .geemap-dark {
                    --jp-widgets-color: white;
                    --jp-widgets-label-color: white;
                    --jp-ui-font-color1: white;
                    --jp-layout-color2: #454545;
                    background-color: #383838;
                }

                .geemap-dark .jupyter-button {
                    --jp-layout-color3: #383838;
                }

                .geemap-colab {
                    background-color: var(--colab-primary-surface-color, white);
                }

                .geemap-colab .jupyter-button {
                    --jp-layout-color3: var(--colab-primary-surface-color, white);
                }
            </style>
            
```
:::
:::

::: {#1d11e81c-daea-407b-9259-d62ae2359f78 .cell .markdown}
*Área de interesse*
:::

::: {#3b09a50a-78c9-48e7-87a3-0177e014b086 .cell .code execution_count="90"}
``` python
polygon_coords = [
    [-43.41287287680422, -21.756108688468274],
    [-43.396307553928246, -21.756108688468274],
    [-43.396307553928246, -21.750887131860495],
    [-43.41287287680422, -21.750887131860495],
    [-43.41287287680422, -21.756108688468274]
]

aoi = ee.Geometry.Polygon(polygon_coords, None, False)
```

::: {.output .display_data}
```{=html}

            <style>
                .geemap-dark {
                    --jp-widgets-color: white;
                    --jp-widgets-label-color: white;
                    --jp-ui-font-color1: white;
                    --jp-layout-color2: #454545;
                    background-color: #383838;
                }

                .geemap-dark .jupyter-button {
                    --jp-layout-color3: #383838;
                }

                .geemap-colab {
                    background-color: var(--colab-primary-surface-color, white);
                }

                .geemap-colab .jupyter-button {
                    --jp-layout-color3: var(--colab-primary-surface-color, white);
                }
            </style>
            
```
:::
:::

::: {#2e2ef316-6ac3-4fbc-83f8-b844b6433dcf .cell .markdown}
### Escolha da coleção de imagens
:::

::: {#9e0fe117-a1ce-4f6e-9f58-33690e6c8e9d .cell .markdown}
Aplicamos os filtros de acordo com os parâmetros:
:::

::: {#619325af-2cdb-4e69-98db-baa391028a7d .cell .code execution_count="91"}
``` python
db = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED') \
                .filterBounds(aoi) \
                .filterDate(ee.Date(data_inicial), ee.Date(data_final)) 
```

::: {.output .display_data}
```{=html}

            <style>
                .geemap-dark {
                    --jp-widgets-color: white;
                    --jp-widgets-label-color: white;
                    --jp-ui-font-color1: white;
                    --jp-layout-color2: #454545;
                    background-color: #383838;
                }

                .geemap-dark .jupyter-button {
                    --jp-layout-color3: #383838;
                }

                .geemap-colab {
                    background-color: var(--colab-primary-surface-color, white);
                }

                .geemap-colab .jupyter-button {
                    --jp-layout-color3: var(--colab-primary-surface-color, white);
                }
            </style>
            
```
:::
:::

::: {#e6d0f02c-ed25-4f4a-8341-f26b3b85caed .cell .markdown}
### Contagem das imagens disponíveis
:::

::: {#aadf0d69-b743-467c-b202-df0a7f8f8565 .cell .code execution_count="92"}
``` python
dbCount = db.size()
print('Quantidade: ', str(dbCount.getInfo())+'\n')
```

::: {.output .display_data}
```{=html}

            <style>
                .geemap-dark {
                    --jp-widgets-color: white;
                    --jp-widgets-label-color: white;
                    --jp-ui-font-color1: white;
                    --jp-layout-color2: #454545;
                    background-color: #383838;
                }

                .geemap-dark .jupyter-button {
                    --jp-layout-color3: #383838;
                }

                .geemap-colab {
                    background-color: var(--colab-primary-surface-color, white);
                }

                .geemap-colab .jupyter-button {
                    --jp-layout-color3: var(--colab-primary-surface-color, white);
                }
            </style>
            
```
:::

::: {.output .stream .stdout}
    Quantidade:  48
:::
:::

::: {#2f57fbb5-0f57-490b-a30e-da2f2c219091 .cell .markdown}
### Extraímos a imagem com menor cobertura de nuvens
:::

::: {#615f8a71-7783-4e56-8e10-52008acd7fcd .cell .code execution_count="93"}
``` python
db = ee.Image(db.sort('CLOUDY_PIXEL_PERCENTAGE').first())
```

::: {.output .display_data}
```{=html}

            <style>
                .geemap-dark {
                    --jp-widgets-color: white;
                    --jp-widgets-label-color: white;
                    --jp-ui-font-color1: white;
                    --jp-layout-color2: #454545;
                    background-color: #383838;
                }

                .geemap-dark .jupyter-button {
                    --jp-layout-color3: #383838;
                }

                .geemap-colab {
                    background-color: var(--colab-primary-surface-color, white);
                }

                .geemap-colab .jupyter-button {
                    --jp-layout-color3: var(--colab-primary-surface-color, white);
                }
            </style>
            
```
:::
:::

::: {#db2aff93-6bc9-47a4-a354-6cc0f098f4c0 .cell .markdown}
### Contagem das bandas disponíveis
:::

::: {#7961bd3f-1361-4c91-9dad-c2b2e4eab25c .cell .code execution_count="95"}
``` python
# Obtém os nomes das bandas
band_names = db.bandNames()

# Conta o número de bandas
number_of_bands = band_names.size().getInfo()

# Imprime o número de bandas
print(f'Número de bandas: {number_of_bands}')
```

::: {.output .display_data}
```{=html}

            <style>
                .geemap-dark {
                    --jp-widgets-color: white;
                    --jp-widgets-label-color: white;
                    --jp-ui-font-color1: white;
                    --jp-layout-color2: #454545;
                    background-color: #383838;
                }

                .geemap-dark .jupyter-button {
                    --jp-layout-color3: #383838;
                }

                .geemap-colab {
                    background-color: var(--colab-primary-surface-color, white);
                }

                .geemap-colab .jupyter-button {
                    --jp-layout-color3: var(--colab-primary-surface-color, white);
                }
            </style>
            
```
:::

::: {.output .stream .stdout}
    Número de bandas: 26
:::
:::

::: {#1afdba6c-b13a-45b2-a9b0-3a4a6691614e .cell .markdown}
### Bandas que iniciam com a letra \'B\'
:::

::: {#cd8deac2-5294-4b59-b738-552d97bbca73 .cell .code execution_count="96"}
``` python
# Leitura das bandas, que começam com "B"
db_bands = db.select('[B].*')

# Criamos um dicionário das bandas
db_band_info = db_bands.getInfo()
type(db_band_info)

# Iteramos sobre o dicionário 
print('Bandas:\n')
for i in range(len(db_band_info['bands'])):
    print(db_band_info['bands'][i]['id'])
```

::: {.output .display_data}
```{=html}

            <style>
                .geemap-dark {
                    --jp-widgets-color: white;
                    --jp-widgets-label-color: white;
                    --jp-ui-font-color1: white;
                    --jp-layout-color2: #454545;
                    background-color: #383838;
                }

                .geemap-dark .jupyter-button {
                    --jp-layout-color3: #383838;
                }

                .geemap-colab {
                    background-color: var(--colab-primary-surface-color, white);
                }

                .geemap-colab .jupyter-button {
                    --jp-layout-color3: var(--colab-primary-surface-color, white);
                }
            </style>
            
```
:::

::: {.output .stream .stdout}
    Bandas:

    B1
    B2
    B3
    B4
    B5
    B6
    B7
    B8
    B8A
    B9
    B11
    B12
:::
:::

::: {#35ab841b-0c6e-4ea6-a954-a5ecaa309298 .cell .markdown}
### Extraímos as propriedades das bandas selecionadas
:::

::: {#e2958888-1a83-469a-bfa4-4cfe0d8625d3 .cell .code execution_count="120"}
``` python
db_bands.getInfo()['bands']
```

::: {.output .display_data}
```{=html}

            <style>
                .geemap-dark {
                    --jp-widgets-color: white;
                    --jp-widgets-label-color: white;
                    --jp-ui-font-color1: white;
                    --jp-layout-color2: #454545;
                    background-color: #383838;
                }

                .geemap-dark .jupyter-button {
                    --jp-layout-color3: #383838;
                }

                .geemap-colab {
                    background-color: var(--colab-primary-surface-color, white);
                }

                .geemap-colab .jupyter-button {
                    --jp-layout-color3: var(--colab-primary-surface-color, white);
                }
            </style>
            
```
:::

::: {.output .execute_result execution_count="120"}
    [{'id': 'B1',
      'data_type': {'type': 'PixelType',
       'precision': 'int',
       'min': 0,
       'max': 65535},
      'dimensions': [1830, 1830],
      'crs': 'EPSG:32723',
      'crs_transform': [60, 0, 600000, 0, -60, 7700020]},
     {'id': 'B2',
      'data_type': {'type': 'PixelType',
       'precision': 'int',
       'min': 0,
       'max': 65535},
      'dimensions': [10980, 10980],
      'crs': 'EPSG:32723',
      'crs_transform': [10, 0, 600000, 0, -10, 7700020]},
     {'id': 'B3',
      'data_type': {'type': 'PixelType',
       'precision': 'int',
       'min': 0,
       'max': 65535},
      'dimensions': [10980, 10980],
      'crs': 'EPSG:32723',
      'crs_transform': [10, 0, 600000, 0, -10, 7700020]},
     {'id': 'B4',
      'data_type': {'type': 'PixelType',
       'precision': 'int',
       'min': 0,
       'max': 65535},
      'dimensions': [10980, 10980],
      'crs': 'EPSG:32723',
      'crs_transform': [10, 0, 600000, 0, -10, 7700020]},
     {'id': 'B5',
      'data_type': {'type': 'PixelType',
       'precision': 'int',
       'min': 0,
       'max': 65535},
      'dimensions': [5490, 5490],
      'crs': 'EPSG:32723',
      'crs_transform': [20, 0, 600000, 0, -20, 7700020]},
     {'id': 'B6',
      'data_type': {'type': 'PixelType',
       'precision': 'int',
       'min': 0,
       'max': 65535},
      'dimensions': [5490, 5490],
      'crs': 'EPSG:32723',
      'crs_transform': [20, 0, 600000, 0, -20, 7700020]},
     {'id': 'B7',
      'data_type': {'type': 'PixelType',
       'precision': 'int',
       'min': 0,
       'max': 65535},
      'dimensions': [5490, 5490],
      'crs': 'EPSG:32723',
      'crs_transform': [20, 0, 600000, 0, -20, 7700020]},
     {'id': 'B8',
      'data_type': {'type': 'PixelType',
       'precision': 'int',
       'min': 0,
       'max': 65535},
      'dimensions': [10980, 10980],
      'crs': 'EPSG:32723',
      'crs_transform': [10, 0, 600000, 0, -10, 7700020]},
     {'id': 'B8A',
      'data_type': {'type': 'PixelType',
       'precision': 'int',
       'min': 0,
       'max': 65535},
      'dimensions': [5490, 5490],
      'crs': 'EPSG:32723',
      'crs_transform': [20, 0, 600000, 0, -20, 7700020]},
     {'id': 'B9',
      'data_type': {'type': 'PixelType',
       'precision': 'int',
       'min': 0,
       'max': 65535},
      'dimensions': [1830, 1830],
      'crs': 'EPSG:32723',
      'crs_transform': [60, 0, 600000, 0, -60, 7700020]},
     {'id': 'B11',
      'data_type': {'type': 'PixelType',
       'precision': 'int',
       'min': 0,
       'max': 65535},
      'dimensions': [5490, 5490],
      'crs': 'EPSG:32723',
      'crs_transform': [20, 0, 600000, 0, -20, 7700020]},
     {'id': 'B12',
      'data_type': {'type': 'PixelType',
       'precision': 'int',
       'min': 0,
       'max': 65535},
      'dimensions': [5490, 5490],
      'crs': 'EPSG:32723',
      'crs_transform': [20, 0, 600000, 0, -20, 7700020]}]
:::
:::

::: {#bf2c5816-0a3e-4606-bd11-b75fc20701d4 .cell .markdown}
## Metadados
:::

::: {#1620031f-e192-4789-98e4-87bb6ad70bbd .cell .markdown}
Para obter os metadados no **Google Earth Engine**, chamamos o método
*getInfo* no objeto *Image* criado.
:::

::: {#4f76010a-ebb3-471f-87a9-5fe89c08152e .cell .code execution_count="97"}
``` python
db_info = db.getInfo()
```

::: {.output .display_data}
```{=html}

            <style>
                .geemap-dark {
                    --jp-widgets-color: white;
                    --jp-widgets-label-color: white;
                    --jp-ui-font-color1: white;
                    --jp-layout-color2: #454545;
                    background-color: #383838;
                }

                .geemap-dark .jupyter-button {
                    --jp-layout-color3: #383838;
                }

                .geemap-colab {
                    background-color: var(--colab-primary-surface-color, white);
                }

                .geemap-colab .jupyter-button {
                    --jp-layout-color3: var(--colab-primary-surface-color, white);
                }
            </style>
            
```
:::
:::

::: {#d96f3cd3-6127-4f5d-9c07-b06d939f8280 .cell .markdown}
## Listamos o dicionário de metadados associados à imagem
:::

::: {#654f008c-d569-4992-bfe5-e319f52a3342 .cell .code execution_count="98"}
``` python
print("Fields of Image Info:")
for key in db_info:
    print(key)
```

::: {.output .display_data}
```{=html}

            <style>
                .geemap-dark {
                    --jp-widgets-color: white;
                    --jp-widgets-label-color: white;
                    --jp-ui-font-color1: white;
                    --jp-layout-color2: #454545;
                    background-color: #383838;
                }

                .geemap-dark .jupyter-button {
                    --jp-layout-color3: #383838;
                }

                .geemap-colab {
                    background-color: var(--colab-primary-surface-color, white);
                }

                .geemap-colab .jupyter-button {
                    --jp-layout-color3: var(--colab-primary-surface-color, white);
                }
            </style>
            
```
:::

::: {.output .stream .stdout}
    Fields of Image Info:
    type
    bands
    version
    id
    properties
:::
:::

::: {#768a0523-6cfe-4c4b-9027-182cfb2ab587 .cell .markdown}
### Listamos as propriedades da imagem
:::

::: {#0a98a790-c8ff-4cb5-9909-e29e658bfe65 .cell .code execution_count="99"}
``` python
properties = db_info.get('properties', {})

print("Principais propriedades da imagem:")
for i, (key, value) in enumerate(properties.items()):
    #if i >= 20:
        #break
    print(f"{key}: {value}")
```

::: {.output .display_data}
```{=html}

            <style>
                .geemap-dark {
                    --jp-widgets-color: white;
                    --jp-widgets-label-color: white;
                    --jp-ui-font-color1: white;
                    --jp-layout-color2: #454545;
                    background-color: #383838;
                }

                .geemap-dark .jupyter-button {
                    --jp-layout-color3: #383838;
                }

                .geemap-colab {
                    background-color: var(--colab-primary-surface-color, white);
                }

                .geemap-colab .jupyter-button {
                    --jp-layout-color3: var(--colab-primary-surface-color, white);
                }
            </style>
            
```
:::

::: {.output .stream .stdout}
    Principais propriedades da imagem:
    SPACECRAFT_NAME: Sentinel-2A
    SATURATED_DEFECTIVE_PIXEL_PERCENTAGE: 0
    BOA_ADD_OFFSET_B12: -1000
    CLOUD_SHADOW_PERCENTAGE: 0.00995
    system:footprint: {'type': 'LinearRing', 'coordinates': [[-44.03267451457267, -21.788855584088807], [-44.032662385691516, -21.788856239844034], [-42.97109377978401, -21.779232822168048], [-42.971049629682724, -21.779195733186448], [-42.971000879022206, -21.779164058236397], [-42.97099805934788, -21.779149213427456], [-42.977874039055614, -21.283517639469775], [-42.98455406573177, -20.787862405187454], [-42.984593753196776, -20.78782139937015], [-42.98462758024532, -20.78777609565119], [-42.98464345521211, -20.787773441354368], [-44.03913056770457, -20.796918334356786], [-44.03917471960215, -20.79695511525505], [-44.0392233737845, -20.79698649838482], [-44.03922635112839, -20.797001281213877], [-44.0360398342679, -21.292902611882845], [-44.03275992505049, -21.788766754140422], [-44.03272027020462, -21.788807991454828], [-44.032686498194764, -21.788853535941655], [-44.03267451457267, -21.788855584088807]]}
    SENSOR_QUALITY: PASSED
    GENERATION_TIME: 1718128606000
    CLOUDY_PIXEL_OVER_LAND_PERCENTAGE: 0.005141
    CLOUD_COVERAGE_ASSESSMENT: 0.005136
    THIN_CIRRUS_PERCENTAGE: 0.002787
    GRANULE_MEAN_WV: 1.10435
    BOA_ADD_OFFSET_B1: -1000
    BOA_ADD_OFFSET_B2: -1000
    DATASTRIP_ID: S2A_OPER_MSI_L2A_DS_2APS_20240611T175646_S20240611T130251_N05.10
    BOA_ADD_OFFSET_B5: -1000
    BOA_ADD_OFFSET_B6: -1000
    BOA_ADD_OFFSET_B3: -1000
    BOA_ADD_OFFSET_B4: -1000
    BOA_ADD_OFFSET_B9: -1000
    BOA_ADD_OFFSET_B7: -1000
    BOA_ADD_OFFSET_B8: -1000
    GRANULE_ID: L2A_T23KPS_A046853_20240611T130251
    MEAN_INCIDENCE_AZIMUTH_ANGLE_B8: 97.4864347670982
    DATATAKE_TYPE: INS-NOBS
    MEAN_INCIDENCE_AZIMUTH_ANGLE_B9: 97.3779326291694
    MEAN_INCIDENCE_AZIMUTH_ANGLE_B6: 97.2183998836317
    MEAN_INCIDENCE_AZIMUTH_ANGLE_B7: 97.2388528413817
    MEAN_INCIDENCE_AZIMUTH_ANGLE_B4: 97.2335071525099
    NOT_VEGETATED_PERCENTAGE: 13.699968
    MEAN_INCIDENCE_AZIMUTH_ANGLE_B5: 97.2272683048588
    RADIOMETRIC_QUALITY: PASSED
    MEAN_INCIDENCE_AZIMUTH_ANGLE_B2: 97.6609343503978
    MEAN_INCIDENCE_AZIMUTH_ANGLE_B3: 97.3695717237413
    MEAN_INCIDENCE_AZIMUTH_ANGLE_B1: 97.3485238601754
    HIGH_PROBA_CLOUDS_PERCENTAGE: 8.6e-05
    UNCLASSIFIED_PERCENTAGE: 0.000348
    OZONE_SOURCE: AUX_ECMWFT
    GRANULE_MEAN_AOT: 0.085022
    BOA_ADD_OFFSET_B8A: -1000
    SNOW_ICE_PERCENTAGE: 0
    BOA_ADD_OFFSET_B11: -1000
    BOA_ADD_OFFSET_B10: -1000
    GEOMETRIC_QUALITY: PASSED
    system:asset_size: 1819769909
    system:index: 20240611T130251_20240611T130251_T23KPS
    DATATAKE_IDENTIFIER: GS2A_20240611T130251_046853_N05.10
    AOT_RETRIEVAL_ACCURACY: 0
    AOT_RETRIEVAL_METHOD: SEN2COR_DDV
    MEAN_INCIDENCE_AZIMUTH_ANGLE_B8A: 97.2561320446082
    MEAN_SOLAR_AZIMUTH_ANGLE: 31.5716899242967
    VEGETATION_PERCENTAGE: 85.9698
    SOLAR_IRRADIANCE_B12: 85.25
    SOLAR_IRRADIANCE_B10: 367.15
    SOLAR_IRRADIANCE_B11: 245.59
    SOLAR_IRRADIANCE_B8A: 955.32
    FORMAT_CORRECTNESS: PASSED
    system:time_end: 1718111293493
    WATER_VAPOUR_RETRIEVAL_ACCURACY: 0
    OZONE_VALUE: 241.240308
    system:time_start: 1718111293493
    PROCESSING_BASELINE: 05.10
    SENSING_ORBIT_NUMBER: 95
    NODATA_PIXEL_PERCENTAGE: 0
    SENSING_ORBIT_DIRECTION: DESCENDING
    GENERAL_QUALITY: PASSED
    REFLECTANCE_CONVERSION_CORRECTION: 0.970721142888558
    MEDIUM_PROBA_CLOUDS_PERCENTAGE: 0.002263
    MEAN_INCIDENCE_ZENITH_ANGLE_B1: 5.63019769686251
    MEAN_INCIDENCE_ZENITH_ANGLE_B5: 5.43079346156931
    MEAN_INCIDENCE_ZENITH_ANGLE_B4: 5.38909919975624
    MEAN_INCIDENCE_ZENITH_ANGLE_B3: 5.32700511814375
    MEAN_INCIDENCE_ZENITH_ANGLE_B2: 5.27119014953764
    MEAN_INCIDENCE_ZENITH_ANGLE_B9: 5.6903673671276
    MEAN_INCIDENCE_ZENITH_ANGLE_B8: 5.29683926495802
    MEAN_INCIDENCE_ZENITH_ANGLE_B7: 5.52535557995827
    DARK_FEATURES_PERCENTAGE: 0.176804
    MEAN_INCIDENCE_ZENITH_ANGLE_B6: 5.47635365568895
    MEAN_SOLAR_ZENITH_ANGLE: 51.3364267931948
    MEAN_INCIDENCE_ZENITH_ANGLE_B8A: 5.57773773999995
    RADIATIVE_TRANSFER_ACCURACY: 0
    MGRS_TILE: 23KPS
    CLOUDY_PIXEL_PERCENTAGE: 0.005136
    PRODUCT_ID: S2A_MSIL2A_20240611T130251_N0510_R095_T23KPS_20240611T175646
    MEAN_INCIDENCE_ZENITH_ANGLE_B10: 5.39214851395173
    SOLAR_IRRADIANCE_B9: 812.92
    DEGRADED_MSI_DATA_PERCENTAGE: 0
    MEAN_INCIDENCE_ZENITH_ANGLE_B11: 5.49321319295798
    L2A_QUALITY: PASSED
    MEAN_INCIDENCE_ZENITH_ANGLE_B12: 5.61348476939753
    SOLAR_IRRADIANCE_B6: 1287.61
    MEAN_INCIDENCE_AZIMUTH_ANGLE_B10: 97.4883401707118
    SOLAR_IRRADIANCE_B5: 1424.64
    MEAN_INCIDENCE_AZIMUTH_ANGLE_B11: 97.4692305008249
    SOLAR_IRRADIANCE_B8: 1041.63
    MEAN_INCIDENCE_AZIMUTH_ANGLE_B12: 97.543013562988
    SOLAR_IRRADIANCE_B7: 1162.08
    SOLAR_IRRADIANCE_B2: 1959.66
    SOLAR_IRRADIANCE_B1: 1884.69
    SOLAR_IRRADIANCE_B4: 1512.06
    SOLAR_IRRADIANCE_B3: 1823.24
    WATER_PERCENTAGE: 0.137989
:::
:::

::: {#a106eb8a-1c9e-4e42-98f0-d1389fef6908 .cell .markdown}
### Acessamos uma propriedade específica
:::

::: {#30dc77d4-3747-4a4b-a567-50965a400fcd .cell .code execution_count="100"}
``` python
# Acessar as propriedades da imagem
properties = db_info.get('properties', {})

# Especificar a propriedade que você quer obter
cloud_shadow_percentage = properties.get('CLOUD_SHADOW_PERCENTAGE')

# Imprimir o valor da propriedade
print(f"PERCENTUAL DE NUVENS: {cloud_shadow_percentage}")
```

::: {.output .display_data}
```{=html}

            <style>
                .geemap-dark {
                    --jp-widgets-color: white;
                    --jp-widgets-label-color: white;
                    --jp-ui-font-color1: white;
                    --jp-layout-color2: #454545;
                    background-color: #383838;
                }

                .geemap-dark .jupyter-button {
                    --jp-layout-color3: #383838;
                }

                .geemap-colab {
                    background-color: var(--colab-primary-surface-color, white);
                }

                .geemap-colab .jupyter-button {
                    --jp-layout-color3: var(--colab-primary-surface-color, white);
                }
            </style>
            
```
:::

::: {.output .stream .stdout}
    PERCENTUAL DE NUVENS: 0.00995
:::
:::

::: {#382fa85e-8fa3-4a21-a36c-de15ab96db4f .cell .markdown}
### Listamos a escala das bandas iniciadas com a letra \'B\'
:::

::: {#f1a90316-b7b2-4663-80c0-a899bf944f92 .cell .code execution_count="122"}
``` python
# Obter a lista de bandas
band_names = db.bandNames().getInfo()

for band in band_names:
    if band.startswith('B'):
        scale = db.select(band).projection().nominalScale()
        print(f'Banda {band} Escala: {scale.getInfo()} metros')
```

::: {.output .display_data}
```{=html}

            <style>
                .geemap-dark {
                    --jp-widgets-color: white;
                    --jp-widgets-label-color: white;
                    --jp-ui-font-color1: white;
                    --jp-layout-color2: #454545;
                    background-color: #383838;
                }

                .geemap-dark .jupyter-button {
                    --jp-layout-color3: #383838;
                }

                .geemap-colab {
                    background-color: var(--colab-primary-surface-color, white);
                }

                .geemap-colab .jupyter-button {
                    --jp-layout-color3: var(--colab-primary-surface-color, white);
                }
            </style>
            
```
:::

::: {.output .stream .stdout}
    Banda B1 Escala: 60 metros
    Banda B2 Escala: 10 metros
    Banda B3 Escala: 10 metros
    Banda B4 Escala: 10 metros
    Banda B5 Escala: 20 metros
    Banda B6 Escala: 20 metros
    Banda B7 Escala: 20 metros
    Banda B8 Escala: 10 metros
    Banda B8A Escala: 20 metros
    Banda B9 Escala: 60 metros
    Banda B11 Escala: 20 metros
    Banda B12 Escala: 20 metros
:::
:::

::: {#127770c8-7844-4fd0-8854-7abc0dc163af .cell .markdown}
### Escala x tamanho do pixel
:::

::: {#6e4e7ef3-2197-4ac7-aa3b-12e6b8dcfbe8 .cell .markdown}
A escala nominal e o tamanho do *pixel* estão relacionados, mas não são
exatamente a mesma coisa. A escala nominal refere-se à resolução
espacial da banda em termos de tamanho de *pixel* no solo, mas é
expressa como um valor em metros. O tamanho do *pixel* refere-se ao
tamanho físico do *pixel* na imagem, geralmente também expresso em
metros.

No entanto, na prática, a escala nominal normalmente coincide com o
tamanho do *pixel* em imagens de satélite, especialmente em projeções
regulares como **UTM (Universal Transverse Mercator)**, sistema de
coordenadas geográficas que divide o mundo em uma série de zonas
longitudinais e projeta a superfície terrestre em um plano 2D,
utilizando a projeção cilíndrica transversal de *Mercator*.
:::

::: {#968cb648-b4eb-401c-8118-f913f4f38ebb .cell .code execution_count="102"}
``` python
# Iterar sobre as bandas que começam com 'B' e obter suas características
for band in band_names:
    if band.startswith('B'):
        scale = db.select(band).projection().nominalScale().getInfo()
        pixel_size = db.select(band).projection().getInfo()['transform'][0]
        print(f'Banda: {band}, Escala: {scale} metros, Tamanho do Pixel: {pixel_size} metros')
```

::: {.output .display_data}
```{=html}

            <style>
                .geemap-dark {
                    --jp-widgets-color: white;
                    --jp-widgets-label-color: white;
                    --jp-ui-font-color1: white;
                    --jp-layout-color2: #454545;
                    background-color: #383838;
                }

                .geemap-dark .jupyter-button {
                    --jp-layout-color3: #383838;
                }

                .geemap-colab {
                    background-color: var(--colab-primary-surface-color, white);
                }

                .geemap-colab .jupyter-button {
                    --jp-layout-color3: var(--colab-primary-surface-color, white);
                }
            </style>
            
```
:::

::: {.output .stream .stdout}
    Banda: B1, Escala: 60 metros, Tamanho do Pixel: 60 metros
    Banda: B2, Escala: 10 metros, Tamanho do Pixel: 10 metros
    Banda: B3, Escala: 10 metros, Tamanho do Pixel: 10 metros
    Banda: B4, Escala: 10 metros, Tamanho do Pixel: 10 metros
    Banda: B5, Escala: 20 metros, Tamanho do Pixel: 20 metros
    Banda: B6, Escala: 20 metros, Tamanho do Pixel: 20 metros
    Banda: B7, Escala: 20 metros, Tamanho do Pixel: 20 metros
    Banda: B8, Escala: 10 metros, Tamanho do Pixel: 10 metros
    Banda: B8A, Escala: 20 metros, Tamanho do Pixel: 20 metros
    Banda: B9, Escala: 60 metros, Tamanho do Pixel: 60 metros
    Banda: B11, Escala: 20 metros, Tamanho do Pixel: 20 metros
    Banda: B12, Escala: 20 metros, Tamanho do Pixel: 20 metros
:::
:::
