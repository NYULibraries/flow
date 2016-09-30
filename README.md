# flow
a digitization-project tracking tool  

## goals
* develop and deploy a minimal application that supports digitization project tracking and reporting 
* reuse / enhance / extend existing infrastructure where possible
* develop new functionality where needed

## human-centric process steps
* archivist generates a "work order" using ArchivesSpace work-order plugin 
* digitization team runs a work-order processing script that:
  * asks the script user to select the R* `partner` and `collection`
  * reads the work-order 
  * for each line in the work order:
    * `POST`s a `JSON` request to the `rsbe` API to create a `source entity resource (se)`
    * `rsbe` creates an `se` resource and returns the identifier back to the 
    * generates directories with the proper names


## enhancements to infrastructure
* add `work order uuid` to ArchivesSpace work-order plug-in

