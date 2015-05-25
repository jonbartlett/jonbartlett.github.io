---
layout: post
title: Comparing Oracle e-Business Suite Personalisations
comments: true
category: geek 
tags: oracle ebs oaf personalizations
---
Managing OAF personalisations between multiple Oracle EBS environments can become a major [PITA](http://www.urbandictionary.com/define.php?term=pita). Problems arise when personalisations are applied both manually and through automated methods such as the JRAD xml importer utility. Oracle did not provide fine level control of personalisations through the JRAD utility. This means that any personalisations applied to a page using this method, results in any existing manually applied personalisation being removed (last in wins).

To overcome these limitations, the entire personalisation (MDS) repository can be maintained centrally (in SCM) by exporting this using Functional Administrator functionality. Personalisations are manually applied to an environment, extracted and applied to this repository, then uploaded to other environments via the Functional Administrator import functionality. This prevents manually applied and extension specific personalisations overriding each other as they are held in one location and always migrated using this method.

In order to manage personalisations in this way we need to be able to compared the entire personalisation config between environments. This is also useful in analysing the impact of patching.

The following steps describe how to compare all personalisations between environments.

## Export Personalisations to File System from Environment 1 and 2 

Create an empty directory on the file system (I used ```$XXCUST_TOP/mds```).

Set profile option value "FND: Personalization Document Root Path" to point to file system path.

Navigate to responsibility "Functional Administrator" and menu path "Personalization / Import Export".

![{{ page.title }}](/public/images/2015-05-25/01-personalisation-listing.png)

Press "Select All" then 'Export" to dump all personalisations to the file system.

![{{ page.title }}](/public/images/2015-05-25/02-personalisations-exported.png)

Repeat these steps on other the environment to compare to.

## Copy Personalisation Zip File

Now zip up the entire export file system tree on one environment and transfer to the other environment where comparison will be made.

```bash
zip -r crp2_mds.zip $XXCUST_TOP/mds
scp crp2_mds.zip applmgr@devclnebs:/tmp/crp2_mds.zip
```

Unzip file to a location where it will be compared. I used ```/tmp```.

```bash
unzip crp2_mds.zip
```
    
Copy personalisations to compare to ```/tmp``` (these personalisation can be compared in situ but I prefer copying them to a common location)

```bash
cp $XXCUST_TOP/mds /tmp/devcln_mds
```

## Compare Personalisations

We can can run a ```diff``` across the two sets of personalisations. You may need to spool to a file if there are many differences found.

```bash
diff -u -r /tmp/crp2_mds /tmp/devcln_mds| grep -v 'Common subdirectories'
```

![{{ page.title }}](/public/images/2015-05-25/03-personalisation-diff.png)

All differences are output by the ```diff``` utility however I find Vim diff mode useful for further analysis.

```bash
vim -d /tmp/crp2_mds/oracle/apps/pay/selfservice/payslip/webui/customizations/localization/GB/PayslipPG.xml /tmp/devcln_mds/oracle/apps/pay/selfservice/payslip/webui/customizations/localization/GB/PayslipPG.xml
```

![{{ page.title }}](/public/images/2015-05-25/04-vimdiff-personalisations.png)


Individual personalisations can then be pulled out and merged back into the central source code version which in turn can be migrated to other environments. Merging involves pulling out the relevant XML retuned and copy+paste into the new version. [Vimdiff](http://vimdoc.sourceforge.net/htmldoc/diff.html) is very useful for performing this analysis and assisting with the merging. Automated tools are also available.

This method was used successfully on a large EBS implementation with many extensions and customisation that included personalisation.


## Further Analysis

The technique above is also extremely useful for determining the impact of a patch on personalisations. Take a personalisation export in a controlled environment or export from source control, then compare with the export of the patched environment. Patch changes will show up and any conflicts between existing personalisations identified.
