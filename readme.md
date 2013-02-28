RedactorUploadAction
========================

`RedactorUploadAction` — action to handle uploads in `ImperaviRedactorWidget`

Action properties
-------------------

`directory` – Path to directory where to save uploaded files(relative path from webroot)

В случае если этот путь не фиксированый (например изменяется в зависимости от пользователя) можно
указать callback, что будет возвращать этот путь.

`validator`  – params for `CFileValidator`, would be used to validate uploaded file.
By default there is no limitations for uploads – so, at least by security reasons you should specify them.


`saveCallback` – If you need to implement some other mechanism to save files(for example: store in database,
 generating file names preserving parts from original name, preprocessing before save, and so on) – You can
 add callback for this, argument passed to callback is CUploadedFile, callback should return url to file on site and
 display name by default .

Usage
-------------

Add action to controller:

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

Set widget options:

```php
$this->widget('ImperaviRedactorWidget', array(
    'selector' => '.redactor',
    'options' => array(
        'imageUpload'=>$this->createUrl('imgUpload'),
        'imageUploadErrorCallback'=>'js:function(obj, json){ alert(json.error); }', // function to show upload error to user

        'fileUpload'=>$this->createUrl('fileUpload'),
        'fileUploadErrorCallback'=>'js:function(obj, json){ alert(json.error); }',

        // if you are using CSRF protection – add following:
        'uploadFields'=>array(
            Yii::app()->request->csrfTokenName => Yii::app()->request->csrfToken,
        ),
    ),
));
```

Example, how to implement own image storage mechanism
-----------------------------------------------------

For example if you want to preserve file names when uploading.
You can do this way:

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