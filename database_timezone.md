# Zeitzone in der Datenbank ändern

## Problem

Angefangen hats das wir einen Shop betreuen, dessen mySQL-Serverzeit mit UTC läuft. Da now() somit "falsche" Ergebnisse liefert
und die Bestellzeiten (und andere Zeiten) im Shop "falsch" sind, hab ich angefangen zu graben.

## Lösungsansatz: Plugin

Kann an dieser Stelle nicht funktionieren. Die Zeitzone muss direkt nach dem Datenbank-Connect erstellt werden.
Die Pluginhooks werden aber erst "ewig" nach dem Connect geholt. Heisst, wenn der Hook ziehen würde, ist der Drops schon lang
geluscht.

## Lösungsansatz: Patch

Scheint mir die einfachste und beste Lösung für diesen Fall zu sein. Ich hatte ja die Coredateien zu editieren, aber
manchmal geht es halt nicht anders. Dafür funktioniert diese Änderung wenigstens mit allen mySQL5-Servern.

### Anmerkungen

Der Wert der Zeitzone kann in 4 Formen in der Konfig eingetragen werden:

* Gar nicht oder Leerstring: Die Zeitzone wird nicht verändert
* "SYSTEM": Die Systemzeit des Servers wird verwendet
* "-1:00" pder "+5:00": Die betreffende Zeit wird auf die aktuelle Zeitzone drauf gerechnet bzw. abgezogen
* "Europe/Berlin": Die Zeitzone wird fix mit ihrem Namen angegeben

Nach dem Ändern der Konfiguration natürlich nicht vergessen den Cache zu leeren!

Weitere Infos dazu im ersten Link am Ende des Dokuments.
Sollte die Angabe via fixer Zeitzone fehlschlagen, lese man auch noch den zweiten Link.

### Codeänderungen

*	In ./engine/Shopware/Configs/Default.php (oder in der ./config.php):

	Suchen nach:

		'adapter' => 'pdo_mysql'
		
	Nach neue Konfigzeile hinzufügen (und Komma in der vorherigen Zeile nicht vergessen!):
		
		'timezone' => ''
		
*	In ./engine/Library/Enlight/Components/Db/Adapter/Pdo/Mysql.php

	Suchen nach:
	
		protected function _connect()
		
	In der Methode nach dem Try-Catch-Block folgende Zeilen hinzufügen:
	
		if (!empty($this->_config["timezone"])) {
			$this->exec("SET time_zone = ".$this->quote($this->_config["timezone"]));
		}


### Tests mit dem Patch

$this->_config["timezone"] = "";

	array
	  '@@global.time_zone' => string 'SYSTEM'
	  '@@session.time_zone' => string 'SYSTEM'
	  'now()' => string '2015-03-20 17:27:49'  

$this->_config["timezone"] = "+10:00";

	array
	  '@@global.time_zone' => string 'SYSTEM'
	  '@@session.time_zone' => string '+10:00'
	  'now()' => string '2015-03-21 02:32:35'

$this->_config["timezone"] = "Europe/Berlin";

	array (size=3)
      '@@global.time_zone' => string 'SYSTEM'
      '@@session.time_zone' => string 'Europe/Berlin'
      'now()' => string '2015-03-20 17:38:24'

## Links
* [http://dev.mysql.com/doc/refman/5.5/en/time-zone-support.html](http://dev.mysql.com/doc/refman/5.5/en/time-zone-support.html)
* [http://stackoverflow.com/questions/5510052/changing-the-connection-timezone-in-mysql](http://stackoverflow.com/questions/5510052/changing-the-connection-timezone-in-mysql)