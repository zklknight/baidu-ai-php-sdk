# baidu-ai-php-sdk
百度AI开放平台 PHP SDK composer安装

百度AI sdk 文档地址https://ai.baidu.com/ai-doc/FACE/zk37c1qrv
使用教程和百度的文档一样,把原本的引入方式(require_once 'AipFace.php') 改成composer的命名空间(use baidu\ai\AipFace)引入

composer 安装:
composer require zkl/baidu-ai-php

使用示例(人脸对比):


        use baidu\ai\AipFace;
        use app\common\model\FaceContrastModel;
        use think\facade\Config;

        class BaiduAiFaceService
        {
            private static $config = [];

            public function __construct()
            {
                self::$config = Config::get('baidu.ai');
            }

            /**
             * 人脸对比接口
             * https://ai.baidu.com/ai-doc/FACE/zk37c1qrv#%E4%BA%BA%E8%84%B8%E5%AF%B9%E6%AF%94
             * @param string $image1 通常为手机、相机拍摄的人像图片
             * @param string $image2 证件照片：如拍摄的身份证、工卡、护照、学生证等证件图片
             * @return float
             */
            public static function getFaceContrast(string $image1, string $image2): float
            {
                $image1 = './storage/upload/my/live.jpg';
                $image2 = './storage/upload/my/card.jpg';
                $client = new AipFace(self::$config['app_id'], self::$config['api_key'], self::$config['secret_key']);
                $result = $client->match(array(
                    array(
                        'image' => base64_encode(file_get_contents($image1)),
                        'image_type' => 'BASE64',
                        'face_type' => 'LIVE'
                    ),
                    array(
                        'image' => base64_encode(file_get_contents($image2)),
                        'image_type' => 'BASE64',
                        'face_type' => 'CERT'
                    ),
                ));
                // $result = '{"error_code":0,"error_msg":"SUCCESS","log_id":3575942510189,"timestamp":1608174532,"cached":0,"result":{"score":91.55084229,"face_list":[{"face_token":"68006dd55d9b84a57532f9ae0b876f9a"},{"face_token":"cc8754354c091eb5bde8e43ae22eeee4"}]}}';
                // $result = json_decode($result,true);
                if ($result['error_code'] == 0) {
                    $score = round($result['result']['score'], 2);
                    //插入记录
                    FaceContrastModel::insert(['live_image' => $result['result']['face_list']['0']['face_token'], 'card_image' => $result['result']['face_list']['1']['face_token'], 'sorce' => $score]);
                } else {
                    $score = 0;
                }
                return $score;
            }
        }
