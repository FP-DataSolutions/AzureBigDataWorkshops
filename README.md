# Azure Big Data Workshops



## PoC



Nasz klient posiada wiele urządzeń do sprzedaży słodyczy/napójów itd. Urządzenia te posiadają dostęp do sieci. Nasz klient chce mieć możliwość monitorowania stanu pracy czy urządzeń w zakresie podstawowych parametrów pracy oraz stanu danego towaru na danym urządzeniu tak aby, mógł szybko zareagować w przypadku wykryciu anamalii lub braku towaru. Dodatkowo nasz klient chce mieć możliwość generowania dziennych/miesięcznych/rocznych statystyk dotyczących funkcjnowania jego biznesu np. ilość sprzedanego towaru danego dnia, zysk z danego urządzenia.

Wymagania niefunkcjonalne:

Rozwiązanie ma być skalowalne (aktualnie nasz klient ma 1000 urządzeń), ale rozwiązania ma umożliwiać obsługę 100 000 urządzeń.



### Architektura rozwiązania

### Repozytorium (dane)

Przykłady dostępne są na repozytorium 

[github]: https://github.com/FP-DataSolutions/AzureBigDataWorkshops/tree/develop	"Azure Big Data Workshops"



| SweetsMachineSimulatorApp | https://github.com/FP-DataSolutions/AzureBigDataWorkshops/tree/develop/SweetMachineSimulator/Binary |
| ------------------------- | --------------------------------------------------------------------------------------------------- |
| Date referencyjne         | https://github.com/FP-DataSolutions/AzureBigDataWorkshops/tree/develop/Data/Ref                     |
|                           |                                                                                                     |
|                           |                                                                                                     |



### Instrukcja

Rozwiązanie zosta zbudowane na bazie stosu technologicznego chmury obliczeniowej Azure. Można wyróżnić dwie części funkcjonalne rozwiązania:
* ścieżkę hot - przetwarzającą dane przyrostowe w czasie near-real-time
* ścieżkę cold - przetwarzającą większy zakres danych.

#### Ścieżka hot 
1. Do zbierania danych potrzebny jest Hub (Event Hub, IoT Hub, Kafka), który umożliwia kolejkowanie, zbieranie danych, a następnie konsumowanie.
2. Do analizy danych potrzebne jest rozwiązanie, które umożliwia konsumowanie danych eventowych, a następnie przetworzenie ich oraz zapis lub przesanie wyników analizy danych w czasie rzeczywistym lub prawie rzeczywistym. Dostępne rozwiązania to Azure Stream Analitics, Databricks.  
3. Przechowywanie danych wynikowych lub danych wykorzystywanych w przetwarzaniu. Do tego celu możemy wykorzystać Blob Storage, Data Lake Storage lub także huby.
4. Reagowanie na pojawienie się nowych informacji. Do tego celu możemy wykorzystać Azure Functons.

#### Ścieżka cold 
1. Miejsce przechowywania danych. Do tego celu możemy wykorzystać wszelkiego rodzaju Storage: Blob Storage, Data Lake Storage. 
2. Analiza dnaych umożliwiająca skalowalne przetwarzanie zwiększającej się ilości danych. Do tego celu można wykorzystać: Data Lake Analitics, HdInsight, Databriks.
3. Prezentacja wyników. Do tego celu można wykorzystać notebooki zeppelin, juypter) oraz Power Bi. 

### Krok 1: Dane 
[Podgląd](./Docs/DataSources.md)
### Krok 2: Urządzenia IoT
[Podgląd](./Docs/IoT.md)
### Krok 3: Przechowywanie danych 
[Podgląd](./Docs/Storage.md)
### Krok 4: Przetwarzanie danych strumieniowych
[Azure Analitics](./Docs/RealTimeProcessingSA.md)
[Spark Streaming](./Docs/Spark.md)
### Krok 5: Przetwarzanie w trybie batchowym
[Podgląd](./Docs/BatchProcessing.md)
#### Azure Data Lake Analytics
[Rozwiązanie](./Docs/BatchProcessingADLA.md)
#### Azure Databricks
[Podgląd](./Docs/Spark.md)
[Rozwiązanie Spark SQL](./Docs/BatchProcessingSparkSQL.md)
### Krok 6: Integracja rozwiązania  Azure Data Factory
[Rozwiązanie -in progress](./Docs/ADF.md)