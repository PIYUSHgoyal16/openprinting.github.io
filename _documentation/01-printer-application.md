---
title: Printer Applications - A new way to print in Linux
toc: true
toc_sticky: true
---

Ever wondered how a Printer works? What are the different steps involved between the print command and the final output?

The following document contains information about the history of printing and its evolution. It describes Printer Applications. What were the issues with the previous methodologies? How Printer Applications helped in solving them? Why it has been referred to as the "Technology for Future"? In the end, it also contains roles of different entities including OpenPrinting, Manufacturers, and the User.  


## Brief History

The first release of CUPS was back in 1997. <sup><a href="https://openprinting.github.io/How-did-this-all-begin/">[1]</a></sup> Since then, Printer Drivers consisted of **PPD files** and **CUPS filters**.

**PPD (PostScript Printer Description)** is a decades-old data format created by Adobe, probably together with PostScript or shortly after, to describe capabilities and user-settable options of PostScript printers and which PostScript commands to embed in the print job to execute the option settings.

**PPD files** describe the printer's capabilities and which filters to use to produce the data format needed by the printer.

This format was adopted because in that time printing under Linux and Unix worked via **PostScript**. Applications sent jobs in PostScript which could be understood by PostScript printers. It allowed users to use all PostScript printers directly. The system supports printers, the user uses printers.

