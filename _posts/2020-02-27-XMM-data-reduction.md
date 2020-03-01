---
layout: post
title: "XMM-Newton data reduction"
---

The very basic data reduction procedure of EPIN-pn and RGS is summarized.

# A short note on the data reduction of XMM-Newton
## For EPIC-pn
### Useful links:
1. XMM-Newton science archive: <https://www.cosmos.esa.int/web/xmm-newton/xsa>
2. Thread for pn spectrum: <https://www.cosmos.esa.int/web/xmm-newton/sas-thread-pn-spectrum>
3. The XMM abc guide: <https://heasarc.gsfc.nasa.gov/docs/xmm/abc/>
4. User's guide to XMM SAS: <https://xmm-tools.cosmos.esa.int/external/xmm_user_support/documentation/sas_usg/USG/>

### Installation of SAS
To reduce data of XMM-Newton, the XMM-Newton Science Analysis System (SAS) has to beinstalled. I installed SAS version 18.0.0 on my macos following the instruction:  
<https://www.cosmos.esa.int/web/xmm-newton/sas-installation/>  
Some tools are prerequired to install SAS:  
<https://www.cosmos.esa.int/web/xmm-newton/sas-requirements>

### Calibration
Current Calibration Files (CCF) are needed to reduce data of XMM-Newton. The CCF can be downloaded here:  
<https://heasarc.gsfc.nasa.gov/docs/xmm/xmmgof_mirror.html>

### Reduction
* To download Observation Data Files (ODF):  
<https://www.cosmos.esa.int/web/xmm-newton/xsa>  
Decompress the data to any directory you want but do remember where it is. In this example, the ODF files are in directory: /home/honghui/fairall-9/0741330101/odf 
* To set the environment for SAS:
    - Initiate SAS
    ```
    %>>. /home/honghui/xmmsas_20190531_1155/setsas.sh
    ```
    - Set environment variables
    ```
    %>>export SAS_CCFPATH=/home/honghui/CCF    
    %>>export SAS_ODF=/home/honghui/fairall-9/0741330101/odf 
    %>>export SAS_CCF=/home/honghui/fairall-9/0741330101/odf
    ```
* To get the CCF Index File (CIF):
```
%>>cd $SAS_ODF  
%>>cifbuild withccfpath=no analysisdate=now category=XMMCCF fullpath=yes
%>>export SAS_CCF=/home/honghui/fairall-9/0741330101/odf/ccf.cif
```

* To extract data from calibration data base:
```
%>>odfingest odfdir=$SAS_ODF outdir=$SAS_ODF
%>>export SAS_ODF=/home/honghui/fairall-9/0741330101/odf/2640_0741330101_SCX00000SUM.SAS
```
* The pipeline:
```
%>>epproc 
```
* Exclude background flaring
```
%>>evselect table=2640_0741330101_EPN_U002_ImagingEvts.ds:EVENTS expression='(PATTERN==0) && (PI in [10000:12000])' withrateset=yes rateset=rate.fits maketimecolumn=yes timecolumn=TIME timebinsize=50 makeratecolumn=yes
%>>tabgtigen table=rate.fits expression='RATE<=0.1' gtiset=gti.fits
%>>evselect table=2640_0741330101_EPN_U002_ImagingEvts.ds:EVENTS withfilteredset=true filteredset=filtered.fits keepfilteroutput=true destruct=true expression='(gti(gti.fits,TIME) && (PI in [100:15000]) && (PATTERN<=4))'
```
* Create a sky image and select source and background region:
Please refer to the [thread](https://www.cosmos.esa.int/web/xmm-newton/sas-thread-pn-spectrum) of EPIC-pn to learn more about how to select regions with ds9.
```
%>>evselect table=filtered.fits withimageset=true imageset=image4040.fits xcolumn=X ycolumn=Y imagebinning=binSize ximagebinsize=40 yimagebinsize=40
%>>ds9 image4040.fit
```
* Extract source and background spectra, response files and light curve:
```
%>>evselect table=filtered.fits withspectrumset=yes spectrumset=spectrum.fits energycolumn=PI withspecranges=yes specchannelmin=0 specchannelmax=20479 spectralbinsize=5 expression='((X,Y) IN circle(23965.507,26545.495,1200)) && (FLAG==0) && (PATTERN<=4)'
%>>evselect table=filtered.fits withspectrumset=yes spectrumset=background.fits energycolumn=PI withspecranges=yes specchannelmin=0 specchannelmax=20479 spectralbinsize=5 expression='((X,Y) IN circle(26405.507,29075.495,1200)) && (FLAG==0) && (PATTERN<=4)'
%>>backscale spectrumset=spectrum.fits badpixlocation=filtered.fits
%>>backscale spectrumset=background.fits badpixlocation=filtered.fits
%>>rmfgen spectrumset=spectrum.fits rmfset=spectrum.rmf
%>>arfgen spectrumset=spectrum.fits arfset=spectrum.arf withrmfset=yes rmfset=spectrum.rmf badpixlocation=filtered.fits
%>>specgroup spectrumset=spectrum.fits minSN=10 rmfset=spectrum.rmf arfset=spectrum.arf backgndset=background.fits
%>>evselect table=filtered.fits withrateset=yes rateset=lightcurve.fits timecolumn=TIME timebinsize=1 expression='((X,Y) IN circle(23965.507,26545.495,1200))'
```

## For RGS
The best guide is the [RGS thread](https://www.cosmos.esa.int/web/xmm-newton/sas-thread-rgs).
### Get the spectra
```
%>>export SAS_ODF=/Users/honghui/projects/LLAGN/0500670201/odf/
%>>export SAS_CCF=/Users/honghui/projects/LLAGN/0500670201/odf/
%>>. ~/softwares/sas_18.0.0-Darwin-16.7.0-64/sas-ini.sh 
%>>cd odf/
%>>cifbuild
%>>export SAS_CCF=/Users/honghui/projects/LLAGN/0500670201/odf/ccf.cif
%>>odfingest
%>>export SAS_ODF=/Users/honghui/projects/LLAGN/0500670201/odf/1405_0500670201_SCX00000SUM.SAS
%>>mkdir rgs
%>>cd rgs/
%>>rgsproc withsrc=yes srclabel=RXJ1720.1+2638 srcstyle=radec srcra=260.0415042 srcdec=+26.6251250
```

### Check background flaring
```
%>>evselect table=P0500670301R1S004EVENLI0000.FIT timebinsize=100 rateset=rgs1_background_lc.fits makeratecolumn=yes maketimecolumn=yes expression='(CCDNR==9) && (REGION(P0500670301R1S004SRCLI_0000.FIT:RGS1_BACKGROUND,M_LAMBDA,XDSP_CORR))'
%>>lcurve nser=1 cfile1=rgs1_background_lc.fits window="-" dtnb=100 outfile="lc" plot=yes plotdev="/xs"
%>>tabgtigen table=rgs1_background_lc.fits gtiset=my_low_back.fits expression='(RATE<0.1)'
%>>rgsproc entrystage=3:filter auxgtitables=my_low_back.fits
```

### To combine the spectra of RGS1 and RGS2
```
%>>rgscombine pha='P0500670201R1S004SRSPEC1003.FIT P0500670201R2S005SRSPEC1003.FIT' rmf='P0500670201R1S004RSPMAT1003.FIT P0500670201R2S005RSPMAT1003.FIT' bkg='P0500670201R1S004BGSPEC1003.FIT P0500670201R2S005BGSPEC1003.FIT' filepha='02-r12_o1_srspec.pha' filermf='02-r12_o1_rmf.fits' filebkg='02-r12_o1_bgspec.fits'
```
