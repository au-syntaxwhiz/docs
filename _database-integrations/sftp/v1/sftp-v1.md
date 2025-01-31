---
# -------------------------- #
#      Page & Formatting     #
# -------------------------- #

title: SFTP
keywords: sftp, database integration, etl sftp, sftp etl
permalink: /integrations/databases/sftp
summary: "Connect and replicate data from files on your SFTP server using Stitch's SFTP integration."
snapshot-type: "databases"
show-in-menus: true
no-schema: true

key: "sftp-setup"

# -------------------------- #
#     Integration Details    #
# -------------------------- #

name: "sftp"
display_name: "SFTP"

singer: true
repo-url: "https://github.com/singer-io/tap-sftp"

hosting-type: "none"

this-version: "1"

file-system: true
db-type: "sftp"
# driver: |
#   [](){:target="new"}

# -------------------------- #
#       Stitch Supports      #
# -------------------------- #

certified: true
setup-name: "SFTP"

frequency: "1 hour"
historical: "1 year"
tier: "Standard"

## Stitch features
api-type: "platform.sftp"
versions: "n/a"
ssh: true
ssl: false

## General replication features

anchor-scheduling: true
cron-scheduling: true

extraction-logs: true
loading-reports: true

table-selection: true
column-selection: true
table-level-reset: false

append-only-tables: true
append-only-tables-description: "Unless Primary Keys are defined for the table, Append-Only loading will be used."

## Replication methods

define-replication-methods: false
define-replication-keys: false

log-based-replication-minimum-version: "n/a"
log-based-replication-master-instance: false
log-based-replication-read-replica: false

## Other Replication Methods

key-based-incremental-replication: true
full-table-replication: false

view-replication: false


# -------------------------- #
#      Setup Requirements    #
# -------------------------- #

requirements-list:
# File requirements are in: _data/taps/extraction/file-systems/file-requirements.yml

  - item: |
      **Files that adhere to Stitch's file requirements**:

      {{ site.data.taps.extraction.file-systems.file-requirements.support-table | flatify }}

# -------------------------- #
#     Setup Instructions     #
# -------------------------- #