**CUPS** made the move from built-in printer drivers in Ghostscript to CUPS filters, with the help of the CUPS Raster device-independent print raster format. CUPS raster drivers used **PPD files** because it used **Ghostscript** or (on IRIX) **Impressario** (a version of Adobe's PostScript interpreter) to produce raster data for printing, and they could use embedded PostScript commands to control page size, colour space, etc.  Since both PostScript and raster printers could then use PPD files, Michael Sweet adopted PPD as a common printer description format, which also got used for his company's ESP Print Pro (GUI) software and then later macOS and GNOME/KDE. CUPS provides reserved directories to drop these **PPD files** and **filters** into, so adding a printer driver was rather easy.

Nowadays applications send jobs in PDF and CUPS does the **PDF-centric** processing, already for 8 years, since the first release of cups-filters back in 2012. So PPD files do not really fit in the picture anymore, and they also had their shortcomings, especially being rather unflexible in the possible types of user-settable options. Also, the need to drop filters and PPDs into reserved directories of CUPS makes it difficult to provide CUPS and printer drivers in sandboxed packages, like **Snaps**.

Already several years ago, mainly due to the advent of smartphones and the desire to also print from these devices, printers got equipped with **driverless IPP** printing functionality: [AirPrint](https://support.apple.com/en-in/HT201311)(released in 2010), IPP Everywhere, [Mopria](https://mopria.org/), Wi-Fi Direct Print. These standards are practically all the same, the printer advertises its presence, its network address, and basic capabilities via DNS-SD (aka BonJour, mDNS, zero-conf, implemented with Avahi in Linux), accepts communication and jobs from clients via **IPP (Internet Printing Protocol)**, from the [Printer Working Group](http://www.pwg.org/), supplies complete capability info to clients via IPP and uses only common Page Description Languages (PDLs) for jobs: PDF, Apple Raster, PWG Raster, PCLm.


## What is a Printer Application?
    
A **Printer Application** is a background service, a GUI application, etc. that happens to respond to HTTP and IPP requests. It detects the supported printers and advertises those printers on the localhost as an IPP Everywhere printer. It can also listen on network interfaces (i.e. not just localhost) so provide access to a printer to all client devices on a network. Printer Applications are the recommended architecture for printer drivers.

The Printer Application emulates a driverless IPP printer, so that the printing system does not need to distinguish, it simply needs to support driverless IPP printers. This way we add support for non-driverless printers and avoid deprecated and inconvenient methods, like using PPD (Postscript Printer Description) files on non-PostScript printers but also allow sandboxed packaging which allows providing OS-distribution-independent driver packages and improves security.

In a **sandboxed package**, we cannot modify directory contents once it is built. Our system is no more modular. We cannot choose which printer driver package to install. Printer Applications address this problem of modularity and give us the same freedom as in the case of printer drivers.

A **Printer Application's Web Interface** provides configurability and makes it more accessible to the user. Instead of the web interface, one can also use the standard interface IPP System Service. This allows defining configurable parameters that a device-independent client can poll from the IPP server and display in a GUI so that the user can change them appropriate to his needs. It allows creation and deletion of printers, viewing active and completed jobs, cancellation of job/jobs, configuring the loaded media and network settings, requesting software updates, etc. The underlying mechanism involves adding custom pages and contents using callbacks, static resources, or external files and directories.


## Advantages of Printer Applications

* <strong>One Driver for one Manufacturer</strong> 
    
    The classical printer driver binaries were specifically built for Linux distributions because the distribution provides the layout of CUPS, the libraries, and so on. Also, they have different packaging formats DEB, RPM, and many more. For each distro, drivers need to be built, packaged, and tested separately. It made the task next to impossible to execute. As a result, most of the hardware manufacturers refrained from making these drivers.

    Printing Applications have reduced the intensity of manufacturer's task to a great extent as now they only need one application for all printers, for all distributions, and distribute it through **snaps**.

* <strong>Usability</strong>

    Users are enabled to directly install and configure CUPS and Printer Application Drivers by just a few simple steps. They do not require to manually drop the PPD files in reserved directories and hence make the process less error-prone. Also, they just need to have a single Printer Application Driver for different printers from the same manufacturer.

* <strong>Easy Configuration</strong>

    Printer Applications also provide a Web Interface to easily configure Printer's Capabilities, helping people who were earlier unaware to change properties of a Printer in PPD files and were forced to pass the same printing options through the Command line, every time.

* <strong>Good Bye PPD!</strong>

    PPD files which were earlier used to describe the printer's capabilities and which filters to use to produce the data format needed by the printer are not required by Printer Application.

* <strong>Diversified Nature</strong>

    Printer Applications have a lot in common. Therefore Michael Sweet has created PAPPL, a library that provides all the common functionality which is needed in every Printer Application. However, the manufacturer is expected to write the drivers on their own (using PAPPL) and defining functions like identifying the start of the page, the start of a line, etc, leading to Printer Application's Diversified Nature.

* <strong>Technology for Future</strong>

    As we know, Linux is moving to sandboxed packaging (Snap for example) and printing is also moving in that direction. In a sandboxed package, we cannot modify directory contents once it is built. Our system is no more modular. We cannot choose which printer driver package to install. Printer Applications address this problem of modularity and give us the same freedom as in the case of printer drivers.


## Roles and Responsiblities

![Roles Flowchart](../../assets/images/roles-flowchart.jpg) 

### OpenPrinting

* <strong>CUPS Snap <sup><a href="https://github.com/OpenPrinting/cups-snap">[2]</a></sup></strong> 

    This is a complete printing stack in a Snap. It contains not only CUPS but also cups-filters, Ghostscript, and Poppler. It is everything which is required for printing, except Printer Drivers. Till Kamppeter has created this in order to provide a complete printing stack for a purely Snap-based Operating System.

* <strong>PAPPL <sup><a href="https://github.com/michaelrsweet/pappl/">[3]</a></sup></strong> 

    Printer Applications have a lot in common. It would be a lot of re-inventing the wheel if everyone who wants to create a printer driver has to implement all this. Therefore Michael Sweet has created PAPPL, a library which provides all the common functionality which is needed in every Printer Application.

* <strong>Web Interface</strong> 

    Printer Applications also provide a Web Interface to easily configure Printer's Capabilities. This GUI helps people who were unaware to change printer properties directly in PPD files and were forced to pass the similar printing options through Command line, each and every time.

* <strong>CUPS filters <sup><a href="https://github.com/OpenPrinting/cups-filters">[8]</a></sup></strong> 
    
    These are APIs that can be utilized to perform the conversion of data of files from one format to another. 

    Currently, PAPPL supports only raster printers and that too for very few specific input formats like JPEG and PNG. For adding support for non-raster printers like PDF and PostScript printers, you need to supply an external utility that converts the whole job’s data into a data stream which the printer understands. Openprinting and CUPS Filters again come to the rescue here.

    Manufacturers can use CUPS Filters to design these external utilities and decide which set of Filters to use and the order in which these should be invoked depending upon the efficiency and input-output formats. Kindly refer to guidelines for designing of printer applications<sup> <a href="../02-designing-printer-drivers/">[4]</a></sup> for more details.

* <strong>Providing Sufficient Guidance</strong> 
    
    OpenPrinting also helps the manufacturer about designing of printer <sup> <a href="../02-designing-printer-drivers/">[4]</a></sup> and scanner drivers.<sup><a href="../03-designing-scanner-drivers/">[5]</a></sup> It also guides them about packaging those drivers and uploading on Snap Store. <sup> <a href="../04-packaging-drivers/">[6]</a></sup>  It also educated users about using Printer Application Drivers. It provides neccessary documentation and tutorial for all the above mentioned things.<sup> <a href="../05-User-Manual/">[7]</a></sup>

 
### Manufacturer

The Manufacturer besides manufacturing Printers and Scanners, is also expected to design drivers for the same. They can use PAPPL<sup><a href="https://github.com/michaelrsweet/pappl/">[3]</a></sup> library which certainly reduces their task to an great extent. Also they could take help from the documentation and tutorial developed by the OpenPrinting team. <sup> <a href="../02-designing-printer-drivers/">[4]</a></sup> 
<sup> <a href="../03-designing-scanner-drivers/">[5]</a></sup>  

After designing the drivers, they need to package them and Upload it to Snap Store. Again, documentation and tutorial for the same have developed by the OpenPrinting team which could be referred.<sup> <a href="../04-packaging-drivers/">[6]</a></sup> 


### User

The user is relieved from most of the complexities and is just required to install CUPS and Printer Application Driver (based on the manufacturer). 

Further, he has the option to change his printer properties through the Web GUI provided with Printer Application.

The OpenPrintring team has also worked out documentation and tutorials about installing CUPS and Drivers and using Web-based GUI, making the switch to this new technology easier for all.<sup> <a href="../05-User-Manual/">[7]</a></sup>

## Resources

[1] <a href="https://openprinting.github.io/How-did-this-all-begin/">How all this began</a>
<br>
[2] <a href="https://github.com/OpenPrinting/cups-snap">CUPS Snap</a>
<br>
[3] <a href="https://github.com/michaelrsweet/pappl/">PAPPL</a>
<br>
[4] <a href="../02-designing-printer-drivers/">Tutorial to Design Printer Drivers</a>
<br>
[5] <a href="../03-designing-scanner-drivers/">Tutorial to Design Scanner Drivers</a>
<br>
[6] <a href="../04-packaging-drivers/">Packaging Drivers and Uploading them to Snap Store</a>
<br>
[7] <a href="../05-User-Manual/">User Manual</a>
<br>
[8] <a href="https://github.com/OpenPrinting/cups-filters">CUPS Filters</a>
