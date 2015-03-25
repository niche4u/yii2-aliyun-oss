###yii2-aliyun-oss

阿里云OSS官方SDK的Composer封装，支持Yii2。
基于https://github.com/johnlui/AliyunOSS 修改，感谢johnlui写的这个，对我开发很有帮助，我将这个支持Yii2了，并且开源出来，希望能帮到更多人。
添加删除文件的功能，修改getUrl（只返回文件外链url，不带其他参数）。


###安装

$ php composer.phar require chonder/yii2-aliyun-oss "dev-master"

###Yii2使用

修改config/params.php

添加：
```php
    'oss'=>array(
        'ossServer' => '', //服务器外网地址，深圳为 http://oss-cn-shenzhen.aliyuncs.com
        'ossServerInternal' => '', //服务器内网地址，深圳为 http://oss-cn-shenzhen-internal.aliyuncs.com
        'AccessKeyId' => '', //阿里云给的AccessKeyId
        'AccessKeySecret' => '', //阿里云给的AccessKeySecret
        'Bucket' => '' //创建的空间名
    ),
```

在components中创建Oss.php，内容如下：

```php

namespace app\components;

use chonder\AliyunOss\AliyunOSS;
use Yii;

class OSS {

    private $ossClient;

    public function __construct($isInternal = false)
    {
        $serverAddress = $isInternal ? Yii::$app->params['oss']['ossServerInternal'] : Yii::$app->params['oss']['ossServer'];
        $this->ossClient = AliyunOSS::boot(
            $serverAddress,
            Yii::$app->params['oss']['AccessKeyId'],
            Yii::$app->params['oss']['AccessKeySecret']
        );
    }

    public static function upload($ossKey, $filePath)
    {
        //$oss = new OSS(true); // 上传文件使用内网，免流量费
        $oss = new OSS();
        $oss->ossClient->setBucket(Yii::$app->params['oss']['Bucket']);
        $oss->ossClient->uploadFile($ossKey, $filePath);
    }

    public static function getUrl($ossKey)
    {
        $oss = new OSS();
        $oss->ossClient->setBucket(Yii::$app->params['oss']['Bucket']);
        return preg_replace('/(.*)\?OSSAccessKeyId=.*/', '$1', $oss->ossClient->getUrl($ossKey, new \DateTime("+1 day")));
    }

    public static function delFile($ossKey)
    {
        $oss = new OSS();
        $oss->ossClient->setBucket(Yii::$app->params['oss']['Bucket']);
        $oss->ossClient->delFile($ossKey);
    }

    public static function createBucket($bucketName)
    {
        $oss = new OSS();
        return $oss->ossClient->createBucket($bucketName);
    }

    public static function getAllObjectKey($bucketName)
    {
        $oss = new OSS();
        return $oss->ossClient->getAllObjectKey($bucketName);
    }

}

```


###使用

```php

use app\components\Oss;

OSS::upload('文件名', '本地路径'); // 上传一个文件

echo OSS::getUrl('某个文件的名称'); // 打印出某个文件的外网链接

OSS::createBucket('一个字符串'); // 新增一个 Bucket。注意，Bucket 名称具有全局唯一性，也就是说跟其他人的 Bucket 名称也不能相同。

OSS::getAllObjectKey('某个 Bucket 名称'); // 获取该 Bucket 中所有文件的文件名，返回 Array。

```

###License
除 “版权所有（C）阿里云计算有限公司” 的代码文件外，遵循 [MIT license](http://opensource.org/licenses/MIT) 开源。
