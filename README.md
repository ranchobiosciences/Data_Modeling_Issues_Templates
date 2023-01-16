# monarch-schemas

Monarch Schema language is a superset of [JSON Schema](https://json-schema.org) specification. JSON Schema is a vocabulary that allows you to annotate and validate JSON documents.

New to JSON Schema? Check out this getting started guide:

https://json-schema.org/learn/getting-started-step-by-step

The Monarch schema language supports additional features, beyond what JSON Schema provides:

- Declare relationships between data (i.e. `links`)
- Hints on how to render objects in a UI (i.e. `uiSettings`)
- Dynamic enumerations and validations
- Extended meta schema validations for superset functionality

Monarch schemas are stored as valid YAML or JSON objects in plaintext `.yaml` or `.json` in this repository. It serves up schemas that inform Kaleido on how to render interfaces for node CRUD operations.

## Branching Model

Four branches of schemas are currently maintained, specific to BMS environent's LDAP configuration:

- `stage/bms-dev`
- `stage/bms-uat`
- `stage/bms-prod`, `master`

## schematool

`schematool` is a stand-alone binary executable that validates, then loads schemas into Monarch from the filesystem (this repository). The source for the `schematool` go package can be found in the Monarch respository at `cggit.pri.bms.com/jacquayj/monarch/schematool`.

It can be built and run as an executable for your target OS and platform, or as a Docker container.

## Build schematool

Clone `monarch` containing `schematool`:

```
$ git clone git@cggit.pri.bms.com/jacquayj/monarch.git
```

From root `monarch` repository directory:

```
$ docker build . -f Dockerfile.schematool -t schematool
```

Or using `go`:

```
$ go build -o ../schematool cggit.pri.bms.com/jacquayj/monarch/schematool
```

Local build from root `monarch` repository directory:
```
$ go build -o ../schematool ./schematool
```
## Update / Add Schemas

```
$ docker run -it \
--network host -e JWT_TOKEN -e MONARCH_PROFILE \
--mount type=bind,source="$(pwd)"/../monarch-schemas,target=/schemas \
-v "$HOME"/.monarch:/.monarch \
schematool \
-schemadir=/schemas \
-endpoint=https://monarch.web-dev.bms.com/api
```

## Delete Schema(s)

**Notice:** This is a dangerous operation, errors in Monarch can occur if links exist to the schema you're deleting. TODO: Throw server-side error if a delete operation breaks things in pre-processing validation.

```
$ docker run -it \
--network host -e JWT_TOKEN -e MONARCH_PROFILE \
--mount type=bind,source="$(pwd)"/../monarch-schemas,target=/schemas \
-v "$HOME"/.monarch:/.monarch \
schematool \
-schemadir=/schemas \
-endpoint=https://monarch.web-dev.bms.com/api \
-delete=goal \
-delete=dataset \
-yes=true
```

## Rename Schema(s)

**Notice:** The rename/delete process will now occur automatically during schema updates. No need to specify a delete flag for renames.

In this example, we'll rename `project` to `goal`.

- First, copy contents of `project.yaml` to new file named `goal.yaml`
- Find and replace all instances of `Project|project` with `Goal|goal` in the new `goal.yaml` file

```
$ docker run -it \
--network host -e JWT_TOKEN -e MONARCH_PROFILE \
--mount type=bind,source="$(pwd)"/../monarch-schemas,target=/schemas \
-v "$HOME"/.monarch:/.monarch \
schematool \
-schemadir=/schemas \
-endpoint=https://monarch.web-dev.bms.com/api
```

# Monarch Schema Language Documentation

## UI settings for Kaleido, and the \_meta schema

Monarch serves up settings within schemas that inform Kaleido on how to render interfaces for node CRUD operations.

### \_meta.yaml

The `_meta` schema instructs Kaleido on which nodes and labels to render in the nav, as well as text labels for communities (a.k.a. hubs).

The object keys in the `hubs` object below relates to the first section of the DNS name. e.g. This URL: https://ctdc.datacommons.web-dev.bms.com will load settings from the `ctdc` key in the `_meta` config object below.

```yaml
id: "https://monarch.web-dev.bms.com/api/schemaref/_meta"

uiSettings:
  headerLabel: Clinical Trial # Hub header label
  globalSearchNodes: Nodes for global searching. If nodes is set, show global search input on UI
  navPrimary: /datasetcatalog/datasets # Path of the primary navigation
  navNodes: # Nodes for top navigation bar
    - dataset # Name of the node
    - file
    - program
    - research_project
    - clinical_trial
    - assay
    - sample
  navItems: # list of items to display on top primary navigation panel. Shown after navHubs
    datamodel: # path value of nav item
      hideInVerticalNav: # hide in vertical navigation menu
      label: Data Model # label of nav item
      uiPath: /datamodel # Where the UI should be mounted in React router
      uiPathFirstNode: /datamodel/graph # First page UI should be mounted in React router
      pages: # items for secondary navigation panel
        graph: # path of secondary nav item
          label: Graps # label of secondary nav item
          icon: ViewList # Icon name from Material UI library https://mui.com/components/material-icons/
        schemaeditor:
          adminAccess: true # allow this item only for admin
          label: Schema Editor
          icon: ViewList # Icon name from Material UI library https://mui.com/components/material-icons/
    query: # path value of nav item
      label: Query Builder # label of nav item

  disableCreate: # (optional) setting to turn off "node specific" create buttons dynamically generated from schemas
    - subject # Node for which create button is turned off

  routerNodes: # valid nodes for application path
    - file
    - subject
  enableAccessRequest: 
    - dataset # Show access request button for node in detail views
  hideRelationships:
    study:
      - file # Hide relationship from rendering in detail views
  hideRelationshipsCreate:
    study:
      - file # Hide relationship from rendering in creation form
```

### Schema-specific UI Settings

`subject.yaml`:

```yaml
# ...
uiSettings:
  icon: add_box # Icon name from Google Fonts https://fonts.google.com/icons
  iconStyle: material-icons-outlined # Style of icon. Outlined, filled, rounded, sharp, two tone
  uiPath: /subjects # Where the UI should be mounted in React router
  plural: Subjects # Plural label for node  
  globalSearchSettings: # Settings for globas search (seearch data by multiple nodes)
    searchFields: # Fields to be searched by globas search
      - dataset_name  
      - identifier
    displayFields: # Display reusult field. Used by priority
      - dataset_name
      - identifier
      - id
  displayField: submitter_id # Used as the display field in UI (modal titles, relationship icon links)
  displaySubField: species # Rendered in 'tag section' in flex detailed view
  defaultSortField: subject_title # Default field for sorting
  primaryPersonField: dataset_owner # User for email notifications
  editValidationUserFields: # List of user field to allow edit node
  nodeIdentifier: id # Field used to identify nodes in the UI
  identifierPrefix: SU # The prefixed used when automatically generating node identifiers, e.g. SU-20210608-1
  fullNodeView: true # Open the node details in full page view rather than flex accordion
  hideAttachments: true # Hide the attachments in both the flex accordion and full node detail view
  createForm: # Node creation form settings
    maxRows: 8 # Max number of rows to render per page in creation wizard
    maxCols: 3 # Max number of columns to render per page in creation wizard
  tableField: # Fields and associated pretty names for rendering node table
    submitter_id: Subject Title
    primary_site: Primary Site
    disease_type: Disease Type
    species: Species
    birth_date: Birth Day
  searchField: # Fields to be searched by filter box
    - submitter_id
    - subject_title
    - primary_site
    - disease_type
    - species
    - birth_date
  expandFilterOptionsByDefault: true # Expand filter options by default
  dataFilterField: # Fields and associated pretty names for drop down filters
    dataset_type:
      label: Data Type  # Label of the filter
      value: 'Yes'  # For single option filter (optional)
      optionLabel: Includes patient data  # Single option label (optional)   
      tooltip: true # Show info icon with tooltip with text from fullNodeRegisterForm field description (optional)
      optionsLimit: 500 # Limit of displayed options (optional, not all filters support this parameter) 
      optionsPromiseLimit: 50000 # limit parameter for data request (request for optionsPromise filter parameter)
    dataset_location_endpoint:
        label: Endpoint # Label of the filter
        tooltip: true  # Show info icon with tooltip with text from fullNodeRegisterForm field description (optional)
        view: dropdown # View Style of filter
        type: endpoint  # Type of a field
        fieldName: dataset_location.endpoint # Custom query fieldName (to be able to query objects in mongo)
    dataset_provider:
      label: Data Provider
    includes_patient: 
      label: Includes Patient
      value: 'Yes'  # value for sigle option checkbox filter
      optionLabel: Includes patient data # label for single option checkbox filter
      optionsLimit: 500 # Limit of displayed options (optional, not all filters support this parameter) 
      optionsPromiseLimit: 50000 # limit parameter for data request (request for optionsPromise filter parameter)
  fullNodeViewForm: # Full page view form - if enable, the popup create node UI is replaced with full page node view
    nameField: dataset_name # name of field, to show as page title
    leftStack: # List of left panel fields groups
      - fields: # Group of fields
          - name: dataset_owner # Field name to display value
            label: Owner # Input label (optionl. If empty, label used from field properties - title)
            inline: true # If true, value is inline with label
    rightStack: # List of right panel fields groups
      - fields:
          - name: tags
            label: Tags
    bottomFields: # List of right panel fields
      - name: data_regulations
        label: Data Regulation
  fullNodeRegisterForm: # Register Data form data - if enabled, the popup create node UI is replaced with full page node create view
    panels: # List of form panels
      - title: Basic Information # Panel Title
        fields: # List of panel fields
          - name: dataset_name # Input field name
            label: Name of Dataset # Input label (optionl. If empty, label used from field properties - title)
            width: 12 # Input width [1..12]. Defines the number of grids the component is going to use  (optional)
            description: Description text for input (optional)
          - name: dataset_description
            label: Description
            width: 12
  horizontalFilterPanel:
    - label: Source # Label of button
      field: endpoint # name of node field to filter
      optionsSourceField: endpoint # get option from data by fields
      options: # List of static filter options
      - label: All
        value: null

    - label: Location
      field: bucket
      optionsSourceField: bucket # get option from data by field
      options:       
      - label: All
        value: null

    - label: File Type
      field: filename
      optionsSourceField: filetype # get option from data by field
      options:
      - label: All
        value: null

    - label: Last Updated
      field: updated_datetime
      filterType: date  # for date filter type, there is specific logit of options values
      options: # All, Past Month, Past Year, Past 3 Years
      - label: All
        value: null
      - label: Past Month
        value: 
          period: months # https://momentjs.com/docs/#/manipulating/
          duration: 1
      - label: Past Year
        value:  
          period: years
          duration: 1
      - label: Past 3 Years
        value:
          period: years
          duration: 3
  join:
    - study
# ...
```

### Alternate Syntax for `schema.uiSettings.tableField`

An alternate syntax for `schema.uiSettings.tableField` was created to allow for a wide range of settings to be applied to the node flex view. For example, adjusting the width of flex view node columns, text wrapping within those columns, and a column flex property.

```yaml
tableField:
  - fieldID: project_title # Field ID
    headerName: Project Title # Field Name
    colFlex: 1 # Set Column Flex Property - needs implementation
    textWrap: true # Should text wrap in table cell? Defaults to false with x-axis scroll bar
    width: "300px" # Set Column Width Property in Flex Detailed View
  - fieldID: created_datetime
    headerName: Created On
    colFlex: 1
    textWrap: true
  - fieldID: investigator
    headerName: Investigator
  - fieldID: institution
    headerName: Institution
    width: "150px"
  - fieldID: research_priority
    headerName: Research Priority
    colFlex: 1
    textWrap: true
    width: "200px"
  - fieldID: ii_on_contact
    headerName: II-ON contact
  - fieldID: project_hypothesis
    headerName: Project Hypothesis
    colFlex: 1
    textWrap: true
```

### Links

`links` in a monarch-schema define relationships between nodes rendered in Kaleido. A graph view and table view of these relationships is available in the [models section of the nav(s) rendered by Kaleido](https://datacommons.web-dev.bms.com/models).

`studies.yaml`:

```yaml
links:
  - name: studies # Name of node to be linked.
    backref: subjects # Reference the relationship of node.
    parent_title: Studies # What's rendered for this node.
    child_title: Subjects # Parent to child relationship.
    label: member_of # Another way to describe multiplicity. Monarch does not utilize label field.
    target_type: study # The singular "node id" of the schema you're linking to.
    multiplicity: many_to_many # Defines the type of relationship. A many to many relationship shows that this node has multiple relationships.
    required: false # A relationship link is not required for an existing node.
  - name: cohorts
    backref: subjects
    parent_title: Cohorts # What's rendered for this node.
    child_title: Subjects
    label: member_of
    target_type: cohort
    multiplicity: many_to_many
    required: false
```

### Properties

`properties` in a monarch-schema will allow you to set the way a node property appears in flex detailed and modal view, as well as validate the property with various JSON Schema rules. A dictionary (also known as a map, hashmap or associative array) is a set of key/value pairs.

`dataset.yaml`:

```yaml
properties:
  # $ref imports objects from other YAML or JSON documents

  # Import all ubiquitous_properties for node
  $ref: "_definitions.yaml#/ubiquitous_properties"

  license_end_date: # Dictionary field name.
    title: License End Date # Dictionary field title to appear in UI flex view.
    description: The license end date for the dataset. # Dictionary field description to appear as accessibility feature in UI flex view.
    type: string # Dictionary field type.
    format: date-time # Type of Date Picker. Options: date-time, date, time
    uiDateFormat: string # The formatting for date value. Note: see moment date patterns (https://momentjs.com/)
```

`studies.yaml`:

```yaml
properties:
  # $ref imports objects from other YAML or JSON documents

  # Import all ubiquitous_properties for node
  $ref: "_definitions.yaml#/ubiquitous_properties"

  study_title: # Dictionary field name.
    title: Study Title # Dictionary field title to appear in UI flex view.
    description: The name of the study. # Dictionary field description to appear as accessibility feature in UI flex view.
    type: string # Dictionary field type.

  study_description:
    title: Description
    description: A brief description of the study being performed.
    type: string

  study_type:
    title: Type
    description: The category that the study falls under
    enum: # Build-in function of JSON Schema to specify controlled selection lists.
      - Longitudinal
      - Retrospective
      - Ex Vivo
      - RWD
      - Rapid Autopsy

  study_objective:
    title: Objective
    description:
      The general objective of the study. What the study hopes to discover
      or determine through testing.
    type: string
    multiline: true # Multiline fields in create modal view for longer descriptions.
    colSpan: 3 # Column span setting for longer description fields in the create modal view.
```

### Ubiquitous Properties

Ubiquitous properties are common to all node types, and typically imported via `$ref: "_definitions.yaml#/ubiquitous_properties"`. They could be considered "system properties", as they are expected by Monarch.

`_definitions.yaml`:

```yaml
ubiquitous_properties:
  type:
    type: string
  id:
    type: string
    description: "Object GUID"
  _id:
    type: string
    description: Database identifer
  created_by:
    title: Created By
    description: "Username or email of user who created the node."
    type: string
  tags:
    title: Keywords
    type: array
    description: Keywords for the nodes
    enum_source:
      type: suggestion
      global:
        true # by default, the suggestion lists will be partiaitoned by node + field.
        # If global is set to true, we want to only partition suggestion list based on field
      enforce:
        "yes" # yes/true/"true": reject any suggestions not registered
        # auto_add: accept any suggestion, add to validation list if not exist
        # false/no/"false": accept any suggestion, don't auto add to validation list
    items:
      type: string
  _icon:
    type: string
  submitter_id:
    type: string
    title: Submitter ID
    description: >
      A project-specific identifier for a node. This property is the calling card/nickname/alias for
      a unit of submission. It can be used in place of the UUID for identifying or recalling a node.
  created_datetime:
    $ref: "#/datetime"
  updated_datetime:
    $ref: "#/datetime"
```

### AppNodes Menu

`appNodes` Creates a menu items in the top left corner of the vertical menu. As many `routes` as needed can be added with `icon` and `label` accessible on hover of primary button icon.

`_meta.yaml`:

```yaml
appNodes:
  icon: Apps
  routes:
    - name: solidtumor
      url: https://solidtumor.datacommons.web-uat.bms.com/subjects
      label: Solid Tumor
      icon: Biotech
```

### Enum Sources

Enum sources allow you to validate dynamic or external enumeration values without having to hard-code them into the schema file. Current implmentations:

**_type: ldap_**

Also see dynamic enum config in UI, for default settings.

```yaml
research_scientist:
  title: Research Scientist
  description: the primary scientist responsible for the study
  type: object
  additionalProperties: true # Specifies the type of values in the dictionary. Dictionary values can be of any type.
  enum_source: # Extension added to fetch dynamic enum values from different sources.
    type: ldap # Type of enum_source, either 'ldap' or 'suggestion'
    filter: (cn=*) # LDAP filter query, overrides default specified in settings UI
    search_base: ou=scientists,dc=example,dc=com # LDAP search base, overrides default specified in settings UI
    return_attrs:
      - uid # Specify return attributes that override settings configured in UI
      - cn
      - dn
    static:
      - Option 1 # Static list of settings to add t
      - Option 2
    enforce: true # Should Monarch enforce validations? Default is true

    # During implementation, we identified a server that required additional join LDAP queries
    # to answer the question "what are the users and their attributes, that belong to group x?"
    # The server didn't support memberOf...

    join_field: uniqueMember # Field to use as foreign key in query to other objects, typically has many values. An LDAP query will be sent for every value.
    join_target_field: members # JSON key to use when embedding results in response
    join_base: "%s" # LDAP search base for the join query, can use golang format syntax, the 'join_field' value is passed in as the only argument
    join_filter: (cn=*) # The LDAP filter to use in the join query
    join_attrs: # Attributes to return in the join query
      - cn
      - uid
    join_project: true # By default (false), join queries will maintain the parent object structure and embed the objects into a new JSON key designated by join_target_field. If this is set to true, only the join objects will be projected into the root "entries" object field, disregarding the parent objects used in the lookup.
```

<img src="/readme-img/ldap-settings.png" height="400px">

**_type: suggestion_**

Suggestions stores previous input values used in multi-select dropdown suggest form control. You can limit the scope/partitioning of values: `node + field` or `field only`.

```yaml
tags:
  title: Keywords
  type: array
  description: Keywords for the nodes
  enum_source:
    type: suggestion
    global:
      true # by default, the suggestion lists will be partiaitoned by node + field.
      # If global is set to true, we want to only partition suggestion list based on field
    enforce:
      "yes" # yes/true/"true": reject any suggestions not registered
      # auto_add: accept any suggestion, add to validation list if not exist
      # false/no/"false": accept any suggestion, don't auto add to validation list
```
