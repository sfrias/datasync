---
layout: with-sidebar
title: DataSync Library/SDK (Java)
bodyclass: homepage
---

DataSync 1.5.3 and higher can be used as a Java library/SDK. This guide documents how to do this as well as provides example code. If you are new to DataSync you should first read about using DataSync in [GUI mode]({{ site.root }}/guides/setup-standard-job.html) or [command-line/headless mode]({{ site.root }}/guides/setup-standard-job-headless.html).

# Importing Into Your Java Project

## Import into an Eclipse Project

1. [Download the latest JAR](https://github.com/socrata/datasync/releases) (must use DataSync 1.5.3 or higher)
2. Follow [this guide](https://wiki.eclipse.org/FAQ_How_do_I_add_an_extra_library_to_my_project%27s_classpath%3F) to import the JAR into your project's classpath.


# Using the DataSync Library/SDK

## Quickstart

Once you have the [DataSync JAR (1.5.3 or higher)](https://github.com/socrata/datasync/releases) imported into your project's classpath you can use this code below to get the library working. 

Be sure to update the following in the code below:  

* `DOMAIN` (e.g. https://data.cityofchicago.org)
* `USERNAME`
* `PASSWORD`
* `APP_TOKEN`
* `/path/to/file.csv`
* `DATASET_ID` (e.g. jhu8-6jm5)

Here is the starter code (import statements excluded for conciseness):  

    public class Main {
        public static void main(String args[]) {
            // Configure User Preferences
            UserPreferencesLib userPrefs = new UserPreferencesLib()
                    .domain("DOMAIN")
                    .username("USERNAME")
                    .password("PASSWORD")
                    .appToken("APP_TOKEN")
                    .load();

            // Configure job to use replace via HTTP
            IntegrationJob job = new IntegrationJob(userPrefs);

            job.setFileToPublish("/path/to/file.csv");
            job.setDatasetID("DATASET_ID");
            job.setPublishViaDi2Http(true); // we strongly recommend using the 'HTTP' update method

            // Set up the ControlFile to control how the file to publish is interpreted
            // ControlFile parameters can be configured using the FileTypeControl object's setter methods
            // Acceptable values are documented in detail here:
            // http://socrata.github.io/datasync/resources/control-config.html
            String publishMethod = "replace"; // valid values: "replace", "upsert", "delete"
            String[] dateTimeFormatsAllowed = {"ISO8601", "MM/dd/yy", "MM/dd/yyyy", "dd-MMM-yyyy"};
            FileTypeControl csvControl = new FileTypeControl()
                    .separator(",")
                    .fixedTimestampFormat(dateTimeFormatsAllowed)
                    .floatingTimestampFormat(dateTimeFormatsAllowed);

            ControlFile controlFile = new ControlFile(publishMethod, null, csvControl, null);
            job.setControlFile(controlFile);

            // Run the job
            JobStatus status = job.run();

            // Output result of job execution
            System.out.println(status.getMessage());
        }
    }


## Configuring the job

The control file settings are used to help DataSync interpret the data within the CSV. In many cases using the default configuration will be sufficient for the job to run successfully. The cases where you will need to modify the Control file content include, but are not limited to:

* If your CSV contains date/time data in a format other than: [ISO8601](http://en.wikipedia.org/wiki/ISO_8601), MM/dd/yyyy, MM/dd/yy, or dd-MMM-yyyy (e.g. "2014-04-22", "2014-04-22T05:44:38", "04/22/2014", "4/22/2014", "4/22/14", and "22-Apr-2014" would all be fine).
* The Socrata dataset has a Location column that will be populated from existing columns (e.g. address, city, state, zipcode).
* The Socrata dataset has a Location column and you are <strong>not</strong> using Socrata's geocoding (i.e. you are providing the latitude/longitude coordinates in the CSV/TSV file).
* If you wish to set the timezone of the dates being imported.

For more detailed information on establishing configuration in the Control file refer to [Control file configuration]({{ site.root }}/resources/control-config.html)

### Configuring without a saved control file

You can use the 'setter' methods within the FileTypeControl class to configure the job. The setter methods reflect the configuration options details in [Control file configuration]({{ site.root }}/resources/control-config.html). For example, to ignore the header row and configure the list of columns use this code:

    String[] columnList = {"column1", "column2"};

    FileTypeControl csvControl = new FileTypeControl()
      .skip(1)
      .columns(columnList);

    ControlFile controlFile = new ControlFile(publishMethod, null, csvControl, null);
    job.setControlFile(controlFile);


#### Location column and geocoding configuration

The syntheticLocations option discussed in more detail in the [Control File documentation]({ site.root }}/resources/control-config.html#location-column-and-geocoding-configuration) allows configuring a Location datatype column to populate from address, city, state, zipcode or latitude/longitude data within existing columns of the CSV/TSV. To configure this use the following code:

    // set up any location columns that need to be populated
    LocationColumn mylocation = new LocationColumn()
            .address("street_fieldname")
            .city("city_fieldname")
            .state("state_fieldname")
            .zip("zipcode_fieldname");

    Map<String, LocationColumn> syntheticLocationToBeCreated = new HashMap<>();
    syntheticLocationToBeCreated.put("location_fieldname", mylocation);

    // set up ControlFile to control how the file to publish is interpreted
    FileTypeControl csvControl = new FileTypeControl()
            .syntheticLocations(syntheticLocationToBeCreated);

    ControlFile controlFile = new ControlFile(publishMethod, null, csvControl, null);
    job.setControlFile(controlFile);

### Loading configuration from a saved control.json file

As an alternative to You can create a `control.json` file, save it, and then import the configuration from the file using the code below, and changing `/path/to/control.json` to point the your saved `control.json` file:


File controlFile = new File("/path/to/control.json");

    try {
        ObjectMapper controlFileMapper =
                new ObjectMapper().enable(DeserializationConfig.Feature.ACCEPT_SINGLE_VALUE_AS_ARRAY);

        ControlFile controlFileFromFile = controlFileMapper.readValue(controlFile, ControlFile.class);

        job.setControlFile(controlFileFromFile);

    } catch (IOException e) {
        // Handle errors parsing JSON in control.json file
        e.printStackTrace();
    }






