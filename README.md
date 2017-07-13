# ITSol Tools

Framework ITSOL Tools

## Cargar la libreria

Para iniciar un script con el framework deberá cargar la libreria con la siguiente línea
`use libs::itsol::tools;`

Ejemplo

#### my_script.pl

```perl
#!/usr/bin/perl
use strict;
use libs::itsol::tools;
```

## Cargar a un Objeto

una vez cargado el framework se deberá instanciar a una variable el 
framework como aparece en la siguiente linea

`my $utls = libs::itsol::tools->new();`

Ejemplo

#### my_script.pl
```perl
#!/usr/bin/perl
use strict;
use libs::itsol::tools;

my $utls = libs::itsol::tools->new();

```

## Variables Globales

si en el script necesitamos algunas variables globales podriamos 
crear un archivo yml para usuarlo de configuracion

Ejemplo

#### my_script.yml
```yaml
config:
  debug: 0
  with_db: 0
  sqlite_dbfilename: 
config_log4perl:
  logs: 0
  level: INFO
  patternLayout: [%d][%p][%F{1}] > %m %n
config_some_vars:
  my_script_var1: foo
  my_script_var2: baz
```
y se carga al script de la siguiente manera

#### my_script.pl
```perl
#!/usr/bin/perl
use strict;
use libs::itsol::tools; # se carga la libreria del framwork
use libs::Data::Dump qw(dump); # Libriria para volcar
#se instancia el framework
my $utls = libs::itsol::tools->new(yml_file => "my_script.yml");

#se imprime el volcado de las variables del objeto $utls
print dump($utls);

```

El resultado nos puede arrojar algunas variables interesantes como

```
bless({
  # tied libs::Tie::SecureHash
  "libs::itsol::tools::dbh"               => undef,                     # Variables asignada para majerar los conectores de base de datos
  "libs::itsol::tools::debug"             => 0,                         # Variable Utilizada para hacer debug en la libreria para uso interno de Mr Satan
  "libs::itsol::tools::level"             => "INFO",                    # Log Level
  "libs::itsol::tools::log"               => undef,                     # Objeto Manejador para Log4perl
  "libs::itsol::tools::logs"              => 0,                         # Varible para depurar el script
  "libs::itsol::tools::my_script_var1"    => "foo",                     # Variable definida para usar en script
  "libs::itsol::tools::my_script_var2"    => "baz",                     # Variable definida para usar en script
  "libs::itsol::tools::patternLayout"     => "[%d][%p][%F{1}] > %m %n", # Patron para Log4perl
  "libs::itsol::tools::script"            => "test.pl",                 # Nombre de script
  "libs::itsol::tools::script_log"        => "test.pl.log",             # Nombre de log del script si se pone como parametro logs: 1 en archivo YML
  "libs::itsol::tools::source"            => "itsol::tools-0.24",       # version fuente del framework
  "libs::itsol::tools::sqlite_dbfilename" => undef,                     # manejador de credenciales si se configurar deberá llevar archivo .db creado por create_db
  "libs::itsol::tools::ua_agent"          => "itsol::tools/0.24",       # Version de LWP Agent client
  "libs::itsol::tools::ua_cookie_file"    => ".cookie_itsol.txt",       # Archivo de cookie
  "libs::itsol::tools::ua_timeout"        => 4,                         # Timeout para LWP
  "libs::itsol::tools::with_db"           => 0,                         # Si es parametro es 1 debera configurar el parametro sqlite_dbfilename con una base
}, "libs::itsol::tools")
```

## Funciones para base de datos

Configuracion

```
create_db -f my_script.db
insert_db -f my_script.db -c oracle -u MiUsuario -p MiPassword -r MiPassphrase -H hostname_or_IP -P 1521 
```

#### my_script.pl

```perl
#!/usr/bin/perl
use strict;
use libs::itsol::tools; # se carga la libreria del framwork
use libs::Data::Dump qw(dump); # Libriria para volcar
#se instancia el framework
my $utls = libs::itsol::tools->new(yml_file => "my_script.yml");
my $bar; #local var scope Global


my $dbh = $utls->get_connection("oracle");
my $stm = q{
  SELECT
    sysdate as FECHA
  FROM
    DUAL
};
my $sth = $dbh->prepare($stm);
$sth->execute();
while (my $row = $sth->fetchrow_hashref){
  print "Mi Fecha: ".$row->{FECHA}."\n";
}
$sth->finish();
$dbh->disconnect();

# Usando las Variables Globales

print "Booo\n" if $utls->{my_script_var1} eq "foo";
print "my_script_var2 value = ".$utls->{my_script_var2}."\n";
print $utls->{my_script_var1}." bar ".$utls->{my_script_var2}."\n" unless defined($bar);

```

## Usando Log4perl

```perl
$utls->{log}->debug("My Debug"); # no lo imprime porque es nivel es  debug (actual es Level es INFO)
$utls->{log}->info("My Info");   # lo imprime
$utls->{log}->warn("My Warn");   # lo imprime
$utls->{log}->error("My Error"); # lo imprime
$utls->{log}->fatal("My Fatal"); # lo imprime
```

## Usando contador Hexadecimal

```perl
my $hex = "00";

for (my $i=0;$i<=100;$i++){
  $hex = $utls->_gen_mac($hex);
  print $hex."\n";
}

#Result 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F ...
```

## Usando Fechas Parametros D + numero de dias, H + numeros de horas, I + numero minutos, S + numero de segundos

