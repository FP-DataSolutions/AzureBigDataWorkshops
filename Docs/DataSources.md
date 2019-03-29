# Źródła danych 

Dane z symulatora **SweetsMachineSimulatorApp** (dane w formacie json)

| Pole         | Opis                                                         |
| ------------ | ------------------------------------------------------------ |
| SerialNumber | Identyfikator urządzenia                                     |
| EventType    | Typ zdarzenia: EventType = 0 -Informacje o temp EventType = 1 Informacje o sprzedarzy, EventType = 2 Zatrzymanie urządzenia |
| EventValue1  | Dla EventType =0 Wartość temperatury; EventType = 1 Value1 -Identyfikator produktu |
| EventValue2  | Dla EventType = 1 Value2 -Aktualny stan produktów na urządzeniu np 23 |
| EventValue3  | Aktualnie nie używane                                        |
| EventTime    | Czas wygenerowania zdarzenia (czas urządzenia)               |

## Dane referencyjne

**SweetDevices.json**

| Pole | Opis                           |
| ---- | ------------------------------ |
| id   | Identyfikator urządzenia       |
| lat  | Położenie urządzenia (GPS Lat) |
| lon  | Położenie urządzenia (GPS Lon) |
| min  | Min temperatura na urządzeniu  |
| max  | Max temperatura na urządzeniu  |

**SweetsProducts.json**

| Pole      | Opis                   |
| --------- | ---------------------- |
| ProductId | Identyfikator produktu |
| Name      | Nazwa produktu         |
| Price     | Cena produktu          |

