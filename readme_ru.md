RedactorUploadAction
========================

`RedactorUploadAction` — это действие предназначеное для обработки загрузки картинок и файлов на серевер из редактора.

Параметры действия
-------------------

`directory` – путь к папке для сохранения файлов, относительно корня сайта.
В случае если этот путь не фиксированый (например изменяется в зависимости от пользователя) можно
указать callback, что будет возвращать этот путь.

`validator`  – параметры для `CFileValidator`, что будут использоваться  при валидации загруженного файла.
По умолчанию нет никаких ограничений на загружаемые файли, нет даже проверки картинки ли это – поэтому,
хотя бы из соображений безопасности их обязательно нужно добавить(например как в примере ниже).


`saveCallback` – Если по каким-то причинам механизм сохранения файлов по умоччанию вас не  устраевает(например если хотите
сохранять файлы в базе, сохранять под другими именами, обрабативать картинки до сохранения, и т.д.)
вы можете указать callback для сохранения файлов. После сохранения callback должен возвращать ссылку файла на сайте и имя
по умолчанию для вставки в текст, как параметр при вызове в callback передаётся обьект `CUploadedFile`.


Использование
-------------

Добавляем действие в контроллер:

```php
'imgUpload'=>array(
    'class' => 'ext.redactor-upload-action.RedactorUploadAction',
    'directory'=>'uploads/images',
    'validator'=>array(
        'mimeTypes' => array('image/png', 'image/jpg', 'image/gif', 'image/jpeg', 'image/pjpeg'),
    )
),
'fileUpload'=>array(
    'class' => 'ext.redactor-upload-action.RedactorUploadAction',
    'directory'=>'uploads/files',
    'validator'=>array(
        'types' => 'txt, pdf, doc, docx',
    )
),
```

Добавляем параметры для виджета:

```php
$this->widget('ImperaviRedactorWidget', array(
    'selector' => '.redactor',
    'options' => array(
        'imageUpload'=>$this->createUrl('imgUpload'), // адрес действия на сервере
        'imageUploadErrorCallback'=>'js:function(obj, json){ alert(json.error); }', // функция для отображения ошибок загрузки пользователю

        'fileUpload'=>$this->createUrl('fileUpload'),
        'fileUploadErrorCallback'=>'js:function(obj, json){ alert(json.error); }',

        // Если используется проверка CSRF – добавляем соответствующие параметры к запросу
        'uploadFields'=>array(
            Yii::app()->request->csrfTokenName => Yii::app()->request->csrfToken,
        ),
    ),
));
```

Пример использования своего способа сохранения файлов
-----------------------------------------------------

Например, если вы хотите сохранять имена файлов при их аплоаде.
Сделать это можно например так:

```php
class TestController extends Controller
{
    /**
     * @param $file CUploadedFile
     * @return string[] url to uploaded file and file name to insert in redactor by default
     * @throws CException
     */
    public function save($file)
    {
        $webroot = Yii::getPathOfAlias('webroot');
        $dstDir = '/uploads/';

        if (!is_dir($webroot . $dstDir)) {
            mkdir($webroot . $dstDir, 0777, true);
        }

        $ext = $file->getExtensionName();
        $name = $file->name;
        if (strlen($ext)) $name = substr($name, 0, -1 - strlen($ext));

        for ($i = 1, $filePath = $dstDir . $name . '.' . $ext; file_exists($webroot . $filePath); $i++) {
            $filePath = $dstDir . $name . " ($i)." . $ext;
        }

        $file->saveAs($webroot . $filePath);
        return array($filePath, $file->name);
    }
    public function actions()
    {
        return array(
            'imgUpload' => array(
                'class' => 'ext.redactor-upload-action.RedactorUploadAction',
                'saveCallback' => array($this, 'save'),
                'validator' => array(
                    'mimeTypes' => array('image/png', 'image/jpg', 'image/gif', 'image/jpeg', 'image/pjpeg'),
                )
            ),
        );
    }
}
```