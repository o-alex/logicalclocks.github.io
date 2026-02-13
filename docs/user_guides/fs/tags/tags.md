# Tags

## Introduction

Hopsworks feature store enables users to attach tags to artifacts, such as feature groups, feature views or training datasets.

A tag is a `{key: value}` pair which provides additional information about the data managed by Hopsworks.
Tags allow you to design custom metadata for your artifacts.
For example, you could design a tag schema that encodes governance rules for your feature store, such as classifying data as personally identifiable, defining a data retention period for the data, and defining who signed off on the creation of some feature.

## Prerequisites

Tags have a schema.
Before you can attach a tag to an artifact and fill in the tag values, you first need to select an existing tag schema or create a new tag schema.

Tag schemas can be defined by Hopsworks administrator in the `Cluster settings` section of the platform.
Schemas are defined globally across all projects.
When users attach tags to an artifact, the tag will be validated against a specific schema.
This allows tags to be consistent no matter the project or the team generating them.

!!! warning "Immutable"
    Tag schemas are immutable.
    Once defined, a tag schema cannot be edited nor deleted.

## Step 1: Define a tag schema

Tag schemas can be defined using the UI wizard in the `Cluster settings` > `Tag schemas` section.
Tag schemas have a name, the name is used to uniquely identify the schema.
You can also provide an optional description.

You can define a schema by using the UI tool or by providing the schema in JSON format.
If you use the UI tool, you should provide the name of the property in the schema, the type of the property, whether or not the property is required and an optional description.

<p align="center">
  <figure>
    <img src="../../../../assets/images/guides/tags/tags_schema_simple.png" alt="UI tag schema definition">
    <figcaption>UI tag schema definition</figcaption>
  </figure>
</p>

