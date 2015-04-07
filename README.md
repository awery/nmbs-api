# Unofficial documentation of the NMBS/SNCB API

This is a compilation of the findings I made about the API used by the [Android NMBS/SNCB app](https://play.google.com/store/apps/details?id=de.hafas.android.sncbnmbs). I used [mitmproxy](http://mitmproxy.org/) to analyse the requests and responses.

No HTTPS is used and no authentication is required (unlike the old Railtime API) for all different endpoints.

## Get all stations
Endpoint still to be found.

## Find a station
POST http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/query.exe/dn

Parameter

    <?xml version="1.0" encoding="UTF-8"?>
    <ReqC ver="3.1.12 (33911)" prod="androidversion" lang="fr">
      <MLcReq>
        <MLc n="Station name to search" t="ALLTYPE"/>
      </MLcReq>
    </ReqC>

### Example 

Request

    curl --data-urlencode '<?xml version="1.0" encoding="UTF-8"?><ReqC ver="3.1.12 (33911)" prod="androidversion" lang="fr"><MLcReq><MLc n="ott" t="ALLTYPE"/></MLcReq></ReqC>' http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/query.exe/dn

Response

    <?xml version="1.0" encoding="UTF-8"?>
    <ResC xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="No path to XML scheme defined" ver="1.1" prod="String" lang="EN">
        <MLcRes flag="FINAL">
            <MLc t="ST" n="Ottignies" i="A=1@O=Ottignies@X=4569360@Y=50673667@U=80@L=008811601@B=1@p=1428275999@n=ac.1=GA@" x="4569360" y="50673667" />
            <MLc t="ST" n="OTTRE Eglise [TEC]" i="A=1@O=OTTRE Eglise [TEC]@X=5835554@Y=50249196@U=80@L=000260702@B=1@p=1428275999@" x="5835554" y="50249196" />
            <MLc t="ST" n="Ottenburg Dorp [De Lijn]" i="A=1@O=Ottenburg Dorp [De Lijn]@X=4615888@Y=50751774@U=80@L=000214807@B=1@p=1428275999@" x="4615888" y="50751774" />
            <MLc t="ST" n="OTTIGNIES Av. Des Combattants [TEC]" i="A=1@O=OTTIGNIES Av. Des Combattants [TEC]@X=4569109@Y=50669164@U=80@L=000243752@B=1@p=1428275999@" x="4569109" y="50669164" />
            [...]
        </MLcRes>
    </ResC>

## Find a connection
POST http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/query.exe/fn (FR)
POST http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/query.exe/nl (NL)
POST http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/query.exe/en (EN)
POST http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/query.exe/de (DE)

| Parameter  | Value |
| ------------- | ------------- |
| `time`  | time in format HH:MM (eg. `07:10`) |
| `date`  | date in format DD.MM.YYYY (eg. `13.04.2015`) |
| `timeSel`  | `depart` or `arrive` |
| `SID`  | [Departure station].i (see **Find a station** response) |
| `ZID`  | [Arrival station].i (see **Find a station** response) |
| `start`  | `1`  |
| `hcount`  | `0`  |
| `REQ0HafasNumCons1`  | `0`  |
| `REQ0JourneyProduct_prod_list_1`  | `11111111111111` |
| `REQ0HafasNumCons2`  | `3`  |
| `ignoreMinuteRound`  | `yes`  |
| `h2g-direct`  | `11`  |
| `REQ0HafasNumCons0`  | `3`  |
| `clientType`  | eg. `ANDROID`  |
| `androidversion`  | eg. `3.1.12 (33911)`  |
| `clientSystem`  | eg. `Android19`  |
| `clientDevice`  | eg. `GT-I9505`  |
| `htype`  | eg. `GT-I9505`  |

### Example
From Ottignies to Brussels-Nord on 13/04/2015, departure after 7:10.

Request

    curl --data-urlencode "start=1&clientType=ANDROID&androidversion=3.1.12%20(33911)&time=07%3a10&hcount=0&REQ0HafasNumCons1=0&clientSystem=Android19&date=13.04.2015&REQ0JourneyProduct_prod_list_1=11111111111111&REQ0HafasNumCons2=3&ignoreMinuteRound=yes&h2g-direct=11&REQ0HafasNumCons0=3&timeSel=depart&SID=A%3d1%40O%3dOttignies%40X%3d4569360%40Y%3d50673667%40U%3d80%40L%3d008811601%40B%3d1%40p%3d1428018535%40n%3dac.1%3dGA%40&ZID=A%3d1%40O%3dBrux.-Nord%20%2f%20Bru.-Noord%40X%3d4360846%40Y%3d50859663%40U%3d80%40L%3d008812005%40B%3d1%40p%3d1428018535%40n%3dac.1%3dGA%40&clientDevice=GT-I9505&htype=GT-I9505" http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/query.exe/fn

Response

    Content-Type:    application/octet-stream

To be decoded...

## Find a train timetable

This is a two steps process:

1. With the train number and the date (optional), find the _train link_
1. Get the timetable with the _train link_ and the date

### Find the train link
POST http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/trainsearch.exe/fn (FR)
POST http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/trainsearch.exe/nl (NL)
POST http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/trainsearch.exe/en (EN)
POST http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/trainsearch.exe/de (DE)

| Parameter  | Value |
| ------------- | ------------- |
| `trainname`  | Train number (eg. `3615`)  |
| `date`  | date in format DD.MM.YYYY (eg. `13.04.2015`) - _optional_ |
| `vtModeTs`  | `weekday`  |
| `productClassFilter`  | `69`  |
| `L`  | `vs_json.vs_hap` (json) |
| `maxResults`  | eg. `50`  |
| `hcount`  | `0`  |
| `clientType`  | eg. `ANDROID`  |
| `androidversion`  | eg. `3.1.12 (33911)`  |
| `clientSystem`  | eg. `Android19`  |
| `clientDevice`  | eg. `GT-I9505`  |
| `htype`  | eg. `GT-I9505`  |

If the `date` parameter is not present, the response may contain multiple suggestions.

#### Example with date parameter
Train IC 2427 on 13/04/2015


Request

     curl --data "vtModeTs=weekday&productClassFilter=69&clientType=ANDROID&androidversion=3.1.12%20(33911)&hcount=0&maxResults=50&clientSystem=Android19&date=13.04.2015&trainname=2427&clientDevice=GT-I9505&htype=GT-I9505&L=vs_json.vs_hap" http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/trainsearch.exe/fn


Response

    { "suggestions": [
        {"value":"IC  2427","cycle":"0","pool":"80","id":"1583","dep":"Luik-Paleis","trainLink":"795198/266649/392280/68926/80","journParam":"identifiedByjourneyID=IC  2427:Luik-Paleis:13.04.2015:05:42&externalId=1583&lineName=1583&internalID=1583&cycle=0&poolUIC=80&trainName=IC  2427&trainType=007&trainClass=2&firstStationName=Luik-Paleis&firstStationEvaID=8841525&firstStationDep=13.04.2015 05:42","pubTime":"20:33","depDate":"13.04.2015","depTime":"05:42","arr":"Brussel-Zuid","arrTime":"07:58","vt":"2. Fév jusqu'au 11. Déc 2015 Lu - Ve; pas 6. Avr, 1., 14., 25. Mai, 21. Jul, 11. Nov"}
    ]};

The response is not valid JSON due to the ending semi-colon.

#### Example without date parameter
Train IC 3615 on 13/04/2015


Request

    curl --data "vtModeTs=weekday&productClassFilter=69&clientType=ANDROID&androidversion=3.1.12%20(33911)&hcount=0&maxResults=50&clientSystem=Android19&trainname=3615&clientDevice=GT-I9505&htype=GT-I9505&L=vs_json.vs_hap" http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/trainsearch.exe/fn

Response

    { "suggestions": [
        {"value":"IC  3615","cycle":"4","pool":"80","id":"2261","dep":"De Panne","trainLink":"795198/267327/392280/68930/80","journParam":"identifiedByjourneyID=IC  3615:De Panne:06/04/15:15:52&externalId=2261&lineName=2261&internalID=2261&cycle=4&poolUIC=80&trainName=IC  3615&trainType=007&trainClass=2&firstStationName=De Panne&firstStationEvaID=8892338&firstStationDep=06/04/15 15:52","pubTime":"20:35","depDate":"06/04/15","depTime":"15:52","arr":"Landen","arrTime":"19:21","vt":"11. jusqu'au 19. Avr 2015 Sa, Di"},
        {"value":"IC  3615","cycle":"1","pool":"80","id":"2268","dep":"De Panne","trainLink":"915888/307564/503416/53589/80","journParam":"identifiedByjourneyID=IC  3615:De Panne:06/04/15:15:52&externalId=2268&lineName=2268&internalID=2268&cycle=1&poolUIC=80&trainName=IC  3615&trainType=007&trainClass=2&firstStationName=De Panne&firstStationEvaID=8892338&firstStationDep=06/04/15 15:52","pubTime":"20:35","depDate":"06/04/15","depTime":"15:52","arr":"Landen","arrTime":"19:21","vt":"14. Déc 2014 jusqu'au 4. Jan 2015 Je, Sa, Di; pas 18. Déc; aussi 28., 29. Mar"},
        {"value":"IC  3615","cycle":"1","pool":"80","id":"2274","dep":"De Panne","trainLink":"716277/241033/773650/148067/80","journParam":"identifiedByjourneyID=IC  3615:De Panne:06/04/15:15:52&externalId=2274&lineName=2274&internalID=2274&cycle=1&poolUIC=80&trainName=IC  3615&trainType=007&trainClass=2&firstStationName=De Panne&firstStationEvaID=8892338&firstStationDep=06/04/15 15:52","pubTime":"20:35","depDate":"06/04/15","depTime":"15:52","arr":"Landen","arrTime":"19:21","vt":"2., 3. Mai"},
        {"value":"IC  3615","cycle":"0","pool":"80","id":"2279","dep":"De Panne","trainLink":"286425/97754/35684/77633/80","journParam":"identifiedByjourneyID=IC  3615:De Panne:06/04/15:15:52&externalId=2279&lineName=2279&internalID=2279&cycle=0&poolUIC=80&trainName=IC  3615&trainType=007&trainClass=2&firstStationName=De Panne&firstStationEvaID=8892338&firstStationDep=06/04/15 15:52","pubTime":"20:35","depDate":"06/04/15","depTime":"15:52","arr":"Landen","arrTime":"19:21","vt":"Lu - Ve, pas 25. Déc, 1. Jan, 26. jusqu'au 30. Jan 2015, 6. Avr, 1., 14., 25. Mai, 21. Jul, 11. Nov"},
        {"value":"IC  3615","cycle":"0","pool":"80","id":"2280","dep":"De Panne","trainLink":"927831/311557/408824/104865/80","journParam":"identifiedByjourneyID=IC  3615:De Panne:06/04/15:15:52&externalId=2280&lineName=2280&internalID=2280&cycle=0&poolUIC=80&trainName=IC  3615&trainType=007&trainClass=2&firstStationName=De Panne&firstStationEvaID=8892338&firstStationDep=06/04/15 15:52","pubTime":"20:35","depDate":"06/04/15","depTime":"15:52","arr":"Landen","arrTime":"19:21","vt":"28. Fév jusqu'au 12. Déc 2015 Sa, Di; pas 7. jusqu'au 29. Mar 2015, 11. jusqu'au 19. Avr 2015, 2., 3. Mai; aussi 6. Avr, 1., 14., 25. Mai, 21. Jul, 11. Nov"}
    ]};

The response is not valid JSON due to the ending semi-colon.


### Get the train timetable
GET http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/traininfo.exe/fn/[train_link] (FR)
GET http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/traininfo.exe/nl/[train_link] (NL)
GET http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/traininfo.exe/en/[train_link] (EN)

| Parameter  | Value |
| ------------- | ------------- |
| `date`  | date in format DD.MM.YYYY (eg. `13.04.2015`) - _optional_ |
| `L`  | `vs_java3` (XML) or `vs_json.vs_hap` (JSON) |
| `hcount`  | `0`  |
| `rt`  | `1`  |
| `clientType`  | eg. `ANDROID`  |
| `androidversion`  | eg. `3.1.12 (33911)`  |
| `clientSystem`  | eg. `Android19`  |
| `clientDevice`  | eg. `GT-I9505`  |
| `htype`  | eg. `GT-I9505`  |

The response will contain different information in the XML format and in JSON:
* Unique in XML: arrival and departure delays
* Unique in JSON: arrival and departure times of non-stopping stations

### Example in XML
Train IC 2427 on 13/04/2015


Request

     curl "http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/traininfo.exe/fn/795198/266649/392280/68926/80?clientType=ANDROID&androidversion=3.1.hcount=0&clientSystem=Android19&date=13.04.2015&clientDevice=GT-I9505&htype=GT-I9505&L=vs_java3&rt=1"


Response

    <Journey approxDelay="0">
        <St evaId="8841525" name="Liege-Palais" depTime="05:42" depDelay="" ddelay="" platform="" x="5570453" y="50646349" capacity="0|0" />
        <St evaId="8841558" name="Liege-Jonfosse" arrTime="05:44" arrDelay="" adelay="" depTime="05:45" depDelay="" ddelay="" platform="" x="5561131" y="50640299" capacity="0|0" />
        <St evaId="8841004" name="Liege-Guillemins" arrTime="05:49" arrDelay="" adelay="" depTime="05:51" depDelay="" ddelay="" platform="" x="5566695" y="50624550" capacity="0|0" />
        <St evaId="8843133" name="Sclessin" platform="" x="5558911" y="50609844" capacity="0|0" />
        <St evaId="8843109" name="Tilleur" platform="" x="5528545" y="50619678" capacity="0|0" />
        <St evaId="8843141" name="Pont-De-Seraing" platform="" x="5510162" y="50619651" capacity="0|0" />
        <St evaId="8843158" name="Jemeppe-Sur-Meuse" platform="" x="5497874" y="50618446" capacity="0|0" />
        <St evaId="8843166" name="Flemalle-Grande" platform="" x="5480983" y="50605349" capacity="0|0" />
        <St evaId="8843224" name="Leman" platform="" x="5468309" y="50600396" capacity="0|0" />
        <St evaId="8843208" name="Flemalle-Haute" arrTime="06:01" arrDelay="" adelay="" depTime="06:02" depDelay="" ddelay="" platform="" x="5457656" y="50595308" capacity="0|0" />
        [...]
    </Journey>

The response contains arrival and departure delays for stopping stations but no time information at all for non-stopping stations.

### Example in JSON
Train IC 2427 on 13/04/2015


Request

     curl "http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/traininfo.exe/fn/795198/266649/392280/68926/80?clientType=ANDROID&androidversion=3.1.hcount=0&clientSystem=Android19&date=13.04.2015&clientDevice=GT-I9505&htype=GT-I9505&L=vs_json.vs_hap&rt=1"


Response

    { "suggestions": [
        {
        "object_0":"identifiedByexternalID=2427&#183;88____&#183;NULL",
        "extId":"2427&#183;88____&#183;NULL",
        "name":"IC  2427",
        [...]
        "locations":[
            {"name":"Liege-Palais","x":"5570453","y":"50646349","evaId":"8841525","depTime":"05:42","arrTime":"","depDate":"13/04/15","arrDate":""},
            {"name":"Liege-Jonfosse","x":"5561131","y":"50640299","evaId":"8841558","depTime":"05:45","arrTime":"05:44","depDate":"13/04/15","arrDate":"13/04/15"},
            {"name":"Liege-Guillemins","x":"5566695","y":"50624550","evaId":"8841004","depTime":"05:51","arrTime":"05:49","depDate":"13/04/15","arrDate":"13/04/15"},
            {"name":"Sclessin","x":"5558911","y":"50609844","evaId":"8843133","depTime":"05:55","arrTime":"05:55","depDate":"13/04/15","arrDate":"13/04/15"},
            {"name":"Tilleur","x":"5528545","y":"50619678","evaId":"8843109","depTime":"05:56","arrTime":"05:56","depDate":"13/04/15","arrDate":"13/04/15"},
            {"name":"Pont-De-Seraing","x":"5510162","y":"50619651","evaId":"8843141","depTime":"05:57","arrTime":"05:57","depDate":"13/04/15","arrDate":"13/04/15"},
            {"name":"Jemeppe-Sur-Meuse","x":"5497874","y":"50618446","evaId":"8843158","depTime":"05:57","arrTime":"05:57","depDate":"13/04/15","arrDate":"13/04/15"},
            {"name":"Flemalle-Grande","x":"5480983","y":"50605349","evaId":"8843166","depTime":"05:58","arrTime":"05:58","depDate":"13/04/15","arrDate":"13/04/15"},
            {"name":"Leman","x":"5468309","y":"50600396","evaId":"8843224","depTime":"05:59","arrTime":"05:59","depDate":"13/04/15","arrDate":"13/04/15"},
            {"name":"Flemalle-Haute","x":"5457656","y":"50595308","evaId":"8843208","depTime":"06:02","arrTime":"06:01","depDate":"13/04/15","arrDate":"13/04/15"},
            [...]
        ],
        "dep":"Liege-Palais",
        "arr":"Bruxelles-Midi",
        "depEva":"8841525",
        "arrEva":"8814001",
        "depTime":"05:42",
        "arrTime":"07:58",
        "pubTime":"20:44",
        "summary":"IC  2427, Liege-Palais (05:42) - Bruxelles-Midi (07:58)",
        [...]
        }
    ]}

The response does not contain delay information but contains arrival and departure time for non-stopping stations.

## Get the liveboard of a station
POST http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/stboard.exe/fn (FR)
POST http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/stboard.exe/nl (NL)
POST http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/stboard.exe/en (EN)
POST http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/stboard.exe/de (DE)

| Parameter  | Value |
| ------------- | ------------- |
| `time`  | time in format HH:MM (eg. `07:10`) |
| `date`  | date in format DD.MM.YYYY (eg. `13.04.2015`) |
| `inputTripelId`  | [Station].i (see **Find a station** response) |
| `boardType`  | `dep` or `arr` |
| `maxJourneys`  | eg. `50` |
| `productsFilter`  | `11111111111111` (all) |
| `L`  | `vs_java3` |
| `start`  | `yes` |
| `showHimMessages`  | `1` |
| `hcount`  | `0` |
| `clientType`  | eg. `ANDROID`  |
| `androidversion`  | eg. `3.1.12 (33911)`  |
| `clientSystem`  | eg. `Android19`  |
| `clientDevice`  | eg. `GT-I9505`  |
| `htype`  | eg. `GT-I9505`  |

`productsFilter` can have the following values:
* `11111111111111` All
* `11011101000111` International trains only
* `01111101000111` IC/IR/P/ICT only
* `01011111000111` City Rail/L only
* `01011101100111` Métro only
* `01011101010111` Bus only
* `01011101001111` Tram only

Combinations are also possible (to be further completed).

### Example

Request

    curl --data "start=yes&inputTripelId=A%3d1%40O%3dOttignies%40X%3d4569360%40Y%3d50673667%40U%3d80%40L%3d008811601%40B%3d1%40p%3d1428018535%40n%3dac.1%3dGA%40&clientType=ANDROID&androidversion=3.1.12%20(33911)&time=07%3a10&hcount=0&productsFilter=11111111111111&clientSystem=Android19&date=13.04.2015&maxJourneys=50&L=vs_java3&showHimMessages=1&clientDevice=GT-I9505&htype=GT-I9505&boardType=dep" http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/stboard.exe/fn

Response

    <StationTable>
        <St evaId="8811601" name="Ottignies" />
        <Journey fpTime="07:12" fpDate="13/04/15" delay="-" targetLoc="Louvain-La-Neuve-Univ." dirnr="8811676" hafasname="L   3956" prod="L   3956#L" class="64" dir="Louvain-La-Neuve-Univ." administration="88____" is_reachable="0" />
        <Journey fpTime="07:12" fpDate="13/04/15" delay="-" targetLoc="WAVRE Gare [TEC]" dirnr="242668" hafasname="Bus   22" prod="Bus   22#Bus" class="512" dir="WAVRE Gare [TEC]" administration="SNCB" depStation="OTTIGNIES Gare [TEC]" is_reachable="0" />
        <Journey fpTime="07:12" fpDate="13/04/15" delay="-" targetLoc="LOUVAIN-LA-NEUVE Gare D'Autobus [TEC]" dirnr="242650" hafasname="Bus    3" prod="Bus    3#Bus" class="512" dir="LOUVAIN-LA-NEUVE Gare D'Autobus [TEC]" administration="SNCB" depStation="OTTIGNIES Gare [TEC]" is_reachable="0" />
        <Journey fpTime="07:14" fpDate="13/04/15" delay="-" targetLoc="Bruxelles-Midi" dirnr="8814001" hafasname="IC  2427" prod="IC  2427#IC" class="4" dir="Bruxelles-Midi" administration="88____" is_reachable="0" />
        <Journey fpTime="07:17" fpDate="13/04/15" delay="-" targetLoc="Bruxelles-Midi" dirnr="8814001" hafasname="L   6578" prod="L   6578#L" class="64" dir="Bruxelles-Midi" administration="88____" is_reachable="0" />
        <Journey fpTime="07:18" fpDate="13/04/15" delay="-" platform="3" targetLoc="Luxembourg (l)" dirnr="8200100" hafasname="IC  2106" prod="IC  2106#IC" class="4" dir="Luxembourg (l)" administration="88____" is_reachable="0" />
        [...]
    </StationTable>

## Get the perturbations RSS feed

GET http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/help.exe/fn?tpl=rss_feed (FR)
GET http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/help.exe/nl?tpl=rss_feed (NL)
GET http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/help.exe/en?tpl=rss_feed (EN)
GET http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/help.exe/de?tpl=rss_feed (DE)

### Example

Request

    curl http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/help.exe/fn?tpl=rss_feed

Response

    <?xml version="1.0" encoding="utf-8"?>
    <rss xmlns:dc="http://purl.org/dc/elements/1.1/" version="2.0">
        <channel>
        <title>SNCB Perturbations sur le réseau</title>
        <description></description>
        <link>http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/help.exe/fn?tpl=him_map</link>
        <language>fr-be</language>
        <copyright>&#169; 2013 SNCB</copyright>
        <pubDate>Mon, 06 Apr 2015 21:16:00 +0100</pubDate>
        <item>
            <title><![CDATA[Tournai [NMBS/SNCB] - Lille Flandres(f): Dégâts à la caténaire.]]></title>
            <description><![CDATA[Dégâts à la caténaire. <br /><br />Trafic perturbé. <br /><br />Des retards et suppressions sont possibles. <br /><br />Certains trains entre Tournai et Lille Flandres sont remplacés par un bus.<br /><br />Durée du dérangement indéterminée.<br /> <br />Ecoutez les annonces faites dans le train. <br />Ecoutez les annonces faites en gare. <br />Consultez les tableaux d'affichage de la gare.<br/><a href="http://www.belgianrail.be/jp/download/brail_him/1425215921219_Aff 62x100-FR-WE-Lille-03.pdf">Horaire week-end et jours fériés</a><br/><a href="http://www.belgianrail.be/jp/download/brail_him/1427104588807_Aff 62x100-Fr-semaine-Lille-03-15.pdf">Horaire en semaine</a><br/>]]></description>
            <link>http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/help.exe/fn?L=vs_himmap&amp;tpl=showmap_external&amp;messageID=5464&amp;channelFilter=timetable&amp;</link>
            <pubDate>Tue, 31 Mar 2015 05:28:36 +0200</pubDate>
            <guid isPermaLink="false">http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/help.exe/fn?tpl=him_map5464</guid>
            <dc:creator></dc:creator>
        </item>
        <item>
            <title><![CDATA[Statte - Andenne: Affaissement de terrain.]]></title>
            <description><![CDATA[Affaissement de terrain.<br /><br /> Trafic perturbé.<br /><br /> Des retards sont possibles.<br /><br /> Durée des travaux indéterminée.<br /><br /> Ecoutez les annonces faites dans le train.]]></description>
            <link>http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/help.exe/fn?L=vs_himmap&amp;tpl=showmap_external&amp;messageID=5979&amp;channelFilter=timetable&amp;</link>
            <pubDate>Mon, 06 Apr 2015 04:56:41 +0200</pubDate>
            <guid isPermaLink="false">http://www.belgianrail.be/jp/sncb-nmbs-routeplanner/help.exe/fn?tpl=him_map5979</guid>
            <dc:creator></dc:creator>
        </item>
        </channel>
    </rss>