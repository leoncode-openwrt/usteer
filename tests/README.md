# Begrenzte Remote-Updates testen

Mit den normalen Usteer-Buildabhängigkeiten:

```sh
cmake -S . -B build-test -DBUILD_TESTING=ON
cmake --build build-test
ctest --test-dir build-test --output-on-failure
```

Für ASan/UBSan zusätzlich beim Konfigurieren:

```sh
-DCMAKE_C_FLAGS="-fsanitize=address,undefined -fno-omit-frame-pointer" \
-DCMAKE_EXE_LINKER_FLAGS="-fsanitize=address,undefined"
```

Der Test nutzt den unveränderten Parser aus `parse.c`. Er prüft die
byteidentische Weitergabe kleiner Nachrichten, eine Grenze exakt auf und
knapp unter der Nachrichtengröße, 1.001 Paketgrenzen von 200 bis 1.200 Byte
mit jeweils 300 Stationen auf zwei APs sowie einen dritten leeren AP.
Stationsdaten dürfen weder fehlen noch doppelt vorkommen. Metadaten,
unbekannte AP-Felder und der Eingabepuffer müssen unverändert bleiben.
Weitere Fälle: unteilbare übergroße Metadaten im letzten AP, übergroße
Stationsdatensätze, zu kleine MTUs, reine Host-Updates und Sendefehler.
Zwei Tests senden und empfangen die Nachrichten über lokale IPv4-/IPv6-
UDP-Sockets; IPv6-Loopback muss in der Testumgebung verfügbar sein.

## Protokoll und Grenzen

Die Aufteilung verwendet ausschließlich bestehende Nachrichtenfelder.
Jeder Teil enthält dieselben Host-Metadaten und dieselbe Sequenznummer des
logischen Updates. Der bestehende Empfänger verwendet die Sequenznummer
nicht zur Duplikatunterdrückung und aktualisiert Stationsdatensätze einzeln;
fehlende Stationen werden nicht wegen eines Teilupdates gelöscht.
Der Patch benötigt keine neue Empfangslogik und keinen gleichzeitigen
Versionswechsel aller Gegenstellen. Dies ist durch Parser- und Datentests
gestützt, ersetzt aber keinen Mehr-AP-Betriebstest.

Das UDP-Nutzlastlimit ist höchstens 1.200 Byte und bei kleineren lokalen
Interface-MTUs zusätzlich um IPv4-/IPv6- und UDP-Header reduziert. Das
ist keine allgemeine Path-MTU-Ermittlung für Tunnel oder fremde Routen.
Die gesamte Nachricht wird vor dem ersten Teil auf Teilbarkeit geprüft.
Ein einzelner zu großer Metadatenblock oder Stationsdatensatz führt zu
`EMSGSIZE` und einem protokollierten Sendefehler, nicht zu Abschneiden oder
einem Rückfall auf übergroße Datagramme. Ein späterer Socket-Sendefehler
kann wie gewöhnlicher UDP-Verlust ein unvollständig empfangenes Update
hinterlassen; die nächste periodische Aktualisierung erfolgt weiterhin.

Der Patch soll IP-Fragmentierung der Usteer-Updates auf den vorgesehenen
LAN-Interfaces vermeiden. Er ist kein Nachweis einer Behebung sonstiger
WLAN-, Shelly- oder HomePod-Probleme.
