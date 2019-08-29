---
title: Views
description: Views Guide for the OriginPHP Framework
extends: _layouts.documentation
section: content
---
# Views

The V in the Model View Controller (MVC). The controller has dealt with the request and probably used a model to generate
data which can be sent to the view, then the view will now display this to user.

## Rendering and Layouts

This framework favors  'convention over configuration' and views are a good example of this, if a user requests `/articles/latest`, the articles controller will be loaded and the view called latest will be rendered. You pass data to the view from the controller using the `set` method.

```php
class ArticlesController extends AppController
{
    public function latest(){
        $articles = $this->Article->find('all',[
            'order'=>'created DESC','limit' =>5
            ]);
        $this->set('articles',$articles);
    }
}
```

The file for the view is `app/View/Articles/latest.ctp` and might look something like this:

```php
<h1>Latest</h1>
<ul>
<?php
    foreach($articles as $article){
        ?>
            <li><?= $article->title ?></li>
        <?php
    }
?>
</ul>
```

That view will be rendered inside a `layout`, the framework comes with two starter layouts `default` and `basic` which can be found in the `app/View/Layout` folder.

```html
<!doctype html>
<html lang="en">
  <head>
    <title><?= $this->title(); ?></title>
    <link rel="stylesheet" href="/css/default.css">
  </head>
  <body>

    <div class="container">
      <?= $this->Flash->messages() ?>
      <?= $this->content() ?>
    </div>
  </body>
</html>
```

To render a different layout change the name of the `layout` property in the controller, or if you do not want to use a layout set the property to `false`.

```php
class ArticlesController extends AppController
{
   public $layout = 'default';
}
```

You can also access the `request` and `response` objects from within your views.

Views are rendered automatically unless you set the `autoRender` property in the controller to false.

## Rendering a different view

If you want to render a different view call the render function with the name of the view that you want to render.

```php
class ArticlesController extends AppController
{
    public function something_else(){
        $articles = $this->Article->find('all',['order'=>'created DESC','limit' =>5]);
        $this->set('articles',$articles);
        $this->render('latest');
    }
}
```

You can also render views in a different directory, you just have to start the name with a forward slash.

```php
$this->render('/Bookmarks/latest');
```

For example, if you called render from the articles controller, `latest` would be the same as `/Articles/latest`.

To render views from `Plugins`, you can use dot notation , followed by the view folder (controller name) and then the action.

```php
$this->render('MyPlugin.Contacts/index');
```

## Rendering Options

The normal views are typically used for HTML but you can use how you see fit.

### Rendering XML

You can render XML setting the xml option and either pass

- string: An existing xml string that you have loaded or generated from elsewhere.
- array: An will be converted to xml using the Xml utility.
- result: you can return a result or set of results and these be be converted to xml.

```php
class ArticlesController extends AppController
{
    public function latest(){
        $articles = $this->Article->find('all',['order'=>'created DESC','limit' =>5]);
        $this->render(['xml'=>$articles]);
    }
}
```

### Rendering JSON

You can render JSON by passing an array with the key json, the value can be either:

- result: you can return a result or set of results and these be be converted to xml.
- anything else: any value that you pass will go through the `json_encode` function.

```php
class ArticlesController extends AppController
{
    public function latest(){
        $articles = $this->Article->find('all',['order'=>'created DESC','limit' =>5]);
        $this->render(['json'=>$articles]);
    }
}
```

### Rendering Text

Sometimes you need to just render a text response, maybe for an ajax or web service request.

```php
$this->render(['text'=>'OK']);
```

### Rendering an external file

If you need to load an external file you can pass the file option, this is different from a view, since it just loads the contents using `file_get_contents`.

```php
$this->render(['file'=>'/var/www/tmp/prerenderedfile.html']);
```

### Setting Status Codes

You can change the status code sent to browser, for example 404. By setting this on the `response` object or by passing a `status` option.

Both ways are demonstrated here.

```php
class ArticlesController extends AppController
{
    public function view($id = null)
    {
        $json = ['errors'=>['message' =>'Not Found']];
        $this->render(['json'=>$json,'status'=>404]);
    }
    public function anotherWay(){
         $this->render(['status'=>404]);
    }
     public function latest(){
        $this->response->status(404);
    }
}
```

There are quite a lot of status codes, but APIs only use a small subset, here are probably the main ones.

| Code      | Definition                                              |
| ----------|---------------------------------------------------------|
| 200       | OK (Success)                                            |
| 400       | Bad Request (Failure - client side problem)             |
| 500       | Internal Error (Failure - server side problem)          |
| 401       | Unauthorized                                            |
| 404       | Not Found                                               |
| 403       | Forbidden (For application level permissions)           |

To status code is just sent to the browser, if you are rending HTML it will be make no difference, if its json and you use ajax queries it will.

If you want to set the code and generate an error page for the status, then you should throw an exception instead.

```php
use Origin\Exception\BadRequestException;
class ArticlesController extends AppController
{
    function backdoor(){
        throw new BadRequestException('Bad Request');
    }
}
```

Exceptions for each of the above status codes are available as well as few other ones can be found in the `origin/Exception` folder. If you have `debug` set to true in your bootstrap file then you will see the error on screen, if not then either 500 or 404 exception will be thrown and the details will be sent to the application.log file in the `logs` folder.

## Helpers

Helpers allow you share code between views, similar to the controller components. To load helpers call the `loadHelper` method in the `initialize` method of your controller.

```php
public function initialize(){
    $this->loadHelper('TableMagic');
}
```

The following helpers come with OriginPHP:

- [Html Helper](/docs/view/html-helper)
- [Form Helper](/docs/view/form-helper)
- [Date Helper](/docs/view/date-helper)
- [Number Helper](/docs/view/number-helper)
- [Cookie Helper](/docs/view/cookie-helper)
- [Session Helper](/docs/view/session-helper)

For more information on this see the [helpers guide](/docs/view/helpers).

## Elements

Sometimes you might use the same block of code inside multiple views, in this case you would want to use elements which are stored in `View/Element` and end with a `.ctp` extension.

Create a file  `View/Element/widget.ctp`

```php
<h2>Widget</h2>
<p>What is 1 + 1 ? <?= $answer ?></p>
```

Now anytime you want to use that element, you can also pass an array options where the data will be converted into variables with the names taken from the key value.

```php
 echo $this->element('widget',['answer'=>2]);
``