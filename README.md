Sidus/EAVModelBundle Documentation
==================================

[![SensioLabsInsight](https://insight.sensiolabs.com/projects/621ec123-268d-4a6b-a1d0-03a1e4e84b48/big.png)](https://insight.sensiolabs.com/projects/621ec123-268d-4a6b-a1d0-03a1e4e84b48)

The canonical version of this documentation can be found here:
[https://vincentchalnot.github.io/SidusEAVModelBundle](https://vincentchalnot.github.io/SidusEAVModelBundle)

## Introduction

This bundle allows you to quickly set up a dynamic model in a Symfony project using Doctrine.  
Model configuration is done in Yaml and everything can be easily extended.

> The main feature of this model that exists nowhere else to my knowledge is the possibility to
> [contextualize data](Documentation/09-context.md) based on **as many contextualization axis you need**.

All EAV model allows you to contextualize data based on the language or the country (yes: internationalization is a
subset of data contextualization), some models allows you to contextualize your data based on some channels or scopes
but this particular implementation allows you to manage contextualization axis to conform with your business logic, not
the other way around.

**Main Features:**

- [The model](Documentation/02-model.md) is defined in [Yaml configuration files](#example), it's easy, it's powerful
  and it's versionned.
- [Dynamic forms:](Documentation/05.1-form.md) no more time spent writting form types
- [Data contextualisation](Documentation/09-context.md) with custom context axis (language, region, channel, version...)
- Compatible with many native Symfony components:
  [Form](Documentation/05.1-form.md),
  [Validator](Documentation/06-validate.md),
  [Translator](Documentation/08-translate.md),
  [Serializer](Documentation/10-serialize.md)...

### Documentation index

- [01 - Installation](Documentation/01-install.md)
- [02 - Model configuration](Documentation/02-model.md)
- [03 - Multiple attributes & collections](Documentation/03-multiple.md)
- [04 - Handling Entities](Documentation/04-entities.md)
- [05 - Forms](Documentation/05.1-form.md)
- [06 - Validation](Documentation/06-validate.md)
- [07 - Query](Documentation/07.1-query.md)
- [08 - Translating the model](Documentation/08-translate.md)
- [09 - EAV Contextualisation](Documentation/09-context.md)
- [10 - Serialization](Documentation/10-serialize.md)
- [11 - Extending & customizing the model](Documentation/11-extend.md)
- [12 - Custom classes](Documentation/12-custom_classes.md)
- [13 - Events](Documentation/13-events.md)
- [14 - Extensions](Documentation/14-extensions.md)

### Other documentation entries

- [IDE Autocomplete](Documentation/100-ide_autocomplete.md)
- [Questions & Answers](Documentation/200-questions.md)
- [Performances](Documentation/300-performances.md)


### Looking for something ?

Check the  [Q&A section](Documentation/200-questions.md) and don't hesitate to ask questions in the issues.

### What’s an EAV model

EAV stands for Entity-Attribute-Value

The main feature consists in storing values in a different table than the entities.
Check the confusing and not-so-accurate
[Wikipedia article](https://en.wikipedia.org/wiki/Entity%E2%80%93attribute%E2%80%93value_model)

This implementation is actually more an E(A)V model than a traditional EAV model because attributes are not stored in
the database but in YAML files.

If you're not familiar with the key concepts of the EAV model, please read the following.

### Why using it

- Allowing ultra-fast model design because it's super easy to bootstrap.
- Grouping in the same place the model and the metadata like: form configuration, validation, serialization options...
- Managing single-value attributes and multiple-values attributes the same way and being able to change your mind after
without having to do data recovery.
- Storing contextual values per: locale, country, channel (web/print/mobile), versions.
- Grouping and ordering attributes.
- Easy CRUD: your forms are already configured !

### Why not using it ?

Performances ? Not a real issue because MySQL is not usable for searching in a vast amount of data anyway, be it an EAV
model or a more standard relational model. Solution: Elastic Search: it’s currently optionally supported through the
[Sidus/FilterBundle](https://github.com/VincentChalnot/SidusFilterBundle) with the
[Sidus/ElasticaFilterBundle](https://github.com/VincentChalnot/SidusElasticaFilterBundle)

#### Relational model

If you a have a complex relational model and you plan to use a lots of joins to retrieve data, it might be best to keep
your relational model outside of the EAV model but both can coexists without any problem. However, there is a technical
limitation when using this implementation of the EAV model: There is only one table for the Data entry and one table for
the Values.

### The implementation

We are using Doctrine as it’s the most widely supported ORM by the Symfony community and we’re aiming at a MySQL/MariaDB
implementation only for data storage.

In any EAV model there are two sides
- The model itself: Families (data types), Attributes and Attribute Types.
- The data: The values and the class that contains the values, called here “Data”.

In some implementation the model is stored in the database but here we chose to maintain the model in Symfony service
configuration for several reasons:

- For performance reasons, you always needs to access a lots of components from your model and lazy loading will
generate many unnecessary SQL requests. Symfony services are lazy loaded from PHP cache system which is very very
fast compared to any other storage system.
- For complexity reason: with services, you can always define new services, use injections, extend existing services
and have complex behaviors for your entities.
- A Symfony configuration is easy to write and to maintain and can be versioned, when your model is stored in your
database along with your data you will have a very hard time to maintain the same model on your different environments.
- Because the final users *NEVER* edits the model directly in production, it’s always some expert or some developer
that does it on a testing environment first and we prefer simple yaml configuration files over a complex administration
system that can fail.
- It allows you to add layers of configuration for your special needs, for example you can configure directly some form
options in the attribute declaration.
- Finally, you can split your configuration and organise it in different files if needed be and you can comment it,
which is a powerful feature when your model starts to grow bigger and bigger with hundreds of different attributes.

Families and attributes are services automatically generated from your configuration, attribute types are standard
Symfony services.

### Example

For a basic blog the configuration will look like this:

````yaml
sidus_eav_model:
    families:
        Post:
            attributeAsLabel: title
            attributes:
                title: # Default type is string
                    required: true

                content:
                    type: html # This type is available through the EAVBootstrapBundle

                publicationDate:
                    type: datetime

                publicationStatus:
                    type: choice
                    form_options: # Symfony form options are passed directly
                        choices:
                            draft: Draft
                            published: Published
                            archived: Archived

                author:
                    type: data_selector # This type allows to select an other entity inside the EAV model
                    options:
                        autoload: true # Autoload author when loading Post
                        allowed_families:
                            - Author

                tags:
                    multiple: true
                    form_options:
                        collection_options:
                            sortable: true

                isFeatured:
                    type: boolean

        Author:
            attributeAsLabel: name
            attributes:
                name:
                    required: true

                email:
                    validation_rules: # Validation rules can be applied directly in the model
                        - Email: ~
````

Note that by convention we declare the families in UpperCamelCase and the attributes as lowerCamelCase and we encourage
you to do so.