setup-steps:
  - title: "Whitelist Stitch's IP addresses"
    anchor: "whitelist-stitch-ips"
    content: |
      {% include note.html type="single-line" content="**Note**: This step is required only if your server is behind a firewall that restricts access based on IP addresses." %}

      {% include shared/whitelisting-ips/generic.html %}

  - title: "Configure SSH for the SFTP server"
    anchor: "configure-ssh-for-server"
    content: |
      To use SFTP, you'll need to configure SSH via key-pair authentication. This allows Stitch to use an SSH tunnel to securely connect to your {{ integration.display_name }} SFTP server.

      If key-pair authentication isn't configured, Stitch can still connect using username/password auth. **Note**: This step is required if key-pairs are required to log into the {{ integration.display_name }} server. Alternatively, you can provide a password to use username/password authentication.

    substeps:
      - title: "Retrieve your Stitch public key"
        anchor: "retrieve-your-public-key"
        content: |
          1. {{ app.menu-paths.add-integration | flatify }}
          2. Click the **{{ integration.display_name }}** icon.
          3. The Stitch public key will be at the top of the page that opens.

          Keep this page handy - you'll need it in the next step.

      - title: "Add the public key to authorized_keys"
        anchor: "add-public-key-to-authorized-keys"
        content: |
          Next, you’ll add the public key to the connecting {{ integration.display_name }} user's `authorized_keys` file on your server. This will allow Stitch to authenticate via a public key and connect to the server.

          {% include shared/ssh/ssh-create-linux-user.html no-notification=true include-create-user-steps=false %}

  - title: "Add {{ integration.display_name }} as a Stitch data source"
    anchor: "add-stitch-data-source"
    content: |
      {% include shared/database-connection-settings.html type="general" %}
  
  - title: "Select an encoding format"
    anchor: "select-encoding-format"
    content: |
      The default encoding format is `utf-8`, but you can update it by entering the name of the format in the **Encoding Format** field.

      **Note**: The selected format will be used for all files selected. If a file is encoded in a different format, an error may occur. If you need to replicate data from files with different encoding formats, you should consider creating a different connection for each format.

  - title: "Configure tables"
    anchor: "configure-tables"
    content: |
      Next, you'll indicate which file(s) you want to include for replication. You can include a single file, or map several files to a table. Refer to the [Setup requirements section](#setup-requirements) for info about what Stitch supports for {{ integration.display_name }} files.

      In the following sections, we'll walk you through how to configure a table in Stitch:

      {% for substep in step.substeps %}
      - [Step 5.{{ forloop.index }}: {{ substep.title }}](#{{ substep.anchor }})
      {% endfor %}

    substeps:
      - title: "Define the table's search settings"
        anchor: "define-table-search-settings"
        content: |
          In this step, you'll tell Stitch which files on your {{ integration.display_name }} server you want to replicate data from. To do this, you'll use the **Search Pattern** and **Directory** fields.

        sub-substeps:
          - title: "Define the Search Pattern"
            anchor: "define-search-pattern"
            content: |
              {% include integrations/shared-setup/configure-table-settings-file-server.html type="define-search-pattern" %}

          - title: "Limit file search to a specific directory"
            anchor: "limit-search-to-directory"
            content: |
              {% include integrations/shared-setup/configure-table-settings-file-server.html type="define-search-directory" location="SFTP server" %}

      - title: "Define the table's name"
        anchor: "define-table-name"
        content: |
          {% include integrations/shared-setup/configure-table-settings-file-server.html type="define-table-name" %}

      - title: "Define the table's Primary Key"
        anchor: "define-table-primary-key"
        content: |
          {% include integrations/shared-setup/configure-table-settings-file-server.html type="define-primary-key" %}

      - title: "Specify datetime fields"
        anchor: "specify-datetime-fields"
        content: |
          {% include integrations/shared-setup/configure-table-settings-file-server.html type="specify-datetime-fields" %}

      - title: "Configure additional tables"
        anchor: "configure-additional-tables"
        content: |
          {% include integrations/shared-setup/configure-table-settings-file-server.html type="configure-additional-tables" %}

  - title: "Define the historical sync"
    anchor: "define-historical-sync"
    content: |
      {% include integrations/shared-setup/configure-table-settings-file-server.html type="define-historical-sync" %}

  - title: "Create a replication schedule"
    anchor: "create-replication-schedule"
    content: |
      {% include integrations/shared-setup/replication-frequency.html %}

  - title: "Select data to replicate"
    anchor: "setting-data-to-replicate"
    content: |
      {% include integrations/shared-setup/data-selection/object-selection.html %}


# -------------------------- #
#      Replication Info      #
# -------------------------- #

replication-sections:
  - content: |
      In this section:

      {% for section in page.replication-sections %}
      {% if section.title %}
      - [{{ section.summary | flatify }}](#{{ section.anchor }})
      {% endif %}
      {% endfor %}

  - title: "Extraction"
    anchor: "extraction-details"
    summary: "Details about object and data type discovery and selecting data for replication"
    content: |
      For every table set to replicate, Stitch will perform the following during Extraction:

      {% for subsection in section.subsections %}
      - [{{ subsection.summary | flatify }}](#{{ subsection.anchor }})
      {% endfor %}

    subsections:
      - title: "Discovery"
        anchor: "extraction--discovery"
        summary: "Discover table schemas and type discovered columns"
        content: |
          During Discovery, Stitch will:

          {% for sub-subsection in subsection.sub-subsections %}
          - [{{ sub-subsection.summary | flatify }}](#{{ sub-subsection.anchor }})
          {% endfor %}
        sub-subsections:
          - title: "Determining table schemas"
            anchor: "discovery--objects"
            summary: "Determine table schemas"
            example:
              - file-name: "customers-006.csv"
                header-row: "id,name,active"
                included-in-discovery: true

              - file-name: "customers-005.csv"
                header-row: "id,name,active"
                included-in-discovery: true

              - file-name: "customers-004.csv"
                header-row: "id,name,active"
                included-in-discovery: true

              - file-name: "customers-003.csv"
                header-row: "id,name,active"
                included-in-discovery: true

              - file-name: "customers-002.csv"
                header-row: "id,name,active"
                included-in-discovery: true

              - file-name: "customers-001.csv"
                header-row: "id,name,has_magic,active"
                included-in-discovery: "<strong>false</strong>"
            content: |
              At the start of each replication job, Stitch will analyze the header rows in the five most recently modified files returned by the table's [search pattern](#define-table-search-pattern). The header rows in these files are used to determine the table's schema.

              For this reason, the structure of files replicated using {{ integration.display_name }} should be the same for every file included in a table's configuration. If the header row in an included file changes after the fifth most recently modified file, Stitch will not detect the difference.

              For example: Based on the files in the table below, the table created from these files would have `id`, `name`, and `active` columns. The `has_magic` column in the `customers-001.csv` file will not be detected, as it's not in the five most recently modified files.

              {% assign columns = "included-in-discovery|file-name|header-row" | split:"|" %}

              <table class="attribute-list">
              <tr>
              <td>
              <strong>Updated</strong>
              </td>
              {% for column in columns %}
              <td>
              <strong>{{ column | replace:"-"," " | capitalize }}</strong>
              </td>
              {% endfor %}
              </tr>

              {% for file in sub-subsection.example %}
              <tr>
              <td>
              {{ forloop.index }} {% if forloop.first == true %}(most recent){% endif %}
              </td>
              {% for column in columns %}
              <td>
              {{ file[column] }}
              </td>
              {% endfor %}
              </tr>
              {% endfor %}
              </table>

          - title: "Data typing"
            anchor: "discovery--data-types"
            summary: "Type the data in discovered columns"
            content: |
              To determine data types, Stitch will analyze the first 1,000 rows in the [files included in object discovery](#discovery--objects).

              If a column has been specified as a [datetime column](#specify-datetime-fields), Stitch will attempt to parse the value as a date. If this fails, the column will be loaded as a nullable `STRING`.

              For all other columns, Stitch will perform the following to determine the column's data type:

              1. Attempt to parse the value as an `INTEGER`
              2. If that fails, attempt to parse the value as a `FLOAT`
              3. If that fails, type the column as a `STRING`. **Note**: If a column contains entirely null values, it will be created as an empty column in the destination with a type of `STRING`.

      - title: "Data replication"
        anchor: "extraction--data-replication"
        summary: "Select records (files) for replication"
        content: |
          After discovery is completed, Stitch will move onto extracting data.

          {% include replication/extraction/file-modification-replication-keys.html %}

          To reduce row usage, only include updated records in new files that match a table's [search pattern](#define-table-search-pattern). This will ensure that only updated records are replicated and counted towards your usage.

  - title: "Loading"
    anchor: "loading-details"
    summary: "Details about how data replicated from {{ integration.display_name }} is loaded into a destination"
    record-example-table: |
      <table class="attribute-list">
      <tr>
      {% for field in fields %}
      <td>
      <strong>{{ field }}</strong>
      </td>
      {% endfor %}
      </tr>
      {% for record in job %}
      <tr>
      {% for field in fields %}
      <td>
      {{ record[field] }}
      </td>
      {% endfor %}
      </tr>
      {% endfor %}
      </table>
    content: |
      How data replicated from an {{ integration.display_name }} integration is loaded into your destination depends on two factors:

      1. **If Primary Keys were specified for the table during integration setup.** If Primary Keys aren't specified during setup, Stitch will load data in an Append-Only manner. This means that new records and updates to existing records are appended to the end of the table as new rows.

      2. **If your destination supports upserts, or updating existing rows**. For destinations that support upserts, Stitch uses Primary Keys to de-dupe data during loading. {{ site.data.tooltips.primary-key }}

      **Note**: For Append-Only destinations, data will be loaded in an Append-Only manner regardless of whether a Primary Key is specified during setup.

    subsections:
      - title: "Loading with defined Primary Keys"
        anchor: "loading--de-duped-keys-example"
        content: |
          If the destination supports upserts and Primary Keys are defined during setup, Stitch will use the Primary Keys to de-dupe records during loading.

          This means that existing rows will be overwritten with the most recent version of the row. A record can only have a single unique Primary Key value, ensuring that only one version of the record exists in the destination at a time.

          For example: The following rows are replicated during the initial replication job:

          {% assign job = site.data.dataloading.examples.append-only | where:"job",1 %}
          {% assign fields = "id|name|type" | split:"|" %}

          {{ section.record-example-table | flatify }}

          Before the next job, the file containing these rows is modified. This means that Stitch will replicate the contents of the entire file, including the rows for `Finn` and `Jake` even if they haven't been updated.

          Stitch will use the Primary Key to de-dupe the records, making the table in the destination look similar to the following:

          {% assign job = site.data.dataloading.examples.append-only | where:"job",2 %}
          {% assign fields = "id|name|type" | split:"|" %}

          {{ section.record-example-table | flatify }}

      - title: "Loading without defined Primary Keys"
        anchor: "loading--append-only-example"
        content: |
          If the destination is Append-Only, or if Primary Keys aren't defined during setup, data will be loaded in an Append-Only manner.

          Additionally, Stitch will append a column (`{{ system-column.primary-key }}`) to the table to function as a Primary Key if one isn't defined.

          **Note**: Appending this column will not enable Stitch to de-dupe data, as a unique value is inserted every time a row is loaded, regardless of whether it's ever been replicated before. This means that a record can have multiple `{{ system-column.primary-key }}` values, each of them unique.

          For example: The following rows are replicated during the initial replication job:

          {% assign job = site.data.dataloading.examples.append-only | where:"job","1" %}
          {% assign fields = "__sdc_primary_key|id|name|type" | split:"|" %}

          {{ section.record-example-table | flatify }}

          Before the next job, the file containing these rows is modified. This means that Stitch will replicate the contents of the entire file, including the rows for `Finn` and `Jake` even if they haven't been updated.

          In the destination, the table might now look like the table below. Notice that records for `Finn` and `Jake` have been appended to the end of the table with new `{{ system-column.primary-key }}` values:

          {% assign job = site.data.dataloading.examples.append-only %}
          {% assign fields = "__sdc_primary_key|id|name|type" | split:"|" %}

          {{ section.record-example-table | flatify }}

          **Note**: Querying Append-Only tables requires a different strategy than you might normally use. For instructions and a sample query, check out the [Querying Append-Only tables guide]({{ link.replication.append-only-querying | prepend: site.baseurl }}).
---
{% assign integration = page %}
{% include misc/data-files.html %}
{% include misc/icons.html %}