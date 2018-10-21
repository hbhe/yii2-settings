Introduction
============

可能是最实用的网站参数模块
-------------------------

### 0. 安装

```
composer require hbhe/yii2-settings
```

### 1. 创建表

php yii migrate/up --migrationPath=@hbhe/settings/migrations

### 2. 配置(main.php)
```
'components' => [
    'ks' => [
        'class' => hbhe\settings\models\KeyStorage::class
    ],
    // ...
],

'modules' => [
    // main.php中配好此module后, 后台可通过此地址直接查看key-value列表, http://127.0.0.1/yii2-adminlte/backend/web/settings/key-storage/index (一般测试时才配置此module)
    'ks' => [
        'class' => 'hbhe\settings\Module',
    ],
    // ...
],
```
### 3. 使用方法, 代码中其它地方可以这样使用
```
$num = Yii::$app->ks->get('demo.number', 100); // 100是默认值
if ($num > 100) {
    // ...
}
```
### 4. 参考代码

controller文件
```
<?php
namespace backend\controllers;

use hbhe\settings\models\FormModel;
use Yii;

/**
 * Settings controller
 */
class SettingsController extends \yii\web\Controller
{
    /**
     * @inheritdoc
     */
    public function actions()
    {
        return [
            'error' => [
                'class' => 'yii\web\ErrorAction',
            ]
        ];
    }

    public function actionIndex()
    {
        $model = new FormModel([
            'keyStorage' => 'ks', //对就main.php中配置的ks组件, 即通过ks组件把值存进去
            'keys' => [
                'frontend.maintenance' => [
                    'label' => Yii::t('backend', 'Frontend maintenance mode'),
                    'type' => FormModel::TYPE_DROPDOWN,
                    'items' => [
                        'disabled' => Yii::t('backend', 'Disabled'),
                        'enabled' => Yii::t('backend', 'Enabled')
                    ]
                ],
                'backend.theme-skin' => [
                    'label' => Yii::t('backend', 'Backend theme'),
                    'type' => FormModel::TYPE_DROPDOWN,
                    'items' => [
                        'skin-black' => 'skin-black',
                        'skin-blue' => 'skin-blue',
                        'skin-green' => 'skin-green',
                        'skin-purple' => 'skin-purple',
                        'skin-red' => 'skin-red',
                        'skin-yellow' => 'skin-yellow'
                    ]
                ],
                'backend.layout-fixed' => [
                    'label' => Yii::t('backend', 'Fixed backend layout'),
                    'type' => FormModel::TYPE_CHECKBOX
                ],
                'backend.layout-boxed' => [
                    'label' => Yii::t('backend', 'Boxed backend layout'),
                    'type' => FormModel::TYPE_CHECKBOX
                ],
                'backend.layout-collapsed-sidebar' => [
                    'label' => Yii::t('backend', 'Backend sidebar collapsed'),
                    'type' => FormModel::TYPE_CHECKBOX
                ]
            ]
        ]);

        if ($model->load(Yii::$app->request->post()) && $model->save()) {
            Yii::$app->session->setFlash('success', '保存成功!');
            return $this->refresh();
        }

        return $this->render('index', [
            'title' => '主题设置',
            'model' => $model
        ]);
    }

    /*
     * 用法:
     * Yii::$app->ks->get('demo.number', 100)
     */
    public function actionDemo()
    {
        $model = new FormModel([
            'keyStorage' => 'ks',
            'keys' => [
                'demo.text' => [
                    'label' => '文本框',
                    'type' => FormModel::TYPE_TEXTINPUT,
                    'options' => ['maxlength' => 10],
                    'rules' => [['string', 'min' => 1, 'max' => 8]],
                ],
                'demo.number' => [
                    'label' => '数字框(1~100)',
                    'type' => FormModel::TYPE_TEXTINPUT,
                    'options' => ['maxlength' => 10],
                    'rules' => [['number', 'min' => 1, 'max' => 100]],
                ],
                'demo.dropdown' => [
                    'label' => '下拉框',
                    'type' => FormModel::TYPE_DROPDOWN,
                    'items' => [
                        'AAAA' => 'AAAA',
                        'BBBB' => 'BBBB',
                    ]
                ],
                'demo.check' => [
                    'label' => '勾选框',
                    'type' => FormModel::TYPE_CHECKBOX
                ],
                'demo.date' => [
                    'label' => '日期(格式如2019-01-01 01:01:01)',
                    'type' => FormModel::TYPE_TEXTINPUT,
                    'options' => ['maxlength' => 24,],
                    'rules' => [['date', 'format' => 'php:Y-m-d H:i:s']],
                ],
            ]
        ]);

        if ($model->load(Yii::$app->request->post()) && $model->save()) {
            Yii::$app->session->setFlash('success', '保存成功!');
            return $this->refresh();
        }
        Yii::info([
            Yii::$app->ks->get('demo.number', 0),
        ]);
        return $this->render('index', [
            'title' => 'DEMO参数设置',
            'model' => $model
        ]);
    }
}
```

view文件
```
<?php
use hbhe\settings\models\FormWidget;
$this->title = empty($title) ? Yii::t('backend', 'Application settings') : $title;
?>

<?= \yii\bootstrap\Nav::widget([
    'options' => [
        'class' => 'nav nav-tabs',
        'style' => 'margin-bottom: 15px'
    ],
    'items' => [
        [
            'label'   => '主题设置',
            'url'     => ['/settings/index'],
        ],
        [
            'label'   => 'DEMO参数',
            'url'     => ['/settings/demo'],
            'active' => Yii::$app->controller->action->id == 'demo',
        ],
    ]
]) ?>

<?php echo FormWidget::widget([
    'model' => $model,
    'formClass' => '\yii\bootstrap\ActiveForm',
    'submitText' => Yii::t('backend', 'Save'),
    'submitOptions' => ['class' => 'btn btn-primary']
]); ?>
```

 ![截图](https://github.com/hbhe/yii2-settings/blob/master/screenshot.png)