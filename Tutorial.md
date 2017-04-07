# 
`r Sys.Date()`  




# Introduction



This is the documentation for the R package **geoSpectral** v.0.17


**geoSpectral** is an R package providing a new S4 class for R that stores spectral, temporal and spatial attributes of measurement data.  Spectral data is usually obtained through radiometric measurements of the electromagnetic energy at various wavelengths. The package also offers methods for accessing and manipulating the spectral (and non-spectral) data. **geoSpectral** was initially created to answer needs encountered in bio-optical oceanography, remote sensing, environmental and earth sciences. However anyone using spatial/temporal/spectral data of any kind can possibly make use of tools and methods provided by the package. The following S4 classes:

* [Spectra](#the-spectra-class) class (stores spatial/temporal/spectral aspects of data)
* [SpcHeader](#the-spcheader-class) class (stores metadata in an R list object)
* [SpcList](#the-spclist-class) class (makes a collection of Spectra objects in an R list)

and basic data access and manipulation methods for :

+ [Importing](#importing-data)
+ [Accessing and Subsetting](#accessing-data) Data
+ Converting to/from R data.frames
+ Analyzing
+ [Plotting](#plotting)
+ Exporting

scientific data are provided. Once spectral data is imported into a *geoSpectral* object, the  statistical and data processing power of R is available for various kinds of scientific analyses. 

**geoSpectral** was build on top of another R package called [spacetime](http://cran.r-project.org/web/packages/spacetime/index.html), which, itself, was built on the packages *rgdal* (built on *sp*) and *xts* (built on *zoo*). *rgdal* and *sp* provide spatial attributes and methods. *xts* and *zoo* provide temporal attributes and methods time series analysis methods. *spacetime* facilitates spatio-temporal characterization of data, standardizing the way space and time attributes are stored in the variable. **geoSpectral** adds the spectral attributes and a simple metadata structure on top of the *STIDF* object provided by *spacetime*.

####License
The package is issued with a [GPLv3](http://www.gnu.org/copyleft/gpl.html) license. Please consult the license documentation if you would like to use **geoSpectral** in your software projects.

# Obtaining and Installing


#### Stable Version
**geoSpectral** depends on several other R packages. These packages need to be installed on your R system before you can install it. The easiest way is to install the stable [CRAN](https://cran.r-project.org/web/packages/geoSpectral/index.html) version using :

```r
install.packages("geoSpectral")
```

The above command will automatically download and install all dependencies as well as **geoSpectral** itself. 

#### Development Version
We develop and host **geoSpectral** at [Github](https://github.com/PranaGeo/geoSpectral). The *dev* branch contains the latest changes and newest features. There are two ways of getting the source code and installing it under your R installation :

###### Installing from within R
You can first install the **devtools** package using the command :

```r
install.packages("devtools")
```

If you have unresolved dependencies to other packages, you can always try to add the parameter `dependencies=TRUE` into `install.packages()`:

```r
install.packages("devtools", dependencies = TRUE)
```

Once **devtools** package is installed, you can clone and install the *dev* branch of  **geoSpectral** using  :

```r
require(devtools)
install_github("PranaGeo/geoSpectral", ref = "dev", dependencies = TRUE)
```

Note that we had problems on our Windows machine with *devtools::install_github()* correctly building the package dependency list. 

##### Installing source package from system command line
If you have *git* installed on your system, you can obtain the source code either by cloning the *git* repository from the command line of your operating system :
```
git clone -b dev https://github.com/PranaGeo/geoSpectral.git
```
or using another (GUI-based) *git* client. This will download the latest *master* branch of theThen, in your shell command prompt, try

```
R CMD INSTALL geoSpectral
```

where *geoSpectral* is the name of the folder the package source code has been downloaded by git. The above command installs the package for the current user only. If you want to install the package globally in a multi-user OS, you have to do the installation with super-user privileges. On a debian-based Linux machine, this can be achieved by prepending the command `sudo` into the previous command. For more information on how to install R packages, consult the official [R Installation and Administration](http://cran.r-project.org/doc/manuals/r-release/R-admin.html#Installing-packages) documentation.

# Provided Classes

## The Spectra class
*Spectra* is the main S4 class provided by the package **geoSpectral**. It contains slots (members) that store the name of the spectral variable (**\@ShortName** and **\@LongName**), wavelengths of spectral data (**\@Wavelengths**), and units of wavelengths (**\@WavelengthsUnit**), spectral data (**\@Spectra**), non-spectral data (**\@data**), non-spectral data units (**\@Units**) metadata (**\@header**), spatial attributes (**\@sp**), temporal attributes (**\@time** and **\@endtime**) and class version (**\@ClassVersion**).

Let us examine the slots and their respective data types: 

```r
library("geoSpectral")
showClass("Spectra")
```

```
Class "Spectra" [package "geoSpectral"]

Slots:
                                                                      
Name:        ShortName        LongName     Wavelengths WavelengthsUnit
Class:       character       character         numeric       character
                                                                      
Name:          Spectra          header           Units        UnitsAnc
Class:          matrix       SpcHeader       character       character
                                                                      
Name:     ShortNameAnc     LongNameAnc      InvalidIdx     SelectedIdx
Class:       character       character         logical         logical
                                                                      
Name:     ClassVersion            data              sp            time
Class:         numeric      data.frame         Spatial             xts
                      
Name:          endTime
Class:         POSIXct

Extends: 
Class "STIDF", directly
Class "STI", by class "STIDF", distance 2
Class "ST", by class "STIDF", distance 3
```
If you have a dataset that is non-spectral, you could use the *ST*, *STI* and *STIDF* classes provided by *spacetime* package that provides the slot **\@data**. This slot is optional in **geoSpectral**, but it makes it possible to work with non-spectral (ancillary) data as well.

## The SpcHeader class
All *Spectra* objects needs to contain a slot called **\@header**, an object of class **SpcHeader**. This class directly inherits the R *list* class. 

```r
showClass("SpcHeader")
```

```
Class "SpcHeader" [package "geoSpectral"]

Slots:
            
Name:  .Data
Class:  list

Extends: 
Class "list", from data part
Class "vector", by class "list", distance 2
```

All the regular methods that can be applied on an R *list* object can be applied to *SpcHeader* objects as well :

```r
h <- new("SpcHeader")
h
```

```
Station  :  NA 
Cruise  :  NA 
Latitude  :  NA 
Longitude  :  NA 
```

```r
h$Station <- "21A"
h$Station
```

```
[1] "21A"
```

## The SpcList class
The *SpcList* is designed to store collections of several related **Spectra** objects together. Just like the *SpcHeader* class, the *SpcList* class as well inherits from R *list*:

# Importing and Subsetting Data


## Creating Spectra Objects
### Using the Constructor Function
This section describes stages of creation of a Spectra object from data located in a csv file. The file is distributed with **geoSpectral** and contains spectral data, a vertical profile of the particulate absorption coefficient collected at sea at discrete depth in 4 different stations. 

The first step is to create a *data.frame* object using the R function *read.table()*. We have a comma-delimited text file with several measured parameters in columns and observations from different bottles (stations and depths). 


```r
fnm = file.path(base::system.file(package = "geoSpectral"), "test_data", "particulate_absorption.csv.gz")
abs = read.table(fnm, sep = ",", header = T)
abs$STATION = factor(abs$STATION)
abs[1:2, 1:17]  #Display only the first 2 rows and first 17 columns if the data frame
```

```
      CRUISE STATION                    TIME CAST METHOD NISKIN REPLICATE
1 MALINA2009     394 2009-08-03 22:29:00 UTC   38      N      2         2
2 MALINA2009     280 2009-08-04 15:15:00 UTC   40      N      5         2
       LAT      LONG   PRES        A      Snap    Offset      X300
1 69.84703 -133.4959  2.245 12.74161 0.0086613 0.0449174 0.9226343
2 70.86902 -130.5137 23.163  1.72918 0.0094426 0.0049271 0.1050914
       X301      X302      X303
1 0.9147542 0.9092862 0.9035436
2 0.1030439 0.1020884 0.1009091
```
   
Ancillary (non-spectral) and spectral data are stored in columns 1:13 and 14:514 respectively. Wavelengths are extracted from column names of spectral data. The string variable *Units* stores the units of the spectral variable. The data columns were then arranged so that ancillary data are located at the right of spectral data columns. 


```r
lbd = as.numeric(gsub("X", "", colnames(abs)[14:514]))
Units = "1/m"
colnames(abs) = gsub("X", paste("anap", "_", sep = ""), colnames(abs))
colnames(abs) = gsub("PRES", "DEPTH", colnames(abs))
abs = abs[, c(14:514, 1:13)]
```

Note that we need to rename the depth column (*PRES*) so that it is called *DEPTH*. The *TIME* column needs to be an R *POSIXct* object with the right timezone information. Type `help(POSIXct)` at the R command prompt to get more information about this class.


```r
tz <- strsplit(as.character(abs$TIME), " ")[[1]][[3]]  #Extract the timezone
abs$TIME = as.POSIXct(as.character(abs$TIME), tz = tz)
```

The *Spectra* object is created with the constructor function *Spectra()*.

```r
myS <- Spectra(abs, Wavelengths = lbd, Units = Units, ShortName = "a_nap")
myS
```

```

 a_nap  : An object of class "Spectra"
 501 spectral channels in columns and 26 observations in rows 
 LongName:  spvar2 longname 	 Units:  1/m 
 Wavelengths :  501 channels with units of nm [ 300 , 800 ] -> 300 301 302 303 304 305  ...
 Spectra Columns:  anap_300 anap_301 anap_302 anap_303 anap_304 anap_305 ...
 Ancillary Columns:  idx CRUISE STATION CAST METHOD NISKIN ...
 Bounding box: LON( -133.4959 -130.5083 ) LAT( 69.84703 72.05774 )
 Time : periodicity of 0 secs between (2009-08-03 22:29:00 - 2009-08-05 16:23:00), tz=UTC 
```
The function Spectra() calls `spacetime::stConstruct()` that creates a `STIDF` object from an input *data.frame* object of long-table format. Type `?spacetime::stConstruct` at the R prompt for more information long and wide tables. *geoSpectral* accepts the following column names when searching latitude and longitude information : `"Latitude"`, `"LATITUDE"`, `"lat"`, `"LAT"`, `"Longitude",`,`"LONGITUDE"`, `"lon"`, `"LON"`,`"long"`, `"LONG"`. If no suitable column is found, *LAT* and *LONG* takes the artificial value of 1. As mentioned above, Time needs to be stored in column `"TIME"`. If no such a column is found, an artificial vector sequence was created from 1 to the number of rows.  According to the way spacetime is built,  if the measurements have been performed over a time interval (instead of instantaneous measurements), the length of intervals can be specified in the column `"ENDTIME"`. If no such column is found, the data is assumed to be acquired in a time instance (`ENDTIME=TIME`).

Ancillary data is stored in the slot **\@data** while the spectral data, in slot **\@Spectra**. A spectra object could also be created with or without ancillary data. Since we will later need Ancillary data, it is best to conserve it in the case of this example. 

## Inquiring About Spectra Objects
It is possible to inquire about the Spectra object using utility functions `dim()`, `ncol()`, `nrow()`, `names()` and `spc.colnames()`. The latter function returns the column names of spectral data, omitting those of ancillary data.

```r
dim(myS)
```

```
[1]  26 501
```

```r
ncol(myS)
```

```
[1] 501
```

```r
nrow(myS)
```

```
[1] 26
```

```r
tail(names(myS))
```

```
[1] "NISKIN"    "REPLICATE" "DEPTH"     "A"         "Snap"      "Offset"   
```

```r
tail(spc.colnames(myS))
```

```
[1] "anap_795" "anap_796" "anap_797" "anap_798" "anap_799" "anap_800"
```

# Accessing Data



##Getting and setting Wavelengths

To extract (get) and to change (set) wavelengths, use *spc.getwavelengths* and *spc.setwavelengths* :

```r
range(spc.getwavelengths(myS))
```

```
[1] 300 800
```

```r
lbd <- 302:802
spc.setwavelengths(myS) <- lbd
range(spc.getwavelengths(myS))
```

```
[1] 302 802
```

The length of the Wavelength slot should always to be equal to the number of channels stored in the object :

```r
length(lbd) == ncol(myS)
```

```
[1] TRUE
```

##Subsetting
One can extract only a portion of the **Spectra** object by subsetting with row/column/spectra idexes using the “[“ operator . This operator returns a **Spectra** object. The following command extracts rows 1 to 10 and spectral columns 30 to 50 (provided in integer format). Ancillary data is unchanged. 

```r
myS[1:10]
```

```

 a_nap  : An object of class "Spectra"
 501 spectral channels in columns and 10 observations in rows 
 LongName:  spvar2 longname 	 Units:  1/m 
 Wavelengths :  501 channels with units of nm [ 302 , 802 ] -> 302 303 304 305 306 307  ...
 Spectra Columns:  anap_300 anap_301 anap_302 anap_303 anap_304 anap_305 ...
 Ancillary Columns:  idx CRUISE STATION CAST METHOD NISKIN ...
 Bounding box: LON( -133.4959 -130.5083 ) LAT( 69.84703 71.26659 )
 Time : periodicity of 0 secs between (2009-08-03 22:29:00 - 2009-08-04 22:07:00), tz=UTC 
```

The “[“ operator also supports column names :

```r
myS[, "anap_400"]
```

```

 a_nap  : An object of class "Spectra"
 1 spectral channel in columns and 26 observations in rows 
 LongName:  spvar2 longname 	 Units:  1/m 
 Wavelengths :  1 channel with units of nm [ 402 , 402 ] -> 402  ...
 Spectra Columns:  anap_400 ...
 Ancillary Columns:  idx CRUISE STATION CAST METHOD NISKIN ...
 Bounding box: LON( -133.4959 -130.5083 ) LAT( 69.84703 72.05774 )
 Time : periodicity of 0 secs between (2009-08-03 22:29:00 - 2009-08-05 16:23:00), tz=UTC 
```

```r
myS[, c("anap_400", "anap_500")]
```

```

 a_nap  : An object of class "Spectra"
 2 spectral channels in columns and 26 observations in rows 
 LongName:  spvar2 longname 	 Units:  1/m 
 Wavelengths :  2 channels with units of nm [ 402 , 502 ] -> 402 502  ...
 Spectra Columns:  anap_400 anap_500 ...
 Ancillary Columns:  idx CRUISE STATION CAST METHOD NISKIN ...
 Bounding box: LON( -133.4959 -130.5083 ) LAT( 69.84703 72.05774 )
 Time : periodicity of 0 secs between (2009-08-03 22:29:00 - 2009-08-05 16:23:00), tz=UTC 
```

If only a set of wavelengths are desired, either the column indexes have to be proveded in integer format or the desired wavelengths have to be provided in numeric (floating) format :


```r
myS[1:10, 30:50]  #Selection of channels by column index
```

```

 a_nap  : An object of class "Spectra"
 21 spectral channels in columns and 10 observations in rows 
 LongName:  spvar2 longname 	 Units:  1/m 
 Wavelengths :  21 channels with units of nm [ 331 , 351 ] -> 331 332 333 334 335 336  ...
 Spectra Columns:  anap_329 anap_330 anap_331 anap_332 anap_333 anap_334 ...
 Ancillary Columns:  idx CRUISE STATION CAST METHOD NISKIN ...
 Bounding box: LON( -133.4959 -130.5083 ) LAT( 69.84703 71.26659 )
 Time : periodicity of 0 secs between (2009-08-03 22:29:00 - 2009-08-04 22:07:00), tz=UTC 
```

```r
lbd = as.numeric(c(412, 440, 490, 555, 670))
myS[1:10, lbd]  #Selection of channels by wavelength
```

```

 a_nap  : An object of class "Spectra"
 5 spectral channels in columns and 10 observations in rows 
 LongName:  spvar2 longname 	 Units:  1/m 
 Wavelengths :  5 channels with units of nm [ 412 , 670 ] -> 412 440 490 555 670  ...
 Spectra Columns:  anap_410 anap_438 anap_488 anap_553 anap_668 ...
 Ancillary Columns:  idx CRUISE STATION CAST METHOD NISKIN ...
 Bounding box: LON( -133.4959 -130.5083 ) LAT( 69.84703 71.26659 )
 Time : periodicity of 0 secs between (2009-08-03 22:29:00 - 2009-08-04 22:07:00), tz=UTC 
```

A subset of the spectra can also be extracted over a range of wavelengths using the following formulation :

```r
myS[1:10, "415::450"]
```

```

 a_nap  : An object of class "Spectra"
 36 spectral channels in columns and 10 observations in rows 
 LongName:  spvar2 longname 	 Units:  1/m 
 Wavelengths :  36 channels with units of nm [ 415 , 450 ] -> 415 416 417 418 419 420  ...
 Spectra Columns:  anap_413 anap_414 anap_415 anap_416 anap_417 anap_418 ...
 Ancillary Columns:  idx CRUISE STATION CAST METHOD NISKIN ...
 Bounding box: LON( -133.4959 -130.5083 ) LAT( 69.84703 71.26659 )
 Time : periodicity of 0 secs between (2009-08-03 22:29:00 - 2009-08-04 22:07:00), tz=UTC 
```

If numeric values are required instead of a *Spectra* object, use the “$" or the “[[" operators

```r
myS$CAST  #Returns Ancillary data
```

```
 [1] 38 40 40 41 41 41 41 41 44 44 44 44 44 44 51 51 51 51 51 51 51 51 51
[24] 51 52 52
```

```r
myS$anap_400  #Returns spectra as numeric vector
```

```
 [1] 0.4345009 0.0472191 0.0431448 0.0334085 0.0366025 0.0453854 0.0513998
 [8] 0.1312243 0.0618284 0.1265693 0.1137473 0.0079286 0.0355068 0.0114464
[15] 0.0114911 0.0086473 0.0060563 0.0034033 0.0047790 0.0049624 0.0069034
[22] 0.0051754 0.0037742 0.0014785 0.0078290 0.0105174
```

```r
head(myS[["anap_400"]])  #Returns spectra as numeric vector
```

```
[1] 0.4345009 0.0472191 0.0431448 0.0334085 0.0366025 0.0453854
```

```r
head(myS[[c("Snap", "Offset")]])  #Returns data.frame
```

```
       Snap    Offset
1 0.0086613 0.0449174
2 0.0094426 0.0049271
3 0.0082714 0.0054899
4 0.0089460 0.0050306
5 0.0085369 0.0046916
6 0.0084656 0.0050392
```

Subsetting can also be achieved using the implementation of the R function subset() for *Spectra* and *SpcList* classes. It is possible to perform a row-wise selection 

```r
subset(myS, DEPTH <= 30)  #Subsetting rows with respect to the value of Ancillary data
```

```

 a_nap  : An object of class "Spectra"
 501 spectral channels in columns and 15 observations in rows 
 LongName:  spvar2 longname 	 Units:  1/m 
 Wavelengths :  501 channels with units of nm [ 302 , 802 ] -> 302 303 304 305 306 307  ...
 Spectra Columns:  anap_300 anap_301 anap_302 anap_303 anap_304 anap_305 ...
 Ancillary Columns:  idx CRUISE STATION CAST METHOD NISKIN ...
 Bounding box: LON( -133.4959 -130.5083 ) LAT( 69.84703 72.05774 )
 Time : periodicity of 0 secs between (2009-08-03 22:29:00 - 2009-08-05 16:23:00), tz=UTC 
```

```r
subset(myS, DEPTH <= 30)$DEPTH
```

```
 [1]  2.245 23.163  2.149 25.965 16.889 10.741  4.878  2.297 18.826  2.078
[11] 29.090 18.758  8.762  2.015  1.956
```

```r
subset(myS, anap_440 <= 0.01)  #Subsetting rows with respect to the value of Spectral data
```

```

 a_nap  : An object of class "Spectra"
 501 spectral channels in columns and 14 observations in rows 
 LongName:  spvar2 longname 	 Units:  1/m 
 Wavelengths :  501 channels with units of nm [ 302 , 802 ] -> 302 303 304 305 306 307  ...
 Spectra Columns:  anap_300 anap_301 anap_302 anap_303 anap_304 anap_305 ...
 Ancillary Columns:  idx CRUISE STATION CAST METHOD NISKIN ...
 Bounding box: LON( -130.8965 -130.6084 ) LAT( 71.26659 72.05774 )
 Time : periodicity of 0 secs between (2009-08-04 22:07:00 - 2009-08-05 16:23:00), tz=UTC 
```

```r
subset(myS, subset = DEPTH <= 30, select = "CAST")  #Selecting Ancillary data columns, leaving Spectral columns intact
```

```

 a_nap  : An object of class "Spectra"
 501 spectral channels in columns and 15 observations in rows 
 LongName:  spvar2 longname 	 Units:  1/m 
 Wavelengths :  501 channels with units of nm [ 302 , 802 ] -> 302 303 304 305 306 307  ...
 Spectra Columns:  anap_300 anap_301 anap_302 anap_303 anap_304 anap_305 ...
 Ancillary Columns:  CAST ...
 Bounding box: LON( -133.4959 -130.5083 ) LAT( 69.84703 72.05774 )
 Time : periodicity of 0 secs between (2009-08-03 22:29:00 - 2009-08-05 16:23:00), tz=UTC 
```

```r
subset(myS, subset = DEPTH <= 30, select = "anap_440")  #Selecting Spectral data columns, leaving Ancillary columns intact
```

```

 a_nap  : An object of class "Spectra"
 1 spectral channel in columns and 15 observations in rows 
 LongName:  spvar2 longname 	 Units:  1/m 
 Wavelengths :  1 channel with units of nm [ 442 , 442 ] -> 442  ...
 Spectra Columns:  anap_440 ...
 Ancillary Columns:  idx CRUISE STATION CAST METHOD NISKIN ...
 Bounding box: LON( -133.4959 -130.5083 ) LAT( 69.84703 72.05774 )
 Time : periodicity of 0 secs between (2009-08-03 22:29:00 - 2009-08-05 16:23:00), tz=UTC 
```

To see the implementation of subset() for the *Spectra* class, try :

```r
showMethods(subset, classes = "Spectra", includeDefs = T)
```

##Selecting and flagging rows
It is possible to select some of the rows manually or with the help of the mouse. As of the writing of this document, only records (rows) of a *Spectra* object can be selected with the mouse. To manually set the first five observations as selected, use :

```r
idx = rep(FALSE, nrow(myS))
idx[1:5] = TRUE
spc.setselected.idx(myS) <- idx
```

Set the selected spectra rows as Invalid :

```r
spc.setinvalid.idx(myS) <- spc.getselected.idx(myS)
```

The function spc.select() needs the Spectra plot created by the spc.plot() method. First call spc.plot(), then use the left button of the mouse to select spectra. To end the selection process, click on an empty area or push the right mouse button : Row indexes of selected curves will display in the R command line. The function `spc.setselected.idx()` stores the indexes in the slot **SelectedIdx** of the variable.


```r
spc.plot(myS)
```

![](Tutorial_files/figure-html/unnamed-chunk-31-1.png)<!-- -->

```r
spc.setselected.idx(myS) <- spc.select(myS)
```

```r
spc.getselected.idx(myS)
```

```
 [1]  TRUE  TRUE  TRUE  TRUE  TRUE FALSE FALSE FALSE FALSE FALSE FALSE
[12] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
[23] FALSE FALSE FALSE FALSE
```

The display of selected and invalid rows is currently not implemented in plotting functions (ex. *spc.plot*, *spc.plot.depth*, *spc.plot.time*). The slot **SelectedIdx** will be reset if the dimensions of the *Spectra* object is changed by subsetting or another operation.

# Plotting




## Graphics using the traditional R way
To plot a Spectra object using the base R graphics, use spc.plot(). This method uses the R function matplot(). If there are too many observations (rows), plotting with matplot() can be very slow and may result in a figure cluttered with too many curves. To avoid such a disturbance such a case, use the argument *maxSp*. The following command will graph four spectra only, distributed evenly between the first and the last rows :



```r
par(mfrow = c(1, 2))
spc.plot(myS)
spc.plot(myS, maxSp = 4)
```

![](Tutorial_files/figure-html/unnamed-chunk-35-1.png)<!-- -->

```r
par(mfrow = c(1, 1))
```

### Depth Plots
If the dataset contains an ancillary data column named *DEPTH*, then plotting the Spectra object with respect to depth can be achieved by *spc.plot.depth* where the depth dimension is drawn on a reversed y axis:


```r
par(mfrow = c(1, 3))
spc.plot.depth(myS, "anap_300")
spc.plot.depth(myS, c("anap_300", "anap_500"))
spc.plot.depth(myS, c("anap_300", "Snap"))
```

![](Tutorial_files/figure-html/unnamed-chunk-36-1.png)<!-- -->

```r
par(mfrow = c(1, 1))
```

### Time Plots
Time plots consist of plotting a Spectra object with respect to observation number (channels with respect to row index) : 


```r
spc.plot.time(myS)
```

![](Tutorial_files/figure-html/unnamed-chunk-37-1.png)<!-- -->

The previous command draws a maximum number of 50 channels (default maxSp=50) among all spectral data. It is also possible to draw only 7 spectra (covering the whole spectral range).


```r
spc.plot.time(myS, maxSp = 7)
```

![](Tutorial_files/figure-html/unnamed-chunk-38-1.png)<!-- -->

### Grid Plots
A SpcList object containing several Spectra objects can be graphed in the same page in a grid structure. The function spc.plot.grid() calls spc.plot() for every grid element. The grid panels are named according to the Station header value.


```r
spc.plot.grid(BL, "spc.plot", nnrow = 2, nncol = 2)
```

![](Tutorial_files/figure-html/unnamed-chunk-39-1.png)<!-- -->

It is also possible to draw a grid of depth profiles if the dataset contains a column called *DEPTH*. The following command will plot depth profiles of 10 columns evenly distributed between the first and last columns. If  the function matches columns that have different units, a new x-axis will be drawn at the top for the additional unit. Only two units will are supported at this moment.


```r
spc.plot.grid(BL, "spc.plot.depth", nnrow = 2, nncol = 2)
```

![](Tutorial_files/figure-html/unnamed-chunk-40-1.png)<!-- -->

Additional arguments that can be used to format regular plotting functions of R can also be passed :
```
spc.plot.grid(BL,"spc.plot.depth", nnrow=2, nncol=2,ylim=c(60,0)) 
```

The function spc.plot.grid() will draw only a number of maxSp columns by default. It is possible to change this behavior by setting the input argument maxSp.


```r
spc.plot.grid(BL, "spc.plot.depth", nnrow = 2, nncol = 2, maxSp = 5)
```

![](Tutorial_files/figure-html/unnamed-chunk-41-1.png)<!-- -->

To plot only one column per panel, make sure the name (or the column index) of the variable is passed to spc.plot.depth() through spc.plot.grid().

```r
spc.plot.grid(BL, "spc.plot.depth", X = "anap_300", nnrow = 2, nncol = 2)
```

![](Tutorial_files/figure-html/unnamed-chunk-42-1.png)<!-- -->

### Plots Overlays
It is also possible to draw spectra from various SpcLists objects on the same graph. Overlay of Spectra plots can be achieved using *spc.plot.overlay*.


```r
spc.plot.overlay(BL, lw = 2)
```

![](Tutorial_files/figure-html/unnamed-chunk-43-1.png)<!-- -->
As of now, only overlaying lines with *spc.plot* or *spc.lines* have been implemented.

## Interactive Graphics with Javascript libraries

To plot a **Spectra** object with **plotly**, simply pass the **Spectra** object to *spc.plot.plotly()*

```r
h <- spc.plot.plotly(myS)
h
```

<!--html_preserve--><div id="htmlwidget-14d211958ed667d18333" style="width:480px;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-14d211958ed667d18333">{"x":{"layout":{"margin":{"b":40,"l":60,"t":25,"r":10},"title":"spvar2 longname","hovermode":"closest","xaxis":{"domain":[0,1],"title":"Wavelength [nm]"},"yaxis":{"domain":[0,1],"title":"a_nap [1/m]"},"showlegend":false},"source":"A","config":{"modeBarButtonsToAdd":[{"name":"Collaborate","icon":{"width":1000,"ascent":500,"descent":-50,"path":"M487 375c7-10 9-23 5-36l-79-259c-3-12-11-23-22-31-11-8-22-12-35-12l-263 0c-15 0-29 5-43 15-13 10-23 23-28 37-5 13-5 25-1 37 0 0 0 3 1 7 1 5 1 8 1 11 0 2 0 4-1 6 0 3-1 5-1 6 1 2 2 4 3 6 1 2 2 4 4 6 2 3 4 5 5 7 5 7 9 16 13 26 4 10 7 19 9 26 0 2 0 5 0 9-1 4-1 6 0 8 0 2 2 5 4 8 3 3 5 5 5 7 4 6 8 15 12 26 4 11 7 19 7 26 1 1 0 4 0 9-1 4-1 7 0 8 1 2 3 5 6 8 4 4 6 6 6 7 4 5 8 13 13 24 4 11 7 20 7 28 1 1 0 4 0 7-1 3-1 6-1 7 0 2 1 4 3 6 1 1 3 4 5 6 2 3 3 5 5 6 1 2 3 5 4 9 2 3 3 7 5 10 1 3 2 6 4 10 2 4 4 7 6 9 2 3 4 5 7 7 3 2 7 3 11 3 3 0 8 0 13-1l0-1c7 2 12 2 14 2l218 0c14 0 25-5 32-16 8-10 10-23 6-37l-79-259c-7-22-13-37-20-43-7-7-19-10-37-10l-248 0c-5 0-9-2-11-5-2-3-2-7 0-12 4-13 18-20 41-20l264 0c5 0 10 2 16 5 5 3 8 6 10 11l85 282c2 5 2 10 2 17 7-3 13-7 17-13z m-304 0c-1-3-1-5 0-7 1-1 3-2 6-2l174 0c2 0 4 1 7 2 2 2 4 4 5 7l6 18c0 3 0 5-1 7-1 1-3 2-6 2l-173 0c-3 0-5-1-8-2-2-2-4-4-4-7z m-24-73c-1-3-1-5 0-7 2-2 3-2 6-2l174 0c2 0 5 0 7 2 3 2 4 4 5 7l6 18c1 2 0 5-1 6-1 2-3 3-5 3l-174 0c-3 0-5-1-7-3-3-1-4-4-5-6z"},"click":"function(gd) { \n        // is this being viewed in RStudio?\n        if (location.search == '?viewer_pane=1') {\n          alert('To learn about plotly for collaboration, visit:\\n https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html');\n        } else {\n          window.open('https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html', '_blank');\n        }\n      }"}],"modeBarButtonsToRemove":["sendDataToCloud"]},"data":[{"x":[300,301,302,303,304,305,306,307,308,309,310,311,312,313,314,315,316,317,318,319,320,321,322,323,324,325,326,327,328,329,330,331,332,333,334,335,336,337,338,339,340,341,342,343,344,345,346,347,348,349,350,351,352,353,354,355,356,357,358,359,360,361,362,363,364,365,366,367,368,369,370,371,372,373,374,375,376,377,378,379,380,381,382,383,384,385,386,387,388,389,390,391,392,393,394,395,396,397,398,399,400,401,402,403,404,405,406,407,408,409,410,411,412,413,414,415,416,417,418,419,420,421,422,423,424,425,426,427,428,429,430,431,432,433,434,435,436,437,438,439,440,441,442,443,444,445,446,447,448,449,450,451,452,453,454,455,456,457,458,459,460,461,462,463,464,465,466,467,468,469,470,471,472,473,474,475,476,477,478,479,480,481,482,483,484,485,486,487,488,489,490,491,492,493,494,495,496,497,498,499,500,501,502,503,504,505,506,507,508,509,510,511,512,513,514,515,516,517,518,519,520,521,522,523,524,525,526,527,528,529,530,531,532,533,534,535,536,537,538,539,540,541,542,543,544,545,546,547,548,549,550,551,552,553,554,555,556,557,558,559,560,561,562,563,564,565,566,567,568,569,570,571,572,573,574,575,576,577,578,579,580,581,582,583,584,585,586,587,588,589,590,591,592,593,594,595,596,597,598,599,600,601,602,603,604,605,606,607,608,609,610,611,612,613,614,615,616,617,618,619,620,621,622,623,624,625,626,627,628,629,630,631,632,633,634,635,636,637,638,639,640,641,642,643,644,645,646,647,648,649,650,651,652,653,654,655,656,657,658,659,660,661,662,663,664,665,666,667,668,669,670,671,672,673,674,675,676,677,678,679,680,681,682,683,684,685,686,687,688,689,690,691,692,693,694,695,696,697,698,699,700,701,702,703,704,705,706,707,708,709,710,711,712,713,714,715,716,717,718,719,720,721,722,723,724,725,726,727,728,729,730,731,732,733,734,735,736,737,738,739,740,741,742,743,744,745,746,747,748,749,750,751,752,753,754,755,756,757,758,759,760,761,762,763,764,765,766,767,768,769,770,771,772,773,774,775,776,777,778,779,780,781,782,783,784,785,786,787,788,789,790,791,792,793,794,795,796,797,798,799,800],"y":[0.9226343,0.9147542,0.9092862,0.9035436,0.8984991,0.89316,0.8882256,0.8834705,0.8777108,0.8708056,0.8658179,0.8613468,0.8562708,0.8509651,0.8473704,0.8412245,0.8349059,0.8293964,0.8271315,0.8234019,0.8188487,0.816031,0.8105991,0.8026479,0.7965954,0.7927803,0.7874272,0.7834165,0.7808139,0.7751558,0.7692347,0.7628092,0.7579468,0.751886,0.7495307,0.7454932,0.7407469,0.7359669,0.7337372,0.7282643,0.7216801,0.7177148,0.715295,0.7116311,0.7077315,0.7054052,0.7036874,0.6981649,0.6916971,0.6859054,0.6820293,0.6768079,0.6714461,0.6676503,0.6660249,0.661705,0.6561351,0.652193,0.6465725,0.6389752,0.6340678,0.6307097,0.6261095,0.6224716,0.6195378,0.6162435,0.6115401,0.6062127,0.6020649,0.5948935,0.587683,0.5827003,0.5777232,0.5691314,0.5642518,0.560105,0.5510644,0.5448766,0.5413538,0.5362695,0.5303307,0.526883,0.5219886,0.5157688,0.5095434,0.5048079,0.5000101,0.494592,0.4900819,0.4862908,0.4792734,0.4743916,0.4698232,0.4637999,0.4594786,0.4559094,0.4508322,0.4463149,0.4424391,0.4381676,0.4345009,0.4311601,0.4279606,0.4253517,0.4214264,0.4173611,0.4136982,0.4089833,0.4042726,0.4001134,0.3964047,0.3926151,0.3894,0.3861165,0.3833058,0.3807199,0.3769914,0.373941,0.3711969,0.3672527,0.3634082,0.3604937,0.3567511,0.3524345,0.3495753,0.3460577,0.3427009,0.3395694,0.3370303,0.3333106,0.3294143,0.3254103,0.3223585,0.3186284,0.315803,0.3132039,0.3104057,0.3078533,0.3050192,0.301408,0.2978478,0.2941734,0.2900479,0.2869221,0.2846548,0.2832628,0.2819592,0.2801397,0.2777133,0.2756711,0.2727592,0.2707951,0.2688353,0.2678399,0.2657943,0.2646078,0.2626321,0.2612595,0.2595747,0.2583114,0.256731,0.2552882,0.2538812,0.2523367,0.2511272,0.2502737,0.2489584,0.2476067,0.2461731,0.2444091,0.2424677,0.2411885,0.2399762,0.2387157,0.2378543,0.2363835,0.2350589,0.2336608,0.2322248,0.2307487,0.229651,0.2283737,0.2270562,0.2255534,0.2238425,0.2222216,0.220892,0.2192833,0.2179516,0.2167974,0.2155479,0.2139788,0.2128275,0.211503,0.2098334,0.2082351,0.2066382,0.2051172,0.2034219,0.2017288,0.199635,0.1979211,0.1959517,0.1945392,0.1930429,0.1919098,0.1905361,0.188949,0.187242,0.1857185,0.1842916,0.1828131,0.1814105,0.1801573,0.1787778,0.1774448,0.1761481,0.1750561,0.1739724,0.1728114,0.1714956,0.1702309,0.168991,0.167542,0.166427,0.1655346,0.164464,0.1635862,0.1624434,0.1611771,0.159678,0.1583066,0.1566888,0.1557802,0.1546167,0.1539364,0.1531533,0.1524533,0.1513102,0.1507116,0.1494447,0.148583,0.1476201,0.1468434,0.1460959,0.1452989,0.1445148,0.143943,0.1431267,0.1421183,0.1415819,0.1408898,0.1400648,0.1392774,0.1386603,0.1379752,0.1369851,0.136216,0.1357229,0.1349586,0.1342569,0.1338689,0.1330612,0.1322698,0.1313215,0.1304985,0.129745,0.1297332,0.1288353,0.1283643,0.1278944,0.1273959,0.1264929,0.1263771,0.1260816,0.125536,0.1250701,0.1242073,0.1229575,0.1219851,0.1213405,0.1203582,0.1198052,0.1197263,0.1194609,0.1188898,0.1188742,0.1187884,0.1188072,0.1188321,0.1188523,0.1186716,0.118398,0.1179091,0.1172035,0.1165738,0.1162599,0.1158837,0.1153332,0.1150201,0.1148556,0.1140287,0.1135886,0.1133297,0.1131848,0.1128413,0.1127168,0.1120665,0.111932,0.1113677,0.1108473,0.1103962,0.1102382,0.109819,0.1095033,0.1093234,0.1088354,0.1082824,0.1077866,0.1071734,0.1066181,0.1062593,0.1059365,0.1052032,0.1050618,0.1044095,0.1038516,0.1032252,0.1033305,0.1036,0.1042127,0.1045468,0.1048345,0.1044204,0.1038591,0.1028343,0.1021243,0.1015154,0.1013533,0.1005924,0.1003824,0.1001498,0.1001173,0.0996395,0.0993211,0.0987836,0.0984792,0.0979038,0.0974845,0.0973023,0.0977592,0.0975197,0.0974853,0.0973912,0.0971849,0.0965922,0.0963924,0.0963838,0.096339,0.0965199,0.0963071,0.0959345,0.0956664,0.0953973,0.0948153,0.0941539,0.0941444,0.0936586,0.0933711,0.0929468,0.0927057,0.0925042,0.0920594,0.0917088,0.0913208,0.0912984,0.0904599,0.0901608,0.0893617,0.0886435,0.0871348,0.0865791,0.0856278,0.0846744,0.0838146,0.0838484,0.0835325,0.0833833,0.0831712,0.083399,0.08359,0.0826078,0.0824581,0.082618,0.0817854,0.0808945,0.0810271,0.0803457,0.0793161,0.07888,0.0781789,0.0778811,0.0777036,0.0777979,0.0779132,0.0784018,0.0782916,0.0785241,0.0789009,0.0789071,0.07892,0.0787667,0.0790284,0.078874,0.079193,0.0789622,0.0789834,0.0780315,0.077141,0.0760933,0.0755637,0.07515,0.0746722,0.0745424,0.0745126,0.0748019,0.0743835,0.0743253,0.0746175,0.0744997,0.0737869,0.0735787,0.0733189,0.0726878,0.0728083,0.0721461,0.0716569,0.0719042,0.0724622,0.0717696,0.0723371,0.072513,0.0717351,0.0704964,0.0700913,0.0694169,0.0694958,0.0694288,0.0699126,0.0701328,0.0700284,0.0700478,0.0700855,0.0697952,0.0697908,0.0693984,0.0680683,0.0675822,0.0667381,0.0664876,0.0664634,0.0666208,0.0658187,0.0660884,0.0667423,0.0665782,0.0669764,0.0675049,0.0673265,0.0663772,0.0667992,0.0656174,0.0653946,0.0645948,0.0647354,0.0646935,0.065785,0.0648912,0.0652899,0.0649246,0.0647274,0.0634635,0.0648733,0.0642769,0.0646494,0.0641465,0.0656006,0.0653471,0.0658633,0.0659664,0.0641733,0.0625011,0.0620664,0.0621468,0.0594575,0.0617891,0.0617103,0.0615145,0.0617705,0.0628926,0.065468],"type":"scatter","mode":"lines","name":"row 1","hoverinfo":"title","line":{"fillcolor":"rgba(31,119,180,1)","color":"rgba(31,119,180,1)"},"xaxis":"x","yaxis":"y"},{"x":[300,301,302,303,304,305,306,307,308,309,310,311,312,313,314,315,316,317,318,319,320,321,322,323,324,325,326,327,328,329,330,331,332,333,334,335,336,337,338,339,340,341,342,343,344,345,346,347,348,349,350,351,352,353,354,355,356,357,358,359,360,361,362,363,364,365,366,367,368,369,370,371,372,373,374,375,376,377,378,379,380,381,382,383,384,385,386,387,388,389,390,391,392,393,394,395,396,397,398,399,400,401,402,403,404,405,406,407,408,409,410,411,412,413,414,415,416,417,418,419,420,421,422,423,424,425,426,427,428,429,430,431,432,433,434,435,436,437,438,439,440,441,442,443,444,445,446,447,448,449,450,451,452,453,454,455,456,457,458,459,460,461,462,463,464,465,466,467,468,469,470,471,472,473,474,475,476,477,478,479,480,481,482,483,484,485,486,487,488,489,490,491,492,493,494,495,496,497,498,499,500,501,502,503,504,505,506,507,508,509,510,511,512,513,514,515,516,517,518,519,520,521,522,523,524,525,526,527,528,529,530,531,532,533,534,535,536,537,538,539,540,541,542,543,544,545,546,547,548,549,550,551,552,553,554,555,556,557,558,559,560,561,562,563,564,565,566,567,568,569,570,571,572,573,574,575,576,577,578,579,580,581,582,583,584,585,586,587,588,589,590,591,592,593,594,595,596,597,598,599,600,601,602,603,604,605,606,607,608,609,610,611,612,613,614,615,616,617,618,619,620,621,622,623,624,625,626,627,628,629,630,631,632,633,634,635,636,637,638,639,640,641,642,643,644,645,646,647,648,649,650,651,652,653,654,655,656,657,658,659,660,661,662,663,664,665,666,667,668,669,670,671,672,673,674,675,676,677,678,679,680,681,682,683,684,685,686,687,688,689,690,691,692,693,694,695,696,697,698,699,700,701,702,703,704,705,706,707,708,709,710,711,712,713,714,715,716,717,718,719,720,721,722,723,724,725,726,727,728,729,730,731,732,733,734,735,736,737,738,739,740,741,742,743,744,745,746,747,748,749,750,751,752,753,754,755,756,757,758,759,760,761,762,763,764,765,766,767,768,769,770,771,772,773,774,775,776,777,778,779,780,781,782,783,784,785,786,787,788,789,790,791,792,793,794,795,796,797,798,799,800],"y":[0.0844165,0.0834158,0.082929,0.0822148,0.0814748,0.0806608,0.0798236,0.0789943,0.078292,0.0778821,0.0772839,0.0766637,0.0763705,0.0760137,0.0754066,0.0750668,0.0747449,0.0742543,0.0740966,0.0737227,0.0735841,0.0731827,0.0727698,0.0717116,0.071027,0.0702981,0.0699409,0.0698432,0.0695876,0.0692936,0.0689741,0.0686234,0.0678181,0.0675377,0.0670416,0.0666706,0.066401,0.0662704,0.0659077,0.0657331,0.0654062,0.0648227,0.0645322,0.0641024,0.0638288,0.0635436,0.0632616,0.0627008,0.0626511,0.0622045,0.061655,0.061204,0.0607872,0.060266,0.0599328,0.0596594,0.0594932,0.0593225,0.058966,0.0585397,0.0581098,0.0575761,0.0572386,0.056566,0.0564939,0.055998,0.0554557,0.05482,0.0546778,0.0540186,0.0535869,0.053164,0.0529337,0.052018,0.0511725,0.0505048,0.0500055,0.0491568,0.0495016,0.0497629,0.0498512,0.0500136,0.0500955,0.0495883,0.0491115,0.0487916,0.0483769,0.0481163,0.0477203,0.0472922,0.0469308,0.0464747,0.0459436,0.0455885,0.0452548,0.0448072,0.0443294,0.0439597,0.0435354,0.0433089,0.0431448,0.0430187,0.0428512,0.0426447,0.0422548,0.0418551,0.0413691,0.0410177,0.0406854,0.0403684,0.0400326,0.0398439,0.0394682,0.0390622,0.038751,0.0384209,0.0380572,0.0378065,0.0375973,0.0373835,0.0370448,0.0367628,0.0364769,0.0360786,0.0356489,0.0353996,0.0350691,0.0348391,0.0347041,0.0343809,0.0340881,0.0338703,0.033574,0.0331709,0.0329047,0.0326665,0.0323208,0.0319428,0.0317291,0.0315455,0.0312878,0.0310161,0.0307726,0.0305519,0.0303501,0.0301521,0.0300041,0.0297782,0.0295471,0.0292993,0.029035,0.028785,0.0285776,0.0283529,0.0281097,0.0279647,0.027834,0.0277009,0.0274898,0.02739,0.0271363,0.0269563,0.0267914,0.0266959,0.0264673,0.0263018,0.0261092,0.0259645,0.0257841,0.0256533,0.0255193,0.0253266,0.0251537,0.0250042,0.0248351,0.0246694,0.0245481,0.0243972,0.0242313,0.0240768,0.0239678,0.0238195,0.0236357,0.023476,0.0233146,0.0230903,0.0229691,0.0228161,0.0226285,0.0224925,0.0223203,0.0221098,0.0219317,0.0218171,0.0216784,0.0215636,0.0214359,0.0213013,0.0211548,0.0209968,0.0208555,0.0206997,0.020583,0.0204557,0.020325,0.0201798,0.0200382,0.0199028,0.019706,0.0195685,0.0194226,0.0192462,0.0191022,0.019015,0.0188724,0.0187558,0.0186657,0.018519,0.0183649,0.0182587,0.0181591,0.0179806,0.0178441,0.0177275,0.0175885,0.0174496,0.0173747,0.0172945,0.0172003,0.0170988,0.016976,0.0168836,0.0167686,0.0166303,0.0164537,0.0163243,0.0161485,0.0160074,0.0159058,0.0158536,0.0157936,0.015764,0.0157123,0.0155716,0.0154451,0.015335,0.0152301,0.0150805,0.0150669,0.0150218,0.0149365,0.0148619,0.0148,0.0147231,0.0146385,0.014542,0.0144629,0.0143574,0.0142383,0.0141475,0.0140644,0.0140178,0.0140182,0.0139364,0.013899,0.0138203,0.013677,0.0135217,0.0134511,0.0133292,0.0131897,0.013152,0.0131653,0.0131479,0.0130911,0.0131457,0.0130911,0.0130323,0.0129595,0.0128689,0.0127626,0.0126664,0.0126095,0.0125539,0.0125534,0.0125461,0.0125607,0.0124808,0.012407,0.0123699,0.0123142,0.0122371,0.012263,0.0122871,0.0122549,0.0122522,0.0122146,0.0121419,0.0120587,0.0119937,0.0119433,0.011933,0.0118425,0.0118016,0.011772,0.0116681,0.0116092,0.011646,0.0116047,0.0116062,0.0116716,0.0116246,0.0115047,0.0114644,0.0113514,0.0112954,0.0111892,0.0111153,0.0109964,0.0109266,0.0107287,0.0106399,0.0105132,0.0104273,0.0103735,0.0104661,0.0105574,0.0107185,0.0108533,0.0108999,0.0108437,0.0107983,0.0107281,0.0106748,0.010673,0.0107449,0.0107771,0.0107139,0.0106219,0.0106179,0.0104943,0.0104418,0.0104939,0.0104767,0.0103465,0.0103659,0.010379,0.0103413,0.0103247,0.0103968,0.0103216,0.0101764,0.0101615,0.0101661,0.0100921,0.0100546,0.010093,0.0099633,0.0098749,0.0097642,0.0097342,0.0096515,0.0095149,0.0094153,0.0094104,0.0093456,0.0093124,0.009383,0.0094077,0.0093875,0.0093818,0.0093402,0.0094211,0.0093341,0.0093076,0.0093463,0.0094823,0.0094522,0.0095244,0.0095602,0.0094927,0.0093949,0.0093004,0.0093001,0.0092787,0.0092662,0.0092657,0.0093148,0.009303,0.0092384,0.0091573,0.0090269,0.0090522,0.0090657,0.0090705,0.0089794,0.0090249,0.0088579,0.0087482,0.008706,0.0088942,0.0089317,0.0090507,0.0091154,0.0090717,0.0089345,0.0088382,0.008736,0.0087395,0.0087745,0.0088366,0.0087164,0.0088247,0.0088074,0.0088535,0.0089024,0.0090029,0.0088796,0.0088274,0.0088156,0.0087362,0.0088138,0.0088418,0.0088553,0.0085695,0.0085163,0.0083597,0.0082924,0.008229,0.0084027,0.008575,0.0087166,0.0087592,0.0088746,0.0088509,0.0084254,0.008201,0.0080729,0.0081032,0.0081775,0.0083832,0.0084825,0.0085626,0.0083962,0.0084286,0.0083464,0.0081708,0.0080939,0.0079739,0.0078749,0.0078634,0.0079048,0.007926,0.0080556,0.0077295,0.0078047,0.0078327,0.0076484,0.0074217,0.00761,0.0075645,0.007552,0.0077689,0.0079484,0.0078887,0.0079062,0.0078305,0.0076915,0.0076431,0.0076598,0.0077219,0.0079529,0.0079655,0.0079214,0.0080353,0.0079205,0.0077245,0.0077669,0.0078371,0.0075989,0.0075527,0.007557,0.0073214,0.007162,0.0072473,0.0074226,0.0074125,0.0075139,0.0074513,0.0075158,0.0073475,0.0071332,0.0072155,0.007194,0.0071946,0.0071027,0.0071925,0.0068709,0.0068953,0.0060827,0.0068974],"type":"scatter","mode":"lines","name":"row 3","hoverinfo":"title","line":{"fillcolor":"rgba(255,127,14,1)","color":"rgba(255,127,14,1)"},"xaxis":"x","yaxis":"y"},{"x":[300,301,302,303,304,305,306,307,308,309,310,311,312,313,314,315,316,317,318,319,320,321,322,323,324,325,326,327,328,329,330,331,332,333,334,335,336,337,338,339,340,341,342,343,344,345,346,347,348,349,350,351,352,353,354,355,356,357,358,359,360,361,362,363,364,365,366,367,368,369,370,371,372,373,374,375,376,377,378,379,380,381,382,383,384,385,386,387,388,389,390,391,392,393,394,395,396,397,398,399,400,401,402,403,404,405,406,407,408,409,410,411,412,413,414,415,416,417,418,419,420,421,422,423,424,425,426,427,428,429,430,431,432,433,434,435,436,437,438,439,440,441,442,443,444,445,446,447,448,449,450,451,452,453,454,455,456,457,458,459,460,461,462,463,464,465,466,467,468,469,470,471,472,473,474,475,476,477,478,479,480,481,482,483,484,485,486,487,488,489,490,491,492,493,494,495,496,497,498,499,500,501,502,503,504,505,506,507,508,509,510,511,512,513,514,515,516,517,518,519,520,521,522,523,524,525,526,527,528,529,530,531,532,533,534,535,536,537,538,539,540,541,542,543,544,545,546,547,548,549,550,551,552,553,554,555,556,557,558,559,560,561,562,563,564,565,566,567,568,569,570,571,572,573,574,575,576,577,578,579,580,581,582,583,584,585,586,587,588,589,590,591,592,593,594,595,596,597,598,599,600,601,602,603,604,605,606,607,608,609,610,611,612,613,614,615,616,617,618,619,620,621,622,623,624,625,626,627,628,629,630,631,632,633,634,635,636,637,638,639,640,641,642,643,644,645,646,647,648,649,650,651,652,653,654,655,656,657,658,659,660,661,662,663,664,665,666,667,668,669,670,671,672,673,674,675,676,677,678,679,680,681,682,683,684,685,686,687,688,689,690,691,692,693,694,695,696,697,698,699,700,701,702,703,704,705,706,707,708,709,710,711,712,713,714,715,716,717,718,719,720,721,722,723,724,725,726,727,728,729,730,731,732,733,734,735,736,737,738,739,740,741,742,743,744,745,746,747,748,749,750,751,752,753,754,755,756,757,758,759,760,761,762,763,764,765,766,767,768,769,770,771,772,773,774,775,776,777,778,779,780,781,782,783,784,785,786,787,788,789,790,791,792,793,794,795,796,797,798,799,800],"y":[0.0917964,0.0891377,0.0890812,0.0881955,0.087406,0.0867068,0.0859258,0.0852419,0.0844348,0.0839545,0.0833917,0.0828403,0.0824815,0.0820719,0.0815407,0.0811019,0.0806935,0.0804335,0.0803456,0.0799669,0.0798327,0.0795339,0.0790564,0.0783497,0.0777353,0.0772035,0.0769527,0.0765115,0.0760214,0.0759867,0.0755029,0.0747664,0.0741946,0.0738037,0.0730978,0.0728995,0.0726886,0.0723203,0.0717783,0.0717439,0.0711673,0.0706557,0.0704064,0.0701608,0.0694358,0.0691155,0.0686601,0.0678849,0.0674396,0.0672898,0.0666802,0.0663791,0.0659554,0.0655129,0.0647349,0.0644051,0.0638722,0.0636042,0.0631784,0.0628371,0.0623253,0.0618429,0.0614243,0.0608957,0.0608176,0.0604831,0.0601146,0.0596563,0.0595009,0.0588248,0.0581859,0.057539,0.0572471,0.056596,0.0556951,0.0550406,0.0545734,0.0537009,0.0534562,0.0535248,0.0533483,0.053145,0.0529678,0.0523624,0.0520229,0.0516585,0.0512038,0.0507009,0.0502449,0.049638,0.0491662,0.0486484,0.0483426,0.0479342,0.0475511,0.0470672,0.0466865,0.0461995,0.0458382,0.045554,0.0453854,0.0451705,0.0449815,0.0447099,0.0443315,0.0438733,0.0433343,0.0430171,0.0425574,0.0421899,0.0419206,0.0416688,0.0412391,0.0409921,0.0406621,0.0403657,0.0400908,0.0397657,0.039381,0.0391006,0.0386463,0.038389,0.038138,0.0378199,0.0373924,0.0370909,0.0366985,0.0363042,0.0360696,0.0357628,0.0355333,0.0352584,0.0349388,0.0345159,0.0342124,0.0338188,0.0333731,0.0331218,0.0329703,0.03269,0.032435,0.0322379,0.0318839,0.0315403,0.0313842,0.0311849,0.0309344,0.0307576,0.0305375,0.0302419,0.030044,0.0299003,0.0296436,0.0294298,0.0291864,0.0289008,0.0286333,0.0284704,0.0282009,0.0280097,0.0277466,0.0275849,0.027401,0.0272854,0.0270972,0.0269758,0.0267872,0.0266492,0.026483,0.0263706,0.0261677,0.0259841,0.0257792,0.0255887,0.0253639,0.0252059,0.0249985,0.0248502,0.0247288,0.0246143,0.0244981,0.0243716,0.0241609,0.023971,0.0237459,0.0234807,0.0232561,0.0230916,0.0228972,0.0227014,0.0225786,0.0224386,0.0221805,0.0219935,0.0218844,0.0217669,0.0215814,0.0214834,0.0213185,0.0211299,0.0209607,0.020845,0.0207094,0.0205861,0.0204892,0.020307,0.020153,0.0200499,0.0198291,0.0196591,0.0194969,0.0193488,0.0191831,0.0191153,0.0189685,0.0188695,0.0187567,0.0185833,0.0184309,0.0183046,0.0181576,0.017963,0.0178177,0.017663,0.0175216,0.0173897,0.017292,0.017192,0.0170767,0.0169551,0.016813,0.0167209,0.0166379,0.0165452,0.0164309,0.0163743,0.0162298,0.0160596,0.015973,0.0158708,0.0157553,0.0156966,0.0156676,0.0155505,0.0154558,0.0153798,0.0152718,0.0151271,0.0150212,0.0149474,0.0148376,0.0147151,0.0146469,0.0145654,0.0144536,0.0144009,0.0143454,0.0143045,0.0142221,0.0141574,0.0140617,0.0139945,0.0139444,0.0139212,0.0138939,0.013847,0.0137515,0.0135488,0.0134108,0.0132588,0.0131201,0.0130706,0.0130962,0.013074,0.0130765,0.0130881,0.0130904,0.0129974,0.0129292,0.0127853,0.012644,0.0124744,0.0124407,0.0123839,0.0123342,0.0123363,0.0123054,0.0122467,0.012149,0.0121493,0.0121023,0.0120678,0.0120189,0.0120488,0.0120011,0.0119501,0.0118711,0.0118125,0.0117137,0.0116602,0.0116133,0.0116306,0.0116054,0.0115731,0.0115587,0.011482,0.0114048,0.0113767,0.011344,0.0113323,0.011372,0.0113589,0.0112177,0.0112033,0.0111235,0.0110418,0.0110471,0.0111295,0.0110742,0.011044,0.0109811,0.0109195,0.0108433,0.0107808,0.0107948,0.010768,0.0106874,0.0107574,0.0108157,0.0107231,0.010676,0.0106156,0.0104841,0.0104175,0.0104356,0.0104746,0.0105029,0.010511,0.0104399,0.0103798,0.0102837,0.0102278,0.0101476,0.0101608,0.0100744,0.0101196,0.010179,0.0102119,0.0101351,0.0101874,0.0100766,0.0099608,0.0099609,0.0099034,0.0098262,0.0098429,0.0098424,0.0097391,0.0098111,0.0097881,0.0097935,0.0097806,0.0097874,0.009725,0.0097376,0.0096462,0.0095826,0.0095323,0.0094315,0.009414,0.0094528,0.0093774,0.0093813,0.0093986,0.0092636,0.0092566,0.0093741,0.00927,0.0091804,0.0091991,0.0091614,0.0090808,0.0090634,0.0091558,0.0090755,0.0089787,0.0089326,0.0089221,0.0088236,0.0088403,0.0087855,0.0087309,0.0087754,0.0087626,0.008881,0.0088447,0.0088605,0.0087857,0.0086955,0.0085526,0.0085658,0.0085384,0.0086096,0.0087148,0.008681,0.0086616,0.0086317,0.0084602,0.0083552,0.0083762,0.0083232,0.0081911,0.0083264,0.0084337,0.0084701,0.0085733,0.0086792,0.0086781,0.0086259,0.0085615,0.0085023,0.0085125,0.0082548,0.0082046,0.008075,0.0079364,0.007872,0.007925,0.0078589,0.0079748,0.0081584,0.0081589,0.0082778,0.0083574,0.0082917,0.0080141,0.0078872,0.0078503,0.0078311,0.0077707,0.0079513,0.0080777,0.0080552,0.0078114,0.0078123,0.0078273,0.0078017,0.0076515,0.0077922,0.0077657,0.0076076,0.0075277,0.0075063,0.0077115,0.0076109,0.0075574,0.0075923,0.0074927,0.0072513,0.0073865,0.0073392,0.0073711,0.007402,0.0074117,0.0071904,0.0074213,0.0074761,0.0073711,0.0073216,0.0074143,0.007316,0.0072943,0.0073001,0.0071772,0.0072817,0.0072871,0.0070824,0.0072894,0.0074877,0.0073483,0.007409,0.0074885,0.0072977,0.007098,0.0069676,0.007125,0.0070039,0.0071544,0.0072695,0.0075044,0.0071779,0.0073145,0.0074465,0.0072006,0.0074163,0.0074829,0.0073452,0.0070542,0.0073376,0.0065098,0.0074943],"type":"scatter","mode":"lines","name":"row 6","hoverinfo":"title","line":{"fillcolor":"rgba(44,160,44,1)","color":"rgba(44,160,44,1)"},"xaxis":"x","yaxis":"y"},{"x":[300,301,302,303,304,305,306,307,308,309,310,311,312,313,314,315,316,317,318,319,320,321,322,323,324,325,326,327,328,329,330,331,332,333,334,335,336,337,338,339,340,341,342,343,344,345,346,347,348,349,350,351,352,353,354,355,356,357,358,359,360,361,362,363,364,365,366,367,368,369,370,371,372,373,374,375,376,377,378,379,380,381,382,383,384,385,386,387,388,389,390,391,392,393,394,395,396,397,398,399,400,401,402,403,404,405,406,407,408,409,410,411,412,413,414,415,416,417,418,419,420,421,422,423,424,425,426,427,428,429,430,431,432,433,434,435,436,437,438,439,440,441,442,443,444,445,446,447,448,449,450,451,452,453,454,455,456,457,458,459,460,461,462,463,464,465,466,467,468,469,470,471,472,473,474,475,476,477,478,479,480,481,482,483,484,485,486,487,488,489,490,491,492,493,494,495,496,497,498,499,500,501,502,503,504,505,506,507,508,509,510,511,512,513,514,515,516,517,518,519,520,521,522,523,524,525,526,527,528,529,530,531,532,533,534,535,536,537,538,539,540,541,542,543,544,545,546,547,548,549,550,551,552,553,554,555,556,557,558,559,560,561,562,563,564,565,566,567,568,569,570,571,572,573,574,575,576,577,578,579,580,581,582,583,584,585,586,587,588,589,590,591,592,593,594,595,596,597,598,599,600,601,602,603,604,605,606,607,608,609,610,611,612,613,614,615,616,617,618,619,620,621,622,623,624,625,626,627,628,629,630,631,632,633,634,635,636,637,638,639,640,641,642,643,644,645,646,647,648,649,650,651,652,653,654,655,656,657,658,659,660,661,662,663,664,665,666,667,668,669,670,671,672,673,674,675,676,677,678,679,680,681,682,683,684,685,686,687,688,689,690,691,692,693,694,695,696,697,698,699,700,701,702,703,704,705,706,707,708,709,710,711,712,713,714,715,716,717,718,719,720,721,722,723,724,725,726,727,728,729,730,731,732,733,734,735,736,737,738,739,740,741,742,743,744,745,746,747,748,749,750,751,752,753,754,755,756,757,758,759,760,761,762,763,764,765,766,767,768,769,770,771,772,773,774,775,776,777,778,779,780,781,782,783,784,785,786,787,788,789,790,791,792,793,794,795,796,797,798,799,800],"y":[0.1349536,0.1307,0.1299586,0.1281645,0.1271095,0.125799,0.124674,0.1232514,0.121844,0.1204316,0.1191953,0.1180495,0.1167969,0.1156775,0.1146001,0.1136183,0.1123108,0.1117054,0.1115474,0.1111178,0.1111252,0.1112134,0.1111502,0.1104207,0.1095545,0.1084559,0.1075311,0.1066661,0.1061013,0.1056566,0.1050186,0.104114,0.1031671,0.1019828,0.1013157,0.1006001,0.0999489,0.0994109,0.0988578,0.0982217,0.0973486,0.0966522,0.0954863,0.0943339,0.0935935,0.0932751,0.092763,0.0921561,0.0916932,0.0903798,0.0892553,0.0881903,0.0875676,0.0869116,0.0864031,0.0854221,0.0847662,0.0841014,0.0832847,0.08282,0.0824363,0.0818859,0.0810551,0.0802133,0.0792376,0.0784099,0.0778126,0.0772944,0.0765729,0.0759154,0.0753169,0.0746839,0.0736977,0.0731026,0.0724816,0.0715247,0.0703598,0.0702484,0.0700131,0.0700023,0.0702904,0.0704473,0.0699017,0.069633,0.0689854,0.068258,0.0676382,0.0672342,0.0665394,0.0660561,0.0655332,0.0652207,0.064619,0.0642952,0.0638711,0.0634469,0.0629003,0.0628067,0.0624685,0.0619883,0.0618284,0.0616864,0.0611282,0.0607755,0.0605924,0.0601972,0.059683,0.0593434,0.0588705,0.0585097,0.0580977,0.057755,0.0572674,0.0570385,0.0565964,0.0560526,0.0559602,0.0559022,0.0555273,0.0552233,0.0550128,0.0544218,0.0539774,0.0535583,0.0530657,0.0526517,0.0523344,0.0520663,0.0516454,0.0512886,0.0509054,0.050383,0.0498287,0.0494438,0.0490601,0.0486735,0.0483942,0.048042,0.04775,0.0474925,0.0471102,0.0467458,0.0464114,0.0460794,0.0456976,0.0454046,0.0449841,0.0446096,0.0443122,0.0439052,0.0435669,0.0434068,0.0430275,0.0425795,0.0423759,0.0420679,0.0416246,0.0413287,0.0410856,0.0407512,0.0404131,0.0402164,0.0400152,0.0397213,0.039324,0.0389807,0.0386259,0.038202,0.0377476,0.0374083,0.0371541,0.0367567,0.0364074,0.0361637,0.035806,0.0354717,0.0352261,0.0351011,0.0348535,0.0345915,0.0342796,0.0339552,0.033583,0.0331649,0.0329259,0.0325786,0.0322953,0.0319416,0.0317893,0.0314845,0.0312842,0.0310852,0.0309214,0.0306372,0.0304567,0.0302086,0.0299548,0.0296073,0.0292724,0.0288989,0.0287414,0.028522,0.0282997,0.0281889,0.028054,0.0277782,0.0274634,0.0272503,0.0270064,0.0268528,0.0266257,0.0264682,0.0262934,0.0261126,0.0258205,0.0256228,0.025389,0.0252903,0.0250691,0.0249139,0.0248043,0.0246699,0.0243987,0.0242391,0.0241218,0.0239194,0.0238084,0.0236762,0.023539,0.0233354,0.0231695,0.022891,0.0227104,0.0225506,0.0223325,0.0220756,0.0220718,0.0218776,0.0217245,0.0216222,0.0215597,0.0213487,0.0213108,0.021182,0.0210601,0.0209395,0.020835,0.0206888,0.020578,0.0204315,0.0203465,0.020239,0.0201185,0.0200061,0.0199105,0.0196974,0.019618,0.0195067,0.0193886,0.0192834,0.0192564,0.0191681,0.0191546,0.0190472,0.0190286,0.0189793,0.0188392,0.0186861,0.0186278,0.0185152,0.0184112,0.0183029,0.0181767,0.0180849,0.0180258,0.0180302,0.0179486,0.0178994,0.0177739,0.017703,0.017522,0.0174274,0.0174337,0.0174659,0.0173157,0.0172246,0.0171838,0.0170651,0.0168996,0.0168478,0.0168606,0.0168473,0.0167621,0.0167654,0.0167223,0.01662,0.0165283,0.0164539,0.0163423,0.0162804,0.0161894,0.0161576,0.0160619,0.0160769,0.0160061,0.0159713,0.0158899,0.0159592,0.0158761,0.0157754,0.015777,0.0157876,0.0157626,0.015764,0.0158072,0.0157658,0.0156578,0.0155793,0.0155282,0.0154805,0.015382,0.0153555,0.015291,0.0152203,0.015139,0.0151732,0.0151205,0.0150914,0.0151442,0.0151219,0.0149944,0.0148525,0.0147406,0.0145915,0.0147293,0.014794,0.0148909,0.014902,0.0149479,0.0147945,0.0146181,0.0145232,0.0144962,0.0144165,0.0144241,0.0145038,0.0146125,0.0146472,0.0146051,0.0146062,0.0146606,0.0146136,0.0146784,0.0148507,0.0148494,0.0148358,0.014871,0.0148889,0.0148989,0.0148594,0.0149844,0.0149943,0.0150651,0.0151106,0.0151647,0.0151234,0.0152378,0.0150921,0.0149943,0.0149056,0.0146796,0.014505,0.0144244,0.0142325,0.014185,0.0142214,0.0140972,0.0140197,0.0140189,0.0139125,0.0137309,0.0136797,0.0133004,0.0129944,0.0129134,0.0127364,0.0124604,0.0124755,0.0123968,0.0122208,0.0121598,0.0120015,0.0120005,0.0119355,0.0117532,0.0116656,0.0116748,0.011452,0.0113173,0.0112353,0.0111178,0.0109868,0.0111429,0.0111157,0.0110948,0.011009,0.0111129,0.010859,0.0108198,0.0107823,0.0108355,0.0106401,0.0106121,0.0107252,0.0106521,0.0105778,0.0105246,0.010403,0.0102998,0.0102923,0.0102209,0.0102776,0.0102248,0.0101022,0.0099933,0.009923,0.009922,0.0100224,0.0099782,0.0101169,0.009968,0.0099488,0.0098069,0.0097258,0.0095377,0.0095703,0.009449,0.0094991,0.0093674,0.0093249,0.0093621,0.0092845,0.0091123,0.0092758,0.0092519,0.0092719,0.009359,0.0095235,0.0093546,0.0092782,0.0090621,0.0091394,0.0091372,0.0092606,0.0093384,0.0093341,0.0092586,0.009145,0.0091447,0.0091263,0.0090001,0.0089368,0.0088643,0.0088752,0.0088521,0.0090786,0.0091253,0.0092293,0.0090881,0.0092697,0.0092085,0.0088864,0.008803,0.0088299,0.0085566,0.008473,0.0086532,0.0086835,0.0087011,0.0088837,0.0086878,0.0087392,0.0086981,0.0084931,0.0084762,0.0087127,0.0083433,0.0084262,0.00897,0.0088972,0.0088448,0.0094159,0.0090386,0.0085636,0.0083087,0.0085197,0.0081218,0.0083694,0.0084062,0.008353,0.0085243],"type":"scatter","mode":"lines","name":"row 9","hoverinfo":"title","line":{"fillcolor":"rgba(214,39,40,1)","color":"rgba(214,39,40,1)"},"xaxis":"x","yaxis":"y"},{"x":[300,301,302,303,304,305,306,307,308,309,310,311,312,313,314,315,316,317,318,319,320,321,322,323,324,325,326,327,328,329,330,331,332,333,334,335,336,337,338,339,340,341,342,343,344,345,346,347,348,349,350,351,352,353,354,355,356,357,358,359,360,361,362,363,364,365,366,367,368,369,370,371,372,373,374,375,376,377,378,379,380,381,382,383,384,385,386,387,388,389,390,391,392,393,394,395,396,397,398,399,400,401,402,403,404,405,406,407,408,409,410,411,412,413,414,415,416,417,418,419,420,421,422,423,424,425,426,427,428,429,430,431,432,433,434,435,436,437,438,439,440,441,442,443,444,445,446,447,448,449,450,451,452,453,454,455,456,457,458,459,460,461,462,463,464,465,466,467,468,469,470,471,472,473,474,475,476,477,478,479,480,481,482,483,484,485,486,487,488,489,490,491,492,493,494,495,496,497,498,499,500,501,502,503,504,505,506,507,508,509,510,511,512,513,514,515,516,517,518,519,520,521,522,523,524,525,526,527,528,529,530,531,532,533,534,535,536,537,538,539,540,541,542,543,544,545,546,547,548,549,550,551,552,553,554,555,556,557,558,559,560,561,562,563,564,565,566,567,568,569,570,571,572,573,574,575,576,577,578,579,580,581,582,583,584,585,586,587,588,589,590,591,592,593,594,595,596,597,598,599,600,601,602,603,604,605,606,607,608,609,610,611,612,613,614,615,616,617,618,619,620,621,622,623,624,625,626,627,628,629,630,631,632,633,634,635,636,637,638,639,640,641,642,643,644,645,646,647,648,649,650,651,652,653,654,655,656,657,658,659,660,661,662,663,664,665,666,667,668,669,670,671,672,673,674,675,676,677,678,679,680,681,682,683,684,685,686,687,688,689,690,691,692,693,694,695,696,697,698,699,700,701,702,703,704,705,706,707,708,709,710,711,712,713,714,715,716,717,718,719,720,721,722,723,724,725,726,727,728,729,730,731,732,733,734,735,736,737,738,739,740,741,742,743,744,745,746,747,748,749,750,751,752,753,754,755,756,757,758,759,760,761,762,763,764,765,766,767,768,769,770,771,772,773,774,775,776,777,778,779,780,781,782,783,784,785,786,787,788,789,790,791,792,793,794,795,796,797,798,799,800],"y":[0.0172168,0.0167917,0.0164745,0.0162154,0.0159521,0.0158521,0.0156274,0.0154539,0.0152787,0.0150503,0.0148012,0.0146904,0.0145163,0.0143351,0.0142454,0.0141762,0.0140648,0.0140106,0.0140313,0.0139728,0.0138502,0.013682,0.0136144,0.0133674,0.0133401,0.0132197,0.0132088,0.0131023,0.0131087,0.0130186,0.0129743,0.0129178,0.0128727,0.0128413,0.0128246,0.012861,0.0127186,0.0126202,0.0124821,0.0124394,0.01228,0.012298,0.012192,0.0121568,0.0119878,0.0119409,0.0117768,0.0117281,0.0117225,0.0116061,0.011448,0.0114515,0.0113,0.0111983,0.0111279,0.0110847,0.0110504,0.011039,0.0108928,0.0108784,0.01071,0.0105478,0.0105114,0.0104073,0.0102564,0.0102647,0.0102024,0.0100354,0.0100101,0.0098972,0.0098137,0.0097343,0.0096233,0.0093653,0.0092207,0.0090532,0.0086851,0.0086587,0.0088131,0.0088661,0.0089706,0.009251,0.0092356,0.0091254,0.0091185,0.008999,0.0089571,0.0089146,0.0088349,0.0087677,0.0087406,0.0086177,0.0085479,0.0084614,0.0083916,0.0083754,0.0083134,0.0082101,0.0081335,0.0080119,0.0079286,0.0078915,0.0078412,0.0077917,0.0077257,0.0075709,0.0074577,0.0074509,0.0073828,0.0073599,0.0073109,0.0072727,0.0071758,0.0071182,0.0070341,0.0070004,0.006964,0.0069403,0.0069056,0.0068619,0.0067932,0.0066926,0.006593,0.0065321,0.0064916,0.0064702,0.0064245,0.006354,0.006327,0.0062921,0.0062644,0.0062403,0.0062388,0.0061868,0.0061178,0.0060368,0.0059962,0.0059459,0.0059086,0.0058755,0.0058556,0.0058079,0.0057366,0.0056621,0.0056299,0.0055905,0.0055395,0.0055088,0.0055146,0.0054871,0.0054745,0.0054483,0.0054565,0.0054138,0.0053834,0.005306,0.0052837,0.0052078,0.0051707,0.0051553,0.0051831,0.0051508,0.0051579,0.0051392,0.0050952,0.0050511,0.0050268,0.0049885,0.0049743,0.004934,0.0049011,0.0048934,0.0048765,0.0048509,0.0048281,0.0048096,0.0047787,0.0047558,0.0047106,0.0046988,0.0046653,0.0046453,0.0046363,0.0046394,0.004605,0.0045522,0.0045285,0.0045139,0.0044874,0.0044874,0.0045048,0.004477,0.0044272,0.0044065,0.0043587,0.0043074,0.0042875,0.0042645,0.0042327,0.0042108,0.0041833,0.0041457,0.0041233,0.0041106,0.0040968,0.0040863,0.0040527,0.0040352,0.0040098,0.0039892,0.0039817,0.0039786,0.0039579,0.0039224,0.0039099,0.0038955,0.0038809,0.0038589,0.0038485,0.0038156,0.0037736,0.0037488,0.0037185,0.0036954,0.0036852,0.0036745,0.0036612,0.0036637,0.0036524,0.0036311,0.0036321,0.0036143,0.0035694,0.0035219,0.0034826,0.0034342,0.00341,0.003417,0.0034181,0.0034234,0.0034279,0.0033895,0.0033678,0.0033537,0.0033357,0.0033255,0.0033291,0.003326,0.0033183,0.0033016,0.0032785,0.0032528,0.0032182,0.0032028,0.0031807,0.0031508,0.003149,0.0031246,0.0030976,0.003091,0.0030948,0.0030999,0.0031401,0.0031399,0.0030994,0.0030825,0.0030649,0.0030364,0.0030341,0.0030847,0.0030758,0.0030846,0.0030686,0.0030527,0.0030057,0.0030011,0.0029782,0.0029891,0.0029692,0.0029645,0.0029581,0.0029364,0.0029309,0.0029356,0.0029279,0.0028964,0.002905,0.002868,0.0028475,0.0028655,0.0028958,0.0028636,0.0028643,0.0028704,0.0028584,0.0028336,0.0028362,0.0028186,0.0027896,0.0027656,0.0027575,0.002745,0.0027539,0.0027663,0.00277,0.002778,0.0027667,0.0027472,0.0027212,0.0027155,0.0026871,0.002702,0.0027047,0.0027218,0.0027102,0.0027195,0.0027152,0.0026732,0.0026544,0.0026546,0.002635,0.0026223,0.0026559,0.0026887,0.0026935,0.0027294,0.0027159,0.0027073,0.0026646,0.0026428,0.0026097,0.002603,0.0025906,0.0026128,0.0026144,0.0025995,0.0025861,0.0025706,0.0025209,0.0024871,0.0024813,0.0024875,0.0025025,0.0025268,0.0025568,0.0025642,0.0026094,0.002618,0.0026054,0.0025882,0.0025731,0.0025052,0.0024888,0.0024739,0.0024858,0.0024999,0.0025006,0.002467,0.0024642,0.0024541,0.0024418,0.0024228,0.0024738,0.0024701,0.0024396,0.0024552,0.0024805,0.0024046,0.0023946,0.0023694,0.002339,0.0023276,0.0023414,0.0023422,0.00234,0.002332,0.0023246,0.002328,0.0023031,0.0023363,0.0023127,0.0022979,0.0022536,0.0022546,0.0022369,0.0022546,0.0022653,0.0022667,0.0022774,0.0022269,0.0022054,0.0021729,0.0021426,0.0021155,0.0021393,0.00209,0.002094,0.0021643,0.0021616,0.0021235,0.0021723,0.0021638,0.0021213,0.002127,0.0021558,0.0021146,0.002121,0.002139,0.0021091,0.0021393,0.002122,0.002102,0.0020672,0.0020863,0.0020309,0.0020912,0.0020676,0.0020491,0.0020237,0.0020158,0.0019841,0.0020469,0.0020459,0.0020005,0.0020423,0.0020501,0.0019896,0.0019505,0.0020155,0.0019809,0.0019151,0.0019461,0.0019999,0.0019982,0.0019428,0.0019407,0.0019222,0.0018963,0.0018356,0.0019268,0.0019897,0.0019775,0.0020164,0.0020779,0.002005,0.0019782,0.0019921,0.0019918,0.0019728,0.0020336,0.0020727,0.0020478,0.0019418,0.0019088,0.0018459,0.0017741,0.0017754,0.0018586,0.0018291,0.0018545,0.0019221,0.0019844,0.0019698,0.0019873,0.0019523,0.0018723,0.0018138,0.0018538,0.0018586,0.0018514,0.0018901,0.0019725,0.0018455,0.0018046,0.0018945,0.0018379,0.0017045,0.0017212,0.0016936,0.001568,0.0015879,0.0015857,0.0015945,0.0015306,0.0015344,0.0013422,0.0014018,0.0013944,0.0014723,0.0014585,0.0014875,0.0013983,0.0014,0.0014309,0.0014654,0.0015866,0.0015885,0.001662,0.001632,0.0019165,0.0015123],"type":"scatter","mode":"lines","name":"row 12","hoverinfo":"title","line":{"fillcolor":"rgba(148,103,189,1)","color":"rgba(148,103,189,1)"},"xaxis":"x","yaxis":"y"},{"x":[300,301,302,303,304,305,306,307,308,309,310,311,312,313,314,315,316,317,318,319,320,321,322,323,324,325,326,327,328,329,330,331,332,333,334,335,336,337,338,339,340,341,342,343,344,345,346,347,348,349,350,351,352,353,354,355,356,357,358,359,360,361,362,363,364,365,366,367,368,369,370,371,372,373,374,375,376,377,378,379,380,381,382,383,384,385,386,387,388,389,390,391,392,393,394,395,396,397,398,399,400,401,402,403,404,405,406,407,408,409,410,411,412,413,414,415,416,417,418,419,420,421,422,423,424,425,426,427,428,429,430,431,432,433,434,435,436,437,438,439,440,441,442,443,444,445,446,447,448,449,450,451,452,453,454,455,456,457,458,459,460,461,462,463,464,465,466,467,468,469,470,471,472,473,474,475,476,477,478,479,480,481,482,483,484,485,486,487,488,489,490,491,492,493,494,495,496,497,498,499,500,501,502,503,504,505,506,507,508,509,510,511,512,513,514,515,516,517,518,519,520,521,522,523,524,525,526,527,528,529,530,531,532,533,534,535,536,537,538,539,540,541,542,543,544,545,546,547,548,549,550,551,552,553,554,555,556,557,558,559,560,561,562,563,564,565,566,567,568,569,570,571,572,573,574,575,576,577,578,579,580,581,582,583,584,585,586,587,588,589,590,591,592,593,594,595,596,597,598,599,600,601,602,603,604,605,606,607,608,609,610,611,612,613,614,615,616,617,618,619,620,621,622,623,624,625,626,627,628,629,630,631,632,633,634,635,636,637,638,639,640,641,642,643,644,645,646,647,648,649,650,651,652,653,654,655,656,657,658,659,660,661,662,663,664,665,666,667,668,669,670,671,672,673,674,675,676,677,678,679,680,681,682,683,684,685,686,687,688,689,690,691,692,693,694,695,696,697,698,699,700,701,702,703,704,705,706,707,708,709,710,711,712,713,714,715,716,717,718,719,720,721,722,723,724,725,726,727,728,729,730,731,732,733,734,735,736,737,738,739,740,741,742,743,744,745,746,747,748,749,750,751,752,753,754,755,756,757,758,759,760,761,762,763,764,765,766,767,768,769,770,771,772,773,774,775,776,777,778,779,780,781,782,783,784,785,786,787,788,789,790,791,792,793,794,795,796,797,798,799,800],"y":[0.0202518,0.0199117,0.0198026,0.0196654,0.0195186,0.0194059,0.0193077,0.0191696,0.0190265,0.01893,0.0188794,0.0187498,0.0186777,0.0185845,0.0184655,0.0183205,0.0182128,0.0180693,0.0180494,0.0179974,0.0179597,0.0179407,0.0179492,0.0178943,0.0177876,0.0177405,0.0176201,0.0175427,0.01742,0.017372,0.0172798,0.0171772,0.0170031,0.0168537,0.016774,0.0166529,0.0165032,0.0165146,0.0164509,0.0163988,0.0163149,0.0162815,0.0161576,0.0160259,0.0158976,0.0158199,0.0157581,0.0156434,0.0156361,0.0155018,0.0154009,0.0153165,0.0152593,0.0150967,0.0150214,0.014931,0.0148532,0.0148342,0.0147791,0.0147897,0.0147236,0.0146137,0.0144613,0.0143496,0.0141886,0.0140307,0.0139305,0.0138106,0.0137348,0.0136615,0.013558,0.013427,0.013338,0.013159,0.0129918,0.0128793,0.0126939,0.0126625,0.0127231,0.0127881,0.0128174,0.0129182,0.0128367,0.0127268,0.0125968,0.0125089,0.0124123,0.012319,0.0122302,0.0121278,0.0120084,0.0119589,0.0118856,0.0118664,0.0118566,0.01183,0.0117423,0.0116864,0.011579,0.0114832,0.0114464,0.0113928,0.0113216,0.0112569,0.0111953,0.0110897,0.0110437,0.0109921,0.010968,0.0109143,0.0108559,0.0107678,0.0106845,0.0105845,0.0104798,0.0103956,0.0103441,0.0103082,0.0102366,0.0102139,0.0101788,0.0101003,0.0100177,0.0099186,0.0097974,0.009719,0.0096382,0.009548,0.0095189,0.0094766,0.0093636,0.0092835,0.0091879,0.0091002,0.0090265,0.0089908,0.0089249,0.0088514,0.008775,0.0086937,0.0085935,0.0084976,0.0084514,0.0083974,0.0083396,0.0082726,0.0082055,0.0081279,0.0080427,0.0079572,0.0079035,0.0078832,0.0078003,0.0077278,0.0076786,0.0076274,0.0075436,0.0074823,0.0074267,0.0073576,0.0072968,0.0072403,0.0072262,0.0071871,0.0071249,0.0070802,0.0070445,0.0069817,0.006919,0.006883,0.0068271,0.0067722,0.0067299,0.0066972,0.0066351,0.0065874,0.0065486,0.0064966,0.0064474,0.006402,0.0063644,0.0062933,0.0062576,0.006185,0.0061454,0.0061001,0.0060742,0.0060146,0.0060088,0.005974,0.0059217,0.0058799,0.0058759,0.0058305,0.0058024,0.0057583,0.0057108,0.0056211,0.0055506,0.005475,0.0054521,0.0054212,0.0053953,0.0053759,0.0053617,0.0053156,0.0052524,0.0052229,0.0051838,0.0051552,0.0051126,0.0050844,0.0050328,0.0050018,0.0049547,0.0049153,0.0048933,0.0048829,0.0048462,0.0047907,0.0047772,0.0047331,0.0046841,0.0046555,0.0046212,0.0045756,0.0045605,0.004519,0.0044787,0.0044572,0.0044246,0.0043599,0.0043274,0.0042904,0.0042596,0.0042243,0.0042173,0.0041909,0.0041645,0.0041456,0.0041253,0.0041014,0.0040738,0.0040607,0.0040381,0.0039941,0.0039654,0.0039435,0.0038948,0.0038561,0.0038528,0.0038362,0.0038302,0.0038154,0.0037861,0.0037319,0.0037216,0.0036812,0.0036762,0.0036519,0.0036733,0.0036527,0.003665,0.0036275,0.0036037,0.0035597,0.0035074,0.0034496,0.0034438,0.0034284,0.0034162,0.0033887,0.0033695,0.0033331,0.0033296,0.0033284,0.003324,0.0033263,0.0033173,0.0033136,0.0032743,0.0032641,0.0032576,0.0032639,0.0032379,0.0032131,0.0032115,0.0032011,0.0031629,0.003157,0.003167,0.0031439,0.0031279,0.0031404,0.0031463,0.0031263,0.0031271,0.0031218,0.0031072,0.0030783,0.003079,0.0030795,0.0030293,0.0030205,0.003018,0.0030139,0.0029939,0.0030126,0.002984,0.002955,0.0029385,0.002941,0.0029336,0.0029472,0.0029611,0.0029501,0.0029282,0.0029253,0.0029198,0.0029037,0.0028919,0.0028743,0.0028676,0.0028389,0.0028239,0.0028256,0.0028012,0.0027934,0.0028026,0.0027949,0.0027624,0.0027456,0.0027188,0.0026907,0.0027218,0.0027447,0.0027621,0.0027622,0.0027829,0.0027506,0.002706,0.0026867,0.0026718,0.0026493,0.0026325,0.0026539,0.002679,0.0026776,0.0026365,0.0026378,0.0026296,0.0026345,0.0026416,0.0026691,0.0026579,0.0026629,0.0026454,0.0026508,0.0026468,0.0026436,0.0026405,0.002634,0.0026376,0.0026671,0.0026603,0.0026442,0.0026333,0.0025967,0.0025542,0.002552,0.0025582,0.0025576,0.0025652,0.0025761,0.0025986,0.0025968,0.002611,0.0026043,0.0026227,0.0026145,0.0025967,0.0025969,0.0025763,0.0025222,0.0024976,0.0024821,0.0024569,0.0024517,0.0024384,0.0024444,0.0024356,0.0024171,0.0024083,0.0023795,0.0023464,0.0023391,0.0023202,0.0023018,0.0023227,0.0023036,0.0022844,0.0022773,0.0022979,0.002298,0.0022976,0.002285,0.002285,0.0022357,0.0022089,0.0022026,0.0022229,0.0021813,0.0022017,0.002206,0.0022075,0.0022035,0.0021969,0.0021614,0.0021856,0.0021484,0.0020999,0.0020955,0.0021015,0.0020731,0.0020656,0.0020562,0.0020721,0.0020566,0.0020425,0.0020374,0.0020465,0.0020195,0.0020113,0.0019832,0.001972,0.0019231,0.0019105,0.0018928,0.0018455,0.0018411,0.0018457,0.0018249,0.0018469,0.0019119,0.0019001,0.0019345,0.0019726,0.0019549,0.0019183,0.0019561,0.0019368,0.001915,0.0019096,0.0019132,0.0019217,0.0018933,0.001896,0.0018782,0.0018918,0.0018302,0.0018225,0.0017768,0.0017428,0.0017324,0.0017704,0.0017985,0.0018725,0.0019186,0.0018821,0.0018681,0.0018485,0.0017693,0.0017356,0.0017304,0.0016971,0.0016615,0.0017104,0.0016879,0.0016706,0.0016587,0.0016006,0.0015595,0.0016184,0.0016259,0.0016707,0.001826,0.0017222,0.0016647,0.001654,0.0015653,0.0014034,0.0014942,0.001498,0.0014578,0.0014518,0.0015323,0.0014867,0.0014953,0.0015666,0.0016306,0.0017499],"type":"scatter","mode":"lines","name":"row 14","hoverinfo":"title","line":{"fillcolor":"rgba(140,86,75,1)","color":"rgba(140,86,75,1)"},"xaxis":"x","yaxis":"y"},{"x":[300,301,302,303,304,305,306,307,308,309,310,311,312,313,314,315,316,317,318,319,320,321,322,323,324,325,326,327,328,329,330,331,332,333,334,335,336,337,338,339,340,341,342,343,344,345,346,347,348,349,350,351,352,353,354,355,356,357,358,359,360,361,362,363,364,365,366,367,368,369,370,371,372,373,374,375,376,377,378,379,380,381,382,383,384,385,386,387,388,389,390,391,392,393,394,395,396,397,398,399,400,401,402,403,404,405,406,407,408,409,410,411,412,413,414,415,416,417,418,419,420,421,422,423,424,425,426,427,428,429,430,431,432,433,434,435,436,437,438,439,440,441,442,443,444,445,446,447,448,449,450,451,452,453,454,455,456,457,458,459,460,461,462,463,464,465,466,467,468,469,470,471,472,473,474,475,476,477,478,479,480,481,482,483,484,485,486,487,488,489,490,491,492,493,494,495,496,497,498,499,500,501,502,503,504,505,506,507,508,509,510,511,512,513,514,515,516,517,518,519,520,521,522,523,524,525,526,527,528,529,530,531,532,533,534,535,536,537,538,539,540,541,542,543,544,545,546,547,548,549,550,551,552,553,554,555,556,557,558,559,560,561,562,563,564,565,566,567,568,569,570,571,572,573,574,575,576,577,578,579,580,581,582,583,584,585,586,587,588,589,590,591,592,593,594,595,596,597,598,599,600,601,602,603,604,605,606,607,608,609,610,611,612,613,614,615,616,617,618,619,620,621,622,623,624,625,626,627,628,629,630,631,632,633,634,635,636,637,638,639,640,641,642,643,644,645,646,647,648,649,650,651,652,653,654,655,656,657,658,659,660,661,662,663,664,665,666,667,668,669,670,671,672,673,674,675,676,677,678,679,680,681,682,683,684,685,686,687,688,689,690,691,692,693,694,695,696,697,698,699,700,701,702,703,704,705,706,707,708,709,710,711,712,713,714,715,716,717,718,719,720,721,722,723,724,725,726,727,728,729,730,731,732,733,734,735,736,737,738,739,740,741,742,743,744,745,746,747,748,749,750,751,752,753,754,755,756,757,758,759,760,761,762,763,764,765,766,767,768,769,770,771,772,773,774,775,776,777,778,779,780,781,782,783,784,785,786,787,788,789,790,791,792,793,794,795,796,797,798,799,800],"y":[0.0106452,0.010746,0.0104961,0.0103228,0.0101665,0.0100751,0.0099194,0.0098762,0.0098564,0.0097328,0.0096786,0.009638,0.0095527,0.0094998,0.0094989,0.0093651,0.0092021,0.00911,0.0090685,0.0090149,0.0090436,0.0090142,0.0090275,0.0089905,0.0090446,0.0090007,0.0089751,0.0089594,0.0088351,0.0087139,0.0086831,0.0086551,0.0086127,0.0086485,0.0085578,0.0083856,0.0083696,0.0082579,0.0082021,0.0082145,0.0082842,0.0082601,0.0083183,0.0082466,0.0081791,0.0080946,0.0080163,0.0079093,0.0079401,0.0079639,0.0079905,0.0080206,0.0080818,0.0080019,0.0079145,0.0078787,0.0078156,0.0077511,0.007749,0.0077357,0.0076914,0.0076753,0.0076353,0.0075666,0.0075439,0.0074854,0.0073721,0.0073293,0.0072912,0.007261,0.0071767,0.0071528,0.0071152,0.007096,0.0071144,0.0071027,0.007038,0.0070373,0.0069186,0.0067819,0.006736,0.0068059,0.0067039,0.0067052,0.0066929,0.0066757,0.0065691,0.0065135,0.0065875,0.0064923,0.0064552,0.006448,0.0064588,0.0063708,0.0063578,0.0063082,0.0063145,0.0062038,0.0061155,0.0060844,0.0060563,0.0059642,0.0060142,0.005989,0.0059357,0.0058894,0.0058442,0.0057369,0.0056712,0.0056799,0.0056428,0.0055888,0.0055878,0.0055917,0.0055247,0.0054789,0.0054656,0.0054299,0.0054006,0.0053468,0.0053235,0.0052535,0.005158,0.0050635,0.0050254,0.0049134,0.0048607,0.0048133,0.0047728,0.0047404,0.004735,0.0047204,0.0047126,0.0046896,0.0046893,0.0046798,0.004649,0.004623,0.0045829,0.0045066,0.0044739,0.0044238,0.0044091,0.0043939,0.004392,0.0043332,0.0043268,0.004255,0.0041765,0.0041214,0.0040965,0.0040392,0.0040416,0.0040657,0.0040245,0.0040362,0.0040217,0.0039857,0.0039329,0.0039451,0.0038948,0.0039164,0.0039244,0.0039278,0.0038926,0.0038797,0.0038388,0.0037738,0.0037614,0.0037364,0.0036951,0.0036766,0.0037007,0.0036437,0.0036049,0.0035952,0.0035473,0.0034939,0.0034918,0.0034698,0.0034336,0.0034195,0.0033867,0.0033567,0.0033538,0.0033594,0.0033312,0.0032753,0.0031714,0.003049,0.0029506,0.0028306,0.0027518,0.0027185,0.0026738,0.0026043,0.0025994,0.0026131,0.0026241,0.0026462,0.0026733,0.0026776,0.0026563,0.0026499,0.002648,0.0026236,0.0026032,0.0025989,0.0025994,0.0025939,0.0026013,0.0026089,0.0025928,0.0025661,0.0025466,0.0025422,0.0025288,0.0025217,0.0025176,0.0024692,0.0024495,0.0024227,0.0023928,0.0023522,0.0023324,0.0022968,0.002277,0.0022701,0.00226,0.0022812,0.0022619,0.002244,0.0022347,0.0022185,0.0021732,0.0021651,0.0021546,0.002107,0.0021033,0.0021276,0.0021194,0.0020859,0.0020949,0.0020939,0.0020786,0.0020835,0.0021061,0.0021224,0.0021226,0.0021155,0.0020996,0.0020827,0.0020593,0.0020267,0.0019977,0.0019847,0.0019538,0.0019348,0.0019521,0.0019535,0.0019536,0.0019675,0.0019765,0.0019454,0.0019626,0.0019662,0.001953,0.0019676,0.0019884,0.0019714,0.0019611,0.0019797,0.0019623,0.0019531,0.0019234,0.0019086,0.001879,0.0018799,0.0018721,0.001896,0.0019081,0.0019003,0.0019019,0.0019008,0.0018886,0.0018813,0.0018757,0.0018452,0.0018471,0.0018258,0.0018183,0.0018529,0.0018699,0.0018175,0.0018222,0.0018164,0.0017819,0.0017684,0.0018172,0.0018232,0.0018067,0.0017946,0.0017736,0.001733,0.0017154,0.0017498,0.0017612,0.0017509,0.0017625,0.0017704,0.0017015,0.0016935,0.0017214,0.0016989,0.0017068,0.0017447,0.0017354,0.0017227,0.0017316,0.0017337,0.0017158,0.0017113,0.0017284,0.0017448,0.0017205,0.0017114,0.0017386,0.0016951,0.0016864,0.0016721,0.0016966,0.0016638,0.001677,0.0016847,0.0016759,0.0016815,0.0016876,0.0016386,0.0016175,0.0016068,0.0015942,0.0016143,0.0016613,0.0016866,0.0017319,0.0017646,0.0017819,0.0018267,0.0018572,0.001882,0.0018889,0.0018999,0.001896,0.0018819,0.0018991,0.0018893,0.0018767,0.0018762,0.0018456,0.0017903,0.0018053,0.0018309,0.0018112,0.0018486,0.0018642,0.0018573,0.0018434,0.0018415,0.0018448,0.001869,0.0018447,0.0018098,0.0018196,0.0018183,0.001776,0.0017949,0.0018234,0.0018655,0.0017803,0.0017431,0.0016865,0.00162,0.0015525,0.0015577,0.0016223,0.0016352,0.0016329,0.0016049,0.0016113,0.0015356,0.0014963,0.0014933,0.0014539,0.0014134,0.0014291,0.001424,0.0014207,0.0014603,0.0014608,0.0014417,0.0014732,0.0014809,0.0014223,0.0014425,0.0014241,0.0014074,0.0013975,0.0014735,0.0015341,0.0015613,0.0015691,0.0015777,0.0015719,0.0015306,0.0015703,0.0015796,0.0015596,0.0015585,0.0014864,0.0014956,0.0014761,0.0015164,0.0015382,0.0016585,0.0016308,0.0016834,0.0016646,0.001697,0.0017081,0.0017646,0.0017688,0.0017909,0.0018241,0.0018486,0.0017993,0.0017468,0.0017442,0.001648,0.0015345,0.0015344,0.0015742,0.0015256,0.0015228,0.0015216,0.0014634,0.0014738,0.0015572,0.0016222,0.0016591,0.0017138,0.0016707,0.0016472,0.001631,0.0016183,0.0016786,0.0016362,0.001632,0.0015391,0.0015743,0.0015155,0.0015958,0.0016242,0.0017722,0.0017606,0.0017554,0.0016802,0.0016229,0.0016028,0.0016601,0.0015974,0.0016276,0.0017377,0.0016266,0.0014787,0.0015289,0.0014851,0.00145,0.001539,0.0016305,0.0016764,0.0017265,0.0017656,0.0017677,0.0017376,0.0017253,0.0017242,0.0017007,0.0015916,0.0016391,0.0016263,0.0017079,0.0015908,0.0017625,0.0017883,0.0017279,0.0015616,0.0015301,0.0014309,0.0013616,0.0016758,0.001488],"type":"scatter","mode":"lines","name":"row 17","hoverinfo":"title","line":{"fillcolor":"rgba(227,119,194,1)","color":"rgba(227,119,194,1)"},"xaxis":"x","yaxis":"y"},{"x":[300,301,302,303,304,305,306,307,308,309,310,311,312,313,314,315,316,317,318,319,320,321,322,323,324,325,326,327,328,329,330,331,332,333,334,335,336,337,338,339,340,341,342,343,344,345,346,347,348,349,350,351,352,353,354,355,356,357,358,359,360,361,362,363,364,365,366,367,368,369,370,371,372,373,374,375,376,377,378,379,380,381,382,383,384,385,386,387,388,389,390,391,392,393,394,395,396,397,398,399,400,401,402,403,404,405,406,407,408,409,410,411,412,413,414,415,416,417,418,419,420,421,422,423,424,425,426,427,428,429,430,431,432,433,434,435,436,437,438,439,440,441,442,443,444,445,446,447,448,449,450,451,452,453,454,455,456,457,458,459,460,461,462,463,464,465,466,467,468,469,470,471,472,473,474,475,476,477,478,479,480,481,482,483,484,485,486,487,488,489,490,491,492,493,494,495,496,497,498,499,500,501,502,503,504,505,506,507,508,509,510,511,512,513,514,515,516,517,518,519,520,521,522,523,524,525,526,527,528,529,530,531,532,533,534,535,536,537,538,539,540,541,542,543,544,545,546,547,548,549,550,551,552,553,554,555,556,557,558,559,560,561,562,563,564,565,566,567,568,569,570,571,572,573,574,575,576,577,578,579,580,581,582,583,584,585,586,587,588,589,590,591,592,593,594,595,596,597,598,599,600,601,602,603,604,605,606,607,608,609,610,611,612,613,614,615,616,617,618,619,620,621,622,623,624,625,626,627,628,629,630,631,632,633,634,635,636,637,638,639,640,641,642,643,644,645,646,647,648,649,650,651,652,653,654,655,656,657,658,659,660,661,662,663,664,665,666,667,668,669,670,671,672,673,674,675,676,677,678,679,680,681,682,683,684,685,686,687,688,689,690,691,692,693,694,695,696,697,698,699,700,701,702,703,704,705,706,707,708,709,710,711,712,713,714,715,716,717,718,719,720,721,722,723,724,725,726,727,728,729,730,731,732,733,734,735,736,737,738,739,740,741,742,743,744,745,746,747,748,749,750,751,752,753,754,755,756,757,758,759,760,761,762,763,764,765,766,767,768,769,770,771,772,773,774,775,776,777,778,779,780,781,782,783,784,785,786,787,788,789,790,791,792,793,794,795,796,797,798,799,800],"y":[0.0097746,0.0095537,0.0094573,0.0093135,0.0092095,0.0090942,0.0089963,0.0089567,0.0089198,0.00884,0.0087907,0.008747,0.0086414,0.0085923,0.0085339,0.0084617,0.0083952,0.0083642,0.0083984,0.0084815,0.0085973,0.0086194,0.0086654,0.0086119,0.0085516,0.0084981,0.0084703,0.0084216,0.008408,0.0084003,0.0083095,0.0082916,0.0082796,0.0081857,0.0080924,0.0080083,0.0079051,0.0077881,0.0077468,0.0077031,0.0077069,0.0077119,0.0076886,0.007639,0.0075532,0.0074276,0.0073333,0.0072717,0.0071936,0.0071519,0.0071701,0.0071167,0.0070777,0.0070495,0.0069969,0.0069408,0.0069022,0.0068352,0.0067962,0.0067984,0.0067892,0.0067336,0.0066845,0.0066041,0.0064827,0.0063535,0.0062907,0.0061815,0.0061239,0.0060712,0.0060293,0.0059859,0.0059816,0.005849,0.0057488,0.00567,0.0054861,0.0054743,0.0055281,0.00556,0.0055678,0.005673,0.0056204,0.0055687,0.0055448,0.0055109,0.0054886,0.0054521,0.0054588,0.0054028,0.0053473,0.0053077,0.0052838,0.0052189,0.0051797,0.0051414,0.0051281,0.0050804,0.0050018,0.0049746,0.0049624,0.0048794,0.0048575,0.0048602,0.004815,0.0047639,0.0047465,0.0046803,0.004625,0.0046329,0.0046205,0.0045594,0.0045566,0.0045452,0.0044917,0.0044603,0.0044784,0.004446,0.0044235,0.0043856,0.0043475,0.004286,0.0042311,0.004187,0.0041559,0.0041175,0.0040992,0.0040697,0.0040347,0.0040147,0.0039914,0.0039732,0.00397,0.0039293,0.0039268,0.0039177,0.0038984,0.0038524,0.0038527,0.0037862,0.0037535,0.0037075,0.0036853,0.003659,0.0036464,0.0035953,0.0035785,0.0035478,0.0035103,0.0034834,0.0034599,0.0034222,0.0034175,0.0034052,0.0033785,0.0033732,0.0033708,0.0033355,0.0033059,0.0033004,0.0032823,0.0032662,0.0032611,0.0032445,0.0032218,0.0031966,0.0031742,0.0031499,0.0031412,0.0031201,0.003095,0.0030798,0.0030715,0.003041,0.0030147,0.0030067,0.0029712,0.0029442,0.0029274,0.0029108,0.0028822,0.0028563,0.0028376,0.002819,0.0028137,0.0028116,0.0028084,0.0027998,0.0028041,0.0027865,0.002777,0.0027669,0.0027443,0.0027158,0.0026933,0.0026613,0.0026396,0.0026422,0.00263,0.0026104,0.0025947,0.0025605,0.0025168,0.0024934,0.0024762,0.0024562,0.0024455,0.0024324,0.0024165,0.0024062,0.0023889,0.0023803,0.002367,0.0023587,0.0023456,0.0023427,0.0023274,0.0023108,0.0022947,0.0022577,0.0022358,0.0022202,0.0022098,0.002189,0.0021912,0.0021844,0.0021661,0.0021668,0.0021627,0.0021572,0.0021269,0.0021179,0.0020872,0.0020641,0.0020317,0.0020342,0.0020249,0.0020018,0.0019975,0.0020039,0.001991,0.0019889,0.0019966,0.0019875,0.0019821,0.0019722,0.0019638,0.001966,0.0019772,0.0019663,0.0019536,0.0019344,0.0019226,0.0018954,0.0018919,0.001893,0.0018792,0.0018541,0.0018452,0.0018182,0.0017995,0.0018093,0.0018183,0.0018091,0.0018249,0.0018273,0.0018054,0.001797,0.0017857,0.0017807,0.001768,0.0017851,0.0017824,0.0017903,0.0017759,0.0017869,0.0017671,0.0017568,0.0017487,0.001743,0.0017246,0.0017108,0.0017095,0.0017081,0.0016934,0.0016931,0.0016933,0.0016795,0.0016813,0.0016764,0.0016791,0.0016882,0.0016937,0.0016715,0.0016781,0.0016637,0.0016489,0.0016399,0.0016391,0.0016363,0.0016271,0.0016147,0.0016029,0.0015994,0.0015793,0.0015943,0.0015967,0.0015858,0.0015862,0.0015955,0.0015596,0.0015531,0.0015642,0.0015478,0.0015464,0.0015505,0.0015492,0.0015458,0.0015436,0.0015377,0.0015363,0.0015267,0.0015343,0.0015493,0.0015502,0.0015564,0.0015607,0.0015384,0.0015259,0.0015044,0.0014942,0.0015057,0.0015083,0.0015201,0.0015306,0.0015403,0.0015225,0.0015076,0.0014926,0.0014786,0.0014681,0.001468,0.0014784,0.0014716,0.0014616,0.0014582,0.0014646,0.0014523,0.0014498,0.001463,0.0014597,0.0014541,0.001457,0.0014468,0.0014511,0.0014615,0.0014481,0.0014473,0.0014433,0.001415,0.0014134,0.0014204,0.001422,0.0014353,0.0014375,0.0014249,0.00142,0.0013986,0.0013822,0.0013929,0.0013864,0.001388,0.0013913,0.0013879,0.0013649,0.001345,0.0013367,0.00134,0.0013241,0.0013159,0.001313,0.0013004,0.001308,0.0013138,0.0013375,0.0013509,0.0013316,0.0012831,0.0012703,0.0012554,0.0012258,0.0012315,0.0012324,0.0012367,0.0012526,0.0012594,0.0012399,0.0012504,0.0012372,0.0011998,0.0011938,0.0012125,0.0012048,0.0012111,0.0012235,0.0012224,0.0012065,0.0012057,0.0011895,0.0011733,0.0011561,0.0011464,0.0011544,0.0011678,0.0011922,0.0012101,0.0012221,0.001244,0.0012251,0.0012401,0.0012384,0.0012448,0.0012171,0.0012236,0.0011971,0.0011959,0.0011594,0.0011836,0.0011917,0.0011577,0.001165,0.0011867,0.0011854,0.0011839,0.0012109,0.0011931,0.0011806,0.001142,0.0011257,0.0011172,0.0011416,0.0011263,0.0011158,0.0010913,0.001065,0.0010362,0.0010607,0.0010639,0.0010602,0.0010994,0.0010843,0.0010979,0.0011051,0.0010949,0.0010569,0.0010614,0.001042,0.0010193,0.0010598,0.0010414,0.0010635,0.0010686,0.0011053,0.001112,0.0011424,0.0011273,0.0011041,0.0011082,0.0010939,0.0010681,0.0010611,0.0011205,0.0010668,0.0009997,0.0010255,0.0010092,0.0009907,0.0010587,0.0011723,0.0011758,0.0012097,0.0012101,0.0012026,0.0011708,0.0011211,0.0011365,0.0011059,0.0010883,0.0010656,0.0010815,0.0010873,0.0011314,0.0011319,0.0011323,0.0011098,0.0010982,0.0010923,0.0010608,0.0010528,0.0009956,0.0010027],"type":"scatter","mode":"lines","name":"row 20","hoverinfo":"title","line":{"fillcolor":"rgba(127,127,127,1)","color":"rgba(127,127,127,1)"},"xaxis":"x","yaxis":"y"},{"x":[300,301,302,303,304,305,306,307,308,309,310,311,312,313,314,315,316,317,318,319,320,321,322,323,324,325,326,327,328,329,330,331,332,333,334,335,336,337,338,339,340,341,342,343,344,345,346,347,348,349,350,351,352,353,354,355,356,357,358,359,360,361,362,363,364,365,366,367,368,369,370,371,372,373,374,375,376,377,378,379,380,381,382,383,384,385,386,387,388,389,390,391,392,393,394,395,396,397,398,399,400,401,402,403,404,405,406,407,408,409,410,411,412,413,414,415,416,417,418,419,420,421,422,423,424,425,426,427,428,429,430,431,432,433,434,435,436,437,438,439,440,441,442,443,444,445,446,447,448,449,450,451,452,453,454,455,456,457,458,459,460,461,462,463,464,465,466,467,468,469,470,471,472,473,474,475,476,477,478,479,480,481,482,483,484,485,486,487,488,489,490,491,492,493,494,495,496,497,498,499,500,501,502,503,504,505,506,507,508,509,510,511,512,513,514,515,516,517,518,519,520,521,522,523,524,525,526,527,528,529,530,531,532,533,534,535,536,537,538,539,540,541,542,543,544,545,546,547,548,549,550,551,552,553,554,555,556,557,558,559,560,561,562,563,564,565,566,567,568,569,570,571,572,573,574,575,576,577,578,579,580,581,582,583,584,585,586,587,588,589,590,591,592,593,594,595,596,597,598,599,600,601,602,603,604,605,606,607,608,609,610,611,612,613,614,615,616,617,618,619,620,621,622,623,624,625,626,627,628,629,630,631,632,633,634,635,636,637,638,639,640,641,642,643,644,645,646,647,648,649,650,651,652,653,654,655,656,657,658,659,660,661,662,663,664,665,666,667,668,669,670,671,672,673,674,675,676,677,678,679,680,681,682,683,684,685,686,687,688,689,690,691,692,693,694,695,696,697,698,699,700,701,702,703,704,705,706,707,708,709,710,711,712,713,714,715,716,717,718,719,720,721,722,723,724,725,726,727,728,729,730,731,732,733,734,735,736,737,738,739,740,741,742,743,744,745,746,747,748,749,750,751,752,753,754,755,756,757,758,759,760,761,762,763,764,765,766,767,768,769,770,771,772,773,774,775,776,777,778,779,780,781,782,783,784,785,786,787,788,789,790,791,792,793,794,795,796,797,798,799,800],"y":[0.0085663,0.0081814,0.0080075,0.0077712,0.0076413,0.0075226,0.0074332,0.0073825,0.0073475,0.0072613,0.0071742,0.0071047,0.0070283,0.0069815,0.0069543,0.0069164,0.0068592,0.0067954,0.0068151,0.0068202,0.0068249,0.0067798,0.0068216,0.0067054,0.0066729,0.00666,0.0066741,0.0066368,0.0066277,0.0065783,0.00653,0.0064726,0.0064225,0.0064219,0.0063863,0.0063623,0.0063472,0.0062719,0.0061462,0.0061125,0.0060507,0.005994,0.0060006,0.005983,0.0059245,0.0058405,0.0057924,0.0057075,0.0057003,0.0056165,0.0056602,0.0056114,0.0056182,0.0055645,0.005545,0.0054721,0.0055033,0.0054251,0.0054142,0.0053875,0.0053133,0.0052086,0.0051877,0.0051312,0.0050577,0.0050631,0.0050689,0.0050145,0.0049536,0.0050065,0.0049137,0.0048239,0.0048274,0.004738,0.004624,0.004586,0.0045581,0.0045073,0.0045826,0.0045845,0.0045773,0.004549,0.004467,0.0043753,0.0043208,0.0043386,0.0043339,0.0043358,0.0043807,0.0043091,0.0042484,0.004209,0.0042064,0.0040936,0.0040812,0.0040081,0.0039858,0.0038803,0.0038216,0.0037869,0.0037742,0.0036887,0.0037019,0.003708,0.0036657,0.0036216,0.0036116,0.0035642,0.0035296,0.0035489,0.0035231,0.0034708,0.0034583,0.0034404,0.0033844,0.0033586,0.0033719,0.0033244,0.0033042,0.003272,0.0032523,0.0031729,0.0031227,0.0030658,0.0030187,0.0029821,0.0029773,0.0029546,0.0029327,0.0029424,0.0029204,0.0029004,0.0029129,0.0028976,0.0028911,0.0028856,0.0028863,0.00286,0.002849,0.0027908,0.0027676,0.0027285,0.0027147,0.0026937,0.002704,0.0026802,0.0026823,0.0026659,0.0026411,0.00262,0.002606,0.002581,0.0025584,0.0025657,0.002552,0.0025661,0.0025505,0.0025408,0.0025204,0.0025269,0.0025014,0.0025152,0.0025204,0.0025188,0.0024946,0.0024749,0.0024571,0.0024403,0.0024451,0.0024276,0.0024174,0.0024172,0.002429,0.0023971,0.0023911,0.00239,0.0023686,0.0023502,0.0023433,0.0023227,0.0022866,0.0022725,0.0022438,0.0022308,0.0022407,0.0022706,0.0022583,0.0022606,0.0022527,0.0022294,0.0022107,0.0021892,0.0021678,0.0021584,0.0021389,0.0021306,0.0021484,0.0021697,0.0021777,0.0021956,0.0021717,0.0021397,0.0021145,0.002101,0.0020776,0.0020797,0.0020851,0.0020686,0.0020524,0.0020479,0.0020394,0.002037,0.002028,0.0020174,0.0020016,0.0020016,0.0019768,0.0019804,0.0019726,0.0019528,0.0019338,0.0019393,0.0019223,0.0019109,0.0019114,0.0019088,0.0018891,0.001883,0.0018741,0.0018747,0.0018389,0.0018326,0.0018233,0.0018191,0.001795,0.0018066,0.0018005,0.001769,0.0017604,0.0017662,0.0017539,0.0017369,0.0017548,0.0017602,0.0017534,0.0017507,0.0017662,0.0017714,0.0017654,0.0017624,0.0017508,0.0017274,0.0017208,0.0017169,0.0017044,0.001709,0.0016963,0.0016784,0.0016551,0.0016515,0.0016354,0.0016278,0.0016277,0.0016362,0.0016282,0.0016244,0.0016224,0.0016062,0.0015845,0.0015778,0.0015661,0.0015726,0.0015832,0.001583,0.0015747,0.0015735,0.0015706,0.0015673,0.0015636,0.0015608,0.0015544,0.0015334,0.0015204,0.0015145,0.0015041,0.0015037,0.0014953,0.0014656,0.001465,0.0014624,0.0014539,0.0014816,0.0015202,0.0015117,0.001513,0.0015067,0.0014715,0.0014399,0.001423,0.0014187,0.0014163,0.0014143,0.0014009,0.0013971,0.0013908,0.0014069,0.0014018,0.0014039,0.0014224,0.001431,0.0014026,0.0014137,0.0014309,0.0014038,0.0013937,0.0013966,0.0013967,0.0013723,0.0013766,0.0013683,0.0013596,0.0013252,0.0013264,0.0013174,0.0012948,0.0013096,0.0013228,0.0013198,0.0013431,0.0013546,0.001351,0.0013724,0.0013831,0.001375,0.0013716,0.0013781,0.0013514,0.0013259,0.0013307,0.0013347,0.0013064,0.001315,0.0013212,0.0013047,0.0012835,0.0013089,0.0013055,0.0013092,0.0013034,0.001344,0.0013227,0.0013297,0.0013543,0.0013532,0.0013323,0.0013625,0.0013498,0.0013271,0.0013233,0.0013045,0.0012744,0.0012979,0.0012869,0.0012948,0.0012878,0.0012979,0.001294,0.0012895,0.0013038,0.0013378,0.0013213,0.0012937,0.0012789,0.0012438,0.0012007,0.0011602,0.001137,0.00113,0.0011078,0.0011036,0.0011125,0.0011008,0.0011191,0.0011212,0.0011351,0.0011515,0.0011517,0.0011041,0.001082,0.0010775,0.0010489,0.0010419,0.0010578,0.001065,0.0010716,0.0010715,0.001038,0.001027,0.0010106,0.0009808,0.0009923,0.0010376,0.0010515,0.001078,0.0010686,0.0010753,0.001076,0.0010624,0.0010527,0.0010622,0.0010291,0.0010203,0.0010528,0.0010681,0.0010581,0.0010557,0.001042,0.0010565,0.0010312,0.0010485,0.001062,0.0010788,0.0010553,0.001086,0.0010776,0.0010563,0.0010149,0.0010265,0.0010042,0.0010134,0.0010372,0.0010313,0.0010368,0.0010664,0.0010744,0.0010379,0.0011021,0.0010824,0.0010213,0.0010247,0.0010947,0.001056,0.0010499,0.0010809,0.0010075,0.0009602,0.0009807,0.0010035,0.0010527,0.0011508,0.0011738,0.0011563,0.0011239,0.0010864,0.0010168,0.0010283,0.0010411,0.0010524,0.0010442,0.0010358,0.0010495,0.0010143,0.0009981,0.0009751,0.0009868,0.0009368,0.0010029,0.0010323,0.0011158,0.001136,0.0011365,0.0011748,0.0011649,0.0010688,0.0010884,0.0010906,0.0010977,0.0012233,0.0012415,0.0012251,0.0012655,0.0012488,0.001156,0.0011521,0.0011337,0.0011624,0.0011018,0.0010144,0.0009732,0.000981,0.0009874,0.0009478,0.001015,0.0011276,0.0011227,0.000984,0.0010141,0.0009497,0.0008574,0.0008612,0.0008236],"type":"scatter","mode":"lines","name":"row 23","hoverinfo":"title","line":{"fillcolor":"rgba(188,189,34,1)","color":"rgba(188,189,34,1)"},"xaxis":"x","yaxis":"y"},{"x":[300,301,302,303,304,305,306,307,308,309,310,311,312,313,314,315,316,317,318,319,320,321,322,323,324,325,326,327,328,329,330,331,332,333,334,335,336,337,338,339,340,341,342,343,344,345,346,347,348,349,350,351,352,353,354,355,356,357,358,359,360,361,362,363,364,365,366,367,368,369,370,371,372,373,374,375,376,377,378,379,380,381,382,383,384,385,386,387,388,389,390,391,392,393,394,395,396,397,398,399,400,401,402,403,404,405,406,407,408,409,410,411,412,413,414,415,416,417,418,419,420,421,422,423,424,425,426,427,428,429,430,431,432,433,434,435,436,437,438,439,440,441,442,443,444,445,446,447,448,449,450,451,452,453,454,455,456,457,458,459,460,461,462,463,464,465,466,467,468,469,470,471,472,473,474,475,476,477,478,479,480,481,482,483,484,485,486,487,488,489,490,491,492,493,494,495,496,497,498,499,500,501,502,503,504,505,506,507,508,509,510,511,512,513,514,515,516,517,518,519,520,521,522,523,524,525,526,527,528,529,530,531,532,533,534,535,536,537,538,539,540,541,542,543,544,545,546,547,548,549,550,551,552,553,554,555,556,557,558,559,560,561,562,563,564,565,566,567,568,569,570,571,572,573,574,575,576,577,578,579,580,581,582,583,584,585,586,587,588,589,590,591,592,593,594,595,596,597,598,599,600,601,602,603,604,605,606,607,608,609,610,611,612,613,614,615,616,617,618,619,620,621,622,623,624,625,626,627,628,629,630,631,632,633,634,635,636,637,638,639,640,641,642,643,644,645,646,647,648,649,650,651,652,653,654,655,656,657,658,659,660,661,662,663,664,665,666,667,668,669,670,671,672,673,674,675,676,677,678,679,680,681,682,683,684,685,686,687,688,689,690,691,692,693,694,695,696,697,698,699,700,701,702,703,704,705,706,707,708,709,710,711,712,713,714,715,716,717,718,719,720,721,722,723,724,725,726,727,728,729,730,731,732,733,734,735,736,737,738,739,740,741,742,743,744,745,746,747,748,749,750,751,752,753,754,755,756,757,758,759,760,761,762,763,764,765,766,767,768,769,770,771,772,773,774,775,776,777,778,779,780,781,782,783,784,785,786,787,788,789,790,791,792,793,794,795,796,797,798,799,800],"y":[0.0169739,0.0168239,0.0166945,0.0165033,0.0163898,0.0162803,0.0161922,0.0161589,0.0161626,0.0160561,0.0160076,0.015901,0.0157909,0.0157248,0.0156577,0.0155568,0.0154469,0.0153921,0.0153534,0.0153711,0.0153677,0.0153932,0.0153528,0.0152524,0.0152071,0.0151113,0.0149721,0.0148806,0.0148027,0.0147296,0.0146515,0.0145939,0.0145598,0.0144705,0.0143763,0.0143189,0.0142897,0.0142019,0.0141205,0.0140798,0.0140527,0.0139949,0.0139358,0.0139372,0.0138562,0.0137507,0.0137113,0.0136189,0.0136058,0.0135696,0.013638,0.0136005,0.0136517,0.0135878,0.0135845,0.0134397,0.013373,0.013303,0.0132383,0.0131469,0.01313,0.013074,0.0130254,0.0129572,0.0128514,0.0127625,0.0126845,0.0125787,0.0124832,0.0123748,0.0122707,0.0121044,0.0120331,0.0118634,0.0117698,0.0116042,0.0115449,0.0115075,0.0115809,0.0115865,0.011683,0.0117498,0.01169,0.0116426,0.0115708,0.0114721,0.0113865,0.0113047,0.0112592,0.0112061,0.0111374,0.0110718,0.0110112,0.0109229,0.0108601,0.0108027,0.0107859,0.0107206,0.0105988,0.0105652,0.0105174,0.0104249,0.010379,0.0103691,0.0102688,0.0101978,0.0101297,0.0100298,0.0099502,0.0099371,0.0098689,0.0097953,0.0097587,0.0096812,0.0095589,0.0095034,0.0094498,0.0093924,0.009377,0.0093449,0.0093088,0.009211,0.009112,0.0089788,0.0088853,0.0087788,0.008707,0.0086033,0.0085473,0.0084834,0.008376,0.0083243,0.0082741,0.0081975,0.0081427,0.0080967,0.0080134,0.0079378,0.0078734,0.0077751,0.0077101,0.0076427,0.0076001,0.0075511,0.007493,0.0074203,0.0073463,0.0072646,0.0071634,0.0070968,0.0070189,0.0069503,0.006887,0.0068393,0.0067741,0.0067442,0.0067141,0.0066622,0.0066255,0.0065913,0.0065186,0.006471,0.0064256,0.0063749,0.0063247,0.0062762,0.0062164,0.0061631,0.0061127,0.0060479,0.0059863,0.0059416,0.0059088,0.0058566,0.0058056,0.0057668,0.0057104,0.0056661,0.0056089,0.0055428,0.0054841,0.0054189,0.0053534,0.0053119,0.0053008,0.0052873,0.0052775,0.0052482,0.0052173,0.0051782,0.0051486,0.0050829,0.0050405,0.0050065,0.0049628,0.0049013,0.0048961,0.0048683,0.0048384,0.0047936,0.0047529,0.0046975,0.0046435,0.0045984,0.0045729,0.0045484,0.0045168,0.0044848,0.0044527,0.0044187,0.0043882,0.0043513,0.004309,0.0042689,0.0042303,0.0042006,0.0041726,0.0041569,0.0041408,0.004084,0.0040512,0.0040152,0.003986,0.0039347,0.0039163,0.0038799,0.0038416,0.0038096,0.0037829,0.0037673,0.0037165,0.0036965,0.0036645,0.003643,0.0036036,0.0036041,0.0035729,0.0035234,0.0034831,0.0034693,0.0034384,0.003412,0.0034048,0.0034065,0.0033864,0.003374,0.0033766,0.0033573,0.0033227,0.0032853,0.0032344,0.0031838,0.0031734,0.0031537,0.0031357,0.0031254,0.0031242,0.0030946,0.0030708,0.0030553,0.0030444,0.0030108,0.0029821,0.0029639,0.0029386,0.0029306,0.0029057,0.0029063,0.0028841,0.0028714,0.0028544,0.0028562,0.0028277,0.0028107,0.0027964,0.0027564,0.0027158,0.0027143,0.0027055,0.0026805,0.0026786,0.002677,0.0026687,0.0026496,0.0026311,0.002632,0.0026148,0.0025856,0.0025772,0.0025808,0.0025588,0.0025583,0.0025572,0.0025441,0.0025266,0.0025046,0.0024813,0.00246,0.0024464,0.0024395,0.0024269,0.0024081,0.002391,0.002359,0.0023408,0.002339,0.002335,0.0023079,0.002314,0.0023227,0.0022836,0.002281,0.0022996,0.0022946,0.0022933,0.0023095,0.002289,0.0022635,0.0022646,0.0022452,0.0022177,0.0022052,0.0022255,0.0021827,0.0021503,0.0021453,0.0021478,0.0020876,0.0020992,0.0020927,0.00209,0.0020829,0.002102,0.0021071,0.0021049,0.0021108,0.0021001,0.002096,0.0020825,0.0020694,0.0020517,0.0020648,0.0020711,0.0020545,0.0020385,0.00203,0.0020001,0.0019488,0.0019412,0.0019677,0.0019619,0.0019558,0.0019669,0.001965,0.0019407,0.0019445,0.0019355,0.0019262,0.0018858,0.0018629,0.0018655,0.0018886,0.0018929,0.0019367,0.0019576,0.0019508,0.0019538,0.0019629,0.0019874,0.0020291,0.002049,0.0020051,0.0020032,0.0019665,0.0019144,0.0018939,0.0019137,0.0019258,0.0019145,0.0018926,0.0018807,0.0018612,0.0018346,0.0018532,0.0018815,0.0018934,0.0018916,0.0018784,0.0018549,0.0018254,0.0017838,0.0017426,0.0017178,0.0017019,0.0017435,0.0017299,0.001724,0.0017379,0.0017277,0.0016738,0.0016824,0.0017114,0.0016995,0.0017235,0.0017395,0.0017248,0.0017171,0.0017086,0.0016911,0.0016755,0.0016709,0.001662,0.0016804,0.0016113,0.0015985,0.001572,0.0015428,0.0015414,0.0015772,0.0015803,0.001618,0.0016503,0.0016454,0.0016621,0.0016291,0.0015695,0.001532,0.0015163,0.0014832,0.0015029,0.0015434,0.0015316,0.001545,0.0015766,0.0015576,0.0015312,0.0015522,0.001492,0.0014482,0.0014459,0.0014634,0.0014459,0.0014509,0.0014499,0.001444,0.0014574,0.0014479,0.0014527,0.0014684,0.0014667,0.001448,0.0014673,0.0014612,0.0014545,0.0014141,0.0013762,0.0013408,0.0013255,0.0013132,0.0013482,0.0013383,0.0013383,0.0013964,0.0013718,0.0013698,0.0013877,0.0014207,0.0013984,0.0013988,0.0013521,0.0013703,0.0013832,0.0013091,0.0012779,0.001272,0.0011837,0.0011361,0.00125,0.0012606,0.0013094,0.001327,0.0013541,0.0013054,0.0013356,0.0012438,0.0013278,0.0013477,0.0012338,0.0012183,0.0012563,0.0012001,0.0011591,0.0012309,0.0012334,0.0012437,0.0011479,0.0010842,0.001064,0.0010472,0.0011401,0.0011787],"type":"scatter","mode":"lines","name":"row 26","hoverinfo":"title","line":{"fillcolor":"rgba(23,190,207,1)","color":"rgba(23,190,207,1)"},"xaxis":"x","yaxis":"y"}],"base_url":"https://plot.ly"},"evals":["config.modeBarButtonsToAdd.0.click"],"jsHooks":[]}</script><!--/html_preserve-->

The variable **myS** contains data from several casts (depth profiles). To isolate a cast, we use *spc.makeSpcList()*.

```r
BL <- spc.makeSpcList(myS, "CAST")
```

To plot the depth profile we pass the **SpcList** object we just created to *spc.plot.depth.plotly()*

```r
spc.plot.depth.plotly(BL[[4]])
```

<!--html_preserve--><div id="htmlwidget-412e8df73c1fc5d713df" style="width:480px;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-412e8df73c1fc5d713df">{"x":{"layout":{"margin":{"b":40,"l":60,"t":25,"r":10},"title":"spvar2 longname","hovermode":"closest","xaxis":{"domain":[0,1],"title":"a_nap [1/m]"},"yaxis":{"domain":[0,1],"title":"Depth [ m ]","rangeslider":{"type":"linear"},"autorange":"reversed"},"showlegend":false},"source":"A","config":{"modeBarButtonsToAdd":[{"name":"Collaborate","icon":{"width":1000,"ascent":500,"descent":-50,"path":"M487 375c7-10 9-23 5-36l-79-259c-3-12-11-23-22-31-11-8-22-12-35-12l-263 0c-15 0-29 5-43 15-13 10-23 23-28 37-5 13-5 25-1 37 0 0 0 3 1 7 1 5 1 8 1 11 0 2 0 4-1 6 0 3-1 5-1 6 1 2 2 4 3 6 1 2 2 4 4 6 2 3 4 5 5 7 5 7 9 16 13 26 4 10 7 19 9 26 0 2 0 5 0 9-1 4-1 6 0 8 0 2 2 5 4 8 3 3 5 5 5 7 4 6 8 15 12 26 4 11 7 19 7 26 1 1 0 4 0 9-1 4-1 7 0 8 1 2 3 5 6 8 4 4 6 6 6 7 4 5 8 13 13 24 4 11 7 20 7 28 1 1 0 4 0 7-1 3-1 6-1 7 0 2 1 4 3 6 1 1 3 4 5 6 2 3 3 5 5 6 1 2 3 5 4 9 2 3 3 7 5 10 1 3 2 6 4 10 2 4 4 7 6 9 2 3 4 5 7 7 3 2 7 3 11 3 3 0 8 0 13-1l0-1c7 2 12 2 14 2l218 0c14 0 25-5 32-16 8-10 10-23 6-37l-79-259c-7-22-13-37-20-43-7-7-19-10-37-10l-248 0c-5 0-9-2-11-5-2-3-2-7 0-12 4-13 18-20 41-20l264 0c5 0 10 2 16 5 5 3 8 6 10 11l85 282c2 5 2 10 2 17 7-3 13-7 17-13z m-304 0c-1-3-1-5 0-7 1-1 3-2 6-2l174 0c2 0 4 1 7 2 2 2 4 4 5 7l6 18c0 3 0 5-1 7-1 1-3 2-6 2l-173 0c-3 0-5-1-8-2-2-2-4-4-4-7z m-24-73c-1-3-1-5 0-7 2-2 3-2 6-2l174 0c2 0 5 0 7 2 3 2 4 4 5 7l6 18c1 2 0 5-1 6-1 2-3 3-5 3l-174 0c-3 0-5-1-7-3-3-1-4-4-5-6z"},"click":"function(gd) { \n        // is this being viewed in RStudio?\n        if (location.search == '?viewer_pane=1') {\n          alert('To learn about plotly for collaboration, visit:\\n https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html');\n        } else {\n          window.open('https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html', '_blank');\n        }\n      }"}],"modeBarButtonsToRemove":["sendDataToCloud"]},"data":[{"x":[0.1349536,0.2888538,0.2444953,0.0172168,0.0815227,0.0202518],"y":[48.9349976,42.8319969,39.6289978,36.8269997,18.8260002,2.0780001],"mode":"lines + markers","name":"anap_300","type":"scatter","line":{"fillcolor":"rgba(31,119,180,1)","color":"rgba(31,119,180,1)"},"xaxis":"x","yaxis":"y"},{"x":[0.0847662,0.1782516,0.1558811,0.0110504,0.0521841,0.0148532],"y":[48.9349976,42.8319969,39.6289978,36.8269997,18.8260002,2.0780001],"mode":"lines + markers","name":"anap_356","type":"scatter","hoverinfo":"name","line":{"fillcolor":"rgba(255,127,14,1)","color":"rgba(255,127,14,1)"},"xaxis":"x","yaxis":"y"},{"x":[0.057755,0.1182633,0.1068568,0.0072727,0.0323918,0.0107678],"y":[48.9349976,42.8319969,39.6289978,36.8269997,18.8260002,2.0780001],"mode":"lines + markers","name":"anap_411","type":"scatter","hoverinfo":"name","line":{"fillcolor":"rgba(44,160,44,1)","color":"rgba(44,160,44,1)"},"xaxis":"x","yaxis":"y"},{"x":[0.038202,0.0743688,0.0693906,0.0049885,0.0205498,0.0069817],"y":[48.9349976,42.8319969,39.6289978,36.8269997,18.8260002,2.0780001],"mode":"lines + markers","name":"anap_467","type":"scatter","hoverinfo":"name","line":{"fillcolor":"rgba(214,39,40,1)","color":"rgba(214,39,40,1)"},"xaxis":"x","yaxis":"y"},{"x":[0.0243987,0.0460983,0.041132,0.0037185,0.014317,0.0046841],"y":[48.9349976,42.8319969,39.6289978,36.8269997,18.8260002,2.0780001],"mode":"lines + markers","name":"anap_522","type":"scatter","hoverinfo":"name","line":{"fillcolor":"rgba(148,103,189,1)","color":"rgba(148,103,189,1)"},"xaxis":"x","yaxis":"y"},{"x":[0.0177739,0.0334135,0.0296104,0.0029692,0.0107439,0.0033173],"y":[48.9349976,42.8319969,39.6289978,36.8269997,18.8260002,2.0780001],"mode":"lines + markers","name":"anap_578","type":"scatter","hoverinfo":"name","line":{"fillcolor":"rgba(140,86,75,1)","color":"rgba(140,86,75,1)"},"xaxis":"x","yaxis":"y"},{"x":[0.0145915,0.026816,0.0247886,0.0026128,0.0089815,0.0026907],"y":[48.9349976,42.8319969,39.6289978,36.8269997,18.8260002,2.0780001],"mode":"lines + markers","name":"anap_633","type":"scatter","hoverinfo":"name","line":{"fillcolor":"rgba(227,119,194,1)","color":"rgba(227,119,194,1)"},"xaxis":"x","yaxis":"y"},{"x":[0.0122208,0.0212725,0.0198056,0.0022269,0.0076563,0.0024444],"y":[48.9349976,42.8319969,39.6289978,36.8269997,18.8260002,2.0780001],"mode":"lines + markers","name":"anap_689","type":"scatter","hoverinfo":"name","line":{"fillcolor":"rgba(127,127,127,1)","color":"rgba(127,127,127,1)"},"xaxis":"x","yaxis":"y"},{"x":[0.0092519,0.0164721,0.0152303,0.002005,0.0070034,0.0019001],"y":[48.9349976,42.8319969,39.6289978,36.8269997,18.8260002,2.0780001],"mode":"lines + markers","name":"anap_744","type":"scatter","hoverinfo":"name","line":{"fillcolor":"rgba(188,189,34,1)","color":"rgba(188,189,34,1)"},"xaxis":"x","yaxis":"y"},{"x":[0.0085243,0.0144256,0.0128589,0.0015123,0.0063669,0.0017499],"y":[48.9349976,42.8319969,39.6289978,36.8269997,18.8260002,2.0780001],"mode":"lines + markers","name":"anap_800","type":"scatter","hoverinfo":"name","line":{"fillcolor":"rgba(23,190,207,1)","color":"rgba(23,190,207,1)"},"xaxis":"x","yaxis":"y"}],"base_url":"https://plot.ly"},"evals":["config.modeBarButtonsToAdd.0.click"],"jsHooks":[]}</script><!--/html_preserve-->

To make a map using **rbokeh** tools, first manually load the **maps** package providing the basemap showing the continents and pass the **Spectra** object to *spc.plot.map.rbokeh()*. The 26 observations stored in this object come from a total of 4 stations which are shown in the map below :

```r
library("maps")
spc.plot.map.rbokeh(myS)
```

<!--html_preserve--><div id="htmlwidget-da6528fd546aeb20b632" style="width:480px;height:480px;" class="rbokeh html-widget"></div>
To plot a **Spectra** object with respect to time, simply pass the object to *spc.plot.time.plotly()*

```r
spc.plot.time.plotly(myS)
```

<!--html_preserve--><div id="htmlwidget-b03ab427269c445a79a4" style="width:480px;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-b03ab427269c445a79a4">{"x":{"layout":{"margin":{"b":40,"l":60,"t":25,"r":10},"title":"spvar2 longname","hovermode":"closest","xaxis":{"domain":[0,1],"title":"Time","rangeslider":{"type":"date"}},"yaxis":{"domain":[0,1],"title":"a_nap [1/m]"},"showlegend":false},"source":"A","config":{"modeBarButtonsToAdd":[{"name":"Collaborate","icon":{"width":1000,"ascent":500,"descent":-50,"path":"M487 375c7-10 9-23 5-36l-79-259c-3-12-11-23-22-31-11-8-22-12-35-12l-263 0c-15 0-29 5-43 15-13 10-23 23-28 37-5 13-5 25-1 37 0 0 0 3 1 7 1 5 1 8 1 11 0 2 0 4-1 6 0 3-1 5-1 6 1 2 2 4 3 6 1 2 2 4 4 6 2 3 4 5 5 7 5 7 9 16 13 26 4 10 7 19 9 26 0 2 0 5 0 9-1 4-1 6 0 8 0 2 2 5 4 8 3 3 5 5 5 7 4 6 8 15 12 26 4 11 7 19 7 26 1 1 0 4 0 9-1 4-1 7 0 8 1 2 3 5 6 8 4 4 6 6 6 7 4 5 8 13 13 24 4 11 7 20 7 28 1 1 0 4 0 7-1 3-1 6-1 7 0 2 1 4 3 6 1 1 3 4 5 6 2 3 3 5 5 6 1 2 3 5 4 9 2 3 3 7 5 10 1 3 2 6 4 10 2 4 4 7 6 9 2 3 4 5 7 7 3 2 7 3 11 3 3 0 8 0 13-1l0-1c7 2 12 2 14 2l218 0c14 0 25-5 32-16 8-10 10-23 6-37l-79-259c-7-22-13-37-20-43-7-7-19-10-37-10l-248 0c-5 0-9-2-11-5-2-3-2-7 0-12 4-13 18-20 41-20l264 0c5 0 10 2 16 5 5 3 8 6 10 11l85 282c2 5 2 10 2 17 7-3 13-7 17-13z m-304 0c-1-3-1-5 0-7 1-1 3-2 6-2l174 0c2 0 4 1 7 2 2 2 4 4 5 7l6 18c0 3 0 5-1 7-1 1-3 2-6 2l-173 0c-3 0-5-1-8-2-2-2-4-4-4-7z m-24-73c-1-3-1-5 0-7 2-2 3-2 6-2l174 0c2 0 5 0 7 2 3 2 4 4 5 7l6 18c1 2 0 5-1 6-1 2-3 3-5 3l-174 0c-3 0-5-1-7-3-3-1-4-4-5-6z"},"click":"function(gd) { \n        // is this being viewed in RStudio?\n        if (location.search == '?viewer_pane=1') {\n          alert('To learn about plotly for collaboration, visit:\\n https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html');\n        } else {\n          window.open('https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html', '_blank');\n        }\n      }"}],"modeBarButtonsToRemove":["sendDataToCloud"]},"data":[{"x":["2009-08-03 22:29:00","2009-08-04 15:15:00","2009-08-04 15:15:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00"],"y":[0.9226343,0.1050914,0.0844165,0.0729477,0.0736624,0.0917964,0.1023239,0.2896777,0.1349536,0.2888538,0.2444953,0.0172168,0.0815227,0.0202518,0.0198416,0.014651,0.0106452,0.0071028,0.0112636,0.0097746,0.0140871,0.0113141,0.0085663,0.0021352,0.0180695,0.0169739],"mode":"lines + markers","name":"anap_300","type":"scatter","line":{"fillcolor":"rgba(31,119,180,1)","color":"rgba(31,119,180,1)"},"xaxis":"x","yaxis":"y"},{"x":["2009-08-03 22:29:00","2009-08-04 15:15:00","2009-08-04 15:15:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00"],"y":[0.6561351,0.0659296,0.0594932,0.045947,0.0521937,0.0638722,0.0715825,0.183069,0.0847662,0.1782516,0.1558811,0.0110504,0.0521841,0.0148532,0.0149781,0.0111026,0.0078156,0.0045384,0.0068351,0.0069022,0.0096388,0.0074317,0.0055033,0.0016301,0.0111222,0.013373],"mode":"lines + markers","name":"anap_356","type":"scatter","hoverinfo":"name","line":{"fillcolor":"rgba(255,127,14,1)","color":"rgba(255,127,14,1)"},"xaxis":"x","yaxis":"y"},{"x":["2009-08-03 22:29:00","2009-08-04 15:15:00","2009-08-04 15:15:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00"],"y":[0.3926151,0.0428867,0.0398439,0.0308428,0.0337945,0.0416688,0.0475071,0.1224269,0.057755,0.1182633,0.1068568,0.0072727,0.0323918,0.0107678,0.0106527,0.0080585,0.0055888,0.0031987,0.0044584,0.0045594,0.0063264,0.0046741,0.0034708,0.0013528,0.0071469,0.0097953],"mode":"lines + markers","name":"anap_411","type":"scatter","hoverinfo":"name","line":{"fillcolor":"rgba(44,160,44,1)","color":"rgba(44,160,44,1)"},"xaxis":"x","yaxis":"y"},{"x":["2009-08-03 22:29:00","2009-08-04 15:15:00","2009-08-04 15:15:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00"],"y":[0.2476067,0.0262149,0.0259645,0.0204032,0.0218011,0.0266492,0.0308139,0.0810381,0.038202,0.0743688,0.0693906,0.0049885,0.0205498,0.0069817,0.0067253,0.0052477,0.0037738,0.0024257,0.003076,0.0031499,0.0044303,0.0031905,0.0024403,0.0011559,0.0052632,0.0061631],"mode":"lines + markers","name":"anap_467","type":"scatter","hoverinfo":"name","line":{"fillcolor":"rgba(214,39,40,1)","color":"rgba(214,39,40,1)"},"xaxis":"x","yaxis":"y"},{"x":["2009-08-03 22:29:00","2009-08-04 15:15:00","2009-08-04 15:15:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00"],"y":[0.168991,0.0163795,0.0178441,0.0138228,0.0147579,0.0178177,0.0210894,0.0533621,0.0243987,0.0460983,0.041132,0.0037185,0.014317,0.0046841,0.0045255,0.0030885,0.0023928,0.0014976,0.0022847,0.0022098,0.0031845,0.00236,0.0019223,0.0008783,0.0039683,0.003986],"mode":"lines + markers","name":"anap_522","type":"scatter","hoverinfo":"name","line":{"fillcolor":"rgba(148,103,189,1)","color":"rgba(148,103,189,1)"},"xaxis":"x","yaxis":"y"},{"x":["2009-08-03 22:29:00","2009-08-04 15:15:00","2009-08-04 15:15:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00"],"y":[0.1229575,0.0115118,0.0129595,0.0101449,0.0109385,0.0129292,0.0154998,0.040218,0.0177739,0.0334135,0.0296104,0.0029692,0.0107439,0.0033173,0.003211,0.0021989,0.0018721,0.0012243,0.0018315,0.0017487,0.0024811,0.0018652,0.0015636,0.0008484,0.0032684,0.0027055],"mode":"lines + markers","name":"anap_578","type":"scatter","hoverinfo":"name","line":{"fillcolor":"rgba(140,86,75,1)","color":"rgba(140,86,75,1)"},"xaxis":"x","yaxis":"y"},{"x":["2009-08-03 22:29:00","2009-08-04 15:15:00","2009-08-04 15:15:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00"],"y":[0.1044204,0.0091465,0.0106748,0.00719,0.0090072,0.0104175,0.0127203,0.0340497,0.0145915,0.026816,0.0247886,0.0026128,0.0089815,0.0026907,0.0026023,0.0017985,0.0016847,0.0011173,0.0015509,0.0015201,0.00216,0.0016886,0.001375,0.0007178,0.0029753,0.0021071],"mode":"lines + markers","name":"anap_633","type":"scatter","hoverinfo":"name","line":{"fillcolor":"rgba(227,119,194,1)","color":"rgba(227,119,194,1)"},"xaxis":"x","yaxis":"y"},{"x":["2009-08-03 22:29:00","2009-08-04 15:15:00","2009-08-04 15:15:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00"],"y":[0.083399,0.0065682,0.0092384,0.007831,0.0076264,0.0088403,0.0107995,0.0290516,0.0122208,0.0212725,0.0198056,0.0022269,0.0076563,0.0024444,0.0020942,0.0014706,0.0015356,0.0009479,0.0014089,0.0012554,0.0019308,0.0015245,0.0010775,0.0007664,0.0023706,0.0018254],"mode":"lines + markers","name":"anap_689","type":"scatter","hoverinfo":"name","line":{"fillcolor":"rgba(127,127,127,1)","color":"rgba(127,127,127,1)"},"xaxis":"x","yaxis":"y"},{"x":["2009-08-03 22:29:00","2009-08-04 15:15:00","2009-08-04 15:15:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00"],"y":[0.0700913,0.0072211,0.0084286,0.007318,0.0069425,0.0078123,0.0098279,0.0230082,0.0092519,0.0164721,0.0152303,0.002005,0.0070034,0.0019001,0.0015986,0.0011163,0.0014634,0.0008157,0.0012585,0.001065,0.0017092,0.0013687,0.0010075,0.0006485,0.0015723,0.001444],"mode":"lines + markers","name":"anap_744","type":"scatter","hoverinfo":"name","line":{"fillcolor":"rgba(188,189,34,1)","color":"rgba(188,189,34,1)"},"xaxis":"x","yaxis":"y"},{"x":["2009-08-03 22:29:00","2009-08-04 15:15:00","2009-08-04 15:15:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 16:33:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-04 22:07:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00","2009-08-05 16:23:00"],"y":[0.065468,0.0080975,0.0068974,0.0070137,0.0060155,0.0074943,0.0083173,0.0207495,0.0085243,0.0144256,0.0128589,0.0015123,0.0063669,0.0017499,0.0011225,0.0005461,0.001488,0.0008505,0.000897,0.0010027,0.0012031,0.0015097,0.0008236,0.0004512,0.0014471,0.0011787],"mode":"lines + markers","name":"anap_800","type":"scatter","hoverinfo":"name","line":{"fillcolor":"rgba(23,190,207,1)","color":"rgba(23,190,207,1)"},"xaxis":"x","yaxis":"y"}],"base_url":"https://plot.ly"},"evals":["config.modeBarButtonsToAdd.0.click"],"jsHooks":[]}</script><!--/html_preserve-->
