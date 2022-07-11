# Quickstart - Meteorologische modeldata

## Data retrieval
### ECMWF
Data komt van het European Centre for Medium Range Weather Forecasting (ECMWF).
Hindcasts (met observaties meegeassembleerd) worden gepubliceerd op Copernicus ([https://cds.climate.copernicus.eu/cdsapp#!/dataset/reanalysis-era5-single-levels?tab=overview](Surface) of [https://cds.climate.copernicus.eu/cdsapp#!/dataset/reanalysis-era5-pressure-levels?tab=overview](Hoogte)).
Voor Forecasts is een licentie nodig van het KNMI. 

Data staat aan de Copernicus-zijde in GRIB-format voor de hele wereld, op tape-cassetes. Je kunt echter aangeven dat voor een kleinere regio of in netCDF-format wil. Doe dat niet.
Deze bewerkingen uitvoeren aan de Copernicus-zijde kost het ECMWF rekencapaciteit, waardoor je een lagere prioriteit in de wachtrij krijgt. Je bent sneller klaar door eerst de data te downloaden en deze operaties lokaal te doen.

Er is bij de CDS onvoldoende capaciteit om alle downloads gelijktijdig te verwerken. Je staat daardoor al snel 75 min in de wachtlijst. Als je grotere datasets download of meerdere requests achter elkaar, daalt je prioriteit, en sta je langer te wachten (tot 7 dagen).

### KNMI
Het KNMI stelt hun forecast ook beschikbaar als GRIB: [https://dataplatform.knmi.nl/dataset/harmonie-arome-cy40-p1-0-2](Surface) en [https://dataplatform.knmi.nl/dataset/harmonie-arome-cy40-p3-0-2](Pressure levels)

Het Harmonie-model van het KNMI gaat echter maar 48uur vooruit, en is beperkt tot (Europees) Nederland. Het Harmoniemodel bouwd voort op het ECMWF model, het gebruikt de forecast van het ECMWF als input, om voor de regio Nederland een hogere resolutie te geven.

### NOAA GFS
Het Amerikaanse model is vrij beschikbaar, en is het enige wereldwijde model dat dat is. Het heeft echter een grovere resolutie dan ECMWF, en daardoor een slechtere verwachtingswaarde. Als het GFS en ECMWF niet met elkaar overeenkomen varen experts vrijwel altijd op ECMWF
Downloaden kan [https://www.nco.ncep.noaa.gov/pmb/products/gfs/](hier).

## Verwerken van GRIB
GRIdded Binary is een antiek format voor 2D data. Meerdere arrays worden achter elkaar opgeslagen, voor hoogte/tijdstappen/parameters.
GRIBs bevatten (om data te besparen) weinig meta-data. Zo wordt niet opgeslagen welke parameter welke is, en ook standaardisatie is ver te zoeken.
Het KNMI slaat temperatuur op onder parameter 11, het ECMWF onder parameter 130. Je hebt dus (software met) de juiste vertaaltabel nodig.

Een van de eerste stappen in het verwerken van GRIB-files is dan ook meestal omzetten naar netCDF. NetCDF is een modern format (een specificatie van HDF5), waarin ook metadata kan worden opgeslagen. De namen van de paramters, eenheden e.d. worden dan ook in deze stap toegevoegd.

Als je met de GRIBfiles zelf aan de slag wil, raad ik de volgende twee tools aan:
*	`eccodes`/`grib_to_netcdf` om ecmwf-files naar NetCDF te converteren. [https://confluence.ecmwf.int/display/ECC](https://confluence.ecmwf.int/display/ECC) 
*	`cdo`, om ingewikkeldere dingen met GRIBfiles te doen [https://code.mpimet.mpg.de/projects/cdo/](https://code.mpimet.mpg.de/projects/cdo/)
Bij beide tools raad ik aan om Linux en python3 te gebruiken. Windows wordt niet ondersteund, al hoewel mijn ervaringen met Windows Subsystem for Linux (WSL2) redelijk goed zijn.

### Instaleren en gebruik eccodes en cdo in Anaconda/Miniconda
Je kunt natuurlijk ook proberen dit in je eigen hoofd-environment te instaleren, maar deze packages gaan niet goed samen met andere. Om dat grote risico op Dependency Hell te voorkomen, instaleer je ze beter in een eigen environment.

#### eccodes met ECMWF-data
```(bash)
conda create -n ecmwf eccodes -c conda-forge
conda activate eccodes
pip install eccodes
grib_to_netcdf -o output.nc download.grib
conda deactivate
```
In dit ecmwf environment kun je ook `cdsapi` instaleren (denk ik), om de data automatisch/scriptmatig van Copernicus af te halen

#### cdo met KNMI-data
```(bash)
tar xvf harm40_v1_p1_2022070900.tar --one-top-level
cd harm40_v1_p1_2022070900
conda create -n grib_cdo cdo python-cdo -c conda-forge
conda activate grib_cdo
# Verplaats het eerste bestand in een KNMI dataset. Die bevat een aantal extra initiatie model-parameters, waardoor deze tijdstap meer parameter heeft. CDO kan daar niet goed mee omgaan.
mv HA40_N25_202207090000_00000_GB HA40_N25_202207090000_00000_GBskip
cdo -t knmi.partab -f nc4 cat *_GB nc_HA40_202207090000.nc4
conda deactivate
```
KNMI.partab kun je downloaden van mijn [https://gist.github.com/Papagaai35/938972d9b113ae2191cded7f8d882466](Gist.github.com)

## Verwerken van netCDF
Het NetCDF format bouwt voort op het HDF5 format, en bevat wel alle metadata die je nodig hebt. In python valt er goed mee te werken met het package `xarray`. (Een soort pandas voor veeldimensionale numpy-arrays en metadata; [https://docs.xarray.dev/en/stable/](https://docs.xarray.dev/en/stable/)). Dat package werkt naadloos met netcdf, en maakt het makkelijk om array’s uit te lezen en weg te schrijven.

## Meteorologische berekeningen - MetPy
Om sommige veelgebruikte meteofuncties te gebruiken (relatieve -> absolute luchtvochtigheid e.d.), kun je MetPy overwegen [https://unidata.github.io/MetPy/latest/index.html](https://unidata.github.io/MetPy/latest/index.html), maar dat package werkt weer met `pint` samen om eenheden consistent te houden. Die samenwerking zorgt (naar mijn idee) voor een onprettige gebruikservaring. De onderliggende numpy-arrays worden verder ingecapseld in een xarray-pint-numpy constructie, waardoor er slecht mee te werken valt. Als ik metPy gebruikt pluk ik de thermodynamische functies uit hun sourcecode, of haal pint er weer zo snel mogelijk tussenuit.

## Plotten/weergeven
* `matplotlib`
* `cartopy` (voor projecties)
* Natural Earth (voor kaartachtergronden en kustlijnen)
* `cmocean` (voor mooie kleurtjes).
Meestal veel gevulde contouren (`contourf`), voor wat vloeiende plaatjes, maar `imshow` werkt ook prima (voor raster).
