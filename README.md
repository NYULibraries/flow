# flow
a digitization-project tracking system

## goals
* develop and deploy a minimal application that supports digitization project tracking and reporting 
* reuse / enhance / extend existing infrastructure where possible
* develop new infrastructure where needed


## 10,000 foot view
* `Archivists` generate `Work Order Files` and give them to `Digitization Teams`
* `Digitization Team Managers` run a script on the `Work Order Files` that create `Unit of Work` directories
* The `Unit of Work` directories contain a tracking URL
* As the `Unit of Work` directories move through the digitization process, `Monitoring Scripts` update the status of the `Unit of Work`  via the tracking URL
* `Digitization Team Managers` and `Project Managers` can use the `Flow Web Application UI` to check the status of `Units of Work`



## human-centric process overview
* An `Archivist` generates a `Work Order file` using the ArchivesSpace work-order plugin 
* The `Archivist` delivers the `Work Order file` to the appropriate `Digitization Team` [1]
* The `Digitization Team Manager` runs a `Work-order-processing script` that:
  * creates a `Unit of Work` directory on the local machine for each `Item` to be digitized
  * places a file with a tracking URL in each `Unit of Work` directory
* The `Digitization Team Manager` assigns work to a `Digitization Team Member`
* The `Digitization Team Member` digitizes the `Item`, placing the digital-object files into the `Unit of Work` directory
* The `Digitization Team Member` moves the `Unit of Work` directory to the `QC` directory
* The `Digitization Team Manager` QCs the `Unit of Work` and, if the content passes, move the `Unit of Work`
A local monitoring script detects the addition of a `Unit of Work` directory to the `QC` directory and updates the
* 
  * asks the `Digitization Manager` to select the R* `partner` and `collection`
  * processes the `Work Order File` line-by-line
    * for each line the script:
      * `POST`s a `JSON` request to the `rsbe` API to create a `source entity resource (se)`
      * `rsbe` creates an `se` resource and returns the `se` URL
      * creates a directory with the `cuid` generates directories with the proper names


## human-centric process steps
* `Archivist` generates a `Work Order` using the ArchivesSpace work-order plugin 
* `Archivist` delivers `Work Order` to appropriate digitization team
* The `Digitization Team Manager` runs a `Work-order-processing script` that:
  * asks the `Digitization Manager` to select the R* `partner` and `collection`
  * processes the `Work Order File` line-by-line
    * for each line the script:
      * `POST`s a `JSON` request to the `rsbe` API to create a `source entity resource (se)`
      * `rsbe` creates an `se` resource and returns the `se` URL
      * creates a directory with the `cuid` generates directories with the proper names

## enhancements to infrastructure
* add `work order uuid` to ArchivesSpace work-order plug-in

[1] [ ] Eric, is this correct? does the `Work Order` go to `Project Manager` first?