The UI tool allows you to define simple not-nested schemas.
For more advanced use cases, more complex schemas (e.g., nested schemas) might be required to fully express the content of a given artifact.
In such cases it is possible to provide the schema directly as JSON string.
The JSON should follow the standard [https://json-schema.org](https://json-schema.org).
An example of complex schema is the following:

```json
{
  "type" : "object",
  "properties" :
  {
    "first_name" : { "type" : "string" },
    "last_name" : { "type" : "string" },
    "age" : { "type" : "integer" },
    "hobbies" : {
        "type" : "array",
        "items" : { "type" : "string" }
    }
  },
  "required" : ["first_name", "last_name", "age"],
  "additionalProperties": false
}
```

Additionally it is also possible to define a single property as tag.
You can achieve this by defining a JSON schema like the following:

```json
{ "type" : "string" }
```

Where the type is a valid primitive type: `string`, `boolean`, `integer`, `number`.

## Step 2: Attach a tag to an artifact

Once the tag schema has been created, you can attach a tag with that schema to a feature group, feature view or training datasets either using the feature store APIs, or by using the UI.

### Using the API

You can attach tags to feature groups and feature views by using the `add_tag()` method of the feature store APIs:

=== "Python"

    ```python
    # Retrieve the feature group
    fg = fs.get_feature_group("transactions_4h_aggs_fraud_batch_fg", version=1)

    # Define the tag
    tag = {
        'business_unit': 'Fraud',
        'data_owner': 'email@hopsworks.ai',
        'pii': True
    }

    # Attach the tag
    fg.add_tag("data_privacy", tag)
    ```

You can see the list of tags attached to a given artifact by using the `get_tags()` method:

=== "Python"

    ```python
    # Retrieve the feature group
    fg = fs.get_feature_group("transactions_4h_aggs_fraud_batch_fg", version=1)

    # Retrieve the tags for this feature group
    fg.get_tags()
    ```

Finally you can remove a tag from a given artifact by calling the `delete_tag()` method:

=== "Python"

    ```python
    # Retrieve the feature group
    fg = fs.get_feature_group("transactions_4h_aggs_fraud_batch_fg", version=1)

    # Retrieve the tags for this feature group
    fg.delete_tag("data_privacy")
    ```

The same APIs work for feature views and training dataset alike.

### Using the UI

You can attach tags to feature groups and feature views directly from the UI.
You can navigate on the artifact page and click on the `Add tags` button.
From there you can select the tag schema of the tag you want to attach and populate the values as shown in the gif below.

<p align="center">
  <figure>
    <img src="../../../../assets/images/guides/tags/tags_ui.gif" alt="Attach tag to a feature group">
    <figcaption>Attach tag to a feature group</figcaption>
  </figure>
</p>

## Step 3: Search

Hopsworks indexes the tags attached to feature groups, feature views and training datasets.
The tags will then be searchable using the free text search box located at the top of the UI.

<p align="center">
  <figure>
    <img src="../../../../assets/images/guides/tags/search_ui.gif" alt="Search for tags in the feature store">
    <figcaption>Search for tags in the feature store</figcaption>
  </figure>
</p>

## Mandatory Tags

!!! info "Enterprise"
    Mandatory tags are available only in Hopsworks Enterprise.

Administrators can designate certain tag schemas as **mandatory** for feature groups, feature views, and/or training datasets.
When a tag schema is mandatory for an artifact type, every artifact of that type must have that tag attached with valid values.
This provides a governance mechanism to ensure that all artifacts carry the metadata your organization requires — for example, data classification, ownership, or retention policies.

### How mandatory tags work

Mandatory tags are enforced at two levels:

- **Cluster-wide:** An administrator can configure mandatory tags in `Cluster Settings`. These apply to all projects on the cluster.
- **Project-specific:** A project administrator can configure additional mandatory tags in `Project Settings`. These apply on top of any cluster-wide mandatory tags.

For each mandatory tag, the administrator selects:

- The **tag schema** to require.
- The **artifact types** it applies to: feature groups, feature views, training datasets, or any combination.

!!! note
    Project-level mandatory tags are additive. They cannot remove or override cluster-wide mandatory tags — they can only add additional requirements for that project.

### Configuring mandatory tags (Administrators)

#### Cluster-wide

Navigate to `Cluster Settings` > `Mandatory tags`. From there, select a tag schema and choose which artifact types it should be required for. Once saved, the mandatory tag applies to all projects on the cluster.

#### Project-specific

Navigate to `Project Settings` > `Mandatory tags`. The process is the same as for cluster-wide settings, but the configuration applies only to the current project.

### Experiencing mandatory tags (Users)

When mandatory tags are configured, users will encounter them in the following ways:

#### In the UI

When creating or editing a feature group, feature view, or training dataset, any mandatory tags will be displayed in the form automatically.
Mandatory tags cannot be removed — they are always shown and must have valid values set.

If an artifact has missing mandatory tags (e.g., because a mandatory tag was added after the artifact was created), a warning indicator will be displayed on the artifact page.

#### In the Python SDK

When you retrieve an artifact that has missing mandatory tags, the SDK will emit a warning:

=== "Python"

    ```python
    fg = fs.get_feature_group("transactions_4h_aggs_fraud_batch_fg", version=1)
    # UserWarning: Feature group 'transactions_4h_aggs_fraud_batch_fg' has missing mandatory tags: ['data_privacy', 'data_owner']
    ```

You can check which mandatory tags are missing on an artifact by using the `missing_mandatory_tags` property:

=== "Python"

    ```python
    fg = fs.get_feature_group("transactions_4h_aggs_fraud_batch_fg", version=1)

    # Check for missing mandatory tags
    fg.missing_mandatory_tags
    # Returns: ['data_privacy', 'data_owner']
    ```

### Attaching mandatory tags

#### At creation time

You can attach tags when creating an artifact by passing a `tags` parameter. This is the recommended approach for mandatory tags, as it ensures the artifact is compliant from the start.

=== "Python"

    ```python
    fg = fs.create_feature_group(
        name="transactions_4h_aggs_fraud_batch_fg",
        version=1,
        primary_key=["cc_num"],
        event_time="datetime",
        tags={
            "data_privacy": {
                "business_unit": "Fraud",
                "data_owner": "email@hopsworks.ai",
                "pii": True
            },
            "data_owner": "fraud-team"
        }
    )
    ```

#### After creation

You can also attach tags to an existing artifact using the `add_tag()` method:

=== "Python"

    ```python
    fg = fs.get_feature_group("transactions_4h_aggs_fraud_batch_fg", version=1)

    # Attach a mandatory tag
    fg.add_tag("data_privacy", {
        "business_unit": "Fraud",
        "data_owner": "email@hopsworks.ai",
        "pii": True
    })
    ```

The same approach works for feature views and training datasets.

### Limitations

- Mandatory tags are an **enterprise-only** feature.
- Mandatory tags are supported on **feature groups**, **feature views**, and **training datasets**. They are not supported on model serving entities.
- Only **administrators** can configure which tags are mandatory (cluster-wide or project-level).
