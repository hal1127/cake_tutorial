# CakePHP チュートリアル

# 1章 ほぼ静的なページ

## プロジェクトの作成
まずは、CakePHPのプロジェクトを作成しましょう。ターミナルに
```bash
composer create-project --prefer-dist "cakephp/app:^3.8" myapp
```
と入力することで、`myapp`ディレクトリ内にCakePHPのプロジェクトを作成することができます。

## サーバーの起動
プロジェクトが作成できましたので、プロジェクトのディレクトリに入り、

```bash
cd myapp
```

開発用サーバーを立ち上げてみましょう。

```bash
bin/cake server
```

この状態で http://localhost:8765 に入り、ページを確認してみましょう。

![最初のページ](https://raw.githubusercontent.com/hal1127/cake_tutorial/main/webroot/img/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-01-02%20004605.png)
このようなページが表示されれば成功です。

このページでは、環境の情報が確認できます。
Databaseの項目が有効でないとなっていますが、この項目はチュートリアルを通して有効にしていきますので問題ありません。

## ほぼ静的なページ
このセクションでは、静的なページを作成しながらCakePHPの基本的な考え方であるMVCの挙動を確認していきましょう。

### MVCとは
MVCとはModel(モデル)、View(ビュー)、Controller(コントローラー)の頭文字を取ったもので、コントローラーがモデルからデータベースを取得し、その情報を元に動的にビューを表示する仕組みです。

## まずはVCから
とは言っても、最初からMVCをフルに使って説明をすると話が難しくなってしましますから、このセクションではビューとコントローラーのみで静的なページを作ってみましょう。

と、いう事でまずは`routes.php`に書き込んでいきましょう。`routes.php`は`config`ディレクトリ内にあります。

```php
<?php
use Cake\Http\Middleware\CsrfProtectionMiddleware;
use Cake\Routing\RouteBuilder;
use Cake\Routing\Router;
use Cake\Routing\Route\DashedRoute;

Router::defaultRouteClass(DashedRoute::class);

Router::scope('/', function (RouteBuilder $routes) {
    $routes->registerMiddleware('csrf', new CsrfProtectionMiddleware([
        'httpOnly' => true,
    ]));

    $routes->applyMiddleware('csrf');

    $routes->connect('/', ['controller' => 'Pages', 'action' => 'display', 'home']);

    $routes->connect('/pages/*', ['controller' => 'Pages', 'action' => 'display']);

    $routes->fallbacks(DashedRoute::class);
});
```

始め上記のようになっている`routes.php`にこのように書き込みましょう。

```php
<?php
use Cake\Http\Middleware\CsrfProtectionMiddleware;
use Cake\Routing\RouteBuilder;
use Cake\Routing\Router;
use Cake\Routing\Route\DashedRoute;

Router::defaultRouteClass(DashedRoute::class);

Router::scope('/', function (RouteBuilder $routes) {
    $routes->registerMiddleware('csrf', new CsrfProtectionMiddleware([
        'httpOnly' => true,
    ]));

    $routes->applyMiddleware('csrf');

    $routes->connect('/', ['controller' => 'Pages', 'action' => 'display', 'home']);

    $routes->connect('/pages/*', ['controller' => 'Pages', 'action' => 'display']);

    // 以下を追記
    $routes->connect('/greet', ['controller' => 'Pages', 'action' => 'greet', 'greet']);

    $routes->fallbacks(DashedRoute::class);
});
```

追記した部分では、 http://localhost:8765/greet というURLと`PagesController`の`greet`と`Pages`ディレクトリ内の`greet.ctp`を繋げる役割をしています。

では、次は`PagesController.php`に書き込んでいきましょう。`PagesController.php`は以下の場所にあります。

```bash
.
└── src
    └── Controller
        └── PagesController.php
```

`PagesController.php`には以下のようになっています。

```php
<?php
namespace App\Controller;

use Cake\Core\Configure;
use Cake\Http\Exception\ForbiddenException;
use Cake\Http\Exception\NotFoundException;
use Cake\View\Exception\MissingTemplateException;

class PagesController extends AppController
{
    public function display(...$path)
    {
        if (!$path) {
            return $this->redirect('/');
        }
        if (in_array('..', $path, true) || in_array('.', $path, true)) {
            throw new ForbiddenException();
        }
        $page = $subpage = null;

        if (!empty($path[0])) {
            $page = $path[0];
        }
        if (!empty($path[1])) {
            $subpage = $path[1];
        }
        $this->set(compact('page', 'subpage'));

        try {
            $this->render(implode('/', $path));
        } catch (MissingTemplateException $exception) {
            if (Configure::read('debug')) {
                throw $exception;
            }
            throw new NotFoundException();
        }
    }
}
```

これを以下のように書き込んでください。

```php
<?php
namespace App\Controller;

use Cake\Core\Configure;
use Cake\Http\Exception\ForbiddenException;
use Cake\Http\Exception\NotFoundException;
use Cake\View\Exception\MissingTemplateException;

class PagesController extends AppController
{
    public function display(...$path)
    {
        if (!$path) {
            return $this->redirect('/');
        }
        if (in_array('..', $path, true) || in_array('.', $path, true)) {
            throw new ForbiddenException();
        }
        $page = $subpage = null;

        if (!empty($path[0])) {
            $page = $path[0];
        }
        if (!empty($path[1])) {
            $subpage = $path[1];
        }
        $this->set(compact('page', 'subpage'));

        try {
            $this->render(implode('/', $path));
        } catch (MissingTemplateException $exception) {
            if (Configure::read('debug')) {
                throw $exception;
            }
            throw new NotFoundException();
        }
    }

    // 以下を追記
    public function greet()
    {

    }
}
```

`greet`関数の中身は今は何も書く必要はありません。

次にビューを作成しましょう。以下の場所に`greet.ctp`を作成しましょう。

```bash
.
└─ src
   └── Template
       └── Pages
           ├── greet.ctp
           └── home.ctp
```

更に`greet.ctp`に以下を書き込みましょう。

```php
<?php
echo "hello, world.";
```

これで、静的なページができました。 http://localhost:8765/greet をひらいてみましょう。

![静的なページ](https://raw.githubusercontent.com/hal1127/cake_tutorial/main/webroot/img/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-01-02%20015939.png)

このようになっていれば成功です。

しかし、これではあまり面白くありません。そのため、`world`の部分を動的に変更してみましょう。`routes.php`を以下のように変更してください。

```php
<?php
use Cake\Http\Middleware\CsrfProtectionMiddleware;
use Cake\Routing\RouteBuilder;
use Cake\Routing\Router;
use Cake\Routing\Route\DashedRoute;

Router::defaultRouteClass(DashedRoute::class);

Router::scope('/', function (RouteBuilder $routes) {
    $routes->registerMiddleware('csrf', new CsrfProtectionMiddleware([
        'httpOnly' => true,
    ]));

    $routes->applyMiddleware('csrf');

    $routes->connect('/', ['controller' => 'Pages', 'action' => 'display', 'home']);

    $routes->connect('/pages/*', ['controller' => 'Pages', 'action' => 'display']);

    // 以下を変更
    $routes->connect('/greet/:name', ['controller' => 'Pages', 'action' => 'greet', 'greet'])
    ->setPass(['name']);

    $routes->fallbacks(DashedRoute::class);
});
```

これでURLからパラメータを取得できるようになりました。

`PagesController.php`に移動して、次のように書き足しましょう。

```php
<?php
namespace App\Controller;

use Cake\Core\Configure;
use Cake\Http\Exception\ForbiddenException;
use Cake\Http\Exception\NotFoundException;
use Cake\View\Exception\MissingTemplateException;

class PagesController extends AppController
{
    // 省略

    public function greet($name = null)
    {
      // 以下を追記
      $this->set(compact('name'));
    }
}
```

これで、URLから受け取ったパラメータをビューに渡すことができました。

次はビューで渡されたパラメータを使って表示を変更しましょう。`greet.ctp`を以下のように書き換えて下さい。

```php
<?php
// 以下を変更
echo "hello, {$name}.";
```

この状態で、 http://localhost:8765/greet/saba を開いてみましょう。

先ほどの`hello, world.`が`hello, saba`に変化したのが分かるでしょうか。察しの通り、URLの末尾を変更することでこの部分は変化します。`oreo`や`rootbeer`に変更して表示してみましょう。

コントローラーで受け取れるのはURLのパラメータだけではありません。`http://localhost:8765/greet/saba?foo=bar`のようなクエリも受け取ることができます。試してみましょう。

`PagesController.php`を以下のように変更してください。

```php
<?php
namespace App\Controller;

use Cake\Core\Configure;
use Cake\Http\Exception\ForbiddenException;
use Cake\Http\Exception\NotFoundException;
use Cake\View\Exception\MissingTemplateException;

class PagesController extends AppController
{
    // 省略

    public function greet($name = null)
    {
      // 以下を追記
      $name2 = $this->request->getQuery('name');

      $this->set(compact('name', 'name2'));
    }
}
```

`greet.ctp`も変更していきましょう。

```php
<?php
echo "hello, {$name}. <br>";
echo "hola, {$name2}.";
```

この状態で `http://localhost:8765/greet/saba?name=yoshi` を開いてみましょう。

> hello, saba. <br>
> hola, yoshi.

と表示されていれば成功です。これで、パラメータとクエリの両方から情報を受け取ることができました。

これでほぼ静的なページは完成です。

言い忘れていましたが、`$name`, `$name2`と同じような変数を使うのは嫌だと言う人は、`PagesController.php`を以下のように変更し、配列をビューに渡しましょう。

```php
<?php
namespace App\Controller;

use Cake\Core\Configure;
use Cake\Http\Exception\ForbiddenException;
use Cake\Http\Exception\NotFoundException;
use Cake\View\Exception\MissingTemplateException;

class PagesController extends AppController
{
    // 省略

    public function greet($name = null)
    {
        // 以下を変更
        $names =[$name, $this->request->getQuery('name')];

        $this->set(compact('names'));
    }
}
```

それに合わせてビューも変更すれば完璧です。

```php
<?php
echo "hello, {$names[0]}. <br>";
echo "hola, {$names[1]}.";
```

そう、ビューには配列も渡すことができるのです。

2章ではモデルを使ってより本格的なサイト作成をしていきましょう。

# 2章 モデルを用いたサイト作成
