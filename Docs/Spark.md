# Databricks

Databricks są usługą typu PaaS czasem zaliczaną do CaaS (Cluster As a Service) umożliwiająca przetwarzanie danych trybie mikro-batchowym lub batchowym, przy jednoczesnym udostępnieniu możliwości szybkiego skalowania. 

Do tworzenia rozwiązań udostępniony jest Notebook.
1. Trzeba wybrać Databricks, następnie Launch Workshop. 

  ![](../Imgs/RunBricks.png)
2. Potem stworzyć własny workbook.

Do komunikacji z klastrem wykorzystujemy pola. Warto wiedzieć, że pojedynczy notebook pamięta wyniki operacji użytkownika do momentu jego ponownego uruchomienia, resetu lub restartu powiązanego (attached) klastra. 

Do komunikacji służy sparkSession  dostępny za pomocą zmiennej Spark. 

Bez zbytniego rozwodzenia się, umożliwia on komunikację z klastrem, na którym uruchomiony jest Apache Spark. Mamy dzięki temu dostęp do wszystkich możliwości Apache Sparka oraz języka python lub scala z poziomu `Notebooka`.

Omówimy kilka przykładowych i przydatnych operacji.

## Odczyt
Możemy odczytywać pliki z różnych źródeł takich jak: Data Lake Store, Apache Hadoop, Azure Blob Storage, Event Hub, IoT Hub, Kafka i wielu innych.

Potrafi przeczytać pliki z różnych formatów takich jak: CSV, Flat File, Parquet.

Komunikacja z innymi usługami możem odbywać się za pomocą wcześniej wspomnianego Active Directory Service Principala.

Potrzebne nam jest ApplicationId, Key oraz TennantId. W przypadku dostępu do Blob Storage potrzebujemy także jego nazwę oraz klucz. Poniższy kod wklejony do Databricks notebooka umożliwi komunikacje.
```python
key=''
appId=''
tenantId= ''

storage_account=''
storage_key=''

spark.conf.set("fs.azure.account.key.{0}.blob.core.windows.net".format(storage_account), storage_key)
spark.conf.set("dfs.adls.oauth2.access.token.provider.type", "ClientCredential")
spark.conf.set("dfs.adls.oauth2.client.id", appId)
spark.conf.set("dfs.adls.oauth2.credential", key)
spark.conf.set("dfs.adls.oauth2.refresh.url", "https://login.microsoftonline.com/{0}/oauth2/token".format(tenantId))
```

Odczyt danych jest prosty, musimy znać tylko scieżkę dostępu albo do Bloba albo ADLS. Różnica jest tylko w początku url. 
* Dostęp do ADLS : "adls://"
* Dostęp do Bloba: "wasbs://"
```python 
storage_path = "wasbs://data@aw2019.blob.core.windows.net/"
ref_data = spark.read.json(storage_path + 'Ref/SweetsDevices.json')
```
## Odczyt danych strumieniowych
Przetwarzanie danych odbywa się w trybie mikrobaczowym. Czekamy albo na odpowiednią ilość danych albo odpowiedni czas i uruchamiamy przetwarzanie. Możemy ten sam kod uruchamiać dla każdego takiego "okna".

Potrzebujemy adres endpointa opisany powyżej:
```python 
cs = 'Endpoint=sb://'
```
Tworzymy konfiguracje:
```python
sourceEhConf = {'eventhubs.connectionString':cs}
```
Tworzymy wstępnego dafaframe z danych strumieniowych:
```python 
df = spark \
  .readStream \
  .format("eventhubs") \
  .options(**sourceEhConf) \
  .load()
```
Następnie należy dane wydobyć. W Evencie będą one przechowywane w body jako format json, które najpierw trzeba castować na String, potem zdeserializować. 
By, uprościć operację nakładamy także schemę na naszego Eventa, który ma znany format. 
```python
from pyspark.sql.functions import from_json
from pyspark.sql.types import StringType, StructType, StructField, IntegerType, TimestampType

schema = StructType([
  StructField("SerialNumber", StringType()),
  StructField("EventType", IntegerType()),
  StructField("EventValue1", IntegerType()),
  StructField("EventValue2", IntegerType()),
  StructField("EventValue3", IntegerType()),
  StructField("EventTime", TimestampType())])

df = df.withColumn("body", df["body"].cast("string"))
df = df.select(from_json(df.body, schema).alias('body'))
```
Stworzenie okna przetwarzania. Chcemy, by co 30 sekund liczył nam średnią dla danych o tym samym SerialNumber. Wyobraźcie sobie grupowanie w SQL z ograniczeniem czasowym. 
```python
from pyspark.sql.functions import window, avg

windowed = df.select(df.body.SerialNumber.alias('SerialNumber'), df.body.EventValue1.alias('Value'), df.body.EventTime.alias('EventTime'), df.body.EventType.alias('EventType'))

windowed = windowed.groupBy(
    window(windowed.EventTime, "30 second"),
    windowed.SerialNumber
).agg(avg(windowed.Value).alias('AVG_VALUE'))

```
## Składnia Sparka 

Jedną z podstawowych abstrakcji jest `DataFrame` w przypadku języka Python w Sparku. `DataFrame` to zbiór obserwacji, reprezentowanych przez wiersze oraz ich właściwości, reprezentowanych przez kolumny.

`DataFrame` może być wynikiem odczytu danych z pliku: np: `spark.read.csv()`
czy też odczytu danych z okna strumienia: `spark.readstream().load()`

Niezależnie od tego, co jest "pod spodem" `DataFrame'u` można na nim operować w taki sam sposób. 

Projekcja (`select`), dodając kolumnę (`withColumn`), filtrując (`where`), joinując (`join`), etc.
To wszystko są transformacje, które charakteryzują się tym, że efektem ich działania jest kolejny imutowalny `DataFrame`. 

Transformacje mają też to do siebie, że należy je traktować jako plan wykonywanych w kolejności operacji. Dopóki nie wykonamy akcji nic się nie stanie z pierwotnymi danych. Żadne przetworzenie.

Akcje to zapis danych, zebranie i wyświetlenie. 


#### Zapis danych

Do zapisu służą analogiczne przestrzenie: `spark.write`, `spark.writeStream`.

W przypadku zwykłego zapisu wystarczy tylko podać typ, w przypadku zapisu do strumienia są dodatkowe ograniczenia np: musi posiadać wcześniej opisane body.
