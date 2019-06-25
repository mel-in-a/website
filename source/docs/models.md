---
title: Models
description: Model Guide for the OriginPHP Framework
extends: _layouts.documentation
section: content
---
# Models 

## What is a Model
Model is the M in MVC (Model View Controller). Model interacts with the database and contains logic for working with that data.

## Conventions
Models are singular camel cased, for example the model for a user profile is `UserProfile` this you can access from the controller or from a related model.

```php
namespace App\Controller;
class UserProfilesController extends AppController
{
    public function index(){
        $records = $this->UserProfile->find('all');
    }
}
```

Table names should be plural and underscored. For example `user_profiles`. Each table in your database should have a primary key,and it should be named `id`. Foreign keys should be the singular underscored name, for example `user_profile_id`.

Models results for individual rows are stored in objects called Entities. Naming conventions for associated data on the entity is camel cased with the first letter in lower case, and if it is `hasOne` or `belongsTo` then it is singular else if it is a `hasMany` or `hasAndBelongsToMany` then it is plural.

Here is an example of the conventions, the user model `hasOne` user profile and `hasAndBelongsToMany` tags.
```php
$users = $this->User->find('all');
foreach($users as $user){

    $userProfile = $user->userProfile; // hasOne or belongsTo
    echo $userProfile->name;

    $tags = $user->tags; // hasMany or hasAndBelongsToMany
    foreach($tags as $tag){
        echo $tag->name;
    }
}
```

If you add datetime fields called `created` and `modified`, then when you create a record the created field is set and each time you modify this the modified field will be updated.

## Creating a Model

To create a model is easy, simply create a file in `src/Model` folder which is a subclass of the `AppModel`.

```php
<?php

namespace App\Model;

class Product extends AppModel
{

}

```

Remember, you can use the code generation tool to do this for you. The code generation shell will also generate the test and fixture for the model.

```linux
$ bin/console generate model Product
```

For more information see [code generation](/docs/development/code-generation)

Then create the table for the model, which should be lower case, underscored and plural, in this example it would
be called `products`.

```sql
CREATE TABLE `products` (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(80) NOT NULL,
  description TEXT DEFAULT NULL
) ENGINE=InnoDB;
```

In the above we created 3 fields. 

To the create the entity object you can do so like this:

```php
    $product = $this->Product->new([
        'name'=>'X500',
        'description'=>'The latest model, with awesome new features.',
        'catalog'= [
            'name' => 'X Series'
        ]
        ]);

```

You can also do it this way, the difference is this below method does not marshal the data, it sets the data directly on the object.

```php
    $product = $this->Product->new();
    $product->name = 'X500';
    $product->description = 'The latest model, with awesome new features.';
    
    $catalog = $this->Product->Catalog->new();
    $catalog->name = 'X Series';
    $product->catalog = $catalog;
```

See the [Entities guide](/docs/model/entities) for more information.

## Using a different table name

Sometimes you might need to use a different table name, you can overide this with the `table` property.

```php
<?php
class Product extends AppModel
{
    public $table = 'productz';
}
```

## Accessing Models

To access a model from the controller, it is the singular name of the controller.

```php
class ArticlesController extends AppController
{
    public function doSomething(){
        $articles = $this->Article->find('all');
    }
}
```
Lets say the article model is [associated](/docs/model/associations) with comments, you can access the comment model using the Article model.

```php
$comments = $this->Article->Comment->find('all');
```

To load a non related model from a controller

```php
$this->loadModel('Product');
$products = $this->Product->find('all');
```

To access [associated](/docs/model/associations) models from within other models.

```php
class Article extends AppModel
{
    public function doSomething(){
       $comments = $this->Comment->find('all');
    }
}
```

If you need to load a non associated model from within another model you do the following

```php
$Product = $this->loadModel('Product');
$products = $Product->find('all');
```

Models should only be used from within controllers or other models. However, if you need to load any model from anywhere else then you can use the the following code:

```php
  use Origin\Model\ModelRegistry;
  $User = ModelRegistry::get('User');
```


## CRUD

