DictionaryBundle
================
[![Build Status](https://travis-ci.org/KnpLabs/DictionaryBundle.svg)](https://travis-ci.org/KnpLabs/DictionaryBundle)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/KnpLabs/DictionaryBundle/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/KnpLabs/DictionaryBundle/?branch=master)

Are you often tired to repeat static choices like gender or civility in your apps ?


## This is a fork

This bundle is a fork. The motivation for forking it is the following:
- Getting a stable release (2.0)
- Adding Symfony 4 compatibility
- Ensure a future maintenance of the project

As today we still hope that the main project will reborn. There is also a lot of work to achieve
to release all unreleased features.

That's why to install this fork you need to add the following lines to your `composer.json` file.

```json
{
    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/biig-io/DictionaryBundle"
        }
    ]
}
```

## Requirements
- Symfony >= 2.8
- PHP >= 5.6

## Installation
Add the DictionaryBundle to your `composer.json`:
```yaml
{
    "require": {
        "knplabs/dictionary-bundle": "~2.0"
    }
}
```
Register the bundle in ``app/AppKernel.php``

```php
$bundles = array(
    // ...
    new Knp\DictionaryBundle\KnpDictionaryBundle(),
);
```

## Maintainers

You can ping us if you need some reviews/comments/help:

 - [@Nek-](https://github.com/Nek-)
 - [@babeou](https://github.com/babeou)

## Basic usage
Define dictionaries in your config.yml file:
```yaml
knp_dictionary:
    dictionaries:
        my_dictionary:      # your dictionary name
            - Foo           # your dictionary content
            - Bar
            - Baz

```
You will be able to retrieve it through the dictionary registry service:
```php
$container->get('knp_dictionary.registry')->get('my_dictionary');
```
### Dictionary form type

Now, use them in your forms:

```php
use Knp\DictionaryBundle\Form\Type\DictionaryType;

public function buildForm(FormBuilderInterface $builder, array $options)
{
    $builder
        ->add('civility', DictionaryType::class, [
            'name' => 'my_dictionary'
        ])

        // Symfony 2.x syntax
        ->add('civility', 'dictionary', [
            'name' => 'my_dictionary'
        ])
    ;
}
```
The dictionary form type extends the [symfony's choice type](http://symfony.com/fr/doc/current/reference/forms/types/choice.html) and its options.

### Validation constraint

You can also use the constraint for validation. The `value` have to be set.

```php
use Knp\DictionaryBundle\Validator\Constraints\Dictionary;

class User
{
    /**
     * @ORM\Column
     * @Dictionary(name="my_dictionary")
     */
    private $civility;
}
```

## Advanced usage
You can specify the indexation mode of each dictionary
```yaml
knp_dictionary:
    dictionaries:
        my_dictionary:                  # your dictionary name
            type: 'key_value'           # your dictionary type
            content:                    # your dictionary content
                "foo": "foo_value"
                "bar": "bar_value"
                "baz": "baz_value"
```
### Available types
- `value` (default) : Natural indexation
- `value_as_key`: Keys are defined from their value
- `key_value`: Define your own keys
- `callable`: Build a dictionary from a callable

### Callable dictionary
You can create a callable dictionary:
```yaml
knp_dictionary:
    dictionaries:
        my_callable_dictionary:         # your dictionary name
            type: 'callable'            # your dictionary type
            service: 'app.service.id'   # a valid service from your application
            method: 'getSomething'      # the method name to execute
```
Callable dictionaries are loaded with a lazy strategy. It means that the callable
will not be called if you do not use the dictionary.

## Transformers
For now, this bundle is only able to resolve your **class constants**:

```yaml
my_dictionary:
    - MyClass::MY_CONSTANT
    - Foo
    - Bar
```
You want to add other kinds of transformations for your dictionary values ?
Feel free to create your own transformer !

### Add your own transformers

Create your class that implements [TransformerInterface](src/Knp/DictionaryBundle/Dictionary/ValueTransformer/TransformerInterface.php).
Load your transformer and tag it as `knp_dictionary.value_transformer`.
```yaml
services:
    my_bundle.my_namespace.my_transformer:
    	class: %my_transformer_class%
    	tags:
        	- { name: knp_dictionary.value_transformer }
```

## Use your dictionary in twig

You can also use your dictionary in your Twig templates via calling ```dictionary``` function (or filter)

```twig
{% for example in dictionary('examples') %}
    {{ example }}
{% endfor %}
```

But you can also access directly to a value by using the same function (or filter)

```twig
{{ 'my_key'|dictionary('dictionary_name') }}
```

## Faker provider

The KnpDictionaryBundle comes with a [faker provider](https://github.com/fzaninotto/Faker) that can be used to provide a random entry from a dictionary.

### Alice

To register the provider in [nelmio/alice](https://github.com/nelmio/alice), you can follow the [official documentation](https://github.com/nelmio/alice/blob/master/doc/customizing-data-generation.md#add-a-custom-faker-provider-class)

or ...

if you use the awesome [knplabs/rad-fixtures-load](https://github.com/knplabs/rad-fixtures-load) library, the dictionary provider will be automaticaly loaded for you :)

```yaml
App\Entity\User:
    john_doe:
        firstname: John
        latnale: Doe
        city: <dictionary('cities')>
```

## Use dictionary command

You can use the following command to show your app dictionaries easily:
```bash
php app/console knp:dictionary:dump # SF2
php bin/console knp:dictionary:dump # SF3 / SF4 
```

If you want to display only one dictionary, you can set it name in argument
```bash
php app/console knp:dictionary:dump my_dictionary # SF2
php bin/console knp:dictionary:dump my_dictionary # SF3 / SF4 
```

## Create your own dictionary implementation

### Dictionary
Your dictionary implementation must implements the interface [Dictionary](src/Knp/DictionaryBundle/Dictionary/Dictionary.php).
In Symfony >= 3.3, your class will be automatically register as dictionary service.

For older Symfony versions (2.8 - 3.2) you need to add the tag `knp_dictionary.dictionary`.

### Dictionary Factory
You must create a dictionary factory that will be responsible to instanciate your dictionary.

```yaml
services:
    # Syntax Symfony >= 3.3
    App\Dictionary\Factory\MyCustomFactory:
        tags:
            { name: 'knp_dictionary.factory' }
            
    # Syntax Symfony < 3.3
    app.dictionary.factory.my_custom_factory:
        class: App\Dictionary\Factory\MyCustomFactory
        tags:
            { name: 'knp_dictionary.factory' }
```
