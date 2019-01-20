# Przewtarzanie batchowe zadania

## Zadanie

W ramach tego zadania należy na postawie danych z urządzeń stworzyć mechanizm który pozwoli generować dzienne raporty:

- Raport z informacją o sprzedaży
- Report  z informacją o średniej temperaturze

**Raport z informacją o sprzedaży powinien zawierać**

- Date - datę (z dzienną ganulacją)  
- Device - Nazwę/Identyfikator urządzenia
- DeviceLat - Współrzędne urządzenia
- DeviceLot - Współrzędne urządzenia
- ProductName -Nazwa produktu
- Quanity -ilość sprzedanego danego dnia produktu
- Price - Cena produktu
- TotalValue - Informacje o całowitej wartości sprzedanego produktu

![Przykładowe raport](../Imgs/SalesReport.png)

**Report  z informacją o średniej temperaturze**

- Date - datę (z dzienną ganulacją)
- Device - Nazwę/Identyfikator urządzenia
- DeviceLat - Współrzędne urządzenia
- DeviceLot - Współrzędne urządzenia
- AvgTemp - Średnia temperaturza na urządzeniu z danego dnia

![](../Imgs/TempReport.png)

**Docelowo raporty powinny zostać zapisane w DW (dedykowane tabele)  w bazie MSSQL w środowisku onPremise. Proces generowania i zapisywania ma być w pełni zautomatyzowany (możliwość określenia godziny generowania raportów) ** 



Zadanie należy wykonać w dwóch etapach. 

W pierwszym etapie należy wyniki zapisać na Azure Data Lake Storage, w drugim zbudować mechanizm który pozwoli zapisać wyniki do struktur docelowych w bazie danych.

Zadanie może być wykonane na Azure Data Lake Analytics lub Azure Databricks.

### Przetwarzanie na Azure Data Lake Analytics (Tips)

Poniżej przykładowy kod, który pobiera z Azure Blob Storage dane referncyjne

```mssql
DECLARE @inputProducts string = "wasb://data@aw2019.blob.core.windows.net/Ref/SweetsProducts.json";
DECLARE @outputProducts string ="/demo/refdata/SweetsProducts.csv";

USE DATABASE AzureDataWorkshops;
REFERENCE ASSEMBLY [Microsoft.Analytics.Samples.Formats];

//Products
@json =
    EXTRACT jsonString string
    FROM @inputProducts
    USING Extractors.Text(delimiter : '\b', quoting : false);

@jsonProd =
    SELECT Microsoft.Analytics.Samples.Formats.Json.JsonFunctions.JsonTuple(jsonString) AS product
    FROM @json;

@products =
    SELECT int.Parse(product["ProductId"]) AS prodId,
           product["Name"]AS prodName,
           double.Parse(product["Price"], new NumberFormatInfo { NumberDecimalSeparator ="." }) AS prodPrice
    FROM @jsonProd;
	

OUTPUT @products
TO @outputProducts
USING Outputters.Csv(outputHeader : true);
```

