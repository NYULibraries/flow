# flow
a digitization-project tracking system

## goals
* develop and deploy a minimal application that supports digitization project tracking and reporting 
* reuse / enhance / extend existing infrastructure where possible
* develop new functionality where needed

## human-centric process overview
* An `Archivist` generates a `Work Order file` using the ArchivesSpace work-order plugin 
* The `Archivist` delivers the `Work Order file` to the appropriate `Digitization Team` [1]
* The `Digitization Team Manager` runs a `Work-order-processing script` that:
  * asks the `Digitization Manager` to select the R* `partner` and `collection`
  * reads the `Work Order File`
    * for each line in the `Work Order File` the script:
      * `POST`s a `JSON` request to the `rsbe` API to create a `source entity resource (se)`
      * `rsbe` creates an `se` resource and returns the `se` URL
      * creates a directory with the `cuid` generates directories with the proper names


## human-centric process steps
* `Archivist` generates a `Work Order` using the ArchivesSpace work-order plugin 
* `Archivist` delivers `Work Order` to appropriate digitization team
* `Digitization Manager` team runs a work-order processing script that:
  * asks the script user to select the R* `partner` and `collection`
  * reads the work-order 
  * for each line in the work order:
    * `POST`s a `JSON` request to the `rsbe` API to create a `source entity resource (se)`
    * `rsbe` creates an `se` resource and returns the `se` URL
    * creates a directory with the `cuid` generates directories with the proper names


## enhancements to infrastructure
* add `work order uuid` to ArchivesSpace work-order plug-in

[1] [ ] Eric, is this correct? does the `Work Order` go to `Project Manager` first?
