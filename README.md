# flow
a digitization-project tracking system

## goals
* develop and deploy a minimal application that supports digitization project tracking and reporting
* reuse / enhance / extend existing infrastructure where possible
* develop new infrastructure where needed


## 10,000 foot view
* An `Archivist` generates a `Work Order File` and delivers it to the appropriate `Digitization Team`
* The `Digital Content Manager` runs a script on the `Work Order File` that creates `Unit of Work` directories
* Each `Unit of Work` directory contains a `Tracking URL`
* A `Digitization Specialist` processes the `Unit of Work` directory through the `Digitization Steps`
* As the `Unit of Work` is processed, `Monitoring Scripts` update the `Unit of Work` status via the `Tracking URL`
* The `Unit of Work` status is available via a web application: the `Flow Web UI`
* `Digital Content Managers` and `Project Managers` use the `Flow Web UI` to track `Unit of Work` statuses


## 1,000 foot view
* An `Archivist` uses the ArchivesSpace UI to select the `Item`s they want digitized
* The `Archivist` generates a `Work Order File` using the ArchivesSpace work-order plugin
* The `Archivist` delivers the `Work Order File` to the appropriate `Digitization Team` [1]
* The `Digital Content Manager` runs the `Work Order Processor` script that:
  * creates a `Unit of Work` directory on the local machine for each `Item`
  * places the `Tracking URL` for the `Unit of Work` in the `Unit of Work` directory
* The `Digital Content Manager` assigns the `Unit of Work` to a `Digitization Specialist`
* The `Digitization Specialist` digitizes the `Item`, placing the digital-object files into the `Unit of Work` directory
* The `Digitization Specialist` moves the `Unit of Work` directory to the `QC Directory`
* The `Digital Content Manager` performs a Quality Control check on the `Unit of Work`
  * if the `Unit of Work` **passes** the Quality Control check, the `Digitization Manager` moves the `Unit of Work` to the `Upload Directory`
  * if the `Unit of Work` **fails** the Quality Control check, the `Digitization Manager` moves the `Unit of Work` to the `Rejected Directory` [2]
* `Monitoring Script`s watch the `QC` and `Upload` directories and update `Unit of Work` status via the `Tracking URL`

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
    * for each line the `Work Order Processor` script:
      * parses the line to extract the relevant information, e.g.,  the `Component Unique Identifier (cuid)`, and `Archival Object URI` [3]
      * generates the `Digitization ID` per the following template: ```<partner code>_<collection code>_<cuid with '.' replaced with '_'>```
        * e.g.,
	    ```
          Given:
            partner_code    = 'fales'
		    collection_code = 'gcn'
		    cuid            = '231.1234'
		  Then
		    digitization_id = 'fales_gcn_231_1234'
	    ```
      * instantiates an `se` object that belongs to the selected `collection` and saves it using the `rsbe::client` gem
      * checks the status of the save operation
      * creates the `Unit of Work` directory on the local machine named with the `Digitization ID` value
      * creates a subdirectory `.rstar` in the `Unit of Work` directory for storing tracking information
      * writes the `se URL` to a file named ``tracking_url`` in the `Unit of Work/.rstar` directory
	  * e.g.,
	  ```
	  $ find . -type f
	  fales_gcn_231_1234/.rstar/tracking_url
	  ```
* The `Digital Content Manager` assigns the `Unit of Work` to a `Digitization Specialist` [4]
* The `Digitization Specialist` digitizes the `Item`, placing the digital-object files into the `Unit of Work` directory
* The `Digitization Specialist` moves the `Unit of Work` directory to the `QC Directory`
  * the `QC Directory Monitor` runs [5] and:
    * for each `Unit of Work` directory in the `QC Directory`
      * reads the ``tracking_url`` file
      * gets the current status of the `se` by passing the `Tracking URL` to the `rsbe::client` gem
      * if the `step` attribute is not == `qc`, then
        * updates the `JSON` representation, setting the `step` attribute to `qc`
        * `POST`s the updated `JSON` to the `Tracking URL`
* The `Digital Content Manager` QCs the `Unit of Work`
  * if the `Unit of Work` **passes** QC, the `Digitization Manager` moves the `Unit of Work` to the `Upload Directory`
    * the `Upload Directory Monitor` runs [5] and:
    * for each `Unit of Work` directory in the `Upload Directory`
      * reads the ``tracking_url`` file
      * uses the `rsbe::client` gem to look up the current state of the `se`
	  * sets the `se` attributes \[`phase`, `step`, `status`\] = \[`upload`, `packaging`, `queued`\]
	  * saves  the `se` object, thus updating the `se` state in the `rsbe` application
  * if the `Unit of Work` **fails** QC, the `Digitization Manager` moves the `Unit of Work` to the `Rejected Directory` [2]
    * the `Rejected Directory Monitor` runs [5] and:
    * for each `Unit of Work` directory in the `Rejected Directory`
      * reads the ``tracking_url`` file
      * uses the `rsbe::client` gem to look up the current state of the `se`
      * sets the `se` attributes \[`phase`, `step`, `status`\] = \[`digitization`, `qc`, `rejected`\]
	  * saves  the `se` object, thus updating the `se` state in the `rsbe` application

## required development
(see [Pivotal Tracker project](https://www.pivotaltracker.com/n/projects/1362644))

## optional infrastructure enhancements
* add `work order uuid` to ArchivesSpace work-order plug-in

## references/action items
* [directory to state mapping](DIR-TO-STATE-MAPPING.md)
* [1] Eric, is this correct? does the `Work Order` go to `Project Manager` first?  
* [2] need to check with `Digitization Teams` to see if the use of a `Rejected Directory` is acceptable  
* [3] TODO: analyze work order file to determine minimal required information, and which information should be part of the SE  
* [4] I propose that we do *not* track this initially
* [5] `fsevent`? `cron` job?
