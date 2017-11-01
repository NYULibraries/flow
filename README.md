# flow
a digitization-project tracking system

## goals
* develop and deploy a minimal application that supports digitization project tracking and reporting
* reuse / enhance / extend existing infrastructure where possible
* develop new infrastructure where needed


## 10,000 foot view
* An `Archivist` generates a `Work Order File` and delivers it to the appropriate `Digitization Team`
* The `Digital Content Manager` runs a script on the `Work Order File` that creates entries in the `Flow` system and `Unit of Work` directories on the file system
* A `Digitization Specialist` processes the `Unit of Work` directory through the `Digitization Steps`
* As the `Unit of Work` is processed, a `Monitoring Script` updates the `Unit of Work` status in the `Flow system`
* The `Unit of Work` status is available via a web application: the `Flow Web UI`
* `Digital Content Managers` and `Project Managers` use the `Flow Web UI` to track `Unit of Work` statuses

## 1,000 foot view
* An `Archivist` uses the ArchivesSpace UI to select the `Item`s they want digitized
* The `Archivist` generates a `Work Order File` using the ArchivesSpace work-order plugin
* The `Archivist` delivers the `Work Order File` to the `Digitization Team` 
* The `Digital Content Manager` runs the `Work Order Processor` script
  * for each line in the `Work Order File` the script:
    * creates a `Digitization ID` and stores that in a `Source Entity` (`se`) record in `Flow`
    * creates a `Unit of Work` directory, using the `Digitization ID` as the directory name
* The `Digital Content Manager` assigns the `Unit of Work` to a `Digitization Specialist`
* The `Digitization Specialist` digitizes the `Item`, placing the digital-object files into the `Unit of Work` directory
* The `Digitization Specialist` moves the `Unit of Work` directory to the `QC Directory`
* The `Digital Content Manager` performs a Quality Control check on the `Unit of Work`
  * if the `Unit of Work` **passes** the Quality Control check, the `Digitization Manager` moves the `Unit of Work` to the `ToServer` directory
  * if the `Unit of Work` **fails** the Quality Control check, the `Digitization Manager` moves the `Unit of Work` to the `DoubleCheck Directory` 
* A `Directory Monitoring Script` watches various directories and updates `Unit of Work` statuses using the `Digitization ID`s as a key

## 100 foot view
* An `Archivist` uses the ArchivesSpace UI to select the `Item`s they want digitized
* The `Archivist` generates a `Work Order File` using the ArchivesSpace work-order plugin
  * the `Work Order File` is in a `tab separated values` format and contains one line per `Item`
* The `Archivist` delivers the `Work Order File` to the appropriate `Digitization Team`
* The `Digital Content Manager` runs the `Work Order Processor` script that:
  * gets a list of `partners` via the `rsbe::client` gem
  * asks the `Digitization Manager` to select the `partner`
  * gets a list of `collections` that belong to the selected `partner` via the `rsbe::client` gem
  * asks the `Digitization Manager` to select the `collection`
  * processes the `Work Order File` line-by-line
    * for each line in the `Work Order File` the `Work Order Processor` script:
      * parses the line to extract the relevant information, e.g.,  the `Component Unique Identifier (cuid)`, and `Archival Object URI` [3]
      * generates the `Digitization ID` per the following template: ```<partner code>_<collection code>_<cuid> with '.' replaced with '_'>```
        * e.g.,
	    ```
          Given
            partner_code    = 'fales'
		    collection_code = 'gcn'
		    cuid            = '231.1234'
		  Then
		    digitization_id = 'fales_gcn_231_1234'
	    ```
      * instantiates an `se` object that belongs to the selected `collection` and saves it to `Flow` using the `rsbe::client` gem
      * checks the status of the save operation
      * creates the `Unit of Work` directory on the `atkins-SAN` volume using the `Digitization ID` for the directory name
* The `Digital Content Manager` assigns the `Unit of Work` to a `Digitization Specialist`
* The `Digitization Specialist` moves the `Unit of Work` directory to the `Processing` directory and waits 5 minutes to allow the `Directory Monitoring Script` to update the status in `Flow`
* The `Digitization Specialist` moves the `Unit of Work` directory to their local machine
* The `Digitization Specialist` digitizes the `Item`, placing the digital-object files into the `Unit of Work` directory
* The `Digitization Specialist` moves the `Unit of Work` directory to the `QC Directory`
  * the `Directory Monitoring Script` runs and:
    * for each `Unit of Work` directory in the `QC Directory`
      * looks up `se` in `Flow` using the `Digitization ID` 
      * sets the `se` status to `QC`
* The `Digital Content Manager` QCs the `Unit of Work`
  * if the `Unit of Work` **passes** QC, the `Digitization Manager` moves the `Unit of Work` to the `ToServer` directory
    * the `Upload Manager Cron Job` runs and:
      * for each `Unit of Work` directory in the `ToServer` directory
        * looks up the Flow `se` in `Flow` using the `Digitization ID` 
        * runs automated quality control checks on the `Unit of Work`
          * if the automated checks **fail**, the `Unit of Work` is moved into the `DoubleCheck` directory
	  * if the automated checks **pass**, the `Unit of Work` is packaged using the Flow `se` `partner` and `collection` and uploaded to `R*`
	    * if packaging and upload **pass**, the `Unit of Work` directory is moved to the `UploadOK` directory
	    * if packaging and upload **fail**, the `Unit of Work` directory is moved to the `UploadFail` directory

  * if the `Unit of Work` **fails** QC, the `Digitization Manager` moves the `Unit of Work` to the `DoubleCheck` directory

  * every 5 minutes the `Directory Monitoring Script` runs and checks each monitored directory, updating `Flow` based on which `Unit of Work` directories are in each monitored directory.
  
## required development
(see [Pivotal Tracker project](https://www.pivotaltracker.com/n/projects/1362644))

## optional infrastructure enhancements
* add `work order uuid` to ArchivesSpace work-order plug-in

## references/action items
* [directory to state mapping](DIR-TO-STATE-MAPPING.md)

