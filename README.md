## provTraceR

The provTraceR package displays information about files used or created 
by an R script or a series of R scripts. The package uses provenance collected by
[rdtLite](https://CRAN.R-project.org/package=rdtLite) and stored in 
[prov-json format](https://github.com/End-to-end-provenance/ExtendedProvJson/blob/master/JSON-format.md).
Output from provTraceR can be used to help manage files and to identify the 
input files needed to reproduce an analysis.

## Installation
To install from GitHub:

```
devtools::install_github("End-to-end-provenance/provTraceR")
```

To load the package after installation:

```
library("provTraceR")
```

## Usage
This package includes two functions:

1. To use existing provenance to trace file lineage:

```
prov.trace(scripts, prov.dir=NULL, file.details=FALSE, console=TRUE, save=FALSE, save.dir=NULL, check=TRUE)
```

2. To run one or more scripts, collect provenance, and trace file lineage:

```
prov.trace.run(scripts, prov.dir=NULL, file.details=FALSE, console=TRUE, save=FALSE, save.dir=NULL, check=TRUE, prov.tool="rdtLite", details=FALSE, ...)
```

The <i>scripts</i> parameter may contain a single script name, a vector
of script names, or a text file (with extension .txt) of script names.

For <i>prov.trace</i> only: If more than one script is specified, the order
of the scripts must match the order of execution as recorded in the 
provenance; otherwise an error message is displayed. For console sessions,
set <i>scripts</i> = "console".

For <i>prov.trace.run</i> only: The provenance collection tool specified by
prov.tool must be "rdtLite" or "rdt". If details = TRUE, fine-grained provenance
is collected. Other optional parameters (...) are passed to rdtLite or rdt.
Scripts are executed in the order listed.

It is assumed that provenance for each script is stored under a single
provenance directory set by the <i>prov.dir</i> option.  If not, the provenance
directory may be specified with the <i>prov.dir</i> parameter. Timestamped 
provenance and provenance in scattered locations are not currently supported.

Files are matched by hash value. INPUTS lists files that are required 
to run the script or scripts. These include files read by a script and not
written by an earlier script or previously written by the same script.
OUTPUTS lists files written by the script or scripts. EXCHANGES lists 
files with the same hash value that were written by one script and read 
by a later script; if the location changed, both locations are listed.

If <i>file.details</i> = TRUE, additional details are displayed, including
script execution timestamps, file timestamps, file hash values, and saved 
file names.

Results of both functions are returned as a string.

If <i>console</i> = TRUE (the default), results are displayed in the console.

If <i>save</i> = TRUE, results are saved to the file <i>prov-trace.txt</i>.

The <i>save.dir</i> parameter determines where the results file is saved. 
If NULL (the default), the R session temporary directory is used. If a period (.),
the current working directory is used. Otherwise the directory specified by
<i>save.dir</i> is used.

If <i>check</i> = TRUE (the default), each file recorded in the provenance is 
checked against the user's file system.  A dash (-) in the output indicates 
that the file no longer exists, a plus (+) indicates that the file exists 
but the hash value has changed, and a colon (:) indicates that the file exists
and the hash value is unchanged. If <i>check</i> = FALSE, no comparison is made.

## Example

In this example, three R scripts are used to gap fill, harmonize, and combine
data from two meteorological stations to create a single dataset. The script
names are contained in the file "update-hf300.txt".

In the first case, the <i>prov.trace.run</i> function is used to run the scripts, 
collect provenance, and display summary file information.

```
prov.trace.run("update-hf300.txt")
```

Console output (below) shows the save message for each script from <i>rdtLite</i>
followed by output from <i>prov.trace.run</i>. Scripts are numbered in the order
of execution. Each line shows the script number, a symbol indicating whether
the file has changed since provenance was collected, and the file path and name.

```
[1] "Saving prov.json in C:/Prov/prov_gap-fill-shaler"
[1] "Saving prov.json in C:/Prov/prov_combine-shaler-fisher"
[1] "Saving prov.json in C:/Prov/prov_calculate-hf-annual-monthly"

SCRIPTS:

1 : C:/TraceR/gap-fill-shaler.R 
2 : C:/TraceR/combine-shaler-fisher.R 
3 : C:/TraceR/calculate-hf-annual-monthly.R 

INPUTS:

1 : C:/TraceR/amherst-ma-1964-2002.csv 
1 : C:/TraceR/bedford-ma-1964-2002.csv 
1 : C:/TraceR/hf000-02-daily-e.csv 
2 : C:/TraceR/hf001-06-daily-m.csv 
2 : C:/TraceR/hf001-08-hourly-m.csv 

OUTPUTS:

1 : C:/TraceR/hf-shaler-gap-filled.csv 
2 : C:/TraceR/hf-shaler-fisher-overlap.csv 
2 : C:/TraceR/hf300-05-daily-m.csv 
2 : C:/TraceR/hf300-06-daily-e.csv 
3 : C:/TraceR/hf300-01-annual-m.csv 
3 : C:/TraceR/hf300-02-annual-e.csv 
3 : C:/TraceR/hf300-03-monthly-m.csv 
3 : C:/TraceR/hf300-04-monthly-e.csv 

EXCHANGES:

1 > 2 : C:/TraceR/hf-shaler-gap-filled.csv 
2 > 3 : C:/TraceR/hf300-05-daily-m.csv
```

In the second case, the <i>prov.trace</i> function is used to display detailed
file information contained in the provenance without running the scripts.

```
prov.trace("update-hf300.txt", file.details=TRUE)
```

For each file, the console output (below) shows the file timestamp, the file 
hash value and algorithm, and the path and name of the saved copy of the file
on the provenance directory. For scripts the execution time stamp is also 
shown.

```
SCRIPTS:

1 : C:/TraceR/gap-fill-shaler.R 
        Timestamp: 2019-10-19T09.42.45EDT 
        Hash:      9ab73da3681ae9cbe85efb912550e432 / md5 
        Saved:     C:/Prov/prov_gap-fill-shaler/scripts/gap-fill-shaler.R 
        Executed:  2020-07-08T10.21.30EDT 

2 : C:/TraceR/combine-shaler-fisher.R 
        Timestamp: 2019-10-19T09.41.59EDT 
        Hash:      848a20e2696b1fb7c9bdeec27df059f5 / md5 
        Saved:     C:/Prov/prov_combine-shaler-fisher/scripts/combine-shaler-fisher.R 
        Executed:  2020-07-08T10.21.35EDT 

3 : C:/TraceR/calculate-hf-annual-monthly.R 
        Timestamp: 2019-10-19T10.16.12EDT 
        Hash:      213661ba5f7e4de68d2205c9fe8c0922 / md5 
        Saved:     C:/Prov/prov_calculate-hf-annual-monthly/scripts/calculate-hf-annual-monthly.R 
        Executed:  2020-07-08T10.21.41EDT 


INPUTS:

1 : C:/TraceR/amherst-ma-1964-2002.csv 
        Timestamp: 2019-10-16T10.51.53EDT 
        Hash:      06c82be1ceeec8f41216ee670f485d77 / md5 
        Saved:     C:/Prov/prov_gap-fill-shaler/data/2-amherst-ma-1964-2002.csv 

1 : C:/TraceR/bedford-ma-1964-2002.csv 
        Timestamp: 2019-10-17T10.43.55EDT 
        Hash:      d7f8e08fd84f4b75941325cd82ca7768 / md5 
        Saved:     C:/Prov/prov_gap-fill-shaler/data/3-bedford-ma-1964-2002.csv 

1 : C:/TraceR/hf000-02-daily-e.csv 
        Timestamp: 2019-10-16T10.37.42EDT 
        Hash:      e9f67f7074eb68059385c683d0410c01 / md5 
        Saved:     C:/Prov/prov_gap-fill-shaler/data/1-hf000-02-daily-e.csv 

2 : C:/TraceR/hf001-06-daily-m.csv 
        Timestamp: 2020-06-01T09.07.21EDT 
        Hash:      5e515ea3e7080543fba92b9b9114810f / md5 
        Saved:     C:/Prov/prov_combine-shaler-fisher/data/2-hf001-06-daily-m.csv 

2 : C:/TraceR/hf001-08-hourly-m.csv 
        Timestamp: 2019-10-17T11.34.57EDT 
        Hash:      af36c84e4c0b8f72632eba5661506129 / md5 
        Saved:     C:/Prov/prov_combine-shaler-fisher/data/3-hf001-08-hourly-m.csv 


OUTPUTS:

1 : C:/TraceR/hf-shaler-gap-filled.csv 
        Timestamp: 2020-07-08T10.21.34EDT 
        Hash:      a5022c912b1ec50e8cd4c20d8ed636cf / md5 
        Saved:     C:/Prov/prov_gap-fill-shaler/data/4-hf-shaler-gap-filled.csv 

2 : C:/TraceR/hf-shaler-fisher-overlap.csv 
        Timestamp: 2020-07-08T10.21.40EDT 
        Hash:      f7334fb30cf16c566f8e1de2b7643cf2 / md5 
        Saved:     C:/Prov/prov_combine-shaler-fisher/data/6-hf-shaler-fisher-overlap.csv 

2 : C:/TraceR/hf300-05-daily-m.csv 
        Timestamp: 2020-07-08T10.21.39EDT 
        Hash:      1c9eabddcd5474e11e36168234a1cfae / md5 
        Saved:     C:/Prov/prov_combine-shaler-fisher/data/4-hf300-05-daily-m.csv 

2 : C:/TraceR/hf300-06-daily-e.csv 
        Timestamp: 2020-07-08T10.21.40EDT 
        Hash:      e463c55ff22f56c2fe5e7a69758d3339 / md5 
        Saved:     C:/Prov/prov_combine-shaler-fisher/data/5-hf300-06-daily-e.csv 

3 : C:/TraceR/hf300-01-annual-m.csv 
        Timestamp: 2020-07-08T10.21.42EDT 
        Hash:      e4969c413d3abce641335ad418b51f5c / md5 
        Saved:     C:/Prov/prov_calculate-hf-annual-monthly/data/2-hf300-01-annual-m.csv 

3 : C:/TraceR/hf300-02-annual-e.csv 
        Timestamp: 2020-07-08T10.21.42EDT 
        Hash:      cc99e71c31fd4696d99e68e724497dc5 / md5 
        Saved:     C:/Prov/prov_calculate-hf-annual-monthly/data/3-hf300-02-annual-e.csv 

3 : C:/TraceR/hf300-03-monthly-m.csv 
        Timestamp: 2020-07-08T10.21.42EDT 
        Hash:      de8267dba4643b5d174d4a3140bd9414 / md5 
        Saved:     C:/Prov/prov_calculate-hf-annual-monthly/data/4-hf300-03-monthly-m.csv 

3 : C:/TraceR/hf300-04-monthly-e.csv 
        Timestamp: 2020-07-08T10.21.42EDT 
        Hash:      bf6841b2b01b81c87b30f843b6dda0b1 / md5 
        Saved:     C:/Prov/prov_calculate-hf-annual-monthly/data/5-hf300-04-monthly-e.csv 


EXCHANGES:

1 > 2 : C:/TraceR/hf-shaler-gap-filled.csv 
        Timestamp: 2020-07-08T10.21.34EDT 
        Hash:      a5022c912b1ec50e8cd4c20d8ed636cf / md5 
        Saved out: C:/Prov/prov_gap-fill-shaler/data/6-hf-shaler-fisher-overlap.csv 
        Saved in:  C:/Prov/prov_combine-shaler-fisher/data/1-hf-shaler-gap-filled.csv 

2 > 3 : C:/TraceR/hf300-05-daily-m.csv 
        Timestamp: 2020-07-08T10.21.39EDT 
        Hash:      1c9eabddcd5474e11e36168234a1cfae / md5 
        Saved out: C:/Prov/prov_combine-shaler-fisher/data/4-hf300-03-monthly-m.csv 
        Saved in:  C:/Prov/prov_calculate-hf-annual-monthly/data/1-hf300-05-daily-m.csv 
```

In both cases, the colon after the script number for each file indicates 
that the file has not changed since the provenance was collected.

