# Serielle Kommunikation
Die serielle Kommunikation mit dem Festo Roboter erfolgt über eine COM-Schnittstelle. Diese wird mit der `serial.py` library für Python gesteuert.

## Die pySerial Library
### Installation
Die library kann mit dem Befehl 
```
pip install pySerial
```
in der Kommandozeile installiert werden.

### Importieren
In Python kann die library wie folgt importiert werden:
```py
import serial
```

### Initialisieren des COM-Port
Mit der `Serial()` Funktion kann dann eine Verbindung mit dem COM-Port hergestellt werden. Das COM-Port-Objekt wird in der variable `ser` gespeichert.
```py
ser = serial.Serial(port="COM4", baudrate=9600, stopbits=serial.STOPBITS_TWO, parity=serial.PARITY_EVEN)
```
Die Baudrate gibt die Datenrate an, mit der das Zielgerät arbeitet. Die Stoppbits und die Parität können in der Dokumentation der Schnittstelle oder des Zielgeräts gefunden werden. Alle Parameter bis auf den Port sind optional.

### Kommunikation
Bevor die Kommunikation gestartet werden kann muss der Port geöffnet werden.
```py
ser.open()
```
Nach dem Senden der Daten sollte der Port wieder geschlossen werden.
```py
ser.close()
```
Für die Kommunikation stehen viele Funktionen zur Verfügung. In diesem Fall möchten wir nur Daten an den Roboter senden und können hierfür die "write" Funktion nutzen.
```py
ser.write(daten)
```
Um Daten zu empfangen kann u. a. die "read()" Funktion genutzt werden.
```py
ser.read(n)
```
n gibt die Anzahl an Bytes an, welche gelesen werden sollen.

### Infos und Dokumentation
Mehr Informationen über die Library und den hier genannten Funktionen können in der offiziellen Dokumentation gefunden werden.
https://pyserial.readthedocs.io/en/latest/pyserial_api.html

## Erklärung der Umsetzung

### Imports
```py
import serial
import serial.tools.list_ports
```
list_ports wird später verwendet, um die verfügbaren COM_Ports auszulesen.

### Robot Klasse
Beim objektorientierten Programmieren werden Klassen erstellt, um Funktionen zu gruppieren, Zugriffsrechte auf sensible Daten zu begrenzen, und vor allem, um die Nutzung zu vereinfachen (Komplexität abbauen).

In unserem Fall, dient die Klasse zum Herstellen einer Verbindung zum COM-Port, Senden von Daten, und zum Auslesen verfügbarer COM-Ports.

```py
class Robot:
    def connect(self, port):
        try:
            self.__ser = serial.Serial(port=port, baudrate=9600, stopbits=serial.STOPBITS_TWO, parity=serial.PARITY_EVEN)
            return 0
        except:
            return 1
```

Die `connect()` Funktion versucht den gegebenen Port zu öffnen, und gibt zurück, ob der Vorgang erfolgreich war (0) oder nicht (1).
```py
    def send(self, str):
        if self.__ser:
            self.__ser.open()
            self.__ser.write(str)
            self.__ser.close()
            return 0
        else:
            return 1
```

Die `send()` Funktion versucht über die Schnittstelle Daten zu senden. Dazu öffnet es den COM-Port, sendet die Daten, und schließt den Port anschließend wieder. Auch hier wird der Erfolg des Vorgangs als 0 (Erfolg) und 1 (Fehler) zurückgegeben.
```py
    def getPorts(self):
        ports = ['COM%s' % (i + 1) for i in range(256)]
        result = []
        for port in ports:
            try:
                s = serial.Serial(port)
                s.close()
                result.append(port)
            except (OSError, serial.SerialException):
                pass
        return result
```

Die Funktion `getPorts()` gibt alle offenen COM-Ports zurück. In Windows ist die maximale Anzahl an COM-Ports 256. Mit der `Serial()` Funktion werden alle 256 getestet, und falls eine Verbindung möglich ist, in einer Liste gespeichert.

# Modul
Um den oben beschriebenen Code zu nutzen, kann er in neuen Projekten als Modul importiert werden.
```py
import test
```
Eine Instanz der Robot Klasse wird erstellt.
```py
roboter = test.Robot()
```
Nun können die Funktionen der Klasse genutzt werden.
```py
print(roboter.getPorts())
roboter.connect("COM4")
roboter.send("Hallo Roboter")
```
