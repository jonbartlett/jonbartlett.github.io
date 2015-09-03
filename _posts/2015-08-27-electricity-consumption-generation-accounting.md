---
layout: post
title: Accounting for Electricity Consumption and Generation 
comments: true
category:
---
![{{ page.title }}](/public/images/2015-08-27/2015-08-27-elect-meter-640px.jpg)

## Utility Consumption without Consequence

I have always found it odd that most people consume household utilities without checking on usage between billing cycles. There are not many other products or services that are consumed with such oblivion to cost. It is of course possible to nip outside to the (often not very accessible) electricity/gas/water meters, note down the numbers and then calculate usage from unit price costs and standing charges. For most people, this is understandable too much. I don't know the history of electricity metering however perhaps electricity was always cheap enough not to bother that much? Or electricity utilities companies haven't put much effort into providing a more friendly monitoring mechanism in fear of reduced usage (and revenue)? When I was at uni we had an electricity meter that was fed with 50 pence coins. This certainly focused the mind on usage (and those house mates with electric heaters hidden in their rooms). 

## Monitoring Solutions

The introduction of smart meters in some areas (Victoria, Australia) solves some of the monitoring issues. In most cases consumption can be seen in real-time via utility provided web portals. Data can be made available to third party application providers where utility providers do not provide such a facility.

Another alternative for electricity at least, is the use of one of the many energy monitor devices. These most often involve the use of a "clamp" device which is installed around the main electric feed into the property. This "clamp" measures current flow and via wireless, sends data to a secondary device that displays real-time usage within the house. Data can also be uploaded to an internet service provider via the this display device. 

For those households with solar panels attached to the grid, an equivalent internet connected device is often included in the panel installation and inverter setup.