CRUD stands for Create Read Update and Delete.

### Create

When you create a record you will usually take data from the request (i.e. when somebody submits a form) or if you need to create something on the fly. 

```php
class ArticlesController extends AppController
{
    public function add(){
        if($this->request->is('post')){
            $article = $this->Article->new($this->request->data());
            $this->Article->save($article);
        }
    }
}
```
This will take the data from the request and create an article entity.

NOTE: You should create the entity from the model that you want to use for saving data.

The bottom line is you will use new to create the entity which is the object and either populate the data when calling new or add it afterwards like this:

```php
class ArticlesController extends AppController
{
    public function add(){
        if($this->request->is('post')){
            $article = $this->Article->new();
            $article->title = $this->request->data('title');
            $article->description = $this->request->data('description');
            $this->Article->save($article);
        }
    }
}
```

You can also whitelist fields (prevent mass assignment attacks) by telling new which fields are allowed.

```php
  $article = $this->Article->new($data,[
      'fields'=> ['title','description']
      );
```

### Read

Below are few examples of how you can retrieve your data, but for more in-depth information look at the [finding data guide](/docs/model/finding-records).

When you use get it will retrieve a record by the primary key id, if the record does not exist
it will throw an exception. This saves having to repeat code to check if record exists and if not throw
a not found exception. 

```php
    $article = $this->Article->get(1000);
```
If you don't want that behavior then you would use the find method.

```php
    $article = $this->Article->find('first',[
        'conditions'=>[
            'id'=>1000
            ]
        ]);
```

To find the first the record use the first finder

```php
    $first = $this->Article->find('first');
```

The all finder will return multiple records
```php
    $all = $this->Article->find('all',[
        'conditions'=>[
            'created <'=>date('Y-m-d H:i:s')
            ]
        ]);
```
The list finder will return arrays like `['a','b','c']` or `['a'=>'b']` or `['c'=>['a'=>'b']]`
```php
    $list = $this->Article->find('list',['fields'=>['id','name']]);
```

The count finder will return an integer of records.
```php
    $count = $this->Article->find('count',[
        'conditions'=>[
            'owner_id' => 1234,
            'status'=> ['Published']
            ]
        );
```
You can learn more about [finding data](/docs/model/finding-records) in the guide.

### Update

Once you have the record that you want to update use the save method on the model.

```php
    $article = $this->Article->get(1000);
    $article->name = 'New Article Name'
    $this->Article->save($article);
```

If you are processing updated data from the request then the preferred way is using the model `patch` method, this will take the existing entity and then patch it using an array of data, the primary key is automatically added.

The patch method marshals the data, includes some security features and is clever enough to work with associated data as well. 

```php
class ArticlesController extends AppController
{
    public function edit($id){
        $article = $this->Article->new();

        if($this->request->is('post')){
            $article = $this->Article->patch($article,$this->request->data());
            $this->Article->save($article);
        }
    }
}
```

You can also whitelist fields (prevent mass assignment attacks) by telling patch which fields are allowed.

```php
  $article = $this->Article->patch($article,$this->request->data(),[
      'fields'=> ['title','description']
      );
```



You can also update in bulk but this wont trigger callbacks, the first array is of
the fields you want to change and the second argument are the conditions.

```php
    $this->Article->updateAll(['status'=>'Checked'],['status'=>'draft']);
```

### Deleting records

To delete records just pass the entity to the delete method, if there is a primary key set
it will attempt to delete it.

```php
    $article = $this->Article->get(1000);
    $this->Article->delete($article);
```

By default dependent records are deleted, you can disable this by setting the second argument to false when deleting. `hasAndBelongsToMany` are always deleted when you delete a record. You will need to set the `dependent` option when configuring the associations.

```php
    $this->Article->delete($article,false);
```

To disable callbacks `beforeDelete` and `afterDelete` set the third argument to false.

```php
    $this->Article->delete($article,true,false);
```

Or you can delete in bulk but this wont trigger callbacks or delete related records.

```php
    $this->Article->deleteAll(['rating'=>'rubbish','author !='=>'Jim']);
```