```perl
my ($sec,$min,$hour,$mday,$mon,$year) = localtime();
$mon += 1;
$year += 1900;
$mon = sprintf("%02d",$mon);
$mday = sprintf("%02d",$mday);
$hour = sprintf("%02d",$hour);
$min = sprintf("%02d",$min);
$sec = sprintf("%02d",$sec);

print "Fecha Actual: ".$mday."-".$mon."-".$year. " Hora: ".$hour.":".$min.":".$sec."\n";

my ($sec_i,$min_i,$hour_i,$mday_i,$mon_i,$year_i) = $utls->increment_date("D1",$sec,$min,$hour,$mday,$mon,$year);
$year_i += 1900;
$mday_i = sprintf("%02d",$mday_i);
$mon_i = sprintf("%02d",$mon_i);
$hour_i = sprintf("%02d",$hour_i);
$min_i = sprintf("%02d",$min_i);
$sec_i = sprintf("%02d",$sec_i);

print "Fecha +1 day : ".$mday_i."-".$mon_i."-".$year_i." Hora: ".$hour_i.":".$min_i.":".$sec_i."\n";

my ($sec_d,$min_d,$hour_d,$mday_d,$mon_d,$year_d) = $utls->decrement_date("D1",$sec,$min,$hour,$mday,$mon,$year);
$year_d += 1900;
$mday_d = sprintf("%02d",$mday_d);
$mon_d = sprintf("%02d",$mon_d);
$hour_d = sprintf("%02d",$hour_d);
$min_d = sprintf("%02d",$min_d);
$sec_d = sprintf("%02d",$sec_d);

print "Fecha -10 hours : ".$mday_d."-".$mon_d."-".$year_d." Hora: ".$hour_d.":".$min_d.":".$sec_d."\n";

```

# AUTHOR

Ismael Loera Gomez I.L.G. aka Le Chef Herbe Verte, El Hefe, Mr Satan <thelittlemenace@hotmail.com>

My Geek Code =) 

```
-----BEGIN GEEK CODE BLOCK-----
Version: 3.1
GCS/IT/L/SS d(-) s+:+ a+ C++ UB P+++ L E W+ N+ o- K- w O- M V- PS+++ PE Y--
PGP- t+ 5 X R tv+ b+++ DI- D G++ e++ h r+++ y+
------END GEEK CODE BLOCK------
```

COPYRIGHT Copyright (c) 2017 I.L.G.

This program is free software; you can redistribute it and/or modify it
under the same terms as Perl itself.

Script with all examples

```perl

#!/usr/bin/perl
use strict;
use libs::itsol::tools; # se carga la libreria del framwork
use libs::Data::Dump qw(dump); # Libriria para volcar
#se instancia el framework
my $utls = libs::itsol::tools->new(yml_file => "my_script.yml");
my $bar; #local var scope Global


my $dbh = $utls->get_connection("oracle");
my $stm = q{
  SELECT
    sysdate as FECHA
  FROM
    DUAL
};
my $sth = $dbh->prepare($stm);
$sth->execute();
while (my $row = $sth->fetchrow_hashref){
  print "Mi Fecha: ".$row->{FECHA}."\n";
}
$sth->finish();
$dbh->disconnect();


#Utilizando las variables globales
print "Booo\n" if $utls->{my_script_var1} eq "foo";

print "my_script_var2 value = ".$utls->{my_script_var2}."\n";

print $utls->{my_script_var1}." bar ".$utls->{my_script_var2}."\n" unless defined($bar);

#usar Log4perl
#
$utls->{log}->debug("My Debug"); # no lo imprime porque es nivel es  debug (actual es Level es INFO)
$utls->{log}->info("My Info");   # lo imprime
$utls->{log}->warn("My Warn");   # lo imprime
$utls->{log}->error("My Error"); # lo imprime
$utls->{log}->fatal("My Fatal"); # lo imprime

#Usando contador Hexadecimal

my $hex = "00";
#
for (my $i=0;$i<=100;$i++){
  $hex = $utls->_gen_mac($hex);
  print $hex."\n";
}
#Result 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F ...


#Usando Fechas Parametros D + numero de dias, H + numeros de horas, I + numero minutos, S + numero de segundos
my ($sec,$min,$hour,$mday,$mon,$year) = localtime();
$mon += 1;
$year += 1900;
$mon = sprintf("%02d",$mon);
$mday = sprintf("%02d",$mday);
$hour = sprintf("%02d",$hour);
$min = sprintf("%02d",$min);
$sec = sprintf("%02d",$sec);
print ;

print "Fecha Actual: ".$mday."-".$mon."-".$year. " Hora: ".$hour.":".$min.":".$sec."\n";

my ($sec_i,$min_i,$hour_i,$mday_i,$mon_i,$year_i) = $utls->increment_date("D1",$sec,$min,$hour,$mday,$mon,$year);
$year_i += 1900;
$mday_i = sprintf("%02d",$mday_i);
$mon_i = sprintf("%02d",$mon_i);
$hour_i = sprintf("%02d",$hour_i);
$min_i = sprintf("%02d",$min_i);
$sec_i = sprintf("%02d",$sec_i);

print "Fecha +1 day : ".$mday_i."-".$mon_i."-".$year_i." Hora: ".$hour_i.":".$min_i.":".$sec_i."\n";

my ($sec_d,$min_d,$hour_d,$mday_d,$mon_d,$year_d) = $utls->decrement_date("D1",$sec,$min,$hour,$mday,$mon,$year);
$year_d += 1900;
$mday_d = sprintf("%02d",$mday_d);
$mon_d = sprintf("%02d",$mon_d);
$hour_d = sprintf("%02d",$hour_d);
$min_d = sprintf("%02d",$min_d);
$sec_d = sprintf("%02d",$sec_d);

print "Fecha -10 hours : ".$mday_d."-".$mon_d."-".$year_d." Hora: ".$hour_d.":".$min_d.":".$sec_d."\n";

 ```