Personally I have been using a [Current Cost EnviR](http://www.currentcost.com/product-envir.html) to monitor household electricity consumption for a number of years now. A web portal is provided for monitoring live and historical electricity consumption (and room temperature). Behind the scenes, this data is aggregated by [Xively](https://personal.xively.com).

##PV Output
A far more functionally rich monitoring and reporting web application is [PV Output](http://pvoutput.org). Intended to monitor solar PV generation, the Current Cost output via Xively can be monitored. In order to work out the actual costs of usage, PV Output provides options to enter electricity tariff information, from which this it can provide electricity costs per day, month, etc. A long way around but this gives some idea of what your next electricity bill will be. PV Output can also email weekly reports to you for monitoring. If you have solar PV via a grid tie, feed in tariffs can also be calculated. They also have an API to access all this generated and calculated data.

![PVOutput live consumptopn graph](/public/images/2015-08-27/2015-08-27-pvoutput-graph.png) 




## Accounting for Consumption (and Generation)
This (southern hemisphere) winter, as a household we have taken more interest in how much electricity we are using and how much this is costing. Our personal finances are kept track of using the excellent [GNUCash](http://www.gnucash.org/) open source software. This got me thinking how I could keep track of un-billed electricity usage via GNUCash, specifically between bills.

From an accounting perspective, between billing cycles it would be possible to create an Expense item and corresponding Liability entry for electric consumption since the last bill. At any point in time I can then see how much money I owe the electricity company before the bill comes in (and budget accordingly).

```
+-----------------------------------+----+----+
|              Account              | CR | DR |
+-----------------------------------+----+----+
| Liabilities:Utilities:Electricity |    | 10 |
| Expenses:Utilities:Electricity    | 10 |    |
+-----------------------------------+----+----+
```  
*Mid-billing cycle consumption data from PVOutput*

When the bill comes in and paid, the Liability entry is credited with a debit (payment) from the Asset (bank) account. 

```
+-----------------------------------+----+----+
|              Account              | CR | DR |
+-----------------------------------+----+----+
| Liabilities:Utilities:Electricity | 10 |    |
| Asset:Current:Bank:Main           |    | 10 |
+-----------------------------------+----+----+
```

If you have enough solar panels to cover day time usage and generate excess electricity to be fed into the grid, this could also be accounted for prior to the credit being made to your bank account. This would look like:

```
+----------------------------------------------------------------+----+----+
|                            Account                             | CR | DR |
+----------------------------------------------------------------+----+----+
| Income:Other Income:Electricity Generation                     | 10 |    |
| Assets:Current Assets:Money owed to you:Electricity Generation |    | 10 |
+----------------------------------------------------------------+----+----+
```
and when the cash from the utility hits your bank account:


```
+----------------------------------------------------------------+----+----+
|                            Account                             | CR | DR |
+----------------------------------------------------------------+----+----+
| Assets:Current Assets:Money owed to you:Electricity Generation | 10 |    |
| Asset:Current:Bank:Main                                        |    | 10 |
+----------------------------------------------------------------+----+----+
```

##PVOutputQIF

GNUCash has the ability to upload QIF files (same as those bank statement files downloaded from the bank), PVOuput has the consumption and costs data but there is no functionality to download such QIF files. 

Using the PVOutput [API](http://pvoutput.org/help.html#api) and the open source Ruby code to [generate QIF files](https://github.com/jemmyw/Qif) I have been working on something that will produce QIF files to create accounting entries for un-billed electricity consumption and/or generation. Here is how it works.

###Enable PVOutput API Access

Within PVOutout, API access is not enabled by default. Log into PVOutput and within the settings, enable API access. Also set a read only key as this will be used to access your data. Also take note of your "System ID".
 
###Configure PVOutputQIF

A yaml [file](https://github.com/jonbartlett/pvoutput-qif/blob/master/pvoutput_qif.yml) contains all the configuration required. Modify this file with the relevant details. Alternatively, all configuration can be passed at runtime. The following settings must be defined:

* sysid = PVOutput System ID
* apikey = the Read Only Key as defined in the PVOutput API settings
* income = the income account defined in your accounting package (i.e Income:Other Income:Electricity Generation)
* asset = the asset account defined in your accounting package (i.e. Assets:Current Assets:Money owed to you:Electricity Generation)
* liability = the liability account defined in your accounting package (i.e  Liabilities:Utilities:Electricity)
* expense = the expense account defined in your accounting package (i.e Expenses:Utilities:Electricity)


###Run PVOutputQif

The default behaviour is for data to be collected from the last run date to the day prior to the current date (the last complete day of data). The last run date is held in the yaml config file and updated on each run. If it is the first time the program has been run (and the last run date is empty), 90 days of data will be retrieved. This behaviour can be overridden by passing start and end dates in at the command line.

Run the program from the command line:

```bash
$ ruby bin/pvoutput.rb

 PVOuput Qif Exporter
 https://github.com/jonbartlett/pvoutput-qif

 QIF file exported: data/pvo2015082720150902.qif

```

View the QIF file created 

```bash
$ cat data/pvo2015082720150902.qif

!Account
NExpenses:Utilities:Electricity
^
!Type:Cash
D20150902
MElectricity consumed 27/08/2015 to 02/09/2015 (6 days) = 96.185 kWh (16.031 kWh/day)
SLiabilities:Utilities:Electricity
$29.73
^
```

###Import QIF file into Accounting Package

If you are using GNUCash and enter your account names correctly into the config file, the import process will automatically match to the correct accounts. Here is the Liability account entries showing the imported PVOutput data on electricity consumption.


![GNUCash Liability Entry](/public/images/2015-08-27/2015-08-27-gnucash-liability.png) 

and the corresponding Expense account entries.

![GNUCash Liability Entry](/public/images/2015-08-27/2015-08-27-gnucash-expense.png) 


All source code can be found on Github [here](https://github.com/jonbartlett/pvoutput-qif). You'll need Ruby installed and be familiar with how to run Ruby code. 



*Caveat: Despite working in and around finance software for most of my profession life, I am no accountant. This solution works for me but may not be strictly correct from an accounting perspective. Happy to take feedback from more qualified persons.* 
