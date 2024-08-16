# Propriedades das imagens geoespaciais

### Características das bandas

A página de desenvolvedor do **Google** fornece informações detalhadas
sobre o catálogo *Harmonized Sentinel-2 MSI: MultiSpectral Instrument,
Level-2A*, assim como os demais. A tabela de referência contendo
informações sobre as bandas das imagens pode ser obtida no endereço
<https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_SR_HARMONIZED#bands>.

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

### Propriedades gerais

Embora nem todas as características das bandas descritas na tabela de
referência possam ser acessadas diretamente, diversas propriedades das
imagens podem ser exibidas através de métodos fornecidos pelo **Google
Earth Engine**, para extração dos metadados.

Para acesso ao catálogo de imagens geoespaciais do **Google Earth
Engine**, repetimos os procedimentos descritos anteriormente

### Importamos as bibliotecas

``` python
import ee
import geemap
import rasterio
from rasterio.plot import show
import matplotlib.pyplot as plt
from matplotlib.patches import Polygon
```

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

### Inicializamos a API

``` python
# Processo de autenticação
ee.Authenticate()

# Inicializamos a biblioteca
ee.Initialize()
```

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

### Definição de parâmetros

*Intervalo temporal*

``` python
data_inicial = '2024-04-01'
data_final = '2024-07-31'
```

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

*Área de interesse*

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

### Escolha da coleção de imagens

Aplicamos os filtros de acordo com os parâmetros:

``` python
db = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED') \
                .filterBounds(aoi) \
                .filterDate(ee.Date(data_inicial), ee.Date(data_final)) 
```

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

### Contagem das imagens disponíveis

``` python
dbCount = db.size()
print('Quantidade: ', str(dbCount.getInfo())+'\n')
```

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

    Quantidade:  48

### Extraímos a imagem com menor cobertura de nuvens

``` python
db = ee.Image(db.sort('CLOUDY_PIXEL_PERCENTAGE').first())
```

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

### Contagem das bandas disponíveis

``` python
# Obtém os nomes das bandas
band_names = db.bandNames()

# Conta o número de bandas
number_of_bands = band_names.size().getInfo()

# Imprime o número de bandas
print(f'Número de bandas: {number_of_bands}')
```

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

    Número de bandas: 26

### Bandas que iniciam com a letra \'B\'

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

### Extraímos as propriedades das bandas selecionadas

``` python
db_bands.getInfo()['bands']
```

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

## Metadados

Para obter os metadados no **Google Earth Engine**, chamamos o método
*getInfo* no objeto *Image* criado.

``` python
db_info = db.getInfo()
```

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

## Listamos o dicionário de metadados associados à imagem

``` python
print("Fields of Image Info:")
for key in db_info:
    print(key)
```

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

    Fields of Image Info:
    type
    bands
    version
    id
    properties

### Listamos as propriedades da imagem

``` python
properties = db_info.get('properties', {})

print("Principais propriedades da imagem:")
for i, (key, value) in enumerate(properties.items()):
    #if i >= 20:
        #break
    print(f"{key}: {value}")
```

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

### Acessamos uma propriedade específica

``` python
# Acessar as propriedades da imagem
properties = db_info.get('properties', {})

# Especificar a propriedade que você quer obter
cloud_shadow_percentage = properties.get('CLOUD_SHADOW_PERCENTAGE')

# Imprimir o valor da propriedade
print(f"PERCENTUAL DE NUVENS: {cloud_shadow_percentage}")
```

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

    PERCENTUAL DE NUVENS: 0.00995

### Listamos a escala das bandas iniciadas com a letra \'B\'

``` python
# Obter a lista de bandas
band_names = db.bandNames().getInfo()

for band in band_names:
    if band.startswith('B'):
        scale = db.select(band).projection().nominalScale()
        print(f'Banda {band} Escala: {scale.getInfo()} metros')
```

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

### Escala x tamanho do pixel

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

``` python
# Iterar sobre as bandas que começam com 'B' e obter suas características
for band in band_names:
    if band.startswith('B'):
        scale = db.select(band).projection().nominalScale().getInfo()
        pixel_size = db.select(band).projection().getInfo()['transform'][0]
        print(f'Banda: {band}, Escala: {scale} metros, Tamanho do Pixel: {pixel_size} metros')
```

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
