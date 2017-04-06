# Accessing Data


##Getting and setting Wavelength information

To extract (get) and to change (set) wavelengths, use *spc.getwavelengths* and *spc.setwavelengths* :

```r
range(spc.getwavelengths(myS))
```

```
## [1] 300 800
```

```r
lbd <- 302:802
spc.setwavelengths(myS) <- lbd
range(spc.getwavelengths(myS))
```

```
## [1] 302 802
```

The length of the Wavelenth slot should always to be equal to the number of channels stored in the object :

```r
length(lbd)==ncol(myS)
```

```
## [1] TRUE
```

##Subsetting
Subsetting with row indexes using the “[“ operator returns a **Spectra** object. The following command extracts rows 1 to 10 and spectral columns 30 to 50 (provided in integer format). Ancillary data is unchanged. 

```r
myS[1:10]
```

```
## 
##  a_nap  : An object of class "Spectra"
##  501 spectral channels in columns and 10 observations in rows 
##  LongName:  spvar2 longname 	 Units:  1/m 
##  Wavelengths :  501 channels with units of nm [ 302 , 802 ] -> 302 303 304 305 306 307  ...
##  Spectra Columns:  anap_300 anap_301 anap_302 anap_303 anap_304 anap_305 ...
##  Ancillary Columns:  idx CRUISE STATION CAST METHOD NISKIN ...
##  Bounding box: LON( -133.4959 -130.5083 ) LAT( 69.84703 71.26659 )
##  Time : periodicity of 0 secs between (2009-08-03 22:29:00 - 2009-08-04 22:07:00), tz=UTC
```

The “[“ operator also supports column names :

```r
myS[,"anap_400"] 
```

```
## 
##  a_nap  : An object of class "Spectra"
##  1 spectral channel in columns and 26 observations in rows 
##  LongName:  spvar2 longname 	 Units:  1/m 
##  Wavelengths :  1 channel with units of nm [ 402 , 402 ] -> 402  ...
##  Spectra Columns:  anap_400 ...
##  Ancillary Columns:  idx CRUISE STATION CAST METHOD NISKIN ...
##  Bounding box: LON( -133.4959 -130.5083 ) LAT( 69.84703 72.05774 )
##  Time : periodicity of 0 secs between (2009-08-03 22:29:00 - 2009-08-05 16:23:00), tz=UTC
```

```r
myS[,c("anap_400","anap_500")] 
```

```
## 
##  a_nap  : An object of class "Spectra"
##  2 spectral channels in columns and 26 observations in rows 
##  LongName:  spvar2 longname 	 Units:  1/m 
##  Wavelengths :  2 channels with units of nm [ 402 , 502 ] -> 402 502  ...
##  Spectra Columns:  anap_400 anap_500 ...
##  Ancillary Columns:  idx CRUISE STATION CAST METHOD NISKIN ...
##  Bounding box: LON( -133.4959 -130.5083 ) LAT( 69.84703 72.05774 )
##  Time : periodicity of 0 secs between (2009-08-03 22:29:00 - 2009-08-05 16:23:00), tz=UTC
```

If only a set of wavelengths are desired, either the column indexes have to be proveded in integer format or the desired wavelengths have to be provided in numeric (floating) format :


```r
myS[1:10,30:50] #Selection of channels by column index
```

```
## 
##  a_nap  : An object of class "Spectra"
##  21 spectral channels in columns and 10 observations in rows 
##  LongName:  spvar2 longname 	 Units:  1/m 
##  Wavelengths :  21 channels with units of nm [ 331 , 351 ] -> 331 332 333 334 335 336  ...
##  Spectra Columns:  anap_329 anap_330 anap_331 anap_332 anap_333 anap_334 ...
##  Ancillary Columns:  idx CRUISE STATION CAST METHOD NISKIN ...
##  Bounding box: LON( -133.4959 -130.5083 ) LAT( 69.84703 71.26659 )
##  Time : periodicity of 0 secs between (2009-08-03 22:29:00 - 2009-08-04 22:07:00), tz=UTC
```

```r
lbd = as.numeric(c(412,440,490,555,670))
myS[1:10,lbd] #Selection of channels by wavelength
```

```
## 
##  a_nap  : An object of class "Spectra"
##  5 spectral channels in columns and 10 observations in rows 
##  LongName:  spvar2 longname 	 Units:  1/m 
##  Wavelengths :  5 channels with units of nm [ 412 , 670 ] -> 412 440 490 555 670  ...
##  Spectra Columns:  anap_410 anap_438 anap_488 anap_553 anap_668 ...
##  Ancillary Columns:  idx CRUISE STATION CAST METHOD NISKIN ...
##  Bounding box: LON( -133.4959 -130.5083 ) LAT( 69.84703 71.26659 )
##  Time : periodicity of 0 secs between (2009-08-03 22:29:00 - 2009-08-04 22:07:00), tz=UTC
```

A subset of the spectra can also be extracted over a range of wavelengths using the following formulation :

```r
myS[1:10,"415::450"] 
```

```
## 
##  a_nap  : An object of class "Spectra"
##  36 spectral channels in columns and 10 observations in rows 
##  LongName:  spvar2 longname 	 Units:  1/m 
##  Wavelengths :  36 channels with units of nm [ 415 , 450 ] -> 415 416 417 418 419 420  ...
##  Spectra Columns:  anap_413 anap_414 anap_415 anap_416 anap_417 anap_418 ...
##  Ancillary Columns:  idx CRUISE STATION CAST METHOD NISKIN ...
##  Bounding box: LON( -133.4959 -130.5083 ) LAT( 69.84703 71.26659 )
##  Time : periodicity of 0 secs between (2009-08-03 22:29:00 - 2009-08-04 22:07:00), tz=UTC
```

If numeric values are required instead of a *Spectra* object, use the “$" or the “[[" operators

```r
myS$CAST #Returns Ancillary data
```

```
##  [1] 38 40 40 41 41 41 41 41 44 44 44 44 44 44 51 51 51 51 51 51 51 51 51
## [24] 51 52 52
```

```r
myS$anap_400 #Returns spectra as numeric vector
```

```
##  [1] 0.4345009 0.0472191 0.0431448 0.0334085 0.0366025 0.0453854 0.0513998
##  [8] 0.1312243 0.0618284 0.1265693 0.1137473 0.0079286 0.0355068 0.0114464
## [15] 0.0114911 0.0086473 0.0060563 0.0034033 0.0047790 0.0049624 0.0069034
## [22] 0.0051754 0.0037742 0.0014785 0.0078290 0.0105174
```

```r
head(myS[["anap_400"]]) #Returns spectra as numeric vector
```

```
## [1] 0.4345009 0.0472191 0.0431448 0.0334085 0.0366025 0.0453854
```

```r
head(myS[[c("Snap","Offset")]]) #Returns data.frame
```

```
##        Snap    Offset
## 1 0.0086613 0.0449174
## 2 0.0094426 0.0049271
## 3 0.0082714 0.0054899
## 4 0.0089460 0.0050306
## 5 0.0085369 0.0046916
## 6 0.0084656 0.0050392
```

Subsetting can also be achieved using the implementation of the R function subset() for *Spectra* and *SpcList* classes. It is possible to perform a row-wise selection 

```r
subset(myS,DEPTH<=30) #Subsetting rows with respect to the value of Ancillary data
```

```
## 
##  a_nap  : An object of class "Spectra"
##  501 spectral channels in columns and 15 observations in rows 
##  LongName:  spvar2 longname 	 Units:  1/m 
##  Wavelengths :  501 channels with units of nm [ 302 , 802 ] -> 302 303 304 305 306 307  ...
##  Spectra Columns:  anap_300 anap_301 anap_302 anap_303 anap_304 anap_305 ...
##  Ancillary Columns:  idx CRUISE STATION CAST METHOD NISKIN ...
##  Bounding box: LON( -133.4959 -130.5083 ) LAT( 69.84703 72.05774 )
##  Time : periodicity of 0 secs between (2009-08-03 22:29:00 - 2009-08-05 16:23:00), tz=UTC
```

```r
subset(myS,DEPTH<=30)$DEPTH
```

```
##  [1]  2.245 23.163  2.149 25.965 16.889 10.741  4.878  2.297 18.826  2.078
## [11] 29.090 18.758  8.762  2.015  1.956
```

```r
subset(myS,anap_440<=0.01) #Subsetting rows with respect to the value of Spectral data
```

```
## 
##  a_nap  : An object of class "Spectra"
##  501 spectral channels in columns and 14 observations in rows 
##  LongName:  spvar2 longname 	 Units:  1/m 
##  Wavelengths :  501 channels with units of nm [ 302 , 802 ] -> 302 303 304 305 306 307  ...
##  Spectra Columns:  anap_300 anap_301 anap_302 anap_303 anap_304 anap_305 ...
##  Ancillary Columns:  idx CRUISE STATION CAST METHOD NISKIN ...
##  Bounding box: LON( -130.8965 -130.6084 ) LAT( 71.26659 72.05774 )
##  Time : periodicity of 0 secs between (2009-08-04 22:07:00 - 2009-08-05 16:23:00), tz=UTC
```

```r
subset(myS,subset=DEPTH<=30,select="CAST") #Selecting Ancillary data columns, leaving Spectral columns intact
```

```
## 
##  a_nap  : An object of class "Spectra"
##  501 spectral channels in columns and 15 observations in rows 
##  LongName:  spvar2 longname 	 Units:  1/m 
##  Wavelengths :  501 channels with units of nm [ 302 , 802 ] -> 302 303 304 305 306 307  ...
##  Spectra Columns:  anap_300 anap_301 anap_302 anap_303 anap_304 anap_305 ...
##  Ancillary Columns:  CAST ...
##  Bounding box: LON( -133.4959 -130.5083 ) LAT( 69.84703 72.05774 )
##  Time : periodicity of 0 secs between (2009-08-03 22:29:00 - 2009-08-05 16:23:00), tz=UTC
```

```r
subset(myS,subset=DEPTH<=30,select="anap_440") #Selecting Spectral data columns, leaving Ancillary columns intact
```

```
## 
##  a_nap  : An object of class "Spectra"
##  1 spectral channel in columns and 15 observations in rows 
##  LongName:  spvar2 longname 	 Units:  1/m 
##  Wavelengths :  1 channel with units of nm [ 442 , 442 ] -> 442  ...
##  Spectra Columns:  anap_440 ...
##  Ancillary Columns:  idx CRUISE STATION CAST METHOD NISKIN ...
##  Bounding box: LON( -133.4959 -130.5083 ) LAT( 69.84703 72.05774 )
##  Time : periodicity of 0 secs between (2009-08-03 22:29:00 - 2009-08-05 16:23:00), tz=UTC
```

To see the implementation of subset() for the *Spectra* class, try :

```r
showMethods(subset,classes="Spectra",includeDefs=T) 
```

##Selecting and flagging rows as invalid
It is possible to select some of the rows manually or with the help of the mouse. As of the writing of this document, only records (rows) of a *Spectra* object can be selected with mouse. To manually set the first five observations as selected, use :

```r
idx=rep(FALSE,nrow(myS)); 
idx[1:5]=TRUE
spc.setselected.idx(myS)<-idx 
```

Set the selected spectra rows as Invalid :

```r
spc.setinvalid.idx(myS)<-spc.getselected.idx(myS) 
```

The function spc.select() needs the Spectra plot created by the spc.plot() method. First call spc.plot(), then use the left button of the mouse to select spectra. To end the selection process, click on an empty area or push the right mouse button : Row indexes of selected curves will display in the R command line. The function `spc.setselected.idx()` stores the indexes in the slot **SelectedIdx** of the variable.


```r
spc.plot(myS)
```

![](Accessing_Data_files/figure-html/unnamed-chunk-13-1.png)<!-- -->

```r
spc.setselected.idx(myS)<-spc.select(myS)
```

```r
spc.getselected.idx(myS)
```

```
##  [1]  TRUE  TRUE  TRUE  TRUE  TRUE FALSE FALSE FALSE FALSE FALSE FALSE
## [12] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
## [23] FALSE FALSE FALSE FALSE
```

The display of selected and invalid rows is currently not implemented in plotting functions (ex. spc.plot(), spc.plot.depth(), spc.plot.time()). The slot **SelectedIdx** will be reset if the dimensions of the *Spectra* object is changed by subsetting or another operation.
