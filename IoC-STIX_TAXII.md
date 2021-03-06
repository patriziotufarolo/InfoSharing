Gli IoC sono esportati in formato STIX 1.2 over TAXII 1.1.

Seguono gli esempi di collezionamento via Minemeld e Cabby

# Minemeld
Seguire la guida per la configurazione di Minemeld [disponibile a questo indirizzo](https://scubarda.com/2018/03/31/minemeld-threat-intelligence-automation-connect-to-an-taxii-service/)

Utilizzare per la configurazione del miner ```taxiing.phishtank``` i seguenti parametri
* _collection_: ```CS-COMMUNITY-TAXII```
* _discovery_service_: ```https://infosharing.cybersaiyan.it/taxii-discovery-service```
* _username_/_password_: NON IMPOSTARE, la connessione è non autenticata

# Cabby
[Installare il software Cabby](https://cabby.readthedocs.io/en/stable/installation.html)

Di seguito la procedura testata su Ubuntu >=16.04
```
sudo apt install virtualenv
virtualenv cabby
. cabby/bin/activate
```
a questo punto la shell sarà nella forma
```
(cabby) gmellini@18-10:~$
```
e si potrà installare il software via pip
```
pip install cabby
```
Una volta installato cabby si procede alla connessione al servizio STIX/TAXXI erogato sul serverinfosharing.cybersaiyan.it

### Fase di Discovery
La fase iniziale è quella di discovery dei servizii erogati dal server STIX/TAXII infosharing.cybersaiyan.it
Per questo si usa il comando _taxii-discovery_ con gli opportuni parametri (la lista è disponbile all'indirizzo /taxii-discovery-service)
```
taxii-discovery --host infosharing.cybersaiyan.it --path /taxii-discovery-service --https
```
L'ouput del comando riporta i servizi disponili
* DISCOVERY
* COLLECTION_MANAGEMENT
* POLL
```
(cabby) gmellini@18-10:~$ taxii-discovery --host infosharing.cybersaiyan.it --path /taxii-discovery-service --https
2018-11-22 15:03:24,459 INFO: Sending Discovery_Request to https://infosharing.cybersaiyan.it/taxii-discovery-service
2018-11-22 15:03:25,323 INFO: 3 services discovered
=== Service Instance ===
  Service Type: DISCOVERY
  Service Version: urn:taxii.mitre.org:services:1.1
  Protocol Binding: urn:taxii.mitre.org:protocol:http:1.0
  Service Address: https://infosharing.cybersaiyan.it/taxii-discovery-service
  Message Binding: urn:taxii.mitre.org:message:xml:1.1
  Available: True
  Message: None

=== Service Instance ===
  Service Type: COLLECTION_MANAGEMENT
  Service Version: urn:taxii.mitre.org:services:1.1
  Protocol Binding: urn:taxii.mitre.org:protocol:http:1.0
  Service Address: https://infosharing.cybersaiyan.it/taxii-collection-management-service
  Message Binding: urn:taxii.mitre.org:message:xml:1.1
  Available: True
  Message: None

=== Service Instance ===
  Service Type: POLL
  Service Version: urn:taxii.mitre.org:services:1.1
  Protocol Binding: urn:taxii.mitre.org:protocol:http:1.0
  Service Address: https://infosharing.cybersaiyan.it/taxii-poll-service
  Message Binding: urn:taxii.mitre.org:message:xml:1.1
  Available: True
  Message: None
  ```

### Lista delle Collection disponibili
Si recupera la lista delle Collection disponibili eseguendo il comando _taxii-collections_
```
taxii-collections --path https://infosharing.cybersaiyan.it/taxii-collection-management-service
```
Il comando evidenzia che è disponibile una unica Collection, **CS-COMMUNITY-TAXII**
```
(cabby) gmellini@18-10:~$ taxii-collections --path https://infosharing.cybersaiyan.it/taxii-collection-management-service
2018-12-10 18:53:36,120 INFO: Sending Collection_Information_Request to https://infosharing.cybersaiyan.it/taxii-collection-management-service
=== Data Collection Information ===
  Collection Name: CS-COMMUNITY-TAXII
  Collection Type: DATA_FEED
  Available: True
  Collection Description: CS-COMMUNITY-TAXII Data Feed
  Supported Content:   urn:stix.mitre.org:xml:1.1.1
  === Polling Service Instance ===
    Poll Protocol: urn:taxii.mitre.org:protocol:http:1.0
    Poll Address: https://infosharing.cybersaiyan.it/taxii-poll-service
    Message Binding: urn:taxii.mitre.org:message:xml:1.1
==================================
```

### Recupero degli IoC dalla Colletion
Il recupero degli indicatori è fatto usando il comando _taxii-poll_
```
taxii-poll --host infosharing.cybersaiyan.it --https --collection CS-COMMUNITY-TAXII --discovery /taxii-discovery-service
```
```
(cabby) gmellini@18-10:~$ taxii-poll --host infosharing.cybersaiyan.it --https --collection CS-COMMUNITY-TAXII --discovery /taxii-discovery-service
[...]
lista degli IoC in formato STIX 1.2 (XML)
[...]
```

### Formato di un generico IoC
Di seguito un esempio di un generico file STIX. I campi principali che descrivono gli IoC sono 
* _indicator:Title_ ==> questo deve definire univocamente la minaccia
* _indicator:Description_ ==> descrizione della minaccia/indicatore
* _indicator:Observable_ ==> in questa sezione sono specificati gli IoC (più di uno anche) associati alla minaccia

```
[...]
    <stix:STIX_Header>
        <stix:Title>Danabot-Gootkit</stix:Title>
        <stix:Description>CERT-PA - Scoperti collegamenti tra Danabot e Gootkit - https://www.cert-pa.it/notizie/scoperti-collegamenti-tra-danabot-e-gootkit/</stix:Description>
        <stix:Short_Description>2018-12-10 15:28:14</stix:Short_Description>
        <stix:Handling>
            <marking:Marking>
                <marking:Controlled_Structure>//node() | //@*</marking:Controlled_Structure>
                <marking:Marking_Structure xsi:type='tlpMarking:TLPMarkingStructureType' color="WHITE"/>
            </marking:Marking>
        </stix:Handling>
        <stix:Information_Source>
            <stixCommon:Identity>
                <stixCommon:Name>CERT-PA via Cyber Saiyan Community</stixCommon:Name>
            </stixCommon:Identity>
        </stix:Information_Source>
    </stix:STIX_Header>
    <stix:Indicators>
        <stix:Indicator id="CYBERSAIYAN:indicator-fd440022-2473-4a86-b76c-2927644c4498" timestamp="2018-12-10T14:28:14.748879+00:00" xsi:type='indicator:IndicatorType'>
            <indicator:Title>Danabot-Gootkit - HASH</indicator:Title>
            <indicator:Type xsi:type="stixVocabs:IndicatorTypeVocab-1.1">File Hash Watchlist</indicator:Type>
            <indicator:Observable id="CYBERSAIYAN:Observable-8578cec3-59ac-457b-a8de-319280513c0a">
                <cybox:Observable_Composition operator="OR">
                    <cybox:Observable id="CYBERSAIYAN:Observable-a7f2defd-f039-4922-8fce-59a10e1bdd46">
                        <cybox:Object id="CYBERSAIYAN:File-db8800f7-4b8e-4ef3-b524-14e712a04b61">
                            <cybox:Properties xsi:type="FileObj:FileObjectType">
                                <FileObj:Hashes>
                                    <cyboxCommon:Hash>
                                        <cyboxCommon:Type xsi:type="cyboxVocabs:HashNameVocab-1.0">SHA256</cyboxCommon:Type>
                                        <cyboxCommon:Simple_Hash_Value>66c3a85ab2f34092fd15cf15e5c289cc70dd65bb86edf8308ca7b5ae1363abb5</cyboxCommon:Simple_Hash_Value>
                                    </cyboxCommon:Hash>
                                </FileObj:Hashes>
                            </cybox:Properties>
                        </cybox:Object>
                    </cybox:Observable>
[...]
```
