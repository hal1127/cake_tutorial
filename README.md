# CakePHP チュートリアル

## プロジェクトの作成
まずは、CakePHPのプロジェクトを作成しましょう。ターミナルに
```bash
composer create-project --prefer-dist "cakephp/app:^3.8" myapp
```
と入力することで、`myapp`ディレクトリ内にCakePHPのプロジェクトを作成することができます。

## サーバーの起動
プロジェクトが作成できましたので、
```bash
cd myapp
```
でプロジェクトのディレクトリに入り、
```bash
bin/cake server
```
で開発用サーバーを立ち上げることができます。 http://localhost:8765 に入り、ページを確認してみましょう。
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

`greet`関数の中身は今は何もかく必要はありません。

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