## Saving Associated Data

You can save records with with associated data such, simply pass an array to the new method on the article and this will build the entity object for you. 

```php
  $data = [
    'title' => 'How to save data with associated data',
    'author' => [         // belongsTo (singular)
        'name' => 'Jane Smith',
      ],
    'approval' => [     // hasOne (singular)
        'approved_by' => 'Tony'
      ],
    'comments' => [    // Has Many (plural)
        ['text' => 'foo'],
        ['text' => 'bar'],
      ]
  ];
  $article = $this->Article->new($data);
  $this->Article->save($article);
```

By default associated records will be saved, you can disable this by setting associated to false. 

```php
  $article = $this->Article->new($data,[
      'associated'=>false]
      );
```

And you can limit this to certain associations by passing an array with the names of the alias for the model that you want to save for.

```php
  $article = $this->Article->new($data,[
      'associated'=>['Comment']]
      );
```

You can also whitelist fields (prevent mass assignment attacks) by telling new or patch which fields to allow.

```php
  $article = $this->Article->new($data,[
      'fields'=> ['title'],
      'associated'=>[
          'Author'=>[
              'fields'=>['name']
              ]
            ]
        ]
      );
```

### Saving hasAndBelongsToMany


You can save `hasAndBelongsToMany` in two ways

1. Using the primary key of the associated model, e.g `id`

```php

  $data = [
    'id' => 1000, // Article Id
    'tags' => [
        ['id' => 1001],
        ['id' => 1002]
     ]
  ];
 $article = $this->Article->new($data);
 $this->Article->save($article);
```

2. Using the `displayField` of the associated model

```php

   $data = [
    'id' => 1000, // Article Id
    'tags' => [
        ['name' => 'New'],
        ['id' => 'Featured']
     ]
  ];
  $entity = $this->Article->new($data);
  $this->Article->save($data);

```
Saving data through this method is a quick and easy method to save `hasAndBelongsToMany` data. However callbacks are only called when creating the associated model, in this example the Tag model.

If you wish to save extra data to the join table or use callbacks then you should make sure the table has a unique primary key field and then save and delete data directly from the join model.

## Save Options

When saving records you can pass an array of options

- validate: default is true, whether to validate data
- callbacks: default is true. whether to trigger callbacks such as beforeSave etc.
- transaction: default is true, whether to wrap the save in a database transaction 
- associated: default is true. here you can specify an array of associated model names that you want to save data for, or true or false. true will enable saving on the first level of assocations.

## Saving a field

If you need to just save one field, you can also pass an array with options such as whether to turn off validation and callbacks.

```php
    $this->Article->saveField(1024,'title','New Article Title');
```

## Validation

Validating data is very important and can easily be setup. You use the `initialize` method which is called immediately after the construct, its basically so you don't have to overide the `__construct()` method.

```php
class Product extends AppModel
{
    public function initialize(array $config){
        parent::initialize($config); // important to remember to call parent!!
        
        $this->validate('password','notBlank'); // String
     
        $this->validate('username', [    // single rule
            'rule' => 'notBlank',
            'message' => 'This is required'
          ]);
          
        $this->validate('email', [
            ['rule'=>'notBlank'],
            ['rule'=>'email']
        ]);
    }
}
```
For more information on validation see our [Validation Guide](/docs/model/validation).

## Callbacks

Callbacks get called a certain moments during a certain lifecycle, such as creating or saving records, deleting records and finding records.

## Finding Records

- beforeFind
- afterFind

## Saving Records

- beforeValidate
- afterValidate
- beforeSave
- afterSave

## Deleting Records

- beforeDelete
- afterDelete

You can find more information about these in the [Callbacks Guide](/docs/model/callbacks).

Hint: If you are using a good ide like [Visual Studio Code](https://code.visualstudio.com/), when you type in the function name, it will autocomplete the function including the arguments.

## Behaviors

You can extend your models with behaviors and share these across models. Behaviors are similar to [components](/docs/controller/components) and you can find more information by seeing the [behaviors guide](/docs/model/behaviors).
